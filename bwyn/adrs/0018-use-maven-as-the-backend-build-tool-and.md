# 0018. Use Maven as the Backend Build Tool and Dependency Manager

- **Status**: Accepted
- **Date**: 2026-11-01

## Context

The bwyn backend is built on Spring Boot (ADR-0008) and organised around bounded contexts using Hexagonal Architecture and DDD (ADR-0017). As the codebase grows across multiple layers and bounded contexts, a build tool is needed to compile Java source code, manage third-party dependencies, run the test suite (including ArchUnit fitness functions from ADR-0009 and Testcontainers-based integration tests from ADR-0012), and produce deployable artefacts. Without a standardised build tool, dependency versions can diverge between modules, build steps become ad hoc, and onboarding new developers is inconsistent.

Maven is a widely adopted, convention-driven build tool for the JVM ecosystem. It manages dependencies through a declarative `pom.xml` descriptor, resolves transitive dependencies from a central repository (Maven Central), and provides a standard lifecycle (compile, test, package, verify, install, deploy) that integrates directly with Spring Boot's Maven plugin and with the annotation processors required by Lombok (ADR-0013). Its multi-module support allows the codebase to be structured into discrete modules that reflect bounded context and layer boundaries (ADR-0017), with dependency relationships between modules declared explicitly.

## Decision

We will use Maven as the build tool and dependency manager for the bwyn backend. All Java dependencies will be declared in `pom.xml` files and resolved from Maven Central (or an approved internal mirror). We will use a multi-module Maven project structure to reflect the bounded context and layer boundaries established in ADR-0017, with a parent POM managing shared dependency versions and plugin configuration. The Spring Boot Maven plugin will be used to produce executable application artefacts. Annotation processor dependencies (e.g. Lombok from ADR-0013) will be declared in the correct Maven scope (`annotationProcessor` / `provided`) to ensure they are not included in the runtime classpath.

## Consequences

**Positive**
- Maven's standard lifecycle integrates directly with Spring Boot's Maven plugin (ADR-0008), producing executable JARs or WARs without custom build scripting.
- Centralised dependency version management in the parent POM prevents version drift across modules and makes upgrades (e.g. Spring Boot, Flyway, Testcontainers) a single-point change.
- Multi-module structure enforces bounded context and layer separation at the build level, complementing the ArchUnit rules (ADR-0009) that enforce it at the code level.
- Maven's widespread adoption in the Spring/Java ecosystem means extensive documentation, IDE support, and developer familiarity are readily available.
- Annotation processor scoping ensures Lombok (ADR-0013) and any future processors (e.g. MapStruct) are correctly isolated from the runtime classpath.

**Negative**
- Maven's XML-based `pom.xml` is verbose; as the number of modules grows, the parent POM and module descriptors require ongoing maintenance.
- Maven builds can be slower than alternatives (e.g. Gradle) for large multi-module projects without deliberate use of incremental build and local repository caching.
- Multi-module project configuration (parent POM inheritance, dependency management sections, inter-module dependencies) adds initial setup complexity that must be understood by all contributors.

**Neutral**
- A parent POM and an initial multi-module layout covering the domain core, application layer, inbound adapters, and outbound adapters (ADR-0017) must be established as an architectural enabler before feature development begins; this work should be coordinated with the baseline project structure from ADR-0008.
- The Maven Surefire and Failsafe plugin configuration must be reviewed to ensure that ArchUnit tests (ADR-0009) and Testcontainers integration tests (ADR-0012) are executed in the correct build phase and with appropriate JVM settings (e.g. `--add-opens` flags if required by the Java version in use).
- Flyway migration scripts (ADR-0010) are not compiled artefacts and will reside in `src/main/resources`; no special Maven configuration is required for them beyond standard resource filtering.
- Any future decision to adopt an alternative build tool (e.g. Gradle) would require a superseding ADR and migration of all `pom.xml` descriptors.
