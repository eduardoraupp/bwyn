# Story: Add links to a story or an architectural enabler with a reusable link picker

- **Story**: 031
- **Created**: 2026-11-09
- **Status**: Draft

## Story
As a team member, I want to link a story or an architectural enabler to the drivers, ADRs, enablers and stories that shape it using a link picker in the file panel and in the prompt, so that the connections are kept with the document and used by the AI and by the sync.

## Acceptance tests

### The `LinkPicker` component (no builder context)
- [ ] **Given** a `LinkPicker` with a label, **Then** it renders one button with a link icon and that label (default "Link"), styled as a pill with a thin border, with `aria-haspopup="dialog"` and `aria-expanded="false"`.
- [ ] **Given** selected options, **Then** the button shows a small round count of the selected values that appear in its groups, and no count when that number is zero; a selected value that belongs to no group is not counted.
- [ ] **Given** the button, **When** the user presses it, **Then** a list opens with `role="dialog"` named "Choose what to link" and `aria-expanded="true"`; **When** the user presses it again, **Then** the list closes.
- [ ] **Given** the list is open, **Then** it shows the groups in the order they were given, each as a `fieldset` with its title as a small upper-case legend, and each option as a checkbox with a label truncated to "…" when long (full label as tooltip); a group with no option is not shown.
- [ ] **Given** no groups, or groups with no options, **Then** the list shows "Nothing to link yet."
- [ ] **Given** an option, **When** the user checks it, **Then** the component calls `onChange` once with the current selection plus that option appended; **When** the user unchecks it, **Then** `onChange` is called with the selection without it, preserving order.
- [ ] **Given** the component is controlled, **Then** it holds no selection state of its own: checkboxes reflect `selected`, and a parent that ignores `onChange` leaves them unchanged.
- [ ] **Given** an option that is locked with a reason, **Then** its checkbox is checked, disabled, and shows the reason as a tooltip; it cannot be unchecked by mouse or keyboard; it is included in the count.
- [ ] **Given** the list is open, **When** the user presses Escape, clicks or presses anywhere outside the button and the list, or presses the button again, **Then** the list closes and the selection is unchanged.
- [ ] **Given** the list opens via keyboard (Enter or Space on the button), **Then** focus moves to the first checkbox; Tab and Shift+Tab cycle through checkboxes; Space toggles; **When** Escape closes the list, **Then** focus returns to the button.
- [ ] **Given** two `LinkPicker` instances on one page with different selections, **Then** each shows its own selection; toggling in one does not affect the other.
- [ ] **Given** a custom `label`, `dialogLabel`, or `emptyText`, **Then** the component renders them in place of the defaults.

### Positioning and portal
- [ ] **Given** the list is open, **Then** it is rendered in a portal at the top of the document, positioned with `position: fixed`, so that no scrolling or clipping ancestor hides it.
- [ ] **Given** the button, **Then** the list is 320 px wide (or the window width minus 16 px when that is smaller), aligned to the left edge of the button, and never closer than 8 px to any window edge.
- [ ] **Given** `placement` "down" (the default), **Then** the list opens below the button; it opens above only when there is less than 200 px below and more room above. **Given** `placement` "up", **Then** it opens above when there are at least 160 px above or more room above than below.
- [ ] **Given** the list is open, **Then** it is at most 320 px tall (less when there is less room) and scrolls internally.
- [ ] **Given** the list is open, **When** the window is resized or any ancestor scrolls, **Then** the list repositions relative to the button.

### The `LinkChip` component
- [ ] **Given** a `LinkChip` with a label, an icon and a tone, **Then** it renders a pill with the icon and the label (truncated to "…" when long) in that tone.
- [ ] **Given** it has an `onOpen`, **Then** the label is a button, underlined on hover and focus. **Given** it has an `onRemove`, **Then** a small X button labelled "Remove link to \<label\>" is shown. **Given** it has neither, **Then** it is plain text.

### The builder's view model (`linkChoices`)
- [ ] **Given** the documents of a project and a target, **Then** the groups are "Drivers", "ADRs", "Architectural enablers" and "Stories" in that order; a group with no options is omitted.
- [ ] **Given** the target document, **Then** it is absent from its own list.
- [ ] **Given** the ADRs, **Then** only Accepted ADRs are offered as new options; an ADR that is already linked but is no longer Accepted is still offered, checked, so it can be unlinked.
- [ ] **Given** the drivers, **Then** every driver is offered regardless of whether it has been written.
- [ ] **Given** a consumer that hides a kind (e.g. the prompt hides "adr"), **Then** that kind is absent from the groups and not counted.
- [ ] **Given** an enabler with exactly one ADR it derives from, **Then** that ADR is offered locked with the reason "An enabler implements at least one ADR." **Given** it has two or more ADRs, **Then** none is locked.
- [ ] **Given** a story, **Then** no option is locked and its ADR links are ordinary links.

### File panel — story and enabler
- [ ] **Given** an open story or enabler, **Then** its panel shows the heading "Linked to" with an "Add link" button on its right, the chips of its links below, and the line "Not linked to any driver, ADR, enabler or story yet." while it has none.
- [ ] **Given** an open enabler, **Then** the picker's checked values are the union of its `derivesFrom` ADRs and its `links`; **When** the user checks an ADR, **Then** it is added to `derivesFrom` and its chip appears under "Derives from"; **When** the user checks anything else, **Then** it is added to `links` and its chip appears under "Linked to".
- [ ] **Given** an open story, **When** the user checks an ADR, **Then** its chip appears under "Linked to" with the others.
- [ ] **Given** a link is added or removed, **Then** the change is persisted immediately with no Save button; the document's text, title and dates are not modified; the panel, the button count and the checkboxes all update in the same render.
- [ ] **Given** the chips under "Linked to", **Then** they appear in the order they were linked, each with the icon and colour of its kind (driver indigo, ADR amber, enabler green, story sky), each with an X; **When** the user presses the X, **Then** that link is removed and its checkbox is unchecked; **When** the user presses the label, **Then** that document opens in the panel.
- [ ] **Given** the chips under "Derives from" on an enabler, **Then** each opens its ADR when pressed and has no X; an ADR is removed only through the picker.
- [ ] **Given** a link to a document that no longer exists, **Then** no chip is shown for it and nothing fails.
- [ ] **Given** a driver or an ADR is open, **Then** its panel has no "Add link" and no "Linked to" section.
- [ ] **Given** any document that another document links to or derives from, **Then** its panel shows "Referenced by" with a chip for each such document; each chip opens that document; **When** a link is added from another open document, **Then** the "Referenced by" section updates immediately.
- [ ] **Given** the page is reloaded, **Then** the links and `derivesFrom` of each document are still present. **Given** two projects, **Then** each keeps its own links.

### Reuse by the prompt and a test screen
- [ ] **Given** the prompt that creates an enabler (story 024), **Then** its "Link" button is this `LinkPicker` with the label "Link", ADRs hidden, `placement` as configured, and chips above the box built from `LinkChip` with an X; all acceptance tests of story 024 still pass.
- [ ] **Given** a test screen unrelated to the builder that supplies its own groups ("Colours", "Sizes") and its own selection, **Then** the component works as specified above, shows only those groups, and shows nothing of drivers, ADRs, enablers or stories.
- [ ] **Given** the builder file panel, the prompt and the test screen rendered simultaneously, **Then** each maintains its own independent selection.

### What the links do
- [ ] **Given** a driver has been linked to an enabler, **When** the user sends a change prompt for that enabler (story 030), **Then** the request's "Linked documents" includes that driver's full text.
- [ ] **Given** a link between two documents, **Then** a sync that covers one document also covers the other (story 017 / story 018).

## Business rules and constraints
- `LinkPicker` and `LinkChip` are generic presentation components (ADR-0007). They import nothing from the builder, the catalog, the ADR list or the driver list, and they enforce no rule about kinds. They live in the shared presentation layer (Architecture).
- The builder's rules — kind order, headings, "not itself", the Accepted-ADR filter, hidden kinds, and the locked-last-ADR constraint — belong to a view model (`linkChoices`) that has no React dependency and is tested independently (ADR-0007).
- A link is a stable file-name reference (`driver:constraints`, `adr:0006`, `enabler:0001-…`, `story:001-…`), never a random id, so it is portable across machines.
- A document cannot link to itself. Bidirectional links and cycles are permitted; the picker does not check for them.
- Only an Accepted ADR may be newly linked. An ADR that was linked and later lost Accepted status remains linked and can be unlinked.
- An enabler must always derive from at least one ADR; its last `derivesFrom` entry is locked in the picker.
- Adding or removing a link changes only `links` and `derivesFrom`; it does not touch the document's text, title or dates (Principle: SRP).
- Drivers and ADRs are immutable in the builder; they carry no links of their own and only appear under "Referenced by".
- Links are sent to the AI in full text (story 028 / story 030) and widen the scope of a sync (story 017 / story 018).
- Persistence is in `localStorage` per project in the browser (spike baseline); `links` and `derivesFrom` are written sorted to `meta.json` in the repository (story 026 / story 017).

## Out of scope
- Linking from a driver or an ADR, or linking to anything other than a driver, ADR, enabler or story (files, URLs).
- A search box, a filter or collapsible groups in the picker list; the list scrolls.
- A rule restricting which kinds may link to which, or a warning about cycles.
- Warning the user about the effect of a link on the sync scope or on the AI context.
- A graph view of the links.
- Removing a story or an enabler and cleaning up links to it (belongs to a delete story).
- Moving the rest of the builder to the final frontend Clean Architecture layers; only `LinkPicker`, `LinkChip` and their view model are placed in the correct layers here (ADR-0007).

## Tasks
- [ ] Write Jest tests for `LinkPicker` using React Testing Library with made-up groups: button rendering and count, open/close (button press, Escape, outside click), checkboxes and `onChange`, controlled mode, locked options, empty state, keyboard navigation and focus, two independent pickers, and portal positioning (mock `getBoundingClientRect`, window size and scroll).
- [ ] Implement `LinkPicker` in the shared presentation layer: props interface, portal rendering with `position: fixed`, placement logic and recompute on resize and scroll (capture), close-on-outside-mousedown, focus management, and Tailwind classes from the spike.
- [ ] Implement `LinkChip` in the shared presentation layer: icon, label with truncation and tooltip, tone classes, optional `onOpen` button, optional `onRemove` X button with accessible label.
- [ ] Write Jest unit tests for the `linkChoices` view model: kind order, self-exclusion, Accepted-ADR filter with already-linked non-Accepted ADR, all drivers offered, hidden kinds, locked-last-ADR for an enabler, no locked options for a story.
- [ ] Implement the `linkChoices` view model and `applyLinkSelection` helper with no React dependency (ADR-0007, Architecture).
- [ ] Build the "Derives from", "Linked to" (with "Add link" button and chips) and "Referenced by" sections of the file panel for stories and enablers, wiring `linkChoices`, `LinkPicker`, `LinkChip` and the store's `patch`; ensure "Referenced by" updates immediately when a link is added elsewhere.
- [ ] Migrate the prompt's "Link" button and its chips (story 024) to use the shared `LinkPicker` and `LinkChip`; verify all story 024 acceptance tests still pass.
- [ ] Add the test screen with its own groups ("Colours", "Sizes") to confirm the component has no builder dependency; keep it in the test suite only.
- [ ] Delete the builder-coupled `LinkPicker.tsx` from the spike once both consumers use the shared component.
- [ ] Verify keyboard-only navigation, dark and light themes, and positioning near the bottom and right edges of the window by hand in a browser.
