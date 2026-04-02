---
name: review-and-fix
description: "Review code changes with a single self-looping reviewer subagent that fixes P1/P2 issues until only stylistic issues remain."
---

# Review and Fix

Dispatch a single reviewer with isolated context that reviews code changes across correctness, simplification, and style dimensions, fixes P1/P2 issues directly, and loops until only P3 (stylistic) issues remain.

## When to Use

- **Standalone:** User invokes the `review-and-fix` workflow directly after making changes
- **Embedded:** Called from `implement-from-plan` as the final review step (typically via **foreground subagent dispatch** when available)

## Inputs

| Input | Required | Source |
|---|---|---|
| `BASE_SHA` | No | User argument or interactive ask. Diff is always base_sha vs working tree. |
| `SPEC_FILE_PATH` | No | Passed by `implement-from-plan` |
| `PLAN_FILE_PATH` | No | Passed by `implement-from-plan` |

If `BASE_SHA` is not provided as an argument, use **interactive user prompt** to choose a base for the diff, for example:
- previous commit (`HEAD~1`)
- main branch tip
- merge base with main

**Fallback:** Ask in plain chat which base to use.

## Control Flow

```python
def review_and_fix(base_sha=None, spec_path=None, plan_path=None):
    if not base_sha:
        base_sha = ask_user_for_base_sha()

    diff = git_diff(base_sha, working_tree=True)
    changed_files = extract_changed_files(diff)

    # Foreground subagent dispatch — reviewer self-loops internally
    result = dispatch_reviewer(
        changed_files=changed_files,
        diff=diff,
        spec_path=spec_path,
        plan_path=plan_path,
    )

    print_summary(result)
```

**Fallback:** If subagents are unavailable, perform the same review yourself using `./reviewer-prompt.md` as your checklist, preserving the fix→test→re-review loop and iteration limit.

## Dispatching the Reviewer

Read `./reviewer-prompt.md`, fill in placeholders, and run a **foreground subagent dispatch** with the prompt body (content below the YAML frontmatter).

Placeholders:
- `[CHANGED_FILES_LIST]`: List of files from `git diff --name-only BASE_SHA`
- `[DIFF_CONTENT]`: Output of `git diff BASE_SHA`
- `[SPEC_FILE_PATH]`: The spec path, or "N/A" if standalone
- `[PLAN_FILE_PATH]`: The plan path, or "N/A" if standalone

The reviewer handles its own fix→test→re-review loop internally (up to 5 iterations). You do not manage iterations yourself.

## Red Flags

**Never:**
- Manage the reviewer's fix→re-review loop yourself (the reviewer self-loops)
- Dispatch multiple reviewer subagents in parallel (use the single consolidated reviewer)
- Proceed without waiting for the reviewer's final status
