# Vite SPA + TanStack — Claude Code AI Template

A zero-code template that turns Claude Code into an expert for the **React + Vite + TypeScript + TanStack Router + TanStack Query + shadcn/ui + Lucide + react-hook-form** stack.

No application code is included. This template provides CLAUDE.md instructions, skills, hooks, and settings that give Claude deep knowledge of the stack so it generates correct, consistent code from the start.

## Requirements

Before using this template, install three Claude Code plugins:

| Plugin | Purpose | Install |
|--------|---------|---------|
| **superpowers** | Planning, TDD, debugging, code review workflows | Claude Code plugin marketplace |
| **frontend-design** | Production-grade UI generation | Claude Code plugin marketplace |
| **context7 MCP** | Live documentation lookup for all stack libraries | See MCP setup below |

### context7 MCP Setup

Add to your MCP configuration (`.claude/mcp.json` or global MCP settings):

```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp@latest"]
    }
  }
}
```

## Quick Start

### Option A: New Project

1. Copy this template to a new directory
2. Open Claude Code in that directory
3. Say: **"Set up the project"**

Claude will run the `01-setup.md` scaffolding skill — an 8-phase process that installs all dependencies, configures the toolchain, and creates the project structure with checkpoints along the way.

### Option B: Drop Into Existing Project

1. Copy `CLAUDE.md`, `.claude/settings.json`, and `.claude/skills/` into your existing project
2. Open Claude Code

Claude will apply conventions to new code only, detect your existing package manager, and warn about any conflicting libraries.

## Common Commands

### Building Features

| What you say | What Claude does |
|---|---|
| "Set up the project" | Runs 8-phase scaffolding with checkpoints |
| "Create a page for /settings" | Creates route file, component, query (if needed), smoke test |
| "Build a user management feature" | Creates full feature module: schemas, queries, components, routes, tests |
| "Add a contact form" | Creates Zod schema, form component with validation, mutation, tests |

### Using Skills Directly

Claude automatically routes your requests to the right skill, but you can also reference them:

| Skill | When to use |
|-------|-------------|
| `phases/01-setup.md` | Scaffold a new project from scratch |
| `phases/02-architecture.md` | Understand the project structure |
| `workflows/new-page.md` | Add a route/page |
| `workflows/new-feature.md` | Build a complete feature end-to-end |
| `workflows/new-form.md` | Build a form with validation |
| `experts/tanstack-router.md` | Working with routing |
| `experts/tanstack-query.md` | Working with data fetching |
| `experts/shadcn-ui.md` | Working with UI components |
| `experts/react-hook-form.md` | Working with forms |
| `experts/lucide.md` | Working with icons |
| `experts/testing.md` | Writing tests |

### Plan Execution & Review

If you have an implementation plan in `docs/superpowers/plans/`:

```
/execute @docs/superpowers/plans/<plan-file>.md
```

After execution completes, review the implementation against the spec:

```
/review-spec @docs/superpowers/plans/<plan-file>.md
```

## What's Included

```
vite-spa-tanstack/
├── CLAUDE.md                          # Master instructions for Claude
├── USER_GUIDE.md                      # This file
├── .claude/
│   ├── settings.json                  # Permissions + convention-validation hooks
│   └── skills/
│       ├── execute/SKILL.md           # /execute slash command
│       ├── review-spec/SKILL.md       # /review-spec slash command
│       ├── phases/
│       │   ├── 01-setup.md            # 8-phase project scaffolding
│       │   └── 02-architecture.md     # Folder structure & conventions
│       ├── experts/
│       │   ├── tanstack-router.md     # Routing patterns
│       │   ├── tanstack-query.md      # Data fetching patterns
│       │   ├── shadcn-ui.md           # UI component patterns
│       │   ├── react-hook-form.md     # Form patterns
│       │   ├── lucide.md              # Icon patterns
│       │   └── testing.md            # Testing patterns
│       └── workflows/
│           ├── new-page.md            # Add a route/page
│           ├── new-feature.md         # Build a complete feature
│           └── new-form.md            # Build a form
```

## How It Works

1. **CLAUDE.md** is loaded automatically when Claude Code opens the project. It declares the stack, sets global conventions, and routes Claude to the right skill based on what you're doing.

2. **Expert skills** contain baked-in patterns for each library plus instructions to verify against live docs via context7 MCP before generating code.

3. **Workflow skills** orchestrate multiple experts for multi-step tasks (e.g., building a feature involves routing + queries + UI + testing).

4. **Convention hooks** in `settings.json` automatically validate that generated code follows the stack's patterns — route files use `createFileRoute`, query files use `queryOptions()`, test files use React Testing Library.

## Conventions Enforced

- TypeScript strict mode, no `any`
- Zod for all runtime validation
- TanStack Query for all server data — never raw `fetch` in components
- TanStack Router file-based routing — never manual route trees
- shadcn/ui over custom UI components
- Lucide for icons, individual imports only
- react-hook-form + Zod resolver for all forms
- Colocated tests with Vitest + React Testing Library
- Feature modules are self-contained in `src/features/`
- API calls go through a typed wrapper in `src/lib/api.ts`
