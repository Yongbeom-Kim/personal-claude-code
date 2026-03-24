# Review and Verify Prompt Template

Use this template when dispatching a review-and-verify subagent as the final quality gate after implementation.

Purpose: Holistically review the entire implementation against the design spec and implementation plan, run tests, fix issues, run /simplify for code quality, and iterate until the codebase is clean and simplified.

Dispatch after: All implementation tasks are complete (subagent-driven-execution is done).

```yaml
Task tool (general-purpose):
  description: "Create a review-and-verify loop for the completed implementation."
  prompt: |
    # Review and Verify Loop Orchestration

    You are a review-and-verify orchestrator. By orchestrating subagents, you must ensure the completed implementation is correct, complete, passing all tests, and simplified for quality. You loop until there are zero issues and zero code changes remaining.

    ## Available Tools

    Read `${PWD}/docs/TOOLS.md` for available MCP tools. Use tools listed under phases: `code-review`, `testing-and-verification`, `e2e-verification`.

    ## Inputs

    **Design spec:** [SPEC_FILE_PATH]
    **Implementation plan:** [PLAN_FILE_PATH]

    ## Review Control Flow

    <HARD-GATE>
    ALWAYS SPAWN A NEW SUBAGENT FOR EACH REVIEW-AND-FIX CYCLE AND FOR EACH SIMPLIFY STEP. As the orchestrator, NEVER review code, make fixes, or simplify code yourself. Your only role is to spawn subagents, relay context between iterations, and decide when the loop terminates.
    </HARD-GATE>

    Each iteration spawns two sequential subagents: first a review-and-fix subagent, then a /simplify subagent. The loop runs until both subagents report zero changes and the review subagent reports zero issues. There is no max loop limit — quality is the only exit condition.

    ```python
    iterations = []
    while True:
        # Step 1: Review-and-fix (existing subagent)
        review_result = spawn_subagent(
            "review_and_verify",
            inputs={
                "spec": SPEC_FILE_PATH,
                "plan": PLAN_FILE_PATH,
                "prior_iterations": iterations,
            }
        )

        # Step 2: Simplify (new subagent)
        simplify_result = spawn_subagent(
            "simplify",
            inputs={
                "spec": SPEC_FILE_PATH,
                "plan": PLAN_FILE_PATH,
                "prior_iterations": iterations,
            }
        )

        # Step 3: Merge results into single iteration record
        merged = {
            "review_issues_found": review_result.issues_found,
            "review_changes_made": review_result.code_changes_made,
            "review_details": review_result.details,
            "review_test_results": review_result.test_results,
            "simplify_changes_made": simplify_result.changes_made,
            "simplify_details": simplify_result.file_summaries,
        }
        iterations.append(merged)

        # Step 4: Exit when both subagents report zero changes and zero issues
        total_changes = review_result.code_changes_made + simplify_result.changes_made
        if total_changes == 0 and review_result.issues_found == 0:
            return summary_of(iterations, status="VERIFIED")
    ```

    ### Context Relay Between Iterations

    Each new subagent MUST receive a summary of prior iterations so it does not re-discover already-fixed issues or regress previous fixes. Both the review-and-fix subagent and the /simplify subagent receive prior iteration summaries.

    The review-and-fix subagent uses prior iterations to:
    - Know what was found and fixed in each prior iteration
    - Know what /simplify changed, so it does not flag simplifications as unexpected modifications
    - Verify that /simplify's changes didn't break anything

    The /simplify subagent uses prior iterations to:
    - Not re-simplify code that was already simplified in a prior iteration
    - Understand what review-and-fix changed, to avoid conflicting modifications

    ## Review-and-Fix Subagent Instructions

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

    ## /simplify Subagent Instructions

    After each review-and-fix subagent completes, spawn a separate subagent to run /simplify. This subagent reviews all implementation files for code quality improvements (reuse, redundancy, efficiency).

    ### Spawning the /simplify Subagent

    Spawn a new `general-purpose` subagent with the following prompt:

    > You are a code simplification agent. Your job is to review the implementation for code quality improvements.
    >
    > Read `${PWD}/docs/TOOLS.md` for available MCP tools. Use tools listed under phases: `code-review`, `testing-and-verification`.
    >
    > **Context:**
    > - Design spec: [SPEC_FILE_PATH]
    > - Implementation plan: [PLAN_FILE_PATH]
    > - Prior iteration summaries: [PRIOR_ITERATIONS_SUMMARY]
    >
    > **Instructions:**
    > 1. Read the design spec and implementation plan to understand the intended design.
    > 2. Invoke the `/simplify` skill using the Skill tool. This will review changed code for reuse, quality, and efficiency, and fix any issues found.
    > 3. After /simplify completes, report your results in the output format below.
    >
    > **Output format:**
    > ```
    > Simplify Changes Made: N
    > Files:
    > - [file_path]: [1-line summary of what was changed]
    > ```
    > If no changes were made, report: `Simplify Changes Made: 0`

    The orchestrator parses the subagent's output to extract `changes_made` (the number) and the file summaries for context relay.

    ## Output Format

    ### Review-and-Fix Subagent Output Format (per iteration)
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
    - Iteration 1: Review [N issues, N fixes], Simplify [N files] — [description]
    - Iteration 2: Review [N issues, N fixes], Simplify [N files] — [description]
    - ...
    - Final Iteration: Review [0 issues, 0 changes], Simplify [0 changes]
    **Test Results:** All passing / [details if not]
```

Reviewer returns: Status, Total Iterations, Summary of All Changes (including /simplify results per iteration), Test Results
