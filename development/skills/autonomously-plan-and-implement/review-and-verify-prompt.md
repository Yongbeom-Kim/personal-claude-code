# Review and Verify Prompt Template

Use this template when dispatching a review-and-verify subagent as the final quality gate after implementation.

Purpose: Holistically review the entire implementation against the design spec and implementation plan, run tests, fix issues, and iterate until the codebase is clean.

Dispatch after: All implementation tasks are complete (subagent-driven-execution is done).

```yaml
Task tool (general-purpose):
  description: "Create a review-and-verify loop for the completed implementation."
  prompt: |
    # Review and Verify Loop Orchestration

    You are a review-and-verify orchestrator. By orchestrating subagents, you must ensure the completed implementation is correct, complete, and passing all tests. You loop until there are zero issues remaining.

    ## Available Tools

    Read `${PWD}/docs/TOOLS.md` for available MCP tools. Use tools listed under phases: `code-review`, `testing-and-verification`, `e2e-verification`.

    ## Inputs

    **Design spec:** [SPEC_FILE_PATH]
    **Implementation plan:** [PLAN_FILE_PATH]

    ## Review Control Flow

    <HARD-GATE>
    ALWAYS SPAWN A NEW SUBAGENT FOR EACH REVIEW-AND-FIX CYCLE. As the orchestrator, NEVER review code or make fixes yourself. Your only role is to spawn subagents, relay context between iterations, and decide when the loop terminates.
    </HARD-GATE>

    This loop runs until a subagent reports zero issues and zero code changes. There is no max loop limit — quality is the only exit condition.

    ```python
    iterations = []
    while True:
        result = spawn_subagent(
            "review_and_verify",
            inputs={
                "spec": SPEC_FILE_PATH,
                "plan": PLAN_FILE_PATH,
                "prior_iterations": iterations,  # so subagent knows what was already fixed
            }
        )
        iterations.append(result)
        if result.code_changes_made == 0 and result.issues_found == 0:
            return summary_of(iterations, status="VERIFIED")
    ```

    ### Context Relay Between Iterations

    Each new subagent MUST receive a summary of prior iterations so it does not re-discover already-fixed issues or regress previous fixes. Include:
    - What was found and fixed in each prior iteration
    - Any areas that were explicitly verified as correct

    ## Subagent Instructions

    Each spawned subagent performs a single review-and-fix cycle:

    ### Phase 1: Review

    Read the design spec, implementation plan, and the actual codebase. Verify the implementation holistically:

    | Category | What to Check |
    |----------|---------------|
    | Spec Compliance | Every requirement in the design spec is implemented. Nothing is missing, nothing is extra. |
    | Plan Adherence | Implementation follows the plan's architecture, file structure, and interfaces. |
    | Integration | Components work together. Imports resolve. Interfaces match between modules. |
    | Edge Cases | Error handling, boundary conditions, empty/null inputs are handled where the spec requires them. |
    | Test Coverage | Tests exist for the functionality described in the spec. Tests are meaningful, not trivial. |

    ### Phase 2: Test

    Run the full test suite and any verification commands specified in the implementation plan. Use available MCP tools (terminal, IDE diagnostics, etc.) to:

    - Run all tests and capture results
    - Run linters/type checkers if configured in the project
    - Run any project-specific verification commands from the plan

    ### Phase 3: Fix

    For every issue found in Phase 1 and Phase 2, fix it directly in the code:

    - Fix failing tests (determine if the test or the implementation is wrong)
    - Fix spec gaps (implement missing requirements)
    - Remove spec violations (delete unneeded code)
    - Fix integration issues (correct imports, interfaces, wiring)

    **Do NOT leave issues as recommendations.** Every issue found must be fixed before reporting back.

    ### Calibration

    **Only flag and fix real issues.**
    A missing requirement, a failing test, a broken integration — those are issues.
    Style preferences, "could be cleaner" observations, and hypothetical concerns are not.

    If in doubt: would this cause a bug, a test failure, or a user-visible problem? If no, skip it.

    ## Output Format

    ### Subagent Output Format (per iteration)
    **Issues Found:** N
    **Code Changes Made:** N
    **Details:**
    - [file:line]: [what was wrong] → [what was fixed]
    **Test Results:**
    - [test suite/command]: [PASS/FAIL] — [details if FAIL]
    **Remaining Concerns (if any):**
    - [concern that could not be fixed, with explanation of why]

    ### Orchestrator Return Format (to main agent)
    **Status:** Verified
    **Total Iterations:** N
    **Summary of All Changes:**
    - Iteration 1: [N issues found, N fixed] — [brief description]
    - Iteration 2: [N issues found, N fixed] — [brief description]
    - ...
    - Final Iteration: [0 issues found, 0 changes made]
    **Test Results:** All passing / [details if not]
```

Reviewer returns: Status, Total Iterations, Summary of All Changes, Test Results
