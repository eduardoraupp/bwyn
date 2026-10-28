# Story: Register a new project

- **Story**: 001-as-a-project-lead-i
- **Created**: 2026-10-28
- **Status**: Draft

## Story
As a project lead, I want to register a new project with its name and an optional description, so that team members have a shared workspace to track upcoming deliverables.

## Acceptance tests
1. **Given** I am on the project creation screen, **When** I submit a project with a valid name and no description, **Then** the project is saved and I am redirected to the landing page and a confirmation message confirms the project was created.
2. **Given** I am on the project creation screen, **When** I submit a project with a valid name and a description, **Then** the project is saved and I am redirected to the landing page and a confirmation message confirms the project was created.
3. **Given** I am on the project creation screen, **When** I attempt to submit without entering a name, **Then** the submission is prevented and an inline validation error indicates that name is required.

## Business rules and constraints
- A project must have a name; description is optional.
- Name is the only mandatory field; no other fields are introduced in this story (Practice: Stories).
- The frontend form lives in the `ui` layer; the ViewModel coordinating the submission lives in the `interface-adapters` layer; the create-project use case lives in the `use-cases` layer (ADR-0007, AE-0002).
- The backend exposes a single command endpoint that accepts name and description; the command handler lives in the application layer and invokes a domain port; the persistence adapter lives in the outbound adapter layer (ADR-0017, AE-0005).
- The domain layer has no dependency on Spring, JPA, or any adapter class (ADR-0017).
- All schema changes are managed by a Flyway migration script; Hibernate DDL auto-generation remains disabled (ADR-0010, ADR-0011).
- The backend endpoint is documented with OpenAPI annotations on the inbound adapter; no domain class appears directly in the API schema (ADR-0014).
- The frontend API base URL is configurable and same-origin by default in production, so the same frontend build works in both development and production deployment modes (Architecture).
- Assumption: a project name may be at most 255 characters; no other length constraint is specified in the request.

## Out of scope
- Listing, editing, or deleting projects.
- Project membership or team-member invitation.
- Any field beyond name and description.
- Authentication and authorisation.
- The landing page itself (assumed to exist or to be built in a separate story).
