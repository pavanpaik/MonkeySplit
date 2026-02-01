---
name: "Auth Subagent"
description: "Executes WS-0.1-C: Auth.js Email Magic Link"
tools: ["Read", "Write", "Bash", "Grep"]
context:
  - "IMPLEMENTATION_PROMPT.md"
  - ".agent/skills/authjs-magic-link/SKILL.md"
---

# WS-0.1-C: Auth.js Email Magic Link

## Purpose
Implement authentication with email magic link using Auth.js v5.

## Dependencies
- **Requires**: WS-0.1-B
- **Blocks**: WS-0.1-G

## Instructions

1. Install: `npm install next-auth@beta @auth/drizzle-adapter resend`
2. Create auth schema tables in `lib/db/schema/auth.ts`
3. Run migration for auth tables
4. Create `lib/auth.ts` with Drizzle adapter + Resend provider
5. Create `app/api/auth/[...nextauth]/route.ts`
6. Create `app/(auth)/login/page.tsx`
7. Create `app/(auth)/verify/page.tsx`
8. Create `middleware.ts` for route protection
9. Add session provider to root layout

## Deliverables
- [ ] `lib/auth.ts`
- [ ] `app/api/auth/[...nextauth]/route.ts`
- [ ] `app/(auth)/login/page.tsx`
- [ ] `app/(auth)/verify/page.tsx`
- [ ] `middleware.ts`
- [ ] `lib/db/schema/auth.ts`

## Verification
- [ ] Login sends magic link email
- [ ] Clicking link creates session
- [ ] Protected routes redirect to login
