# Resumption Prompt Template

Use this template when generating a resumption prompt for a recovered conversation. Fill in each section from the conversation's brain artifacts.

---

```markdown
# Resume: <Title> (<short-id>)

> Paste this into a new <workspace> session to continue where the lost conversation left off.

---

## Context

<1-2 paragraph description of what this conversation was doing. Include the overall goal and any important background.>

## What Was Completed

<Checklist of completed items from task.md. Use [x] notation.>

- [x] Item 1
- [x] Item 2

## What Was In Progress

<Items marked [/] in task.md, with details about partial progress.>

- [/] Item 3 — <describe how far it got>

## What Remains

<Uncompleted items from task.md.>

- [ ] Item 4
- [ ] Item 5

## Key Findings / Decisions

<If the conversation produced research or made design decisions, summarize them here. Include:>
- Root causes identified
- Approved designs or skeletons
- Important constraints or gotchas discovered

## Evidence

<Code snippets, file paths, line numbers, and other concrete evidence from the research. This is critical for the new session to pick up without re-investigating.>

```language
// relevant code with file:line references
```

## Files Involved

| File | Role |
|------|------|
| `path/to/file.ts` | Description of relevance |

## Prompt

<Clear instruction to the new session about what to do next. Be specific — e.g., "Continue implementing Bug 1 fixes starting with trajectories.ts" rather than "Continue the work.">
```
