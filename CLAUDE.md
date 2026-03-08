# jj-spr Claude Plugin

## Critical: Preserving jj Change Descriptions

When working in any jj repository that uses SPR (stacked PRs), you MUST
preserve the `Pull Request:` line in change descriptions.

### The Rule

**NEVER use `jj describe -m "..."` without first reading the current
description and preserving the `Pull Request:` line.**

The `Pull Request: https://...` trailer is SPR's ONLY mechanism for
tracking which local change maps to which GitHub PR. If you remove it,
`jj spr diff` will create a DUPLICATE PR.

### Safe Pattern

```bash
# 1. Always read first
jj log -r <change-id> --no-graph -T 'description'

# 2. If it contains "Pull Request:", include it in the new description
jj describe -r <change-id> -m 'New title here

New summary here

Pull Request: https://github.com/org/repo/pull/12345'
```

### What NOT To Do

```bash
# WRONG - blindly overwrites, losing the Pull Request: line
jj describe -m "refactor: new description"

# WRONG - using jj describe without checking current description
jj describe -r abc -m "fix: something"
```

### Why This Matters

Every time the `Pull Request:` line is lost, `jj spr diff` creates a new
PR with a suffixed branch name (e.g., `-1`, `-2`, `-3`). This creates
orphaned remote branches and bookmark clutter that degrades `jj log`
readability across the entire repository.
