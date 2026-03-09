# {UUID}-{NN}: {Title}
Estimated: {6-12} hours

## ⛔ Worktree Required
**Worktree:** `{PROJECT}-wt-{uuid}-{nn}` | **Branch:** `{type}/{uuid}-{nn}`

```bash
cd {PROJECT_ROOT}
git worktree add -b {type}/{uuid}-{nn} ../{PROJECT}-wt-{uuid}-{nn} main
cd ../{PROJECT}-wt-{uuid}-{nn}
{PKG_MGR} install
```

### Baseline Capture ⛔ MANDATORY
> [!CAUTION]
> **BEFORE any code changes**, capture baseline:
```bash
{TEST_CMD} 2>&1 | tee /tmp/baseline_tests_{uuid}-{nn}.txt
echo "Baseline failures: $(grep -c 'FAIL\|✗' /tmp/baseline_tests_{uuid}-{nn}.txt || echo 0)"
```

## Execution
- **Parallel group**: {A, B, C, or SEQUENTIAL}
- **Depends on**: {uuid}-{NN} (or NONE)

## Goal
What this plan accomplishes. One paragraph max.

## Context
Background needed: prior decisions, key paths, patterns to follow, constraints.

## Skeleton
From research report: {link to research report}

### Module: `{filename}` [NEW|MODIFY|DELETE]
**Purpose**: What this module does

- `functionName(param: Type, param2: Type): ReturnType` — purpose
  - Calls: `existingModule.existingFn()`
  - Errors: throws `ErrorType` when {condition}

### New Types
```
interface TypeName {
  field: Type  // purpose
}
```

### Data Flow
{Component A} →[transform]→ {Component B} →[validate]→ {Component C}

### Error Strategy
| Error | Source | Handling |
|-------|--------|----------|
| {ErrorType} | `fn()` | Catch and return default / rethrow / log+ignore |

## Tasks
All tasks required—no optional items.
1. Specific action with file path
2. ...

## Migration (if replacing existing code)

> [!CAUTION]
> **Migration = FULL REPLACEMENT.** No legacy adapters. No compatibility layers.
> Legacy code is ONLY allowed with explicit user approval.

### Legacy Inventory
| Item | Location | Type |
|------|----------|------|
| `oldFunction()` | `path/to/file.js` | Function |

### Purge Gate
After implementation, must return zero:
```bash
rg "oldFunction\(" {SRC_DIR} --type js && echo "⛔ LEGACY REMAINS" || echo "✓ Purged"
```

### Anti-Patterns to Avoid
- ❌ Deprecation comments (delete the code instead)
- ❌ Compatibility shims (`export const oldName = newName`)
- ❌ Feature flags branching old/new paths
- ❌ Wrapper functions calling legacy code

### Shim Detection
Verify NO shims exist before marking complete:
```bash
rg "wrapper|shim|compat|legacy" {SRC_DIR} --type js
rg "export.*from.*legacy|= old|= deprecated" {SRC_DIR} --type js
```

## Verification
- Tests to run, manual checks, expected outcomes
- **Pattern presence**: Grep for new patterns
- **Legacy absence** (if migration): Grep old patterns
- **File inspection**: View modified files

## Files Affected
| File | Change |
|------|--------|
| `path/to/file.js` | Add/Modify/Delete |

## ⛔ Implementation Contract
> [!CAUTION]
> Modify ALL listed files. Implement EXACT patterns. Verify with grep. No shims.

## After Implementation
1. `/verify` → 2. Archive: `mv .agent/plans/{uuid}-{nn}-*.md .agent/plans/archive/` → 3. `/finish`
