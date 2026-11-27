---
name: worktree-isolation
description: Isolate concurrent or risky work in a git worktree and clean up stale ones. Use on "worktree", "need isolation", "parallel agents on the same repo", "แยก branch ทำงาน".
---

# Git worktree isolation
Use plain git. Skip this for read-only research and tiny single-agent edits.

1. Preflight: `git -C "<repo>" worktree list` and `git -C "<repo>" status --short`. Note stale entries.
2. Create: `git -C "<repo>" worktree add "<repo>/.worktrees/<name>" -b "<name>"`. Add `.worktrees/` to `.git/info/exclude` if it is not ignored.
3. Work through absolute paths or `git -C "<repo>/.worktrees/<name>" ...`. Never `cd` into the worktree in a persistent shell: if it is removed while the shell is inside, the shell dies. Use a subshell `( cd "<path>" && cmd )` when a command needs it.
4. One worktree per parallel worker, each on its own branch. A shared contract (types, schema) goes in first on the base branch, then workers branch from it.
5. Merge: `git -C "<repo>" merge --no-ff "<name>"` from the base branch after the worker's checks pass.
6. Clean up only a worktree that is merged or has no unique commits: `git -C "<repo>" worktree remove "<repo>/.worktrees/<name>"` then `git -C "<repo>" branch -d "<name>"`. Unmerged commits: show them to the user and ask; never `--force` without approval.
7. Shared state outside git (memory, notes, ledgers) is never branched; workers report to the lead.
