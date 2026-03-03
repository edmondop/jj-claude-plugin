---
name: jj-spr-stacked-prs
description: Use when creating, updating, or managing stacked PRs in a jj repository. Triggers on "create PR", "push changes", "spr diff", "open PR", "stacked PRs", or after making changes that need to go to GitHub.
---

# jj SPR Stacked PRs

## How SPR Works (must understand before using)

SPR pushes **synthetic commits** to GitHub, never the real jj commits. Each
PR branch gets a synthetic commit with the same tree content but different
parents (forming an append-only history for clean review diffs).

For stacked PRs, SPR creates a **base branch** (`spr/.../master.<title>`)
with a synthetic commit whose tree matches the parent commit's tree. This
makes GitHub show only each PR's own changes in the diff.

**The `Pull Request:` URL in the commit message is SPR's ONLY way to track
which local commit maps to which PR.** There is no cache, config, or other
mechanism. If this URL is missing or stale, SPR loses track.

## How SPR Handles Immutable Commits

SPR's workflow is two phases:
1. **Push branch + create PR on GitHub** (irreversible)
2. **`jj describe` to write PR URL back into commit message** (can fail)

If a commit is immutable, Phase 1 succeeds but Phase 2 fails. The PR exists
on GitHub but SPR can't write the URL back. This means:
- The PR is created and correct — no action needed on GitHub
- SPR has no local record of it — running `jj spr diff` again would create
  a DUPLICATE PR
- **Never include an immutable commit in SPR ranges after its PR exists**

## The `@` Positioning Convention

**After any code edits to a jj change, always reposition `@`:**

```bash
jj new <change-id>   # @ is now an empty change, @- is the modified change
```

Never leave `@` pointing at a PR change after edits. The user needs `@-` to
point at the modified change so they can review with `jj diff -r @-`.

## Pre-flight Checklist

Before running any SPR command:

1. **Strip stale PR URLs** — if you reorganized the stack (rebase/squash),
   every commit may have `Pull Request: https://...` lines from old PRs.
   Run `jj desc <change-id> -m "<clean message>"` for each affected commit.

2. **Identify immutable commits** — look for `◆` in `jj log`. If an
   immutable commit already has its PR, exclude it from the range.

3. **Determine the range** — never use `jj spr diff --all` blindly. Always
   use an explicit range:
   ```bash
   jj spr diff -r <first-change>::<last-change>
   ```

4. **Ensure `@` is on the stack** — SPR operates relative to `@`. If `@`
   is an empty side change, move it first:
   ```bash
   jj new <top-of-stack>
   ```

5. **Run SPR from colocated workspace** — SPR needs a `.git` directory.

## Decision Tree

### Single Change (one PR)
```bash
jj spr diff -r <change-id>
```

### Stack of Changes (multiple PRs)
```bash
jj spr diff -r <first-mutable-change>::<last-change>
```

SPR creates one PR per commit with automatic base branch chaining.

### Never
```bash
# DON'T — includes everything back to trunk, possibly immutable commits
jj spr diff --all
```

## Creating Stacked PRs

```bash
# From colocated workspace, with @ on top of stack:
jj spr diff -r <first-mutable-change>::<last-change>
```

SPR creates one PR per commit with automatic base branch chaining via
synthetic base branches. Do NOT manually change PR base branches — SPR's
synthetic commits handle the diffs correctly.

## Common Mistakes

- **Never use `jj git push` + `gh pr create`** for PR workflows. Always
  use SPR. Manual PRs don't share ancestry with SPR's synthetic commits,
  so GitHub diffs will be wrong.
- **Never manually change a PR's base branch** away from SPR's managed
  base branches.
- **Never use `jj spr diff --all`** without verifying no immutable commits
  with existing PRs are in the ancestry path.
- **Never forget to clean commit messages** after closing old PRs and
  before running SPR again.
- **Never include an immutable commit in an SPR range twice** — it will
  create duplicate PRs since SPR can't write the URL back.
