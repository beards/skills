# Goal and fact pack

You prepare the ground for a code review. You record facts; you make no judgement about whether the change is good. Code is checked out at `$STATE/head/`.

## 1. Find the Goal

Read every source below; each one adds context:

1. **The PR itself**: title, description, linked issues (`gh pr view <n> --json title,body,closingIssuesReferences,commits`). For a branch with no PR: commit messages and the branch name.
2. **Tickets** referenced anywhere above (`#123`, `PROJ-456`, URLs). Fetch each one through, in order:
   - the project's `AGENTS.md` / `CLAUDE.md`, if it says how tickets are fetched;
   - an installed skill whose description says it fetches tickets or issues from a tracker;
   - a generic CLI that matches the reference (`gh issue view`, `glab issue view`);
   - otherwise list the reference as unresolved.
   Follow links one level further when a ticket points at a parent epic, a bug report or a design doc.
3. **Specs** in the repo matching the branch or ticket (`docs/`, `specs/`, `.scratch/`).

Write `$STATE/goal.md`:

```markdown
confirmed: pending
kind: feature | bugfix | refactor | other
## Goal
<2-5 sentences: what the change must achieve. For a bug: the symptom, and the expected behaviour.>
## Acceptance
- <each concrete requirement, quoted from its source>
## Sources
- <url or path> — <what it contributed>
## Unresolved
- <references you could not fetch>
```

When no source states the Goal, infer it from the diff and commits and say so in the Goal section.

## 2. Build the fact pack

Write `$STATE/fact-pack.md` with these sections:

- **Refs**: base, merge-base, head SHAs.
- **Diff**: the command that reproduces it (`git diff <merge-base> <head>`), `--stat` output, and the commit list.
- **Changed symbols**: per file, the functions, types, config keys or endpoints added, changed or removed.
- **Call sites**: for each changed or removed symbol, where it is used across the repo (`file:line`), found by search.
- **Conventions**: paths of files that document how code is written here (`AGENTS.md`, `CLAUDE.md`, `CONTRIBUTING.md`, `CODING_STANDARDS.md`, linters' configs).
- **Tests**: how the project runs its tests, and which tests touch the changed files.
- **Base drift**: commits on the base since the merge-base (`git log <merge-base>..<base>`) that touch the same files or symbols, with a one-line summary each. Write `none` when there are none.

Facts only. Opinions about the change do not belong in either file.

Return one line: the two file paths.
