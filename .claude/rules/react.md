---
paths:
  - "**/*.tsx"
---

# React, Routing, shadcn, Forms, Lucide Rules

## Context7 — Verify Before Generating

```
mcp__context7__query-docs("/tanstack/router", "<question>")
mcp__context7__query-docs("/shadcn/ui", "<component> usage props variants")
mcp__context7__query-docs("/react-hook-form/documentation", "<question>")
mcp__context7__query-docs("/websites/lucide_dev_guide_packages", "<icon search or props>")
```

## React Patterns

**DO:**

```tsx
// Derived state — compute during render
const fullName = `${user.firstName} ${user.lastName}`

// Server data — use TanStack Query
const { data } = useQuery(userQueryOptions(id))

// Stable keys from data
{users.map(user => <UserCard key={user.id} user={user} />)}

// Memoize objects/callbacks passed as props
const style = useMemo(() => ({ color: theme.primary }), [theme.primary])
const handleClick = useCallback((id: string) => selectUser(id), [selectUser])

// When useEffect IS appropriate, always clean up
useEffect(() => {
  const handler = (e: KeyboardEvent) => {
    if (e.key === 'Escape') onClose()
  }
  window.addEventListener('keydown', handler)
  return () => window.removeEventListener('keydown', handler)
}, [onClose])
```

**DON'T:**

```tsx
// Never useEffect for data fetching
useEffect(() => {
  fetch('/api/users').then(r => r.json()).then(setUsers)
}, []) // ❌ use TanStack Query

// Never useEffect + useState for derived state
const [fullName, setFullName] = useState('')
useEffect(() => {
  setFullName(`${user.firstName} ${user.lastName}`)
}, [user]) // ❌ compute during render

// Never useEffect to sync props to state
const [localValue, setLocalValue] = useState(props.value)
useEffect(() => {
  setLocalValue(props.value)
}, [props.value]) // ❌ use the prop directly

// Never use index as key in reorderable lists
{users.map((user, i) => <UserCard key={i} />)} // ❌ use user.id

// Never use random values as keys
{items.map(item => <Row key={Math.random()} />)} // ❌ re-mounts every render

// Never create objects/functions inline in JSX (causes re-renders)
<Component style={{ color: 'red' }} onClick={() => handleClick(id)} /> // ❌ memoize
```

---

## React 19 Patterns

### The `use` Hook

`use` reads values from Promises or Context. Unlike other hooks, it can be called in conditionals and loops.

```tsx
import { use } from 'react'

// Conditional context reading (can't do this with useContext)
function HorizontalRule({ show }: { show: boolean }) {
  if (show) {
    const theme = use(ThemeContext)
    return <hr className={theme} />
  }
  return null
}

// Reading promises — suspends until resolved
function Message({ messagePromise }: { messagePromise: Promise<string> }) {
  const content = use(messagePromise)
  return <p>{content}</p>
}
```

**DON'T:**

```tsx
// Never use `use` in try-catch blocks — it breaks Suspense
try {
  const data = use(dataPromise) // ❌ Suspense won't work
} catch (e) { ... }

// Never call `use` in event handlers
function Component({ dataPromise }) {
  function handleClick() {
    const data = use(dataPromise) // ❌ must be in component body
  }
}

// Never create promises during render in client components
function Component() {
  const data = use(fetch('/api/data')) // ❌ recreates every render
}
// Pass stable promises from parent or use TanStack Query instead
```

### Suspense + ErrorBoundary

Wrap async components in Suspense for loading states and ErrorBoundary for errors:

```tsx
import { Suspense } from 'react'
import { ErrorBoundary } from 'react-error-boundary'

function PostsPage() {
  return (
    <ErrorBoundary fallback={<p>Something went wrong</p>}>
      <Suspense fallback={<Skeleton />}>
        <PostsList />
      </Suspense>
    </ErrorBoundary>
  )
}
```

### Nested Suspense for Progressive Loading

```tsx
// Biography loads first, albums load independently
<Suspense fallback={<PageSkeleton />}>
  <Biography artistId={id} />
  <Suspense fallback={<AlbumsSkeleton />}>
    <Albums artistId={id} />
  </Suspense>
</Suspense>
```

### Prevent Fallback During Navigation with startTransition

```tsx
import { startTransition, useTransition } from 'react'

function App() {
  const [page, setPage] = useState('/')
  const [isPending, startTransition] = useTransition()

  function navigate(url: string) {
    // Keeps current content visible while loading, won't show fallback
    startTransition(() => setPage(url))
  }

  return (
    <div style={{ opacity: isPending ? 0.7 : 1 }}>
      <Suspense fallback={<Skeleton />}>
        <PageContent page={page} />
      </Suspense>
    </div>
  )
}
```

### Reset Key for Route Changes

```tsx
// Reset Suspense boundaries when params change
<ProfilePage key={userId} />
```

---

## TanStack Query + Suspense

### useSuspenseQuery — data is always defined

```tsx
import { useSuspenseQuery } from '@tanstack/react-query'

function PostsList() {
  // No need to check isLoading/error — handled by Suspense/ErrorBoundary
  const { data } = useSuspenseQuery(postsQueryOptions())

  return data.map(post => <PostCard key={post.id} post={post} />)
}
```

### QueryErrorResetBoundary — retry after errors

```tsx
import { QueryErrorResetBoundary } from '@tanstack/react-query'
import { ErrorBoundary } from 'react-error-boundary'

function PostsPage() {
  return (
    <QueryErrorResetBoundary>
      {({ reset }) => (
        <ErrorBoundary
          onReset={reset}
          fallbackRender={({ resetErrorBoundary }) => (
            <div>
              Something went wrong.
              <Button onClick={() => resetErrorBoundary()}>Try again</Button>
            </div>
          )}
        >
          <Suspense fallback={<Skeleton />}>
            <PostsList />
          </Suspense>
        </ErrorBoundary>
      )}
    </QueryErrorResetBoundary>
  )
}
```

### Prefetch in Parent for Parallel Suspense Fetching

```tsx
import { usePrefetchQuery } from '@tanstack/react-query'

function ArticleLayout({ id }: { id: string }) {
  // Prefetch comments while article loads — fetches in parallel
  usePrefetchQuery(articleCommentsQueryOptions(id))

  return (
    <Suspense fallback={<Skeleton />}>
      <Article id={id} />
    </Suspense>
  )
}

function Article({ id }: { id: string }) {
  const { data } = useSuspenseQuery(articleQueryOptions(id))
  // Comments are likely already cached from prefetch
  return <ArticleView article={data} />
}
```

### Keep Current Content During QueryKey Changes (useTransition)

`placeholderData` was intentionally removed from `useSuspenseQuery`. Use React's `useTransition` instead — this is a React concern, not a query library concern (per TkDodo / TanStack maintainers).

```tsx
function PostsAdmin() {
  const [page, setPage] = useState(1)
  const [isPending, startTransition] = useTransition()

  function goToPage(newPage: number) {
    // Wrap the state update — keeps current content visible, no fallback flash
    startTransition(() => setPage(newPage))
  }

  return (
    <div style={{ opacity: isPending ? 0.5 : 1 }}>
      <Suspense fallback={<TableSkeleton />}>
        <PostsTable page={page} />
      </Suspense>
      <Pagination page={page} onPageChange={goToPage} />
    </div>
  )
}

function PostsTable({ page }: { page: number }) {
  const { data } = useSuspenseQuery(postsQueryOptions({ page }))
  return <DataTable data={data} />
}
```

### Keep Current Content During Search (useDeferredValue)

For search/filter inputs, use `useDeferredValue` — it lets the input update immediately while the query result lags behind showing dimmed stale content:

```tsx
function SearchPage() {
  const [query, setQuery] = useState('')
  const deferredQuery = useDeferredValue(query)
  const isStale = query !== deferredQuery

  return (
    <>
      <Input value={query} onChange={(e) => setQuery(e.target.value)} />
      <Suspense fallback={<Skeleton />}>
        <div style={{ opacity: isStale ? 0.5 : 1 }}>
          <SearchResults query={deferredQuery} />
        </div>
      </Suspense>
    </>
  )
}

function SearchResults({ query }: { query: string }) {
  const { data } = useSuspenseQuery(searchQueryOptions(query))
  return <ResultsList results={data} />
}
```

### When to Use Which

| Scenario | Pattern |
|----------|---------|
| Pagination / tab switching | `useTransition` — wrap the state update |
| Search / filter input | `useDeferredValue` — defer the query param |
| Navigation between routes | `startTransition` in navigate handler |
| First load (no existing content) | Let Suspense fallback show normally |

**DON'T with Suspense queries:**

```tsx
// Never conditionally enable useSuspenseQuery — it doesn't support `enabled`
const { data } = useSuspenseQuery({
  ...userQueryOptions(userId),
  enabled: !!userId, // ❌ not supported — use useQuery for conditional fetching
})

// Never use placeholderData with useSuspenseQuery — intentionally removed
// Use useTransition or useDeferredValue instead (see above)

// Never forget the ErrorBoundary — rejected queries throw
<Suspense fallback={<Skeleton />}>
  <PostsList /> {/* ❌ unhandled error if query fails */}
</Suspense>

// Never change queryKey without transition — causes fallback flash
setPage(newPage) // ❌ shows fallback — wrap in startTransition
```

---

## TanStack Router

### File-Based Routing

Routes live in `src/routes/`. The Vite plugin auto-generates `routeTree.gen.ts`.

**File naming:**

| File | Route |
|------|-------|
| `__root.tsx` | Root layout (wraps all routes) |
| `index.tsx` | Index route for current directory |
| `about.tsx` | Static route (`/about`) |
| `$paramName.tsx` | Dynamic route (`/posts/$postId`) |
| `posts/` directory | Nested routes (`/posts/...`) |
| `_layout.tsx` | Layout route (wraps children, no URL segment) |

### Route Definition

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

```tsx
export const Route = createFileRoute('/posts/')({
  loader: ({ context: { queryClient } }) =>
    queryClient.ensureQueryData(postsQueryOptions()),
  pendingComponent: () => <div>Loading...</div>,
  errorComponent: ({ error }) => <div>Error: {error.message}</div>,
  component: PostsPage,
})
```

Every route with a loader MUST define `pendingComponent` and `errorComponent`.

### Search Params with Zod

```tsx
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
}
```

### Navigation

```tsx
// Declarative (preferred)
<Link to="/posts/$postId" params={{ postId: '123' }}>View Post</Link>

// Programmatic
const navigate = useNavigate()
navigate({ to: '/posts/$postId', params: { postId: '123' } })
```

**DON'T:**

```tsx
// Never edit routeTree.gen.ts — auto-generated
// Never create route files outside src/routes/
// Never use react-router-dom APIs — this is TanStack Router

window.location.href = '/users' // ❌ full page reload
<a href="/users">Users</a> // ❌ use <Link>
const params = new URLSearchParams(window.location.search) // ❌ use validateSearch
<Link to={path as any} params={params as any} /> // ❌ fix the types
```

---

## shadcn/ui Components

### Installing

Always use the CLI — never copy code:

```bash
npx shadcn@latest add button card dialog input form table
```

### Using Components

```tsx
import { Button } from '@/components/ui/button'
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card'

<Card>
  <CardHeader>
    <CardTitle>Title</CardTitle>
  </CardHeader>
  <CardContent>
    <Button variant="outline" size="sm">Click me</Button>
  </CardContent>
</Card>
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

CSS variables in `src/index.css`. Always use semantic tokens (`bg-primary`, `text-muted-foreground`), never hardcoded colors.

### Frontend Design

When building UI, ALWAYS invoke the **frontend-design** skill (`frontend-design:frontend-design`) for production-grade output.

**DON'T:**

```tsx
// Never edit files in src/components/ui/ — managed by CLI
// Never copy component code from shadcn website — use npx shadcn@latest add
// Never use raw HTML (<button>, <input>, <table>) when shadcn has an equivalent
// Never hardcode colors — use CSS variable tokens
// Never skip the frontend-design skill when building visible UI

// Never prop-drill more than 2 levels
<Parent user={user}>
  <Child user={user}>
    <GrandChild user={user} /> {/* ❌ use context or composition */}
  </Child>
</Parent>
```

---

## react-hook-form

### Form Architecture

1. **Schema** in `src/features/[feature]/schemas/` — Zod defines shape + validation
2. **Type** inferred via `z.infer<>` — never duplicate
3. **Component** uses `useForm` + `zodResolver` + shadcn `<Form>` components

### Schema (Single Source of Truth)

```ts
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
import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { loginSchema, type LoginFormData } from '@/features/auth/schemas/login-schema'
import {
  Form, FormControl, FormField, FormItem, FormLabel, FormMessage,
} from '@/components/ui/form'
import { Input } from '@/components/ui/input'
import { Button } from '@/components/ui/button'

export function LoginForm({ onSubmit }: { onSubmit: (data: LoginFormData) => void }) {
  const form = useForm<LoginFormData>({
    resolver: zodResolver(loginSchema),
    defaultValues: { email: '', password: '' },
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
type ApiError = { message: string; field?: keyof LoginFormData }

const loginMutation = useMutation({
  mutationFn: (data: LoginFormData) => api.post('/auth/login', data),
  onSuccess: () => { /* navigate, toast, etc. */ },
  onError: (error: ApiError) => {
    if (error.field) {
      form.setError(error.field, { message: error.message })
    } else {
      form.setError('root', { message: error.message })
    }
  },
})
```

**DON'T:**

```tsx
// Never duplicate types manually
type LoginForm = { email: string; password: string } // ❌ use z.infer<>

// Never validate manually
if (!data.email.includes('@')) { ... } // ❌ use Zod

// Never use <form> without shadcn's <Form> wrapper
// Never define validation rules inline ({ required: true }) — always Zod schemas
// Never use uncontrolled forms without react-hook-form
// Never forget defaultValues (causes uncontrolled → controlled warning)
// Never ignore server-side mutation errors — wire onError to form.setError
```

---

## Lucide Icons

### Importing

```tsx
import { Search, Plus, Trash2, ChevronRight } from 'lucide-react'
```

### Size Conventions

| Context | Size | Example |
|---------|------|---------|
| Inline with text | 16 | `<Search size={16} />` |
| Inside buttons | 16 | `<Button><Plus size={16} /> Add</Button>` |
| Standalone / nav | 24 | `<Settings size={24} />` |
| Empty states / hero | 48+ | `<Inbox size={48} />` |

### Props

```tsx
<Search
  size={20}            // Width and height in px
  color="currentColor" // Inherits text color by default
  strokeWidth={2}      // Default stroke width
  className="shrink-0" // Tailwind classes work
/>
```

### With shadcn

```tsx
// Icon + text button
<Button variant="outline" size="sm">
  <Plus size={16} />
  New Item
</Button>

// Icon-only button
<Button variant="ghost" size="icon">
  <Trash2 size={16} />
</Button>
```

**DON'T:**

```tsx
// Never import all icons (kills tree-shaking)
import * as Icons from 'lucide-react' // ❌

// Never use raw SVGs when a Lucide icon exists
<svg viewBox="0 0 24 24">...</svg> // ❌

// Never hardcode colors — use className for theming
<Search color="#ff0000" /> // ❌ use className="text-destructive"

// Never use inconsistent sizes — follow the table above
```
