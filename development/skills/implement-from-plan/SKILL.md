---
name: implement-from-plan
description: "Execute a design spec and implementation plan produced by design-and-plan-heavy or design-and-plan-light. Starts with fresh context — reads all requirements from disk."
---

# Implement from Plan

Execute an implementation plan that was produced by the `design-and-plan-heavy` or `design-and-plan-light` skill. This skill starts with **zero prior context** — everything it needs is in the artifacts on disk.

## Inputs

Expected invocation format:

```
/development:implement-from-plan Design Doc: <path>, Implementation Plan: <path>
```

Parse the `Design Doc:` and `Implementation Plan:` paths from the arguments. If paths are not provided, ask the user:

```
AskUserQuestion(
  questions=[{
    question: "Which design spec does it correspond to?",
    header: "Design spec file path",
    options: []
  }, {
    question: "Which implementation plan should I execute?",
    header: "Implementation plan file path",
    options: []
  }]
)
```

## Steps

1. **Register tools**: Execute the `register-development-tools` skill.
2. **Read artifacts**: Read both the design spec and implementation plan from disk. These are your sole source of truth — do not assume any prior context.
3. **Record base SHA**: Run `git rev-parse HEAD` and save it as `BASE_SHA` (the commit before implementation begins).
4. **Implement**: Invoke the `subagent-driven-execution` skill with the implementation plan.
5. **Review and fix**: Invoke the `review-and-fix` skill with `BASE_SHA`, `SPEC_FILE_PATH`, and `PLAN_FILE_PATH`.
