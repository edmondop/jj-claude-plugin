---
name: spr-land
description: Land a PR via SPR and clean up
---

# /spr-land

Land (squash-merge) a PR on GitHub and sync the local repository.

## Instructions

Use the `jj-spr-landing` skill to guide the landing process.

1. Identify which PR to land (ask user or infer from context)
2. Run pre-checks (approval status, CI)
3. `jj spr land -r <change-id>`
4. `jj git fetch` and `jj rebase -r @ -d main@origin`
5. If part of a stack, rebase remaining changes and update their PRs

**Always land bottom-up** in a stack. Warn the user if they're trying to
land from the middle or top.
