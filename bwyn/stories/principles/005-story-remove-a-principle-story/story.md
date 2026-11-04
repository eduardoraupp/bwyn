# Story: Remove a Principle

- **Story**: 005-story-remove-a-principle-story
- **Created**: 2026-11-02
- **Status**: Draft

## Story
As a team member, I want to remove a principle the team no longer holds to, after a confirmation, so that the list and the prompts only carry principles that are still true.

## Acceptance tests

**Starting a removal**
- [X] **Given** a saved principle is open in the dialog, **Then** a "Remove" button is visible on the left of the footer, opposite the Cancel and Save buttons.
- [X] **Given** the add-new-principle dialog is open, **Then** no "Remove" button is present.
- [X] **Given** a saved principle is open and the user presses "Remove", **Then** the "Remove" button is replaced by "Confirm remove" and "Keep"; nothing is deleted yet.

**Confirming and keeping**
- [X] **Given** "Confirm remove" and "Keep" are showing, **When** the user presses "Keep", **Then** the "Remove" button returns and the principle is unchanged.
- [X] **Given** "Confirm remove" and "Keep" are showing, **When** the user presses "Confirm remove", **Then** the principle is deleted, the dialog closes, its card is gone from the list, and the remaining cards stay in the same order.
- [X] **Given** the principles tab shows a count of 3, **When** a principle is removed, **Then** the count shows 2.
- [X] **Given** the principles tab shows a count of 1, **When** the last principle is removed, **Then** the count disappears and the empty state reads "No principles yet" with the description "A principle is a short statement the team stands by, such as how it decides or what it never compromises."
- [X] **Given** the user edited the principle's name or description before pressing "Confirm remove", **Then** those unsaved edits are discarded along with the principle and no further prompt is shown.
- [X] **Given** "Confirm remove" and "Keep" are showing, **When** the user presses Cancel, the Close (X) button, or Escape, **Then** the dialog closes and the principle is untouched.
- [X] **Given** the user pressed "Remove" and then closed the dialog without confirming, **When** the same principle is opened again, **Then** the dialog shows "Remove", not "Confirm remove" and "Keep".

## Business rules and constraints
- Removal requires two deliberate actions: pressing "Remove" and then pressing "Confirm remove". A single click never deletes a principle.
- Only a saved principle can be removed; the add-new-principle dialog has no "Remove" button because there is nothing to remove yet.
- Removal is permanent within BWYN; there is no undo.
- Removal is local only: BWYN does not delete files from the repository. A previously synced principle's file remains in the repository after removal (Practice: Incremental design — the sync-deletion behaviour is deferred).
- While the principle dialog is open, syncing is blocked; a removal therefore never occurs mid-sync.
- Unsaved edits made before confirming removal are silently discarded with the principle; no additional confirmation is required for them.
- Assumption: a principle is identified by a stable internal identity, not by its name, so a new principle with the same name is independent of the removed one.

## Out of scope
- Deleting the principle's file from the repository or preventing it from returning on the next sync.
- An undo or archive mechanism.
- Removing several principles at once.
- Any remembered list of removed principles to prevent re-import from the repository.

## Tasks
- [X] Add a "Remove" button to the left of the footer in the saved-principle dialog; omit it from the add-new-principle dialog (Architecture: UI layer, `@ui`)
- [X] Implement the two-step confirmation state in the dialog: pressing "Remove" replaces it with "Confirm remove" and "Keep"; pressing "Keep" restores "Remove"; reset the state to "Remove" each time the dialog opens (Architecture: UI layer)
- [X] Implement the `RemovePrincipleUseCase` that deletes a principle by its identity from the principles store (Architecture: use-cases layer)
- [X] Wire the use case into the `PrinciplesViewModel` with an `onRemove(id)` command; close the dialog and emit the updated list on success (Architecture: interface-adapters layer, ADR-0007)
- [X] Update the principles count and empty-state display in the ViewModel so they react to the removal (Architecture: interface-adapters layer)
- [X] Add a `DELETE /api/principles/{id}` endpoint in the inbound adapter and a corresponding application service command handler; the outbound adapter deletes the record from the database (Architecture: ADR-0017, `bwyn-rest` → `bwyn-application` → `bwyn-core` → `bwyn-infrastructure`)
- [X] Write unit tests for `RemovePrincipleUseCase` covering: successful removal, removal of the last principle, and removal of a non-existent id (Architecture: `bwyn-core` / `bwyn-application`, `*Test.java`)
- [X] Write unit tests for `PrinciplesViewModel` covering: confirmation state transitions, count update, and empty-state trigger
- [X] Write an integration test for the `DELETE` endpoint verifying the principle is absent from subsequent `GET` responses (Architecture: `bwyn-rest`, `*IT.java`)
- [X] Verify the full build and test suite passes within the ten-minute budget (Practice: Ten-Minute Build, Practice: Continuous Integration)
