# MonkeySplit Skills

Skills are modular capabilities that extend Claude's ability to work on this codebase. Each skill contains patterns, conventions, and implementation guidance.

## Available Skills

| Skill | Path | Purpose |
|-------|------|---------|
| **Next.js App Router** | [`.agent/skills/nextjs-app-router/SKILL.md`](.agent/skills/nextjs-app-router/SKILL.md) | App Router patterns, server components, actions |
| **Drizzle ORM** | [`.agent/skills/drizzle-orm/SKILL.md`](.agent/skills/drizzle-orm/SKILL.md) | Database schema, queries, migrations |
| **Auth.js Magic Link** | [`.agent/skills/authjs-magic-link/SKILL.md`](.agent/skills/authjs-magic-link/SKILL.md) | Email authentication setup |
| **Balance Calculation** | [`.agent/skills/balance-calculation/SKILL.md`](.agent/skills/balance-calculation/SKILL.md) | Core splitting algorithm |
| **Vitest Testing** | [`.agent/skills/vitest-testing/SKILL.md`](.agent/skills/vitest-testing/SKILL.md) | Unit and E2E testing patterns |
| **Workstream Protocol** | [`.agent/skills/workstream-protocol/SKILL.md`](.agent/skills/workstream-protocol/SKILL.md) | Execution and handoff format |
| **Design System** | [`.agent/skills/design-system/SKILL.md`](.agent/skills/design-system/SKILL.md) | Visual design guidelines |

## How Skills Work

Skills are referenced by subagents (`.agent/subagents/*.md`) when executing workstreams. They provide:

1. **Code patterns** — Copy-paste examples
2. **Conventions** — Naming, structure, style
3. **Commands** — CLI tools and verification steps
4. **MCP integration** — Natural language shortcuts

## Creating New Skills

```
.agent/skills/my-skill/
└── SKILL.md
```

Format:
```markdown
---
name: "Skill Name"
description: "What this skill provides"
---

# Skill Name

[Patterns, examples, and guidance]
```
