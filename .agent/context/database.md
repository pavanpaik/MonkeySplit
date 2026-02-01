# Database Context Reference

## Quick Reference

| Table | Purpose |
|-------|---------|
| `users` | User accounts |
| `groups` | Expense groups |
| `group_members` | Group membership (admin/member) |
| `expenses` | Expense records |
| `expense_participants` | Who paid/owes what |

## Key Conventions
- All PKs are UUIDs
- All tables have `created_at`, `updated_at`
- Soft delete via `deleted_at` column
- Money stored as `DECIMAL(12,2)`

## Full Schema
See `IMPLEMENTATION_PROMPT.md` Section 2: Database Schema
