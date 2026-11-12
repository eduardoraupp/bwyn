# Story: A reusable link picker

- **Story**: 038-story-1-a-reusable-link
- **Created**: 2026-11-12
- **Status**: Draft

## Story
As a developer, I want a `LinkPicker` and a `LinkChip` that are plain React components with no knowledge of the builder, so that the file panel, the prompt, and any other component can offer a list of things to link using their own groups and rules.

## Acceptance tests

**The button**
- [X] **Given** a `LinkPicker` with a label, **Then** it renders one button with a link icon and that label (default: "Link"), styled as a pill with a thin border, with `aria-haspopup="dialog"` and `aria-expanded="false"`.
- [X] **Given** options are selected, **Then** the button shows a small round count of the selected options that belong to its groups, and shows nothing when the count is zero; a selected value that belongs to no group is not counted.
- [X] **Given** the button, **When** the user presses it, **Then** a list opens with `role="dialog"` and accessible name "Choose what to link", and `aria-expanded` becomes `"true"`; **When** the user presses it again, **Then** the list closes.

**The list**
- [X] **Given** the list, **Then** it renders the groups in the order they were given, each as a `fieldset` with its title as a small upper-case `legend`; each option is a checkbox with its label truncated with "…" when long and the full label as a tooltip; a group with no options is not rendered.
- [X] **Given** no groups, or groups with no options, **Then** the list renders the text "Nothing to link yet."
- [X] **Given** an option, **When** the user checks it, **Then** `onChange` is called once with the current selection plus that option appended; **When** the user unchecks it, **Then** `onChange` is called with the selection without it, preserving order.
- [X] **Given** the component is controlled, **Then** it holds no selection state of its own: checkboxes reflect `selected`, and a parent that ignores `onChange` leaves them unchanged.
- [X] **Given** an option that is locked with a reason, **Then** its checkbox is checked, disabled, and carries the reason as its tooltip; it cannot be unchecked by mouse or keyboard; it is counted as selected.
- [X] **Given** custom `label`, `dialogLabel`, or `emptyText` props, **Then** the component renders them in place of the defaults.
- [X] **Given** two `LinkPicker` instances on one page with different selections, **Then** each shows its own selection, and toggling in one does not affect the other.

**Closing and the keyboard**
- [X] **Given** the list is open, **When** the user presses Escape, clicks anywhere outside the button and the list, or presses the button again, **Then** the list closes and the selection is unchanged; a pointer event that starts inside the list and ends outside keeps it open.
- [X] **Given** the list opens via keyboard (Enter or Space on the button), **Then** focus moves to the first checkbox; Tab and Shift+Tab cycle through checkboxes; Space toggles the focused checkbox; **When** Escape closes the list, **Then** focus returns to the button.
- [X] **Given** the list opens via mouse, **Then** focus is not moved programmatically.

**Where the list appears**
- [X] **Given** the list is open, **Then** it is rendered in a portal attached to the page root, positioned with `position: fixed`, so that no scrolling or clipping ancestor hides it, and it appears above dialogs.
- [X] **Given** the list is open, **Then** it is 320 px wide, or the window width minus 16 px when that is smaller; it is left-aligned with the button and never closer than 8 px to any window edge.
- [X] **Given** `placement="down"` (the default), **Then** the list opens below the button; it opens above only when fewer than 200 px remain below and more room exists above. **Given** `placement="up"`, **Then** it opens above when at least 160 px are available above or more room exists above than below.
- [X] **Given** the list is open, **Then** it is at most 320 px tall (less when less room is available) and scrolls internally.
- [X] **Given** the list is open, **When** the window is resized or any ancestor scrolls, **Then** the list position is recalculated relative to the button.

**The chip**
- [X] **Given** a `LinkChip` with a label, an icon, and a tone, **Then** it renders a pill with the icon and the label (truncated with "…" when long and the full label as a tooltip), coloured by the tone.
- [X] **Given** `onOpen` is provided, **Then** the label is a button, underlined on hover and on focus. **Given** `onRemove` is provided, **Then** a small × button labelled "Remove link to \<label\>" is shown. **Given** neither is provided, **Then** the label is plain text.

**Free of the builder**
- [X] **Given** a test screen that supplies its own groups ("Colours", "Sizes"), its own selection, and its own chips, **Then** every acceptance test above passes and nothing related to drivers, ADRs, enablers, or stories appears.
- [X] **Given** the source files of `LinkPicker` and `LinkChip`, **Then** they contain no import from the builder (the catalogue, the entries, the ADR rule); a lint rule or import-assertion test fails the build if any such import is introduced (ADR-0016).

## Business rules and constraints
- `LinkPicker` and `LinkChip` are view components in the framework/UI layer (ADR-0007). They render and forward events; they contain no business logic.
- The component is fully controlled: the parent owns the selection and passes it via `selected`; the component calls `onChange` with the complete new selection and never stores a copy (Principle: Declarative over imperative).
- A value is a stable opaque string chosen by the caller; the component never constructs or interprets one.
- The visual appearance follows the existing spike so that screens built on top of it look consistent. Assumption: the spike's CSS class names are available in the shared stylesheet established by AE-0001.
- The portal must be rendered above dialogs; a `z-index` convention consistent with the rest of the UI is assumed to exist or to be established as part of this story.

## Out of scope
- The builder's kinds, headings, and ADR-selection rules (covered by stories 029, 031, and 037).
- Placing the picker inside the file panel or the prompt (covered by stories 031 and 037).
- A search box or collapsible groups within the list.
- A single-select variant or a variant that creates new options.
- Backend changes of any kind.

## Tasks
- [X] Write Jest + React Testing Library tests for the button (label, count, `aria-*` attributes) before writing any implementation.
- [X] Write tests for the list (group order, `fieldset`/`legend` structure, truncation, empty state, `onChange` calls, controlled behaviour, locked options, custom labels, two independent instances).
- [X] Write tests for closing and keyboard behaviour (Escape, outside click, button re-press, Tab/Shift+Tab cycle, Space toggle, focus return, mouse-open focus rule), mocking `getBoundingClientRect`, window dimensions, and scroll events for placement tests.
- [X] Implement `LinkPicker` in `src/ui/components/link-picker/LinkPicker.tsx` with the portal, fixed-position placement logic, resize/scroll listener, and all closing rules.
- [X] Implement `LinkChip` in `src/ui/components/link-picker/LinkChip.tsx` with icon, label, tone, optional `onOpen` button, and optional `onRemove` button.
- [X] Add the import-isolation test (or lint rule) that fails if `LinkPicker` or `LinkChip` import anything from the builder.
- [X] Add the test screen with its own "Colours" and "Sizes" groups, used only in tests, confirming end-to-end wiring with no builder knowledge.
- [ ] Verify manually in a browser: keyboard-only navigation, dark and light themes, positioning near the bottom and right edges of the window. (Open: needs a person in a browser; the keyboard, theme and edge-of-window behaviour is covered by tests that mock the layout, and the chip tones have light-theme colours, but none of it has been seen on screen.)
- [ ] Confirm the full build and test suite completes within the ten-minute budget (Practice: Ten-Minute Build). (Open: the suites that touch this story pass, but a full frontend run takes about 25 minutes on this machine, beyond the ten-minute budget.)

## Files of the spike

In `bwyn-frontend/spikes/home-page/builder/`.

| File | What it holds | Stories |
|---|---|---|
| `LinkPicker.tsx` | `RefChip` (the pill with icon, label, optional open and optional remove) and `LinkPicker` (the button with its count, the portal, the fixed placement with `place()`, the closing rules, the groups, `toggle`). It imports `KIND_ORDER`, `KIND_META` and `isAcceptedAdr`, which is what the stories remove. | **Split in two**: 1 (component), 2 to 4 (rules) |
| `catalog.ts` | `KIND_ORDER` (driver, adr, enabler, story), `KIND_META` (plural heading, icon and tone per kind), `buildEntries` (a story is `story:<id>` with label `<number>. <title>` and folder `['stories', theme?, id]`; an enabler is `enabler:<id>` with `derivesFrom` and `links`). | 2, 3, 4 |
| `generate.ts` | `isAcceptedAdr`. | 2, 3 |
| `FileView.tsx` | The sections "Derives from" (shown when there are ADRs), "Linked to" (the picker labelled "Add link", the chips, the empty line) and "Referenced by". For an enabler the picker's `selected` is `derivesFrom + links` and `onChange` splits the `adr:` refs; the X of a chip calls `onLinksChange(entry.links.filter(...))` with no second argument. The spike shows "Add link" only when `entry.editable`. | 2, 3 |
| `ChatPanel.tsx` | The prompt's "Link" button (`hideKinds` ADR when `asEnabler`, `placement` up or down) and the removable chips above the box (`RefChip` with `onRemove`). It sends `links` without `adr:` refs for an enabler. The cross enablers (`crossRefs`, `autoLinked`, `changeAsEnabler`) are selected here. | 4 |
| `BuilderWorkspace.tsx` | `patchOpen({ links, derivesFrom })`, which calls the store's `patch` for the open story or enabler; `submit`, which passes the links to the request. | 2, 3, 4 |
| `useBuilder.ts` | `patch(kind, id, change)`: replaces `links` and `derivesFrom` and leaves the text alone (for a story it also re-reads tasks only when `content` changes); `removeItem`, which takes a deleted document out of the others' `links`; `removeAdr`, which refuses a used ADR. | 2, 3 |
| `types.ts` | `Entry` (`links`, `derivesFrom`, `kind`, `status`, `label`, `cross`, `editable`), `StoredStory`, `StoredEnabler`. | 2, 3 |
| `repoFiles.ts` | Writes `links` (and `derivesFrom` for an enabler), sorted, in `meta.json`, and reads them back. | 2, 3 |
| `linkedDocs.ts` | `linkedDocumentsText` (the relations "derives from", "linked", and one step through a linked enabler; 12,000 characters). | 2, 3, 4 |
| `git/scope.ts` | The link graph that decides what a sync covers. | 2, 3 |
| `../icons.tsx` | `LinkIcon`, `XIcon`, and the kind icons. | all |

## What the picker looks like in the spike

Tailwind classes, to be kept (the look of the Practices tab is exported in `practices-tab-style.md` in the same way, and the prompt's in
`story-create-story.md`).

| Element | Classes |
|---|---|
| Button | `inline-flex items-center gap-1.5 rounded-full border border-white/10 px-3 py-1.5 text-xs font-medium text-slate-300 transition hover:border-white/20 hover:bg-white/[0.06] focus:outline-none focus-visible:ring-2 focus-visible:ring-indigo-400` (icon size 14) |
| Count | `grid min-w-4 place-items-center rounded-full bg-indigo-500 px-1 text-[10px] font-semibold text-white` |
| List | `z-[60] overflow-hidden rounded-xl border border-white/10 bg-slate-900 shadow-2xl shadow-black/60`, fixed, 320 px; inner `overflow-y-auto p-2` with the max height |
| Group | `fieldset` `mb-1 last:mb-0`; legend `px-2 pb-1 pt-2 text-[11px] font-medium uppercase tracking-wider text-slate-500` |
| Option | `label` `flex cursor-pointer items-center gap-2.5 rounded-lg px-2 py-1.5 text-sm text-slate-200 hover:bg-white/[0.07]`; checkbox `size-4 shrink-0 accent-indigo-400`; text `truncate` |
| Empty | `px-2 py-3 text-sm text-slate-500` |
| Chip | `inline-flex max-w-full items-center gap-1.5 rounded-full py-1 pl-2.5 text-xs font-medium` + the kind's tone, `pr-1` with an X and `pr-2.5` without; X `grid size-4 shrink-0 place-items-center rounded-full hover:bg-white/15` |
| Tones | driver `bg-indigo-500/15 text-indigo-300`, ADR `bg-amber-500/15 text-amber-300`, enabler `bg-emerald-500/15 text-emerald-300`, story `bg-sky-500/15 text-sky-300` |
| Panel section heading | `text-xs font-medium uppercase tracking-wider text-slate-500`; chips list `mt-2 flex flex-wrap gap-1.5`; empty line `mt-2 text-sm text-slate-600` |
