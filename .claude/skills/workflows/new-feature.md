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
