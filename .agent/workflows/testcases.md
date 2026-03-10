---
description: Test lifecycle management — audit, add, update, delete
mode: EXECUTION
exceptions: [REQUIRE_WORKTREE]
read_when:
  - Creating tests for new or changed code
  - Backfilling tests for untested code
  - Cleaning up stale or obsolete tests
  - After /develop, before /verify
---
# Test Cases

Full test lifecycle: audit → analyze → add → update → delete → validate.

**Autonomous — no user interaction needed.**

> [!IMPORTANT]
> **Err on the side of too many tests.** Every exported function, every branch, every edge case.

## 0. Worktree ⛔ GATE
`pwd` → `/{PROJECT}`? STOP (you're in main). `*-wt-stream-*`? ✓ | `*-wt-*`? ✓ Continue.

## 0.5 Scope Detection

Determine scope automatically — do NOT ask user.

| Context | Scope | How to Detect |
|---------|-------|---------------|
| Invoked from `/develop`, `/verify`, or with plan reference | **Changed files only** | `git diff main --name-only` |
| Invoked with explicit file/path arguments | **Specified files only** | Arguments provided |
| Invoked standalone with no context | **Full codebase** | No plan, no diff, no arguments |

```bash
# Auto-detect scope
SCOPE_FILES=$(git diff main --name-only 2>/dev/null | grep -E '{SOURCE_EXTENSIONS}' | grep -v '{TEST_GLOB}')
if [ -z "$SCOPE_FILES" ]; then
  # No diff from main — standalone mode, audit full codebase
  SCOPE_FILES=$(fd {SOURCE_EXT_FLAGS} {SRC_DIR} -g '!{TEST_GLOB}' -g '!*.config.*')
  echo "SCOPE: Full codebase"
else
  echo "SCOPE: Changed files only ($(echo "$SCOPE_FILES" | wc -l) files)"
fi
```

> [!CAUTION]
> When scoped to changed files, do NOT audit or modify tests outside that scope.

// turbo-all

## 1. Audit

Four-pass scan against scoped files:

```bash
# Pass 1: Coverage gaps — source files with no test
for f in $SCOPE_FILES; do
  base="${f%.*}"; ext="${f##*.}"
  [ ! -f "${base}{TEST_FILE_SUFFIX}" ] && echo "MISSING: $f"
done

# Pass 2: Stale tests — test files whose source was deleted
for t in $(fd {SOURCE_EXT_FLAGS} {SRC_DIR} -g '{TEST_GLOB}'); do
  src=$(echo "$t" | sed 's/{TEST_FILE_SUFFIX}/.{SOURCE_EXT}/'); [ ! -f "$src" ] && echo "STALE: $t"
done

# Pass 3: Thin tests — test files with few/no assertions
for f in $SCOPE_FILES; do
  base="${f%.*}"; t="${base}{TEST_FILE_SUFFIX}"
  [ -f "$t" ] || continue
  count=$(rg -c '{ASSERTION_PATTERN}' "$t" 2>/dev/null || echo 0)
  [ "$count" -lt 3 ] && echo "THIN($count): $t"
done

# Pass 4: Orphan mocks — mocks referencing deleted functions
rg -l '{MOCK_PATTERN}' {SRC_DIR} -g '{TEST_GLOB}' {SOURCE_TYPE_FLAG}
```

**Output:** Categorized list: `MISSING`, `STALE`, `THIN`, `OK`.

## 2. Logical Analysis — Find Non-Obvious Cases

For each scoped source file, read the implementation and reason about:

| Analysis | What to Look For |
|----------|-----------------|
| **Implicit contracts** | Assumptions the code makes but doesn't validate (param ranges, type expectations, required fields) |
| **State coupling** | Functions that depend on external state (module-level vars, singletons, caches) — test state leaks |
| **Order dependence** | Logic that assumes calls happen in a specific order — test reordering |
| **Concurrency hazards** | Async/parallel operations — test race conditions |
| **Silent failures** | Code that swallows errors or returns defaults instead of signaling failure — test both paths |
| **Cascading effects** | One function's output feeds another's input — test with corrupted intermediate values |
| **Numeric edge cases** | Math operations — test boundary values, overflow, division by zero |
| **String edge cases** | String operations — test unicode, empty, very long strings, injection patterns |
| **Timing assumptions** | Code using delays, debounce, or wall-clock time — test with mocked time |
| **Configuration sensitivity** | Behavior that changes based on config/env — test with missing, empty, and invalid config |

> [!IMPORTANT]
> Don't just test what the function *does*. Test what happens when everything around it goes wrong.

## 3. Add — New Tests

Structure every test with **AAA**: Arrange → Act → Assert.

### Required Test Categories Per Function

| Category | What to Test | Min |
|----------|-------------|-----|
| **Happy path** | Expected input → expected output | 1 |
| **Invalid input** | Wrong types, missing params, nulls, extra fields → graceful failure | 2+ |
| **Boundary values** | Empty, zero, max length, off-by-one | 2+ |
| **Error paths** | Fails on invalid conditions, error messages are useful | 1+ |
| **Edge cases** | Concurrent, rapid succession, timeout, reentrant calls | 1+ |
| **State transitions** | Before/after, init/destroy, repeated calls | 1+ |
| **Return shapes** | Every distinct return value/structure | 1 |
| **Side effects** | Events emitted, state mutated, external calls made | 1+ |
| **Integration mocks** | Verify mock called with correct args, handles mock failure | 1+ |
| **Non-obvious** (from step 2) | Cases discovered by logical analysis | varies |

### Invalid Input Testing ⛔ MANDATORY

Every function that accepts parameters MUST have invalid input tests:

- Null/missing value handling
- Wrong type handling
- Empty value handling
- Missing required params (multi-param functions)
- Extra unexpected params (multi-param functions)

> [!CAUTION]
> "Gracefully" means: throws a clear error, returns a defined fallback, or rejects with a useful message.
> Crashing, hanging, or returning corrupt data is NOT graceful.

### Conventions
- **Naming**: Group by module → function → `should {behavior} when {condition}`
- **File**: Co-located with source using `{TEST_FILE_SUFFIX}`
- **Independence**: No shared mutable state; use setup/teardown hooks
- **No logic in tests**: Tests are declarative — arrange data, call function, check result

## 4. Update — Revise Existing Tests

Trigger map for changed source files:

| Source Change | Test Action |
|--------------|-------------|
| New function | Add test group with full category coverage |
| Signature changed | Update all calling tests + add invalid input tests for new params |
| Behavior changed | Update assertions, add new edge cases |
| Function renamed | Rename in tests, grep for stale references |
| Function deleted | Remove corresponding test group |
| New error/branch | Add error-path / branch-specific test |
| Event added/removed | Add/remove emission test |
| New validation added | Add tests proving invalid input is rejected |

```bash
# Find changed source files needing test updates
git diff main --name-only --diff-filter=M | grep -E '{SOURCE_EXTENSIONS}' | grep -v '{TEST_GLOB}'
```

## 5. Delete — Remove Obsolete Tests

| Condition | Action |
|-----------|--------|
| Source file deleted | Delete entire test file |
| Function removed | Delete its test group |
| Duplicate test | Keep the more thorough one |
| Tests implementation details | Rewrite as behavior test or delete |
| Permanently flaky, no fix | Delete and document why |

```bash
# Verify function is truly gone before deleting its tests
rg "functionName" {SRC_DIR} {SOURCE_TYPE_FLAG} -g '!{TEST_GLOB}'
# Zero results → safe to delete
```

> [!CAUTION]
> Log every deletion with rationale. Never silently remove tests.

## 6. Validate ⛔ GATE

```bash
{TEST_CMD}   # Zero failures
{LINT_CMD}   # Zero errors

# No untested source files (within scope)
for f in $SCOPE_FILES; do
  test_file="${f%.*}{TEST_FILE_SUFFIX}"
  [ ! -f "$test_file" ] && echo "⛔ UNTESTED: $f"
done

# No orphaned test files
for t in $(fd {SOURCE_EXT_FLAGS} {SRC_DIR} -g '{TEST_GLOB}'); do
  src=$(echo "$t" | sed 's/{TEST_FILE_SUFFIX}/.{SOURCE_EXT}/')
  [ ! -f "$src" ] && echo "⛔ ORPHANED: $t"
done

# No assertion-free tests
for t in $(fd {SOURCE_EXT_FLAGS} {SRC_DIR} -g '{TEST_GLOB}'); do
  count=$(rg -c '{ASSERTION_PATTERN}' "$t" || echo 0)
  [ "$count" -eq 0 ] && echo "⛔ NO ASSERTIONS: $t"
done

# No missing invalid-input tests
for f in $SCOPE_FILES; do
  test_file="${f%.*}{TEST_FILE_SUFFIX}"
  [ ! -f "$test_file" ] && continue
  count=$(rg -c 'null|undefined|nil|None|invalid|missing|empty' "$test_file" 2>/dev/null || echo 0)
  [ "$count" -eq 0 ] && echo "⚠ NO INVALID INPUT TESTS: $test_file"
done
```

## 7. Report

```markdown
# Test Maintenance Report
## Scope
{Full codebase | Changed files (N files)}

| Metric | Count |
|--------|-------|
| Source files (scoped) | N |
| Test files | N |
| File coverage | N% |
| Added | N |
| Updated | N |
| Deleted | N |
| Non-obvious cases found | N |

## Changes
### Added
- `path{TEST_FILE_SUFFIX}` — fn1, fn2, fn3

### Updated
- `path{TEST_FILE_SUFFIX}` — edge cases for fn1, invalid input for fn2

### Deleted
- `path{TEST_FILE_SUFFIX}` — source removed (reason)

### Non-Obvious Cases
- `path{TEST_FILE_SUFFIX}` — race condition in async handler, silent null coercion

## Remaining Gaps
- `path` — needs integration test
```

## After `/testcases`
Complete → `/verify` then `/finish` | Gaps → Document in report
