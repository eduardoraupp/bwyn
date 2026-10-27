# 0001. Use of PostgreSQL instead of Oracle DB

- **Status**: Proposed
- **Date**: 2026-10-27

## Context

The cinema ticketing system requires a relational database to store films, showings, seats, and purchase records. Oracle Database is a common enterprise choice for this kind of workload, but it carries significant licensing costs and typically requires dedicated DBA expertise to operate. The project has a single developer and no dedicated operations or QA resource, making the overhead of an Oracle deployment disproportionate. PostgreSQL is a mature, open-source relational database with no licensing cost, broad community support, and strong compatibility with the SQL standard.

## Decision

We will use PostgreSQL as the relational database for the cinema ticketing system instead of Oracle DB.

## Consequences

**Positive**
- No database licensing cost is incurred.
- The developer can run PostgreSQL locally and in production without specialist DBA knowledge.
- A large ecosystem of open-source tooling, drivers, and hosting options is available.

**Negative**
- Oracle-specific SQL features or PL/SQL procedures cannot be used; any future migration of Oracle assets would require rewriting them.
- The team is responsible for database maintenance tasks (backups, upgrades, vacuuming) that a managed Oracle service might otherwise handle.

**Neutral**
- A managed PostgreSQL hosting option (e.g. a cloud provider's managed service) should be evaluated to reduce operational burden on the single developer.
- Connection pooling and query optimisation will still be needed to meet the 2-second response-time requirement under normal load.
