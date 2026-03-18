---
name: shadcn-ui-expert
description: shadcn/ui expertise — CLI component installation, composition, theming, frontend-design skill integration
---

# shadcn/ui Expert

## Context7 Verification — DO THIS FIRST

Before using any shadcn component, query context7:

```
mcp__context7__query-docs("/shadcn/ui", "<component name> usage props variants examples")
```

Verify: Available components, current prop APIs, variant options, composition patterns.

## Core Patterns

### Installing Components

Always use the CLI — never copy code from the website:

```bash
npx shadcn@latest add button
npx shadcn@latest add card
npx shadcn@latest add dialog
npx shadcn@latest add input
npx shadcn@latest add form    # Required for react-hook-form integration
```

Install multiple at once:
```bash
npx shadcn@latest add button card dialog input form table
```

### Using Components

```tsx
import { Button } from '@/components/ui/button'
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card'

function MyComponent() {
  return (
    <Card>
      <CardHeader>
        <CardTitle>Title</CardTitle>
      </CardHeader>
      <CardContent>
        <Button variant="outline" size="sm">Click me</Button>
      </CardContent>
    </Card>
  )
}
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

shadcn uses CSS variables for theming. Customize in `src/index.css` (or equivalent globals file):

```css
:root {
  --primary: 222.2 47.4% 11.2%;
  --primary-foreground: 210 40% 98%;
  /* etc. */
}
```

Always use semantic tokens (`bg-primary`, `text-muted-foreground`), never hardcoded colors.

### Icons with Lucide

shadcn is configured with `iconLibrary: "lucide"`. Icons are used alongside shadcn components:

```tsx
import { Button } from '@/components/ui/button'
import { Plus } from 'lucide-react'

<Button>
  <Plus size={16} />
  Add Item
</Button>
```

For icon-only buttons:

```tsx
<Button variant="ghost" size="icon">
  <Trash2 size={16} />
</Button>
```

### Frontend Design Integration

When building UI, ALWAYS invoke the **frontend-design** skill (`frontend-design:frontend-design`) for production-grade output. Don't generate generic-looking UI — use the skill to ensure high design quality.

## Don'ts

- Never edit files in `src/components/ui/` manually — they're managed by the CLI
- Never copy component code from the shadcn website — use `npx shadcn@latest add`
- Never use raw HTML elements (`<button>`, `<input>`, `<table>`) when shadcn has an equivalent
- Never hardcode colors — use CSS variable tokens
- Never skip the frontend-design skill when building visible UI
