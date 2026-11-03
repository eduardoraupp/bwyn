# Story: Connect an LLM provider with an API key

- **Story**: 006-story-connect-an-llm-provider
- **Created**: 2026-11-03
- **Status**: Draft

## Story
As a team member, I want to choose where the AI comes from (Claude on Amazon Bedrock, Claude on the Anthropic API, or Azure OpenAI) and connect it with an API key, so that the project can use the AI service that my team already pays for.

## Acceptance tests

**Choosing a provider**

- [ ] **Given** the Integrations screen for a project with no connection, **Then** the "LLM provider" card shows the status "Not connected".
- [ ] **Given** the card is opened, **Then** three options appear: "Claude on Amazon Bedrock", "Claude on the Anthropic API", and "Azure OpenAI"; none is selected, no field is shown, and Connect is disabled.
- [ ] **Given** the user selects "Claude on Amazon Bedrock", **Then** the form shows: Amazon Bedrock API key (placeholder `bedrock-api-key-...`), Region (default `us-east-1`), Model id (default `us.anthropic.claude-sonnet-4-6`), Prompt cache lifetime (default `5 minutes`), and Longest answer (default `4096` tokens).
- [ ] **Given** the user selects "Claude on the Anthropic API", **Then** the form shows: Anthropic API key (placeholder `sk-ant-...`), Model id (default `claude-sonnet-4-6`), Prompt cache lifetime (default `5 minutes`), and Longest answer (default `4096` tokens); no Region field appears.
- [ ] **Given** the user selects "Azure OpenAI", **Then** the form shows: Azure OpenAI API key (placeholder `Key 1 or Key 2 of the resource`), Endpoint (placeholder `https://your-resource.openai.azure.com`), Deployment name (placeholder `the name you gave the model in Azure`), and Longest answer (default `4096` tokens); no Region, Model id, or Prompt cache lifetime field appears.
- [ ] **Given** the user has typed a key and then selects a different provider option, **Then** the key field is empty.

**Connecting**

- [ ] **Given** a provider is selected and a non-empty key is entered, **When** the user presses Connect, **Then** the button reads "Connecting…"; when the provider responds successfully, the card shows the chip "Connected", the provider name, the model (or deployment for Azure OpenAI), and for Bedrock the Region or for Azure OpenAI the endpoint — for example "Claude on the Anthropic API · claude-sonnet-4-6" or "Azure OpenAI · my-gpt · your-resource.openai.azure.com" — and the key field is gone.
- [ ] **Given** the key field is empty or contains only spaces, **When** the user presses Connect, **Then** a message reads "Paste the [provider's key name]." and nothing is sent to the backend.
- [ ] **Given** Azure OpenAI is selected and the endpoint or deployment name is empty, **When** the user presses Connect, **Then** the message reads "Enter the endpoint of the Azure resource." or "Enter the deployment name." respectively, and nothing is sent.
- [ ] **Given** the endpoint does not start with `https://` or is not a valid URL, **When** the user presses Connect, **Then** the message reads "The endpoint looks wrong, for example https://your-resource.openai.azure.com." and nothing is sent.
- [ ] **Given** the key contains spaces within it, **When** the user presses Connect, **Then** the message reads "Paste the key without spaces." and nothing is sent.
- [ ] **Given** Longest answer is set to `100` or `70000`, **When** the user presses Connect, **Then** the message reads "Maximum tokens must be a whole number between 256 and 64000." and nothing is sent.
- [ ] **Given** the provider refuses the key, **Then** the card remains "Not connected", nothing is persisted, and the message names the provider and states the key was refused — for example "Anthropic refused the key. Check that it is valid and active."
- [ ] **Given** the model id does not exist for the provider, or for Azure OpenAI the deployment does not exist on that endpoint, **Then** the card remains "Not connected", nothing is persisted, and the message says to check the model id or the deployment name and endpoint.
- [ ] **Given** the provider cannot be reached, **Then** the card remains "Not connected" and the message says the provider could not be reached, with a Connect button to try again.
- [ ] **Given** a successful Connect, **Then** the short test request is recorded in the project's usage (Costs), with input, output, cache read, and cache write counts.

**Using the connection**

- [ ] **Given** a project connected to Amazon Bedrock, the Anthropic API, or Azure OpenAI, **When** the user sends a prompt in the builder, **Then** the story streams in the same way for all three providers, and the token line shows input, output, cache read, and cache write.
- [ ] **Given** a project connected to Azure OpenAI, **Then** cache write is `0` and cache read reflects what Azure OpenAI reports as cached.
- [ ] **Given** a project connected to a provider, **When** the user uses "Improve with AI", the driver assistant, or a diagram, **Then** the same provider answers.
- [ ] **Given** project A connected to Amazon Bedrock and project B connected to Azure OpenAI, **Then** each project's requests go to its own provider with its own key.
- [ ] **Given** no provider is connected to the project, **When** the user sends a prompt, **Then** the message reads "No model is connected for this project. Connect one in Integrations."

**Changing, switching, and removing**

- [ ] **Given** a connected project, **Then** the card shows the provider, the model or deployment, the non-key settings, and the text "Key: entered here, kept by the backend"; no part of the key is shown.
- [ ] **Given** a connected project, **When** the user presses "Change settings", **Then** the form of the same provider appears with current non-key values and the key field empty with hint "Leave empty to keep the current key"; **When** the user saves with an empty key, **Then** the settings are validated against the stored key.
- [ ] **Given** a connected project, **When** the user presses "Change provider", **Then** the three provider options appear with the note "Connecting replaces the current connection ([current provider name])." and a Cancel button that leaves everything unchanged.
- [ ] **Given** the user connects a different provider with a valid key, **Then** the card shows the new provider and the previous key and settings are deleted.
- [ ] **Given** the user attempts to connect a different provider and the check fails, **Then** the original connection continues to work and nothing from the failed attempt is persisted.
- [ ] **Given** a connected project, **When** the user presses "Remove the key", **Then** the card shows "Not connected" and the key and settings are deleted.

**The key stays on the backend**

- [ ] **Given** any provider, **Then** the browser sends the key exactly once, to the BWYN backend over its REST API; the browser never calls a provider directly, and the key is absent from the bundle, browser storage, and every response returned to the browser.
- [ ] **Given** a request for the connection status, **Then** the response contains the provider, the model or deployment, the non-key settings, and the connected flag, and never the key.
- [ ] **Given** the key is refused or the request fails, **Then** the key does not appear in the error message or in the backend logs.
- [ ] **Given** the backend, **Then** the connect, status, and remove endpoints for all three providers are described in the OpenAPI document (ADR-0014).

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

- [ ] Define the `LlmConnection` aggregate and its value objects (provider type, model id or deployment name, region, endpoint, prompt cache lifetime, max tokens) in `bwyn-core` with no framework dependency (ADR-0017)
- [ ] Define the `LlmConnectionRepository` port interface and the `LlmProviderPort` outbound port (check connection, stream reply) in `bwyn-core` (ADR-0017)
- [ ] Write unit tests for the `LlmConnection` aggregate covering construction, validation rules, and the rule that a failed attempt does not replace a working connection
- [ ] Implement the `ConnectLlmProviderCommand` and `RemoveLlmConnectionCommand` application services in `bwyn-application`, routing through `LlmProviderPort` (ADR-0017)
- [ ] Implement the `GetLlmConnectionStatusQuery` application service in `bwyn-application`; ensure the response DTO never includes the key
- [ ] Add a Flyway migration in `bwyn-runtime` for the `llm_connection` table (provider, encrypted-key placeholder column, settings columns, project FK) (ADR-0010)
- [ ] Implement the `LlmConnectionJpaRepository` outbound adapter in `bwyn-infrastructure` with a JPA entity separate from the domain aggregate; map between them explicitly (ADR-0017, ADR-0011)
- [ ] Implement the Amazon Bedrock adapter in `bwyn-infrastructure` implementing `LlmProviderPort`: check, stream, and usage (input, output, cache read, cache write)
- [ ] Implement the Anthropic API adapter in `bwyn-infrastructure` implementing `LlmProviderPort`: check, stream, prompt cache lifetime, and usage
- [ ] Implement the Azure OpenAI adapter in `bwyn-infrastructure` implementing `LlmProviderPort`: check against endpoint and deployment, stream, and usage with Azure cached tokens as cache read; cache write is always 0
- [ ] Add endpoint validation (must start with `https://`, must be a valid URL) in the inbound adapter before the command is dispatched (ADR-0017)
- [ ] Add error translation in the inbound adapter: refused key, unknown model or deployment, and unreachable provider each produce a distinct error message without the key (ADR-0017)
- [ ] Expose `POST /projects/{projectId}/llm-connection`, `GET /projects/{projectId}/llm-connection`, `PUT /projects/{projectId}/llm-connection`, and `DELETE /projects/{projectId}/llm-connection` as Spring MVC controllers in `bwyn-rest`, annotated with OpenAPI descriptions (ADR-0014, ADR-0008)
- [ ] Write integration tests for each provider adapter against a fake provider HTTP server using Testcontainers (ADR-0012), verifying check, stream, and usage without a real key
- [ ] Extend the ArchUnit rules to assert that `LlmProviderPort` is implemented only in `bwyn-infrastructure` and that no key value appears in response DTOs (ADR-0009)
- [ ] Add the `LlmConnectionUseCase` interface and its ViewModel (`LlmConnectionViewModel`) in the frontend interface-adapter layer, injected via Inversify (ADR-0006, ADR-0007)
- [ ] Replace the existing Bedrock card in the frontend `ui` layer with the "LLM provider" card: three provider options, per-provider fields with defaults and placeholders, and client-side field validation (ADR-0007, AE-0002)
- [ ] Implement the connected view: chip "Connected", provider and model or deployment summary line, no key field, and "Change settings", "Change provider", and "Remove the key" actions
- [ ] Implement "Change settings" flow: form pre-filled with current non-key values, empty key field with hint, and save that validates against the stored key when the key field is left empty
- [ ] Implement "Change provider" flow: show all three options with the replacement warning and a Cancel that restores the current state; replace the connection only on a successful check
- [ ] Wire the builder, "Improve with AI", driver assistant, and diagram features to call the project's connected provider port; display "No model is connected for this project. Connect one in Integrations." when none is set
- [ ] Add ArchUnitTS rules asserting that `react-router-dom` and provider-specific imports appear only in the `ui` layer and that the key is never referenced outside the use-case layer (ADR-0016, AE-0003)
- [ ] Verify the full build and test suite passes within the ten-minute budget (Practice: Ten-Minute Build, Practice: Continuous Integration)
