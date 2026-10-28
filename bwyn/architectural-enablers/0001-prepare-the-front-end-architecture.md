# AE-0001. Frontend Architecture Scaffold

- **Status**: Proposed
- **Created**: 2026-10-28
- **Completed**:
- **Implements ADR(s)**: ADR-0005 (Webpack, React, Jest), ADR-0006 (Inversify), ADR-0007 (Clean Architecture with MVVM), ADR-0015 (TypeScript)

## Goal
Put in place the complete frontend skeleton — folder structure, build toolchain, TypeScript compiler, DI container, and layer conventions — so that every subsequent feature story has a working, correctly wired foundation to build on. No feature story may be started until this enabler is complete.

## Scope

**In scope**
- `package.json` with all required runtime and development dependencies (React, TypeScript, Webpack, Babel, Jest, Inversify, `reflect-metadata`) (ADR-0005, ADR-0006, ADR-0015)
- `tsconfig.json` with `strict`, `experimentalDecorators`, and `emitDecoratorMetadata` enabled, and module resolution aligned with Webpack (ADR-0015, ADR-0006)
- Shared Webpack configuration split into `webpack.common.js`, `webpack.dev.js`, and `webpack.prod.js`, with a TypeScript loader and a Babel/JSX transform (ADR-0005, ADR-0015)
- Jest configuration that handles `.ts` and `.tsx` files using the same transpilation choice as Webpack (ADR-0005, ADR-0015)
- Reference folder structure that maps to Clean Architecture layers: `src/entities`, `src/use-cases`, `src/interface-adapters` (ViewModels), and `src/ui` (React components) (ADR-0007)
- Application entry point (`src/index.tsx`) that imports `reflect-metadata` before any other module (ADR-0006)
- Composition root (`src/composition-root.ts`) that creates and exports the Inversify container, with no bindings yet (ADR-0006, ADR-0007)
- A placeholder React component in `src/ui` and a placeholder ViewModel in `src/interface-adapters` that together prove the end-to-end wiring compiles and renders (ADR-0007)
- A passing Jest smoke test for the placeholder ViewModel that injects a test double via the container (ADR-0005, ADR-0006)
- A `README` section documenting the layer conventions and the rule that dependencies point inward (ADR-0007)

## Tasks
- [ ] Initialise `package.json`; add React, ReactDOM, Inversify, and `reflect-metadata` as runtime dependencies
- [ ] Add TypeScript, Webpack, Babel (with `@babel/preset-typescript`, `@babel/preset-react`, and decorator plugin), `ts-loader` or `babel-loader`, Jest, and `@types/*` packages as development dependencies
- [ ] Create `tsconfig.json` with `strict: true`, `experimentalDecorators: true`, `emitDecoratorMetadata: true`, and module resolution consistent with Webpack output (ADR-0015)
- [ ] Create `webpack.common.js` with entry point, TypeScript/Babel loader, and resolve extensions; create `webpack.dev.js` and `webpack.prod.js` extending it (ADR-0005)
- [ ] Configure Jest (`jest.config.ts`) to transform `.ts`/`.tsx` files using the same transpiler chosen for Webpack, and set up module resolution to match (ADR-0005, ADR-0015)
- [ ] Create the reference folder structure: `src/entities`, `src/use-cases`, `src/interface-adapters`, `src/ui` with `.gitkeep` files where empty (ADR-0007)
- [ ] Create `src/index.tsx` with `import 'reflect-metadata'` as the first statement, mounting the placeholder React component (ADR-0006)
- [ ] Create `src/composition-root.ts` with an empty Inversify `Container` exported as the composition root (ADR-0006, ADR-0007)
- [ ] Create a placeholder ViewModel in `src/interface-adapters` decorated with `@injectable()` and a placeholder React component in `src/ui` that consumes it (ADR-0007)
- [ ] Write a Jest smoke test that binds the placeholder ViewModel in a test container, resolves it, and asserts it is defined (ADR-0005, ADR-0006)
- [ ] Add a `README` section describing the four layers, the inward dependency rule, and the composition root's role (ADR-0007)
- [ ] Verify `npm run build` (production Webpack) and `npm test` (Jest) both pass with no errors

## Definition of done
- [ ] `npm run build` produces a production bundle with no TypeScript or Webpack errors
- [ ] `npm test` runs the full Jest suite (including the ViewModel smoke test) and all tests pass
- [ ] `tsc --noEmit` exits with code 0, confirming `strict` mode and decorator options are correctly configured (ADR-0015)
- [ ] The four Clean Architecture layer folders exist under `src/` and are referenced in the `README` (ADR-0007)
- [ ] `src/index.tsx` imports `reflect-metadata` before any other import, verified by code review (ADR-0006)
- [ ] The composition root exports an Inversify `Container` and is the only place in production code where the container is constructed (ADR-0006, ADR-0007)
- [ ] The placeholder ViewModel smoke test passes, confirming decorator metadata and container resolution work end-to-end (ADR-0006)
- [ ] A code-review sign-off confirms no layer boundary violations exist in the scaffold (ADR-0007, Practice: Code review)
