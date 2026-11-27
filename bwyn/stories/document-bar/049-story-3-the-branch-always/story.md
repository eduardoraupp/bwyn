# Story: The branch always wins

- **Story**: 049
- **Created**: 2026-11-27
- **Status**: Draft

## Story
As a team member, I want what was pushed to the branch of the story or enabler I have open to replace my copy and be saved straight away, so that when two of us touch the same document we never have a conflict to solve: the last push is the document.

## Acceptance tests

**Noticing a push**
- [ ] **Given** a story or enabler is open and GitHub is connected, **When** 30 seconds pass or the window regains focus, **Then** BWYN fetches the head commit of the document's branch.
- [ ] **Given** the fetched head matches the last commit this machine pulled or pushed, **Then** nothing changes and no notice is shown.
- [ ] **Given** the fetched head is a different commit, **Then** BWYN downloads the document's files from the branch, replaces the open document with them, saves them in the database, and records the new head as the last commit known for this document on this machine.
- [ ] **Given** the document has no open branch, **Then** BWYN watches the base branch for the document's own files in the same way.

**Replacing**
- [ ] **Given** the branch changed `story.md` (or the enabler's markdown file), `meta.json`, a diagram file, or the UI example, **Then** each changed file is replaced in the open document and on screen.
- [ ] **Given** a file that belonged to the document (for example `diagrams/class.mmd`) was deleted on the branch, **Then** it is deleted here too and the Diagrams button no longer shows it.
- [ ] **Given** the story lives in a different theme folder on the branch, **Then** it moves to that theme in the tree here.
- [ ] **Given** the replace succeeded, **Then** a notice below the header reads "Updated from story-003 and saved." and lists the affected files, for example "story.md, diagrams/sequence.mmd".
- [ ] **Given** the user had unsaved edits in the text when the replace happened, **Then** those edits are overwritten and the notice adds "Your unsaved changes were replaced."
- [ ] **Given** a diagram or UI-example dialog was open when the replace happened, **Then** the dialog closes and the notice names it.
- [ ] **Given** the page is reloaded after a replace, **Then** the new version is shown: it was saved in the database before the reload.

**Never a conflict**
- [ ] **Given** the same file was edited locally and changed on the branch, **Then** the branch version is kept and no conflict UI is shown.
- [ ] **Given** a story or enabler is not open, **Then** nothing is fetched for it; **When** it is opened, **Then** its branch is checked immediately.
- [ ] **Given** GitHub cannot be reached during a poll, **Then** nothing changes locally and BWYN retries at the next scheduled poll.

## Business rules and constraints
- The branch always overwrites the local copy. There is no merge, no conflict dialog, and no "Force pull" action for a story or an enabler. (ADR-0017, Architecture: branch-wins rule stated in the request)
- The replaced content is saved in the database immediately; the user does not press Save after a pull.
- The last commit known is stored per machine per document, so a push from this machine is not pulled back as if it were a teammate's change.
- One person per branch is the intended way of working; when two people work on the same document, the one who pushed last wins.
- Polling and on-focus checks apply only to the document currently open; no background fetching occurs for documents that are not open. (Practice: Ten-Minute Build — background work must not slow the build or the test suite)
- The backend endpoint that fetches the head commit and downloads files from GitHub is an outbound port implemented in the infrastructure layer; the application layer calls it through that port. (ADR-0017, Architecture)
- The frontend polling logic and replace flow live in the ViewModel layer; the React component only renders state and the notice. (ADR-0007, Architecture)

## Out of scope
- Showing who pushed or what changed (a diff view).
- Keeping the replaced version anywhere so it can be restored.
- Pulling documents that are not currently open.
- Conflict resolution UI of any kind.
- Support for GitLab (story 017 covers the provider connection; this story covers GitHub only, consistent with story 048).
- Choosing the polling interval from the UI.

## Tasks
- [ ] Add a backend outbound port `BranchHeadPort` with a method that returns the current head commit SHA for a given document's branch (or the base branch when the document has no open branch), implemented in the infrastructure layer as a GitHub API call (ADR-0017)
- [ ] Add a backend application-layer command handler `PullDocumentFromBranchHandler` that calls `BranchHeadPort`, compares the returned SHA with the stored last-known SHA, downloads changed and deleted files when they differ, persists the new content, updates the last-known SHA, and returns the list of affected files
- [ ] Add a Flyway migration that adds a `last_known_branch_sha` column (per document per installation) to the relevant table (ADR-0010)
- [ ] Expose a `POST /projects/{projectId}/documents/{documentId}/pull-if-changed` endpoint in `bwyn-rest` that delegates to the handler and returns the list of replaced files and a flag for whether unsaved changes were present (ADR-0014, ADR-0017)
- [ ] Write ArchUnit tests asserting that `BranchHeadPort` lives in the domain layer and its implementation lives in the infrastructure layer (ADR-0009)
- [ ] Write a Testcontainers integration test for the pull handler covering: head unchanged (no-op), head changed with file updates, head changed with a deleted file, and GitHub unreachable (ADR-0012)
- [ ] In the frontend ViewModel, add a polling effect that calls the pull endpoint every 30 seconds and immediately when the window regains focus, and exposes a `replaceNotice` observable (ADR-0007)
- [ ] In the frontend ViewModel, handle the response: update the document state, close any open diagram or UI-example dialog, and set the notice text including the file list and the unsaved-changes warning when applicable
- [ ] Render the notice below the document header in the React component; dismiss it when the user navigates away (ADR-0007)
- [ ] Remove the conflict dialog, the "waiting" state, and the "Force pull" action from the story and enabler sync flow established in story 048
- [ ] Write Jest unit tests for the ViewModel covering: no-op on matching SHA, replace with file list, replace with deleted file, replace with theme move, unsaved-changes warning, dialog close, and GitHub-unreachable no-op (ADR-0005, Principle: UI/Front end Tests pay only for what they assert)
- [ ] Verify the full build and test suite completes within the ten-minute budget (Practice: Ten-Minute Build, Practice: Continuous Integration)

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