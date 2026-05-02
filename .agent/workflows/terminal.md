---
description: Terminal output workarounds for unreliable command output
---

# /terminal — Reliable Terminal Execution

Terminal output does not always return to the agent due to known bugs. Follow these rules to work around the issue reliably.

## Core Rules

### 1. Always redirect output to a file

Every command must write its results to a file so they can be read later regardless of terminal state.

```bash
# Simple commands — redirect inline
npm run build > /tmp/build_myfeature_a1b2.txt 2>&1

# Complicated commands — wrap in a script first
cat > /tmp/lint_check_a1b2.sh << 'EOF'
#!/bin/bash
set -euo pipefail
eslint src/ --ext .ts 2>&1
echo "EXIT_CODE=$?"
EOF
chmod +x /tmp/lint_check_a1b2.sh
/tmp/lint_check_a1b2.sh > /tmp/lint_check_a1b2.txt 2>&1
```

### 2. Use unique output filenames

Multiple agents may run commands concurrently. Name output files by **what** you're running and **which conversation** it relates to.

Format: `/tmp/<action>_<context>_<short-id>.txt`

Examples:
- `/tmp/build_statestore_f3c1.txt`
- `/tmp/vitest_chatview_9ab2.txt`
- `/tmp/tsc_bridge_e7d4.txt`

Does not need to be cryptographically unique — just distinct enough to avoid collisions.

### 3. Handling silent completions

Commands frequently finish executing but never return output to the agent. This is the primary bug — **not** hanging commands. Assume commands complete quickly and poll for results.

1. **Wait 10 seconds** after launching the command, then check the output file with `view_file`.
2. **If the file has content** — the command finished. Cancel the command and move on.
3. **If the file is empty or missing** — wait another 10 seconds and check again.
4. **Repeat every 10 seconds** until results appear. Most commands finish well within the first check.
5. **Do not wait for the terminal** — the file is the only reliable indicator of completion.

### 4. Compare output across calls

The same command may return different amounts of output between runs. When verifying results:

- Read the output file (`view_file`) rather than trusting inline terminal output.
- If output looks incomplete, re-run with file redirect and compare.
- The file is the source of truth, not the terminal display.

### 5. Script vs. inline

| Use inline redirect | Use a script |
|---------------------|--------------|
| Single command | Multi-step or piped commands |
| Simple `npm`/`npx` calls | Commands with conditionals or loops |
| Quick file checks (`cat`, `wc`) | Anything parsing or transforming output |
| | Blocking runtimes (`python3`, see §6) |

### 6. Blocking commands

Some commands put the agent into a **running state that blocks parallel execution** — no other tools can run until the command finishes. This applies to any long-running interpreter (`python3`, `node`, etc.).

**Fix:** Write the script to a file, make it executable, and wrap in bash so the background command ID is returned and file polling works:

```bash
# 1. Write the script
cat > /tmp/probe_a1b2.sh << 'EOF'
#!/usr/bin/env bash
python3 my_script.py > /tmp/probe_a1b2.txt 2>&1
echo "EXIT_CODE=$?"
EOF
chmod +x /tmp/probe_a1b2.sh

# 2. Execute via bash wrapper (NOT `python3 script.py` directly)
bash /tmp/probe_a1b2.sh > /tmp/probe_a1b2.txt 2>&1
# Now use command_status + view_file to poll results
```

**⛔ Do NOT run these directly — they block the agent:**
- `python3 script.py` → blocks
- `bash script.sh` → blocks (unless via redirect+background)
- `python script.py` → blocks
- `node script.js` → blocks

If you discover other commands that block parallel execution, add them here.

### 7. Cleanup

Output files in `/tmp/` are ephemeral but accumulate. Clean up after a workflow completes:

```bash
# Remove your session's output files
rm /tmp/*_<short-id>.txt 2>/dev/null
```

## Quick Reference

```bash
# Pattern: run + redirect + poll + read
my_command > /tmp/result_context_id.txt 2>&1   # run
# after 10s: check file, cancel command if content present, continue
view_file /tmp/result_context_id.txt            # read result
```
