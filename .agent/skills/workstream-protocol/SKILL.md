---
name: "Workstream Protocol"
description: "Workstream execution and handoff protocol"
---

# Workstream Protocol Skill

## Execution Flow

1. **Read** workstream definition from subagent config
2. **Load** referenced skills for patterns
3. **Execute** each implementation step
4. **Verify** using verification checklist
5. **Generate** handoff report

## Handoff Report Format

Create `handoffs/WS-X.Y-Z.md` after completing a workstream:

```markdown
# WS-0.1-B Handoff

## Status: COMPLETE ✅

## Completed At
2024-01-15T14:30:00Z

## Files Created
- `lib/db/index.ts` — Database client singleton
- `lib/db/schema/users.ts` — Users table schema
- `lib/db/schema/groups.ts` — Groups table schema

## Files Modified
- `package.json` — Added drizzle-orm dependencies

## Decisions Made
- Used UUID for all primary keys (deterministic, no conflicts)
- Added soft-delete columns to all entities
- Chose Neon serverless driver for edge compatibility

## Verification Results
- [x] `npx drizzle-kit studio` opens and shows tables
- [x] Can insert and query test data
- [x] All FK relationships valid

## Known Issues / Tech Debt
- No indexes defined yet (defer to 0.6.0)
- Missing created_by on some tables

## Notes for Dependent Workstreams
- DB client: `import { db } from '@/lib/db'`
- Schemas: `import { users, groups } from '@/lib/db/schema'`
- Query pattern: `db.query.users.findMany()`
```

## Verification Commands

Run after completing any workstream:
```bash
npm run type-check    # TypeScript
npm run lint          # ESLint
npm run test          # Unit tests
npm run build         # Production build
```

## Conflict Resolution

1. **File Ownership**: Each workstream owns specific files
2. **Shared Files**: `package.json`, `lib/db/schema/index.ts` — append only
3. **Blocking**: If must modify shared code, document and coordinate
4. **Integration**: WS-X.Y-G resolves all conflicts at merge

## Workstream States

| State | Description |
|-------|-------------|
| `PENDING` | Not started, waiting for dependencies |
| `IN_PROGRESS` | Currently being executed |
| `BLOCKED` | Waiting for clarification or dependency |
| `COMPLETE` | All verification passed |
| `FAILED` | Verification failed, needs retry |
