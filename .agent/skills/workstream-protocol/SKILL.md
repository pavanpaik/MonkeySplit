---
name: "workstream-protocol"
description: "Defines the workstream execution and handoff protocol including status tracking, handoff report format, and verification steps. Use this skill when starting a workstream, creating handoff reports, or coordinating between workstreams."
---

# Workstream Protocol Skill

## When to Use
- Starting execution of a workstream
- Creating handoff reports after completion
- Coordinating dependencies between workstreams
- Resolving conflicts or blockers

## Execution Flow
1. Read subagent file for instructions
2. Check dependencies are complete
3. Read referenced skill files
4. Implement deliverables
5. Run verification commands
6. Create handoff report
7. Update workstream status

## Handoff Report Format
```markdown
# WS-X.Y-Z Handoff

## Status: COMPLETE | BLOCKED | FAILED
## Completed At: ISO timestamp
## Duration: Xh Ym

## Files Created/Modified
- `path/to/file.ts` — Description

## Decisions Made
- Decision and rationale

## Known Issues
- Issue description (if any)

## Verification Results
- [x] Command 1 passed
- [x] Command 2 passed

## Next Workstream
WS-X.Y-Z+1 can now proceed.
```

## Verification Commands
```bash
# Run before marking complete
npm run type-check
npm run lint
npm run test
npm run build
```

## Workstream States

| State | Description |
|-------|-------------|
| `PENDING` | Not started, waiting for dependencies |
| `IN_PROGRESS` | Currently being executed |
| `BLOCKED` | Waiting for clarification or dependency |
| `COMPLETE` | All verification passed |
| `FAILED` | Verification failed, needs retry |

## Dependency Check
Before starting a workstream, verify:
```typescript
// Check handoff exists
const handoffs = fs.readdirSync('handoffs/');
const required = ['WS-0.1-A.md']; // from subagent dependencies
const ready = required.every(ws => handoffs.includes(ws));
```

## Conflict Resolution

| Conflict | Resolution |
|----------|------------|
| File already exists | Check if from dependency, merge if needed |
| Schema mismatch | Re-run migrations after schema update |
| Test failure | Fix before marking complete |
| Unclear requirement | Create BLOCKED handoff, request clarification |

## Handoff Directory Structure
```
handoffs/
├── WS-0.1-A.md    # Scaffolding
├── WS-0.1-B.md    # Database
├── WS-0.1-C.md    # Auth
├── WS-0.1-D.md    # Balance Engine
├── WS-0.1-E.md    # Groups
├── WS-0.1-F.md    # Expenses
└── WS-0.1-G.md    # Integration
```

## Error Handling

| Error | Cause | Solution |
|-------|-------|----------|
| Dependency not met | Prior workstream incomplete | Check handoff status |
| Verification failed | Code issues | Fix code, re-run verification |
| Blocked state | Missing info | Document blocker, notify user |

## References
- Workstream definitions: `.agent/subagents/*.md`
- Full spec: `IMPLEMENTATION_PROMPT.md`
