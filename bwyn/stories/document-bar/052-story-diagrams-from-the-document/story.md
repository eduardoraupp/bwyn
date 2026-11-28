# Story: Diagrams from the document bar

- **Story**: 052-story-diagrams-from-the-document
- **Created**: 2026-11-28
- **Status**: Draft

## Story
As a team member, I want to generate a sequence or a class diagram of an open story from its bar, look at it on a full screen, and keep it with the story, so that the pair that builds the story sees who talks to whom and which classes it needs, drawn the way our architecture and linked ADRs say.

## Acceptance tests

**The Diagrams button and its menu**

- [ ] **Given** an open story, **Then** the bar shows "Diagrams" with the workflow icon and a down arrow.
- [ ] **Given** an open architectural enabler, driver or ADR, **Then** there is no Diagrams button.
- [ ] **When** the user presses Diagrams, **Then** a menu labelled "Diagrams" opens below it with two items in this order: "Sequence diagram" ("Who talks to whom, step by step. Not a C4 diagram, drawn in the C4 style.") and "Class diagram" ("The classes and interfaces behind the story (C4 code level)."), each with its icon.
- [ ] **Given** the story holds a diagram of that kind, **Then** its item shows the chip "In the story" and the line "Open the diagram that the story holds." instead of the description.
- [ ] **When** the user presses Escape, clicks outside the menu, or presses Diagrams again, **Then** the menu closes.
- [ ] **When** the user picks an item, **Then** the menu closes and the full screen of that kind opens.

**Generating a diagram on the full screen**

- [ ] **Given** the story does not hold a diagram of the picked kind, **When** it is picked, **Then** a full-screen dialog opens above the builder with the icon and title of the kind, the chip "Generated", the path of the story, the C4 colour legend (Person, Software system, Container, Component, External), and "Drawing the diagram...", and a request is sent to Claude.
- [ ] **Given** the request, **Then** it carries the kind, the path of the story, the text of the story as it is in the editor (even unsaved), the full text of the documents linked to it, the ADRs a linked enabler derives from, and the project context with its principles and its architecture; the system prompt asks for Mermaid only, in the C4 conventions, with at most eight elements.
- [ ] **While** Claude answers, **Then** the Mermaid source shows as it streams, open, and the footer shows only "Stop".
- [ ] **When** the user presses Stop, **Then** the request ends and the screen says "Stopped before the diagram was complete."
- [ ] **When** the answer is complete, **Then** the source is cleaned of any code fence, the diagram is drawn on white with Mermaid in the C4 palette, fitted to the width, the source is folded, and the footer shows the input, output, cache read, and cache write tokens.
- [ ] **Given** Claude returned nothing, **Then** the screen says "No diagram was received from the model."
- [ ] **Given** the request failed, **Then** the screen shows its error.
- [ ] **Given** Mermaid cannot draw the source, **Then** the screen says "The diagram could not be drawn: \<error\>", opens the source, and shows "Ask Claude to fix it", which sends the error back to Claude in the same conversation and draws the corrected source.

**What the full screen offers**

- [ ] **Given** a drawn diagram, **Then** "Fit to width" and "Actual size" switch its size; "Fit to width" is the default.
- [ ] **When** the user presses "Copy source", **Then** the Mermaid source is copied and the button says "Copied" for a moment.
- [ ] **When** the user presses "Regenerate", **Then** the same request is sent again from scratch and the chip says "Generated".
- [ ] **When** the user presses the close button or Escape, **Then** the full screen closes, any request in progress is stopped, and the story is unchanged.

**Adding the diagram to the story**

- [ ] **Given** a drawn diagram and the story holds none of that kind, **Then** the main button says "Add to story"; **When** it is pressed, **Then** the diagram is stored with the story and the full screen closes.
- [ ] **Given** the story holds a diagram of that kind and the one on screen is different, **Then** the main button says "Replace in story"; pressing it replaces the stored one.
- [ ] **Given** the one on screen is the one the story holds, **Then** the main button says "In the story" and is disabled.
- [ ] **Given** nothing is drawn yet, or the source does not render, **Then** the main button is disabled.
- [ ] **Given** the story holds a diagram of that kind, **Then** "Remove from story" shows; **When** it is pressed, **Then** the diagram is removed from the story and the full screen closes.

**The diagrams in the story**

- [ ] **Given** the story holds one or more diagrams, **Then** below the header it shows the section "Diagrams" with one button per diagram, sequence first, each with the icon and label of its kind.
- [ ] **Given** the story holds no diagram, **Then** the section is hidden.
- [ ] **When** the user presses the button of a diagram, or picks a kind the story holds in the menu, **Then** the full screen opens with the stored diagram, the chip "From the story", and the footer "Drawn from the story. No request was made."; no request is sent.
- [ ] **Given** the stored diagram is open, **When** the user presses Regenerate or "Ask Claude to fix it", **Then** Claude continues as if it had drawn the stored one and the chip changes to "Generated".

**A diagram is stored with the story at once**

- [ ] **Given** the user added, replaced, or removed a diagram, **Then** the story's diagrams are stored in the project's database immediately, without pressing Save, and no request is made to GitHub.
- [ ] **Given** the user added a diagram, **When** the page is reloaded or the story is closed and opened again, **Then** its button is in the story and it opens with the same source.
- [ ] **Given** the text of the story has unsaved changes, **When** the user adds a diagram, **Then** the diagram is stored and the text keeps its unsaved changes.
- [ ] **Given** the user added, replaced, or removed a diagram, **Then** the Markdown text of the story is unchanged.
- [ ] **Given** the user deletes the story, **Then** its diagrams are deleted with it and the confirmation says so ("together with its diagrams").

**On GitHub**

- [ ] **Given** a story with a diagram, **When** it is synced, **Then** the diagram is written as `diagrams/sequence.mmd` or `diagrams/class.mmd` in the story's folder, with the trimmed source and a final newline, and `story.md` is not changed by it.
- [ ] **Given** a pull brings `diagrams/<kind>.mmd` for a story, **Then** that diagram is stored with the story and its button shows; a kind the pull does not bring is kept as it is locally.

## Business rules and constraints

- A story holds at most one sequence diagram and one class diagram; a diagram is its Mermaid source by kind, and the drawing is produced from the source each time it is shown (ADR-0007, Architecture).
- What is stored is what the user explicitly added or replaced, never a draft being generated or shown on the full screen.
- A stored diagram is never regenerated by opening it; only Regenerate and "Ask Claude to fix it" send a request.
- Diagrams are stored on every change, like tasks, links, and the UI example; only the story's text waits for Save.
- Diagrams are not part of the Markdown; they live in the story's record in the database and in `diagrams/` on GitHub (ADR-0010, ADR-0017).
- The request carries the linked documents, the principles, and the architecture; Claude must not contradict an accepted ADR nor name a technology none of them names (ADR-0007).
- Mermaid is loaded in strict security mode and only the first time a diagram is drawn (Principle: KISS).
- Only stories have diagrams; enablers, drivers, and ADRs do not.
- The diagram kind column in the database is an enum of `sequence` and `class`; no other values are accepted (ADR-0017).
- The backend endpoint that sets or removes a diagram belongs to the inbound adapter layer (`bwyn-rest`); the diagram record belongs to the domain core (`bwyn-core`); persistence belongs to the outbound adapter (`bwyn-infrastructure`) (ADR-0017, Architecture).
- The `DiagramViewModel` resides in the interface-adapter layer and the diagram full-screen component resides in the UI layer; neither imports the other's layer directly (ADR-0007, AE-0002).

## Out of scope

- Diagram kinds other than sequence and class (C4 context, container, component).
- Editing the Mermaid source by hand on the full screen.
- Asking Claude for a change to a diagram in words (only Regenerate and Fix are provided).
- Exporting the diagram as an image.
- Deleting `diagrams/<kind>.mmd` on GitHub when a diagram is removed locally; a file that is only missing locally is left alone by Sync.
- The icons of the diagrams the story holds on the Diagrams button (covered by story 047).

## Tasks

- [ ] Add a `diagrams` column (Mermaid source keyed by kind: `sequence`, `class`) to the story entity in `bwyn-core` and write the Flyway migration that adds the `story_diagrams` table (ADR-0010, ADR-0017).
- [ ] Implement the outbound adapter in `bwyn-infrastructure` that persists and retrieves diagrams for a story (ADR-0011, ADR-0017).
- [ ] Implement the application-layer command handlers for set-diagram and remove-diagram in `bwyn-application`, and the inbound REST adapter endpoints in `bwyn-rest` with OpenAPI annotations (ADR-0008, ADR-0014, ADR-0017).
- [ ] Extend the story deletion handler to cascade-delete diagrams and update the confirmation message to say "together with its diagrams" (ADR-0017).
- [ ] Add `diagrams` to the story's domain model on the frontend (`entities` layer) and extend the `StoryRepository` port to load and save diagrams (ADR-0007, AE-0002).
- [ ] Implement `DiagramViewModel` in the interface-adapter layer: holds the current kind, stored diagrams, generation state (idle / streaming / complete / error / stopped), source, token usage, and conversation history for Fix (ADR-0007, AE-0002).
- [ ] Build the Diagrams button and its two-item menu in the `DocumentActionBar` UI component, showing the "In the story" chip for kinds the story already holds (ADR-0007, AE-0002).
- [ ] Build the diagram full-screen dialog UI component: title, origin chip ("Generated" / "From the story"), C4 legend, streaming source view, Stop, Mermaid renderer (strict mode, lazy-loaded), render-error state with "Ask Claude to fix it", source fold/unfold, token footer, Fit to width / Actual size, Copy source, Regenerate, Close and Escape (ADR-0005, ADR-0007).
- [ ] Implement the diagram request builder: assembles kind, story path, editor text, linked document texts, ADRs through linked enablers, principles, and architecture; sends to Claude with the C4 Mermaid system prompt; streams the answer; strips code fences (ADR-0007).
- [ ] Implement "Add to story", "Replace in story", "In the story" (disabled), and "Remove from story" actions in the full-screen dialog, calling the ViewModel which calls the backend endpoint immediately (ADR-0007).
- [ ] Show the Diagrams section in the story panel (sequence first); opening a stored diagram must not send a request; Regenerate and Fix must resume the conversation (ADR-0007, AE-0002).
- [ ] Extend the sync adapter to write `diagrams/sequence.mmd` and `diagrams/class.mmd` in the story's folder on push, and to read them back on pull, storing the result without touching `story.md` (ADR-0017).
- [ ] Write Jest tests covering: menu rendering, "In the story" chip, generate flow, Stop, Mermaid render error and Fix, Add / Replace / Remove, reload persistence, opening a stored diagram without a request, and the `.mmd` round-trip (Practice: Continuous integration, ADR-0005).

## Reference: spike files

The behaviour described here is the one of these files, under `bwyn-frontend/spikes/home-page/`. Use them as the reference when building
the story.

| File | What it holds |
|---|---|
| `builder/DiagramMenu.tsx` | `useDiagramSession` (open, insert, remove), `DiagramMenu` (the button and its menu), `SavedDiagrams` (the buttons in the story), `DiagramDialog` (the full screen) |
| `builder/useDiagram.ts` | generate, load a stored diagram without a request, regenerate, fix, stop and reset; the streaming and its errors |
| `builder/diagramPrompts.ts` | the two kinds and their texts, the C4 colours, the system prompt, the request, the fix request, `cleanSource` |
| `builder/diagramRenderer.ts` | Mermaid loaded on first use, in strict mode and the C4 palette |
| `builder/linkedDocs.ts` | the text of the linked documents and the ADRs reached through linked enablers |
| `builder/prompt.ts` | `buildContext`, the project context with the principles and the architecture |
| `builder/bedrockClient.ts` | `streamDraft`, the streamed request to Claude and its token usage |
| `builder/types.ts` | `DiagramKind` and `Diagrams` (Mermaid source by kind) on `StoredStory` and `Entry` |
| `builder/FileView.tsx` | where Diagrams sits in the bar, and where the diagrams and the full screen sit in the story |
| `builder/BuilderWorkspace.tsx` | `onDiagramChange`, which stores or removes a diagram of the open story at once |
| `builder/useBuilder.ts` | `updateItem`, and the deletion of a story with its diagrams |
| `builder/repoFiles.ts` | `diagrams/<kind>.mmd` written on sync and read back on pull |
| `useArchitecture.ts`, `usePrinciples.ts` | the architecture and principles sent with the request |
| `icons.tsx` | `WorkflowIcon`, `SequenceDiagramIcon`, `ClassDiagramIcon` and the icons of the full screen |
