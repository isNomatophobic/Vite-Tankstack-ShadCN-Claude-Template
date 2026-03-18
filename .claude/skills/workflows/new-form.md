---
name: new-form
description: Workflow for building a form — Zod schema, react-hook-form + shadcn Form, mutation, loading/error states, tests
---

# Workflow: Build a Form

Follow these steps in order when creating a form with validation.

## Prerequisites

Rules load automatically when touching relevant files (`.claude/rules/react.md`, `.claude/rules/typescript.md`, `.claude/rules/testing.md`). Follow the patterns defined there.

## Steps

### 1. Define Zod Schema

Create in `src/features/[feature]/schemas/`:

```typescript
// src/features/[feature]/schemas/[form-name]-schema.ts
import { z } from 'zod'

export const formSchema = z.object({
  name: z.string().min(1, 'Name is required'),
  email: z.string().email('Invalid email address'),
  // ... add fields with validation messages
})

export type FormData = z.infer<typeof formSchema>
```

### 2. Install Required shadcn Components

Ensure the form-related shadcn components are installed:

```bash
npx shadcn@latest add form input label button
# Add more as needed: select, textarea, checkbox, radio-group, switch
```

### 3. Create Form Component

Build the form in `src/features/[feature]/components/`:

Use `useForm` with `zodResolver`, shadcn `<Form>` wrapper, and `<FormField>` for each input.

See the react-hook-form expert skill for the full pattern.

Invoke **frontend-design** skill for the form's visual design.

### 4. Wire Mutation

Connect the form to an API endpoint via `useMutation`:

```tsx
const mutation = useMutation({
  mutationFn: (data: FormData) => api.post('/endpoint', data),
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ['relevant-key'] })
    // navigate, show toast, close dialog, etc.
  },
})
```

### 5. Handle Loading and Error States

- Disable submit button during `form.formState.isSubmitting` or `mutation.isPending`
- Show mutation errors (server-side) above the form or as a toast
- Show field-level validation errors via `<FormMessage />`

### 6. Write Tests

Test three scenarios at minimum:

**Validation errors:**
```tsx
it('shows validation errors for empty required fields', async () => {
  const user = userEvent.setup()
  renderWithProviders(<MyForm onSubmit={vi.fn()} />)
  await user.click(screen.getByRole('button', { name: /submit/i }))
  await waitFor(() => {
    expect(screen.getByText(/name is required/i)).toBeInTheDocument()
  })
})
```

**Successful submission:**
```tsx
it('calls onSubmit with valid data', async () => {
  const user = userEvent.setup()
  const onSubmit = vi.fn()
  renderWithProviders(<MyForm onSubmit={onSubmit} />)
  await user.type(screen.getByLabelText(/name/i), 'John')
  await user.type(screen.getByLabelText(/email/i), 'john@example.com')
  await user.click(screen.getByRole('button', { name: /submit/i }))
  await waitFor(() => {
    expect(onSubmit).toHaveBeenCalledWith({ name: 'John', email: 'john@example.com' })
  })
})
```

**Disabled during submission:**
```tsx
it('disables submit button while submitting', async () => {
  // ... trigger submit, verify button is disabled
})
```

### 7. Verify

- Run `npm test` — all form tests pass
- Run `npx tsc --noEmit` — no type errors
- Test in browser — fill form, submit, verify validation, verify success flow
