# Report

Write `$STATE/report.md` in the language the user converses in (their agent instructions decide it; code and identifiers stay as they are), in this order. Every item is one root cause, the unit the verifiers judged; its scenarios are listed under it, never as separate items.

1. **Header**: target and URL, head SHA, and the Goal in one or two sentences. When `goal.md` says `confirmed: no`, say so on the first line. When steps ran at different heads (see `state.md`), say which. List open related PRs that were detected but not reviewed with it.
2. **Verdict**: one line. `Ready to merge`, `Ready after should-fix items`, or `Not ready: <n> must-fix`.
3. **must-fix**, then **should-fix**: CONFIRMED, not pre-existing, one block each:

   ```markdown
   ### <title naming the root cause>
   `cause file:line` · lens: <lenses>
   - <where it surfaces>: <input → wrong result>

   **Evidence** (<evidence level>): <the decisive evidence, short; method in one line>

   **Fix**: <smallest change>
   ```

   When `prior-match.md` marks the item `extends` or `reraised`, add a line linking the comment: "Extends <link>" or "Raised before and marked resolved, still present at head: <link>".

   For a GitHub PR, link locations as permalinks with the full head SHA.
4. **nit**: inside `<details><summary>nit, not verified (<n>)</summary>…</details>`, in the order of `nits.md`, one line each with its location and entry id, under a first line saying they were not verified and may be wrong.
5. **Unverifiable**: one line each, with what would settle it.
6. **Already raised**: the `prior` matches, one line each with the comment's link. They were not verified.
7. **Pre-existing problems**: one line each.
8. **Stats**: Findings raised, already raised by others, refuted, confirmed (with the count per evidence level), unverifiable, nits (not verified); path to `$STATE`.

Refuted Findings stay out of the report; their verdict files remain in `$STATE/verdicts/`.
