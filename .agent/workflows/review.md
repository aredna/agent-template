---
description: File clarity review
mode: PLANNING
exceptions: [NO_PLAN_ARTIFACT]
read_when:
  - Reviewing a file for clarity and consistency
  - Checking documentation for ambiguity
  - Auditing a spec or design doc for open questions
---
# Review

Review a file for clarity, logical consistency, and open questions. `/review <filename or description>`

## 1. Locate File
If the argument is a path, open it directly. Otherwise search:
```bash
fd "<description>" . --type f
rg -l "<description>" .
```

## 2. Read & Create Artifact
- Read the full file
- Reproduce it verbatim in a **brain artifact** (`review_<basename>.md`)

## 3. Identify Questions & Inconsistencies
Scan for:
- Direct questions left in the file (`?`, `TBD`, `TODO`)
- Logical contradictions (A says X, B says Y)
- Ambiguous statements that could be read multiple ways
- Missing definitions or undefined terms

## 4. Research Each Question
For every question found, **attempt to answer it** before escalating:
```bash
rg "<term>" . --type md   # Related docs
rg "<term>" . --type js   # Codebase usage
```
Check related files, specs, and codebase for answers.

## 5. Annotate the Artifact
Insert inline annotations directly below the relevant line in the artifact:

```markdown
> **Q**: What happens when X and Y conflict?
> **A**: Resolved in `gdd-105-combat.md` § Damage Priority — X takes precedence.
```

For unresolved questions:
```markdown
> **Q**: What happens when X and Y conflict?
> **A**: ⚠️ OPEN — no definition found in codebase or related docs.
```

## 6. Auto-Fix Large Ambiguity
If a point is genuinely ambiguous **and no other file clarifies it**:
- Add a minimal note to the **source file** (comment appropriate to file type)
- Keep it to one line when possible
- If a corner case is covered in another file, add a cross-reference instead of duplicating:
  `<!-- See gdd-105-combat.md § Damage Priority -->`

**Do not** add notes for things that are clear in context or explained elsewhere.

## 7. Summarize to User
Present via `notify_user`:
- Count of questions found vs. self-resolved
- List of **open questions only** (ones the agent could not answer)
- Any notes added to the source file

## Principles
- **Research first** — answer before asking
- **Concise** — no unnecessary comments in source
- **Cross-reference** — point to other files, don't duplicate
- **Artifact is the workspace** — Q&A lives in the artifact, not the source
