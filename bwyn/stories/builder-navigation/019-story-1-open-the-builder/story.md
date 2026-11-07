# Story: Open the builder from the Builder page

- **Story**: 019-story-1-open-the-builder
- **Created**: 2026-11-07
- **Status**: Draft

## Story
As a team member, I want to reach the builder from the Builder page and have the folder structure of my project open, so that I can see where my project's documents live before I do anything with them.

## Acceptance tests

**The Builder page**

- [ ] **Given** a project, **When** the user chooses "Builder" in the menu, **Then** the item is the selected one, the header reads "<project> / Builder", and the page shows the Builder icon, the title "Builder", and the description "Turn intent into working software, one guided step at a time."
- [ ] **Given** the Builder page, **Then** three numbered cards show side by side on a wide screen (one under the other on a narrow one): "1 Prompt: Describe what a user needs in plain language, like a chat.", "2 Story: Claude drafts a user story with acceptance tests, and BWYN creates its folder once you approve.", and "3 Link: Connect the story to drivers, ADRs and architectural enablers."
- [ ] **Given** the Builder page, **Then** under the three cards a card "Start a new build" says "Prompt your way to a new story, or to an architectural enabler." and has a button "Open the builder" with an arrow.
- [ ] **Given** the application starts, or a project is created, **Then** the Builder page is shown with the workspace closed.
- [ ] **Given** the Builder page, **Then** it can be read with no model connected and no repository connected.

**Opening the builder**

- [ ] **Given** the Builder page, **When** the user presses "Open the builder", **Then** the main menu shrinks to a rail of icons (each item keeps its icon, its name as a tooltip, and the selected mark), the page is replaced by the workspace, and the header reads "<project> / Builder / Files".
- [ ] **Given** the workspace, **Then** on the left a tree titled "Files", with the project name under it, shows four folders in this order: `drivers`, `adrs`, `architectural-enablers`, `stories`; each has a chevron and a folder icon, and they are all closed.
- [ ] **Given** the workspace with no file open, **Then** the rest of the area shows a centred message "Select a file in the tree to read it.", with no prompt box and no "New story" button.
- [ ] **Given** the rail, **When** the user presses the project's avatar or "Expand menu", **Then** the menu is full again and the workspace stays open; **When** the user presses "Collapse menu", **Then** the rail is back.
- [ ] **Given** the keyboard, **Then** "Open the builder", every menu item, and every row of the tree can be reached with Tab, and a visible focus ring shows where the focus is.

**Going away and coming back**

- [ ] **Given** the workspace, **When** the user chooses another item of the menu (not Settings), **Then** the workspace closes, the menu is full, and that page shows; **When** the user chooses "Builder" again, **Then** the Builder page shows with the workspace closed.
- [ ] **Given** the workspace, **When** the user chooses "Builder" in the menu, **Then** the workspace stays open.
- [ ] **Given** the workspace, **When** the user chooses another project, **Then** the workspace shows that project's tree with every folder closed and no file open.
- [ ] **Given** the workspace, **When** the page is reloaded, **Then** the Builder page shows with the workspace closed.

## Business rules and constraints

- The tree structure is fixed: four folders in the order `drivers`, `adrs`, `architectural-enablers`, `stories`, and no others; their names match the repository folder names.
- The workspace state (open or closed) belongs to the current project session and resets to closed on page reload and on project switch.
- The workspace contains no prompt box or "New story" button in this story; the three numbered cards describe the full builder that later stories will build.
- All interactive elements must be keyboard-reachable with a visible focus ring (ADR-0007, Architecture: UI layer).
- The Builder page must render without an LLM provider or a Git repository being connected.
- The workspace open/closed state is held in the ViewModel for the Builder feature; it is not persisted to the backend (Architecture: interface-adapters layer) (ADR-0007).
- The menu collapse/expand state is driven by whether the workspace is open, not stored independently; collapsing the menu is a side-effect of opening the workspace (ADR-0007).
- Route definitions and navigation logic reside in the framework/UI layer only; the ViewModel exposes commands and state, not routing calls (ADR-0007, ADR-0020).

## Out of scope

- The prompt box, "New story" button, runway diagram, and all builder interactions that create or edit documents (later stories).
- Opening files and showing their content; what is inside the four folders (stories 2–4 of the builder).
- The menu bar, project selector, and Settings (covered by story 003).
- Backend endpoints: this story is entirely frontend state and presentation; no new API calls are introduced.
- Persisting the workspace open/closed state across browser sessions or devices.

## Tasks

- [ ] Add a `BuilderViewModel` in the interface-adapters layer that holds `isWorkspaceOpen: boolean` and exposes `openWorkspace()` and `closeWorkspace()` commands, with a Jest unit test (ADR-0007, Architecture)
- [ ] Bind `BuilderViewModel` in the Inversify composition root (ADR-0006)
- [ ] Build the `BuilderPage` React component in the UI layer: Builder icon, title, description, three numbered step cards (responsive layout), and the "Start a new build" card with the "Open the builder" button; wire it to `BuilderViewModel.openWorkspace()` (ADR-0007)
- [ ] Build the `BuilderWorkspace` React component in the UI layer: the file tree (title "Files", project name, four closed folders with chevrons) and the empty-state message "Select a file in the tree to read it." (ADR-0007)
- [ ] Implement menu-rail mode in the existing menu component: when `isWorkspaceOpen` is true, collapse to icon-only rail with tooltips and selected mark; expose "Expand menu" / "Collapse menu" controls (ADR-0020, Architecture)
- [ ] Update the breadcrumb in the UI layer to append "/ Files" when the workspace is open (ADR-0007)
- [ ] Wire navigation so that choosing any non-Builder menu item calls `closeWorkspace()` and navigating back to Builder shows the page with the workspace closed; choosing Builder while the workspace is open leaves it open (ADR-0020)
- [ ] Reset workspace state to closed on project switch and on page reload (ADR-0007)
- [ ] Add keyboard focus management: verify Tab order reaches "Open the builder", every menu item, and every tree row; add visible focus rings where missing (Architecture)
- [ ] Write Jest rendering tests for `BuilderPage` and `BuilderWorkspace` covering the acceptance tests above (ADR-0005)
- [ ] Confirm ArchUnitTS rules still pass with the new files in place (ADR-0016)
- [ ] Verify the full frontend build and test suite completes within the ten-minute budget (Practice: Ten-Minute Build)

## Files of the spike

In `bwyn-frontend/spikes/home-page/`. These are the base for the stories. The spike's workspace does much more than these stories ask
for (see the next section), so each file is used in part.

| File | What it holds | Stories |
|---|---|---|
| `data.ts` | `NAV_ITEMS` (the Builder entry: label, title, description, icon), `BUILD_STEPS` (the three cards), `FIXED_DRIVERS` (the five fixed drivers, with their meaning). | 1, 2, 3 |
| `Sections.tsx` | `Builder`: the three numbered cards and the card "Start a new build" with the button "Open the builder" (`onOpen`). `SectionContent` mounts it for the "builder" section. | 1 |
| `App.tsx` | `builderOpen` and `workspace` (open only on the Builder section with a project), `openBuilder` (collapses the menu and opens the workspace), the collapsed rail (`compact`, `collapsed`, `menuExpanded`, "Expand menu", "Collapse menu"), the header breadcrumb, and the choice of what the main area shows (page or workspace). The workspace is keyed by project. | 1 |
| `builder/BuilderWorkspace.tsx` | The frame of the workspace: the tree on the left and the middle. `expanded` (the open folders, `DEFAULT_EXPANDED`), `openRef` (the file that is open), `treeCollapsed` and `treeForced` (the rail and "Show files"), `perform` (opens or closes a file and opens its folder), and the entries built for the tree. Its prompt, drafts, runways, sync, themes and deletes are not used here. | 1, 2, 3, 4 |
| `builder/FolderTree.tsx` | The tree. `REFERENCE_FOLDERS` (drivers and adrs locked, and the enablers), `FolderRow` (chevron, icon, name, lock, count, `aria-expanded`), `FileRow` (icon, file name, tooltip, `aria-current`), and the collapsed rail ("Show files", the icon of the kind that is open). Its themes, "New story", "New theme", runway buttons and dragging are not used here. | 1, 2, 3, 4 |
| `builder/catalog.ts` | `buildEntries` (every driver, written or not, and every ADR becomes an entry with its folder, file name, label, content and `editable: false`), `KIND_META` (icon and colour per kind), `entryPath`, `KIND_ORDER`. | 2, 3, 4 |
| `builder/types.ts` | `Entry` (`ref`, `kind`, `fileName`, `folderPath`, `label`, `content`, `editable`) and `StoredAdr`. | 2, 3, 4 |
| `builder/FileView.tsx` | The panel of an open file: header (icon, path, the lock and `READ_ONLY_NOTE`: "Read-only here. Edit this driver from the Drivers page." and "... this ADR from the ADRs page."), the Close button, the two views and the source block for a read-only file. Its Save, Delete, Sync, links, tasks, diagrams and UI parts are not used here. | 3, 4 |
| `builder/MarkdownView.tsx` | The Markdown reader (`MarkdownView`, built from elements, never HTML) and `MarkdownModeToggle` (rendered and Markdown). | 3, 4 |
| `builder/generate.ts` | `driverMarkdown` and `DRIVER_PLACEHOLDER` ("_Not written yet. Edit this driver from the Drivers page._"): how a driver is written as a file. | 2, 3 |
| `useDrivers.ts` | The drivers of a project: the texts of the fixed ones and the custom list. | 2, 3 |
| `builder/useBuilder.ts` | The ADRs of a project (`adrs`, sorted by number). Its stories, enablers, themes and the rest are not used here. | 2, 4 |
| `useProjects.ts`, `ProjectSelect.tsx` | The active project and its avatar, which the rail shows. | 1 |
| `icons.tsx` | `WrenchIcon`, `FolderIcon`, `ChevronDownIcon`, `ChevronRightIcon`, `LockIcon`, `PanelLeftIcon`, `CompassIcon`, `BookOpenIcon`, `FileTextIcon`, `ZapIcon`, `XIcon`, `EyeIcon`, `CodeIcon`. | 1 to 4 |

## What the spike does that these stories take out or change

- **The middle of the workspace is the prompt.** In the spike, `ChatPanel` fills it, with the chat, the draft and the buttons that create
  stories and enablers. These stories put an empty state there ("Select a file in the tree to read it."). With a file open, the spike shows
  the file on the left and the prompt on the right (a 26 rem column); here the file takes the whole width.
- **The tree header has two buttons**: the runway diagram and "New story". Both are taken out.
- **The tree has more than four folders' worth of rows.** Under `stories` it has themes (create, rename with double click or F2, drag and
  drop, a runway button each), story folders and their file. All of that is taken out. `architectural-enablers` shows its files in the
  spike, and opening them offers editing, links, sync and delete: here it shows "Coming in another story." instead.
- **Folders start open in the spike**: `DEFAULT_EXPANDED` is `stories` and `enablers`. Here all four start closed, since the two that were
  open are inert.
- **The file panel of a driver or an ADR is the same component as a story's**, with the editing parts hidden by `entry.editable`. In the
  spike it also shows "Linked to" and "Referenced by" when something links to the file. Here those are left out until stories and enablers
  exist.
- **The breadcrumb ends with "Prompt"** while the workspace is open. Here it ends with "Files".