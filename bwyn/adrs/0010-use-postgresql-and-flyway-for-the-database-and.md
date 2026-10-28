# 0010. Use PostgreSQL and Flyway for the Database and Its Evolution

- **Status**: Accepted
- **Date**: 2026-10-28

## Context

The bwyn backend (ADR-0008) requires a persistent data store. A relational database is a natural fit for structured, transactional data, and a migration tool is needed to evolve the schema in a controlled, repeatable way across development, test, and production environments. Without versioned schema migrations, database state can diverge between environments, making deployments unreliable and rollbacks difficult.

PostgreSQL is a widely adopted, open-source relational database with strong support for ACID transactions, rich data types, and a mature ecosystem. Flyway is a database migration tool that manages schema evolution through versioned SQL scripts, integrating directly with Spring Boot's auto-configuration (ADR-0008) so that migrations are applied automatically on application startup.

## Decision

We will use PostgreSQL as the relational database for the bwyn backend. We will use Flyway to manage all schema changes, with versioned SQL migration scripts stored in the repository and applied automatically by Spring Boot's Flyway auto-configuration on startup. No schema changes will be made outside of Flyway-managed scripts.

## Consequences

**Positive**
- Flyway's versioned migration scripts ensure that database schema and application code evolve together, keeping all environments (development, CI, production) consistent.
- Spring Boot's Flyway auto-configuration (ADR-0008) requires minimal setup and applies pending migrations automatically at startup, reducing operational overhead.
- Storing migration scripts in version control makes schema history auditable, reviewable, and reproducible from scratch.
- PostgreSQL's maturity, rich feature set, and broad ecosystem support reduce risk and provide well-maintained integrations with Spring Data JPA.

**Negative**
- Flyway's versioned migration model is append-only by convention; correcting a mistake in an already-applied migration requires a new migration script, which adds overhead during early, fast-moving development.
- Running PostgreSQL locally and in CI requires infrastructure provisioning (e.g. Docker containers); this adds a small setup cost for new developers.
- Flyway migrations that include destructive changes (column drops, table renames) must be handled carefully to avoid data loss in production; the team must establish a review discipline for such scripts.

**Neutral**
- A baseline Flyway configuration and an initial schema migration script must be established as architectural enablers before feature development begins; this work should be coordinated with the project structure defined under ADR-0008.
- Decisions about the Spring Data access layer (e.g. Spring Data JPA with Hibernate, or an alternative such as jOOQ) will need to be addressed in a follow-up ADR or ticket.
- Any future change of database engine would require a superseding ADR and a review of all existing migration scripts for compatibility.
