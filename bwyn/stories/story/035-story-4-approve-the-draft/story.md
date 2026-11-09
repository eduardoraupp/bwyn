# Story: Approve the draft and find the story in the tree

- **Story**: 035-story-4-approve-the-draft
- **Created**: 2026-11-09
- **Status**: Draft

## Story
As a team member, I want to approve a ready draft so that it becomes a story of my project and appears in its own folder in the tree, so that my team can read it and build it.

## Acceptance tests

**Approving**

- [ ] **Given** a Ready draft of a story, **Then** the card shows "Discard" and "Approve and create story", and "Approve and create story" is enabled; **Given** any other draft state, **Then** "Approve and create story" is disabled.
- [ ] **Given** a Ready draft, **When** the user presses "Approve and create story", **Then** the story is stored with: a sequential number, a slug derived from the first words of the title, the title, the user's first prompt, today's date, the Markdown as drafted (trimmed, with one final newline), the links, and its tasks. It has no theme.
- [ ] **Given** the Markdown, **Then** the title is read from its first `# ` line with the prefixes "Story:", "Feature specification:" and "AE-`<digits>`." stripped; **Given** no such line, **Then** the title is the first sentence of the prompt, truncated to 67 characters and followed by "..." when longer than 70, with the first letter capitalised.
- [ ] **Given** approval, **Then** two messages are added: the user's original prompt, and "Approved. I created the folder `stories/001-as-a-returning-customer-i/` with `story.md`." with " It is linked to 2 other items." appended when there are two links, or "1 other item" for one, and a card showing the path `stories/001-as-a-returning-customer-i/story.md` and an "Open file" action.
- [ ] **Given** approval, **Then** the draft is gone, the prompt box is empty and unticked, and the folders `stories` and `001-as-a-returning-customer-i` are open in the tree.
- [ ] **Given** a second approved story, **Then** its number is `002`, even when its first words are the same as the first story's.

**The tasks of the story**

- [ ] **Given** the Markdown contains a `## Tasks` section, **Then** the story is stored with one task per non-indented list item (bulleted with `-`, `*`, or `+`, or numbered with `1.` or `1)`), with or without a checkbox, in order; the list ends at a horizontal rule, the next heading, or the end of the text.
- [ ] **Given** a list item that is only an empty checkbox (`- [ ]`), or an item indented under another, **Then** it is not stored as a task.
- [ ] **Given** two items with the same text (ignoring case, extra spaces, and trailing punctuation), **Then** only the first is kept.
- [ ] **Given** the Markdown has no `## Tasks` section, **Then** the story is stored with no tasks.
- [ ] **Given** stored tasks, **Then** each has its own identifier and its text, and the Markdown of the story is unchanged.

**The tree**

- [ ] **Given** an approved story, **Then** the `stories` folder lists a folder named `001-as-a-returning-customer-i`, ordered by number, with the folder count updated, and opening it shows `story.md` whose tooltip is `1. <title>`.
- [ ] **Given** no approved story, **Then** the open `stories` folder shows the italic line "No stories yet. Write a prompt."
- [ ] **Given** stories in themes, **Then** a story approved here starts outside any theme, among the top-level folders of `stories`.

**Reading it**

- [ ] **Given** the "Open file" card in the conversation, or `story.md` in the tree, **When** the user activates it, **Then** the file opens in the same panel used for enablers, drivers, and ADRs: the story icon, the path `stories/001-.../story.md`, a Close control, rendered and Markdown views, the tree as a thin rail, and the prompt as a narrow column on the right.
- [ ] **Given** the open story panel, **Then** a lock indicator and the message "Read-only here. Changing a story comes in another story." are shown, and no Save, Delete, Sync, "Add task", diagram, UI example, or "Add link" control is present.
- [ ] **Given** the open story panel, **Then** its links appear under "Linked to" and documents that reference it appear under "Referenced by".

**Discarding and keeping**

- [ ] **Given** a draft on screen, **When** the user presses "Discard", **Then** nothing is stored and the draft is removed.
- [ ] **Given** a draft on screen, **When** the user presses "New", **Then** a confirmation is shown before the draft is discarded.
- [ ] **Given** the page is reloaded, **Then** all approved stories, their tasks, their links, and the conversation are still present.
- [ ] **Given** two projects, **Then** each project keeps its own stories, tasks, links, and conversation independently.

## Business rules and constraints

- Only a Ready draft may be approved; approving is always the user's explicit action and the AI never stores anything autonomously (Practice: Pair programming).
- A story's identity is its folder name `<number>-<slug>`, and the reference to it is `story:<number>-<slug>`; it derives from the title and never from a random identifier, so that a link means the same on every machine. It does not include a theme, so moving a story never breaks a link.
- The title is read from the document so that it can be corrected in the text later without re-approving.
- Tasks are copied once from `## Tasks` at approval time and become the story's own list from that point on.
- Storage in this story is `localStorage` per project; the final persistence mechanism is a separate decision.
- A story has no ADRs it derives from; an ADR it uses is an ordinary link.
- The "No stories yet." placeholder replaces the earlier "Coming in another story." text (Architecture: UI layer).

## Out of scope

- Editing the story by hand or by prompt after approval.
- Showing, adding, removing, and completing tasks.
- Diagrams, the UI example, deleting a story, moving it into a theme, and synchronising it (which writes `stories/001-.../story.md`, `meta.json`, `ui.html`, and diagram files).
- Creating a story without the AI.
- Importing stories from the repository (they appear in the tree through the sync story).
- The link picker and "Add link" control (covered by story 031).

## Tasks

- [ ] Add `addStory` to the project store: persist the story record, its links, its tasks, and the two approval messages together in `localStorage` (Architecture: infrastructure layer).
- [ ] Implement title extraction from the first `# ` line of the Markdown, stripping "Story:", "Feature specification:", and "AE-`<digits>`." prefixes, with the prompt-sentence fallback (Architecture: use-case layer).
- [ ] Implement task extraction from `## Tasks`: parse non-indented list items, skip empty checkboxes and indented items, stop at a horizontal rule or the next heading, and deduplicate by normalised text (Architecture: use-case layer).
- [ ] Build the approval confirmation message and the "Open file" card, including the link-count suffix (Architecture: interface-adapter layer).
- [ ] Enable "Approve and create story" only when the draft state is Ready; keep it disabled for all other states (Architecture: UI layer).
- [ ] Update the `stories` folder in the tree: list story folders in number order, show `story.md` inside each, update the folder count, open `stories` and the new folder after approval, and replace the placeholder text with "No stories yet. Write a prompt." when empty (Architecture: UI layer).
- [ ] Implement the read-only story file panel: story icon, path, Close, rendered and Markdown views, tree rail, prompt column, lock indicator with read-only message, and no editing controls (Architecture: UI layer).
- [ ] Wire the "Open file" card in the conversation and the `story.md` node in the tree to open the read-only panel (Architecture: UI layer).
- [ ] Write Jest tests covering: approval happy path, title extraction rules, task extraction rules (list styles, empty checkboxes, indented items, horizontal-rule termination, duplicates, no-Tasks section), second story numbering, tree state after approval, page-reload persistence, two-project isolation, and discard behaviour (Practice: Ten-Minute Build).
