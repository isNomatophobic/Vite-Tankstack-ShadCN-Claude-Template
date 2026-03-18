---
name: react-hook-form-expert
description: react-hook-form expertise — useForm + Zod resolver, shadcn Form integration, type inference from schemas
---

# react-hook-form Expert

## Context7 Verification — DO THIS FIRST

Before generating form code, query context7:

```
mcp__context7__query-docs("/react-hook-form/documentation", "<your specific question — useForm options, field registration, resolver setup>")
```

Verify: `useForm` options, `zodResolver` integration, `FormField`/`FormItem` component API from shadcn.

## Core Patterns

### Form Architecture

Every form follows this structure:

1. **Schema** in `src/features/[feature]/schemas/` — Zod schema defines shape + validation
2. **Type** inferred from schema via `z.infer<>` — never duplicate
3. **Component** uses `useForm` with `zodResolver` — paired with shadcn `<Form>` components

### Zod Schema (Single Source of Truth)

```typescript
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
// src/features/auth/components/LoginForm.tsx
import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { loginSchema, type LoginFormData } from '@/features/auth/schemas/login-schema'
import {
  Form,
  FormControl,
  FormField,
  FormItem,
  FormLabel,
  FormMessage,
} from '@/components/ui/form'
import { Input } from '@/components/ui/input'
import { Button } from '@/components/ui/button'

export function LoginForm({ onSubmit }: { onSubmit: (data: LoginFormData) => void }) {
  const form = useForm<LoginFormData>({
    resolver: zodResolver(loginSchema),
    defaultValues: {
      email: '',
      password: '',
    },
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
        <FormField
          control={form.control}
          name="password"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Password</FormLabel>
              <FormControl>
                <Input type="password" {...field} />
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
import { useMutation } from '@tanstack/react-query'
import { api } from '@/lib/api'
import { LoginForm } from './LoginForm'
import type { LoginFormData } from '@/features/auth/schemas/login-schema'

function LoginPage() {
  const loginMutation = useMutation({
    mutationFn: (data: LoginFormData) => api.post('/auth/login', data),
    onSuccess: () => {
      // navigate, show toast, etc.
    },
  })

  return <LoginForm onSubmit={(data) => loginMutation.mutate(data)} />
}
```

### Default Values

Always provide typed `defaultValues`:

```typescript
const form = useForm<FormData>({
  resolver: zodResolver(schema),
  defaultValues: {
    name: '',
    email: '',
    age: 0,
  },
})
```

## Don'ts

- Never use uncontrolled forms without react-hook-form
- Never define validation rules inline (`{ required: true }`) — always Zod schemas
- Never manually type form data — always `z.infer<typeof schema>`
- Never use `<form>` without shadcn's `<Form>` wrapper
- Never skip `defaultValues` — react-hook-form needs them for proper reset/dirty tracking
