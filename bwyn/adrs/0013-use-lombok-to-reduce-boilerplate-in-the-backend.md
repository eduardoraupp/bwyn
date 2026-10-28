# 0013. Use Lombok to Reduce Boilerplate in the Backend

- **Status**: Accepted
- **Date**: 2026-10-28

## Context

The bwyn backend is built on Spring Boot (ADR-0008) using Java as the implementation language. JPA entity classes (ADR-0011) and other domain and infrastructure objects routinely require large amounts of repetitive code: constructors, getters, setters, `equals`, `hashCode`, `toString`, and builder methods. Writing and maintaining this boilerplate by hand increases the size of each class, obscures the meaningful parts of the code, and introduces a source of mechanical errors.

Lombok is a Java annotation-processing library that generates this boilerplate at compile time from concise annotations (e.g. `@Getter`, `@Setter`, `@Builder`, `@RequiredArgsConstructor`, `@EqualsAndHashCode`). Because generation happens during compilation, no Lombok code appears in the compiled bytecode or at runtime, and the library does not affect the Spring Boot application context (ADR-0008) or the Hibernate entity model (ADR-0011).

## Decision

We will use Lombok in the bwyn backend to eliminate repetitive boilerplate. Lombok annotations will be used on entity classes, DTOs, and other value-carrying objects to generate constructors, accessors, builders, and `equals`/`hashCode`/`toString` implementations. We will not use Lombok annotations that obscure control flow or introduce non-obvious behaviour (e.g. `@SneakyThrows`); their use must be justified and approved in code review.

## Consequences

**Positive**
- Entity and DTO classes become significantly shorter and easier to read, with attention focused on fields and business-relevant annotations rather than mechanical accessor code.
- Lombok's `@Builder` and `@RequiredArgsConstructor` integrate cleanly with Spring's dependency injection model (ADR-0008) and Inversify-style constructor injection patterns, keeping wiring explicit without verbose constructor bodies.
- Reducing hand-written `equals` and `hashCode` implementations lowers the risk of subtle bugs in JPA entity identity comparisons (ADR-0011).
- Lombok is widely adopted in the Spring/Java ecosystem, so most developers are already familiar with its common annotations.

**Negative**
- Lombok requires IDE plugin support (e.g. the IntelliJ IDEA or Eclipse Lombok plugin) to avoid false-positive compilation errors in the IDE; all contributors must install the plugin, adding a small onboarding step.
- Annotation processing can occasionally interact unexpectedly with other processors (e.g. MapStruct, if introduced later) or with Hibernate's bytecode enhancement; such interactions must be monitored.
- Generated code is invisible in the source tree, which can make debugging or stack-trace interpretation slightly less straightforward for developers unfamiliar with Lombok.
- Restricting certain annotations (e.g. `@SneakyThrows`) requires a team convention and code-review discipline to enforce consistently.

**Neutral**
- The Lombok dependency must be declared as an `annotationProcessor` (and `compileOnly`) scope dependency in the build configuration, coordinated with the baseline project structure established under ADR-0008.
- ArchUnit rules (ADR-0009) are unaffected by Lombok, as ArchUnit analyses compiled bytecode rather than source annotations; no rule changes are required.
- Hibernate DDL auto-generation remains disabled (ADR-0011); Lombok-generated `equals`/`hashCode` on entity classes must be reviewed to ensure they do not include mutable or lazy-loaded fields that could cause issues with Hibernate's session management.
- If a future ADR introduces a mapping library such as MapStruct, annotation processor ordering between Lombok and MapStruct will need to be explicitly configured in the build.
