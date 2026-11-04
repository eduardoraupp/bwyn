# Story: Add a driver

- **Story**: 010-story-2-add-a-driver
- **Created**: 2026-11-04
- **Status**: Draft

## Story
As a team member, I want to add a driver that is specific to my project, and optionally ask AI to make its text clearer, so that our decisions and stories take into account forces that the five fixed drivers do not cover, such as compliance.

## Acceptance tests

**Opening the dialog**

- [ ] **Given** the Drivers screen, **When** the user presses "Add a driver", **Then** a dialog titled "Add a driver" opens with the hint "Custom drivers sit next to the fixed ones and can be removed at any time.", a Name field in focus (placeholder "e.g. Compliance"), a Description field of seven rows (placeholder "Write what the builder should know about this driver."), a collapsed "Improve with AI" button, a Cancel button, and an "Add driver" button that is disabled.
- [ ] **Given** the dialog is open and nothing has been typed, **Then** "Add driver" remains disabled; **When** any character is typed in Name or Description, **Then** "Add driver" becomes enabled.

**Saving a driver**

- [ ] **Given** the name "Compliance" and no description, **When** the user presses "Add driver", **Then** the driver is saved, the dialog closes, and a new card with the chip "Custom" and the text "Click to describe this driver." appears after the last existing custom card and before the "Add a driver" card.
- [ ] **Given** a name and a description that have leading or trailing spaces, **When** "Add driver" is pressed, **Then** both are saved with the surrounding spaces removed.
- [ ] **Given** the Name field is empty or contains only spaces and the Description has text, **When** "Add driver" is pressed, **Then** the message "Give the driver a name." appears below the Name field, the field is marked invalid, and nothing is saved; typing in the Name field clears the message.
- [ ] **Given** a driver named "Compliance" already exists, **When** another driver is added with the name "Compliance", **Then** it is saved as a second driver; names are not required to be unique, even relative to fixed driver names.

**Identity assignment**

- [ ] **Given** the name "Compliance", **Then** the driver's identity is `compliance` and its file path is `drivers/compliance.md`.
- [ ] **Given** the identity `compliance` is already taken, **When** a driver named "Compliance" is added, **Then** its identity is `compliance-2`; if that is also taken, `compliance-3`, and so on.
- [ ] **Given** the name "Constraints", **Then** its identity is `constraints-2` because `constraints` belongs to a fixed driver.
- [ ] **Given** a name of more than six words, **Then** the identity is formed from the first six words, lower-cased and joined by `-`.

**Closing without saving**

- [ ] **Given** the dialog is open, **When** the user presses Cancel, the Close (X) button, or Escape, **Then** the dialog closes and nothing is saved.
- [ ] **Given** nothing has been typed, **When** the user clicks outside the dialog, **Then** it closes; **Given** something has been typed, **When** the user clicks outside, **Then** the dialog stays open.
- [ ] **Given** the user typed text, closed the dialog with Cancel, and then pressed "Add a driver" again, **Then** the dialog opens empty.

**Improve with AI**

- [ ] **Given** the dialog is open, **When** the user presses "Improve with AI", **Then** a panel opens with the title "Improve with AI", a Hide (X) button, four quick actions ("Improve this", "Make it measurable", "Ask me questions", "Shorten"), a free-text field with the placeholder "Ask for a change, for example: focus on security", and a Send button that is disabled while the field is empty.
- [ ] **Given** the user presses a quick action or types an instruction and presses Send or Enter, **Then** the instruction appears as the user's message, "Thinking…" is shown until the first token arrives, the reply streams in, and all four quick actions are disabled until streaming ends; pressing Enter does not submit the dialog.
- [ ] **Given** a completed reply, **Then** it contains a short comment (at most three sentences or three short questions) and, only when new text is proposed, a "Proposed text" card with a "Use this text" button.
- [ ] **Given** the user presses "Use this text", **Then** the Description field is replaced with the proposed text, the Name field is unchanged, the button reads "In the editor" and becomes disabled, and nothing is saved until the user presses "Add driver".
- [ ] **Given** a reply that is still streaming or was cut off, **Then** "Use this text" is disabled; a cut-off reply additionally shows "Incomplete, so it cannot be used."
- [ ] **Given** a reply is streaming, **Then** a Stop button replaces Send; pressing Stop halts streaming, keeps the text received so far, disables "Use this text", and re-enables Send.
- [ ] **Given** the backend cannot be reached or returns an error, **Then** the error message is shown in the panel with a Retry button; pressing Retry resends the last instruction without adding a second user message to the conversation.
- [ ] **Given** no model is connected for the project, **Then** the error message reads "No model connected. Connect the model in Integrations."
- [ ] **Given** a completed reply, **Then** a line below it shows the token counts: input, output, cache read, and cache write.
- [ ] **Given** the user hides the panel, **Then** the Name and Description fields retain whatever was typed.
- [ ] **Given** the Name field is empty when the user sends an instruction, **Then** the AI receives the driver name as "(not named yet)" and still replies; **Given** the Name field has text, **Then** its current value travels with every instruction.
- [ ] **Given** a Description with no numbers and the user presses "Make it measurable", **Then** the proposal marks every missing number as a placeholder in square brackets (e.g. `[target response time]`) and invents no concrete values.

## Business rules and constraints

- A driver has a name (a few words) and an optional description; the description is not required to save.
- A driver's identity is derived from its name: the first six words, lower-cased and joined by `-`. If that identity is already taken by a fixed or custom driver, `-2`, `-3`, and so on are appended. The identity is also the file name in the repository (`drivers/<identity>.md`).
- The AI only suggests; it never saves and never modifies the Name field (Architecture: UI layer owns field state).
- The stable (cached) context sent to the model contains all other drivers with their text and the project's ADRs. The name and description as they currently appear in the dialog travel with every instruction. The AI is given the meaning of a fixed driver only when one is present in the context.
- A reply contains a short comment (at most three sentences or three short questions) and, only when new text is proposed, a proposed-text block. The AI keeps the user's intent, language, and facts; it invents nothing (missing numbers become placeholders in square brackets), prefers short, specific, testable statements, uses bullets for separate points, and writes no headings or code fences. It never contradicts an ADR whose status is Accepted; if the input text does, the comment says so.
- The conversation belongs to the open dialog instance and is discarded when the dialog closes.
- The "Improve with AI" button and panel are the shared `ImproveWithAI` component delivered by story 007. What is specific to drivers (the noun "driver", the four quick actions, the system instructions, and the apply-proposal callback) is passed to the component as props; no driver-specific logic is written inside the shared component (Principle: Declarative over imperative).
- While the dialog is open, syncing is blocked ("Save or close the driver first") and any download from the repository waits.
- Assumption: the identity uniqueness check covers both fixed driver identities and the identities of all custom drivers already saved for the project.

## Out of scope

- Editing and removing a driver (separate stories).
- Syncing drivers with the repository.
- Choosing an icon for a custom driver, ordering drivers, and linking a driver to a story from the Builder.
- How the model call is made and streamed (covered by story 007).
- GitHub sync of the new driver file.

## Tasks

- [ ] Add an `AddCustomDriverCommandHandler` in `bwyn-application` that derives the identity from the name (first six words, lower-case, `-2`/`-3` suffix when taken), persists the driver, and returns the assigned identity (ADR-0017).
- [ ] Add a Flyway migration in `bwyn-runtime` for the `custom_driver` table (`id`, `project_id`, `identity`, `name`, `description`) (ADR-0010).
- [ ] Implement the outbound JPA repository adapter in `bwyn-infrastructure` for custom drivers, including the uniqueness query used by the identity-derivation logic (ADR-0011, ADR-0017).
- [ ] Expose a `POST /projects/{projectId}/drivers` endpoint in `bwyn-rest`, annotated with OpenAPI metadata, that invokes `AddCustomDriverCommandHandler` and returns the created driver's identity (ADR-0008, ADR-0014).
- [ ] Add the `AddCustomDriverCommand` and `DriverIdentity` value object to `bwyn-core`, with the identity-derivation rule expressed as a pure domain function (ADR-0017).
- [ ] Build the "Add a driver" dialog in the `ui` layer: Name field (auto-focused), seven-row Description field, dirty-state guard on "Add driver", name-empty validation message, closing rules (Cancel / X / Escape clears state; outside-click only when pristine) (ADR-0007).
- [ ] Place the "Add a driver" card at the end of the custom-driver list on the Drivers screen and wire it to open the dialog (ADR-0007).
- [ ] Write the driver-assistant system prompt, define the four quick actions ("Improve this", "Make it measurable", "Ask me questions", "Shorten"), and implement the context builder (other drivers + ADRs) and the per-turn payload (name, current description, instruction) in the `use-cases` layer (ADR-0007).
- [ ] Mount the shared `ImproveWithAI` component (story 007) in the dialog, passing the noun "driver", the four quick actions, the system instructions, the context builder, and the apply-to-description callback; verify the Name field is never touched by the component (ADR-0007).
- [ ] Expose the page-level "dialog open" flag so that the sync mechanism blocks and pending downloads wait while the dialog is open.
- [ ] Write Jest unit tests for the identity-derivation logic (six-word rule, suffix increment, fixed-driver collision) (ADR-0005).
- [ ] Write a JUnit unit test for `AddCustomDriverCommandHandler` using a test-double repository port (ADR-0009, ADR-0017).
- [ ] Verify the full build and test suite passes within the ten-minute budget (Practice: Ten-Minute Build).

## Files of the spike

In `bwyn-frontend/spikes/home-page/`:

| File | What it holds |
|---|---|
| `Drivers.tsx` | The screen: `Drivers` (what is open: a fixed driver, a custom one, or a new one), `DriverCard`, and `DriverModal`, the one dialog for add, edit and remove (`nameEditable`, `onRemove`, `confirmRemove`, the dirty check, the closing rules). It also sets up the sync of all drivers as one (`drivers:project`) and blocks it while a dialog is open. |
| `DriverAssistant.tsx` | The "Improve with AI" button and its panel: quick actions, streaming reply, "Use this text", Stop, Retry. Props let other screens give it their own `system`, `noun` and `actions` (principles do). |
| `useDriverAssistant.ts` | The conversation with the model for one dialog: turns, streaming, status, error, usage, send, retry, stop. |
| `driverPrompts.ts` | `DRIVER_SYSTEM_PROMPT`, `buildDriverContext` (the other drivers and the ADRs), `buildDriverTurn` (name, meaning, current text, instruction), `QUICK_ACTIONS` and `parseReply` (the `---DRIVER---` marker that separates the comment from the proposal). |
| `useDrivers.ts` | The drivers of a project, kept in the browser: `texts` of the fixed drivers and the `custom` list, with `saveFixed`, `addCustom` (identity from `slugify(name, 6)`, "-2" when taken), `saveCustom`, `removeCustom` and `importDrivers`. |
| `data.ts` | `FIXED_DRIVERS` (the five, with meaning, icon and tags) and the "Drivers" menu entry (title and description). |
| `Sections.tsx` | Mounts `Drivers` for the "drivers" section. |
| `icons.tsx` | The icons of the cards and buttons. |
| `builder/generate.ts` | `driverMarkdown` and `DRIVER_PLACEHOLDER` (how a driver is written as a file) and `slugify`. |
| `builder/catalog.ts` | Turns every driver, written or not, into a read-only `driver:<identity>` entry (`drivers/<identity>.md`) that stories can link to. |
| `builder/FileView.tsx` | The note "Read-only here. Edit this driver from the Drivers page." |
| `builder/prompt.ts`, `builder/linkedDocs.ts` | A driver goes to the model only when linked, in full. |
| `builder/repoFiles.ts`, `builder/git/*` | The files `drivers/<identity>.md` (empty drivers left out), reading them back, and the sync scope of all drivers. |
| `builder/bedrockClient.ts`, `server/bedrock.js` | Today's call to the model, through the dev server. `story-improve-with-ai.md` replaces it with the backend REST API. |

## Spike details worth keeping in mind

- Escape closes the dialog even with unsaved text, while a click outside does not.
- The add dialog's subtitle ("Custom drivers sit next to the fixed ones...") is also handed to the assistant as the driver's "Meaning".
  Story 2 asks for the meaning only for fixed drivers, which have one.
- A fixed driver's text can be cleared and saved; the card then shows the italic prompt.
- A name made only of signs (such as "???") passes the name check and gets the identity "untitled". Nothing in these stories decides
  about it.
