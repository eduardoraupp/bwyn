# Story: Rename a theme

- **Story**: 043-story-2-rename-a-theme
- **Created**: 2026-11-15
- **Status**: Draft

## Story
As a team member, I want to rename a theme when I find a better name for it, so that the name says what its stories have in common, without moving every story by hand.

## Acceptance tests

**Starting and ending a rename**
- [ ] **Given** a theme in the tree, **When** the user double-clicks it, **Then** its name becomes an edit field with the whole text selected, and the folder is open or closed as it was before the double-click.
- [ ] **Given** a theme has the focus, **When** the user presses F2, **Then** its name becomes an edit field, as with a double-click.
- [ ] **Given** a theme row, **When** the pointer rests on it, **Then** the tooltip reads "Click twice (or press F2) to rename".
- [ ] **Given** the edit field is open, **When** the user presses Enter or leaves the field, **Then** the new name is saved; **When** the user presses Escape, **Then** nothing changes and the field closes.
- [ ] **Given** the edit field is open, **When** the user keeps the same name and presses Enter, **Then** nothing changes and the field closes.
- [ ] **Given** a story folder, the `stories` folder, another folder, or a file, **When** the user double-clicks it or presses F2, **Then** nothing is renamed.

**The name**
- [ ] **Given** the user types `"  Cobrança Online  "`, **When** Enter, **Then** the theme is named `cobranca-online`: leading and trailing spaces are removed, words are lower-cased and joined by `-`, accents are stripped, and any other non-letter non-digit character becomes `-`.
- [ ] **Given** the user clears the name or types only spaces or non-letter characters, **When** Enter, **Then** the message "Give the theme a name." appears under the field, the field stays open, and nothing is saved.
- [ ] **Given** the user types `"2024-goals"`, **When** Enter, **Then** the message "A theme name starts with a letter." appears and nothing is saved.
- [ ] **Given** a theme `checkout-flow` already exists, **When** another theme is renamed `"Checkout Flow"`, **When** Enter, **Then** the message "A theme with this name already exists." appears and nothing is saved.
- [ ] **Given** a validation message is shown under the field, **When** the user types again, **Then** the message disappears.
- [ ] **Given** the edit field contains an invalid name, **When** the user leaves the field (blur), **Then** the theme reverts to its old name and no message remains.
- [ ] **Given** the user types a name longer than 40 characters, **Then** the value is truncated to 40 characters, with no `-` at the end.

**What moves with it**
- [ ] **Given** a theme `checkout-flow` containing stories `003-checkout-form` and `007-pay-by-card`, **When** it is renamed `payments`, **Then** both story folders are inside `payments`, and their identities, texts, statuses, links, tasks, diagrams, and UI examples are unchanged.
- [ ] **Given** the currently open story is inside the theme being renamed, **When** the rename is saved, **Then** the story remains open and its displayed path reads `stories/payments/003-checkout-form/story.md`.
- [ ] **Given** the theme was open before the rename, **When** the rename is saved, **Then** the theme is still open under its new name and appears in its correct alphabetical position.
- [ ] **Given** the theme has a saved runway, **When** the theme is renamed, **Then** the runway is associated with the theme under its new name.

## Business rules and constraints
- Keeping the same name is allowed; the uniqueness check excludes the theme being renamed.
- A rename does not alter the text, status, tasks, links, diagrams, or UI examples of any story inside the theme.
- In the repository, each story moves to its new path only when it is synchronised (a later story); a rename does not open a pull request on its own.
- The slug transformation is: trim whitespace, lower-case, strip diacritics, replace any character that is not a letter or digit with `-`, collapse consecutive `-` into one, remove a trailing `-`, and truncate to 40 characters.
- A valid slug must start with a letter (after transformation).

## Out of scope
- Undoing a rename.
- Renaming story folders, the `stories` root folder, or individual files from the tree.
- Synchronising every story of a renamed theme in a single pull request.
- Backend persistence changes: theme membership is stored in the frontend state established by story 042; no new API endpoints are introduced here.

## Tasks
- [ ] Add double-click and F2 handlers to the theme row component that open an inline name field pre-filled with the current name and fully selected (Architecture: UI layer)
- [ ] Add a tooltip `"Click twice (or press F2) to rename"` to the theme row (Architecture: UI layer)
- [ ] Implement the slug-transformation function (trim, lower-case, strip diacritics, non-alphanumeric → `-`, collapse, truncate to 40) in the entities layer and unit-test it (Architecture: entities layer)
- [ ] Implement validation rules (non-empty after transformation, must start with a letter, must be unique excluding self) in the use-case layer and unit-test each rule (Architecture: use-cases layer)
- [ ] Implement a `RenameThemeViewModel` that holds edit state, exposes the validation message, and calls the rename use case; unit-test it with a test double for the use case (Architecture: interface-adapters layer, ADR-0007)
- [ ] Wire the rename use case: update the theme name in the project state, reassign the theme field of every story in that theme, and update the runway association (Architecture: use-cases layer)
- [ ] Update the tree to keep the renamed theme open and re-sort it alphabetically after the rename (Architecture: UI layer)
- [ ] Update the open-story path display when the story's theme is renamed (Architecture: UI layer)
- [ ] Write a Jest integration test covering: rename `checkout-flow` to `payments` with two stories (one open), verify story contents are unchanged, the open story path updates, the theme is open in its new alphabetical position, and the runway is reassigned (Practice: Pair programming, ADR-0005)
- [ ] Write Jest tests for each validation message and for the blur-reverts-to-old-name behaviour (Practice: Pair programming)
- [ ] Verify the full build and test suite completes within the ten-minute budget (Practice: Ten-Minute Build)
