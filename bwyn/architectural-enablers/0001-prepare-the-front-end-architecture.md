# AE-0001. Frontend Build and DI Scaffold

- **Status**: Proposed
- **Created**: 2026-10-28
- **Completed**:
- **Implements ADR(s)**: ADR-0005 (Webpack, React, Jest), ADR-0006 (Inversify), ADR-0015 (TypeScript)

## Goal
Put in place the frontend build toolchain, TypeScript compiler configuration, and Inversify DI container so that every subsequent feature story has a working, correctly wired foundation to build on. No feature story may be started until this enabler is complete.

## Scope

**In scope**
- `package.json` with all required runtime and development dependencies (React, TypeScript, Webpack, Babel, Jest, Inversify, `reflect-metadata`) (ADR-0005, ADR-0006, ADR-0015)
- `tsconfig.json` with `strict`, `experimentalDecorators`, and `emitDecoratorMetadata` enabled, and module resolution aligned with Webpack (ADR-0015, ADR-0006)
- Shared Webpack configuration split into `webpack.common.js`, `webpack.dev.js`, and `webpack.prod.js`, with a TypeScript loader and a Babel/JSX transform (ADR-0005, ADR-0015)
- Jest configuration that handles `.ts` and `.tsx` files using the same transpilation choice as Webpack (ADR-0005, ADR-0015)
- Application entry point (`src/index.tsx`) that imports `reflect-metadata` before any other module (ADR-0006)
- Composition root (`src/composition-root.ts`) that creates and exports the Inversify container, with no bindings yet (ADR-0006)
- A placeholder React component in `src/ui` and a placeholder injectable class that together prove the end-to-end wiring compiles and renders (ADR-0005, ADR-0006)
- A passing Jest smoke test for the placeholder injectable that resolves it via the container using a test double (ADR-0005, ADR-0006)

**Out of scope**
- Clean Architecture folder structure and layer conventions (covered by a follow-up architectural enabler)
- ViewModel pattern and MVVM wiring (covered by a follow-up architectural enabler)
- Any feature bindings in the composition root

## Tasks
- [ ] Initialise `package.json`; add React, ReactDOM, Inversify, and `reflect-metadata` as runtime dependencies
- [ ] Add TypeScript, Webpack, Babel (with `@babel/preset-typescript`, `@babel/preset-react`, and decorator plugin), `ts-loader` or `babel-loader`, Jest, and `@types/*` packages as development dependencies
- [ ] Create `tsconfig.json` with `strict: true`, `experimentalDecorators: true`, `emitDecoratorMetadata: true`, and module resolution consistent with Webpack output (ADR-0015)
- [ ] Create `webpack.common.js` with entry point, TypeScript/Babel loader, and resolve extensions; create `webpack.dev.js` and `webpack.prod.js` extending it (ADR-0005)
- [ ] Configure Jest (`jest.config.ts`) to transform `.ts`/`.tsx` files using the same transpiler chosen for Webpack, and set up module resolution to match (ADR-0005, ADR-0015)
- [ ] Create `src/index.tsx` with `import 'reflect-metadata'` as the first statement, mounting the placeholder React component (ADR-0006)
- [ ] Create `src/composition-root.ts` with an empty Inversify `Container` exported as the composition root (ADR-0006)
- [ ] Create a placeholder injectable class decorated with `@injectable()` and a placeholder React component in `src/ui` that consumes it (ADR-0006)
- [ ] Write a Jest smoke test that binds the placeholder injectable in a test container, resolves it, and asserts it is defined (ADR-0005, ADR-0006)
- [ ] Verify `npm run build` (production Webpack) and `npm test` (Jest) both pass with no errors

## Definition of done
- [ ] `npm run build` produces a production bundle with no TypeScript or Webpack errors
- [ ] `npm test` runs the full Jest suite (including the injectable smoke test) and all tests pass
- [ ] `tsc --noEmit` exits with code 0, confirming `strict` mode and decorator options are correctly configured (ADR-0015)
- [ ] `src/index.tsx` imports `reflect-metadata` before any other import, verified by code review (ADR-0006)
- [ ] The composition root exports an Inversify `Container` and is the only place in production code where the container is constructed (ADR-0006)
- [ ] The placeholder injectable smoke test passes, confirming decorator metadata and container resolution work end-to-end (ADR-0006)
- [ ] A code-review sign-off confirms the scaffold contains no premature layer structure (Practice: Code review)
