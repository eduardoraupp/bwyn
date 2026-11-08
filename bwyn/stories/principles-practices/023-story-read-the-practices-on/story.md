# Story: Read the practices on the Practices tab of P&P

- **Story**: 023-story-read-the-practices-on
- **Created**: 2026-11-08
- **Status**: Draft

## Story
As a team member, I want to open the Practices tab of the Principles & Practices page and read the practices that every BWYN team follows, so that I know which practices are non-negotiable before I write a story or a decision, and I can trust that they are the same for every project and cannot be changed from the application.

## Acceptance tests

**Reaching the tab**
- [ ] **Given** a project, **When** the user chooses "P&P" in the menu, **Then** the page is titled "Principles & Practices" with the description "The principles this team holds to, and the Extreme Programming and evolutionary architecture practices it follows.", and two tabs: "Principles" (selected) and "Practices" (with a small lock icon).
- [ ] **Given** the P&P page, **When** the user presses the "Practices" tab, **Then** that tab becomes selected (`aria-selected="true"`) and the Practices content replaces the Principles content.
- [ ] **Given** the tab bar, **Then** both tabs are reachable with Tab and activatable with Enter or Space, and a visible focus ring indicates the focused tab.
- [ ] **Given** the user leaves the P&P page and returns, **Then** the Principles tab is selected; the tab choice is not remembered across navigations.

**The tab content**
- [ ] **Given** the Practices tab, **Then** a read-only note with a lock icon reads "Read-only. These practices are part of BWYN and are the same for every project, so they cannot be edited or removed.", and no button to add, edit, or remove anything is present.
- [ ] **Given** the Practices tab, **Then** the first group is "Extreme Programming practices" (amber panel) with the rule "Non-negotiable. The team MUST practise every one of them." and exactly seven practices in this order: Stories, Continuous integration, Ten-Minute Build, Pair programming, Incremental design, Single code base, Code review.
- [ ] **Given** the Practices tab, **Then** the second group is "Evolutionary architecture practices" (green panel) with the rule "Applied through ADRs and architectural enablers. Architectural fitness functions are non-negotiable." and exactly seven practices in this order: Build the skeleton first, Include automation early, Use architectural fitness functions, Defer reversible decisions, Grow incrementally, Design for testability, Align teams with Conway's law.
- [ ] **Given** any practice, **Then** it displays its number within its group, its name, a badge for its group ("Extreme Programming" or "Evolutionary architecture"), its description text, a line beginning "In BWYN:" describing how it shows up in the product, and, when present, a small italic remark.
- [ ] **Given** all fourteen practices, **Then** every Extreme Programming practice carries a "Non-negotiable" label, and in the second group only "Use architectural fitness functions" carries it — eight "Non-negotiable" labels in total.
- [ ] **Given** the displayed texts, **Then** each practice's name, description, "In BWYN" line, and remark match exactly the values in migration script `V202611081001__practices_load_practices.sql`.

**Loading behaviour**
- [ ] **Given** the Practices tab is opened, **Then** the browser sends `GET /api/practices` to the BWYN backend to retrieve the groups and practices.
- [ ] **Given** the response has not yet arrived, **Then** the tab shows "Loading the practices…" below the read-only note.
- [ ] **Given** the backend cannot be reached or returns an error, **Then** the tab shows "The practices could not be loaded." with a "Try again" button, and the Principles tab is unaffected.
- [ ] **Given** the "Try again" button is pressed, **Then** `GET /api/practices` is called again.
- [ ] **Given** two different projects, **Then** both show the same fourteen practices in the same order.
- [ ] **Given** the response, **Then** the two groups appear in display order and the practices within each group appear in display order, with no practice repeated or missing.

**The backend endpoint**
- [ ] **Given** `GET /api/practices`, **Then** the response body matches the shape `{ "groups": [ { "id", "headline", "badge", "rule", "practices": [ { "id", "name", "text", "inBwyn", "nonNegotiable", "note" } ] } ] }` with both groups and all fourteen practices.
- [ ] **Given** any HTTP method other than GET on `/api/practices`, **Then** the backend responds with 405 Method Not Allowed.
- [ ] **Given** the running application, **Then** the OpenAPI document describes `GET /api/practices` with its response schema, and Swagger UI is accessible in non-production profiles.

**The database**
- [ ] **Given** an empty PostgreSQL instance, **When** the application starts, **Then** Flyway applies `V202611081000__practices_create_tables.sql` then `V202611081001__practices_load_practices.sql` in order, and the `practices` schema contains two rows in `practice_group` and fourteen rows in `practice`.
- [ ] **Given** the loaded data, **Then** eight practices have `non_negotiable = TRUE` (all seven in group `xp` and `use-architectural-fitness-functions` in group `ea`), the `display_order` values within each group are 1–7, and every name, description, `in_bwyn`, and `note` value matches the second migration script exactly.
- [ ] **Given** an already-started application that is restarted, **Then** Flyway applies no new migrations and the practice data is unchanged.
- [ ] **Given** either migration script is edited after it was applied, **Then** Flyway's checksum validation prevents the application from starting.

**Consistency between database and frontend list**
- [ ] **Given** the migration data and the `PRACTICE_GROUPS` constant in `practices.ts`, **Then** a test asserts that every practice's name, description, `inBwyn`, note, `nonNegotiable` flag, and position within its group are identical in both sources, so the two cannot drift until `practicesText()` and `practicesMarkdown()` are migrated in a later story.

## Business rules and constraints
- Practices are fixed data for every project and can never be created, edited, or deleted through the application. They are hardcoded in Flyway migrations; the only way to change a practice is a new migration script (ADR 0010).
- The `practices` PostgreSQL schema is owned exclusively by the `practices` module. No other module reads or writes it directly (Architecture: modular monolith).
- Hibernate DDL auto-generation is disabled; `spring.jpa.hibernate.ddl-auto` is set to `validate` or `none` in all environments. Flyway is solely responsible for schema state (ADR 0011, ADR 0010).
- The query side returns read models, not JPA entities, and the endpoint is documented in the OpenAPI specification (ADR 0014, ADR 0017).
- Each practice has a stable slug identifier (e.g. `continuous-integration`) so that references to it remain meaningful across deployments.
- The JSON response shape mirrors the existing `Practice` and `PracticeGroup` types in `practices.ts`, with `id` fields added, so the tab changes where it reads data from without changing how it renders (Principle: Incremental design).
- `practicesText()` and `practicesMarkdown()` in `practices.ts` continue to read from the frontend constant until their own migration story; the consistency test keeps the two sources in sync in the meantime.
- The backend API base URL is configurable and same-origin by default in production, so the frontend client works in both development and production deployment modes (Architecture: deployment modes).

## Out of scope
- The Principles tab and the Sync button (their own stories).
- Sending practices to the AI model and writing `practices.md` from the database; `practicesText()` and `practicesMarkdown()` remain on the frontend list until a later story.
- Adding, editing, translating, ordering, or versioning practices from the application, and project-specific practices.
- ArchUnitTS rules restricting `react-router-dom` imports to the framework/UI layer (covered by AE-0003).
- Choosing the PostgreSQL version and the full module layout of the rest of the backend.

## Tasks
- [ ] Add migration `V202611081000__practices_create_tables.sql` to `src/main/resources/db/migration/` with the `practices` schema and the two tables exactly as in the Reference.
- [ ] Add migration `V202611081001__practices_load_practices.sql` to `src/main/resources/db/migration/` with the two groups and fourteen practices exactly as in the Reference.
- [ ] Write a Testcontainers integration test (`PracticesMigrationIT.java`) that starts a PostgreSQL container, runs both migrations, and asserts the schema, row counts, non-negotiable flags, display orders, and exact text values.
- [ ] Write a test that asserts a second application start applies no new migrations and leaves the data unchanged.
- [ ] Create the `practices` module with `domain`, `application`, and `infrastructure` packages inside the hexagonal structure established by AE-0005 (ADR 0017).
- [ ] Add the `PracticesQuery` port in the application layer and its handler; the handler returns read models, not JPA entities.
- [ ] Add JPA mappings for `practice_group` and `practice` in the infrastructure layer, annotated with `@Table(schema = "practices")`; confirm `ddl-auto` remains `validate` (ADR 0011).
- [ ] Add the outbound adapter that implements `PracticesQuery` using Spring Data projections and returns the read models in display order.
- [ ] Add the `GET /api/practices` REST controller in the inbound adapter with `@Operation` and `@ApiResponse` Swagger annotations; reject all other HTTP methods with 405 (ADR 0014).
- [ ] Write a controller integration test (`PracticesControllerIT.java`) against Testcontainers that verifies the response shape, group and practice order, and 405 for non-GET methods.
- [ ] Verify the generated OpenAPI document describes `GET /api/practices` correctly (`tsc --noEmit` equivalent: `./mvnw verify` with springdoc assertion).
- [ ] Confirm ArchUnit rules still pass with the new `practices` module added (ADR 0009).
- [ ] Write the consistency test that compares the migration data with `PRACTICE_GROUPS` in `practices.ts` (name, description, `inBwyn`, note, `nonNegotiable`, order).
- [ ] Add a typed HTTP client in the frontend (`src/use-cases/` or `src/interface-adapters/`) that calls `GET /api/practices` and maps the response to the existing `PracticeGroup` shape with `id` fields added (ADR 0007).
- [ ] Update `PracticesTab` in `PrinciplesPractices.tsx` to fetch groups from the client, show "Loading the practices…" while waiting, and show "The practices could not be loaded." with a "Try again" button on error; remove the direct reference to `PRACTICE_GROUPS` from the tab renderer.
- [ ] Keep `PRACTICE_GROUPS` in `practices.ts` and confirm `practicesText()` and `practicesMarkdown()` still compile and return the same output.
- [ ] Run the full build and test suite and confirm it completes within ten minutes (Practice: Ten-Minute Build).


### Script 1: `V202611081000__practices_create_tables.sql`

```sql
CREATE SCHEMA practices;

COMMENT ON SCHEMA practices IS 'Module practices: the fixed practices of the BWYN way of working.';

CREATE TABLE practices.practice_group (
    id            VARCHAR(16)  NOT NULL,
    display_order SMALLINT     NOT NULL,
    headline      VARCHAR(120) NOT NULL,
    badge         VARCHAR(60)  NOT NULL,
    rule          VARCHAR(300) NOT NULL,
    CONSTRAINT pk_practice_group PRIMARY KEY (id),
    CONSTRAINT uq_practice_group_display_order UNIQUE (display_order)
);

CREATE TABLE practices.practice (
    id             VARCHAR(64)  NOT NULL,
    group_id       VARCHAR(16)  NOT NULL,
    display_order  SMALLINT     NOT NULL,
    name           VARCHAR(120) NOT NULL,
    description    TEXT         NOT NULL,
    in_bwyn        TEXT         NOT NULL,
    non_negotiable BOOLEAN      NOT NULL,
    note           TEXT,
    CONSTRAINT pk_practice PRIMARY KEY (id),
    CONSTRAINT fk_practice_group FOREIGN KEY (group_id) REFERENCES practices.practice_group (id),
    CONSTRAINT uq_practice_group_order UNIQUE (group_id, display_order)
);

COMMENT ON TABLE practices.practice_group IS 'Fixed reference data. The same for every project. Changed only by a new migration.';
COMMENT ON TABLE practices.practice IS 'Fixed reference data. The same for every project. Changed only by a new migration.';
COMMENT ON COLUMN practices.practice.in_bwyn IS 'How the practice shows up in BWYN.';
```

### Script 2: `V202611081001__practices_load_practices.sql`

```sql
INSERT INTO practices.practice_group (id, display_order, headline, badge, rule) VALUES
    ('xp', 1, 'Extreme Programming practices', 'Extreme Programming', 'Non-negotiable. The team MUST practise every one of them.'),
    ('ea', 2, 'Evolutionary architecture practices', 'Evolutionary architecture', 'Applied through ADRs and architectural enablers. Architectural fitness functions are non-negotiable.');

INSERT INTO practices.practice (id, group_id, display_order, name, description, in_bwyn, non_negotiable, note) VALUES
    ('stories', 'xp', 1, 'Stories',
        'Work is told in small stories that are easy to iterate on and test, giving a fast feedback loop. A story is small enough for a pair to finish in one iteration, and what does not fit is left out of it.',
        'Every story starts from acceptance criteria written as Given/When/Then, and the builder drafts the smallest valuable slice and names what it leaves out under Out of scope.',
        TRUE, NULL),
    ('continuous-integration', 'xp', 2, 'Continuous integration',
        'Changes are integrated and the full test suite is run many times a day, so integration problems show up within hours and not at the end.',
        'A change merges only when the build passes. Changes reach the main branch through pull requests.',
        TRUE, NULL),
    ('ten-minute-build', 'xp', 3, 'Ten-Minute Build',
        'The full build and tests finish in about ten minutes. Regressions beyond that are fixed before new feature work.',
        'The ten-minute budget is one of the gates a change has to pass before it merges.',
        TRUE, NULL),
    ('pair-programming', 'xp', 4, 'Pair programming',
        'Two people work on the code together. In BWYN, the AI depends on the human for review and feedback on each iteration.',
        'The AI proposes and the human decides: a drafted story, enabler or ADR is only kept when the human approves it, and nothing is accepted without that review.',
        TRUE, NULL),
    ('incremental-design', 'xp', 5, 'Incremental design',
        'Design grows in small steps, iteratively, as the stories need it, not all upfront.',
        'Architecture grows through ADRs and the architectural enablers that implement them, added when a story needs them and not before.',
        TRUE, NULL),
    ('single-code-base', 'xp', 6, 'Single code base',
        'One code base for the system, with no long-lived forks or copies.',
        'The drivers, stories, enablers, ADRs and diagrams of a project live in one repository.',
        TRUE, NULL),
    ('code-review', 'xp', 7, 'Code review',
        'Every pull request is reviewed before it is merged, to check that it follows the principles and practices of the team.',
        'Syncing opens pull requests, so every change has a place to be reviewed before it reaches the main branch.',
        TRUE, 'Not a classic XP practice, but it is kept here among the Extreme Programming practices of the team.'),
    ('build-the-skeleton-first', 'ea', 1, 'Build the skeleton first',
        'Implement a vertical slice of real functionality that crosses all the technical tiers, instead of building horizontal layers in isolation, such as finishing the whole database schema before anything uses it. A thin end-to-end slice proves the architecture works and exposes the surprises early, while they are cheap.',
        'Architectural enablers put the skeleton in place and stories are built on it. The reference slice of the dependency injection enabler (port, use case, adapter, view model and view) proves the wiring end to end.',
        FALSE, NULL),
    ('include-automation-early', 'ea', 2, 'Include automation early',
        'Set up the automated build, the deployment scripts and the continuous integration pipeline from day one, before there is much to build. Automation that arrives late is paid for with manual work, and with defects that a pipeline would have caught.',
        'The build, the type check and the test run were put in place as a Maven module by the first enabler, before any story was built on it.',
        FALSE, NULL),
    ('use-architectural-fitness-functions', 'ea', 3, 'Use architectural fitness functions',
        'Write automated, objective checks into the build pipeline, inspired by evolutionary computing, that measure whether the architecture still has the qualities it must keep: coupling limits, latency caps, security compliance. They guide change safely, because a change that breaks an architectural property fails the build.',
        'ArchUnit rules encode the module, layer and dependency constraints of the backend (ADR 0005). A change merges only when all fitness functions pass. Frontend rules need their own ADR and enabler.',
        TRUE, 'Architectural fitness functions MUST encode the architectural constraints and run in the build.'),
    ('defer-reversible-decisions', 'ea', 4, 'Defer reversible decisions',
        'Postpone major framework and infrastructure choices until the last responsible moment, so that flexibility is kept while requirements are still moving. Decide early only what is costly to change, and learn before committing to the rest.',
        'A decision is recorded as an ADR when it is needed. One that proves wrong is replaced by a new ADR that supersedes it, and a change to the technology stack needs an ADR.',
        FALSE, NULL),
    ('grow-incrementally', 'ea', 5, 'Grow incrementally',
        'Expand the skeleton by adding features one at a time, keeping the system continuously executable and deployed. At every step there is a working system, never a pile of unfinished parts that only work together at the end.',
        'Each story adds one slice on top of the skeleton, and the architecture changes only through the ADRs and enablers that a story needs. The database changes through small, incremental Flyway migrations.',
        FALSE, NULL),
    ('design-for-testability', 'ea', 6, 'Design for testability',
        'Write automated tests into the skeleton, so that structural changes can be verified immediately instead of feared. Code that is easy to test is also easier to change: dependencies are visible and can be substituted.',
        'Dependencies are injected and depend on abstractions, so use cases and view models can be tested with substitutes. Acceptance criteria are automated as BDD tests with mocked adapters, and adapters have integration tests against real infrastructure.',
        FALSE, 'BDD acceptance tests and integration tests are required. Isolated unit tests are not.'),
    ('align-teams-with-conways-law', 'ea', 7, 'Align teams with Conway''s law',
        'Structure the development teams to mirror the architecture they must produce, so that the communication paths of the teams match the boundaries of the modules and bottlenecks do not appear between them. An organisation tends to build systems that copy its own structure, so the structure is chosen on purpose.',
        'The system is a modular monolith whose modules expose only explicit public APIs, so a module can be owned by one team without reaching into the internals of another.',
        FALSE, NULL);
```