# Story: View and manage principles on the P&P screen

- **Story**: 002-as-i-subject-matter-expert
- **Created**: 2026-10-28
- **Status**: Draft

## Story
As a subject matter expert, I want to open the Principles & Practices screen from the landing page and add principles to it, so that the team has a shared, living record of the principles that guide their work.

## Acceptance tests
1. **Given** I am on the landing page, **When** I expand the left-hand menu and click "P&P", **Then** the Principles & Practices screen opens with two tabs: "Principles" and "Practices".
2. **Given** I am on the Principles & Practices screen, **When** the screen first loads, **Then** the "Principles" tab is active by default.
3. **Given** I am on the Principles tab, **When** I submit a new principle with valid text, **Then** the principle appears in the list on the same screen and a confirmation message indicates it was saved.
4. **Given** I am on the Principles tab, **When** I attempt to submit a principle with no text, **Then** the submission is prevented and an inline validation error indicates that text is required.
5. **Given** I am on the Practices tab, **When** the tab is active, **Then** the tab content is visible but no add or edit controls are present, confirming the tab is read-only.

## Business rules and constraints
- The left-hand expandable menu and the P&P navigation item live in the `ui` layer; no business logic resides in the component (ADR-0007, Architecture).
- The Principles tab delegates to a ViewModel in the `interface-adapters` layer; the ViewModel calls a use case in the `use-cases` layer (ADR-0007, Architecture).
- A principle must have text; no other fields are required at this stage.
- Assumption: principle text may not exceed 1 000 characters; this limit will be confirmed with the team before the story is played.
- On the backend, the command to add a principle is handled by an application-layer command handler; the handler does not depend on any adapter-layer class (ADR-0017, Architecture).
- The domain layer contains the Principle entity; it has no dependency on Spring, JPA, or any adapter (ADR-0017, Architecture).
- Persistence is handled by an outbound adapter implementing the repository port defined in the domain layer; Flyway manages the schema (ADR-0010, ADR-0011).
- The backend API endpoint is documented with OpenAPI annotations on the inbound adapter; no domain class appears directly in the API schema (ADR-0014).
- The Practices tab is read-only; its content is not yet defined and will be provided in a later story.
- The backend API base URL is configurable so the same frontend build works in both development and production deployment modes (Architecture).

## Out of scope
- Editing or deleting existing principles.
- The content and data model for the Practices tab (read-only placeholder only).
- Ordering or categorising principles.
- Authentication and authorisation.
- The landing page itself and the left-hand menu beyond the P&P navigation item (assumed to be delivered by story 001 or a separate story).
