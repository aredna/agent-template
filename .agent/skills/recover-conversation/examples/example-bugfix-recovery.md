# Example: Bugfix Session Recovery

This example shows a recovered bugfix session where implementation was partially complete.

---

```markdown
# Resume: Model & Workspace Selection Bugs (5e6470aa)

> Paste this into a new magyc workspace session to continue the bugfix.

---

## Context

A previous session investigated and began fixing two high-severity bugs in the PWA.

## What Was Completed

- [x] End-to-end pipeline trace (PWA → API → SDK)
- [x] Root cause identification for both bugs
- [x] Research report written

## What Was In Progress

- [/] Fix Bug 1 in `new-convo.js` — was editing `updateModels` to preserve selection

## What Remains

- [ ] Fix Bug 1 in `trajectories.ts` — forward `plannerType`
- [ ] Fix Bug 1 in `sdk-bridge.ts` — accept `plannerType`
- [ ] Fix Bug 2 in `sessions/index.js` — add fingerprint guard
- [ ] Fix Bug 2 in `new-convo.js` — preserve selection on rebuild
- [ ] Fix Bug 2 in `app.js` — debounce `_fetchUserStatus`
- [ ] Run tests and lint

## Key Findings / Decisions

**Bug 1** — `modelSelect.value` can be empty string → `undefined`, and `plannerType` is silently dropped at `trajectories.ts:89`.

**Bug 2** — `state-changed` events trigger `_fetchUserStatus()` which calls `updateModels()` which does `modelSelect.innerHTML = ''` (full rebuild). The chat view has a fingerprint guard but sessions view does not.

## Evidence

```js
// new-convo.js L76-84 — model can be empty string
opts.onSubmit?.({ text, model: modelSelect.value || undefined });

// trajectories.ts L88-99 — plannerType dropped
const { text, model } = req.body;  // plannerType not destructured
```

## Files Involved

| File | Role |
|------|------|
| `src/pwa/js/sessions/new-convo.js` | Dropdown rebuild, form submission |
| `src/server/routes/trajectories.ts` | Drops `plannerType` |
| `src/bridge/sdk-bridge.ts` | No model fallback in `createAgent` |
| `src/pwa/js/chat/index.js` | Reference: has fingerprint guard |

## Prompt

Continue fixing the two model/workspace selection bugs. Bug 1 was partially in progress (new-convo.js changes started). Start there, then fix Bug 2. Run tests and lint when done.
```

## Recovery Notes

This prompt was built from:
1. `task.md` — showed Bug 1 partially in-progress, Bug 2 not started
2. `research_model_workspace_bugs.md` — full research report with code evidence

The key was preserving the exact code snippets and line numbers so the new session doesn't need to re-investigate.
