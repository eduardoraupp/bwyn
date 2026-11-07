# Story: Connect a Git repository

- **Story**: 017-story-1-connect-a-git
- **Created**: 2026-11-07
- **Status**: Draft

## Story
As a team member, I want to choose GitHub or GitLab and connect my project to a repository there by pasting an API key in Integrations, so that BWYN can keep the project's documents in the repository my team already uses, and my key never leaves the backend.

## Acceptance tests

### The card and the choice of provider

- [X] **Given** Integrations in a project with no connection, **Then** the row "Git repository" shows the sub-line "Not connected" and no chip; **When** it is opened, **Then** a paragraph explains that drivers, architecture, ADRs, principles, architectural enablers, stories and diagrams are kept in a repository so that code generation can use them, and that connecting downloads everything from the base branch once, after which Sync moves changes in both directions. (Deviation: the card is always open, so the paragraph is always visible while nothing is connected, and there is no row to open.)
- [X] **Given** the backend lists GitHub and GitLab, **Then** two options "GitHub" and "GitLab" are shown, none is selected, no field is visible, and Connect is disabled.
- [X] **Given** the backend lists only GitHub, **Then** no provider choice is shown and the GitHub fields appear directly.
- [X] **Given** the user selects "GitHub", **Then** the fields shown are: "GitHub API key" (password field, placeholder `github_pat_…`), Repository (placeholder `owner/name`), Base branch (default `main`), and Folder in the repository (default `bwyn`).
- [X] **Given** the user selects "GitLab", **Then** the fields shown are: "GitLab API key" (placeholder `glpat-…`), Host (placeholder and default `https://gitlab.com`), Project (placeholder `group/project`), Base branch (default `main`), and Folder in the repository (default `bwyn`).
- [X] **Given** a key has been typed and the user switches to the other provider option, **Then** the key field is empty, so that a key is never sent to a provider it does not belong to.
- [X] **Given** the GitHub key field is visible, **Then** a hint states that a fine-grained personal access token for the repository is required, with Contents and Pull requests both set to read and write, that for an organisation repository it must be created with that organisation as resource owner, and that it is sent to the BWYN backend which checks and keeps it and the browser does not keep it.
- [X] **Given** the GitLab key field is visible, **Then** a hint states that a personal, project or group access token with the `api` scope and the role Developer or higher on the project is required, and that it is sent to the BWYN backend which checks and keeps it and the browser does not keep it.
- [X] **Given** the form is open, **Then** Connect is disabled while Repository (GitHub) or Project (GitLab) is empty.

### Checking the fields

- [X] **Given** a GitHub repository entered as `acme/platform`, `https://github.com/acme/platform`, or `https://github.com/acme/platform.git/` with surrounding spaces, **Then** all are parsed as owner `acme` and name `platform`; **Given** any other format such as `platform` alone, **When** Connect is pressed, **Then** "Write the repository as owner/name." is shown and nothing is sent.
- [X] **Given** a GitLab project entered as `acme/platform`, `acme/team/platform`, or `https://gitlab.com/acme/team/platform.git/`, **Then** all are accepted as a valid project path; **Given** a single word with no slash, **When** Connect is pressed, **Then** "Write the project as group/name, or group/subgroup/name." is shown and nothing is sent.
- [X] **Given** a GitLab Host that is not an `https` address (e.g. `http://gitlab.acme.com` or `gitlab`), **When** Connect is pressed, **Then** "The host looks wrong, for example https://gitlab.com." is shown and nothing is sent; **Given** the project field contains an address whose host differs from the Host field, **Then** "This address is not on \<host\>." is shown and nothing is sent.
- [X] **Given** an empty key or a key consisting only of spaces, **When** Connect is pressed, **Then** "Paste the GitHub API key." or "Paste the GitLab API key." is shown and nothing is sent; **Given** a key containing internal spaces or longer than 500 characters, **Then** "Paste the key without spaces." is shown and nothing is kept.
- [X] **Given** an empty Base branch field, **Then** `main` is used; **Given** a Folder value with leading or trailing slashes, **Then** they are stripped, and an empty folder is treated as the repository root and displayed as `(root)`.
- [X] **Given** a Folder value containing `..`, or a character other than letters, digits, `_`, `.`, `/`, and `-`, **When** Connect is pressed, **Then** "Invalid base folder." is shown and nothing is kept.

### Connecting

- [X] **Given** all fields are valid, **When** Connect is pressed, **Then** the button reads "Connecting…", the backend verifies the key with the provider, then the repository, then the base branch; on success the card shows the chip "Connected" and the sub-line `acme/platform · main`, and the form is replaced by the connected view.
- [X] **Given** the connected view, **Then** it shows: Provider (`GitHub` or `GitLab`), Repository (a link opening the repository on the provider), Host only when it is not `gitlab.com`, Base branch and folder (`main · bwyn` or `main · (root)`), Last sync (`Not synced yet`), and Signed in as (the key's user).
- [X] **Given** the key cannot push to the repository (GitHub) or its role is below Developer (GitLab), **Then** an amber note reads "The key may not be allowed to push to this repository, so uploads can fail."
- [X] **Given** the provider refuses the key, **Then** "GitHub did not accept this API key." or "GitLab did not accept this API key." is shown, the card remains "Not connected", and nothing is kept.
- [X] **Given** the repository does not exist or the key cannot see it, **Then** the error message from the provider is shown (e.g. "GitHub 404: Not Found") and nothing is kept; **Given** the provider returns 403, **Then** the message also shows the advice: for GitHub, "Check the key on GitHub: its repository access must include this repository, Contents and Pull requests must both be "Read and write", and for an organization repository the key must be created with that organization as resource owner."; for GitLab, "Check the key on GitLab: it needs the api scope, and the role Developer or higher on the project."
- [X] **Given** a branch that does not exist, **Then** `The branch "x" was not found.` is shown and nothing is kept.
- [X] **Given** the provider cannot be reached, **Then** "GitHub could not be reached." or "GitLab could not be reached at \<host\>." is shown, the card stays as it was, and Connect can be retried.
- [X] **Given** a project on a self-managed GitLab host, **Then** the key is sent only to that host and to no other. (Verified in `GitLabProviderAdapterTest`: the key goes to the host of the connection and a redirect is never followed. The integration test covers GitHub only, because a GitLab host must be https and the fake server is not.)
- [X] **Given** a project that was previously connected and the provider no longer accepts the saved key, **Then** the card shows in red "\<Provider\> did not accept the saved key: \<reason\>". (The backend asks the provider each time the status is read and reports the problem; the connection stays as it was kept.)

### The key stays on the backend

- [X] **Given** Connect is pressed, **Then** the browser sends the key once to the BWYN backend over its REST API; it never calls GitHub or GitLab directly, and the key is absent from the frontend bundle, from browser storage, and from every response returned to the browser. (Verified by the acceptance test, which finds the key nowhere on the page or in browser storage, and by `layer-rules.test.ts`, which allows only the Git and LLM connection files to mention the key and forbids them storage, cookies and the console.)
- [X] **Given** the connection status is fetched, **Then** the response contains provider, host, repository, branch, folder, and the key's user, and never the key itself.
- [X] **Given** a refused key or a failed request, **Then** the key is absent from the error message and from every log line. (`GitConnectionIT` checks every log line; the adapter tests check the messages, including a provider that echoes the key.)
- [X] **Given** two projects each with their own connection, **Then** a request for one project never uses the other project's key.
- [X] **Given** the page is reloaded after connecting, **Then** the project is still shown as connected with the same fields.

### Changing and removing

- [X] **Given** a connected project, **When** "Change repository" is pressed, **Then** the form of the same provider shows with the current repository (or project), branch, and folder pre-filled, and the key field is empty with the placeholder "Leave empty to keep the current key"; **When** saved with a new repository, **Then** all fields are checked with the kept key and the last-synchronised record is cleared. (There is no last-synchronised record yet, so none is cleared.)
- [X] **Given** a connected project, **When** "Change provider" is pressed, **Then** the provider options appear with the note "Connecting replaces the current connection (GitHub)." and a Cancel button that leaves everything unchanged.
- [X] **Given** the user connects the other provider and the check passes, **Then** the card shows the new connection and the old key, settings, and synchronisation record are deleted.
- [X] **Given** the check of a changed repository or provider fails, **Then** the old connection is still present and working, and nothing from the failed attempt is kept.
- [X] **Given** a connected project, **When** "Remove the key" is pressed, **Then** the card shows "Not connected", the key and settings are deleted from the backend, and the synchronisation record is cleared. (There is no synchronisation record yet, so none is cleared.)
- [ ] **Given** no connection exists, **Then** every Sync button in the application is disabled with the hint "Connect a Git repository in Integrations first"; **Given** a connection to either provider, **Then** all Sync buttons are enabled. (Not met yet: Sync does not work yet, so enabling its buttons would add buttons that do nothing. Every Sync button is still disabled. This waits for the Sync stories (3 to 6).)

## Business rules and constraints

- The two supported providers are GitHub and GitLab (gitlab.com or a self-managed `https` host). No other provider is offered. The backend lists available providers from its configuration (ADR-0008).
- An API key is the only credential. OAuth, SSH keys, deploy keys, and account sign-in are not used.
- A project has exactly one connection at a time: one provider, one key, one repository, one base branch, one folder. Each project has its own independent connection.
- A GitLab project is identified by a path of one or more groups and a name; BWYN accepts the bare path or the full URL.
- The key is stored by the backend only and is never included in any response to the browser. How it is encrypted at rest is deferred to a separate ADR (ADR-0006 neutral note).
- "Connected" means the provider accepted the key, the repository is visible with that key, and the base branch exists. A failed connection attempt never removes or overwrites a working connection.
- The folder isolates BWYN's files within the repository. All reads and writes by BWYN are confined to that folder.
- The browser communicates only with the BWYN backend through a provider-neutral REST contract; it never calls GitHub or GitLab directly (Architecture: frontend must not assume deployment mode; backend API base URL is configurable).
- Provider-specific terminology (key name, repository or project label, pull request or merge request, links) is supplied by the backend per provider and is not hard-coded in shared UI components (ADR-0007, Principle: Declarative over imperative).
- The inbound REST adapter lives in `bwyn-rest`, the provider port interface and domain objects in `bwyn-core`, the connection application service in `bwyn-application`, and the GitHub and GitLab HTTP client implementations in `bwyn-infrastructure` (ADR-0017, AE-0005).
- On the frontend the connection ViewModel lives in the interface-adapters layer, the connection use-case interface in the use-cases layer, and the React card component in the UI layer (ADR-0007, AE-0002).

## Out of scope

- Downloading existing repository content on first connect (story 2 of the Git integration).
- Sending and receiving changes (Sync stories 3–6 of the Git integration).
- GitHub Enterprise Server, GitHub Apps, GitLab account sign-in, and OAuth flows.
- Multiple providers or repositories connected to one project simultaneously.
- One key shared across several projects.
- Encryption of the key at rest (deferred to a separate ADR).

## Tasks

- [X] Define the OpenAPI contract for `GET /providers`, `PUT /connection`, `GET /connection`, and `DELETE /connection`, with no key field in any response (ADR-0014) (`GET /api/git/providers` and `GET`, `PUT`, `DELETE /api/projects/{projectId}/git-connection`; checked in `GitConnectionIT`.)
- [X] Add the `GitProvider` port interface and `Connection` domain object to `bwyn-core` (ADR-0017, AE-0005) (`GitProviderPort`, `GitConnection`, `GitConnectionRules` and `GitApiKey` in `bwyn-core`.)
- [X] Add the `ConnectRepositoryCommand` and `GetConnectionQuery` application service interfaces to `bwyn-application` (ADR-0017) (Built as `GitConnectRepositoryCommand` and `GitGetConnectionQuery`, with their handlers, `GitRemoveConnectionCommand` and `ListGitProvidersQuery`: every Git class starts with `Git`.)
- [X] Implement the `GitHubProviderAdapter` in `bwyn-infrastructure`: verify key (user), repository visibility and push permission, and branch existence, returning the structured error messages defined in the acceptance tests (ADR-0017)
- [X] Implement the connection persistence outbound adapter in `bwyn-infrastructure`: store and retrieve one connection per project, never returning the key (ADR-0010, ADR-0011)
- [X] Add the Flyway migration for the connection table to `bwyn-runtime` (ADR-0010)
- [X] Implement the `ConnectionController` inbound adapter in `bwyn-rest` with the OpenAPI annotations, wiring to the application service (ADR-0014, ADR-0017) (`GitConnectionController`.)
- [X] Write unit tests for `GitHubProviderAdapter` against a fake GitHub HTTP server covering: valid key, refused key, missing repository, 403 with advice, missing branch, unreachable server, and key absent from every response and log line (ADR-0012)
- [X] Add the `GitLabProviderAdapter` in `bwyn-infrastructure`: verify key, project path (including subgroups), role, and branch on the configured host; send the key only to that host (ADR-0017)
- [X] Write unit tests for `GitLabProviderAdapter` against a fake GitLab server covering the same cases as GitHub plus subgroup project path and self-managed host (ADR-0012)
- [X] Add the provider-choice toggle and the "Change provider" flow to the frontend card, clearing the key field on provider switch (ADR-0007, AE-0002)
- [X] Build the GitHub field set with client-side validation (repository format, key emptiness, key length, folder characters) and the connected view (ADR-0007, AE-0002)
- [X] Build the GitLab field set with client-side validation (host must be `https`, project path format, host/project consistency) and extend the connected view with the Host field (ADR-0007, AE-0002)
- [X] Implement the connection ViewModel in the interface-adapters layer and wire it via the Inversify composition root (ADR-0006, ADR-0007, AE-0002)
- [X] Implement "Change repository" (pre-fill form, empty key with placeholder, clear sync record on success) and "Remove the key" (delete connection, clear sync record) (ADR-0007)
- [ ] Read the enabled state of every Sync button from the connection status and show the hint when no connection exists (ADR-0007) (Deferred with the criterion above: it needs the connection status shared with the Drivers and Architecture pages, and a Sync that does something.)
- [X] Extend ArchUnit rules to assert that `GitHubProviderAdapter` and `GitLabProviderAdapter` reside in `*.adapter.out.*` and that no key value appears in any type in `*.adapter.in.*` or `*.application.*` (ADR-0009) (`GitArchitectureTest`: the adapters live in `adapter.out` and implement the port; the key type and key-like fields stay out of the inbound adapters, the responses and the status.)
- [ ] Verify the full build and test suite passes within the ten-minute budget (Practice: Ten-Minute Build, Practice: Continuous Integration) (Not met yet: the Java modules build and pass in about 8 minutes, and the frontend module takes about 6 minutes and fails two architecture rules (`layer-rules.test.ts`, "Where the drivers live") on this machine.)

## Files of the spike

In `bwyn-frontend/spikes/home-page/`:

| File | What it holds | Stories |
|---|---|---|
| `GitIntegrationCard.tsx` | The card "Git (GitHub)": the token and repository form, `REPOSITORY` (only `owner/name`), `submit` (saves the token, connects, makes the first commit of an empty repository, then downloads), `downloadAll`, `describeImport` (the summary in words), the connected view, "Change repository" and "Remove the token". | 1, 2 |
| `Sections.tsx` | `Integrations`: the rows, one open at a time; mounts the card. | 1 |
| `data.ts` | The `INTEGRATIONS` list that the other rows come from. | 1 |
| `builder/git/gitClient.ts` | **The browser side of the contract**: `fetchGitStatus`, `saveToken`, `removeToken`, `connectRepo`, `initRepo`, `fetchHead`, `fetchTree`, `fetchBlob`, `fetchPrs`, `uploadFiles` and `GitError`. All calls go to `/api/git/*`. The final client keeps these functions and changes what is behind them. | 1 to 6 |
| `server/github.js` | **The GitHub provider, in the dev server**: `status`, `saveToken` (checked before kept), `connect`, `initialize`, `tree`, `blob`, `prs`, `upload` (branch, GraphQL commit, pull request), `checkFiles` (300 files, 1,000,000 bytes, paths inside the folder), the 403 advice (`PERMISSION_HINT`), the head cache with ETag, and the route table. There is no GitLab file: it is new. | 1 to 4 |
| `server/secrets.js` | Where the dev server keeps keys: one ignored file, `.spike-secrets.json`, mode 600. | 1 |
| `server/bedrock.js` | `readBody` and `sendJson` that the Git server reuses, and the LLM provider with the same per-project key idea (`story-llm-provider.md`). | 1 |
| `builder/git/useGitState.ts` | The connection of each project (`config`) and the record of what was synchronised, in the browser (`bwyn-spike-home-page.git`). `connect` and `disconnect` clear the record. `GitConfig` has no provider or host yet. | 1, 2 |
| `builder/git/syncEngine.ts` | `GitConfig`, `GitState`, `classify`, `computePlan` (tree and pull request states, then the plan), `downloadFiles` (six at a time), `uploadPlan` (titles and body), and the `record*` functions that remember what was downloaded and sent. | 2, 3, 4 |
| `builder/git/useGitSync.ts` | The sync of the document on screen: `check`, `sync`, `forcePull`, `removeMissing`, `autoDownload`, the 30-second look at the head and on focus, and the notices. | 3, 4 |
| `builder/git/SyncButton.tsx` | The button (its hints, its dots), the dialog "Sync with GitHub" (upload, delete, download, awaiting, conflicts, force pull, "not on main"), and the notice bar. | 3, 4 |
| `builder/git/importExisting.ts` | The one-time download: which files, which are left alone, and what is brought in. | 2 |
| `builder/git/scope.ts` | Which files belong to the document on screen and to what is connected with it (`pathMatcher`, `refOfMeta`, the link graph). | 3, 4 |
| `builder/git/deleteFromRepo.ts` | Removes a story or an enabler from the repository by a pull request with deletions. Uses `uploadFiles`. | 3, 5 |
| `builder/git/publishAdr.ts` | Sends one saved ADR as a pull request. Uses `uploadFiles`. | 3, 5 |
| `builder/git/blobSha.ts` | The Git blob SHA of a text, so that local files compare with the provider's. Both providers use Git's blob SHA. | 2 to 6 |
| `builder/repoFiles.ts` | The files BWYN writes and reads (`buildRepoFiles`, `parsePulled`, `isModelPath`, `withBase`): the folder layout of the repository. | 2 to 6 |
| `builder/BuilderWorkspace.tsx`, `Drivers.tsx`, `ArchitecturePage.tsx`, `PrinciplesPractices.tsx`, `AdrPage.tsx` | The pages that mount `useGitSync` and the Sync button, each with its own document. | 3 to 6 |
| `webpack.spike.config.js`, `spike.env.example` | Mounts the dev server middleware; the optional `GITHUB_TOKEN` of `.env.spike`. | 1 |

## Places that say "GitHub" and must take the provider's words

`GitIntegrationCard.tsx` (title, token text, repository link, "Remove the token"); `builder/git/SyncButton.tsx` ("Sync with GitHub",
"Connect GitHub in Integrations first", every "pull request", `repoUrl`, `/blob/`, `/pull/`); `builder/git/useGitSync.ts` (notices);
`builder/git/syncEngine.ts` (the body of the pull request); `builder/git/deleteFromRepo.ts`, `builder/git/publishAdr.ts` (titles, bodies);
`builder/git/importExisting.ts` (a comment about listing); `AdrPage.tsx`, `builder/BuilderWorkspace.tsx`, `builder/FileView.tsx` (the notices
and the removal text); `builder/git/gitClient.ts` (a comment on where the token comes from).

## Spike details worth keeping in mind

- **The token is for the whole dev server, not for a project.** It is one value in `.spike-secrets.json` (or `GITHUB_TOKEN` from
  `.env.spike`), used for every project. The repository settings (`config`) and the record of the synchronisation are per project, in the
  browser. These stories make the key per project, and take the `.env.spike` fallback, "Use another token" and "(token from .env.spike)" out.
- **One click did three things**: keep the token, check the repository, download. The new Connect still does all three in the backend, and
  the card shows the first two as one step and the download as a separate result.
- "Change repository" in the spike disconnects (and clears the record) but keeps the token; "Remove the token" shows only for a token that was
  entered, not for one from `.env.spike`.
- A token that was saved and later refused shows `status.error` ("GitHub did not accept the saved token: ...") in red.
- The first commit of an empty repository is written to the base branch directly, and only there (`initialize`).
- The head is read with an ETag, so that a 304 costs nothing against GitHub's rate limit. GitLab has no equal promise, so the look every 30
  seconds for each open document needs to be checked there.
- The upload is atomic with `expectedHeadOid`: GitHub refuses the commit when the base branch moved. GitLab's `start_sha` has to give the
  same.
- The branch name is `bwyn/<login>-<UTC date and time to the second>`: two syncs by one user in the same second collide.
- The first download does **not** leave an ADR alone: ADR files are not filtered by what exists here, and `mergePulled` sets them by number,
  so an ADR of the repository replaces a local one with the same number (see `story-adrs.md`).
- `canPush` comes from the repository's `permissions.push`, which describes the user, not the key, so it is only a hint.
- The limits (300 files, 1,000,000 bytes) are the dev server's (`MAX_FILES`, `MAX_FILE_BYTES`), not GitHub's.
- The dev server also answers `features: ['deletions']`, so that the app can say that the server is old. The backend does not need it.
