---
name: design-frontend
description: "Design frontend UI through visual mockups in the browser. Produces a frontend design spec that feeds into design-and-plan."
---

# Design Frontend

Collaborate with the user to design frontend UI using a browser-based visual companion. Show mockups, wireframes, and layout options — iterate until the user is satisfied — then produce a frontend design spec document.

<HARD-GATE>
Do NOT write any application code. This skill produces a design spec only. The flow after this skill is: `/design-and-plan` (using the spec as input) → `/implement-from-plan`.
</HARD-GATE>

## Steps

1. **Start visual companion**: Read `./visual-companion-guide.md` for the full reference. Launch the server:
   ```bash
   skills/design-frontend/scripts/start-server.sh --project-dir ${PWD}
   ```
   Save `screen_dir` and `state_dir` from the response. Tell the user to open the URL.

2. **Explore project context**: Check files, docs, existing UI code, design tokens, component libraries.

3. **Clarifying questions**: Ask via `AskUserQuestion` — use the terminal for conceptual questions, the browser for visual ones.
   - Purpose, target users, key workflows
   - Existing design system or component library?
   - Responsive requirements, accessibility needs
   - Visual style preferences (show examples in browser)

4. **Design through mockups**: For each major UI area, show 2-3 layout options in the browser and iterate:
   - Page structure and navigation
   - Key screen layouts
   - Component designs and interactions
   - Responsive breakpoints (if relevant)

   Follow the visual companion loop: write HTML → tell user what's on screen → read events + terminal feedback → iterate or advance.

   **Per-question decision:** Use the browser when the user needs to *see* something. Use the terminal for conceptual choices, tradeoffs, and scope questions.

5. **Write frontend design spec**: After the user has validated all visual decisions, write the spec to:
   ```
   ${PWD}/docs/development/design/YYYY-MM-DD-<topic>-frontend-design.md
   ```

6. **User reviews spec**: Ask the user to review the written spec. Make changes if requested.

### Output

After step 6, print a copy-pasteable command and stop:

```
## Frontend Design Complete

Copy this into a new conversation to continue with architecture and planning:

/development:design-and-plan <frontend_design_spec_path>
```

## Frontend Design Spec Format

```markdown
# [Feature Name] Frontend Design

**Goal:** [One sentence]
**Target users:** [Who will use this]

## Visual Decisions

### Page Structure
[Description of overall layout, navigation, page hierarchy]
[Reference mockup files in .visual-companion/ if persisted]

### Screen: [Screen Name]
[Layout description, component arrangement, interactions]

### Screen: [Screen Name]
...

## Component Inventory
[List of UI components needed, with descriptions]
- [Component]: [purpose, behavior, states]

## Responsive Behavior
[How the UI adapts across breakpoints, if applicable]

## Design Constraints
[Accessibility requirements, browser support, performance targets]
[Existing design system/component library to use]

## Open Questions
[Anything not resolved during the visual design phase]
```

## Key Principles

- **Show, don't describe** — use the browser for any visual decision
- **Iterate before advancing** — don't move to the next screen until the current one is validated
- **2-3 options max** per screen — too many choices paralyze
- **Scale fidelity to the question** — wireframes for layout, polish for polish
- **Use real content when it matters** — placeholder text obscures design issues
