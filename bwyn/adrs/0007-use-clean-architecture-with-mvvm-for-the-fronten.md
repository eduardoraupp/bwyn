# 0007. Use Clean Architecture with MVVM for the Frontend

- **Status**: Accepted
- **Date**: 2026-10-28

## Context

As the bwyn frontend grows in complexity, a clear structural pattern is needed to organise code, separate concerns, and keep the codebase maintainable and testable. Without an explicit architecture, business logic tends to leak into UI components, making units hard to test in isolation and the codebase hard to navigate.

Clean Architecture structures the codebase into concentric layers (entities, use cases, interface adapters, and frameworks/drivers), with dependencies pointing inward so that core business logic is independent of UI frameworks and infrastructure. MVVM (Model–View–ViewModel) is a presentation pattern that fits naturally within Clean Architecture's interface-adapter layer: the ViewModel mediates between the View (React components, per ADR-0005) and the use-case layer, exposing observable state and commands while keeping the View free of business logic.

The dependency injection container provided by Inversify (ADR-0006) supports this structure by allowing use cases and other dependencies to be injected into ViewModels without tight coupling.

## Decision

We will structure the bwyn frontend using Clean Architecture, organising code into distinct layers (entities, use cases, interface adapters, and the framework/UI layer). Within the interface-adapter layer we will apply the MVVM pattern, using ViewModels to mediate between React components and use cases, and using Inversify (ADR-0006) to inject dependencies across layer boundaries.

## Consequences

**Positive**
- Business logic in the use-case and entity layers is independent of React and other framework details, making it straightforward to unit-test with Jest (ADR-0005) without mounting components.
- ViewModels can be tested in isolation by injecting test doubles via the Inversify container (ADR-0006), keeping UI tests focused on rendering concerns.
- The layered structure provides clear conventions for where new code belongs, reducing ambiguity as the team grows.
- Inward-pointing dependencies mean the UI framework (React) can be replaced or upgraded with minimal impact on business logic.

**Negative**
- Clean Architecture introduces more files and indirection than a flat or feature-folder structure; the initial setup cost is higher.
- Developers unfamiliar with Clean Architecture or MVVM face a learning curve before they can contribute confidently.
- Strict layer boundaries require discipline to maintain; without code-review enforcement, shortcuts that violate the dependency rule can accumulate.

**Neutral**
- A reference folder structure and layer boundary conventions must be established as an architectural enabler before feature development begins.
- Decisions about how to handle cross-cutting concerns (e.g. error handling, logging, authentication) within the layer model will need to be addressed in follow-up ADRs or tickets.
- The composition root for the Inversify container (established under ADR-0006) will serve as the wiring point where use cases and ViewModels are bound, and its design should reflect the layer boundaries defined here.
