# Story: Link from the prompt, for a story and for an enabler

- **Story**: 041-story-4-link-from-the
- **Created**: 2026-11-12
- **Status**: Draft

## Story
As a team member, I want to link a new story or enabler to other documents while I write its prompt, using the same picker as in the file panel, so that the AI reads those documents when it drafts and what I link is kept with the document I approve.

## Acceptance tests

**The Link button**
- [X] **Given** the prompt is in story-creation mode (checkbox unticked), **When** the prompt is rendered, **Then** a "Link" button using the shared `LinkPicker` component (story 038) is visible next to the "Mark as architectural enabler" checkbox, and the button is absent while a draft is on screen (ADR-0007, Architecture)
- [X] **Given** the checkbox is unticked (story mode), **When** the picker opens, **Then** it offers four groups: Drivers, ADRs, Architectural enablers, and Stories, following the rules of story 039
- [X] **Given** the checkbox is ticked (enabler mode), **When** the picker opens, **Then** the ADRs group is hidden and ADRs are excluded from the count, because the enabler selects its ADRs as green chips separately
- [X] **Given** the user linked an ADR while in story mode and then ticks the checkbox, **Then** that ADR disappears from the list and the count, and when the user unticks the checkbox, **Then** the ADR reappears checked; given the user sends while the checkbox is ticked, **Then** that ADR is not stored as a link of the resulting enabler
- [X] **Given** one or more documents are checked, **Then** the button shows a count and a `LinkChip` with icon and label appears above the prompt box for each; when the user activates the chip's "Remove link to \<label\>" control, **Then** the document is unchecked and its chip disappears
- [X] **Given** no linkable documents exist in the project, **When** the picker opens, **Then** it displays "Nothing to link yet."

**Default enablers for every story**
- [X] **Given** one or more enablers are marked cross, **When** the prompt is in story mode, **Then** each cross enabler is pre-checked in the picker and its chip is shown above the prompt box from the moment the prompt is opened
- [X] **Given** a default enabler is pre-checked and the user removes its chip, **Then** it is not re-added automatically; when a new cross enabler is later marked, **Then** only that new one is added to the defaults
- [X] **Given** the user ticks "Mark as architectural enabler", **Then** all default enablers are removed from the links, because an enabler does not receive them; when the user unticks it, **Then** all default enablers are restored
- [X] **Given** the user sends the prompt, **Then** the link selection resets to the default enablers only, ready for the next story
- [X] **Given** no enabler is marked cross, **Then** the link selection starts empty

**What the model and the stored document receive**
- [X] **Given** the user sends, **Then** the linked documents are included in the request body under the heading `## Linked documents (full text)`, each document appearing once in full, truncated at 12,000 characters
- [X] **Given** the draft card is shown, **Then** it displays the chips of the linked documents and those chips cannot be changed on the card
- [X] **Given** a story is being drafted, **Then** each linked ADR is labelled "linked"; given an enabler is being drafted, **Then** its ADR chips are labelled "derives from" and its other links are labelled "linked"
- [X] **Given** the draft is approved, **Then** the links are persisted with the story or enabler (an enabler's ADRs stored in `derivesFrom`), and the confirmation message reads "It is linked to N other items."
- [X] **Given** the user requests changes to a draft (revision flow), **Then** the Link button is not shown and the links remain unchanged for the revision

## Business rules and constraints
- The prompt and the file panel share the same `LinkPicker` component and the same ViewModel; a rule is written once (Principle: DRY and KISS; ADR-0007)
- The `LinkPicker` receives a `hideKinds` parameter; the prompt passes `['ADR']` when the checkbox is ticked (ADR-0007)
- Default enablers are pre-checked, not locked; the user may remove any of them and they will not be re-added automatically (Principle: Composition over orchestration)
- Ticking "Mark as architectural enabler" clears default enablers because an enabler does not receive them; unticking restores them
- Links are fixed at the moment the prompt is sent; changing links on an approved document is covered by stories 039 and 040
- The `LinkPicker` and `LinkChip` components reside in the `ui` layer; the ViewModel managing link state resides in the `interface-adapters` layer (ADR-0007, Architecture)
- Bindings for the link ViewModel are declared in the composition root only (ADR-0006, Architecture)

## Out of scope
- The ADR panel, the "Mark as architectural enabler" checkbox, and the prompt submission itself (stories 024 and 032)
- Marking an enabler as cross and changing which enablers are cross (a separate story referenced as `story-cross-enabler.md`)
- Persisting the link selection across page reloads
- Changing links on an already-approved story or enabler (stories 039 and 040)
- A search box or collapsible groups within the picker list (story 038)
- Backend endpoint changes beyond storing `links` and `derivesFrom` on approval

## Tasks
- [X] Write Jest tests for the prompt link ViewModel: groups shown when checkbox is unticked and ticked, count excluding hidden kinds, an ADR link preserved across tick/untick, default enablers present on open, removal without re-addition, restoration on untick, clearance on tick, and reset after send
- [X] Extend the link ViewModel to accept a `hideKinds` list and to manage default-enabler state (pre-check, removal, tick/untick, post-send reset), placing it in the `interface-adapters` layer (ADR-0007)
- [X] Wire the prompt's Link button to the shared `LinkPicker` and `LinkChip` components from story 038, passing `hideKinds: ['ADR']` when the checkbox is ticked
- [X] Implement default-enabler pre-selection: on prompt open, query cross enablers and pre-check each one; on send, reset to defaults
- [X] Update the prompt's send handler to include linked documents in the request under `## Linked documents (full text)`, each truncated at 12,000 characters, with "linked" or "derives from" labels as appropriate
- [X] Update the draft card to display link chips (read-only) and pass `derivesFrom` and `links` to the approval handler
- [X] Update the approval handler to persist `links` and `derivesFrom` on the stored story or enabler and return the count for the confirmation message
- [X] Remove the builder-coupled `LinkPicker.tsx` once both the file panel and the prompt use the shared component, and confirm stories 039 and 040 acceptance tests still pass
- [ ] Verify keyboard-only navigation, dark and light themes, and narrow-column layout by hand
  _Open: needs a person in a real browser. Keyboard handling is covered by the LinkPicker tests (story 038)._
