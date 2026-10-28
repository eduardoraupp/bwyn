# AE-0003. ArchUnitTS Fitness Functions for Frontend Layer Boundaries

- **Status**: Proposed
- **Created**: 2026-10-28
- **Completed**:
- **Implements ADR(s)**: ADR-0007 Use Clean Architecture with MVVM for the Frontend, ADR-0016 Use ArchUnitTS for Frontend Architectural Fitness Functions

## Goal
Put in place an initial suite of ArchUnitTS rules, expressed as Jest tests, that enforce the Clean Architecture layer dependency constraints, MVVM structural conventions, and naming rules agreed in ADR-0007, using the ArchUnitTS tooling mandated by ADR-0016. Feature stories cannot be considered architecturally safe until these checks run automatically on every test run and pass against the folder structure established in AE-0002.

## Scope
**In scope**
- Install and configure the `archunit-ts` package as a development dependency, aligned with the Jest configuration from ADR-0005 and the `tsconfig.json` from ADR-0015.
- Write a Jest test file containing ArchUnitTS rules that enforce the Clean Architecture inward-dependency rule: entities ← use cases ← interface adapters ← framework/UI (ADR-0007).
- Write rules that enforce MVVM conventions: ViewModels reside in the interface-adapter layer; React components reside in the framework/UI layer and do not import directly from the entity or use-case layers (ADR-0007).
- Write naming-convention rules: files in the ViewModel layer match `*.viewmodel.ts`; files in the use-case layer match `*.usecase.ts`; entity files match `*.entity.ts` (ADR-0007).
- Write a rule that the Inversify composition root is the only module permitted to import from the DI container configuration (ADR-0006, ADR-0007).
- Verify that all rules pass against the scaffold produced by AE-0001 and AE-0002 with zero violations.
- Confirm the ArchUnitTS test file is picked up by the existing Jest configuration and runs as part of `npm test` (or equivalent).

**Out of scope**
- Rules for backend layers (covered by ArchUnit under ADR-0009).
- Rules for Inversify binding scope (singleton vs. transient); deferred per ADR-0006.
- Rules for cycle detection beyond the inward-dependency constraint.
- Rules for OpenAPI annotation coverage (backend concern, ADR-0014).
- Any changes to the folder structure itself (established in AE-0002).

## Tasks
- [ ] Add `archunit-ts` (and any required peer dependencies) to `devDependencies` in `package.json`.
- [ ] Verify that the ArchUnitTS import-resolution mode is compatible with the `tsconfig.json` module-resolution settings from ADR-0015; adjust `tsconfig.json` paths if needed.
- [ ] Create `src/architecture/layer-rules.test.ts` as the home for all ArchUnitTS fitness-function tests.
- [ ] Implement the Clean Architecture inward-dependency rule: entities must not import from use cases, interface adapters, or the framework layer; use cases must not import from interface adapters or the framework layer; interface adapters must not import from the framework layer (ADR-0007).
- [ ] Implement the MVVM convention rule: React components (files under `framework/`) must not import directly from `entities/` or `usecases/` (ADR-0007).
- [ ] Implement the ViewModel placement rule: all ViewModel files must reside in the interface-adapter layer and not in the framework/UI or domain layers (ADR-0007).
- [ ] Implement the ViewModel naming rule: all files under the interface-adapter layer whose name ends in `ViewModel` must match the pattern `*.viewmodel.ts` (ADR-0007).
- [ ] Implement the use-case naming rule: all files under `usecases/` must match the pattern `*.usecase.ts` (ADR-0007).
- [ ] Implement the entity naming rule: all files under `entities/` must match the pattern `*.entity.ts` (ADR-0007).
- [ ] Implement the composition-root isolation rule: only the composition-root module may import from the Inversify container configuration file (ADR-0006, ADR-0007).
- [ ] Run the full Jest suite and confirm all ArchUnitTS rules pass with zero violations against the current scaffold.
- [ ] Confirm total test-suite duration remains within the ten-minute build budget (Practice: Ten-Minute Build).

## Definition of done
- [ ] `archunit-ts` is listed in `devDependencies` and installs cleanly with no peer-dependency conflicts.
- [ ] `npm test` (or equivalent) executes `layer-rules.test.ts` without requiring any additional command or flag.
- [ ] The inward-dependency rule fails with a clear violation message when a deliberate test import is added that crosses a layer boundary, and passes when that import is removed (ADR-0007).
- [ ] The MVVM convention rule fails when a React component file is given a direct import from `entities/`, and passes when that import is removed (ADR-0007).
- [ ] The ViewModel placement rule fails when a ViewModel file is placed outside the interface-adapter layer, and passes when it is moved to the correct location (ADR-0007).
- [ ] The ViewModel, use-case, and entity naming rules each fail when a file with a non-conforming name is added to the respective layer, and pass when the file is renamed correctly (ADR-0007).
- [ ] The composition-root isolation rule fails when a non-composition-root module imports the container configuration, and passes when that import is removed (ADR-0006, ADR-0007).
- [ ] All rules pass against the unmodified scaffold from AE-0002 with zero violations reported.
- [ ] The full test suite, including the new ArchUnitTS tests, completes within the ten-minute build budget (Practice: Ten-Minute Build).
- [ ] The `layer-rules.test.ts` file is reviewed and merged via a pull request (Practice: Code Review).
