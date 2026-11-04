# Story: Read the drivers of my project on one screen

- **Story**: 009-story-1-read-the-drivers
- **Created**: 2026-11-04
- **Status**: Draft

## Story
As a team member, I want one screen with the five fixed drivers and the drivers my team added, each with what was written, so that I can see at a glance what steers our decisions and stories, and which drivers still need to be written.

## Acceptance tests

**The screen**
- [ ] **Given** a project, **When** the user chooses "Drivers" in the menu, **Then** the page is titled "Drivers" with the description "The forces that steer decisions. Describe each driver and save it: they feed the skills used by the builder."
- [ ] **Given** the page, **Then** the sentence "Drivers are kept in the repository as drivers/ files. Sync uploads what you changed as a pull request and downloads what changed on the base branch." shows, with a Sync button on its right.
- [ ] **Given** the page, **Then** the drivers are cards in a grid of two columns on a wide screen and one on a narrow screen.
- [ ] **Given** the page, **Then** it can be read with no model connected and no repository connected.

**Fixed drivers**
- [ ] **Given** any project, **Then** the first five cards are, in this order: Core Functionality, Architectural Concerns, Constraints, Quality Attributes, and Design Purpose; they cannot be removed and have no "Custom" chip.
- [ ] **Given** a fixed card, **Then** it shows an icon, the name in bold, a pencil, and the driver's meaning: "What the system must do for its users, in the terms they would use." (Core Functionality), "The structural worries that shape the design, such as integration, data and deployment." (Architectural Concerns), "What cannot be negotiated: the technology that must be used, the rules to follow and the date to meet." (Constraints), "How well the system must perform: speed, security, reliability and ease of change." (Quality Attributes), "Why this design exists and which problem it is meant to solve." (Design Purpose).
- [ ] **Given** the Constraints card, **Then** it also shows three tags: "Technical", "Policy", and "Delivery date".
- [ ] **Given** a fixed driver with text, **Then** its card shows the text under a line, with line breaks kept and cut after three lines.
- [ ] **Given** a fixed driver with no text, **Then** its card shows "Click to describe this driver." in italics.

**Custom drivers**
- [ ] **Given** the team added drivers, **Then** each appears as a card after the fixed ones, in the order they were added, with the chip "Custom", a pencil, the name in bold, and the text (or "Click to describe this driver." in italics); a custom card has no meaning line.
- [ ] **Given** the last card, **Then** it is a dashed card with a plus icon and the label "Add a driver".

**Opening**
- [ ] **Given** a driver card, **When** the user presses it (or uses Enter or Space on it), **Then** the driver opens for editing; the card's accessible name is "Edit \<name\>".
- [ ] **Given** the "Add a driver" card, **When** the user presses it, **Then** the dialog to add a driver opens.

**Keeping drivers**
- [ ] **Given** drivers with text, **When** the page is reloaded, **Then** the same drivers show with the same text.
- [ ] **Given** two projects, **Then** each shows its own texts and its own custom drivers.

**What drivers are for**
- [ ] **Given** a driver (written or not), **When** the user opens the Builder, **Then** it is listed as `drivers/<identity>.md`, read-only, with the note "Read-only here. Edit this driver from the Drivers page.", and a story can link to it.
- [ ] **Given** a story linked to a driver, **When** its prompt is built, **Then** the driver's full text is sent to the model; a driver that no story links to is not sent.

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

- [ ] Define the fixed-driver list (identity, name, meaning, icon, tags) as an immutable constant in the entity layer (ADR-0007, Architecture)
- [ ] Add a `Driver` entity type and a `DriverRepository` port interface in `bwyn-core` for persisting per-project driver texts and custom driver records (ADR-0017, Architecture)
- [ ] Add a Flyway migration script that creates the `driver` table with columns for project id, driver identity, text, and display order (ADR-0010)
- [ ] Implement the `DriverRepository` port as a Spring Data JPA outbound adapter in `bwyn-infrastructure` (ADR-0011, ADR-0017, Architecture)
- [ ] Write a `GetDriversQuery` and its handler in `bwyn-application` that returns the fixed drivers merged with per-project texts and the project's custom drivers, in display order (ADR-0017, Architecture)
- [ ] Add a `GET /api/projects/{projectId}/drivers` endpoint in `bwyn-rest` annotated with OpenAPI annotations, returning the merged driver list (ADR-0014, ADR-0017, Architecture)
- [ ] Write a Testcontainers integration test for the `DriverRepository` adapter confirming read and write against a real PostgreSQL instance (ADR-0012)
- [ ] Write a use case in the frontend use-case layer that calls the drivers endpoint and maps the response to driver view models (ADR-0007, Architecture)
- [ ] Write a `DriversViewModel` in the interface-adapter layer that exposes the ordered driver list and the loading state (ADR-0007, Architecture)
- [ ] Build the `DriverCard` React component in the UI layer: icon, name in bold, meaning line (fixed only), tags (Constraints only), "Custom" chip, three-line text preview with line breaks, italic placeholder, and pencil icon; accessible name "Edit \<name\>" (ADR-0007, Architecture)
- [ ] Build the `AddDriverCard` React component: dashed border, plus icon, label "Add a driver", keyboard-accessible (ADR-0007, Architecture)
- [ ] Build the `DriversPage` React component: page title, description, repository sentence with Sync button placeholder, two-column / one-column responsive grid of cards (ADR-0007, Architecture)
- [ ] Register the Drivers route in the top-level route configuration and add "Drivers" to the left-bar menu (ADR-0020)
- [ ] Add a read-only `drivers/<identity>.md` file entry per driver in the Builder file list, with the note "Read-only here. Edit this driver from the Drivers page."
- [ ] Ensure that when a story prompt is built, only drivers linked by that story are included in the payload sent to the model
- [ ] Write Jest unit tests for the fixed-driver constant, the `DriversViewModel`, and the `DriverCard` component (ADR-0005)
- [ ] Extend ArchUnitTS rules to verify that the fixed-driver constant resides in the entity layer and that `DriversViewModel` resides in the interface-adapter layer (ADR-0016)
- [ ] Verify the full build and test suite completes within the ten-minute budget (Practice: Ten-Minute Build)
