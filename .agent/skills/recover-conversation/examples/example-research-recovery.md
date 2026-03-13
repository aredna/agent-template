# Example: Research Session Recovery

This example shows a recovered research synthesis session that linked to 9 sub-reports.

---

```markdown
# Resume: Research Synthesis — Opportunities & Recommendations (806e73db)

> Paste this into a new magyc workspace session to provide full context.

---

## Context

A previous session synthesized 9 parallel research reports into a master recommendations document. The synthesis is complete — all reports read, findings prioritized, implementation order defined.

## What Was Completed

- [x] Read all 9 research reports
- [x] Cross-reference findings
- [x] Prioritize into 4 tiers (Foundation → Quick Wins → New Features → Future)
- [x] Write synthesis document with implementation roadmap
- [x] Document deferred items with prerequisites

## What Remains

- [ ] Begin implementation (Tier 1 first)

## Key Findings / Decisions

- 67 undocumented commands discovered in Antigravity binaries
- CommandBridge wiring through SdkBridge identified as highest-leverage change
- 3 safety tiers defined for command proxy: safe / approval / blocked
- explainAndFixProblem confirmed dead (not in current build)

## Source Reports (all on disk)

| Brain ID | Report | Focus |
|----------|--------|-------|
| `3bb94f3f` | `deep-probe-validation-report.md` | RPC validation |
| `d542464d` | `research-report.md` | 134 commands, 67 new |
| `6308a4b1` | `research-report.md` | CommandBridge wiring |
| ... | ... | ... |

## Prompt

The research synthesis is complete. Begin implementation starting with Tier 1: Wire CommandBridge through SdkBridge, then Command Execute Proxy API. Read the source reports above for deeper detail.
```

## Recovery Notes

This prompt was built from:
1. `task.md` — showed all items completed
2. `research-synthesis.md` — the main 263-line artifact with full roadmap
3. `deferred-research-todos.md` — parked items
4. 9 linked research reports in other brain directories (followed UUID references)

The `.pb` file was not needed because the brain artifacts contained all the information.
