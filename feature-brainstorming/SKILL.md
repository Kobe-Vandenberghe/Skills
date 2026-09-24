---
name: feature-brainstorming
description: Structured protocol for turning an ambiguous feature idea into an aligned, testable product and engineering direction before implementation.
---

# Feature Brainstorming Protocol

Use this skill when a request describes a feature, workflow, product behavior, or system change but leaves important decisions unstated. The goal is to expose assumptions early, establish a shared definition of success, and produce a concise design brief before code is written.

## Objectives

1. Clarify the user problem, desired outcome, and primary audience.
2. Define the behavior, rules, data, and state transitions precisely.
3. Explore edge cases, failure modes, abuse cases, and lifecycle events.
4. Define the user experience, feedback, and accessibility expectations.
5. Establish scope, dependencies, phasing, and measurable acceptance criteria.

## Five-Pillar Framework

### 1. Intent and Value

- Who is this for?
- What problem or opportunity does it address?
- What should become easier, faster, safer, or possible?
- How will success be recognized or measured?
- Which adjacent workflows or systems does it affect?

### 2. Behavior and Rules

- What are the inputs, outputs, states, and transitions?
- What are the exact calculations, limits, defaults, and precedence rules?
- Which data is static configuration and which data changes at runtime?
- What should happen for empty, invalid, duplicate, or conflicting input?
- Which behavior must be deterministic or idempotent?

### 3. Lifecycle and Failure Modes

- What happens during startup, restart, cancellation, timeout, disconnect, or partial completion?
- What happens when a dependency is unavailable or returns malformed data?
- Can retries duplicate work or produce inconsistent state?
- What permissions, privacy, security, or abuse concerns exist?
- What is the recovery or rollback behavior?

### 4. Experience and Interface

- Where does the user encounter this behavior?
- What feedback communicates progress, success, failure, and next steps?
- What are the keyboard, touch, screen-reader, localization, and reduced-motion needs?
- What should happen on narrow screens, slow devices, or poor connectivity?
- Which parts are required for the first usable version?

### 5. Scope and Delivery

- What is explicitly in scope and out of scope?
- Which existing components, APIs, schemas, or contracts change?
- What is the smallest valuable first phase?
- What can be deferred without creating rework?
- What observable tests or acceptance checks will prove the work is complete?

## Conversation Practices

- Ask grouped questions, not an unstructured questionnaire.
- Resolve decisions that change architecture before polishing details.
- Offer sensible defaults and explain the tradeoff when a choice is open.
- Separate confirmed requirements, assumptions, open questions, and deferred ideas.
- Prefer a small number of high-value questions over exhaustive interrogation.
- When the request is already clear, state assumptions and proceed instead of asking unnecessary questions.

## Required Output Before Implementation

Summarize the alignment in a short design brief containing:

- Problem and target users
- Desired behavior and key rules
- Important states and lifecycle behavior
- Edge cases and failure handling
- User experience expectations
- In-scope and out-of-scope work
- Dependencies and risks
- Acceptance criteria
- Open decisions, if any

Once this brief is stable, use a specification and implementation-planning workflow before making substantial code changes.
