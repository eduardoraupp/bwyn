# AE-0004. Backend Service Technology Scaffold

- **Status**: Proposed
- **Created**: 2026-10-28
- **Completed**:
- **Implements ADR(s)**: ADR-0008 Use Spring Boot as the Foundation of the Backend Application, ADR-0009 Use ArchUnit for Backend Architectural Fitness Functions, ADR-0010 Use PostgreSQL and Flyway for the Database and Its Evolution, ADR-0011 Use Spring Data JPA with Hibernate for Persistence, ADR-0012 Use Testcontainers for Tests That Need Real Infrastructure, ADR-0013 Use Lombok to Reduce Boilerplate in the Backend, ADR-0014 Use Swagger and OpenAPI for Backend API Documentation

## Goal
Put in place the complete backend scaffold so that every subsequent feature story has a working Spring Boot application to build on: a Maven multi-module project with a `bwyn-runtime` module that owns application startup and cross-cutting configuration, all declared dependencies, ArchUnit fitness functions that enforce layer boundaries from day one, Flyway configured for PostgreSQL schema versioning, a shared Testcontainers base for integration tests, Lombok enabled, and springdoc-openapi exposing Swagger UI. The domain, application, REST adapter, and infrastructure modules that carry DDD and Hexagonal Architecture packages are addressed in a later architectural enabler. No feature logic or domain code is delivered; the scaffold exists solely to prove that all the pieces assemble and that the fitness functions pass on a green build (ADR-0008, ADR-0009, ADR-0010, ADR-0011, ADR-0012, ADR-0013, ADR-0014).

## Scope

**In scope**
- Maven multi-module project with a parent POM and a `bwyn-runtime` module that is the Spring Boot application entry point and owns cross-cutting configuration (ADR-0018)
- Parent `pom.xml` declaring all required dependencies and plugins under `dependencyManagement`: `spring-boot-starter-web`, `spring-boot-starter-data-jpa`, `spring-boot-starter-actuator`, `flyway-core`, `postgresql` (runtime), `lombok` (annotationProcessor + compileOnly), `springdoc-openapi-starter-webmvc-ui`, `archunit-junit5`, `testcontainers` with the PostgreSQL module, and `spring-boot-starter-test` (ADR-0008, ADR-0010, ADR-0011, ADR-0012, ADR-0013, ADR-0014)
- `bwyn-runtime` module `pom.xml` declaring the Spring Boot Maven plugin and a Maven dependency on the `bwyn-frontend` Maven module so that the packaged frontend bundle is available on the classpath and served as static content when the application runs in production (ADR-0019)
- `bwyn-runtime` contains only: the Spring Boot entry point class, cross-cutting configuration beans (OpenAPI, Actuator), `application.yml` / `application-prod.yml`, Flyway migration scripts, the shared Testcontainers base class, and the ArchUnit fitness-function test; it does not contain domain, application-service, REST adapter, outbound-adapter, or any other domain packages — those belong to modules introduced in later enablers (ADR-0017)
- `application.yml` for the `default` profile: data-source placeholder properties, `spring.jpa.hibernate.ddl-auto=validate`, Flyway enabled, Actuator health endpoint exposed, springdoc-openapi API metadata and Swagger UI path configured, Swagger UI disabled for the `prod` profile (ADR-0008, ADR-0010, ADR-0011, ADR-0014)
- Placeholder Flyway migration `V1__init.sql` (empty schema, establishes the migration baseline) stored under `src/main/resources/db/migration` in `bwyn-runtime` (ADR-0010)
- `OpenApiConfig` bean in `bwyn-runtime` with title, version, and description for the bwyn API (ADR-0014)
- `ArchitectureFitnessTest` JUnit 5 class in `bwyn-runtime` covering the layer boundary rules that can be verified against the modules present at this stage: no cycles between top-level packages; repository interfaces must not reside outside the domain layer (rule is declared now and will pass trivially until domain modules exist); the rules are written to be extended as further modules are added in the next enabler (ADR-0009, ADR-0017)
- Shared Testcontainers base class `PostgresIntegrationTest` in `bwyn-runtime` that starts a single reusable PostgreSQL container, overrides `spring.datasource.*` properties via `@DynamicPropertySource`, and applies Flyway migrations on first startup (ADR-0012)
- A smoke `@SpringBootTest` `ApplicationContextSmokeTest` in `bwyn-runtime` extending `PostgresIntegrationTest` that loads the full application context against the Testcontainers PostgreSQL instance and asserts the context starts without errors (ADR-0008, ADR-0012)
- Lombok `lombok.config` at the project root disabling `@SneakyThrows` and enabling `lombok.addLombokGeneratedAnnotation=true` so that JaCoCo excludes generated code (ADR-0013)

**Out of scope**
- Domain, application-service, REST adapter, and outbound-adapter modules and their package structures — these are addressed in the next architectural enabler (ADR-0017)
- Any feature domain model, use case, HTTP endpoint, or core domain code
- Spring Security configuration
- CI pipeline definition (addressed separately)
- Client SDK or OpenAPI artefact publication
- Testcontainers for infrastructure other than PostgreSQL
- Static resource controller or CDN configuration for frontend serving (the `bwyn-runtime` dependency on `bwyn-frontend` places the bundle on the classpath; how it is served in detail is addressed in a follow-up ADR or ticket)

## Tasks
- [ ] Create the parent `pom.xml` with Spring Boot parent BOM, `dependencyManagement` for all in-scope dependencies, Maven Surefire plugin configured for JUnit 5, and `bwyn-runtime` declared as a module
- [ ] Create the `bwyn-runtime` module `pom.xml` with the Spring Boot Maven plugin, a dependency on `bwyn-frontend`, and all runtime-scoped dependencies inherited from the parent
- [ ] Create the main application class `BwynApplication` in `bwyn-runtime` with `@SpringBootApplication`
- [ ] Add `application.yml` in `bwyn-runtime` with data-source placeholder properties, `ddl-auto=validate`, Flyway enabled, Actuator health endpoint, and springdoc-openapi metadata
- [ ] Add `application-prod.yml` in `bwyn-runtime` that sets `springdoc.swagger-ui.enabled=false`
- [ ] Add `src/main/resources/db/migration/V1__init.sql` in `bwyn-runtime` as an empty baseline migration
- [ ] Add `OpenApiConfig` bean in `bwyn-runtime` with API title, version, and description
- [ ] Add `lombok.config` at the project root with `@SneakyThrows` disabled and `addLombokGeneratedAnnotation=true`
- [ ] Add `ArchitectureFitnessTest` JUnit 5 class in `bwyn-runtime` with ArchUnit rules for the layer boundaries verifiable at this stage, written to be extended as further modules are added
- [ ] Add `PostgresIntegrationTest` abstract base class in `bwyn-runtime` with Testcontainers PostgreSQL, `@DynamicPropertySource`, and reusable container
- [ ] Add `ApplicationContextSmokeTest` in `bwyn-runtime` extending `PostgresIntegrationTest` annotated with `@SpringBootTest` asserting the context loads
- [ ] Run the full build locally (`mvn verify`) and confirm all tests pass and the fitness functions report no violations

## Definition of done
- [ ] `mvn verify` completes successfully with no test failures and no ArchUnit violations
- [ ] `bwyn-runtime` contains no domain, application-service, REST adapter, outbound-adapter, or any other domain packages
- [ ] `ArchitectureFitnessTest` and `ApplicationContextSmokeTest` both reside in the `bwyn-runtime` module
- [ ] `ArchitectureFitnessTest` contains the layer boundary rules listed in scope and all rules pass on the scaffold as built
- [ ] `ApplicationContextSmokeTest` passes against a Testcontainers-managed PostgreSQL container with Flyway migrations applied
- [ ] `spring.jpa.hibernate.ddl-auto` is set to `validate` in `application.yml` and is not overridden to `create`, `create-drop`, or `update` in any non-test profile
- [ ] Swagger UI is reachable at the configured path when the application starts with the default profile
- [ ] Swagger UI is disabled when the application starts with the `prod` profile
- [ ] `lombok.config` is present at the project root and `@SneakyThrows` is flagged as forbidden
- [ ] The `bwyn-runtime` module declares a Maven dependency on `bwyn-frontend` and the frontend bundle is present on the runtime classpath after `mvn package`
- [ ] A second developer can check out the repository, run `mvn verify` with Docker available, and get a green build without manual environment setup
