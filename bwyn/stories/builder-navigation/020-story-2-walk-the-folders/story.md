# Story: Walk the folders of the builder

- **Story**: 020-story-2-walk-the-folders
- **Created**: 2026-11-07
- **Status**: Draft

## Story
As a team member, I want to open and close the folders of the builder tree and see which drivers and ADRs my project has, so that I know what exists and where each file lives.

## Acceptance tests

**Opening and closing**
- [ ] **Given** a closed folder, **When** the user clicks its row or presses Enter or Space on it, **Then** it opens, its chevron points down, and its row has `aria-expanded="true"`; **When** the user activates it again, **Then** it closes and `aria-expanded="false"`.
- [ ] **Given** the four folders, **Then** each opens and closes independently and several can be open at the same time.
- [ ] **Given** folders that are open, **When** a file is opened and closed (stories 3 and 4), **Then** the open/closed state of every folder is unchanged.
- [ ] **Given** folders that are open, **When** the workspace is closed and reopened, **Then** all folders are closed.

**Folder rows**
- [ ] **Given** any folder row, **Then** it shows a chevron, a folder icon, the folder name (truncated with "…" when long), and a count of items on the right.
- [ ] **Given** the `drivers` and `adrs` folder rows, **Then** each shows a lock icon labelled "Read-only"; the `architectural-enablers` and `stories` rows do not.
- [ ] **Given** the `drivers` folder, **Then** its count equals the five fixed drivers plus any drivers the team added; **Given** `adrs`, **Then** its count equals the number of ADRs; **Given** `architectural-enablers` and `stories`, **Then** each count equals the number of items the project holds.

**`drivers` folder**
- [ ] **Given** `drivers` is open, **Then** it lists the five fixed drivers first, in this order: `core-functionality.md`, `architectural-concerns.md`, `constraints.md`, `quality-attributes.md`, `design-purpose.md`, followed by team-added drivers in the order they were added, each as `<identity>.md`.
- [ ] **Given** a fixed driver that has no text yet, **Then** it is listed all the same.
- [ ] **Given** a file row in `drivers`, **Then** it shows a file icon, the file name truncated with "…" when long, and a tooltip containing the driver's display name (e.g. "Core Functionality").

**`adrs` folder**
- [ ] **Given** `adrs` is open and the project has at least one ADR, **Then** it lists one file per ADR in ascending order of number, with the repository file name (e.g. `0011-use-typescript-for-the-frontend.md`) and a tooltip of the form "ADR 0011. Use TypeScript for the frontend".
- [ ] **Given** `adrs` is open and the project has no ADRs, **Then** it shows the italic line "No ADRs yet."
- [ ] **Given** ADRs of any status (Proposed, Accepted, Deprecated, Superseded), **Then** all are listed.

**`architectural-enablers` and `stories` folders**
- [ ] **Given** either of these two folders is opened, **Then** it shows the italic line "Coming in another story." and can be closed again like any folder.
- [ ] **Given** the tree, **Then** there is no button to create a story, an enabler, or a theme; nothing can be dragged or dropped.

**Immutability**
- [ ] **Given** any file or folder row inside `drivers` or `adrs`, **Then** a double-click, F2, a drag, a right-click, and Delete each have no effect on the tree.

**Keeping up with the project**
- [ ] **Given** a driver was added on the Drivers page, **When** the user opens the workspace, **Then** the tree lists it and the `drivers` count is correct.
- [ ] **Given** an ADR was added on the ADR page, **When** the user opens the workspace, **Then** the tree lists it and the `adrs` count is correct.
- [ ] **Given** two projects, **Then** each shows only its own drivers and ADRs.

## Business rules and constraints
- The tree is read-only: it displays what the project holds and never writes anything (Architecture: UI layer, no side effects).
- A file's path is its path in the repository relative to the BWYN folder: `drivers/<identity>.md` and `adrs/<number>-<slug>.md`.
- The five fixed drivers are always listed, written or not, because they belong to every project. A driver the team has removed is not listed.
- `architectural-enablers` and `stories` are placeholders: their counts reflect the items the project holds, but the items are not listed, opened, or acted on until their own stories.
- Folder open/closed state is session-local UI state; it is not persisted across workspace sessions (ADR-0007: state lives in the ViewModel, not in the entity or use-case layer).
- The tree is built from a static description of the four folders (key, display name, kind, lock flag); the list of folders is not fetched from the backend.

## Out of scope
- Opening a file to read or edit its content (stories 3 and 4 of the builder).
- Listing, opening, creating, editing, moving, linking, or deleting items in `architectural-enablers` and `stories`.
- Themes, the prompt box, and any other builder interaction that creates or edits documents.
- Writing drivers and ADRs (covered by their own pages).
- Search or filter within the tree.
- Persisting folder open/closed state across browser sessions or devices.
- Backend endpoints: this story is entirely frontend state and presentation; no new API calls are introduced.

## Tasks
- [ ] Define a static folder-descriptor array (key, display name, kind, lock flag, placeholder text) in the `ui` layer and derive the tree from it (ADR-0007, Principle: Declarative over imperative)
- [ ] Implement the `BuilderTreeViewModel` in the `interface-adapters` layer, exposing folder open/closed state, item lists for `drivers` and `adrs`, and counts for all four folders (ADR-0007)
- [ ] Bind `BuilderTreeViewModel` in the Inversify composition root; inject the existing driver and ADR query use cases (ADR-0006)
- [ ] Build the `FolderRow` React component: chevron, folder icon, name with overflow ellipsis, optional lock badge, count; toggle on click, Enter, and Space; set `aria-expanded` (ADR-0005)
- [ ] Build the `FileRow` React component: file icon, file name with overflow ellipsis, tooltip with display name (ADR-0005)
- [ ] Render `drivers` file list: five fixed drivers first in specified order, then team-added drivers in creation order, each as `<identity>.md`
- [ ] Render `adrs` file list: ADRs in ascending number order; show italic "No ADRs yet." when the list is empty; include all statuses
- [ ] Render `architectural-enablers` and `stories` with italic "Coming in another story." placeholder when open; omit all create/rename/drag controls
- [ ] Suppress all mutation interactions on `drivers` and `adrs` rows: double-click, F2, drag, right-click context menu, and Delete key do nothing
- [ ] Reset all folder open/closed state to closed when the workspace is closed
- [ ] Write Jest unit tests for `BuilderTreeViewModel`: correct order and counts for drivers (including unwritten fixed drivers), empty-ADR case, multi-project isolation (ADR-0005)
- [ ] Write Jest unit tests for `FolderRow` and `FileRow`: aria attributes, truncation, lock badge presence/absence, tooltip content
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

## Spike details worth keeping in mind

- The folder keys in the spike are `drivers`, `adrs`, `enablers` (for `architectural-enablers`) and `stories`.
- `buildEntries` gives each driver the file name `<identity>.md` and the label of its name; each ADR its stored `fileName` and the label
  `ADR 0011. <title>`; both `editable: false` and `links: []`.
- Opening a file makes `treeCollapsed` true unless `treeForced` is set by "Show files". Opening another file from the full tree clears
  `treeForced`, so the rail returns.
- `perform` opens the folder of the file that is opened, so a file opened from a link elsewhere (in later stories) shows in its folder.
- Choosing "Builder" in the menu while the workspace is open does nothing: only another item closes it (`setBuilderOpen(false)`), and the
  Builder page appears only after going away and coming back.
- Changing the project keeps `builderOpen`, and `BuilderWorkspace` is keyed by project, so it opens again on the new project with its state
  reset.
- The count of a folder is `entries.filter(kind).length`. For `drivers` that is always at least five.
- A driver written as Markdown is `# <name>`, the meaning, and the text, or `_Not written yet. Edit this driver from the Drivers page._`.
- The rendered view builds React elements from the Markdown and has a message "Nothing written yet." for a file with no blocks, which a driver
  or an ADR never reaches.
- Everything is read from the browser's storage in the spike, per project, so the workspace shows what the Drivers and ADR pages saved there.
