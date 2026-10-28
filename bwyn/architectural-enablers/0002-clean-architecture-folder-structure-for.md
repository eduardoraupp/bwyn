# AE-0002. Clean Architecture Folder Structure for the Frontend

- **Status**: Proposed
- **Created**: 2026-10-28
- **Completed**:
- **Implements ADR(s)**: ADR-0007 Use Clean Architecture with MVVM for the Frontend

## Goal
Establish the canonical folder structure and layer boundary conventions for the bwyn frontend so that every subsequent feature story has an unambiguous place to put its code and the inward-pointing dependency rule (ADR-0007) is enforced from the first commit.

## Scope
**In scope**
- A reference folder tree that maps to the four Clean Architecture layers: `entities`, `use-cases`, `interface-adapters` (containing ViewModels), and `ui` (React components) (ADR-0007)
- An index or barrel file per layer that defines the public surface of that layer, enforcing information hiding (Principle: Declarative over imperative)
- A written layer boundary convention document (e.g. `docs/architecture/frontend-layers.md`) that states which imports are permitted between layers
- At least one placeholder module in each layer (e.g. a stub entity, a stub use case, a stub ViewModel, a stub React component) so the structure is navigable and verifiable
- Jest path-alias configuration aligned with the folder structure, coordinated with the Webpack and TypeScript setup from AE-0001 (ADR-0005, ADR-0015)

**Out of scope**
- The Inversify composition root and container bindings (covered by the ADR-0006 enabler)
- Any real feature code; placeholders are stubs only
- Cross-cutting concerns such as error handling, logging, or authentication (deferred per ADR-0007)
- Backend folder structure (covered by ADR-0008 and its enablers)

## Tasks
- [ ] Agree the exact top-level folder names for the four layers with the team and record them in `docs/architecture/frontend-layers.md`
- [ ] Create the folder tree under `src/` with one placeholder module per layer
- [ ] Add a barrel (`index.ts`) to each layer folder that exports only the intended public surface
- [ ] Add TypeScript path aliases in `tsconfig.json` for each layer root (e.g. `@entities`, `@use-cases`, `@interface-adapters`, `@ui`), coordinated with the Webpack alias configuration (ADR-0005, ADR-0015)
- [ ] Configure the same aliases in Jest so tests resolve them identically (ADR-0005)
- [ ] Write the layer boundary convention document stating permitted and forbidden import directions
- [ ] Add a passing Jest test for each placeholder module to confirm the structure is importable and the aliases resolve correctly
- [ ] Verify the full build and test suite still completes within the ten-minute budget (Practice: Ten-Minute Build)

## Definition of done
- [ ] `src/` contains folders for all four Clean Architecture layers, each with at least one placeholder module and a barrel file
- [ ] `docs/architecture/frontend-layers.md` exists and states the permitted import directions between layers
- [ ] TypeScript path aliases for all four layers are present in `tsconfig.json` and resolve without error (`tsc --noEmit` passes)
- [ ] The same aliases resolve correctly in Jest (`jest --passWithNoTests` passes with no module-not-found errors)
- [ ] A placeholder import that violates the inward-pointing dependency rule (e.g. an entity importing from the UI layer) is demonstrably rejected by the TypeScript compiler or a lint rule, confirming the boundary is enforceable
- [ ] The full build and test suite passes in CI within ten minutes (Practice: Ten-Minute Build, Practice: Continuous Integration)
- [ ] The pull request has been reviewed and approved (Practice: Code review)
