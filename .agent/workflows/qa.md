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

## Investigation Patterns
Search for these categories using language-appropriate patterns:

| Category | What to Search For |
|----------|--------------------|
| Async gaps | Async functions missing proper awaiting/error propagation |
| Event leaks | Event listeners or subscriptions never cleaned up |
| Timer leaks | Scheduled/recurring tasks never cancelled |
| State mutations | Unprotected shared state changes |
| Silent failures | Caught errors that are swallowed without logging or re-throwing |
| Resource leaks | Open handles (files, connections, streams) without cleanup |
| Race conditions | Concurrent access to shared resources without synchronization |
| Null/empty paths | Missing null checks before dereference |

```bash
rg "{ASYNC_GAP_PATTERN}" {SRC_DIR} {SOURCE_TYPE_FLAG}       # Async gaps
rg "{EVENT_SUBSCRIBE_PATTERN}" {SRC_DIR} {SOURCE_TYPE_FLAG}  # Event leaks
rg "{TIMER_PATTERN}" {SRC_DIR} {SOURCE_TYPE_FLAG}            # Timer leaks
rg "{ERROR_CATCH_PATTERN}" {SRC_DIR} {SOURCE_TYPE_FLAG}      # Silent failures
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
