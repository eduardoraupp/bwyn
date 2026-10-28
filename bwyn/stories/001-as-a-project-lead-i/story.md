# Story: Register a new project

- **Story**: 001-as-a-project-lead-i
- **Created**: 2026-10-28
- **Status**: Draft

## Story
As a project lead, I want to register a new project with its name and an optional description, so that team members have a shared workspace to track upcoming deliverables.

## Acceptance tests
1. **Given** I am on the project creation screen, **When** I submit a project with a valid name and no description, **Then** the project is saved and I am redirected to the landing page, and a confirmation message confirms the project was created.
2. **Given** I am on the project creation screen, **When** I submit a project with a valid name and a description, **Then** the project is saved and I am redirected to the landing page, and a confirmation message confirms the project was created.
3. **Given** I am on the project creation screen, **When** I attempt to submit without entering a name, **Then** the submission is prevented and an inline validation error indicates that name is required.

## Business rules and constraints
- A project must have a name; description is optional.
- The project creation form lives in the `ui` layer (React component) and delegates to a ViewModel in the `interface-adapters` layer; no business logic resides in the component (ADR-0007, Architecture).
- The ViewModel calls a use case in the `use-cases` layer; the use case does not reference any React or infrastructure concern (ADR-0007, Architecture).
- On the backend, the command to create a project is handled by an application-layer command handler; the handler does not depend on any adapter-layer class (ADR-0017, Architecture).
- The domain layer contains the Project entity; it has no dependency on Spring, JPA, or any adapter (ADR-0017, Architecture).
- Persistence is handled by an outbound adapter that implements the repository port defined in the domain layer; the adapter uses Spring Data JPA with Hibernate and Flyway manages the schema (ADR-0011, ADR-0010).
- The backend API endpoint is documented with OpenAPI annotations on the inbound adapter (Spring MVC controller); no domain class appears directly in the API schema (ADR-0014).
- The backend API base URL is configurable so the same frontend build works in both development and production deployment modes (Architecture).
- Assumption: a project name may not exceed 255 characters; this limit will be confirmed with the team before the story is played.

## Out of scope
- Listing, editing, or deleting projects.
- Assigning team members to a project.
- The landing page itself (assumed to exist or to be delivered by a separate story).
- Authentication and authorisation.
- The Inversify composition root wiring for the new use case and ViewModel (depends on AE-0002 and the ADR-0006 enabler being complete).
- The hexagonal package structure for the new bounded context (depends on AE-0005 being complete).
