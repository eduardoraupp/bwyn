# Story: Verify ArchUnit fitness functions pass after adding a new backend endpoint

- **Story**: 002-remove-this-verify-that-the
- **Created**: 2026-11-11
- **Status**: Draft

## Story
As a user, I want the ArchUnit fitness functions to continue passing after a new backend endpoint is added, so that I know the architectural layer boundaries established in AE-0001 are not broken by new code.

## Acceptance tests
- [ ] **Given** the backend compiles with a new Spring MVC inbound adapter endpoint added under `adapter.in`, **When** `mvn verify` is run, **Then** all ArchUnit fitness functions from AE-0001 pass and the build exits green.
- [ ] **Given** a class that belongs to the `domain` package imports a Spring MVC annotation, **When** `mvn verify` is run, **Then** the ArchUnit rule asserting no Spring dependency in the `domain` package fails and the build exits red.
- [ ] **Given** a class that belongs to the `application` package directly references a class in an `adapter` package, **When** `mvn verify` is run, **Then** the ArchUnit rule asserting no `adapter` dependency in the `application` package fails and the build exits red.

## Business rules and constraints
- ArchUnit fitness functions must run as part of the standard Maven `test` phase and must not require a separately provisioned service (ADR-0009).
- The `domain` package must have no dependency on Spring, JPA, or any `adapter` package (AE-0001, ADR-0017).
- The `application` package must have no dependency on any `adapter` package or on JPA (AE-0001, ADR-0017).
- `adapter.in` and `adapter.out` packages must depend only on `domain` or `application` packages, not on each other (AE-0001, ADR-0017).
- A change that breaks an architectural property must fail the build (Practice: Use architectural fitness functions).
- The new endpoint class must reside in `adapter.in`; no routing or controller concern may appear in the `domain` or `application` layer (Architecture).

## Out of scope
- Introducing any real domain model, entity, use case, or business feature (AE-0001).
- Flyway migration scripts or JPA entity mappings (AE-0001).
- The Maven multi-module structure itself — delivered by AE-0002.
- The package skeleton and initial ArchUnit rule authorship — delivered by AE-0001.
- Cross-cutting concerns such as transaction boundaries, error handling, and domain event dispatch (ADR-0017).
- OpenAPI annotation coverage on the new endpoint (ADR-0014); that is follow-up work.

## Tasks
- [ ] Add a minimal Spring MVC controller class in the `adapter.in` package of the `example` bounded context, implementing a single endpoint that returns a fixed response.
- [ ] Run `mvn verify` locally and confirm all existing ArchUnit rules from AE-0001 pass with the new controller present.
- [ ] Write a negative-path ArchUnit test (or temporarily introduce a violating class) that places a Spring annotation inside the `domain` package, confirm the rule fires and the build fails, then remove the violation.
- [ ] Write a negative-path ArchUnit test (or temporarily introduce a violating class) that references an `adapter` class from the `application` package, confirm the rule fires and the build fails, then remove the violation.
- [ ] Confirm `mvn verify` exits green with only the compliant controller in place.
