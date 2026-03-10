# Agent Template

A drop-in `AGENTS.md` + workflow kit for **[Google Antigravity](https://deepmind.google/)**. Gives agents structured processes, git safety rails, and living project context.

## Quick Start

1. **Copy into your project:**
   ```bash
   cp -r .agent/ /path/to/your-project/
   cp AGENTS.md /path/to/your-project/
   ```
2. **Run setup** — open `.agent/SETUP_PROMPT.md` with the agent. It auto-detects your tech stack, fills `AGENTS.md` placeholders, and prunes unused sections.
3. **Use workflows** — `/research`, `/plan`, `/develop`, `/verify`, `/finish`.

## Antigravity Modes

This template is built around the three modes in Antigravity's system prompt. Each workflow declares its mode in YAML frontmatter (`mode: PLANNING`, etc.), so the agent switches automatically as it moves through the development lifecycle.

| Mode | What the agent does | What it assumes |
|------|--------------------|-----------------| 
| **PLANNING** | Reads code, evaluates design options, produces plans/reports, asks for user approval. | Nothing gets implemented yet. All output is artifacts and documentation. |
| **EXECUTION** | Writes code, modifies files, implements designs inside a git worktree. | A plan or clear direction already exists. Unexpected complexity → back to Planning. |
| **VERIFICATION** | Runs tests/linters, validates correctness, produces proof-of-work walkthroughs. | Implementation is complete. Checking, not building. Minor fixes stay here; fundamental issues → back to Planning. |

## Workflows

The primary development loop:

```
/research (PLANNING) → /plan (PLANNING) → /develop (EXECUTION) → /testcases (EXECUTION) → /verify (VERIFICATION) → /finish (VERIFICATION)
```

| Command | Mode | Purpose |
|---------|------|---------|
| `/research` | Planning | Investigate code, evaluate design alternatives, produce an approved skeleton. **Only interactive workflow** — all others run autonomously from its output. |
| `/plan` | Planning | Decompose a research skeleton into sequenced, self-contained plan files in `.agent/plans/`. No design decisions here. |
| `/develop` | Execution | Implement a plan (or ad-hoc instructions) inside a git worktree. Includes persona-based self-review and a mandatory test gate. |
| `/testcases` | Execution | Full test lifecycle: audit coverage, add/update/delete test cases. Auto-scopes to changed files when called from `/develop`, or audits the full codebase standalone. |
| `/verify` | Verification | Confirm every planned pattern exists, run persona sweep on diffs, sync `AGENTS.md`. Required before `/finish`. |
| `/finish` | Verification | Stage, commit, rebase onto `main`, fast-forward merge, remove worktree. |

Supporting workflows:

| Command | Mode | Purpose |
|---------|------|---------|
| `/stream` | Execution | Spin up an isolated worktree with its own dev server for sustained parallel work. |
| `/finish-stream` | Verification | Merge a stream worktree back to `main`. **User-only** — agents never run this. |
| `/audit` | Planning | Read-only codebase health check (docs accuracy, dependencies, code quality, test coverage). |
| `/qa` | Planning | Deep exploratory bug hunting with structured severity reporting. |
| `/review` | Planning | Review a file for clarity, logical consistency, and open questions. Inline Q&A in a brain artifact; auto-adds notes only for genuine ambiguity. |
| `/cleanup` | Execution | Safely kill orphaned dev servers and remove stale worktrees. |
| `/validate-ki` | Verification | Verify Knowledge Item artifacts haven't drifted from the actual codebase. |

## Git Safety

All code changes happen in **git worktrees**, never directly on `main`. Worktrees are created automatically by `/develop` and `/stream`. Merging only happens through `/finish` (or `/finish-stream` for parallel streams, run by the user). Pre-commit hooks block direct main commits, and `/verify` must pass before `/finish`.

Multi-agent rules prevent parallel agents from stomping on each other: no stashing, no branch switching, scoped commits, and unrecognized files are ignored.

## AGENTS.md Sections

After setup, `AGENTS.md` contains: engineering rules, tech stack, architecture layers & constraints, a categorized file index, a semantics index (events, entities, domain concepts), and known error patterns. Agents self-update this file as the codebase evolves — the `/verify` gate enforces it.

## License

This project is released into the public domain under [The Unlicense](LICENSE).
