# Rules & Pitfalls — Design Spec

## Summary

Add a three-tier rules architecture to the project:

1. **CLAUDE.md** — Expand Global Conventions with missing universal base rules (no code examples)
2. **`.claude/rules/`** — NEW directory with path-scoped rule files containing DO/DON'T code examples for each library
3. **`.claude/skills/`** — Unchanged (already has rich workflows and expert procedures)

## Architecture

| Layer | Loaded | Content style | Purpose |
|---|---|---|---|
| CLAUDE.md | Always | One-liner rules, no code | Universal truths |
| `.claude/rules/` | When matching files touched | DO/DON'T code examples | Library-specific enforcement |
| `.claude/skills/` | On-demand | Rich workflows, procedures | In-depth expertise, non-deterministic |

## Changes to CLAUDE.md

Add these missing base rules to the existing `## Global Conventions` section:

- No `as` type assertions — fix the type, don't cast
- No `@ts-expect-error` — fix the underlying issue
- No `!` non-null assertions — use optional chaining or narrow properly
- Named exports only — no `export default`
- Use `@/` path alias — no deep relative paths
- No barrel files (`index.ts` re-exporting)
- No `useEffect` for derived state — compute during render
- No `index` as `key` in lists that reorder — use stable IDs
- Memoize objects/callbacks passed as props

## New `.claude/rules/` Files

### 1. `react.md` — `**/*.tsx`
- No useEffect for data fetching (use TanStack Query)
- No useEffect + useState for derived state (compute during render)
- No useEffect to sync props to state
- Memoize objects/callbacks passed as props
- No index as key in reorderable lists
- No Math.random()/crypto.randomUUID() as keys
- When useEffect IS appropriate, always return cleanup

### 2. `tanstack-query.md` — `src/**/queries.*`, `src/**/api.*`, `src/lib/api.*`
- All server data through TanStack Query
- All fetch calls through src/lib/api.ts
- Use queryOptions() factories
- Always handle isLoading and error states
- Invalidate cache after mutations
- Use enabled option instead of conditional hook calls

### 3. `tanstack-router.md` — `src/routes/**/*`
- File-based routing only
- Never edit routeTree.gen.ts
- Link/useNavigate only — no window.location or <a href="">
- Validate search params with Zod via validateSearch
- Let TypeScript infer Link to/params — never silence with as any

### 4. `react-hook-form.md` — `src/**/form*`, `src/**/*Form*`, `src/**/*-form*`
- Zod schema is single source of truth
- Always react-hook-form + zodResolver
- Always provide defaultValues
- Never duplicate types alongside schemas
- Wire mutation onError to form setError for server errors

### 5. `shadcn.md` — `src/components/**/*`, `src/features/**/components/**/*`
- Use shadcn/ui over custom UI
- Install via CLI, never copy-paste source
- Compose primitives into domain components
- Max 2 levels of prop drilling

### 6. `testing.md` — `**/*.test.*`
- Colocate tests next to source
- Query priority: getByRole > getByLabelText > getByText > getByTestId
- userEvent over fireEvent
- findBy for async content
- Test behavior, not implementation

### 7. `lucide.md` — `**/*.tsx`
- Individual named imports — never barrel or wildcard
- Size conventions: 16 inline/buttons, 24 standalone, 48+ heroes
- Use currentColor for theme-aware icons

### 8. `zod.md` — `src/**/schemas.*`, `src/**/*.schema.*`, `src/**/validators.*`
- Use z.infer<> for types — never duplicate manually
- Use .safeParse() on API responses
- Use .strict() over .passthrough() unless explicitly needed
- Don't use .transform() in schemas shared between forms and API

### 9. `vite.md` — `vite.config.*`, `.env*`, `src/lib/env.*`
- import.meta.env, not process.env
- VITE_ prefix required for client-exposed vars
- Never import Node.js modules in client code
- Never commit .env files with secrets

## Design Decisions

- **Three-tier split** — right content at right time: universal always, library rules when relevant, rich expertise on-demand
- **Path-scoped rules** — only load when touching relevant files, keeps context lean
- **Code examples in rules** — concrete DO/DON'T patterns are more effective than prose
- **Skills unchanged** — they already serve the "rich expertise" tier well

## Success Criteria

- CLAUDE.md Global Conventions expanded with missing base rules
- 9 rule files created in `.claude/rules/` with correct path frontmatter
- Each rule file has DO/DON'T code examples
- No duplication between tiers (CLAUDE.md = what, rules = how, skills = deep expertise)
- All files valid markdown
