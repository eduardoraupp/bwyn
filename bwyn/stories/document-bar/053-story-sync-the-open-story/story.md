# Story: Sync the open story with GitHub on a full screen

- **Story**: 053-story-sync-the-open-story
- **Created**: 2026-11-28
- **Status**: Draft

## Story
As a team member, I want to sync the story I am working on from its bar and see on one full screen what goes to its branch and why, so that I know exactly what my team will review in its pull request before I push.

This story builds on stories 048 and 049: a story goes to its own branch (`story-003`) with one pull request, a sync holds the story's own files, and the branch always wins. It changes how the sync is shown, not what a sync does.

## Acceptance tests

**The Sync button**

- [X] **Given** an open story and the project is connected to GitHub, **Then** the bar shows "Sync" with the refresh icon and the hint "Sync the project with GitHub".
- [X] **Given** the project is not connected to GitHub, **Then** Sync is disabled with the hint "Connect GitHub in Integrations first".
- [X] **While** a comparison or a sync runs, **Then** the refresh icon of the Sync button spins.
- [X] **When** the user presses Sync, **Then** the full screen opens and a comparison with the story's branch starts; nothing is changed here or on GitHub until "Sync now" is pressed.

**The full screen**

- [X] **Given** the full screen is open, **Then** it covers the whole window above the builder, in the same frame as the diagram screen: a header with the branch icon, the title "Sync with GitHub", the line `<owner>/<repo> · <branch> · <base folder or (root)>` and a close button; a scrolling body; and a footer with a status on the left and the actions on the right.
- [X] **While** comparing, **Then** the body says "Comparing with `<branch>`…"; **while** syncing, "Syncing…".
- [X] **When** the user presses the close button, "Cancel" or "Close", or Escape, **Then** the screen closes; **while** a comparison or a sync runs, those are disabled and Escape does nothing.
- [X] **Given** a request failed, **Then** the body shows its error message.
- [X] **Given** the repository is too large for GitHub to list in full, **Then** the body warns "The repository is too large for GitHub to list in full. Some files may be missing from this comparison."
- [X] **Given** a comparison was made, **Then** the footer says "Compared with `<branch>` at `<short head SHA>`", followed by "· Last synced `<date and time>`" when this story was synced before.

**What the body shows**

- [X] **Given** files to push, **Then** the body says what story 048 says ("Push N files to the new branch story-003" or "Push N files to story-003 (pull request #12)") and lists them, with "A pull request to `<base branch>` opens with it." for a new branch.
- [X] **Given** the story was moved to another theme folder, **Then** the files of the old folder are listed under "Delete N files from the old folder".
- [X] **Given** nothing to push, **Then** the body says "Everything is on `<branch>`." and "Sync now" is disabled.
- [X] **Given** the branch has a commit this machine does not know, **Then** the body says that a sync replaces the story here with it and pushes nothing (story 049).
- [X] **Given** a comparison, **Then** a "Details for each file" section is folded at the end of the body, listing each file of the story with what the sync does with it ("Push", "Delete", "Unchanged" or "Not here") and the short SHAs of the file here (L), on the branch now (R) and at the last sync (B), "—" where there is none.

**Sync now**

- [X] **When** the user presses "Sync now", **Then** the sync of story 048 runs, the body says "Syncing…", and then what the sync did ("Pushed to story-003" with the link to its pull request, "Everything is on story-003.", or that the branch replaced the story here).
- [X] **Given** the sync pushed or pulled, **Then** the time of the sync is remembered for the story, and the footer of the next comparison shows it.

## Business rules and constraints

- What a sync does is the one of stories 048 and 049: one branch and one pull request per story, the story's own files only, and the branch always wins. There is no conflict, no waiting state and no Force pull for a story.
- The comparison is asked of the backend; the browser never calls GitHub (ADR-0017).
- The last time a story was synced is kept with what the project knows of its branch, in the database (ADR-0010).
- The sync full screen and the diagram full screen share the same frame: header, scrolling body and footer. The frame is extracted as a shared component only when a third screen uses it (Principle: DRY and KISS); until then the two screens use the same CSS classes.
- No test uses real timers or artificial delays (Principle: UI/Front end Tests pay only for what they assert).

## Out of scope

- A sync of the documents connected with the story (what it links to and what links to it).
- Comparing with the base branch, conflicts, files waiting for a pull request, and Force pull (replaced by the branch per story of stories 048 and 049).
- "Save the story first", until the text of a story can be edited in the builder.
- Showing a diff of a file's content.
- Syncing architectural enablers, drivers, the architecture, runways and principles from this screen.

## Tasks

- [X] Add a Flyway migration that keeps the time of the last sync with what the project knows of the branch of a document, and set it when a sync pushes or pulls (ADR-0010). _(`V26__remember_when_a_document_was_synced.sql`; set by `SyncDocumentCommandHandler` when it pushes or pulls.)_
- [X] Return with the comparison the head SHA it was made at, the time of the last sync, whether the repository listing was truncated, the repository and the base folder, and the details of each file of the story (path, what the sync does with it, SHA here, on the branch and at the last sync) (ADR-0017).
- [X] Turn the Sync dialog into a full screen with the frame of the diagram screen: header, scrolling body, footer with status and actions (ADR-0007). _(`SyncDialog` uses the `diagram-screen` classes of `DiagramDialog`; with two screens the frame is not yet extracted.)_
- [X] Spin the refresh icon of the Sync button while a comparison or a sync runs, and disable closing while busy (ADR-0007).
- [X] Show the footer status, the truncated warning and the folded "Details for each file" section (ADR-0007).
- [X] Write integration tests for the new fields of the comparison and the time of the last sync (ADR-0012). _(In `DocumentSyncIT`.)_
- [X] Write acceptance tests for the full screen: comparing and syncing states, the header line, the footer status, the truncated warning, the details section, closing disabled while busy, and the spinner (Principle: UI/Front end Tests pay only for what they assert). _(`sync-full-screen.acceptance.test.tsx`.)_
- [X] Verify the full build and test suite completes within the ten-minute budget (Practice: Ten-Minute Build). _(`mvn verify` 5m07s plus Jest about 52 s on WSL.)_
