---
name: implement-from-plan
description: "Execute a design spec and implementation plan produced by design-and-plan. Starts with fresh context — reads all requirements from disk."
---

# Implement from Plan

Execute an implementation plan that was produced by the `design-and-plan` skill. This skill starts with **zero prior context** — everything it needs is in the artifacts on disk.

## Inputs

Expected information from the user:

- **Design Doc:** path to the design spec
- **Implementation Plan:** path to the plan

Parse `Design Doc:` and `Implementation Plan:` from the user message when provided. If either path is missing, use **interactive user prompt** to ask for:
- which design spec file to use
- which implementation plan file to use

**Fallback:** If structured forms are unavailable, ask in plain chat until both paths are known.

## Steps

1. **Register tools**: Execute the `register-development-tools` skill.
2. **Read artifacts**: Read both the design spec and implementation plan from disk. These are your sole source of truth — do not assume any prior context.
3. **Record base SHA**: Run `git rev-parse HEAD` and save it as `BASE_SHA` (the commit before implementation begins).
4. **Implement**: Invoke the `subagent-driven-execution` skill with the implementation plan.
5. **Review and fix**: Invoke the `review-and-fix` skill with `BASE_SHA`, `SPEC_FILE_PATH`, and `PLAN_FILE_PATH`.
