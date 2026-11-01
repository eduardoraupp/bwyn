# AE-0001. Frontend Build and DI Scaffold

- **Status**: Proposed
- **Created**: 2026-10-28
- **Completed**:
- **Implements ADR(s)**: ADR-0005 (Webpack, React, Jest), ADR-0006 (Inversify), ADR-0015 (TypeScript), ADR-0019 (Maven frontend module), ADR-0020 (React Router)

## Goal
Put in place the frontend Maven module, build toolchain, TypeScript compiler configuration, Inversify DI container, and React Router top-level route configuration so that every subsequent feature story has a working, correctly wired foundation to build on. No feature story may be started until this enabler is complete.

## Scope

**In scope**
- A `bwyn-frontend/` Maven module added to the root multi-module project, with a `pom.xml` that declares the `frontend-maven-plugin`, pins the Node.js and npm versions, and binds `npm install`, `npm test`, and `npm run build` to the correct Maven lifecycle phases (ADR-0019)
- The root parent `pom.xml` updated to include `bwyn-frontend` as a module (ADR-0019)
- `bwyn-frontend/package.json` with all required runtime and development dependencies (React, TypeScript, Webpack, Babel, Jest, Inversify, `reflect-metadata`, `react-router-dom`) (ADR-0005, ADR-0006, ADR-0015, ADR-0020)
- `bwyn-frontend/tsconfig.json` with `strict`, `experimentalDecorators`, and `emitDecoratorMetadata` enabled, and module resolution aligned with Webpack (ADR-0015, ADR-0006)
- Shared Webpack configuration split into `webpack.common.js`, `webpack.dev.js`, and `webpack.prod.js`, with a TypeScript loader and a Babel/JSX transform, located under `bwyn-frontend/` (ADR-0005, ADR-0015)
- Jest configuration that handles `.ts` and `.tsx` files using the same transpilation choice as Webpack, located under `bwyn-frontend/` (ADR-0005, ADR-0015)
- Application entry point (`bwyn-frontend/src/index.tsx`) that imports `reflect-metadata` before any other module (ADR-0006)
- Composition root (`bwyn-frontend/src/composition-root.ts`) that creates and exports the Inversify container, with no bindings yet (ADR-0006)
- A top-level route configuration module in `bwyn-frontend/src/ui/routes.tsx` that declares a placeholder route using React Router v6, residing in the framework/UI layer (ADR-0020)
- A placeholder React component in `bwyn-frontend/src/ui` and a placeholder injectable class that together prove the end-to-end wiring compiles and renders (ADR-0005, ADR-0006)
- A passing Jest smoke test for the placeholder injectable that resolves it via the container using a test double (ADR-0005, ADR-0006)

**Out of scope**
- Clean Architecture folder structure and layer conventions (covered by a follow-up architectural enabler)
- ViewModel pattern and MVVM wiring (covered by a follow-up architectural enabler)
- Any feature bindings in the composition root
- ArchUnitTS rules restricting `react-router-dom` imports to the framework/UI layer (covered by AE-0003)
- Decisions about how the packaged frontend bundle is served (ADR-0019)
- Backend catch-all route configuration for direct URL access (ADR-0020)

## Tasks
- [ ] Add a `bwyn-frontend/` directory and create `bwyn-frontend/pom.xml` declaring the `frontend-maven-plugin` with explicit Node.js and npm versions (ADR-0019)
- [ ] Update the root parent `pom.xml` to include `bwyn-frontend` as a module and manage the `frontend-maven-plugin` version in `<pluginManagement>` (ADR-0019)
- [ ] Bind `npm install` to the `generate-resources` phase, `npm test` to the `test` phase, and `npm run build` to the `prepare-package` phase in `bwyn-frontend/pom.xml` (ADR-0019)
- [ ] Initialise `bwyn-frontend/package.json`; add React, ReactDOM, Inversify, `reflect-metadata`, and `react-router-dom` as runtime dependencies
- [ ] Add TypeScript, Webpack, Babel (with `@babel/preset-typescript`, `@babel/preset-react`, and decorator plugin), `ts-loader` or `babel-loader`, Jest, and `@types/*` packages as development dependencies
- [ ] Create `bwyn-frontend/tsconfig.json` with `strict: true`, `experimentalDecorators: true`, `emitDecoratorMetadata: true`, and module resolution consistent with Webpack output (ADR-0015)
- [ ] Create `bwyn-frontend/webpack.common.js` with entry point, TypeScript/Babel loader, and resolve extensions; create `webpack.dev.js` and `webpack.prod.js` extending it (ADR-0005)
- [ ] Configure `bwyn-frontend/jest.config.ts` to transform `.ts`/`.tsx` files using the same transpiler chosen for Webpack, and set up module resolution to match (ADR-0005, ADR-0015)
- [ ] Create `bwyn-frontend/src/index.tsx` with `import 'reflect-metadata'` as the first statement, mounting the placeholder React component wrapped in the React Router `<BrowserRouter>` (ADR-0006, ADR-0020)
- [ ] Create `bwyn-frontend/src/composition-root.ts` with an empty Inversify `Container` exported as the composition root (ADR-0006)
- [ ] Create `bwyn-frontend/src/ui/routes.tsx` declaring a single placeholder route that maps `/` to the placeholder React component, using React Router v6 (ADR-0020)
- [ ] Create a placeholder injectable class decorated with `@injectable()` and a placeholder React component in `bwyn-frontend/src/ui` that consumes it (ADR-0006)
- [ ] Write a Jest smoke test that binds the placeholder injectable in a test container, resolves it, and asserts it is defined (ADR-0005, ADR-0006)
- [ ] Verify `mvn verify` from the repository root runs `npm install`, `npm test`, and `npm run build` for the `bwyn-frontend` module with no errors (ADR-0019)

## Definition of done
- [ ] `mvn verify` from the repository root completes successfully, executing the `bwyn-frontend` `npm install`, `npm test`, and `npm run build` steps in the correct Maven lifecycle phases (ADR-0019)
- [ ] `npm run build` (invoked directly under `bwyn-frontend/`) produces a production bundle with no TypeScript or Webpack errors
- [ ] `npm test` (invoked directly under `bwyn-frontend/`) runs the full Jest suite (including the injectable smoke test) and all tests pass
- [ ] `tsc --noEmit` (run under `bwyn-frontend/`) exits with code 0, confirming `strict` mode and decorator options are correctly configured (ADR-0015)
- [ ] `bwyn-frontend/src/index.tsx` imports `reflect-metadata` before any other import, verified by code review (ADR-0006)
- [ ] The composition root exports an Inversify `Container` and is the only place in production code where the container is constructed (ADR-0006)
- [ ] The placeholder injectable smoke test passes, confirming decorator metadata and container resolution work end-to-end (ADR-0006)
- [ ] `bwyn-frontend/src/ui/routes.tsx` declares the top-level route configuration using React Router v6 and resides in the framework/UI layer; no routing imports appear outside that layer, verified by code review (ADR-0020)
- [ ] The root parent `pom.xml` lists `bwyn-frontend` as a module and manages the `frontend-maven-plugin` version centrally (ADR-0019)
- [ ] A code-review sign-off confirms the scaffold contains no premature layer structure (Practice: Code review)
