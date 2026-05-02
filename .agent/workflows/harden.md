---
description: Test case coverage improver (Fuzzing, Red-Green-Refactor loop)
mode: EXECUTION
exceptions: [REQUIRE_WORKTREE]
read_when:
  - The user invokes /harden
  - You need to perform deep, targeted test coverage expansion and auto-fixing of bugs 
---
# /harden Workflow

**Autonomous — limited user interaction needed.**

Unlike standard coverage sweeps that work horizontally across the board, `/harden` works **vertically**: it focuses deeply on one specific area, relentlessly hitting it with edge cases until a test fails (Red), and then fixes the source code to recover gracefully (Green).

## 0. Worktree ⛔ GATE
`pwd` → `/{PROJECT}`? STOP (you're in main). `*-wt-stream-*`? ✓ | `*-wt-*`? ✓ Continue.

## 1. Target Selection

The agent focuses on a single component or module at a time.
1. **User Defined**: If the user provides an argument (e.g., `/harden {SRC_DIR}/store/manager`), target that file or function.
2. **Auto Selection**: If invoked standalone, auto-detect a high value or complex file (e.g., checking `{SRC_DIR}` for critical logic).

## 2. Functional Baseline (Green Path)

Establish behavior correctness for the target function or module.
1. Create or open the corresponding test file (`{TEST_FILE_SUFFIX}`).
2. Generate **Happy Path tests** verifying normal, expected inputs.
3. Validate that these initial tests **pass (Green)**. If they don't, fix the existing setup or correct the understanding of the codebase before moving on.

> [!TIP]
> Ensure tests adhere to AAA (Arrange, Act, Assert) principles.

## 3. The Fouling Loop (Red Path / Agentic Fuzzing)

Leverage agentic reasoning to generate a progression of increasingly adversarial "fouling" tests. Instead of purely random fuzzing, use **Domain-Specific Synthetic Generation**:
*   Analyze the required schema and intentionally break implicit contracts (e.g., missing nested properties, corrupted JSON).
*   Inject type mismatches, prototype pollution vectors, Infinity, NaN, and boundary extremes.
*   Simulate non-deterministic failures (network drops, mocked components throwing unhandled exceptions).

**The Loop:**
1. Generate batch of fouling tests. *Crucial: test the behavior properties (e.g., "does this throw a validation error") rather than just strict equality, to prevent brittle tests.*
2. Run test file: `{TEST_CMD} <filename>`.
3. If tests **pass**, the code handles the bad data gracefully. Generate harder, more context-aware boundary tests.
4. If a test **fails (Red)**, BREAK out of the loop and move immediately to Phase 4.

## 4. Fix and Refactor (The Green Loop)

A vulnerability or bug has been exposed. Stop generating new tests.

1. Analyze the failing test output and inspect the precise source code trace.
2. Apply the **minimal, safest code fix** to the source code to gracefully handle the fouling condition.
    * *Example: Add runtime type checking, default parameter fallbacks, `try/catch` wrappers, or specific validation throws instead of catastrophic crashes.*
    * *Agentic Guardrail:* Your generated fix is subject to AI hallucinations. Rely strictly on the `{TEST_CMD}` output as the absolute source of truth.
3. Re-run `{TEST_CMD} <filename>`.
4. If it fails, iterate on the code fix.
5. If it passes (Green), the function is securely hardened against this case. Ensure no existing "Golden Path" tests from Phase 2 were broken.

## 5. Escalation: Module Interaction Tests

Once single functions pass their tests, move up the stack to integration coverage.
1. Construct tests that ensure an **entire module works** cohesively (e.g., ensuring Store ingestors correctly map to the ViewModels).
2. Construct tests that ensure **modules interact as expected** (e.g., Transport components updating Store components).
3. Apply the same Fouling / Red Path loop to these interactions (e.g., what happens if the Transport sends out-of-order state updates or drops packets?).
4. If a failure is found, fix the coordination logic in the source code.

## 6. Report and Stop

Stop the workflow once a failure has been successfully identified and fixed, or a significant milestone in module integration has been hardened. 

Provide a summary to the user detailing:
*   Which target area/module was selected.
*   The Happy Path and Fouling test cases that were successfully created and added.
*   The exact failing scenario/issue that was discovered.
*   The codebase correction applied to fix it.
