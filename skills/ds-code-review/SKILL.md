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
- **Model tiers are an Enhancement.** When the harness lets you pick a subagent's model, use the tier the step names: *strongest*, *standard* or *light*. Effort is never lowered: run every subagent at high effort or above, because lower effort skips required work such as fetching tickets and searching call sites. A harness without a model choice uses its default for every step.
- **Steps can run elsewhere.** When the user's or project's agent instructions name a command for running a step (`goal-and-facts`, `find`, `verify`, `report`), dispatch that step's subagents through it, giving each the same reference file (absolute path), `$STATE` and inputs, and asking for the same output files and one-line status. Check the output files exist before ticking the step.
- **Record usage.** When the harness reports a subagent's token use, append `<step> | <lens or entry ids> | <tier> | <tokens>` to `$STATE/usage.md`. For a step run through an instructed command, write the command's name as the tier and `-` when it reports no tokens.

## Step 0: Target, state, resume

1. **Target.** The argument is a PR number, PR URL, or branch, plus an optional `--base <ref>` and an optional `--full` (review from scratch even after a finished review). With no argument: if the current branch has a PR (`gh pr view --json number`), review that PR; otherwise review the current branch against the default branch. Other hosting platforms: use an installed skill or project instruction that covers fetching their PRs; otherwise ask the user for the base and head refs. Several PRs (URLs or `owner/repo#n`) form a **Review set**: follow [references/multi-pr.md](references/multi-pr.md) as well, and hand it to every subagent along with its own reference file.
2. **Target id.** `pr-<number>` or `branch-<name>` with `/` replaced by `-`.
3. **State directory**, decided by the directory you were invoked in. Inside a git repo: if its `AGENTS.md` or `CLAUDE.md` names a location for working or scratch files *and* `git check-ignore` confirms that location is ignored, use `<that location>/ds-code-review/<target-id>/`; otherwise use `$(git rev-parse --path-format=absolute --git-common-dir)/ds-code-review/<target-id>/`. Outside any repo: `<current directory>/.ds-code-review/<target-id>/`. Call it `$STATE` below; always pass it to subagents as an absolute path.
4. **Resolve refs.** When the repo has a remote, fetch the head (`git fetch origin pull/<n>/head` for a GitHub PR) and the base; a local branch with no remote uses the local refs. Record the head SHA and the merge-base.
5. **Preflight.** Empty diff (`git diff <merge-base> <head>`): stop and tell the user. PR merged or closed: tell the user and ask whether to review anyway. Draft PRs are reviewed normally.
   **Related PRs.** For a single PR, look in its description and commit messages for links to other PRs marked as related in either direction ("depends on", "requires", "part of", "consumers"). Do this now, before any ticket is fetched. Add merged or closed ones as Background PRs without asking. For open ones, ask the user whether to review them together; each yes joins the Review set. When nobody can answer, review the one PR and list the open related PRs in the report header. Any second member makes this a Review set.
6. **Resume.** If `$STATE/state.md` exists:
   - `--full` given: move the old contents to `$STATE/archive/<old-sha>/` and start over;
   - a step not yet ticked: continue from it, following **Heads can move** below when the head moved;
   - every step ticked, head moved: this is a **Follow-up review**. Follow [references/follow-up.md](references/follow-up.md) in place of steps 0.7 to 4, and hand it to every subagent along with its own reference file;
   - every step ticked, same head: tell the user the report is current and give its path.
7. Create `$STATE/head/` as a detached worktree at the head SHA (`git worktree add --detach $STATE/head <sha>`) and write `$STATE/state.md`:

   ```markdown
   target: <target-id>   url: <PR url or ->
   base: <base ref>   merge-base: <sha>   head: <sha>
   - [ ] 1 goal-and-facts
   - [ ] 2 find
   - [ ] 3 verify
   - [ ] 4 report
   ```

   Tick a box only after that step's output files exist, and append ` @<short head sha>` to it.

**Heads can move.** Before each later step, check whether the head moved (`gh pr view <n> --json headRefOid`, or the branch ref). When it did: keep the finished work, append an `## Update` section to `fact-pack.md` with the new commits and their diff, move `$STATE/head` to the new head (`git worktree remove --force`, then add it again), and run every remaining step at the new head. Verifiers therefore judge each entry at the latest head.

## Step 1: Goal and fact pack

Dispatch two *standard*-tier subagents with `references/fact-pack.md`, `$STATE` and the target: one for its section 1, which writes `$STATE/goal.md`, and one for its section 2, which writes `$STATE/fact-pack.md`. Run them in parallel when you can.

As soon as `goal.md` exists, set its `confirmed:` line by its `source:`:

- `source: stated` (the PR description, a ticket, a spec or a design doc states what the change must achieve): set `confirmed: not-needed` and carry on without asking.
- `source: inferred` (the Goal had to be pieced together from the title, commits or diff): show the user the Goal section (a few lines) and ask whether it is right. Record their correction and set `confirmed: yes`. When nobody can answer (a CI or scheduled run), set `confirmed: no` and carry on; the report carries that flag at the top.

## Step 2: Find

Dispatch one finder subagent per Lens in parallel, each with `references/lenses.md`, its Lens name and `$STATE`:

- `goal`, `regression`, `edge` (*strongest*) and `design` (*standard*) always;
- `drift` (*standard*) only when `fact-pack.md` lists base commits that touch the same files or features.

Each writes `$STATE/findings/<lens>.md`. When all are done, dispatch the `sweep` finder (*strongest*; same file, Lens `sweep`), which reads the others and writes `$STATE/findings/sweep.md`.

Then merge by **root cause**: Findings that one fix would resolve become one entry, wherever they surface. Write each entry to its own file `$STATE/entries/<id>.md` (ids `F01`, `F02`, …):

```markdown
# <title naming the root cause>
- kind: finding | suspicion
- lenses: <every Lens that raised it>
- cause: <file:line>
- severity: <highest among the merged Findings; omit for a suspicion>
## Scenarios
- <file:line where it surfaces>: <input or state> → <wrong result>
```

A Suspicion becomes its own `suspicion` entry unless it restates an entry already there.

Then match against **prior comments**. Only now, after every finder has finished, read what others have already said on the PR, so that no finder or verifier is steered by it:

1. Collect every comment that raises a problem into `$STATE/prior-comments.md`, one line each: URL, author, location, gist, and whether its thread is resolved or the author replied that it is fixed. On GitHub, read inline review comments (`gh api --paginate repos/{owner}/{repo}/pulls/<n>/comments`), reviews and conversation comments (`gh pr view <n> --json reviews,comments`), and thread resolution (`reviewThreads { isResolved }` through `gh api graphql`). Other platforms: use an installed skill or project instruction that covers them; otherwise skip this matching. Comments from people and bots both count; CI status notices and the PR author's own explanations do not.
2. Compare each entry with them by root cause, the same test as merging: one fix resolves both. Record every match in `$STATE/prior-match.md` as `<id> | <kind> | <comment URL> | <entry title>`:
   - `prior`: an open comment already raises it. Delete the entry file; it is neither verified nor reported as new.
   - `extends`: the entry adds a materially different scenario or a clearly higher severity than the comment. Keep it.
   - `reraised`: the comment's thread is resolved or marked fixed, yet the entry still finds the problem at head. Keep it.

Then cap the nits. When more than ten `nit` entries remain, rank them by how much each matters to the Goal and to the code's future readers, and add the line `- verify: skipped (nit cap)` to every entry past the tenth.

## Step 3: Verify

Group the entries in `$STATE/entries/` not marked `verify: skipped`: all `nit` entries form one group, whatever their files; the rest are grouped by the file of their `cause`, splitting a group when it grows past about five entries. Dispatch one fresh verifier subagent per group with `references/verify.md`, `$STATE` and the group's entry ids, at the group's tier: the nit group *light*; any other group by its most severe entry, must-fix *strongest*, should-fix or suspicion *standard*. Run as many in parallel as the harness allows. A verifier writes one `$STATE/verdicts/<id>.md` per entry. It reads only its own entry files, `goal.md`, the fact pack and the code, so it forms its own view of each problem.

## Step 4: Report

Write `$STATE/report.md` following `references/report.md`, from `goal.md`, the verdicts, the skipped entries and `prior-match.md`. Reply to the user in their language with: the one-line merge verdict, counts per severity, the must-fix titles, and the path to `report.md`.

Post to the PR only when the user asks. Show them the exact text first and post after they approve it. A post leaves out the `prior` matches, which others have already raised. Record each posted comment in `$STATE/posted.md` as `<id> | <comment URL>`.

Finally remove the worktrees this skill created under `$STATE` (`git worktree remove --force`; they hold only build and test leftovers), keeping every other file in `$STATE`.
