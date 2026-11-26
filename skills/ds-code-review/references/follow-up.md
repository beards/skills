# Follow-up review

A **Follow-up review** reviews a change again after a finished review, covering only the **delta** (what changed since the last round) and the problems the last round confirmed. Everything in the other reference files still applies; this file says only what changes.

## Setting up (orchestrator)

1. **Rounds.** Move every file in `$STATE` except `rounds/` into `$STATE/rounds/<n>/`, where `<n>` is one more than the highest round already there (the first move makes `rounds/1/`). Call that directory `$PREV`.
   When `$PREV/posted.md` is missing (a round from an older version), rebuild it: list the PR's comments written by the reviewing account (`gh api user`), and any posting note in `$PREV/state.md`; match each to an entry by location and title; write the lines to `$PREV/posted.md`.
2. **Goal.** Copy `$PREV/goal.md` to `$STATE/goal.md`. Fetch the PR description again, and every ticket `goal.md` cites. When what the change must achieve now differs, escalate (step 5).
3. **Delta.** Take the old merge-base and head from `$PREV/state.md`.
   - Merge-base unchanged: the delta is `git diff <old head> <new head>`.
   - Merge-base moved (a rebase): run `git range-diff <old merge-base>..<old head> <new merge-base>..<new head>`. The delta is the new commits plus the old commits whose patch changed; commits that changed only by the rebase are not delta. The base's own new commits go to the `drift` lens.
4. **`state.md`.** Create `$STATE/head/` at the new head as usual and write `state.md` with an extra line `round: <n + 1>   previous: <old head sha>` and these steps:

   ```markdown
   - [ ] 1 delta
   - [ ] 2 find
   - [ ] 3 verify
   - [ ] 4 report
   ```

5. **Escalate** to a full review when the Goal changed, or when `range-diff` cannot pair most old commits with new ones. Tell the user which, then start a full review as `--full` does. The delta's size alone never escalates: the `main` finder covers every Lens on the delta, however large. In a Review set, compute the delta per member; a member with no new commits has no delta, and one member escalating makes the whole set a full review.

## Step 1: delta

Dispatch one *standard* subagent with `references/fact-pack.md`, this file and `$STATE`. It copies `$PREV/fact-pack.md` to `$STATE/fact-pack.md` and appends `## Follow-up <n + 1>`: the delta's commits, the delta (diff or `range-diff` output), the symbols it changes, and their call sites at the new head.

## Step 2: find

- Finders: `main` and then `sweep`, as in a full review, on the delta. `main` looks hardest at whether a fix for a previous problem breaks something else, and covers `drift` when the base gained commits that touch the same files or features.
- A Finding belongs here only when its cause is in the delta or the delta exposes it.
- New entry ids continue from the highest id in `$PREV/entries/` and `$PREV/verdicts/`.
- Prior comments: match only comments not listed in `$PREV/prior-comments.md`, and leave out the comments recorded in `$PREV/posted.md`.
- New nits and Suspicions are set aside as usual; `nits.md` lists new nits only.

## Step 3: verify

**Previous items**, by their verdict in `$PREV/verdicts/`:

| Last verdict | This round |
|---|---|
| CONFIRMED must-fix or should-fix, not pre-existing | verify again |
| UNVERIFIABLE | verify again only when the delta touches its `cause` file |
| nit or suspicion (verified or not) | not verified; record whether the delta touches its `cause` lines |
| REFUTED, pre-existing, `prior` | carry over |

An item whose member has no delta is carried over, unless its thread gained a reply from the author since the last round; then verify it again.

Copy each item to verify again from `$PREV/entries/` into `$STATE/entries/`, keeping its id, and group these items as a full review groups entries (must-fix *strongest*, the rest *standard*, split only past about fifteen). They need nothing from the finders, so dispatch their verifiers at the start of step 2, alongside the `main` finder. New entries are verified after the finders, as usual. Tick step 3 when both are done. Give a verifier of previous items, for each one: the entry, `$PREV/verdicts/<id>.md`, and, when `$PREV/posted.md` lists a comment for it, that comment's thread with the author's replies.

A verifier of a previous item writes the usual verdict file, adding:

```markdown
round: <n + 1>
status: fixed | still present | partial | accepted | disputed
```

- `fixed`: no scenario happens at the new head (verdict REFUTED).
- `still present`: the scenario happens as before.
- `partial`: some scenarios are fixed and others remain, or the fix covers only part of the cause.
- `accepted`: the author declined to change it, and their reason holds (verdict REFUTED).
- `disputed`: the author declined to change it, and their reason does not hold. Quote the reason under Evidence, next to your own evidence.

## Step 4: report

`report.md` follows `references/report.md`, with these changes:

- The header says "Follow-up review, round <n + 1>, against <old head>" and links `$PREV/report.md`.
- Before the severity parts, **Previous items**: one line each with its status, title and comment link; a previous nit or suspicion shows `cause touched` or `cause untouched` in place of a status. `still present`, `partial` and `disputed` items then get the usual block, under their severity.
- The other parts hold new entries only. Carried-over items stay in `$PREV/report.md`.
- The verdict line counts `still present`, `partial`, `disputed` and new CONFIRMED items.
- Stats add the count per status.

Posting: answer each previous item in its own thread from `$PREV/posted.md`. A `fixed` or `accepted` item gets a one-line confirmation, the others their evidence and fix. New items get new comments. Record every comment and reply in `$STATE/posted.md`, so the next round finds each thread.
