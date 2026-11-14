# Story: Link an architectural enabler to other documents

- **Story**: 040-story-3-link-an-architectural
- **Created**: 2026-11-12
- **Status**: Draft

## Story
As a team member, I want to link an architectural enabler to the documents that shape it and to the ADRs it implements, from its own panel, so that the connection is kept with the enabler, the enabler always implements at least one ADR, and the AI and the sync use it.

## Acceptance tests

**The rules of the builder (the view model, checked without rendering)**

- [X] **Given** an enabler as the target, **Then** the groups and their rules are those of story 029: the order, not itself, every driver, every other enabler and story, only Accepted ADRs and the linked one that is not.
  _Tested in story 040 (link-from-enabler-panel.acceptance.test.tsx), which owns this behaviour; the duplicate test here was removed._
- [X] **Given** the enabler's ADRs and links, **Then** the checked options are the ADRs it derives from **and** its links.
- [X] **Given** an enabler that derives from exactly one ADR, **Then** that ADR is offered locked, with the reason "An enabler implements at least one ADR." **Given** it derives from two or more, **Then** none is locked. **Given** the only ADR is no longer Accepted, **Then** it is still offered, checked and locked.
- [X] **Given** an enabler that derives from no ADR (one that came from a repository with none), **Then** nothing is locked and no ADR is checked. **When** the user checks one, **Then** it becomes the one that is locked.
- [X] **Given** a selection, **Then** `applyLinkSelection` returns `derivesFrom` with the references that start with `adr:` and `links` with the others, each in the order they were given.
- [X] **Given** a selection with no `adr:` reference, **Then** it returns an empty `derivesFrom`, so the panel can show the problem, and never reaches it from the picker, because the last ADR is locked.

**The panel of an enabler**

- [X] **Given** an open enabler, **Then** its panel has "Derives from" (a chip for each ADR, opening it, with no X), "Linked to" with the button "Add link" and its chips, and an empty line while it has no link. "Derives from" is not shown while the enabler has no ADR.
- [X] **Given** the user checks an ADR, **Then** it is added to the enabler's `derivesFrom` and its chip appears under "Derives from". **Given** the user checks anything else, **Then** it is added to its `links` and its chip appears under "Linked to".
- [X] **Given** the user unchecks one of two ADRs, **Then** it leaves "Derives from". **Given** the enabler has one ADR, **Then** the checkbox cannot be unchecked with the mouse or the keyboard.
- [X] **Given** the X of a chip in "Linked to", **When** the user presses it, **Then** only that link is removed, and the ADRs it derives from are left alone.
- [X] **Given** a link was added or removed, **Then** it is kept at once, with no Save, and the text, the title and the first prompt of the enabler are not touched.
- [X] **Given** the note of the panel, **Then** it reads: "Read-only here. Describe the change in the prompt to update this enabler." The panel has no text box and no Save.

**What the links do**

- [X] **Given** an enabler's ADRs, **Then** a request about it reads them as "derives from" first, and the links as "linked" in full, each document once.
- [X] **Given** the user changes the enabler's ADRs or links in the panel, **When** the user then sends a change in the prompt, **Then** the request has the new ADRs and links, because it reads them from the enabler, not from the first request.
- [ ] **Given** the enabler is stored in the repository, **Then** its `.meta.json` has `derivesFrom` and `links`, sorted, and a pull brings them back.
  _Waiting for repository sync: the product does not write or pull enabler `.meta.json` yet (synchronising is out of scope here, as in story 039)._

**Keeping them**

- [X] **Given** the page is reloaded, **Then** the ADRs and the links of each enabler are still there. **Given** two projects, **Then** each keeps its own.

## Business rules and constraints

- An enabler always implements at least one ADR; its last ADR cannot be unchecked in the picker (ADR-0007, Architecture).
- For an enabler, ADRs are stored in `derivesFrom` and everything else in `links`. The picker presents one list; the split into the two fields is the view model's responsibility (Architecture: interface-adapter layer, ADR-0007).
- An enabler cannot link to itself. It may link to a story, a driver, or another enabler; loops are allowed.
- Only an Accepted ADR is newly offered. An ADR that is already derived from and is no longer Accepted stays, remains checked, and keeps its lock if it is the last one.
- Links and `derivesFrom` for each enabler are persisted in the backend database (ADR-0008, ADR-0011); they are the authoritative source of truth, not the browser.
- A change to links or `derivesFrom` is saved immediately with no explicit Save action; the enabler's text, title, and first prompt are not modified by this operation.

## Out of scope

- Editing the enabler's text by hand, the *Cross enabler* flag, marking it Done, deleting it, and synchronising it with the repository.
- Changing the ADRs or the links by prompt (the prompt only updates the text), and a warning when an enabler that stories already use changes its ADRs.
- Choosing the ADRs when the enabler is first created (covered by story 024).
- Adding or removing links of an already-approved story (covered by story 039).
- A graph view of the links.

## Tasks

- [X] Write failing unit tests for `linkChoices` and `applyLinkSelection` for an enabler: checked options (ADRs and links together), the lock with one, two, and no ADR, a non-Accepted last ADR, and the split into `derivesFrom` and `links` (ADR-0007, Architecture: interface-adapter layer).
- [X] Extend the `EnablerLinkViewModel` with `linkChoices`, `applyLinkSelection`, and the locked-last-ADR rule, making the unit tests pass.
- [X] Add a `PATCH /projects/{projectId}/enablers/{enablerId}/links` endpoint in the inbound adapter (`bwyn-rest`) that accepts `{ derivesFrom, links }` and delegates to an application service (ADR-0017, Architecture).
- [X] Add the `UpdateEnablerLinksCommand` and its handler in `bwyn-application`; the handler updates `derivesFrom` and `links` on the enabler aggregate in `bwyn-core` and persists via the outbound port (ADR-0017).
- [X] Add or extend the Flyway migration in `bwyn-infrastructure` to store `derives_from` and `links` for an enabler (ADR-0010, ADR-0011).
- [X] Implement the outbound adapter in `bwyn-infrastructure` that persists and retrieves `derivesFrom` and `links` using Spring Data JPA (ADR-0011).
- [X] Build the enabler panel's "Derives from" section (chips, no X) and "Linked to" section ("Add link" button and chips) in the `ui` layer, wired to the view model and calling `PATCH` on every change (ADR-0007, Architecture: ui layer).
- [X] Make the X on a "Linked to" chip remove only that link and call `PATCH`; verify ADRs in "Derives from" are unaffected.
- [X] Write integration tests (extending `PostgresIntegrationTest`) that verify `derivesFrom` and `links` are persisted, survive a reload, and are isolated per project (ADR-0012).
- [X] Verify that ArchUnit rules still pass: the application service does not depend on adapter classes, and the domain aggregate has no Spring or JPA dependency (ADR-0009, ADR-0017).
- [X] Verify that ArchUnitTS rules still pass: the view model resides in the interface-adapter layer and the panel component in the ui layer (ADR-0016, ADR-0007).
- [X] Run the full build and confirm it completes within ten minutes (Practice: Ten-Minute Build).
