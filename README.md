# personal-workflow

A personal [Claude Code](https://claude.com/claude-code) plugin marketplace (`personal-plugins`), plus loose workflow notes.

## Plugins

### `vendorsmart`

Personal VendorSmart workflow helpers.

| Skill | Invoke | What it does |
|---|---|---|
| `vs-create-branch` | `/vendorsmart:vs-create-branch <task-id> [type]` | Creates and switches to `{type}/{ID}-{description}` from a ClickUp task, inferring the type from the task and kebab-casing its name. Requires a connected ClickUp MCP server. |
| `vs-commit` | `/vendorsmart:vs-commit [description]` | Stages everything and commits as `{type}: #{ID} short description`, with the type and ticket ID parsed from the current branch name. Subject capped at 70 characters; body only when strictly necessary. |

Both use the same branch types: `feat`, `fix`, `hotfix`, `docs`, `refactor`, `test`, `chore`.

### `git`

Generic git helpers for repos that do not track work with ticket IDs.

| Skill | Invoke | What it does |
|---|---|---|
| `commit` | `/git:commit [description]` | Stages everything and commits as `{type}: short description`. The type comes from the branch prefix when there is one, otherwise it is inferred from the diff. No ticket ID in the subject. Subject capped at 70 characters; body only when strictly necessary. |

Use this instead of `vs-commit` on personal repos; use `vs-commit` where the subject needs `#{ID}`.

## Install

```
/plugin marketplace add C:/Users/mathe/personal/personal-workflow
/plugin install vendorsmart@personal-plugins
/plugin install git@personal-plugins
/reload-plugins
```

A local absolute path works as a marketplace source, so this is usable straight from a clone — no need to push first. To track the remote instead, point the marketplace at `msmadeira/personal-workflow` on GitHub.

## Notes

- [`git-helpers.md`](git-helpers.md) — git config, aliases, and shell functions worth keeping around.

## Adding a plugin

1. Create `plugins/<name>/.claude-plugin/plugin.json` with `name`, `description`, and `author`.
2. Add skills under `plugins/<name>/skills/<skill>/SKILL.md`. They are auto-discovered — no `skills` key in the manifest.
3. Register the plugin in [`.claude-plugin/marketplace.json`](.claude-plugin/marketplace.json).
4. Validate before installing:

   ```bash
   claude plugin validate .
   ```
