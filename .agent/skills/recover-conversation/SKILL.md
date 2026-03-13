---
name: recover-conversation
description: Recover lost or missing Antigravity conversations. Scans brain artifact directories and .pb conversation files to identify missing sessions, extract recoverable information, and generate resumption prompts. Use when conversations have disappeared from the UI after a crash or system failure.
---

# Conversation Recovery Skill

Recover lost Antigravity conversations by scanning on-disk artifacts and protobuf files to reconstruct what was lost, then generating resumption prompts to continue the work in a new session.

## When to Use

- User reports conversations missing from the UI after a crash
- User wants to find a specific lost conversation
- User needs to reconstruct context from a lost session

## Architecture Overview

Antigravity stores conversation data in two locations:

| Location | Format | Contains |
|----------|--------|----------|
| `~/.gemini/antigravity/conversations/*.pb` | Protobuf binary | Full conversation history (messages, tool calls, responses) |
| `~/.gemini/antigravity/brain/<conversation-id>/` | Markdown artifacts | Task checklists, research reports, implementation plans, walkthroughs, screenshots |

The UI maintains its own index of conversations. After a crash, the `.pb` files and brain directories may still exist on disk even when the UI no longer shows them.

## Recovery Workflow

Follow these steps in order. Use file system tools (`list_dir`, `view_file`, `find_by_name`) rather than terminal commands if possible — terminal reliability may be degraded after a crash.

### Step 1: Inventory Brain Directories

List the brain directory sorted by modification time to find recent conversation IDs:

```
list_dir: ~/.gemini/antigravity/brain/
```

Filter for directories modified on the date(s) of interest. Each directory name is a conversation UUID.

### Step 2: Inventory Conversation Files

List the conversations directory to get all `.pb` files:

```
list_dir: ~/.gemini/antigravity/conversations/
```

### Step 3: Cross-Reference

Compare the two lists:
- **Brain dir exists + .pb exists** → Conversation data is on disk, just not in UI. Recovery is possible.
- **Brain dir exists + .pb missing** → Conversation file was deleted. Only brain artifacts survive.
- **No brain dir + .pb exists** → Conversation had no artifacts. Only raw protobuf extraction is possible.

### Step 4: Identify Each Conversation

For each brain directory from the relevant time period, read its artifacts to identify what the conversation was about:

1. **Read `task.md`** first — it's the checklist and usually has the title/objective
2. If no `task.md`, **list the directory** and read any `.md` files (research reports, implementation plans, walkthroughs)
3. Note the completion status from `task.md` checkboxes (`[x]` done, `[/]` in-progress, `[ ]` not started)

### Step 5: Present Findings

Present the user with a table of all identified conversations:

```markdown
| ID (short) | Title / Topic | Status | .pb File | Brain Artifacts |
|------------|---------------|--------|----------|-----------------|
| `abc123de` | Widget Refactor | In-progress | ✅ | task.md, impl_plan.md |
| `def456gh` | API Research | Complete | ✅ | task.md, research.md |
| `ijk789lm` | (empty brain) | Unknown | ✅ | (none) |
```

**Ask the user which conversations they want resumption prompts for**, unless they've already specified or asked for all.

### Step 6: Generate Resumption Prompts

For each selected conversation, create a self-contained resumption prompt markdown file. See [prompt-template.md](prompt-template.md) for the template and [examples/](examples/) for reference.

**Information sources** (in priority order):
1. `task.md` — checklist with completion status
2. `implementation_plan.md` — approved technical plan with file lists and skeletons
3. `walkthrough.md` — completed work summary
4. Research reports and other `.md` artifacts — findings, recommendations
5. `.pb` file via `strings` extraction — raw text dump (last resort, lossy)

**Key elements of a good resumption prompt:**
- Clear statement of what the conversation was doing
- What was completed vs. what remains
- All relevant code evidence (file paths, function names, line numbers)
- Any approved designs or skeletons that were produced
- Specific files involved with their roles
- A clear "Prompt" section at the end telling the new session what to do next

### Step 7: Deliver

Save each prompt as a separate `.md` file in the current conversation's brain directory, named `resume-<short-id>-<topic>.md`. Present them to the user for review.

## Tips

- **Terminal unreliable?** Use `list_dir`, `view_file`, `find_by_name`, and `grep_search` tools instead of `run_command`. These are more reliable after crashes.
- **Large conversations**: Brain artifacts are the richest source. The `.pb` protobuf files are binary and `strings` extraction is lossy — use it only as a supplement.
- **Linked artifacts**: Research reports often reference other brain directories by UUID. Follow those links to gather the full picture.
- **Git state**: If the conversation involved code changes, check the relevant git repository for worktrees, branches, and uncommitted changes to determine how far implementation progressed.

## Additional Resources

- See [prompt-template.md](prompt-template.md) for the resumption prompt template
- See [examples/example-research-recovery.md](examples/example-research-recovery.md) for an example of a recovered research session
- See [examples/example-bugfix-recovery.md](examples/example-bugfix-recovery.md) for an example of a recovered bugfix session
