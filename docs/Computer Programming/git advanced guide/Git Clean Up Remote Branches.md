# Git: Clean Up Remote Branches



### 1. The "Prune" Command (Recommended)

The most efficient way to sync your local record with the remote is to use the `prune` flag. This removes the "remote-tracking" branches (the ones that show up as `origin/branch-name`) that no longer exist on the server.

* **To prune a specific remote:**
`git fetch origin --prune`
* **To prune all remotes at once:**
`git fetch --all --prune`

### 2. Set it and Forget it

If you don't want to manually prune every time, you can tell Git to do this automatically every time you run `git fetch` or `git pull`:

```bash
git config --global fetch.prune true

```

---

### 3. Deleting the Local Branch

Pruning only removes the **references** to the remote branch. If you actually checked that branch out locally (so it appears in `git branch`), you still need to delete it manually:

1. **List your branches** to see what's left: `git branch`
2. **Delete the branch:** `git branch -d <branch-name>`
*(Note: Use `-D` if you want to force delete it without checking if it was merged.)*

### Summary of Commands

| Action | Command |
| --- | --- |
| **Sync remote list** | `git fetch -p` |
| **Delete local copy** | `git branch -d <name>` |
| **Check status** | `git remote show origin` |

> **Pro-tip:** Running `git remote show origin` is a great way to see a "report" of which local branches are out of sync with the remote before you start deleting things.

---
@end