# Story: Register a new project

- **Story**: 001-as-a-project-lead-i
- **Created**: 2026-10-28
- **Status**: Draft

## Story
As a project lead, I want to register a new project with its name and an optional description, so that team members have a shared workspace to track upcoming deliverables.

## Acceptance tests
- [ ] **Given** I am on the project creation screen, **When** I submit a project with a valid name and no description, **Then** the project is saved and I am redirected to the landing page, and a confirmation message confirms the project was created.
- [ ] **Given** I am on the project creation screen, **When** I submit a project with a valid name and a description, **Then** the project is saved and I am redirected to the landing page, and a confirmation message confirms the project was created.
- [ ] **Given** I am on the project creation screen, **When** I attempt to submit without entering a name, **Then** the submission is prevented and an inline validation error indicates that name is required.
- [ ] **Given** I submit a project name that exceeds 255 characters, **When** the form is submitted, **Then** the submission is prevented and an inline validation error indicates the name is too long.

## Business rules and constraints
- A project must have a name; description is optional.
- A project name may not exceed 255 characters (Assumption: to be confirmed with the team before the story is played).
- The project creation form lives in the `ui` layer (React component) and delegates to a ViewModel in the `interface-adapters` layer; no business logic resides in the component (ADR-0007, Architecture).
- The ViewModel calls a use case in the `use-cases` layer; the use case does not reference any React or infrastructure concern (ADR-0007, Architecture).
- On the backend, the command to create a project is handled by an application-layer command handler; the handler does not depend on any adapter-layer class (ADR-0017, Architecture).
- The domain layer contains the `Project` entity and its repository port; neither has any dependency on Spring, JPA, or any adapter (ADR-0017, Architecture).
- The inbound adapter is a Spring MVC controller; it translates the HTTP request into a command and delegates to the application-layer command handler (ADR-0008, ADR-0017, Architecture).
- Persistence is handled by an outbound adapter that implements the repository port defined in the domain layer; the adapter uses Spring Data JPA with Hibernate and Flyway manages the schema (ADR-0011, ADR-0010).
- The backend API endpoint is documented with OpenAPI annotations on the inbound adapter; no domain class appears directly in the API schema (ADR-0014).
- The backend API base URL is configurable so the same frontend build works in both development and production deployment modes (Architecture).

## Out of scope
- Listing, editing, or deleting projects.
- Assigning team members to a project.
- The landing page itself (assumed to exist or to be delivered by a separate story).
- Authentication and authorisation.
- The Inversify composition root wiring for the new use case and ViewModel (depends on AE-0002 and the ADR-0006 enabler being complete).
- The hexagonal package structure for the new bounded context (depends on AE-0005 being complete).

## Tasks
- [ ] Create the `Project` domain entity and repository port in the domain layer with name and optional description fields, enforcing the 255-character name constraint (ADR-0017, Architecture)
- [ ] Write a Flyway migration script that creates the `project` table with `id`, `name`, and `description` columns (ADR-0010)
- [ ] Implement the `CreateProjectCommand` and its application-layer command handler, delegating persistence through the repository port (ADR-0017, Architecture)
- [ ] Implement the outbound JPA adapter that maps between the `Project` domain entity and its JPA persistence class, and implements the repository port (ADR-0011, ADR-0017)
- [ ] Write a Testcontainers integration test that verifies the Flyway migration applies cleanly and the outbound adapter persists and retrieves a project correctly (ADR-0012)
- [ ] Implement the Spring MVC inbound adapter (controller) with a `POST` endpoint, annotated with OpenAPI annotations; translate the request body into the command and return a confirmation response (ADR-0008, ADR-0014)
- [ ] Write a unit test for the command handler using a test-double repository port
- [ ] Write a unit test for the inbound adapter using MockMvc, covering the valid-name, valid-name-with-description, missing-name, and name-too-long cases
- [ ] Define the `CreateProjectUseCase` port and `CreateProjectViewModel` in the frontend `use-cases` and `interface-adapters` layers respectively, wiring the ViewModel to call the backend API (ADR-0007, Architecture)
- [ ] Implement the project creation React component in the `ui` layer, delegating all logic to the ViewModel; include inline validation for the required name and 255-character limit (ADR-0007, Architecture)
- [ ] Write Jest unit tests for the ViewModel covering the four acceptance-test scenarios
- [ ] Write a Jest rendering test for the React component covering the validation error states
- [ ] Verify the full build and test suite passes in CI within ten minutes (Practice: Ten-Minute Build, Practice: Continuous Integration)
