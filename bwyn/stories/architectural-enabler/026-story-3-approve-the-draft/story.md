# Story: Approve the draft and find the enabler in the tree

- **Story**: 026-story-3-approve-the-draft
- **Created**: 2026-11-08
- **Status**: Draft

## Story
As a team member, I want to approve a ready draft so that it becomes an architectural enabler of my project and appears in the tree, so that stories can later build on it and my team can read it.

## Acceptance tests

**Approving**

- [X] **Given** a Ready draft, **Then** the card shows "Discard" and "Approve and create enabler", and "Approve and create enabler" is enabled. **Given** any other draft state, **Then** "Approve and create enabler" is disabled.
- [X] **Given** a Ready draft, **When** the user presses "Approve and create enabler", **Then** the enabler is stored with: id `0003-<slug>`, number, slug, title, the user's first prompt, today's date, the Markdown as drafted (trimmed, with one final newline), the links, and the ADRs it derives from.
- [X] **Given** the stored Markdown has a first `# ` line, **Then** the title is read from that line with the prefixes "Story:", "Feature specification:", and "AE-`<digits>`." stripped. **Given** the Markdown has no such line, **Then** the title is built from the first sentence of the prompt, capped at 70 characters (cut with "..."), with the first letter in capitals.
- [X] **Given** approval, **Then** two messages are added to the conversation: the user's original prompt, and "Approved. I created architectural enabler AE-0003 in architectural-enablers/0003-`<slug>`.md. It derives from ADR 0006, ADR 0007." with " It is linked to 2 other items." appended when there are two or more links, or " It is linked to 1 other item." for exactly one link; and a card showing the file path and an "Open file" button.
- [X] **Given** approval, **Then** the draft is gone, the prompt box is empty, the ADR chips are cleared, and the `architectural-enablers` folder is open in the tree.
- [X] **Given** the user approves a second draft whose prompt starts with the same words as the first, **Then** its number is `0004`.

**The tree**

- [X] **Given** at least one approved enabler, **Then** the `architectural-enablers` folder lists `0003-<slug>.md` in ascending order of number, each entry has a tooltip showing "AE-0003. `<title>`", and the folder's item count is correct.
- [X] **Given** no approved enabler exists, **Then** the open `architectural-enablers` folder shows the italic line "No architectural enablers yet. Write a prompt." (replacing the previous placeholder "Coming in another story.").
- [X] **Given** the `stories` folder, **Then** it remains inert, unchanged from story 019.

**Reading the enabler**

- [X] **Given** the user presses "Open file" on the approval card, or presses the file entry in the tree, **Then** the enabler opens in the same file panel used for drivers and ADRs: icon, file path, Close button, rendered and Markdown views, the tree as a thin rail, and the prompt as a narrow column on the right.
- [X] **Given** the open enabler panel, **Then** it shows a "Derives from" section with a chip for each ADR, and a "Linked to" section with a chip for each linked item when links exist. **When** the user presses a chip, **Then** that file opens in the panel.
- [X] **Given** the open enabler panel, **Then** it displays a locked read-only notice "Read-only here. Changing an architectural enabler comes in another story." and shows no Save, Delete, Sync, "Add link", or "Cross enabler" control.

**Discarding and keeping**

- [X] **Given** a draft is on screen, **When** the user presses "Discard", **Then** nothing is stored and the draft is removed.
- [X] **Given** a draft is on screen, **When** the user presses "New" in the tree header, **Then** a confirmation "Discard the draft that is waiting for approval?" appears. **When** the user confirms, **Then** the draft is dropped. **When** the user cancels, **Then** nothing changes.
- [X] **Given** the user presses "New", **Then** any open file panel closes, the prompt appears in the centre with an empty conversation (earlier messages are hidden, not deleted), focus moves to the prompt box, and the main menu collapses to its rail.
- [X] **Given** the page is reloaded, **Then** all approved enablers, their links, and the conversation are still present. **Given** two projects, **Then** each project keeps its own enablers and conversation independently.

## Business rules and constraints

- Only a Ready draft may be approved; the AI never stores anything autonomously — approval is always the user's explicit action (Principle: Declarative over imperative).
- An enabler's identity is its file name without `.md`: `0003-<slug>`. References use `enabler:0003-<slug>`. The identity derives from the slug, never from a random id, so that a link means the same on every machine.
- The title is extracted from the document so that editing the document later can update the title without a separate field.
- The `derivesFrom` field holds the ADR references; ADRs are never duplicated in `links`.
- Numbers are assigned sequentially across all approved enablers for the project; the slug is derived from the title.
- All state is kept in `localStorage` per project for this story. Assumption: the final persistence mechanism is decided in a later ADR.

## Out of scope

- Editing an enabler's text by hand or by prompt after approval.
- Deleting an enabler, the "Cross enabler" flag, marking it Done, and setting "Completed".
- Synchronising the enabler with the repository (writes `architectural-enablers/0003-<slug>.md` and `0003-<slug>.meta.json`).
- Creating an enabler without the AI.
- Importing enablers from the repository (they appear in the tree through the sync story).
- The `stories` folder beyond confirming it remains inert.

## Tasks

- [X] Add `addEnabler` to the project store: persist the enabler record (id, number, slug, title, prompt, date, trimmed Markdown, links, derivesFrom) and the two conversation messages together in `localStorage`, keyed by project
- [X] Implement the title-extraction function: read the first `# ` line and strip the prefixes "Story:", "Feature specification:", and "AE-`<digits>`."; fall back to the first sentence of the prompt capped at 70 characters with "..."
- [X] Implement sequential number assignment across existing enablers and slug derivation from the title
- [X] Wire "Approve and create enabler" to call `addEnabler`, clear the draft, clear the prompt box and ADR chips, and open the `architectural-enablers` folder in the tree; keep the button disabled for non-Ready states
- [X] Build the approval confirmation message text (ADR list, link count) and the "Open file" card, and append both messages to the conversation
- [X] Update the `architectural-enablers` folder to list approved enablers in ascending number order with tooltips, and show the italic empty-state line "No architectural enablers yet. Write a prompt." when none exist
- [X] Implement the enabler file panel: icon, path, Close, rendered and Markdown views, "Derives from" chips, "Linked to" chips (when present), chip navigation to other files, and the locked read-only notice with no editing controls
- [X] Wire the "Open file" card and the tree file entry to open the enabler file panel
- [X] Add the "New" button to the tree header: show the confirmation dialog when a draft is present, hide earlier messages without deleting them, clear the prompt box, move focus to it, and collapse the main menu to its rail
- [X] Test: approve stores the correct record; title extraction from Markdown and from the prompt; second enabler gets number `0004`; reload restores enablers and conversation; two projects stay independent; Discard stores nothing; "New" with and without a draft in progress

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
