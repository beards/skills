# Skills

A public collection of independently installable Agent Skills, some of which can be composed into workflows.

## Collection

**Skill**:
A single unit of agent capability, defined by one `SKILL.md` and its supporting files, conforming to the Agent Skills standard.
_Avoid_: Plugin, command, prompt

**Prefix**:
The fixed `ds-` that begins every skill's name, keeping it distinct from same-named skills from other sources.
_Avoid_: Namespace, scope

**Leaf skill**:
A skill that works entirely on its own and never depends on another skill by name.

**Capability fallback**:
A step that uses whichever installed skill, or project instruction, covers a capability (such as fetching an issue), and falls back to a generic built-in method when none does.
_Avoid_: Optional dependency, soft dependency

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

## Review

**Goal**:
What a change under review set out to achieve (a feature to add or a bug to fix), taken from wherever it is plainly stated (PR description, ticket, spec), or confirmed by the user when it had to be inferred.
_Avoid_: Intent, purpose

**Fact pack**:
The judgement-free record of a change under review (its Goal, diff, changed files and relevant call sites) that every Lens starts from.
_Avoid_: Context summary, brief

**Lens**:
One angle a change is reviewed from, such as regression or over-design.
_Avoid_: Axis, dimension, angle

**Finding**:
A suspected problem tied to a concrete location and a concrete failure scenario.
_Avoid_: Issue, comment

**Suspicion**:
A concern too vague to be a Finding, handed to verification rather than reported.

**Verdict**:
The outcome of verifying a Finding: confirmed with evidence, refuted, or unverifiable.

**Prior comment**:
A comment by someone other than the change's author, person or bot, that already raises a problem on the change under review.
_Avoid_: Existing review, other feedback

**Pre-existing problem**:
A problem the change did not introduce, expose further, or make easier to trigger.
