---
name: "Vitest Testing"
description: "Vitest unit testing and Playwright E2E patterns"
---

# Vitest Testing Skill

## Setup
```bash
npm install -D vitest @testing-library/react @testing-library/jest-dom
npm install -D @playwright/test
```

## Vitest Config
```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    setupFiles: ['./test/setup.ts'],
    coverage: { reporter: ['text', 'html'] },
  },
});
```

## Unit Test Pattern
```typescript
// lib/balance/__tests__/calculator.test.ts
import { describe, it, expect, vi } from 'vitest';
import { calculateEqualSplit } from '../calculator';

describe('calculateEqualSplit', () => {
  it('splits $10 equally among 3 people', () => {
    const result = calculateEqualSplit(10, 3);
    expect(result).toEqual([3.34, 3.33, 3.33]);
    expect(result.reduce((a, b) => a + b, 0)).toBeCloseTo(10);
  });

  it('handles zero amount', () => {
    const result = calculateEqualSplit(0, 3);
    expect(result).toEqual([0, 0, 0]);
  });
});
```

## Mocking Database
```typescript
import { vi } from 'vitest';

vi.mock('@/lib/db', () => ({
  db: {
    query: {
      expenses: {
        findMany: vi.fn().mockResolvedValue([]),
      },
    },
  },
}));
```

## Component Testing
```typescript
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { ExpenseForm } from '../expense-form';

it('submits expense form', async () => {
  const user = userEvent.setup();
  render(<ExpenseForm groupId="123" />);
  
  await user.type(screen.getByLabelText('Amount'), '10.00');
  await user.click(screen.getByRole('button', { name: /submit/i }));
  
  expect(screen.getByText('Expense created')).toBeInTheDocument();
});
```

## E2E Test (Playwright)
```typescript
// e2e/expense-flow.spec.ts
import { test, expect } from '@playwright/test';

test('create expense and view balance', async ({ page }) => {
  await page.goto('/login');
  await page.fill('input[name="email"]', 'test@example.com');
  await page.click('button[type="submit"]');
  
  // ... complete flow
  
  await expect(page.getByText('owes')).toBeVisible();
});
```

## Commands
```bash
npm run test             # Run unit tests
npm run test:coverage    # With coverage
npm run test:e2e         # Playwright E2E
```

## MCP Integration

With Playwright MCP Server (Release 0.6.0), you can:

```
# Natural language E2E test generation
"Write a test for the login flow"
"Test creating an expense and viewing balances"
"Generate accessibility tests for the expense form"
```

