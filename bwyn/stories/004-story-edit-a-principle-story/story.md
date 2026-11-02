# Story: Edit a principle

- **Story**: 004-story-edit-a-principle-story
- **Created**: 2026-11-02
- **Status**: Draft

## Story

As a team member, I want to open a principle and change its name or its description, so that our principles stay true and keep helping us decide.

## Acceptance tests

Opening

- [ ] **Given** the Principles tab with principles, **When** the user presses a card (or uses Enter or Space on it), **Then** a dialog titled "Edit principle" opens, with the Name and the Description filled in with what was saved, the Name field in focus, Cancel, and Save.
- [ ] **Given** the dialog just opened, **Then** Save is disabled, because nothing changed.
- [ ] **Given** a principle with no description, **When** it is opened, **Then** the Description is empty and shows its placeholder "What the team holds to, and why."

Changing and saving

- [ ] **Given** the dialog, **When** the user changes the name or the description, **Then** Save is enabled. **When** the user puts the text back as it was, **Then** Save is disabled again.
- [ ] **Given** a change, **When** the user presses Save, **Then** the card shows the new name and description, in the same place in the list, the dialog closes, and nothing else in the list changes.
- [ ] **Given** a name and a description with spaces around them, **When** Save, **Then** both are saved without the spaces.
- [ ] **Given** the user clears the name (or leaves only spaces), **When** Save, **Then** "Give the principle a name." shows under the field, the field is marked invalid, and nothing is saved. Typing in the field clears the message.
- [ ] **Given** the user clears the description, **When** Save, **Then** it is saved. The card then shows "Click to describe this principle." in italics.
- [ ] **Given** the user renames a principle, **Then** it is still the same principle: its position in the list does not change.
- [ ] **Given** two principles with the same name, **When** one is renamed to the name of the other, **Then** it is saved. Names do not have to be different.

Closing without saving

- [ ] **Given** the dialog, **When** the user presses Cancel, the Close (X) button or Escape, **Then** it closes and the principle is as it was.
- [ ] **Given** nothing changed, **When** the user clicks outside the dialog, **Then** it closes. **Given** there are unsaved changes, **Then** a click outside does nothing.
- [ ] **Given** the user opens the same principle again after closing without saving, **Then** the dialog shows the saved text, not the discarded changes.

After saving

- [ ] **Given** an edited principle with a description, **Then** the next story, enabler or diagram that is drafted gets the new text under "Principles of the team".

## Business rules and constraints

- A principle is a name (a few words) and a description of one to three sentences: what the team holds to, and why, written so that it helps to decide between two options.
- A principle keeps its identity for life. The identity is derived from the name when the principle is first added (the first six words, lower-case, joined by "-") and is never recalculated on a rename.
- "Changed" means any difference from the saved text. Saving trims whitespace, so a change of spaces alone can be saved and result in no visible difference.
- Escape always closes the dialog, even with unsaved changes. A click outside closes the dialog only when nothing has changed.
- Names do not have to be unique.
- The frontend calls the backend over HTTP; the backend API base URL is configurable so the same frontend build works in both development and production (Architecture).

## Out of scope

- Adding a principle, removing one, and AI help with the text (each is its own story).
- Reordering principles and a history of changes to a principle.
- GitHub sync integration.

## Tasks

- [ ] Add a `PUT /api/principles/{id}` endpoint in the inbound adapter (`bwyn-rest`) that accepts `{ name, description }` and returns the updated principle (ADR-0008, ADR-0017, AE-0005).
- [ ] Add an `UpdatePrincipleCommand` and its handler in the application layer (`bwyn-application`) that trims both fields, validates the name is non-blank, and updates the principle by identity (ADR-0017, AE-0005).
- [ ] Add or extend the `Principle` aggregate in the domain core (`bwyn-core`) with an `update(name, description)` method that enforces the non-blank name invariant (ADR-0017, AE-0005).
- [ ] Add or extend the outbound adapter (`bwyn-infrastructure`) to persist the updated principle via Spring Data JPA (ADR-0011, AE-0005).
- [ ] Write a Testcontainers integration test for the repository that verifies the updated name and description are persisted and the principle's identity and position are unchanged (ADR-0012).
- [ ] Write a unit test for the `UpdatePrincipleCommand` handler covering trim-on-save, blank-name rejection, and identity preservation (ADR-0009).
- [ ] Extend ArchUnit rules to verify that the `UpdatePrincipleCommand` handler in `*.application.*` has no dependency on `*.adapter.*` (ADR-0009, AE-0005).
- [ ] Add an `EditPrincipleDialog` React component (UI layer, ADR-0007) that receives the saved name and description, opens with the Name field in focus, and renders the title "Edit principle", both fields, Cancel, and Save.
- [ ] Wire the card's click, Enter, and Space handlers in the Principles tab to open `EditPrincipleDialog` with the selected principle's data (UI layer, ADR-0007).
- [ ] Implement dirty-state tracking in the dialog's ViewModel (interface-adapters layer, ADR-0007) so that Save is enabled only when the trimmed current values differ from the saved values.
- [ ] Implement the empty-name validation in the ViewModel: block Save, show "Give the principle a name." beneath the field, mark the field invalid, and clear the message on any keystroke.
- [ ] Implement the `save(id, name, description)` use-case call (use-cases layer, ADR-0007) that calls the `PUT /api/principles/{id}` endpoint through an outbound port and updates the principle in place by identity, preserving its position.
- [ ] Update the card rendering to show the new name and description after a successful save, and to show "Click to describe this principle." in italics when the description is empty.
- [ ] Implement the closing rules in the ViewModel: Cancel and X close unconditionally; Escape closes unconditionally; a click outside closes only when nothing has changed.
- [ ] Write Jest unit tests for the ViewModel covering dirty-state, validation, trim-on-save, identity preservation, and each closing rule (ADR-0005).
- [ ] Write a Jest component test for `EditPrincipleDialog` covering initial focus, placeholder text, Save disabled state, and the invalid-name message (ADR-0005).
- [ ] Add or extend an ArchUnitTS rule to verify that `react-router-dom` and dialog-state imports do not appear in the use-cases or entities layers (ADR-0016).
- [ ] Run the full frontend and backend build and test suite and confirm it completes within the ten-minute budget (Practice: Ten-Minute Build).
