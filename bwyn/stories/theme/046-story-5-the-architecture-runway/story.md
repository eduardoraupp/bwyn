# Story: The architecture runway of a theme

- **Story**: 046
- **Created**: 2026-11-15
- **Status**: Draft

## Story
As a team member, I want to see the architecture runway of one theme, so that I know which enablers its stories need and in which order the work of that theme can be built, without the rest of the project in the way.

## Acceptance tests

**Opening the runway**
- [ ] **Given** a theme that contains at least one story, **Then** its row shows a runway button labelled "Architecture runway of \<theme\>". **When** the user presses it, **Then** a full-screen view opens with the title "Architecture runway of \<theme\>" and the count of its enablers and stories.
- [ ] **Given** a theme that contains no story, **Then** its runway button is disabled with the tooltip "Add a story to this theme to draw its runway".
- [ ] **Given** the runway of a theme has never been saved, **When** the runway screen opens, **Then** it displays the message "Generate it from the stories in the folder '\<theme\>' and the architectural enablers linked to them …" and a "Generate runway" button.

**What the runway draws**
- [ ] **Given** a theme whose stories are linked to architectural enablers, **When** the runway is generated, **Then** it draws only the stories in that theme's folder and the enablers linked to them (whichever side holds the link); stories from other themes, stories outside any theme, and enablers linked only to those other stories or to other enablers are not drawn.
- [ ] **Given** an enabler linked to a story of the theme, **Then** the enabler is placed to the left of the story. **Given** two stories of the theme are linked to each other, **Then** they appear in ascending order of their story numbers.
- [ ] **Given** a generated runway, **Then** the text "Only the stories in the folder '\<theme\>' and the architectural enablers linked to them are drawn." appears above the explanation of how to read the diagram.
- [ ] **Given** no story of the theme is linked to an enabler or to another story of the same theme, **When** the runway is generated, **Then** it displays the message "No story of this theme is linked to an architectural enabler or to another story of the theme, so there is no order to draw. Use 'Add link' on a story."

**Saving the runway**
- [ ] **Given** a generated runway, **When** the user presses Save, **Then** the runway is saved for the theme together with the time it was saved.
- [ ] **Given** a saved runway, **When** a link or a story of the theme changes after the save, **Then** the screen indicates that links or items have changed since the runway was saved, and pressing Regenerate redraws it from the current state.
- [ ] **Given** a generated runway that has not been saved, **When** the user closes the runway screen, **Then** a confirmation is required and the unsaved draft is discarded on confirmation.
- [ ] **Given** the runway of a theme is saved or regenerated, **Then** the runway of the whole project is not affected, and changes to the project runway do not affect the theme runway.

**Keeping the runway in the repository**
- [ ] **Given** a saved theme runway, **Then** the footer states it is kept in the repository as `stories/<theme>/runway.mmd`, and Sync covers that one file only with the note "The saved architecture runway of the theme \<theme\>, which is one file, stories/\<theme\>/runway.mmd (Mermaid source). Other documents are left out."
- [ ] **Given** the runway has not been saved, or the open runway has unsaved changes, **Then** Sync is disabled with the reason displayed.
- [ ] **Given** the repository contains `stories/<theme>/runway.mmd` and the theme exists in the project, **When** the project connects or pulls, **Then** the runway file is downloaded exactly as stored, the import summary counts it as a theme runway, and any runway already saved locally for that theme is left unchanged.
- [ ] **Given** a story of the theme is synchronised, **Then** `stories/<theme>/runway.mmd` is not included in that sync.

## Business rules and constraints
- The runway is computed from the links between documents with no request to a model, in the same way as the project-level runway (Principle: Declarative over imperative).
- One file per theme: `stories/<theme>/runway.mmd`, placed next to the story folders of that theme. The filename `runway.mmd` cannot be confused with a story folder.
- Content downloaded from the repository is stored byte-for-byte as received, so that it compares equal to the repository and is never shown as changed.
- The runway button and the runway screen reside in the `ui` layer; the logic that computes runway entries from links resides in the use-case layer; the saved runway state resides in the entity layer (ADR-0007, AE-0002).
- The backend stores and serves the saved runway content through the existing hexagonal port/adapter structure; no domain logic is placed in an adapter (ADR-0017, AE-0005).

## Out of scope
- A single runway that shows every theme side by side, or themes as groups in the project-level runway.
- Editing the Mermaid source by hand.
- Reordering stories within the runway by hand.
- A runway for stories that are not in any theme.

## Tasks
- [ ] Write and test a `themeRunwayEntries` use-case function that, given a theme, returns the stories in its folder and the enablers linked to them, sorted by story number (use-case layer, ADR-0007)
- [ ] Extend the runway screen component to accept an optional theme parameter and render the theme-specific title, description texts, empty-state message, and repository path (ui layer, ADR-0007)
- [ ] Add saved-theme-runway state to the project entity (`themeRunways` map keyed by theme name) and update it on rename so the saved runway moves with the theme (entity layer, ADR-0007)
- [ ] Add the runway button to the theme row in the builder tree, disabled with tooltip when the theme has no stories (ui layer, ADR-0007)
- [ ] Add a backend endpoint and outbound adapter to persist and retrieve the saved runway for a theme as `stories/<theme>/runway.mmd` (adapter layer, ADR-0017)
- [ ] Implement Sync for the theme runway as a single-file operation (`themerunway:<theme>`) with the scoped note in the sync window, and disable Sync when the runway is unsaved or has changes
- [ ] Implement pull and import of `stories/<theme>/runway.mmd`: store the content exactly as received, count it in the import summary, and leave any existing local runway unchanged on connect
- [ ] Add a Flyway migration for any new persistence column or table required to store the theme runway content and its saved timestamp (ADR-0010)
- [ ] Verify the full build and test suite passes within the ten-minute budget (Practice: Ten-Minute Build)

# Reference

## Files of the spike

All under `bwyn-frontend/spikes/home-page/`. The new files are untracked; the others have uncommitted changes on the branch `spike-only-fe`.

| File | What it holds for themes | Story |
|---|---|---|
| `builder/themes.ts` (new) | `themeSlug` (accents, case, "-", 40 characters), `checkThemeName` and its three messages, `nextThemeName` (`new-theme`, `new-theme-2`), `sortThemes` (alphabetical, no repeats). | 1, 2 |
| `builder/storyPaths.ts` (new) | `THEME_PATTERN`, `STORY_FILE`, `STORY_META`, `THEME_RUNWAY`, `storyFolder`, `themeRunwayPath`, `storyPlaceOf` (theme, identity and folder of a path). | 4, 5 |
| `builder/FolderTree.tsx` | The "New theme" button on `stories`, the theme rows (count, tooltip, double-click and F2, runway button), `ThemeNameField`, the empty hint, the order (themes, then stories by number), `renderStory`, and the drag and drop (`dropProps`, `canDropOn`, the highlight). | 1, 2, 3, 5 |
| `builder/BuilderWorkspace.tsx` | `newTheme`, `renameThemeTo` (keeps the theme open under its new name), `moveStoryTo` (opens the theme it lands in), `perform` (opens the theme of a story opened from a link), and the runway of a theme (`runwayTheme`, `useThemeRunway`, the sync focus `themerunway:<theme>`). | 1, 2, 3, 5 |
| `builder/useBuilder.ts` | `themes`, `themeRunways`, `createTheme`, `renameTheme` (stories and runway move with it), `moveStory`, `saveThemeRunway`, and `migrate` for projects saved without themes. | 1, 2, 3, 5 |
| `builder/types.ts` | `Entry.theme`, `StoredStory.theme`, `ProjectBuilder.themes`, `ProjectBuilder.themeRunways`, `SavedRunway`. | 1, 3, 5 |
| `builder/catalog.ts` | `buildEntries`: the folder path of a story is `['stories', <theme>?, '<number>-<slug>']`, and the entry carries `theme`. | 3 |
| `builder/repoFiles.ts` | `buildRepoFiles` (story files in the theme folder, theme runways), `parsePulled` (theme from the folder, the warning for a story in two places, theme runways), `mergePulled` (themes and runways, what is here wins when the place is not clear), `MODEL_PATHS`. | 4, 5 |
| `builder/git/syncEngine.ts` | The `delete` action, `deletions` in the plan, `leftBehind`, `DELETED_SHA`, the conflict and waiting cases, the pull request title and description with deletions, `recordUpload` with deletions. | 4 |
| `builder/git/scope.ts` | `pathMatcher` (a story's files wherever its folder is, `themerunway:<theme>`), `refOfMeta` for a `meta.json` in a theme. | 4, 5 |
| `builder/git/useGitSync.ts` | Passes `themeRunways` to the files, sends uploads and deletions together, the title "Architecture runway of <theme>". | 4, 5 |
| `builder/git/SyncButton.tsx` | The section "Delete N files from the old folder", the note for the runway of a theme, and the counts that include deletions. | 4, 5 |
| `builder/git/deleteFromRepo.ts` | Removes a story's folder wherever it is, with `storyPlaceOf`. | 4 |
| `builder/git/importExisting.ts` | Leaves alone, when connecting, a story here already in any theme folder, and a theme runway saved here. | 4, 5 |
| `GitIntegrationCard.tsx` | Counts "N theme runways" in the import summary, and passes `themeRunwayIds`. | 5 |
| `builder/runway.ts` | `themeRunwayEntries`. | 5 |
| `builder/RunwayDialog.tsx` | The optional `theme`: title, texts, empty message, the path `stories/<theme>/runway.mmd`; `LEGEND` exported. | 5 |
| `builder/useThemeRunway.ts` (new) | The draft, save and import of the runway of one theme, with the shape of the project runway, and `themeRunwayRef`. | 5 |
| `builder/ThemeRunwayDialog.tsx` (new) | A first, read-only version of the theme runway, not saved and not synced. **Not used anywhere**: `RunwayDialog` with `theme` replaced it. | none |
| `icons.tsx` | `FolderPlusIcon` for "New theme". `WorkflowIcon` (already there) for the runway button. | 1, 5 |
| `builder/themes.test.ts` (new) | Name rules, new names, the story paths in the repository, the warning for a story in two places, the theme runway file and its sync scope. | 1, 2, 4, 5 |
| `builder/runway.theme.test.ts` (new) | What the runway of a theme draws and its order. | 5 |
| `builder/git/syncEngine.themes.test.ts` (new) | A moved story: upload and delete in one change, conflict, waiting for an open pull request, a story that did not move. | 4 |
| `builder/git/importExisting.themes.test.ts` (new) | Connecting with a story here already in a theme folder, and a theme runway saved here. | 4, 5 |
| `tsconfig.spike-check.json` (new, in `bwyn-frontend/`) | Type-checks the spike files (`noEmit`). | all |
