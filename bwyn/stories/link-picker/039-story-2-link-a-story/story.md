# Story: Link a story to other documents

- **Story**: 039-story-2-link-a-story
- **Created**: 2026-11-12
- **Status**: Draft

## Story
As a team member, I want to link a story to the drivers, ADRs, enablers and other stories that shape it from the story's own panel, so that the connection is kept with the story and the AI and the sync use it.

## Acceptance tests

**View model (checked without rendering)**

- [ ] **Given** the documents of a project and a story as the target, **Then** the picker groups are, in order: "Drivers", "ADRs", "Architectural enablers", "Stories"; a group with no options is omitted.
- [ ] **Given** the target story, **Then** it is absent from its own list so a story can never link to itself.
- [ ] **Given** the drivers of the project, **Then** every driver is offered whether or not it has been written. **Given** the stories of the project, **Then** every other story is offered, inside a theme or not, labelled `001. Sign in`, sorted by number.
- [ ] **Given** the ADRs of the project, **Then** only Accepted ADRs are offered. **Given** an ADR that is already linked to the story and is no longer Accepted, **Then** it is still offered and shown as checked so it can be unlinked.
- [ ] **Given** a story as the target, **Then** `applyLinkSelection` places every selected reference in `links` and returns no `derivesFrom`.
- [ ] **Given** a consumer that hides a kind, **Then** that kind is absent from the groups and absent from the count.

**Story panel**

- [ ] **Given** an open story, **Then** its panel shows a "Linked to" heading with an "Add link" button on its right, the chips of its current links beneath it, and the line "Not linked to any driver, ADR, enabler or story yet." when there are none; the panel has no "Derives from" section.
- [ ] **Given** the "Add link" button, **Then** it opens the shared `LinkPicker` from story 038 with the groups produced by the view model and displays the count of the story's current links.
- [ ] **Given** the user checks an option in the picker, **Then** the link is saved immediately with no separate Save action; the chip, the count on the button, and the checkbox all update in the same moment; the story's text, title, and tasks are not changed.
- [ ] **Given** the user checks an ADR, **Then** its chip appears in the "Linked to" section alongside the others, in the order the links were added.
- [ ] **Given** the chips, **Then** each carries the icon and tone of its kind (driver indigo, ADR amber, enabler green, story blue) and an × button. **When** the user presses ×, **Then** that link is removed and its checkbox is unchecked. **When** the user presses the chip label, **Then** that document opens in the panel.
- [ ] **Given** a link whose target document no longer exists, **Then** no chip is shown for it and nothing fails.
- [ ] **Given** a story that is in a theme or is moved into one, **Then** its links and the links pointing to it are unchanged, because a link names the folder and not the theme.
- [ ] **Given** the read-only note in the panel, **Then** it still reads "Read-only here. Changing a story comes in another story."; the panel has no text box and no Save; only the links section changes.

**"Referenced by" in every panel**

- [ ] **Given** any document that another document links to (a driver, an ADR, an enabler, or a story), **Then** its panel shows a "Referenced by" section with a chip for each document that links to it or derives from it, and each chip opens that document in the panel.
- [ ] **Given** a link is added from another document while a panel is open, **Then** the open panel reflects it immediately.
- [ ] **Given** a driver or an ADR is open, **Then** its panel has no "Add link" and no "Linked to" section, only "Referenced by".

**What the links do**

- [ ] **Given** a story is linked to an ADR, **Then** a generation request for that story includes that ADR in full as a linked document (consistent with story 036). **Given** the story is linked to an enabler, **Then** the documents that enabler links to and its ADRs are also included, one level deeper.
- [ ] **Given** a link between two documents, **Then** the sync scope for the story covers the documents it links to and the documents that link to it (consistent with story 035).
- [ ] **Given** the story's `meta.json` in the repository, **Then** it contains a `links` array, sorted, and a pull restores those links.

**Persistence**

- [ ] **Given** the page is reloaded, **Then** the links of each story are still present. **Given** two projects exist, **Then** each project keeps its own links independently.

## Business rules and constraints

- A link is a stable reference formed from the document's file name (`driver:constraints`, `adr:0006`, `enabler:0001-set-up-…`, `story:001-…`), never from a random identifier, so that it is portable across machines and survives a story being moved into a theme (ADR-0007).
- A story cannot link to itself. Links may exist in both directions between two documents and a chain may loop; the picker does not validate for cycles.
- Only an Accepted ADR may be newly linked. An already-linked ADR that has since lost Accepted status remains in the list so it can be removed.
- Adding or removing a link changes only the `links` field. It does not change the story's text, title, tasks, or dates.
- Drivers and ADRs are immutable in the builder: they carry no `links` of their own and show no "Add link" control (ADR-0007).
- The `LinkPicker` component used here is the one delivered by story 038; this story adds only the story-specific view-model logic and the panel wiring on top of it (Practice: DRY and KISS).

## Out of scope

- Editing the story's text, tasks, diagrams, or UI example by hand; deleting a story; synchronising a story with the repository.
- Linking from the prompt box, and the "derives from" ADR relationship used by enablers (story 029).
- A search box in the picker, grouping stories by theme in the picker list, a rule restricting which kinds may link to which, a graph view of links, and a warning about the effect of a link on sync scope or AI context size.
- Cleaning up links to a deleted story or enabler (belongs to the delete story).
- Moving the rest of the builder to the final Clean Architecture layers; only the link view model, the panel section, and the "Referenced by" widget are placed in the correct layers here (ADR-0007).

## Tasks

- [ ] Write failing unit tests for `storyLinkChoices`: correct group order, target absent from its own list, every driver offered, every other story offered by label and number, only Accepted ADRs plus any already-linked non-Accepted one, hidden kinds excluded from groups and count.
- [ ] Implement `storyLinkChoices` in the interface-adapters layer until all tests pass (ADR-0007).
- [ ] Write failing unit tests for `applyStoryLinkSelection`: all selections land in `links`, `derivesFrom` is absent.
- [ ] Implement `applyStoryLinkSelection` until all tests pass.
- [ ] Add the "Linked to" section to the story panel: "Add link" button wired to the shared `LinkPicker` (story 038), `LinkChip` list with open-document and remove actions, empty-state line; write through the store's `patch` without touching the story text.
- [ ] Add the "Referenced by" section to every document panel (driver, ADR, enabler, story); update it reactively when any link changes.
- [ ] Remove the "Add link" control from driver and ADR panels; confirm "Referenced by" remains.
- [ ] Write integration tests: add a link, remove a link, link to an ADR, missing target shows no chip, story in a theme, page reload, two projects, story text unchanged after link operations.
- [ ] Write tests confirming that linked documents are included in the generation request and that `meta.json` contains a sorted `links` array.
- [ ] Verify the full build and test suite completes within the ten-minute budget (Practice: Ten-Minute Build).
