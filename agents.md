# Personal Agent Skills Catalog

This repository is my personal catalog of reusable agent skills. Treat each skill directory inside the category folders as a capability that can be loaded, reviewed, improved, or reused across agent workflows.

The current categories are:

- `architecture/` for design, domain modeling, and technical decision skills
- `discovery/` for framing, brainstorming, and specification skills
- `engineering-quality/` for implementation, review, logging, and testing skills
- `meta/` for catalog-authoring guidance

When creating or revising a skill, use the [skill-writing](meta/skill-writing/SKILL.md) skill first. It is the catalog's canonical guidance for deciding whether something belongs as a skill, shaping the instructions, and keeping the skill useful for modern agents. Catalog links should now use category-prefixed paths such as `meta/skill-writing/SKILL.md`, and uncategorized skill paths such as `skill-writing/SKILL.md` are no longer valid in this repository.

Keep repo-level guidance short. Put domain-specific procedures inside the relevant skill directory, and put detailed supporting material in that skill's `references/` folder when it would otherwise bloat the core instructions.