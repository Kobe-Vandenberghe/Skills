---
name: architecture-proportionality
description: >
  Assess whether a feature, module, or change justifies its architectural
  complexity before designing or implementing it. Use at the start of any
  architecture-planning or implementation task — new features, endpoints,
  services, refactors, or "just implement X" requests — to decide how much
  structure (plain CRUD vs. service boundaries vs. entities/aggregates vs.
  bounded contexts/events) the problem actually warrants, and to catch cases
  where a request should stop for a design conversation instead of being
  built as asked.
---

# Architecture proportionality

Architectural complexity should match problem complexity, not habit, not the
richest pattern the agent knows, and not blind imitation of the most complex
part of the codebase.

This assessment runs **before** design or implementation work, even when the
user says "just implement it." A request phrased as a quick task is not
permission to skip the assessment — it's often exactly when skipping it does
the most damage, because nothing else in the request forces a second look.

## Core rule

Preserve the existing architecture unless the requested change exposes a
concrete design problem. Introduce a pattern only when it protects a real
boundary, invariant, or source of change.

"Might need it later," "more consistent," and "more professional" are not
justifications on their own. Every added layer, interface, or abstraction is
a cost — it must be paid for by a concrete thing it protects.

## Inspect before deciding

Before choosing an approach, look at:

- **Existing repository structure and conventions** — what pattern do
  neighboring features actually use? Consistency with the surrounding
  codebase is itself a form of proportionality.
- **Size and expected lifetime of the feature** — a throwaway internal tool
  and a core product surface don't earn the same investment.
- **Number and complexity of business rules** — one or two conditionals is
  not a rules engine.
- **Whether state has meaningful lifecycle transitions** — does the thing
  move through stages with rules about what's allowed when, or is it just
  data that gets replaced?
- **Whether external systems are involved** — I/O, third-party APIs, or
  processes outside the app's control raise the value of isolating boundaries.
- **Expected change and volatility** — is this an area that shifts often
  and needs to absorb change cheaply, or is it stable?
- **Transactional and consistency requirements** — does correctness depend
  on atomicity, ordering, or eventual consistency across boundaries?
- **Testing requirements** — does the logic need isolated unit testing
  independent of infrastructure, or is integration-level testing sufficient?

Don't skip this because the request sounds simple. The inspection is what
tells you whether it actually is.

## The scale

| Complexity | Signals | Likely architecture |
|---|---|---|
| Simple CRUD | Data in, data out, no real rules, no lifecycle | Endpoint, application logic, persistence |
| Moderate feature | A few explicit rules, identifiable feature boundary | Feature module, explicit service boundaries, typed IDs |
| Domain-rich system | Real invariants, lifecycle transitions, meaningful behavior tied to state | Entities, value objects, aggregates, domain services |
| Distributed workflow | Crosses systems/contexts, needs eventual consistency, async processing | Bounded contexts, integration events, outbox, eventual consistency |

Higher rows are not "better." Landing on a lower row because that's what the
problem needs is a correct outcome, not a missed opportunity.

## Decide, don't default

For each proposed pattern or layer, ask: what real boundary, invariant, or
source of change does this specific piece protect, in this specific feature?
If there's no concrete answer, don't add it — regardless of what other parts
of the codebase do.

Example: an admin settings table with a handful of fields and no lifecycle
does not need an aggregate root, domain events, a repository interface, a
unit of work, and a presenter — there's no invariant to protect and no
boundary being crossed. A contribution workflow with identity, lifecycle
transitions (draft → submitted → reviewed → merged), permissions, invariants,
external operations (e.g. version control), and asynchronous processing
justifies richer modeling — each piece of that structure is paying for
something real: state that can become invalid, a boundary against an
external system, rules that must hold regardless of entry point.

When two features in the same codebase warrant different points on the
scale, that's the assessment working correctly, not an inconsistency to fix.

## Stop, explain, and ask

If the assessment surfaces a concrete design problem — the request conflicts
with an existing invariant, quietly breaks a boundary, needs a lifecycle rule
the request didn't account for, or the right shape genuinely depends on
something only the user knows (expected scale, who else will consume this,
how long it needs to live) — stop before implementing.

State plainly:
- what the concrete problem is
- why it matters for this specific feature (not a general principle restated)
- the options, with the trade-off each one makes

Then ask. Do not silently pick the more complex option "to be safe," and do
not silently build what was literally asked for if it's built on a broken
premise. Guessing in either direction spends the user's trust; asking spends
one turn.

This is different from ordinary implementation judgment calls — small,
reversible choices (naming, file layout, which existing helper to reuse)
don't need this. Reserve stopping for design problems that would be
expensive or awkward to unwind later.

## Check before finishing

- Does the chosen architecture map to a row on the scale that the inspection
  actually supports, not the row that felt more thorough?
- Does every introduced pattern, layer, or abstraction protect a specific,
  nameable boundary or invariant in this feature?
- Does the result match surrounding conventions unless there's a concrete
  reason to diverge?
- If a real design problem was found, did the agent stop and ask instead of
  guessing or building around it silently?
