# Story: Improve a text with AI

- **Story**: 007-story-improve-a-text-with
- **Created**: 2026-11-03
- **Status**: Draft

## Story
As a team member, I want an "Improve with AI" button in the dialog where I edit a principle that asks the AI for a clearer text and lets me use its proposal, so that I write better principles without leaving the dialog. The same button must be usable in any other dialog or screen that edits a text.

## Acceptance tests

**The button in the Edit principle dialog**

- [X] **Given** the Edit principle dialog is open, **Then** a collapsed "Improve with AI" button is visible below the Description field.
- [X] **Given** the collapsed button, **When** the user presses it, **Then** a panel opens showing the title "Improve with AI", a Hide (X) button, four quick actions ("Improve this", "Make it actionable", "Ask me questions", "Shorten"), a free-text field with the placeholder "Ask for a change, for example: focus on security", and a Send button that is disabled while the field is empty.
- [X] **Given** the user hides the panel and then opens it again, **Then** the conversation history and the text typed in the field are still present.
- [X] **Given** the Add a principle dialog is open, **Then** the same collapsed "Improve with AI" button is visible below the Description field and opens the same panel; **When** the user asks, **Then** the name and description typed so far are sent, every saved principle is part of the cached context (none is being edited), **When** the user presses "Use this text", **Then** the Description is replaced, the Name is unchanged, and no principle is created until the user presses Save.

**Asking and using the answer**

- [X] **Given** the panel is open, **When** the user presses "Shorten", **Then** "Shorten" appears as the user's message, "Thinking…" is shown until the first token arrives, the reply streams in, and all four quick actions are disabled until streaming ends.
- [X] **Given** the user types "focus on security" in the field and presses Enter, **Then** the instruction is sent as a user message and the Edit principle dialog is not submitted.
- [X] **Given** a reply that proposes new text, **Then** the panel shows a short comment (at most three sentences or three short questions) and a "Proposed text" card containing the proposed text and a "Use this text" button.
- [X] **Given** a proposal, **When** the user presses "Use this text", **Then** the Description field in the dialog is replaced with the proposed text, the Name field is unchanged, the "Use this text" button reads "In the editor" and becomes disabled, and no save is triggered.
- [X] **Given** a reply that is still streaming or was cut off, **Then** "Use this text" is disabled; a cut-off reply additionally shows the label "Incomplete, so it cannot be used."
- [X] **Given** a reply is streaming, **When** the user presses Stop, **Then** streaming halts, the text received so far remains visible, "Use this text" is disabled, and the Send button becomes active again.
- [X] **Given** any completed reply, **Then** a line below the reply shows the token counts: input, output, cache read, and cache write.

**When the request fails**

- [X] **Given** the backend cannot be reached or returns an error, **Then** the error message is shown in the panel together with a Retry button, and pressing Retry resends the last instruction without adding a second user message to the conversation.
- [X] **Given** no model is connected for the project, **Then** the error message reads "No model connected. Connect the model in Integrations."

**The connection to the model goes through the backend**

- [X] **Given** the user sends an instruction, **Then** the browser calls only the BWYN backend REST API; no call is made directly to the model provider, and no API key or token appears in the frontend bundle or in any response sent to the browser.
- [X] **Given** the backend receives the request, **Then** the request body contains the project identifier, the instruction, the full conversation so far, the name and description as they currently appear in the dialog, and the current text in the editor at the moment the instruction is sent; the response is a stream of text chunks, followed by a usage object (input tokens, output tokens, cache-read tokens, cache-write tokens), followed by an end marker, or an error object.
- [X] **Given** the project has no model connection, **Then** the backend returns a clear error status and message and does not call the model provider.
- [X] **Given** the user closes the Edit principle dialog while a reply is streaming, **Then** the browser cancels the HTTP request and the backend stops calling the model.
- [X] **Given** the backend endpoint exists, **Then** it is described in the OpenAPI document with its request schema, streamed response format, usage object, end marker, and error responses (ADR-0014).

**The button is reusable**

- [X] **Given** a second screen (a test screen using the noun "driver") that mounts the shared component with its own noun, quick actions, and instructions, **Then** the panel shows that screen's quick actions, the proposal replaces that screen's text field, and nothing specific to the principle dialog is visible.
- [X] **Given** both the Edit principle dialog and the test screen are mounted simultaneously, **Then** each maintains its own independent conversation and neither sees the other's messages.
- [X] **Given** any screen that uses the shared component, **Then** it supplies only: the project identifier, the current text, the noun, the quick actions, the system instructions, and a callback for applying a proposal; the component contains no knowledge of principles, drivers, or any other domain concept.

## Business rules and constraints

- The AI only suggests; it never saves and never modifies the Name field (Architecture: UI layer owns field state).
- A reply contains a short comment (at most three sentences or three short questions) and, only when new text is proposed, a proposed-text block.
- The AI preserves the user's intent, language, and facts; it invents nothing and prefers plain, specific, positive wording. It never contradicts a non-negotiable practice; if the input text does, the comment says so.
- The stable (cached) context sent to the model contains the project's practices and its other principles, never the principle being edited. The name and description as they stand in the dialog, together with the current text in the editor at the moment the instruction is sent, travel with every instruction.
- The conversation belongs to the open dialog instance and is discarded when the dialog closes.
- Model API keys are stored on the backend only; each project has its own model connection configured in Integrations (story 006).
- The shared component receives everything screen-specific (instructions, context, quick actions, noun, apply-proposal callback) as props or injected dependencies; screen-specific logic is never written inside the component (Principle: Declarative over imperative).
- The existing direct call from the frontend to the dev server (`/api/draft`) is replaced by the new backend REST call; the old call is not kept alongside the new one.
- Assumption: the streamed response uses newline-delimited JSON, with one object per line: `{"type":"text","value":"…"}`, `{"type":"usage","inputTokens":…,"outputTokens":…,"cacheReadTokens":…,"cacheWriteTokens":…}`, and `{"type":"end"}` or `{"type":"error","message":"…"}`.

## Out of scope

- Adding the button to the Drivers, ADRs, or Architecture screens (each is its own story); this story proves reusability with one additional test screen only.
- Connecting a model provider and displaying project-level token totals (story 006 and Integrations).
- Persisting the conversation after the dialog closes.
- Letting the user choose the model from within the dialog.
- Encryption of model API keys at rest on the backend (deferred to a separate ADR).

## Tasks

- [X] Define the streamed-response format (text chunk, usage, end, error) and document the `/improve` endpoint in the OpenAPI specification with its request schema (including the current text field), response stream, and error responses (ADR-0014).
- [X] Review any existing `StreamLlmReplyCommandHandler` and remove or replace it so that no orphaned handler remains alongside the new one introduced by this story.
- [X] Create the `StreamTextImprovementCommandHandler` in `bwyn-application`, with a port interface in `bwyn-core` for the outbound LLM call; the handler checks the project's model connection, builds the cached context, and delegates streaming to the outbound port (ADR-0017).
- [X] Implement the outbound LLM adapter in `bwyn-infrastructure` that calls the configured model provider and streams tokens back through the port.
- [X] Return a typed error from `StreamTextImprovementCommandHandler` when the project has no model connection, and propagate it as the documented error response.
- [X] Implement the Spring MVC inbound adapter in `bwyn-rest` that accepts the request, invokes `StreamTextImprovementCommandHandler`, streams the response, and cancels the model call when the client disconnects (ADR-0008, ADR-0014).
- [X] Replace the existing `bedrockClient.ts` dev-server call with a typed frontend API client that calls the BWYN backend `/improve` endpoint, passing the current text at the moment of sending, and surfaces the same stream of text chunks, usage, and end or error events (ADR-0005, ADR-0015).
- [X] Build the shared `ImproveWithAI` component (button + panel) in the `ui` layer, accepting project id, current text, noun, quick actions, system instructions, and an apply-proposal callback as its only inputs; wire conversation state through the `useImproveWithAI` hook in the `interface-adapters` layer (ADR-0007).
- [X] Integrate the shared component into the Edit principle dialog, passing principle-specific instructions, quick actions, and the apply-to-description callback.
- [X] Offer the same shared component in the Add a principle dialog, with no subject id so that all saved principles are in the cached context.
- [X] Add a second, self-contained test screen that mounts the shared component with the noun "driver" and its own quick actions, verifying that no principle-specific logic leaks into the component.
- [X] Write Jest unit tests for `useImproveWithAI`: streaming state transitions, Stop cancellation, Retry behaviour, and the "no model connected" error path (ADR-0005).
- [X] Write a JUnit unit test for `StreamTextImprovementCommandHandler` using a test-double outbound port (ADR-0009, ADR-0017).
- [ ] Verify the full build and test suite passes within the ten-minute budget (Practice: Ten-Minute Build).
