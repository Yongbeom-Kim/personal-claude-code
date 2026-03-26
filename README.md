# Yongbeom's Personal Claude Code Marketplace

A Claude Code plugin marketplace by Kim Yongbeom. It provides development and learning skills built around a design-first, subagent-driven workflow with iterative review loops.

## Installation

1. Add the marketplace:

   ```bash
   claude plugin marketplace add https://github.com/Yongeom-Kim/personal-claude-code.git
   ```

2. Install a plugin:

   ```bash
   claude plugin install development
   claude plugin install learning
   ```

3. Verify the plugins are available by running a skill:

   ```
   /development:register-development-tools
   ```

## Plugins

### Development (v1.0.5)

Skills for code development workflows. Many are derived from [obra/superpowers](https://github.com/obra/superpowers/tree/main).

- **`/development:autonomously-plan-and-implement`**: Turn ideas into fully-tested implementations through collaborative design, planning, and subagent execution.
- **`/development:write-implementation-plan`**: Create a detailed, step-by-step implementation plan from a design spec.
- **`/development:subagent-driven-execution`**: Execute implementation plans using specialized subagents with spec compliance and code quality reviews.
- **`/development:review-and-fix`**: Review code changes with 4 parallel specialized reviewers and iterate until issues are resolved.
- **`/development:register-development-tools`**: Index available MCP servers and categorize them by development phase.

### Learning (v1.0.0)

Skills for guided learning.

- **`/learning:socratic-instruction`**: Socratic tutor that guides learning through questions, never giving direct answers.

## Credits

- **Kim Yongbeom** (dernbu@gmail.com)
- Development skills derived from [obra/superpowers](https://github.com/obra/superpowers/tree/main)
