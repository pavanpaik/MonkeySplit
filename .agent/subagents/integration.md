---
name: "Integration Subagent"
description: "Executes WS-0.1-G: Integration & Polish"
tools: ["Read", "Write", "Bash", "Grep", "Browser"]
context:
  - "IMPLEMENTATION_PROMPT.md"
  - ".agent/skills/nextjs-app-router/SKILL.md"
  - ".agent/skills/vitest-testing/SKILL.md"
---

# WS-0.1-G: Integration & Polish

## Purpose
Integrate all workstreams and polish the Alpha 0.1.0 release.

## Dependencies
- **Requires**: WS-0.1-C, WS-0.1-D, WS-0.1-E, WS-0.1-F
- **Blocks**: None (release gate)

## Instructions

1. Create balance display:
   - `app/(app)/groups/[groupId]/balances/page.tsx`
   - `components/groups/balance-summary.tsx`
2. Build app layout:
   - `app/(app)/layout.tsx` — App shell with sidebar
   - `components/layout/sidebar.tsx`
3. Add navigation between all pages
4. Polish UI: consistent spacing, loading states
5. Write E2E test: login → create group → add expense → view balance

## Deliverables
- [ ] `app/(app)/groups/[groupId]/balances/page.tsx`
- [ ] `components/groups/balance-summary.tsx`
- [ ] `app/(app)/layout.tsx`
- [ ] `components/layout/sidebar.tsx`
- [ ] E2E integration test

## Verification
- [ ] Full flow: login → create group → add member → add expense → view balance
- [ ] Balance calculation matches expected values
- [ ] Navigation is intuitive
- [ ] No console errors
- [ ] `npm run build` succeeds
- [ ] E2E test passes
