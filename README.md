# Yongbeom's Personal Claude Code Marketplace

A Claude Code plugin marketplace by Kim Yongbeom. It provides development and learning skills built around a design-first, subagent-driven workflow with iterative review loops.

The same skill folders are consumable by **[Vercel `skills` / `npx skills`](https://github.com/vercel-labs/skills)**: the CLI reads `.claude-plugin/marketplace.json` and installs each skill under `development/skills/` and `learning/skills/` (you keep **one SKILL.md per workflow**, not a single merged file per plugin).

## Installation

### Claude Code (marketplace + plugins)

1. Add the marketplace:

   ```bash
   claude plugin marketplace add https://github.com/Yongbeom-Kim/personal-claude-code.git
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

### Cursor and other agents (Vercel `npx skills`)

From a project directory (or add `-g` / `--global` for user-wide install):

```bash
# Preview what will be installed
npx skills add Yongbeom-Kim/personal-claude-code --list

# Install everything into this project’s agent skill dirs (non-interactive)
npx skills add Yongbeom-Kim/personal-claude-code --all

# Cursor only
npx skills add Yongbeom-Kim/personal-claude-code --agent cursor -y

# Claude Code + Cursor
npx skills add Yongbeom-Kim/personal-claude-code --agent claude-code cursor -y
```

Install specific skills by name (see skill list below):

```bash
npx skills add Yongbeom-Kim/personal-claude-code --skill design-and-plan socratic-instruction -y --agent cursor
```

Use `--copy` if you prefer copies instead of symlinks into agent directories.

## Plugins

### Development (v1.0.5)

Skills for code development workflows. Many are derived from [obra/superpowers](https://github.com/obra/superpowers/tree/main).

In Claude Code these appear as slash commands; with `npx skills` the **skill id** is the name before the colon (use with `--skill`).

- **`design-and-plan`** (`/development:design-and-plan`): Turn ideas into reviewed design specs and implementation plans through collaborative dialogue. Produces artifacts on disk.
- **`implement-from-plan`** (`/development:implement-from-plan`): Execute a design spec and implementation plan with fresh context. Pair with `design-and-plan`.
- **`write-implementation-plan`** (`/development:write-implementation-plan`): Create a detailed, step-by-step implementation plan from a design spec.
- **`subagent-driven-execution`** (`/development:subagent-driven-execution`): Execute implementation plans using specialized subagents with spec compliance and code quality reviews.
- **`review-and-fix`** (`/development:review-and-fix`): Review code changes with 4 parallel specialized reviewers and iterate until issues are resolved.
- **`register-development-tools`** (`/development:register-development-tools`): Index available MCP servers and categorize them by development phase.

### Learning (v1.0.0)

Skills for guided learning.

- **`socratic-instruction`** (`/learning:socratic-instruction`): Socratic tutor that guides learning through questions, never giving direct answers.

## Credits

- **Kim Yongbeom** (dernbu@gmail.com)
- Development skills derived from [obra/superpowers](https://github.com/obra/superpowers/tree/main)
