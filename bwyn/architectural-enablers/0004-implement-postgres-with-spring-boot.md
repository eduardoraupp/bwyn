# AE-0004. Integrate PostgreSQL with Spring Boot via JPA and Hibernate

- **Status**: Proposed
- **Created**: 2026-10-27
- **Completed**:
- **Implements ADR(s)**: ADR 0001 – Use of PostgreSQL instead of Oracle DB

## Goal
Add a working PostgreSQL data source to the Spring Boot backend, configured through JPA and Hibernate, so that subsequent stories can persist and query domain data without any further database wiring. (ADR 0001)

## Scope
**In scope**
- Adding the PostgreSQL JDBC driver and Spring Data JPA dependencies to the Maven project
- Configuring a JPA data source that connects to a PostgreSQL instance (local and, by convention, production)
- Enabling Hibernate as the JPA provider with dialect set for PostgreSQL
- Verifying the connection with a smoke test that confirms the application context starts and can reach the database
- Providing a documented example of the required environment variables or configuration properties so the single developer can reproduce the setup (Driver: Constraints)

**Out of scope**
- Defining any domain entity, table or schema beyond what is needed to verify connectivity
- Database migrations or schema management tooling (a follow-up enabler should address this)
- Connection pooling tuning or query optimisation (required to meet the 2-second response target but belongs to a later performance enabler) (Driver: Quality Attributes)
- Evaluation or provisioning of a managed PostgreSQL hosting option (noted as neutral consequence in ADR 0001)

## Tasks
- [ ] Add `spring-boot-starter-data-jpa` and `postgresql` JDBC driver dependencies to `pom.xml`
- [ ] Add `spring.datasource.*` and `spring.jpa.database-platform` properties, reading credentials from environment variables
- [ ] Set Hibernate DDL auto to `validate` (or `none`) so the application does not silently alter the database on startup
- [ ] Write a Spring context integration test that starts the application and executes a trivial query (e.g. `SELECT 1`) to confirm connectivity
- [ ] Document the required environment variables in the project README

## Definition of done
- [ ] `./mvnw verify` passes, including the connectivity integration test, against a local PostgreSQL instance
- [ ] The application fails fast with a clear error when the database is unreachable, confirmed by running the test with an invalid host
- [ ] No Oracle-specific dialect, driver or dependency is present in `pom.xml` or any configuration file (ADR 0001)
- [ ] The README lists every environment variable needed to connect to the database
