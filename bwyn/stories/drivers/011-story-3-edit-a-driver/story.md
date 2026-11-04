# Story: Edit a driver

- **Story**: 011-story-3-edit-a-driver
- **Created**: 2026-11-04
- **Status**: Draft

## Story
As a team member, I want to open any driver and change its text (and its name when it is one of ours), optionally with AI help, so that the drivers stay true and keep steering our decisions and stories.

## Acceptance tests

**Opening a fixed driver**

- [ ] **Given** the Constraints card, **When** the user presses it, **Then** a dialog titled "Constraints" opens with its meaning under the title, no Name field, the Description in focus with the saved text, a collapsed "Improve with AI" button, Cancel, and Save, which is disabled. There is no "Remove" button.

**Opening a custom driver**

- [ ] **Given** a custom card, **When** the user presses it, **Then** a dialog titled "Edit driver" opens with the Name (in focus) and the Description filled in with what was saved, a collapsed "Improve with AI" button, "Remove" on the left, Cancel, and Save, which is disabled.

**Changing and saving**

- [ ] **Given** the dialog, **When** the user changes the name or the description, **Then** Save is enabled. **When** the user puts the text back as it was, **Then** Save is disabled again.
- [ ] **Given** a change, **When** the user presses Save, **Then** the card shows the new text in the same place, the dialog closes, and nothing else on the screen changes.
- [ ] **Given** a name and a description with spaces around them, **When** Save, **Then** both are saved without the spaces.
- [ ] **Given** the user clears the description of a fixed or a custom driver, **When** Save, **Then** it is saved, and the card shows "Click to describe this driver." in italics.
- [ ] **Given** a custom driver, **When** the user clears the name (or leaves only spaces) and presses Save, **Then** "Give the driver a name." shows under the field, the field is marked invalid, and nothing is saved. Typing in the field clears the message.
- [ ] **Given** the user renames a custom driver, **Then** it is still the same driver: its position and its identity (and therefore the name of its file and the links that stories have to it) do not change.
- [ ] **Given** two drivers with the same name, **When** one is renamed to the name of the other, **Then** it is saved.
- [ ] **Given** a fixed driver, **Then** its name cannot be changed.

**Closing without saving**

- [ ] **Given** the dialog, **When** the user presses Cancel, the Close (X) button, or Escape, **Then** it closes and the driver is as it was.
- [ ] **Given** nothing changed, **When** the user clicks outside the dialog, **Then** it closes. **Given** there are unsaved changes, **Then** a click outside does nothing.
- [ ] **Given** the user opens the same driver again after closing without saving, **Then** the dialog shows the saved text, not the discarded changes.

**Improve with AI**

- [ ] **Given** the dialog, **When** the user presses "Improve with AI", **Then** the same panel as in story 007 opens: the same title, Hide (X), the four quick actions, the instruction field, and Send. It streams, stops, retries, and shows its tokens in the same way.
- [ ] **Given** the Constraints dialog with the Description changed but not saved, **When** the user presses "Improve this", **Then** the instruction carries the name "Constraints", its meaning, and the Description as it is in the dialog right now.
- [ ] **Given** a custom driver whose name was changed in the dialog but not saved, **Then** the AI is given the name as it is in the field now.
- [ ] **Given** the Constraints dialog, **Then** the context sent to the AI contains the other four fixed drivers, the custom drivers, and the ADRs, and does not contain Constraints itself.
- [ ] **Given** a proposal, **When** the user presses "Use this text", **Then** it replaces the Description only, not the Name, and nothing is saved until the user presses Save. The button then reads "In the editor" and is disabled.
- [ ] **Given** the user hides the panel, **Then** the dialog keeps what was typed. **Given** the dialog closes, **Then** the conversation is lost.

**After saving**

- [ ] **Given** an edited driver with text, **Then** the next story, enabler, or diagram drafted with a link to it gets the new text.
- [ ] **Given** an edited driver, **When** the user opens the Builder, **Then** `drivers/<identity>.md` shows the new name and text.
- [ ] **Given** GitHub is connected, **Then** the next sync of the drivers uploads the changed file as a pull request.

## Business rules and constraints

- A fixed driver has a meaning (part of BWYN) and a description that the team may change; its name is fixed and cannot be edited.
- A custom driver has a name and a description. Its identity does not change when it is renamed (ADR-0017).
- "Changed" means any difference from the saved text. Saving trims spaces, so a change of spaces alone can be saved and change nothing.
- The AI follows the same rules as in story 007. For a fixed driver it is also given the meaning; the driver being edited is never in its stable context, because the current text travels with each instruction (story 007).
- While the dialog is open, syncing is blocked ("Save or close the driver first") and a download from the repository waits.
- Assumption: the five fixed drivers are identified by a well-known constant key; custom drivers are identified by a stable UUID assigned at creation.

## Out of scope

- Adding a driver (story 010) and removing a driver (a separate story).
- Renaming a fixed driver.
- A history of changes to a driver.
- GitHub sync (its own story).

## Tasks

- [ ] Add a `UpdateFixedDriverCommand` and its handler in `bwyn-application` that accepts the driver's well-known key and the new description, trims it, and persists via the outbound port (ADR-0017).
- [ ] Add a `UpdateCustomDriverCommand` and its handler in `bwyn-application` that accepts the driver's stable identity, the new name, and the new description, validates that the name is non-blank after trimming, trims both fields, and persists via the outbound port (ADR-0017).
- [ ] Implement the outbound persistence adapter in `bwyn-infrastructure` for both update operations, keeping the driver's identity and position unchanged on rename (ADR-0017).
- [ ] Add a Flyway migration script for any schema change required by the update operations (ADR-0010).
- [ ] Expose a `PATCH /projects/{projectId}/drivers/{driverIdentity}` endpoint in `bwyn-rest`, document it in the OpenAPI specification with its request schema and error responses, and return a typed error when the name is blank (ADR-0014, ADR-0008).
- [ ] Build the fixed-driver dialog variant in the `ui` layer: title, meaning subtitle, no Name field, Description in focus, collapsed "Improve with AI" button, no Remove button, Cancel, and Save disabled until a change is detected (ADR-0007).
- [ ] Build the custom-driver dialog variant in the `ui` layer: title "Edit driver", Name in focus, Description, collapsed "Improve with AI" button, Remove button, Cancel, and Save disabled until a change is detected (ADR-0007).
- [ ] Wire the dirty-state check in the `interface-adapters` layer: Save enabled on any difference from saved text, disabled when the text is restored; click-outside closes only when nothing is changed (ADR-0007).
- [ ] Add the name-blank validation to the custom-driver ViewModel: show "Give the driver a name." under the field, mark it invalid, clear the message on any keystroke (ADR-0007).
- [ ] Mount the shared `ImproveWithAI` component (story 007) in both dialog variants, passing the driver's current name, its meaning (fixed only), the current description, the context without this driver, and the apply-to-description callback (ADR-0007).
- [ ] Notify the Drivers screen that a dialog is open so that sync is blocked and repository downloads wait.
- [ ] Write Jest unit tests for the dirty-state logic, the name-blank validation, and the apply-proposal callback in the ViewModel (ADR-0005).
- [ ] Write a JUnit unit test for `UpdateFixedDriverCommandHandler` and `UpdateCustomDriverCommandHandler` using test-double outbound ports (ADR-0009, ADR-0017).
- [ ] Verify the full build and test suite passes within the ten-minute budget (Practice: Ten-Minute Build).
