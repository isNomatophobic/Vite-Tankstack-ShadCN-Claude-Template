---
paths:
  - "vite.config.*"
  - ".env*"
  - "src/lib/env.*"
  - "src/vite-env.d.ts"
---

# Vite & Environment Rules

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
