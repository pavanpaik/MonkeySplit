---
name: "drizzle-orm"
description: "Provides Drizzle ORM patterns for PostgreSQL with Neon including schema definitions, database operations, and migrations. Use this skill when creating database tables, writing queries, or managing schema changes."
---

# Drizzle ORM Skill

## When to Use
- Defining database schema
- Writing database queries
- Running migrations
- Setting up database client

## Setup
```typescript
// drizzle.config.ts
import { defineConfig } from 'drizzle-kit';
export default defineConfig({
  schema: './lib/db/schema/index.ts',
  out: './drizzle/migrations',
  dialect: 'postgresql',
  dbCredentials: { url: process.env.DATABASE_URL! },
});
```

## Database Client
```typescript
// lib/db/index.ts
import { neon } from '@neondatabase/serverless';
import { drizzle } from 'drizzle-orm/neon-http';
import * as schema from './schema';

const sql = neon(process.env.DATABASE_URL!);
export const db = drizzle(sql, { schema });
```

## Schema Pattern
```typescript
// lib/db/schema/users.ts
import { pgTable, uuid, varchar, timestamp } from 'drizzle-orm/pg-core';

export const users = pgTable('users', {
  id: uuid('id').defaultRandom().primaryKey(),
  name: varchar('name', { length: 255 }).notNull(),
  email: varchar('email', { length: 255 }).notNull().unique(),
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
  deletedAt: timestamp('deleted_at'), // soft delete
});

export type User = typeof users.$inferSelect;
export type NewUser = typeof users.$inferInsert;
```

## Money Fields
```typescript
amount: decimal('amount', { precision: 12, scale: 2 }).notNull(),
```

## Common Operations
```typescript
// Insert
const [user] = await db.insert(users).values({ name, email }).returning();

// Select with relations
const group = await db.query.groups.findFirst({
  where: eq(groups.id, id),
  with: { members: { with: { user: true } } },
});

// Update
await db.update(users).set({ name }).where(eq(users.id, id));

// Soft delete
await db.update(items).set({ deletedAt: new Date() }).where(eq(items.id, id));
```

## Commands

> Note: The SQL dialect is configured in `drizzle.config.ts`.

```bash
npx drizzle-kit generate   # Generate migrations
npx drizzle-kit push       # Apply to database
npx drizzle-kit studio     # Open Drizzle Studio
```

## Error Handling

| Error | Cause | Solution |
|-------|-------|----------|
| `connection refused` | Invalid DATABASE_URL | Check Neon connection string |
| `relation does not exist` | Table not created | Run `npx drizzle-kit push` |
| `duplicate key` | Unique constraint violation | Check for existing record first |
| `column does not exist` | Schema out of sync | Re-run migrations |

## References
- Full schema: `IMPLEMENTATION_PROMPT.md` Section 2
- MCP integration: `.mcp/config.json` for Neon server
