# **Narayana Feature Stage Definition and Implementation Guide**

This document specifies the definitions and mandatory implementation standards for all features in the codebase, classifying them into three distinct stages: Experimental, Tech Preview, and Stable.

This guide is intended to ensure clarity, consistency, and traceability regarding the stability and readiness of features, particularly emphasizing the constraints on production usage for non-stable features.

# **1. Feature Classification Definitions**

All features must be explicitly categorized based on their current stage of development and stability.

| Stage | Definition | Stability |
| :---- | :---- | :---- |
| Experimental | Early-stage development, high chance of significant change or removal. Only for internal testing and initial feedback. | Very Low |
| Tech Preview | Feature is stable enough for external use and testing, but may still have breaking changes. Requires explicit opt-in. | Moderate |
| Stable | Feature is stable, well-tested, and ready for general availability. | High |

**CRITICAL RULE: Experimental and Tech Preview features MUST NOT be used in a production environment.**

# **2. Implementation Requirements by Stage (Java Codebase)**

The following rules dictate the mandatory code and documentation standards for each feature stage within the Java codebase.

# **2.1. Experimental Features**

Experimental features MUST be guarded by a unique, compile-time flag and contain explicit warnings in the Javadoc and initialization logic.

Experimental features are expected to have a "Sponsor". That person is expected to commit to maintaining the feature to remain compatible with the Narayana branch it is available from, for as long as the feature is in the branch at this definition level.

## **Javadoc and Initialization Warnings**

Any Java class or method that is part of an Experimental feature MUST include a warning in its Javadoc and an operational log warning upon initialization. 

| Component | Requirement | Example |
| :---- | :---- | :---- |
| **Annotation** | SHOULD be annotated with `@Experimental`. The annotation SHOULD log a `WARN` message upon first use or class instantiation. The implementation of this annotation is subject to the outcome of this proposal and is tracked separately in [JBTM-4041](https://issues.redhat.com/browse/JBTM-4041). | `@Experimental(String)` |
| **Javadoc** | MUST include a clear warning about its Experimental status and potential for breaking changes. | `NOTE: This is an Experimental feature (EXP_FEATURE_NAME). It is not recommended for production systems and may contain breaking changes in future releases.` |
| **Manuals** | The feature must have (at least) minimal documentation that clearly indicates the feature is experimental. The feature should be documented in the intended final place in the manual. The manuals already have a way to mark warnings ([notes and warnings](https://jbosstm.github.io/public/public//docs/project/index.html#_notes_and_warnings)) this should be used. | `WARNING: This is an experimental feature. It is not recommended for production systems and may contain breaking changes in future releases.` |
| **Issue tracking** | The JIRA or GitHub issue must add the label `experimental` | |
| **Code Guard** | Consider using a flag in order to use the experimental feature | `if(!experimental){ Throws exception }` |
| **Tracking** | Link to the internal design document. | // Internal Design: File |
| **Dependency Isolation** | If the feature introduces new dependencies, it MUST follow the dependency isolation model (see Section 6). | Separate Maven module or `<optional>true</optional>` |

# **2.2. Tech Preview Features**

Tech Preview features MUST be guarded by a runtime configuration and include deprecation warnings, as they are not for production use.

## **Javadoc and Initialization Warnings**

Any Java class or method that is part of a Tech Preview feature MUST include a warning in its Javadoc and an operational log warning upon initialization.

| Component | Requirement | Example |
| :---- | :---- | :---- |
| **Javadoc** | MUST include a clear warning about its Tech Preview status and potential for breaking changes. | `NOTE: This is a Tech Preview feature (TP_FEATURE_NAME). It is not recommended for production systems and may contain breaking changes in future releases.` |
| **Annotation** | SHOULD be annotated with `@TechPreview`. The annotation SHOULD log a `WARN` message upon first use or class instantiation. The implementation of this annotation is subject to the outcome of this proposal and is tracked separately in [JBTM-4041](https://issues.redhat.com/browse/JBTM-4041). | `@TechPreview(String)` |
| **Manuals** | The feature must have (at least) minimal documentation that clearly indicates the feature is tech preview. The feature should be documented in the intended final place in the manual. The manuals already have a way to mark warnings ([notes and warnings](https://jbosstm.github.io/public/public//docs/project/index.html#_notes_and_warnings)) this should be used. | `WARNING: This is a Tech Preview feature. It is not recommended for production systems and may contain breaking changes in future releases.` |
| **Issue tracking** | The JIRA or GitHub issue must add the label `tech-preview` | |
| **Code Guard** | MUST be protected by a runtime configuration check, defaulting to disabled. | `if (Configuration.isTechPreviewEnabled("TP_FEATURE_NAME"))` |
| **Promotion Plan** | Issue number to refer to for tracking the features promotion from "Tech Preview" to Stable. | |
| **Dependency Isolation** | If the feature introduces new dependencies, it MUST follow the dependency isolation model (see Section 6). | Separate Maven module or `<optional>true</optional>` |

# **2.3. Stable Features**

Stable features should be implemented directly consistent with our existing policies, for example [support guarantees](https://github.com/jbosstm/narayana?tab=readme-ov-file#support-guarantees)  
Stable features are well-tested, and ready for general availability.

# **3. Transition Process Summary**

The transition between stages requires approval from the team (discussion can happen on zulip,slack or github). For all transitions, ensure compliance with the target stage's implementation requirements before the change is submitted for review.

# **4. Deprecation/Removal process**

Requirements: 

* Deprecation happens before removal

Possible reasons that lead to the deprecation/removal choice:

* When 2 tech preview features overlap and maintaining both is a big effort  
* Security concerns arise from a tech preview feature and are not addressed within a certain amount of time  
* A feature is impacting the project performance  
* Experimental feature does not appear to have an active "sponsor"  

Decision making:

* Advertise to the community that the feature is proposed for removal on Zulip. These discussions should be open for feedback and volunteers to maintain the experimental feature  
* Use a manner compatible with our decision making process: https://github.com/wildfly/wildfly-governance/blob/main/narayana/GOVERNANCE.md#decision-making
* When a decision is made announce it to the narayana-user mail group and in the release note of the next release

Extra considerations:

* For dependency isolation requirements related to experimental/tech-preview features, see Section 6.

# **5. Prior Art: Feature Staging in WildFly and Quarkus**

Feature-stage definitions are not a new problem. WildFly and Quarkus, the two primary consumers of Narayana, have both established models. Understanding their approaches helps Narayana choose a strategy that integrates well with both ecosystems.

## **5.1. WildFly Stability Levels**

WildFly defines four stability levels, governed by the [Feature Development Process](https://docs.wildfly.org/wildfly-proposals/FEATURE_PROCESS.html):

| Level | Description |
| :---- | :---- |
| **Experimental** | Bleeding-edge functionality that may never advance. No distribution enables it by default. |
| **Preview** | Sufficiently stable for WildFly Preview; generally expected to promote to Community eventually (not guaranteed). |
| **Community** | Available by default in standard WildFly. Not expected to change incompatibly. |
| **Default** | Additional vetting for long-term compatibility guarantees. |

**Isolation mechanism:** WildFly enforces isolation at the **packaging/provisioning layer** via separate Galleon feature packs (`wildfly-ee`, `wildfly`, `wildfly-preview`). Each feature pack carries stability-level metadata. Features below the active stability threshold are excluded from provisioning entirely -- they are not present on disk unless explicitly requested.

**Opt-in:** Two complementary paths:
- **Runtime:** The `--stability` CLI flag (`experimental`, `preview`, `community`, `default`) controls what features are available at server start. Servers can also be reloaded to a different stability level without restart.
- **Provisioning-time:** The Galleon CLI/Maven plugin accepts `--stability-level` to control what gets installed.

**Intra-module isolation:** WildFly also supports **per-attribute and per-resource stability** within a single subsystem. Individual management model attributes and resources carry a `Stability` level (set via `AttributeDefinition.Builder.setStability(Stability.PREVIEW)`). At server boot, the management model registration filters out any attribute or resource whose stability level is below the server's active `--stability` threshold. Non-stable items are simply never registered — they are invisible to the CLI, admin console, and all management operations. This means a stable subsystem (e.g., Undertow) can contain preview-level attributes that only appear when the server runs at `--stability=preview` or lower.

**Key takeaway:** WildFly's approach is **artifact-driven** at the feature-pack level and **framework-enforced** at the intra-module level. Both layers use the same `Stability` enum, so the model is consistent from provisioning down to individual attributes.

References: [Different Flavors of WildFly](http://docs.wildfly.org/40/Different_Flavors_of_WildFly.html), [Stability in Provisioning (WFLY-19021)](https://docs.wildfly.org/wildfly-proposals/wf-galleon/WFLY-19021-Stability_In_Provisioning.html)

## **5.2. Quarkus Extension Maturity Model**

Quarkus defines three maturity levels, declared via the `status` field in each extension's `META-INF/quarkus-extension.yaml`:

| Level | Description |
| :---- | :---- |
| **Experimental** | Earliest stage; significant API/config changes expected. |
| **Preview** | Backward compatibility and continued presence not guaranteed. Plans to become stable are underway. |
| **Stable** | Production-ready; backward compatibility guaranteed. |

**Isolation mechanism:** Quarkus does **not** separate stable and non-stable extensions into different Maven artifacts or BOMs. All platform extensions (regardless of status) are included in `io.quarkus.platform:quarkus-bom`. Isolation is **metadata-driven**: the `status` field is surfaced by tooling (`quarkus ext list`, the extensions page on quarkus.io) but not enforced at the dependency level.

**Opt-in:** Per-extension build-time configuration properties (e.g., `quarkus.<extension>.<feature>.enabled=true`). There is no framework-level "experimental mode" toggle; each extension manages its own opt-in independently.

**Intra-extension isolation:** Quarkus does **not** have a per-config-property stability annotation. When experimental features exist within a stable extension (e.g., OpenTelemetry metrics within the stable OpenTelemetry extension), the feature is simply **disabled by default** via a config property and documentation marks it as experimental. There is no framework-level enforcement — it is a convention based on default values.

**Key takeaway:** Quarkus's approach is **metadata-driven** at the extension level and **convention-based** at the intra-extension level. Maturity is informational and surfaced by tooling, but all extensions share the same dependency graph. Intra-extension experimental features rely on disabled-by-default config properties.

References: [Extension Metadata Guide](https://quarkus.io/guides/extension-metadata), [Platform Guide](https://quarkus.io/guides/platform)

## **5.3. Comparison**

| Aspect | WildFly | Quarkus |
| :---- | :---- | :---- |
| **Isolation granularity** | Artifact-level (separate feature packs) | Metadata-level (status field in YAML) |
| **Opt-in mechanism** | `--stability` flag + provisioning control | Per-extension config properties |
| **Non-stable code on disk** | Only if consumer opts in | Always present |
| **Dependency impact** | Non-stable features cannot add transitive deps to stable installations | All extensions share the BOM and dependency graph |
| **Promotion cost** | Feature moves between feature packs (packaging change) | Status field update in YAML (metadata change) |
| **Intra-module experimental code** | Framework-enforced: per-attribute `Stability` level, filtered at model registration | Convention-based: disabled-by-default config property, no framework enforcement |

# **6. Dependency Isolation Model**

## **6.1. Guiding Principle**

A consumer that depends only on stable Narayana artifacts MUST NOT transitively pull in dependencies that exist solely to support Experimental or Tech Preview features.

This is critical for both WildFly and Quarkus integration. In WildFly, extra dependencies must be added to `module.xml` for classloading. In Quarkus, extra dependencies increase build time and application image size. In both cases, pulling in dependencies for features the consumer did not ask for is unacceptable.

## **6.2. Isolation Mechanisms**

The following mechanisms are available, applied based on the scope and dependency footprint of the feature:

| Mechanism | When to use | Example |
| :---- | :---- | :---- |
| **Separate Maven module** | Feature introduces new external dependencies (Infinispan, gRPC, etc.) or is large enough to warrant its own artifact. This is the **preferred** approach for features with a significant dependency footprint. | `coordinator` (stable, no Infinispan dep) vs `coordinator-ha-infinispan` (tech-preview, adds Infinispan) |
| **Optional Maven dependency** | Feature adds a small integration point with an existing dependency that is only activated at runtime if the dependency is present on the classpath. | `<optional>true</optional>` in the POM; consumers add the dependency themselves if they want the feature |
| **Same-module with config guard** | Feature uses no additional dependencies beyond what the module already has. No Maven-level isolation is needed; the feature is guarded by a config property that defaults to disabled, following the same-module pattern described in Section 6.4. | `narayana.experimental.<feature>.enabled=false` (default); code path throws or is skipped unless explicitly enabled |

### Decision Flowchart

When adding a new Experimental or Tech Preview feature, follow this decision process:

1. **Does the feature introduce new external dependencies?**
   - **Yes, significant (new library/framework):** Create a **separate Maven module**.
   - **Yes, minor (small utility already common in the ecosystem):** Use **`<optional>true</optional>`** on the dependency and guard activation with a classpath check.
   - **No:** Use the **same-module pattern with a config guard** (see Section 6.4).

2. **For separate modules:**
   - The core module MUST NOT have any compile-time dependency on the new module.
   - The new module depends on the core module, not the other way around.
   - The new module's POM MUST document the feature's stability stage in its `<description>` element.
   - Integration is opt-in: consumers add the new module as a Maven dependency to activate the feature.
   - Define an SPI or interface in the core module that the feature module implements, so the core remains decoupled from the feature's dependencies (e.g., `ClusterCoordinationService` in the coordinator module).

3. **For optional dependencies:**
   - The code MUST gracefully handle the absence of the optional dependency (e.g., `ClassNotFoundException` check, `ServiceLoader` with no provider).
   - Javadoc MUST document which optional dependency is required and at what version.

## **6.3. BOM Guidance**

If Narayana publishes a BOM (`narayana-bom`):
- Non-stable modules SHOULD be included in the BOM for version alignment purposes.
- The BOM only manages versions; consumers choose which artifacts to depend on.
- The BOM's documentation or comments SHOULD clearly indicate which managed artifacts are Experimental or Tech Preview.

## **6.4. Same-Module Pattern: Experimental Code in a Stable Module**

When a new experimental or tech-preview feature is added to an existing stable module (i.e., no separate Maven module), the question is: how do you prevent the non-stable code path from being active by default, and how do you give users control?

Three mechanisms are involved. Each serves a distinct purpose:

| Mechanism | Purpose | Gives user control? |
| :---- | :---- | :---- |
| **Configuration property** | Opt-in gate — the user explicitly chooses to enable the feature | **Yes** |
| **`@Experimental` / `@TechPreview` annotation** | Source-level documentation — marks the code as non-stable for developers and tooling | No (by itself) |
| **WARN log** | Operational visibility — informs the user that a non-stable feature is active | No |

**The configuration property is the control mechanism.** The annotation and warning are complementary, not alternatives.

### Recommended Pattern

```java
@Experimental("Feature_ABC")
public class FeatureABC implements FeatureInterface {

    public FeatureABC(Configuration config) {
        if (!config.isExperimentalEnabled("Feature_ABC")) {
            throw new IllegalStateException(
                "Feature Feature_ABC is experimental and disabled by default. "
                + "Set narayana.experimental.feature_abc.enabled=true to enable it.");
        }
        logger.warn("ARJUNA-XXXX: Experimental feature Feature_ABC is enabled. "
            + "Not recommended for production use.");
    }
}
```

**How the three mechanisms work together:**

1. **Config property** (`narayana.experimental.<feature>.enabled`, default `false`): This is the gate. The feature is unreachable unless the user explicitly sets it to `true`. This addresses the concern that warnings alone don't give users control — they do not; the config property does.

2. **`@Experimental` annotation**: A lightweight marker (no AOP, no proxies, no bytecode manipulation). It serves two purposes: (a) clearly documents the stability stage in source code for developers and reviewers, and (b) can be used by tooling or test infrastructure to locate all experimental code paths. The annotation does NOT need to be the enable/disable mechanism — that is the config property's job.

3. **WARN log**: Emitted **only when the feature is activated** (i.e., after the config check passes). This ensures operational visibility in server logs without spamming users who never enabled the feature.

### Why This Works for Narayana

- **Simpler than WildFly's approach.** Narayana is a library, not an application server with a management model. Building a framework-level per-attribute stability filtering system would be overengineering. The config-property approach is the pattern Quarkus uses for sub-features within stable extensions, and it is proven to work.
- **Annotations remain lightweight.** The annotation is a marker, not an interceptor. It adds no runtime overhead. The complexity concern raised about annotations applies to annotations-as-gates (AOP, proxies); it does not apply to annotations-as-markers.
- **Users have explicit control.** The feature is invisible (throws on use) unless the user opts in via configuration. This is stricter than warnings alone and achieves the goal without the complexity of annotation-based enforcement.

### SPI Pattern for Larger Same-Module Features

For features that span multiple classes within the same module, define an SPI interface in the stable code and have the experimental code implement it. The stable code discovers the implementation via `ServiceLoader` and only activates it if the config property is enabled:

```java
// Stable code — SPI interface
public interface FeatureInterface { ... }

// Stable code — discovery with config guard
ClusterCoordinationService service = config.isExperimentalEnabled("Feature_ABC")
    ? ServiceLoader.load(FeatureInterface.class)
        .findFirst()
        .orElseThrow(() -> new IllegalStateException("Feature_ABC enabled but no implementation found"))
    : new NoOpFeature();
```

This keeps the stable code path completely decoupled from the experimental implementation, even within the same module.

## **6.5. Impact Assessment for WildFly and Quarkus Integration**

Before merging a new Experimental or Tech Preview feature, the author MUST evaluate and document:
- **WildFly:** Does the feature require new dependencies in the WildFly feature pack's `module.xml`? If the dependency can be marked optional (using `optional="true"` in `module.xml`) without breaking existing stable features, the same-module pattern with `<optional>true</optional>` in the POM is acceptable. If the dependency must be mandatory (i.e., required for existing stable features to function), the feature MUST be in a separate Maven module to avoid forcing the dependency on consumers who don't use the feature.
- **Quarkus:** Does the feature require a new Quarkus extension or changes to an existing extension's deployment module? If so, document the integration path and ensure the Quarkus extension's `quarkus-extension.yaml` reflects the correct maturity status.
