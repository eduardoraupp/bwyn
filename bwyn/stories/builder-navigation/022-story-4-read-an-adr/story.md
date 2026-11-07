# Story: Read an ADR from the tree, read-only

- **Story**: 022-story-4-read-an-adr
- **Created**: 2026-11-07
- **Status**: Draft

## Story
As a team member, I want to open an ADR from the builder tree and read it, so that I can check a decision of my project from the builder, knowing that I cannot change it there.

## Acceptance tests
- [X] **Given** the `adrs` folder is open in the builder tree, **When** the user selects an ADR file, **Then** a read-only file panel opens showing the ADR icon, the path `adrs/<filename>.md`, and a lock note reading "Read-only here. Edit this ADR from the ADRs page."
- [X] **Given** an ADR file panel is open, **Then** it offers the same rendered and raw-Markdown views, the "Show files" toggle, and the Close control as the driver panel (story 021), and contains no control that creates, edits, moves, deletes, or changes the status of the ADR.
- [X] **Given** an ADR file panel is open in rendered view, **Then** the content is the Markdown stored for that ADR rendered faithfully: the title, the Status and Date lines, and the Context, Decision, and Consequences sections — nothing added, nothing removed.
- [X] **Given** an ADR file panel is open in raw-Markdown view, **Then** the content is the exact Markdown text stored for that ADR, unchanged.
- [X] **Given** an ADR whose status reads Accepted (in any case), **When** the user opens it, **Then** the panel opens normally. An ADR that is not accepted is not listed in the tree (story 020), so it cannot be selected here.
- [X] **Given** the ADR cannot be read from the backend, **When** the user selects it, **Then** no file is open, the tree is shown in full, and the workspace says "This file could not be read. Try again."
- [X] **Given** an ADR whose stored text contains only a title line, **When** the user opens it, **Then** the panel shows that title and no placeholder or filler text.
- [X] **Given** a team member edited an ADR on the ADRs page and then opens the builder, **When** they select that ADR from the tree, **Then** the panel shows the saved text.
- [X] **Given** an ADR is open in the file panel, **When** the user selects a driver from the tree, **Then** the driver replaces the ADR in the panel with its own icon and read-only note, and the rail highlights the `drivers` folder icon.
- [X] **Given** a driver is open in the file panel, **When** the user selects an ADR from the tree, **Then** the ADR replaces the driver in the panel with its own icon and read-only note, and the rail highlights the `adrs` folder icon.

## Business rules and constraints
- The ADR is immutable in the builder. Nothing in the builder changes, moves, deletes, or changes the status of an ADR (ADR-0007, Architecture).
- The file is displayed under the name it has in the repository; its number and slug never change in the panel header.
- The rail icon reflects the folder of whichever file is currently open, updating when the user switches between a driver and an ADR (Architecture).
- The file panel is a UI-layer component; it reads ADR data through the use-case and interface-adapter layers and does not access the data store directly (ADR-0007, Architecture).
- The panel of a driver (story 021) is reused as it is for an ADR: the same `FileView`, `FilesRail`, `MarkdownView` and view toggle, given a different icon, path and read-only note. No second panel component exists.
- Only the ADRs the tree lists, the accepted ones (story 020), can be opened; the Markdown shown is what `GET /api/projects/{projectId}/adrs/{number}` answers in `body`, unchanged.
- The backend serves ADR content through the same read port used for drivers; no new port is introduced if the existing one is sufficient (ADR-0017).

## Out of scope
- Writing, editing, or changing the status of an ADR (covered by story 016).
- Linking an ADR to a story or an architectural enabler from the builder.
- Anything related to stories and architectural enablers in the builder tree.
- "Linked to" and "Referenced by" sections in the file panel.
- Copying the path, downloading the file, or printing.
- The prompt box, "New story" button, runway diagram, and any builder interaction that creates or edits documents.

## Tasks
- [X] Reuse the file-panel component of the driver (`FileView`) for an ADR, giving it the ADR icon, the `adrs/<file>` path and the ADR read-only note, with no second panel component (ADR-0007, AE-0002)
- [X] Wire the ADR selection in the builder tree to the existing read-ADR use case (or introduce a minimal one if absent), following the Clean Architecture inward-dependency rule (ADR-0007, AE-0002)
- [X] Display the stored ADR Markdown unchanged in both the rendered and raw-Markdown views, reusing the view-toggle already present for drivers (story 021)
- [X] Update the rail so it reflects the folder icon (`adrs` or `drivers`) of whichever file is currently open, covering the switch in both directions
- [X] Write Jest unit tests for the ViewModel: ADRs of an accepted status in any case, ADR with only a title, an ADR that cannot be read, a slow ADR that a later choice replaces, and switching from a driver to an ADR and back
- [X] Write a Jest rendering test for the file-panel component confirming no edit controls are present when an ADR is displayed
- [ ] Verify the full frontend test suite, including ArchUnitTS fitness functions, passes within the ten-minute build budget (Practice: Ten-Minute Build, ADR-0016)
