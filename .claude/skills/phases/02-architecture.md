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
