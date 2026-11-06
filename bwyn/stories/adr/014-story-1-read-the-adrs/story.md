# Story: Read the ADRs of my project on one screen

- **Story**: 014-story-1-read-the-adrs
- **Created**: 2026-11-06
- **Status**: Draft

## Story
As a team member, I want to open "ADR" in the menu and see all the architecture decisions of my project in one list with their status, so that I know what was decided, what is still proposed, and what is no longer valid.

## Acceptance tests

**The page**
- [X] **Given** a project, **When** the user chooses "ADR" in the menu, **Then** the item becomes selected (`aria-current="page"`), the header reads "&lt;project&gt; / ADR", and the page is titled "Architecture decisions" with the description "The decision log of this project."
- [X] **Given** the page, **Then** a card shows the sentence "The decisions of this project. An ADR goes to GitHub the first time with the story or the enabler that uses it. After that, saving an edit of a used ADR opens a pull request.", an "Add ADR" button on its right, and the list below it.
- [X] **Given** the page, **Then** it renders correctly with no LLM provider connected and no repository connected.

**No ADRs**
- [X] **Given** a project with no ADRs, **Then** the list shows an icon, the text "No ADRs yet", and the hint "Record the first architecture decision, or connect GitHub in Integrations and sync a story to bring in the ones that are in the repository."

**The rows**
- [X] **Given** a project with ADRs, **Then** each ADR is a row showing its number (4 digits, monospace), its title on one line truncated with "…" when long, its date, its status chip, and a pencil icon; rows are ordered by number ascending.
- [X] **Given** an ADR that at least one story or enabler links to or derives from, **Then** its row shows a chip with the count of those stories and enablers and the tooltip "Used by a story or an enabler"; an ADR that nothing uses shows no chip. (The chip logic is tested with counts given by the test backend. The real backend sends none yet, because nothing can link to an ADR.)
- [X] **Given** a status chip, **Then** "Accepted" is green, "Proposed" is amber, "Deprecated" and "Superseded" are grey.
- [X] **Given** an ADR whose status is not one of the four recognised values (for example an ADR imported from the repository with no Status line), **Then** its chip is neutral and shows "Unknown".
- [X] **Given** a row, **Then** it is a single button with the accessible name "Edit ADR 0004, &lt;title&gt;"; **When** the user activates it with a click, Enter, or Space, **Then** the edit dialog opens (story 3, out of scope here).
- [X] **Given** the list, **Then** no row, dialog, or menu offers a way to remove an ADR: no trash button and no "Remove" option.

**Persistence and isolation**
- [X] **Given** ADRs saved to a project, **When** the page is reloaded, **Then** the same ADRs appear in the same order.
- [X] **Given** two projects each with their own ADRs, **Then** each project's ADR screen shows only its own ADRs.
- [ ] **Given** a repository containing ADR files, **When** the project syncs, **Then** files whose names start with a four-digit number appear in the list; the template file `0000` and files that do not start with a number are not imported as ADRs. (Not verified: syncing with a repository is a separate story and does not exist yet. The list reads whatever the project keeps, so ADRs that sync brings in will show.)

## Business rules and constraints
- ADRs are ordered by number only; there is no filter, search, or alternative ordering.
- The chip on a row shows the count of stories and enablers that use the ADR; it never names them (the edit dialog does, story 3).
- An ADR cannot be removed from BWYN; a decision that is no longer valid is kept and marked Deprecated or Superseded (story 4).
- The four recognised status values are: Accepted, Proposed, Deprecated, Superseded. Any other value, including absence of a Status line, is displayed as "Unknown" with a neutral chip.
- The "Add ADR" button is rendered on this screen but its action is out of scope (story 2); it may be rendered as disabled or inert.

## Out of scope
- Adding an ADR (story 2), editing an ADR and changing its status (story 3), and marking an ADR Deprecated or Superseded (story 4).
- Syncing ADRs with the GitHub repository (a separate story).
- Linking an ADR to a story from the Builder.
- Filtering, searching, or grouping ADRs by status.
- Showing the ADR's full text in the list.
- The edit dialog content (story 3).

## Tasks
- [X] Add an "ADR" entry to the left-hand menu with its icon, route, title "Architecture decisions", and description "The decision log of this project." (ADR-0020, Architecture: ui layer) (The entry was already in the menu; this story fills the page it opens.)
- [X] Define the `Adr` entity (number, title, status, date, file name, Markdown body) in the entities layer and export it from the layer barrel (ADR-0007, Architecture: entities layer)
- [ ] Implement a `ListAdrsUseCase` interface in the use-cases layer that returns the ADRs of a project ordered by number, and a `GetAdrUsageCountUseCase` interface that returns the count of stories and enablers linked to each ADR (ADR-0007, Architecture: use-cases layer) (`ListAdrsUseCase` is done. `GetAdrUsageCountUseCase` is deferred: nothing can link to an ADR yet, so there is nothing to count.)
- [X] Implement a `AdrRepository` port in the use-cases layer and a concrete in-memory implementation in the interface-adapters layer that persists ADRs per project (ADR-0007, Architecture: interface-adapters layer) (Deviation: the port is `AdrGateway` in the use-cases layer, with an HTTP adapter; the ADRs are kept by the backend, not in memory. The in-memory version is a test double.)
- [X] Implement an `AdrListViewModel` in the interface-adapters layer that exposes the ordered ADR list, the usage counts, and the empty-state flag, injected via Inversify (ADR-0006, ADR-0007) (The usage count is `usedBy` on each row; it is 0 until something can link to an ADR.)
- [X] Add the backend `GET /api/projects/{projectId}/adrs` query endpoint in `bwyn-rest` returning ADR number, title, status, date, and usage count; annotate with OpenAPI `@Operation` and `@ApiResponse` (ADR-0014, ADR-0017, Architecture: inbound adapter) (Deviation: no usage count is returned, because nothing can link to an ADR yet. The list never carries the text of an ADR.)
- [X] Add the `AdrQueryRepository` outbound port in `bwyn-core` and its Spring Data JPA implementation in `bwyn-infrastructure`; add a Flyway migration for the `adr` table (ADR-0010, ADR-0011, ADR-0017, Architecture: outbound adapter) (`V10__create_adr_table.sql`.)
- [X] Build the `AdrListPage` React component: page header, info card with "Add ADR" button placeholder, list of rows, and empty state (ADR-0007, Architecture: ui layer) (Built as `AdrPage`. "Add ADR" is disabled until story 015.)
- [X] Build the `AdrRow` React component: monospace 4-digit number, truncated title, date, coloured status chip, usage chip (conditional), and accessible button with name "Edit ADR &lt;number&gt;, &lt;title&gt;" (ADR-0007, Architecture: ui layer)
- [X] Wire the `AdrListViewModel` into the Inversify composition root and connect it to `AdrListPage` (ADR-0006)
- [X] Write Jest unit tests for `ListAdrsUseCase`, `AdrListViewModel`, `AdrRow`, and `AdrListPage` covering ordering, status colours, usage chip visibility, empty state, and the no-removal invariant (ADR-0005) (`ListAdrsUseCase` is tested through the entity and view-model tests; the row and the page through `adrs.acceptance.test.tsx`.)
- [X] Write a `@DataJpaTest` integration test for `AdrQueryRepository` using the Testcontainers base class (ADR-0012) (Done as `AdrPersistenceIT`, a full-context test on the Testcontainers base class, like the other persistence tests.)
- [ ] Verify the full build and test suite completes within the ten-minute budget (Practice: Ten-Minute Build) (Not met yet: the Java modules build and pass in about 6 minutes, but the frontend module takes about 6 minutes and its `npm test` fails on two architecture rules (`layer-rules.test.ts`, "Where the drivers live") that need forward-slash paths and fail when node runs on this Windows drive.)


Files of the spike

In `bwyn-frontend/spikes/home-page/`:

| File | What it holds | Stories |
|---|---|---|
| `data.ts` | The menu entry `adr` (label "ADR", title "Architecture decisions", `BookOpenIcon`). Its description in the spike is stale (see below). | 1 |
| `Sections.tsx` | `case 'adr'` mounts `AdrPage`. | 1 |
| `AdrPage.tsx` | The whole screen. `AdrPage`: the list, the empty state, the card, `usedBy` (stories and enablers that use an ADR), the notices (`Notice`, `NOTICE_STYLE`), `publish` (what to say after a save, and the pull request), and the opening of the dialogs (`Dialog`: new or edit). `EditAdrDialog`: the one dialog for add and edit (plain text, status, headings, `dirty`, the used-by note, `save` and the status-only shortcut, `replaceWith` for the AI). `Shell`: the modal frame and its closing rules. `RemoveAdrDialog` and `deleteAdr`: the removal that these stories take out. | 1, 2, 3, 4 |
| `adrText.ts` | `viewOf` (Markdown to plain text, status, date and the marks of the sections) and `composeAdr` (plain text back to Markdown, null without a title). | 2, 3 |
| `builder/generate.ts` | `nextNumber`, `slugify`, `parseAdr` (reads a file: number, title, status, date), `adrFromMarkdown` (new ADR, file name from number and title), `withAdrStatus` (rewrites only the Status line), `ADR_STATUSES`, `isAcceptedAdr`. | 2, 3, 4 |
| `builder/useBuilder.ts` | The ADRs of a project in the browser: `addAdr` (sorted by number), `saveAdr` (reads title, status and date again; keeps the file name), `removeAdr` and `removedAdrs`. | 1, 2, 3, 4 |
| `builder/types.ts` | `StoredAdr` and `ProjectBuilder.adrs` / `removedAdrs`. | 1 |
| `AdrAssistant.tsx` | The AI of the ADR dialog, **built apart from the shared component**: the "Generate from template with AI" button, the "Suggested ADR" card (`Suggestion`, readable text through `viewOf`), "Replace my text with this", Stop, Retry, tokens. It is what the task of story 2 replaces. | 2 |
| `adrPrompts.ts` | `ADR_SYSTEM_PROMPT` (template and rules), `ADR_REPLY_MARKER` (`---ADR---`), `buildAdrContext` (project name and the other ADRs), `buildAdrTurn` (number, title, date, text, instruction) and `ADR_QUICK_ACTIONS` (four). | 2, 3 |
| `DriverAssistant.tsx`, `useDriverAssistant.ts`, `driverPrompts.ts` | **The shared "Improve with AI" component**, its conversation hook and `parseReply`. Used today by drivers, principles and the architecture. See the first task of story 2 for what it lacks for ADRs. | 2, 3 |
| `builder/bedrockClient.ts`, `server/bedrock.js` | Today's call to the model, through the dev server. `story-improve-with-ai.md` replaces it with the backend REST API. | 2, 3 |
| `builder/git/publishAdr.ts` | Sends one saved ADR as a pull request, compared with the base branch and with what was last synchronised: `pr`, `waiting`, `conflict` or `unchanged`. | 3, 4 |
| `builder/git/useGitState.ts` | The connection and the record of what was synchronised, used by the page for `publishAdr`. | 3, 4 |
| `builder/git/deleteFromRepo.ts` | Deletes an ADR file in a pull request (`kind: 'adr'`). **Removed by these stories** for ADRs. | 4 |
| `builder/catalog.ts` | Makes each ADR a read-only `adr:<number>` entry (`adrs/<file>`) for the Builder. | 4 |
| `builder/LinkPicker.tsx`, `builder/ChatPanel.tsx` | Offer only Accepted ADRs (`isAcceptedAdr`); one already linked stays visible. | 4 |
| `builder/repoFiles.ts`, `builder/git/scope.ts`, `builder/git/syncEngine.ts` | Reading `adrs/*.md` back (`parsePulled`, `mergePulled`), the scope `adrs/<number>-`, and "an ADR goes up only when something links to it". `removedAdrs` is used here too. | 1, 3, 4 |
| `icons.tsx` | `BookOpenIcon`, `PencilIcon`, `PlusIcon`, `GitBranchIcon`, `CheckIcon`, `XIcon`. | 1 to 4 |

## Removal in the spike

The spike lets the user remove an ADR that nothing uses (trash button, `RemoveAdrDialog`, `deleteAdr`, `removeAdr`), by a pull request that
deletes the file, and remembers the number in `removedAdrs` so that it is not reused and the file is not brought back. These stories say an
ADR is never removed, so the following goes away: the trash button, `RemoveAdrDialog` and `deleteAdr` in `AdrPage.tsx`; `removeAdr` in
`useBuilder.ts`; the `'adr'` kind of `deleteFromRepo.ts`; and `removedAdrs`, which `AdrPage`, `ArchitecturePage`, `Drivers`,
`PrinciplesPractices`, `BuilderWorkspace`, `useGitSync`, `syncEngine` and `mergePulled` carry only to keep a removed ADR out.

## Spike details worth keeping in mind

- The ADR menu description in `data.ts` still says "Decisions are recorded elsewhere, so this view is read-only.", which is false. The
  stories use "The decision log of this project." from `story-navigation-and-menu-bar.md`.
- The AI today is `AdrAssistant`, not the shared component, and differs from it: a button "Generate from template with AI" (two
  instructions, with and without text), a "Suggested ADR" card as readable text, and "Replace my text with this". The shared component
  calls `parseReply(text)` with the driver's marker, builds its turn with `buildDriverTurn(name, hint, ...)`, shows the proposal as raw text
  and compares it to the box as raw text, and has the label "Ask Claude to change this driver". That is why story 2 has a task to give it
  what an ADR needs before the dialog can use it.
- `buildAdrTurn` does not send the status, and the box holds plain text without the Status line, so the model cannot tell that an ADR is
  Accepted. The rule in the system prompt ("An ADR is immutable once Accepted ...") has nothing to act on.
- "Use this text" can change the status selector: `replaceWith` takes the status of the proposal when it has one.
- A new ADR takes today's date from `new Date().toISOString()`, which is the UTC date, not the local one.
- Escape closes the dialog even with unsaved text, while a click outside closes it only when nothing changed. The tests follow the spike.
- `saveAdr` reads the ADR again with the stored file name, so the number and the file name never follow an edited title.
- When only the status changed, `EditAdrDialog` keeps the stored Markdown and rewrites just the Status line (`withAdrStatus`). Any text
  change rebuilds the whole Markdown from the plain text (`composeAdr`).
- Everything is in the browser (`localStorage`, the key of the builder store). Two browsers do not share ADRs until the repository does.
