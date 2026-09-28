# EventStorming board format

Text stand-in for the sticky-note wall. Use it so a session can be saved,
resumed, diffed and fed to other agents.

## Notation

| Element | Sticky | Wording rule | Levels |
|---|---|---|---|
| Domain event | orange | Past tense business fact: `OrderPlaced` | all |
| Person | small yellow | A role, not a name: `Cashier` | Big Picture+ |
| System | pink | External or not-yet-decided black box | Big Picture+ |
| Hot spot | magenta / red | A question, conflict or problem, with a short reason | all |
| Opportunity | green | An idea the user already has | Big Picture |
| Policy | lilac | `Whenever <event>, then <command>` | Process+ |
| Command | blue | Verb + noun, intent: `PlaceOrder` | Process+ |
| Read model | green | The information needed to decide | Process+ |
| Aggregate | large yellow | A noun that owns commands and events | Design |
| Bounded context | white / outline | Named after the language it protects | Design |

Every item in the board file carries a provenance tag: `[stated]` or
`[proposed]`.

## Board file template

```markdown
# Event storm: <scope>
Level: Big Picture | Process | Design    Time: as-is | to-be
Scope in: ...    Scope out: ...
Perspectives present: ...    Perspectives missing: ...

## Timeline
<!-- one line per event, in order; pivotal events marked with * -->
1. [stated] *OrderPlaced          (Customer, WebShop)
2. [stated] PaymentReceived       (PaymentProvider)
3. [proposed] StockReserved       ? confirm: does this happen before payment?

## Process flows (Process / Design level)
<!-- one line per step, grammar: Event -> Policy -> Command -> System|Aggregate -> Event -->
[stated] OrderPlaced
  -> policy: whenever OrderPlaced, then RequestPayment
  -> command: RequestPayment  -> system: PaymentProvider
  -> event: PaymentRequested
[stated] read model: OrderSummary, Customer -> command: ConfirmOrder -> aggregate: Order -> event: OrderConfirmed

## Hot spots
- H1 [stated] "Customer" means account holder in Sales, payer in Billing
- H2 [proposed] Who cancels after shipment? (unclear owner)

## Opportunities
- O1 [stated] ...

## Priorities (the user's picks)
1. H1
2. ...

## Glossary
| Term | Meaning (user's words) | Used by | Conflicts |
|---|---|---|---|

## Rules and policies
- R1 [stated] A ... cannot ...   (source: user, phase where it appeared)

## Out of scope
- [stated] <event or feature deliberately not modeled or not built>

## Candidate bounded contexts (Big Picture output)
| Context | Owns (events / concepts) | Vocabulary or ownership signal | Status |
|---|---|---|---|
| Ordering | OrderPlaced, OrderConfirmed | "Order" = customer intent | [proposed] |

## Aggregates (Design output)
| Aggregate | Context | Commands accepted | Events emitted | Invariants |
|---|---|---|---|---|

## Open questions
- Q1 ...  (who could answer: ...)
```

## Prompts that surface events without supplying them

Use these to unblock the user; they are prompts, not content.

- Where does money change hands, or a commitment get made?
- What needs an approval, signature or confirmation?
- What happens because time passed (deadlines, expiry, monthly runs)?
- What happens when something goes wrong or is refused?
- Which other team or external party gets involved, and what do they call it?
- What do you do by hand or in a spreadsheet today?

## Challenge questions for the walkthrough

- What triggered this? Who or what did it?
- What must already be true for this to happen?
- Could this fail or be refused? What happens then?
- Do you and the other team use the same word for this?
- What do you need to see before you can do this? (a read model)
- Is this a real business event, or a technical side effect?

## Heuristics for candidate bounded contexts

Treat these as clues, not rules; every candidate stays `[proposed]` until the
user accepts it.

- The same word means different things on either side of a point in the flow.
- A pivotal event marks a transition where ownership or vocabulary changes.
- A swimlane follows an independent process, possibly with its own timeline.
  Not every swimlane is a context; sometimes it is one rule.
- Different people or roles appear in different stretches of the wall.

A bounded context is a language and ownership boundary. It does not imply a
separate service or deployment.
