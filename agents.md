# Personal Agent Skills Catalog

This repository is my personal catalog of reusable agent skills. Treat each skill directory inside the category folders as a capability that can be loaded, reviewed, improved, or reused across agent workflows.

The current categories are:

- `architecture/` for design, domain modeling, and technical decision skills:
  [architecture-proportionality](architecture/architecture-proportionality/SKILL.md),
  [assess-dotnet-architecture](architecture/assess-dotnet-architecture/SKILL.md),
  [model-dotnet-domain](architecture/model-dotnet-domain/SKILL.md),
  [record-technical-decision](architecture/record-technical-decision/SKILL.md)
- `discovery/` for framing, brainstorming, and specification skills:
  [domain-discovery-facilitator](discovery/domain-discovery-facilitator/SKILL.md),
  [feature-brainstorming](discovery/feature-brainstorming/SKILL.md),
  [spec-driven-development](discovery/spec-driven-development/SKILL.md)
- `engineering-quality/` for implementation, review, logging, and testing skills:
  [code-review-resolution](engineering-quality/code-review-resolution/SKILL.md),
  [implement-dotnet-logging](engineering-quality/implement-dotnet-logging/SKILL.md),
  [write-dotnet-tests](engineering-quality/write-dotnet-tests/SKILL.md)
- `meta/` for catalog-authoring guidance:
  [skill-writing](meta/skill-writing/SKILL.md)

When creating or revising a skill, use the [skill-writing](meta/skill-writing/SKILL.md) skill first. It is the catalog's canonical guidance for deciding whether something belongs as a skill, shaping the instructions, and keeping the skill useful for modern agents. Repository-level indexes and guidance now reference skills with category-prefixed paths such as `meta/skill-writing/SKILL.md`.

Keep repo-level guidance short. Put domain-specific procedures inside the relevant skill directory, and put detailed supporting material in that skill's `references/` folder when it would otherwise bloat the core instructions.