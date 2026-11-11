# Story: Let the model know the project, for a story

- **Story**: 036
- **Created**: 2026-11-09
- **Status**: Draft

## Story
As a team member, I want the AI to know the decisions, the principles, the practices, the architecture and the work already described in my project, so that the story it drafts fits them and does not contradict or repeat them.

## Acceptance tests

**What the model receives (verified against the request a test double captures)**

- [ ] **Given** a request to draft a story, **Then** the first user message opens with the stable project context: ADRs, links, scope of every story and enabler, principles and practices, followed by a cache point, followed by the per-request content.
- [ ] **Given** the project's Architecture page has saved text, **Then** the stable context ends with a section headed `## Architecture of the system (the layers a story must respect)` containing exactly that saved text, with no leading or trailing blank lines added around it.
- [ ] **Given** the Architecture page is empty or has never been saved, **Then** the architecture section is omitted entirely from the context.
- [ ] **Given** the user has typed but not saved new text on the Architecture page, **Then** the context contains the previously saved text, not the unsaved draft.
- [ ] **Given** a request to draft an **architectural enabler**, **Then** the context contains no architecture section.
- [ ] **Given** the per-request block for a story, **Then** it contains `Document type: user story`, `Assigned identifier: <next-id>-<slug>`, `Today's date: <date>`, the linked documents (story 029/031), `## Existing architectural enablers`, `## Existing stories`, and `## Description`, and it contains no `Derives from (ADRs)` field.
- [ ] **Given** the same project state and the same links, **When** the user submits two story requests, **Then** the stable context block is byte-for-byte identical in both requests, so that a prompt cache hit is possible on the second call.
- [ ] **Given** any change to an ADR, a link, a scope entry, a principle, a practice, or the saved architecture text, **Then** the stable context block changes on the next request.
- [ ] **Given** the system prompt, **Then** it contains the user-story template (title, role, capability, benefit, acceptance tests, business rules and constraints, out of scope, tasks) and the story-writing rules (one Markdown document, no preamble, no closing remark, no code fence).

**What the draft contains (verified by reading drafts from the real model; not automated)**

- [ ] **Given** a project with a defined architecture, **Then** the draft places each rule in the layer it belongs to, names the layer when it helps, cites it as `(Architecture)`, and lists any part of the request that does not fit the layers under Out of scope.
- [ ] **Given** a request that conflicts with an Accepted ADR, **Then** the draft leaves that part out and names it under Out of scope, citing the ADR.
- [ ] **Given** a story or an enabler that already covers part of the request, **Then** the draft does not repeat or take over that coverage, and respects what the existing document leaves out of scope.
- [ ] **Given** a non-negotiable practice or a principle that shapes a rule, **Then** the draft does not contradict it and cites it in brackets, for example `(Practice: Stories)` or `(Principle: DRY and KISS)`.
- [ ] **Given** a linked driver or ADR that shapes a rule, **Then** the draft cites it in brackets, for example `(Driver: Constraints)` or `(ADR 0007)`.
- [ ] **Given** drivers in the project, **Then** they appear in the draft only when the user has linked them (story 029/031); they are never injected from the project context.

## Business rules and constraints

- The stable context block must be assembled once per project-state snapshot and reused unchanged for the draft, any revision (story 034), and any future update of the same story, so that prompt-cache hits are maximised (Principle: DRY and KISS).
- The architecture section is included only for story requests; an enabler request omits it because an enabler is the work that builds the layers, not a feature checked against them (ADR-0007).
- Drivers are not part of the project context; they reach the model only through an explicit link made by the user (stories 029 and 031).
- The system prompt for stories is separate from the system prompt for enablers; each contains only the template and rules relevant to its document type (Principle: SRP).
- The saved architecture text is used verbatim; the backend reads the last persisted value and does not accept unsaved client-side state (ADR-0008, Architecture: application layer).

## Out of scope
- Verifying that a draft is correct; the user reviews it (Practice: Pair programming).
- Choosing or tuning the model, and displaying token costs (covered by stories 006 and 008).
- Stop, retry, and failure-display behaviour during streaming (covered by story 025).
- Asking for changes to a draft (covered by story 034).
- The enabler's own project context (the counterpart of this story for enablers, covered by story 028).
- Sending drivers automatically as part of the project context (drivers are linked explicitly, covered by stories 029 and 031).
- The linked-documents expansion beyond one level (covered by stories 029 and 031).

## Tasks
- [ ] Extend the story-draft backend endpoint to read the saved architecture text from the database and include it in the stable context block when non-empty, omitting it when empty (ADR-0008, ADR-0017: application layer reads the port, outbound adapter fetches the value)
- [ ] Write the system prompt for story drafting: user-story template plus story-writing rules (one Markdown document, no preamble, no closing remark, no code fence), kept in a named constant separate from the enabler system prompt (Principle: SRP)
- [ ] Assemble the stable context block (ADRs, links, scopes, principles, practices, optional architecture section) in a dedicated application-layer service, with a cache key derived from a hash of its inputs so the block is rebuilt only when its inputs change
- [ ] Write unit tests asserting the exact stable context text for a small fixed project fixture: with architecture text, without architecture text, and confirming the architecture section is absent for an enabler request
- [ ] Write a unit test asserting that submitting two requests with identical project state produces an identical stable context block, and that changing the saved architecture text produces a different block
- [ ] Write a unit test asserting that the per-request block contains `Document type: user story`, `Assigned identifier`, `Today's date`, linked documents, `## Existing architectural enablers`, `## Existing stories`, `## Description`, and no `Derives from (ADRs)` field
- [ ] Run ten story drafts against the real model with varied prompts, layer configurations, ADR conflicts, and existing stories; adjust the system prompt until all manual checklist items above pass consistently
