# Report

Write `$STATE/report.md` in the user's language, in this order:

1. **Header**: target and URL, head SHA, and the Goal in one or two sentences. When `goal.md` says `confirmed: no`, say so on the first line.
2. **Verdict**: one line. `Ready to merge`, `Ready after should-fix items`, or `Not ready: <n> must-fix`.
3. **must-fix**, then **should-fix**: CONFIRMED, not pre-existing, one block each:

   ```markdown
   ### <title>
   `file:line` · lens: <lenses>
   <scenario: input → wrong result>
   Evidence: <the decisive evidence, short; method in one line>
   Fix: <smallest change>
   ```

   For a GitHub PR, link locations as permalinks with the full head SHA.
4. **nit**: inside `<details><summary>nit (<n>)</summary>…</details>`, one line each.
5. **Unverifiable**: one line each, with what would settle it.
6. **Pre-existing problems**: one line each.
7. **Stats**: Findings raised, refuted, confirmed, unverifiable; path to `$STATE`.

Refuted Findings stay out of the report; their verdict files remain in `$STATE/verdicts/`.
