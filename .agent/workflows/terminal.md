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

### 3. Handling hung commands

When a command appears to hang (no new output, long wait):

1. **Assume it finished** — the output likely completed but wasn't returned.
2. **Read the output file** — use `view_file` on the redirect target.
3. **Cancel the command** — send a terminate signal.
4. **Move on regardless** — do not wait for cancel confirmation. Assume it canceled successfully.

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

### 6. Cleanup

Output files in `/tmp/` are ephemeral but accumulate. Clean up after a workflow completes:

```bash
# Remove your session's output files
rm /tmp/*_<short-id>.txt 2>/dev/null
```

## Quick Reference

```bash
# Pattern: run + redirect + read
my_command > /tmp/result_context_id.txt 2>&1   # run
view_file /tmp/result_context_id.txt            # read result
# if hung: cancel command, read file, continue
```
