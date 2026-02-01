---
name: "Balance Calculation"
description: "Balance calculation algorithm for expense splitting"
---

# Balance Calculation Skill

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

  // 5. Match debtors to creditors
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
function calculateEqualSplit(amount: number, participantCount: number): number[] {
  const baseShare = Math.floor(amount * 100 / participantCount) / 100;
  const remainder = Math.round((amount - baseShare * participantCount) * 100);
  
  return Array.from({ length: participantCount }, (_, i) => 
    i < remainder ? baseShare + 0.01 : baseShare
  );
}
// $10 / 3 = [3.34, 3.33, 3.33]
```

## Test Cases
- Simple 2-person: A pays $10, both owe → A owed $5
- 3-person equal: A pays $10, all owe → B owes A $3.33, C owes A $3.33
- Zero amount: Handle gracefully
- Single participant: No debts
