---
name: "balance-calculation"
description: "Provides the balance calculation algorithm for expense splitting including types, computation logic, and test cases. Use this skill when implementing group balances, calculating who owes whom, or writing balance-related tests."
---

# Balance Calculation Skill

## When to Use
- Implementing balance calculation for groups
- Computing who owes whom
- Writing tests for expense splitting
- Understanding the debt simplification algorithm

## Types
```typescript
// lib/balance/types.ts
export type Debt = {
  fromUserId: string;
  toUserId: string;
  amount: number; // positive = fromUser owes toUser
};

export type BalanceResult = {
  debts: Debt[];
  totalOwed: number;
  totalPaid: number;
};
```

## Algorithm
```typescript
// lib/balance/calculator.ts
export async function calculateGroupBalances(groupId: string): Promise<BalanceResult> {
  // 1. Fetch all non-deleted expenses for group
  const expenses = await db.query.expenses.findMany({
    where: and(eq(expenses.groupId, groupId), isNull(expenses.deletedAt)),
    with: { participants: true },
  });

  // 2. Calculate net balance per user (in cents to avoid float drift)
  const netBalances = new Map<string, number>();
  
  for (const expense of expenses) {
    for (const p of expense.participants) {
      const current = netBalances.get(p.userId) || 0;
      const net = Number(p.paidShare) - Number(p.owedShare);
      netBalances.set(p.userId, current + net);
    }
  }

  // 3. Compute totals from net balances
  let totalPaid = 0;
  let totalOwed = 0;
  for (const balance of netBalances.values()) {
    if (balance > 0) totalPaid += balance;
    else totalOwed += Math.abs(balance);
  }
  totalPaid = Math.round(totalPaid * 100) / 100;
  totalOwed = Math.round(totalOwed * 100) / 100;

  // 4. Convert to debts (creditors vs debtors)
  const debts: Debt[] = [];
  const creditors: Array<{ userId: string; amount: number }> = [];
  const debtors: Array<{ userId: string; amount: number }> = [];

  for (const [userId, balance] of netBalances) {
    if (balance > 0) creditors.push({ userId, amount: balance });
    else if (balance < 0) debtors.push({ userId, amount: -balance });
  }

  // 5. Match debtors to creditors (greedy algorithm)
  creditors.sort((a, b) => b.amount - a.amount);
  debtors.sort((a, b) => b.amount - a.amount);

  let i = 0, j = 0;
  while (i < debtors.length && j < creditors.length) {
    const amount = Math.min(debtors[i].amount, creditors[j].amount);
    debts.push({
      fromUserId: debtors[i].userId,
      toUserId: creditors[j].userId,
      amount: Math.round(amount * 100) / 100,
    });
    debtors[i].amount -= amount;
    creditors[j].amount -= amount;
    if (debtors[i].amount < 0.01) i++;
    if (creditors[j].amount < 0.01) j++;
  }

  return { debts, totalOwed, totalPaid };
}
```

## Equal Split Calculation
```typescript
function calculateEqualSplit(amountCents: number, participantIds: string[]): Map<string, number> {
  const count = participantIds.length;
  const baseCents = Math.floor(amountCents / count);
  const remainder = amountCents - (baseCents * count);
  
  // Sort IDs for deterministic remainder distribution
  const sorted = [...participantIds].sort();
  
  const shares = new Map<string, number>();
  sorted.forEach((id, i) => {
    shares.set(id, baseCents + (i < remainder ? 1 : 0));
  });
  
  return shares;
}
// $10.00 / 3 = {A: 334, B: 333, C: 333} cents
```

## Test Cases
```typescript
describe('calculateGroupBalances', () => {
  it('simple 2-person: A pays $10, both owe → A owed $5', () => {});
  it('3-person equal: A pays $10, all owe → B owes A $3.33, C owes A $3.33', () => {});
  it('zero amount: returns empty debts', () => {});
  it('single participant: no debts', () => {});
});
```

## Error Handling

| Error | Cause | Solution |
|-------|-------|----------|
| Float precision | Using `number` for money | Use integer cents, convert at display |
| Non-deterministic split | Remainder varies by order | Sort participant IDs before distribution |
| Negative balance shown | Sign confusion | Positive = owed TO you, negative = you OWE |

## References
- Algorithm details: `IMPLEMENTATION_PROMPT.md` Section 4
- Edge cases: `IMPLEMENTATION_PROMPT.md` Section 5
