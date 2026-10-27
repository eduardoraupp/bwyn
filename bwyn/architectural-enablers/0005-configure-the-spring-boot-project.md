# AE-0005. Configure Spring Boot with JPA, Security, and Redis starters

- **Status**: Proposed
- **Created**: 2026-10-27
- **Completed**:
- **Implements ADR(s)**: ADR 0002 – Use of Spring Boot Instead of Jakarta EE

## Goal
Add the Spring Data JPA, Spring Security, and Spring Data Redis starters to the Maven build so that subsequent stories can use data access, authentication, and caching without each one introducing its own infrastructure setup. This enabler must be complete before any story that depends on secured endpoints, persistent data, or cached responses can be built.

## Scope
**In scope**
- Adding `spring-boot-starter-data-jpa`, `spring-boot-starter-security`, and `spring-boot-starter-data-redis` as Maven dependencies in `pom.xml`
- Providing minimal baseline configuration for each starter (e.g. a default security filter chain that permits all requests as a safe placeholder, a JPA datasource reference, and a Redis connection configuration) so the application starts cleanly
- Verifying that the application context loads without errors after the starters are added

**Out of scope**
- Defining authentication rules, user roles, or login flows (belongs to a security story)
- Designing or migrating any database schema (covered by AE-0004)
- Implementing any caching logic or cache eviction policy (belongs to individual feature stories)
- Choosing a Redis hosting provider or deployment topology

## Tasks
- [ ] Add `spring-boot-starter-data-jpa` to `pom.xml` (if not already present from AE-0004)
- [ ] Add `spring-boot-starter-security` to `pom.xml`
- [ ] Add `spring-boot-starter-data-redis` to `pom.xml`
- [ ] Add a `SecurityFilterChain` bean that permits all requests, so the application starts without blocking existing endpoints while security rules are not yet defined
- [ ] Add Redis connection properties (`spring.data.redis.host`, `spring.data.redis.port`) to `application.properties` or `application.yml` with localhost defaults for local development
- [ ] Confirm JPA datasource properties are present and consistent with AE-0004
- [ ] Run the full test suite and confirm the Spring application context loads successfully

## Definition of done
- [ ] `mvn dependency:tree` shows `spring-boot-starter-data-jpa`, `spring-boot-starter-security`, and `spring-boot-starter-data-redis` on the compile classpath
- [ ] `mvn spring-boot:run` starts the application without errors or unsatisfied bean dependencies
- [ ] The placeholder `SecurityFilterChain` bean is present and all existing endpoints remain reachable (verified by any existing integration or smoke test)
- [ ] Redis connection properties are present in the application configuration file with documented localhost defaults
- [ ] `mvn test` passes with no regressions

## Open questions
- [NEEDS CLARIFICATION: Should Redis be used for HTTP session storage, response caching, or both? The answer will determine which additional auto-configuration properties are needed beyond the connection settings added here.]
- [NEEDS CLARIFICATION: Is a managed Redis instance (e.g. a cloud provider's service) already chosen, or will local Redis suffice until a hosting decision is made?]
