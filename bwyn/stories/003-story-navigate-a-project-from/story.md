# Story: Navigate a project from the menu bar

- **Story**: 003-story-navigate-a-project-from
- **Created**: 2026-10-31
- **Status**: Draft

## Story
As a team member, I want a persistent left bar with a project selector and a page menu, so that after I create a project I land on the Builder page and can move between pages and between projects from one place.

## Acceptance tests

### A. First visit and creating a project

- [ ] **Given** no project exists, **When** the app opens, **Then** the left bar shows the label "Project", a selector reading "No project yet / Create one to get started", the label "Menu", and the eight menu items all disabled and dimmed; the main area shows "Welcome to BWYN", the sentence "Create your first project to start feeding drivers, reviewing decisions and building.", and a "Create project" button.
- [ ] **Given** any state, **When** the user activates "New project" in the header, "New project" in the project dropdown, or "Create project" on the welcome page, **Then** a "Create project" dialog opens containing the text "A project groups the drivers, decisions and integrations of one product.", a Name field with focus (placeholder "e.g. Payments platform"), an optional Description field, and Cancel and "Create project" buttons.
- [ ] **Given** the dialog is open, **When** the user submits an empty name or a name of only whitespace, **Then** the message "Give the project a name." appears beneath the Name field and no project is created.
- [ ] **Given** a project named "Payments" exists, **When** the user submits "payments " (any combination of case and surrounding whitespace), **Then** the message "A project with this name already exists." appears beneath the Name field and no project is created; typing in the field clears the message.
- [ ] **Given** the dialog is open, **When** the user presses Escape, activates the Close (×) button, activates Cancel, or clicks outside the dialog, **Then** the dialog closes and no project is created.
- [ ] **Given** a valid name is entered, **When** the user activates "Create project", **Then** the dialog closes, the new project becomes active, and the main area shows the Builder page with "Builder" selected in the menu.

### B. Left bar with an active project

- [ ] **Given** an active project, **Then** the top of the left bar shows the label "Project" and a selector containing the project's avatar (its first two letters in upper case on a coloured square), its name, and the text "Active project".
- [ ] **Given** an active project, **Then** the menu shows, in order: P&P, Builder, Drivers, Architecture, ADR, Integrations, Plugins, Settings — each with an icon — and all items are enabled.
- [ ] **Given** a menu item is selected, **Then** it has a light-indigo background, a thin indigo bar on its left edge, and `aria-current="page"`.
- [ ] **Given** the bottom of the left bar, **Then** it shows the logo "B", the name "BWYN", and a "Spike" tag.
- [ ] **Given** projects are created one after another, **Then** each new project receives the next colour from a palette of five, cycling back to the first after the fifth.

### C. Switching projects

- [ ] **Given** two or more projects exist, **When** the user activates the project selector, **Then** a listbox (`role="listbox"`) opens below it showing each project's avatar and name, a check mark beside the active project, and a "New project" option at the bottom after a divider.
- [ ] **Given** the listbox is open, **When** the user selects a project, **Then** the listbox closes, that project becomes active, and the currently selected page remains selected but shows that project's content.
- [ ] **Given** the listbox is open, **When** the user presses Escape or clicks outside the listbox, **Then** it closes and the active project is unchanged.
- [ ] **Given** the listbox is open, **When** the user selects "New project", **Then** the listbox closes and the "Create project" dialog opens.

### D. Navigating between pages

- [ ] **Given** an active project, **When** the user activates a menu item, **Then** the main area transitions to that page with a short fade and the activated item becomes selected.
- [ ] **Given** any page is shown, **Then** the header displays a folder icon, the project name, a "/", and the page label — for example "Payments / Drivers"; with no active project it reads "No project / Builder".
- [ ] **Given** any page other than Settings is shown, **Then** the main area begins with a large icon, the page title, and its one-sentence description, as follows:

  | Menu item    | Title                    | Description |
  |---|---|---|
  | P&P          | Principles & Practices   | The principles this team holds to, and the Extreme Programming and evolutionary architecture practices it follows. |
  | Builder      | Builder                  | Turn intent into working software, one guided step at a time. |
  | Drivers      | Drivers                  | The forces that steer decisions. Describe each driver and save it: they feed the skills used by the builder. |
  | Architecture | Architecture             | The architecture of the system: its style, its layers and the rules between them. Stories are checked against it. |
  | ADR          | Architecture decisions   | The decision log of this project. |
  | Integrations | External integrations    | Connect the tools the project already lives in. |
  | Plugins      | Plugins                  | Extend the builder with extra skills, checks and generators. |
  | Settings     | Settings                 | The settings of this project. |

- [ ] **Given** the application starts and at least one project exists, **Then** the Builder page is shown first.

### E. Collapsed rail for the workspace

- [ ] **Given** the Builder page, **When** the user activates "Open the builder", **Then** the left bar shrinks to a narrow icon rail: the active project's avatar at the top (tooltip "&lt;project name&gt; (expand menu)"), the eight menu items as icons with their label as tooltip and accessible name, and an "Expand menu" button at the bottom; the header appends "/ Prompt" after "Builder"; the workspace fills the remaining width.
- [ ] **Given** the Settings page is shown, **Then** the same icon rail appears and the page opens with its own secondary menu beside the main area.
- [ ] **Given** the icon rail is shown, **When** the user activates the project avatar or "Expand menu", **Then** the full left bar returns and a "Collapse menu" button appears beside the logo.
- [ ] **Given** the builder workspace is open, **When** the user activates any page other than Builder, **Then** the workspace closes and that page opens at full width; **When** the user then activates Builder, **Then** the Builder page opens without the workspace.
- [ ] **Given** the builder workspace is open, **When** the user switches to another project, **Then** the workspace shows the new project's files with nothing open.

### F. Theme and persistence

- [ ] **Given** the header, **Then** a Dark / Light toggle (two buttons, the active one in the pressed state) and a "New project" button are visible on its right.
- [ ] **Given** the user activates a theme, **Then** the whole application changes immediately and the choice is remembered across reloads; Dark is the default.
- [ ] **Given** the user reloads the browser, **Then** the projects and the active project are restored and the app opens on the Builder page; if the remembered active project no longer exists, the first project is used instead.
- [ ] **Given** the browser's storage is blocked or contains invalid data, **Then** the app starts in the empty state and no error is shown to the user.

## Business rules and constraints

- A project record holds: a unique id, a trimmed name (unique, case-insensitively), an optional description, a creation timestamp, and a palette index. (Assumption: these are the minimal fields needed for this story.)
- Name uniqueness is checked after trimming and case-folding; the duplicate message clears as soon as the user types.
- The project selector, the menu, and the logo form one left bar; the header and main area are to its right.
- Menu items are disabled until a project is active, because every page belongs to a project.
- The main area is keyed by both project id and page, so content from one project cannot appear under another.
- Creating a project always navigates to the Builder page, regardless of which page was previously shown.
- All interactive elements are keyboard-reachable with a visible focus ring; dialogs and listboxes close on Escape; the project selector carries `aria-haspopup="listbox"` and `aria-expanded`. (ADR-0007, ADR-0015)
- Projects, the active project id, and the theme preference are stored in the browser only; no backend persistence is required for this spike. (Architecture)
- The frontend must not assume a deployment mode; the backend API base URL is configurable and same-origin by default. (Architecture)
- All new frontend files use `.ts` or `.tsx` extensions. (ADR-0015)
- The left bar, project selector, menu items, and page frames are React components in the `ui` layer; state and selection logic belong in the `interface-adapters` layer as ViewModels; the project entity and its uniqueness rule belong in the `entities` layer. (ADR-0007, AE-0002)
- ArchUnitTS fitness functions must continue to pass after this story is merged. (ADR-0016)

## Out of scope

- The content of each page beyond its title, icon, and description (covered by individual page stories).
- The builder workspace and the Settings secondary menu items.
- Renaming, deleting, archiving, or reordering projects.
- Project settings, users, roles, and permissions.
- Persisting the last-visited page across browser sessions.
- A mobile or responsive layout.

## Tasks

- [ ] Define the `Project` entity (id, name, description, createdAt, paletteIndex) and its name-uniqueness rule in the `entities` layer (ADR-0007, AE-0002)
- [ ] Implement a `ProjectViewModel` in the `interface-adapters` layer with load, create, and select operations, backed by browser storage (ADR-0007)
- [ ] Build the "Create project" dialog component with empty-name and duplicate-name validation
- [ ] Build the project selector component with the avatar, the listbox, and keyboard and outside-click handling; add `aria-haspopup="listbox"` and `aria-expanded`
- [ ] Declare the eight menu-item descriptors (id, label, title, description, icon) as a single data structure and render the menu from it, with selected and disabled states
- [ ] Build the left bar, composing the selector, the menu, and the logo strip
- [ ] Build the header with the breadcrumb, the Dark/Light theme toggle, and the "New project" button
- [ ] Build the welcome page and the page frame (large icon, title, description, keyed fade transition by project id and page id)
- [ ] Implement the collapsed icon rail and the rules that expand and collapse it
- [ ] Apply the theme to the document root and persist the preference to browser storage; default to Dark
- [ ] Verify that all ArchUnitTS fitness-function tests still pass (ADR-0016)
- [ ] Verify the full frontend build and test suite completes within the ten-minute budget (Practice: Ten-Minute Build)
