---
name: review-and-fix
description: "Review code changes with 4 parallel specialized reviewers, then implement fixes in a loop until only stylistic issues remain."
---

# Review and Fix

Review code changes by dispatching 4 specialized reviewer subagents in parallel, collecting structured recommendations, implementing fixes, and iterating until only P3 (stylistic) issues remain.

<HARD-GATE>
You MUST NOT ignore any P1 or P2 recommendation. You may ONLY skip P3
recommendations (stylistic suggestions and personal preferences).
Functional correctness, performance, security, error handling, and all other
non-functional concerns MUST be addressed before the loop can exit.
</HARD-GATE>

## When to Use

- **Standalone:** User invokes `/review-and-fix` directly after making changes
- **Embedded:** Called from `implement-from-plan` as the final review step (runs in a subagent)

## Inputs

| Input | Required | Source |
|---|---|---|
| `BASE_SHA` | No | User argument or interactive ask. Diff is always base_sha vs working tree. |
| `SPEC_FILE_PATH` | No | Passed by `implement-from-plan` |
| `PLAN_FILE_PATH` | No | Passed by `implement-from-plan` |

If `BASE_SHA` is not provided as an argument, ask the user interactively:

```
AskUserQuestion(
  questions=[{
    question: "What base commit should I diff against?",
    header: "Base SHA",
    options: [
      { label: "HEAD~1", description: "Compare against the previous commit" },
      { label: "main/master", description: "Compare against the main branch" },
      { label: "Merge base", description: "Auto-detect merge base with main branch" }
    ],
    multiSelect: false
  }]
)
```

## Control Flow

```python
def review_and_fix(base_sha=None, spec_path=None, plan_path=None):
    # Step 1: Determine base commit
    if not base_sha:
        base_sha = ask_user_for_base_sha()

    iterations = []

    while True:
        # Step 2: Get current diff (always base_sha vs working tree)
        diff = git_diff(base_sha, working_tree=True)
        changed_files = extract_changed_files(diff)

        # Step 3: Spawn 4 reviewers in parallel
        reviewer_context = {
            "diff": diff,
            "changed_files": changed_files,
            "spec_path": spec_path,      # None in standalone mode
            "plan_path": plan_path,       # None in standalone mode
        }
        results = parallel_spawn([
            simplify_reviewer(reviewer_context),   # ./simplify-reviewer-prompt.md
            style_reviewer(reviewer_context),       # ./style-reviewer-prompt.md
            broad_reviewer(reviewer_context),       # ./broad-reviewer-prompt.md
            deep_dive_reviewer(reviewer_context),   # ./deep-dive-reviewer-prompt.md
        ])

        # Step 4: Collect and deduplicate recommendations
        all_recommendations = collect_and_deduplicate(results)

        # Step 5: Categorize by severity
        p1_p2 = [r for r in all_recommendations if r.severity in ("P1", "P2")]
        p3 = [r for r in all_recommendations if r.severity == "P3"]

        # Step 6: Exit check — if no P1/P2, we're done
        if len(p1_p2) == 0:
            log_skipped_p3s(p3)
            print_final_summary(iterations)
            return

        # Step 7: Resolve conflicts between reviewers (use your judgment)
        resolved = resolve_conflicts(p1_p2)

        # Step 8: Implement all P1/P2 fixes
        implement_fixes(resolved)

        # Step 8b: Run tests and fix any failures
        test_results = run_tests_and_fix_failures()

        # Step 9: Record iteration and summarize
        iteration = {
            "recommendations": all_recommendations,
            "fixes_applied": resolved,
            "p3_skipped": p3,
            "test_results": test_results,
        }
        iterations.append(iteration)
        print_iteration_summary(iteration)

        # Loop back to Step 2
```

## Dispatching Reviewers

Spawn all 4 reviewers as `general-purpose` subagents using the Agent tool, all in a single message (parallel). Each reviewer's prompt is in its own file:

1. **Simplify Reviewer** — `./simplify-reviewer-prompt.md`
2. **Style Reviewer** — `./style-reviewer-prompt.md`
3. **Broad Reviewer** — `./broad-reviewer-prompt.md`
4. **Deep Dive Reviewer** — `./deep-dive-reviewer-prompt.md`

For each, fill in the template placeholders:
- `[CHANGED_FILES_LIST]`: List of files from `git diff --name-only BASE_SHA`
- `[DIFF_CONTENT]`: Output of `git diff BASE_SHA`
- `[SPEC_FILE_PATH]`: The spec path, or "N/A" if standalone
- `[PLAN_FILE_PATH]`: The plan path, or "N/A" if standalone

## Deduplication

When multiple reviewers flag the same file:line, keep the recommendation with the highest severity. If same severity, merge descriptions.

## Conflict Resolution

When reviewers disagree (e.g., Simplify says "extract helper", Style says "keep inline"):
1. Correctness reviewers (Broad, Deep Dive) always take priority over Style and Simplify
2. For Style vs Simplify conflicts, use your judgment — consider which recommendation better serves the codebase
3. Log the conflict and your reasoning in the iteration summary

## Implementing Fixes

You (the orchestrator) implement all P1/P2 fixes directly. Do NOT spawn a subagent for fixes.
After implementing:
1. Run the project's test suite
2. Run linters/type checkers if configured
3. Fix any test failures or lint errors
4. Then proceed to the next iteration

## Iteration Summary Format

Print after each iteration:

```
## Iteration N Summary

### Recommendations Received
- Simplify: X recommendations (P1: a, P2: b, P3: c)
- Style: X recommendations (P1: a, P2: b, P3: c)
- Broad: X recommendations (P1: a, P2: b, P3: c)
- Deep Dive: X recommendations (P1: a, P2: b, P3: c)

### Conflicts Resolved
- [file:line]: Simplify said X, Style said Y → chose Y because [reason]

### Fixes Applied (P1/P2)
- [file:line]: [description] → [fix applied]

### P3 Skipped
- [file:line]: [description] — skipped because [reason]

### Test Results
- [test suite/command]: [PASS/FAIL] — [details if FAIL]

### Status: CONTINUING | COMPLETE
```

## Final Summary Format

Print when exiting the loop (no P1/P2 remaining):

```
## Review and Fix Complete

### Total Iterations: N
### Iteration History
- Iteration 1: [X fixes applied, Y P3 skipped]
- ...

### All P3 Recommendations Skipped
- [file:line]: [description] — [reason]

### Test Results: All passing / [details if not]

### Status: VERIFIED
```

## Severity Classification

| Severity | Description | Action |
|---|---|---|
| **P1 (Critical)** | Bugs, crashes, data loss, security issues, broken logic | Must fix. Cannot skip. |
| **P2 (Important)** | Performance issues, error handling gaps, missing edge cases, non-functional concerns | Must fix. Cannot skip. |
| **P3 (Minor)** | Style preferences, naming suggestions, "could be cleaner" observations | May skip. Logged. |

## Red Flags

**Never:**
- Ignore a P1 or P2 recommendation
- Skip running tests after implementing fixes
- Let a reviewer subagent write code or edit files
- Proceed to next iteration without summarizing the current one
- Exit the loop with unresolved P1/P2 issues
- Implement P3 fixes (skip them and log)
