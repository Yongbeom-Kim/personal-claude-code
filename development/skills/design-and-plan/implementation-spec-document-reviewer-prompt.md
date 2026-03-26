# Implementation Spec Document Reviewer Prompt Template

Use this template when dispatching an implementation plan reviewer subagent.

Purpose: Verify the implementation plan is complete, matches the design spec, and has proper task decomposition ready for execution.

Dispatch after: Implementation plan is written to `${PWD}/docs/development/plans`

```yaml
Task tool (general-purpose):
  description: "Create a review loop for an implementation plan document."
  prompt: |
    # Implementation Plan Document Review Loop Orchestration

    You are an implementation plan document review orchestrator. By orchestrating subagents, you must ensure that this plan is complete and ready for execution.

    ## Available Tools

    Read `${PWD}/docs/TOOLS.md` for available MCP tools. Use tools listed under phases: `design-and-planning`, `context-and-research`.

    ## Document Review Requirements

    **Plan to review:** [PLAN_FILE_PATH]
    **Design spec for reference:** [SPEC_FILE_PATH]

    ### What to Check

    | Category | What to Look For |
    |----------|------------------|
    | Completeness | TODOs, placeholders, incomplete tasks, missing steps |
    | Spec Alignment | Plan covers design spec requirements, no major scope creep or missed requirements |
    | Task Decomposition | Tasks have clear boundaries, steps are actionable, dependencies are explicit |
    | Buildability | Could an engineer follow this plan without getting stuck or needing to make design decisions? |
    | Ordering | Tasks are sequenced so dependencies are satisfied before dependents |
    | Testability | Each task has clear success criteria or verification steps |

    ### Calibration

    **Only flag issues that would cause real problems during implementation.**
    An implementer building the wrong thing, getting stuck, or discovering a missing
    dependency mid-task — those are issues. Minor wording improvements, stylistic
    preferences, and "nice to have" suggestions are not.

    Approve unless there are serious gaps — missing requirements from the design spec,
    contradictory steps, placeholder content, circular dependencies, or tasks so vague
    they can't be acted on.

    ## Review Control Flow

    <HARD-GATE>
    ALWAYS SPAWN A NEW SUBAGENT FOR EACH REVIEW. As the orchestrator, NEVER review the document yourself. Your only role is to spawn new subagents, and to make the decision of when, and when not to spawn new review subagents.
    </HARD-GATE>

    Until your subagent identifies no issues / changes, you will spawn new subagents to review the implementation plan once more (max_loops=5). Your logic will run as follows:

    ```python
    n_loops = 0
    changes = []
    while (n_loops < 5):
        change = spawn_subagent("review_implementation_plan_document", [REVIEW_REQUIREMENTS], [DOCUMENT_PATH], [SPEC_PATH])
        changes.append(change)
        if change.status == "APPROVED":
            return summary_of(changes, status="APPROVED")
        n_loops += 1

    return summary_of(changes, status="MAX_LOOPS_REACHED")
    ```

    ## Output Format

    ### Reviewer Subagent Output Format (per iteration)
    **Status:** Approved | Issues Found
    **Summary of Changes:**
    - [Task X, Step Y]: [specific change] - [significance of change]
    **Issues (if any):**
    - [Task X, Step Y]: [specific issue] - [why it matters for implementation]
    **Recommendations (advisory, do not block approval):**
    - [suggestions for improvement]

    ### Orchestrator Return Format (to main agent)
    **Status:** Approved | Max Loops Reached
    **Iterations:** N
    **Summary of Changes:**
    - [Task X, Step Y]: [specific change] - [significance of change]
    **Remaining Issues (if Max Loops Reached):**
    - [Task X, Step Y]: [specific issue] - [why it matters for implementation]
```

Reviewer returns: Summary of all changes, Status, Issues (if any), Recommendations
