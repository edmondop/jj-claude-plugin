---
name: jj-spr-reorganize
description: Use when reorganizing a jj change stack that has existing PRs. Triggers on "reorganize stack", "rebase changes", "squash changes", "split change", "reorder PRs", or after restructuring a stack that had open PRs.
---

# jj SPR Stack Reorganization

## Overview

Reorganizing a stack (rebase, squash, split, reorder) when PRs already exist
is the most dangerous SPR operation. SPR tracks PRs solely via
`Pull Request:` URLs in commit messages. After reorganization, old URLs are
stale and will cause errors or wrong updates.

**The only safe path is: close → clean → reorganize → recreate.**

## Why Close-and-Recreate?

- `Pull Request:` URLs are SPR's ONLY tracking mechanism
- Stale URLs point to closed or wrong PRs
- Partial cleanup (updating some URLs) is fragile and error-prone
- A clean slate is the only reliable approach

## Step 1: Close All Affected PRs

Close every PR in the range you're about to reorganize:

```bash
# List current PRs to know what to close
jj spr list

# Close each one (deletes the remote branch too)
gh pr close <number-1> --delete-branch
gh pr close <number-2> --delete-branch
# ... repeat for all affected PRs
```

**Do not skip `--delete-branch`.** Leftover SPR branches cause confusion.

## Step 2: Strip Stale `Pull Request:` URLs

For every commit in the stack, remove the `Pull Request:` line from the
commit message:

```bash
# For each change in the stack:
jj desc <change-id> -m "<message without Pull Request: line>"
```

**Check every commit.** A single leftover URL will cause SPR to try to
update a closed PR, resulting in errors.

To see which commits have URLs:

```bash
jj log -T 'change_id ++ " " ++ description'
```

## Step 3: Perform the Reorganization

Now that PRs are closed and URLs are stripped, use standard jj commands:

### Rebase (move a change to a new parent)
```bash
jj rebase -r <change> -d <new-parent>
```

### Squash (fold a change into its parent)
```bash
jj squash -r <change>
```

### Split (break one change into two)
```bash
jj split -r <change>
```

### Reorder (move changes around in the stack)
```bash
jj rebase -r <change> --after <target>
# or
jj rebase -r <change> --before <target>
```

## Step 4: Verify the New Stack

```bash
jj log
```

Confirm:
- Changes are in the desired order
- No unexpected conflicts
- Commit messages are clean (no stale `Pull Request:` URLs)

## Step 5: Recreate PRs

```bash
jj spr diff -r <first-change>::<last-change>
```

SPR creates fresh PRs with correct base branch chaining.

## Step 6: Reposition `@`

```bash
jj new <top-of-stack>
```

`@` is now an empty change above the stack. Ready for new work.

## Quick Reference

```
1. jj spr list                              # note PR numbers
2. gh pr close <N> --delete-branch           # for each PR
3. jj desc <id> -m "<clean message>"         # for each change
4. jj rebase / squash / split               # reorganize
5. jj log                                    # verify shape
6. jj spr diff -r <range>                   # recreate PRs
7. jj new <top>                              # reposition @
```

## Common Mistakes

- **Reorganizing without closing PRs first** — stale URLs cause SPR to
  update wrong PRs or error out
- **Forgetting to strip `Pull Request:` URLs** — even one leftover URL
  means SPR thinks a PR already exists for that change
- **Trying to be clever with partial cleanup** — updating some URLs while
  keeping others. This is fragile. Always do a full close-and-recreate.
- **Not verifying the stack shape before recreating** — if the stack has
  conflicts or wrong order, PRs will be created in the wrong state
- **Leaving `@` on a PR change** — always `jj new` after reorganization
