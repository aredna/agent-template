---
description: Merge stream worktree to main (user-only)
mode: VERIFICATION
exceptions: [USER_ONLY, REBASE_FF_ONLY]
read_when:
  - User wants to merge a stream worktree
  - Stream session is complete
---
# Finish Stream

> [!CAUTION]
> **USER-ONLY WORKFLOW.** Agents must NEVER run this autonomously.

Merges stream worktree to main. Verify: research complete, changes approved, tests pass.

## 1. Verify Stream Worktree
```bash
pwd | grep -q "stream" && echo "✓ Stream confirmed" || echo "⛔ Not stream worktree"
```
Not stream? → Use `/finish` instead.

## 2. Test, Stage, Commit
```bash
cd {SRC_DIR} && {TEST_CMD}
git add .; git commit -m "stream: description"
```

## 3. Rebase & Merge
```bash
git fetch {PROJECT_ROOT} main; git rebase main
branch=$(git branch --show-current)
git -C {PROJECT_ROOT} merge $branch --ff-only
git -C {PROJECT_ROOT} log --oneline -3
```

## 4. Cleanup (Optional)
```bash
worktree_path=$(pwd); pkill -f "$worktree_path.*dev" 2>/dev/null || true
cd {PROJECT_ROOT}; git worktree remove $worktree_path; git branch -d $branch
```

## ⚠️ Agent Restrictions
DO NOT execute autonomously. DO NOT suggest. If merge needed, tell user: "Use `/finish-stream` when ready."
