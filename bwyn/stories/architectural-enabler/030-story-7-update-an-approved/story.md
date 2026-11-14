# Story: Update an approved enabler with a prompt

- **Story**: 030
- **Created**: 2026-11-09
- **Status**: Draft

## Story
As a team member, I want to open an architectural enabler and describe a change in the prompt, so that the AI updates that enabler from its current text, its ADRs and its links, and I do not end up with a second enabler for what is the same work.

## Acceptance tests

**The prompt follows the opened file**

- [X] **Given** an approved enabler is open, **When** the user opens it from the tree, from a draft card, or from a "Derives from" or "Linked to" chip, **Then** the prompt column shows a chip with a pencil and the text "Editing architectural-enablers/0003-\<slug\>.md" in the enabler colour, the ADR panel and the Link button are hidden, the placeholder reads "Describe the change to make to this enabler...", and the send button reads "Draft changes to this enabler".
- [X] **Given** the enabler is open and no conversation exists yet, **Then** the column shows the heading "What should change?" and the description "Describe the change. Claude drafts the updated file, and it replaces the current text once you approve it.", with no examples.
- [X] **Given** the text box contains text, **Then** the send button is enabled without requiring an ADR to be selected.
- [X] **Given** the user closes the file or opens a driver or an ADR, **Then** the prompt reverts to the new-enabler state: the ADR panel and the Link button reappear and the editing chip is gone.
- [X] **Given** the user had typed text before opening the enabler, **Then** that text remains in the box after the prompt switches to editing mode.
- [X] **Given** a draft is already on screen when the user opens another file, **Then** the prompt continues to work on that draft and does not switch target.
- [X] **Given** the file panel of the open enabler, **Then** the read-only note reads "Read-only here. Describe the change in the prompt to update this enabler.", and no Save, Delete, Sync, "Add link", "Cross enabler" control, or text box is present.
  _Tested in story 040 (link-from-enabler-panel.acceptance.test.tsx), which owns this behaviour; the duplicate test here was removed._

**Sending the change**

- [X] **Given** the user submits a change, **Then** the box is cleared, the text appears as a message on the right, and a draft card appears showing the enabler's own path `architectural-enablers/0003-\<slug\>.md` and the chip "Writing..."; no new number is assigned and the card shows no ADR or link chips.
- [X] **Given** the request is built, **Then** it carries the full project context from story 028, unchanged, so it is served from the prompt cache as it is for a new enabler.
- [X] **Given** the request is built, **Then** it contains "Document type: architectural enabler", "Task: edit an existing document", "File: architectural-enablers/0003-\<slug\>.md", and today's date, and it contains no "Assigned identifier" field.
- [X] **Given** the request is built, **Then** it contains "## Derives from (ADRs)" listing the ADRs of the opened enabler, and "## Linked documents" containing the documents the enabler links to in full, plus the documents one step further through its linked enablers (as story 029 builds them, or "- None"), but not the opened enabler itself even if a linked document references it back; the full text of the ADRs is in the project context of story 028 and is not repeated.
- [X] **Given** the request is built, **Then** it contains "## Existing architectural enablers" (all enablers, including the opened one) and "## Existing stories", as in story 028.
- [X] **Given** the request is built, **Then** it ends with "## Current document" containing the stored text of the enabler (trimmed) followed by "## Requested change" containing the user's text.
- [X] **Given** the enabler was updated previously, **Then** the next request's "## Current document" contains the updated text, not the original.
- [X] **Given** the system prompt, **Then** it instructs the model that when the task is to edit an existing document, the current document is given and the model returns the complete document with only the requested change applied, keeping the identifier, the creation date, and everything the change does not touch.

**Following, revising and discarding the draft**

- [X] **Given** an update draft is in progress, **Then** all behaviour from story 025 applies: the chip, Stop, Retry, the incomplete note, the error state, and the token chips.
- [X] **Given** an update draft is Ready, **Then** it can be revised as in story 027: the box reads "Ask for changes to the draft...", and the turns sent are the first request, the draft, and the revision.
- [X] **Given** an update draft is Ready, **Then** the approve button reads "Apply changes" and is enabled only when the draft is in the Ready state.
- [X] **Given** the user presses Discard, **Then** the draft disappears and the enabler's stored text is exactly as it was before the draft was started.

**Applying**

- [X] **Given** the user presses "Apply changes", **Then** the stored text of that enabler is replaced by the draft text (trimmed, with one trailing newline); the id, number, slug, first prompt, creation date, ADRs it derives from, and its links are not changed; no new enabler is created and the folder count stays the same.
- [X] **Given** the enabler is open when "Apply changes" is pressed, **Then** the file panel immediately shows the new text in both the rendered view and the Markdown view without the user reopening the file.
- [X] **Given** "Apply changes" is pressed, **Then** two messages are added to the conversation: the user's change text, and "Applied. I updated architectural-enablers/0003-\<slug\>.md." with a card that opens the file.
- [X] **Given** "Apply changes" is pressed, **Then** the draft is gone, the box is empty, and the prompt remains in editing mode for the same enabler.
- [X] **Given** the new text contains a first `# ` heading line, **Then** the enabler's title is read from it and the tree tooltip and all labels update to reflect it; **Given** the new text has no `# ` heading, **Then** the previous title is kept.
- [X] **Given** the page is reloaded after applying, **Then** the updated text is still present; **Given** two projects exist, **Then** each retains its own enabler text independently.
- [X] **Given** an update draft is on screen and the user opens a different file, **Then** the draft card still names the enabler it will update, and pressing "Apply changes" updates that enabler, not the currently open file.

## Business rules and constraints
- The target of the prompt is determined solely by the file that is open when no draft is on screen; only an approved enabler can be a target — drivers and ADRs are immutable (Architecture).
- Updating never creates: no new number, slug, prompt record, creation date, ADR selection, or link is stored (ADR-0007, ADR-0017).
- The stored text of the enabler is the only field that changes on apply; the title is re-read from the new text's first `# ` line.
- While a draft is on screen, the prompt works on that draft regardless of which file is subsequently opened; a new target is taken only when no draft is on screen (Practice: Incremental design).
- All draft states, revision turns (up to 40), streaming behaviour, and context assembly rules from stories 025, 027, and 028 apply unchanged to update drafts.
- The AI only drafts; the user decides with "Apply changes", and Discard leaves everything as it was (Practice: Pair programming).
- The enablers are kept by the backend in the project's database (story 026): the browser sends the identity of the enabler and the change, never the stored document; the backend reads the document, its ADRs and its links to build the request. The text on screen travels back only with the update, because a draft exists only in the browser.
- The update is one backend command: the new text, the title read again and the two messages are stored together in one transaction, so that neither is kept without the other (ADR-0017). The schema needs no change.
- An enabler stays editable when an ADR it derives from is no longer Accepted, because it was approved when that ADR was Accepted; an enabler that no longer exists is refused before the model is called.

## Out of scope
- Editing the enabler's text by hand in the file panel (a text box with Save and Ctrl+S).
- Changing the ADRs or the links of an enabler after creation, the *Cross enabler* flag, deleting an enabler, marking it Done, and synchronising it with the repository.
- Undoing an apply, a history of versions, and comparing old and new text.
- Updating a story, a driver, or an ADR by prompt (drivers and ADRs are immutable in the builder; stories have their own update stories).
- A warning when an enabler that stories already build on is changed.
- Detecting that a teammate changed the same enabler meanwhile: the update replaces the stored text with the draft.

## Tasks
- [X] Make an approved enabler the target of the prompt without making it editable by hand: the open file carries the identity of the enabler, and the panel stays read-only (ADR-0007)
- [X] Implement target selection: when no draft is on screen and the open file is an approved enabler, set that enabler as the prompt target; clear the target when the file is closed or a non-enabler file is opened
- [X] Update the composer component to render editing mode: the "Editing …" chip, hidden ADR panel and Link button, updated placeholder, updated send-button label, "What should change?" heading, and no examples; restore new-enabler mode when the target is cleared (ADR-0007, ADR-0016)
- [X] Build the request for an edit in the backend, from the identity of the enabler and the change: document type, task, file path, date, no identifier, the ADRs of the enabler by label, its linked documents in full (one step through linked enablers, excluding the enabler itself), the existing enablers and stories, "## Current document", and "## Requested change", after the project context of story 028; refuse an unknown enabler before the model is called (ADR-0017)
- [X] Add the edit instruction to the system prompt so the model returns the complete document with only the requested change applied (ADR-0007)
- [X] Start the draft in edit mode: record the target enabler's path and id in the draft, assign no new number, and show no ADR or link chips on the draft card
- [X] Change the approve button label to "Apply changes" for an edit draft and gate it on the Ready state
- [X] Implement the update in the backend (command and handler, `update` on the repository and its JPA adapter, `PUT` on the enabler with OpenAPI): replace the enabler's stored text, re-read the title from the first `# ` line, append the two confirmation messages, in one transaction (ADR-0017, ADR-0011, ADR-0014)
- [X] Update the file panel to re-render immediately on apply without requiring the file to be reopened, and update the read-only note to the editing-mode wording
- [X] Write Jest unit and acceptance tests: target selection rules (enabler, driver, ADR, no file, with a draft in progress), the request carrying the identity of the enabler, apply (text replaced, title updated, messages added, panel and tree refreshed), refusal and failure with the button pressable again, discard (text unchanged), title fallback, persistence with the browser's storage cleared, and a draft that outlives the open file (ADR-0005, ADR-0016)
- [X] Write JUnit tests for the domain and the application (revise, title, messages, unknown enabler, the exact edit request, no Accepted check, the cap of 40 turns), a Testcontainers test for the update and its persistence, REST controller tests (200, 400, 404 and the draft field), and an ArchUnit rule that the new command and adapters stay in their hexagonal layers (ADR-0005, ADR-0009, ADR-0012, ADR-0014)

## Files of the spike

In `bwyn-frontend/spikes/home-page/`. The spike's builder writes stories and enablers with the same box, so each file below does more than
these stories ask for. The column says which part is the base.

| File | What it holds | Stories |
|---|---|---|
| `Sections.tsx`, `App.tsx` | The Builder page and its "Open the builder" button, and the menu that shrinks when the workspace opens (see `story-builder-folders.md`). | 1 |
| `builder/BuilderWorkspace.tsx` | The frame: tree on the left, file (when open), prompt on the right or in the middle. `submit` (the number, the slug, the planned path, the context and the first turn of a new draft, and `revise` when a draft exists), `approve` (title, the stored enabler, the two messages, `expand('enablers')`, `discard`), `newStory` (the "New" button: confirm, discard, close the file, `sessionStart`, focus), `perform` and `request` (opening a file), `DEFAULT_EXPANDED`. **Edit mode** (story 7): `editTarget` (the open entry when it is editable), `submit` with a target (`start({ mode: 'edit', targetRef, path, number: editTarget.number, slug: '', ... firstTurn: buildEditText(...) })` and its `through` without the target), and `approve` for `mode === 'edit'` (`applyEdit`, the two messages "Applied. I updated ...", `discard`). Themes, runways, sync and delete are not used here. | 1, 3, 4, 7 |
| `builder/ChatPanel.tsx` | The whole prompt. `Composer` (the box that grows to 200 px, Enter and Shift+Enter, the green ADR panel with its chips and its two messages, the Link button, `canSend`, `sendLabel`, the placeholders for each state, Stop and send buttons); `MessageView` (the prompt as a message, and the assistant's message with the card that opens the file); `DraftView` (path, chip, chips of the links, the text, error, incomplete note, token chips, Discard, Retry, Approve); `ChatPanel` (the empty and the conversation layouts, the banner, the caption, the suggestions, the scroll, the focus). | 1, 2, 3, 4, 6 |
| `builder/useDraft.ts` | The draft: `Draft` and `DraftStart`, the four states, `run` (the call and the stream, abort, the "No text was received" error), `start`, `revise` (only Ready or Stopped), `retry`, `stop`, `discard`, `draftText`. | 1, 2, 4 |
| `builder/prompt.ts` | `SYSTEM_PROMPT` (the rules and the two templates), `buildContext` (ADRs, links, scopes, principles, practices; `linksText`, `scopeText`, `sectionOf`), `buildRequestText`, and `buildEditText` (the request for a change: "Task: edit an existing document", "File", "Current document", "Requested change"; story 7). Its story-only parts (the architecture context and its rule, the tasks of a story) are not used here. | 1, 4, 5, 7 |
| `builder/linkedDocs.ts` | `linkedDocumentsText` (the section, the relations, the 12,000-character cut) and `throughLinkedEnablers` (one step through a linked enabler). | 1, 5, 6 |
| `builder/LinkPicker.tsx` | `RefChip` and `LinkPicker` (the Link button, the popover grouped by kind, `hideKinds`, Escape and outside click, the count). Only an Accepted ADR is offered, in the spike. | 6 |
| `builder/generate.ts` | `makeId`, `today`, `slugify` (first words, accents out, 48 characters, `untitled`), `titleFrom` (first sentence of the prompt, 70 characters), `nextNumber`, `extractTitle` (the first `# ` line without "Story:", "Feature specification:", "AE-n."), `isAcceptedAdr`. | 1, 3 |
| `builder/useBuilder.ts` | The project's store in the browser (`bwyn-spike-home-page.builder`): `enablers`, `messages`, `addEnabler(enabler, messages)`, and `applyEdit(kind, id, content, messages)`, which for an enabler replaces **only** `content` and adds the messages (story 7). | 3, 7 |
| `builder/types.ts` | `StoredEnabler` (extends `StoredStory` with `derivesFrom` and `cross`), `Entry`, `ChatMessage`, `Submission`. | 1, 3, 6 |
| `builder/catalog.ts` | `buildEntries`: each enabler is an `Entry` with ref `enabler:<id>`, file name `<number>-<slug>.md`, folder `architectural-enablers`, label `AE-<number>. <title>`, `editable: true`. That one flag does two things in the spike: it makes the enabler the target of the prompt (`editTarget`) **and** it turns on the manual editing of the file panel (text box, Save, "Add link", Cross). These stories need the first without the second, so the flag must be split in two (for example `promptable` and `editable`). `KIND_META` for its icon (`ZapIcon`) and colour. | 3 |
| `builder/FolderTree.tsx` | The folder `architectural-enablers` (key `enablers`) and its file rows; the "New story" button of the header. | 3 |
| `builder/FileView.tsx` | The panel of an open file, with "Derives from", "Linked to" and "Referenced by", the two views and Close. It is rebuilt when the stored text changes (`key` with `sync.revision`, and its own copy of the text follows `entry.content`), which is how an applied update shows at once. Its Save, text box, Delete, Sync, Cross enabler, task, diagram and UI parts are not used here. | 3, 6, 7 |
| `builder/MarkdownView.tsx` | The Markdown reader and the rendered/Markdown toggle. | 3 |
| `builder/bedrockClient.ts` | `streamDraft` (POST `/api/draft`, reads NDJSON: `text`, `usage`, `done`, `error`), `fetchStatus` (the banner and the caption), `Usage`. | 1, 2 |
| `server/bedrock.js` | The dev server: `POST /api/draft` (the stream, the cache points, 40 turns, temperature 0.3, the errors 400, 503 and 502), `GET /api/draft/status`. `story-improve-with-ai.md` and `story-llm-provider.md` replace it with the backend. | 1, 2, 4 |
| `practices.ts`, `usePrinciples.ts`, `useDrivers.ts`, `useBuilder.ts` (ADRs) | The sources of the context: `practicesText()`, the principles with text, the drivers to link, and the ADRs. | 5, 6 |
| `icons.tsx` | `ZapIcon`, `SparklesIcon`, `ArrowUpIcon`, `StopIcon`, `LinkIcon`, `CheckIcon`, `TrashIcon`, `RotateIcon`, `ArrowRightIcon`. | 1 to 6 |
