# Story: Ask for changes to the draft

- **Story**: 027
- **Created**: 2026-11-08
- **Status**: Draft

## Story
As a team member, I want to tell the AI what to change in the draft, in my own words, so that I get the enabler I mean before I approve it.

## Acceptance tests
- [ ] **Given** a draft that is Ready or Incomplete, **Then** the composer box shows the placeholder "Ask for changes to the draft…", the send button is labelled "Send changes", the ADR panel and the Link button are hidden, and a line reads "Revising the draft below"
- [ ] **Given** the user types a change request and presses Enter or "Send changes", **Then** the message appears on the right of the conversation, the draft chip reads "Writing…", and the card begins showing the new text in place of the old one as it streams in
- [ ] **Given** the stream ends successfully, **Then** the chip reads "Ready for review" and the card shows the complete revised document, not a diff
- [ ] **Given** a revision request is sent, **Then** the turns delivered to the backend are: the original prompt as the first user turn, the previous draft as the assistant turn, and the change instruction as the next user turn; earlier revision turns are included in order
- [ ] **Given** a revision that failed, **When** the user presses Retry, **Then** the same turns are sent again without alteration
- [ ] **Given** a draft that is Writing or Failed, **Then** the change box is disabled and explains why, consistent with the behaviour established in story 025
- [ ] **Given** a revision completes, **Then** the draft's identifier, slug, planned path, chosen ADRs, and links are unchanged
- [ ] **Given** a draft revised twice and then approved, **Then** the stored document text is the last revision and the stored prompt is the original first prompt
- [ ] **Given** the accumulated turns reach 40, **Then** the backend refuses the next change request and returns a clear error message, which the card displays with a Retry option
- [ ] **Given** the user discards the draft while in revision mode, **Then** the composer returns to the new-enabler prompt state established in story 024
- [ ] **Given** a revision request, **Then** the backend sets a prompt-cache point after the system prompt, after the project context, and after the previous assistant turn; the token chips reflect cache hits as shown in story 025

## Business rules and constraints
- A change instruction is an instruction on the whole draft; the AI returns the complete revised document, preserving the identifier and the creation date, and changes only what was asked (Principle: DRY and KISS)
- The turns sent to the model alternate user and assistant, start and end with a user turn, and are capped at 40 turns total; the backend enforces this limit and returns a descriptive error when it is reached
- The change text is sent exactly as typed; the draft text sent as the assistant turn is the one currently on screen, not an earlier version
- The composer state (placeholder, button label, hidden panels, "Revising" line) is driven by the draft's status value, keeping presentation logic in the ViewModel layer (ADR-0007, Architecture)
- The composer ViewModel and the draft-revision use case reside in the interface-adapter and use-case layers respectively; no routing or AI-provider concern leaks into those layers (ADR-0007, Architecture)
- The backend turn-limit enforcement belongs in the application layer; the inbound REST adapter translates the error into an HTTP response, and the outbound AI-provider adapter is not called (ADR-0017, Architecture)

## Out of scope
- Changing the ADRs or the links of a draft after it has been created (the user must discard and restart)
- Navigating back to an earlier version of the draft or comparing two versions
- Asking for changes to an enabler that has already been approved
- Stop and token-count behaviour beyond what story 025 already establishes
- Connecting an LLM provider and the streaming protocol internals (story 006 and story 007)

## Tasks
- [ ] Update the composer ViewModel to expose a `revisionMode` state that activates when the draft status is Ready or Incomplete, setting the placeholder text, button label, and visibility flags for the ADR panel and Link button
- [ ] Add a `requestRevision(changeText: string)` command to the draft use case that appends a user turn and re-invokes the generation port, preserving the draft's identifier, slug, planned path, ADRs, and links
- [ ] Update the draft card component to replace its displayed text in place as the revised stream arrives, reusing the streaming display established in story 025
- [ ] Implement turn accumulation in the backend application service: build the full turn list (original prompt → previous draft → change → …) and pass it to the AI provider port
- [ ] Add a cache-point instruction to the backend AI provider adapter after the system prompt, after the project context, and after the most recent assistant turn
- [ ] Add a guard in the backend application service that rejects a revision request when the turn count would exceed 40, returning a descriptive error
- [ ] Map the 40-turn error to an HTTP response in the inbound REST adapter and surface it in the draft card with a Retry option
- [ ] Write ViewModel unit tests covering: entering revision mode, a successful single revision, two successive revisions, a failed revision followed by Retry, and discard returning to new-enabler state
- [ ] Write a backend unit test verifying the exact turns sent for a first revision, a second revision, and the 40-turn rejection

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