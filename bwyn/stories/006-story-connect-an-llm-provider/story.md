# Story: Connect an LLM provider with an API key

- **Story**: 006-story-connect-an-llm-provider
- **Created**: 2026-11-03
- **Status**: Draft

## Story
As a team member, I want to choose where the AI comes from (Claude on Amazon Bedrock, Claude on the Anthropic API, or Azure OpenAI) and connect it with an API key, so that the project can use the AI service that my team already pays for.

## Acceptance tests

**Choosing a provider**

- [X] **Given** the Integrations screen for a project with no connection, **Then** the "LLM provider" card shows the status "Not connected".
- [X] **Given** the card is opened, **Then** three options appear: "Claude on Amazon Bedrock", "Claude on the Anthropic API", and "Azure OpenAI"; none is selected, no field is shown, and Connect is disabled.
- [X] **Given** the user selects "Claude on Amazon Bedrock", **Then** the form shows: Amazon Bedrock API key (placeholder `bedrock-api-key-...`), Region (default `us-east-1`), Model id (default `us.anthropic.claude-sonnet-4-6`), Prompt cache lifetime (default `5 minutes`), and Longest answer (default `4096` tokens).
- [X] **Given** the user selects "Claude on the Anthropic API", **Then** the form shows: Anthropic API key (placeholder `sk-ant-...`), Model id (default `claude-sonnet-4-6`), Prompt cache lifetime (default `5 minutes`), and Longest answer (default `4096` tokens); no Region field appears.
- [X] **Given** the user selects "Azure OpenAI", **Then** the form shows: Azure OpenAI API key (placeholder `Key 1 or Key 2 of the resource`), Endpoint (placeholder `https://your-resource.openai.azure.com`), Deployment name (placeholder `the name you gave the model in Azure`), and Longest answer (default `4096` tokens); no Region, Model id, or Prompt cache lifetime field appears.
- [X] **Given** the user has typed a key and then selects a different provider option, **Then** the key field is empty.
- [X] **Given** the dark theme is on and the user opens the "Prompt cache lifetime" select, **Then** the closed field and every option in its list have an opaque background with readable text; the light theme looks exactly as before.

**Connecting**

- [X] **Given** a provider is selected and a non-empty key is entered, **When** the user presses Connect, **Then** the button reads "Connecting…"; when the provider responds successfully, the card shows the chip "Connected", the provider name, the model (or deployment for Azure OpenAI), and for Bedrock the Region or for Azure OpenAI the endpoint — for example "Claude on the Anthropic API · claude-sonnet-4-6" or "Azure OpenAI · my-gpt · your-resource.openai.azure.com" — and the key field is gone.
- [X] **Given** the key field is empty or contains only spaces, **When** the user presses Connect, **Then** a message reads "Paste the [provider's key name]." and nothing is sent to the backend.
- [X] **Given** Azure OpenAI is selected and the endpoint or deployment name is empty, **When** the user presses Connect, **Then** the message reads "Enter the endpoint of the Azure resource." or "Enter the deployment name." respectively, and nothing is sent.
- [X] **Given** the endpoint does not start with `https://` or is not a valid URL, **When** the user presses Connect, **Then** the message reads "The endpoint looks wrong, for example https://your-resource.openai.azure.com." and nothing is sent.
- [X] **Given** the key contains spaces within it, **When** the user presses Connect, **Then** the message reads "Paste the key without spaces." and nothing is sent.
- [X] **Given** Longest answer is set to `100` or `70000`, **When** the user presses Connect, **Then** the message reads "Maximum tokens must be a whole number between 256 and 64000." and nothing is sent.
- [X] **Given** the provider refuses the key, **Then** the card remains "Not connected", nothing is persisted, and the message names the provider and states the key was refused — for example "Anthropic refused the key. Check that it is valid and active."
- [X] **Given** the model id does not exist for the provider, or for Azure OpenAI the deployment does not exist on that endpoint, **Then** the card remains "Not connected", nothing is persisted, and the message says to check the model id or the deployment name and endpoint.
- [X] **Given** the provider cannot be reached, **Then** the card remains "Not connected" and the message says the provider could not be reached, with a Connect button to try again.
- [X] **Given** a successful Connect, **Then** the short test request is recorded in the project's usage (Costs), with input, output, cache read, and cache write counts.

**Using the connection**

- [X] **Given** a project connected to Azure OpenAI, **Then** cache write is `0` and cache read reflects what Azure OpenAI reports as cached.
- [X] **Given** project A connected to Amazon Bedrock and project B connected to Azure OpenAI, **Then** each project's requests go to its own provider with its own key.

**Changing, switching, and removing**

- [X] **Given** a connected project, **Then** the card shows the provider, the model or deployment, the non-key settings, and the text "Key: entered here, kept by the backend"; no part of the key is shown.
- [X] **Given** a connected project, **When** the user presses "Change settings", **Then** the form of the same provider appears with current non-key values and the key field empty with hint "Leave empty to keep the current key"; **When** the user saves with an empty key, **Then** the settings are validated against the stored key.
- [X] **Given** a connected project, **When** the user presses "Change provider", **Then** the three provider options appear with the note "Connecting replaces the current connection ([current provider name])." and a Cancel button that leaves everything unchanged.
- [X] **Given** the user connects a different provider with a valid key, **Then** the card shows the new provider and the previous key and settings are deleted.
- [X] **Given** the user attempts to connect a different provider and the check fails, **Then** the original connection continues to work and nothing from the failed attempt is persisted.
- [X] **Given** a connected project, **When** the user presses "Remove the key", **Then** the card shows "Not connected" and the key and settings are deleted.

**The key stays on the backend**

- [X] **Given** any provider, **Then** the browser sends the key exactly once, to the BWYN backend over its REST API; the browser never calls a provider directly, and the key is absent from the bundle, browser storage, and every response returned to the browser.
- [X] **Given** a request for the connection status, **Then** the response contains the provider, the model or deployment, the non-key settings, and the connected flag, and never the key.
- [X] **Given** the key is refused or the request fails, **Then** the key does not appear in the error message or in the backend logs.
- [X] **Given** the backend, **Then** the connect, status, and remove endpoints for all three providers are described in the OpenAPI document (ADR-0014).

## Business rules and constraints

- Three providers are supported: Claude on Amazon Bedrock, Claude on the Anthropic API, and Azure OpenAI. Account-based sign-in is not used; an API key is the only credential for all three.
- Azure OpenAI additionally requires an endpoint (an `https://` URL) and a deployment name; it has no model id field because the deployment name is the team's own identifier.
- A project holds exactly one provider connection at a time; each project has its own connection and its own key (Architecture: backend domain layer, one aggregate per project).
- Provider-specific settings: Region is Bedrock-only; endpoint and deployment name are Azure OpenAI-only; prompt cache lifetime is Claude-only (Bedrock and Anthropic API). Azure OpenAI manages caching internally.
- Bedrock and the Anthropic API default to a model id; Azure OpenAI has no default because the deployment name is user-defined.
- "Connected" means the backend performed a short test request with the supplied credentials and received a valid response. A failed attempt never removes a working connection.
- The key is never returned to the browser in any response, error message, or log entry (Architecture: outbound adapter responsibility).
- All AI calls from the builder, assistants, and diagrams are routed through the project's connected provider port; the callers are unaware of which provider is used (Architecture: application layer port, ADR-0017).
- Usage is recorded as input tokens, output tokens, cache read tokens, and cache write tokens for every provider so that the Costs screen remains consistent.
- The backend endpoint for each provider is configurable; the Azure OpenAI endpoint is the one the user entered for that project, and the backend calls only that address.
- Assumption: the Longest answer field accepts integers in the range 256–64000 inclusive; values outside this range are rejected before the request is sent.

## Out of scope

- More than one provider connected to a project at once, fallback between providers, and per-feature provider selection.
- Providers other than the three named (OpenAI directly, Gemini, local models) and account-based authentication.
- A browsable list of available models or deployments; the user types the model id or deployment name.
- Monetary cost display in Costs (token counts only).
- Encryption of keys at rest on the backend (to be decided in a separate ADR).

## Tasks

- [X] Define the `LlmConnection` aggregate and its value objects (provider type, model id or deployment name, region, endpoint, prompt cache lifetime, max tokens) in `bwyn-core` with no framework dependency (ADR-0017)
- [X] Define the `LlmConnectionRepository` port interface and the `LlmProviderPort` outbound port (check connection, stream reply) in `bwyn-core` (ADR-0017)
- [X] Write unit tests for the `LlmConnection` aggregate covering construction, validation rules, and the rule that a failed attempt does not replace a working connection
- [X] Implement the `ConnectLlmProviderCommand` and `RemoveLlmConnectionCommand` application services in `bwyn-application`, routing through `LlmProviderPort` (ADR-0017)
- [X] Implement the `GetLlmConnectionStatusQuery` application service in `bwyn-application`; ensure the response DTO never includes the key
- [X] Add a Flyway migration in `bwyn-runtime` for the `llm_connection` table (provider, encrypted-key placeholder column, settings columns, project FK) (ADR-0010)
- [X] Implement the `LlmConnectionJpaRepository` outbound adapter in `bwyn-infrastructure` with a JPA entity separate from the domain aggregate; map between them explicitly (ADR-0017, ADR-0011)
- [X] Implement the Amazon Bedrock adapter in `bwyn-infrastructure` implementing `LlmProviderPort`: check, stream, and usage (input, output, cache read, cache write)
- [X] Implement the Anthropic API adapter in `bwyn-infrastructure` implementing `LlmProviderPort`: check, stream, prompt cache lifetime, and usage
- [X] Implement the Azure OpenAI adapter in `bwyn-infrastructure` implementing `LlmProviderPort`: check against endpoint and deployment, stream, and usage with Azure cached tokens as cache read; cache write is always 0
- [X] Add endpoint validation (must start with `https://`, must be a valid URL) in the inbound adapter before the command is dispatched (ADR-0017) (Done in the domain value object ProviderSettings, before any provider is contacted; the controller maps it to 400.)
- [X] Add error translation in the inbound adapter: refused key, unknown model or deployment, and unreachable provider each produce a distinct error message without the key (ADR-0017)
- [X] Expose `POST /projects/{projectId}/llm-connection`, `GET /projects/{projectId}/llm-connection`, `PUT /projects/{projectId}/llm-connection`, and `DELETE /projects/{projectId}/llm-connection` as Spring MVC controllers in `bwyn-rest`, annotated with OpenAPI descriptions (ADR-0014, ADR-0008)
- [X] Write integration tests for each provider adapter against a fake provider HTTP server using Testcontainers (ADR-0012), verifying check, stream, and usage without a real key (Done with an in-process fake HTTP server; Postgres tests use Testcontainers.)
- [X] Extend the ArchUnit rules to assert that `LlmProviderPort` is implemented only in `bwyn-infrastructure` and that no key value appears in response DTOs (ADR-0009)
- [X] Add the `LlmConnectionUseCase` interface and its ViewModel (`LlmConnectionViewModel`) in the frontend interface-adapter layer, injected via Inversify (ADR-0006, ADR-0007)
- [X] Replace the existing Bedrock card in the frontend `ui` layer with the "LLM provider" card: three provider options, per-provider fields with defaults and placeholders, and client-side field validation (ADR-0007, AE-0002)
- [X] Implement the connected view: chip "Connected", provider and model or deployment summary line, no key field, and "Change settings", "Change provider", and "Remove the key" actions
- [X] Implement "Change settings" flow: form pre-filled with current non-key values, empty key field with hint, and save that validates against the stored key when the key field is left empty
- [X] Implement "Change provider" flow: show all three options with the replacement warning and a Cancel that restores the current state; replace the connection only on a successful check
- [X] Wire the builder, "Improve with AI", driver assistant, and diagram features to call the project's connected provider port; display "No model is connected for this project. Connect one in Integrations." when none is set (Done as the backend hook StreamLlmReplyCommandHandler, which gave the no-connection message. Story 007 removed that handler and replaced it with StreamTextImprovementCommandHandler, which gives the same message and powers "Improve with AI" through POST /api/improve; the builder, assistants and diagrams are not in the production frontend yet.)
- [X] Add ArchUnitTS rules asserting that `react-router-dom` and provider-specific imports appear only in the `ui` layer and that the key is never referenced outside the use-case layer (ADR-0016, AE-0003)
- [X] Give select fields an opaque background and `color-scheme` in the dark theme only (`:root:not([data-theme="light"]) select.field-input` and its options in `styles.ts`), so that the "Prompt cache lifetime" list is readable in dark mode and the light theme is untouched, with a Jest test on the stylesheet
- [ ] Verify the full build and test suite passes within the ten-minute budget (Practice: Ten-Minute Build, Practice: Continuous Integration)
