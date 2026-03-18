---
paths:
  - "**/*.ts"
  - "**/*.tsx"
---

# TypeScript, Imports, Zod & TanStack Query Rules

## Context7 — Verify Before Generating

```
mcp__context7__query-docs("/tanstack/query", "<your specific question>")
```

## TypeScript

**DO:**

```ts
// Use unknown and validate with Zod
function parse(input: unknown): User {
  return userSchema.parse(input)
}

// Narrow types properly
if (user !== undefined) {
  console.log(user.name)
}

// Optional chaining
const name = user?.profile?.name
```

**DON'T:**

```ts
// Never use any
function parse(input: any) { ... } // ❌ use unknown

// Never use as type assertions to silence errors
const user = data as User // ❌ fix the type, don't cast

// Never suppress errors
// @ts-ignore   // ❌ fix the underlying issue
// @ts-expect-error  // ❌ fix the underlying issue

// Never use non-null assertion
const name = user!.name // ❌ use optional chaining or narrow
```

---

## Imports & Exports

**DO:**

```ts
// Named exports for components and utilities
export function UserCard() { ... }
export const userSchema = z.object({ ... })

// Path aliases
import { api } from '@/lib/api'
import { Button } from '@/components/ui/button'

// Individual icon imports
import { Search } from 'lucide-react'
import { ChevronDown } from 'lucide-react'
```

**DON'T:**

```ts
// Never use default exports
export default function UserCard() { ... } // ❌ use named exports

// Never use deep relative paths when alias exists
import { api } from '../../../lib/api' // ❌ use @/lib/api

// Never create barrel files (index.ts re-exporting everything)
// src/features/users/index.ts
export * from './UserCard' // ❌ hurts tree-shaking, causes circular deps

// Never import all icons (kills tree-shaking)
import * as Icons from 'lucide-react' // ❌
import { icons } from 'lucide-react' // ❌
```

---

## Zod

**DO:**

```ts
// Infer types from schemas — single source of truth
const userSchema = z.object({
  id: z.string().uuid(),
  name: z.string().min(1),
  email: z.string().email(),
})
type User = z.infer<typeof userSchema>

// Validate API responses with safeParse
const result = z.array(userSchema).safeParse(response)
if (!result.success) {
  console.error(result.error.flatten())
  return
}
const users = result.data

// Use strict() to reject unknown properties
const createUserSchema = userSchema.omit({ id: true }).strict()
```

**DON'T:**

```ts
// Never duplicate types alongside schemas
type User = { id: string; name: string; email: string } // ❌ use z.infer<>

// Never skip validation on API responses
const users: User[] = await api.get('/users') // ❌ validate with Zod

// Never use .passthrough() unless explicitly needed
const schema = z.object({ name: z.string() }).passthrough() // ❌ use .strict()

// Never use .transform() in schemas shared between forms and API validation
const schema = z.object({
  price: z.string().transform(Number), // ❌ breaks defaultValues typing in forms
}) // create separate schemas for form input vs API payload
```

---

## TanStack Query

### queryOptions Factories

All queries defined as `queryOptions()` factories in `src/features/[feature]/queries/`:

```ts
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
// Standard — manually handle loading/error
const { data, isLoading, error } = useQuery(postsQueryOptions())

if (isLoading) return <Skeleton />
if (error) return <ErrorDisplay error={error} />
return data.map(post => <PostCard key={post.id} post={post} />)
```

### Suspense Mode — data is always defined

```tsx
import { useSuspenseQuery } from '@tanstack/react-query'

// Wrap in <Suspense> + <ErrorBoundary> — see react.md for full patterns
const { data } = useSuspenseQuery(postsQueryOptions())
// data is guaranteed defined — no isLoading/error checks needed
return data.map(post => <PostCard key={post.id} post={post} />)
```

### Mutations with Cache Invalidation

```tsx
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
loader: ({ context: { queryClient } }) =>
  queryClient.ensureQueryData(postsQueryOptions())
```

### Conditional Queries

```ts
// Use enabled — never call hooks conditionally
const { data } = useQuery({
  ...userQueryOptions(userId ?? ''),
  enabled: !!userId,
})
```

### QueryClient Configuration

```ts
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60,      // 1 minute
      gcTime: 1000 * 60 * 5,     // 5 minutes
      retry: 1,
      refetchOnWindowFocus: false,
    },
  },
})
```

### DevTools

```tsx
import { ReactQueryDevtools } from '@tanstack/react-query-devtools'

// In root layout
<ReactQueryDevtools initialIsOpen={false} />
```

**DON'T:**

```ts
// Never raw fetch in components
const [users, setUsers] = useState([])
useEffect(() => {
  fetch('/api/users').then(r => r.json()).then(setUsers)
}, []) // ❌ use TanStack Query

// Never hardcode query keys as loose strings
useQuery({ queryKey: ['users'], queryFn: fetchUsers }) // ❌ use queryOptions() factory

// Never call hooks conditionally — violates Rules of Hooks
if (userId) {
  const { data } = useQuery(userQueryOptions(userId)) // ❌ use enabled option
}

// Never mutate query data directly
const { data } = useQuery(usersQueryOptions())
data.push(newUser) // ❌ mutates cache — use mutations

// Never skip error/loading states
const { data } = useQuery(usersQueryOptions())
return <div>{data.name}</div> // ❌ crashes when loading/error

// Never bypass the API client
useQuery({
  queryKey: ['users'],
  queryFn: () => fetch('/api/users').then(r => r.json()), // ❌ use api.get()
})

// Never create a new QueryClient per render — it must be stable
```
