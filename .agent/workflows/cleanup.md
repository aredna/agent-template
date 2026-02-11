---
description: Cleanup orphaned processes
mode: EXECUTION
exceptions: [MAINTENANCE_ONLY, OWNERSHIP_RULES]
read_when:
  - Orphaned dev servers or processes detected
  - Stale worktrees need removal
  - System resource issues
---
# Cleanup

Identify and safely remove orphaned processes and worktrees.

## ⛔ Safety
- Only kill YOUR OWN processes/worktrees
- NEVER: `git worktree prune`, `pkill -9 node` (kills ALL), `rm -rf ../*-wt-*`, remove unmerged branches

## 1. Find Orphaned Processes
```bash
pgrep -af "vite|dev|serve" | grep -v grep
pgrep -af "node|bun" | grep -v grep
lsof -i :{DEFAULT_PORT} 2>/dev/null; lsof -i :5173 -i :8080 2>/dev/null  # Common dev ports
```

## 2. Kill YOUR Processes
```bash
current_wt=$(pwd)
pkill -f "$current_wt.*dev" 2>/dev/null || true
pkill -f "$current_wt.*vite" 2>/dev/null || true
```

## 3. Worktree Cleanup
```bash
git worktree list
branch="feature/x"; git log main..$branch --oneline  # Empty = merged, safe to remove
cd {PROJECT_ROOT}; git worktree remove ../{PROJECT}-wt-240205-143022; git branch -d $branch
```

## 4. System Check
```bash
free -h; ps aux --sort=-%mem | head -10; df -h
```

## Notify User If
- Many orphaned worktrees (>3) you didn't create
- High CPU/memory processes
- Disk space issues
