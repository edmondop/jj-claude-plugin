---
name: jj-spr-landing
description: Use when landing (merging) a PR via SPR and cleaning up afterward. Triggers on "land PR", "merge PR", "spr land", "ship it", or when a PR is approved and ready to merge.
---

# jj SPR Landing

## Overview

Land a PR by squash-merging it on GitHub, then sync the local repo. Covers
single PRs and stacked PR landing with proper cleanup.

## Pre-checks

Before landing:

1. **PR is approved** (if `spr.requireApproval` is configured)
   ```bash
   gh pr view <number> --json reviewDecision
   ```

2. **CI is passing**
   ```bash
   gh pr checks <number>
   ```

3. **No merge conflicts** — SPR will refuse to land if the PR can't be
   cleanly squash-merged

## Landing a Single PR

```bash
jj spr land -r <change-id>
```

This:
- Squash-merges the PR on GitHub
- Deletes the remote PR branch
- Updates the commit message with merge metadata

Then sync locally:

```bash
jj git fetch
jj rebase -r @ -d main@origin
```

## Landing the Bottom of a Stack

**Always land bottom-up.** Landing from the middle or top of a stack is
dangerous and will leave orphaned PRs.

### Step 1: Land the bottom PR

```bash
jj spr land -r <bottom-change-id>
```

### Step 2: Sync locally

```bash
jj git fetch
```

### Step 3: Rebase remaining stack onto updated main

```bash
jj rebase -s <next-change> -d main@origin
```

Where `<next-change>` is the change that was directly above the landed one.

### Step 4: Update remaining PRs

```bash
jj spr diff -r <next-change>::<top-of-stack>
```

This updates the base branches so remaining PRs show correct diffs.

### Step 5: Verify

```bash
jj log          # clean graph, no orphaned changes
jj spr list     # remaining PRs look correct
```

## Landing Multiple Stacked PRs

If all PRs in a stack are approved, land them one at a time, bottom-up:

```bash
# Land first
jj spr land -r <change-1>
jj git fetch
jj rebase -s <change-2> -d main@origin

# Land second
jj spr land -r <change-2>
jj git fetch
jj rebase -s <change-3> -d main@origin

# Continue until done
```

**Do not try to land all at once.** Each land changes the trunk, and the
next change needs to be rebased before it can be landed.

## Post-Landing Cleanup

After all landing is complete:

```bash
# Sync and rebase working copy
jj git fetch
jj rebase -r @ -d main@origin

# Verify clean state
jj log
```

The landed changes will appear as immutable commits (`◆`) in the log.

## Common Mistakes

- **Landing from the middle of a stack** — leaves upper PRs orphaned with
  wrong base branches. Always land bottom-up.
- **Forgetting `jj git fetch`** — local repo won't know about the merge
  on GitHub. Must fetch before rebase.
- **Forgetting to rebase remaining stack** — remaining PRs will have stale
  base branches and show wrong diffs on GitHub.
- **Forgetting to update remaining PRs** — after rebase, run
  `jj spr diff -r <range>` to push updated synthetic commits.
- **Landing with failing CI** — SPR may allow it, but reviewers and
  automation expect green CI first.
