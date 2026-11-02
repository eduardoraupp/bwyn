# AE-0005. Hexagonal Architecture and DDD Package Structure for the Backend

- **Status**: Proposed
- **Created**: 2026-10-28
- **Completed**:
- **Implements ADR(s)**: ADR-0017 Use Hexagonal Architecture with Domain-Driven Design for the Backend

## Goal
Establish the reference package structure that separates the domain core, application layer, inbound adapters, and outbound adapters for each bounded context, so that feature stories have unambiguous, enforced homes for every class they introduce. This enabler must exist before any feature development begins, as ADR-0017 requires the structure and its ArchUnit rules to be in place from the outset.

## Scope

**In scope**
- Four Maven modules reflecting the hexagonal layers (ADR-0017, ADR-0018):
  - `bwyn-core` — domain core: entities, value objects, aggregates, domain services, domain events, and repository port interfaces. No dependency on Spring, JPA, or any other framework.
  - `bwyn-application` — application layer: application services that orchestrate use cases by coordinating domain objects and invoking ports. Depends on `bwyn-core` only.
  - `bwyn-rest` — inbound adapter: Spring MVC controllers and any other inbound driving adapters. Depends on `bwyn-application` and `bwyn-core`.
  - `bwyn-infrastructure` — outbound adapters: Spring Data JPA repository implementations, external HTTP clients, and other driven infrastructure. Depends on `bwyn-application` and `bwyn-core`.
- A reference package layout under a placeholder bounded context in each module (e.g. `com.bwyn.<context>.domain`, `com.bwyn.<context>.application`, `com.bwyn.<context>.adapter.in`, `com.bwyn.<context>.adapter.out`), with a representative empty class in each layer.
- Parent POM updated to declare all four modules and manage their inter-module dependency declarations (ADR-0018).
- A README or inline documentation that names each layer, states what belongs in it, and states what dependencies it may have.
- ArchUnit rules (ADR-0009) enforcing:
  - The domain core has no dependency on Spring, JPA, or any adapter-layer class.
  - The application layer depends on the domain core only, not on any adapter-layer class.
  - Inbound and outbound adapters depend only on ports defined in the domain or application layer.
- Adapter integration tests use the `*IT.java` suffix and are run by the Maven Failsafe plugin; unit tests in the domain and application layers use the `*Test.java` suffix and are run by Surefire.
- Each story's inbound adapter (REST controller or other driving port) is the entry point that drives the application service; the reference structure makes this pattern explicit by providing a named driving-port stub.
- Verification that `spring.jpa.hibernate.ddl-auto` is set to `validate` or `none` in all environment configurations, consistent with ADR-0011 and ADR-0010.
- Coordination with the baseline project structure established in AE-0004.

**Out of scope**
- Any real bounded context or business feature.
- Flyway migration scripts for a real schema.
- Mapping code between domain objects and JPA entities.
- Cross-cutting concerns such as transaction boundaries, error handling, and domain event dispatch (deferred per ADR-0017).
- OpenAPI annotation conventions on inbound adapters (covered by ADR-0014 in a separate enabler).
- Lombok usage conventions on domain objects beyond what the ArchUnit rules enforce structurally.

## Tasks
- [X] Add `bwyn-core`, `bwyn-application`, `bwyn-rest`, and `bwyn-infrastructure` as modules in the parent POM, declaring inter-module dependencies (`bwyn-application` → `bwyn-core`; `bwyn-rest` → `bwyn-application`, `bwyn-core`; `bwyn-infrastructure` → `bwyn-application`, `bwyn-core`) (ADR-0018).
- [X] Agree and document the canonical package naming convention for bounded contexts and their four layers.
- [X] Create the `bwyn-core` module with the reference package `com.bwyn.<context>.domain` and a representative empty class for each domain building block (entity interface, value object, repository port interface).
- [X] Create the `bwyn-application` module with the reference package `com.bwyn.<context>.application` and a representative empty application service interface.
- [X] Create the `bwyn-rest` module with the reference package `com.bwyn.<context>.adapter.in` and a representative empty Spring MVC controller stub.
- [X] Create the `bwyn-infrastructure` module with the reference package `com.bwyn.<context>.adapter.out` and a representative empty Spring Data JPA repository stub.
- [X] Write the README describing each module and layer, its responsibilities, and its permitted dependencies.
- [X] Add ArchUnit rules asserting that classes in `*.domain.*` have no dependency on `org.springframework`, `javax.persistence`, `jakarta.persistence`, or `*.adapter.*`.
- [X] Add ArchUnit rules asserting that classes in `*.application.*` have no dependency on `*.adapter.*`.
- [X] Add ArchUnit rules asserting that classes in `*.adapter.in.*` and `*.adapter.out.*` depend only on types in `*.domain.*` or `*.application.*`.
- [X] Verify that `spring.jpa.hibernate.ddl-auto` is set to `validate` or `none` in `application.yml` and in any test configuration `.yml` files.
- [X] Run the full backend test suite and confirm all ArchUnit rules pass and the build completes within the ten-minute budget (Practice: Ten-Minute Build).

## Definition of done
- [X] All four Maven modules (`bwyn-core`, `bwyn-application`, `bwyn-rest`, `bwyn-infrastructure`) are declared in the parent POM and build cleanly with `./mvnw verify`.
- [X] Each module contains a reference package under a named placeholder bounded context with at least one representative empty class.
- [X] A README is present that names each module, its layer, and its dependency rules.
- [X] The `bwyn-rest` module contains a driving-port stub that illustrates the inbound adapter entry point expected by every story.
- [X] Maven Failsafe is configured and executes `*IT.java` tests in adapter modules; Maven Surefire executes `*Test.java` tests in domain and application modules.
- [X] All three ArchUnit rule sets (domain, application, adapter) are present as JUnit tests and pass on `./mvnw verify`.
- [X] `spring.jpa.hibernate.ddl-auto` is confirmed as `validate` or `none` in all environment configurations via `application.yml` and test `.yml` files, and verified by an ArchUnit or property-assertion test.
- [X] The full backend build and test suite completes in ten minutes or fewer.
- [ ] A pull request has been reviewed and approved before merge (Practice: Code review).
