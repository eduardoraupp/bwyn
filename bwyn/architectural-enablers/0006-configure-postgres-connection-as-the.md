# AE-0006. PostgreSQL and Spring Data JPA Baseline Configuration

- **Status**: Proposed
- **Created**: 2026-10-31
- **Completed**:
- **Implements ADR(s)**: ADR-0010 Use PostgreSQL and Flyway for the Database and Its Evolution, ADR-0011 Use Spring Data JPA with Hibernate for Persistence

## Goal
Put in place the baseline datasource configuration, Flyway wiring, and Spring Data JPA settings that all persistence-related stories depend on. Without this enabler, no outbound adapter can connect to PostgreSQL, no migration script can be applied, and no JPA repository can be exercised.

## Scope
**In scope**
- Add `spring-boot-starter-data-jpa` and the PostgreSQL JDBC driver as runtime dependencies (ADR-0011).
- Add the Flyway dependency; Spring Boot's Flyway auto-configuration will apply migrations on startup (ADR-0010).
- Provide `application.yml` datasource properties (`spring.datasource.url`, `username`, `password`, `driver-class-name`) pointing at a local PostgreSQL instance, with values externalised via environment variables so no credentials are committed to the repository (ADR-0010).
- Set `spring.jpa.hibernate.ddl-auto=validate` in all environment configurations so Hibernate never manages the schema; Flyway remains solely responsible for schema state (ADR-0011).
- Set `spring.jpa.open-in-view=false` to keep the persistence context within the application-layer boundary (ADR-0017).
- Add an initial Flyway baseline migration script (`V1__baseline.sql`) that creates the schema version table and records the empty baseline, proving the migration pipeline works end-to-end (ADR-0010).
- Verify the configuration by confirming that the Spring Boot application context starts successfully and Flyway reports zero pending migrations against a running PostgreSQL instance.

**Out of scope**
- Testcontainers integration test setup (deferred to AE-0007, per ADR-0012).
- Any feature entity, repository interface, or domain migration script; those belong to feature stories.
- Production or CI environment datasource configuration; only local development configuration is established here.
- Connection-pool tuning or advanced Hibernate query logging.

## Tasks
- [ ] Add `spring-boot-starter-data-jpa`, PostgreSQL JDBC driver, and Flyway dependencies to the build file.
- [ ] Add `application.yml` datasource block with `spring.datasource.url`, `username`, `password`, and `driver-class-name`, reading values from environment variables.
- [ ] Set `spring.jpa.hibernate.ddl-auto=validate` and `spring.jpa.open-in-view=false` in `application.yml`.
- [ ] Create `src/main/resources/db/migration/V1__baseline.sql` as an empty baseline migration script.
- [ ] Start a local PostgreSQL instance and verify the application context loads without errors and Flyway reports zero pending migrations.
- [ ] Add or extend an ArchUnit rule asserting that no class in the domain core layer imports a JPA or Spring Data package (ADR-0009, ADR-0017).
- [ ] Raise a pull request and obtain a code review (Practice: Code review).

## Definition of done
- [ ] `./gradlew bootRun` (or equivalent) starts without errors when a PostgreSQL instance is reachable at the configured URL.
- [ ] Flyway log on startup shows `Successfully applied 1 migration` for `V1__baseline.sql` and zero pending migrations on a fresh database.
- [ ] `spring.jpa.hibernate.ddl-auto` is set to `validate` in every `application*.yml` file in the repository; no file sets it to `create`, `create-drop`, or `update`.
- [ ] The ArchUnit rule asserting no domain-core class imports JPA or Spring Data passes in the test suite.
- [ ] No datasource credentials appear as plain values in any committed file; all are read from environment variables.
