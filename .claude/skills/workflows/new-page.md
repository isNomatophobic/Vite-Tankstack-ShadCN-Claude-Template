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
