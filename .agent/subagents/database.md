---
name: "Database Subagent"
description: "Executes WS-0.1-B: Database Schema & Drizzle Setup"
tools: ["Read", "Write", "Bash", "Grep"]
context:
  - ".agent/context/core.md"
  - ".agent/context/database.md"
  - ".agent/skills/drizzle-orm/SKILL.md"
---

# WS-0.1-B: Database Schema & Drizzle Setup

## Purpose
Set up Drizzle ORM with PostgreSQL schema for MonkeySplit.

## Dependencies
- **Requires**: WS-0.1-A
- **Blocks**: WS-0.1-C, WS-0.1-D, WS-0.1-E, WS-0.1-F

## Instructions

1. Install: `npm install drizzle-orm @neondatabase/serverless`
2. Install dev: `npm install -D drizzle-kit`
3. Create `drizzle.config.ts`
4. Create schema files in `lib/db/schema/`:
   - `users.ts`, `groups.ts`, `group-members.ts`
   - `expenses.ts`, `expense-participants.ts`
5. Export from `lib/db/schema/index.ts`
6. Create db client in `lib/db/index.ts`
7. Run `npx drizzle-kit generate:pg`
8. Run `npx drizzle-kit push:pg`

## Deliverables
- [ ] `drizzle.config.ts`
- [ ] `lib/db/index.ts`
- [ ] `lib/db/schema/*.ts` — All schema files
- [ ] `drizzle/migrations/*`

## Verification
```bash
npx drizzle-kit studio  # Opens and shows tables
```

## Handoff
Document: UUID primary keys, soft-delete columns, Neon driver choice.
