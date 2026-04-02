---
name: design-and-plan
description: "Turn ideas into reviewed design specs and implementation plans through collaborative dialogue. Produces artifacts on disk; invoke `implement-from-plan` separately to execute."
---

# Design and Plan

Collaborate with the user to turn an idea into a reviewed design doc and implementation plan. This skill produces two artifacts on disk and then **stops** — implementation is a separate invocation with fresh context.

<HARD-GATE>
Do NOT write any code, scaffold any project, or invoke any implementation skill. This skill ends after the implementation plan is reviewed.
</HARD-GATE>

## Inputs

This skill accepts an optional frontend design spec produced by the `design-frontend` skill. Pass the path as an argument when your agent supports it (for example: `design-and-plan` with `<frontend_design_spec_path>`).

If a frontend spec path is provided, read it at the start. It contains validated visual decisions (layouts, components, screens) that should be incorporated into the design doc — do not re-ask questions the frontend spec already answers.

## Steps

Create a task for each step and complete them in order:

1. **Register tools**: Execute the `register-development-tools` skill.
2. **Read frontend spec** (if provided): Read the frontend design spec. Note which areas are already decided (UI layouts, component inventory, responsive behavior) so you can skip those in clarifying questions.
3. **Explore project context**: Check files, docs, recent commits.
4. **Clarifying questions**: Run **4–7 rounds** of **interactive user prompt** — structured questions with short options where possible (up to 4 questions per round). After each round, re-explore context based on answers.
    - <WARNING>No clarification is too small. Miscommunicated assumptions cause the most wasted work.</WARNING>
    - Early rounds: purpose/scope/success criteria. Middle: constraints/dependencies/edge cases. Later: design preferences/verification.
    - If a frontend spec was provided, focus on technical architecture, data flow, API design, and backend concerns — skip visual/UI questions already answered by the spec.
    - **Fallback:** If structured multi-choice prompts are not available, ask the same questions in plain chat, one focused question at a time.
5. **Propose 2-3 approaches**: Trade-offs and your recommendation. This is the last human feedback before autonomous work.
6. **Write design doc**: Save to `${PWD}/docs/development/design/YYYY-MM-DD-<topic>-design.md`. If a frontend spec was provided, incorporate its visual decisions into the design doc (reference or inline the relevant sections).
7. **Design spec review**: Read `./design-spec-document-reviewer-prompt.md` and use the body (below the frontmatter) as the prompt for a **foreground subagent dispatch** — isolated context, wait until finished, then read the subagent’s output. **Fallback:** If subagents are unavailable, perform the same review yourself in this session using the same checklist and iteration limit.
8. **Write implementation plan**: Re-read the (potentially updated) design doc, then invoke the `write-implementation-plan` skill.
9. **Implementation plan review**: Read `./implementation-spec-document-reviewer-prompt.md` and use it as the prompt for another **foreground subagent dispatch** (same fallback as step 7).

### Output

After step 9, print a copy-pasteable instruction for a fresh session and stop:

```
## Ready for Implementation

Start a new conversation and run implement-from-plan with:

Design Doc: <design_doc_path>, Implementation Plan: <plan_path>
```

**Compatibility (Claude Code):** `/development:implement-from-plan Design Doc: <design_doc_path>, Implementation Plan: <plan_path>`

## Scope Assessment

Before detailed questions, assess scope. If the request describes multiple independent subsystems, flag it immediately — don't refine details of something that needs decomposition first. Each sub-project gets its own design → plan → implementation cycle.

## Design Principles

- **YAGNI ruthlessly** — remove unnecessary features
- **Explore alternatives** — always 2-3 approaches before settling
- **Design for isolation** — small units, clear interfaces, testable independently
- **Follow existing patterns** in existing codebases
- **Every project gets a design**, even "simple" ones — the design can be short, but it must exist and be approved
