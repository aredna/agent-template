# AGENTS.md
> {PROJECT} Agent Instructions

## ⛔ Default PLANNING. Switch only when user directs or workflow specifies.

## Engineering Rules
- **Audits**: Present findings and discuss before creating a plan
- **Verbosity**: Concise when tests pass — report failures and changes only
- **DRY**: Never duplicate. Extract shared behavior. Exception? Ask with specifics.
- **File size**: Keep under ~500 LOC; split when it improves clarity
- **Lint churn**: Auto-resolve formatting-only diffs. Only ask about logic/data/behavior.
- **Always `gio trash` over `rm`**
- **Migration**: Fully replace. No adapters, no legacy. Blocked? Ask with details.
- **Do the hard work**: Best long-term solution, even when harder upfront. No shortcuts or tech debt.

## Comment Discipline
- **No narration**: Don't restate what code obviously does
- **Why, not what**: Intent, constraints, non-obvious behavior
- **Max 1 line** unless public API or complex algorithm
- **No banners, no commented-out code**
- **TODO format**: `{COMMENT} TODO(username): description` — no bare TODOs
- **Doc comments**: Public APIs only; document parameters + return values

## Review Personas
| ID | Focus | Checks |
|----|-------|--------|
| ARCH | Architecture | Decomposition, coupling, layer violations |
| SEC | Security | Input validation, auth boundaries, secrets |
| PERF | Performance | Hot paths, resource leaks, timers/listeners |
| MAINT | Maintainability | Naming, complexity (>50 LOC, >3 nesting), duplication, hardcoded values |
| TEST | Testability | Logic path coverage, edge cases, assertions |
| COMPAT | Compatibility | Breaking changes, removed exports, version bumps |
| SCOPE | Scope | Dead code, over/under-engineering, only what was asked |
| FUTURE | Futureproofing | Extension points for known upcoming features |
| DATA | Data integrity | Validation, consistent state, safe migrations |
| UX | User experience | Error messages, discoverability, API ergonomics |
| OPS | Operations | Logging, config externalization, graceful failure |
| DOCS | Documentation | Stale comments, commented-out code, public API docs |

## Git Safety

### ⛔ No Direct Main Commits
- Pre-commit hook blocks main commits
- All changes via worktree branches only
- All merges via `/finish` only
- NEVER use `--no-verify`

### ⛔ Merge Gate
`/verify` must pass before `/finish`

### ⛔ Multi-Agent Safety
- NEVER create/apply/drop `git stash` unless explicitly asked
- NEVER switch branches unless explicitly asked
- "push" → `git pull --rebase` first
- "commit" → scope to YOUR changes only
- Unrecognized files from other agents → ignore

---

## Project Overview
> **Purpose**: {DESCRIPTION}
> **Phase**: {PHASE}
> **Audience**: {TARGET_AUDIENCE}

## Tech Stack
| Tool | Value |
|------|-------|
| Language | {LANGUAGE} |
| UI Framework | {UI_FRAMEWORK} |
| CSS | {CSS_APPROACH} |
| Package Manager | {PKG_MGR} |
| Build | {BUILD_CMD} |
| Dev Server | {DEV_CMD} |
| Dev Port | {DEFAULT_PORT} |
| Formatter | {FORMATTER} |
| Lint | {LINT_CMD} |
| Test | {TEST_CMD} |
| Database | {DATABASE} |
| API | {API_STYLE} |
| Monorepo | {MONOREPO_TOOL} |
| CI | {CI_PLATFORM} |

<!-- Setup: delete rows that don't apply -->

### Language Tokens
<!-- Setup: fill these for your language/framework -->
| Token | Purpose | Value |
|-------|---------|-------|
| `{SOURCE_EXT}` | Primary source extension | |
| `{SOURCE_EXTENSIONS}` | Grep pattern for source files | |
| `{SOURCE_EXT_FLAGS}` | `fd` extension flags | |
| `{SOURCE_TYPE_FLAG}` | `rg` type filter | |
| `{TEST_FILE_SUFFIX}` | Test file naming convention | |
| `{TEST_GLOB}` | Glob for test files | |
| `{ASSERTION_PATTERN}` | Grep for test assertions | |
| `{MOCK_PATTERN}` | Grep for test mocks | |
| `{RUNTIME}` | Runtime process name | |
| `{COMMENT}` | Single-line comment prefix | |
| `{DEPS_INSTALLED_CHECK}` | Check if deps are installed | |
| `{DEP_CHECK_CMD}` | Find unused dependencies | |
| `{CIRCULAR_CHECK_CMD}` | Find circular imports | |
| `{LOG_PATTERN}` | Debug logging pattern | |
| `{EVENT_EMIT_PATTERN}` | Event emission pattern | |
| `{ASYNC_GAP_PATTERN}` | Async without proper await | |
| `{EVENT_SUBSCRIBE_PATTERN}` | Event listener registration | |
| `{TIMER_PATTERN}` | Scheduled/recurring tasks | |
| `{ERROR_CATCH_PATTERN}` | Error catch blocks | |
| `{SHIM_ALIAS_PATTERN}` | Re-export or aliasing | |
| `{DEV_SERVER_START}` | Dev server start command | |

## Architecture

### Layers
TBD — Setup: analyze imports and populate (e.g., `core → features → ui`)

### Constraints
TBD — Setup: document observed dependency rules (e.g., core has no UI imports)

### Key Directories
- `.agent/`: **SOURCE OF TRUTH** for AI context
- `{SRC_DIR}/`: Source code

## Technical Priorities
1. {PRIORITY_1}
2. {PRIORITY_2}
3. {PRIORITY_3}

---

## Workflows
`/research → /plan → /develop → /verify → /finish` | `/stream → /finish-stream`

See `.agent/workflows/` for definitions.

---

## File Index
```json
{"cats":{"entry":"entrypoints","core":"pure logic","features":"business","ui":"presentation","utils":"helpers"},
"files":{"entry":[],"core":[],"features":[],"ui":[],"utils":[]},
"lookup":{"main":"{SRC_DIR}/{MAIN_ENTRY}"}}
```

## Semantics
```json
{"v":"1.0",
"concepts":[],"entities":[],"events":[],"endpoints":[],
"commands":[],"lifecycle_hooks":[],"state_keys":[],
"feature_flags":[],"config_keys":[],"errors":[]}
```

## Project-Specific Rules
TBD — Setup: note conventions, forbidden patterns, gotchas

## Errors
| Symptom | Cause | Fix |
|---------|-------|-----|
| hang | runaway watch/loop | kill, use `--run` |
| module not found | stale/missing import | check File Index |
| circular dep | tight coupling | EventBus / DI |
| layer violation | wrong dependency | move to correct layer |
| recurring bug | similar code elsewhere | `rg` patterns, fix all |

## ⛔ Self-Update
Update as changes happen — don't wait for `/finish`:
- **File Index** → file create/delete/move
- **Semantics** → events, entities, concepts change
- **Errors** → new patterns encountered
- **Rules** → decisions or gotchas discovered
- On `/audit`, verify File Index + Semantics match actual state

---

## Setup Checklist
When adopting this template for a new project:

1. **Project Overview** — Fill `{DESCRIPTION}`, `{PHASE}`, `{TARGET_AUDIENCE}`
2. **Tech Stack** — Fill all tool values, delete rows that don't apply
   - Required: `{LANGUAGE}`, `{PKG_MGR}`, `{BUILD_CMD}`, `{TEST_CMD}`, `{LINT_CMD}`
   - If applicable: `{UI_FRAMEWORK}`, `{CSS_APPROACH}`, `{DEV_CMD}`, `{DEFAULT_PORT}`, `{DATABASE}`, `{API_STYLE}`
3. **Language Tokens** — Fill **every** token in the Language Tokens table
   - All values must be valid shell commands/patterns for your stack
   - Test each grep/fd pattern against your codebase before committing
4. **Architecture** — Analyze imports, populate Layers + Constraints
   - Layers: dependency order between directories
   - Constraints: which layers must not import from which
5. **Key Directories** — Set `{SRC_DIR}`, `{PROJECT}`, `{PROJECT_ROOT}`, add project-specific dirs
6. **File Index** — Populate with actual source files, set `{MAIN_ENTRY}`
   - Categorize every source file into entry/core/features/ui/utils
7. **Semantics** — Populate concepts, entities, events, endpoints from codebase
8. **Project-Specific Rules** — Document conventions, forbidden patterns, gotchas
9. **Technical Priorities** — Set `{PRIORITY_1-3}`
10. **Errors** — Customize symptom table for your stack's common failure modes
11. **Workflows** — Review `.agent/workflows/` — all use tokens defined above
    - Verify `{KI_PATH}` in `stream.md` points to your KI location
12. **Plan Template** — Review `.agent/plans/_template.md` — uses same tokens
