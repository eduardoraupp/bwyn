# Story: View and manage principles and practices on the P&P screen

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

## Tasks
- [ ] Write a Flyway migration script that creates the `principles` table with `id`, `text`, and `created_at` columns (ADR-0010)
- [ ] Add the `Principle` domain entity to the domain layer with no Spring or JPA dependency (ADR-0017, Architecture)
- [ ] Define the `PrincipleRepository` port interface in the domain layer (ADR-0017, Architecture)
- [ ] Implement the `AddPrincipleCommand` and its application-layer command handler, enforcing the 1 000-character text constraint (ADR-0017, Architecture)
- [ ] Implement the outbound adapter: a Spring Data JPA repository and JPA entity that fulfils the `PrincipleRepository` port; keep the JPA `@Entity` out of the domain layer (ADR-0011, ADR-0017, Architecture)
- [ ] Implement the inbound adapter: a Spring MVC controller with an `POST /api/principles` endpoint, annotated with OpenAPI annotations; no domain class appears in the API schema (ADR-0008, ADR-0014, Architecture)
- [ ] Write a Testcontainers integration test that verifies the Flyway migration applies cleanly and the outbound adapter persists and retrieves a principle (ADR-0012)
- [ ] Add the `AddPrinciple` use case to the `use-cases` layer; inject the repository port via Inversify (ADR-0006, ADR-0007, Architecture)
- [ ] Add the `PrinciplesViewModel` to the `interface-adapters` layer; it exposes the principle list, the add command, and validation state (ADR-0007, Architecture)
- [ ] Build the `PrinciplesTab` React component in the `ui` layer: renders the principle list, the add form with inline validation, and a confirmation message on success (ADR-0005, ADR-0007, Architecture)
- [ ] Build the `PracticesTab` React component in the `ui` layer as a read-only placeholder with no add or edit controls (ADR-0007, Architecture)
- [ ] Build the `PrinciplesAndPracticesScreen` React component in the `ui` layer: two tabs defaulting to "Principles", containing `PrinciplesTab` and `PracticesTab` (ADR-0007, Architecture)
- [ ] Add the P&P navigation item to the left-hand expandable menu in the `ui` layer and wire it to the `PrinciplesAndPracticesScreen` (ADR-0007, Architecture)
- [ ] Write Jest unit tests for the `PrinciplesViewModel` covering valid submission, empty-text rejection, and confirmation state (ADR-0005)
- [ ] Verify the full build and test suite completes within the ten-minute budget (Practice: Ten-Minute Build, Practice: Continuous Integration)
