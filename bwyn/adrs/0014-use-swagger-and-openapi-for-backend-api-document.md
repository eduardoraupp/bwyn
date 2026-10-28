# 0014. Use Swagger and OpenAPI for Backend API Documentation

- **Status**: Accepted
- **Date**: 2026-10-28

## Context

The bwyn backend exposes HTTP endpoints built on Spring Boot (ADR-0008). As the API grows, the frontend team (ADRs 0005–0007) and any other consumers need accurate, up-to-date documentation of available endpoints, request/response schemas, and error contracts. Maintaining documentation separately from the code creates a risk of drift: the documentation becomes stale as endpoints evolve, and consumers cannot rely on it.

OpenAPI 3 is a widely adopted, machine-readable specification standard for HTTP APIs. Swagger UI is a browser-based interface that renders an OpenAPI specification as interactive documentation, allowing developers to explore and manually test endpoints without a separate tool. The springdoc-openapi library integrates both into a Spring Boot application (ADR-0008) by generating an OpenAPI specification automatically from Spring MVC annotations and exposing Swagger UI at a well-known endpoint, keeping documentation and implementation in sync.

## Decision

We will use OpenAPI 3 as the specification format for the bwyn backend API, and we will use the springdoc-openapi library to generate the OpenAPI specification automatically from Spring Boot controller annotations and expose Swagger UI for interactive exploration. We will enrich generated documentation with OpenAPI annotations (e.g. `@Operation`, `@ApiResponse`, `@Schema`) where the default inference is insufficient to describe the contract accurately.

## Consequences

**Positive**
- The OpenAPI specification is generated from the running application code, keeping documentation and implementation in sync and eliminating manual drift.
- Swagger UI gives frontend developers (ADRs 0005–0007) and other consumers an interactive way to explore and test endpoints without additional tooling.
- The machine-readable OpenAPI specification can be used to generate client SDKs, contract tests, or mock servers in future, providing a foundation for further automation.
- springdoc-openapi integrates with Spring Boot's auto-configuration (ADR-0008) with minimal setup, requiring only a single starter dependency.

**Negative**
- Accurate documentation of complex request/response schemas requires deliberate use of OpenAPI annotations alongside business logic; this adds a small but ongoing authoring overhead for each new endpoint.
- Swagger UI is exposed as an application endpoint; it must be explicitly secured or disabled in production environments to avoid unintended disclosure of API structure.
- springdoc-openapi adds a dependency that must be kept aligned with the Spring Boot version in use (ADR-0008); version mismatches can cause startup or generation failures.

**Neutral**
- A baseline springdoc-openapi configuration (API metadata, server URLs, security scheme declarations, and Swagger UI access controls) must be established as an architectural enabler before feature endpoints are documented.
- Decisions about whether to publish the generated OpenAPI specification as a build artefact (e.g. for contract testing or client generation) will be addressed in follow-up ADRs or tickets.
- ArchUnit rules (ADR-0009) may be extended to enforce that OpenAPI annotations are present on all public controller methods, if the team decides to mandate documentation coverage.
- Lombok-generated classes used as request/response DTOs (ADR-0013) must be verified to produce correct OpenAPI schema output; additional `@Schema` annotations may be needed where Lombok's generated accessors are not picked up correctly by springdoc-openapi.
