# Skill Format

This document defines the standard structure and format for agent skills.

Skills are stored as Markdown files inside the knowledge base and are discovered and read by agents through the skill capabilities provided by the system.

## Directory Structure

Each skill lives in its own directory. In a dedicated skills catalog, those directories may be top-level entries; in a larger knowledge base, they may live under `skills/`.

```text
skills/
└── <skill-name>/
    ├── SKILL.md
    └── references/
        └── ...
```

`SKILL.md` is the entry point for the skill.

A skill may optionally contain a `references/` directory for supporting material.

## Skill Name

Skill directory names should:

* use lowercase kebab-case
* describe the capability or procedure
* be concise but specific
* avoid implementation-specific names where possible

Examples:

```text
contribution-drafting
skill-writing
knowledge-research
meeting-processing
```

Avoid names such as:

```text
helper
general-skill
use-search-tool
document-stuff
```

## SKILL.md

Every skill must contain a file named exactly:

```text
SKILL.md
```

This file contains:

1. front matter describing the skill
2. the instructions the agent follows when using the skill
3. references to supporting material when required

## Front Matter

Every `SKILL.md` begins with YAML front matter.

Minimum structure:

```yaml
---
id: skill-writing
kind: agent-skill
name: skill-name
description: Short description of what the skill enables the agent to do.
---
```

### `kind`

For normal agent skills:

```yaml
kind: agent-skill
```

This identifies the document as an executable agent skill rather than ordinary knowledge.

### `name`

The skill name should match the skill directory name.

Example:

```yaml
name: contribution-drafting
```

Use lowercase kebab-case.

### `description`

The description explains when and why the skill is useful.

It should be concise enough to support skill discovery while specific enough for an agent to distinguish the skill from similar skills.

Example:

```yaml
description: Turn rough information into a structured contribution draft while preserving important details and identifying uncertainty.
```

Avoid vague descriptions such as:

```yaml
description: Helps with contributions.
```

## Recommended SKILL.md Structure

Skills may vary, but a typical skill should use the following structure:

```markdown
---
kind: agent-skill
name: example-skill
description: Description of the skill.
---

# Example Skill

## Purpose

Explain what this skill enables the agent to do and when it should be used.

## Principles

Describe important rules, constraints, and judgment the agent should apply.

## Process

Describe the procedure the agent should follow.

### 1. First step

Instructions.

### 2. Second step

Instructions.

### 3. Third step

Instructions.

## Handling Uncertainty

Explain what the agent should do when information is incomplete, conflicting, or ambiguous.

## Output

Describe what a successful result should contain when an output format matters.

## References

List supporting documents that should be consulted when relevant.
```

Not every skill needs every section.

The structure should follow the needs of the skill rather than forcing unnecessary headings.

## References

Supporting information that does not belong directly in the core skill instructions should be placed in `references/`.

Example:

```text
skills/
└── contribution-drafting/
    ├── SKILL.md
    └── references/
        └── contribution-model.md
```

The skill can reference the file using a relative path:

```markdown
See [Contribution Model](./references/contribution-model.md).
```

Relative references beginning with `./` resolve from the skill directory.

## Knowledge Base References

A skill may also reference documents elsewhere in the knowledge base.

Use a path beginning at the knowledge-base root:

```markdown
See [AI Usage Policy](/policies/ai-usage.md).
```

The distinction is:

```text
./references/example.md
```

refers to a file contained within the skill.

```text
/policies/example.md
```

refers to knowledge elsewhere in the knowledge base.

## What Belongs in SKILL.md

`SKILL.md` should contain the instructions required for an agent to perform the skill.

Typical contents include:

* purpose
* decision rules
* workflow
* constraints
* quality requirements
* handling of uncertainty
* expected behavior
* important exceptions

Keep essential instructions in `SKILL.md`.

An agent should be able to understand the core behavior of the skill without first reading every reference.

## What Belongs in References

References should contain supporting material that is useful but would make the main skill unnecessarily large or specific.

Examples include:

* data models
* schemas
* detailed examples
* templates
* domain terminology
* product-specific documentation
* longer policies or standards

References should not be used merely to split a short skill into multiple files.

## Progressive Loading

Design the directory so agents can load only what they need:

* front matter is for discovery and should stay short
* `SKILL.md` is for the core procedure and judgment
* `references/` is for supporting material loaded on demand
* scripts, fixtures, templates, and schemas belong in supporting files when they make execution more reliable or keep the main instructions lean

Do not assume every reference file will be read automatically. Link supporting files from the relevant instruction and explain when to use them.

## Tool Independence

Skills describe what the agent should accomplish and how it should reason about the task.

They should generally avoid depending on exact runtime tool names.

Prefer:

```markdown
Search the available company knowledge for relevant policies.
```

instead of:

```markdown
Call `search_documents`.
```

Exact tool behavior belongs to the tool definition and runtime.

A skill may describe the capability it requires, but should avoid coupling its procedure to an implementation unless that implementation is essential to the skill.

## Self-Contained Core Instructions

The core procedure should remain understandable on its own.

Do not assume that:

* another skill has already been read
* the agent knows undocumented company conventions
* references will always be loaded automatically
* a particular tool implementation will always exist

When another skill or document is required, reference it explicitly.

## Markdown

Skills use normal Markdown.

Prefer:

* clear headings
* short paragraphs
* ordered steps for procedures
* bullet lists for rules
* examples where they materially improve understanding
* code blocks for paths, schemas, or structured examples

Avoid excessive formatting that does not help the agent perform the task.

## Complete Example

```text
skills/
└── knowledge-research/
    ├── SKILL.md
    └── references/
        └── source-quality.md
```

```markdown
---
kind: agent-skill
name: knowledge-research
description: Investigate questions using company knowledge and produce grounded answers from relevant sources.
---

# Knowledge Research

## Purpose

Use this skill when a question requires investigation of company knowledge.

## Principles

- Search before assuming.
- Prefer authoritative sources.
- Distinguish evidence from inference.
- Identify meaningful conflicts.

## Process

### 1. Understand the question

Identify what needs to be established.

### 2. Find relevant knowledge

Search for documents related to the question.

### 3. Read primary sources

Read enough context to understand the relevant information.

### 4. Verify

Check related sources when they could materially change the answer.

### 5. Answer

Synthesize the findings into a grounded response.

## References

For additional guidance on evaluating sources, see:

[Source Quality](./references/source-quality.md)
```

## Summary

A valid skill therefore has the basic shape:

```text
skills/<skill-name>/SKILL.md
```

with:

```yaml
---
kind: agent-skill
name: <skill-name>
description: <discovery description>
---
```

and optionally:

```text
skills/<skill-name>/references/
```

The format provides consistency while leaving the actual skill instructions flexible enough to match the capability being taught.