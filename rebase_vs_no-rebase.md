# Git: Handling Divergent Branches - Merge vs Rebase

## Scenario: Multiple People Working on the Same Branch

This happens when:
- You commit locally to a branch
- Someone else pushes to the same branch before you
- Your local branch and the remote branch have diverged

---

## The Problem

```
Your Local:     A → B → C (your commits)
Remote Branch:  A → D → E (someone else's commits)
```

Both branches started from the same point (A) but have different commits. When you try to push, Git rejects it because the branches have diverged.

---

## Solution 1: `--no-rebase` (Merge Strategy)

### Concept
- **Parallel branches**: Treats local and remote as separate branches
- Preserves both histories
- Creates a merge commit to combine them

### Visual Example

**Before:**
```
A (common ancestor)
├─ B (your local commit)
│  └─ C (your local commit)
│
└─ D (remote commit)
   └─ E (remote commit)
```

**After `git pull --no-rebase`:**
```
A (common ancestor)
├─ B (your local commit)
│  └─ C (your local commit)
│      └─ M (merge commit)
│         /
└─ D (remote commit)
   └─ E (remote commit)
```

### Commands
```bash
git pull origin <branch-name> --no-rebase
git push origin <branch-name>
```

### Result
- ✅ Your commit IDs: **B** and **C** (unchanged)
- ✅ Remote commit IDs: **D** and **E** (unchanged)
- ✅ New merge commit: **M** (combines both branches)
- ✅ History shows both branches merging

### When to Use
- ✅ Team prefers merge history
- ✅ Working on shared/main branches
- ✅ You want to preserve exact commit history
- ✅ You've already pushed your commits

---

## Solution 2: `--rebase` (Rebase Strategy)

### Concept
- **Linear history**: Remote changes first, then local changes on top
- No merge commit
- Your commits get new IDs (replayed)

### Visual Example

**Before:**
```
A (common ancestor)
├─ B (your local commit)
│  └─ C (your local commit)
│
└─ D (remote commit)
   └─ E (remote commit)
```

**After `git pull --rebase`:**
```
A → D → E → B' → C'
     ↑        ↑
  Remote   Local (replayed on top)
```

### Commands
```bash
git pull origin <branch-name> --rebase
git push origin <branch-name>
```

### Result
- ✅ Remote commit IDs: **D** and **E** (unchanged)
- ✅ Your commit IDs: **B'** and **C'** (new IDs - replayed)
- ✅ No merge commit
- ✅ Clean linear history

### When to Use
- ✅ Prefer clean linear history
- ✅ Your commits haven't been shared yet
- ✅ Working on feature branches
- ✅ You want a simpler timeline

---

## Key Differences Summary

| Aspect | `--no-rebase` (Merge) | `--rebase` |
|--------|----------------------|------------|
| **Structure** | Parallel branches | Linear single line |
| **Your commit IDs** | Keep original (B, C) | Get new IDs (B', C') |
| **Remote commit IDs** | Keep original (D, E) | Keep original (D, E) |
| **Merge commit** | Creates one (M) | No merge commit |
| **History** | Shows both paths | Single timeline |
| **Data loss** | ❌ No | ❌ No |
| **Final files** | ✅ Same content | ✅ Same content |

---

## Important Notes

### Both strategies:
- ✅ Preserve all changes (no data loss)
- ✅ Result in identical final files
- ✅ Only differ in commit history structure

### Conflict resolution:
- If both you and remote modify the same lines, you'll need to resolve conflicts in both cases
- Git will pause and ask you to choose/combine the changes

---

## Common Workflow

### When you encounter divergent branches:

1. **Check what's different:**
   ```bash
   git log --oneline HEAD..origin/<branch>  # See remote commits
   git log --oneline origin/<branch>..HEAD  # See your commits
   ```

2. **Choose your strategy:**
   - **Merge:** `git pull origin <branch> --no-rebase`
   - **Rebase:** `git pull origin <branch> --rebase`

3. **Resolve conflicts** (if any)

4. **Push:**
   ```bash
   git push origin <branch>
   ```

---

## Best Practices

- ✅ Use `--no-rebase` (merge) for shared/main branches
- ✅ Use `--rebase` for feature branches before merging
- ✅ Always pull before pushing to avoid divergence
- ✅ Communicate with your team about which strategy to use

---

## Quick Reference

### Merge (Recommended for shared branches)
```bash
git pull origin <branch> --no-rebase
git push origin <branch>
```

### Rebase (For cleaner history)
```bash
git pull origin <branch> --rebase
git push origin <branch>
```

### Configure default behavior
```bash
# Set merge as default
git config pull.rebase false

# Set rebase as default
git config pull.rebase true
```

---

## Summary

- **`--no-rebase`** = Parallel branches (local + remote) → Merge commit
- **`--rebase`** = Linear (remote first, then local on top) → No merge commit

**Both preserve all changes - only history structure differs!**
