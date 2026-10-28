# 0016. Use ArchUnitTS for Frontend Architectural Fitness Functions

- **Status**: Accepted
- **Date**: 2026-10-28

## Context

The bwyn frontend is structured using Clean Architecture and MVVM (ADR-0007), implemented in TypeScript (ADR-0015), and wired with Inversify (ADR-0006). As with the backend (ADR-0009), strict layer boundaries are central to the architecture: dependencies must point inward, React components must not reference use-case or entity internals directly, and ViewModels must not leak framework concerns into the domain. Without automated enforcement, these constraints can erode silently as the codebase grows, and code-review alone is not a reliable safety net.

Fitness functions are automated checks that verify architectural properties continuously as part of the normal build or test cycle. ArchUnitTS is a TypeScript-native library modelled on ArchUnit (ADR-0009) that allows architectural rules—layer dependency constraints, naming conventions, import restrictions, and package structure rules—to be expressed as Jest tests (ADR-0005). Because it analyses TypeScript source structure rather than compiled bytecode, it fits naturally into the existing frontend toolchain without additional infrastructure.

## Decision

We will use ArchUnitTS to implement architectural fitness functions for the bwyn frontend. Architectural rules—including Clean Architecture layer dependency constraints, MVVM structural conventions, and naming rules—will be expressed as ArchUnitTS Jest tests (ADR-0005) and executed as part of the standard frontend test suite. No additional CI tooling or external services are required beyond the existing Jest configuration.

## Consequences

**Positive**
- Layer boundary violations are caught automatically on every test run, reducing reliance on manual code-review for structural enforcement and complementing the equivalent backend enforcement provided by ArchUnit (ADR-0009).
- ArchUnitTS rules are expressed as Jest tests (ADR-0005), requiring no additional tooling; they integrate naturally into the existing frontend test setup.
- Architectural rules are versioned, reviewable, and self-documenting alongside the code they govern, making the intended structure explicit and auditable.
- Early detection of violations keeps the cost of correction low compared to discovering drift after it has accumulated across many features.

**Negative**
- ArchUnitTS rules must be written and maintained by the team; poorly scoped or overly strict rules can produce false positives that slow development.
- Developers unfamiliar with ArchUnitTS face a small learning curve in writing and interpreting rule definitions, even if they are already familiar with ArchUnit on the backend.
- Rules that are too coarse may miss subtle violations; achieving good coverage requires deliberate investment in rule authorship.

**Neutral**
- An initial set of ArchUnitTS rules covering the agreed Clean Architecture layer structure and MVVM conventions (ADR-0007) must be established as an architectural enabler before feature development begins; this work should be coordinated with the reference folder structure and layer boundary conventions defined under ADR-0007.
- The ArchUnitTS configuration must be aligned with the TypeScript compiler and module-resolution settings established in the `tsconfig.json` from ADR-0015 to ensure that import paths are resolved correctly during rule evaluation.
- Decisions about which additional rule categories to enforce (e.g. Inversify container bindings residing in the composition root, React components not importing directly from the entity layer) will be addressed in follow-up tickets.
- As the architecture evolves, ArchUnitTS rules will need to be updated in step; stale rules that no longer reflect intent should be removed or revised promptly.
