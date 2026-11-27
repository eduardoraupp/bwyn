# Story: Sync a story or an enabler to its own branch

- **Story**: 048
- **Created**: 2026-11-27
- **Status**: Draft

## Story
As a team member, I want each story and enabler to go to GitHub on one branch named after it, with one pull request, so that my team knows where to find the work on it and every sync adds to that same pull request.

## Acceptance tests

**Branch naming**
- [ ] **Given** the story `003-checkout-form`, **Then** its branch is `story-003`. **Given** the enabler `0004-hexagonal-architecture`, **Then** its branch is `architectural-enabler-0004`. The number is written as in the folder or file name, with its leading zeros.
- [ ] **Given** the story is moved to another theme or its title changes, **Then** its branch stays `story-003`.
- [ ] **Given** an open story or enabler, **Then** the sub-line of the header shows its branch and pull request when there are any: "Branch story-003 · Pull request #12" (the number is a link to it on GitHub), or "Not on GitHub yet".

**First sync**
- [ ] **Given** a story that has no branch on GitHub, **When** the user presses Sync, **Then** the dialog says "Push 3 files to the new branch story-003", lists the files, and says "A pull request to main opens with it."
- [ ] **When** the user confirms, **Then** the backend creates `story-003` from the head of the base branch, makes one commit with the files, opens a pull request from `story-003` to the base branch titled "BWYN: story 003 Checkout form", and the dialog says "Pushed to story-003" with the link "Pull request #12".
- [ ] **Given** the push worked, **Then** this machine remembers the commit it pushed as the last one it knows of `story-003`.

**Subsequent syncs**
- [ ] **Given** `story-003` exists with an open pull request and its head is the commit this machine last knows, **When** the user changes the story, saves, and presses Sync, **Then** the dialog says "Push 1 file to story-003 (pull request #12)", and confirming adds one commit to `story-003`. No new branch and no new pull request are made.
- [ ] **Given** the story is the same here as on its branch, **Then** the dialog says "Everything is on story-003." and "Sync now" is disabled.
- [ ] **Given** the head of `story-003` is a commit this machine does not know (a teammate pushed), **When** the user presses Sync, **Then** nothing is pushed: the branch is pulled and replaces the story here, and the dialog says "story-003 had newer changes. They replaced the story here and were saved. Nothing was pushed."
- [ ] **Given** a teammate pushes between the moment the dialog was opened and the moment the user confirms, **Then** the commit is refused by GitHub (the head moved), nothing is pushed, and the branch is pulled as above.

**After the pull request is merged or closed**
- [ ] **Given** pull request #12 was merged, **When** the user changes the story and syncs, **Then** `story-003` is started again from the head of the base branch (deleted and created again if it still exists), the commit goes on it, and a new pull request opens.
- [ ] **Given** pull request #12 was closed without merging, **Then** the next sync starts `story-003` again from the base branch the same way, and the sub-line no longer shows #12.
- [ ] **Given** no open branch for the story, **Then** the base branch is the place the story is compared with and pulled from.

**What a sync holds**
- [ ] **Given** a story, **Then** the push holds its own files only: `story.md`, `meta.json`, `ui.html`, and its diagrams. The documents it links to are not in it.
- [ ] **Given** the story uses an ADR that is not on GitHub yet, **Then** that ADR goes up in the same commit.
- [ ] **Given** the story was moved to another theme folder since its last sync, **Then** the same commit adds its files in the new folder and deletes them from the old one.
- [ ] **Given** an enabler, **Then** the push holds its `.md` and `.meta.json` only.

## Business rules and constraints
- The branch name is the identity of the document: `story-<number>` or `architectural-enabler-<number>`. It never contains the user or the date.
- BWYN never writes to the base branch. Everything reaches it through the pull request of the document's branch.
- A push only goes on top of the commit this machine knows. The commit is made with the expected head of the branch, so a push can never overwrite a teammate's commit; when the head has moved, the branch wins.
- One open pull request per branch. A sync looks for the open pull request of the branch and opens one only when there is none.
- The branch name is derived from the document's number, which is stable: moving a story to a different theme or renaming it does not change the branch name (Architecture: frontend use-case layer owns the `branchOf` derivation; the number comes from the document's identity, not its folder path).
- The frontend stores per-document sync state (branch name, last known commit SHA, pull request number) in the same persistence used by the rest of the builder (ADR-0007, Architecture).
- The backend `upload` endpoint is the single point that creates branches, commits, and pull requests; the frontend never calls the GitHub API directly (Architecture: outbound adapter in `bwyn-infrastructure`, ADR-0017).

## Out of scope
- Choosing the branch name or a prefix per team.
- Draft pull requests, reviewers, and labels.
- Squashing the commits of a branch.
- Syncing linked documents (drivers, ADRs, other stories) as part of a story's sync, except for ADRs not yet on GitHub.
- Conflict resolution beyond the "branch wins" rule already described.
- Support for GitLab (story 017 covers the provider connection; this story covers GitHub only).

## Tasks
- [ ] Implement `branchOf(entry)` in the use-case layer: returns `story-<number>` for a story and `architectural-enabler-<number>` for an enabler, reading the number from the document's identifier (ADR-0007, Architecture).
- [ ] Extend the per-document sync state model (entity or value object in `bwyn-core`) to hold: branch name, last known commit SHA, and open pull request number (ADR-0017).
- [ ] Add a backend query endpoint `GET /api/branches/{branch}/head` that returns the current head SHA of a branch, or a 404 when the branch does not exist (ADR-0008, ADR-0014).
- [ ] Extend the backend `upload` command handler in `bwyn-application` to accept the branch name and the expected head SHA; implement the logic: create the branch from the base branch when missing, restart it when its pull request was merged or closed, commit with the expected head (rejecting if the head has moved), and open a pull request only when none is open (ADR-0017).
- [ ] Add a Flyway migration to persist the new sync-state fields (branch, last known SHA, pull request number) for stories and enablers (ADR-0010).
- [ ] Implement the "compare with branch" use case: diff the local files of a story or enabler against the branch head (or the base branch when no open branch exists), producing the list of changed files (ADR-0007).
- [ ] Update the Sync dialog ViewModel and React component: show "Push N files to story-003", the pull request link, "Everything is on story-003." with Sync disabled, and the "branch wins" outcome message; remove the conflict, waiting, and Force pull sections that do not apply to per-document branches (ADR-0007).
- [ ] Add the branch and pull request sub-line to the story and enabler header component: "Branch story-003 · Pull request #12" (link) or "Not on GitHub yet" (ADR-0007).
- [ ] Handle the "teammate pushed between dialog open and confirm" case in the upload command handler: return a distinct error code so the frontend can trigger the branch-pull flow (ADR-0017).
- [ ] Write unit tests for `branchOf` covering stories and enablers with leading zeros, title changes, and theme moves.
- [ ] Write unit tests for the compare use case: no changes, one changed file, moved theme folder (old path deleted, new path added), and ADR not yet on GitHub.
- [ ] Write integration tests (`*IT.java`) for the upload command handler: first sync, second sync on the same pull request, refused push when head moved, restart after merge, and restart after close (ADR-0012).
- [ ] Write Jest tests for the Sync dialog: first-sync message, up-to-date message, second-sync message, and branch-pulled message; and for the header sub-line: with branch and PR, and "Not on GitHub yet" (Principle: UI/Front end Tests pay only for what they assert).
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
