---
description: Development workflow
mode: EXECUTION
exceptions: [REQUIRE_WORKTREE, PLAN_ADHERENCE_GREP]
read_when:
  - Implementing features, fixes, or refactors
  - User requests code changes
  - Following an approved plan
---
# Develop

Implement features, fixes, refactors, and tests.

> [!WARNING]
> **Code in `main` is at risk.** Always use a worktree.

## 0. Worktree ⛔ GATE
`pwd` → `/{PROJECT}`? STOP, create worktree:
```bash
cd {PROJECT_ROOT}; TIMESTAMP=$(date +%y%m%d%H%M%S); BRANCH="fix/your-branch"
git worktree add -b $BRANCH ../{PROJECT}-wt-$TIMESTAMP main; cd ../{PROJECT}-wt-$TIMESTAMP
```
`*-wt-stream-*`? ✓ (if yours or user's stream) | `*-wt-*`? ✓ Continue

**Branch types:** `feature/`, `fix/`, `refactor/`, `test/`

## 0.5 Environment ⛔ GATE
```bash
[ -f package.json ] && [ ! -d node_modules ] && {PKG_MGR} install
{BUILD_CMD} || { echo "⛔ BUILD BROKEN before any changes"; exit 1; }
```

## Usage
- `/develop` (no args) → Ask user for instructions
- `/develop {uuid}-{nn}` → Implement plan from `.agent/plans/`
- `/develop {instructions}` → Follow user instructions

// turbo-all

## 1. Load Context
Plan: `cat .agent/plans/{uuid}-{nn}-*.md` — follow Tasks exactly.
On-demand: `AGENTS.md` (File Index, Semantics, Errors).

## 2. Plan Adherence ⛔ GATE

> [!CAUTION]
> **NO SHORTCUTS.** Tasks immutable. All files must be modified. All patterns must match. Tests passing ≠ complete.

Before marking task complete:
```bash
grep -l "pattern_from_plan" path/to/file.js || echo "⛔ PATTERN NOT FOUND"
```

## 3. Implement
Create/modify files per plan or instructions.

## 3.5 Self-Check (Persona Sweep)
Before proceeding to test gate, run MAINT, TEST, UX, SCOPE, DOCS, OPS personas
(see `AGENTS.md` → Review Personas).

## 4. Test ⛔ GATE
```bash
{TEST_CMD}  # Zero failures required
{LINT_CMD}  # Zero errors/warnings
```

**Pre-existing failure?** Prove with `git diff > /tmp/my_changes.patch` → `git checkout .` → test → `git apply /tmp/my_changes.patch`. Document in commit.

## 5. Finish
→ `/verify` then `/finish` (or notify user if stream worktree)
