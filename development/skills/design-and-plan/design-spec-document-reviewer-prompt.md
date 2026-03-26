# Design Spec Document Reviewer Prompt Template

Use this template when dispatching a spec document reviewer subagent.

Purpose: Verify the spec is complete, consistent, and ready for implementation planning.

Dispatch after: Design spec document is written to `${PWD}/docs/development/design`

```yaml
Task tool (general-purpose):
  description: "Create a review loop for a design specifications document."
  prompt: |
    # Design Specifications Document Review Loop Orchestration

    You are a design spec document review orchestrator. By orchestrating subagents, you must ensure that this spec is complete and ready for implementation planning.

    ## Available Tools

    Read `${PWD}/docs/TOOLS.md` for available MCP tools. Use tools listed under phases: `design-and-planning`, `context-and-research`.

    ## Document Review Requirements

    **Spec to review:** [SPEC_FILE_PATH]

    ### What to Check

    | Category | What to Look For |
    |----------|------------------|
    | Completeness | TODOs, placeholders, "TBD", incomplete sections |
    | Consistency | Internal contradictions, conflicting requirements |
    | Clarity | Requirements ambiguous enough to cause someone to build the wrong thing |
    | Scope | Focused enough for a single plan — not covering multiple independent subsystems |
    | YAGNI | Unrequested features, over-engineering |

    ### Calibration

    **Only flag issues that would cause real problems during implementation planning.**
    A missing section, a contradiction, or a requirement so ambiguous it could be
    interpreted two different ways — those are issues. Minor wording improvements,
    stylistic preferences, and "sections less detailed than others" are not.

    ## Review Control Flow

    <HARD-GATE>
    ALWAYS SPAWN A NEW SUBAGENT FOR EACH REVIEW. As the orchestrator, NEVER review the document yourself. Your only role is to spawn new subagents, and to make the decision of when, and when not to spawn new review subagents.
    </HARD-GATE>

    Until your subagent identifies no issues / changes, you will spawn new subagents to review the design spec once more (max_loops=5). Your logic will run as follows:

    ```python
    n_loops = 0
    changes = []
    while (n_loops < 5):
        change = spawn_subagent("review_design_spec_document", [REVIEW_REQUIREMENTS], [DOCUMENT_PATH])
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
    - [Section X]: [specific change] - [significance of change]
    **Issues (if any):**
    - [Section X]: [specific issue] - [why it matters for planning]
    **Recommendations (advisory, do not block approval):**
    - [suggestions for improvement]

    ### Orchestrator Return Format (to main agent)
    **Status:** Approved | Max Loops Reached
    **Iterations:** N
    **Summary of Changes:**
    - [Section X]: [specific change] - [significance of change]
    **Remaining Issues (if Max Loops Reached):**
    - [Section X]: [specific issue] - [why it matters for planning]
```

Reviewer returns: Summary of all changes, Status, Issues (if any), Recommendations