---
id: event-storming
kind: agent-skill
name: event-storming
description: >
  Facilitate an EventStorming session with the user as the domain expert, to
  explore a business domain, model a process, or find bounded contexts and
  aggregates before a DDD / clean-architecture project. Use when the user wants
  to event storm, brainstorm domain events, map a business flow, discover
  bounded contexts, or prepare domain modeling before designing or coding.
  Not for implementing aggregates in code or reviewing an existing codebase's
  architecture.
---

# EventStorming (facilitator mode)

You facilitate; the user is the domain expert. Draw the knowledge out,
challenge it, and record it. Do not supply the domain yourself: a model built
from plausible guesses looks convincing and is wrong in ways nobody notices
until code exists.

Notation, board template and output templates live in
[board-format.md](./references/board-format.md). Read it before creating the
board file.

## Ground rules

- **Track provenance.** Tag every board item `[stated]` (the user said or
  confirmed it) or `[proposed]` (you suggested it). Proposed items are
  questions until confirmed. Nothing `[proposed]` reaches the final output as
  fact. Rules and constraints that are never written down are the ones
  generated code silently omits.
- **One question at a time.** Ask the cheapest useful question, wait, record,
  then ask the next.
- **Use the user's words.** If two words name one thing, or one word carries
  two meanings, do not reconcile it silently. Record a hot spot. Language
  mismatches are the strongest signal of a bounded context boundary.
- **Events are past-tense business facts** (`InvoicePaid`, `AddressCorrected`),
  not CRUD (`InvoiceUpdated`). When the user says "updated", ask what actually
  happened in business terms; different reasons often mean different events.
- **Disagreement or uncertainty becomes a hot spot**, not a stall. Record it,
  move on, revisit later.
- **Keep a running board file.** Update it after each phase and show only what
  changed. End each phase by asking the user to confirm or correct.
- **Name the missing perspectives.** Big Picture is valuable because several
  departments disagree. A single expert holds one view, so ask which other
  roles would describe this differently and mark those spots as hot spots.

## Pick the level

| Situation | Level |
|---|---|
| Domain unfamiliar, scope wide or fuzzy, need shared understanding or context boundaries | Big Picture |
| One process with a clear start and goal | Process Modeling |
| A bounded context is chosen and will be built | Software Design |

Levels stack: each adds notation to the same event timeline. If the user's
request skips a level the situation needs (for example, asking for aggregates
in a domain nobody has mapped), say so and propose starting one level up.

Prefer `domain-storytelling` instead when many actors or systems cooperate and
the interesting part is who hands what to whom, or when the goal is
documentation of a to-be process. The two combine well: model a critical
cooperative stretch as a domain story, then return here.

## Big Picture workflow

1. **Frame.** Ask: as-is or to-be? What is in scope, what is explicitly out,
   what outcome does the user want from the session?
2. **Brain dump.** The user lists events in any order, unedited. Prompt with
   cheap questions (money moving, contracts or approvals, external parties,
   time-based triggers, things that go wrong), but let the user supply the
   events. Stop when they run dry, then ask "what is still missing?" once.
3. **Normalize.** Rewrite to past tense, merge duplicates, flag synonyms and
   overloaded words as hot spots. Confirm the merges.
4. **Timeline.** Propose an order (`[proposed]`) and let the user correct it.
   Suggest pivotal events (business-critical moments or phase transitions) as
   anchors. Introduce swimlanes only for genuinely parallel flows.
5. **People and systems.** Ask who or what triggers or receives each stretch.
   Skip when obvious or when no system exists yet.
6. **Walkthrough, then reverse.** Retell the chain forward and let the user
   challenge gaps. Then walk it backward from the last event: each event needs
   a plausible predecessor. Missing predecessors expose missing events.
7. **Problems and opportunities.** Hot spot = known problem without a solution.
   Opportunity = an idea the user already has. Give this its own pass.
8. **Prioritize.** Ask the user to pick the two or three most urgent items
   (the arrow vote). Do not pick for them.
9. **Emerging contexts.** Group events by language shifts, pivotal events and
   swimlanes. Propose candidate bounded contexts (`[proposed]`), each with the
   vocabulary that justifies it. The user accepts, renames or rejects.

## Process Modeling workflow

Frame start trigger and end goal. Map the happy path first, then only the
alternatives the user judges important; mark the rest as hot spots. Use the
grammar `Event → Policy → Command → System → Event`, inserting
`Policy → Person → Read model → Command` where a human decides.

- A **policy** is a "whenever X, then Y" reaction. Phrase it that way.
- A **command** is verb + noun, an intent to change state. Vague verbs
  ("check", "verify", "review") usually hide either a missing read model
  (they are looking at information) or a real state change that needs a
  precise name (`MarkOrderAsChecked`).
- A **read model** is the information needed to decide. Ask "what do they need
  to see before they can do this?"

## Software Design workflow

Precondition: a chosen bounded context and a process model for it. If either
is missing, go back a level.

1. Decide per system note: **build** (becomes an aggregate) or **integrate**
   (stays an external black box). Ask the user; do not assume everything is
   built.
2. Apply proportionality before creating aggregates. An aggregate earns its
   place through real invariants, lifecycle transitions or consistency needs.
   A part that is only data in, data out stays a plain feature, not an
   aggregate. If `architecture-proportionality` is available, use its scale to
   sanity-check the result.
3. Consolidate duplicates (many "Order" notes become one). For each aggregate
   list commands accepted and events emitted. Check the lifecycle for gaps: an
   `UpdateShippingAddress` with no creating command means a missing start.
4. Record invariants and policies as explicit rules with the user's wording.
5. Hand off. If `bounded-context-first` is available, use it to relate the
   contexts to hexagonal structure and context-mapping patterns.

## Outputs

Produce one board file plus the summary sections defined in
[board-format.md](references/board-format.md): timeline or process flows,
glossary, rules and policies, out-of-scope decisions, hot spots and
opportunities, candidate contexts or aggregates, open questions. Save it where
the user's project keeps design docs; if unclear, ask once, defaulting to
`docs/domain/`.

## Check before finishing

- Every item is tagged, and no `[proposed]` item is presented as confirmed.
- Every event is past tense and in the user's language; CRUD names were
  challenged.
- The reverse walkthrough was done, not only the forward one.
- Rules, constraints and explicit out-of-scope decisions are written down, not
  only implied by the flow.
- Every candidate context lists the vocabulary or ownership difference that
  justifies it.
- Aggregates exist only where an invariant or lifecycle justifies them.
- Open hot spots are listed with who could resolve them.
