# AGENTS.md

## Writing a skill

Vocabulary (Leaf skill, Orchestrator skill, Enhancement, Prefix) is defined in `GLOSSARY.md`.

- Each skill is one flat folder, `skills/<name>/SKILL.md`, with the `ds-` prefix on the name (ADR-0001).
- The core flow uses only Agent Skills standard content, so it runs on every target agent. Harness-specific features are Enhancements: the skill uses them when present and falls back to a single-threaded flow otherwise.
- Leaf skills stand alone. Chaining lives only in Orchestrator skills, which name the skills they call and tell the user which one to install when it is missing.
- Distribution is `npx skills` only; the repo carries no harness plugin manifests.
- The root `README.md` skills table is the index: add a row with each new skill.
- `scripts/validate` must pass before committing; CI runs the same script.

## Agent skills

### Issue tracker

Issues and specs live as local markdown files under `.scratch/<feature-slug>/`. See `docs/agents/issue-tracker.md`.

### Triage labels

The five default triage roles, each label string equal to its name (`needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`). See `docs/agents/triage-labels.md`.

### Domain docs

Single-context: one `GLOSSARY.md` and `docs/adr/` at the repo root. See `docs/agents/domain.md`.
