# Personal agent skills

Development and learning skills by **Kim Yongbeom**, packaged for **Claude Code**, **Cursor**, **Codex**, and other agents that load folder-based skills. Workflows are design-first, subagent-driven when available, and use iterative review loops.

Each workflow is a **single `SKILL.md`** under `development/skills/` or `learning/skills/`. The same folders install through **[Vercel `skills` / `npx skills`](https://github.com/vercel-labs/skills)**, which reads `.claude-plugin/marketplace.json` and links or copies skills into your agent’s skill directories.

## Installation (recommended): `npx skills`

From a project directory (use `-g` / `--global` for a user-wide install):

```bash
# Preview what would be installed from this repo
npx skills add . --list

# Install every skill into this project’s agent skill dirs (non-interactive)
npx skills add . --all

# Cursor only
npx skills add . --agent cursor -y

# Claude Code + Cursor
npx skills add . --agent claude-code cursor -y

# List with a specific agent filter (when supported by your CLI version)
npx skills add . --list --agent cursor
npx skills add . --list --agent codex
```

Install specific skills by **skill id** (the `name` in each `SKILL.md`):

```bash
npx skills add . --skill design-and-plan socratic-instruction -y --agent cursor
```

Use `--copy` if you prefer copies instead of symlinks into agent directories.

**Remote install** (replace with your fork or upstream URL):

```bash
npx skills add https://github.com/Yongbeom-Kim/personal-claude-code.git --list
npx skills add https://github.com/Yongbeom-Kim/personal-claude-code.git --all
```

If **`npx skills add` fails to clone** with `SSL_ERROR_SYSCALL` or similar (common behind corporate proxies or VPNs):

```bash
npx skills add git@github.com:Yongbeom-Kim/personal-claude-code.git --list
git clone git@github.com:Yongbeom-Kim/personal-claude-code.git
npx skills add ./personal-claude-code --all --agent cursor -y
```

### Claude Code (marketplace + plugins)

Compatibility install for Claude Code’s plugin marketplace:

1. Add the marketplace:

   ```bash
   claude plugin marketplace add https://github.com/Yongbeom-Kim/personal-claude-code.git
   ```

2. Install plugins:

   ```bash
   claude plugin install development
   claude plugin install learning
   ```

3. In Claude Code, skills are exposed as **slash commands** (for example `development:design-and-plan`). Other agents invoke the same workflows by **skill name** and natural-language instructions; see each `SKILL.md`.

## Plugins

### Development (v1.0.5)

Skills for software development workflows. Many are derived from [obra/superpowers](https://github.com/obra/superpowers/tree/main).

- **design-and-plan** — Turn ideas into reviewed design specs and implementation plans through dialogue. Produces artifacts on disk.
- **implement-from-plan** — Execute a design spec and implementation plan with fresh context. Pairs with `design-and-plan`.
- **write-implementation-plan** — Create a detailed, step-by-step implementation plan from a design spec.
- **subagent-driven-execution** — Run implementation plans with per-task implementation and review when subagents are available.
- **review-and-fix** — Review changes with a self-looping reviewer until substantive issues are resolved.
- **register-development-tools** — Index available MCP servers and categorize them by development phase.
- **design-frontend** — Browser-based UI design mockups; output feeds `design-and-plan`.

**Claude Code:** these map to commands like `/development:design-and-plan`. **Cursor / Codex:** invoke by skill name (for example “run the `design-and-plan` skill”) or your client’s skill picker.

### Learning (v1.0.0)

- **socratic-instruction** — Socratic tutor: guides learning through questions, not direct answers.

## Credits

- **Kim Yongbeom** (dernbu@gmail.com)
- Development skills derived from [obra/superpowers](https://github.com/obra/superpowers/tree/main)
