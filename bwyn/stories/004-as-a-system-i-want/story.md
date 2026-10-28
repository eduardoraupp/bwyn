# Story: Publish an event when a new user is created

- **Story**: 004-as-a-system-i-want
- **Created**: 2026-10-27
- **Status**: Draft

## Story
As a system, I want to publish an event whenever a new user account is created, so that other parts of the system can react to the registration without being tightly coupled to it.

## Acceptance tests
1. **Given** a new user account has just been successfully created, **When** the creation completes, **Then** an event containing the new user's identifier and the timestamp of creation is published.
2. **Given** the event has been published, **When** a subscriber consumes it, **Then** the event payload includes at minimum the user identifier and the creation timestamp.
3. **Given** the user creation fails (for example, due to a validation error or a database constraint violation), **When** the failure occurs, **Then** no event is published.

## Business rules and constraints
- The event must be published only after the user record has been durably persisted in PostgreSQL, so that no event is emitted for a creation that was rolled back (ADR 0001).
- The system has a single developer with no dedicated QA resource; the story must be small enough to build, test, and ship without handoff (Driver: Constraints).
- Assumption: the event is published in-process (for example, via an application-level event mechanism) unless a message broker is already in place. If a broker exists or is required, this should be confirmed before implementation begins.

## Out of scope
- Defining or building any subscriber or consumer of the event.
- Choosing or configuring a message broker or external event bus.
- Retry or dead-letter handling for failed event delivery.
- Changes to the user creation flow itself.
