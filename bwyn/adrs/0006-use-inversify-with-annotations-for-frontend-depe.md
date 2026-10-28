# 0006. Use Inversify with Annotations for Frontend Dependency Injection

- **Status**: Accepted
- **Date**: 2026-10-28

## Context

As the bwyn frontend grows, components and services will accumulate dependencies (e.g. API clients, stores, utilities). Without a structured approach, dependency wiring becomes implicit, scattered, and difficult to test. A dependency injection (DI) container provides a centralised, explicit way to declare and resolve dependencies.

Inversify is a TypeScript-first inversion-of-control container that uses decorator-based annotations (`@injectable`, `@inject`) to declare and wire dependencies. This annotation style aligns with common TypeScript patterns and keeps DI metadata co-located with the classes that use it. The project already uses Babel (via Webpack, per ADR-0005), which can be configured to support the TC39 decorator proposal required by Inversify.

## Decision

We will use Inversify as the dependency injection container for the bwyn frontend, and we will use its decorator-based annotation style (`@injectable`, `@inject`, and `reflect-metadata`) to declare and resolve dependencies.

## Consequences

**Positive**
- Dependency graphs become explicit and centralised, making the codebase easier to navigate and reason about.
- Inversify's annotation style reduces boilerplate compared to manual factory wiring and keeps DI declarations close to the classes they describe.
- Dependencies can be swapped for test doubles by reconfiguring the container, improving testability in Jest (per ADR-0005).
- Inversify is TypeScript-first, providing strong type safety and IDE support for bindings and resolutions.

**Negative**
- Decorator support requires enabling `experimentalDecorators` (TypeScript) or the equivalent Babel plugin, and `reflect-metadata` must be imported at the application entry point; this adds configuration overhead.
- Inversify introduces a learning curve for developers unfamiliar with IoC containers or TypeScript decorators.
- Decorator syntax is still a stage-3 TC39 proposal; reliance on the legacy `experimentalDecorators` flag may require migration work if the standard diverges.

**Neutral**
- A container configuration module (the composition root) must be established as an architectural enabler before feature services are wired up.
- Babel or TypeScript compiler configuration changes to support decorators must be coordinated with the shared Webpack configuration established under ADR-0005.
- Decisions about container scope (singleton vs. transient bindings) and modular container composition will need to be addressed in follow-up tickets or ADRs.
