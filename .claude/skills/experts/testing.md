---
name: testing-expert
description: Vitest + React Testing Library expertise — component testing patterns, query priority, provider wrappers, TDD approach
---

# Testing Expert

## Context7 Verification

Before writing test setup or using unfamiliar testing APIs:

```
mcp__context7__query-docs("/vitest-dev/vitest", "<specific question about test config, matchers, mocking>")
```

## Core Patterns

### Test File Location

Colocated — test files live next to source files:

```
src/features/posts/components/
  PostCard.tsx
  PostCard.test.tsx
```

Only test infrastructure lives in `src/test/`:
- `setup.ts` — global setup (jest-dom imports)
- `test-utils.tsx` — custom render with providers

### Custom Render with Providers

Create a test render function that wraps components with required providers:

```tsx
// src/test/test-utils.tsx
import { render, type RenderOptions } from '@testing-library/react'
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
import {
  RouterProvider,
  createRouter,
  createRootRoute,
  createRoute,
  createMemoryHistory,
} from '@tanstack/react-router'

function createTestQueryClient() {
  return new QueryClient({
    defaultOptions: {
      queries: { retry: false, gcTime: 0 },
      mutations: { retry: false },
    },
  })
}

export function renderWithProviders(
  ui: React.ReactElement,
  options?: Omit<RenderOptions, 'wrapper'>,
) {
  const queryClient = createTestQueryClient()

  function Wrapper({ children }: { children: React.ReactNode }) {
    return (
      <QueryClientProvider client={queryClient}>
        {children}
      </QueryClientProvider>
    )
  }

  return {
    ...render(ui, { wrapper: Wrapper, ...options }),
    queryClient,
  }
}
```

### Query Priority

Always use the most accessible query available:

1. `getByRole` — buttons, headings, links, form elements
2. `getByLabelText` — form inputs with labels
3. `getByPlaceholderText` — inputs with placeholders
4. `getByText` — visible text content
5. `getByTestId` — LAST RESORT only

```tsx
// GOOD
screen.getByRole('button', { name: /submit/i })
screen.getByRole('heading', { name: /welcome/i })
screen.getByLabelText(/email/i)

// BAD — only if no accessible query works
screen.getByTestId('submit-button')
```

### User Events

Always use `userEvent` over `fireEvent`:

```tsx
import userEvent from '@testing-library/user-event'

test('submits the form', async () => {
  const user = userEvent.setup()
  renderWithProviders(<LoginForm onSubmit={mockSubmit} />)

  await user.type(screen.getByLabelText(/email/i), 'test@example.com')
  await user.type(screen.getByLabelText(/password/i), 'password123')
  await user.click(screen.getByRole('button', { name: /sign in/i }))

  expect(mockSubmit).toHaveBeenCalledWith({
    email: 'test@example.com',
    password: 'password123',
  })
})
```

### What to Test

**Routes:** Smoke render test — mounts without crashing.

**Components:** Renders correct content, responds to user interaction, shows correct states (loading, error, empty).

**Forms:** Validation errors display, successful submission calls handler, disabled during submission.

**Queries/Mutations:** Test through the component that uses them, not in isolation.

### Example: Component Test

```tsx
import { describe, it, expect, vi } from 'vitest'
import { screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import { renderWithProviders } from '@/test/test-utils'
import { PostCard } from './PostCard'

describe('PostCard', () => {
  const mockPost = { id: '1', title: 'Test Post', body: 'Test body' }

  it('renders post title and body', () => {
    renderWithProviders(<PostCard post={mockPost} />)

    expect(screen.getByRole('heading', { name: /test post/i })).toBeInTheDocument()
    expect(screen.getByText(/test body/i)).toBeInTheDocument()
  })

  it('calls onDelete when delete button is clicked', async () => {
    const user = userEvent.setup()
    const onDelete = vi.fn()
    renderWithProviders(<PostCard post={mockPost} onDelete={onDelete} />)

    await user.click(screen.getByRole('button', { name: /delete/i }))

    expect(onDelete).toHaveBeenCalledWith('1')
  })
})
```

### Example: Form Validation Test

```tsx
import { describe, it, expect, vi } from 'vitest'
import { screen, waitFor } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import { renderWithProviders } from '@/test/test-utils'
import { LoginForm } from './LoginForm'

describe('LoginForm', () => {
  it('shows validation errors for invalid input', async () => {
    const user = userEvent.setup()
    renderWithProviders(<LoginForm onSubmit={vi.fn()} />)

    await user.click(screen.getByRole('button', { name: /sign in/i }))

    await waitFor(() => {
      expect(screen.getByText(/invalid email/i)).toBeInTheDocument()
    })
  })

  it('submits with valid data', async () => {
    const user = userEvent.setup()
    const onSubmit = vi.fn()
    renderWithProviders(<LoginForm onSubmit={onSubmit} />)

    await user.type(screen.getByLabelText(/email/i), 'valid@email.com')
    await user.type(screen.getByLabelText(/password/i), 'password123')
    await user.click(screen.getByRole('button', { name: /sign in/i }))

    await waitFor(() => {
      expect(onSubmit).toHaveBeenCalledWith({
        email: 'valid@email.com',
        password: 'password123',
      })
    })
  })
})
```

## Don'ts

- Never test implementation details (internal state, private methods)
- Never use `getByTestId` as first choice — use accessible queries
- Never skip provider wrappers — tests must mirror real rendering context
- Never use `jest` imports — always `vitest` (`vi.fn()`, `vi.mock()`, etc.)
- Never use `fireEvent` when `userEvent` can do the job
- Never test library internals (don't test that React Query caches — test your component's behavior)
