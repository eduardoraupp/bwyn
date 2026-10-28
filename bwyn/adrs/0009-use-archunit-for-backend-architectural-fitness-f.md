# 0009. Use ArchUnit for Backend Architectural Fitness Functions

- **Status**: Accepted
- **Date**: 2026-10-28

## Context

The bwyn backend is built on Spring Boot (ADR-0008) and, like the frontend (ADR-0007), is expected to follow a layered architectural structure. Without automated enforcement, layer boundary violations and other structural rules can accumulate silently over time, especially as the team grows. Code-review alone is not a reliable safety net for architectural constraints.

Fitness functions are automated checks that verify architectural properties continuously, catching violations as part of the normal build or test cycle. ArchUnit is a Java library that allows architectural rules to be expressed as JUnit tests, enabling layer dependency rules, naming conventions, package structure constraints, and other structural properties to be verified on every build without additional tooling infrastructure.

## Decision

We will use ArchUnit to implement architectural fitness functions for the bwyn backend. Architectural rules (such as layer dependency constraints, package structure conventions, and naming rules) will be expressed as ArchUnit JUnit tests and executed as part of the standard backend test suite run by Spring Boot's test infrastructure (ADR-0008).

## Consequences

**Positive**
- Layer boundary violations are caught automatically on every build, reducing reliance on manual code-review for structural enforcement.
- ArchUnit tests are plain JUnit tests, requiring no additional CI tooling or external services; they integrate naturally with the existing Spring Boot test setup (ADR-0008).
- Architectural rules are expressed in code, making them versioned, reviewable, and self-documenting alongside the codebase they govern.
- Early detection of violations keeps the cost of correction low compared to discovering drift after it has accumulated.

**Negative**
- ArchUnit rules must be written and maintained by the team; poorly scoped or overly strict rules can produce false positives that slow development.
- Developers unfamiliar with ArchUnit face a small learning curve in writing and interpreting rule definitions.
- Rules that are too coarse may miss subtle violations; achieving good coverage requires deliberate investment in rule authorship.

**Neutral**
- An initial set of ArchUnit rules covering the agreed backend layer structure must be established as an architectural enabler before feature development begins; this work should be coordinated with the baseline project structure defined under ADR-0008.
- Decisions about which additional rule categories to enforce (e.g. cycle detection, Spring-specific annotation constraints, naming conventions) will be addressed in follow-up tickets.
- As the architecture evolves, ArchUnit rules will need to be updated in step; stale rules that no longer reflect intent should be removed or revised promptly.
