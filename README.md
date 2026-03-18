# Vite SPA + TanStack — Claude Code AI Template

An AI expertise package that makes Claude Code an expert in the **React + Vite + TypeScript + TanStack Router + TanStack Query + shadcn/ui + Lucide + react-hook-form** stack.

Copy it into a new or existing project and Claude generates correct, consistent code for this stack from the start.

## What This Is

- **A set of instructions, skills, and hooks** — all markdown and JSON, zero application code
- **A knowledge layer** — teaches Claude the patterns, conventions, and best practices for this specific stack
- **A workflow engine** — provides step-by-step skills for scaffolding projects, building features, creating forms, and adding pages
- **A quality gate** — hooks validate that generated code follows conventions automatically

## What This Is NOT

- **Not a starter template** — there is no `src/`, no `package.json`, no application code to run
- **Not a component library** — it tells Claude *how* to use shadcn/ui, it doesn't provide components
- **Not a runtime dependency** — nothing here ships to production or runs in a browser
- **Not a replacement for documentation** — it uses context7 MCP to verify against live docs at generation time

## Requirements

Three Claude Code plugins must be installed:

- **[superpowers](https://github.com/anthropics/claude-code-plugins)** — planning, TDD, debugging workflows
- **frontend-design** — production-grade UI generation
- **context7 MCP** — live documentation lookup

## Usage

```bash
# New project
cp -r vite-spa-tanstack/ my-new-app/
cd my-new-app && claude
# Then say: "Set up the project"

# Existing project
cp -r vite-spa-tanstack/{CLAUDE.md,.claude} my-existing-app/
cd my-existing-app && claude
```

See [USER_GUIDE.md](USER_GUIDE.md) for full documentation.

## License

MIT
