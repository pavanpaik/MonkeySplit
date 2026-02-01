---
name: "Design System"
description: "Visual design guidelines for MonkeySplit UI"
---

# Design System Skill

## Brand Identity

**MonkeySplit** — Friendly, playful, trustworthy expense splitting.

## Color Palette

### Primary Colors
```css
--primary: 142 76% 36%;      /* Green - #22c55e */
--primary-foreground: 0 0% 100%;
```

### Semantic Colors
```css
/* Positive (you're owed) */
--positive: 142 76% 36%;     /* Green */

/* Negative (you owe) */
--negative: 0 84% 60%;       /* Red - #ef4444 */

/* Neutral (settled) */
--neutral: 240 5% 65%;       /* Gray */
```

### Dark Mode
```css
--background: 240 10% 4%;    /* Near black */
--foreground: 0 0% 98%;      /* Near white */
--card: 240 10% 8%;
--muted: 240 5% 15%;
```

## Typography

### Font Stack
```css
--font-sans: 'Inter', system-ui, sans-serif;
--font-mono: 'JetBrains Mono', monospace;
```

### Scale
| Token | Size | Use |
|-------|------|-----|
| `text-xs` | 12px | Labels, badges |
| `text-sm` | 14px | Body text, inputs |
| `text-base` | 16px | Primary content |
| `text-lg` | 18px | Card titles |
| `text-xl` | 20px | Section headers |
| `text-2xl` | 24px | Page titles |
| `text-4xl` | 36px | Hero amounts |

### Money Display
```tsx
// Large amounts: bold, colored by owe/owed
<span className="text-4xl font-bold text-positive">$125.50</span>
<span className="text-4xl font-bold text-negative">-$42.00</span>
```

## Spacing System

Use Tailwind's default 4px scale:
- `space-1` = 4px (tight)
- `space-2` = 8px (default gap)
- `space-4` = 16px (section padding)
- `space-6` = 24px (card padding)
- `space-8` = 32px (page margins)

## Component Patterns

### Cards
```tsx
<Card className="p-6 hover:shadow-lg transition-shadow">
  <CardHeader className="pb-2">
    <CardTitle className="text-lg">{title}</CardTitle>
  </CardHeader>
  <CardContent>{children}</CardContent>
</Card>
```

### Balance Display
```tsx
// Positive balance (you're owed)
<div className="flex items-center gap-2">
  <ArrowDownLeft className="text-positive" />
  <span className="text-positive font-semibold">owes you $25.00</span>
</div>

// Negative balance (you owe)
<div className="flex items-center gap-2">
  <ArrowUpRight className="text-negative" />
  <span className="text-negative font-semibold">you owe $25.00</span>
</div>
```

### Expense Cards
```tsx
<div className="flex items-center gap-4 p-4 border-b">
  <CategoryIcon category={expense.category} />
  <div className="flex-1">
    <p className="font-medium">{expense.description}</p>
    <p className="text-sm text-muted-foreground">
      {expense.payer.name} paid ${expense.amount}
    </p>
  </div>
  <span className={cn("font-semibold", balance > 0 ? "text-positive" : "text-negative")}>
    {balance > 0 ? "+" : ""}{formatMoney(balance)}
  </span>
</div>
```

## Responsive Breakpoints

| Breakpoint | Width | Layout |
|------------|-------|--------|
| Mobile | < 768px | Bottom tabs, stacked cards |
| Tablet | 768-1024px | Sidebar collapsed, 2-col grid |
| Desktop | > 1024px | Full sidebar, 3-col dashboard |

## Animation

```css
/* Subtle transitions */
--transition-fast: 150ms ease;
--transition-base: 200ms ease;
--transition-slow: 300ms ease;

/* Use for */
.hover-lift { transition: transform var(--transition-fast); }
.hover-lift:hover { transform: translateY(-2px); }
```

## Icons

Use **Lucide React** icons consistently:
- `Users` — Groups
- `Receipt` — Expenses
- `Wallet` — Settlements
- `Activity` — Activity feed
- `Settings` — Settings
