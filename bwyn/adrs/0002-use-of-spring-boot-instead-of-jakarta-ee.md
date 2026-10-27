# 0002. Use of Spring Boot Instead of Jakarta EE

- **Status**: Accepted
- **Date**: 2026-10-27

## Context

The cinema ticketing system requires a backend application framework to handle HTTP routing, business logic, data access, and integration with PostgreSQL (ADR-0001). Jakarta EE is a well-established standard for enterprise Java applications, but it typically requires a full application server (such as WildFly or Payara), involves significant configuration overhead, and assumes a team with the capacity to manage that infrastructure. The project has a single developer with no dedicated QA or operations resource. Spring Boot is a widely adopted Java framework that provides an embedded server, auto-configuration, and a large ecosystem of production-ready integrations, allowing a single developer to build, test, and ship features with minimal infrastructure setup.

## Decision

We will use Spring Boot as the backend application framework instead of Jakarta EE.

## Consequences

**Positive**
- The embedded server (e.g. Tomcat) removes the need to install and manage a separate application server, reducing operational burden on the single developer.
- Auto-configuration and Spring Boot starters reduce boilerplate, allowing the developer to deliver and verify user stories faster.
- Spring Data JPA integrates directly with PostgreSQL (ADR-0001), simplifying data access layer implementation.
- A large community and extensive documentation lower the cost of problem-solving for a solo developer.

**Negative**
- Spring Boot's auto-configuration can obscure underlying behaviour, making certain performance or security issues harder to diagnose.
- The framework introduces opinionated defaults that may need to be overridden as the system grows, adding future maintenance work.

**Neutral**
- Jakarta EE APIs (e.g. JPA, Bean Validation) are still used under the hood via Spring's implementations; the team is not entirely departing from the Jakarta EE standard.
- Performance tuning (connection pooling, caching) will still be required to meet the 2-second response-time requirement under normal load.
