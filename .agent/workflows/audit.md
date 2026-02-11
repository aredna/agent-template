---
description: Codebase health check
mode: PLANNING
exceptions: [NO_PLAN_ARTIFACT, READ_ONLY]
read_when:
  - Assessing overall project health
  - Checking AI docs accuracy
  - Pre-release quality review
---
# Audit

Comprehensive codebase health check. **Read-only—creates report artifact only.**

## 1. AI Documentation Audit

### Index Accuracy
```bash
# Check File Index in AGENTS.md (inline JSON)
grep -oE '"[^"]+\.(js|ts)"' AGENTS.md | tr -d '"' | while read f; do
  [ ! -f "$f" ] && echo "STALE: $f"
done
fd -e js -e ts {SRC_DIR} --type f | while read f; do
  grep -q "$(basename "$f")" AGENTS.md || echo "UNINDEXED: $f"
done
```

### Semantics Accuracy
```bash
# Check events in AGENTS.md Semantics JSON
rg "emit\(['\"](\w+)['\"]" {SRC_DIR} -o --type js | grep -oE "'[^']+'" | sort -u | while read evt; do
  grep -q "$evt" AGENTS.md || echo "UNDOCUMENTED EVENT: $evt"
done
```

### Architecture Accuracy
Review `AGENTS.md` Architecture section against actual codebase.

## 2. Dependency Audit
```bash
{PKG_MGR} x depcheck              # Unused
npx madge --circular {SRC_DIR}    # Circular
{PKG_MGR} outdated                # Outdated
```

## 3. Code Quality Audit
```bash
rg '\b[0-9]+\b' {SRC_DIR} --type js -g '!*.test.js' | grep -v 'CONFIG\|const\|//\|0\|1\|2'  # Magic numbers
rg 'console\.(log|warn|error)' {SRC_DIR} --type js -g '!*.test.js'  # Console
rg 'TODO|FIXME|HACK|XXX' {SRC_DIR} --type js  # TODOs
rg 'function (\w+)' {SRC_DIR} -o --type js | sort | uniq -c | sort -n  # Dead code candidates
```

## 4. Test Audit
```bash
for f in $(fd -e js {SRC_DIR} -g '!*.test.js'); do
  test_file="${f%.js}.test.js"; [ ! -f "$test_file" ] && echo "NO TEST: $f"
done
```
Look for: tests without assertions, happy-path only, hardcoded timing.

## 5. Structural Review (Persona Sweep)

Walk through each source file with all personas (see `AGENTS.md` → Review Personas).

## 6. Report Format
```markdown
# Audit Report
| Category | Issues |
|----------|--------|
| AI Docs | N |
| Dependencies | N |
| Code Quality | N |
| Tests | N |

## Findings
### {Category}: {Title}
**Severity**: Critical|High|Medium|Low | **Location**: `path` | **Fix**: Action
```

## After Audit
Issues found → `/develop` | Architecture issues → `/plan` | Clean → Document
