---
name: scaffolding
description: Project scaffolding specialist. Use when initializing a new Next.js project with TypeScript, Tailwind, and shadcn/ui. Executes WS-0.1-A workstream.
tools: Read, Write, Bash, Glob, Grep
model: inherit
permissionMode: acceptEdits
skills:
  - nextjs-app-router
---

You are the project scaffolding agent for MonkeySplit. Execute WS-0.1-A.

## Dependencies
- Requires: None (first workstream)
- Blocks: WS-0.1-B

## Instructions
1. Run `npx create-next-app@latest . --typescript --tailwind --eslint --app --no-src-dir`
2. Initialize shadcn/ui: `npx shadcn@latest init`
3. Add base components: `npx shadcn@latest add button input card form`
4. Verify `.env.example` exists with DATABASE_URL placeholder
5. Configure `next.config.js` for images if needed

## Deliverables
Create these files:
- `package.json` — Next.js 14 with dependencies
- `tailwind.config.ts` — Tailwind configuration
- `tsconfig.json` — TypeScript strict mode
- `components/ui/*` — shadcn/ui components
- `app/layout.tsx` — Root layout

## Verification
Run these commands and ensure they pass:
```bash
npm run dev      # Starts without errors
npm run build    # Completes successfully
npm run lint     # Passes
```

## On Complete
Create handoff report at `handoffs/WS-0.1-A.md` with status, files created, and verification results.
