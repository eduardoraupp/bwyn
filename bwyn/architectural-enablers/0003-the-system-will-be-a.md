# AE-0003. Initialise Spring Boot backend project with Maven

- **Status**: Proposed
- **Created**: 2026-10-27
- **Completed**: 
- **Implements ADR(s)**: ADR 0001 – Use Spring Boot for the backend

## Goal
Put a runnable Spring Boot project in place, built and managed by Maven, so that all backend stories have a working application skeleton to build on.

## Scope
**In scope**
- Maven project structure (`pom.xml`) with Spring Boot parent and required starter dependencies
- Application entry point and default configuration
- A health-check or root endpoint confirming the application starts and responds
- Ability to run the application locally with `mvn spring-boot:run`
- Ability to run the test suite with `mvn test`

**Out of scope**
- Database configuration or persistence layer
- Security or authentication configuration
- Frontend integration or static asset serving (ADR 0004 is still Proposed)
- Any business logic or domain model

## Tasks
- [ ] Create a Maven project with the Spring Boot parent POM and `spring-boot-starter-web` dependency
- [ ] Add a main application class annotated with `@SpringBootApplication`
- [ ] Add `application.properties` with a configurable server port (default 8080)
- [ ] Implement a `/health` endpoint that returns HTTP 200
- [ ] Add a smoke test that starts the application context and asserts the `/health` endpoint returns HTTP 200
- [ ] Confirm `mvn test` passes with no failures
- [ ] Confirm `mvn spring-boot:run` starts the application without errors

## Definition of done
- [ ] `mvn test` completes with a green build
- [ ] `mvn spring-boot:run` starts the application and `GET /health` returns HTTP 200
- [ ] No hardcoded port: the port is read from configuration
- [ ] The project structure follows standard Maven conventions (`src/main/java`, `src/test/java`)
