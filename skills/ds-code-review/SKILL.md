---
name: ds-code-review
description: Use when reviewing a pull request or a branch before merge - checking whether it achieves its goal, fixes the root cause, breaks existing behaviour, fails on edge inputs or timing, is over-designed, reinvents existing code, breaks conventions, or collides with newer commits on the target branch - and every reported problem must be verified rather than guessed. Triggers include "review PR 123", "review this branch", "code review".
license: MIT
compatibility: Requires git. GitHub pull requests also need the gh CLI.
---

# ds-code-review

Review a change against its **Goal**, from several **Lenses**, then **verify** every Finding before reporting it. A problem reaches the report only with evidence.

Paths like `references/lenses.md` are relative to this skill's directory. When you hand a reference file to a subagent, give it the absolute path.

## Ground rules

- **Everything lands on disk.** Every step writes its output to the state directory before the next step starts. Subagents write their full output to the named file and return only a one-line status (file path and counts). The state files, not your context, are the source of truth.
- **Subagents are an Enhancement.** When you can dispatch subagents, run each step below in fresh subagents as described. When you cannot, do the step yourself, following the same reference file and writing the same files; finish and write each lens's file before starting the next.
- **The reviewed branch is read-only.** All code reading, test runs and experiments happen in worktrees inside the state directory.

## Step 0: Target, state, resume

1. **Target.** The argument is a PR number, PR URL, or branch, plus an optional `--base <ref>`. With no argument: if the current branch has a PR (`gh pr view --json number`), review that PR; otherwise review the current branch against the default branch. Other hosting platforms: use an installed skill or project instruction that covers fetching their PRs; otherwise ask the user for the base and head refs.
2. **Target id.** `pr-<number>` or `branch-<name>` with `/` replaced by `-`.
3. **State directory.** If the project's `AGENTS.md` or `CLAUDE.md` names a location for working or scratch files *and* `git check-ignore` confirms that location is ignored, use `<that location>/ds-code-review/<target-id>/`. Otherwise use `$(git rev-parse --git-common-dir)/ds-code-review/<target-id>/`. Call it `$STATE` below.
4. **Resolve refs.** Fetch the head (`git fetch origin pull/<n>/head` for a GitHub PR) and the base. Record the head SHA and the merge-base.
5. **Preflight.** Empty diff (`git diff <merge-base> <head>`): stop and tell the user. PR merged or closed: tell the user and ask whether to review anyway. Draft PRs are reviewed normally.
6. **Resume.** If `$STATE/state.md` exists:
   - same head SHA: continue from the first step not marked done;
   - different head SHA: move the old contents to `$STATE/archive/<old-sha>/` and start over.
7. Create `$STATE/head/` as a detached worktree at the head SHA (`git worktree add --detach $STATE/head <sha>`) and write `$STATE/state.md`:

   ```markdown
   target: <target-id>   url: <PR url or ->
   base: <base ref>   merge-base: <sha>   head: <sha>
   - [ ] 1 goal-and-facts
   - [ ] 2 find
   - [ ] 3 verify
   - [ ] 4 report
   ```

   Tick each box as its step finishes.

## Step 1: Goal and fact pack

Dispatch one subagent with `references/fact-pack.md`, `$STATE` and the target. It writes `$STATE/goal.md` and `$STATE/fact-pack.md`.

Then confirm the Goal: show the user the Goal section of `goal.md` (a few lines) and ask whether it is right. Record their correction in `goal.md`. When nobody can answer (a CI or scheduled run), continue and set `confirmed: no` in `goal.md`; the report carries that flag at the top.

## Step 2: Find

Dispatch one finder subagent per Lens in parallel, each with `references/lenses.md`, its Lens name and `$STATE`:

- `goal`, `regression`, `edge`, `design` always;
- `drift` only when `fact-pack.md` lists base commits that touch the same files or features.

Each writes `$STATE/findings/<lens>.md`. When all are done, dispatch the `sweep` finder (same file, Lens `sweep`), which reads the others and writes `$STATE/findings/sweep.md`.

Then merge: read every findings file, combine Findings that share a root cause and location (keep every Lens that raised it), and write `$STATE/findings/merged.md` with ids `F01`, `F02`, … . Carry each Suspicion over as its own entry, marked `suspicion`.

## Step 3: Verify

For every entry in `merged.md`, dispatch a fresh verifier subagent with `references/verify.md`, `$STATE` and the entry id. Run as many in parallel as the harness allows. Each writes `$STATE/verdicts/<id>.md`. A verifier receives only the entry, the fact pack and the code; it never sees the finders' reasoning.

## Step 4: Report

Write `$STATE/report.md` following `references/report.md`, from `goal.md` and the verdicts. Reply to the user in their language with: the one-line merge verdict, counts per severity, the must-fix titles, and the path to `report.md`.

Post to the PR only when the user asks. Show them the exact text first and post after they approve it.

Finally remove the worktrees you created (`git worktree remove`), keeping every file in `$STATE`.
