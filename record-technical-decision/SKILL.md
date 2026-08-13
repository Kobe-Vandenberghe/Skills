---
name: record-technical-decision
description: Record a consequential technical choice as a lightweight Architecture Decision Record in `docs/decisions/`. Use after multiple realistic options were considered; when making a non-obvious architecture or technology choice; when a decision affects storage, system boundaries, integrations, security, deployment, reliability, or operations; or when reversing it later would be expensive. Capture context, drivers, options, the chosen option and why it won, consequences, assumptions, revisit triggers, and related code or documentation. Do not use for routine implementation details, obvious existing conventions, easily reversible local choices, or decisions that have not actually been made.
---

# Record Technical Decision

Preserve why a consequential choice was made while the evidence and tradeoffs are still fresh. Write a short decision record, not a design essay or implementation plan.

Use [assets/adr-template.md](assets/adr-template.md) as the default structure. Follow an existing naming or heading convention within `docs/decisions/` when one is already established.

## Gate the decision

Create an ADR when at least one of these is true:

- two or more realistic options were seriously considered
- the winning choice is non-obvious without its context
- the choice affects data ownership, storage, boundaries, integrations, security, deployment, reliability, or operations
- reversal would require migration, coordinated rollout, major refactoring, downtime, or compatibility work

Do not create an ADR for naming, formatting, routine library usage, small refactors, standard CRUD, an obvious repository convention, or a cheap local choice. If explicitly invoked for such a decision, state briefly that no ADR is warranted and why.

Record only a decision that has actually been made. If the choice remains open, finish the comparison first or label the record `Proposed` only when the user wants a proposal captured.

## Gather evidence

Before writing:

1. Read repository instructions and existing files in `docs/decisions/`.
2. Trace the related code, issue, design document, or conversation far enough to establish the real context.
3. Identify the decision owner or deciders only when known.
4. Separate facts, decision drivers, and assumptions.
5. Recover the options that were genuinely considered. Do not invent weak alternatives to make the decision look rigorous.

Never reconstruct missing rationale as fact. Mark uncertainty as an assumption or ask one focused question when it would change the recorded reason.

## Choose the file

Store the record under `docs/decisions/`.

- Reuse the repository's existing numbering and filename convention when present.
- Otherwise use `YYYY-MM-DD-short-kebab-title.md`.
- Choose a durable title describing the decision, such as `store-contributions-in-postgres`, not a ticket title such as `implement-feature-42`.
- Avoid collisions and do not rename older ADRs merely to normalize them.

If a new decision replaces an older ADR, create a new record and link the superseded decision. Preserve the old reasoning; update its status or back-link only when the repository convention expects it.

## Write the ADR

Keep the record easy to scan, usually 300 to 700 words and rarely above 800. Include:

### Context and problem

Describe the forces that made a decision necessary. Include only enough system background to understand the choice. Avoid retelling the whole project.

### Decision drivers

List the criteria that actually separated the options, ordered by importance. Use concrete drivers such as transactional consistency, expected scale, team familiarity, auditability, portability, latency, operating cost, security boundaries, or migration risk.

### Options considered

Summarize each realistic option fairly in one or two bullets. State its strongest benefit and the cost that mattered. Do not create strawman options.

### Decision and why it won

State the chosen option in one direct sentence. Explain why it best satisfies the drivers and why its disadvantages are acceptable now. Do not say merely that it is "simpler" or "more scalable" without the relevant comparison.

### Consequences and tradeoffs

Record both gains and costs, including new coupling, operational burden, migration work, failure modes, or constraints. A useful ADR makes the downside visible.

### Assumptions and revisit triggers

List assumptions the decision depends on. Convert vague future concerns into observable triggers, such as a volume threshold, a new tenant-isolation requirement, repeated operational incidents, a vendor capability change, or the need for independent deployment.

### References

Link related files, issues, pull requests, diagrams, benchmarks, and documentation. Prefer repository-relative links for files. Link to evidence rather than duplicating it.

## Status and lifecycle

Use a small status vocabulary:

- `Proposed`: documented but not yet agreed
- `Accepted`: agreed and current
- `Superseded`: replaced by a newer ADR
- `Deprecated`: intentionally no longer followed without a direct replacement

An ADR records the reasoning available at the time. Do not rewrite its rationale after implementation outcomes are known. Add a new ADR when the decision changes.

## Writing behavior

- Use neutral, factual language and short sections.
- Preserve disagreement or uncertainty when it affected the choice.
- Keep implementation steps in issues or plans, not the ADR.
- Include code snippets only when a tiny contract or schema fragment is essential to the decision.
- Avoid generic benefits that apply to every option.
- Make revisit triggers actionable rather than writing "revisit if requirements change."

## Output behavior

When the task authorizes repository changes, create the ADR before implementing the affected decision and report its path. When the user requested only analysis or review, return the proposed ADR content and target path without modifying the repository.

After writing, verify that:

- the chosen option is unmistakable
- its reason maps to the stated drivers
- rejected options are represented fairly
- negative consequences are present
- assumptions are not disguised as facts
- revisit triggers are concrete
- references resolve or are clearly marked as pending
- the document remains lightweight
