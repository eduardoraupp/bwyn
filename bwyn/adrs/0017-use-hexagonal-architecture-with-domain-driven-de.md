# 0017. Use Hexagonal Architecture with Domain-Driven Design for the Backend

- **Status**: Accepted
- **Date**: 2026-10-28

## Context

The bwyn backend is built on Spring Boot (ADR-0008), uses Spring Data JPA with Hibernate for persistence (ADR-0011), and has its architectural layer boundaries enforced by ArchUnit fitness functions (ADR-0009). As the backend grows in complexity, a clear structural pattern is needed to organise business logic, separate domain concerns from infrastructure, and keep the codebase maintainable and testable.

Without an explicit architecture, business logic tends to leak into Spring controllers or JPA repository implementations, making units hard to test in isolation and the codebase hard to navigate. The frontend already applies a layered separation of concerns via Clean Architecture and MVVM (ADR-0007); a complementary, explicit structure is needed on the backend side.

Hexagonal Architecture (also known as Ports and Adapters) structures the application around a central domain core that is entirely independent of frameworks, databases, and transport mechanisms. The domain core communicates with the outside world exclusively through ports (interfaces) and adapters (implementations of those interfaces). Inbound adapters (e.g. Spring MVC controllers) drive the application; outbound adapters (e.g. Spring Data JPA repositories) are driven by it. This makes the domain core independently testable and replaceable at its boundaries.

Domain-Driven Design (DDD) provides a set of strategic and tactical patterns for modelling complex business domains. Tactically, the relevant building blocks are: entities (objects with identity), value objects (immutable, identity-free objects), aggregates (consistency boundaries around clusters of entities and value objects), domain services (stateless operations that do not belong to a single entity), repositories (aggregate-scoped persistence interfaces defined in the domain), domain events (records of significant occurrences within the domain), and application services (orchestrators of use cases that coordinate domain objects and ports). Strategically, the codebase will be organised around bounded contexts that reflect distinct areas of the business domain.

Together, Hexagonal Architecture and DDD reinforce each other: DDD defines what belongs in the domain core and how it is modelled; Hexagonal Architecture defines how that core is isolated from and connected to infrastructure. Spring Boot (ADR-0008) and Spring Data JPA (ADR-0011) are confined to the adapter layer, leaving the domain core free of framework annotations. Lombok (ADR-0013) may be used in adapter-layer classes but must be used with care on domain objects to avoid obscuring domain semantics. ArchUnit fitness functions (ADR-0009) will enforce the resulting layer and dependency rules.

## Decision

We will structure the bwyn backend using Hexagonal Architecture (Ports and Adapters) combined with Domain-Driven Design tactical patterns. Concretely:

- The **domain core** will contain entities, value objects, aggregates, domain services, domain events, and repository interfaces (ports). It will have no dependency on Spring, JPA, or any other framework or infrastructure library.
- **Application services** will orchestrate use cases by coordinating domain objects and invoking ports; they will reside in an application layer that depends on the domain core but not on infrastructure.
- **Inbound adapters** (e.g. Spring MVC controllers, event listeners) will translate external requests into application service calls.
- **Outbound adapters** (e.g. Spring Data JPA repository implementations, external HTTP clients) will implement the outbound ports defined in the domain core.
- We will organise the codebase by **bounded context**, with each bounded context owning its own domain core, application layer, and adapters.
- All schema changes remain the responsibility of Flyway (ADR-0010); JPA mappings in outbound adapters must be kept in sync with migration scripts manually.
- ArchUnit rules (ADR-0009) will be extended to enforce that the domain core has no outward dependencies on Spring, JPA, or adapter-layer classes, and that inbound and outbound adapters depend only on ports defined in the domain or application layer.

## Consequences

**Positive**
- The domain core is entirely free of framework dependencies, making domain logic straightforward to unit-test with plain JUnit without loading a Spring application context (ADR-0008).
- Ports and adapters make infrastructure concerns (database, HTTP, messaging) replaceable without touching the domain; this reduces the risk of framework lock-in and simplifies future infrastructure changes.
- DDD tactical patterns provide clear conventions for where new code belongs (entity vs. value object vs. domain service vs. application service), reducing ambiguity as the team grows.
- Bounded context boundaries provide natural module seams, supporting future decomposition into separate deployable units if required.
- ArchUnit (ADR-0009) can enforce the strict inward-pointing dependency rule of the hexagonal model, providing automated, continuous verification of structural integrity.

**Negative**
- Hexagonal Architecture introduces more files, interfaces, and indirection than a flat or layered structure; the initial setup cost and ongoing navigation overhead are higher.
- Mapping between domain objects and JPA entities (in outbound adapters) requires explicit translation code; this is more verbose than annotating domain objects directly with `@Entity`, but is necessary to keep the domain core free of JPA.
- Developers unfamiliar with Hexagonal Architecture or DDD tactical patterns face a learning curve before they can contribute confidently; the distinction between entities, value objects, aggregates, and application services must be understood and consistently applied.
- Strict port/adapter boundaries require discipline to maintain; without code-review and ArchUnit enforcement, shortcuts that bypass ports can accumulate.

**Neutral**
- A reference package structure covering domain core, application layer, inbound adapters, and outbound adapters must be established as an architectural enabler before feature development begins; this work should be coordinated with the baseline project structure from ADR-0008.
- ArchUnit rules (ADR-0009) must be extended to cover the hexagonal dependency constraints (domain core has no dependency on Spring or JPA, adapters depend only on ports) as part of the same enabler work.
- Hibernate DDL auto-generation must remain disabled (ADR-0011); JPA `@Entity` annotations will appear only in outbound adapter classes, not on domain objects, and Flyway remains solely responsible for schema state (ADR-0010).
- Lombok (ADR-0013) should be used with care on domain value objects and entities: generated `equals`/`hashCode` must reflect domain identity rules (identity-based for entities, structural for value objects) and must not include lazy-loaded or mutable fields that could cause Hibernate issues.
- Decisions about cross-cutting concerns within the hexagonal model (e.g. transaction boundaries, error handling, domain event dispatch) will need to be addressed in follow-up ADRs or tickets.
- The OpenAPI documentation layer (ADR-0014) is unaffected: Spring MVC controllers remain inbound adapters and continue to be annotated with OpenAPI annotations; no domain core classes should appear in API schemas directly.
