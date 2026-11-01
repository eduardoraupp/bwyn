# 0020. Use React Router for Frontend Client-Side Routing

- **Status**: Accepted
- **Date**: 2026-11-01

## Context

The bwyn frontend is a React single-page application (ADR-0005) structured with Clean Architecture and MVVM (ADR-0007), implemented in TypeScript (ADR-0015), and built with Webpack (ADR-0005). As the application grows, it will present multiple distinct views or screens to the user. Without a routing solution, navigating between views requires custom state management, the browser's address bar does not reflect the current view, deep-linking and bookmarking are not supported, and the browser's back and forward buttons do not behave as users expect.

React Router is the de-facto standard client-side routing library for React applications. It maps URL paths to React components, manages browser history, and provides hooks and components for declarative navigation. React Router v6 integrates naturally with React's component model and TypeScript (ADR-0015), and requires no additional build tooling beyond what Webpack (ADR-0005) already provides.

## Decision

We will use React Router (v6) as the client-side routing solution for the bwyn frontend. Route definitions will be declared in a dedicated routing configuration that resides in the framework/UI layer (ADR-0007). React components acting as Views will be mapped to routes; no routing concerns will leak into ViewModels or use-case layers. Navigation will be performed through React Router's provided hooks and components (`useNavigate`, `<Link>`, `<NavLink>`) rather than direct manipulation of the browser history API.

## Consequences

**Positive**
- URL paths reflect the current view, enabling deep-linking, bookmarking, and correct browser back/forward navigation without custom implementation.
- React Router v6's hook-based API (`useNavigate`, `useParams`, `useLocation`) integrates cleanly with TypeScript (ADR-0015) and React functional components, keeping routing code concise and type-safe.
- Confining route definitions to the framework/UI layer (ADR-0007) keeps routing concerns out of ViewModels and use cases, preserving Clean Architecture layer boundaries.
- React Router is widely adopted in the React ecosystem, meaning extensive documentation and developer familiarity are readily available.

**Negative**
- Introducing a client-side router requires the backend (ADR-0008) to serve the frontend's `index.html` for all non-API routes (a catch-all route), so that direct URL access and page refreshes work correctly; this is a deployment and server configuration concern that must be addressed.
- React Router's API has changed significantly across major versions; future major-version upgrades may require non-trivial migration work.
- Developers unfamiliar with React Router v6's data-router model or its differences from v5 face a small learning curve.

**Neutral**
- A top-level route configuration module must be established as an architectural enabler before feature views are added; its placement within the framework/UI layer should be consistent with the reference folder structure defined under ADR-0007.
- ArchUnitTS fitness functions (ADR-0016) should be extended to verify that routing imports (`react-router-dom`) appear only in the framework/UI layer and not in ViewModel, use-case, or entity layers.
- Decisions about route-level code splitting (using React's `lazy` and `Suspense` with Webpack dynamic imports, ADR-0005) will be addressed in follow-up tickets.
- Any future change of routing library would require a superseding ADR.
