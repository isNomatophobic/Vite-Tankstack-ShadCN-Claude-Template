# Rules & Pitfalls Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add path-scoped rule files with DO/DON'T code examples and expand CLAUDE.md base rules.

**Architecture:** Three-tier rules: CLAUDE.md (universal one-liners, always loaded) → `.claude/rules/` (library-specific code examples, path-scoped) → `.claude/skills/` (rich workflows, on-demand). This plan creates tier 2 and expands tier 1.

**Tech Stack:** Markdown, Claude Code rules system (YAML frontmatter with `paths:`)

**Spec:** `docs/superpowers/specs/2026-03-18-rules-and-pitfalls-design.md`

---

### Task 1: Expand CLAUDE.md Global Conventions

**Files:**
- Modify: `CLAUDE.md:57-72` (Global Conventions section)

- [ ] **Step 1: Add missing base rules to Global Conventions**

Add these bullets to the existing list in `## Global Conventions`, after the existing TypeScript bullet (line 59):

```markdown
- **TypeScript (strict):** No `as` type assertions, no `@ts-expect-error`, no `!` non-null assertions — fix the type or narrow properly
- **Exports:** Named exports only — no `export default`. Use `@/` path alias — no deep relative paths. No barrel files (`index.ts` re-exporting).
- **React:** No `useEffect` for derived state — compute during render. No `index` as `key` in reorderable lists. Memoize objects/callbacks passed as props.
```

These expand the existing TypeScript bullet and add React/export rules that are universal across all files. The existing bullets remain unchanged.

- [ ] **Step 2: Commit**

```bash
git add CLAUDE.md
git commit -m "chore: expand global conventions with strict TypeScript, export, and React base rules"
```

---

### Task 2: Create `.claude/rules/react.md`

**Files:**
- Create: `.claude/rules/react.md`

- [ ] **Step 1: Create the rule file**

```markdown
---
paths:
  - "**/*.tsx"
---

# React Rules

**DO:**

```tsx
// Derived state — compute during render
const fullName = `${user.firstName} ${user.lastName}`;

// Server data — use TanStack Query
const { data } = useQuery(userQueryOptions(id));

// Stable keys from data
{users.map(user => <UserCard key={user.id} user={user} />)}

// Memoize objects/callbacks passed as props
const style = useMemo(() => ({ color: theme.primary }), [theme.primary]);
const handleClick = useCallback((id: string) => selectUser(id), [selectUser]);

// When useEffect IS appropriate, always clean up
useEffect(() => {
  const handler = (e: KeyboardEvent) => { ... };
  window.addEventListener('keydown', handler);
  return () => window.removeEventListener('keydown', handler);
}, []);
```

**DON'T:**

```tsx
// Never useEffect for data fetching
useEffect(() => {
  fetch('/api/users').then(r => r.json()).then(setUsers);
}, []); // ❌ use TanStack Query

// Never useEffect + useState for derived state
const [fullName, setFullName] = useState('');
useEffect(() => {
  setFullName(`${user.firstName} ${user.lastName}`);
}, [user]); // ❌ compute during render

// Never useEffect to sync props to state
const [localValue, setLocalValue] = useState(props.value);
useEffect(() => {
  setLocalValue(props.value);
}, [props.value]); // ❌ use the prop directly

// Never use index as key in reorderable lists
{users.map((user, i) => <UserCard key={i} />)} // ❌ use user.id

// Never use random values as keys
{items.map(item => <Row key={Math.random()} />)} // ❌ re-mounts every render
{items.map(item => <Row key={crypto.randomUUID()} />)} // ❌ same problem

// Never create objects/functions inline in JSX (causes re-renders)
<Component style={{ color: 'red' }} onClick={() => handleClick(id)} /> // ❌ memoize
```
```

- [ ] **Step 2: Commit**

```bash
git add .claude/rules/react.md
git commit -m "chore: add React rules with DO/DON'T patterns"
```

---

### Task 3: Create `.claude/rules/tanstack-query.md`

**Files:**
- Create: `.claude/rules/tanstack-query.md`

- [ ] **Step 1: Create the rule file**

```markdown
---
paths:
  - "src/**/queries.*"
  - "src/**/api.*"
  - "src/lib/api.*"
  - "src/**/*Query*"
  - "src/**/*query*"
---

# TanStack Query Rules

**DO:**

```ts
// Define query options in a factory
export const usersQueryOptions = () =>
  queryOptions({
    queryKey: ['users'],
    queryFn: () => api.get<User[]>('/users'),
  });

// Use in components
const { data, isLoading, error } = useQuery(usersQueryOptions());

// Handle all states
if (isLoading) return <Skeleton />;
if (error) return <ErrorMessage error={error} />;
return <UserList users={data} />;

// Prefetch in route loaders
export const Route = createFileRoute('/users/')({
  loader: ({ context }) =>
    context.queryClient.ensureQueryData(usersQueryOptions()),
});

// Use enabled for conditional queries
const { data } = useQuery({
  ...userQueryOptions(userId!),
  enabled: !!userId,
});

// Invalidate after mutations
const mutation = useMutation({
  mutationFn: (user: CreateUser) => api.post('/users', user),
  onSuccess: () => queryClient.invalidateQueries({ queryKey: ['users'] }),
});
```

**DON'T:**

```ts
// Never raw fetch in components
const [users, setUsers] = useState([]);
useEffect(() => {
  fetch('/api/users').then(r => r.json()).then(setUsers);
}, []); // ❌ use TanStack Query

// Never hardcode query keys as loose strings
useQuery({ queryKey: ['users'], queryFn: fetchUsers }); // ❌ use queryOptions() factory

// Never call hooks conditionally
if (userId) {
  const { data } = useQuery(userQueryOptions(userId)); // ❌ violates Rules of Hooks
} // use enabled option instead

// Never mutate query data directly
const { data } = useQuery(usersQueryOptions());
data.push(newUser); // ❌ mutates cache — use mutations

// Never skip error/loading states
const { data } = useQuery(usersQueryOptions());
return <div>{data.name}</div>; // ❌ crashes when loading/error

// Never bypass the API client
useQuery({
  queryKey: ['users'],
  queryFn: () => fetch('/api/users').then(r => r.json()), // ❌ use api.get()
});
```
```

- [ ] **Step 2: Commit**

```bash
git add .claude/rules/tanstack-query.md
git commit -m "chore: add TanStack Query rules with DO/DON'T patterns"
```

---

### Task 4: Create `.claude/rules/tanstack-router.md`

**Files:**
- Create: `.claude/rules/tanstack-router.md`

- [ ] **Step 1: Create the rule file**

```markdown
---
paths:
  - "src/routes/**/*"
---

# TanStack Router Rules

**DO:**

```ts
// File-based routes — let the Vite plugin generate the route tree
// src/routes/users/$userId.tsx
export const Route = createFileRoute('/users/$userId')({
  component: UserPage,
});

// Search params validated with Zod
const searchSchema = z.object({
  page: z.number().default(1),
  filter: z.string().optional(),
});

export const Route = createFileRoute('/users/')({
  validateSearch: searchSchema,
});

// Declarative navigation
<Link to="/users/$userId" params={{ userId: '123' }}>View</Link>

// Programmatic navigation
const navigate = useNavigate();
navigate({ to: '/users/$userId', params: { userId: '123' } });

// Let TypeScript infer Link params — type errors mean wrong route/params
```

**DON'T:**

```ts
// Never edit routeTree.gen.ts — it's auto-generated
// Never create manual route trees
const router = createRouter({ routeTree: manualTree }); // ❌

// Never use window.location for navigation
window.location.href = '/users'; // ❌ full page reload

// Never use <a href=""> for internal links
<a href="/users">Users</a> // ❌ use <Link>

// Never use unvalidated search params
const params = new URLSearchParams(window.location.search); // ❌ use validateSearch + Zod

// Never silence Link type errors
<Link to={path as any} params={params as any} /> // ❌ fix the types
```
```

- [ ] **Step 2: Commit**

```bash
git add .claude/rules/tanstack-router.md
git commit -m "chore: add TanStack Router rules with DO/DON'T patterns"
```

---

### Task 5: Create `.claude/rules/react-hook-form.md`

**Files:**
- Create: `.claude/rules/react-hook-form.md`

- [ ] **Step 1: Create the rule file**

```markdown
---
paths:
  - "src/**/form*"
  - "src/**/*Form*"
  - "src/**/*-form*"
---

# react-hook-form Rules

**DO:**

```ts
// Schema is the single source of truth
const loginSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
});
type LoginForm = z.infer<typeof loginSchema>;

// react-hook-form + zodResolver
const form = useForm<LoginForm>({
  resolver: zodResolver(loginSchema),
  defaultValues: { email: '', password: '' },
});

// Wire mutation errors to form
const mutation = useMutation({
  mutationFn: (data: LoginForm) => api.post('/auth/login', data),
  onError: (error) => {
    if (error.field) {
      form.setError(error.field, { message: error.message });
    } else {
      form.setError('root', { message: error.message });
    }
  },
});
```

**DON'T:**

```ts
// Never duplicate types manually
type LoginForm = { email: string; password: string }; // ❌ duplicates schema
const loginSchema = z.object({ email: z.string(), password: z.string() });

// Never validate manually
const handleSubmit = (data) => {
  if (!data.email.includes('@')) { ... } // ❌ use Zod
};

// Never use uncontrolled forms without schema validation
<form onSubmit={handleSubmit}>
  <input name="email" /> {/* ❌ use react-hook-form */}
</form>

// Never forget defaultValues (causes uncontrolled → controlled warning)
const form = useForm<LoginForm>({
  resolver: zodResolver(loginSchema),
  // ❌ missing defaultValues
});

// Never ignore server-side errors from mutations
mutation.mutate(data); // ❌ wire onError to form.setError
```
```

- [ ] **Step 2: Commit**

```bash
git add .claude/rules/react-hook-form.md
git commit -m "chore: add react-hook-form rules with DO/DON'T patterns"
```

---

### Task 6: Create `.claude/rules/shadcn.md`

**Files:**
- Create: `.claude/rules/shadcn.md`

- [ ] **Step 1: Create the rule file**

```markdown
---
paths:
  - "src/components/**/*"
  - "src/features/**/components/**/*"
---

# shadcn/ui Rules

**DO:**

```tsx
// Install shadcn components via CLI
// npx shadcn@latest add button card dialog

// Compose shadcn primitives into domain components
import { Button } from '@/components/ui/button';
import { Card, CardHeader, CardContent } from '@/components/ui/card';

function UserCard({ user }: { user: User }) {
  return (
    <Card>
      <CardHeader>{user.name}</CardHeader>
      <CardContent>{user.email}</CardContent>
    </Card>
  );
}

// Domain components go in src/components/shared/ or src/features/*/components/
```

**DON'T:**

```tsx
// Never build custom UI that shadcn already provides
function MyButton({ children, ...props }) { // ❌
  return <button className="bg-blue-500 ..." {...props}>{children}</button>;
}

// Never copy-paste shadcn source code — use the CLI
// Copying from shadcn docs or GitHub ❌ — run `npx shadcn@latest add`

// Never prop-drill more than 2 levels — use composition or context
<Parent user={user}>
  <Child user={user}>
    <GrandChild user={user} /> {/* ❌ use context or composition */}
  </Child>
</Parent>
```
```

- [ ] **Step 2: Commit**

```bash
git add .claude/rules/shadcn.md
git commit -m "chore: add shadcn/ui rules with DO/DON'T patterns"
```

---

### Task 7: Create `.claude/rules/testing.md`

**Files:**
- Create: `.claude/rules/testing.md`

- [ ] **Step 1: Create the rule file**

```markdown
---
paths:
  - "**/*.test.*"
---

# Testing Rules

**DO:**

```tsx
// Colocate tests next to source
// src/features/users/UserCard.tsx
// src/features/users/UserCard.test.tsx

// Use the custom render with providers
import { render } from '@/test/render';

// Query by accessibility role first
const button = screen.getByRole('button', { name: /submit/i });

// Use userEvent over fireEvent
const user = userEvent.setup();
await user.click(button);

// Wait for async content
const name = await screen.findByText('John');

// Test behavior from the user's perspective
expect(screen.getByRole('alert')).toHaveTextContent('Invalid email');
```

**DON'T:**

```tsx
// Never use getByTestId as first choice
screen.getByTestId('submit-btn'); // ❌ use getByRole('button', { name: /submit/i })

// Never use fireEvent when userEvent works
fireEvent.click(button); // ❌ use await user.click(button)

// Never forget to await async operations
screen.getByText('John'); // ❌ if data is async, use findByText

// Never test implementation details
expect(component.state.loading).toBe(true); // ❌
expect(setState).toHaveBeenCalledWith(true); // ❌

// Never put tests in a separate __tests__ directory
// src/__tests__/UserCard.test.tsx  ❌ — colocate with source file

// Never use getBy + waitFor when findBy works
await waitFor(() => {
  expect(screen.getByText('John')).toBeInTheDocument(); // ❌ use findByText
});
```
```

- [ ] **Step 2: Commit**

```bash
git add .claude/rules/testing.md
git commit -m "chore: add testing rules with DO/DON'T patterns"
```

---

### Task 8: Create `.claude/rules/lucide.md`

**Files:**
- Create: `.claude/rules/lucide.md`

- [ ] **Step 1: Create the rule file**

```markdown
---
paths:
  - "**/*.tsx"
---

# Lucide Icons Rules

**DO:**

```tsx
// Individual named imports
import { Search } from 'lucide-react';
import { ChevronDown } from 'lucide-react';

// Size conventions
<Search size={16} />      {/* inline text, buttons */}
<Search size={24} />      {/* standalone, nav */}
<Search size={48} />      {/* hero sections, empty states */}

// Theme-aware with currentColor (default)
<Search className="text-muted-foreground" />
```

**DON'T:**

```tsx
// Never import all icons (kills tree-shaking)
import * as Icons from 'lucide-react'; // ❌
import { icons } from 'lucide-react'; // ❌

// Never use raw SVGs when Lucide has the icon
<svg viewBox="0 0 24 24">...</svg> // ❌ check Lucide first

// Never hardcode colors — use className for theming
<Search color="#ff0000" /> // ❌ use className="text-destructive"
```
```

- [ ] **Step 2: Commit**

```bash
git add .claude/rules/lucide.md
git commit -m "chore: add Lucide icons rules with DO/DON'T patterns"
```

---

### Task 9: Create `.claude/rules/zod.md`

**Files:**
- Create: `.claude/rules/zod.md`

- [ ] **Step 1: Create the rule file**

```markdown
---
paths:
  - "src/**/schemas.*"
  - "src/**/*.schema.*"
  - "src/**/validators.*"
---

# Zod Rules

**DO:**

```ts
// Infer types from schemas — single source of truth
const userSchema = z.object({
  id: z.string().uuid(),
  name: z.string().min(1),
  email: z.string().email(),
});
type User = z.infer<typeof userSchema>;

// Validate API responses
const response = await api.get('/users');
const users = z.array(userSchema).parse(response);

// Use safeParse when you need to handle errors gracefully
const result = userSchema.safeParse(data);
if (!result.success) {
  console.error(result.error.flatten());
  return;
}

// Use strict() to reject unknown properties
const createUserSchema = userSchema.omit({ id: true }).strict();
```

**DON'T:**

```ts
// Never duplicate types alongside schemas
type User = { id: string; name: string; email: string }; // ❌ use z.infer<>

// Never skip validation on API responses
const users: User[] = await api.get('/users'); // ❌ validate with .parse()

// Never use .passthrough() unless explicitly needed
const schema = z.object({ name: z.string() }).passthrough(); // ❌ use .strict()

// Never use .transform() in schemas shared between forms and API validation
const schema = z.object({
  price: z.string().transform(Number), // ❌ breaks defaultValues typing in forms
}); // create separate schemas for form input vs API payload
```
```

- [ ] **Step 2: Commit**

```bash
git add .claude/rules/zod.md
git commit -m "chore: add Zod rules with DO/DON'T patterns"
```

---

### Task 10: Create `.claude/rules/vite.md`

**Files:**
- Create: `.claude/rules/vite.md`

- [ ] **Step 1: Create the rule file**

```markdown
---
paths:
  - "vite.config.*"
  - ".env*"
  - "src/lib/env.*"
---

# Vite / Environment Rules

**DO:**

```ts
// Use import.meta.env for environment variables
const apiUrl = import.meta.env.VITE_API_URL;

// Prefix client-exposed vars with VITE_
// .env
VITE_API_URL=http://localhost:3001
VITE_APP_TITLE=My App

// Type your env vars
// src/vite-env.d.ts
interface ImportMetaEnv {
  readonly VITE_API_URL: string;
  readonly VITE_APP_TITLE: string;
}
```

**DON'T:**

```ts
// Never use process.env in client code
const apiUrl = process.env.API_URL; // ❌ use import.meta.env

// Never forget the VITE_ prefix (var won't be exposed to client)
// .env
API_URL=http://localhost:3001 // ❌ not available in client — use VITE_API_URL

// Never import Node.js modules in client code
import fs from 'fs'; // ❌ not available in browser
import path from 'path'; // ❌ not available in browser

// Never commit .env files with secrets
// .env.local with API keys ❌ — add to .gitignore
```
```

- [ ] **Step 2: Commit**

```bash
git add .claude/rules/vite.md
git commit -m "chore: add Vite/environment rules with DO/DON'T patterns"
```

---

### Task 11: Update CLAUDE.md with rules tier documentation

**Files:**
- Modify: `CLAUDE.md` (add note about `.claude/rules/` to Skill Routing or new section)

- [ ] **Step 1: Add rules tier reference below Global Conventions**

Add a brief note after the Global Conventions section explaining the three-tier system, so users of the template understand the architecture:

```markdown
## Rules (`.claude/rules/`)

Path-scoped rule files with DO/DON'T code examples. These load automatically when you touch matching files — no need to invoke them. They enforce the conventions above with concrete patterns.

| Rule file | Activates on |
|---|---|
| `react.md` | `**/*.tsx` |
| `tanstack-query.md` | `src/**/queries.*`, `src/**/api.*`, `src/lib/api.*` |
| `tanstack-router.md` | `src/routes/**/*` |
| `react-hook-form.md` | `src/**/form*`, `src/**/*Form*` |
| `shadcn.md` | `src/components/**/*`, `src/features/**/components/**/*` |
| `testing.md` | `**/*.test.*` |
| `lucide.md` | `**/*.tsx` |
| `zod.md` | `src/**/schemas.*`, `src/**/*.schema.*` |
| `vite.md` | `vite.config.*`, `.env*` |
```

Insert this between `## Global Conventions` and `## Skill Routing`.

- [ ] **Step 2: Commit**

```bash
git add CLAUDE.md
git commit -m "chore: document rules tier in CLAUDE.md"
```

---

### Task 12: Final verification

- [ ] **Step 1: Verify all files exist**

```bash
ls -la .claude/rules/
```

Expected: 9 `.md` files (react, tanstack-query, tanstack-router, react-hook-form, shadcn, testing, lucide, zod, vite)

- [ ] **Step 2: Verify frontmatter format**

```bash
head -5 .claude/rules/*.md
```

Expected: Each file starts with `---`, has `paths:`, and closes with `---`

- [ ] **Step 3: Verify CLAUDE.md structure**

Read CLAUDE.md and confirm:
- Global Conventions has expanded base rules
- New Rules section exists between Global Conventions and Skill Routing
- No broken markdown formatting
