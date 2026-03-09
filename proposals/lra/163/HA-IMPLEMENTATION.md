# LRA Coordinator High Availability Implementation

## Overview

This implementation enables multiple LRA coordinators to work together in a cluster, addressing the key requirements:

1. **Shared Object Store**: Different coordinators can share an object store by embedding node ID in LRA IDs
2. **Any Coordinator Can Manage Any LRA**: Optimistic concurrency control (CAS with version numbers) ensures safe concurrent access
3. **Single Recovery Manager**: JGroups coordinator election ensures only one node performs recovery

## Architecture

### Module Structure

The HA implementation follows a **plugin architecture** with a separate, optional Maven module:

- **Core coordinator** (`coordinator/`) — defines HA interfaces (`LRAStore`, `ClusterCoordinationService`, `LRAState`) with no Infinispan dependency
- **HA Infinispan module** (`coordinator-ha-infinispan/`) — provides the Infinispan-based implementation, added as an optional dependency

This design ensures **zero overhead** when HA is disabled. Projects opt in by:
1. Adding the `lra-coordinator-ha-infinispan` Maven dependency
2. Setting `lra.coordinator.ha.enabled=true`

### Components

1. **InfinispanStore** — Distributed LRA state storage
   - Three replicated caches: `lra-active`, `lra-recovering`, `lra-failed`
   - Automatic state replication across cluster
   - Any coordinator can access any LRA state
   - Dual-write strategy: updates go to both ObjectStore (traditional Narayana persistence) and Infinispan (distributed)

2. **Optimistic Concurrency Control** — Version-based CAS (Compare-And-Swap)
   - Prevents concurrent modifications to same LRA via version numbers
   - `saveOrFail()` uses `putIfAbsent()` for new entries, `replace()` with version verification for updates
   - `StaleStateException` thrown on version conflicts, with automatic retry (up to 5 attempts)
   - No distributed lock manager needed — simpler and more performant than clustered locks

3. **InfinispanClusterCoordinator** — JGroups coordinator election
   - First node in cluster view becomes coordinator
   - Automatic failover when coordinator fails
   - Only coordinator performs recovery scans
   - Listens to `@ViewChanged` events for cluster membership changes
   - Fallback scheduler checks coordinator status every 10 seconds

4. **InfinispanLRAState** — Serializable LRA representation
   - Immutable value object with ProtoStream annotations for efficient binary serialization
   - Contains all LRA state: id, parentId, clientId, status, startTime, finishTime, timeLimit, nodeId, version
   - Stores raw serialized bytes from `LongRunningAction.save_state()` for Arjuna deserialization
   - `withVersion()` creates immutable copies with updated version

5. **InfinispanConfiguration** — CDI producer for Infinispan infrastructure
   - **WildFly subsystem mode**: JNDI lookup at `java:jboss/infinispan/container/lra`
   - **Embedded mode**: Standalone/Quarkus use self-created `DefaultCacheManager`
   - Configures caches with ProtoStream marshalling
   - Sets up JGroups transport with configurable cluster name, node name, bind address

### Node ID Embedding

LRA IDs are formatted as:
- **Single-instance mode**: `http://host:port/lra-coordinator/{uid}`
- **HA mode**: `http://host:port/lra-coordinator/{nodeId}/{uid}`

This allows:
- Multiple coordinators to use the same object store without conflicts
- Easy identification of which node created an LRA
- Single recovery manager to recover LRAs from all nodes

## How It Works

### Starting an LRA

1. Client calls `POST /lra-coordinator/start`
2. Coordinator creates LRA with node ID embedded in URI
3. LRA state is saved to:
   - Local ObjectStore (traditional Narayana persistence)
   - Infinispan `lra-active` cache (replicated across cluster)
4. Any coordinator can now access this LRA

### Accessing an LRA

1. Client calls any coordinator with LRA ID
2. Coordinator checks local memory first
3. If not found, loads from Infinispan cache (checks `lra-active`, `lra-recovering`, `lra-failed` in order)
4. Uses optimistic concurrency control (version-based CAS) for modifications
5. Updates are saved to both ObjectStore and Infinispan (dual-write)

### Ending an LRA

1. Any coordinator can close/cancel any LRA
2. Version-based CAS prevents conflicts (retries on `StaleStateException`)
3. State transitions move LRAs between caches (e.g., `lra-active` to `lra-recovering`)
4. Cross-cache moves are atomic: remove-then-put with fallback
5. Completed LRAs are removed from caches

### Recovery

1. Only the JGroups cluster coordinator performs recovery
2. Recovery scans Infinispan caches for LRAs in recovering/failed states
3. Uses version-based CAS to avoid conflicts with other coordinators
4. If coordinator fails, new coordinator takes over automatically

### Coordinator Failover

1. JGroups detects node failure via `@ViewChanged` events
2. New cluster view is computed
3. First surviving node becomes new coordinator
4. New coordinator immediately starts recovery scan
5. In-flight LRAs continue on other nodes (state is in Infinispan)

## Configuration

### Enable HA Mode

```properties
lra.coordinator.ha.enabled=true
lra.coordinator.cluster.name=lra-cluster
lra.coordinator.node.id=lra-coord-1
lra.coordinator.jgroups.config=jgroups-tcp.xml
lra.coordinator.jgroups.bind_addr=0.0.0.0
lra.coordinator.infinispan.persistent.location=/var/lra/state
lra.coordinator.infinispan.cache.mode=REPL_SYNC
```
