---
name: "vitest-testing"
description: "Provides Vitest unit testing and Playwright E2E testing patterns including setup, configuration, mocking, and test examples. Use this skill when writing unit tests, component tests, E2E tests, or setting up test infrastructure."
---

# Vitest Testing Skill

## When to Use
- Writing unit tests for business logic
- Testing React components
- Creating E2E tests with Playwright
- Mocking database calls

## Setup
```bash
npm install -D vitest @testing-library/react @testing-library/jest-dom @testing-library/user-event
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
    const result = calculateEqualSplit(1000, ['a', 'b', 'c']); // cents
    expect([...result.values()]).toEqual([334, 333, 333]);
    expect([...result.values()].reduce((a, b) => a + b, 0)).toBe(1000);
  });

  it('handles zero amount', () => {
    const result = calculateEqualSplit(0, ['a', 'b', 'c']);
    expect([...result.values()]).toEqual([0, 0, 0]);
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

## Error Handling

| Error | Cause | Solution |
|-------|-------|----------|
| `ReferenceError: document` | Wrong test environment | Set `environment: 'jsdom'` |
| Mock not working | Import order | Mock before importing module |
| Timeout in E2E | Slow page load | Increase timeout or use waitFor |
| Act warning | State update after unmount | Wrap in `act()` or await properly |

## References
- Testing guidelines: `IMPLEMENTATION_PROMPT.md` Section 12
- Coverage requirements: 80% for business logic
