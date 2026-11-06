# Story: Edit an ADR and change its status

- **Story**: 016-story-3-edit-the-text
- **Created**: 2026-11-06
- **Status**: Draft

## Story
As a team member, I want to open an ADR and correct or complete its text and status, with the same help from the AI, so that the decision log says what we really decided and reflects whether the decision is still current.

## Acceptance tests

**Opening**
- [ ] **Given** a row in the ADR list, **When** the user opens it, **Then** a dialog opens titled with the ADR's title, showing its text as plain text in the box, its status selected in a selector, and Save disabled.
- [ ] **Given** an ADR that nothing uses, **Then** the note says "Not used by any story or enabler. It stays here, and goes to GitHub the first time together with the story or the enabler that uses it."
- [ ] **Given** an ADR that stories or enablers use, **Then** the note, in blue, says "Used by N documents. The first time, it goes to GitHub with them when you sync. After that, saving an edit opens a pull request for it." ("1 document" when there is one).
- [ ] **Given** the dialog is open, **Then** the AI is given the project name and every ADR except this one, and the ADR as it is in the box.

**Status selector**
- [ ] **Given** the dialog is open, **Then** a Status selector shows the current status of the ADR, with the options Proposed, Accepted, Deprecated, and Superseded.
- [ ] **Given** an ADR whose status is none of those four (for example "Unknown"), **Then** that status is also in the selector, first, and is kept until the user chooses another.
- [ ] **Given** any status, **Then** the user can choose any other; no transition is blocked.

**Saving**
- [ ] **Given** only the text changed, **When** the user presses Save, **Then** the Markdown is rebuilt with the stored date, the title, status, and date of the row are re-read from it, the dialog closes, and the row shows them.
- [ ] **Given** only the status changed, **When** Save, **Then** only the Status line of the stored Markdown changes (or is added under the title when there was none); the rest of the text and the date stay byte for byte as they were.
- [ ] **Given** both the text and the status changed, **When** Save, **Then** the Markdown is rebuilt with the new status and the stored date.
- [ ] **Given** the title (first line) was changed, **Then** the row shows the new title, and the number and the file name do not change.
- [ ] **Given** the first line was emptied, **When** Save, **Then** a validation message appears and nothing is stored.
- [ ] **Given** nothing changed, **Then** Save is disabled.
- [ ] **Given** the dialog is open, **Then** Escape, Close (X), Cancel, and a click outside close it without saving anything.
- [ ] **Given** a status was saved, **Then** the row shows its chip in the new colour, in the same place in the list.

**No removal**
- [ ] **Given** any ADR in any status, **Then** the list and the dialog have no trash button, no "Remove" option, and no confirmation dialog; there is no way to delete an ADR.
- [ ] **Given** a Deprecated or Superseded ADR, **Then** it stays in the list with its text.

**Improve with AI**
- [ ] **Given** the dialog is open, **Then** the same AI improvement panel with the same five quick actions works as in story 007, and "Use this text" replaces the text (and the status, when the proposal contains one) in the box without saving.

**After saving — repository**
- [ ] **Given** GitHub is not connected, **Then** saving shows no notice.
- [ ] **Given** GitHub is connected and no story or enabler uses the ADR, **Then** a notice says "ADR NNNN was saved here. It reaches GitHub the first time together with the story or the enabler that uses it."
- [ ] **Given** GitHub is connected and the ADR is used, **Then** a notice says "Opening a pull request for ADR NNNN…", and then:
  - **When** it is on the base branch: "Pull request #N opened for ADR NNNN. Merge it on GitHub." with a link "Open it".
  - **When** it never reached GitHub: "ADR NNNN was saved here. It is not on GitHub yet, so it goes with the story or the enabler that uses it, the next time you sync it."
  - **When** the base branch changed the same ADR: a warning "ADR NNNN was also changed on \<branch\>, so nothing was uploaded. Solve it on GitHub, then sync."
  - **When** it is the same as on the base branch, or already in an open pull request: "ADR NNNN is already the same on GitHub, or already waiting in an open pull request."
  - **When** the request fails: an error "ADR NNNN was saved here, but the pull request could not be opened: \<reason\>."
- [ ] **Given** a notice is shown, **Then** it can be dismissed with its X.

## Business rules and constraints
- Saving never commits an ADR to GitHub for the first time. That happens with the story or the enabler that uses it when it is synced. Only an ADR that is already on GitHub and used by something goes up by itself, in a pull request, compared with the base branch and with what was last synchronised; a conflict uploads nothing.
- The number, the file name, and the date of an ADR do not change when its text or status is edited.
- A status-only change rewrites only the Status line; all other bytes of the Markdown are unchanged.
- The status is one of Proposed, Accepted, Deprecated, and Superseded. A status read from the repository that is none of these is kept as-is until the user chooses another.
- An ADR is never removed, so its number is never reused. The next number is always the highest existing number plus one (ADR-0017).
- Notices carry a tone (info, success, warning, error), may carry a link, and can be dismissed.
- Only an Accepted ADR can be newly linked to a story or enabler in the Builder. An ADR of any other status that is already linked keeps its link and still appears in the Builder so it can be unlinked. All ADRs, regardless of status, are sent to the model as project context (ADR-0007, ADR-0017).

## Out of scope
- "Superseded by NNNN": choosing which ADR supersedes another and the link between the two.
- Changing the sync engine and the pull-request flows of stories and enablers.
- Locking the text of an Accepted ADR.
- Removing an ADR, restoring one, or hiding ADRs by status.
- Warning about stories already linked to an ADR that stopped being Accepted.
- The backend `/improve` endpoint and streaming protocol (covered by story 007).
- Connecting GitHub and the sync dialog (covered by a separate story).

## Tasks
- [ ] Open the edit dialog from a row, pre-filled with the ADR's text, status selector, and used-by note, with Save disabled until something changes (ADR-0007)
- [ ] Add the Status selector with the four statuses plus any unrecognised status kept as the first option (ADR-0007)
- [ ] Wire the AI context: pass the project name and every ADR except the one being edited (ADR-0007)
- [ ] Implement text-only save: rebuild the full Markdown with the stored date, re-read title, status, and date, keep number and file name (ADR-0017)
- [ ] Implement status-only save: rewrite only the Status line (or insert it under the title when absent), leaving all other bytes unchanged (ADR-0017)
- [ ] Implement combined text-and-status save: rebuild Markdown with the new status and stored date (ADR-0017)
- [ ] Colour the status chip on the row for each of the four statuses, with a neutral colour for any other value (ADR-0007)
- [ ] Remove all removal affordances: delete the confirmation dialog, `removeAdr` from the builder store, `deleteFromRepo` for ADRs, and the removed-numbers list; derive the next number from the highest existing number only (ADR-0017)
- [ ] Implement the five repository notices (pull request opened, not on GitHub, conflict, unchanged/already open, error) with link and dismiss button (ADR-0007)
- [ ] Implement `publishAdr`: open a pull request for an ADR that is on GitHub and used, and record it for the next sync (ADR-0017)
- [ ] Write unit tests: status-only save keeps Markdown byte-for-byte; unknown status is preserved; no removal path exists; text-and-status save rebuilds correctly (Practice: Pair programming, Practice: Continuous integration)
- [ ] Write an integration test with a fake repository: edit of a used ADR on the base branch, one that never reached GitHub, and a conflict (Practice: Continuous integration)

Files of the spike

In `bwyn-frontend/spikes/home-page/`:

| File | What it holds | Stories |
|---|---|---|
| `data.ts` | The menu entry `adr` (label "ADR", title "Architecture decisions", `BookOpenIcon`). Its description in the spike is stale (see below). | 1 |
| `Sections.tsx` | `case 'adr'` mounts `AdrPage`. | 1 |
| `AdrPage.tsx` | The whole screen. `AdrPage`: the list, the empty state, the card, `usedBy` (stories and enablers that use an ADR), the notices (`Notice`, `NOTICE_STYLE`), `publish` (what to say after a save, and the pull request), and the opening of the dialogs (`Dialog`: new or edit). `EditAdrDialog`: the one dialog for add and edit (plain text, status, headings, `dirty`, the used-by note, `save` and the status-only shortcut, `replaceWith` for the AI). `Shell`: the modal frame and its closing rules. `RemoveAdrDialog` and `deleteAdr`: the removal that these stories take out. | 1, 2, 3, 4 |
| `adrText.ts` | `viewOf` (Markdown to plain text, status, date and the marks of the sections) and `composeAdr` (plain text back to Markdown, null without a title). | 2, 3 |
| `builder/generate.ts` | `nextNumber`, `slugify`, `parseAdr` (reads a file: number, title, status, date), `adrFromMarkdown` (new ADR, file name from number and title), `withAdrStatus` (rewrites only the Status line), `ADR_STATUSES`, `isAcceptedAdr`. | 2, 3, 4 |
| `builder/useBuilder.ts` | The ADRs of a project in the browser: `addAdr` (sorted by number), `saveAdr` (reads title, status and date again; keeps the file name), `removeAdr` and `removedAdrs`. | 1, 2, 3, 4 |
| `builder/types.ts` | `StoredAdr` and `ProjectBuilder.adrs` / `removedAdrs`. | 1 |
| `AdrAssistant.tsx` | The AI of the ADR dialog, **built apart from the shared component**: the "Generate from template with AI" button, the "Suggested ADR" card (`Suggestion`, readable text through `viewOf`), "Replace my text with this", Stop, Retry, tokens. It is what the task of story 2 replaces. | 2 |
| `adrPrompts.ts` | `ADR_SYSTEM_PROMPT` (template and rules), `ADR_REPLY_MARKER` (`---ADR---`), `buildAdrContext` (project name and the other ADRs), `buildAdrTurn` (number, title, date, text, instruction) and `ADR_QUICK_ACTIONS` (four). | 2, 3 |
| `DriverAssistant.tsx`, `useDriverAssistant.ts`, `driverPrompts.ts` | **The shared "Improve with AI" component**, its conversation hook and `parseReply`. Used today by drivers, principles and the architecture. See the first task of story 2 for what it lacks for ADRs. | 2, 3 |
| `builder/bedrockClient.ts`, `server/bedrock.js` | Today's call to the model, through the dev server. `story-improve-with-ai.md` replaces it with the backend REST API. | 2, 3 |
| `builder/git/publishAdr.ts` | Sends one saved ADR as a pull request, compared with the base branch and with what was last synchronised: `pr`, `waiting`, `conflict` or `unchanged`. | 3, 4 |
| `builder/git/useGitState.ts` | The connection and the record of what was synchronised, used by the page for `publishAdr`. | 3, 4 |
| `builder/git/deleteFromRepo.ts` | Deletes an ADR file in a pull request (`kind: 'adr'`). **Removed by these stories** for ADRs. | 4 |
| `builder/catalog.ts` | Makes each ADR a read-only `adr:<number>` entry (`adrs/<file>`) for the Builder. | 4 |
| `builder/LinkPicker.tsx`, `builder/ChatPanel.tsx` | Offer only Accepted ADRs (`isAcceptedAdr`); one already linked stays visible. | 4 |
| `builder/repoFiles.ts`, `builder/git/scope.ts`, `builder/git/syncEngine.ts` | Reading `adrs/*.md` back (`parsePulled`, `mergePulled`), the scope `adrs/<number>-`, and "an ADR goes up only when something links to it". `removedAdrs` is used here too. | 1, 3, 4 |
| `icons.tsx` | `BookOpenIcon`, `PencilIcon`, `PlusIcon`, `GitBranchIcon`, `CheckIcon`, `XIcon`. | 1 to 4 |

## Removal in the spike

The spike lets the user remove an ADR that nothing uses (trash button, `RemoveAdrDialog`, `deleteAdr`, `removeAdr`), by a pull request that
deletes the file, and remembers the number in `removedAdrs` so that it is not reused and the file is not brought back. These stories say an
ADR is never removed, so the following goes away: the trash button, `RemoveAdrDialog` and `deleteAdr` in `AdrPage.tsx`; `removeAdr` in
`useBuilder.ts`; the `'adr'` kind of `deleteFromRepo.ts`; and `removedAdrs`, which `AdrPage`, `ArchitecturePage`, `Drivers`,
`PrinciplesPractices`, `BuilderWorkspace`, `useGitSync`, `syncEngine` and `mergePulled` carry only to keep a removed ADR out.

## Spike details worth keeping in mind

- The ADR menu description in `data.ts` still says "Decisions are recorded elsewhere, so this view is read-only.", which is false. The
  stories use "The decision log of this project." from `story-navigation-and-menu-bar.md`.
- The AI today is `AdrAssistant`, not the shared component, and differs from it: a button "Generate from template with AI" (two
  instructions, with and without text), a "Suggested ADR" card as readable text, and "Replace my text with this". The shared component
  calls `parseReply(text)` with the driver's marker, builds its turn with `buildDriverTurn(name, hint, ...)`, shows the proposal as raw text
  and compares it to the box as raw text, and has the label "Ask Claude to change this driver". That is why story 2 has a task to give it
  what an ADR needs before the dialog can use it.
- `buildAdrTurn` does not send the status, and the box holds plain text without the Status line, so the model cannot tell that an ADR is
  Accepted. The rule in the system prompt ("An ADR is immutable once Accepted ...") has nothing to act on.
- "Use this text" can change the status selector: `replaceWith` takes the status of the proposal when it has one.
- A new ADR takes today's date from `new Date().toISOString()`, which is the UTC date, not the local one.
- Escape closes the dialog even with unsaved text, while a click outside closes it only when nothing changed. The tests follow the spike.
- `saveAdr` reads the ADR again with the stored file name, so the number and the file name never follow an edited title.
- When only the status changed, `EditAdrDialog` keeps the stored Markdown and rewrites just the Status line (`withAdrStatus`). Any text
  change rebuilds the whole Markdown from the plain text (`composeAdr`).
- Everything is in the browser (`localStorage`, the key of the builder store). Two browsers do not share ADRs until the repository does.
