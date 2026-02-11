---
description: Deep bug hunting
mode: PLANNING
exceptions: [NO_PLAN_ARTIFACT, READ_ONLY]
read_when:
  - Investigating bugs or suspicious behavior
  - Pre-release audit
  - User reports unexpected behavior
---
# QA

Deep exploratory bug hunting. **Read-only—creates report artifact only.**

## Investigation Depth
- Read source of relevant dependencies AND all related local code before concluding
- High-confidence root cause — verify hypotheses with `rg`, not assumptions

## Investigation Tools
```bash
rg -l 'async\s+\w+\(' {SRC_DIR} --type js | xargs rg -L 'await'  # Async without await
rg "addEventListener\(" {SRC_DIR} --type js           # Event listeners
rg "setTimeout|setInterval" {SRC_DIR} --type js       # Timers
rg "this\.\w+\s*=" {SRC_DIR} --type js | head -20     # State mutations
rg "catch\s*\(" {SRC_DIR} --type js                   # Error handling
rg "\.catch\(" {SRC_DIR} --type js
```

## Report Format
```markdown
# QA Report: {Area}
## Scope
What was examined.

## Bugs Found
### Bug 1: {Title}
**Severity**: Critical|High|Medium|Low | **Category**: Logic|Race|State|Memory|Edge
**Location**: `path:L123` | **Trigger**: Repro steps | **Impact**: What breaks
**Evidence**: Code | **Fix**: Solution

## Summary
Critical: N | High: N | Medium: N | Low: N
```

## After QA
Bugs found → `/develop` | Architecture issues → `/plan` | Clean → Document
