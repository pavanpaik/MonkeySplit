---
name: "Auth.js Magic Link"
description: "Auth.js v5 with email magic link authentication"
---

# Auth.js Magic Link Skill

## Installation
```bash
npm install next-auth@beta @auth/drizzle-adapter resend
```

## Configuration
```typescript
// lib/auth.ts
import NextAuth from 'next-auth';
import { DrizzleAdapter } from '@auth/drizzle-adapter';
import Resend from 'next-auth/providers/resend';
import { db } from '@/lib/db';

export const { handlers, auth, signIn, signOut } = NextAuth({
  adapter: DrizzleAdapter(db),
  providers: [Resend({ from: 'noreply@monkeysplit.app' })],
  pages: { signIn: '/login', verifyRequest: '/verify' },
  callbacks: {
    session: ({ session, user }) => ({
      ...session,
      user: { ...session.user, id: user.id },
    }),
  },
});
```

## Route Handler
```typescript
// app/api/auth/[...nextauth]/route.ts
import { handlers } from '@/lib/auth';
export const { GET, POST } = handlers;
```

## Auth Schema Tables
Required tables: `accounts`, `sessions`, `verification_tokens`

## Middleware
```typescript
// middleware.ts
import { auth } from '@/lib/auth';
import { NextResponse } from 'next/server';

export default auth((req) => {
  if (!req.auth && req.nextUrl.pathname.startsWith('/groups')) {
    return NextResponse.redirect(new URL('/login', req.url));
  }
});
```

## Login Form
```typescript
// app/(auth)/login/page.tsx
import { signIn } from '@/lib/auth';

export default function LoginPage() {
  return (
    <form action={async (formData) => {
      'use server';
      await signIn('resend', formData);
    }}>
      <input type="email" name="email" required />
      <button type="submit">Sign in</button>
    </form>
  );
}
```

## Getting Session
```typescript
// Server Component or Server Action
const session = await auth();
if (!session?.user) throw new Error('Unauthorized');
const userId = session.user.id;
```

## Environment Variables
```env
NEXTAUTH_URL=http://localhost:3000
NEXTAUTH_SECRET=your-secret-key
RESEND_API_KEY=re_...
```
