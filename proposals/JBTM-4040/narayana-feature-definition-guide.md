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

* Extra dependencies can be brought from experimental/tech-preview features, those dependencies might be required to integrate the artifact (e.g. narayana-jta) into Quarkus/WildFly (e.g. adding a dependency into the module.xml) for code compilation. This impact should be evaluated before integrating the new feature
