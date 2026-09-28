---
id: skill-writing
name: skill-writing
description: Design and write high-quality reusable agent skills that teach agents procedures, judgment, and domain-specific ways of working.
kind: agent-skill
---

# Skill Writing

## Purpose

Use this skill when creating, reviewing, or improving an agent skill.

A skill should teach an agent how to perform a repeatable task or apply reusable expertise. It should contain enough guidance to improve the agent's behavior beyond what can be achieved through a tool description, a repo instruction, or a small prompt fragment.

For the required file structure and formatting conventions, see [Skill Format](./references/skill-format.md).

## Modern Agent Skill Practices

Current agent runtimes increasingly treat skills as progressively loaded, filesystem-based capabilities: lightweight metadata is available for discovery, core instructions load when relevant, and supporting files or scripts are accessed only when needed. Write skills with that lifecycle in mind.

Apply these practices when designing or revising a skill:

* Make the front matter discoverable. The description should say what the skill does and when to use it, because agents may rely on that text to decide whether to load the skill.
* Keep `SKILL.md` focused on execution. Put bulky schemas, examples, templates, source notes, and deterministic scripts in references or supporting files, then link them from the point where they matter.
* Prefer progressive disclosure. Give the agent enough in the core instructions to act, but avoid loading every detail into context before the task requires it.
* Separate instructions from tools. The skill should teach judgment and workflow; tool definitions, APIs, and runtime mechanics should remain in tool or platform documentation unless a specific capability is essential.
* Include guardrails where misuse is plausible. Name permissions, destructive actions, privacy constraints, safety boundaries, approval points, and when the agent should stop.
* Account for state and cleanup when the skill involves long-running tasks, generated files, sessions, sandboxes, caches, or external systems.
* Design for composition. A skill should be independently understandable while cooperating cleanly with other skills through explicit references and stable boundaries.
* Add an evaluation path for non-trivial skills. Define examples, review checks, fixtures, validation commands, or expected outputs that can reveal whether the skill actually improves agent behavior.

## Decide Whether Something Should Be a Skill

Before writing a skill, determine whether the proposed behavior actually belongs in one.

A skill is appropriate when the agent needs reusable guidance about:

* how to approach a task
* which steps to follow
* what judgment to apply
* which constraints matter
* how to handle ambiguity or exceptions
* what a good result looks like
* domain-specific ways of working

A skill should represent a meaningful capability or procedure, not merely a piece of information.

### Skills vs Tools

A tool gives the agent a controlled capability to interact with data or systems.

Examples:

* search knowledge
* read a document
* create a task
* query a business system
* send a message

A skill teaches the agent how to use available capabilities effectively to achieve a goal.

Examples:

* research a company question
* process meeting notes
* draft a contribution
* assess a policy
* plan a knowledge update

Do not create a skill that merely restates how to call a tool.

### Skills vs Knowledge

Knowledge describes facts, rules, concepts, policies, models, or organizational information.

A skill describes how to act.

If the content is primarily something the agent should know, it usually belongs in ordinary knowledge or a reference document.

If the content is primarily something the agent should do, it may belong in a skill.

### Skills vs Small Prompt Rules

Do not create a separate skill for every small behavior.

Rules such as:

* use concise titles
* do not invent missing facts
* preserve important details

may belong inside a broader skill when they support that capability.

Create a separate skill only when the behavior is independently meaningful and reusable.

## Choose the Right Scope

A good skill should be broad enough to be reusable but specific enough to provide meaningful guidance.

Too broad:

```text
do-good-work
```

This does not give the agent a clear capability.

Too narrow:

```text
write-three-word-contribution-title
```

This is better handled as part of a larger drafting skill.

Better:

```text
contribution-drafting
knowledge-research
meeting-processing
skill-writing
```

A useful test is:

> Would this capability reasonably be taught as a repeatable way of working to another capable person?

If yes, it is likely a reasonable skill boundary.

## Define the Outcome First

Before writing detailed instructions, identify what successful execution of the skill should accomplish.

Ask:

* What is the agent trying to produce or achieve?
* What makes the result good?
* What mistakes should the agent avoid?
* Where does judgment matter?
* What information or capabilities might the agent need?
* How could a reviewer or evaluation tell whether the skill worked?

The skill should be organized around this outcome rather than around the implementation details of the current system.

## Write for Agent Execution

Skills are instructions for an agent, not general documentation for a human reader.

Write instructions that are:

* explicit
* actionable
* ordered where sequencing matters
* clear about judgment
* clear about constraints
* specific about important edge cases

Prefer:

```text
Before proposing a new knowledge document, search for an existing canonical document covering the same subject.
```

over:

```text
Knowledge duplication should generally be avoided.
```

The first version tells the agent what to do.

## Teach Judgment, Not Just Steps

The most valuable part of a skill is often the reasoning that cannot be captured by a simple tool call.

Explain how the agent should make decisions.

For example:

* when to search more broadly
* when enough evidence has been found
* when to ask for clarification
* when to preserve uncertainty
* how to choose between conflicting sources
* when to update existing information instead of creating something new

Do not turn every skill into a rigid checklist if the task requires judgment.

Use procedures as guidance while explaining where adaptation is expected.

## Keep the Core Skill Focused

`SKILL.md` should contain the instructions needed to perform the capability.

Avoid filling it with large amounts of supporting information.

If detailed background material, schemas, models, templates, executable helpers, fixtures, or examples are required, move them into references or supporting files and link them from the skill.

The core skill should still explain when those references are relevant.

Prefer:

```text
When constructing a contribution object, follow the canonical contribution model in the referenced document.
```

with a linked reference.

Do not assume the agent will automatically read every reference.

When a supporting file is optional, say when to read or run it. When it is required, make that dependency explicit before the agent needs it.

## Avoid Tool Coupling

Describe required capabilities rather than hard-coding tool implementations unless a specific implementation is essential.

Prefer:

```text
Search the available company knowledge for related policies.
```

over:

```text
Call `search_documents` with the query.
```

This allows the skill to remain valid when tools evolve.

Tool schemas and invocation rules belong to the tool definitions.

If the skill requires a deterministic operation, prefer referencing a bundled script, validator, fixture, or command contract over asking the agent to recreate the operation from prose every time.

## Avoid Duplicating Existing Knowledge

Before embedding substantial domain information into a skill, determine whether that information already belongs elsewhere in the knowledge base.

Prefer referencing canonical knowledge over copying it into the skill.

This prevents:

* stale duplicated information
* conflicting definitions
* difficult maintenance
* oversized skills

A skill may summarize a principle when it is essential to execution, but detailed authoritative information should normally remain in its canonical source.

## Make Dependencies Explicit

If successful use of a skill requires another document, model, policy, or skill, reference it explicitly.

Do not rely on hidden assumptions about what the agent has already loaded.

Use dependencies sparingly.

A skill should not require a long chain of other skills simply to perform its basic function.

When possible, keep skills composable and independently understandable.

Avoid hidden platform assumptions. If a skill depends on filesystem access, command execution, network access, a sandbox, an approval flow, or a specific runtime capability, say so in the skill.

## Handle Uncertainty

Include guidance for uncertainty when the task can encounter incomplete or ambiguous information.

Depending on the capability, instruct the agent when to:

* continue investigating
* state an assumption
* preserve uncertainty in the output
* ask a focused clarification
* stop rather than fabricate information

Avoid generic instructions that require clarification for every missing detail.

The skill should explain which missing information actually matters.

For agentic workflows, also identify which uncertainties are safe to resolve through investigation and which require human approval because they affect permissions, irreversible actions, public output, cost, privacy, or security.

## Define Important Boundaries

State what the skill does not do when the distinction prevents common mistakes.

For example, a drafting skill may propose content but not persist or publish it.

A research skill may investigate company knowledge but not modify it.

Boundaries should protect the intended responsibility of the skill without becoming an exhaustive list of prohibited actions.

Include explicit boundaries for destructive operations, credential handling, personal data, external side effects, and publication when those risks are plausible.

## Include Examples Where They Teach Behavior

Examples are useful when they demonstrate:

* a difficult judgment
* expected structure
* common mistakes
* transformation from poor input to good output

Do not add examples purely to make the skill longer.

Prefer a few representative examples over many repetitive ones.

For complex or high-risk skills, include at least one negative example or review scenario that shows what the agent should reject, question, or escalate.

## Plan Validation

Before finalizing a non-trivial skill, decide how it will be checked.

Useful validation approaches include:

* a small example task with expected behavior
* a review checklist tied to the skill's stated outcome
* fixtures or sample inputs in `references/`
* a validator or script for structured outputs
* regression notes for common failure modes

Keep validation proportional. A narrow drafting skill may only need examples and a checklist; a skill that drives external systems may need explicit safety checks, approval points, and cleanup verification.

## Review the Skill Before Finalizing

Check the completed skill against the following questions:

* Is the capability clearly defined?
* Is this genuinely a skill rather than a tool or knowledge document?
* Is the scope neither too broad nor too narrow?
* Does the skill teach actionable behavior?
* Does it explain important judgment?
* Are required constraints explicit?
* Are uncertainty and edge cases handled where relevant?
* Is implementation-specific tool coupling avoided?
* Is duplicated knowledge kept to a minimum?
* Are references used appropriately?
* Is the description strong enough for automatic skill discovery?
* Does the skill use progressive disclosure instead of front-loading bulky context?
* Are permissions, safety boundaries, state, and cleanup covered where relevant?
* Is there a practical way to evaluate or review whether the skill works?
* Could an agent follow the skill without undocumented assumptions?

If the skill mainly describes a concept rather than a procedure, reconsider whether it should be ordinary knowledge.

If the skill mainly describes API calls or system operations, reconsider whether it belongs in tool documentation.

## Quality Standard

A good skill should make a capable general-purpose agent noticeably better at a specific repeatable task.

It should provide enough structure to create consistent behavior without removing the agent's ability to adapt to context.

The objective is not to encode every possible action.

The objective is to capture the reusable expertise, procedure, and judgment that make the task reliably executable.