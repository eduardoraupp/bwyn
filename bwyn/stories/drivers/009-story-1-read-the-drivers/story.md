# Story: Read the drivers of my project on one screen

- **Story**: 009-story-1-read-the-drivers
- **Created**: 2026-11-04
- **Status**: Draft

## Story
As a team member, I want one screen with the five fixed drivers and the drivers my team added, each with what was written, so that I can see at a glance what steers our decisions and stories, and which drivers still need to be written.

## Acceptance tests

**The screen**
- [X] **Given** a project, **When** the user chooses "Drivers" in the menu, **Then** the page is titled "Drivers" with the description "The forces that steer decisions. Describe each driver and save it: they feed the skills used by the builder."
- [X] **Given** the page, **Then** the sentence "Drivers are kept in the repository as drivers/ files. Sync uploads what you changed as a pull request and downloads what changed on the base branch." shows, with a Sync button on its right.
- [X] **Given** the page, **Then** the drivers are cards in a grid of two columns on a wide screen and one on a narrow screen.
- [X] **Given** the page, **Then** it can be read with no model connected and no repository connected.

**Fixed drivers**
- [X] **Given** any project, **Then** the first five cards are, in this order: Core Functionality, Architectural Concerns, Constraints, Quality Attributes, and Design Purpose; they cannot be removed and have no "Custom" chip.
- [X] **Given** a fixed card, **Then** it shows an icon, the name in bold, a pencil, and the driver's meaning: "What the system must do for its users, in the terms they would use." (Core Functionality), "The structural worries that shape the design, such as integration, data and deployment." (Architectural Concerns), "What cannot be negotiated: the technology that must be used, the rules to follow and the date to meet." (Constraints), "How well the system must perform: speed, security, reliability and ease of change." (Quality Attributes), "Why this design exists and which problem it is meant to solve." (Design Purpose).
- [X] **Given** the Constraints card, **Then** it also shows three tags: "Technical", "Policy", and "Delivery date".
- [X] **Given** a fixed driver with text, **Then** its card shows the text under a line, with line breaks kept and cut after three lines.
- [X] **Given** a fixed driver with no text, **Then** its card shows "Click to describe this driver." in italics.

**Custom drivers**
- [X] **Given** the team added drivers, **Then** each appears as a card after the fixed ones, in the order they were added, with the chip "Custom", a pencil, the name in bold, and the text (or "Click to describe this driver." in italics); a custom card has no meaning line.
- [X] **Given** the last card, **Then** it is a dashed card with a plus icon and the label "Add a driver".

**Opening**
- [X] **Given** a driver card, **When** the user presses it (or uses Enter or Space on it), **Then** the driver opens for editing; the card's accessible name is "Edit \<name\>". (Verified up to the view model, which records which card was pressed; the edit dialog itself is out of scope and belongs to story 011.)
- [X] **Given** the "Add a driver" card, **When** the user presses it, **Then** the dialog to add a driver opens. (Verified up to the view model, which records that the add card was pressed; the add dialog itself is out of scope and belongs to story 010.)
  _Tested in story 010 (add-driver.acceptance.test.tsx), which owns this behaviour; the duplicate test here was removed._

**Keeping drivers**
- [X] **Given** drivers with text, **When** the page is reloaded, **Then** the same drivers show with the same text.
- [X] **Given** two projects, **Then** each shows its own texts and its own custom drivers.

**What drivers are for**
- [ ] **Given** a driver (written or not), **When** the user opens the Builder, **Then** it is listed as `drivers/<identity>.md`, read-only, with the note "Read-only here. Edit this driver from the Drivers page.", and a story can link to it. (NOT DONE: the production Builder is still a placeholder with no file list, so there is nothing to add driver files to yet.)
- [ ] **Given** a story linked to a driver, **When** its prompt is built, **Then** the driver's full text is sent to the model; a driver that no story links to is not sent. (NOT DONE: the production Builder has no stories or prompt building yet.)

## Business rules and constraints

- The five fixed drivers — Core Functionality, Architectural Concerns, Constraints, Quality Attributes, and Design Purpose — are defined by BWYN and are the same in every project; their text belongs to the project.
- Custom drivers belong to one project; a project never sees the drivers of another project.
- A driver is stored as a Markdown file: its name as the title, its meaning (fixed drivers only), and its text. A driver with no text is represented in the file as "Not written yet. Edit this driver from the Drivers page." and is omitted from what is uploaded to the repository.
- Drivers are not part of the project context that every request carries; a driver reaches the model only when a story links to it (ADR-0017, application layer reaches external systems only through ports).
- The fixed driver list (identity, name, meaning, icon, tags) lives in the entity layer of the frontend (ADR-0007); the per-project driver texts and the list of custom drivers are persisted on the backend (ADR-0008).
- The Sync button is present on the page but non-functional in this story; its behaviour is out of scope.
- Assumption: "wide screen" means a viewport of 768 px or wider; "narrow screen" means narrower than 768 px.

## Out of scope

- Adding, editing, and removing a driver (separate stories).
- Syncing drivers with the repository.
- Linking a driver to a story from the Builder.
- Ordering or grouping drivers, and a search.
- The dialog to add a driver and the dialog to edit a driver.

## Tasks

- [X] Define the fixed-driver list (identity, name, meaning, icon, tags) as an immutable constant in the entity layer (ADR-0007, Architecture)
- [X] Add a `Driver` entity type and a `DriverRepository` port interface in `bwyn-core` for persisting per-project driver texts and custom driver records (ADR-0017, Architecture) (Done as a `Driver` record with identity, name, text, custom and position, and a `DriverRepository` with `save` and `findByProjectId`, in `com.bwyn.driver.domain`.)
- [X] Add a Flyway migration script that creates the `driver` table with columns for project id, driver identity, text, and display order (ADR-0010) (V8__create_driver_table.sql: project id, identity, name, text, custom flag and position; primary key is project and identity. There is no back-fill: no drivers were stored before this story.)
- [X] Implement the `DriverRepository` port as a Spring Data JPA outbound adapter in `bwyn-infrastructure` (ADR-0011, ADR-0017, Architecture)
- [X] Write a `GetDriversQuery` and its handler in `bwyn-application` that returns the fixed drivers merged with per-project texts and the project's custom drivers, in display order (ADR-0017, Architecture) (Deviation: the handler returns only what the project stores. The fixed drivers are merged in by the frontend use case, because the story keeps the fixed list in the frontend entity layer.)
- [X] Add a `GET /api/projects/{projectId}/drivers` endpoint in `bwyn-rest` annotated with OpenAPI annotations, returning the merged driver list (ADR-0014, ADR-0017, Architecture) (Returns only the stored drivers, for the same reason; the fixed ones are listed once their text is written.)
- [X] Write a Testcontainers integration test for the `DriverRepository` adapter confirming read and write against a real PostgreSQL instance (ADR-0012) (DriverPersistenceIT, a Spring Boot test with Testcontainers, covers the adapter, the endpoint, the migration and the OpenAPI entry.)
- [X] Write a use case in the frontend use-case layer that calls the drivers endpoint and maps the response to driver view models (ADR-0007, Architecture) (GetDriversUseCase merges the stored drivers with FIXED_DRIVERS.)
- [X] Write a `DriversViewModel` in the interface-adapter layer that exposes the ordered driver list and the loading state (ADR-0007, Architecture)
- [X] Build the `DriverCard` React component in the UI layer: icon, name in bold, meaning line (fixed only), tags (Constraints only), "Custom" chip, three-line text preview with line breaks, italic placeholder, and pencil icon; accessible name "Edit \<name\>" (ADR-0007, Architecture)
- [X] Build the `AddDriverCard` React component: dashed border, plus icon, label "Add a driver", keyboard-accessible (ADR-0007, Architecture)
- [X] Build the `DriversPage` React component: page title, description, repository sentence with Sync button placeholder, two-column / one-column responsive grid of cards (ADR-0007, Architecture)
- [X] Register the Drivers route in the top-level route configuration and add "Drivers" to the left-bar menu (ADR-0020) (Already in place from the navigation story: the "drivers" page id, its menu entry and the /:pageId route; this story only gives that page its content.)
- [ ] Add a read-only `drivers/<identity>.md` file entry per driver in the Builder file list, with the note "Read-only here. Edit this driver from the Drivers page." (NOT DONE: the production Builder is a placeholder; see the criterion on the Builder above.)
- [ ] Ensure that when a story prompt is built, only drivers linked by that story are included in the payload sent to the model (NOT DONE: the production Builder has no prompt building yet.)
- [X] Write Jest unit tests for the fixed-driver constant, the `DriversViewModel`, and the `DriverCard` component (ADR-0005)
- [X] Extend ArchUnitTS rules to verify that the fixed-driver constant resides in the entity layer and that `DriversViewModel` resides in the interface-adapter layer (ADR-0016)
- [ ] Verify the full build and test suite completes within the ten-minute budget (Practice: Ten-Minute Build) (Not met yet: the full build took about 10.5 minutes in the last measurement.)

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
