# Story: Describe the architecture of my project

- **Story**: 013-story-describe-the-architecture-of
- **Created**: 2026-11-05
- **Status**: Draft

## Story
As a team member, I want one screen where I write the architecture of my system and save it, so that every story I ask the Builder to create or edit is checked against those layers and each rule is placed in the right one.

## Acceptance tests

**The screen**

- [X] **Given** a project, **When** the user chooses "Architecture" in the menu, **Then** the page is titled "Architecture" with the description "The architecture of the system: its style, its layers and the rules between them. Stories are checked against it."
- [X] **Given** the page, **Then** a blue note reads: "This description goes with the prompts of a story, so that the story is checked against the layers and each rule is placed in the right one. It is not sent with enablers, drivers or ADRs. It is kept in the repository as architecture.md when you press Sync." and a Sync button sits on its right. (The Sync button is shown but disabled, as the story's out of scope says.)
- [X] **Given** the page, **Then** one resizable text area of 16 rows shows the text, with the accessible name "The architecture of <project name>" and the placeholder "Describe the architecture: the style (for example a modular monolith), the layers and what each one is responsible for, which layer may depend on which, and the modules. You can leave it empty."
- [X] **Given** the page, **Then** it can be read and written with no model connected and no repository connected.
- [X] **Given** two projects, **Then** each shows its own architecture text. (Verified in the view model test and, in the backend, by `ArchitecturePersistenceIT`.)

**Writing and saving**

- [X] **Given** a project that never had an architecture, **Then** the box is empty, "Empty. That is allowed." shows under it, Save is disabled, and there is no "Discard changes".
- [X] **Given** the box, **When** the user changes the text, **Then** "Unsaved changes." shows in amber, Save is enabled, and "Discard changes" appears.
- [X] **Given** unsaved changes, **When** the user presses Save, **Then** the text is stored as typed, "Saved." shows, the button reads "Saved" for about two and a half seconds and is disabled, and "Discard changes" disappears.
- [X] **Given** unsaved changes, **When** the user presses "Discard changes", **Then** the box reverts to the saved text and nothing else changes.
- [X] **Given** the box has focus, **When** the user presses Enter, **Then** a new line is added and the page is not submitted.
- [X] **Given** a saved text, **When** the user clears the box and saves, **Then** the empty text is stored and "Empty. That is allowed." shows.
- [X] **Given** a saved text, **When** the page is reloaded, **Then** the same text appears with its line breaks and spaces as typed. (Verified by opening the page again, and by the backend integration test, which reads the text back from PostgreSQL.)

**Improve with AI**

- [X] **Given** the page, **Then** an "Improve with AI" button shows below the Save row. **When** pressed, **Then** the shared `ImproveWithAI` panel opens (story 007) with the title "Improve with AI", a "Hide the assistant" button, four quick actions ("Improve this", "Clarify the layers", "Add dependency rules", "Ask me questions"), a free-text box with the placeholder "Ask for a change, for example: focus on security", and a Send button disabled while the box is empty. (Deviation: the shared panel's Hide button is labelled "Hide", as on drivers and principles.)
- [X] **Given** the panel, **When** the user presses a quick action or types a request and presses Enter or Send, **Then** the request appears as a bubble on the right and the reply streams in on the left; Enter does not save the page. (The streaming itself is the shared panel's; this story tests the bubble and that Enter in the panel does not save the page.)
- [X] **Given** a request, **Then** the model receives the text currently in the box (or "(empty)" when there is none) and the instruction; the context contains the project name and the ADRs only — no drivers, stories, enablers, principles, or practices. (Verified in the backend by `StreamTextImprovementCommandHandlerTest`. There are no ADRs in the product yet, so none are listed.)
- [X] **Given** a reply, **Then** a short comment shows and, when the model proposes a new text, a "Proposed text" card appears with a "Use this text" button. **When** the user presses it, **Then** the proposal appears in the box, "Unsaved changes." shows, nothing is saved, and the button reads "In the editor" and is disabled while the box holds that text.
- [X] **Given** "Ask me questions", **Then** the reply contains questions (up to three) and no proposed text. (Verified that the quick action tells the model to ask questions and propose no text; what the model writes cannot be tested here.)
- [X] **Given** a reply cut by the length limit, **Then** the card shows "Incomplete, so it cannot be used." and the button is disabled. (Behaviour of the shared panel, tested in `improve-with-ai.acceptance.test.tsx`; the page only mounts it.)
- [X] **Given** a reply is streaming, **Then** quick actions are disabled and a Stop button replaces Send. **When** the user presses Stop, **Then** what was received stays as an incomplete reply. (Behaviour of the shared panel, tested in `improve-with-ai.acceptance.test.tsx`; the page only mounts it.)
- [X] **Given** a request that fails, **Then** the reason shows in red with a Retry button; pressing Retry resends the last instruction without adding a second user message to the conversation. (Behaviour of the shared panel, tested in `improve-with-ai.acceptance.test.tsx`; the page only mounts it.)
- [X] **Given** the user hides the panel and opens it again, **Then** the conversation is still there. **When** the user leaves the page, **Then** it is gone.
- [X] **Given** a reply, **Then** the token counts of the last request (input, output, cache read, cache write) show under the panel. (Behaviour of the shared panel, tested in `improve-with-ai.acceptance.test.tsx`; the page only mounts it.)

**Where the architecture is used**

- [ ] **Given** a saved architecture, **When** the user asks the Builder to create a story, edit a story, draw a diagram of a story, or make a UI example of a story, **Then** the request carries the section "Architecture of the system (the layers a story must respect)" with the saved text, trimmed of surrounding spaces. (Not verified: there is no Builder in the product yet. The saved text is available from `GET /api/projects/{projectId}/architecture`.)
- [ ] **Given** an empty saved architecture, **Then** that section is omitted from those requests. (Not verified: there is no Builder in the product yet.)
- [ ] **Given** unsaved text on this page, **Then** the Builder sends the saved text, not the draft. (Not verified: there is no Builder in the product yet. The draft lives only in the page and is never sent to the backend before Save.)
- [ ] **Given** a request about an architectural enabler, **Then** the architecture section is not sent; it is not sent by the driver assistant, the principles assistant, or the ADR assistant either. (Not verified for the Builder, which does not exist yet. The driver and principles assistants already do not send it: their requests use their own context kinds.)
- [ ] **Given** a story request that carries an architecture, **Then** the model is instructed to check the story against it: each rule belongs in the right layer, nothing may break a dependency rule between layers, the layer is named when it helps and cited as "(Architecture)", and any part of the request that does not fit the layers is left out and named under Out of scope. (Not verified: there is no Builder in the product yet.)
- [ ] **Given** a diagram request that carries an architecture, **Then** the model is instructed to place each container or class in the layer it belongs to and to draw no relationship that breaks a dependency rule. (Not verified: there is no Builder in the product yet.)

## Business rules and constraints

- The architecture is one free-text document per project; any text is accepted, including empty (Architecture: domain layer owns the document, no schema enforced).
- The AI only proposes; it never saves and never modifies the box by itself. Nothing is saved until the user presses Save.
- The saved text is what is used: the Builder and the sync read the saved text, not the draft in the box.
- The `ImproveWithAI` panel is the shared component introduced in story 007; it is mounted here with the noun "architecture", the four quick actions listed above, the system prompt for architecture, and a callback that puts the proposal in the box (ADR-0007, Principle: DRY and KISS).
- The context sent to the model from this page contains the project name and the ADRs only; no drivers, stories, enablers, principles, or practices are included.
- The assistant preserves the user's intent, language, and facts. It does not invent layers, modules, technologies, names, or rules: it asks, or leaves a placeholder in square brackets. Its comment has at most three sentences or at most three short questions. A proposal is the whole new text, not a diff.
- The assistant is consistent with the ADRs it is given. When the text contradicts an Accepted ADR, the comment says so.
- The architecture is sent only with story-related requests (create, edit, diagram, UI example); it is not sent with enabler, driver, ADR, or principle requests (Architecture: application layer decides what context to include).
- Assumption: the architecture text is stored per project in the same persistence mechanism used by other project documents.

## Out of scope

- GitHub sync of `architecture.md` (the Sync button is shown but disabled; sync behaviour is a follow-up story).
- Checking stories that already exist when the architecture changes.
- A history of versions, version comparison, and a Markdown preview.
- A structured editor (a list of layers and rules).
- The `ImproveWithAI` panel internals, the backend `/improve` endpoint, and the streaming protocol (covered by story 007).
- Connecting GitHub and the sync dialog (covered by a separate story).
- Sending the architecture to the model for enablers, drivers, ADRs, or principles.

## Tasks

- [X] Add the "Architecture" entry to the menu with its icon, title, and description (ADR-0020, Architecture: UI layer). (The entry was already in the menu; this story fills the page it opens.)
- [X] Implement the backend endpoint to persist and retrieve one architecture text per project, with no schema validation (ADR-0008, ADR-0017: domain layer, application layer, inbound adapter, outbound adapter). (`GET` and `PUT /api/projects/{projectId}/architecture`; a project that never saved answers an empty text, an unknown project is a 404.)
- [X] Add a Flyway migration for the architecture text column or table (ADR-0010). (`V9__create_architecture_table.sql`.)
- [X] Build the `ArchitectureViewModel` in the interface-adapters layer: load, save, and track dirty state (ADR-0007). (It also keeps the 2.5 second "Saved" moment.)
- [X] Build the Architecture page in the UI layer: the note, the Sync button (disabled), the resizable text area, the status line ("Unsaved changes.", "Saved.", "Empty. That is allowed."), Save with its two-and-a-half-second "Saved" state, and "Discard changes" (ADR-0007).
- [X] Mount the shared `ImproveWithAI` component on the Architecture page, passing the noun "architecture", the four quick actions, the architecture system prompt, the ADR context builder, and the apply-proposal callback (story 007, ADR-0007). (The context builder is on the backend: the `architecture` context kind carries the project name and the ADRs.)
- [ ] Add the saved architecture text to the Builder's story context (create and edit) as the section "Architecture of the system (the layers a story must respect)", omitting the section when the text is empty (Architecture: application layer). (Deferred: there is no Builder in the product yet.)
- [ ] Add the same section to the diagram and UI-example prompts, with the layer-placement instruction for diagrams (Architecture: application layer). (Deferred: there is no Builder in the product yet.)
- [ ] Add the story-checking instruction to the story prompt: each rule in the right layer, no broken dependency, cite "(Architecture)", leave non-fitting parts under Out of scope. (Deferred: there is no Builder in the product yet.)
- [X] Write Jest unit tests for `ArchitectureViewModel`: load, save, dirty tracking, discard, and empty-text handling (ADR-0005).
- [X] Write a JUnit unit test for the save and retrieve application service using a test-double repository port (ADR-0009, ADR-0017). (`ArchitectureHandlersTest`.)
- [ ] Verify the full build and test suite passes within the ten-minute budget (Practice: Ten-Minute Build). (Not met yet: the Java modules build and pass in about 5 minutes, but the frontend module takes about 6 minutes and its `npm test` fails on two architecture rules (`layer-rules.test.ts`, "Where the drivers live") that need forward-slash paths and fail when node runs on this Windows drive.)
