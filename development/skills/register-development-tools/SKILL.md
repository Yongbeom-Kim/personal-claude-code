---
name: register-development-tools
description: "This is a skill for Claude Code agents to understand and index the different MCP servers and tools it has available."
---

# Register and Index Development Tools

The most important part of agentic development is for you to understand what are the tools you have available to you, and what purpose they should be used for.

These tools will be saved in a `${PWD}/docs/TOOLS.md` file.

<REQUIREMENT>
The `${PWD}/docs/TOOLS.md` file must be relative to the directory you are invoked in. Do NOT infer where the `docs/` directory based on signals such as git directories. Directly use the current working directory.
</REQUIREMENT>

## Checklist

Complete the following items in order:

1. **List all available MCP Servers**: List all configured MCP Servers available to you.
2. **Read `${PWD}/docs/TOOLS.md`**: Read the TOOLS.md file, and verify that all available MCP servers are indexed within the file. If they are present, then no further action is needed for the skill.
3. **Human Review**: Present all MCP servers (both listed and in `${PWD}/docs/TOOLS.md`), and prompt the user to review and assign a purpose and phase to each MCP server, or whether they should not be used.
4. **Update `${PWD}/docs/TOOLS.md`**: Update the TOOLS.md file with the MCP servers, their purposes, and their phases. Use the phase enum defined below. An example can be found below.

## Development Phase Enum

Every MCP tool must be categorized under exactly one of these phases. These phases correspond to the stages of the `autonomously-plan-and-implement` workflow.

| Phase | Description | Who Uses It |
|-------|-------------|-------------|
| `context-and-research` | Exploring the codebase, fetching external documentation, understanding project structure | Main agent during clarification; all subagents for orientation |
| `design-and-planning` | Writing and reviewing design specs and implementation plans | Main agent; design-spec-reviewer; implementation-spec-reviewer |
| `code-implementation` | Writing code, semantic code retrieval, code generation | Implementer subagents |
| `testing-and-verification` | Running test suites, linters, type checkers, IDE diagnostics | Implementer subagents; review-and-fix subagent |
| `e2e-verification` | Browser automation, API testing, integration testing against running services | Review-and-fix subagent |
| `code-review` | Static analysis, code quality scanning, complexity analysis | Code quality reviewer; spec compliance reviewer; review-and-fix subagent |

### Phase-to-Workflow Mapping

```
autonomously-plan-and-implement workflow:
│
├─ Step 1: Register tools ────────────── (produces TOOLS.md)
├─ Step 2: Explore context ───────────── context-and-research
├─ Step 3: Clarifying questions ──────── context-and-research
├─ Step 4: Propose approaches ────────── context-and-research, design-and-planning
├─ Step 5: Write design doc ──────────── design-and-planning
├─ Step 6: Design spec review ────────── design-and-planning
├─ Step 7: Write implementation plan ─── design-and-planning
├─ Step 8: Implementation plan review ── design-and-planning
├─ Step 9: Implement (per task) ──────── code-implementation, testing-and-verification
│   ├─ Spec compliance review ────────── code-review
│   └─ Code quality review ──────────── code-review
└─ Step 10: Review and fix ──────────── code-review, testing-and-verification, e2e-verification
```

## Example TOOLS.md

MCP servers must be categorized under phase headers using the exact phase enum values.

```md
# Phase: context-and-research
- Context7: Fetch up-to-date library and framework documentation
- Serena: Semantic code search and symbol resolution

# Phase: design-and-planning
- Serena: Navigate code structure to inform design decisions

# Phase: code-implementation
- Serena: Code semantic retrieval during implementation
- Context7: Fetch API documentation while coding

# Phase: testing-and-verification
- Chrome DevTools: Inspect console for runtime errors during test runs

# Phase: e2e-verification
- Chrome DevTools: Orchestrate browser for end-to-end testing
- Playwright: Browser automation for E2E test scenarios

# Phase: code-review
- Serena: Semantic code analysis for review
```

<NOTE>
A single MCP server may appear under multiple phases if it serves different purposes in each. The purpose description should reflect what the tool is used for in that specific phase.
</NOTE>