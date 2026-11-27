# Story: One action bar for stories and enablers

- **Story**: 047-story-1-one-action-bar
- **Created**: 2026-11-27
- **Status**: Draft

## Story
As a team member, I want the same bar of actions on every story and enabler I open, and to see at a glance which diagrams a story already has, so that I always find an action in the same place and do not generate a diagram that already exists.

## Acceptance tests

**Bar composition**
- [X] **Given** an open story, **Then** the bar shows, from left to right: "Add task", "Diagrams", "UI", "Sync", "Save", "Delete", then the close button.
- [X] **Given** an open architectural enabler, **Then** the bar shows "Sync", "Save", "Delete" and the close button, in that order, and no Add task, Diagrams or UI button.
- [X] **Given** a story and an enabler, **Then** their Sync, Save and Delete buttons have the same label, icon, size, colours, tooltip and keyboard focus behaviour.
- [X] **Given** a driver or an ADR open in the builder, **Then** none of the six action buttons appears; only the close button is shown.
- [X] **Given** a narrow window, **Then** the buttons keep their fixed order and wrap to a second line instead of being cut off.

**Diagram indicators on the Diagrams button**
- [X] **Given** a story with no diagram, **Then** the Diagrams button shows its icon, the label "Diagrams", and the expand arrow, and nothing else.
- [X] **Given** a story that holds a sequence diagram, **Then** the Diagrams button also shows the sequence-diagram icon, and its accessible name is "Diagrams: the story has a sequence diagram".
- [X] **Given** a story that holds a class diagram, **Then** the Diagrams button shows the class-diagram icon, and its accessible name is "Diagrams: the story has a class diagram".
- [X] **Given** a story that holds both a sequence diagram and a class diagram, **Then** the Diagrams button shows both icons with the sequence icon first, and its accessible name is "Diagrams: the story has a sequence diagram and a class diagram".
- [X] **Given** the Diagrams button shows at least one diagram icon, **When** the pointer rests on it, **Then** the tooltip names the diagrams the story holds, for example "Sequence and class diagrams in the story".
- [X] **Given** the Diagrams menu is open, **Then** each kind the story already holds shows the chip "In the story" and the line "Open the diagram that the story holds.", and a kind it does not hold shows its description as before.
- [X] **Given** the user inserts a diagram, **Then** its icon appears on the Diagrams button immediately. **Given** the user removes it, **Then** the icon disappears immediately.
- [X] **Given** a story that holds a UI example, **Then** the UI button keeps its green dot.

**Save stores locally only**
- [ ] **Given** an open story or enabler, **When** the user presses Save or Ctrl+S in the text, **Then** the text is stored in the project's database, the sub-line shows "Saved." briefly, and no request is made to GitHub.
- [ ] **Given** the user changed a task, a link, a diagram, the UI example, or the Cross-enabler flag, **Then** that change is stored in the database and no request is made to GitHub.
- [X] **Given** the document was saved but not yet synced, **Then** the Sync button shows a dot and the tooltip "Saved here, not on GitHub yet. Sync to push it."
- [X] **Given** there are unsaved changes in the text, **Then** the Sync button is disabled with the tooltip "Save the story first" or "Save the enabler first".

## Business rules and constraints
- The bar is one React component used by both stories and enablers; which buttons appear is determined by the document kind passed as a prop, not by duplicated markup (ADR-0007, Architecture: UI layer).
- The button order is fixed: Add task → Diagrams → UI → Sync → Save → Delete. An enabler shows only Sync, Save, and Delete (ADR-0007).
- The diagram icons shown on the Diagrams button reflect only diagrams already stored with the story, not a draft being generated and not yet inserted.
- Save never communicates with GitHub; only Sync pushes changes and only Delete opens a pull request.
- The component lives in the `ui` layer (ADR-0007, AE-0002); it receives document kind, diagram state, and callbacks as props and contains no business logic.
- Tests replace all styling, animation, and network concerns; no test introduces an artificial delay (Principle: UI/Front end Tests pay only for what they assert).

## Out of scope
- Diagrams, UI examples, and tasks for an architectural enabler.
- Diagram kinds other than sequence and class.
- A task count badge on the Add task button.
- Syncing a document with GitHub (the Sync action itself is a separate story).

## Tasks
- [X] Create the `DocumentActionBar` React component in the `ui` layer, accepting `kind`, `diagramsInStory`, `hasSavedUnsynced`, `hasUnsavedText`, and action callbacks as props (ADR-0007, AE-0002).
- [X] Render the six buttons for a story and the three buttons for an enabler in the fixed order, and render only the close button for drivers and ADRs.
- [X] Add the sequence-diagram and class-diagram icons to the Diagrams button when the corresponding diagrams are present in `diagramsInStory`, with the correct accessible name and tooltip.
- [X] Add the "saved but not synced" dot and tooltip to the Sync button, and disable Sync with its tooltip when `hasUnsavedText` is true.
- [ ] Replace the existing header buttons in `FileView` with `DocumentActionBar` for both stories and enablers.
- [X] Write Jest tests covering: button set per document kind, fixed order, diagram icons for none / sequence only / class only / both, accessible names, Sync dot and disabled state, and immediate icon update on diagram insert and remove (Principle: UI/Front end Tests pay only for what they assert).
- [ ] Verify the full build and test suite completes within the ten-minute budget (Practice: Ten-Minute Build).

## Files of the spike

All under `bwyn-frontend/spikes/home-page/`.

| File | What it holds today | What changes | Story |
|---|---|---|---|
| `builder/FileView.tsx` | The header of the open story or enabler with its buttons (Add task, UI, Diagrams, Sync, Save, Delete) written inline, and the delete confirmation. | The buttons move to `DocumentActionBar`; the order becomes Add task, Diagrams, UI; the sub-line gets the branch and pull request. | 1, 2 |
| `builder/DocumentActionBar.tsx` (new) | | The bar, shared by the story and the enabler. | 1 |
| `builder/DiagramMenu.tsx` | `DiagramMenu` (the button and menu with "In the story"), `SavedDiagrams`, `DiagramDialog`, `useDiagramSession` with `savedSource`. | The button shows the icon of each kind the story holds. | 1 |
| `builder/Tasks.tsx`, `builder/UiMockup.tsx` | `AddTaskButton`, `UiButton` (with its green dot). | Used by the bar as they are. | 1 |
| `builder/git/SyncButton.tsx` | The Sync button, the dialog (upload, delete, download, awaiting, conflicts, Force pull) and the notice bar. | For stories and enablers: push to the branch, the pull request, no conflict, waiting or Force pull. | 2, 3 |
| `builder/git/useGitSync.ts` | `check`, `sync`, `forcePull`, `autoDownload` and the poll of the base branch head every 30 seconds (`POLL_MS`). | Poll the document's branch; a new head replaces and saves, even with unsaved changes. | 2, 3 |
| `builder/git/syncEngine.ts` | `classify` (with `conflict` and `awaiting`), `computePlan` over the document and the documents connected with it, `uploadPlan`, the records of the sync. | Own files only; the branch wins; remember branch, last known head and pull request per document. | 2, 3 |
| `builder/git/deleteFromRepo.ts` | Deletes on a new `bwyn/...` branch with its own pull request. | Uses the document's branch; closes the pull request and deletes the branch when it was never merged. | 4 |
| `builder/git/gitClient.ts` | `fetchHead`, `fetchTree`, `fetchBlob`, `fetchPrs`, `uploadFiles`. | Branch and expected head in `uploadFiles`; head of a branch that may not exist; close a pull request; delete a branch. | 2, 4 |
| `server/github.js` | `upload` always makes a new branch `bwyn/<login>-<date and time>` from the base branch and a new pull request; `head` fails on a missing branch. | `upload` on a given branch, created when missing or started again after a merge, with the expected head, reusing its open pull request; `head` answers "none"; close and delete. | 2, 4 |
| `builder/BuilderWorkspace.tsx` | `deleteOpen` (GitHub first, then `removeItem`), the sync focus of the open document, `FileView` remounted on `sync.revision`. | Passes the branch state to the bar. | 2, 3, 4 |
