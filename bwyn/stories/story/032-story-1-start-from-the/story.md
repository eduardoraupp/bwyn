# Story: Start from the generic "New" button and choose what to create

- **Story**: 032-story-1-start-from-the
- **Created**: 2026-11-09
- **Status**: Draft

## Story
As a team member, I want one "New" button and one prompt where I choose whether I am writing a story or an architectural enabler, so that I start either from the same place and never look for a different button for each.

## Acceptance tests

**The "New" button**

- [X] **Given** the workspace is open, **Then** the tree header has a round-cornered button with a plus icon whose accessible name and tooltip are "New", and it is the same whether a story or an enabler will be written.
- [X] **Given** a file is open and the user presses "New", **Then** the file is closed, the prompt appears in the middle with earlier messages hidden (not deleted), and the focus moves to the text box.
- [X] **Given** a draft is waiting for approval and the user presses "New", **Then** a confirmation asks "Discard the draft that is waiting for approval?"; confirming drops the draft and opens the prompt; cancelling leaves everything unchanged.
- [X] **Given** the user has typed text, ticked the checkbox, or chosen links, **When** the user presses "New", **Then** the text, the tick, and the links are kept; only the screen and the conversation are reset.
- [X] **Given** the tree folders, **Then** pressing "New" does not open or close any folder.

**The prompt**

- [X] **Given** the prompt with no conversation, **Then** it shows a sparkles icon, the title "Which story shall we tell today?", the line "Describe what a user needs. Claude drafts the story, and once you approve it BWYN creates its folder.", the text box, and three examples in this order: "As a returning customer, I want to sign in with my email and password, so that I can see my orders.", "As a project lead, I want to export the project report as a PDF file.", "Set up continuous integration for the frontend build."
- [X] **Given** the examples are shown and the user presses one, **Then** it fills the box replacing any existing text, the focus stays in the box, and the checkbox state is unchanged.
- [X] **Given** a conversation has started, **Then** the three examples are no longer shown.
- [X] **Given** no file is open, **Then** the prompt occupies the middle of the workspace and shows the title and the line; **Given** a file is open, **Then** the prompt is a narrow column (26 rem) on the right of the file showing only the box, the Link button, and the checkbox, without the title or the line. (Beside an open enabler, the column keeps the heading "What should change?" of story 030, because the box is then aimed at that enabler.)
- [X] **Given** no model is connected, **Then** the same banner and caption that appear on the architectural-enabler screen appear here too.
- [X] **Given** the text box, **Then** its placeholder is "Describe a user story...", it grows with its content up to 200 px and then scrolls, and it has the focus when the workspace opens, when "New" is pressed, and when an example is pressed.

**The checkbox**

- [X] **Given** the prompt for a new document, **Then** below the box, next to the Link button, a checkbox labelled "Mark as architectural enabler" with a zap icon is shown and is not ticked.
- [ ] **Given** the checkbox is not ticked, **Then** the placeholder is "Describe a user story...", the box has a faint indigo border when focused, no ADR panel is shown, the Link button offers ADRs, and the send button is labelled "Draft story". (Open: everything here is done except that the Link button offers ADRs; what the picker lists for a story comes with story 037, and today it offers the drivers and the enablers.)
- [X] **Given** the user ticks the checkbox, **Then** the label turns green, the placeholder becomes "Describe the architectural enabler...", the box gets a green border, a green panel "Derives from an Accepted ADR (required)" appears inside the box, the Link button no longer offers ADRs, and the send button is labelled "Draft architectural enabler".
- [X] **Given** the checkbox is ticked and the user unticks it, **Then** the story prompt is restored; the text in the box is kept; the ADRs that were chosen remain chosen and are reused if the checkbox is ticked again, but they are never sent with a story prompt.
- [X] **Given** the title and the line above the box, **Then** they do not change when the checkbox is ticked or unticked.
- [X] **Given** a draft is on screen or an open enabler is the target of the prompt, **Then** the checkbox and the Link button are not shown.
- [X] **Given** the keyboard, **Then** the checkbox is reachable with Tab, toggled with Space, and shows a visible focus ring; it is a native checkbox inside its label so that pressing the label also toggles it.
- [X] **Given** the user sends the prompt, **Then** the box is emptied, the checkbox is unticked, and the ADRs and links are cleared.

**Sending**

- [X] **Given** the checkbox is not ticked and the box contains text that is not only spaces, **Then** the send button is enabled and labelled "Draft story" regardless of the ADRs and links chosen.
- [ ] **Given** the checkbox is not ticked and the user presses Enter (not Shift+Enter, not while an input method is composing), **Then** a story draft is initiated. (Open: drafting a story comes with story 033. Until then Enter keeps the text and the prompt says "Drafting stories comes in another story. Tick "Mark as architectural enabler" to draft an enabler.")
- [X] **Given** the checkbox is ticked and the user presses Enter, **Then** an enabler draft is initiated.

## Business rules and constraints

- The prompt is one component with one "New" button; what is created depends only on the checkbox state at the moment of sending (ADR-0007, Architecture).
- The default state is a story prompt; the enabler is the exception, chosen by ticking the checkbox.
- A draft cannot change kind; to change it the user must discard the draft and start again.
- The title and the line of the prompt remain unchanged regardless of the checkbox state.
- All interactive elements must be reachable and operable by keyboard (Principle: SOLID — ISP).
- Until stories can be drafted (story 033), sending with the checkbox not ticked changes nothing: the prompt keeps the text and says so, and no request is made.
- What the user typed, ticked and chose survives "New" and the "Try again" of the ADR panel; only loading the workspace, and sending an enabler, reset it.

## Out of scope

- Drafting a story and approving a draft (separate stories).
- Drafting an architectural enabler, the ADR panel, and the rules about required ADRs (enabler stories).
- The Link button and its picker, and the default enablers that stories receive automatically (separate stories).
- Two separate buttons ("New story", "New enabler"), a dropdown menu on "New", and keyboard shortcuts for "New".
- Persisting the text, the tick, or the links across page reloads.

## Tasks

- [X] Rename the tree header button to "New" (accessible name and tooltip) and verify it confirms before discarding a waiting draft, closes any open file, hides earlier messages, and moves focus to the box without touching the box content or the folders.
- [X] Implement the story-default prompt: sparkles icon, title, line, placeholder "Describe a user story...", send button labelled "Draft story", three examples shown only while the conversation is empty, and a growing box capped at 200 px.
- [X] Implement the narrow-column layout (26 rem) when a file is open, hiding the title and the line.
- [X] Add the "Mark as architectural enabler" checkbox with its zap icon and its unticked default appearance.
- [X] Implement the ticked state: green label, placeholder "Describe the architectural enabler...", green border, ADR panel placeholder, Link button without ADRs, send button labelled "Draft architectural enabler"; preserve box text, ADR choices, and links across ticking and unticking.
- [X] Hide the checkbox and the Link button when a draft is on screen or an enabler is the target of the prompt.
- [X] Reset the box, the checkbox, the ADRs, and the links when the prompt is sent.
- [X] Write Jest tests covering: button accessible name; "New" with and without a waiting draft; "New" with typed text (text is kept); examples fill the box and disappear after a conversation starts; checkbox ticked and unticked (placeholder, border, panel, send label, text and ADR preservation); reset after sending; checkbox hidden while a draft is on screen.

# Reference

## Files of the spike

In `bwyn-frontend/spikes/home-page/`. The spike writes stories and enablers with the same prompt, so most files below are shared with
`story-architectural-enablers.md`. The column says which part these stories use.

| File | What it holds | Stories |
|---|---|---|
| `Sections.tsx`, `App.tsx` | The Builder page and "Open the builder", and the menu that shrinks into a rail when the workspace opens. | 1 |
| `builder/FolderTree.tsx` | The tree. Its **header**: "Files" and the project name, and the button **New story** (`aria-label="New story"`, `title="New story"`, a plus icon, `onNew`), to be renamed "New". `stories` (`ROOT_DROP`), its `renderStory` (a `FolderRow` named after the folder, and a `FileRow` for `story.md`), "No stories yet. Write a prompt.", the folder rows and file rows, and the collapsed rail. Its themes, runway and drag and drop are not used here. | 1, 4 |
| `builder/BuilderWorkspace.tsx` | The frame. `newStory` (the "New" button: confirm "Discard the draft that is waiting for approval?", `discard`, close the file, `sessionStart`, `focusSignal`, `onNewStory`), `submit` (the number from `nextNumber(...)` with 3 digits for a story and 4 for an enabler, the `slug`, the path `stories/<number>-<slug>/story.md`, `buildContext` with `{ architecture }` for a story, `buildRequestText` with `asEnabler` false, `throughLinkedEnablers`), `approve` (`extractTitle`, `titleFrom`, `withMarkdownTasks('', markdown, [])`, `addStory`, the two messages, `expand('stories', story-folder)`, `discard`), `perform` and `request` (opening a file). The edit mode, themes, runways, sync and delete are not used here. | 1, 2, 4, 5, 6 |
| `builder/ChatPanel.tsx` | The prompt. `Composer` (the box, the checkbox **Mark as architectural enabler** and its `changeAsEnabler`, `canSend`, `sendLabel`, the placeholders, the green ADR panel, the Link button and its `hideKinds`, the Stop and send buttons), `ChatPanel` (the empty and the conversation layouts, the three `SUGGESTIONS`, the heading "Which story shall we tell today?", the cross enablers that are selected by default with `autoLinked`, the banner and the caption), `MessageView` and `DraftView` (the draft card for a story or an enabler: its icon from `KIND_META[kind]`, path, chip, links, text, tokens, buttons). | 1, 2, 3, 4, 6 |
| `builder/useDraft.ts` | The draft: its four states, `start`, `revise`, `retry`, `stop`, `discard`, `draftText`. | 2, 3 |
| `builder/prompt.ts` | `SYSTEM_PROMPT` (the story rules and the template of a story, then the template of an enabler), `buildContext` (ADRs, links, scopes, principles, practices and, for a story, the architecture), `buildRequestText`, `linksText`, `scopeText`. | 2, 5, 6 |
| `builder/linkedDocs.ts`, `builder/LinkPicker.tsx` | The linked documents in the request (12,000 characters, one step through an enabler), and the Link button and chips. | 6 |
| `builder/storyTasks.ts` | `tasksInMarkdown` (the list under `## Tasks`, with its end rules) and `withMarkdownTasks` (adds the tasks a text gained, without repeats). | 4 |
| `builder/generate.ts` | `makeId`, `today`, `slugify`, `titleFrom`, `nextNumber`, `extractTitle`, `isAcceptedAdr`. | 2, 4 |
| `builder/useBuilder.ts` | The store: `stories`, `messages`, `addStory(story, messages)`. | 4 |
| `builder/types.ts` | `StoredStory` (`id`, `number`, `slug`, `title`, `prompt`, `createdAt`, `content`, `links`, `diagrams`, `ui`, `tasks`, `theme`), `StoryTask`, `Entry`, `ChatMessage`, `Submission`. | 4 |
| `builder/catalog.ts` | `buildEntries`: a story is an `Entry` with ref `story:<id>`, file name `story.md`, folder `['stories', theme?, '<number>-<slug>']`, label `<number>. <title>`; `KIND_META.story` (the `FileTextIcon` and the tone `bg-sky-500/15 text-sky-300`); `entryPath`. | 2, 4 |
| `builder/FileView.tsx` | The panel of an open file. For a story the spike adds the UI example, diagrams, tasks, Sync, Save and Delete, which are not used here. | 4 |
| `builder/MarkdownView.tsx` | The Markdown reader and the rendered/Markdown toggle. | 4 |
| `builder/bedrockClient.ts`, `server/bedrock.js` | The stream client and the dev server. `story-improve-with-ai.md` replaces them with the backend. | 2, 3 |
| `useArchitecture.ts` | The saved text of the architecture, read by the workspace and sent with a story. | 5 |
| `icons.tsx` | `PlusIcon`, `SparklesIcon`, `ZapIcon`, `FileTextIcon`, `LinkIcon`, `ArrowUpIcon`, `StopIcon`, `CheckIcon`, `TrashIcon`, `RotateIcon`, `ArrowRightIcon`, `FolderIcon`, `ChevronDownIcon`, `ChevronRightIcon`. | 1 to 6 |
| `styles.css` | The theme: the animations `fade` and `pop`, the light theme variables, and the rules that keep solid gradient buttons white in light. | all |

## The style

The final implementation should look like the spike. This section is the style of the screens of these stories: the tokens, then the
Tailwind classes of each element. They are copied from `ChatPanel.tsx` and `FolderTree.tsx` (the file panel header from `FileView.tsx`). The
look of the Practices tab, with the plain-CSS version, the light-theme values and the notes on `oklch`, is in `practices-tab-style.md`, and
the link picker's in `story-link-picker.md`. Tailwind 4.3.3 (`@import "tailwindcss";`).

### Tokens (dark, the default)

The page is `bg-slate-950 text-slate-200` with a faint indigo glow at the top right. Light is `:root[data-theme="light"]` in `styles.css`, which
redefines `--color-white` as dark ink and turns the slate scale around.

| Token | Dark value | Used for |
|---|---|---|
| `slate-950` / `slate-900` | `oklch(12.9% 0.042 264.695)` / `oklch(20.8% 0.042 265.755)` | page, panels, popovers |
| `slate-200` / `slate-300` / `slate-400` / `slate-500` / `slate-600` | `oklch(92.9% 0.013 255.508)` / `oklch(86.9% 0.022 252.894)` / `oklch(70.4% 0.04 256.788)` / `oklch(55.4% 0.046 257.417)` / `oklch(44.6% 0.043 257.281)` | text, from strongest to faintest |
| `slate-100` | `oklch(96.8% 0.007 247.896)` | text in the user's message |
| `white` | `#fff` | titles and, at 4 to 15 percent, faint surfaces and lines |
| `indigo-200 / 300 / 400 / 500` | `oklch(87% 0.065 274.039)` / `oklch(78.5% 0.115 274.713)` / `oklch(67.3% 0.182 276.935)` / `oklch(58.5% 0.233 277.117)` | focus rings, the user's message tint, the primary gradient |
| `violet-300 / 500` | `oklch(81.1% 0.111 293.571)` / `oklch(60.6% 0.25 292.717)` | the heading gradient, the primary gradient |
| `pink-300` | `oklch(82.3% 0.12 346.018)` | the heading gradient |
| `emerald-400 / 500` | `oklch(76.5% 0.177 163.223)` / `oklch(69.6% 0.17 162.48)` | the enabler: border, panel, chips, Approve |
| `teal-500` | `oklch(70.4% 0.14 182.503)` | the end of the Approve gradient |
| `sky-300 / 500` | `oklch(82.8% 0.111 230.318)` / `oklch(68.5% 0.169 237.323)` | the story's tile and chips |
| `amber-200 / 300 / 400 / 500` | `oklch(92.4% 0.12 95.746)` / `oklch(87.9% 0.169 91.605)` / `oklch(82.8% 0.189 84.429)` / `oklch(76.9% 0.188 70.08)` | the banner, notes and "Incomplete" |
| `rose-300 / 500` | `oklch(81% 0.117 11.638)` / `oklch(64.5% 0.246 16.439)` | errors and Discard |
| Radius | `xl` 0.75rem, `2xl` 1rem, `3xl` 1.5rem, `full` | rows and buttons, cards, the composer, round buttons |
| Shadow | `lg` `0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1)`; `2xl` `0 25px 50px -12px rgb(0 0 0 / 0.25)` | buttons; the composer |
| Font | `ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, sans-serif`, antialiased; `font-mono` for paths and drafts | |
| Motion | `animate-fade` 0.2 s ease-out on each message; every animation and transition off for `prefers-reduced-motion` | |

The primary gradient is `bg-gradient-to-br from-indigo-500 to-violet-500` (white text). Its twin for approval is `from-emerald-500 to-teal-500`.

### The tree: header, rows

| Element | Classes |
|---|---|
| Tree | `flex h-full w-72 shrink-0 flex-col border-r border-white/5 bg-slate-900/40` (the rail: `w-14 items-center gap-1 py-3`) |
| Header | `flex items-center justify-between gap-2 px-4 pb-2 pt-4`; "Files": `block text-[11px] font-medium uppercase tracking-wider text-slate-500`; project name: `block truncate text-sm font-medium text-slate-200` |
| **"New" button** | `grid size-8 place-items-center rounded-lg bg-white/[0.06] text-slate-300 transition hover:bg-indigo-500 hover:text-white focus:outline-none focus-visible:ring-2 focus-visible:ring-indigo-400`, plus icon 16 |
| List | `flex-1 space-y-0.5 overflow-y-auto px-2 pb-4 pt-1` |
| Folder row | `flex w-full items-center gap-1.5 rounded-lg py-1.5 pr-2 text-left text-sm text-slate-300 transition hover:bg-white/[0.06] focus:outline-none focus-visible:ring-2 focus-visible:ring-indigo-400`; indent `paddingLeft: 8 + depth * 16` px; chevron 14 `text-slate-500`; folder icon 16 `text-indigo-300/80`; name `min-w-0 flex-1 truncate`; count `text-xs text-slate-600`; lock 12 `text-slate-600` |
| File row | `flex w-full items-center gap-2 rounded-lg py-1.5 pr-2 text-left text-sm transition focus:outline-none focus-visible:ring-2 focus-visible:ring-indigo-400`; selected `bg-indigo-500/15 text-white`, else `text-slate-400 hover:bg-white/[0.06] hover:text-slate-100`; icon 15 (`text-indigo-300` selected, else `text-slate-500`); a 14 px spacer where the chevron would be |
| Empty line | `py-1.5 pr-2 text-sm italic text-slate-600` (indented one level) |

### The prompt: the middle, empty

| Element | Classes |
|---|---|
| Wrapper | `flex h-full flex-col items-center justify-center overflow-y-auto px-6 py-10`; inner `w-full max-w-3xl` |
| Icon tile | `grid size-12 place-items-center rounded-2xl bg-gradient-to-br from-indigo-500 to-violet-500 text-white shadow-lg shadow-indigo-500/30`, sparkles 24 |
| Title | `mt-6 bg-gradient-to-r from-indigo-300 via-violet-300 to-pink-300 bg-clip-text font-semibold tracking-tight text-transparent text-4xl` (`text-2xl` when a file is open) |
| Line | `mt-2 text-lg text-slate-500` |
| Composer | in `mt-8` |
| Examples | list `mt-4 flex flex-wrap gap-2`; each `rounded-full border border-white/10 px-3.5 py-2 text-sm text-slate-400 transition hover:border-indigo-400/40 hover:bg-white/[0.05] hover:text-slate-100 focus:outline-none focus-visible:ring-2 focus-visible:ring-indigo-400` |
| Caption | `mt-3 text-center text-xs text-slate-600` |
| Banner | `mb-3 rounded-2xl border border-amber-400/20 bg-amber-500/10 px-4 py-3 text-sm text-amber-200` |
| Column with a file open | `h-full w-[26rem] shrink-0`; without: `h-full min-w-0 flex-1` |

### The composer

| Element | Classes |
|---|---|
| Form | `rounded-3xl border bg-slate-900/80 p-3 shadow-2xl shadow-black/40 backdrop-blur transition focus-within:border-indigo-400/50` + `border-emerald-400/30` (an enabler being created) or `border-white/10` |
| Chips of links (above) | list `mb-2 flex flex-wrap gap-1.5 px-1` |
| Box | `block max-h-52 w-full resize-none bg-transparent px-2 py-2 text-[15px] text-white placeholder:text-slate-500 focus:outline-none`; one row, grows to 200 px |
| ADR panel (ticked) | `mx-1 mb-2 mt-1 rounded-2xl bg-emerald-500/[0.07] p-3`; title `text-xs font-medium text-emerald-300`; chips list `mt-2 flex flex-wrap gap-1.5`; messages `mt-2 text-xs text-amber-300` |
| ADR chip | `rounded-full px-2.5 py-1 text-xs font-medium transition focus:outline-none focus-visible:ring-2 focus-visible:ring-emerald-300` + selected `bg-emerald-400 text-slate-950`, else `bg-white/[0.06] text-slate-300 hover:bg-white/[0.12]` |
| Bottom row | `flex items-center justify-between gap-3`; left `flex min-w-0 flex-wrap items-center gap-2` |
| **Checkbox "Mark as architectural enabler"** | label `inline-flex cursor-pointer items-center gap-2 rounded-full border px-3 py-1.5 text-xs font-medium transition focus-within:ring-2 focus-within:ring-emerald-300` + ticked `border-emerald-400/40 bg-emerald-500/15 text-emerald-300`, else `border-white/10 text-slate-300 hover:border-white/20 hover:bg-white/[0.06]`; input `size-3.5 accent-emerald-400 focus:outline-none`; zap 14 |
| Send | `grid size-10 shrink-0 place-items-center rounded-full bg-gradient-to-br from-indigo-500 to-violet-500 text-white shadow-lg shadow-indigo-500/25 transition enabled:hover:brightness-110 focus:outline-none focus-visible:ring-2 focus-visible:ring-indigo-300 disabled:cursor-not-allowed disabled:from-white/10 disabled:to-white/10 disabled:text-slate-500 disabled:shadow-none`, arrow up 20 |
| Stop | `grid size-10 shrink-0 place-items-center rounded-full bg-white text-slate-900 transition hover:bg-slate-200 focus:outline-none focus-visible:ring-2 focus-visible:ring-indigo-300`, stop 16 filled |
| "Revising the draft below" | `px-2 text-xs text-slate-500` |
| Chip "Editing ..." | `inline-flex max-w-full items-center gap-1.5 rounded-full px-3 py-1.5 text-xs font-medium` + the kind's tone |

### The conversation

| Element | Classes |
|---|---|
| Layout | `flex h-full flex-col`; scroll `flex-1 overflow-y-auto`; inner `mx-auto max-w-3xl space-y-7 px-6 py-8`; composer area `mx-auto w-full max-w-3xl px-6 pb-5` |
| User's message | in `flex justify-end`: `max-w-[85%] whitespace-pre-wrap rounded-3xl rounded-br-lg bg-indigo-500/20 px-5 py-3 text-[15px] text-slate-100` |
| Assistant's message | row `flex gap-3`; avatar `grid size-8 shrink-0 place-items-center rounded-full bg-gradient-to-br from-indigo-500 to-violet-500 text-white` (sparkles 16); column `min-w-0 flex-1 space-y-3 pt-1`; text `whitespace-pre-wrap text-[15px] leading-relaxed text-slate-200` |
| Card that opens the file | `group flex w-full max-w-md items-center gap-3 rounded-2xl border border-white/10 bg-white/[0.04] p-3 text-left transition hover:border-indigo-400/40 hover:bg-white/[0.07] focus:outline-none focus-visible:ring-2 focus-visible:ring-indigo-400`; tile `grid size-9 shrink-0 place-items-center rounded-xl` + tone; path `block truncate font-mono text-sm text-slate-100`; "Open file" `block text-xs text-slate-500`; arrow `text-slate-500 transition group-hover:text-indigo-300` |

### The draft card

| Element | Classes |
|---|---|
| Card | `min-w-0 flex-1 rounded-2xl border border-white/10 bg-white/[0.03]` |
| Header | `flex flex-wrap items-center justify-between gap-2 border-b border-white/5 px-4 py-3`; kind tile `grid size-7 shrink-0 place-items-center rounded-lg` + tone (icon 14); path `truncate font-mono text-xs text-slate-300` |
| Kind tones | story `bg-sky-500/15 text-sky-300` (document icon); enabler `bg-emerald-500/15 text-emerald-300` (zap icon) |
| Status chip | `rounded-full px-2.5 py-1 text-xs font-medium` + Writing... `animate-pulse bg-indigo-500/15 text-indigo-300`; Ready for review `bg-emerald-500/15 text-emerald-300`; Incomplete `bg-amber-500/15 text-amber-300`; Failed `bg-rose-500/15 text-rose-300` |
| Links row | `flex flex-wrap gap-1.5 border-b border-white/5 px-4 py-3` |
| Text | `max-h-[28rem] min-h-16 overflow-auto whitespace-pre-wrap px-4 py-4 font-mono text-[13px] leading-relaxed text-slate-300` |
| Error | `border-t border-white/5 px-4 py-3 text-sm text-rose-300`; incomplete note the same with `text-amber-200` |
| Tokens | list `flex flex-wrap gap-1.5 border-t border-white/5 px-4 py-3 text-[11px]`; chip `rounded-full bg-white/[0.06] px-2 py-0.5 text-slate-400`; cache read above 0 `bg-emerald-500/15 text-emerald-300` |
| Actions | `flex flex-wrap items-center justify-end gap-2 border-t border-white/5 px-4 py-3` |
| Stop (card) | `inline-flex items-center gap-1.5 rounded-xl px-3.5 py-2 text-sm font-medium text-slate-200 transition hover:bg-white/10 focus:outline-none focus-visible:ring-2 focus-visible:ring-indigo-400` |
| Discard | the same with `text-rose-300 hover:bg-rose-500/10` and `focus-visible:ring-rose-400` |
| Retry | `inline-flex items-center gap-1.5 rounded-xl bg-white/10 px-3.5 py-2 text-sm font-semibold text-white transition hover:bg-white/15 focus:outline-none focus-visible:ring-2 focus-visible:ring-indigo-400` |
| Approve | `inline-flex items-center gap-1.5 rounded-xl bg-gradient-to-br from-emerald-500 to-teal-500 px-3.5 py-2 text-sm font-semibold text-white shadow-lg shadow-emerald-500/20 transition enabled:hover:brightness-110 focus:outline-none focus-visible:ring-2 focus-visible:ring-emerald-300 disabled:cursor-not-allowed disabled:opacity-40 disabled:shadow-none` |

### The file panel of an open story

| Element | Classes |
|---|---|
| Panel | `min-w-0 flex-1 border-r border-white/5 bg-slate-950/40`, inside `flex h-full min-w-0 flex-col` |
| Header | `flex shrink-0 items-start justify-between gap-4 border-b border-white/5 px-6 py-4`; tile `grid size-10 shrink-0 place-items-center rounded-xl` + story tone (icon 20); path `truncate font-mono text-sm font-semibold text-white`; note `mt-0.5 flex items-center gap-1.5 text-xs text-slate-500` with a lock 12 |
| Close | `grid size-9 place-items-center rounded-xl text-slate-400 transition hover:bg-white/10 hover:text-white focus:outline-none focus-visible:ring-2 focus-visible:ring-indigo-400`, X 18 |
| Body | `flex-1 space-y-5 overflow-y-auto px-6 py-5`; section headings `text-xs font-medium uppercase tracking-wider text-slate-500` |
