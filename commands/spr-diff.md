---
name: spr-diff
description: Create or update PRs for jj changes using SPR
---

# /spr-diff

Create or update GitHub pull requests for jj changes.

## Instructions

Use the `jj-spr-stacked-prs` skill to manage PRs.

1. Run `jj log` to understand the current change stack
2. Follow the pre-flight checklist from the skill
3. Determine the correct range (single change vs. stack)
4. Run `jj spr diff -r <range>`
5. Report the created/updated PR URLs

If the user specifies a revision, use it. Otherwise, analyze the stack and
suggest the appropriate range before proceeding.

**Always follow the `@` positioning convention** — if any edits were made,
run `jj new <change-id>` before running `jj spr diff`.
