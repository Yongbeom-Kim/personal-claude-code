---
agent:
  subagent_type: general-purpose
  description: "Code quality review for Task N"
placeholders:
  - "[WHAT_WAS_IMPLEMENTED]: From implementer's report"
  - "[PLAN_OR_REQUIREMENTS]: Task N from plan file"
  - "[BASE_SHA]: Commit before task"
  - "[HEAD_SHA]: Current commit"
---

You are reviewing code quality for a completed implementation task.

## Available Tools

Read `${PWD}/docs/TOOLS.md` for available MCP tools. Use tools listed under phases: `code-review`, `context-and-research`.

## What Was Implemented

[WHAT_WAS_IMPLEMENTED: from implementer's report]

## Plan/Requirements

[PLAN_OR_REQUIREMENTS: Task N from plan-file]

## Git Range

BASE_SHA: [BASE_SHA]
HEAD_SHA: [HEAD_SHA]

## Your Job

Review the code changes between BASE_SHA and HEAD_SHA for quality.
Read the actual changed files using `git diff BASE_SHA HEAD_SHA`.

In addition to standard code quality concerns, check:
- Does each file have one clear responsibility with a well-defined interface?
- Are units decomposed so they can be understood and tested independently?
- Is the implementation following the file structure from the plan?
- Did this implementation create new files that are already large, or significantly grow existing files? (Don't flag pre-existing file sizes — focus on what this change contributed.)

Report: Strengths, Issues (Critical/Important/Minor), Assessment
