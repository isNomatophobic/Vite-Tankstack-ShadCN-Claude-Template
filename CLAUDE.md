# Vite SPA + TanStack Stack — Claude Code Instructions

## Required Plugins — CHECK FIRST

Before doing ANY work, verify these plugins/tools are available. If any are missing, STOP and instruct the user to install them.

### superpowers
Check: Can you invoke skills like `superpowers:brainstorming`?
If missing: "You need the superpowers plugin. Install it via Claude Code plugin marketplace or see https://github.com/anthropics/claude-code-plugins"

### frontend-design
Check: Can you invoke the `frontend-design:frontend-design` skill?
If missing: "You need the frontend-design plugin. Install it via Claude Code plugin marketplace."

### context7 MCP
Check: Can you call `mcp__context7__resolve-library-id`?
If missing: "You need the context7 MCP server. Add it to your MCP configuration:
```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp@latest"]
    }
  }
}
```"

---

## Stack

This project uses the following stack. Always verify APIs against context7 before generating code.

| Library | context7 ID | Purpose |
|---------|-------------|---------|
| React | — | UI library |
| Vite | `/vitejs/vite` | Build tool + dev server |
| TypeScript | — | Type safety |
| TanStack Router | `/tanstack/router` | File-based routing |
| TanStack Query | `/tanstack/query` | Server state + data fetching |
| shadcn/ui | `/shadcn/ui` | UI component library |
| Lucide React | `/websites/lucide_dev_guide_packages` | Icons |
| react-hook-form | `/react-hook-form/documentation` | Form management |
| Zod | — | Schema validation |
| Vitest | `/vitest-dev/vitest` | Testing framework |
| React Testing Library | — | Component testing |

**Context7 usage:** Before generating code that uses any library with a context7 ID above, call `mcp__context7__query-docs` with that ID to verify current API signatures. If context7 is unreachable, fall back to the baked-in patterns in expert skills and warn the user.

---

## Global Conventions

- **TypeScript:** strict mode, no `any`, no `@ts-ignore`
- **Validation:** Zod for ALL runtime validation — forms, API responses, search params
- **Data fetching:** All server data through TanStack Query — never raw `fetch` in components, never `useEffect` + `useState` for server data
- **Routing:** TanStack Router file-based routing — never manual route trees, never edit `routeTree.gen.ts`
- **UI:** shadcn/ui components over custom UI — compose, don't reinvent. Always use the **frontend-design** skill when building UI.
- **Icons:** Lucide React, individual imports only (`import { Icon } from 'lucide-react'`), tree-shaken
- **Forms:** react-hook-form + Zod resolver, always. Types inferred via `z.infer<>`, never duplicated.
- **Tests:** Colocated (`Component.test.tsx` next to `Component.tsx`), Vitest + React Testing Library
- **State:** React context + `useReducer` for client/UI state. TanStack Query for server state. No additional state libraries.
- **Env vars:** `import.meta.env` with Vite's `VITE_` prefix. `.env` for local config, never commit secrets.
- **API client:** Typed fetch wrapper in `src/lib/api.ts` — single source for base URL, auth headers, error handling. All `queryFn`/`mutationFn` call through this.
- **Navigation:** TanStack Router `<Link>` for declarative, `useNavigate` for programmatic. Never `window.location`.
- **Package manager:** `npm` by default. If dropped into existing project, detect lock files (`pnpm-lock.yaml` → pnpm, `yarn.lock` → yarn, `package-lock.json` → npm).
- **Skills:** Always use **superpowers** skills for planning, TDD, debugging workflows.

---

## Skill Routing

Based on what you're doing, invoke the appropriate skill:

| Intent | Skill(s) |
|--------|----------|
| "Set up the project" / new empty project | `.claude/skills/phases/01-setup.md` |
| "Create a page" / "add a route" | `.claude/skills/workflows/new-page.md` |
| "Build a feature" / "add [feature]" | `.claude/skills/workflows/new-feature.md` |
| "Build a form" / "add a form" | `.claude/skills/workflows/new-form.md` |
| Working with routing code | `.claude/skills/experts/tanstack-router.md` |
| Working with data fetching / queries | `.claude/skills/experts/tanstack-query.md` |
| Working with UI components | `.claude/skills/experts/shadcn-ui.md` |
| Working with forms | `.claude/skills/experts/react-hook-form.md` |
| Working with icons | `.claude/skills/experts/lucide.md` |
| Writing or fixing tests | `.claude/skills/experts/testing.md` |
| Understanding project structure | `.claude/skills/phases/02-architecture.md` |

When multiple skills apply, load the workflow skill first (it orchestrates the experts).

---

## Drop-in Mode

When this template is dropped into an existing project:

- Apply conventions to **new code only** — don't refactor existing code unprompted
- Adapt architecture skill to existing structure — describe ideal but work with what exists
- Detect and warn about conflicting libraries (e.g., React Router vs TanStack Router, Formik vs react-hook-form) — ask user whether to coexist or migrate, never silently overwrite
- Detect package manager from lock files and use it
- Skip already-installed dependencies during setup
