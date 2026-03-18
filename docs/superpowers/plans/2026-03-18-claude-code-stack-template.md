# Claude Code AI Expertise Template — Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create a zero-code Claude Code AI expertise package (CLAUDE.md, skills, hooks, settings) that turns Claude into an expert for the React + Vite + TypeScript + TanStack Router + TanStack Query + shadcn/ui + Lucide + react-hook-form stack.

**Architecture:** All files are markdown or JSON — no application code. CLAUDE.md is the master router that directs Claude to specialized skills organized in three tiers: phases (one-time setup), experts (per-library knowledge), and workflows (multi-library recipes). `.claude/settings.json` configures permissions and convention-validation hooks.

**Tech Stack:** Claude Code skills (markdown), Claude Code hooks (JSON in settings.json), context7 MCP for live doc verification.

**Spec:** `docs/superpowers/specs/2026-03-18-claude-code-stack-template-design.md`

---

## File Map

| File | Responsibility |
|------|---------------|
| `CLAUDE.md` | Master instructions: plugin gate, stack declaration, conventions, skill routing |
| `.claude/settings.json` | Permissions (context7, npm, git, vitest, eslint always allowed) + PostToolUse convention-validation hooks |
| `.claude/skills/phases/01-setup.md` | 8-phase checkpointed project scaffolding skill |
| `.claude/skills/phases/02-architecture.md` | Canonical folder structure and module organization patterns |
| `.claude/skills/experts/tanstack-router.md` | File-based routing expertise: createFileRoute, loaders, search params, navigation |
| `.claude/skills/experts/tanstack-query.md` | Data fetching expertise: queryOptions, useQuery, useMutation, cache patterns |
| `.claude/skills/experts/shadcn-ui.md` | UI component expertise: CLI usage, composition, theming, frontend-design integration |
| `.claude/skills/experts/react-hook-form.md` | Form expertise: useForm + zod resolver, shadcn Form components, type inference |
| `.claude/skills/experts/lucide.md` | Icon expertise: tree-shaken imports, size conventions, props |
| `.claude/skills/experts/testing.md` | Testing expertise: Vitest + RTL patterns, query priority, provider wrappers |
| `.claude/skills/workflows/new-page.md` | Workflow: add a route/page (router + query + UI + test) |
| `.claude/skills/workflows/new-feature.md` | Workflow: build a complete feature (schemas + queries + components + routes + tests) |
| `.claude/skills/workflows/new-form.md` | Workflow: build a form (zod + react-hook-form + shadcn + mutation + tests) |

---

### Task 1: Create `.claude/settings.json`

**Files:**
- Create: `.claude/settings.json`

This is the foundation — permissions and hooks that apply to the entire project.

**Note:** The spec's "Pre-commit quality gates" (tsc, eslint, vitest related) are NOT Claude Code hooks — there is no `PreCommit` hook event. These are standard git pre-commit hooks that get set up during project scaffolding (`01-setup.md` Phase 8) via husky or similar. The Claude Code hooks here are `PostToolUse` convention-validation hooks only.

- [ ] **Step 1: Create the settings.json file**

```json
{
  "permissions": {
    "allow": [
      "mcp__context7__resolve-library-id",
      "mcp__context7__query-docs",
      "Bash(npm:*)",
      "Bash(npx:*)",
      "Bash(git:*)",
      "Bash(vitest:*)",
      "Bash(npx vitest:*)",
      "Bash(eslint:*)",
      "Bash(npx eslint:*)",
      "Bash(node:*)",
      "Bash(tsc:*)",
      "Bash(npx tsc:*)"
    ]
  },
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.file_path // .tool_response.filePath // empty' | { read -r f; case \"$f\" in */src/routes/*.tsx) grep -q 'createFileRoute' \"$f\" || echo '{\"hookSpecificOutput\":{\"hookEventName\":\"PostToolUse\",\"additionalContext\":\"WARNING: Route file does not use createFileRoute. All route files in src/routes/ MUST export Route via createFileRoute. Fix this immediately.\"}}' ;; */features/*/queries/*.ts|*/features/*/queries/*.tsx) grep -q 'queryOptions' \"$f\" || echo '{\"hookSpecificOutput\":{\"hookEventName\":\"PostToolUse\",\"additionalContext\":\"WARNING: Query file does not use queryOptions(). All query files in features/*/queries/ MUST use queryOptions() factory pattern. Fix this immediately.\"}}' ;; *.test.tsx|*.test.ts) grep -q '@testing-library/react' \"$f\" || echo '{\"hookSpecificOutput\":{\"hookEventName\":\"PostToolUse\",\"additionalContext\":\"WARNING: Test file does not import from @testing-library/react. All component tests MUST use React Testing Library. Fix this immediately.\"}}' ;; esac; } 2>/dev/null || true",
            "timeout": 10,
            "statusMessage": "Validating conventions..."
          }
        ]
      }
    ]
  }
}
```

- [ ] **Step 2: Verify the JSON is valid**

Run: `cat .claude/settings.json | jq .`
Expected: Pretty-printed JSON with no errors

- [ ] **Step 3: Verify the hook command extracts file paths correctly**

Run: `echo '{"tool_name":"Write","tool_input":{"file_path":"/tmp/test.tsx"}}' | jq -r '.tool_input.file_path // .tool_response.filePath // empty'`
Expected: `/tmp/test.tsx`

- [ ] **Step 4: Commit (if git repo exists)**

```bash
git rev-parse --git-dir 2>/dev/null && git add .claude/settings.json && git commit -m "feat: add Claude Code settings with permissions and convention hooks" || echo "No git repo — skipping commit"
```

---

### Task 2: Create `CLAUDE.md`

**Files:**
- Create: `CLAUDE.md`

The master instruction file. Contains the plugin gate, stack declaration, global conventions, and skill routing table.

- [ ] **Step 1: Write CLAUDE.md**

```markdown
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
```

- [ ] **Step 2: Verify CLAUDE.md reads correctly**

Run: `head -5 CLAUDE.md`
Expected: Shows the title and first section header

- [ ] **Step 3: Commit (if git repo exists)**

```bash
git rev-parse --git-dir 2>/dev/null && git add CLAUDE.md && git commit -m "feat: add CLAUDE.md master instruction file" || echo "No git repo — skipping commit"
```

---

### Task 3: Create Phase Skills

**Files:**
- Create: `.claude/skills/phases/01-setup.md`
- Create: `.claude/skills/phases/02-architecture.md`

- [ ] **Step 1: Create directory structure**

Run: `mkdir -p .claude/skills/phases .claude/skills/experts .claude/skills/workflows`

- [ ] **Step 2: Write `01-setup.md`**

```markdown
---
name: project-setup
description: Scaffold a complete Vite + React + TypeScript + TanStack + shadcn project from scratch with checkpointed phases
---

# Project Setup — Scaffolding Skill

This skill scaffolds a complete project. Execute phases automatically with checkpoints.

## Before Starting

1. Verify all required plugins are available (see CLAUDE.md)
2. Confirm the directory is empty (or identify existing deps if drop-in)

## Phase 1 — Research (DO NOT SKIP)

Before installing anything, fetch the latest documentation for EVERY library:

```
mcp__context7__query-docs("/vitejs/vite", "create vite project setup React TypeScript")
mcp__context7__query-docs("/tanstack/router", "file-based routing Vite plugin setup installation")
mcp__context7__query-docs("/tanstack/query", "React Query setup QueryClient QueryClientProvider")
mcp__context7__query-docs("/shadcn/ui", "Vite React TypeScript setup CLI init")
mcp__context7__query-docs("/react-hook-form/documentation", "useForm setup zod resolver installation")
mcp__context7__query-docs("/vitest-dev/vitest", "setup React Testing Library jsdom configuration")
mcp__context7__query-docs("/websites/lucide_dev_guide_packages", "React installation usage")
```

From the results:
- Note the current stable versions
- Note any setup commands that differ from what's baked into this skill
- Note any breaking changes or deprecations
- Check Node.js version: `node --version` (Vite requires Node 18+)
- Detect package manager: check for `pnpm-lock.yaml`, `yarn.lock`, or `package-lock.json`. Default to `npm`.

**Use the context7 results to adjust the commands in subsequent phases.** The commands below are illustrative — always prefer what context7 returns.

## Phase 2 — Vite + React + TypeScript

```bash
npm create vite@latest . -- --template react-ts
npm install
```

Configure `tsconfig.json` — add path aliases:
```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

Configure `vite.config.ts` — add path alias resolution:
```typescript
import path from "path"

export default defineConfig({
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"),
    },
  },
  // ... existing config
})
```

Verify: `npm run build` should succeed.

## Phase 3 — TanStack Router

```bash
npm install @tanstack/react-router
npm install -D @tanstack/router-plugin
```

Update `vite.config.ts`:
```typescript
import { TanStackRouterVite } from '@tanstack/router-plugin/vite'

export default defineConfig({
  plugins: [
    TanStackRouterVite({
      routesDirectory: './src/routes',
      generatedRouteTree: './src/routeTree.gen.ts',
      autoCodeSplitting: true,
    }),
    react(),
  ],
  // ...
})
```

Create root route at `src/routes/__root.tsx` and index route at `src/routes/index.tsx`.
Wire the router in `src/main.tsx`.

**CHECKPOINT:** Run `npm run dev` — verify the dev server starts and the route tree is generated at `src/routeTree.gen.ts`.

## Phase 4 — TanStack Query

```bash
npm install @tanstack/react-query @tanstack/react-query-devtools
```

Create `QueryClient` with defaults and wrap the app with `QueryClientProvider` in the root route/layout. Add `ReactQueryDevtools` in development.

Verify: App still runs with devtools visible in browser.

## Phase 5 — shadcn/ui + Lucide + Tailwind

```bash
npx shadcn@latest init -t vite
```

Follow the CLI prompts. Then verify/update `components.json`:
- `iconLibrary` should be `"lucide"`
- `aliases` should use `@/` paths matching tsconfig

```bash
npm install lucide-react
```

**CHECKPOINT:** Install a test component (`npx shadcn@latest add button`), import it somewhere, verify it renders. Then remove the test usage (keep the component installed).

## Phase 6 — Forms

```bash
npm install react-hook-form @hookform/resolvers zod
```

No starter code needed. Dependencies are ready for use.

Install the shadcn form component:
```bash
npx shadcn@latest add form
```

## Phase 7 — Testing

```bash
npm install -D vitest @testing-library/react @testing-library/jest-dom @testing-library/user-event jsdom
```

Add test config to `vite.config.ts`:
```typescript
export default defineConfig({
  // ...existing
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: './src/test/setup.ts',
    css: true,
  },
})
```

Create `src/test/setup.ts`:
```typescript
import '@testing-library/jest-dom'
```

Add to `package.json` scripts:
```json
{
  "test": "vitest run",
  "test:watch": "vitest",
  "test:coverage": "vitest run --coverage"
}
```

**CHECKPOINT:** Create a minimal smoke test, run `npm test` — verify it passes. Remove the smoke test.

## Phase 8 — Finalize

Set up ESLint (should already be included from Vite template — verify and enhance if needed).

Set up git pre-commit hooks for quality gates:
```bash
npm install -D husky lint-staged
npx husky init
```

Configure `.husky/pre-commit`:
```bash
npx lint-staged
npx tsc --noEmit
npx vitest related --run
```

Configure `lint-staged` in `package.json`:
```json
{
  "lint-staged": {
    "*.{ts,tsx}": ["eslint --fix", "prettier --write"],
    "*.{json,md,css}": ["prettier --write"]
  }
}
```

Set up Prettier:
```bash
npm install -D prettier eslint-config-prettier
```

Create `.prettierrc`:
```json
{
  "semi": false,
  "singleQuote": true,
  "trailingComma": "all",
  "printWidth": 100,
  "tabWidth": 2
}
```

Ensure `.gitignore` covers: `node_modules`, `dist`, `.env`, `.env.local`, `*.gen.ts`.

Clean up any default Vite starter code — remove demo components, CSS, assets. Leave a minimal working app shell.

Create the project structure from `02-architecture.md`:
```bash
mkdir -p src/{features,components/shared,lib,hooks,test}
```

Initialize git and create initial commit:
```bash
git init
git add -A
git commit -m "feat: initial project setup — Vite + React + TS + TanStack + shadcn"
```

Print a summary of what was installed and the project structure.
```

- [ ] **Step 3: Write `02-architecture.md`**

```markdown
---
name: project-architecture
description: Canonical folder structure, module organization, and naming conventions for the Vite SPA + TanStack stack
---

# Project Architecture

## Folder Structure

```
src/
├── routes/                    # TanStack Router file-based routes
│   ├── __root.tsx             # Root layout route
│   ├── index.tsx              # Home page (/)
│   ├── about.tsx              # Static page (/about)
│   └── posts/
│       ├── index.tsx          # List page (/posts)
│       └── $postId.tsx        # Detail page (/posts/:postId)
│
├── features/                  # Feature modules — self-contained domains
│   └── [feature-name]/
│       ├── components/        # Feature-specific UI components
│       │   ├── FeatureList.tsx
│       │   └── FeatureList.test.tsx
│       ├── hooks/             # Feature-specific custom hooks
│       ├── queries/           # TanStack Query queryOptions factories
│       │   └── feature-queries.ts
│       ├── schemas/           # Zod validation schemas
│       │   └── feature-schema.ts
│       └── types/             # Feature-specific TypeScript types
│
├── components/
│   ├── ui/                    # shadcn/ui components (managed by CLI — DO NOT edit)
│   └── shared/                # App-wide reusable components built from shadcn primitives
│
├── lib/                       # Utilities and shared infrastructure
│   ├── api.ts                 # Typed fetch wrapper — base URL, auth, error handling
│   └── utils.ts               # General utility functions (cn() helper from shadcn, etc.)
│
├── hooks/                     # Shared custom hooks (not feature-specific)
│
└── test/                      # Test infrastructure
    └── setup.ts               # Vitest setup (imports jest-dom, etc.)
```

## Naming Conventions

| Type | Convention | Example |
|------|-----------|---------|
| Route files | kebab-case or `$param` | `about.tsx`, `$postId.tsx` |
| Components | PascalCase | `UserProfile.tsx` |
| Hooks | camelCase with `use` prefix | `useAuth.ts` |
| Query files | kebab-case with `-queries` suffix | `user-queries.ts` |
| Schema files | kebab-case with `-schema` suffix | `login-schema.ts` |
| Test files | Same name with `.test` suffix | `UserProfile.test.tsx` |
| Utilities | camelCase | `formatDate.ts` |

## Module Rules

1. **Features are self-contained.** A feature's components, hooks, queries, schemas, and types live together. One feature never imports from another feature's internals.

2. **Shared code goes in `lib/`, `hooks/`, or `components/shared/`.** If two features need the same thing, extract it to a shared location.

3. **`components/ui/` is sacred.** Managed by shadcn CLI. Never edit these files manually. Customization happens in `components/shared/` by composing `ui/` primitives.

4. **Queries use `queryOptions()` factories.** Every file in `queries/` exports functions that return `queryOptions({...})`. This enables type-safe prefetching in route loaders and reuse across components.

5. **Schemas are the single source of truth.** Form types, API response types — all inferred from Zod schemas via `z.infer<>`. Never duplicate types.

6. **Tests are colocated.** `Component.test.tsx` lives next to `Component.tsx`. Only test infrastructure (setup files, custom render helpers) lives in `src/test/`.

7. **API calls go through `lib/api.ts`.** The fetch wrapper handles base URL, auth headers, content-type, and error parsing. Query functions (`queryFn`, `mutationFn`) call it.
```

- [ ] **Step 4: Commit (if git repo exists)**

```bash
git rev-parse --git-dir 2>/dev/null && git add .claude/skills/phases/ && git commit -m "feat: add phase skills — setup scaffolding and architecture patterns" || echo "No git repo — skipping commit"
```

---

### Task 4: Create Expert Skills — TanStack Router + TanStack Query

**Files:**
- Create: `.claude/skills/experts/tanstack-router.md`
- Create: `.claude/skills/experts/tanstack-query.md`

- [ ] **Step 1: Write `tanstack-router.md`**

```markdown
---
name: tanstack-router-expert
description: TanStack Router expertise — file-based routing, createFileRoute, loaders, search params, navigation patterns
---

# TanStack Router Expert

## Context7 Verification — DO THIS FIRST

Before generating any routing code, query context7:

```
mcp__context7__query-docs("/tanstack/router", "<your specific question about the API you're about to use>")
```

Verify: `createFileRoute` API, loader patterns, search param validation, `<Link>` props, `useNavigate` API.

## Core Patterns

### File-Based Routing

Routes live in `src/routes/`. The TanStack Router Vite plugin auto-generates `routeTree.gen.ts`. Never edit the generated file.

**File naming:**
- `__root.tsx` — root layout (wraps all routes)
- `index.tsx` — index route for current directory
- `about.tsx` — static route (`/about`)
- `$paramName.tsx` — dynamic route (`/posts/$postId`)
- `posts/` directory — nested routes (`/posts/...`)
- `_layout.tsx` — layout route (wraps children, no URL segment)

### Route Definition

Every route file MUST export `Route` via `createFileRoute`:

```tsx
import { createFileRoute } from '@tanstack/react-router'

export const Route = createFileRoute('/posts/')({
  component: PostsPage,
})

function PostsPage() {
  return <div>Posts</div>
}
```

### Route Loaders with TanStack Query

Prefetch data in loaders using `queryClient.ensureQueryData()`:

```tsx
import { createFileRoute } from '@tanstack/react-router'
import { postsQueryOptions } from '@/features/posts/queries/post-queries'

export const Route = createFileRoute('/posts/')({
  loader: ({ context: { queryClient } }) =>
    queryClient.ensureQueryData(postsQueryOptions()),
  component: PostsPage,
})
```

### Search Params with Zod Validation

```tsx
import { createFileRoute } from '@tanstack/react-router'
import { z } from 'zod'

const searchSchema = z.object({
  page: z.number().default(1),
  search: z.string().optional(),
})

export const Route = createFileRoute('/posts/')({
  validateSearch: searchSchema,
  component: PostsPage,
})

function PostsPage() {
  const { page, search } = Route.useSearch()
  // ...
}
```

### Error and Loading States

Every route with a loader MUST define `pendingComponent` and `errorComponent`:

```tsx
export const Route = createFileRoute('/posts/')({
  loader: /* ... */,
  pendingComponent: () => <div>Loading...</div>,
  errorComponent: ({ error }) => <div>Error: {error.message}</div>,
  component: PostsPage,
})
```

### Navigation

Declarative (preferred):
```tsx
import { Link } from '@tanstack/react-router'

<Link to="/posts/$postId" params={{ postId: '123' }}>View Post</Link>
```

Programmatic:
```tsx
import { useNavigate } from '@tanstack/react-router'

const navigate = useNavigate()
navigate({ to: '/posts/$postId', params: { postId: '123' } })
```

## Don'ts

- Never create route files outside `src/routes/`
- Never manually edit `routeTree.gen.ts`
- Never use `useEffect` for data fetching — use route loaders
- Never use `window.location` for navigation — use `<Link>` or `useNavigate`
- Never use `react-router-dom` APIs — this project uses TanStack Router
```

- [ ] **Step 2: Write `tanstack-query.md`**

```markdown
---
name: tanstack-query-expert
description: TanStack Query expertise — queryOptions factories, useQuery, useMutation, cache invalidation, prefetching
---

# TanStack Query Expert

## Context7 Verification — DO THIS FIRST

Before generating any query/mutation code, query context7:

```
mcp__context7__query-docs("/tanstack/query", "<your specific question about the API you're about to use>")
```

Verify: `queryOptions` API, `useQuery`/`useMutation` signatures, `invalidateQueries` patterns, `QueryClient` config.

## Core Patterns

### queryOptions Factories

All queries are defined as `queryOptions()` factories in `src/features/[feature]/queries/`:

```typescript
import { queryOptions } from '@tanstack/react-query'
import { api } from '@/lib/api'

export const postsQueryOptions = () =>
  queryOptions({
    queryKey: ['posts'],
    queryFn: () => api.get<Post[]>('/posts'),
  })

export const postQueryOptions = (postId: string) =>
  queryOptions({
    queryKey: ['posts', postId],
    queryFn: () => api.get<Post>(`/posts/${postId}`),
  })
```

### Using Queries in Components

```tsx
import { useQuery } from '@tanstack/react-query'
import { postsQueryOptions } from '@/features/posts/queries/post-queries'

function PostsList() {
  const { data, isLoading, error } = useQuery(postsQueryOptions())

  if (isLoading) return <Skeleton />
  if (error) return <ErrorDisplay error={error} />

  return data.map(post => <PostCard key={post.id} post={post} />)
}
```

### Mutations with Cache Invalidation

```tsx
import { useMutation, useQueryClient } from '@tanstack/react-query'
import { api } from '@/lib/api'

function useCreatePost() {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: (newPost: CreatePostInput) =>
      api.post<Post>('/posts', newPost),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['posts'] })
    },
  })
}
```

### Prefetching in Route Loaders

```tsx
// In route file
loader: ({ context: { queryClient } }) =>
  queryClient.ensureQueryData(postsQueryOptions())
```

This ensures data is fetched before the route renders. The component then uses `useQuery` with the same `queryOptions` — it hits the cache instead of refetching.

### QueryClient Configuration

```typescript
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60, // 1 minute
      gcTime: 1000 * 60 * 5, // 5 minutes
      retry: 1,
      refetchOnWindowFocus: false,
    },
  },
})
```

Override `staleTime` and `gcTime` per-query based on data freshness needs.

### DevTools

Always include in development:
```tsx
import { ReactQueryDevtools } from '@tanstack/react-query-devtools'

// In root layout
<ReactQueryDevtools initialIsOpen={false} />
```

## Don'ts

- Never use `useEffect` + `useState` for server data — use `useQuery`
- Never put query keys as inline arrays — always through `queryOptions()`
- Never mutate query cache directly — use `invalidateQueries` or `setQueryData`
- Never call `fetch` in components — go through `api.ts` via `queryFn`
- Never create a new `QueryClient` per render — it must be stable
```

- [ ] **Step 3: Commit (if git repo exists)**

```bash
git rev-parse --git-dir 2>/dev/null && git add .claude/skills/experts/tanstack-router.md .claude/skills/experts/tanstack-query.md && git commit -m "feat: add TanStack Router and Query expert skills" || echo "No git repo — skipping commit"
```

---

### Task 5: Create Expert Skills — shadcn/ui + react-hook-form

**Files:**
- Create: `.claude/skills/experts/shadcn-ui.md`
- Create: `.claude/skills/experts/react-hook-form.md`

- [ ] **Step 1: Write `shadcn-ui.md`**

```markdown
---
name: shadcn-ui-expert
description: shadcn/ui expertise — CLI component installation, composition, theming, frontend-design skill integration
---

# shadcn/ui Expert

## Context7 Verification — DO THIS FIRST

Before using any shadcn component, query context7:

```
mcp__context7__query-docs("/shadcn/ui", "<component name> usage props variants examples")
```

Verify: Available components, current prop APIs, variant options, composition patterns.

## Core Patterns

### Installing Components

Always use the CLI — never copy code from the website:

```bash
npx shadcn@latest add button
npx shadcn@latest add card
npx shadcn@latest add dialog
npx shadcn@latest add input
npx shadcn@latest add form    # Required for react-hook-form integration
```

Install multiple at once:
```bash
npx shadcn@latest add button card dialog input form table
```

### Using Components

```tsx
import { Button } from '@/components/ui/button'
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card'

function MyComponent() {
  return (
    <Card>
      <CardHeader>
        <CardTitle>Title</CardTitle>
      </CardHeader>
      <CardContent>
        <Button variant="outline" size="sm">Click me</Button>
      </CardContent>
    </Card>
  )
}
```

### Building Custom Components

Build in `src/components/shared/` by composing shadcn primitives:

```tsx
// src/components/shared/StatusBadge.tsx
import { Badge } from '@/components/ui/badge'
import { cn } from '@/lib/utils'

interface StatusBadgeProps {
  status: 'active' | 'inactive' | 'pending'
}

export function StatusBadge({ status }: StatusBadgeProps) {
  return (
    <Badge
      variant={status === 'active' ? 'default' : 'secondary'}
      className={cn(status === 'inactive' && 'opacity-50')}
    >
      {status}
    </Badge>
  )
}
```

### Theming

shadcn uses CSS variables for theming. Customize in `src/index.css` (or equivalent globals file):

```css
:root {
  --primary: 222.2 47.4% 11.2%;
  --primary-foreground: 210 40% 98%;
  /* etc. */
}
```

Always use semantic tokens (`bg-primary`, `text-muted-foreground`), never hardcoded colors.

### Icons with Lucide

shadcn is configured with `iconLibrary: "lucide"`. Icons are used alongside shadcn components:

```tsx
import { Button } from '@/components/ui/button'
import { Plus } from 'lucide-react'

<Button>
  <Plus size={16} />
  Add Item
</Button>
```

### Frontend Design Integration

When building UI, ALWAYS invoke the **frontend-design** skill (`frontend-design:frontend-design`) for production-grade output. Don't generate generic-looking UI — use the skill to ensure high design quality.

## Don'ts

- Never edit files in `src/components/ui/` manually — they're managed by the CLI
- Never copy component code from the shadcn website — use `npx shadcn@latest add`
- Never use raw HTML elements (`<button>`, `<input>`, `<table>`) when shadcn has an equivalent
- Never hardcode colors — use CSS variable tokens
- Never skip the frontend-design skill when building visible UI
```

- [ ] **Step 2: Write `react-hook-form.md`**

```markdown
---
name: react-hook-form-expert
description: react-hook-form expertise — useForm + Zod resolver, shadcn Form integration, type inference from schemas
---

# react-hook-form Expert

## Context7 Verification — DO THIS FIRST

Before generating form code, query context7:

```
mcp__context7__query-docs("/react-hook-form/documentation", "<your specific question — useForm options, field registration, resolver setup>")
```

Verify: `useForm` options, `zodResolver` integration, `FormField`/`FormItem` component API from shadcn.

## Core Patterns

### Form Architecture

Every form follows this structure:

1. **Schema** in `src/features/[feature]/schemas/` — Zod schema defines shape + validation
2. **Type** inferred from schema via `z.infer<>` — never duplicate
3. **Component** uses `useForm` with `zodResolver` — paired with shadcn `<Form>` components

### Zod Schema (Single Source of Truth)

```typescript
// src/features/auth/schemas/login-schema.ts
import { z } from 'zod'

export const loginSchema = z.object({
  email: z.string().email('Invalid email address'),
  password: z.string().min(8, 'Password must be at least 8 characters'),
})

export type LoginFormData = z.infer<typeof loginSchema>
```

### Form Component with shadcn

```tsx
// src/features/auth/components/LoginForm.tsx
import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { loginSchema, type LoginFormData } from '@/features/auth/schemas/login-schema'
import {
  Form,
  FormControl,
  FormField,
  FormItem,
  FormLabel,
  FormMessage,
} from '@/components/ui/form'
import { Input } from '@/components/ui/input'
import { Button } from '@/components/ui/button'

export function LoginForm({ onSubmit }: { onSubmit: (data: LoginFormData) => void }) {
  const form = useForm<LoginFormData>({
    resolver: zodResolver(loginSchema),
    defaultValues: {
      email: '',
      password: '',
    },
  })

  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-4">
        <FormField
          control={form.control}
          name="email"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Email</FormLabel>
              <FormControl>
                <Input type="email" placeholder="you@example.com" {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />
        <FormField
          control={form.control}
          name="password"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Password</FormLabel>
              <FormControl>
                <Input type="password" {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />
        <Button type="submit" disabled={form.formState.isSubmitting}>
          {form.formState.isSubmitting ? 'Signing in...' : 'Sign In'}
        </Button>
      </form>
    </Form>
  )
}
```

### Wiring with TanStack Query Mutation

```tsx
import { useMutation } from '@tanstack/react-query'
import { api } from '@/lib/api'
import { LoginForm } from './LoginForm'
import type { LoginFormData } from '@/features/auth/schemas/login-schema'

function LoginPage() {
  const loginMutation = useMutation({
    mutationFn: (data: LoginFormData) => api.post('/auth/login', data),
    onSuccess: () => {
      // navigate, show toast, etc.
    },
  })

  return <LoginForm onSubmit={(data) => loginMutation.mutate(data)} />
}
```

### Default Values

Always provide typed `defaultValues`:

```typescript
const form = useForm<FormData>({
  resolver: zodResolver(schema),
  defaultValues: {
    name: '',
    email: '',
    age: 0,
  },
})
```

## Don'ts

- Never use uncontrolled forms without react-hook-form
- Never define validation rules inline (`{ required: true }`) — always Zod schemas
- Never manually type form data — always `z.infer<typeof schema>`
- Never use `<form>` without shadcn's `<Form>` wrapper
- Never skip `defaultValues` — react-hook-form needs them for proper reset/dirty tracking
```

- [ ] **Step 3: Commit (if git repo exists)**

```bash
git rev-parse --git-dir 2>/dev/null && git add .claude/skills/experts/shadcn-ui.md .claude/skills/experts/react-hook-form.md && git commit -m "feat: add shadcn/ui and react-hook-form expert skills" || echo "No git repo — skipping commit"
```

---

### Task 6: Create Expert Skills — Lucide + Testing

**Files:**
- Create: `.claude/skills/experts/lucide.md`
- Create: `.claude/skills/experts/testing.md`

- [ ] **Step 1: Write `lucide.md`**

```markdown
---
name: lucide-expert
description: Lucide React icon expertise — tree-shaken imports, size conventions, prop customization
---

# Lucide React Expert

## Context7 Verification

When looking for a specific icon or checking prop API:

```
mcp__context7__query-docs("/websites/lucide_dev_guide_packages", "available icons for <description> OR icon component props")
```

## Core Patterns

### Importing Icons

Always import individual icons — never from a barrel:

```tsx
import { Search, Plus, Trash2, ChevronRight } from 'lucide-react'
```

Each icon is a React component that renders inline SVG. Tree-shaking ensures only imported icons ship to the bundle.

### Size Conventions

Use consistent sizes across the app:

| Context | Size | Example |
|---------|------|---------|
| Inline with text | 16 | `<Search size={16} />` |
| Inside buttons | 16 | `<Button><Plus size={16} /> Add</Button>` |
| Standalone / page icons | 24 | `<Settings size={24} />` |
| Empty states / hero | 48+ | `<Inbox size={48} />` |

### Props

```tsx
<Search
  size={20}           // Width and height in px
  color="currentColor" // Inherits text color by default
  strokeWidth={2}      // Default stroke width
  className="shrink-0" // Tailwind classes work
/>
```

### Usage with shadcn

Icons go inside shadcn components, typically before text:

```tsx
import { Button } from '@/components/ui/button'
import { Plus } from 'lucide-react'

<Button variant="outline" size="sm">
  <Plus size={16} />
  New Item
</Button>
```

For icon-only buttons:

```tsx
<Button variant="ghost" size="icon">
  <Trash2 size={16} />
</Button>
```

## Don'ts

- Never import all icons (`import * as icons from 'lucide-react'`)
- Never use raw SVGs when a Lucide icon exists for the concept
- Never use inconsistent sizes — follow the convention table above
- Never use hardcoded color values — let icons inherit via `currentColor`
```

- [ ] **Step 2: Write `testing.md`**

```markdown
---
name: testing-expert
description: Vitest + React Testing Library expertise — component testing patterns, query priority, provider wrappers, TDD approach
---

# Testing Expert

## Context7 Verification

Before writing test setup or using unfamiliar testing APIs:

```
mcp__context7__query-docs("/vitest-dev/vitest", "<specific question about test config, matchers, mocking>")
```

## Core Patterns

### Test File Location

Colocated — test files live next to source files:

```
src/features/posts/components/
  PostCard.tsx
  PostCard.test.tsx
```

Only test infrastructure lives in `src/test/`:
- `setup.ts` — global setup (jest-dom imports)
- `test-utils.tsx` — custom render with providers

### Custom Render with Providers

Create a test render function that wraps components with required providers:

```tsx
// src/test/test-utils.tsx
import { render, type RenderOptions } from '@testing-library/react'
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
import {
  RouterProvider,
  createRouter,
  createRootRoute,
  createRoute,
  createMemoryHistory,
} from '@tanstack/react-router'

function createTestQueryClient() {
  return new QueryClient({
    defaultOptions: {
      queries: { retry: false, gcTime: 0 },
      mutations: { retry: false },
    },
  })
}

export function renderWithProviders(
  ui: React.ReactElement,
  options?: Omit<RenderOptions, 'wrapper'>,
) {
  const queryClient = createTestQueryClient()

  function Wrapper({ children }: { children: React.ReactNode }) {
    return (
      <QueryClientProvider client={queryClient}>
        {children}
      </QueryClientProvider>
    )
  }

  return {
    ...render(ui, { wrapper: Wrapper, ...options }),
    queryClient,
  }
}
```

### Query Priority

Always use the most accessible query available:

1. `getByRole` — buttons, headings, links, form elements
2. `getByLabelText` — form inputs with labels
3. `getByPlaceholderText` — inputs with placeholders
4. `getByText` — visible text content
5. `getByTestId` — LAST RESORT only

```tsx
// GOOD
screen.getByRole('button', { name: /submit/i })
screen.getByRole('heading', { name: /welcome/i })
screen.getByLabelText(/email/i)

// BAD — only if no accessible query works
screen.getByTestId('submit-button')
```

### User Events

Always use `userEvent` over `fireEvent`:

```tsx
import userEvent from '@testing-library/user-event'

test('submits the form', async () => {
  const user = userEvent.setup()
  renderWithProviders(<LoginForm onSubmit={mockSubmit} />)

  await user.type(screen.getByLabelText(/email/i), 'test@example.com')
  await user.type(screen.getByLabelText(/password/i), 'password123')
  await user.click(screen.getByRole('button', { name: /sign in/i }))

  expect(mockSubmit).toHaveBeenCalledWith({
    email: 'test@example.com',
    password: 'password123',
  })
})
```

### What to Test

**Routes:** Smoke render test — mounts without crashing.

**Components:** Renders correct content, responds to user interaction, shows correct states (loading, error, empty).

**Forms:** Validation errors display, successful submission calls handler, disabled during submission.

**Queries/Mutations:** Test through the component that uses them, not in isolation.

### Example: Component Test

```tsx
import { describe, it, expect, vi } from 'vitest'
import { screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import { renderWithProviders } from '@/test/test-utils'
import { PostCard } from './PostCard'

describe('PostCard', () => {
  const mockPost = { id: '1', title: 'Test Post', body: 'Test body' }

  it('renders post title and body', () => {
    renderWithProviders(<PostCard post={mockPost} />)

    expect(screen.getByRole('heading', { name: /test post/i })).toBeInTheDocument()
    expect(screen.getByText(/test body/i)).toBeInTheDocument()
  })

  it('calls onDelete when delete button is clicked', async () => {
    const user = userEvent.setup()
    const onDelete = vi.fn()
    renderWithProviders(<PostCard post={mockPost} onDelete={onDelete} />)

    await user.click(screen.getByRole('button', { name: /delete/i }))

    expect(onDelete).toHaveBeenCalledWith('1')
  })
})
```

### Example: Form Validation Test

```tsx
import { describe, it, expect, vi } from 'vitest'
import { screen, waitFor } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import { renderWithProviders } from '@/test/test-utils'
import { LoginForm } from './LoginForm'

describe('LoginForm', () => {
  it('shows validation errors for invalid input', async () => {
    const user = userEvent.setup()
    renderWithProviders(<LoginForm onSubmit={vi.fn()} />)

    await user.click(screen.getByRole('button', { name: /sign in/i }))

    await waitFor(() => {
      expect(screen.getByText(/invalid email/i)).toBeInTheDocument()
    })
  })

  it('submits with valid data', async () => {
    const user = userEvent.setup()
    const onSubmit = vi.fn()
    renderWithProviders(<LoginForm onSubmit={onSubmit} />)

    await user.type(screen.getByLabelText(/email/i), 'valid@email.com')
    await user.type(screen.getByLabelText(/password/i), 'password123')
    await user.click(screen.getByRole('button', { name: /sign in/i }))

    await waitFor(() => {
      expect(onSubmit).toHaveBeenCalledWith({
        email: 'valid@email.com',
        password: 'password123',
      })
    })
  })
})
```

## Don'ts

- Never test implementation details (internal state, private methods)
- Never use `getByTestId` as first choice — use accessible queries
- Never skip provider wrappers — tests must mirror real rendering context
- Never use `jest` imports — always `vitest` (`vi.fn()`, `vi.mock()`, etc.)
- Never use `fireEvent` when `userEvent` can do the job
- Never test library internals (don't test that React Query caches — test your component's behavior)
```

- [ ] **Step 3: Commit (if git repo exists)**

```bash
git rev-parse --git-dir 2>/dev/null && git add .claude/skills/experts/lucide.md .claude/skills/experts/testing.md && git commit -m "feat: add Lucide and testing expert skills" || echo "No git repo — skipping commit"
```

---

### Task 7: Create Workflow Skills

**Files:**
- Create: `.claude/skills/workflows/new-page.md`
- Create: `.claude/skills/workflows/new-feature.md`
- Create: `.claude/skills/workflows/new-form.md`

- [ ] **Step 1: Write `new-page.md`**

```markdown
---
name: new-page
description: Workflow for adding a new route/page — creates route file, query, component with UI, and smoke test
---

# Workflow: Add a New Page

Follow these steps in order when creating a new page/route.

## Prerequisites

Load these expert skills for reference:
- `.claude/skills/experts/tanstack-router.md`
- `.claude/skills/experts/tanstack-query.md` (if page fetches data)
- `.claude/skills/experts/shadcn-ui.md`

## Steps

### 1. Determine Route Path and File Location

Map the desired URL to a route file in `src/routes/`:

| URL | File |
|-----|------|
| `/about` | `src/routes/about.tsx` |
| `/settings` | `src/routes/settings.tsx` |
| `/posts` | `src/routes/posts/index.tsx` |
| `/posts/:id` | `src/routes/posts/$postId.tsx` |

### 2. Define Search Params (if needed)

If the page accepts search/query params, define a Zod schema:

```typescript
const searchSchema = z.object({
  page: z.number().default(1),
  filter: z.string().optional(),
})
```

### 3. Create Query (if page fetches data)

Create `queryOptions` factory in `src/features/[feature]/queries/`:

```typescript
export const pageDataQueryOptions = (params?: SearchParams) =>
  queryOptions({
    queryKey: ['page-name', params],
    queryFn: () => api.get('/endpoint', { params }),
  })
```

### 4. Create Route File

Use `createFileRoute` with loader (if data), search validation (if params):

```tsx
export const Route = createFileRoute('/path/')({
  validateSearch: searchSchema,  // if search params
  loader: ({ context: { queryClient } }) =>
    queryClient.ensureQueryData(pageDataQueryOptions()),
  pendingComponent: () => <LoadingSkeleton />,
  errorComponent: ({ error }) => <ErrorDisplay error={error} />,
  component: PageComponent,
})
```

### 5. Build the Component

Invoke the **frontend-design** skill (`frontend-design:frontend-design`) to build the page UI with shadcn/ui components and Lucide icons. Do not generate generic-looking UI.

### 6. Write Smoke Test

Create `[PageName].test.tsx` colocated with the component or route:

```tsx
import { describe, it, expect } from 'vitest'
import { screen } from '@testing-library/react'
import { renderWithProviders } from '@/test/test-utils'

describe('PageName', () => {
  it('renders without crashing', () => {
    renderWithProviders(<PageComponent />)
    expect(screen.getByRole('heading')).toBeInTheDocument()
  })
})
```

### 7. Verify

- Run the dev server and navigate to the new route
- Confirm the route appears in `routeTree.gen.ts`
- Run `npm test` to verify the smoke test passes
```

- [ ] **Step 2: Write `new-feature.md`**

```markdown
---
name: new-feature
description: Workflow for building a complete feature — schemas, queries, components, routes, and tests
---

# Workflow: Build a Complete Feature

Follow these steps in order when building a new feature end-to-end.

## Prerequisites

Load these expert skills for reference:
- `.claude/skills/experts/tanstack-router.md`
- `.claude/skills/experts/tanstack-query.md`
- `.claude/skills/experts/shadcn-ui.md`
- `.claude/skills/experts/react-hook-form.md` (if feature has forms)
- `.claude/skills/experts/testing.md`

Use **superpowers TDD** workflow if available.

## Steps

### 1. Create Feature Directory

```bash
mkdir -p src/features/[feature-name]/{components,hooks,queries,schemas,types}
```

### 2. Define Schemas

Create Zod schemas in `src/features/[feature]/schemas/`:

```typescript
// src/features/[feature]/schemas/[feature]-schema.ts
import { z } from 'zod'

export const featureSchema = z.object({
  // ... define shape
})

export type FeatureData = z.infer<typeof featureSchema>

// If there's a form, create a form-specific schema:
export const createFeatureSchema = z.object({
  // ... form fields
})

export type CreateFeatureInput = z.infer<typeof createFeatureSchema>
```

### 3. Define Queries

Create `queryOptions` factories in `src/features/[feature]/queries/`:

```typescript
// src/features/[feature]/queries/[feature]-queries.ts
import { queryOptions } from '@tanstack/react-query'
import { api } from '@/lib/api'
import type { FeatureData } from '../schemas/[feature]-schema'

export const featureListQueryOptions = () =>
  queryOptions({
    queryKey: ['features'],
    queryFn: () => api.get<FeatureData[]>('/features'),
  })

export const featureDetailQueryOptions = (id: string) =>
  queryOptions({
    queryKey: ['features', id],
    queryFn: () => api.get<FeatureData>(`/features/${id}`),
  })
```

### 4. Build Components

Create components in `src/features/[feature]/components/`.

Invoke the **frontend-design** skill (`frontend-design:frontend-design`) for every visible component. Build with shadcn/ui primitives and Lucide icons.

Typical components for a feature:
- `FeatureList.tsx` — list/table view
- `FeatureCard.tsx` — individual item display
- `FeatureForm.tsx` — create/edit form (see `new-form` workflow)
- `FeatureDetail.tsx` — detail view

### 5. Create Hooks (if needed)

Custom hooks in `src/features/[feature]/hooks/` for reusable logic:

```typescript
// src/features/[feature]/hooks/useFeatureMutations.ts
export function useCreateFeature() {
  const queryClient = useQueryClient()
  return useMutation({
    mutationFn: (data: CreateFeatureInput) => api.post('/features', data),
    onSuccess: () => queryClient.invalidateQueries({ queryKey: ['features'] }),
  })
}
```

### 6. Wire Routes

Create route files in `src/routes/` following the TanStack Router expert patterns:
- List page: `src/routes/features/index.tsx`
- Detail page: `src/routes/features/$featureId.tsx`

Each route uses loaders to prefetch via `queryClient.ensureQueryData()`.

### 7. Write Tests

Follow the testing expert patterns. At minimum:
- Smoke test for each component
- User interaction tests for interactive components
- Form validation + submission tests (if forms exist)
- Run: `npm test`

### 8. Verify

- Run `npm run dev` and test all routes manually
- Run `npm test` to verify all tests pass
- Run `npx tsc --noEmit` to verify type safety
```

- [ ] **Step 3: Write `new-form.md`**

```markdown
---
name: new-form
description: Workflow for building a form — Zod schema, react-hook-form + shadcn Form, mutation, loading/error states, tests
---

# Workflow: Build a Form

Follow these steps in order when creating a form with validation.

## Prerequisites

Load these expert skills for reference:
- `.claude/skills/experts/react-hook-form.md`
- `.claude/skills/experts/shadcn-ui.md`
- `.claude/skills/experts/tanstack-query.md`
- `.claude/skills/experts/testing.md`

## Steps

### 1. Define Zod Schema

Create in `src/features/[feature]/schemas/`:

```typescript
// src/features/[feature]/schemas/[form-name]-schema.ts
import { z } from 'zod'

export const formSchema = z.object({
  name: z.string().min(1, 'Name is required'),
  email: z.string().email('Invalid email address'),
  // ... add fields with validation messages
})

export type FormData = z.infer<typeof formSchema>
```

### 2. Install Required shadcn Components

Ensure the form-related shadcn components are installed:

```bash
npx shadcn@latest add form input label button
# Add more as needed: select, textarea, checkbox, radio-group, switch
```

### 3. Create Form Component

Build the form in `src/features/[feature]/components/`:

Use `useForm` with `zodResolver`, shadcn `<Form>` wrapper, and `<FormField>` for each input.

See the react-hook-form expert skill for the full pattern.

Invoke **frontend-design** skill for the form's visual design.

### 4. Wire Mutation

Connect the form to an API endpoint via `useMutation`:

```tsx
const mutation = useMutation({
  mutationFn: (data: FormData) => api.post('/endpoint', data),
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ['relevant-key'] })
    // navigate, show toast, close dialog, etc.
  },
})
```

### 5. Handle Loading and Error States

- Disable submit button during `form.formState.isSubmitting` or `mutation.isPending`
- Show mutation errors (server-side) above the form or as a toast
- Show field-level validation errors via `<FormMessage />`

### 6. Write Tests

Test three scenarios at minimum:

**Validation errors:**
```tsx
it('shows validation errors for empty required fields', async () => {
  const user = userEvent.setup()
  renderWithProviders(<MyForm onSubmit={vi.fn()} />)
  await user.click(screen.getByRole('button', { name: /submit/i }))
  await waitFor(() => {
    expect(screen.getByText(/name is required/i)).toBeInTheDocument()
  })
})
```

**Successful submission:**
```tsx
it('calls onSubmit with valid data', async () => {
  const user = userEvent.setup()
  const onSubmit = vi.fn()
  renderWithProviders(<MyForm onSubmit={onSubmit} />)
  await user.type(screen.getByLabelText(/name/i), 'John')
  await user.type(screen.getByLabelText(/email/i), 'john@example.com')
  await user.click(screen.getByRole('button', { name: /submit/i }))
  await waitFor(() => {
    expect(onSubmit).toHaveBeenCalledWith({ name: 'John', email: 'john@example.com' })
  })
})
```

**Disabled during submission:**
```tsx
it('disables submit button while submitting', async () => {
  // ... trigger submit, verify button is disabled
})
```

### 7. Verify

- Run `npm test` — all form tests pass
- Run `npx tsc --noEmit` — no type errors
- Test in browser — fill form, submit, verify validation, verify success flow
```

- [ ] **Step 4: Commit (if git repo exists)**

```bash
git rev-parse --git-dir 2>/dev/null && git add .claude/skills/workflows/ && git commit -m "feat: add workflow skills — new-page, new-feature, new-form" || echo "No git repo — skipping commit"
```

---

### Task 8: Final Verification

- [ ] **Step 1: Verify all files exist**

Run: `find . -name "*.md" -o -name "*.json" | grep -E '(CLAUDE|skills|settings)' | sort`

Expected output (13 files):
```
./CLAUDE.md
./.claude/settings.json
./.claude/skills/experts/lucide.md
./.claude/skills/experts/react-hook-form.md
./.claude/skills/experts/shadcn-ui.md
./.claude/skills/experts/tanstack-query.md
./.claude/skills/experts/tanstack-router.md
./.claude/skills/experts/testing.md
./.claude/skills/phases/01-setup.md
./.claude/skills/phases/02-architecture.md
./.claude/skills/workflows/new-feature.md
./.claude/skills/workflows/new-form.md
./.claude/skills/workflows/new-page.md
```

- [ ] **Step 2: Validate settings.json syntax**

Run: `jq '.' .claude/settings.json`
Expected: Valid JSON output

- [ ] **Step 3: Verify no application code exists**

Run: `find . -name "*.ts" -o -name "*.tsx" -o -name "*.js" -o -name "*.jsx" | grep -v node_modules | grep -v '.gen.'`
Expected: No output (no application code files)

- [ ] **Step 4: Final commit (if git repo exists and uncommitted changes)**

```bash
git rev-parse --git-dir 2>/dev/null && git add -A && git diff --cached --quiet || git commit -m "chore: final verification — all template files in place" || echo "No git repo or no changes — skipping commit"
```
