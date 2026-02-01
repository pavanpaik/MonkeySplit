---
name: "Expenses Subagent"
description: "Executes WS-0.1-F: Basic Expense Form (Equal Split)"
tools: ["Read", "Write", "Bash", "Grep"]
context:
  - "IMPLEMENTATION_PROMPT.md"
  - ".agent/skills/nextjs-app-router/SKILL.md"
---

# WS-0.1-F: Basic Expense Form (Equal Split)

## Purpose
Implement expense creation with equal split functionality.

## Dependencies
- **Requires**: WS-0.1-B
- **Blocks**: WS-0.1-G

## Instructions

1. Create server action `app/actions/expenses.ts`:
   - `createExpense(groupId, description, amount, payerId, participantIds)`
   - Calculate equal split: `owed_share = amount / participantCount`
   - Handle remainder cents deterministically
2. Create page: `app/(app)/groups/[groupId]/expenses/new/page.tsx`
3. Create components:
   - `components/expenses/expense-form.tsx`
   - `components/expenses/participant-selector.tsx`
   - `components/expenses/expense-card.tsx`

## Deliverables
- [ ] `app/actions/expenses.ts`
- [ ] `app/(app)/groups/[groupId]/expenses/new/page.tsx`
- [ ] `components/expenses/expense-form.tsx`
- [ ] `components/expenses/participant-selector.tsx`
- [ ] `components/expenses/expense-card.tsx`

## Verification
- [ ] Can create expense with description and amount
- [ ] Equal split: $10 / 3 = $3.34, $3.33, $3.33
- [ ] Expense appears in group list
- [ ] Payer and participants recorded correctly
