---
description: Create implementation plans from conversation
mode: PLANNING
exceptions: [CUSTOM_PLAN_PATH, MULTI_FILE_PLANS]
read_when:
  - Creating implementation plans
  - Breaking down complex work into plan files
  - User asks to plan before implementing
---
# Plan

Create self-contained implementation plans from research output. **Autonomous — no user interaction needed.**

**Target time: 36–48 hours of work** unless the user specifies a different scope during invocation.

> [!IMPORTANT]
> Plans go to `.agent/plans/`, NOT `implementation_plan.md`.

> [!CAUTION]
> `/plan` does not make design decisions. If no research report with an
> approved skeleton exists, STOP and tell user: "Run `/research` first."

## 0. Worktree ⛔ GATE
`pwd` → `/{PROJECT}`? STOP, create worktree first. `*-wt-*` or `*-wt-stream-*`? ✓ Continue.

## 1. Migration Detection

> [!CAUTION]
> **Migration = FULL REPLACEMENT.** No legacy adapters. No compatibility layers.
> Legacy code is ONLY allowed with explicit user approval.

**This is a migration if ANY apply:**
- Replacing one API/property/method with another
- Unifying or consolidating two patterns into one
- Removing a feature flag or compatibility layer
- Introducing a new abstraction that replaces existing code
- Renaming identifiers across multiple files
- Deprecating or removing functionality

**If unclear, ASK USER:** "Should I treat this as a migration requiring complete legacy removal, or is backward compatibility acceptable?"

## 2. Load Research Output
Read the research report — skeleton, decisions, migration requirements.

## 3. Identify Work Units
Break skeleton into plan chunks by: component, layer (data→logic→UI),
dependency order, risk level. This is mechanical decomposition — all
design decisions were made in `/research`.

Identify parallelism: which plans can run together vs. sequentially.

### SCOPE Check
- Does the work decomposition match the research skeleton exactly?
- Nothing added that wasn't in the approved skeleton?
- Nothing removed or simplified from the approved skeleton?

## 4. Plan Naming

**Format:** `{WORD}-{NN}-{name}.md`
- WORD: a random 4–6 character common English word, all lowercase (e.g. `mint`, `spark`, `bloom`, `crane`, `flint`, `grove`). Pick one that doesn't collide with existing plan prefixes.
- NN: 2-digit sequence (00=execution plan, 01-99=implementation)
- Migration prefix: `migrate_`

```bash
# Get next sequence
ls .agent/plans/ 2>/dev/null | grep -oE "^$WORD-[0-9]{2}" | grep -oE '[0-9]{2}' | sort -n | tail -1
```

**Related plans:** One WORD, sequential NN values, create 00 execution plan last.

## 5. Create Plan

```bash
cp .agent/plans/_template.md .agent/plans/{word}-{nn}-{name}.md
```

Fill all sections. Copy skeleton from research output. For migrations, complete Legacy Inventory and Purge Gate.

## 6. Execution Plan (NN=00)

For multi-plan work:
```markdown
# Execution Plan: {WORD}-00-{group_name}

| Order | Plan | Dependencies |
|-------|------|--------------|
| 1 | {WORD}-01 | None |
| 2 | {WORD}-02 | 01 |

After all: verify legacy deleted (if migration), tests pass, lint passes.
```

## 7. Complete

> [!CAUTION]
> **STOP after plan creation.** Do NOT implement code.
> Tell user: "Plans ready. Use `/develop {word}-{nn}` to implement."
