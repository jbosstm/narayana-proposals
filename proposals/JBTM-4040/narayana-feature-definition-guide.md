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

Experimental features MUST be implemented in separate Maven modules to enforce opt-in dependency management. This ensures that users explicitly choose to include experimental code, and that stable artifacts have zero transitive dependencies on experimental features.

Experimental features are expected to have a "Sponsor". That person is expected to commit to maintaining the feature to remain compatible with the Narayana branch it is available from, for as long as the feature is in the branch at this definition level.

## **Module Structure and Documentation Requirements**

| Component | Requirement | Example |
| :---- | :---- | :---- |
| **Maven Module** | MUST be in a separate module under an `experimental/` directory. The module name SHOULD include `-experimental` suffix for clarity. | `experimental/coordinator-ha-infinispan-experimental/` |
| **Module POM** | MUST document the experimental status in the `<description>` element and include a WARNING comment at the top. | `<!-- WARNING: EXPERIMENTAL FEATURE - Not for production use -->` |
| **Javadoc** | MUST include a clear warning about its Experimental status and potential for breaking changes in package-info.java or main classes. | `NOTE: This is an Experimental feature (EXP_FEATURE_NAME). It is not recommended for production systems and may contain breaking changes in future releases.` |
| **Manuals** | The feature must have (at least) minimal documentation that clearly indicates the feature is experimental. The feature should be documented in the intended final place in the manual. The manuals already have a way to mark warnings ([notes and warnings](https://jbosstm.github.io/public/public//docs/project/index.html#_notes_and_warnings)) this should be used. | `WARNING: This is an experimental feature. It is not recommended for production systems and may contain breaking changes in future releases.` |
| **Issue tracking** | The JIRA or GitHub issue must add the label `experimental` | |
| **Initialization Warning** | SHOULD log a `WARN` message upon module initialization to alert users at runtime. | `WARN: Experimental feature Feature_ABC loaded. Not recommended for production use.` |
| **Tracking** | Link to the internal design document in module README or package-info.java. | // Internal Design: [link to design doc] |
| **Core Module Decoupling** | The core module MUST NOT have any compile-time dependency on the experimental module. Use SPI/interface pattern in core if integration is needed. | Core defines `FeatureInterface`, experimental module provides implementation |

**Note on Opt-In Mechanism:**
The Maven module dependency serves as the primary opt-in mechanism for experimental features. Users must explicitly add the experimental module as a dependency, and then explicitly configure the feature (via ServiceLoader, properties, or API). Additional runtime enable/disable properties are typically NOT needed unless the feature has automatic initialization with side effects. The combination of (1) explicit dependency, (2) explicit configuration, and (3) runtime warning provides sufficient gating without redundant configuration layers.

# **2.2. Tech Preview Features**

Tech Preview features MUST be implemented in separate Maven modules to enforce opt-in dependency management. This ensures users explicitly choose to include tech preview code, and that stable artifacts have zero transitive dependencies on tech preview features.

## **Module Structure and Documentation Requirements**

| Component | Requirement | Example |
| :---- | :---- | :---- |
| **Maven Module** | MUST be in a separate module under a `tech-preview/` directory. The module name SHOULD include `-tp` or `-preview` suffix for clarity. | `tech-preview/coordinator-feature-tp/` |
| **Module POM** | MUST document the tech preview status in the `<description>` element and include a WARNING comment at the top. | `<!-- WARNING: TECH PREVIEW FEATURE - Not for production use -->` |
| **Javadoc** | MUST include a clear warning about its Tech Preview status and potential for breaking changes in package-info.java or main classes. | `NOTE: This is a Tech Preview feature (TP_FEATURE_NAME). It is not recommended for production systems and may contain breaking changes in future releases.` |
| **Manuals** | The feature must have (at least) minimal documentation that clearly indicates the feature is tech preview. The feature should be documented in the intended final place in the manual. The manuals already have a way to mark warnings ([notes and warnings](https://jbosstm.github.io/public/public//docs/project/index.html#_notes_and_warnings)) this should be used. | `WARNING: This is a Tech Preview feature. It is not recommended for production systems and may contain breaking changes in future releases.` |
| **Issue tracking** | The JIRA or GitHub issue must add the label `tech-preview` | |
| **Initialization Warning** | SHOULD log a `WARN` message upon module initialization to alert users at runtime. | `WARN: Tech Preview feature TP_FEATURE_NAME loaded. Not recommended for production use.` |
| **Runtime Configuration** | MAY use runtime configuration to enable/disable specific sub-features within the tech preview module. | `if (Configuration.isTechPreviewEnabled("TP_FEATURE_NAME"))` |
| **Promotion Plan** | Issue number to refer to for tracking the features promotion from "Tech Preview" to Stable. | |
| **Core Module Decoupling** | The core module MUST NOT have any compile-time dependency on the tech preview module. Use SPI/interface pattern in core if integration is needed. | Core defines `FeatureInterface`, tech preview module provides implementation |

**Note on Opt-In and Runtime Configuration:**
Like experimental features, the Maven module dependency is the primary opt-in mechanism. Runtime configuration (row "Runtime Configuration" above) is optional and typically only needed for enabling/disabling sub-features within the tech preview module, not for enabling the module itself. The combination of (1) explicit dependency and (2) explicit feature configuration provides the necessary gating. See the Experimental Feature Implementation Plan for guidance on when additional runtime properties are appropriate.

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

## **6.2. Module-Based Isolation (Primary Approach)**

All Experimental and Tech Preview features MUST be implemented in separate Maven modules under dedicated directories:

- **Experimental features:** `experimental/<module-name>-experimental/`
- **Tech Preview features:** `tech-preview/<module-name>-tp/`

### **Module Structure Requirements:**

1. **Directory Organization:**
   - Core modules remain in their current locations (e.g., `ArjunaJTA/`, `ArjunaCore/`, `rts/lra/`)
   - Experimental modules go under `experimental/` at the repository root
   - Tech Preview modules go under `tech-preview/` at the repository root

2. **Dependency Direction:**
   - The core module MUST NOT have any compile-time dependency on experimental/tech-preview modules
   - Experimental/tech-preview modules depend on core modules (one-way dependency)
   - Define SPIs or interfaces in the core module that experimental/tech-preview modules implement

3. **POM Requirements:**
   - Module POM MUST include WARNING comments at the top indicating stability level
   - The `<description>` element MUST document the feature's stability stage
   - Module MUST NOT be included in the default reactor build profile (see Section 6.3)

4. **Integration Pattern:**
   - Consumers opt-in by adding the experimental/tech-preview module as an explicit Maven dependency
   - Use ServiceLoader pattern for loose coupling: core defines interface, experimental module provides implementation


## **6.3. Build Profile Configuration**

Experimental and Tech Preview modules MUST NOT be included in the default Maven reactor build. This ensures that:
- Standard builds do not compile non-stable code
- CI/CD pipelines can choose whether to include non-stable features
- Release artifacts by default exclude experimental/tech-preview modules

### **Profile Configuration:**

```xml
<!-- In parent pom.xml -->
<profiles>
    <profile>
        <id>experimental</id>
        <modules>
            <module>experimental/coordinator-ha-infinispan-experimental</module>
            <!-- other experimental modules -->
        </modules>
    </profile>
    <profile>
        <id>tech-preview</id>
        <modules>
            <module>tech-preview/feature-xyz-tp</module>
            <!-- other tech-preview modules -->
        </modules>
    </profile>
    <profile>
        <id>all-features</id>
        <modules>
            <!-- All experimental modules -->
            <!-- All tech-preview modules -->
        </modules>
    </profile>
</profiles>
```

To build with experimental or tech-preview features:
```bash
mvn clean install -Pexperimental
mvn clean install -Ptech-preview
mvn clean install -Pall-features
```

## **6.4. BOM and Version Management**

If Narayana publishes a BOM (`narayana-bom`):
- Non-stable modules SHOULD be included in the BOM for version alignment purposes
- The BOM only manages versions; consumers choose which artifacts to depend on
- The BOM's documentation or comments MUST clearly indicate which managed artifacts are Experimental or Tech Preview
- Consider separate BOM sections or comments delineating stability levels:

```xml
<!-- Stable Modules -->
<dependency>
    <groupId>org.jboss.narayana</groupId>
    <artifactId>coordinator</artifactId>
    <version>${narayana.version}</version>
</dependency>

<!-- Tech Preview Modules -->
<!-- WARNING: Not for production use -->
<dependency>
    <groupId>org.jboss.narayana.tech-preview</groupId>
    <artifactId>example-feature-tp</artifactId>
    <version>${narayana.version}</version>
</dependency>

<!-- Experimental Modules -->
<!-- WARNING: EXPERIMENTAL - Subject to breaking changes or removal -->
<dependency>
    <groupId>org.jboss.narayana.experimental</groupId>
    <artifactId>example-experimental</artifactId>
    <version>${narayana.version}</version>
</dependency>
```

## **6.5. Impact Assessment for WildFly and Quarkus Integration**

Before merging a new Experimental or Tech Preview feature, the author MUST evaluate and document downstream integration impact:

### **WildFly Integration:**

Since experimental/tech-preview features are in separate Maven modules:
- The feature's module.xml will be separate from stable Narayana modules
- The feature can be included in WildFly via an optional Galleon feature-pack layer
- Dependencies specific to the experimental/tech-preview feature remain isolated in its own module.xml
- Stable WildFly installations have zero knowledge of the experimental/tech-preview code
- For WildFly stability levels integration, the feature should map:
  - Experimental → WildFly `experimental` stability level
  - Tech Preview → WildFly `preview` stability level
  
Document:
- Required module.xml dependencies
- Suggested Galleon feature-pack layer name
- WildFly stability level mapping

### **Quarkus Integration:**

Since experimental/tech-preview features are in separate Maven modules:
- Each feature should have a corresponding Quarkus extension with appropriate maturity status
- The extension's `quarkus-extension.yaml` MUST reflect the correct status field:
  - Experimental → `status: experimental`
  - Tech Preview → `status: preview`
- The extension should NOT be included in the default platform BOM, or clearly marked with warnings if included
- Guide metadata should indicate the feature is not production-ready

Document:
- Quarkus extension name and groupId/artifactId
- Maturity status in `quarkus-extension.yaml`
- Whether the extension should be in platform BOM or separate
- Build-time configuration properties for opt-in (if applicable)

