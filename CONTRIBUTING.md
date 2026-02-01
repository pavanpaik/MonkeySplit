# Contributing to MonkeySplit

## Development Workflow

### 1. Pick a Workstream

Check `.agent/subagents/` for available workstreams. Each has:
- Purpose and dependencies
- Implementation steps
- Verification checklist

### 2. Create a Branch

```bash
git checkout -b feature/WS-0.1-X-description
```

### 3. Follow the Skill Files

Reference `.agent/skills/` for patterns:
- `nextjs-app-router/SKILL.md` — Page and component patterns
- `drizzle-orm/SKILL.md` — Database operations
- `authjs-magic-link/SKILL.md` — Auth setup
- `balance-calculation/SKILL.md` — Core algorithm
- `vitest-testing/SKILL.md` — Testing patterns

### 4. Run Verification

```bash
npm run type-check    # TypeScript
npm run lint          # ESLint
npm run test          # Unit tests
npm run build         # Production build
```

### 5. Create Handoff Report

After completing a workstream, create `handoffs/WS-X.Y-Z.md`:

```markdown
# WS-0.1-B Handoff

## Status: COMPLETE ✅
## Completed At: 2024-01-15T14:30:00Z

## Files Created
- `lib/db/index.ts`
- `lib/db/schema/*.ts`

## Decisions Made
- Used UUID for primary keys
- Added soft-delete columns

## Verification Results
- [x] Drizzle Studio shows tables
- [x] All tests pass
```

### 6. Open a PR

```bash
git push origin feature/WS-0.1-X-description
gh pr create
```

## Code Standards

From `CLAUDE.md`:

- **TypeScript**: Strict mode, no `any`
- **Naming**: Files `kebab-case.ts`, Components `PascalCase.tsx`
- **Database**: UUIDs, soft-delete via `deleted_at`
- **Validation**: Zod schemas for all inputs

## Using Claude Code

With MCP servers configured:

```bash
# Start with MCP config
claude --mcp-config .mcp/config.json

# Example commands
"Create the expenses table schema"
"Add a server action for creating groups"
"Write tests for balance calculation"
```

## Questions?

Open an issue or check `IMPLEMENTATION_PROMPT.md` for full specs.
