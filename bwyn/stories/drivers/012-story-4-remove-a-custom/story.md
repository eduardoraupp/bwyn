# Story: Remove a custom driver

- **Story**: 012-story-4-remove-a-custom
- **Created**: 2026-11-04
- **Status**: Draft

## Story
As a team member, I want to remove a custom driver the team no longer needs, after a confirmation, so that the screen, the stories that linked to it and the prompts only carry drivers that still apply.

## Acceptance tests

**Starting a removal**
- [ ] **Given** a custom driver was opened from its card, **Then** a "Remove" button (red, with a trash icon) is on the left of the dialog's footer, opposite Cancel and Save.
- [ ] **Given** the dialog of a fixed driver, **Then** it has no "Remove" button, because fixed drivers cannot be removed.
- [ ] **Given** the dialog to add a driver, **Then** it has no "Remove" button, because there is nothing to remove yet.
- [ ] **Given** "Remove" was pressed, **Then** nothing is deleted yet: "Remove" is replaced by two buttons, "Confirm remove" and "Keep".
- [ ] **Given** the driver is linked from other documents, **When** "Remove" is pressed, **Then** a line next to the buttons says how many, for example "Linked from 2 stories and 1 enabler. Removing it also removes those links."

**Confirming and keeping**
- [ ] **Given** the two buttons, **When** the user presses "Keep", **Then** "Remove" comes back and the driver is untouched.
- [ ] **Given** the two buttons, **When** the user presses "Confirm remove", **Then** the driver is deleted, the dialog closes, its card is gone, the other cards stay in the same order and the "Add a driver" card is still last.
- [ ] **Given** the user typed changes before pressing "Confirm remove", **Then** they are discarded together with the driver and nothing is asked about them.
- [ ] **Given** the confirmation is showing, **When** the user presses Cancel, the Close (X) button or Escape, **Then** the dialog closes and the driver is untouched.
- [ ] **Given** the user pressed "Remove", then closed the dialog without confirming, **When** the driver is opened again, **Then** the dialog shows "Remove" again, not the confirmation.
- [ ] **Given** a driver was removed, **Then** it cannot be brought back from BWYN: there is no undo.

**After removing**
- [ ] **Given** stories and enablers that linked to the removed driver, **Then** the link is taken away from each of them, and the Builder no longer lists `drivers/<identity>.md`.
- [ ] **Given** a removed driver "Compliance", **When** a new driver named "Compliance" is added later, **Then** it is a new driver with the identity "compliance" again (no "-2"), and no story is linked to it.
- [ ] **Given** a driver with text was removed, **Then** the next story, enabler or diagram that is drafted is not given it, and the AI of another driver's dialog does not have it in its context.
- [ ] **Given** a removed driver was already synced, **Then** its file is not deleted from the repository, and the removal is not uploaded.

## Business rules and constraints
- Only a custom driver can be removed, and only from inside its edit dialog. Removing requires two deliberate steps: "Remove" then "Confirm remove". One click never deletes.
- Removing is permanent in BWYN and affects only this project's drivers.
- A removed driver leaves no link behind: stories and enablers that linked to it lose the link, in the same way as when any document is removed. Otherwise a new driver with the same identity would silently inherit the old links.
- Removing does not touch the repository. BWYN does not delete files by syncing: a file that is only missing here is left alone.
- While the dialog is open, syncing is blocked ("Save or close the driver first"), so a removal never happens in the middle of a sync. The block is lifted when the dialog closes, whether by confirmation or cancellation.
- The confirmation state (showing "Confirm remove" and "Keep") is local to the dialog session; it resets every time the dialog is opened.
- Unsaved edits made before pressing "Remove" are discarded together with the driver on confirmation; no separate prompt is shown for them.

## Out of scope
- Deleting the driver's file from the repository or preventing it from returning on the next sync.
- An undo or archive mechanism.
- Removing several drivers at once.
- Removing a fixed driver (clearing its text is editing, covered by story 011).

## Tasks
- [ ] Add a `RemoveDriverCommandHandler` in `bwyn-application` that accepts a driver identity, deletes the driver, and strips its link from every story and enabler that references it (ADR-0017, Architecture)
- [ ] Add a `GetDriverLinkCountQueryHandler` in `bwyn-application` that returns the count of stories and enablers linked to a given driver identity, broken down by document type (ADR-0017, Architecture)
- [ ] Add the outbound port interfaces for driver removal and link-count query in `bwyn-core`; implement them in `bwyn-infrastructure` (ADR-0017, Architecture)
- [ ] Add a Flyway migration script that supports the removal operation at the persistence level (ADR-0010)
- [ ] Expose a `DELETE /api/projects/{projectId}/drivers/{driverIdentity}` endpoint in `bwyn-rest`, annotated with OpenAPI `@Operation` and `@ApiResponse` (ADR-0014, Architecture)
- [ ] Add a `RemoveDriverUseCase` interface and a `removeDriver` method to the driver ViewModel in the interface-adapters layer, injected via Inversify (ADR-0006, ADR-0007, Architecture)
- [ ] Extend the driver edit dialog React component (ui layer) to show the "Remove" button only for custom drivers, replacing it with "Confirm remove" and "Keep" on press, and displaying the link-count line when links exist (ADR-0007, Architecture)
- [ ] Reset the confirmation state to "Remove" each time the dialog is opened, and ensure Cancel, Close (X), and Escape close the dialog without deleting (ADR-0007)
- [ ] On "Confirm remove", call the ViewModel's `removeDriver`, close the dialog, remove the card from the drivers screen, and notify the page that no dialog is open so syncing is unblocked (ADR-0007)
- [ ] Exclude removed drivers from the Builder file list, the AI prompt context, and the AI context passed to other driver dialogs (Architecture)
- [ ] Write unit tests for `RemoveDriverCommandHandler` and `GetDriverLinkCountQueryHandler` using test doubles for the outbound ports (ADR-0009)
- [ ] Write a ViewModel Jest unit test verifying the two-step confirmation flow and the reset-on-open behaviour (ADR-0005, ADR-0007)
- [ ] Write a React component Jest test verifying that the "Remove" button is absent for fixed drivers and for the add-driver dialog (ADR-0005)
- [ ] Verify the full build and test suite completes within the ten-minute budget (Practice: Ten-Minute Build)
