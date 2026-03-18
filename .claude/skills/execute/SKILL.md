---
name: execute
description: Execute an implementation plan by name. Usage - /execute <plan-name> (matches files in docs/superpowers/plans/)
user-invocable: true
---

# Execute Plan

You are about to execute an implementation plan. Follow this process exactly.

## 1. Resolve the Plan

The user provided a plan name or partial name as arguments: `$ARGUMENTS`

Find the matching plan file in `docs/superpowers/plans/`. If the argument is:

- A full filename: use it directly (e.g., `2026-03-18-claude-code-stack-template.md`)
- A partial match: glob for `docs/superpowers/plans/*$ARGUMENTS*` and use the best match
- Empty: list available plans in `docs/superpowers/plans/` and ask the user to pick one

If no match is found, tell the user and list available plans.

## 2. Read the Plan and Spec

**Before doing anything else:**

1. Read the full plan file
2. Find the `**Spec:**` line in the plan header — it points to the design spec
3. Read the full spec file
4. Understand both documents completely before proceeding

## 3. Verify Required Plugins

Check that these are available — if any are missing, STOP and tell the user how to install them:

- **superpowers** — try invoking a superpowers skill
- **frontend-design** — check for `frontend-design:frontend-design` skill
- **context7 MCP** — check for `mcp__context7__resolve-library-id` tool

Do NOT proceed without all three.

## 4. Check for Resume

Scan the plan for task steps. If any steps are already marked `- [x]` (completed), report:

> "Found progress: Tasks 1-N are already completed. Resuming from Task X, Step Y."

If all steps are `- [ ]`, start from the beginning.

## 5. Execute

Now invoke `superpowers:executing-plans` to execute the plan.

**Rules during execution:**

- Follow each step exactly as written in the plan
- If a step includes code, write exactly that code — do not improvise or "improve" it
- If a step says to verify via context7, actually call `mcp__context7__query-docs` before proceeding
- Mark each step `- [x]` in the plan file as you complete it
- If a step fails, STOP and explain what went wrong — do not skip steps

## 6. After Completion

When all tasks are done:

1. Run the final verification task (usually the last task in the plan)
2. Update all remaining `- [ ]` to `- [x]` in the plan
3. Tell the user, replacing `<plan-file>` with the actual plan filename:

> "Plan execution complete. All N tasks finished.
>
> **Recommended next step:** Clear context and run:
> ```
> /review-spec @docs/superpowers/plans/<plan-file>
> ```"
