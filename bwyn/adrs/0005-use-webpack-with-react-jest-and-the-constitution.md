# 0005. Use Webpack with React, Jest and the Constitution Frontend Stack

- **Status**: Accepted
- **Date**: 2026-10-28

## Context

The bwyn project requires a frontend build and test toolchain. The organisation maintains a constitutionally mandated frontend stack that prescribes React as the UI library and Jest as the test framework. A module bundler must be chosen to tie these tools together, handle asset pipelines, and produce optimised production bundles. Webpack is a widely adopted bundler with mature integration support for both React (via Babel/JSX transforms) and Jest (via compatible transform configuration).

## Decision

We will use Webpack as the module bundler for the bwyn frontend, together with React as the UI library and Jest as the test framework, in conformance with the organisation's constitution frontend stack.

## Consequences

**Positive**
- Alignment with the organisation's mandated frontend stack reduces friction for developers already familiar with it and eases cross-team support.
- Webpack's large ecosystem provides well-maintained loaders and plugins for CSS, images, code splitting, and tree shaking.
- Jest integrates cleanly with the Babel transform already required for React/JSX, keeping the toolchain consistent.

**Negative**
- Webpack configuration can become complex and verbose as the project grows; ongoing maintenance of the config is required.
- Build times for large bundles can be slow without additional optimisation (e.g. caching, thread-loader).

**Neutral**
- A shared Webpack configuration (common/dev/prod split) will need to be established as an architectural enabler before feature development begins.
- Decisions about code-splitting strategy, bundle analysis, and environment-specific configuration will need to be made in follow-up ADRs or tickets.
- Any future migration to an alternative bundler (e.g. Vite, esbuild) would require a superseding ADR.
