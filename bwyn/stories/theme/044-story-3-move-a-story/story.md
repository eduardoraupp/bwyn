# Story: Move a story into a theme by drag and drop

- **Story**: 044
- **Created**: 2026-11-15
- **Status**: Draft

## Story
As a team member, I want to drag a story folder into a theme, out of it, or into another theme, so that I decide myself which stories sit together.

## Acceptance tests

**Dragging and dropping**

- [ ] **Given** the story folder `003-checkout-form` is outside any theme and the theme `checkout-flow` exists, **When** the user drags `003-checkout-form` onto `checkout-flow` and drops it, **Then** `003-checkout-form` appears inside `checkout-flow` and no longer outside it, the count of `checkout-flow` increases by one, the total count of `stories` is unchanged, and `checkout-flow` opens.
- [ ] **Given** a story folder is being dragged, **When** the pointer moves over a theme row or the `stories` row, **Then** that row is highlighted as a valid drop target.
- [ ] **Given** `003-checkout-form` is inside `checkout-flow`, **When** the user drops it on the `stories` row, **Then** `003-checkout-form` is outside any theme, positioned in ascending number order among the other unthemed stories.
- [ ] **Given** `003-checkout-form` is inside theme `checkout-flow`, **When** the user drops it on theme `reporting`, **Then** `003-checkout-form` is inside `reporting` and is no longer inside `checkout-flow`.
- [ ] **Given** `003-checkout-form` is inside `checkout-flow` and is being dragged, **When** the pointer moves over `checkout-flow`, **Then** `checkout-flow` is not highlighted and dropping there changes nothing.
- [ ] **Given** `003-checkout-form` is outside any theme and is being dragged, **When** the pointer moves over the `stories` row, **Then** the `stories` row is not highlighted and dropping there changes nothing.
- [ ] **Given** a story folder is being dragged, **When** the user presses Escape, **Then** the story does not move and no highlight remains.
- [ ] **Given** a story folder is being dragged, **When** it is dropped on a file, a driver, an ADR, or an architectural enabler, **Then** the story does not move and no highlight remains.

**What can be dragged**

- [ ] **Given** the builder tree, **Then** only story folders have a drag handle; theme rows, individual files, and all items under `drivers`, `adrs`, and `architectural-enablers` do not.
- [ ] **Given** a theme row is dragged, **Then** it cannot be dropped onto another theme, so themes never nest.

**What a move keeps**

- [ ] **Given** `003-checkout-form` is moved into `checkout-flow`, **Then** its identifier, text, status, links (both directions), tasks, diagrams, and UI example are unchanged.
- [ ] **Given** `003-checkout-form` was open before the move, **Then** it remains open after the move, showing its new path.
- [ ] **Given** stories inside a theme, **Then** they are ordered by their number in ascending order.
- [ ] **Given** a link that targets `003-checkout-form` which is inside the closed theme `checkout-flow`, **When** the link is followed, **Then** `stories`, `checkout-flow`, and `003-checkout-form` open, and `003-checkout-form` is selected.
- [ ] **Given** `003-checkout-form` is inside `checkout-flow`, **Then** the path shown in the builder and the "File:" line of every prompt about it is `stories/checkout-flow/003-checkout-form/story.md`.
- [ ] **Given** stories have been moved, **When** the page is reloaded, **Then** each story is in the theme it was moved to.

**Persistence**

- [ ] **Given** `003-checkout-form` is dropped onto `checkout-flow`, **When** the drop completes, **Then** the backend records `003-checkout-form` in the list of story identifiers that belong to `checkout-flow`, and the previous theme (if any) no longer lists `003-checkout-form`.
- [ ] **Given** `003-checkout-form` is dropped onto the `stories` row, **When** the drop completes, **Then** the backend records `003-checkout-form` as belonging to no theme.
- [ ] **Given** the backend call to persist the move fails, **When** the drop completes, **Then** the tree reverts to its pre-drop state and an error message is shown; no partial state is persisted.
- [ ] **Given** the page is reloaded after a move, **When** the builder tree loads, **Then** the frontend reads the theme membership from the backend and each story appears in the theme the backend recorded.

## Business rules and constraints

- A story belongs to at most one theme. Moving it into a theme removes it from the theme it was in (ADR-0007, Architecture).
- A move does not alter the story's text, status, identifier, or links; it changes only the folder path (Architecture).
- Dropping a story where it already lives is not a valid drop target; the tree never offers a move that does nothing.
- Story folders inside a theme are ordered by their number in ascending order (consistent with story 042).
- Theme membership is persisted in the backend so that it survives a page reload; the theme record holds the ordered list of story identifiers that belong to it (Architecture).
- The path `stories/<theme>/<identity>/story.md` is the canonical path for a story that belongs to a theme; `stories/<identity>/story.md` is the canonical path for an unthemed story (Architecture).
- The frontend optimistically updates the tree on drop and reverts if the backend call fails (Architecture).

## Out of scope

- Moving a story with the keyboard or on a touch screen; drag and drop with a mouse or pen only.
- Moving several stories at once.
- Reordering stories by hand within a theme.
- Themes inside themes; a theme cannot be a drop target for another theme.
- Renaming a theme (covered by story 043).
- Removing a theme and deciding what happens to its stories.
- Synchronising the new path with the Git repository (a separate sync story).

## Tasks

- [ ] Add a `PATCH /projects/{projectId}/themes/{themeId}/stories` endpoint (or equivalent) in the inbound adapter that accepts a story identifier and records the move; add a corresponding `PATCH` to remove a story from all themes when it is moved to the `stories` root (ADR-0017, Architecture)
- [ ] Implement the application-layer command handler that updates the theme aggregate: adds the story identifier to the destination theme's list and removes it from the source theme's list (ADR-0017)
- [ ] Add or extend the outbound adapter and Flyway migration so that the theme record stores the ordered list of story identifiers that belong to it (ADR-0010, ADR-0011)
- [ ] Extend the story data model and its store to record the theme a story belongs to, and derive its folder path as `stories/<theme>/<identity>` or `stories/<identity>` accordingly (ADR-0007, Architecture)
- [ ] Add a drag handle to story folder rows and make them draggable; leave theme rows, files, and all non-story items non-draggable
- [ ] Make theme rows and the `stories` row into drop targets; highlight a valid target while the pointer is over it; suppress the highlight when the story is already there
- [ ] Handle the drop: optimistically move the story in the store, call the backend, revert and show an error on failure; on success open the destination theme and order stories by number
- [ ] Handle Escape and drops on invalid targets: restore the tree to its pre-drag state with no highlight
- [ ] Update the path shown in the builder and the "File:" line in prompts to reflect the story's current theme
- [ ] Open the theme of a story when the story is navigated to via a link, selecting the story
- [ ] On builder load, read theme membership from the backend and populate the store
- [ ] Write Jest tests covering: move into a theme, move between two themes, move back to `stories`, drop where the story already is, Escape during drag, link navigation to a story inside a closed theme, and backend failure reverting the tree (Practice: Pair programming, ADR-0005)
- [ ] Write backend unit and integration tests covering the command handler and the persistence of story-identifier lists in themes (ADR-0009, ADR-0012)
- [ ] Verify the full build and test suite still completes within the ten-minute budget (Practice: Ten-Minute Build)
