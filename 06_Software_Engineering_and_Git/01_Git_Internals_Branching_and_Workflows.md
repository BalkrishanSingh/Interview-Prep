# Module 06: Software Engineering — Git Internals, Branching & Workflows

---

## 1. Git Object Model Architecture

Git is a content-addressable storage filesystem built on top of a **Directed Acyclic Graph (DAG)** of immutable objects referenced by 40-character SHA-1 hashes.

```
Commit Object (SHA: a1b2c3d)
├── Tree Pointer (SHA: e4f5g6h)
│   ├── Blob: "main.py" (File contents)
│   └── Tree: "utils/"  (Subdirectory)
│       └── Blob: "helpers.py"
├── Parent Commit Pointer (SHA: 9z8y7x)
├── Author / Committer Timestamp
└── Commit Message
```

### The 4 Foundational Git Object Types:
1. **Blob**: Stores raw binary file contents (does NOT store file name, permissions, or timestamps).
2. **Tree**: Represents a directory. Maps file names, modes, and permissions to corresponding Blob and sub-Tree SHA-1 hashes.
3. **Commit**: Points to a root Tree object, parent commit SHA(s), author metadata, and commit message.
4. **Annotated Tag**: A permanent pointer to a specific commit containing tagger metadata and a cryptographic signature.

---

## 2. Git Merge vs. Git Rebase

```
INITIAL STATE:
      D---E (feature)
     /
A---B---C (main)

GIT MERGE (git checkout main && git merge feature):
      D---E
     /     \
A---B---C---M (main with 3-way merge commit M)
• Preserves historical truth and context
• Non-linear, branch-cluttered history graph

GIT REBASE (git checkout feature && git rebase main):
A---B---C---D'---E' (feature replayed on top of main)
• Clean, linear commit history
• Rewrites commit SHAs (D -> D', E -> E')
```

### Detailed Comparison Table
| Dimension | `git merge` | `git rebase` |
| :--- | :--- | :--- |
| **Commit History** | Non-linear; preserves all branch divergence and convergence points. | Strictly **linear**; eliminates visual branch clutter. |
| **Commit Hashes** | Preserves existing commit SHAs; creates a new merge commit. | **Rewrites commit SHAs** by re-applying patches sequentially. |
| **Golden Rule** | Safe to use anywhere. | **Never rebase commits that have been pushed to a shared public branch** (breaks teammates' histories). |
| **Conflict Resolution**| Resolve conflicts once during the creation of the merge commit. | Must resolve conflicts commit-by-commit as each patch is replayed. |

---

## 3. Essential Git Operations

### 3.1 `git reset` vs. `git revert`
- **`git revert <commit>`**: Creates a **new inverse commit** that undoes the changes introduced by the target commit. Does NOT rewrite history. Safe for shared remote branches.
- **`git reset <commit>`**: Moves the `HEAD` pointer backward in history (rewriting local branch history):
  - `--soft`: Moves `HEAD`. Leaves changes staged in the Index.
  - `--mixed` (Default): Moves `HEAD` and unstages changes into the Working Directory.
  - `--hard`: Moves `HEAD` and **permanently discards** all working directory modifications.

### 3.2 `git cherry-pick <commit-hash>`
Copies the patch introduced by a specific commit from another branch and applies it onto the current branch as a new commit with a new hash.

### 3.3 `git stash`
Temporarily shelves (stashes) uncommitted modifications in the working directory and index so you can switch branches without committing unfinished work:
```bash
git stash save "WIP: payment integration"
git checkout main
# ... complete urgent fix ...
git checkout feature
git stash pop
```

---

## 4. Branching Strategies: Git Flow vs. Trunk-Based

```
GIT FLOW (Heavyweight / Release Cycles):
main     ───────────────────────────────────────● (Production Release v1.0)
develop  ───────●─────────●─────────●───────────
feature         └───●─●───┘ (Merged via PR)

TRUNK-BASED DEVELOPMENT (Agile / Continuous Deployment):
main (Trunk) ───●───────●───────●───────●───────● (Direct short-lived merges)
                └──●─●──┘ (Merged within 24 hours, guarded by Feature Flags)
```

| Dimension | Git Flow | Trunk-Based Development |
| :--- | :--- | :--- |
| **Branch Lifespan** | Weeks to months (long-lived feature/release branches). | Hours to 1-2 days (short-lived, micro-branches). |
| **Merge Overhead** | High ("Merge Hell" when integrating long-diverged branches). | Minimal (frequent, incremental micro-merges). |
| **Deployment Cadence**| Scheduled batched release cycles (e.g., bi-weekly). | Continuous Deployment (multiple production releases per day). |
| **Risk Management** | Branch isolation. | **Feature Flags (Toggles)** to decouple deployment from release. |
