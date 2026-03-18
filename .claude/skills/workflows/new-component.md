---
name: new-component
description: Workflow for creating a reusable shared component — composing shadcn/ui primitives with props, variants, and a colocated test
---

# Workflow: Add a Shared Component

Follow these steps when creating a reusable component in `src/components/shared/`.

## Prerequisites

Rules load automatically when touching relevant files (`.claude/rules/react.md`, `.claude/rules/typescript.md`, `.claude/rules/testing.md`). Follow the patterns defined there.

## Steps

### 1. Determine Location and Name

Shared components go in `src/components/shared/`:

```
src/components/shared/
  StatusBadge.tsx
  StatusBadge.test.tsx
```

Use PascalCase for the filename. If the component grows complex enough to need multiple files, promote it to a directory:

```
src/components/shared/DataTable/
  DataTable.tsx
  DataTable.test.tsx
  DataTablePagination.tsx
```

### 2. Define Props Interface

```tsx
interface StatusBadgeProps {
  status: 'active' | 'inactive' | 'pending'
  className?: string
}
```

- Use an explicit interface, not inline types
- Include `className?: string` if the component renders a visible element (allows consumer styling via `cn()`)
- Infer from Zod schema via `z.infer<>` if the props map to a domain type

### 3. Build the Component

Invoke the **frontend-design** skill (`frontend-design:frontend-design`) for the visual design.

Compose from shadcn/ui primitives — never build raw HTML when a shadcn component exists:

```tsx
import { Badge } from '@/components/ui/badge'
import { cn } from '@/lib/utils'

export function StatusBadge({ status, className }: StatusBadgeProps) {
  return (
    <Badge
      variant={status === 'active' ? 'default' : 'secondary'}
      className={cn(status === 'inactive' && 'opacity-50', className)}
    >
      {status}
    </Badge>
  )
}
```

Rules:
- Named export only — no `export default`
- Use `cn()` to merge classNames
- Use Lucide icons with individual imports if needed

### 4. Write Test

Create a colocated test file:

```tsx
import { describe, it, expect } from 'vitest'
import { screen } from '@testing-library/react'
import { renderWithProviders } from '@/test/test-utils'
import { StatusBadge } from './StatusBadge'

describe('StatusBadge', () => {
  it('renders the status text', () => {
    renderWithProviders(<StatusBadge status="active" />)
    expect(screen.getByText('active')).toBeInTheDocument()
  })

  it('applies secondary variant for inactive status', () => {
    renderWithProviders(<StatusBadge status="inactive" />)
    expect(screen.getByText('inactive')).toBeInTheDocument()
  })
})
```

### 5. Verify

- Run `npm test` — test passes
- Run `npx tsc --noEmit` — no type errors
- Import and render the component somewhere to verify visually
