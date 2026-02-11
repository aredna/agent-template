---
description: Pre-finish verification gate
mode: VERIFICATION
exceptions: [REQUIRE_WORKTREE, GREP_PATTERN_CHECK]
read_when:
  - Development is complete, ready to verify
  - Before running /finish
---
# Verify

Pre-merge verification. Run before `/finish`.

**When to use:** Before `/finish`, after `/develop`, plan completion check.

## 0. Worktree ⛔ GATE
`pwd` → `/{PROJECT}`? STOP (you're in main). `*-wt-stream-*`? Use `/finish-stream`. `*-wt-*`? ✓ Continue.

## 1. Plan Completion (if implementing plan)
```bash
cat .agent/plans/{uuid}-{nn}-*.md
```
For each task: implemented (not skipped), pattern exists (grep), file modified (git diff).

```bash
grep -r "expected_pattern" path/to/file.js || echo "⛔ PATTERN MISSING"
git diff main --name-only
```

## 1.5 Change Review (Persona Sweep)
For each changed file, run ARCH, SEC, PERF, TEST, COMPAT, SCOPE, OPS personas
(see `AGENTS.md` → Review Personas).

## 2. New Files → Tests
```bash
git diff main --name-only --diff-filter=A | grep -E '\.(js|ts)$' | grep -v '\.test\.'
```
Each new source file needs a test.

## 3. AGENTS.md Sync ⛔ GATE
| Change | Update |
|--------|--------|
| New/deleted source file | File Index in `AGENTS.md` |
| Events/entities change | Semantics in `AGENTS.md` |

## 4. Result
✓ All pass → `/finish` | ✗ Missing patterns → Continue `/develop` | ✗ Bugs → Fix or escalate
