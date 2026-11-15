# Story: Create a theme and name it

- **Story**: 042-story-1-create-a-theme
- **Created**: 2026-11-15
- **Status**: Draft

## Story
As a team member, I want to create a theme folder inside `stories` and give it a name, so that I have a place to gather the stories that belong together.

## Acceptance tests

Creating

- [ ] **Given** the tree with the `stories` folder open, **When** the user presses "New theme" (the folder-with-a-plus button on the `stories` row), **Then** a folder `new-theme` shows inside `stories`, with the count 0, and its name is an edit field that has the focus and the whole text selected.
- [ ] **Given** the `stories` folder is closed, **When** the user presses "New theme", **Then** `stories` opens, and the new theme shows as above.
- [ ] **Given** a theme `new-theme` exists, **When** the user creates another, **Then** it is named `new-theme-2`, then `new-theme-3`.
- [ ] **Given** the `drivers`, `adrs` and `architectural-enablers` rows, **Then** none of them has a "New theme" button.

Naming it

- [ ] **Given** the name field of a new theme, **When** the user types "Checkout flow" and presses Enter, **Then** the folder is named `checkout-flow`.
- [ ] **Given** the name field, **When** the user leaves it (Tab or a click elsewhere) with a good name, **Then** the name is saved as with Enter.
- [ ] **Given** the name field of a new theme, **When** the user presses Escape, **Then** the theme stays, named `new-theme`.
- [ ] **Given** a name that is empty, starts with a digit, contains accents, exceeds 40 characters, or duplicates an existing theme slug, **Then** the field shows the matching validation message and the name is not saved.

What the tree shows

- [ ] **Given** a theme with no story, **When** it is open, **Then** it shows "Empty. Drag a story folder here." in italics.
- [ ] **Given** themes and story folders outside themes, **Then** the themes come first, in alphabetical order, then the story folders outside any theme, by number.
- [ ] **Given** a theme, **Then** its row shows the count of the stories in it. The count of `stories` is the number of all stories, in themes or not.
- [ ] **Given** a project with themes but no story, **Then** `stories` shows its themes and not "No stories yet. Write a prompt.".
- [ ] **Given** a theme, **When** the user clicks it once, **Then** it opens or closes, like any folder.

Keeping it, and new stories

- [ ] **Given** themes, empty ones too, **When** the page is reloaded, **Then** the same themes are there.
- [ ] **Given** a project saved before themes existed, **When** it is opened, **Then** it has no theme, every story is outside any theme, and nothing in the stories needed a change.
- [ ] **Given** a story in a theme is open, **When** a new story is approved, **Then** the new story folder is in `stories`, outside any theme, and its number continues the count of all stories across every theme (with 001 to 003 in themes, it is 004).

## Business rules and constraints

- A theme name is lower-case words joined by `-`, with no accents, starting with a letter, of at most 40 characters, and different from every other theme in the project.
- A new theme is named `new-theme`, or `new-theme-<n>` with the first free number from 2.
- A theme with no story is kept in the project (the list of themes holds empty ones too). It reaches the repository only with its first story, since Git keeps no empty folder.
- The `themes` table stores `id`, `project_id`, and `slug`; `project_id` references the projects table (ADR-0010, ADR-0017).
- Hibernate DDL auto-generation remains disabled; the table is created by a Flyway migration script (ADR-0010, ADR-0011).
- The domain core (`bwyn-core`) holds the `Theme` entity and the `ThemeRepository` port interface; the outbound adapter in `bwyn-infrastructure` implements the port using Spring Data JPA (ADR-0017).
- The inbound adapter in `bwyn-rest` exposes the create-theme command; it calls the application service in `bwyn-application`, which invokes the repository port (ADR-0017).
- The frontend `Theme` entity and `ThemeRepository` port live in the `entities` layer; the use case lives in `use-cases`; the ViewModel lives in `interface-adapters`; the tree component lives in `ui` (ADR-0007, AE-0002).

## Out of scope

- Removing a theme and what happens to its stories.
- Themes in `drivers`, `adrs`, or `architectural-enablers`. Themes inside themes.
- A description, colour, or icon of a theme. Reordering themes by hand.
- A filter or search by theme. Sending the theme to the model in a prompt.
- Moving a story into or out of a theme (a separate story).

## Tasks

- [ ] Write Flyway migration `V2__create_themes.sql` adding the `themes` table with `id`, `project_id` (FK to projects), and `slug` columns (ADR-0010).
- [ ] Add the `Theme` entity and `ThemeRepository` port interface in `bwyn-core` under the builder bounded context (ADR-0017).
- [ ] Implement `ThemeRepositoryAdapter` in `bwyn-infrastructure` using Spring Data JPA; verify ArchUnit rules still pass (ADR-0009, ADR-0017).
- [ ] Add `CreateThemeCommand`, `CreateThemeUseCase` port, and `CreateThemeService` in `bwyn-application`; enforce the slug rules and `next-theme-name` logic there (ADR-0017).
- [ ] Add `POST /api/projects/{projectId}/themes` in `bwyn-rest`, annotated with OpenAPI `@Operation` and `@ApiResponse` (ADR-0014); call `CreateThemeUseCase`.
- [ ] Add a `GET /api/projects/{projectId}/themes` endpoint in `bwyn-rest` that returns the ordered list of themes for a project (ADR-0014).
- [ ] Write a Testcontainers integration test for `ThemeRepositoryAdapter` covering create and list (ADR-0012).
- [ ] Add the `Theme` entity, `ThemeRepository` port, and `CreateThemeUseCase` to the frontend `entities` and `use-cases` layers; bind them in the composition root (ADR-0007, AE-0002).
- [ ] Add `ThemeViewModel` in `interface-adapters` with `createTheme`, `themes` (sorted: themes first alphabetically, then story folders by number), and `nextThemeName` (ADR-0007).
- [ ] Add `themeSlug`, `checkThemeName` (with validation messages for empty, digit-first, accents, too long, duplicate), `nextThemeName`, and `sortThemes` pure functions in `use-cases` and cover them with Jest unit tests (ADR-0005).
- [ ] Update the builder tree React component in `ui` to render themes above story folders, show the per-theme story count and the total count on `stories`, and show the empty-theme hint (ADR-0007, AE-0002).
- [ ] Add the "New theme" button on the `stories` row; wire it to `ThemeViewModel.createTheme`, open `stories` if closed, and open the inline name field with focus and full selection (ADR-0007).
- [ ] Implement the inline name field: Enter saves, Tab/blur saves a valid name, Escape keeps the default slug, validation message shown below the field; no artificial delays in tests (Principle: UI/Front end Tests pay only for what they assert).
- [ ] Verify that projects saved before themes existed load without error and show no themes (backward-compatibility test).
- [ ] Confirm the full build and test suite completes within the ten-minute budget (Practice: Ten-Minute Build).

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

The four test files pass (27 tests). They are outside the `testMatch` of `jest.config.js` (`src/**`), so they run with
`npx jest --roots '<rootDir>/spikes' --testMatch '**/*.test.ts'` from `bwyn-frontend/`.