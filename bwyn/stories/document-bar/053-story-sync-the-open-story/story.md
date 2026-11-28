# Story: Sync the open story with GitHub

- **Story**: 053-story-sync-the-open-story
- **Created**: 2026-11-28
- **Status**: Draft

## Story
As a team member, I want to sync the story I am working on with GitHub from its bar, and see on one full screen what goes up, what comes down and what blocks it, so that my team reviews my changes in a pull request and I always work on the latest version of the story and the documents connected with it.

## Acceptance tests

**The Sync button**

- [ ] **Given** an open story and the project is connected to GitHub, **Then** the bar shows "Sync" with the refresh icon and the hint "Sync the project with GitHub".
- [ ] **Given** the project is not connected to GitHub, **Then** Sync is disabled with the hint "Connect GitHub in Integrations first".
- [ ] **Given** the text of the story has unsaved changes, **Then** Sync is disabled with the hint "Save the story first".
- [ ] **While** a comparison or a sync runs, **Then** the refresh icon spins.
- [ ] **Given** the last comparison found a conflict, **Then** the button shows an amber dot labelled "Conflict with the base branch".
- [ ] **Given** the last comparison found files to download and no conflict, **Then** the button shows a blue dot labelled "Changes on the base branch".
- [ ] **When** the user presses Sync, **Then** the full screen opens and a comparison starts; nothing is changed here or on GitHub until Sync now or Force pull is pressed.

**The full screen**

- [ ] **Given** the full screen is open, **Then** it covers the whole window above the builder with a header containing the branch icon, the title "Sync with GitHub", the line `<owner>/<repo> · <base branch> · <base folder or (root)>`, and a close button; a row of document chips; a scrolling body; and a footer with a status on the left and actions on the right.
- [ ] **While** comparing, **Then** the body says "Comparing with `<base branch>`…"; **while** syncing, "Syncing…".
- [ ] **When** the user presses the close button, "Close", or Escape, **Then** the screen closes; **while** a comparison or a sync runs those three are disabled.
- [ ] **Given** a request failed, **Then** the body shows its error message.
- [ ] **Given** the repository is too large for GitHub to list in full, **Then** the body warns "The repository is too large for GitHub to list in full. Some files may be missing from this comparison."
- [ ] **Given** a comparison was made, **Then** the footer says "Compared with `<base branch>` at `<short head SHA>`", followed by "· Last synced `<date and time>`" when the project was synced before.

**What takes part in the sync**

- [ ] **Given** the comparison, **Then** the row shows the documents in this sync as chips, up to twelve and then "+N more", and the line "This story and the N documents connected with it: what it links to and what links to it. Other documents are left out. ADRs are always checked: new and changed ones come down, and one that is used goes up with this story."
- [ ] **Given** the story, **Then** its own files take part: `story.md`, `meta.json`, `diagrams/<kind>.mmd`, and `ui.html` in its folder whether it is in a theme or not.
- [ ] **Given** the story links to or derives from a document, or a document links to it, **Then** that document's files take part, including a document that links to the story only in its `meta.json` on GitHub.
- [ ] **Given** any ADR, **Then** it is always compared: a new or changed one comes down; an ADR changed here goes up only when a document in this sync links to it; an ADR removed here is not brought back.
- [ ] **Given** a document that is not connected to the story, **Then** it is left out even if it changed here or on GitHub.

**The decision for each file**

- [ ] **Given** the file is the same here and on the base branch, **Then** nothing is done.
- [ ] **Given** the file changed on the base branch and not here, **Then** it is downloaded.
- [ ] **Given** the file changed here and not on the base branch, **Then** it is uploaded.
- [ ] **Given** the file changed here and was sent in a pull request that is still open, and the base branch did not change it, **Then** it is waiting for that pull request.
- [ ] **Given** the file changed here and was sent in an open pull request, and the base branch did change it meanwhile, **Then** it is a conflict.
- [ ] **Given** the pull request it was sent in was merged, **Then** the version of the base branch is taken.
- [ ] **Given** the pull request it was sent in was closed without merging, **Then** it is forgotten and the file is uploaded again at the next sync.
- [ ] **Given** the file changed here and on the base branch, **Then** it is a conflict.
- [ ] **Given** a file exists only on GitHub because it was deleted here, **Then** it is left alone.
- [ ] **Given** the story was moved to another theme folder here, **Then** its files in the old folder are deleted in the same pull request that adds them in the new folder, unless they changed on GitHub since the last sync, which is a conflict.

**What the body shows**

- [ ] **Given** files to upload, **Then** a section "Upload N files as a pull request" with the hint "BWYN opens a new branch from `<base branch>` and a pull request. Review and merge it on GitHub." and their paths relative to the base folder.
- [ ] **Given** files to delete, **Then** a section "Delete N files from the old folder" with a hint that the story was moved and that the same pull request adds it in the new location and deletes it in the old one.
- [ ] **Given** files to download, **Then** a section "Download N files from `<base branch>`" with their paths.
- [ ] **Given** files waiting, **Then** a section "N files are waiting for a pull request" with the hint "They come back in sync once the pull request is merged." and a link "Pull request #N" for each.
- [ ] **Given** nothing to do, **Then** the body says "Everything is in sync with `<base branch>`."
- [ ] **Given** a comparison, **Then** a "Details for each file" section is folded at the end, listing each file with its decision and the short SHAs here (L), on the base branch now (R), and at the last sync (B).

**A conflict stops everything**

- [ ] **Given** at least one conflict, **Then** the body shows "Conflict with `<base branch>`. Nothing was changed.", the line "These files changed here and on `<base branch>`. Solve it on GitHub, by merging or updating the pull request, and sync again. Until then BWYN does nothing.", each conflicting file with an "Open" link to it on the base branch, and the pull requests waiting; no upload, download, or delete section is shown.
- [ ] **Given** a conflict, **Then** Sync now is disabled and nothing is downloaded or uploaded, not even files without a conflict.

**Sync now**

- [ ] **Given** no conflict and something to upload, delete, or download, **Then** "Sync now" is enabled; otherwise it is disabled.
- [ ] **When** the user presses Sync now, **Then** the comparison is made again first; **given** it now has a conflict, **Then** it is shown and nothing is done.
- [ ] **Given** no conflict after the re-comparison, **Then** files to download are downloaded and stored at once, replacing what is here, and the open story is shown from what was stored.
- [ ] **Given** files to upload or delete, **Then** the backend makes a new branch from the base branch with one commit and opens a pull request titled "BWYN: update N files of `<project>`" (with "and delete N files" when there are deletions), whose body lists the files and says "Review and merge this pull request on GitHub."
- [ ] **When** the sync is done, **Then** the body says "Synced", "N downloaded and saved, M uploaded.", lists the files that came down, shows the link "Pull request #N on `<branch>`" when one was opened, and shows any warning from reading the files; Sync now is hidden and the comparison is made again quietly.
- [ ] **Given** the sync worked, **Then** the project stores the version of each file on the base branch, the files sent in each pull request, and the time of the sync.

**Force pull**

- [ ] **Given** a comparison with a result, **Then** the footer shows "Force pull" on the left of the actions with the hint "Replace this story with its version on `<base branch>`, even when it clashes".
- [ ] **When** the user presses Force pull, **Then** the footer's left side asks "Replace this story with the version on `<base branch>`? What you changed here is lost." with "Keep mine" and "Replace".
- [ ] **When** the user presses Replace, **Then** only the story's own files are downloaded and stored whatever changed here; documents connected with it are not touched; a diagram that exists only here is kept; the body says "Replaced with the version on `<base branch>`" and lists the files that came down.
- [ ] **Given** the base branch has no file for this story, **Then** nothing is replaced and the body says "This story is not on `<base branch>`", explains that it was never merged there or was removed, and offers "Keep it here" and "Remove from here".
- [ ] **When** the user presses "Remove from here", **Then** the story and its diagrams are deleted from this project (not from GitHub), what was remembered about its files is forgotten, and the screen closes.

**Changes that come down by themselves**

- [ ] **Given** an open story and a connected project, **Then** the head of the base branch is checked every 30 seconds and also when the window regains focus.
- [ ] **Given** the head moved and the comparison finds files to download and no conflict, and the story has no unsaved changes, **Then** the files are downloaded and stored, the story is shown from them, and a notice bar says "Updated from `<base branch>` and saved: `<up to four files>` and N more."
- [ ] **Given** the story has unsaved changes when files to download are found, **Then** nothing is downloaded and the notice says "A newer version of this document or its connected documents is on the base branch. Save or discard your changes and it is downloaded."; when the story is saved it is checked again.
- [ ] **Given** a conflict is found during the background check, **Then** nothing is downloaded and the notice, in amber, says "The base branch has changes to this document or its connected documents that clash with yours. Nothing was changed. Solve it on GitHub, then sync."
- [ ] **When** the user dismisses the notice or opens another document, **Then** it disappears.

## Business rules and constraints

- `classify` and `computePlan` in `syncEngine.ts` decide the outcome for each file; this story does not change their logic (Architecture: outbound adapter / infrastructure layer).
- The base branch is the source of truth. BWYN never merges, never force-pushes, and never resolves a conflict: conflicts are resolved on GitHub.
- One sync produces at most one new branch and one pull request, opened from the base branch and named by the backend (ADR-0017: application service responsibility).
- Deletions are not synced, except the old folder of a story that was moved, which is deleted in the same pull request that creates the new folder.
- Downloads are stored atomically, replacing what is here; the screen is rebuilt from what was stored.
- Only the open story and the documents connected with it take part in the sync, plus all ADRs.
- The sync full screen shares the frame (header, document-chip row, scrolling body, footer) with the diagram screen; the frame may be extracted as a shared component if it is used in three or more places (Principle: DRY and KISS), but the two screens' contents remain separate.
- The remembered state (file versions, pull-request associations, last-sync time) moves from browser storage to the project's database if it is not already there (ADR-0010, ADR-0011).
- Assumption: "base branch" is the default branch of the connected repository as stored at connection time (story 017).

## Out of scope

- A branch per story and "last push wins" (stories 048 and 049).
- Syncing architectural enablers, drivers, architecture description, runways, and principles from this screen (they keep their existing dialog until moved here by a later story).
- Choosing which individual files go up.
- Showing a diff of a file's content.
- Deleting a story on GitHub (covered by story 050).

## Tasks

- [ ] Move the remembered sync state (file SHAs, pull-request associations, last-sync timestamp) from browser storage to the project's database with a Flyway migration (ADR-0010, ADR-0011).
- [ ] Add a backend application-service command and inbound REST adapter endpoint that accepts a list of files to upload or delete, creates a branch from the base branch, commits the files, and opens a pull request (ADR-0017).
- [ ] Add a backend query endpoint that returns the current head SHA of the base branch and the status of open pull requests for the project (ADR-0017).
- [ ] Refactor `SyncDialog` into a full-screen component sharing the frame layout (header, document-chip row, scrolling body, footer) with `DiagramDialog`; extract the frame only if a third screen already uses it (Principle: DRY and KISS) (ADR-0007, AE-0002).
- [ ] Move the scope explanation line under the document-chip row and the Force pull confirmation into the footer's left side (ADR-0007).
- [ ] Wire `SyncButton` (disabled states, spinner, amber/blue dots) to open the full screen and start a comparison (ADR-0007, AE-0002).
- [ ] Implement the background polling hook (30-second interval plus window-focus trigger) that runs `classify`/`computePlan` and dispatches downloads or shows the notice bar without user action (ADR-0007, AE-0002).
- [ ] Implement the "Sync now" flow: re-compare, download files atomically, call the upload/delete endpoint, update stored state, and rebuild the screen from stored data (ADR-0007).
- [ ] Implement the Force pull flow: download only the story's own files, handle the "not on base branch" case with "Keep it here" / "Remove from here" (ADR-0007).
- [ ] Write ViewModel unit tests for every comparison outcome (nothing to do, upload, download, waiting, conflict, moved-story deletion) and for the Sync now and Force pull flows, using test doubles for `gitClient` and the backend endpoints (ADR-0007, AE-0002).
- [ ] Write React component tests for the full screen covering: comparing state, each body section, conflict state, result state, Force pull confirmation, "not on base branch" state, and that Escape and Close are disabled while busy (Principle: UI/Front end Tests pay only for what they assert).
- [ ] Verify the full build and test suite completes within the ten-minute budget (Practice: Ten-Minute Build).

## Reference: spike files

The behaviour described here is the one of these files, under `bwyn-frontend/spikes/home-page/`. Use them as the reference when building
the story; only the layout of `SyncDialog` changes.

| File | What it holds |
|---|---|
| `builder/git/SyncButton.tsx` | `SyncButton` (the button, its dot and hints), `SyncDialog` (the screen to turn into a full screen), `SyncNoticeBar` |
| `builder/git/useGitSync.ts` | `check`, `sync`, `forcePull`, `removeMissing`, the poll every 30 seconds and the automatic download |
| `builder/git/syncEngine.ts` | `classify` (the decision per file), `computePlan`, `uploadPlan` (title and body of the pull request), what is remembered |
| `builder/git/scope.ts` | which documents are connected with the story, and which files are theirs |
| `builder/git/gitClient.ts` | the backend endpoints: head, tree, blob, pull requests, upload |
| `builder/git/useGitState.ts` | the remembered state of the project: base SHAs, uploaded files, pull requests, last sync |
| `builder/repoFiles.ts` | the files of a story (`story.md`, `meta.json`, `diagrams/<kind>.mmd`, `ui.html`) and how a pull is read back |
| `builder/storyPaths.ts` | the folder of a story, in a theme or not, to find a story that was moved |
| `builder/DiagramMenu.tsx` | `DiagramDialog`, the full screen whose layout the sync screen copies |
| `builder/FileView.tsx` | where the Sync button sits in the bar, and "Save the story first" |
