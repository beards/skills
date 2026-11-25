# Reviewing a Review set

A **Review set** is several PRs reviewed together, usually one change split across repos (a library and its consumers). Members that are merged or closed are **Background PRs**: they give context, and no problem is reported against them. Everything in the other reference files still applies; this file says only what changes.

## Setting up (orchestrator)

1. **Members.** Each argument is a PR URL or `owner/repo#n`; a bare number means the repo of the current directory. Give each member an id `<repo>-pr-<n>`. The target id is `group-<first member id>`.
2. **Clones.** For each repo, use a clone in the current directory or a sibling directory whose `origin` remote matches. When there is none, run `gh repo clone <owner/repo> $STATE/clones/<repo>` without asking.
3. **Background PRs.** A member that is merged or closed is a Background PR. Other members get the normal preflight; drop a member with an empty diff and tell the user.
4. **Worktrees.** Every member, Background PRs included, gets `git -C <clone> worktree add --detach $STATE/head/<member id> <its head sha>`.
5. **`state.md`** lists one line per member, in place of the single target lines: `<member id> | <url> | review or background | base | merge-base | head`.

## goal.md and the fact pack

- `goal.md` keeps its template. `## Goal` holds the shared Goal; add `## Per PR` with one line per member saying its part in it, Background PRs marked as such. `source:` is `inferred` when any part had to be inferred.
- `fact-pack.md` gets one `## <member id>` section per member, each with the usual sections. Search call sites across every member's worktree, so a symbol one PR adds is traced into the PRs that use it. A Background PR's section needs only Refs, Diff and Changed symbols.

## For every subagent

- Code for a member is at `$STATE/head/<member id>/`.
- Write every location as `<member id>/<repo-relative path>:<line>`.
- Finders review all non-background members together, and look hard at where the PRs meet: an API one PR adds and another calls, a config one writes and another reads.
- A verifier makes one worktree per member that holds a `cause` in its group, from that member's clone: `git -C <clone> worktree add --detach $STATE/worktrees/<first id>-<member id> <that member's head>`.

## Attribution and report (orchestrator)

- An entry belongs to the member holding its `cause`. A cause in a Background PR or outside the set belongs to the member whose change exposes it; when none does, it is pre-existing.
- Match prior comments per member, against that PR's own comments.
- `report.md`: one header for the whole set (shared Goal, every member with its head, Background PRs listed), then one `## <member id>` section per reviewed member with its own verdict line and the severity, Unverifiable, Already raised and Pre-existing parts. One Stats part closes the report.
- Posting to a PR uses only that PR's section.
