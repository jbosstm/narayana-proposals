# LRA Coordinator High Availability Implementation

## Overview

This implementation enables multiple LRA coordinators to work together in a cluster, addressing the key requirements:

1. **Shared Object Store**: Different coordinators can share an object store by embedding node ID in LRA IDs
2. **Any Coordinator Can Manage Any LRA**: Distributed locking ensures safe concurrent access
3. **Single Recovery Manager**: JGroups coordinator election ensures only one node performs recovery

## Architecture

### Components

1. **InfinispanStore** - Distributed LRA state storage
   - Three replicated caches: active, recovering, failed
   - Automatic state replication across cluster
   - Any coordinator can access any LRA state

2. **DistributedLockManager** - Infinispan clustered locks
   - Prevents concurrent modifications to same LRA
   - Uses Infinispan's lock API (backed by JGroups)
   - Automatic lock release on node failure

3. **ClusterCoordinator** - JGroups coordinator election
   - First node in cluster view becomes coordinator
   - Automatic failover when coordinator fails
   - Only coordinator performs recovery scans

4. **LRAState** - Serializable LRA representation
   - Contains all LRA state for distributed storage
   - Includes node ID for tracking LRA origin
   - Used for Infinispan cache entries

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
   - Infinispan active cache (replicated across cluster)
4. Any coordinator can now access this LRA

### Accessing an LRA

1. Client calls any coordinator with LRA ID
2. Coordinator checks local memory first
3. If not found, loads from Infinispan cache
4. Acquires distributed lock before modifications
5. Updates are saved to both ObjectStore and Infinispan

### Ending an LRA

1. Any coordinator can close/cancel any LRA
2. Distributed lock prevents conflicts
3. State transitions are persisted atomically
4. Completed LRAs are removed from caches

### Recovery

1. Only the JGroups cluster coordinator performs recovery
2. Recovery scans both ObjectStore and Infinispan
3. Uses distributed locks to avoid conflicts
4. If coordinator fails, new coordinator takes over automatically

### Coordinator Failover

1. JGroups detects node failure
2. New cluster view is computed
3. First surviving node becomes new coordinator
4. New coordinator immediately starts recovery scan
5. In-flight LRAs continue on other nodes

## Configuration

### Enable HA Mode

```properties
lra.coordinator.ha.enabled=true
lra.coordinator.cluster.name=lra-cluster
lra.coordinator.node.id=lra-coord-1
```
