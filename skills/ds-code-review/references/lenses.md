# Finding problems through Lenses

You review one change through the Lenses your brief names, one after another, giving each its full checklist below. Start from `$STATE/goal.md` and `$STATE/fact-pack.md`; read the code at `$STATE/head/` and anywhere else in the repo as deeply as each Lens needs.

## What you report

Report every problem you can tie to a concrete failure, however minor. Filtering is the verifier's job, so a Finding you hold back is a problem lost. Each Finding has all of:

```markdown
### <short title>
- lens: <the Lens that found it>
- cause: <repo-relative file:line at head where the faulty code is>
- surfaces: <file:line where the wrong result shows, when it differs>
- scenario: <the input, state or sequence> → <the wrong result>
- why: <the code that produces it, quoted briefly>
- severity: must-fix | should-fix | nit
```

Report problems outside your Lenses too when you see them; the merge step folds duplicates. A concern you cannot turn into a concrete scenario goes under a final `## Suspicions` heading as one line each, with its location. Verification will dig into it.

Severity: **must-fix** produces wrong behaviour, misses the Goal or breaks existing behaviour; **should-fix** is a design, convention or maintainability problem with a concrete reason; **nit** is minor.

Write everything to the findings file your brief names (`$STATE/findings/<finder>.md`) and return one line: the path and the number of Findings and Suspicions.

## Lenses

### goal
Does the change achieve the Goal? Check every item under Acceptance in `goal.md`. For a bug fix, trace the reported symptom to its cause and check the change removes that cause: a fix that special-cases one input, suppresses the error, adds a retry or catches and ignores, while the cause remains, is a Finding.

### regression
Does the change break behaviour outside the Goal? Audit every removed or rewritten line: what did it do, and who relied on it? For every changed or removed symbol, visit each call site in the fact pack and check it still holds: signature, return values, error behaviour, side effects, defaults, ordering.

### edge
Does the change fail under particular inputs or timing? Inputs: empty, missing, null, zero, negative, boundary, huge, malformed, non-ASCII. Timing: concurrent calls, ordering, retries, partial failure, timeouts, cancellation, resource cleanup. Language pitfalls in the languages the diff uses: the traps of that language's semantics, standard library and runtime that the code walks into.

### design
Is there a simpler, more direct way? Abstractions, parameters or configuration with no second user, indirection that only delegates. Reinvented wheels: search the repo and its dependencies for an existing utility that does the job, and name it with its location. Convention breaches: cite the documented rule (file and rule) or show the established pattern with two existing examples.

### drift
Do newer commits on the base collide with this change? For each commit under Base drift in the fact pack: does it implement the same thing (duplicate), change an assumption this change relies on (semantic conflict, even when the merge is clean), or touch the same lines (textual conflict)? Cite the base commit SHA.

### sweep
Read every `$STATE/findings/*.md` first. Then hunt what the other Lenses leave behind: error-handling paths, tests that do not actually exercise the change or that were weakened or deleted, missing tests for a bug fix, security (injection, authorization, secrets), migrations and data compatibility, docs and public API contracts. Report only Findings absent from the other files.
