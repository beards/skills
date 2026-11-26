# Verifying Findings

You receive a group of entries, `$STATE/entries/<id>.md`, often in different files. Judge each entry on its own: your job is to **try to refute it**. Start from `$STATE/goal.md` and `$STATE/fact-pack.md`, and read the actual code at head yourself.

## Gather evidence

Pick the strongest evidence the Finding allows:

1. **Reproduce.** Write a failing test or a small script that shows the scenario. Work in your own worktree: `git worktree add --detach $STATE/worktrees/<id> <head sha>`. A verifier handling several entries makes one worktree named after the first entry's id. To compare with the old behaviour, add `$STATE/worktrees/<id>-base` at the merge-base. Keep scripts inside these worktrees and remove them with `git worktree remove --force` when done.
2. **Run** the existing tests, a local build, or the program itself. A missing runner can be installed inside your worktree (a virtualenv, `npx`, `uvx`); when that is not possible, say so under Method.
3. **Trace** the code path line by line, quoting each `file:line` that carries the scenario.
4. **Point at the thing itself**: the existing utility for a reinvented wheel, the documented rule for a convention breach, the base commit for drift.

To reach a real environment (a running service, a database, a device), use an installed skill or project instruction that covers it. Read-only access and anything local or isolated is fine. Ask the user before anything that changes a shared system: deploying, writing to a shared database, calling an external write API, commenting anywhere. The reviewed branch stays untouched.

## Decide

- **CONFIRMED**: your evidence shows the scenario happens.
- **REFUTED**: the code, a test or a guard shows it cannot happen, or it does not matter.
- **UNVERIFIABLE**: the evidence needed is out of reach. Say exactly what would settle it.

Record the **evidence level** your decisive evidence reached, the highest that applies:

- `reproduced`: shown in the running program or a real environment.
- `ran`: a test or script you ran against the real code fails, or passes, exactly as the verdict says.
- `traced`: you walked the code path line by line and quoted each step.
- `pointed`: you quoted the decisive line, rule, utility or commit.

A verdict with none of these is UNVERIFIABLE. When a higher level was within reach and you stopped short, say why under Method.

Then classify a CONFIRMED problem: **pre-existing** if the change neither introduced it, exposed it further, nor made it easier to trigger; otherwise it belongs to this change. Falling short of the Goal always belongs to this change. Re-assess severity (must-fix, should-fix, nit) from what you found.

## Write

`$STATE/verdicts/<id>.md`:

```markdown
verdict: CONFIRMED | REFUTED | UNVERIFIABLE
severity: must-fix | should-fix | nit      (CONFIRMED only)
pre-existing: yes | no                     (CONFIRMED only)
evidence: reproduced | ran | traced | pointed   (CONFIRMED or REFUTED)
## Finding
<title, location, scenario as you now understand it>
## Method
<what you did, with the commands>
## Evidence
<command output, test results or quoted lines that decide the verdict>
## Fix
<the smallest change that resolves it>
```

Return one line: each entry id with its verdict.
