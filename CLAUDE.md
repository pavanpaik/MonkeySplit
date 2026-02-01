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

## Architecture

### Agents (`.claude/agents/`)
Reusable specialists — use `/agents` in Claude CLI:
- `scaffolder`, `database-architect`, `auth-implementer`
- `tester`, `reviewer`, `debugger`, `feature-builder`

### Skills (`.agent/skills/`)
Pattern libraries for specific technologies.

### Workstreams (`.agent/subagents/`)
Project-specific task definitions (WS-0.1-A through WS-0.1-G).

### MCP (`.mcp/config.json`)
External tool integrations (Neon, GitHub).
