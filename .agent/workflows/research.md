---
description: Investigation workflow
mode: PLANNING
exceptions: [NO_PLAN_ARTIFACT, READ_ONLY]
read_when:
  - Investigating how something works
  - Exploring unfamiliar code
  - Tracing execution paths or data flow
  - Evaluating design approaches before /plan
  - Making architectural decisions collaboratively
---
# Research

Investigate codebase behavior and make design decisions with user. **Interactive — this is where design decisions happen.**

> [!IMPORTANT]
> `/research` is the only interactive workflow. All downstream (`/plan` → `/develop` → `/verify` → `/finish`) run autonomously from research output.

## Outcome
Report documenting: how functionality works, component interactions, findings, **approved design decisions**, and **structural skeleton** (if applicable).

## 1. Define Focus
What to investigate: How does X work? What calls Y? Where is Z used?

## 2. Trace
```bash
rg "functionName\(" {SRC_DIR} --type js  # Entry points
```
Follow execution path. Track data transformations.

## 2.5 Design Alternatives (for architectural research)

When the research goal involves choosing an approach:

1. **Generate 2-3 alternatives** with distinct tradeoffs
   - Each: one-paragraph description, key pros, key cons
2. **Score against project constraints**
   | Alternative | Complexity | Performance | Maintainability | Risk |
   |-------------|-----------|-------------|-----------------|------|
   | A: ... | Low | Med | High | Low |
3. **Recommend one** with brief justification
4. **Get user approval** on chosen approach before proceeding to skeleton

Skip this step for pure investigation research (tracing bugs, understanding behavior).

## 2.7 Structural Skeleton

After approach is chosen, produce a skeleton detailed enough that
`/plan` can decompose and `/develop` can implement without making
design decisions: module boundaries, signatures, types, error strategy,
data flow, integration points.

> [!CAUTION]
> If `/develop` would need to decide between two approaches, the skeleton
> is incomplete. Every ambiguity must be resolved here.

### Skeleton Review (with user)
Run all personas and present concerns to user
(see `AGENTS.md` → Review Personas).

Revise skeleton based on user feedback. Include approved skeleton
in the research report under a "## Approved Skeleton" section.

## 3. Document
```markdown
# Research Report: {Topic}

## Summary
Investigation focus.

## Methodology
Files examined, patterns searched, traces followed.

## Findings
### Finding 1: {Title}
**Severity**: Critical|High|Medium|Low|Note
**Category**: Logic|Performance|Security|Architecture|Documentation
**Location**: `path:L123` | **Description**: What was found
**Evidence**: Code | **Recommendation**: Action

## Design Alternatives (if applicable)
### Alternative A: {Name}
Pros: ... | Cons: ... | Score: ...
### ✅ Chosen: {Name} — {rationale}

## Approved Skeleton (if applicable)
Use skeleton format from `.agent/plans/_template.md` (Module, Types, Data Flow, Error Strategy sections).

## Files Examined
- `path/to/file.js` — Relevance
```

## After Research
Bugs → `/develop` | Architecture decisions made → `/plan` | Documentation only → Report is deliverable
