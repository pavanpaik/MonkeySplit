---
name: "Groups Subagent"
description: "Executes WS-0.1-E: Group CRUD & Members"
tools: ["Read", "Write", "Bash", "Grep"]
context:
  - ".agent/context/core.md"
  - ".agent/context/features.md"
  - ".agent/skills/nextjs-app-router/SKILL.md"
---

# WS-0.1-E: Group CRUD & Members

## Purpose
Implement group creation, listing, and member management.

## Dependencies
- **Requires**: WS-0.1-B
- **Blocks**: WS-0.1-G

## Instructions

1. Create server actions in `app/actions/groups.ts`:
   - `createGroup(name)` — Insert group + add creator as admin
   - `addMemberByEmail(groupId, email)` — Add or invite
2. Create pages:
   - `app/(app)/groups/page.tsx` — List groups
   - `app/(app)/groups/new/page.tsx` — Create form
   - `app/(app)/groups/[groupId]/page.tsx` — Dashboard
3. Create components:
   - `components/groups/group-card.tsx`
   - `components/groups/member-list.tsx`

## Deliverables
- [ ] `app/actions/groups.ts`
- [ ] `app/(app)/groups/page.tsx`
- [ ] `app/(app)/groups/new/page.tsx`
- [ ] `app/(app)/groups/[groupId]/page.tsx`
- [ ] `components/groups/group-card.tsx`
- [ ] `components/groups/member-list.tsx`

## Verification
- [ ] Can create group
- [ ] Creator is admin member
- [ ] Can add member by email
- [ ] Groups list shows all groups
