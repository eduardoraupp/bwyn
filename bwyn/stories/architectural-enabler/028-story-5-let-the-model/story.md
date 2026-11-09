# Story: Let the model know the project

- **Story**: 028
- **Created**: 2026-11-08
- **Status**: Draft

## Story
As a team member, I want the AI to know the decisions, the principles, the practices and the work already described in my project, so that the enabler it drafts fits them and does not contradict or repeat them.

## Acceptance tests

**What the model is given (checked on the request a fake backend receives)**

- [X] **Given** a request for a draft, **Then** the first user message has a stable **project context** block, then a cache point, then the **request**.
- [X] **Given** the context block, **Then** it contains, in this order: `# Project context` and `Project: <name>`; `## Architecture decision records (read-only)` with the full Markdown of every ADR of the project, whatever its status, joined by `---`; `## Links between documents` (one line per enabler and story: what it derives from and what it links to, by label); `## Scope of the stories and architectural enablers` (the scope of each enabler, and the statement and the out-of-scope of each story); `## Principles of the team` (name and text of the principles that have text, or `None written yet`); and `## Practices of the team (fixed)` (the practices as text, with the non-negotiable ones marked).
- [X] **Given** the context block, **Then** it does **not** contain drivers (a driver reaches the model only when the user links it), and it does **not** contain the architecture of the system (that goes with stories only).
- [X] **Given** the request part, **Then** it contains: `Document type: architectural enabler`; `Assigned identifier: AE-0003`; `Today's date: <date>`; `## Derives from (ADRs)` (labels); the linked documents in full; `## Existing architectural enablers` (labels, or `- None`); `## Existing stories` (labels, or `- None`); and `## Description` with the user's text. Repeated blank lines are collapsed to one.
- [X] **Given** a linked document that is longer than 12,000 characters, **Then** it is cut at that point and ends with `[...cut: the document is longer]`.
- [X] **Given** the same project and the same links, **When** the user submits two requests, **Then** the context block is byte-for-byte identical, so that the second request reads it from the prompt cache. **Given** a new enabler is approved or an ADR is changed, **Then** the context block changes.
- [X] **Given** the system prompt, **Then** it instructs the model to output one Markdown document with no preamble, no closing remark and no code fence, and it supplies the architectural enabler template (title, Status, Created, Completed, Implements ADR(s), Goal, Scope with In scope and Out of scope, Tasks, Definition of done).

**What the draft is like (checked by reading drafts from a real model; not automated)**

- [ ] **Given** a draft, **Then** it follows the template: title `# AE-NNNN. <short noun phrase>`; metadata lines Status (Proposed), Created (today's date), Completed (empty), and Implements ADR(s) with the number and title of each chosen ADR; and the sections Goal, Scope (In scope / Out of scope), Tasks, and Definition of done.
- [ ] **Given** a draft, **Then** its Goal refers to the decision of the ADR it implements, its Tasks are small steps in the order they are done, and its Definition of done contains statements that a command, a test or an inspection can verify.
- [ ] **Given** a request that conflicts with an Accepted ADR, **Then** the draft leaves that part out and names it under Out of scope.
- [ ] **Given** an existing enabler that already covers part of the request, **Then** the draft does not repeat or take over that part.
- [ ] **Given** a non-negotiable practice, **Then** the draft does not contradict it, and cites it in brackets when a rule comes from it, for example `(Practice: Ten-Minute Build)` or `(ADR 0006)`.
- [ ] **Given** a draft, **Then** it invents no identifiers, numbers, names or technologies; what is missing is stated as the smallest reasonable assumption.

## Business rules and constraints

- The context block is stable: it changes only when an ADR, a link, a scope, a principle or the practices change. Everything that varies per request (identifier, date, chosen ADRs, linked documents, the user's description) comes after the cache point (Principle: Declarative over imperative).
- The same context block is used for every draft and revision request about enablers and stories. It is not sent to the driver, ADR, principle or architecture assistants, which have their own, smaller contexts.
- Drivers are not included in the context block; a driver reaches the model only when the user explicitly links it to the request.
- The architecture of the system is not included in the context block for enablers; it is sent only with story requests.
- Principles with no text written yet are represented by the placeholder `None written yet` rather than being omitted.
- The practices are fixed and sourced from the Practices tab; non-negotiable practices are marked as such in the context block.
- The 12,000-character cut applies per linked document in the request part, not to the context block as a whole.

## Out of scope

- The prompt for stories and the inclusion of the architecture-of-the-system description (a separate story).
- Checking that a draft is correct; the user reviews it (Practice: Pair programming).
- Choosing or tuning the model, and displaying token costs (covered by stories 006 and 008).
- Stop, retry and failure-display behaviour during streaming (covered by story 025).
- Asking for changes to a draft (covered by story 027).

## Tasks

- [X] Define and implement the context-block builder: assemble ADRs (all statuses, joined by `---`), links, scopes, principles and practices in the fixed order, and verify the output is identical across two calls with the same project state
- [X] Define and implement the request-block builder: identifier, date, ADR labels, linked documents (with the 12,000-character cut and the `[...cut: the document is longer]` suffix), enabler list, story list, and description, collapsing repeated blank lines
- [X] Write the system prompt for architectural enablers: output rules (one Markdown document, no preamble, no code fence), the enabler template, and the rules on Accepted ADRs, non-negotiable practices, existing scope, and assumptions
- [X] Wire the context block and the request block into the existing draft endpoint so the cache point separates them in the user message
- [X] Write unit tests covering: exact serialised output for a small project, blank-line collapsing, the 12,000-character cut, context-block stability across two calls, and context-block change after an ADR update or a new enabler is approved
- [ ] Send ten draft requests to the real model with varied prompts and ADR selections; adjust the system prompt until all manual checklist items above pass consistently

## Files of the spike

In `bwyn-frontend/spikes/home-page/`. The spike's builder writes stories and enablers with the same box, so each file below does more than
these stories ask for. The column says which part is the base.

| File | What it holds | Stories |
|---|---|---|
| `Sections.tsx`, `App.tsx` | The Builder page and its "Open the builder" button, and the menu that shrinks when the workspace opens (see `story-builder-folders.md`). | 1 |
| `builder/BuilderWorkspace.tsx` | The frame: tree on the left, file (when open), prompt on the right or in the middle. `submit` (the number, the slug, the planned path, the context and the first turn of a new draft, and `revise` when a draft exists), `approve` (title, the stored enabler, the two messages, `expand('enablers')`, `discard`), `newStory` (the "New" button: confirm, discard, close the file, `sessionStart`, focus), `perform` and `request` (opening a file), `DEFAULT_EXPANDED`. The edit mode, themes, runways, sync and delete are not used here. | 1, 3, 4 |
| `builder/ChatPanel.tsx` | The whole prompt. `Composer` (the box that grows to 200 px, Enter and Shift+Enter, the green ADR panel with its chips and its two messages, the Link button, `canSend`, `sendLabel`, the placeholders for each state, Stop and send buttons); `MessageView` (the prompt as a message, and the assistant's message with the card that opens the file); `DraftView` (path, chip, chips of the links, the text, error, incomplete note, token chips, Discard, Retry, Approve); `ChatPanel` (the empty and the conversation layouts, the banner, the caption, the suggestions, the scroll, the focus). | 1, 2, 3, 4, 6 |
| `builder/useDraft.ts` | The draft: `Draft` and `DraftStart`, the four states, `run` (the call and the stream, abort, the "No text was received" error), `start`, `revise` (only Ready or Stopped), `retry`, `stop`, `discard`, `draftText`. | 1, 2, 4 |
| `builder/prompt.ts` | `SYSTEM_PROMPT` (the rules and the two templates), `buildContext` (ADRs, links, scopes, principles, practices; `linksText`, `scopeText`, `sectionOf`), `buildRequestText`, and the edit text. Its story-only parts (the architecture context and its rule, the tasks of a story) are not used here. | 1, 4, 5 |
| `builder/linkedDocs.ts` | `linkedDocumentsText` (the section, the relations, the 12,000-character cut) and `throughLinkedEnablers` (one step through a linked enabler). | 1, 5, 6 |
| `builder/LinkPicker.tsx` | `RefChip` and `LinkPicker` (the Link button, the popover grouped by kind, `hideKinds`, Escape and outside click, the count). Only an Accepted ADR is offered, in the spike. | 6 |
| `builder/generate.ts` | `makeId`, `today`, `slugify` (first words, accents out, 48 characters, `untitled`), `titleFrom` (first sentence of the prompt, 70 characters), `nextNumber`, `extractTitle` (the first `# ` line without "Story:", "Feature specification:", "AE-n."), `isAcceptedAdr`. | 1, 3 |
| `builder/useBuilder.ts` | The project's store in the browser (`bwyn-spike-home-page.builder`): `enablers`, `messages`, `addEnabler(enabler, messages)`. | 3 |
| `builder/types.ts` | `StoredEnabler` (extends `StoredStory` with `derivesFrom` and `cross`), `Entry`, `ChatMessage`, `Submission`. | 1, 3, 6 |
| `builder/catalog.ts` | `buildEntries`: each enabler is an `Entry` with ref `enabler:<id>`, file name `<number>-<slug>.md`, folder `architectural-enablers`, label `AE-<number>. <title>`, `editable: true` (here: false). `KIND_META` for its icon (`ZapIcon`) and colour. | 3 |
| `builder/FolderTree.tsx` | The folder `architectural-enablers` (key `enablers`) and its file rows; the "New story" button of the header. | 3 |
| `builder/FileView.tsx` | The panel of an open file, with "Derives from", "Linked to" and "Referenced by", the two views and Close. Its Save, Delete, Sync, Cross enabler, task, diagram and UI parts are not used here. | 3, 6 |
| `builder/MarkdownView.tsx` | The Markdown reader and the rendered/Markdown toggle. | 3 |
| `builder/bedrockClient.ts` | `streamDraft` (POST `/api/draft`, reads NDJSON: `text`, `usage`, `done`, `error`), `fetchStatus` (the banner and the caption), `Usage`. | 1, 2 |
| `server/bedrock.js` | The dev server: `POST /api/draft` (the stream, the cache points, 40 turns, temperature 0.3, the errors 400, 503 and 502), `GET /api/draft/status`. `story-improve-with-ai.md` and `story-llm-provider.md` replace it with the backend. | 1, 2, 4 |
| `practices.ts`, `usePrinciples.ts`, `useDrivers.ts`, `useBuilder.ts` (ADRs) | The sources of the context: `practicesText()`, the principles with text, the drivers to link, and the ADRs. | 5, 6 |
| `icons.tsx` | `ZapIcon`, `SparklesIcon`, `ArrowUpIcon`, `StopIcon`, `LinkIcon`, `CheckIcon`, `TrashIcon`, `RotateIcon`, `ArrowRightIcon`. | 1 to 6 |

## What changes from the spike

- **The box writes enablers only.** The spike's box has a tick "Mark as architectural enabler" that switches between a story and an enabler.
  Here there is no tick and `asEnabler` is always true: the title, the line, the placeholder and the examples are the enabler's. The tick
  returns with the stories.
- **Story-only parts are left out:** the story examples ("As a returning customer, ..."), the three suggestions (one is kept), the
  automatic link of cross enablers, the architecture in the context, the tasks that a story copies into its task list, and the edit mode
  with its "Editing ..." chip.
- **The folder `architectural-enablers` lists the enablers, and the panel is read-only.** In the spike an open enabler can be edited,
  linked, marked Cross, synced and deleted. Here none of that is shown, and the note says so.
- **The button of the tree header is "New"**, named "New architectural enabler". The spike's is "New story" and also resets the folders to
  `stories` and `enablers` open. Here it only resets the conversation.
- **The workspace's middle** is the prompt again (in `story-builder-folders.md` it was an empty state), and with a file open the prompt is a
  narrow column on the right of the file.
- **The call to the model** is the backend's, not the dev server's.
