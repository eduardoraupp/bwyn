# 0019. Use Maven to Build the Frontend, Invoking npm as a Lifecycle Step

- **Status**: Accepted
- **Date**: 2026-11-01

## Context

The bwyn backend already uses Maven as its build tool and dependency manager (ADR-0018), organised as a multi-module project that reflects the bounded context and layer boundaries of the Hexagonal Architecture (ADR-0017). The frontend is a separate TypeScript/React codebase (ADRs 0005, 0015) built with Webpack and tested with Jest (ADR-0005), with architectural fitness functions enforced by ArchUnitTS (ADR-0016).

Without a unified build entry point, the frontend and backend are built by separate, uncoordinated toolchains. This makes it harder to produce a single deployable artefact, to enforce a consistent build lifecycle in CI, and to onboard developers who must learn and invoke two independent build systems. Aligning the frontend build under Maven allows both codebases to be compiled, tested, and packaged through a single `mvn` invocation, using the same lifecycle phases and the same CI pipeline configuration.

The `frontend-maven-plugin` is the standard mechanism for this integration: it downloads a project-local Node.js and npm distribution (avoiding a global Node.js installation requirement), then delegates to npm scripts for installation, linting, testing, and bundling within the appropriate Maven lifecycle phases. The resulting frontend bundle can be packaged as a Maven module and included as a dependency of the backend application module, or served as a separate artefact, depending on the deployment model.

## Decision

We will add the frontend codebase as a module in the Maven multi-module project established in ADR-0018. We will use the `frontend-maven-plugin` to manage a project-local Node.js and npm installation and to invoke npm scripts from within the Maven build lifecycle. Concretely:

- `npm install` will be bound to the `generate-resources` phase to install frontend dependencies.
- `npm test` (Jest, including ArchUnitTS fitness functions from ADR-0016) will be bound to the `test` phase.
- `npm run build` (Webpack production bundle, ADR-0005) will be bound to the `prepare-package` phase.

No global Node.js or npm installation will be required on developer machines or CI agents; the `frontend-maven-plugin` will provision the correct versions locally. The Node.js and npm versions to use will be declared explicitly in the plugin configuration and managed in the parent POM alongside other dependency versions (ADR-0018).

## Consequences

**Positive**
- A single `mvn verify` invocation builds, tests, and packages both the frontend and the backend, simplifying CI pipeline configuration and developer workflows.
- The `frontend-maven-plugin` provisions Node.js and npm locally, eliminating the need for a pre-installed Node.js on developer machines or CI agents and ensuring all contributors use the same versions.
- Frontend Jest tests (ADR-0005) and ArchUnitTS fitness functions (ADR-0016) are executed automatically as part of the standard Maven `test` phase, giving them the same visibility and failure semantics as backend JUnit and ArchUnit tests (ADR-0009).
- Centralised version management in the parent POM (ADR-0018) extends to the Node.js and npm versions, making upgrades a single-point change.
- The frontend module participates in Maven's dependency graph, making it straightforward to declare the backend application module's dependency on the packaged frontend bundle.

**Negative**
- Wrapping npm in Maven adds a layer of indirection; frontend developers accustomed to running `npm` directly must understand which Maven phase corresponds to which npm script, or continue to invoke npm directly for day-to-day development (which remains possible).
- The `frontend-maven-plugin` downloads Node.js and npm on first use; in environments without internet access, an internal mirror or pre-cached distribution must be configured.
- Maven's sequential lifecycle model does not parallelise frontend and backend builds by default; build times may be longer than running the two toolchains concurrently, unless Maven's parallel module build option (`-T`) is used.
- An additional Maven module and plugin configuration increases the size and complexity of the parent POM, requiring ongoing maintenance as Node.js, npm, and the plugin itself are upgraded.

**Neutral**
- A frontend Maven module (`pom.xml`, `frontend-maven-plugin` configuration, and npm script bindings) must be established as an architectural enabler before the unified build is used in CI; this work should be coordinated with the parent POM structure from ADR-0018.
- Frontend developers may continue to invoke `npm install`, `npm test`, and `npm run build` directly during local development; the Maven wrapper is primarily for CI and full-project builds.
- Decisions about how the packaged frontend bundle is served (embedded in the Spring Boot JAR, deployed as a separate static artefact, or served by a CDN) are outside the scope of this ADR and will be addressed in follow-up ADRs or tickets.
- Any future change of frontend package manager (e.g. from npm to pnpm or Yarn) would require updating the `frontend-maven-plugin` configuration and a corresponding ADR or ticket, but would not require superseding this ADR provided Maven remains the outer build orchestrator.
