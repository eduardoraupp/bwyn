# Story: Keep the themes in the repository

- **Story**: 045
- **Created**: 2026-11-15
- **Status**: Draft

## Story
As a team member, I want the themes and the place of each story to be in the repository, so that the team and the other tools see the same folders I see, and a story is never in the repository twice.

## Acceptance tests

**Where the files go**
- [ ] **Given** a story in the theme `payments`, **When** it is synchronised, **Then** its files go to `stories/payments/003-checkout-form/` (`story.md`, `meta.json`, `ui.html`, and the diagrams), and `meta.json` does not hold the theme name.
- [ ] **Given** a story outside any theme, **When** it is synchronised, **Then** its files go to `stories/003-checkout-form/`, as before.

**A story that moved**
- [ ] **Given** a story previously synchronised at `stories/003-checkout-form/` and since moved into the theme `payments`, **When** the user opens Sync on it, **Then** the sync window lists the files to upload under the theme folder and a section "Delete 4 files from the old folder" with the old paths, and the hint "The story was moved to another folder here. The same pull request adds it there and deletes it here, so the repository never holds it twice."
- [ ] **Given** that sync window, **When** the user sends it, **Then** one pull request adds the files in the theme folder and deletes the old ones; its title is "BWYN: update 4 files and delete 4 files of \<project\>", and its description lists the deleted files under "Deleted, because the story was moved to another folder (its files are added there above):".
- [ ] **Given** that pull request is still open, **When** the user syncs the story again, **Then** the old files are shown as waiting for that pull request, and no second pull request deletes them.
- [ ] **Given** an old file of the moved story was changed in the repository since the last sync, **When** the user syncs, **Then** that file is shown as a conflict, nothing is deleted, and the hint says it must be resolved on GitHub.
- [ ] **Given** a theme renamed from `checkout-flow` to `payments` with stories 003 and 007 previously synchronised under `checkout-flow`, **When** only story 003 is synchronised, **Then** only 003 moves in the pull request; story 007 stays under `checkout-flow` in the repository until it is synchronised, and neither story is lost or shown twice.
- [ ] **Given** a story that did not move, **When** it is synchronised, **Then** nothing of it is deleted and nothing of another story is touched.

**Reading the repository**
- [ ] **Given** a repository with stories in theme folders, **When** the project connects to it or pulls, **Then** the themes appear in the builder tree with their stories inside them.
- [ ] **Given** the project connects and a story is already present locally in any theme folder, **When** the same story exists in the repository, **Then** the local story is left as it is and its theme placement is not overwritten.
- [ ] **Given** the repository holds the same story in two places (e.g. `stories/003-checkout-form/` and `stories/payments/003-checkout-form/`), **When** it is pulled, **Then** the story appears once in the position it has locally (outside any theme if it is new locally), and a warning reads "The story 003-checkout-form is in the repository in more than one place (stories/003-checkout-form and stories/payments/003-checkout-form). It is shown once, where it is here."; nothing is deleted.
- [ ] **Given** a `meta.json` inside a theme folder, **When** it is read, **Then** its links are treated as the links of the story in that folder.

**Removing a story and empty themes**
- [ ] **Given** a story inside a theme, **When** the user removes it, **Then** its folder is deleted from inside the theme folder in the repository.
- [ ] **Given** a theme that contains no stories, **Then** it exists locally only; it reaches the repository with the first story moved into it and synchronised.

## Business rules and constraints
- The folder path is the sole source of truth for a story's location: `stories/<theme>/<NNN-slug>/` or `stories/<NNN-slug>/`. Theme folder names start with a letter; story folder names start with a digit, so the two are never ambiguous.
- A sync deletes only the old files of a story that moved, and only those that were synchronised before and have not been changed in the repository since. No other files are ever deleted by a sync.
- A move is expressed as a deletion of the old files and an upload of the new ones in the same pull request; Git rename is not used.
- Each story moves in the repository only when it is explicitly synchronised; a local rename or drag-and-drop does not reach the repository by itself.
- `meta.json` must not store the theme name; the folder path alone encodes it (Principle: Declarative over imperative).
- Assumption: the sync infrastructure (pull-request creation, conflict detection, and the "waiting for open PR" state) is already established by earlier sync stories; this story extends it with the delete action and the moved-story path logic.

## Out of scope
- Synchronising a whole theme as one unit.
- Moving files in the repository using a Git rename operation.
- Resolving a conflict of a moved story inside BWYN.
- Reordering stories within a theme in the repository.

## Tasks
- [ ] Create `storyPaths.ts` in the interface-adapters layer that encodes all story path patterns: story folder with and without a theme, its files, and its `meta.json` (ADR-0007, Architecture)
- [ ] Update the sync path builder to place a story's files under its theme folder, and update the pull reader to derive the theme from the folder path when pulling
- [ ] Add duplicate-location detection on pull: when the same story slug appears in two paths, emit the warning and keep the local placement
- [ ] Add the `delete` action to the sync plan: identify old files of a moved story, check each for conflicts and for an open pull request, and record them as pending deletion
- [ ] Extend the sync window component (UI layer) to display the "Delete N files from the old folder" section and the moved-story hint
- [ ] Extend pull-request creation in the outbound adapter to include the deletions in the same request, with the prescribed title and description format
- [ ] Update story removal to delete the story folder from inside its theme folder in the repository
- [ ] Update the connect/pull flow to leave a story's local theme placement unchanged when the same story already exists locally
- [ ] Write Jest tests against a fake repository covering: a story moved into a theme, a conflict on an old file, an open pull request blocking re-deletion, a story that did not move, a pull with themes, and a story present in two repository locations (Practice: Pair programming, ADR-0005)
- [ ] Run the full build and confirm the ten-minute budget is met (Practice: Ten-Minute Build, Practice: Continuous Integration)
