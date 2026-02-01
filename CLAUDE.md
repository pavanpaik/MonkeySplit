# MonkeySplit

> Open-source Splitwise clone on the Vercel stack

## Tech Stack
- **Framework**: Next.js 14+ (App Router)
- **Language**: TypeScript (strict mode)
- **Styling**: Tailwind CSS + shadcn/ui
- **Database**: PostgreSQL (Neon) + Drizzle ORM
- **Auth**: Auth.js v5 (Email magic link → OAuth later)

## Coding Standards
- Strict TypeScript, no `any` types
- Zod schemas for all input validation
- Files: `kebab-case.ts`, Components: `PascalCase.tsx`
- Database: UUIDs, soft-delete via `deleted_at`

## Key Commands
```bash
npm run type-check && npm run lint && npm run test && npm run build
```

## Subagents & Skills
- **Subagents**: `.agent/subagents/*.md` — Workstream executors
- **Skills**: `.agent/skills/*/SKILL.md` — Modular capabilities
- **MCP Servers**: `.mcp/config.json` — External tool integrations
- **Plugins**: See `PLUGINS.md` for recommended plugins by release

## Current Phase
**Alpha 0.1.0** — Groups, equal-split expenses, balances.
See `.agent/subagents/` for workstream definitions.

