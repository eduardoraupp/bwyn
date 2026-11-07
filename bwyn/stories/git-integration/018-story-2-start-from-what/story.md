# Story: Start from what the repository already has

- **Story**: 018-story-2-start-from-what
- **Created**: 2026-11-07
- **Status**: Draft

## Story
As a team member, I want connecting to a repository to bring in the documents that are already there (and prepare an empty repository for its first commit), so that I start from what my team has and never from a blank project.

## Acceptance tests

**Empty repository**
- [X] **Given** a repository with no commit, **When** the user connects, **Then** BWYN makes its first commit on the base branch containing an empty `README.md` in the configured folder with the message "BWYN: first commit", and the connection completes successfully
- [X] **Given** the provider refuses an empty file, **Then** the `README.md` holds the single line `# <repository name>` instead
- [X] **Given** the first commit was made, **Then** a note says "The repository was empty, so BWYN made its first commit: an empty README.md in &lt;folder&gt;/ on &lt;branch&gt;." and BWYN never reads that `README.md` as a document

**One-time download**
- [ ] **Given** a repository with files in the configured folder, **When** the user connects, **Then** every file BWYN knows (drivers, architecture, runways, principles, ADRs, architectural enablers, stories with their diagrams, task lists, and UI examples) is downloaded once from the base branch; files outside the folder and files BWYN does not recognise are ignored
- [ ] **Given** a document with the same identity already exists in the project (a driver, a principle, a story, an enabler, the architecture, a runway), **Then** it is left alone and will be compared at the next sync
- [X] **Given** `practices.md` is present in the repository, **Then** it is not downloaded because practices are fixed in BWYN
- [X] **Given** the download completes, **Then** a note says "Downloaded once from &lt;branch&gt; when connecting: 3 drivers, the architecture, 2 principles, 2 ADRs and 4 stories with 3 diagrams. Documents that already exist here were left alone, and the practices are fixed in BWYN, so they are not downloaded."
- [X] **Given** nothing was brought in (folder is empty or everything already exists), **Then** the note says "There was nothing to download from &lt;branch&gt; yet, or everything is already here."
- [X] **Given** the provider could not list the complete repository contents, **Then** an amber line says "&lt;Provider&gt; could not list the whole repository, so some files may be missing. Sync a document to get it."
- [X] **Given** a file that could not be understood (for example a story found in two folders), **Then** a warning names that file and nothing is deleted
- [X] **Given** the files that were downloaded, **Then** their versions are stored as the last synchronised versions so that the first Sync compares against them

**Download failure**
- [X] **Given** the connection succeeded and the download failed, **Then** an error says "Connected, but what is in the repository could not be downloaded: &lt;reason&gt;", the card remains "Connected", and an amber box says "What is in &lt;branch&gt; has not been downloaded into this project yet." with a "Download from &lt;branch&gt;" button that reads "Downloading…" while in progress
- [X] **Given** the download completed successfully once, **Then** it is not triggered again automatically; **Given** the page is reloaded, **Then** the amber box does not reappear

**GitLab**
- [X] **Given** a GitLab project, **Then** every acceptance test above holds, with "GitLab" in place of "GitHub" and "project" in place of "repository" where the screen uses those words
- [X] **Given** a GitLab project whose file list spans more than one page of API results, **Then** all files are listed and the list is never reported as cut
- [X] **Given** a GitLab project nested inside groups and subgroups, **Then** its files are found and downloaded

## Business rules and constraints

- The download happens exactly once: at connection time, or via the "Download from &lt;branch&gt;" button after a failure. After that, Sync moves changes (stories 3–6) (Architecture: backend serves the frontend only as static resources; the browser never holds the API key)
- The download never overwrites an existing document and never deletes anything
- The first commit on an empty repository is the only time BWYN writes directly to the base branch; all subsequent writes go through a pull request or merge request
- All repository API calls (`tree`, `blob`, `init`) are made from the backend; the browser never receives or uses the provider API key (Architecture)
- Assumption: the configured folder and base branch are those set when the repository was connected (story 017)
- Assumption: "documents BWYN knows" means drivers, architecture, runways, principles, ADRs, architectural enablers, and stories with their diagrams, task lists, and UI examples — as listed in the acceptance tests

## Out of scope

- Sending and receiving changes after the initial download (stories 3–6 of the Git integration)
- Choosing which documents to download selectively
- Force-pulling a document to replace an existing one (story 4 of the Git integration)
- Connecting a repository (covered by story 017)
- Encryption of the provider API key at rest (deferred to a separate ADR)

## Tasks

- [ ] Define `tree`, `blob`, and `init` operations in the OpenAPI document (ADR-0014), placing the controller in `bwyn-rest` and the port interfaces in `bwyn-core` (ADR-0017)
- [X] Implement the `init` operation in the GitHub outbound adapter (`bwyn-infrastructure`): commit an empty `README.md` (falling back to `# <repo name>` if the provider rejects an empty file) on the base branch with the message "BWYN: first commit" (ADR-0017)
- [X] Implement the `tree` operation in the GitHub outbound adapter: list all files under the configured folder from the base branch, returning each file's path and blob SHA and a flag indicating whether the list was truncated (ADR-0017)
- [X] Implement the `blob` operation in the GitHub outbound adapter: fetch the text of one file by its SHA (ADR-0017)
- [X] Implement the `importExisting` application service in `bwyn-application`: orchestrate `tree` + `blob` calls, apply the "exists here, leave alone" rule per document kind, skip `practices.md`, store downloaded versions as the last-synchronised baseline, and record that the download was completed (ADR-0017)
- [X] Implement the frontend ViewModel and UI feedback in the interface-adapter and UI layers: first-commit note, download summary note, cut-list amber line, per-file warnings, download-failure amber box with "Download from &lt;branch&gt;" button, and suppression of the box after a successful download (AE-0002, ADR-0007)
- [X] Write Jest unit tests for the download ViewModel covering: empty-repository note, full-download summary, nothing-to-download note, cut-list line, unrecognised-file warning, failure state with retry button, and post-success suppression (ADR-0005)
- [X] Write backend unit tests (`*Test.java`) for the `importExisting` application service covering: empty repository, repository with every document kind, documents that already exist, `practices.md` skipped, truncated list, and unrecognisable file (ADR-0009)
- [ ] Write a GitHub integration test (`*IT.java`) against a fake GitHub server covering: empty repository (and provider-rejects-empty-file variant), repository with every document kind, documents that already exist, and a truncated file list (ADR-0012)
- [X] Implement the `init`, `tree`, and `blob` operations in the GitLab outbound adapter, handling pagination so the full file list is always returned regardless of page count, and supporting projects nested in groups and subgroups (ADR-0017)
- [ ] Write a GitLab integration test (`*IT.java`) against a fake GitLab server covering: all cases from the GitHub test, a multi-page file list, and a subgroup-nested project (ADR-0012)
- [ ] Verify the full build and test suite completes within the ten-minute budget (Practice: Ten-Minute Build)

## Verification notes

Verified by `mvn verify` (core, application, rest, infrastructure and runtime modules, with Testcontainers Postgres), `tsc`, `eslint` and Jest (49 suites, 865 tests).

Left open on purpose:

- **Acceptance tests 4 and 5** (every kind of file; leave-alone for stories, enablers, runways): the product has no store yet for runways, architectural enablers, stories, diagrams, task lists or UI examples, so only the architecture, drivers, principles and ADRs are downloaded. The summary note lists the kinds that exist.
- **Task 1**: `tree`, `blob` and `init` are operations of the provider port in `bwyn-core`, called only by the backend; the browser never calls them, so the OpenAPI document describes only `POST /api/projects/{id}/git-connection/import` and the new `imported` and `firstCommit` fields.
- **Task 9**: `GitConnectionIT` covers an empty repository, every supported kind, documents that already exist, a failed download and a second download. The provider that refuses an empty file and the truncated list are covered by the adapter tests (`GitHubProviderAdapterTest`, `GitLabProviderAdapterTest`) and the handler test, not by the IT.
- **Task 11**: there is no GitLab IT against a fake GitLab server; GitLab is covered by `GitLabProviderAdapterTest` (first commit, paging, subgroup, blob) and the view-model tests.
- **Task 12**: the ten-minute build was not measured. The Java modules take about 4 minutes; the frontend build and Jest take several minutes more on this machine.
- A GitLab list of more than 50 pages (5,000 files) is reported as cut short instead of being listed.
- A fixed driver's optional hint line written by the spike is not stripped from the imported text; only the "Not written yet" placeholder is.

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
