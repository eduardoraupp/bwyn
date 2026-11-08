# Story: Follow the draft as it is written

- **Story**: 025
- **Created**: 2026-11-08
- **Status**: Draft

## Story
As a team member, I want to see the state of the draft and to stop it, retry it or discard it, so that I never wait without knowing what is happening and never approve a text that is cut off.

## Acceptance tests

**State**
- [X] **Given** the draft is being written, **Then** its chip reads "Writing..." (pulsing), the card shows the text as it arrives, and shows "Waiting for the first tokens..." until the first text arrives.
- [X] **Given** the draft ended normally, **Then** its chip reads "Ready for review" (green).
- [X] **Given** the model stopped because it ran out of tokens, **Then** the chip reads "Incomplete" (amber) and the card says "The draft is incomplete, so it cannot be approved. Ask for changes, or discard it."
- [X] **Given** the request failed, **Then** the chip reads "Failed" (red) and the card shows the error under the text.
- [X] **Given** the model answered with no text, **Then** the chip reads "Failed" and the error is "No text was received from the model."

**Stop, Retry, Discard**
- [X] **Given** the draft is being written, **Then** the round send button is replaced by a white Stop button ("Stop writing"), and the card has a Stop button. **When** the user presses either, **Then** the request is cancelled, the text received so far stays, and the chip reads "Incomplete".
- [X] **Given** a Stop before any text arrived, **Then** the chip reads "Failed" with "No text was received from the model."
- [X] **Given** the draft failed, **Then** the card has "Retry" and "Discard". **When** the user presses Retry, **Then** the same turns are sent again, without adding a message.
- [X] **Given** a draft that is not being written, **Then** Discard shows. **When** the user presses it, **Then** the draft disappears and nothing is stored.
- [X] **Given** the draft is being written, **Then** there is no Discard (the user stops it first).

**The composer while there is a draft**
- [X] **Given** the draft is being written, **Then** the box says "Waiting for the draft..." and sending is blocked.
- [X] **Given** the draft failed, **Then** the box says "Retry or discard the draft first." and sending is blocked.
- [ ] **Given** a draft that is Ready or Incomplete, **Then** the box asks for changes (story 4).

**Tokens**
- [X] **Given** the draft ended (or was stopped after the model reported them), **Then** the card lists "Input 5230", "Output 812", "Cache read 4102" and "Cache write 0". **Given** the cache read is more than 0, **Then** that chip is green; the tooltips say "Input tokens read from the prompt cache" and "Input tokens written to the prompt cache".
- [X] **Given** the draft failed before the model answered, **Then** there are no token counts.

**The call**
- [X] **Given** the backend answers before the stream starts with a 400, 503 or 502, **Then** the error shows in the card as a normal message (for 503: the project has no connection to the model).
- [X] **Given** the stream fails after it started, **Then** the chip reads "Failed" and the card shows the error. The text received so far is not kept, and Retry writes the draft again from the start.
- [X] **Given** the user closes the workspace, leaves the page or sends something new, **Then** the call in flight is cancelled and the backend stops asking the model.

## Business rules and constraints
- A draft has exactly four states: *streaming*, *ready*, *stopped* (cut by Stop or by the token length limit), or *error*. Only a draft in the *ready* state can be approved (ADR-0007, Architecture).
- A *stopped* draft can be revised (story 4) or discarded; an *error* draft can be retried or discarded.
- A draft lives on screen only. It is not persisted and is lost on page reload (Practice: Incremental design).
- The reply from the backend is a stream of text chunks, followed by usage figures, followed by an end signal, or an error — consistent with the protocol established in story 007.
- Default maximum answer length is 4096 tokens; temperature is 0.3. These values are fixed for this story; making them configurable is out of scope.
- The draft state object holds: state, accumulated text, the turns that produced it, any error message, and usage figures. It is owned by the ViewModel in the interface-adapter layer (ADR-0007, Architecture).
- Retry re-sends the identical turns with no new user message appended.
- Cancellation of an in-flight HTTP request must propagate to the backend so the backend stops forwarding tokens to the model provider (Architecture).

## Out of scope
- Approving a ready draft (story 3 of the builder).
- Asking for changes to a draft (story 4 of the builder).
- A history of drafts and persisting a draft across page reloads.
- Connecting an LLM provider and the `/improve` endpoint internals (story 006 and story 007).
- Displaying project-level token totals in Settings > Costs (story 008).
- The prompt box, "New story" toggle, and any builder interaction beyond the draft card and composer state described here.

## Tasks
- [X] Define a `DraftState` union type (`streaming | ready | stopped | error`) and a `Draft` value object holding state, text, turns, error, and usage in the entities or use-cases layer (ADR-0007, Architecture)
- [X] Implement a `DraftViewModel` in the interface-adapter layer that owns the `Draft` object, exposes observable state, and drives transitions between the four states (ADR-0007, Architecture)
- [X] Wire `DraftViewModel` in the Inversify composition root (ADR-0006)
- [X] Implement the streaming use case: open the SSE/chunked response, append text tokens to the draft, record usage on the end signal, and transition to *ready*, *stopped*, or *error* accordingly
- [X] Implement cancellation: abort the in-flight request when Stop is pressed, when a new request starts, or when the component unmounts; verify the backend receives the cancellation and stops the model call
- [X] Build the draft card React component: "Waiting for the first tokens..." placeholder, streaming text, status chip (pulsing / green / amber / red), incomplete note, error message, Stop button, Retry button, Discard button (ADR-0005, Architecture)
- [X] Build the token chip row: Input, Output, Cache read (green when > 0 with tooltip), Cache write chips; hide the row when usage is absent
- [X] Update the composer React component: block sending and show the correct placeholder text for each draft state; replace the send button with Stop while streaming
- [X] Handle pre-stream HTTP errors (400, 502, 503) by displaying the error message in the card without entering the streaming state
- [X] Write Jest unit tests for `DraftViewModel` state transitions using a fake streaming backend (streams normally, ends at token limit, returns no text, fails before first token, fails after first token) (ADR-0005)
- [X] Write Jest unit tests for the token chip row rendering with and without usage data
- [X] Verify the full frontend build and test suite completes within the ten-minute budget (Practice: Ten-Minute Build)

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