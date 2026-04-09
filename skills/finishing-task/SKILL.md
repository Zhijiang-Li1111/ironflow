---
name: finishing-task
description: "Ensure the user sees and approves what was built before any integration happens. Use when implementation and reviews are complete and the work is ready to be delivered, merged, or shared with the team. Make sure to use this skill whenever work is ready to ship, even if it's a small change."
---

# Finishing a Task

After serial-review passes, present a summary to the user, get their confirmation, then integrate the work.

Why: "Done implementing" is not the same as "done." The user needs to see what was built, confirm it matches expectations, and decide how to integrate it — before any merge or PR happens.

## Process

### 1. Verify tests pass

Run the full test suite. If anything fails, stop and fix before proceeding.

### 2. Present summary to user

Before taking any action, give the user a clear summary:

- **What was built** — brief description of the changes
- **Files changed** — list of created/modified/deleted files
- **Tests** — what tests were added or modified, current test status
- **Workarounds** — any non-ideal solutions used (from serial-review's workaround summary)
- **Known issues** — anything worth noting

Wait for the user to review and confirm. The user may have questions, want adjustments, or want to inspect specific files before proceeding — rushing past this step means the user loses their chance to catch issues before integration.

### 3. Present integration options

After user confirms the summary, offer these choices:

- **Merge locally** — merge into a target branch on this machine
- **Push and create a Pull Request** — push the branch and open a PR for team review
- **Keep the branch as-is** — leave it for now, come back later
- **Discard this work** — delete the branch and changes

Wait for the user to choose — assuming an integration path can lead to unwanted merges or PRs that are hard to undo.

### 4. Execute the choice

For **merge locally**: ask the user which target branch to merge into (do not assume the default branch), merge (default to merge commit unless the user prefers rebase or squash), verify tests pass after merge.

For **PR**: push the branch and create a PR with a proper title and description. When the PR receives review feedback, use the receiving-review skill to process it.

For **keep as-is**: do nothing. Remind the user of the branch name for later.

For **discard**: confirm with the user once more, then delete the branch and clean up.

### 5. Clean up

Clean up depends on the isolation style and integration choice:

**Worktree style:**

- **Merge**: `ExitWorktree` with `action: "remove"` — work is delivered.
- **Discard**: `ExitWorktree` with `action: "remove"` and `discard_changes: true`.
- **PR**: do **not** clean up. Notify the user that the worktree and branch will remain until the PR is merged, and that they can clean up later.
- **Keep as-is**: `ExitWorktree` with `action: "keep"`.

**Branch only style:**

- **Merge**: switch back to the default branch, delete the task branch.
- **Discard**: switch back to the default branch, delete the task branch.
- **PR**: do **not** clean up. Notify the user that the branch will remain until the PR is merged.
- **Keep as-is**: switch back to the default branch, leave the task branch intact.

## Common Mistakes to Avoid

- Merging without running the full test suite first
- Merging to the wrong target branch (always confirm)
- Forgetting to push before creating a PR
- Cleaning up a worktree or branch the user wanted to keep
- Skipping the user summary and jumping straight to "merge or PR?"
- Creating a PR without mentioning the receiving-review skill for handling feedback
