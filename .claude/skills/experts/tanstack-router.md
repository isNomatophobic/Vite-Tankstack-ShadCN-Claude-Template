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
