# Story: Read a driver from the tree, read-only

- **Story**: 021-story-3-read-a-driver
- **Created**: 2026-11-07
- **Status**: Draft

## Story
As a team member, I want to open a driver from the builder tree and read it rendered or as Markdown, so that I can see what steers my project without leaving the builder, knowing that I cannot change it there.

## Acceptance tests

**Opening a file**
- [X] **Given** the `drivers` folder is open in the tree, **When** the user presses a driver file row, **Then** the file opens in a panel to the right of the tree, its row is marked `aria-current`, and its folder stays open.
- [X] **Given** a file is open, **Then** the tree shrinks to a rail showing a "Show files" button, a thin separator, and the icon of the open file's kind labelled "Viewing a file from drivers" with a tooltip "drivers".
- [X] **Given** the rail is shown, **When** the user presses "Show files", **Then** the full tree appears beside the open file. **When** the user selects another driver from the full tree, **Then** that driver opens and the tree returns to the rail.
- [X] **Given** a file is open, **When** the user presses the same file row again, **Then** nothing changes. **When** the user presses a different driver row, **Then** that driver replaces the first in the panel.

**The file panel**
- [X] **Given** the file panel is open for `constraints.md`, **Then** the header shows the driver icon, the path `drivers/constraints.md` in monospace (truncated with "…" and shown in full as a tooltip), a lock icon with the note "Read-only here. Edit this driver from the Drivers page.", and a Close button labelled "Close drivers/constraints.md".
- [X] **Given** the file panel is open, **Then** it contains no Save, Delete, Sync, "Add link", "Add task", diagram, UI, or "Cross enabler" button, and no field that accepts keyboard input.
- [X] **Given** the file panel is open, **Then** two icon buttons let the user choose between the rendered view (active when the file first opens) and the Markdown source view.
- [X] **Given** the rendered view is active, **Then** headings, paragraphs, lists, bold, italic, inline code, block quotes, tables, and links are rendered as HTML elements built from React components, never from `dangerouslySetInnerHTML`. A link whose `href` begins with `https://` or `http://` opens in a new tab; any other address is rendered as plain text only.
- [X] **Given** the Markdown source view is active, **Then** the raw Markdown is displayed in a read-only block with long lines wrapped; the user can select and copy the text but cannot type into it.
- [X] **Given** a driver that has text, **Then** the Markdown content is `# <name>`, the driver's meaning (fixed drivers only), and the saved text, in that order.
- [X] **Given** a driver that has no text, **Then** the panel shows "_Not written yet. Edit this driver from the Drivers page._" in italics in the rendered view and the equivalent Markdown literal in the source view.
- [X] **Given** the user edited the driver on the Drivers page and then opens the builder workspace, **Then** the file panel shows the saved text, not any unsaved draft.
- [X] **Given** the user has switched to the Markdown source view and then opens a different file, **Then** the new file opens in the rendered view.

**Closing**
- [X] **Given** the file panel is open, **When** the user presses the Close button, **Then** the panel closes, the tree returns to its full width, the empty state "Select a file in the tree to read it." is shown in the content area, and the folders that were open before the file was opened remain open.

## Business rules and constraints
- A driver is immutable in the builder. No write operation (edit, save, rename, move, delete) is available from the file panel (ADR-0007, Architecture).
- The panel always shows the persisted text from the project store, never a draft from the Drivers page.
- The Markdown renderer builds React elements only; it never sets raw HTML, so content imported from a repository cannot inject markup (Principle: Declarative over imperative).
- The "Linked to" and "Referenced by" sections are absent from the driver file panel; they are introduced when stories and enablers exist.
- The five fixed drivers always appear in the tree, whether or not their text has been written; custom drivers appear only if they have been added on the Drivers page.
- Assumption: the project store (used by the Drivers page) is readable from the builder workspace within the same browser session; no new persistence mechanism is introduced by this story.

## Out of scope
- Opening an ADR from the tree (story 4 of the builder).
- Anything related to stories and architectural enablers in the tree.
- Editing a driver or navigating to the Drivers page from the file panel beyond the read-only note.
- "Linked to" and "Referenced by" sections in the file panel.
- Copying the path, downloading the file, and printing.
- The prompt box, "New story" button, runway diagram, and any builder interaction that creates or edits documents.

## Tasks
- [X] Add open-file state to the workspace: track which entry is open, mark its tree row `aria-current`, keep its folder open, and collapse the tree to the rail on open.
- [X] Build the rail: "Show files" button, separator, and kind icon with label and tooltip; restore the full tree on "Show files" and return to the rail when a new file is chosen from the full tree.
- [X] Build the read-only file panel shell: driver icon, truncated path with tooltip, lock note, and Close button; wire Close to clear the open-file state and restore the full tree.
- [X] Enforce read-only: confirm the panel contains no Save, Delete, Sync, "Add link", "Add task", diagram, UI, "Cross enabler" button, or editable field.
- [X] Add the rendered/source toggle: default to rendered on every file open; switch to source on demand; reset to rendered when a new file opens.
- [X] Implement the Markdown renderer using React elements (no `dangerouslySetInnerHTML`): headings, paragraphs, lists, bold, italic, inline code, block quotes, tables, and safe external links in a new tab.
- [X] Implement the Markdown source block: read-only, wrappable, selectable and copyable plain-text display.
- [X] Implement `driverMarkdown`: compose `# <name>`, the meaning line for fixed drivers, and the saved text; substitute the "Not written yet" italic placeholder when text is absent.
- [X] Read driver text from the project store so the panel always reflects the last saved state from the Drivers page.
- [X] Show the empty state "Select a file in the tree to read it." when no file is open; restore it and the pre-open folder state on Close.
- [X] Write Jest tests covering: opening a driver, switching files, the rail and "Show files" toggle, closing, both views, a driver with text, a driver without text, and the absence of any editable control in the panel.

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
