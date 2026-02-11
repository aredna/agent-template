---
description: Set up agent infrastructure for a project
---
# Setup Agent Infrastructure

Initializes the `.agent` folder and root `AGENTS.md`. Delete this file after setup.

## 1. Copy Template
```bash
cp -r /home/gotem/dev/project_template/.agent {TARGET_PROJECT}/
cp /home/gotem/dev/project_template/AGENTS.md {TARGET_PROJECT}/
rm {TARGET_PROJECT}/.agent/SETUP_PROMPT.md
```

## 2. Auto-Detect Settings
```bash
basename $(pwd); pwd
[ -f bun.lockb ] && echo bun; [ -f pnpm-lock.yaml ] && echo pnpm; [ -f package-lock.json ] && echo npm
[ -d src ] && echo src; [ -d app ] && echo app
jq -r '.scripts|keys[]' package.json | grep -E "test|lint|dev"
```

## 3. Confirm with User
Present detected values in a table. **Wait for confirmation before proceeding.**

## 4. Apply Settings
Create `settings.env` with detected values:
```bash
cat > /tmp/settings.env << 'EOF'
PROJECT=value
PROJECT_ROOT=value
SRC_DIR=value
LANGUAGE=value
UI_FRAMEWORK=value
CSS_APPROACH=value
PKG_MGR=value
BUILD_CMD=value
DEV_CMD=value
DEFAULT_PORT=value
TEST_CMD=value
LINT_CMD=value
FORMATTER=value
DATABASE=value
API_STYLE=value
MONOREPO_TOOL=value
CI_PLATFORM=value
KI_PATH=value
DESCRIPTION=value
PHASE=value
TARGET_AUDIENCE=value
PRIORITY_1=value
PRIORITY_2=value
PRIORITY_3=value
EOF
```
Apply:
```bash
cd {TARGET_PROJECT}
while IFS='=' read -r key val; do
  [ -z "$key" ] && continue
  find . \( -name AGENTS.md -o -path './.agent/*.md' \) -exec sed -i "s|{$key}|$val|g" {} +
done < /tmp/settings.env
```

## 4b. Prune Tech Stack
Delete any Tech Stack rows from `AGENTS.md` that don't apply to this project (e.g., no Database, no Monorepo). The table should only contain tools actively used.

## 5. Populate AGENTS.md (Agent-Driven)
Analyze the codebase and populate these sections based on your best judgment:
- **Architecture → Layers**: Identify dependency layers from the source (e.g., core → features → ui)
- **Architecture → Constraints**: Document observed patterns (e.g., core has no DOM imports)
- **File Index JSON**: Categorize source files (entry, core, features, ui, utils)
- **Semantics JSON**: Add events, entities, domain concepts
- **Project-Specific Rules**: Note any conventions, forbidden patterns, or gotchas discovered
- **Errors**: Pre-populate with any known error patterns from the codebase

## 6. Verify
```bash
grep -rn '{[A-Z_]*}' AGENTS.md .agent/ && echo "⛔ Fix placeholders" || echo "✓ Done"
```
