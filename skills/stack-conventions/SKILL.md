---
name: stack-conventions
description: "Standards and conventions for the application stack. Covers Next.js 15 App Router, Supabase (auth/db/storage), Neon Postgres, Hono, Drizzle ORM, fp-ts, Zod, Vitest, and Playwright."
risk: safe
source: custom
date_added: "2026-05-27"
tags: ['stack', 'conventions', 'nextjs', 'supabase', 'neon', 'drizzle', 'fp-ts', 'tailwind']
tools: ["claude", "cursor", "gemini", "antigravity"]
---

# Application Stack Conventions

This skill documents the coding standards, architectural patterns, and folder conventions for our specific stack.

## Technical Stack
- **Framework**: Next.js 15 App Router (React Server Components default).
- **Backend & Compute**: Hono (for Edge APIs/Cloudflare Workers), Next.js Route Handlers.
- **Database**: Neon serverless Postgres with Drizzle ORM.
- **Client/Auth**: Supabase (realtime, auth, storage).
- **Logic & Control**: fp-ts (functional programming patterns), Zod (validation).
- **Styling**: shadcn/ui + Tailwind CSS v4 (CSS-first configuration).
- **Testing**: Vitest (unit/integration), Playwright (E2E).

## Folder Conventions
```
├── app/                  # Next.js App Router (pages & server actions)
├── components/           # Reusable UI components (shadcn/ui)
├── server/               # Backend logic, database config, schemas
│   ├── db/               # Drizzle schemas, migrations, client
│   └── api/              # Hono app and route handlers
├── lib/                  # Shared utilities and fp-ts pipelines
├── tests/                # E2E and unit tests
```

## Data Mutating & Fetching Guidelines
- **RSC (React Server Components)**: Fetch data directly in Server Components using `await db.select(...)`. No client-side `useEffect` for initial load.
- **Server Actions**: Use Server Actions for mutations. Wrap them in a robust input-validation layer using Zod.
- **State Management**: Prefer server state using search parameters or URL state. Use Zustand for complex client-only state.
- **Tailwind v4**: Configure theme tokens inside `@theme` in `index.css` rather than `tailwind.config.js`.

## Error Handling Pattern (fp-ts)
Always handle errors as values using `Either` or `TaskEither`.
```typescript
import * as E from 'fp-ts/Either';
import * as TE from 'fp-ts/TaskEither';
import { pipe } from 'fp-ts/function';

// Database query wrapped in TaskEither
const getUser = (id: string): TE.TaskEither<Error, User> => 
  TE.tryCatch(
    async () => {
      const user = await db.query.users.findFirst({ where: eq(users.id, id) });
      if (!user) throw new Error("User not found");
      return user;
    },
    (reason) => reason as Error
  );
```
