# skills

Agent Skills by beards, following the [Agent Skills specification](https://agentskills.io/specification). Every skill is installable on its own; installing all of them at once never conflicts.

Primary targets: Claude Code, Codex, Grok Build, and Pi.

## Install

Every install goes through [`npx skills`](https://github.com/vercel-labs/skills). Pick your agent with `-a` (`claude-code`, `codex`, `grok`, `pi`) and add `-g` for a user-level install.

```sh
# list what's available
npx skills add beards/skills --list

# one skill
npx skills add beards/skills -s ds-code-review -a claude-code -g

# every skill
npx skills add beards/skills -s '*' -a claude-code -g

# update installed skills
npx skills update
```

Stick to `npx skills` for every skill you take from this repo. Another channel (a Claude Code plugin, `gh skill install`, `pi install`) installs a second copy under a different path, and the agent then sees the skill twice.

## Skills

| Skill | What it does |
| ----- | ------------ |
| `ds-code-review` | Reviews a PR or branch against its goal from several lenses (goal, regression, edge cases, design, base drift), then verifies every finding with evidence before reporting it. |

## Contributing

Conventions for adding a skill live in [AGENTS.md](AGENTS.md). Run `scripts/validate` before committing; CI runs the same checks.

## License

[MIT](LICENSE)
