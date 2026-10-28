# 0008. Use Spring Boot as the Foundation of the Backend Application

- **Status**: Accepted
- **Date**: 2026-10-28

## Context

The bwyn project requires a backend application to serve data and business logic to the frontend (ADRs 0005–0007). A foundation framework must be chosen to handle concerns such as HTTP request routing, dependency injection, configuration management, and application lifecycle. Without a shared foundation, these concerns would need to be assembled and maintained manually from disparate libraries, increasing setup cost and inconsistency across the team.

Spring Boot is a widely adopted, opinionated framework built on the Spring ecosystem for the JVM. It provides auto-configuration, an embedded web server, and production-ready features (health checks, metrics, externalised configuration) out of the box, reducing the amount of boilerplate infrastructure code the team must write and maintain.

## Decision

We will use Spring Boot as the foundation of the bwyn backend application. All backend HTTP endpoints, dependency injection, configuration management, and application lifecycle concerns will be built on Spring Boot conventions and its auto-configuration model.

## Consequences

**Positive**
- Spring Boot's auto-configuration and embedded server significantly reduce initial setup and infrastructure boilerplate, allowing the team to focus on business logic sooner.
- The Spring ecosystem provides mature, well-maintained integrations for common backend concerns (data access, security, messaging, testing), reducing the need to evaluate and wire third-party libraries manually.
- Spring Boot's widespread adoption means extensive documentation, community support, and developer familiarity are readily available.
- Built-in production-ready features (Actuator health checks, metrics endpoints, externalised configuration via `application.properties`/`application.yml`) are available without additional setup.

**Negative**
- Spring Boot's opinionated auto-configuration can be difficult to override or debug when default behaviour does not match project requirements.
- The framework and its dependency tree add startup time and memory overhead compared to leaner alternatives; this may be relevant if serverless or constrained deployment targets are introduced later.
- Developers unfamiliar with the Spring ecosystem face a learning curve around concepts such as the application context, bean lifecycle, and annotation-driven configuration.

**Neutral**
- A baseline project structure (package layout, configuration files, and module boundaries) must be established as an architectural enabler before feature development begins.
- Decisions about specific Spring modules to adopt (e.g. Spring Data, Spring Security, Spring Web MVC vs. WebFlux) will need to be addressed in follow-up ADRs or tickets.
- Any future migration away from Spring Boot would require a superseding ADR and significant rework of infrastructure and wiring code.
