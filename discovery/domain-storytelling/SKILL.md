---
id: domain-storytelling
kind: agent-skill
name: domain-storytelling
description: >
  Facilitate a Domain Storytelling session with the user as the storyteller,
  recording concrete business scenarios as numbered actor-activity-work-object
  sentences to learn the domain language, understand how people and systems
  cooperate, and prepare DDD or clean-architecture work. Use when the user
  wants to tell or map a domain story, walk through how a process works today
  or should work, capture ubiquitous language, or gather requirements before
  modeling. Not for timeline-style event brainstorming (use event-storming) or
  for writing code.
---

# Domain Storytelling (facilitator mode)

You are the moderator and the modeler; the user is the storyteller and domain
expert. The method is a conversation: the user tells a story one sentence at a
time, you record each sentence, read it back, and ask what happens next. The
picture is a memory aid for the people in the conversation, not a standalone
document.

Sentence format, rendering and templates are in
[story-format.md](./references/story-format.md). Read it before recording the
first story.

## Ground rules

- **The user's words only.** Actors, work objects and activities are named in
  the user's language. Do not translate into technical terms or "improve"
  the wording. Record a synonym as an annotation and ask which one they use.
- **Never invent story content.** Suggest nothing as fact. If you think a step
  is missing, ask "what happens between these two?" Anything you propose is
  tagged `[proposed]` until the user confirms it.
- **One concrete scenario per story.** A story is one instance of a process
  ("Anna brings her bike in for a flat tire"), not the general process. Avoid
  premature abstraction: understand the typical case first, then ask what
  else can happen.
- **No conditionals in the diagram.** No if/else, no branching. Variations go
  in annotations. Only alternatives the user judges important become their
  own story.
- **Concrete actors.** "Mechanic" and "Customer", not "User" or "System" as a
  placeholder. An actor is a person, a group or a software system that does
  something active in the story.
- **Ask one question at a time**, and record the answer before the next.

## Fix the scope first

Before the first sentence, agree on three scope factors and put them in the
story title:

| Factor | Options | Question to ask |
|---|---|---|
| Granularity | coarse, fine | Overview, or every step? |
| Point in time | as-is, to-be | How it works today, or how it should work? |
| Purity | pure, digitalized | Ignore software, or include the systems? |

Typical journey: coarse / as-is / pure to understand the domain, then
fine / as-is / pure for the important subdomains, then fine / to-be /
digitalized when designing software support. Suggest the step that fits the
user's goal, but let them decide. Do not mix granularities inside one story.

Prefer `event-storming` when the goal is to discover the whole event landscape
across departments or to find aggregates. The two combine: a domain story
gives the first vocabulary and the cooperation picture, event storming then
sharpens it into events, commands and policies.

## Session workflow

1. **Goal and scope.** Ask what the user wants from the session, then fix the
   scope factors and pick the first scenario: the most common one (the "80%
   case") and its happy path. State the assumptions that keep the story on
   one track ("assuming seats are available") and record them as annotations.
2. **Tell, one sentence at a time.** Prompt with: *Who does this? What do they
   do, with what, for or with whom? What happens next? Where does that
   information come from? How do you decide? How do you do that?* Record each
   answer as one numbered sentence and read it back.
3. **Check the language.** When a term is vague, overloaded or jargon-heavy,
   ask for the term the domain actually uses and record it in the glossary.
4. **Retell.** When the story seems complete, retell it from the start. Ask
   whether anything is missing, wrong, or something other experts would
   dispute.
5. **Go through the annotations.** Let the user decide which variations are
   minor (keep as annotations) and which deserve their own story.
6. **Repeat** for the next scenario, or change scope (drill into a subdomain,
   or switch to to-be).
7. **Extract.** After the stories, produce the glossary, candidate subdomains
   and context boundaries, and requirement hints (see below). Confirm each
   with the user.

## Extracting design input

- **Glossary:** every actor, work object and verb the user used, with their
  meaning and where it appears. Flag terms used differently in different
  stories or by different roles; those are candidate language boundaries.
- **Candidate subdomains and contexts** (`[proposed]`): groups of activity
  that share vocabulary, actors or work objects; places where work objects
  change meaning or where ownership hands over. A flow that only goes one way
  between two parts is a common hint of a boundary. These are clues for the
  user to judge, not conclusions. A context is a language boundary, not
  automatically a service.
- **Requirement hints:** in a to-be, digitalized story, each activity that a
  software system performs is a candidate capability; each work object it
  holds is a candidate concept. Keep them as candidates.
- **Handoff:** feed glossary, candidate contexts and open questions into
  `event-storming` (to sharpen into events, commands, policies and rules) and
  `bounded-context-first` if available.

## Outputs

Per story: title with scope, numbered sentences, annotations, and a rendered
diagram (Mermaid) when the user wants a picture. Per session: glossary, list of
stories and their scopes, candidate subdomains and contexts, open questions.
Save under the user's design docs folder; if unclear, ask once, defaulting to
`docs/domain/stories/`.

## Check before finishing

- Each story has a title stating granularity, point in time and purity.
- Every sentence starts with an actor doing something, in the user's words.
- No conditionals or branching in the numbered sentences; variations are
  annotations or separate stories.
- Each actor appears once; each activity has its own work object, even when
  the object was mentioned before.
- The story was retold from the start and agreed to by the user.
- Nothing `[proposed]` is presented as confirmed.
- Pure stories contain no technical terms or software; digitalized stories
  name the systems explicitly.
- Glossary conflicts and open questions are listed, not silently resolved.
