# Story: Describe an enabler and choose the ADRs it implements

- **Story**: 024-story-1-describe-an-enabler
- **Created**: 2026-11-08
- **Status**: Draft

## Story
As a team member, I want to describe in the prompt the technical work that stories need first and choose the Accepted ADRs it implements, so that the AI drafts an architectural enabler that is tied to the decisions we already made.

## Acceptance tests

**The prompt**
- [ ] **Given** the workspace with no file open, **Then** the middle shows the prompt: a sparkles icon, the title "Which architectural enabler shall we build?", the line "Describe the technical work that stories need first. Claude drafts the enabler, and once you approve it BWYN creates its file.", a textarea with the placeholder "Describe the architectural enabler..." and the focus in it, and a round send button.
- [ ] **Given** the prompt, **Then** below the textarea there is one example chip the user can press, "Set up continuous integration for the frontend build.", which puts that text in the textarea and keeps the focus there.
- [ ] **Given** the textarea, **When** the user types more than one line, **Then** it grows with the text up to 200 px and then scrolls.
- [ ] **Given** a file is open in the tree, **Then** the prompt is a narrower column on the right of the file (about 26 rem) with the same textarea; **Given** no file is open, **Then** it is in the middle.
- [ ] **Given** the workspace opens, or the user presses "New", **Then** the focus goes to the textarea.
- [ ] **Given** the textarea is not focused, **Then** it has a green border; **Given** it has the focus, **Then** the border is indigo.

**The ADRs**
- [ ] **Given** the prompt, **Then** a green panel inside the box says "Derives from an Accepted ADR (required)" and lists one chip per Accepted ADR of the project, named `ADR 0011`, with the full label (e.g. "ADR 0011. Use TypeScript for the frontend") as its tooltip.
- [ ] **Given** the chips, **When** the user presses one, **Then** it turns selected (`aria-pressed`, solid green chip with dark text); **When** the user presses it again, **Then** it is deselected; several chips can be selected simultaneously.
- [ ] **Given** ADRs whose status is Proposed, Deprecated, or Superseded, **Then** they are not offered in the panel.
- [ ] **Given** no ADR in the project has status Accepted, **Then** the panel says "There are no Accepted ADRs yet. Only an Accepted ADR can be linked. Accept one on the ADRs page, or sync a story to bring in the ones in the repository." and the send button is disabled.
- [ ] **Given** the textarea has text and no ADR chip is selected, **Then** the panel says "Select at least one ADR to derive the enabler from." and the send button is disabled.

**Sending**
- [ ] **Given** text (not only spaces) and at least one selected ADR chip, **Then** the send button is enabled and labelled "Draft architectural enabler"; **Given** either condition is not met, **Then** the button is disabled.
- [ ] **Given** the textarea has the focus, **When** the user presses Enter, **Then** the prompt is sent; **When** the user presses Shift+Enter, **Then** a new line is added; **Given** an input method is composing text, **Then** Enter does not send.
- [ ] **Given** no model is connected for the project, **Then** a banner above the textarea says "No model is connected for this project. Connect one in Integrations, then the prompt works. No restart is needed."; **When** the user sends, **Then** the draft fails with the message "No model is connected for this project. Connect one in Integrations."
- [ ] **Given** the prompt, **Then** a caption beneath it names the model and its provider, for example "Claude Sonnet 4.6 on Amazon Bedrock (us-east-1) · prompt cache 5m", or "Claude on Amazon Bedrock" while the status is not yet known.
- [ ] **Given** the user sends, **Then** the textarea, the selected ADR chips, and the example link are cleared; the prompt appears as a message on the right (text as typed, line breaks kept); and a draft card appears beneath it.
- [ ] **Given** the draft card, **Then** it shows the enabler icon, the planned path `architectural-enablers/0003-<slug>.md` (complete path as a tooltip), the chip "Writing...", the chips of the ADRs it derives from, and the text as it streams in.

**The number and the slug**
- [ ] **Given** enablers 0001 and 0002 exist, **Then** the planned number is `0003` (the highest existing number plus one, zero-padded to four digits; `0001` when none exist yet); the identifier sent to the model is `AE-0003`.
- [ ] **Given** the prompt text "Set up PostgreSQL and Flyway so that stories can use a database", **Then** the slug is `set-up-postgresql-and-flyway-so`: the first five words, lower-cased, accents removed, only characters `a-z` and `0-9` joined by `-`, at most 48 characters total, and `untitled` when nothing remains after normalisation.
- [ ] **Given** two enablers whose prompt texts start with the same five words, **Then** their paths differ because their numbers differ.

**The call**
- [ ] **Given** the user sends, **Then** the browser calls only the BWYN backend draft endpoint and never calls the model provider directly; no API key is present in the browser.
- [ ] **Given** the request body, **Then** it contains: `"Document type: architectural enabler"`, `"Assigned identifier: AE-0003"`, today's date, the full text of each selected ADR (identified by label), and the user's description under `"Description"`; it does not yet include the project context or the lists of enablers and stories (added in story 5).
- [ ] **Given** a new send while a previous draft is still streaming, **Then** the in-flight request is cancelled before the new one is started.

## Business rules and constraints
- An enabler must derive from at least one Accepted ADR; only ADRs with status Accepted are offered (ADR-0007, ADR-0017).
- The AI only drafts; nothing is persisted until the user approves (out of scope here, covered by story 3).
- The prompt text and the selected ADRs belong to the open workspace session and are not retained if the workspace is closed (Principle: KISS).
- The enabler number is the highest existing enabler number plus one, zero-padded to four digits. The slug is derived from the user's text. Together they form the permanent identity of the enabler and never change after creation.
- The model is asked to return one Markdown document with no preamble, no closing remark, and no code fence.
- Model API keys stay on the backend; each project has its own provider connection (ADR-0008, story 006).
- The frontend must not assume a deployment mode; the backend API base URL is configurable and same-origin by default in production (Architecture).
- The prompt and draft card reside in the framework/UI layer; the use case that builds the request and calls the backend resides in the use-case layer (ADR-0007, AE-0002).

## Out of scope
- Stop, Retry, failure display, and token counts for the draft in progress (story 2 of the builder).
- Approving the draft and writing the file (story 3 of the builder).
- Asking for changes to a draft (story 4 of the builder).
- Sending the project context, the list of existing enablers, and the list of existing stories with the request (story 5 of the builder).
- Linking an enabler to stories or other documents (story 6 of the builder).
- Writing stories from the same prompt box; the "Mark as architectural enabler" toggle is not shown here.
- Sending the architecture of the system description with the request (that applies to stories only).

## Tasks
- [ ] Add a `GetAcceptedAdrsUseCase` in the use-case layer that returns the Accepted ADRs for the current project, sourced from the existing ADR store (AE-0002, ADR-0007).
- [ ] Build the `EnablerPromptViewModel` in the interface-adapter layer: holds textarea text, selected ADR ids, send-enabled rule, next number, and slug derivation logic; bind it in the composition root (AE-0002, ADR-0006).
- [ ] Render the enabler prompt panel in the UI layer: sparkles icon, title, description line, growing textarea (max 200 px), green/indigo border, example chip, and focus-on-open behaviour (ADR-0007, AE-0002).
- [ ] Render the ADR chips panel inside the prompt box: fetch Accepted ADRs via the ViewModel, toggle selection with `aria-pressed`, and show the two validation messages (ADR-0007).
- [ ] Implement send-on-Enter (not Shift+Enter, not while composing) and send-on-button-click; clear the composer and append the user message to the conversation on send.
- [ ] Implement next-number and slug derivation (highest existing number + 1, zero-padded; first five normalised words, max 48 chars, fallback `untitled`) and expose them from the ViewModel.
- [ ] Write the system prompt for architectural enablers and assemble the request body (document type, identifier, date, selected ADR labels and full texts, description).
- [ ] Call the backend draft endpoint (reusing the streaming client from story 007), cancel any in-flight request before starting a new one, and stream the response text into the draft card.
- [ ] Render the draft card: enabler icon, planned path with full-path tooltip, "Writing..." chip, ADR-derives-from chips, and streaming text.
- [ ] Render the model-connection banner and caption from the project's provider status; surface the "no model connected" error message on send failure.
- [ ] Position the prompt panel in the middle when no file is open and in a ~26 rem right column when a file is open (ADR-0007, AE-0002).
- [ ] Write Jest unit tests for the ViewModel: send-enabled rule, number and slug derivation, and the request body that is assembled (AE-0002, ADR-0005).

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