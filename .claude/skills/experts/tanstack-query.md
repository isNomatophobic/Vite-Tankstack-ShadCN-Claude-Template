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
