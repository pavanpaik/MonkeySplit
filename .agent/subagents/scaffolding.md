---
name: "Scaffolding Subagent"
description: "Executes WS-0.1-A: Project Scaffolding"
tools: ["Read", "Write", "Bash", "Grep"]
context:
  - "IMPLEMENTATION_PROMPT.md"
---

# WS-0.1-A: Project Scaffolding

## Purpose
Initialize Next.js 14 project with TypeScript, Tailwind, and shadcn/ui.

## Dependencies
- **Requires**: None (first workstream)
- **Blocks**: WS-0.1-B

## Instructions

1. Run `npx create-next-app@latest . --typescript --tailwind --eslint --app --src-dir=false`
2. Initialize shadcn/ui: `npx shadcn-ui@latest init`
3. Add base components: `npx shadcn-ui@latest add button input card form`
4. Create `.env.example` with DATABASE_URL placeholder
5. Configure `next.config.js` for images

## Deliverables
- [ ] `package.json` — Next.js 14 with dependencies
- [ ] `tailwind.config.ts` — Tailwind configuration
- [ ] `tsconfig.json` — TypeScript strict mode
- [ ] `components/ui/*` — shadcn/ui components
- [ ] `app/layout.tsx` — Root layout
- [ ] `.env.example` — Environment template

## Verification
```bash
npm run dev      # Starts without errors
npm run build    # Completes successfully
npm run lint     # Passes
```

## Handoff
Generate handoff report per workstream-protocol skill.
