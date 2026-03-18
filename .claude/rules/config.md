---
paths:
  - "vite.config.*"
  - ".env*"
  - "src/lib/env.*"
  - "src/vite-env.d.ts"
---

# Vite & Environment Rules

## Validated Environment with Zod

All env vars go through a single validated `env.ts` file. Never access `import.meta.env` directly in application code.

```ts
// src/lib/env.ts
import { z } from 'zod'

const envSchema = z.object({
  VITE_API_URL: z.string().url(),
  VITE_APP_TITLE: z.string().min(1),
  VITE_ENABLE_ANALYTICS: z
    .string()
    .transform((val) => val === 'true')
    .default('false'),
})

// Validate at startup — fails fast if env is misconfigured
export const env = envSchema.parse(import.meta.env)
```

```ts
// src/vite-env.d.ts — keep in sync with the Zod schema
interface ImportMetaEnv {
  readonly VITE_API_URL: string
  readonly VITE_APP_TITLE: string
  readonly VITE_ENABLE_ANALYTICS?: string
}
```

```ts
// .env
VITE_API_URL=http://localhost:3001
VITE_APP_TITLE=My App
VITE_ENABLE_ANALYTICS=false
```

### Using env vars in application code

```ts
// Always import from env.ts — never access import.meta.env directly
import { env } from '@/lib/env'

const apiUrl = env.VITE_API_URL
const title = env.VITE_APP_TITLE

// In src/lib/api.ts
export const api = {
  get: <T>(path: string) =>
    fetch(`${env.VITE_API_URL}${path}`).then(r => r.json() as Promise<T>),
}
```

**DON'T:**

```ts
// Never access import.meta.env directly in application code
const apiUrl = import.meta.env.VITE_API_URL // ❌ use env.VITE_API_URL from @/lib/env

// Never use process.env in client code
const apiUrl = process.env.API_URL // ❌ use env from @/lib/env

// Never forget the VITE_ prefix (var won't be exposed to client)
// .env
API_URL=http://localhost:3001 // ❌ not available in client — use VITE_API_URL

// Never import Node.js modules in client code
import fs from 'fs' // ❌ not available in browser
import path from 'path' // ❌ not available in browser

// Never commit .env files with secrets
// .env.local with API keys ❌ — add to .gitignore

// Never duplicate env access — single source of truth is src/lib/env.ts
// If you need a new env var: add to .env, add to schema in env.ts, add to vite-env.d.ts
```
