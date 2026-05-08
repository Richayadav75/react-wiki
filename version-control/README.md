- Category: DevOps / Tools
- Difficulty: Beginner
- Related: ci-cd

### Version Control with Git — A Time Machine for Your Code
Git is a distributed version control system that tracks changes to files over time. Every change is recorded as a snapshot (commit), making it possible to go back to any point in history, collaborate with a team without overwriting each other's work, and ship features safely through branches.

**Analogy**
Google Docs' version history — but for code, across a whole project, used by a whole team simultaneously. Every save is labelled, every version retrievable, every author tracked. And unlike Docs, you can create parallel "copies" of the document (branches) that can be merged back together later.

---

### 1. Core Concepts — Repository, Commit, Branch, Remote

**Theory**
| Concept | Meaning |
| :--- | :--- |
| **Repository (repo)** | A project folder tracked by Git, contains all history |
| **Commit** | A saved snapshot of changes, with a message and author |
| **Branch** | A parallel line of development (like a copy of the codebase) |
| **Remote** | A hosted copy of the repo (GitHub, GitLab, Bitbucket) |
| **Clone** | Download a remote repo to your machine |
| **Push** | Upload local commits to the remote |
| **Pull** | Download latest remote changes to your local |

**Working Flow**

![flow-chart](flow-chart.png)

**Example**
```bash
# Initialize a new repo
git init

# Check status — what has changed
git status

# Stage changes (add to next snapshot)
git add file.js          # specific file
git add .                # all changes in current folder

# Commit — save a snapshot
git commit -m "Add login form validation"

# See history
git log --oneline
# a1b2c3d Add login form validation
# e4f5g6h Add signup page
# i7j8k9l Initial commit

# Connect to remote (GitHub)
git remote add origin https://github.com/user/project.git

# Push to remote
git push -u origin main

# Pull latest from remote
git pull origin main
```

**Output**
```
git status   → modified: src/Login.jsx
git add .    → file staged
git commit   → [main a1b2c3d] Add login form validation
git push     → To github.com/user/project.git
               main -> main
```

---

### 2. Essential Commands Reference

**Theory**
Every day with Git involves a small set of commands. Mastering these covers 90% of daily usage.

**Working Flow**

![flow-chart-2](flow-chart-2.png)

**Example**
```bash
# ── Setup ──────────────────────────────────────────────
git config --global user.name "Your Name"
git config --global user.email "you@example.com"

# ── Starting ───────────────────────────────────────────
git init                    # new repo in current folder
git clone <url>             # copy remote repo locally
git clone <url> my-folder   # clone into specific folder

# ── Daily workflow ─────────────────────────────────────
git status                  # see what changed
git diff                    # see exact line changes
git diff --staged           # see what's staged

git add <file>              # stage a file
git add -p                  # stage hunks interactively
git restore <file>          # undo unstaged changes
git restore --staged <file> # unstage a file

git commit -m "message"
git commit --amend          # edit last commit (only if not pushed)

# ── Remote ─────────────────────────────────────────────
git push
git push -u origin feat/login   # push + set upstream
git pull                         # fetch + merge
git fetch                        # fetch only (no merge)

# ── Inspection ─────────────────────────────────────────
git log --oneline --graph --all  # visual branch history
git show <commit>                # details of a commit
git blame <file>                 # who changed each line
```

---

### 3. Branching — Parallel Lines of Development

**Theory**
Branches let each developer work on a feature independently without breaking the main codebase. The convention: `main` (production-ready), `develop` (integration), `feat/feature-name` (new feature), `fix/bug-name` (bug fix), `chore/...` (non-code work).

**Working Flow**

![flow-chart-3](flow-chart-3.png)

**Example**
```bash
# Create and switch to a new branch
git checkout -b feat/user-profile
# Modern syntax:
git switch -c feat/user-profile

# List all branches
git branch -a
# * feat/user-profile    ← current (starred)
#   main
#   remotes/origin/main

# Work on your feature, commit as usual
git add .
git commit -m "Add user profile page"
git commit -m "Add avatar upload"

# Push branch to remote
git push -u origin feat/user-profile

# Switch back to main
git switch main

# Merge the feature branch
git merge feat/user-profile

# Delete branch after merge
git branch -d feat/user-profile
git push origin --delete feat/user-profile
```

**Output**
```
git switch -c feat/user-profile  → Switched to new branch 'feat/user-profile'
git log --oneline:
  c3d4e5f Add avatar upload
  a1b2c3d Add user profile page
  e4f5g6h Last commit on main

git merge feat/user-profile  →  Updating e4f5g6h..c3d4e5f
                                 Fast-forward
                                 src/Profile.jsx  | 42 +++++++++++++++
```

---

### 4. Merge vs Rebase

**Theory**
Both integrate changes from one branch into another — they differ in how history looks:
- **Merge** — creates a merge commit, preserves the full branch history
- **Rebase** — replays your commits on top of the target, creates a linear history

**Working Flow**

![flow-chart-4](flow-chart-4.png)

**Example**
```bash
# Scenario: you're on feat/login, main has moved forward

# ── MERGE ──────────────────────────────────────────────
git switch feat/login
git merge main
# Creates: M── Merge branch 'main' into feat/login

# History looks like:
# * a1b (HEAD) Merge branch 'main' into feat/login
# |\
# | * c3d (main) Fix critical bug
# * | b2e Add login form
# |/
# * d4f Initial commit

# ── REBASE ─────────────────────────────────────────────
git switch feat/login
git rebase main
# Replays feat/login commits on top of main

# History looks like:
# * b2e' (HEAD feat/login) Add login form  ← replayed commit (new hash)
# * c3d  (main)            Fix critical bug
# * d4f                    Initial commit

# Golden rule: never rebase commits that have been pushed to a shared branch
```

**Output**
```
Merge:  non-linear history, shows branch happened, safe for shared branches
Rebase: clean linear history, easier to read git log, only for local/private branches
```

---

### 5. Resolving Merge Conflicts

**Theory**
A conflict happens when two branches change the same line differently. Git cannot decide which version to keep — it marks the conflict and asks you to resolve it manually.

**Working Flow**

![flow-chart-5](flow-chart-5.png)

**Example**
```bash
# Both main and feat/login edited src/config.js line 5

git merge feat/login
# CONFLICT (content): Merge conflict in src/config.js
# Automatic merge failed; fix conflicts and then commit the result.

# Open src/config.js — Git marks the conflict:
```

```javascript
// src/config.js after conflict
const config = {
<<<<<<< HEAD (main branch version)
  apiUrl: 'https://api.prod.com',
=======
  apiUrl: 'https://api.staging.com',
>>>>>>> feat/login (incoming branch version)
  timeout: 5000,
};
```

```bash
# Resolve — edit the file to the correct final version:
```

```javascript
// Resolved version:
const config = {
  apiUrl: process.env.NODE_ENV === 'production'
    ? 'https://api.prod.com'
    : 'https://api.staging.com',
  timeout: 5000,
};
```

```bash
# Mark as resolved and finish merge
git add src/config.js
git commit -m "Merge feat/login — resolve API URL conflict"
```

**Output**
```
git status           → both modified: src/config.js
git add config.js    → conflict marked as resolved
git commit           → [main f7g8h9i] Merge feat/login
```

---

### 6. Team Workflow — PR / MR Process

**Theory**
In a team, the standard workflow is: create branch → commit → push → open Pull Request (GitHub) or Merge Request (GitLab) → code review → CI passes → merge.

**Working Flow**

![flow-chart-6](flow-chart-6.png)

**Example**
```bash
# Day-in-the-life git workflow for a new feature

# 1. Start from updated main
git switch main
git pull origin main

# 2. Create feature branch
git switch -c feat/search-bar

# 3. Write code, commit in logical chunks
git add src/SearchBar.jsx src/SearchBar.test.jsx
git commit -m "feat: add search bar component with debounce"

git add src/App.jsx
git commit -m "feat: integrate search bar into header"

# 4. Keep branch up to date (optional daily sync)
git fetch origin
git rebase origin/main

# 5. Push and open PR
git push -u origin feat/search-bar
# GitHub CLI:
gh pr create --title "Add search bar" --body "Closes #42"

# 6. After code review + CI passes → merge on GitHub

# 7. Clean up locally
git switch main
git pull origin main
git branch -d feat/search-bar
```

**Output**
```
PR opened: feat/search-bar → main
CI checks: ✓ lint, ✓ tests, ✓ build
Code review: ✓ approved by 1 reviewer
Merged: "feat: add search bar component with debounce" → main
```

---

[View Interview Questions](./interview.md)
