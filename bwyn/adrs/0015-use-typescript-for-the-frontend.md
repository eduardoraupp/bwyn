# 0015. Use TypeScript for the Frontend

- **Status**: Accepted
- **Date**: 2026-10-28

## Context

The bwyn frontend is built with React, Webpack, and Jest (ADR-0005), structured using Clean Architecture and MVVM (ADR-0007), and wired with Inversify for dependency injection (ADR-0006). Inversify's decorator-based annotation style (`@injectable`, `@inject`) and its `reflect-metadata` requirement depend on TypeScript's type system and its `experimentalDecorators` compiler option to function correctly. Without TypeScript, the decorator-based DI approach chosen in ADR-0006 cannot be used as intended.

More broadly, as the frontend grows in complexity across multiple architectural layers, plain JavaScript provides no compile-time guarantees about interface contracts between layers. Type errors that cross layer boundaries—between use cases, ViewModels, and React components—would only surface at runtime, increasing the cost of defects and making refactoring riskier.

## Decision

We will use TypeScript as the implementation language for all bwyn frontend source code. All new frontend files will use `.ts` or `.tsx` extensions. The TypeScript compiler will be configured with `strict` mode enabled and with `experimentalDecorators` and `emitDecoratorMetadata` enabled to support Inversify (ADR-0006). Webpack (ADR-0005) will be configured to transpile TypeScript via an appropriate loader (e.g. `ts-loader` or `babel-loader` with `@babel/preset-typescript`).

## Consequences

**Positive**
- TypeScript's static type system enforces interface contracts across Clean Architecture layer boundaries (ADR-0007), catching mismatches at compile time rather than at runtime.
- Strong typing and IDE tooling (autocompletion, inline errors, safe refactoring) improve developer productivity and reduce the cost of navigating a multi-layer codebase.
- TypeScript is a prerequisite for Inversify's decorator-based DI (ADR-0006); adopting it resolves the compiler configuration dependency already implied by that decision.
- `strict` mode prevents common categories of bugs (implicit `any`, unchecked nulls) from entering the codebase from the outset.
- TypeScript is widely adopted in the React ecosystem; most libraries ship type definitions, reducing friction when integrating third-party packages.

**Negative**
- TypeScript adds a compilation step and requires `tsconfig.json` to be maintained alongside the Webpack configuration (ADR-0005); misconfiguration can produce subtle build or type-checking discrepancies.
- Developers unfamiliar with TypeScript's type system face a learning curve, particularly around generics, utility types, and decorator metadata.
- Enabling `experimentalDecorators` and `emitDecoratorMetadata` relies on a non-finalised decorator proposal (as noted in ADR-0006); this configuration may require revisiting if the TC39 standard diverges.

**Neutral**
- A `tsconfig.json` covering compiler options (`strict`, `experimentalDecorators`, `emitDecoratorMetadata`, module resolution) must be established as an architectural enabler before feature development begins, coordinated with the shared Webpack configuration from ADR-0005.
- Jest (ADR-0005) must be configured to handle TypeScript files, either via `ts-jest` or via `babel-loader` with `@babel/preset-typescript`; this configuration should be aligned with the Webpack transpilation choice.
- Decisions about TypeScript path aliases, project references for layer separation, and strict null-handling conventions will need to be addressed in follow-up tickets or ADRs.
- Any future change to the frontend language or transpilation toolchain would require a superseding ADR.
