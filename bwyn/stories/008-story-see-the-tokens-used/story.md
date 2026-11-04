# Story: See the tokens used, provider by provider

- **Story**: 008-story-see-the-tokens-used
- **Created**: 2026-11-04
- **Status**: Draft

## Story
As a team member, I want Settings > Costs to show the tokens my project used for each LLM provider separately, so that I can tell what each provider costs us, because the same number of tokens costs a different amount at each of them.

## Acceptance tests

**One block per provider**

- [ ] **Given** a project that used Claude on Amazon Bedrock and Azure OpenAI, **When** the user opens Settings > Costs, **Then** two blocks appear — "Claude on Amazon Bedrock" and "Azure OpenAI" — each showing its own Input tokens, Output tokens, Total tokens, number of requests, and the date and time it was last used.
- [ ] **Given** those blocks, **Then** no figure adds up tokens across providers: there is no grand total and no combined bar, and a line under the title reads "Providers are counted apart, because their tokens are priced differently."
- [ ] **Given** a project that used only Claude on the Anthropic API, **Then** one block appears and the other two providers are not listed.
- [ ] **Given** the blocks, **Then** they appear in the order: Claude on Amazon Bedrock, Claude on the Anthropic API, Azure OpenAI; the provider the project is connected to now carries the chip "Connected now".

**Counting inside a block**

- [ ] **Given** Claude on Amazon Bedrock with 2 requests that reported 1,200 new input tokens, 8,000 read from the cache, 3,000 written to the cache, and 900 output tokens, **Then** its block shows Input tokens 12,200 (new input 1,200, read from the prompt cache 8,000, written to the prompt cache 3,000), Output tokens 900, Total tokens 13,100, and "2 requests".
- [ ] **Given** Claude on the Anthropic API with 1 request of 500 new input and 300 output tokens, **Then** its block shows Input 500, Output 300, Total 800, "1 request", and 0 for both cache lines.
- [ ] **Given** Azure OpenAI answered a request that reported 5,000 prompt tokens of which 4,000 were cached, and 400 completion tokens, **Then** its block shows new input 1,000, read from the prompt cache 4,000, written to the prompt cache 0, Input 5,000, Output 400, and Total 5,400; the cached tokens are not counted twice.
- [ ] **Given** each block, **Then** the input bar shows the share of input and output for that provider only.
- [ ] **Given** a provider whose block has no tokens yet but is connected now, **Then** its block shows zeros and "Not used yet".

**What is counted**

- [ ] **Given** the user connects a provider, **Then** the short test request is counted under that provider, not under any previously connected provider.
- [ ] **Given** the user sends a prompt or uses "Improve with AI", **Then** the tokens are added to the block of the provider that answered, and only to the block of the project in use.
- [ ] **Given** the user stops a request before the model finishes, **Then** nothing is added.
- [ ] **Given** a failed request (key refused, provider unreachable), **Then** nothing is added.

**Changing providers**

- [ ] **Given** a project that used Claude on Amazon Bedrock and then switched to Azure OpenAI, **Then** the Bedrock block keeps its figures without "Connected now", and the Azure OpenAI block grows from the switch onwards.
- [ ] **Given** a project that used Claude on Amazon Bedrock and then switched to Claude on the Anthropic API, **Then** the Bedrock block keeps its figures without "Connected now", and the Anthropic API block grows from the switch onwards.
- [ ] **Given** the user removes the key of a provider, **Then** its figures stay.
- [ ] **Given** the user connects the same provider again later, **Then** its block continues from the figures it had before.
- [ ] **Given** two projects, **Then** each shows only its own figures for each provider.

**Before providers were told apart**

- [ ] **Given** figures that were counted before providers were distinguished (only Bedrock existed), **Then** they appear in the Claude on Amazon Bedrock block and none are lost.

**Empty and error states**

- [ ] **Given** a project that never used a model and has no provider connected, **Then** the page shows "No tokens used yet", states that no provider is connected, and offers an "Open Integrations" button.
- [ ] **Given** a project that never used a model but has a provider connected, **Then** the block of the connected provider shows zeros and the page states that tokens appear as soon as the model is used.
- [ ] **Given** the count cannot be read, **Then** the page shows "The count could not be read right now, so nothing is shown." and does not display zeros as if nothing was used.
- [ ] **Given** the user presses Refresh, **Then** the icon spins, the buttons wait, and the figures are read again.

**Where the count lives**

- [ ] **Given** the user opens Costs, **Then** the browser reads the figures from the bwyn backend over its REST API as one response containing a list of providers each with its token counts, and never directly from a provider.
- [ ] **Given** the backend, **Then** the endpoint that returns usage by provider is described in the OpenAPI document (ADR-0014).
- [ ] **Given** the response, **Then** it contains no API key and no text from any prompt or reply.

## Business rules and constraints

- Token figures are kept per project and per provider. No figure is ever summed across providers, because a token of one provider is not worth a token of another.
- Every provider block exposes the same five figures: new input, read from the prompt cache, written to the prompt cache, output, number of requests, and last-used timestamp. Input tokens = new input + cache read + cache write.
- Provider reports are normalised to the shared model before storage. Amazon Bedrock and the Anthropic API report input excluding cached tokens; Azure OpenAI reports prompt tokens including cached tokens, so the backend subtracts the cached portion to derive new input. Azure OpenAI has no cache-write figure; it is stored as 0 (ADR-0017).
- A usage record belongs to the provider that answered the request, regardless of what the project is connected to afterwards. Changing or removing a connection never alters stored figures.
- Usage figures recorded before this story are attributed to Claude on Amazon Bedrock, because it was the only provider at that time. No figures are lost in the migration.
- A request is counted only when the provider has reported its usage. Stopped or failed requests are not counted. The test request sent when connecting a provider is counted.
- The page displays tokens, not money. Each block notes that the price depends on the provider, model, and region.
- The backend performs all counting, because every model request passes through it (ADR-0017). The Costs page only reads the backend.
- The REST endpoint for usage is placed in the inbound adapter layer (`bwyn-rest`) and the counting logic in the application layer (`bwyn-application`), consistent with the hexagonal structure (ADR-0017, AE-0005).
- The frontend `CostsPanel` ViewModel resides in the interface-adapters layer and the React component in the UI layer, consistent with Clean Architecture (ADR-0007, AE-0002).

## Out of scope

- Money: prices, a price table, cost estimates, and budget alerts.
- A breakdown by model, by feature (builder, assistant, diagram), or by day; usage history and charts over time.
- Resetting, exporting, or deleting figures.
- Summing tokens across providers or comparing them in a single figure.
- Connecting LLM providers (covered by story 006).
- Encryption of model API keys at rest on the backend (deferred to a separate ADR).

## Tasks

- [ ] Add a `provider` field to the usage-record domain model in `bwyn-core` so that each stored record is associated with the provider that answered (ADR-0017).
- [ ] Write a Flyway migration script that adds the `provider` column to the usage table and back-fills existing rows with `CLAUDE_AMAZON_BEDROCK` (ADR-0010).
- [ ] Implement normalisation logic in `bwyn-application` that converts each provider's raw usage report (Bedrock, Anthropic API, Azure OpenAI) into the shared five-figure model, including the Azure OpenAI cached-token subtraction.
- [ ] Update the application service in `bwyn-application` that records usage after a completed request to pass the provider identifier alongside the token counts.
- [ ] Add a query handler in `bwyn-application` that returns usage grouped by provider for a given project, with providers ordered Bedrock → Anthropic API → Azure OpenAI.
- [ ] Add a REST endpoint `GET /api/projects/{projectId}/usage` in `bwyn-rest` that invokes the query handler and returns the provider list; annotate it with OpenAPI annotations (ADR-0014).
- [ ] Write a `@DataJpaTest` integration test using Testcontainers that verifies the migration, the back-fill, and the per-provider query (ADR-0012).
- [ ] Write an integration test with a fake provider HTTP server for each of the three providers, including a mid-session provider switch between Bedrock and Anthropic API, to verify that counts accumulate under the correct provider.
- [ ] Add a `UsageByProviderUseCase` port interface in `bwyn-core` and bind it in the composition root (ADR-0017).
- [ ] Create or update the `CostsViewModel` in the interface-adapters layer to call the new REST endpoint, map the response to per-provider view state, and expose a `refresh` command and an error state (ADR-0007, AE-0002).
- [ ] Build the provider block React component in the UI layer showing new input, cache read, cache write, output, total, input bar, request count, last-used timestamp, and the "Connected now" chip (ADR-0007, AE-0002).
- [ ] Update `CostsPanel` to render one block per provider returned, display the "Providers are counted apart…" note, and handle the three empty/error states (no provider connected, provider connected but unused, count unreadable) (ADR-0007).
- [ ] Add a Jest unit test for `CostsViewModel` covering the refresh command, the error state, and the "Connected now" derivation using a test double for the fetch port (ADR-0005).
- [ ] Verify that ArchUnitTS rules still pass with the new frontend modules in place (ADR-0016, AE-0003).
- [ ] Verify that ArchUnit rules still pass with the new backend modules in place (ADR-0009, AE-0005).
