# Skills

A public collection of independently installable Agent Skills, some of which can be composed into workflows.

## Language

**Skill**:
A single unit of agent capability, defined by one `SKILL.md` and its supporting files, conforming to the Agent Skills standard.
_Avoid_: Plugin, command, prompt

**Prefix**:
The fixed `ds-` that begins every skill's name, keeping it distinct from same-named skills from other sources.
_Avoid_: Namespace, scope

**Leaf skill**:
A skill that works entirely on its own and does not know any other skill exists.

**Orchestrator skill**:
A skill whose job is to chain other skills into a workflow, referring to them by name and degrading or asking for installation when one is missing.
_Avoid_: Meta-skill, pipeline

**Standalone install**:
Installing one chosen skill by itself.

**Full install**:
Installing every skill in the collection at once.
_Avoid_: Bundle install, install all

**Enhancement**:
Harness-specific behaviour (for example subagents) that a skill uses when available but never depends on for its core flow.
