---
name: lucide-expert
description: Lucide React icon expertise — tree-shaken imports, size conventions, prop customization
---

# Lucide React Expert

## Context7 Verification

When looking for a specific icon or checking prop API:

```
mcp__context7__query-docs("/websites/lucide_dev_guide_packages", "available icons for <description> OR icon component props")
```

## Core Patterns

### Importing Icons

Always import individual icons — never from a barrel:

```tsx
import { Search, Plus, Trash2, ChevronRight } from 'lucide-react'
```

Each icon is a React component that renders inline SVG. Tree-shaking ensures only imported icons ship to the bundle.

### Size Conventions

Use consistent sizes across the app:

| Context | Size | Example |
|---------|------|---------|
| Inline with text | 16 | `<Search size={16} />` |
| Inside buttons | 16 | `<Button><Plus size={16} /> Add</Button>` |
| Standalone / page icons | 24 | `<Settings size={24} />` |
| Empty states / hero | 48+ | `<Inbox size={48} />` |

### Props

```tsx
<Search
  size={20}           // Width and height in px
  color="currentColor" // Inherits text color by default
  strokeWidth={2}      // Default stroke width
  className="shrink-0" // Tailwind classes work
/>
```

### Usage with shadcn

Icons go inside shadcn components, typically before text:

```tsx
import { Button } from '@/components/ui/button'
import { Plus } from 'lucide-react'

<Button variant="outline" size="sm">
  <Plus size={16} />
  New Item
</Button>
```

For icon-only buttons:

```tsx
<Button variant="ghost" size="icon">
  <Trash2 size={16} />
</Button>
```

## Don'ts

- Never import all icons (`import * as icons from 'lucide-react'`)
- Never use raw SVGs when a Lucide icon exists for the concept
- Never use inconsistent sizes — follow the convention table above
- Never use hardcoded color values — let icons inherit via `currentColor`
