---
description: Complete work
mode: VERIFICATION
exceptions: [REQUIRE_VERIFY_FIRST, REBASE_FF_ONLY]
read_when:
  - Work is complete and verified
  - Ready to merge worktree to main
---
# Finish

End-of-task workflow. Commits and merges after `/verify`.

> [!IMPORTANT]
> **Run `/verify` first.** This assumes all pre-merge checks passed.

## 0. Worktree ⛔ GATE
`pwd` → `/{PROJECT}`? STOP (you're in main). `*-wt-stream-*`? Use `/finish-stream`. `*-wt-*`? ✓ Continue.

// turbo-all

## 1. Stage & Commit
```bash
git add .; git status
git commit -m "type: description"
```
Format: `type: concise description` — group related changes per commit.

Check: all new files staged, no untracked, no unstaged changes.
> [!CAUTION]
> **NEVER bypass pre-commit hooks with `--no-verify`.**

## 2. Rebase & Merge
```bash
git fetch {PROJECT_ROOT} main; git rebase main
branch=$(git branch --show-current); worktree_path=$(pwd)
# Fast-forward main without checkout (stays in worktree)
git fetch . $branch:main
```
If conflicts: resolve → `git add .` → `git rebase --continue` → re-run tests.
If "not a fast-forward": rebase didn't complete — go back and rebase again.
If "cannot lock ref": `git -C {PROJECT_ROOT} merge $branch --ff-only`

## 3. Verify & Cleanup
```bash
git log main --oneline -1; git diff main~1..main --name-only
pkill -f "$worktree_path.*dev" 2>/dev/null || true
git worktree remove $worktree_path; git branch -d $branch
```
