---
name: autonomously-plan-and-implement
description: "This is a skill used to plan, implement and test simpler requirements. All intents, requirements and clarifications are specified and clarified at the beginning of the workflow, before the agent autonomously takes over the implementation."
---

# Autonomously Plan and Implement a Feature

Help to turn ideas into fully-formed design and specifications, and then well-tested implementations through collaborative dialogue.

Start by understanding the current project context, then ask questions to refine the idea. Once you understand what you're building, present the design to the user.

<HARD-GATE>
Do NOT invoke any implementation skill, write any code, scaffold any project, or take any implementation action until you have presented an approach and the user has approved it. This applies to EVERY project regardless of perceived simplicity.
</HARD-GATE>

## Anti-Pattern: "This Is Too Simple To Need A Design"

Every project goes through this process. A todo list, a single-function utility, a config change — all of them. "Simple" projects are where unexamined assumptions cause the most wasted work. The approach design can be short (a few sentences for truly simple projects), but you MUST present it and get approval.

## Checklist

You MUST create a task for each of these items and complete them in order:

1. **View and Register Tools**: Index all available MCP servers by executing the `register-development-tools` skill.
2. **Explore project context**: check files, docs, recent commits
3. **Ask Clarifying Questions**: Ask **THREE ROUNDS** of clarifying questions, each round asking between 7-10 questions to the user. With this, understand purpose, constraints, and intended design. After each round, go back and further explore the project context.
    - <WARNING>NO CLARIFICATION IS TOO SMALL TO ASK. A great deal of wasted work lies in simple but miscommunicated "assumptions" that has to be corrected later on.</WARNING>
    - Since there are many questions in each round, write it to `${PWD}/CLARIFICATIONS.md`. Delete the file after each round. An example is below.
4. **Propose 2-3 approaches**: Clearly establish trade-offs and your recommendation for human review. This is the last chance to get human feedback, so be thorough.
5. **Write design doc**: Write a design doc at `${PWD}/docs/development/design/YYYY-MM-DD-<topic>-design.md`.
6. **Spec review**: Create a design-spec-document-reviewer subagent with precisely crafted review context (never your session history). The prompt in `./design-spec-document-reviewer-prompt.md` should be part of the subagent's system prompt.
7. **Write implementation plan**: Invoke the `write-implementation-plan` skill to create the implementation plan.
8. **Implementation Plan Review**: Create a implementation-spec-document-reviewer subagent with precisely crafted review context (never your session history). The prompt in `./implementation-spec-document-reviewer-prompt.md` should be part of the subagent's system prompt.
9. **Implement**: invoke the `subagent-driven-execution` skill to implement the implementation plan.
10. **Review and Verify**: Create a review-and-verify subagent. The subagent's prompt should include the design and implementation doc, as well as the prompt in `./review-and-verify-prompt.md`.

### Sample `CLARIFICATIONS.md`

```md
# Clarifications:
1. [Question]
> {user will write answer here}
2. [Question]
> {user will write answer here}
3. [Question]
> {user will write answer here}
4. [Question]
> {user will write answer here}
5. [Question]
> {user will write answer here}
6. [Question]
> {user will write answer here}
7. [Question]
> {user will write answer here}
8. [Question]
> {user will write answer here}
```

## High-Level Control Flow

```python
def autonomously_plan_and_implement(requirement: str) -> None:

    # Phase 1: Context & Clarification (main agent)
    register_development_tools()
    explore_project_context()

    for round in range(3):
        ask_clarifying_questions()  # 7-10 questions per round
        explore_project_context()   # re-explore after each round

    # Phase 2: Design (main agent)
    approaches = propose_approaches(count=3)  # with tradeoffs + recommendation
    approved_approach = wait_for_human_approval(approaches)
    design_doc = write_design_doc(approved_approach)

    changes = invoke_subagent("design-spec-reviewer", input=design_doc)

    # Phase 3: Implementation Planning (main agent)
    read_and_catch_up_with_updates(design_doc)
    implementation_plan = write_implementation_plan(design_doc)
    changes = invoke_subagent("implementation-spec-reviewer", input=implementation_plan)

    # Phase 4: Execution (subagents)
    read_and_catch_up_with_updates(implementation_plan)
    implementation = invoke_subagent("implementation", input=implementation_plan)
    invoke_subagent("review-and-verify", input=(design_doc, implementation))
```

## The Process
**Understanding the idea:**

- Check out the current project state first (files, docs, recent commits)
- Before asking detailed questions, assess scope: if the request describes multiple independent subsystems (e.g., "build a platform with chat, file storage, billing, and analytics"), flag this immediately. Don't spend questions refining details of a project that needs to be decomposed first.
- If the project is too large for a single spec, help the user decompose into sub-projects: what are the independent pieces, how do they relate, what order should they be built? Then brainstorm the first sub-project through the normal design flow. Each sub-project gets its own spec → plan → implementation cycle.
- For appropriately-scoped projects, ask questions one at a time to refine the idea
- Prefer multiple choice questions when possible, but open-ended is fine too
- Focus on understanding: purpose, constraints, success criteria

**Exploring approaches:**

- Propose 2-3 different approaches with trade-offs
- Present options conversationally with your recommendation and reasoning
- Lead with your recommended option and explain why

**Design for isolation and clarity:**

- Break the system into smaller units that each have one clear purpose, communicate through well-defined interfaces, and can be understood and tested independently
- For each unit, you should be able to answer: what does it do, how do you use it, and what does it depend on?
- Can someone understand what a unit does without reading its internals? Can you change the internals without breaking consumers? If not, the boundaries need work.
- Smaller, well-bounded units are also easier for you to work with - you reason better about code you can hold in context at once, and your edits are more reliable when files are focused. When a file grows large, that's often a signal that it's doing too much.

**Working in existing codebases:**

- Explore the current structure before proposing changes. Follow existing patterns.
- Where existing code has problems that affect the work (e.g., a file that's grown too large, unclear boundaries, tangled responsibilities), include targeted improvements as part of the design - the way a good developer improves code they're working in.
- Don't propose unrelated refactoring. Stay focused on what serves the current goal.

## Key Principles

- **YAGNI ruthlessly** - Remove unnecessary features from all designs
- **Explore alternatives** - Always propose 2-3 approaches before settling
- **Incremental validation** - Present design, get approval before moving on
- **Be flexible** - Go back and clarify when something doesn't make sense