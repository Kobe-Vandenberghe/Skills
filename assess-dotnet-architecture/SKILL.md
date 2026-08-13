---
name: assess-dotnet-architecture
description: Assess a .NET design before implementation by clarifying behavior, sizing architectural pressure, locating the bounded context, mapping existing project and dependency boundaries, classifying driving and driven adapters, defining transaction and consistency boundaries, comparing a few proportional designs, and recording the decision. Use when planning a substantial feature, creating or reshaping a module, moving responsibilities across boundaries, introducing a significant integration, coordinating multiple contexts, or reviewing a proposed architecture. Do not use for routine bug fixes, renames, mechanical refactors, small CRUD endpoints, or changes whose implementation follows an obvious existing pattern.
---

# Assess .NET Architecture

Produce an evidence-based design decision before writing implementation code. Prefer the smallest change that protects real invariants and keeps plausible future changes affordable.

## Operating rules

- Inspect the repository before prescribing architecture. Follow repository instructions and existing terminology.
- Treat architecture as a response to coupling, invariants, ownership, and change pressure, not as a checklist of patterns.
- Preserve existing boundaries unless evidence shows they are the problem.
- Distinguish facts found in the codebase from assumptions and recommendations.
- Ask a focused question only when the answer would materially change ownership, consistency, or the chosen design. Otherwise state the assumption and continue.
- Do not begin implementation until the assessment has been recorded in the response or the repository's established decision format.

## Workflow

### 1. Frame the behavior

Restate the requested outcome in domain language:

- actor or trigger
- successful outcome
- business rules and invariants
- important failure and retry behavior
- data read and changed
- external side effects
- explicit exclusions

Separate required behavior from a proposed technical solution. Call out unresolved assumptions that affect the design.

### 2. Inspect the current architecture

Read only enough of the codebase to trace one representative path from entry point to persistence or external effect. Inspect, when present:

- repository instructions and architecture or decision records
- solution and project files, including project references
- module or feature directories and namespaces
- HTTP endpoints, controllers, message consumers, scheduled jobs, or CLI entry points
- application handlers or use cases
- domain models and policies
- persistence implementations, `DbContext` ownership, migrations, and external clients
- dependency registration and composition roots
- tests that reveal intended boundaries

Summarize the current boundary and dependency model. Do not assume Clean Architecture, DDD, CQRS, or ports and adapters merely from folder names.

### 3. Assess architectural pressure

Evaluate the change across these dimensions:

| Dimension | Low pressure | Higher pressure |
| --- | --- | --- |
| Behavior | familiar operation | new workflow or policy |
| Invariants | local validation | rules spanning multiple objects |
| Ownership | one clear module | contested or cross-context data |
| Consistency | one local transaction | external effects or multiple stores |
| Integration | existing stable adapter | new or volatile external system |
| Change risk | isolated and reversible | broad, operationally risky, or hard to reverse |

Classify the change as low, medium, or high architectural pressure and explain the dominant reasons. File count and endpoint count are not measures of domain complexity.

If pressure is low and the established pattern is clear, recommend following it and stop the architecture exercise. Do not manufacture alternatives.

### 4. Locate the bounded context

Identify the context that owns the behavior by checking:

- the vocabulary used by the business
- who owns the rules and lifecycle
- which data is authoritative
- where the relevant invariants already live
- which existing module changes for the same business reasons

Place behavior in an existing context when ownership is coherent. Recommend a new context or module only when there is a distinct model, lifecycle, authority, or rate of change. A bounded context is a semantic boundary, not automatically a project, assembly, service, or deployment unit.

For cross-context behavior, name the coordinator and the contracts exchanged. Avoid sharing mutable domain entities across contexts.

### 5. Map dependencies and adapters

Describe compile-time dependencies separately from runtime calls. Confirm that business policy does not depend directly on delivery, persistence, or vendor details unless that is the repository's deliberate architecture.

Classify relevant edges:

- **Driving adapters:** initiate behavior, such as HTTP endpoints, message consumers, jobs, CLI commands, or tests.
- **Driven adapters:** are called by the application, such as databases, file stores, queues, email, clocks, identity providers, or third-party APIs.
- **Ports:** stable capabilities or contracts at a meaningful boundary.

Introduce a port only when it protects a real boundary, permits a relevant substitution, or enables deterministic testing of an external effect. Do not create an interface solely because a class exists.

### 6. Define transaction and consistency boundaries

State explicitly:

- what must succeed or fail atomically
- which module owns the transaction
- which data can be temporarily inconsistent
- when external effects occur
- how retries, duplicate delivery, and partial failure are handled

Prefer one local transaction for one consistency boundary. Use eventual consistency, idempotency, an outbox, or compensating action only when an actual cross-boundary failure mode justifies it. Do not default to distributed transactions.

### 7. Compare proportional designs

Compare two or three genuinely reasonable designs when architectural pressure is medium or high. Always include the smallest viable design. Add a more isolated design only when a plausible change axis or boundary motivates it.

Use a compact comparison:

| Design | Boundary fit | Complexity now | Coupling | Future options | Main risk |
| --- | --- | --- | --- | --- | --- |

Reject ornamental variants. Do not propose microservices, event-driven choreography, CQRS, repositories, mediator layers, new projects, or new abstractions without tying them to a concrete pressure discovered above.

### 8. Recommend and record

Recommend one design and make the decision implementable. Include:

1. **Decision:** one concise statement.
2. **Why:** the strongest evidence and tradeoff.
3. **Placement:** owning context, project, and main code locations.
4. **Flow:** driving adapter → application behavior → domain policy → driven adapters.
5. **Dependencies:** allowed direction and any contract introduced.
6. **Consistency:** transaction boundary and external-effect strategy.
7. **Rejected alternatives:** why the other reasonable options lose today.
8. **Consequences:** what becomes easier, what remains coupled, and the likely extraction seam.
9. **Assumptions and follow-ups:** only unresolved items that could change the decision.

Use the repository's established ADR or design-note convention when it exists and the user's request authorizes repository changes. Otherwise record this assessment in the response before implementation. Do not create a permanent ADR for a reversible local decision unless the repository expects one.

## Proportionality guardrails

- Prefer a feature slice inside the current modular monolith when it can own its behavior and data cleanly.
- Add a project boundary only when compile-time enforcement is worth its operational and navigation cost.
- Keep orchestration in the application layer and business invariants with the model that owns them.
- Reference other contexts by contracts and stable identifiers rather than their persistence models.
- Keep persistence and transport models out of domain policy when a meaningful domain layer exists.
- Prefer direct code over an abstraction whose future variation is merely hypothetical.
- Preserve an extraction seam through ownership and contracts; do not pre-build the extracted service.

## Review mode

When reviewing an existing design, use the same workflow but focus the result on findings:

- identify boundary, dependency, or consistency violations with concrete code evidence
- distinguish correctness risks from taste or optional improvements
- propose the smallest corrective change
- state when the current design is already proportional and should remain unchanged
