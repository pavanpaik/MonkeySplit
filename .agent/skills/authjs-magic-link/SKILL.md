---
name: "authjs-magic-link"
description: "Provides Auth.js v5 configuration for email magic link authentication including route handlers, schema tables, and middleware. Use this skill when implementing authentication, protecting routes, or accessing user sessions."
---

# Auth.js Magic Link Skill

## When to Use
- Setting up authentication
- Protecting routes with middleware
- Accessing user sessions
- Creating login/logout flows

## Installation
```bash
npm install next-auth@beta @auth/drizzle-adapter resend
```

## Configuration
```typescript
// lib/auth.ts
import NextAuth from 'next-auth';
import Resend from 'next-auth/providers/resend';
import { DrizzleAdapter } from '@auth/drizzle-adapter';
import { db } from './db';

export const { handlers, auth, signIn, signOut } = NextAuth({
  adapter: DrizzleAdapter(db),
  providers: [
    Resend({ from: 'noreply@yourdomain.com' }),
  ],
  pages: { signIn: '/login' },
});
```

## Route Handler
```typescript
// app/api/auth/[...nextauth]/route.ts
import { handlers } from '@/lib/auth';
export const { GET, POST } = handlers;
```

## Required Schema Tables
Auth.js requires: `users`, `accounts`, `sessions`, `verificationTokens`.
See Drizzle adapter docs for exact schema.

## Middleware
```typescript
// middleware.ts
import { auth } from '@/lib/auth';

export default auth((req) => {
  if (!req.auth && req.nextUrl.pathname.startsWith('/app')) {
    return Response.redirect(new URL('/login', req.url));
  }
});

export const config = { matcher: ['/app/:path*'] };
```

## Login Form
```typescript
'use client';
import { signIn } from 'next-auth/react';

export function LoginForm() {
  return (
    <form action={async (formData) => {
      await signIn('resend', { email: formData.get('email') });
    }}>
      <input name="email" type="email" required />
      <button type="submit">Send Magic Link</button>
    </form>
  );
}
```

## Get Session
```typescript
// Server Component
import { auth } from '@/lib/auth';
const session = await auth();

// Client Component
import { useSession } from 'next-auth/react';
const { data: session } = useSession();
```

## Environment Variables
```bash
NEXTAUTH_URL=http://localhost:3000
NEXTAUTH_SECRET=xxx  # openssl rand -base64 32
RESEND_API_KEY=re_xxx
```

## Error Handling

| Error | Cause | Solution |
|-------|-------|----------|
| `NEXTAUTH_URL missing` | Env var not set | Add to `.env.local` |
| `Email failed to send` | Invalid Resend key | Check RESEND_API_KEY |
| `Adapter error` | Schema mismatch | Verify Auth.js tables exist |
| `Callback URL mismatch` | Wrong NEXTAUTH_URL | Match production domain |

## References
- Auth flow: `IMPLEMENTATION_PROMPT.md` Section 3.1
- Security: `IMPLEMENTATION_PROMPT.md` Section 10
