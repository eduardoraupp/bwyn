# 0011. Use Spring Data JPA with Hibernate for Persistence

- **Status**: Accepted
- **Date**: 2026-10-28

## Context

The bwyn backend (ADR-0008) uses PostgreSQL as its relational database, with Flyway managing schema evolution (ADR-0010). Application code needs a way to map Java objects to database tables, execute queries, and participate in transactions without writing low-level JDBC boilerplate for every data-access operation.

Spring Data JPA provides a repository abstraction over the JPA standard, generating common query implementations automatically and integrating with Spring Boot's auto-configuration (ADR-0008). Hibernate is the de-facto standard JPA provider in the Spring ecosystem and ships as the default implementation in Spring Boot's `spring-boot-starter-data-jpa` dependency. Together they offer a consistent, well-documented persistence layer that aligns with the existing technology choices in the project.

## Decision

We will use Spring Data JPA, backed by Hibernate as the JPA provider, for all persistence concerns in the bwyn backend. Entity classes will be mapped using JPA annotations, and data-access operations will be expressed through Spring Data repository interfaces. All schema changes remain the responsibility of Flyway (ADR-0010); Hibernate's DDL auto-generation will be disabled in all environments.

## Consequences

**Positive**
- Spring Data JPA's repository abstraction eliminates boilerplate CRUD and common query implementations, allowing the team to focus on business logic.
- Deep integration with Spring Boot's auto-configuration (ADR-0008) means minimal setup: a single starter dependency wires the data source, entity manager, and transaction manager automatically.
- Hibernate's maturity and widespread adoption provide extensive documentation, community support, and reliable behaviour with PostgreSQL (ADR-0010).
- Spring Data JPA integrates naturally with Spring's transaction management, keeping transactional boundaries declarative and consistent across the application.

**Negative**
- Hibernate's abstraction can obscure the SQL it generates; poorly written queries or fetch strategies (e.g. N+1 problems) may go unnoticed without deliberate query logging and review.
- JPA's object-relational mapping model can become awkward for complex queries or non-trivial schema shapes; workarounds (native queries, projections) add complexity.
- Disabling DDL auto-generation (in favour of Flyway) requires that entity mappings and migration scripts are kept manually in sync, which demands discipline during schema changes.

**Neutral**
- Hibernate DDL auto-generation (`spring.jpa.hibernate.ddl-auto`) must be explicitly set to `validate` or `none` in all environment configurations as an architectural enabler, coordinated with the Flyway setup from ADR-0010.
- Decisions about integration-testing strategy for the persistence layer (e.g. use of Testcontainers, `@DataJpaTest` slice tests, or an in-memory substitute) will need to be addressed in a follow-up ADR or ticket.
- If complex reporting queries or bulk operations prove poorly suited to JPA, a follow-up ADR may introduce a complementary query approach (e.g. jOOQ or native queries) for those specific cases.
- ArchUnit rules (ADR-0009) should be extended to enforce that repository interfaces reside in the correct layer and are not referenced directly from use-case or entity layers.
