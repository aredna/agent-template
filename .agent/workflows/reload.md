---
description: Commit all changes, deploy, and reload the extension host (user-only)
mode: VERIFICATION
exceptions: [USER_ONLY]
read_when:
  - User explicitly invokes /reload
---

> [!CAUTION]
> **USER-ONLY WORKFLOW.** Agents must NEVER run this autonomously. Auto-reloading kills all other agents!

## 1. Commit pending changes.
Stage and commit any modifications with a **descriptive commit message** summarizing the actual changes. Generic messages like "Auto-commit before reload" are **NOT allowed**. The message must describe what was modified (e.g., `"fix(data-fetcher): restore S3 catch-up on stream end"`). If there are no changes, skip this step gracefully.
```bash
git add .
git commit -m "<descriptive message about what changed>" > .reload_commit.txt 2>&1 || true
```

## 2. Run the deploy process.
Build and sync the application.
```bash
npm run deploy
```

## 3. Validate the deploy process.
Verify the output of the deploy command. If the output contains errors or fails, STOP the workflow immediately and notify the user about the failure. DO NOT proceed to the next step.

## 4. Trigger the extension reload process.
Provide a quick node script to connect to the PWA's WebSocket and execute the `restart-extension-host` action.
```bash
node -e "const WebSocket=require('ws'); const ws=new WebSocket('ws://localhost:{DEFAULT_PORT}'); ws.on('open', () => { ws.send(JSON.stringify({type: 'ACTION', action: 'restart-extension-host'})); setTimeout(() => process.exit(0), 500); }); ws.on('error', () => process.exit(0));"
```

## 5. Finish.
Conclude the workflow by declaring that changes have been committed, deployed, and the extension Host is reloading.

## ⚠️ Agent Restrictions
DO NOT execute autonomously. DO NOT suggest. If a reload is needed, tell the user: "Use `/reload` when ready."
