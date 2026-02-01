# MonkeySplit Core Context

Lightweight context loaded for every agent session.

## Project Identity
**MonkeySplit** — Open-source Splitwise clone on the Vercel stack.

## Tech Stack
| Layer | Technology |
|-------|-----------|
| Framework | Next.js 14+ (App Router) |
| Language | TypeScript (strict) |
| Styling | Tailwind CSS + shadcn/ui |
| Database | PostgreSQL (Neon) + Drizzle ORM |
| Auth | Auth.js v5 (Email magic link) |

## Coding Standards
- **TypeScript**: Strict mode, no `any` types
- **Files**: `kebab-case.ts`, Components: `PascalCase.tsx`
- **Database**: UUIDs, soft-delete via `deleted_at`
- **Validation**: Zod schemas for all inputs

## Key Patterns
- Default to Server Components
- Server actions in `app/actions/*.ts`
- Return `{ success: boolean, error?: string, data?: T }`

## Verification Commands
```bash
npm run type-check && npm run lint && npm run test && npm run build
```

## Full Specification
For detailed specs, see `IMPLEMENTATION_PROMPT.md`:
- Section 2: Database Schema
- Section 3: Feature Specifications
- Section 7: Design System
- Section 8: Key UI Components
