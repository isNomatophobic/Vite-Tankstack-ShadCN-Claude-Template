---
paths:
  - "**/*.test.*"
---

# Testing Rules

## Context7 — Verify Before Generating

```
mcp__context7__query-docs("/vitest-dev/vitest", "<specific question about config, matchers, mocking>")
```

## Test File Location

Colocated — test files live next to source:

```
src/features/posts/components/
  PostCard.tsx
  PostCard.test.tsx
```

Only test infrastructure lives in `src/test/`:
- `setup.ts` — global setup (jest-dom imports)
- `test-utils.tsx` — custom render with providers

## Custom Render with Providers

```tsx
// src/test/test-utils.tsx
import { render, type RenderOptions } from '@testing-library/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

function createTestQueryClient() {
  return new QueryClient({
    defaultOptions: {
      queries: { retry: false, gcTime: 0 },
      mutations: { retry: false },
    },
  });
}

export function renderWithProviders(
  ui: React.ReactElement,
  options?: Omit<RenderOptions, 'wrapper'>,
) {
  const queryClient = createTestQueryClient();

  function Wrapper({ children }: { children: React.ReactNode }) {
    return (
      <QueryClientProvider client={queryClient}>
        {children}
      </QueryClientProvider>
    );
  }

  return {
    ...render(ui, { wrapper: Wrapper, ...options }),
    queryClient,
  };
}
```

## Query Priority

Always use the most accessible query:

1. `getByRole` — buttons, headings, links, form elements
2. `getByLabelText` — form inputs with labels
3. `getByPlaceholderText` — inputs with placeholders
4. `getByText` — visible text content
5. `getByTestId` — LAST RESORT only

```tsx
// GOOD
screen.getByRole('button', { name: /submit/i });
screen.getByRole('heading', { name: /welcome/i });
screen.getByLabelText(/email/i);

// BAD — only if no accessible query works
screen.getByTestId('submit-button');
```

## User Events

Always `userEvent` over `fireEvent`:

```tsx
import userEvent from '@testing-library/user-event';

const user = userEvent.setup();
await user.type(screen.getByLabelText(/email/i), 'test@example.com');
await user.click(screen.getByRole('button', { name: /sign in/i }));
```

## Async Content

Use `findBy` — never `getBy` + `waitFor` when `findBy` works:

```tsx
const name = await screen.findByText('John');
```

## What to Test

- **Routes:** Smoke render — mounts without crashing
- **Components:** Correct content, user interaction, loading/error/empty states
- **Forms:** Validation errors display, successful submission calls handler, disabled during submission
- **Queries/Mutations:** Test through the component, not in isolation
- **Suspense components:** Verify fallback renders, then content appears
- **ErrorBoundary:** Verify error fallback renders on failure

---

## Common Pitfalls

### 1. act() Warnings

```tsx
// ❌ Causes "not wrapped in act()" warning — state update after test ends
test('loads data', () => {
  renderWithProviders(<UserProfile />);
  expect(screen.getByText('Loading')).toBeInTheDocument();
  // Test ends while query is still fetching — state updates after unmount
});

// ✅ Wait for the async operation to complete
test('loads data', async () => {
  renderWithProviders(<UserProfile />);
  expect(await screen.findByText('John')).toBeInTheDocument();
});
```

### 2. Not Awaiting userEvent

```tsx
// ❌ Missing await — event hasn't fired when assertion runs
test('clicks button', () => {
  const user = userEvent.setup();
  renderWithProviders(<Counter />);
  user.click(screen.getByRole('button')); // ❌ not awaited
  expect(screen.getByText('1')).toBeInTheDocument(); // fails
});

// ✅ Always await userEvent calls
test('clicks button', async () => {
  const user = userEvent.setup();
  renderWithProviders(<Counter />);
  await user.click(screen.getByRole('button'));
  expect(screen.getByText('1')).toBeInTheDocument();
});
```

### 3. Testing Suspense Components

```tsx
// ❌ Missing Suspense wrapper — throws error
test('renders post', async () => {
  renderWithProviders(<PostsList />); // ❌ useSuspenseQuery needs Suspense boundary
});

// ✅ Wrap in Suspense + provide fallback
test('renders post list', async () => {
  renderWithProviders(
    <Suspense fallback={<div>Loading...</div>}>
      <PostsList />
    </Suspense>
  );
  expect(await screen.findByText('Test Post')).toBeInTheDocument();
});
```

### 4. Testing ErrorBoundary

```tsx
import { QueryErrorResetBoundary } from '@tanstack/react-query';
import { ErrorBoundary } from 'react-error-boundary';

test('shows error fallback on query failure', async () => {
  // Mock API to reject
  vi.spyOn(api, 'get').mockRejectedValueOnce(new Error('Network error'));

  renderWithProviders(
    <QueryErrorResetBoundary>
      {({ reset }) => (
        <ErrorBoundary onReset={reset} fallbackRender={({ resetErrorBoundary }) => (
          <div>
            <p>Error</p>
            <button onClick={resetErrorBoundary}>Retry</button>
          </div>
        )}>
          <Suspense fallback={<div>Loading...</div>}>
            <PostsList />
          </Suspense>
        </ErrorBoundary>
      )}
    </QueryErrorResetBoundary>
  );

  expect(await screen.findByText('Error')).toBeInTheDocument();
});
```

### 5. QueryClient Leaking Between Tests

```tsx
// ❌ Shared QueryClient — cache leaks between tests
const queryClient = new QueryClient();

test('test A', () => {
  renderWithProviders(<Component />, { queryClient }); // ❌ same client
});
test('test B', () => {
  renderWithProviders(<Component />, { queryClient }); // ❌ gets cached data from test A
});

// ✅ renderWithProviders creates a fresh QueryClient per test (see implementation above)
// If you need to pre-seed cache, use the returned queryClient:
test('renders with data', () => {
  const { queryClient } = renderWithProviders(<Component />);
  queryClient.setQueryData(['key'], mockData);
});
```

### 6. waitFor Timeout — Test Hangs Then Fails

```tsx
// ❌ Assertion never becomes true — waitFor times out after 1000ms
await waitFor(() => {
  expect(screen.getByText('Data loaded')).toBeInTheDocument();
});

// Common causes:
// - API mock not set up correctly — query never resolves
// - Wrong text/role — check what actually rendered
// - Missing provider — component silently fails

// ✅ Debug: check what rendered
screen.debug(); // prints current DOM to console

// ✅ Or use findBy instead (includes built-in waitFor)
expect(await screen.findByText('Data loaded')).toBeInTheDocument();
```

### 7. Mocking API Calls

```tsx
// ✅ Mock the api module
vi.mock('@/lib/api', () => ({
  api: {
    get: vi.fn(),
    post: vi.fn(),
  },
}));

import { api } from '@/lib/api';

beforeEach(() => {
  vi.mocked(api.get).mockResolvedValue([
    { id: '1', title: 'Test Post', body: 'Content' },
  ]);
});

afterEach(() => {
  vi.restoreAllMocks();
});
```

### 8. Testing Loading States

```tsx
test('shows loading skeleton', () => {
  // Don't resolve the API call — keep it pending
  vi.mocked(api.get).mockReturnValue(new Promise(() => {})); // never resolves

  renderWithProviders(
    <Suspense fallback={<div data-testid="skeleton">Loading</div>}>
      <PostsList />
    </Suspense>
  );

  expect(screen.getByTestId('skeleton')).toBeInTheDocument();
});
```

### 9. Testing Custom Hooks

```tsx
import { renderHook, waitFor } from '@testing-library/react';

test('useCreatePost calls API and invalidates cache', async () => {
  const wrapper = ({ children }) => (
    <QueryClientProvider client={createTestQueryClient()}>
      {children}
    </QueryClientProvider>
  );

  const { result } = renderHook(() => useCreatePost(), { wrapper });

  result.current.mutate({ title: 'New Post', body: 'Content' });

  await waitFor(() => {
    expect(result.current.isSuccess).toBe(true);
  });
  expect(api.post).toHaveBeenCalledWith('/posts', { title: 'New Post', body: 'Content' });
});
```

### 10. Cleanup — Avoid Memory Leaks

```tsx
// ✅ Vitest + RTL auto-cleanup after each test (via setup.ts)
// If you see warnings about updates after unmount, the test is
// ending before async operations complete. Fix: await the result.

// ❌ Manual timer cleanup forgotten
test('debounced search', async () => {
  vi.useFakeTimers();
  renderWithProviders(<SearchInput />);
  // ... test ...
  vi.useRealTimers(); // ❌ if test fails before this line, timers leak
});

// ✅ Use afterEach for cleanup
afterEach(() => {
  vi.useRealTimers();
});
```

---

## Example: Component Test

```tsx
import { describe, it, expect, vi } from 'vitest';
import { screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { renderWithProviders } from '@/test/test-utils';
import { PostCard } from './PostCard';

describe('PostCard', () => {
  const mockPost = { id: '1', title: 'Test Post', body: 'Test body' };

  it('renders post title and body', () => {
    renderWithProviders(<PostCard post={mockPost} />);
    expect(screen.getByRole('heading', { name: /test post/i })).toBeInTheDocument();
    expect(screen.getByText(/test body/i)).toBeInTheDocument();
  });

  it('calls onDelete when delete button is clicked', async () => {
    const user = userEvent.setup();
    const onDelete = vi.fn();
    renderWithProviders(<PostCard post={mockPost} onDelete={onDelete} />);
    await user.click(screen.getByRole('button', { name: /delete/i }));
    expect(onDelete).toHaveBeenCalledWith('1');
  });
});
```

## Example: Form Validation Test

```tsx
import { describe, it, expect, vi } from 'vitest';
import { screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { renderWithProviders } from '@/test/test-utils';
import { LoginForm } from './LoginForm';

describe('LoginForm', () => {
  it('shows validation errors for invalid input', async () => {
    const user = userEvent.setup();
    renderWithProviders(<LoginForm onSubmit={vi.fn()} />);
    await user.click(screen.getByRole('button', { name: /sign in/i }));
    await waitFor(() => {
      expect(screen.getByText(/invalid email/i)).toBeInTheDocument();
    });
  });

  it('submits with valid data', async () => {
    const user = userEvent.setup();
    const onSubmit = vi.fn();
    renderWithProviders(<LoginForm onSubmit={onSubmit} />);
    await user.type(screen.getByLabelText(/email/i), 'valid@email.com');
    await user.type(screen.getByLabelText(/password/i), 'password123');
    await user.click(screen.getByRole('button', { name: /sign in/i }));
    await waitFor(() => {
      expect(onSubmit).toHaveBeenCalledWith({
        email: 'valid@email.com',
        password: 'password123',
      });
    });
  });
});
```

---

## DON'Ts

- Never test implementation details (internal state, private methods)
- Never use `getByTestId` as first choice — use accessible queries
- Never skip provider wrappers — tests must mirror real rendering context
- Never use `jest` imports — always `vitest` (`vi.fn()`, `vi.mock()`)
- Never use `fireEvent` when `userEvent` can do the job
- Never test library internals (don't test that React Query caches)
- Never put tests in `__tests__/` — colocate with source
- Never use `getBy` + `waitFor` when `findBy` works
- Never forget to await `userEvent` calls — assertions will run before events fire
- Never reuse a `QueryClient` across tests — create a fresh one per test
- Never forget Suspense wrapper when testing `useSuspenseQuery` components
- Never let tests end with pending async operations — always await completion
