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
