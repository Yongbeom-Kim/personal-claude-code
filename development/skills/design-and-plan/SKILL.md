---
name: design-and-plan
description: "Turn ideas into reviewed design specs and implementation plans through collaborative dialogue. Produces artifacts on disk; invoke `implement-from-plan` separately to execute."
---

# Design and Plan

Collaborate with the user to turn an idea into a reviewed design doc and implementation plan. This skill produces two artifacts on disk and then **stops** — implementation is a separate invocation with fresh context.

<HARD-GATE>
Do NOT write any code, scaffold any project, or invoke any implementation skill. This skill ends after the implementation plan is reviewed.
</HARD-GATE>

## Steps

Create a task for each step and complete them in order:

1. **Register tools**: Execute the `register-development-tools` skill.
2. **Explore project context**: Check files, docs, recent commits.
3. **Clarifying questions**: Ask **4–7 rounds** via `AskUserQuestion` (up to 4 questions per round, prefer multiple-choice). After each round, re-explore context based on answers.
    - <WARNING>No clarification is too small. Miscommunicated assumptions cause the most wasted work.</WARNING>
    - Early rounds: purpose/scope/success criteria. Middle: constraints/dependencies/edge cases. Later: design preferences/verification.
4. **Propose 2-3 approaches**: Trade-offs and your recommendation. This is the last human feedback before autonomous work.
5. **Write design doc**: Save to `${PWD}/docs/development/design/YYYY-MM-DD-<topic>-design.md`.
6. **Design spec review**: Dispatch a foreground subagent using `./design-spec-document-reviewer-prompt.md`. Do NOT proceed until it completes and you've read its output.
7. **Write implementation plan**: Re-read the (potentially updated) design doc, then invoke the `write-implementation-plan` skill.
8. **Implementation plan review**: Dispatch a subagent using `./implementation-spec-document-reviewer-prompt.md`.

### Output

After step 8, print a copy-pasteable command for a fresh window and stop:

```
## Ready for Implementation

Copy this into a new conversation to start implementation:

/development:implement-from-plan Design Doc: <design_doc_path>, Implementation Plan: <plan_path>
```

## Scope Assessment

Before detailed questions, assess scope. If the request describes multiple independent subsystems, flag it immediately — don't refine details of something that needs decomposition first. Each sub-project gets its own design → plan → implementation cycle.

## Design Principles

- **YAGNI ruthlessly** — remove unnecessary features
- **Explore alternatives** — always 2-3 approaches before settling
- **Design for isolation** — small units, clear interfaces, testable independently
- **Follow existing patterns** in existing codebases
- **Every project gets a design**, even "simple" ones — the design can be short, but it must exist and be approved