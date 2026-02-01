---
name: "design-system"
description: "Provides visual design guidelines for MonkeySplit UI including color palette, typography, spacing, and component patterns. Use this skill when styling components, implementing dark mode, or ensuring visual consistency."
---

# Design System Skill

## When to Use
- Creating new UI components
- Styling balance displays (positive/negative)
- Implementing responsive layouts
- Adding dark mode support

## Brand Identity
**MonkeySplit** — Friendly, playful, trustworthy expense splitting.

## Color Palette

### CSS Variables
```css
:root {
  --primary: 142 76% 36%;      /* #22c55e Green */
  --primary-foreground: 0 0% 100%;
  
  /* Semantic */
  --positive: 142 76% 36%;     /* You're owed */
  --negative: 0 84% 60%;       /* You owe - #ef4444 */
  --neutral: 240 5% 65%;       /* Settled */
  
  /* Dark Mode */
  --background: 240 10% 4%;
  --foreground: 0 0% 98%;
  --card: 240 10% 8%;
  --muted: 240 5% 15%;
}
```

## Typography

| Scale | Size | Weight | Usage |
|-------|------|--------|-------|
| `text-xs` | 12px | 400 | Labels, badges |
| `text-sm` | 14px | 400 | Body text, inputs |
| `text-base` | 16px | 400 | Primary content |
| `text-lg` | 18px | 500 | Card titles |
| `text-xl` | 20px | 600 | Section headers |
| `text-2xl` | 24px | 700 | Page titles |
| `text-4xl` | 36px | 700 | Hero amounts |

**Font Stack**: `'Inter', system-ui, sans-serif`

## Money Display
```tsx
// Positive (you're owed)
<span className="text-4xl font-bold text-positive">+$125.50</span>

// Negative (you owe)
<span className="text-4xl font-bold text-negative">-$42.00</span>

// Settled
<span className="text-muted-foreground">$0.00</span>
```

## Balance Display Pattern
```tsx
// Who owes whom
<div className="flex items-center gap-2">
  <ArrowDownLeft className="h-4 w-4 text-positive" />
  <span className="text-positive font-semibold">owes you $25.00</span>
</div>

<div className="flex items-center gap-2">
  <ArrowUpRight className="h-4 w-4 text-negative" />
  <span className="text-negative font-semibold">you owe $25.00</span>
</div>
```

## Spacing Scale
| Token | Value | Usage |
|-------|-------|-------|
| `1` | 4px | Tight inline |
| `2` | 8px | Default gap |
| `4` | 16px | Section padding |
| `6` | 24px | Card padding |
| `8` | 32px | Page margins |

## Responsive Breakpoints
| Breakpoint | Width | Layout |
|------------|-------|--------|
| Mobile | < 768px | Bottom tabs, stacked |
| Tablet | 768-1024px | Collapsed sidebar |
| Desktop | > 1024px | Full sidebar |

## Icons
Use **Lucide React** consistently:
- `Users` — Groups
- `Receipt` — Expenses
- `Wallet` — Settlements
- `Activity` — Activity feed
- `Settings` — Settings
- `Plus` — Add actions

## Error Handling

| Issue | Cause | Solution |
|-------|-------|----------|
| Colors not showing | Missing CSS vars | Check globals.css |
| Dark mode flicker | No initial theme | Use `next-themes` with `defaultTheme` |
| Inconsistent spacing | Ad-hoc values | Use spacing tokens only |

## References
- Full design spec: `IMPLEMENTATION_PROMPT.md` Section 7
- Component list: `IMPLEMENTATION_PROMPT.md` Section 8
