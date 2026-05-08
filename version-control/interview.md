# Version Control (Git) Interview Questions

1. **What is version control and why is it important?**
   - A system that records changes to files over time so you can recall specific versions later. It enables collaboration (multiple developers without overwriting each other), rollback (undo bad changes), and history (who changed what and why).

2. **What is the difference between git fetch and git pull?**
   - `git fetch` downloads remote changes but does NOT apply them to your working branch — you review first. `git pull` = `git fetch` + `git merge` — it downloads and immediately merges into your current branch.

3. **What is the difference between git merge and git rebase?**
   - **Merge** creates a merge commit preserving full branch history. **Rebase** replays your commits on top of the target branch, creating a linear history. Never rebase commits that have been pushed to a shared branch — it rewrites history and breaks others' copies.

4. **What is a merge conflict and how do you resolve it?**
   - When two branches change the same line differently, Git can't decide which to keep and marks the conflict with `<<<<<<<`, `=======`, `>>>>>>>`. You manually edit the file to the correct final state, then `git add` and `git commit` to finish the merge.

5. **What is the difference between git reset and git revert?**
   - `git reset` moves the branch pointer backward — it removes commits from history. Dangerous on shared branches. `git revert` creates a new commit that undoes a previous commit — safe for shared branches because it doesn't rewrite history.

6. **What does git stash do?**
   - Temporarily saves your uncommitted changes to a stack so you can switch branches or pull updates with a clean working directory. `git stash pop` restores them.

7. **What is a Pull Request (PR) or Merge Request (MR)?**
   - A request to merge a feature branch into another branch (usually main). It enables code review, runs CI checks, and creates a discussion thread before code is integrated. PRs are GitHub's term; MRs are GitLab's.

8. **What is the difference between git add . and git add -p?**
   - `git add .` stages all changes in the current directory. `git add -p` (patch mode) lets you interactively stage individual hunks — useful for committing only part of your changes.

9. **What is git blame?**
   - Shows who last modified each line of a file, along with the commit hash and timestamp. Useful for understanding why code exists and who to ask about it.

10. **What is a good commit message format?**
    - Follow Conventional Commits: `type(scope): short description`. Types: `feat` (new feature), `fix` (bug fix), `docs`, `refactor`, `test`, `chore`. Keep under 72 chars. The body explains the *why*, not the *what*.
    ```
    feat(auth): add JWT refresh token rotation
    fix(cart): prevent duplicate items on rapid click
    docs(readme): update setup instructions for Node 20
    ```
