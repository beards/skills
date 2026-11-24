# Report

Write `$STATE/report.md` in the language the user converses in (their agent instructions decide it; code and identifiers stay as they are), in this order. Every item is one root cause, the unit the verifiers judged; its scenarios are listed under it, never as separate items.

1. **Header**: target and URL, head SHA, and the Goal in one or two sentences. When `goal.md` says `confirmed: no`, say so on the first line.
2. **Verdict**: one line. `Ready to merge`, `Ready after should-fix items`, or `Not ready: <n> must-fix`.
3. **must-fix**, then **should-fix**: CONFIRMED, not pre-existing, one block each:

   ```markdown
   ### <title naming the root cause>
   `cause file:line` · lens: <lenses>
   - <where it surfaces>: <input → wrong result>

   **Evidence**: <the decisive evidence, short; method in one line>

   **Fix**: <smallest change>
   ```

   When `prior-match.md` marks the item `extends` or `reraised`, add a line linking the comment: "Extends <link>" or "Raised before and marked resolved, still present at head: <link>".

   For a GitHub PR, link locations as permalinks with the full head SHA.
4. **nit**: at most ten, ranked by how much each matters to the Goal and to the code's future readers, most important first; inside `<details><summary>nit (<shown> of <total>)</summary>…</details>`, one line each with its fix. The rest stay in `$STATE/verdicts/`.
5. **Unverifiable**: one line each, with what would settle it.
6. **Already raised**: the `prior` matches, one line each with the comment's link. They were not verified.
7. **Pre-existing problems**: one line each.
8. **Stats**: Findings raised, already raised by others, refuted, confirmed, unverifiable, nits left out; path to `$STATE`.

Refuted Findings stay out of the report; their verdict files remain in `$STATE/verdicts/`.
