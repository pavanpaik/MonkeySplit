# MonkeySplit Release Roadmap: 0 to 60

This roadmap guides an engineer from zero code to a production-ready V1.0.0 (Sprint 5), broken down by release versions.

**Total Releases:** 5 Major Milestones (v0.1.0 through v1.0.0)

---

## 🏁 Release v0.1.0: Foundation (Sprint 1)
**Goal:** Technical setup, database foundation, authentication, and user profile basics.

### Steps for Engineer
1. **WS-0.1-A: Scaffolding**
   - Initialize Next.js, Tailwind, shadcn/ui.
   - Configure Linting & TypeScript.
2. **WS-0.1-B: Database**
   - Set up Neon & Drizzle.
   - Define minimal schema (Users table).
3. **WS-0.1-C: Auth**
   - Implement Auth.js (Email Magic Links).
   - *Add*: Google/GitHub providers (Sprint 1 scope).
4. **Seed Data**
   - Populate Currencies and Categories tables.
5. **Profile & Settings**
   - Create User Profile page (name, avatar, currency preference).
6. **Observability**
   - Integrate Sentry and Logging (Pino).

**🚀 Deliverable:** A deployed app where users can login and update their profile.

---

## 📦 Release v0.2.0: Core MVP (Sprint 2)
**Goal:** The core "Splitwise" functionality. Groups, expenses, and balances.

### Steps for Engineer
1. **Friends Module**
   - Add/Invite friends, list friends.
2. **WS-0.1-E: Groups**
   - Create/Edit Groups, add members.
3. **WS-0.1-F: Expenses**
   - Expense creation form.
   - Support "Equal Split" (WS-0.1 scope).
   - *Add*: Exact amounts, Percentages, Shares splitting logic.
4. **WS-0.1-D: Balance Engine**
   - Implement the `totalOwed` / `totalPaid` logic.
   - Implement Debt Simplification algorithm.
5. **Settle Up**
   - "Record a Payment" flow to clear debts.

**🚀 Deliverable:** Functional expense splitter. Users can track debts and settle up.

---

## 💬 Release v0.3.0: Social & Real-time (Sprint 3)
**Goal:** Making it collaborative and alive.

### Steps for Engineer
1. **Activity Feed**
   - Global and Group-level activity logs ("Alice added 'Lunch'").
2. **Comments**
   - Threaded comments on specific expenses.
3. **Notifications**
   - In-app notification center.
   - Email notifications via Resend (e.g., "You were added to a group").
4. **Real-time Updates**
   - Integrate Pusher/Ably.
   - Auto-refresh balances and feeds without page reload.

**🚀 Deliverable:** A collaborative social finance app.

---

## ✨ Release v0.4.0: Power Features (Sprint 4)
**Goal:** Advanced utility for power users.

### Steps for Engineer
1. **Recurring Expenses**
   - Database schema for schedules.
   - Vercel Cron jobs to auto-create expenses.
2. **Multi-Currency**
   - FX rate integration (OpenExchangeRates/similar).
   - Convert foreign expenses to group/user default currency.
3. **Receipt Scanning**
   - Integrate OCR API (e.g., Veryfi or GPT-4o Vision).
   - Auto-fill expense form from image.
4. **Search & Filters**
   - Global search for expenses/people.
   - Filter by date, category, group.

**🚀 Deliverable:** Feature-complete competitor to paid alternatives.

---

## 🏆 Release v1.0.0: Production Polish (Sprint 5)
**Goal:** Compliance, stability, and public launch readiness.

### Steps for Engineer
1. **Data Visualization**
   - Charts/Graphs for "Monthly Spending", "Category breakdown".
2. **Export & Portability**
   - CSV/PDF Export functionality.
   - "Invite via Link/QR" flows.
3. **Legal & Compliance**
   - GDPR "Export my data" & "Delete my account" flows.
   - TOS & Privacy Policy pages.
4. **Polish**
   - Mobile responsive audit + PWA manifest.
   - Performance tuning (Lighthouse score 90+).
5. **CI/CD**
   - Finalize GitHub Actions pipelines for testing/deploy.

**🚀 Deliverable:** Public Launch V1.0.0.

---

## Execution Strategy

**How to run this?**
Load the `feature-builder` agent for each item:
```
"Use feature-builder to implement [Step Name]"
```
Or use the `scaffolder` / `database-architect` for foundational steps.
