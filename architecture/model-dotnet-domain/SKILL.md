---
name: model-dotnet-domain
description: Model meaningful business behavior in .NET by defining ubiquitous language, entities and value objects, strongly typed aggregate IDs, identity equality, valid-by-construction types, aggregate ownership, state transitions, invariants, transaction boundaries, domain services, commands, results, and domain events. Use when a feature or design has nontrivial business rules, identity, lifecycle, state transitions, or consistency requirements, or when reviewing and refactoring a rich domain model. Do not use for routine CRUD, DTO mapping, infrastructure-only changes, simple validation, or behavior with no meaningful domain decisions; first determine whether DDD is justified.
---

# Model .NET Domain

Turn business rules into the smallest expressive domain model that makes invalid behavior difficult. Treat DDD as an optional toolset, not a mandatory architecture.

Read [references/csharp-domain-patterns.md](references/csharp-domain-patterns.md) when producing or reviewing C# code. Adapt those examples to the repository's conventions instead of copying every building block.

## Operating rules

- Inspect existing code, tests, terminology, and persistence constraints before modeling.
- Begin with behavior and language, not classes, tables, or patterns.
- Separate domain rules from input validation, authorization, orchestration, and infrastructure concerns.
- Preserve established conventions unless they fail to protect a real invariant.
- Make the model no richer than the domain requires.
- Record the modeling decisions before implementation when the choice changes identity, ownership, or transaction boundaries.

## Workflow

### 1. Understand the behavior and language

Extract concrete scenarios, including successful actions, rejected actions, state changes, timing, and concurrency. Build a small glossary of business terms and use one term per concept.

Clarify:

- who performs the action and why
- what identity persists over time
- which facts may change
- which rules must always be true
- which decisions depend on current state
- which outcomes the business recognizes
- which concepts are synonyms and which only appear similar

Do not preserve ambiguous technical names when the business uses a clearer term. Flag conflicting meanings instead of hiding them behind a generic model.

### 2. Decide how much domain modeling is justified

Classify the behavior:

| Domain pressure | Typical shape | Default approach |
| --- | --- | --- |
| Low | storage and retrieval with local field validation | transaction script or existing CRUD pattern |
| Medium | a few rules, meaningful types, or state-dependent actions | application use case plus targeted value objects and behavior |
| High | rich lifecycle, interacting rules, concurrency, or consistency boundary | explicit aggregate model and domain concepts |

Use DDD selectively. Strong types or one behavior-rich entity do not require a full DDD stack. If the feature is low pressure, say so and stop rather than inventing aggregates, repositories, services, or events.

### 3. Classify entities and value objects

For each concept, ask:

- **Entity:** Does continuity matter even when its attributes change? Is it referenced by identity or tracked through a lifecycle?
- **Value object:** Is it defined entirely by its values, interchangeable with an equal value, and replaced as a whole?
- **Primitive or DTO:** Does it merely cross a boundary without domain semantics?

Use value objects when they centralize meaningful validation, normalization, units, or operations. Do not wrap every primitive. Avoid giving value objects surrogate identities or mutable state.

### 4. Define identity and equality

Give every aggregate root its own strongly typed ID, such as `BookingId`, so unrelated identities cannot be mixed. Add typed IDs to internal entities only when their independent identity is meaningful.

Define equality deliberately:

- entities compare by stable identity and concrete entity type
- value objects compare structurally by their components
- collections and mutable attributes do not participate in entity equality
- assign entity IDs at creation when practical to avoid transient-entity equality ambiguity

Account for serialization, EF Core conversion, route binding, and default struct values at system boundaries. Strong typing must not make persistence or APIs silently accept invalid identifiers.

### 5. Make concepts valid by construction

Keep constructors narrow or private and expose named factories when creation can fail. Normalize before validating when normalization is part of the concept.

Choose failure semantics intentionally:

- return a result for expected business rejection or invalid user-supplied values
- throw for programmer errors, corrupted trusted data, or impossible internal states according to repository convention
- collect multiple validation errors only when the caller can act on them together

Do not put database lookups, remote calls, or cross-aggregate queries inside value-object factories. Those belong in orchestration or an explicit policy boundary.

### 6. Draw aggregate ownership

Choose the smallest aggregate boundary that can enforce invariants synchronously. For each root, state:

- what it owns and may mutate
- which child entities and value objects live inside it
- which invariants it protects
- which operations form its public behavior
- which other aggregates it references by typed ID

Expose child collections as read-only views and mutate them only through intention-revealing aggregate methods. Load and save through the aggregate root; do not create repositories for owned children.

Do not make a large object graph one aggregate merely because the UI edits it on one screen. Conversely, if a rule must be atomic across two supposed aggregates, reconsider the boundary before accepting eventual inconsistency.

### 7. Model state transitions and invariants

Represent actions as domain verbs such as `Approve`, `Withdraw`, or `AddLine`, not as public setters. Each operation should:

1. verify state-dependent preconditions
2. reject invalid transitions without partial mutation
3. apply the complete state change
4. raise a domain event only when another behavior genuinely cares that it happened

Make legal transitions visible in tests or a compact transition table. Distinguish invariants from policies that may change independently. Use optimistic concurrency when two valid commands could otherwise violate a state rule.

### 8. Place behavior correctly

Prefer this order:

1. Put behavior on the entity or value object that owns the required state.
2. Use a domain service for a stateless domain policy spanning concepts when no single object naturally owns it.
3. Use an application service or handler for authorization, loading aggregates, transactions, external I/O, retries, and orchestration.
4. Use infrastructure adapters for databases, clocks, queues, files, and third-party systems.

A domain service is not a place for leftover logic. Keep it expressed in ubiquitous language and free of transport or persistence details.

### 9. Define transaction boundaries

Treat an aggregate as the default consistency boundary, not as a guarantee that every operation touches exactly one table. State:

- what must commit atomically
- which aggregate version protects concurrent updates
- what may become eventually consistent
- how cross-aggregate rules are checked and rechecked
- when events or external side effects are published

Do not keep a database transaction open across remote calls. Dispatch integration events after a successful commit; use an outbox only when reliable delivery is a real requirement.

### 10. Define commands, results, and events

Keep their semantics distinct:

- **Command:** an application-level request expressing intent; it may be rejected.
- **Result:** the explicit success or expected failure returned to the caller.
- **Domain event:** an immutable fact that the domain says occurred after a successful state change.
- **Integration event:** a stable, externally published contract created from committed domain state.

Do not turn every method call into a command class or every mutation into an event. Avoid putting aggregates, EF entities, service instances, or mutable collections inside events.

## Required output

Produce a concise domain-model decision containing:

1. **DDD justification:** low, medium, or high domain pressure and why.
2. **Ubiquitous language:** key terms and rejected ambiguous names.
3. **Concept model:** entities, value objects, typed IDs, and their equality rules.
4. **Aggregate boundaries:** roots, owned members, references, and protected invariants.
5. **Behavior:** commands or domain methods, results, and legal state transitions.
6. **Transactions:** atomic boundary, concurrency strategy, and eventual consistency.
7. **Services and events:** only those justified by the behavior.
8. **C# shape:** a focused sketch using repository conventions.
9. **Alternatives rejected:** especially a simpler transaction script or a richer model.
10. **Open assumptions:** only questions that could change identity, ownership, or invariants.

If implementing, create the model only after this decision is recorded. Keep domain tests centered on behavior and invariants rather than private implementation details.

## Review mode

When reviewing an existing domain model:

- report invariant leaks, illegal state exposure, broken equality, and confused ownership as correctness risks
- distinguish anemic modeling that loses real rules from intentionally simple CRUD
- identify persistence concerns leaking into domain behavior
- prefer moving behavior to its natural owner over adding generic services
- recommend the smallest refactor that restores the model's semantics
- say explicitly when the existing simple design is already proportional
