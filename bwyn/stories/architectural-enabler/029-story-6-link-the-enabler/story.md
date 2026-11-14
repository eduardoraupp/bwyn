# Story: Link the enabler to other documents

- **Story**: 029-story-6-link-the-enabler
- **Created**: 2026-11-08
- **Status**: Draft

## Story
As a team member, I want to link the enabler to the drivers and other enablers that shape it, so that the AI uses what they say and the connection is kept with the enabler.

## Acceptance tests

**Choosing links**

- [X] **Given** the enabler composer is open, **Then** a "Link" button with a link icon sits below the prompt box; **When** the user presses it, **Then** a dialog (`role="dialog"`, title "Choose what to link") opens, groups documents by kind under the headings "Drivers", "Architectural enablers", and "Stories", and shows a checkbox and label for each document.
- [X] **Given** the link dialog is open for an enabler prompt, **Then** ADRs are absent from the list, because they are chosen as chips in the ADR panel (story 024).
  _Tested in story 037/041 (link-the-story.acceptance.test.tsx), which owns this behaviour; the duplicate test here was removed._
- [X] **Given** a driver exists with no text yet, **Then** it still appears in the list; **Given** no linkable documents exist at all, **Then** the dialog shows "Nothing to link yet."
- [X] **Given** the user checks a document, **Then** the Link button shows a count badge, and a chip with the document's icon and label appears above the prompt box with a "Remove link to \<label\>" button; **When** the user activates that button, **Then** the chip disappears and the checkbox is cleared.
- [X] **Given** the link dialog is open, **When** the user presses Escape or clicks outside the dialog, **Then** the dialog closes and the current selection is preserved.
- [X] **Given** the user submits the prompt, **Then** the link chips are cleared from the composer alongside the prompt text.
- [X] **Given** a draft card is displayed, **Then** it shows chips for both the ADRs it derives from and the linked documents, and neither set can be changed on the card; **Given** a revision prompt is shown (story 027), **Then** the Link button is not present.

**What the model receives**

- [X] **Given** at least one link is chosen, **Then** the request part contains a section "## Linked documents" with one subsection per document headed "### \<label\> (\<relation\>, \<path\>)" and the document's full text truncated at 12,000 characters, in the order the links were chosen, with the relation "linked"; each document appears exactly once. **Then** the ADRs the enabler derives from are not repeated there: they are named under "## Derives from (ADRs)" and their full text is in the project context (story 028).
- [X] **Given** a linked enabler is included, **Then** the request also contains, one step further, the ADRs that linked enabler derives from (relation "derived from by the linked \<label\>", with their full text because the project context holds only the ADRs the new enabler derives from, and left out when the new enabler derives from them itself) and the drivers and enablers that linked enabler itself links to (relation "linked from the linked \<label\>", with their text); documents reached this way are not expanded further.
- [X] **Given** no links are chosen, **Then** the linked-documents section reads "## Linked documents" followed by "- None", and the ADRs the enabler derives from are still named under "## Derives from (ADRs)" and are in full in the project context.
- [X] **Given** the system prompt, **Then** it instructs the model to treat linked documents as context and to cite one only when a rule comes from it, for example "(Driver: Constraints)".

**Keeping them**

- [X] **Given** the user approves the draft, **Then** the stored enabler records the chosen links (excluding the ADRs it derives from, which are stored separately), and the confirmation message reads "It is linked to N other items."
- [X] **Given** the approved enabler is opened in the read-only file panel, **Then** a "Linked to" section shows a chip for each linked document, and activating a chip opens that document's file panel; **Given** a document the enabler links to is opened, **Then** it shows the enabler under "Referenced by".
  _Tested in story 040 (link-from-enabler-panel.acceptance.test.tsx), which owns this behaviour; the duplicate test here was removed._
- [X] **Given** an ADR the enabler derives from is opened, **Then** its panel shows the enabler under "Referenced by", by number, and activating the chip opens the enabler; an ADR that no enabler derives from (and no story links to) shows no "Referenced by".
- [X] **Given** a document that was linked to an enabler is later removed from the project, **Then** the "Linked to" section omits that chip silently rather than showing an error.

## Business rules and constraints

- The model receives only outgoing links: the documents the enabler links to and, through a linked enabler, what that enabler is tied to. Documents that merely reference the enabler are not sent (Architecture: application layer reaches external systems only through ports).
- The full text of an ADR the enabler derives from is given once, in the project context of story 028; the request names those ADRs by label and never repeats their text. An ADR reached only through a linked enabler is not in the context, so it comes with its full text under "## Linked documents". A document that is linked or reached more than once is given once, and a link whose document is gone is left out without a word.
- One step of transitive expansion is followed through a linked enabler only; drivers, stories, and ADRs reached transitively are included but not expanded further.
- A link reference is derived from the document's file name (e.g. `driver:constraints`, `enabler:0001-set-up-...`), never from a generated identifier, so that references remain stable (Principle: Declarative over imperative).
- Links are fixed at draft time. Changing the links of an approved enabler is a separate story.
- Full text of each linked document is truncated at 12,000 characters before being sent to the model.

## Out of scope
- Adding or removing links of an already-approved enabler.
- The *Cross enabler* flag and links between stories.
- Showing document links as a graph.
- Transitive expansion through stories, drivers, or ADRs.

## Tasks
- [X] Add the Link button and dialog to the enabler composer: grouped by kind (Drivers, Architectural enablers, Stories), no ADRs, with checkboxes, count badge, and removable chips above the prompt box (ADR-0007, AE-0002).
- [X] Wire the dialog open/close behaviour: Escape and outside-click close it while preserving selection; chips clear on submit.
- [X] Display link chips on the draft card alongside ADR chips, read-only; hide the Link button from the revision prompt (story 027).
- [X] Extend the backend request builder to assemble "## Linked documents": the linked documents in the order chosen (or "- None"), each with its relation and path, truncated at 12,000 characters, each appearing once; implement `FetchLinkedDocumentsPort` with the project's drivers, enablers and ADRs in place of the placeholder adapter (ADR-0017, AE-0005).
- [X] Implement one-step transitive expansion for linked enablers: include their derives-from ADRs and their own links, without recursing further (ADR-0017).
- [X] Update the system-prompt template to instruct the model to treat linked documents as context and cite them only when a rule comes from them.
- [X] Persist the chosen links (excluding derives-from ADRs) on the enabler aggregate in `bwyn-core`; expose them via the approval command in `bwyn-application` (ADR-0017, AE-0005).
- [X] Add the confirmation message "It is linked to N other items." to the approval response (story 026).
- [X] Render "Linked to" chips in the read-only file panel and "Referenced by" on the panels of linked documents; suppress missing-document chips silently (AE-0002).
- [X] Write Jest tests: dialog grouping, ADR absence, chip add/remove, the links sent with the first draft, a change and a retry, the links sent with the approval, "Linked to" with a removed target left out silently, and "Referenced by" (ADR-0005, ADR-0016). Write JUnit tests for the request with and without links, the 12,000-character truncation, one-step transitive expansion, and the adapter against the database (ADR-0005, ADR-0012).
- [X] Write a JUnit test asserting that the linked-documents port is implemented only by an outbound adapter and that no domain class references adapter types (ADR-0009, ADR-0017).

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

- **The checkbox "Mark as architectural enabler" chooses what the box writes.** The spike's box has that tick. It comes with story 032, which
  makes the story the default: the enabler is written when it is ticked (`asEnabler`), and the placeholder, the box, the ADR panel and the
  button of the enabler below are those of the ticked state.
- **Story-only parts are left out:** the story examples ("As a returning customer, ..."), the three suggestions (one is kept), the
  automatic link of cross enablers, the architecture in the context, the tasks that a story copies into its task list, and the edit mode
  with its "Editing ..." chip.
- **The folder `architectural-enablers` lists the enablers, and the panel is read-only.** In the spike an open enabler can be edited,
  linked, marked Cross, synced and deleted. Here none of that is shown, and the note says so.
- **The button of the tree header is "New"**, named "New" (story 032). The spike's is "New story" and also resets the folders to
  `stories` and `enablers` open. Here it only resets the conversation.
- **The workspace's middle** is the prompt again (in `story-builder-folders.md` it was an empty state), and with a file open the prompt is a
  narrow column on the right of the file.
- **The call to the model** is the backend's, not the dev server's.