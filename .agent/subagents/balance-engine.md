---
name: "Balance Engine Subagent"
description: "Executes WS-0.1-D: Balance Calculation Engine"
tools: ["Read", "Write", "Bash", "Grep"]
context:
  - "IMPLEMENTATION_PROMPT.md"
  - ".agent/skills/balance-calculation/SKILL.md"
  - ".agent/skills/vitest-testing/SKILL.md"
---

# WS-0.1-D: Balance Calculation Engine

## Purpose
Implement core balance calculation algorithm for group expenses.

## Dependencies
- **Requires**: WS-0.1-B
- **Blocks**: WS-0.1-G

## Instructions

1. Create `lib/balance/types.ts`:
   - `Balance`, `Debt`, `BalanceResult` types
2. Create `lib/balance/calculator.ts`:
   - `calculateGroupBalances(groupId)` function
3. Algorithm (use integer cents to avoid float drift):
   - Fetch non-deleted expenses for group
   - For each participant: `netCents = paidShareCents - owedShareCents`
   - Aggregate by user pair
   - Round final amounts to 2 decimal places
4. Create `lib/balance/__tests__/calculator.test.ts`

## Deliverables
- [ ] `lib/balance/types.ts`
- [ ] `lib/balance/calculator.ts`
- [ ] `lib/balance/__tests__/calculator.test.ts`

## Verification
```bash
npm run test -- balance  # 100% coverage
```

Test cases:
- Simple 2-person expense
- 3+ person equal split
- Zero amount edge case
- Single participant edge case
