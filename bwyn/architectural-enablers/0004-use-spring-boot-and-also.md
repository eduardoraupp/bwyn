# AE-0004. Backend Service Technology Scaffold

- **Status**: Proposed
- **Created**: 2026-10-28
- **Completed**:
- **Implements ADR(s)**: ADR-0008 Use Spring Boot as the Foundation of the Backend Application, ADR-0009 Use ArchUnit for Backend Architectural Fitness Functions, ADR-0010 Use PostgreSQL and Flyway for the Database and Its Evolution, ADR-0011 Use Spring Data JPA with Hibernate for Persistence, ADR-0012 Use Testcontainers for Tests That Need Real Infrastructure, ADR-0013 Use Lombok to Reduce Boilerplate in the Backend, ADR-0014 Use Swagger and OpenAPI for Backend API Documentation

## Goal
Put in place the complete backend scaffold so that every subsequent feature story has a working Spring Boot application to build on: a Maven project with the agreed package structure, all declared dependencies, a wired-but-empty Hexagonal Architecture skeleton, ArchUnit fitness functions that enforce layer boundaries from day one, Flyway configured for PostgreSQL schema versioning, a shared Testcontainers base for integration tests, Lombok enabled, and springdoc-openapi exposing Swagger UI. No feature logic is delivered; the scaffold exists solely to prove that all the pieces assemble and that the fitness functions pass on a green build (ADR-0008, ADR-0009, ADR-0010, ADR-0011, ADR-0012, ADR-0013, ADR-0014).

## Scope

**In scope**
- Maven `pom.xml` declaring all required dependencies and plugins: `spring-boot-starter-web`, `spring-boot-starter-data-jpa`, `spring-boot-starter-actuator`, `flyway-core`, `postgresql` (runtime), `lombok` (annotationProcessor + compileOnly), `springdoc-openapi-starter-webmvc-ui`, `archunit-junit5`, `testcontainers` with the PostgreSQL module, and `spring-boot-starter-test` (ADR-0008, ADR-0010, ADR-0011, ADR-0012, ADR-0013, ADR-0014)
- Top-level package layout reflecting Hexagonal Architecture with DDD bounded-context organisation: `domain` (entities, value objects, aggregates, domain events, repository ports), `application` (application services, use-case ports), `adapter.in` (inbound adapters), `adapter.out` (outbound adapters), `infrastructure` (Spring Boot entry point, configuration beans) (ADR-0008, ADR-0017)
- `application.yml` for the `default` profile: data-source placeholder properties, `spring.jpa.hibernate.ddl-auto=validate`, Flyway enabled, Actuator health endpoint exposed, springdoc-openapi API metadata and Swagger UI path configured, Swagger UI disabled for the `prod` profile (ADR-0008, ADR-0010, ADR-0011, ADR-0014)
- Placeholder Flyway migration `V1__init.sql` (empty schema, establishes the migration baseline) stored under `src/main/resources/db/migration` (ADR-0010)
- `OpenApiConfig` bean in the `infrastructure` layer with title, version, and description for the bwyn API (ADR-0014)
- ArchUnit JUnit 5 test class `ArchitectureFitnessTest` covering: domain layer has no dependency on Spring or JPA packages; application layer depends only on domain; inbound adapters depend only on application and domain ports; outbound adapters depend only on domain ports; no cycles between bounded-context packages; repository interfaces reside in the domain layer (ADR-0009, ADR-0011, ADR-0017)
- Shared Testcontainers base class `PostgresIntegrationTest` that starts a single reusable PostgreSQL container, overrides `spring.datasource.*` properties via `@DynamicPropertySource`, and applies Flyway migrations on first startup (ADR-0012)
- A smoke `@SpringBootTest` that loads the full application context against the Testcontainers PostgreSQL instance and asserts the context starts without errors (ADR-0008, ADR-0012)
- Lombok `lombok.config` at the project root disabling `@SneakyThrows` and enabling `lombok.addLombokGeneratedAnnotation=true` so that JaCoCo excludes generated code (ADR-0013)

**Out of scope**
- Any feature domain model, use case, or HTTP endpoint
- Spring Security configuration
- Spring Boot AI dependency (no ADR accepted for Spring Boot AI; a superseding ADR is required before it can be added)
- CI pipeline definition (addressed separately)
- Client SDK or OpenAPI artefact publication
- Testcontainers for infrastructure other than PostgreSQL

## Tasks
- [ ] Create the Maven `pom.xml` with Spring Boot parent, all in-scope dependencies, and the Maven Surefire plugin configured to run JUnit 5
- [ ] Create the main application class `BwynApplication` under the `infrastructure` package with `@SpringBootApplication`
- [ ] Create the package skeleton: `domain`, `application`, `adapter.in`, `adapter.out`, `infrastructure` with a `.gitkeep` or placeholder README in each
- [ ] Add `application.yml` with data-source placeholder properties, `ddl-auto=validate`, Flyway enabled, Actuator health endpoint, and springdoc-openapi metadata
- [ ] Add `application-prod.yml` that sets `springdoc.swagger-ui.enabled=false`
- [ ] Add `src/main/resources/db/migration/V1__init.sql` as an empty baseline migration
- [ ] Add `OpenApiConfig` bean in `infrastructure` with API title, version, and description
- [ ] Add `lombok.config` at the project root with `@SneakyThrows` disabled and `addLombokGeneratedAnnotation=true`
- [ ] Add `ArchitectureFitnessTest` JUnit 5 class with ArchUnit rules for all in-scope layer constraints
- [ ] Add `PostgresIntegrationTest` abstract base class with Testcontainers PostgreSQL, `@DynamicPropertySource`, and reusable container
- [ ] Add `ApplicationContextSmokeTest` extending `PostgresIntegrationTest` annotated with `@SpringBootTest` asserting the context loads
- [ ] Run the full build locally (`mvn verify`) and confirm all tests pass and the fitness functions report no violations

## Definition of done
- [ ] `mvn verify` completes successfully with no test failures and no ArchUnit violations
- [ ] `ArchitectureFitnessTest` contains rules for every layer boundary listed in scope and all rules pass on the empty scaffold
- [ ] `ApplicationContextSmokeTest` passes against a Testcontainers-managed PostgreSQL container with Flyway migrations applied
- [ ] `spring.jpa.hibernate.ddl-auto` is set to `validate` in `application.yml` and is not overridden to `create`, `create-drop`, or `update` in any non-test profile
- [ ] Swagger UI is reachable at the configured path when the application starts with the default profile
- [ ] Swagger UI is disabled when the application starts with the `prod` profile
- [ ] `lombok.config` is present at the project root and `@SneakyThrows` is flagged as forbidden
- [ ] The package layout matches the Hexagonal Architecture structure described in scope with no production class residing in the wrong layer
- [ ] A second developer can check out the repository, run `mvn verify` with Docker available, and get a green build without manual environment setup
