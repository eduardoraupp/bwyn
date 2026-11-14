# Story: Add an ADR

- **Story**: 015-story-2-add-an-adr
- **Created**: 2026-11-06
- **Status**: Draft

## Story
As a team member, I want to record an architecture decision in my own words and, if I want, ask the AI to fill the ADR template from them, so that the decision becomes an ADR that stories and enablers can use.

## Acceptance tests

**Opening the dialog**

- [X] **Given** ADRs 0001 to 0012 exist, **When** the user presses "Add ADR", **Then** a dialog opens showing the label "ADR 0013" and the heading "New ADR"; the number is the highest existing number plus one, zero-padded to four digits, and when no ADR exists the number is "ADR 0001". (The number is reserved when the dialog opens, from the ADRs loaded on the page; the backend gives the number again when saving, so two people adding at once never share one.)
- [X] **Given** the dialog is open, **Then** it shows: a grey note "Not used by any story or enabler. It stays here, and goes to GitHub the first time together with the story or the enabler that uses it."; the label "Describe the decision"; a Status selector defaulting to Proposed with options Proposed, Accepted, Deprecated, Superseded; a plain-text box of 14 rows with focus and the placeholder "Describe the decision in your own words. The first line is its title. Or let the AI suggest the ADR from it."; an "Improve with AI" button; a Cancel button; and a Save button that is disabled.
- [X] **Given** the dialog is open, **Then** the box holds plain text only; the user never sees Markdown; the title is on the first line and the section names Context, Decision, and Consequences appear as plain lines.

**Saving**

- [X] **Given** the box contains "Use PostgreSQL for persistence" on the first line and further text, and the status is Proposed, **When** the user presses Save, **Then** the ADR is stored with number 0013, title "Use PostgreSQL for persistence", status Proposed, today's date in YYYY-MM-DD format, and the file name `0013-use-postgresql-for-persistence.md` (the title slugified to at most 8 words); the dialog closes; and the new row appears in the ADR list in its correct numerical position. (Number, date and file name are given by the backend and verified there; the dialog and the list are verified in `add-adr.acceptance.test.tsx`.)
- [X] **Given** the stored file for ADR 0013, **Then** its content begins with `# 0013. Use PostgreSQL for persistence`, a blank line, `- **Status**: Proposed`, `- **Date**: <today>`, a blank line, and the remaining text, in which any line that reads exactly "Context", "Decision", or "Consequences" and is surrounded by blank lines is stored as a `##` heading. (Verified in the backend by `AdrDocumentTest`, `CreateAdrCommandHandlerTest` and `AdrPersistenceIT`.)
- [X] **Given** the first line starts with `#` or with `ADR 0013.`, **Then** those prefixes are stripped before the title is stored.
- [X] **Given** the box is empty or the first line is blank after stripping, **When** the user presses Save, **Then** the message "Write the title in the first line, for example: Use PostgreSQL for persistence" appears below the box, the dialog stays open, and nothing is stored; typing any character clears the message.
- [ ] **Given** GitHub is connected and the ADR is saved, **Then** a notice reads "ADR 0013 was saved here. It reaches GitHub the first time together with the story or the enabler that uses it." (Not verified: GitHub cannot be connected yet, so this notice is not built.)
- [X] **Given** GitHub is not connected and the ADR is saved, **Then** no GitHub notice is shown. (No GitHub notice exists yet, so none is shown.)
- [X] **Given** the dialog is open with text typed, **When** the user presses Escape, the Close (X) button, or Cancel, **Then** the dialog closes and nothing is stored, and the number is free again.
- [X] **Given** the dialog is open with text typed, **When** the user clicks outside the dialog, **Then** the dialog stays open.
- [X] **Given** the dialog is open with nothing typed or changed, **When** the user clicks outside the dialog, **Then** the dialog closes.

**Improve with AI**

- [X] **Given** the dialog is open, **When** the user presses "Improve with AI", **Then** the shared panel opens with five quick actions: "Write the ADR", "Improve the context", "Add consequences", "Shorten", "Ask me questions"; and a free-text field with the placeholder "Ask for a change, for example: add the option of a managed service".
- [X] **Given** the panel is open and the user presses Enter in the free-text field, **Then** the instruction is sent and the dialog is not submitted.
- [X] **Given** the quick action "Write the ADR" is pressed and the box contains rough notes, **Then** the instruction asks the model to produce a complete ADR from those notes following the template, preserving the user's intent and facts.
- [X] **Given** the quick action "Write the ADR" is pressed and the box is empty, **Then** the instruction asks the model to produce an ADR from the title and project context, and to ask questions if something important is missing. (The instruction is a function of the box, so the same quick action asks for the right thing in both cases. What the model writes in reply cannot be tested here.)
- [X] **Given** any instruction is sent, **Then** the model receives as stable context the project name and the full text of every existing ADR of the project; and with each instruction it receives the current ADR's number, title (first line), date, and the text as it stands in the box at the moment of sending; no drivers, stories, enablers, or architecture description are sent. (Verified in the backend by `StreamTextImprovementCommandHandlerTest`: the `adrs` context kind, and the date and status in `details`.)
- [X] **Given** the model replies with a proposed ADR, **Then** the panel shows a short comment and a card that renders the proposal as readable text (title large, section headings in bold), never as raw Markdown.
- [X] **Given** a proposed ADR is shown, **When** the user presses "Use this text", **Then** the box is replaced with the proposal as plain text; if the proposal contains a status, the Status selector is updated to that status; nothing is saved; and the button reads "In the editor" and is disabled while the box holds that text.

## Business rules and constraints

- A new ADR exists only once it is saved; the number is reserved when the dialog opens and released if the dialog is closed without saving.
- The number is four digits and is never `0000`. The date is today's date in YYYY-MM-DD format and is written at save time, not before.
- The Markdown structure is built on saving and is never typed by the user (Principle: Declarative over imperative).
- The AI only suggests; it never saves and never changes the number or the date. The user chooses to apply a proposal and then to save.
- The AI follows the template (title, Status, Date, Context, Decision, Consequences), writes the decision as "We will …", keeps the user's facts, invents no numbers, names, or technologies, defaults to Proposed unless the user says otherwise, and stays consistent with the other ADRs of the project. When the user's notes contradict an Accepted ADR, the comment says so (ADR-0017).
- The conversation belongs to the open dialog instance and is discarded when the dialog closes.
- The shared `ImproveWithAI` component receives everything ADR-specific (instructions, context, quick actions, reply-marker, apply-proposal callback) as props or injected dependencies; no ADR-specific logic is written inside the component (Principle: Declarative over imperative; ADR-0007).
- Streaming, Stop, Retry, cut-off replies, and token counts behave as established in story 007 and are not re-specified here.
- The backend `/improve` endpoint established in story 007 is reused without modification (ADR-0008).
- Assumption: the file-name slug is produced by lower-casing the title, replacing spaces and punctuation with hyphens, and keeping at most the first eight words.

## Out of scope

- Editing an ADR and changing its status (separate stories).
- The pull-request flow and GitHub sync.
- The backend `/improve` endpoint and token totals (covered by story 007 and Integrations).
- Linking an ADR to a story or an enabler from the Builder.
- "Superseded by NNNN" links between ADRs.

## Tasks

- [X] Add a `CreateAdrCommand` and its handler in `bwyn-application`, with a repository port in `bwyn-core` that persists the new ADR (ADR-0017). (`CreateAdrCommandHandler`, with the `AdrRepository` port and the pure rules in `AdrDocument`.)
- [X] Add a Flyway migration for the ADR table (number, title, status, date, body, file name) in `bwyn-runtime` (ADR-0010). (Deviation: no new migration. The `adr` table of story 014 (`V10`) already holds all of it.)
- [X] Implement the Spring Data JPA outbound adapter for the ADR repository port in `bwyn-infrastructure` (ADR-0011). (`AdrRepositoryAdapter`: one insert-only statement that never replaces an ADR, so two simultaneous adds get different numbers.)
- [X] Implement the Spring MVC inbound adapter in `bwyn-rest` with a `POST /projects/{projectId}/adrs` endpoint annotated with OpenAPI metadata; the endpoint derives the next number, builds the Markdown, and delegates to the command handler (ADR-0008, ADR-0014). (Deviation: the number, the date and the Markdown are derived in `CreateAdrCommandHandler` and `AdrDocument`, not in the controller, so the adapter holds no rules. The path is `POST /api/projects/{projectId}/adrs`, next to the `GET`.)
- [X] Write a JUnit unit test for `CreateAdrCommandHandler` using a test-double repository port (ADR-0009).
- [X] Write a Testcontainers integration test for the JPA adapter that verifies the ADR is persisted and the next-number query is correct (ADR-0012). (`AdrPersistenceIT`, including two requests at once.)
- [X] Build the `AddAdrDialog` React component in the `ui` layer: number label, grey note, status selector, plain-text box, validation message, Cancel and Save buttons, and the closing rules (click-outside only when unchanged) (ADR-0007).
- [X] Implement the `useAddAdr` hook in the `interface-adapters` layer: next-number derivation, save call, validation, and GitHub-notice flag (ADR-0007). (Deviation: it is part of `AdrListViewModel`, like the add dialog of drivers, and `AddAdrUseCase` does the validation and the call. The GitHub-notice flag is left out, as no GitHub can be connected yet.)
- [X] Wire the `ImproveWithAI` shared component into `AddAdrDialog`, supplying the ADR-specific quick actions, instructions, context builder (project name + all ADRs), reply-marker, and apply-proposal callback that updates the box and the status selector (ADR-0007). (`AdrImproveWithAI`. The context builder is the backend's `adrs` kind. Deviation: the reply marker stays the backend's `---PROPOSAL---`, not `---ADR---`. The shared component gained props only: placeholder, how a proposal is read and drawn, details, and quick actions that see the box.)
  _Tested in story 015 (add-adr.acceptance.test.tsx), which owns this behaviour; the duplicate test here was removed._
- [X] Delete `AdrAssistant.tsx` once the shared component handles the ADR dialog, and add a Jest test that mounts the shared component with the ADR dialog and with one additional screen to confirm no ADR-specific logic leaks into the component (ADR-0005). (`AdrAssistant.tsx` only existed in the spike, so there is nothing to delete here. `shared-improve-with-ai.acceptance.test.tsx` mounts the shared component with the ADR assistant and with another screen, and `layer-rules.test.ts` keeps ADR words out of the shared files.)
- [ ] Verify the full build and test suite passes within the ten-minute budget (Practice: Ten-Minute Build). (Not met yet: the Java modules build and pass in about 5 minutes, but the frontend module takes about 6 minutes and its `npm test` fails on two architecture rules (`layer-rules.test.ts`, "Where the drivers live") that need forward-slash paths and fail when node runs on this Windows drive.)

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
