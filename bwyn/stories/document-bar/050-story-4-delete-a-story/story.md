# Story: Delete a story or an enabler, here and on GitHub

- **Story**: 050-story-4-delete-a-story
- **Created**: 2026-11-27
- **Status**: Draft

## Story
As a team member, I want Delete to remove a story or enabler from the project and to open the pull request that removes it from GitHub, so that a deleted document does not come back with the next pull.

## Acceptance tests

**Confirming**
- [ ] **Given** an open story or enabler, **When** the user presses Delete, **Then** a confirmation dialog appears showing what will be removed locally, what will happen on GitHub, and the names of other documents that link to it.
- [ ] **Given** the confirmation dialog is open, **When** the user presses Cancel, **Then** nothing changes in the database, the tree, or GitHub.

**Deleting — document is on the base branch**
- [ ] **Given** GitHub is connected and story 003 "Checkout form" has been merged to the base branch and has no open pull request, **When** the user confirms deletion, **Then** a commit is pushed to branch `story-003` (created from the base branch) that removes the story's folder, a pull request titled "BWYN: delete story 003 Checkout form" is opened, and the chat reads "Deleted the story "Checkout form" and its diagrams. Pull request #13 removes its 4 files from acme/platform: \<link\>".
- [ ] **Given** GitHub is connected and an enabler has been merged to the base branch, **When** the user confirms deletion, **Then** the same flow runs against the enabler's two files on branch `architectural-enabler-0004`.

**Deleting — document is only on its branch**
- [ ] **Given** GitHub is connected and story 003 has an open pull request but has never been merged to the base branch, **When** the user confirms deletion, **Then** the pull request is closed, the branch `story-003` is deleted, and the chat reads "Its pull request #12 was closed and the branch story-003 deleted."

**Deleting — document was never synced**
- [ ] **Given** GitHub is connected but the document has never been pushed to any branch, **When** the user confirms deletion, **Then** no pull request is opened and no branch is created, and the chat reads "It was not on GitHub."

**Deleting — GitHub not connected**
- [ ] **Given** GitHub is not connected to the project, **When** the user confirms deletion, **Then** the document is removed from the database and the tree with no GitHub interaction.

**Local cleanup on success**
- [ ] **Given** the GitHub request (or the no-GitHub path) succeeded, **Then** the document is removed from the database, disappears from the tree, the file panel closes, and every link from other documents to the deleted document is removed.

**Failure handling**
- [ ] **Given** the GitHub request failed, **Then** nothing is deleted from the database or the tree, and the confirmation dialog shows "It could not be deleted from GitHub, so nothing was deleted: \<reason\>".

## Business rules and constraints
- GitHub is called before the database is changed; a GitHub failure leaves the document and all links intact (Architecture: application layer orchestrates the use case through ports; infrastructure failure must not leave partial state).
- Nothing is deleted from the base branch directly; the document is removed when the pull request that this story opens is merged (ADR-0017).
- The deletion uses the document's own branch (`story-NNN` or `architectural-enabler-NNNN`), keeping one document to one branch, consistent with story 048.
- Links from other documents to the deleted document are removed as part of the same local transaction that removes the document itself (ADR-0010, ADR-0011).
- The confirmation dialog must name every document that links to the one being deleted, so the user can assess the impact before confirming (Principle: Declarative over imperative).
- The backend endpoint follows the hexagonal model: the delete command handler in the application layer calls a GitHub port and a repository port; neither the controller nor the JPA adapter contains orchestration logic (ADR-0017, Architecture).
- The React component for the confirmation dialog lives in the `ui` layer; the delete command and its result live in the `use-cases` layer; the ViewModel mediating between them lives in the `interface-adapters` layer (ADR-0007, AE-0002).

## Out of scope
- Undo, a recycle bin, or restoring a deleted document from GitHub.
- Deleting a theme folder.
- Deleting a driver or an ADR from the builder.
- Synchronising linked documents as part of the deletion pull request.
- Support for GitLab (consistent with story 048).

## Tasks
- [ ] Add a `DELETE /projects/{projectId}/stories/{storyId}` and a `DELETE /projects/{projectId}/enablers/{enablerId}` command handler in the application layer (`bwyn-application`) that orchestrates: resolve GitHub state → act on GitHub → delete from database (AE-0005, ADR-0017)
- [ ] Implement a `GitHubDeletionPort` outbound port in `bwyn-core` with operations: push delete commit and open pull request, close pull request, delete branch
- [ ] Implement the `GitHubDeletionPort` adapter in `bwyn-infrastructure` using the existing GitHub API client
- [ ] Add a `DELETE /projects/{projectId}/stories/{storyId}` and `DELETE /projects/{projectId}/enablers/{enablerId}` REST endpoint in `bwyn-rest` annotated with OpenAPI `@Operation` and `@ApiResponse` (ADR-0014)
- [ ] Add a Flyway migration to add a `links` cascade-delete or cleanup step so that links pointing to the deleted document are removed in the same transaction (ADR-0010)
- [ ] Write unit tests for the delete command handler covering: document on base branch, document only on branch, document not on GitHub, GitHub failure (ADR-0009)
- [ ] Write a `DeleteDocumentIT` integration test extending `PostgresIntegrationTest` that verifies the database row and its incoming links are removed on success and untouched on GitHub failure (ADR-0012)
- [ ] Add a `deleteDocument` use case in the `use-cases` layer of the frontend that calls the backend delete endpoint (AE-0002, ADR-0007)
- [ ] Add a `DeleteDocumentViewModel` in the `interface-adapters` layer that resolves the list of linking documents and exposes `confirm()` and `cancel()` commands (AE-0002, ADR-0007)
- [ ] Build the confirmation dialog React component in the `ui` layer, listing linked documents, the GitHub outcome, and the failure message when present (AE-0002, ADR-0007)
- [ ] Wire the Delete action into the action bar established by story 047, guarded by the confirmation dialog
- [ ] Remove the deleted document from the tree and close the file panel on success, and remove its links from the in-memory state of other open documents
- [ ] Write Jest tests for `DeleteDocumentViewModel` using a test double for the use case, covering all four GitHub-state branches and the failure path (ADR-0005, Principle: UI/Front end Tests pay only for what they assert)
- [ ] Verify the full build and test suite passes within the ten-minute budget (Practice: Ten-Minute Build, Practice: Continuous Integration)

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
