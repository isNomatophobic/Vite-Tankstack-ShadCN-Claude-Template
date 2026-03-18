# Claude Code AI Expertise Template — Vite SPA + TanStack Stack

**Date:** 2026-03-18
**Status:** Approved
**Scope:** Zero-code Claude Code setup that turns Claude into a stack expert for React + Vite + TypeScript + TanStack Router + TanStack Query + shadcn/ui + Lucide + react-hook-form

---

## 1. Purpose

An AI-focused template containing only configuration, skills, hooks, and markdown files — no application code. When copied to a new directory, it gives Claude Code deep expertise in this specific stack, enabling it to:

- Scaffold a complete project from scratch (automatic with checkpoints)
- Generate code that follows current best practices (verified via context7 MCP)
- Apply consistent patterns across all libraries in the stack
- Be dropped into existing projects to upgrade Claude's knowledge

## 2. Required Plugins

Claude MUST verify these are available before any task. If missing, stop and instruct the user to set them up:

| Plugin | Purpose |
|--------|---------|
| **superpowers** | Skills, brainstorming, TDD, debugging, planning workflows |
| **frontend-design** | Production-grade UI generation with high design quality |
| **context7 MCP** | Live documentation lookup for all stack libraries |

The plugin check goes at the top of CLAUDE.md so it's the first thing evaluated.

## 3. File Structure

```
vite-spa-tanstack/
├── CLAUDE.md                              # Master instruction file
├── .claude/
│   ├── settings.json                      # Hooks + permissions
│   └── skills/
│       ├── phases/
│       │   ├── 01-setup.md                # Scaffolding skill (8 phases)
│       │   └── 02-architecture.md         # Folder structure & patterns
│       ├── experts/
│       │   ├── tanstack-router.md         # File-based routing expertise
│       │   ├── tanstack-query.md          # Data fetching expertise
│       │   ├── shadcn-ui.md               # UI component expertise
│       │   ├── react-hook-form.md         # Form handling expertise
│       │   ├── lucide.md                  # Icon usage expertise
│       │   └── testing.md                 # Vitest + RTL expertise
│       └── workflows/
│           ├── new-page.md                # Add a route/page
│           ├── new-feature.md             # Build a complete feature
│           └── new-form.md                # Build a form with validation
```

## 4. CLAUDE.md — Master File

### 4.1 Required Plugin Gate

First section of CLAUDE.md. Before any task, Claude checks for superpowers, frontend-design, and context7 MCP. If any are missing, Claude outputs specific setup instructions and refuses to proceed.

### 4.2 Stack Declaration

Lists every library with its context7 library ID for fast resolution:

| Library | context7 ID |
|---------|-------------|
| TanStack Router | `/tanstack/router` |
| TanStack Query | `/tanstack/query` |
| shadcn/ui | `/shadcn/ui` |
| React Hook Form | `/react-hook-form/documentation` |
| Vite | `/vitejs/vite` |
| Vitest | `/vitest-dev/vitest` |
| Lucide React | `/websites/lucide_dev_guide_packages` |

Instruction: "Before generating code for any library, query context7 with the appropriate ID to verify current API signatures and patterns."

### 4.3 Global Conventions

- TypeScript strict mode, no `any`
- Zod for all runtime validation (forms, API responses, search params)
- All data fetching through TanStack Query — never raw `fetch` in components
- File-based routing with TanStack Router — never manual route trees
- shadcn/ui components over custom UI — compose, don't reinvent
- Lucide for all icons, individual imports only (tree-shaking)
- Tests colocated: `Component.test.tsx` next to `Component.tsx`
- Always use **frontend-design** skill when building UI
- Always use **superpowers** skills for planning, TDD, debugging
- Always use **context7 MCP** for doc verification
- Client-side state: use React context + `useReducer` for local/UI state; TanStack Query handles all server state — no additional state management library
- Environment variables: use `import.meta.env` with Vite's `VITE_` prefix convention; `.env` files for local config, never commit secrets
- API client: define a typed fetch wrapper in `src/lib/api.ts` that all `queryFn` and `mutationFn` functions call — single source for base URL, auth headers, error handling
- Navigation: use TanStack Router's `<Link>` component for declarative navigation, `useNavigate` for programmatic navigation — never `window.location`
- Package manager: assume `npm` by default; when dropped into an existing project, detect and use the project's existing package manager (check for lock files)
- Context7 fallback: if context7 is unreachable, fall back to baked-in patterns in expert skills and warn the user that docs could not be verified

### 4.4 Skill Routing

CLAUDE.md contains a routing table that maps user intent to skills:

| User intent | Skill(s) to invoke |
|---|---|
| "Set up the project" / new empty project | `phases/01-setup.md` |
| "Create a page" / "add a route" | `workflows/new-page.md` → `experts/tanstack-router.md` |
| "Build a feature" / "add [feature]" | `workflows/new-feature.md` → relevant experts |
| "Build a form" / "add a form" | `workflows/new-form.md` → `experts/react-hook-form.md` + `experts/shadcn-ui.md` |
| Working with routing | `experts/tanstack-router.md` |
| Working with data fetching | `experts/tanstack-query.md` |
| Working with UI components | `experts/shadcn-ui.md` |
| Working with icons | `experts/lucide.md` |
| Writing tests | `experts/testing.md` |
| Understanding project structure | `phases/02-architecture.md` |

## 5. Phase Skills

### 5.1 `01-setup.md` — Project Scaffolding

Runs when the user says "set up the project" from a blank directory. Automatic execution with checkpoints.

**Phase 1 — Research:**
- Fetch latest docs from context7 for ALL libraries using embedded IDs
- Check current stable versions, setup commands, breaking changes
- Verify Node.js version compatibility (Vite 6+ requires Node 18+)
- Detect existing package manager if dropped into existing project (check for lock files)
- Determine optimal install order

**Phase 2 — Vite + React + TypeScript:**
- `npm create vite@latest . -- --template react-ts`
- Configure `tsconfig.json`: strict mode, path aliases (`@/*` → `./src/*`)
- Configure `vite.config.ts`: path alias resolution

**Phase 3 — TanStack Router:**
- Install `@tanstack/react-router` + `@tanstack/router-plugin`
- Add `TanStackRouterVite` plugin to vite config: `routesDirectory: './src/routes'`, `autoCodeSplitting: true`
- Create `src/routes/` with root route and index route
- **Checkpoint:** confirm route tree generation works

**Phase 4 — TanStack Query:**
- Install `@tanstack/react-query` + `@tanstack/react-query-devtools`
- Set up `QueryClient` with sensible defaults
- Wire `QueryClientProvider` into root layout

**Phase 5 — shadcn/ui + Lucide + Tailwind:**
- Run `npx shadcn@latest init -t vite`
- Configure `components.json`: `iconLibrary: "lucide"`, correct path aliases
- Install `lucide-react`
- **Checkpoint:** confirm a shadcn component renders

**Phase 6 — Forms:**
- Install `react-hook-form` + `@hookform/resolvers` + `zod`
- Dependencies ready, no starter code

**Phase 7 — Testing:**
- Install `vitest` + `@testing-library/react` + `@testing-library/jest-dom` + `@testing-library/user-event` + `jsdom`
- Configure `test` block in `vite.config.ts`: `environment: 'jsdom'`, setup files
- Add test scripts to `package.json`
- **Checkpoint:** confirm `vitest run` passes

**Phase 8 — Finalize:**
- ESLint + Prettier setup
- `.gitignore`
- `git init` + initial commit
- Print summary of what was created

### 5.2 `02-architecture.md` — Folder Structure & Patterns

Canonical project structure all other skills assume:

```
src/
├── routes/              # TanStack Router file-based routes
├── components/
│   ├── ui/              # shadcn/ui components (auto-managed by CLI)
│   └── shared/          # Reusable app-level components
├── features/            # Feature modules (co-located)
│   └── [feature]/
│       ├── components/  # Feature-specific components
│       ├── hooks/       # Feature-specific hooks
│       ├── queries/     # queryOptions factories
│       ├── schemas/     # Zod schemas
│       └── types/       # Feature-specific types
├── lib/                 # Utilities, API client, helpers
├── hooks/               # Shared hooks
└── test/                # Test utilities, setup files
```

Conventions:
- Feature modules are self-contained — everything related to a feature lives together
- `queries/` always exports `queryOptions()` factories, never raw query configs
- `schemas/` Zod schemas are shared between form validation and API response validation
- `components/ui/` is managed by shadcn CLI — never edit manually
- `components/shared/` is for app-wide reusable components built from shadcn primitives

## 6. Expert Skills

All expert skills follow the same structure:
1. Core patterns (baked in from context7 research)
2. Context7 verification instructions (for version-sensitive APIs)
3. Do's and don'ts
4. Common mistakes to avoid

### 6.1 `tanstack-router.md`

**Core patterns:**
- Always use `createFileRoute` — never manual route trees
- File naming: `index.tsx` (index route), `$paramName.tsx` (dynamic), `_layout.tsx` (layout)
- Search params via `validateSearch` with Zod schemas
- Route loaders integrate with TanStack Query via `queryClient.ensureQueryData(queryOptions)`
- Every route defines `pendingComponent` and `errorComponent`
- Code splitting automatic via `autoCodeSplitting: true`

**Context7 verification:** Before generating route code, query `/tanstack/router` for current `createFileRoute` API, loader patterns, and search param validation.

**Don'ts:**
- Never create routes outside `src/routes/`
- Never manually edit `routeTree.gen.ts`
- Never use `useEffect` for data fetching in routes — use loaders
- Never use `window.location` for navigation — use `<Link>` or `useNavigate`

### 6.2 `tanstack-query.md`

**Core patterns:**
- Define queries as `queryOptions()` factories in `features/[x]/queries/`
- Never call `fetch` in components — always `useQuery`/`useMutation`
- Cache invalidation: `invalidateQueries({ queryKey })` in mutation `onSuccess`
- Prefetch in route loaders: `queryClient.ensureQueryData(myQueryOptions())`
- DevTools always enabled in development
- `staleTime` and `gcTime` configured per-query based on data freshness needs

**Context7 verification:** Before generating query code, query `/tanstack/query` for current `queryOptions`, `useQuery`, `useMutation` API signatures.

**Don'ts:**
- Never use `useEffect` + `useState` for server data
- Never put query keys as inline arrays — always through `queryOptions()`
- Never mutate cache directly — use `invalidateQueries` or `setQueryData`

### 6.3 `shadcn-ui.md`

**Core patterns:**
- Install via CLI: `npx shadcn@latest add [component]` — never copy manually
- Compose shadcn primitives — check if shadcn has a component before building custom
- Theming via CSS variables — use design tokens, not hardcoded colors
- Lucide icons via shadcn's built-in `iconLibrary: "lucide"` integration
- Always invoke **frontend-design** skill when building UI for production-grade output

**Context7 verification:** Before using a shadcn component, query `/shadcn/ui` for current component API and available variants.

**Don'ts:**
- Never edit files in `components/ui/` manually
- Never install shadcn components by copying from the website
- Never use raw HTML elements when a shadcn component exists

### 6.4 `react-hook-form.md`

**Core patterns:**
- Always pair with Zod: `useForm({ resolver: zodResolver(schema) })`
- Schema in `features/[x]/schemas/` — shared between form and API validation
- Type inference: `z.infer<typeof schema>` — never duplicate types manually
- Use shadcn `<Form>` component wrapper for consistent styling
- `defaultValues` always provided, always typed
- Use `FormField`, `FormItem`, `FormLabel`, `FormControl`, `FormMessage` from shadcn

**Context7 verification:** Before generating form code, query `/react-hook-form/documentation` for current `useForm` options and resolver integration.

**Don'ts:**
- Never use uncontrolled forms without react-hook-form
- Never define validation rules inline — always Zod schemas
- Never duplicate types between schema and form — infer from schema

### 6.5 `lucide.md`

**Core patterns:**
- Individual imports: `import { Camera } from 'lucide-react'`
- Props: `size`, `color`, `strokeWidth`, `className`
- Default size convention: 16 for inline, 20 for buttons, 24 for standalone
- Fully tree-shakeable — only imported icons ship to bundle

**Context7 verification:** Query `/websites/lucide_dev_guide_packages` for available icons and current prop API.

**Don'ts:**
- Never import from a barrel/index — always individual icons
- Never use raw SVGs when a Lucide icon exists
- Never use inconsistent sizes — follow the convention

### 6.6 `testing.md`

**Core patterns:**
- Colocated tests: `Component.test.tsx` next to `Component.tsx`
- Use `@testing-library/react` — test behavior, not implementation
- Query priority: `getByRole` > `getByLabelText` > `getByText` > `getByTestId`
- Mock API at network level (MSW or query client injection), not module mocks
- Every route: minimum smoke render test
- Every form: validation + submission + error handling tests
- Use `userEvent` over `fireEvent` for realistic user interactions
- Wrap components with necessary providers (QueryClient, Router) in test utils

**Context7 verification:** Query `/vitest-dev/vitest` for current configuration API and test runner features.

**Don'ts:**
- Never test implementation details (internal state, method calls)
- Never use `getByTestId` as first choice — use accessible queries
- Never skip provider wrappers — tests should mirror real rendering context
- Never use `jest` imports — always `vitest`

## 7. Workflow Skills

### 7.1 `new-page.md`

Triggered when user asks to add a page or route. Steps:

1. Create route file in `src/routes/` following TanStack Router file naming
2. Define `createFileRoute` with `validateSearch` if search params needed (Zod schema)
3. If page fetches data: create `queryOptions` factory in `src/features/[feature]/queries/`
4. Wire loader with `queryClient.ensureQueryData()` for prefetching
5. Build component using shadcn/ui + Lucide — invoke **frontend-design** skill
6. Add `pendingComponent` and `errorComponent`
7. Create `[Page].test.tsx` smoke test
8. Verify route appears in generated route tree

Invokes: `experts/tanstack-router.md`, `experts/tanstack-query.md`, `experts/shadcn-ui.md`, frontend-design skill

### 7.2 `new-feature.md`

Triggered when user asks to build a complete feature. Steps:

1. Create feature directory: `src/features/[feature]/`
2. Define Zod schemas in `schemas/`
3. Define `queryOptions` factories in `queries/`
4. Build components in `components/` using shadcn/ui — invoke **frontend-design** skill
5. Create custom hooks in `hooks/` if needed
6. Wire route(s) in `src/routes/`
7. Write tests for: queries, components, form validation (if applicable)
8. Run tests to verify

Invokes: all relevant expert skills, frontend-design, superpowers TDD

### 7.3 `new-form.md`

Triggered when user asks to build a form. Steps:

1. Define Zod schema in `src/features/[feature]/schemas/`
2. Infer TypeScript type via `z.infer<typeof schema>`
3. Create form component: `useForm({ resolver: zodResolver(schema) })`
4. Build UI with shadcn `<Form>`, `<FormField>`, `<FormItem>`, `<Input>`, etc. — invoke **frontend-design**
5. Wire `useMutation` for submission with cache invalidation
6. Add loading/error states
7. Write tests: validation, successful submission, error handling

Invokes: `experts/react-hook-form.md`, `experts/shadcn-ui.md`, `experts/tanstack-query.md`, frontend-design

## 8. Hooks & Settings

### 8.1 `.claude/settings.json`

**Permissions (always allowed, never prompt):**
- All `mcp__context7__*` tool calls (resolve-library-id, query-docs)
- npm/npx commands (installs, shadcn CLI)
- git commands
- vitest/eslint execution

**Hooks:**

**Post-file-write — Convention validation (implemented as `PostToolUse` hooks on Write/Edit tools):**
- Files in `src/routes/`: verify exports `Route` via `createFileRoute`
- Files in `src/features/*/queries/`: verify uses `queryOptions()`
- `.test.tsx` files: verify imports from `@testing-library/react`

**Pre-commit — Quality gates:**
- `tsc --noEmit` (type checking)
- `eslint` on staged files
- `vitest related` on staged files (tests related to changed code only)

**Post-scaffold checkpoints:**
- After each phase of `01-setup.md`, verify step succeeded before continuing

### 8.2 Drop-in Compatibility

When dropped into an existing project:
- CLAUDE.md conventions apply to new code only — don't refactor existing code unprompted
- `02-architecture.md` describes ideal structure but adapts to what exists
- Expert skills apply regardless — they're about patterns, not folder structure
- Setup skill (`01-setup.md`) detects existing deps and skips already-installed libraries
- **Conflicting libraries:** if the project uses alternative libraries (e.g., React Router instead of TanStack Router, Formik instead of react-hook-form), Claude warns the user about the conflict and asks whether to coexist or suggest a migration path — never silently overwrite
- **Package manager detection:** check for `pnpm-lock.yaml`, `yarn.lock`, or `package-lock.json` and use the matching package manager

## 9. Context7 Integration Strategy

**Hybrid approach:**
- Core patterns baked into expert skills (from research done during template creation)
- Version-sensitive details (API signatures, new features, breaking changes) always verified at runtime via context7
- Each expert skill contains the exact context7 query to run before generating code
- CLAUDE.md embeds resolved library IDs so Claude skips the resolve step

**Runtime flow:**
1. Claude receives a task
2. CLAUDE.md routes to relevant skill(s)
3. Skill instructs Claude to query context7 for current API details
4. Claude merges baked-in patterns with fresh API data
5. Claude generates code

## 10. Success Criteria

- Copying this folder and saying "set up the project" produces a fully working, well-structured app
- Every piece of generated code follows the conventions without the user having to correct Claude
- Context7 verification prevents stale API usage
- Drop-in to existing projects adds expertise without disrupting existing code
- The three required plugins (superpowers, frontend-design, context7) are always used
