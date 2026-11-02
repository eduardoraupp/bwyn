# Story: Remove a Principle

- **Story**: 005-story-remove-a-principle-story
- **Created**: 2026-11-02
- **Status**: Draft

## Story
As a team member, I want to remove a principle the team no longer holds to, after a confirmation, so that the list and the prompts only carry principles that are still true.

## Acceptance tests

**Starting a removal**

- [ ] **Given** a principle card is shown, **Then** a red trash-icon button sits to the right of the pencil (edit) button on the card.
- [ ] **Given** the trash-icon button is pressed, **Then** a confirmation dialog appears asking the user to confirm the removal before anything is deleted.
- [ ] **Given** the confirmation dialog is open, **When** the user presses "Cancel" or Escape, **Then** the dialog closes and the principle is untouched.

**Confirming and keeping**

- [ ] **Given** the confirmation dialog is open, **When** the user presses "Confirm remove", **Then** the principle is deleted, the dialog closes, its card is gone, and the other cards stay in the same order.
- [ ] **Given** the principles tab shows a count, **When** a principle is removed, **Then** the count goes down by one.
- [ ] **Given** the removed principle was the last one, **Then** the count disappears and the empty state shows: "No principles yet" and "A principle is a short statement the team stands by, such as how it decides or what it never compromises."
- [ ] **Given** a principle was removed, **Then** it cannot be brought back from BWYN: there is no undo.

**After removing**

- [ ] **Given** a principle with a description was removed, **Then** the next story, enabler or diagram that is drafted no longer gets it under "Principles of the team".
- [ ] **Given** a principle was removed, **When** another principle is opened and the AI panel is used, **Then** the removed one is not in the context the AI is given.
- [ ] **Given** a principle was removed and later a new one is added with the same name, **Then** it is treated as a new principle and gets the file name `<name>.md` again (no "-2"), because the earlier one no longer exists.
- [ ] **Given** a removed principle was already synced, **Then** its file is not deleted from the repository and the removal is not uploaded.

**Backend endpoint**

- [ ] **Given** a `DELETE /api/projects/{projectId}/principles/{principleId}` request is made with a valid `principleId`, **Then** the backend returns `204 No Content` and the principle no longer appears in subsequent `GET` responses.
- [ ] **Given** a `DELETE /api/projects/{projectId}/principles/{principleId}` request is made with an unknown `principleId`, **Then** the backend returns `404 Not Found`.
- [ ] **Given** a `DELETE /api/projects/{projectId}/principles/{principleId}` request is made with a `projectId` that does not exist, **Then** the backend returns `404 Not Found`.

## Business rules and constraints

- Removing requires two deliberate steps: pressing the trash-icon button on the card, then pressing "Confirm remove" in the confirmation dialog. One click never deletes.
- Only a saved principle can be removed; the trash-icon button appears only on existing principle cards, not on the add form.
- Removing is permanent in BWYN and affects only this project's principles.
- Removing does not touch the repository. BWYN does not delete files by syncing: a file that is only missing here is left alone.
- While a principle dialog is open, syncing is blocked ("Save or close the principle first"), so a removal never happens in the middle of a sync.
- The trash-icon button sits to the right of the pencil (edit) button on each principle card (Architecture: UI layer, framework/UI layer per ADR-0007).
- The removal operation is handled through the use-case layer; the UI layer calls it through the ViewModel (ADR-0007).
- The backend `DELETE` endpoint lives in `PrincipleController` in the inbound adapter layer (`bwyn-rest`, `*.adapter.in.*`); it delegates to an application service in `bwyn-application` through a port, never touching the domain or persistence directly (ADR-0017, AE-0005).
- The application service invokes the repository port defined in `bwyn-core`; the Spring Data JPA implementation of that port lives in `bwyn-infrastructure` (`*.adapter.out.*`) (ADR-0017, ADR-0011, AE-0005).
- The domain core (`bwyn-core`) has no dependency on Spring, JPA, or any adapter class; ArchUnit rules enforce this (ADR-0009, AE-0005).
- The `DELETE` endpoint is documented with `@Operation` and `@ApiResponse` annotations so that the OpenAPI specification stays accurate (ADR-0014).

## Out of scope

- Deleting the principle's file from the repository or preventing it from returning on the next sync.
- An undo or archive mechanism.
- Removing several principles at once.
- Any remembered list of removed principles to prevent re-import from the repository.

## Tasks

- [ ] Add a red trash-icon button to the right of the pencil button on each saved principle card in the UI layer (ADR-0007).
- [ ] Wire the trash-icon button to open a confirmation dialog that asks the user to confirm before any deletion occurs.
- [ ] Implement "Confirm remove" and "Cancel" actions in the confirmation dialog, closing it without deleting on cancel.
- [ ] Implement the `RemovePrincipleUseCase` port interface in `bwyn-core` and its application service implementation in `bwyn-application` that deletes the principle by its identity (ADR-0017, AE-0005).
- [ ] Implement the `PrincipleRepository` outbound port method for deletion in `bwyn-core` and its Spring Data JPA adapter in `bwyn-infrastructure` (ADR-0011, AE-0005).
- [ ] Add `DELETE /api/projects/{projectId}/principles/{principleId}` to `PrincipleController` in `bwyn-rest`, returning `204 No Content` on success and `404 Not Found` when the principle or project does not exist; annotate with `@Operation` and `@ApiResponse` (ADR-0014, AE-0005).
- [ ] Update the frontend ViewModel to expose the remove action and the updated count, and reflect the empty state when the list is empty (ADR-0007).
- [ ] Implement the `remove(id)` use case call in the frontend use-case layer and wire it through the ViewModel to the UI (ADR-0007).
- [ ] Exclude removed principles from the AI context in `buildContext` so they no longer appear in prompts for stories, enablers, and diagrams.
- [ ] Release the sync block after removal so that syncing is permitted again.
- [ ] Verify the full build and test suite passes within the ten-minute budget (Practice: Ten-Minute Build).
