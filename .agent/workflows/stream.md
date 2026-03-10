---
description: Parallel development stream
mode: EXECUTION
exceptions: [NO_PLAN_ARTIFACT, REQUIRE_WORKTREE, NO_MERGE]
read_when:
  - Starting a parallel development stream
  - Need isolated dev server for sustained work
---
# Stream

Create isolated worktree for parallel development. Use for full features or batched fixes.

**Does:** Creates worktree + dev server, copies KIs, creates handoff file
**Does NOT:** Implement features (unless requested), create plan artifacts, merge

## Preparation Phase

### 1. Parse Command
Format: `/stream <NAME>`. If missing, ask user for session name.

### 2. Create Worktree
```bash
TIMESTAMP=$(date +%y%m%d%H%M%S); NAME="<user-provided>"
WORKTREE_NAME="{PROJECT}-wt-stream-${NAME}-${TIMESTAMP}"
BRANCH_NAME="stream/${NAME}-${TIMESTAMP}"
cd {PROJECT_ROOT}
git worktree add -b $BRANCH_NAME ../$WORKTREE_NAME main
cd ../$WORKTREE_NAME
```

### 3. Copy KIs & Install
```bash
mkdir -p .agent/KI; echo ".agent/KI/" >> .gitignore
cp -r {KI_PATH}/artifacts/* .agent/KI/; cp {KI_PATH}/metadata.json .agent/KI/
LAST4=$(echo $TIMESTAMP | tail -c 5); PORT="1${LAST4}"
cd {SRC_DIR} && {PKG_MGR} install
```

// turbo-all

### 4. Create Handoff File
Write handoff prompt to `.agent/HANDOFF.md`:

```markdown
# Stream Session Handoff

**Worktree**: `{WORKTREE_NAME}`
**Branch**: `{BRANCH_NAME}`

## Start Dev Server
```bash
cd {SRC_DIR} && {DEV_SERVER_START} {PORT}
```

**URL**: http://localhost:{PORT}

## Knowledge Index
Project KIs in `.agent/KI/`:
- `overview.md` — Project overview
- `metadata.json` — KI summary

## Guidelines
1. **Research Mode**: Findings go in artifacts, not chat
2. **No Plans**: Use `/plan` if needed
3. **Implementation**: Only when explicitly requested
4. **Isolation**: Changes stay in this worktree
5. **Merging**: Only user can merge via `/finish-stream`

## Work Goals
[Describe targets here]
```

Notify user: "Handoff file created at `.agent/HANDOFF.md`. Open new workspace and read that file."

## Development Phase

**DO:** Create research artifacts, run dev server tests, reference `.agent/KI/`, notify human at ~50 messages, run verification before commits
**DO NOT:** Create `implementation_plan.md`, implement without request, commit/push without direction, merge

### Pre-Commit (MANDATORY)
```bash
cd {SRC_DIR} && {LINT_CMD} && {TEST_CMD}
```

## Finishing

> [!CAUTION]
> **NEVER use `/finish` on stream worktree.** Only USER runs `/finish-stream`.

To discard:
```bash
worktree_path=$(pwd); pkill -f "$worktree_path.*dev" 2>/dev/null || true
cd {PROJECT_ROOT}; git worktree remove $worktree_path --force; git branch -D {BRANCH_NAME}
```
