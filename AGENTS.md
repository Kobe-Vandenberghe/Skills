# Agent Guidelines

## Architecture

- Assess complexity before choosing an architecture. Use the simplest design that handles the current requirements without blocking likely changes.
- Do not introduce DDD, hexagonal architecture, extra layers, or new abstractions without a concrete problem they solve.
- For substantial features, new modules, integrations, or boundary changes, use the `assess-dotnet-architecture` skill before implementation.
- Decide which bounded context or module owns a capability before deciding where its classes belong. Avoid sharing domain concepts merely because their names or data look similar.
- Keep dependencies pointing inward. Domain code must not depend on Application, Infrastructure, transport frameworks, persistence frameworks, or external services.
- Treat inbound APIs, commands, consumers, and scheduled jobs as driving adapters. Treat databases, message brokers, file systems, and external APIs as driven adapters. Connect both to the application through explicit ports when substitution, isolation, or testing justifies the boundary.

## Modular Monolith Boundaries

- `AgenticOS.Api` is the executable host and composition root. It owns process-wide configuration, dependency injection, middleware, authentication setup, logging setup, health checks, and module registration. It must not contain business rules, persistence models, or use-case orchestration.
- Organize each implemented business module into Domain, Application, and Infrastructure projects. Do not create empty modules or projects for hypothetical future capabilities.
- A module's Domain project may reference only the minimal shared domain kernel. Application may reference its Domain. Infrastructure may reference Application and Domain. The API host may reference Infrastructure for module registration and endpoint mapping.
- A module must not reference another module's Infrastructure project, persistence entities, or `DbContext`. Never query another module's tables directly.
- When modules must collaborate, use stable contracts owned by the providing module and exchange identifiers or immutable data. Do not share mutable domain entities across module boundaries.
- Use architecture tests to enforce project-reference and dependency rules that cannot be guaranteed by naming or folders alone.

## Shared Code

- Keep the shared domain kernel deliberately small. It may contain proven primitives such as `Entity<TId>`, `AggregateRoot<TId>`, and `IDomainEvent`, but not module-specific business concepts.
- Do not create a global contracts or utilities dumping ground. A contract used by several modules remains owned by the module that provides it.
- Prefer the framework's established abstractions, such as `ILogger<T>` and `TimeProvider`, instead of wrapping them in application-specific interfaces without a concrete need.
- Share a building block only after its semantics are genuinely identical across consumers. Similar-looking code in two modules is not by itself enough reason to centralize it.

## Domain Model

- For features with meaningful business rules, identity, lifecycle, state transitions, or invariants, use the `model-dotnet-domain` skill before implementation. Do not use rich domain modeling for simple CRUD.
- Use strongly typed IDs for domain entities and aggregate roots. Do not pass primitive identifiers such as `Guid`, `string`, or `int` across domain APIs when their meaning matters.
- Entity equality is based on identity. Value-object equality is based on all semantic components of the value.
- Keep domain objects valid by construction. Validate value objects and required entity state at their creation boundary.
- Prefer intention-revealing behavior methods over public setters. Business rules and state transitions belong with the domain state they protect.
- Enforce invariants inside the domain model rather than relying on controllers, handlers, or callers to remember them.
- Create an aggregate boundary only where consistency and ownership require one. Modify child entities through their aggregate root, and avoid transactions that casually span multiple aggregates.
- Put an aggregate repository or store port in Domain only when it represents access to that aggregate. Do not introduce a generic repository abstraction.

## Application Boundaries

- Application use cases coordinate a business flow, transactions, ports, and domain operations. They should not reimplement business rules owned by the domain.
- Keep use-case requests, results, filters, and DTOs close to the use case that owns them. HTTP transport models belong to the Web API adapter, not the Domain.
- Put ports for databases, Git, file systems, notifications, search indexes, and other external capabilities in the layer that consumes the capability. Implement those ports in Infrastructure.
- Do not create interfaces for use cases or services solely for symmetry. Introduce a port when it protects a real external boundary, supports a relevant substitution, or enables deterministic testing.
- Keep controllers, endpoints, consumers, and persistence mappings thin. Their job is translation and integration, not domain decision-making.
- Follow an established local pattern when it already fits. If a change requires a new pattern or a non-obvious tradeoff, explain and record the decision before implementing it.

## Persistence and Consistency

- Each module owns its persistence schema, `DbContext`, mappings, and migrations. Sharing one PostgreSQL database does not imply shared data ownership.
- A command should normally complete within one module-owned database transaction. State explicitly when a workflow cannot be atomic and which temporary inconsistencies are acceptable.
- Do not hold a database transaction open while calling GitHub, Git, an identity provider, or another remote system.
- Persist durable work before performing an external side effect when failure or retry matters. Workers and integration handlers must be idempotent and safe to retry.
- Domain events communicate facts within the owning consistency boundary. Use explicit integration contracts for cross-module effects, and add an outbox only when a real delivery guarantee requires one.

## Errors, Security, and Observability

- Represent expected validation, not-found, conflict, and business-rule outcomes explicitly. Reserve exceptions for exceptional failures and invalid states, and map uncaught exceptions to safe Problem Details at the API boundary.
- Enforce authorization at the application operation or domain transition that changes protected state. Endpoint policies and hidden UI controls are not sufficient on their own.
- Never log or persist access tokens, secrets, provider response bodies, credential-bearing URLs, or sensitive internal paths. Use stable error codes and sanitized summaries at external boundaries.
- Add structured logs around module boundaries, external operations, retries, and failures. Avoid logging entire domain entities or request bodies by default.

## Testing and Change Discipline

- Test domain invariants and state transitions without infrastructure. Test adapters against their real boundary where practical, including PostgreSQL integration tests for persistence behavior.
- Add an architecture test when introducing a boundary rule that should remain mechanically enforceable.
- Keep changes scoped to the requested behavior. Do not combine feature work with unrelated architectural cleanup or speculative abstraction.
- Record architectural decisions when they establish or change module ownership, dependency direction, consistency boundaries, or an important integration strategy.
