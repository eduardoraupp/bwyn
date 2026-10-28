# 0012. Use Testcontainers for Tests That Need Real Infrastructure

- **Status**: Accepted
- **Date**: 2026-10-28

## Context

The bwyn backend uses PostgreSQL as its database (ADR-0010) with Spring Data JPA and Hibernate for persistence (ADR-0011). ADR-0011 explicitly deferred the decision on integration-testing strategy for the persistence layer, naming Testcontainers as a candidate approach. Tests that exercise repository interfaces, Flyway migrations, or any other code path that depends on a real database cannot be reliably covered by unit tests alone; in-memory substitutes (e.g. H2) do not faithfully reproduce PostgreSQL behaviour and can mask dialect-specific bugs.

Testcontainers is a Java library that starts and stops real Docker containers (e.g. a PostgreSQL instance) as part of the JUnit test lifecycle. It integrates with Spring Boot's test infrastructure (ADR-0008) and allows tests to run against the same database engine used in production, without requiring a pre-provisioned external service. This approach is consistent with the project's existing use of Docker for local development and CI (noted in ADR-0010).

## Decision

We will use Testcontainers to provide real infrastructure for any backend test that cannot be adequately covered against a mock or in-memory substitute. Concretely, all Spring Data JPA integration tests and Flyway migration tests will run against a PostgreSQL container managed by Testcontainers. We will configure a shared, reusable container instance per test suite to keep test execution times acceptable. No test will rely on a pre-provisioned external database service; the container lifecycle will be fully managed by Testcontainers within the test run.

## Consequences

**Positive**
- Tests run against the same PostgreSQL engine used in production (ADR-0010), eliminating false confidence from in-memory substitutes and exposing dialect-specific issues early.
- Flyway migration scripts (ADR-0010) are exercised automatically as part of container startup, verifying that migrations apply cleanly on every test run.
- The container lifecycle is fully managed within the test run, so no external database service needs to be provisioned or maintained for CI or local development.
- Testcontainers integrates with Spring Boot's `@DataJpaTest` and `@SpringBootTest` slices (ADR-0008), keeping the testing approach consistent with existing conventions.

**Negative**
- Tests that use Testcontainers require Docker to be available in the local development environment and in CI; this adds an infrastructure prerequisite for all contributors.
- Container startup time adds latency to the test suite compared to in-memory alternatives; a shared, reusable container strategy mitigates this but must be deliberately configured.
- Flaky behaviour can arise from container startup races or resource constraints in CI environments; the team must invest in stable container configuration and appropriate timeouts.

**Neutral**
- A shared Testcontainers base configuration (reusable PostgreSQL container, Spring Boot property overrides) must be established as an architectural enabler before persistence integration tests are written; this work should be coordinated with the Flyway and JPA setup from ADRs 0010 and 0011.
- ArchUnit rules (ADR-0009) should be reviewed to ensure that Testcontainers-based test classes reside in the correct test layer and do not introduce unintended dependencies into production code.
- Decisions about whether to extend Testcontainers to other infrastructure (e.g. message brokers, external HTTP services) will be addressed in follow-up ADRs or tickets as those dependencies are introduced.
- Hibernate DDL auto-generation must remain disabled in the Testcontainers environment, consistent with the constraint established in ADR-0011; Flyway is solely responsible for schema state.
