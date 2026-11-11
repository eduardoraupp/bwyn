# Story: Follow, revise and discard the draft of a story

- **Story**: 034-story-3-follow-revise-and
- **Created**: 2026-11-09
- **Status**: Draft

## Story
As a team member, I want to see the state of the story draft, stop it, retry it, ask for changes, or discard it, so that I never wait without knowing what is happening and I get the story I mean before I approve it.

## Acceptance tests

**Draft card appearance**
- [X] **Given** a story draft is on screen, **When** the card renders, **Then** it shows a document icon in blue, the path `stories/<number>-<slug>/story.md`, and an approve button labelled "Approve and create story".

**States, Stop, Retry and Discard behave identically to enabler drafts**
- [X] **Given** a story draft is streaming, **When** the stream is in progress, **Then** the chip reads "Writing…" and the Stop button is enabled.
- [X] **Given** a story draft has finished streaming without interruption, **Then** the chip reads "Ready for review" and the approve button is enabled.
- [X] **Given** the user presses Stop while the draft is streaming, **Then** streaming halts, the chip reads "Incomplete", and the approve button is disabled.
- [X] **Given** the backend call fails, **Then** the chip reads "Failed", an error message is shown, and a Retry button is enabled.
- [X] **Given** the user presses Retry after a failure or a stop, **Then** a new streaming request is sent and the chip returns to "Writing…".
- [X] **Given** the user presses Discard, **Then** the draft is removed from the screen, nothing is persisted, the prompt box reappears with the "Mark as architectural enabler" checkbox unchecked, and the Link button is visible again.

**Prompt box while a draft is present**
- [X] **Given** a story draft is on screen (in any state), **Then** the "Mark as architectural enabler" checkbox and the Link button are not shown.
- [X] **Given** a story draft is on screen, **Then** the prompt box placeholder reads "Ask for changes to the draft…" and the send button reads "Send changes".
- [ ] **Given** the draft is discarded or approved, **Then** the prompt box reverts to the new-document state with the checkbox unchecked and the Link button visible. (Open: discarding is built and tested; keeping a story on approval comes with story 035, so the approved half cannot be verified yet.)

**Asking for changes**
- [X] **Given** a ready story draft and the user types a change instruction and presses "Send changes", **Then** the AI returns the whole story document, the chip returns to "Writing…" during streaming, and the card shows the updated text when done.
- [X] **Given** a revision completes, **Then** the story number, slug, planned path `stories/<number>-<slug>/story.md`, identifier, and links are identical to those of the original draft.
- [X] **Given** a revision is requested, **Then** the AI is asked for the complete story including tasks, not a partial update.
- [X] **Given** two successive revisions are sent, **Then** each returns the whole document and the number, slug, path, identifier and links remain unchanged across both turns.
- [X] **Given** the conversation reaches 40 turns, **Then** the "Send changes" button is disabled and a message explains that the limit is reached.

**Incomplete draft cannot be approved**
- [X] **Given** a draft that was stopped mid-task, **Then** the approve button is disabled and "Send changes" is available so the user can ask the AI to complete it.

**Token display**
- [ ] **Given** a story draft completes or a revision completes, **Then** the token chip shows the cumulative token count for the session in the same format used for enabler drafts. (Open: each draft shows the four counts of its last answer, as enabler drafts do and as story 025 established; no running total across the session is built.)

## Business rules and constraints
- A draft has exactly one state at a time: *streaming*, *ready*, *stopped*, or *error*. Only a *ready* draft may be approved (ADR-0007, Architecture).
- A draft lives only in memory. Reloading the page loses it; no persistence is introduced here (Practice: Incremental design).
- A revision sends the full current draft and the user's instruction to the model and expects the whole story document back, identifier and date unchanged.
- The number, slug, planned path, identifier, and links assigned when the draft was first created are immutable for the lifetime of that draft; a revision may not change them.
- Changing the links of a draft requires discarding it and starting again (Practice: Incremental design).
- The 40-turn limit applies to the combined prompt-plus-revision conversation for a single draft, consistent with the enabler draft behaviour established in story 025 and story 027.

## Out of scope
- Approving the draft and writing the file to the tree (a later story).
- A history of drafts, persisting a draft across page reloads, navigating back to an earlier version, and comparing two versions (consistent with story 025 and story 027 out-of-scope).
- Changing the ADRs or links of a draft after creation; the user must discard and restart (consistent with story 027).
- Stop, Retry, failure display, and token-count mechanics at the infrastructure level; those are established by story 025 and reused here without change.
- Sending project context with the request (covered by story 028).
- Connecting an LLM provider and the streaming protocol internals (covered by story 006 and story 007).
- Displaying project-level token totals in Settings > Costs (covered by story 008).

## Tasks
- [X] Extend the draft-card component to read the document kind and render the story icon (blue), the path `stories/<number>-<slug>/story.md`, and the label "Approve and create story" when the kind is `story` (ADR-0007, `ui` layer).
- [X] Add a ViewModel property that exposes the document kind so the draft card and the prompt box can derive their presentation from it without containing logic (ADR-0007, `interface-adapters` layer).
- [X] Update the prompt-box component to hide the "Mark as architectural enabler" checkbox and the Link button while any draft is present, and restore them on discard or approval (ADR-0007, `ui` layer).
- [X] Update the prompt-box placeholder and send-button label to "Ask for changes to the draft…" / "Send changes" while a story draft is present, reusing the same mechanism already built for enabler drafts (Practice: DRY and KISS).
- [X] Verify in a unit test that the ViewModel enforces the rule that only a *ready* draft may transition to *approved*, and that *stopped* and *error* states block approval (ADR-0007, `use-cases` layer).
- [X] Write a Jest integration test with a fake backend covering: a story draft that streams, finishes, is stopped, fails, is retried, is revised twice, and is discarded; assert that number, slug, path, identifier and links are unchanged across revisions (Practice: Continuous integration).
- [ ] Verify the full build and test suite passes within the ten-minute budget (Practice: Ten-Minute Build). (Open: the suite passes, but a full run takes about 25 minutes on this machine, beyond the ten-minute budget.)
