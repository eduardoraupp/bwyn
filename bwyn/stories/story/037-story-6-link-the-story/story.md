# Story: Link the story to other documents, and the enablers that every story gets

- **Story**: 037
- **Created**: 2026-11-09
- **Status**: Draft

## Story
As a team member, I want to link the story I am writing to the drivers, ADRs, enablers and stories that shape it, and to have the enablers that apply to every story linked from the start, so that the AI uses what they say and no rule that every story needs is forgotten.

## Acceptance tests

**The Link button**

- [X] **Given** the prompt box is not ticked as an architectural enabler, **When** the user opens the Link button, **Then** it shows the shared picker with sections for Drivers, ADRs, Architectural enablers, and Stories (ADR-0007).
- [X] **Given** the prompt box is ticked as an architectural enabler, **When** the user opens the Link button, **Then** the ADRs section is absent, because ADRs are chosen through the enabler's own ADR panel, not through the link picker.
- [X] **Given** the ADR section is shown, **Then** only ADRs with status Accepted are offered; an ADR the user selects stays listed until explicitly removed.
- [X] **Given** the user checks a document in the picker, **Then** the Link button shows a count badge, and a chip with the document's icon and label appears above the prompt box with an ×  button labelled "Remove link to \<label\>"; **When** the user activates that ×, **Then** the chip disappears and the document is unchecked in the picker.
- [X] **Given** the user submits the prompt, **Then** all manually chosen links are cleared from the composer (default enablers remain), and the draft card shows a chip for each link that was sent; those chips are read-only on the draft card.

**The enablers that every story gets**

- [X] **Given** one or more enablers are marked as cross-project (applying to every story), **When** the prompt box is not ticked as an architectural enabler, **Then** each cross enabler is pre-checked in the picker and its chip appears above the prompt box before the user does anything.
- [X] **Given** the user removes a pre-checked cross-enabler chip, **Then** that enabler is not re-added automatically; **When** a new cross enabler is later marked, **Then** only that new one is added to the pre-checked set.
- [X] **Given** the user ticks "Mark as architectural enabler", **Then** all default-enabler chips are removed from the composer, because cross enablers apply to stories only; **When** the user unticks it, **Then** the cross enablers that were not manually removed are restored.
- [X] **Given** the user submits the prompt, **Then** the composer resets to the default cross enablers only (manually added links are cleared).
- [X] **Given** no enabler is marked as cross-project, **Then** the link list starts empty.

**What the model receives**

- [X] **Given** one or more links, **Then** the request body contains a section headed `## Linked documents (full text)` with one sub-section per linked document headed `### <label> (<relation>, <path>)` followed by the document's full text truncated to 12,000 characters; each document appears exactly once even if reachable by multiple paths.
- [X] **Given** a linked enabler, **Then** the documents one step further through it — its ADRs, its own linked enablers, and its linked stories — are included in the same section, each marked with the relation `derived from <enabler label>`; documents reached through a story, a driver, or an ADR are not expanded further.
- [X] **Given** an ADR linked directly to the story, **Then** its relation label is `linked` (stories have no "derives from" relation).
- [X] **Given** no links, **Then** the section reads `## Linked documents` with a single line `- None`.
- [X] **Given** the system prompt, **Then** it instructs the model to treat linked documents as context and to cite a document only when a rule directly comes from it.

**Keeping the links**

- [X] **Given** the user approves the draft, **Then** the links are persisted with the story, and the confirmation message reads "It is linked to N other items." (where N ≥ 1) or omits the sentence when N = 0.
- [X] **Given** the user opens an approved story, **Then** a "Linked to" section shows a chip for each outgoing link; activating a chip opens that document's file panel.
- [X] **Given** a document that links to the story, **Then** that document's file panel shows the story under "Referenced by".
- [X] **Given** a linked document is later removed from the project, **Then** its chip is hidden silently; no error is shown.

## Business rules and constraints

- The model receives outgoing links only; documents that merely reference the story are not sent (Principle: DRY and KISS).
- Transitive expansion follows exactly one step, and only through enablers — not through stories, drivers, or ADRs.
- A link reference is a stable identifier derived from the file name: `driver:constraints`, `enabler:0001-set-up-...`, `story:001-...`, `adr:0006`.
- Default cross enablers are pre-selected, not forced: the user may remove any of them, and a removed one does not return unless the user ticks and unticks the enabler box (which resets the selection to the current cross-enabler set minus those the user had removed before the tick). (Principle: Declarative over imperative)
- An enabler prompt does not receive default cross enablers; the ADR panel covers that concern for enablers.
- Links are fixed at draft time. Changing links on an approved story is out of scope here and is covered by story 031.
- Only Accepted ADRs may be linked to a story (ADR-0007).
- Document text is truncated at 12,000 characters per document before being sent to the model.
- The "Referenced by" relationship is derived at read time from the stored outgoing links of other documents; it is not stored separately (Principle: DRY and KISS).

## Out of scope

- Marking an enabler as cross-project or removing that flag (belongs to the enabler editing flow).
- Adding or removing links of an already-approved story (covered by story 031).
- A graph view of the links.
- Warning the user about the effect of links on sync scope or on AI context size.
- Changing the ADRs or other links of a draft after it has been submitted; the user must discard and restart (consistent with story 027).

## Tasks

- [X] Extend the story composer ViewModel to hold a link set initialised from the current cross enablers, and expose a command to add, remove, and reset links (ADR-0007, Architecture).
- [X] Wire the shared link picker into the story composer, hiding the ADR section when the enabler box is ticked and showing it otherwise (ADR-0007).
- [X] Implement the cross-enabler default logic: pre-check cross enablers on mount, track user removals, restore on untick, clear non-default links on submit (Architecture).
- [X] Write a use-case function that assembles the `## Linked documents (full text)` section: fetches each linked document's text, truncates to 12,000 characters, expands one step through linked enablers, deduplicates, and formats each entry with its relation label (ADR-0007, Architecture).
- [X] Update the story-draft request handler to include the linked-documents section and the system-prompt instruction to cite documents only when a rule comes from them (Architecture).
- [X] Add a `links` field to the story persistence model and store the outgoing link references on approval; update the approval command handler (ADR-0017).
- [X] Expose a backend query endpoint that returns the links stored with a story, used by the file panel to render "Linked to" chips (ADR-0017, ADR-0014).
- [X] Derive "Referenced by" at read time in the file-panel ViewModel by querying which other documents list this story in their stored links (ADR-0007, Architecture).
- [X] Handle a missing linked document gracefully: hide its chip silently in the "Linked to" section (Principle: KISS).
- [X] Write Jest unit tests covering: link list for a story vs. an enabler; default-enabler pre-selection, user removal, new cross enabler, tick/untick, post-submit reset; request assembly with and without links, truncation at 12,000 characters, one-step expansion through an enabler, deduplication, and a removed target (ADR-0005, Practice: Pair programming).
- [ ] Verify the full build and test suite still passes within the ten-minute budget (Practice: Ten-Minute Build, Practice: Continuous Integration). (Open: the suites that touch this story pass, but a full frontend run takes about 25 minutes on this machine, beyond the ten-minute budget.)
