# jj-spr-claude-plugin

A [Claude Code plugin](https://docs.anthropic.com/en/docs/claude-code) that teaches Claude how to use [jj SPR](https://github.com/LucioFranco/jj-spr) for GitHub pull request management with [Jujutsu](https://github.com/jj-vcs/jj).

## Credits

[jj SPR](https://github.com/LucioFranco/jj-spr) is created by [Lucio Franco](https://github.com/LucioFranco). This plugin provides Claude Code skills for working with jj SPR — it does not bundle or modify the tool itself.

## What This Does

Provides skills and slash commands for the full jj + SPR lifecycle:

- **`/spr-init`** — Set up SPR in a repository
- **`/spr-diff`** — Create or update PRs
- **`/spr-amend`** — Amend a change and update its PR
- **`/spr-land`** — Land a PR and clean up
- **`/spr-reorg`** — Reorganize a stack with existing PRs
- **`/spr-health`** — Diagnose stack health (read-only)
- **`/spr-recover`** — Recover a broken stack (stale PRs, ghost changes)
- **`/spr-status`** — Show change-to-PR mapping
- **`/spr-clean`** — Strip stale PR URLs from commit messages

## Install

```bash
claude --plugin-dir /path/to/jj-spr-claude-plugin
```

## Prerequisites

- [jj](https://github.com/jj-vcs/jj) installed and in PATH
- [jj-spr](https://github.com/LucioFranco/jj-spr) installed and in PATH
- A colocated repository (both `.jj/` and `.git/` directories)
- GitHub personal access token with `repo` scope

## Skills

| Skill | Triggers On |
|-------|-------------|
| `jj-spr-init` | "set up spr", "configure jj spr", "init spr" |
| `jj-spr-stacked-prs` | "create PR", "push changes", "spr diff", "stacked PRs" |
| `jj-spr-amend-update` | "update PR", "address review feedback", "amend and push" |
| `jj-spr-landing` | "land PR", "merge PR", "spr land", "ship it" |
| `jj-spr-reorganize` | "reorganize stack", "rebase changes", "squash changes" |
| `jj-spr-recovery` | "fix stack", "recover PRs", "stale PRs", "ghost changes", "stack is broken" |

## Key Convention

After Claude edits code in a jj change, it always repositions `@`:

```bash
jj new <change-id>   # @ is empty, @- is the modified change
```

This lets you review changes with `jj diff -r @-` before pushing.

## License

MIT
