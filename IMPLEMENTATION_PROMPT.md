# MonkeySplit — Full Implementation Prompt

> An open-source Splitwise clone built on the Vercel stack.

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Framework | **Next.js 14+ (App Router)** |
| Language | **TypeScript** |
| Styling | **Tailwind CSS + shadcn/ui** |
| Database | **PostgreSQL** (via Neon or Supabase) |
| ORM | **Drizzle ORM** |
| Auth | **NextAuth.js v5 (Auth.js)** with Google, GitHub, Email magic-link providers |
| Real-time | **Pusher** or **Ably** (for live balance/activity updates) |
| File Storage | **Vercel Blob** or **Uploadthing** (receipt images) |
| Payments | **Stripe Connect** (optional peer-to-peer settlement) |
| Hosting | **Vercel** |
| Email | **Resend** (transactional emails, notifications) |
| Cron Jobs | **Vercel Cron** (recurring expenses) |
| State Management | **React Server Components + TanStack Query** for client cache |

---

## Database Schema

### Core Tables

```
users
├── id                  UUID PK
├── name                VARCHAR(255)
├── email               VARCHAR(255) UNIQUE
├── email_verified      TIMESTAMP
├── image               TEXT (avatar URL)
├── password_hash       TEXT (nullable, for credential auth)
├── default_currency    VARCHAR(3) DEFAULT 'USD'
├── locale              VARCHAR(10) DEFAULT 'en'
├── date_format         VARCHAR(20) DEFAULT 'MM/DD/YYYY'
├── created_at          TIMESTAMP
└── updated_at          TIMESTAMP

friendships
├── id                  UUID PK
├── user_id             UUID FK → users
├── friend_id           UUID FK → users
├── status              ENUM('pending', 'accepted', 'blocked')
├── created_at          TIMESTAMP
└── updated_at          TIMESTAMP
└── UNIQUE(user_id, friend_id)

groups
├── id                  UUID PK
├── name                VARCHAR(255)
├── group_type          ENUM('apartment', 'house', 'trip', 'couple', 'other')
├── cover_image         TEXT
├── simplify_debts      BOOLEAN DEFAULT false
├── whiteboard          TEXT (shared notepad)
├── invite_code         VARCHAR(32) UNIQUE
├── default_currency    VARCHAR(3) DEFAULT 'USD'
├── created_by          UUID FK → users
├── created_at          TIMESTAMP
├── updated_at          TIMESTAMP
└── deleted_at          TIMESTAMP (soft delete)

group_members
├── id                  UUID PK
├── group_id            UUID FK → groups
├── user_id             UUID FK → users
├── role                ENUM('admin', 'member')
├── joined_at           TIMESTAMP
└── UNIQUE(group_id, user_id)

expenses
├── id                  UUID PK
├── group_id            UUID FK → groups (nullable for non-group expenses)
├── friendship_id       UUID FK → friendships (nullable)
├── description         VARCHAR(500)
├── amount              DECIMAL(12,2)
├── currency_code       VARCHAR(3)
├── date                DATE
├── category_id         INT FK → categories
├── notes               TEXT
├── receipt_url         TEXT
├── is_payment          BOOLEAN DEFAULT false (true = settlement)
├── repeat_interval     ENUM('none','weekly','biweekly','monthly','yearly') DEFAULT 'none'
├── next_repeat_date    DATE
├── expense_bundle_id   UUID (groups related expenses)
├── created_by          UUID FK → users
├── updated_by          UUID FK → users
├── deleted_by          UUID FK → users
├── created_at          TIMESTAMP
├── updated_at          TIMESTAMP
└── deleted_at          TIMESTAMP (soft delete)

expense_participants
├── id                  UUID PK
├── expense_id          UUID FK → expenses
├── user_id             UUID FK → users
├── paid_share          DECIMAL(12,2) DEFAULT 0  (how much this user paid)
├── owed_share          DECIMAL(12,2) DEFAULT 0  (how much this user owes)
└── UNIQUE(expense_id, user_id)

expense_items (line-item itemization)
├── id                  UUID PK
├── expense_id          UUID FK → expenses
├── description         VARCHAR(255)
├── amount              DECIMAL(12,2)
└── created_at          TIMESTAMP

expense_item_assignments
├── id                  UUID PK
├── expense_item_id     UUID FK → expense_items
├── user_id             UUID FK → users
├── share               DECIMAL(12,2)
└── UNIQUE(expense_item_id, user_id)

categories
├── id                  SERIAL PK
├── name                VARCHAR(100)
├── icon                VARCHAR(50)
├── parent_id           INT FK → categories (nullable; null = top-level)
└── display_order       INT

comments
├── id                  UUID PK
├── expense_id          UUID FK → expenses
├── user_id             UUID FK → users
├── content             TEXT
├── is_system           BOOLEAN DEFAULT false (system-generated edit comments)
├── created_at          TIMESTAMP
└── deleted_at          TIMESTAMP

activity_log
├── id                  UUID PK
├── actor_id            UUID FK → users
├── action              ENUM('expense_created','expense_updated','expense_deleted',
│                            'payment_created','group_created','member_added',
│                            'member_removed','comment_added','settings_changed')
├── entity_type         VARCHAR(50)
├── entity_id           UUID
├── group_id            UUID FK → groups (nullable)
├── metadata            JSONB (stores diff/details)
├── created_at          TIMESTAMP

notifications
├── id                  UUID PK
├── user_id             UUID FK → users
├── activity_id         UUID FK → activity_log
├── read                BOOLEAN DEFAULT false
├── emailed             BOOLEAN DEFAULT false
├── created_at          TIMESTAMP

currencies
├── code                VARCHAR(3) PK
├── name                VARCHAR(100)
├── symbol              VARCHAR(10)
└── decimal_places      INT DEFAULT 2

default_splits
├── id                  UUID PK
├── group_id            UUID FK → groups
├── user_id             UUID FK → users
├── percentage          DECIMAL(5,2)
└── UNIQUE(group_id, user_id)
```

---

## Feature Specifications

### Phase 1: Core Foundation

#### 1.1 Authentication & User Management
- Sign up / sign in with email magic link, Google OAuth, GitHub OAuth
- User profile page: name, avatar upload, default currency, locale, date format
- Password-optional (magic link primary, optional password set)
- App lock toggle (re-auth before sensitive actions)

#### 1.2 Friends
- Add friends by email (sends invite if not registered)
- Accept / reject friend requests
- Friends list page showing each friend with per-currency balances
- Unfriend (only when settled up)
- Search friends by name/email

#### 1.3 Groups
- Create group with name, type (apartment/house/trip/couple/other), cover image
- Auto-generate shareable invite link (`/invite/:code`)
- Add/remove members (remove only when settled with group)
- Group settings: rename, change type, cover image, toggle simplify debts, whiteboard
- Soft-delete groups (restorable)
- Group dashboard: member list, balances, recent expenses

#### 1.4 Expenses — CRUD
- **Add expense** form with:
  - Description, amount, currency selector (100+ currencies seeded)
  - Date picker (defaults to today)
  - Category picker (two-level: parent → subcategory)
  - "Paid by" selector (single or multiple payers with amounts)
  - Split method selector:
    - **Equal** — divide evenly among selected participants
    - **Exact amounts** — manually enter each person's share
    - **Percentages** — enter % per person (must total 100%)
    - **Shares** — enter share count per person (e.g., 2, 1, 1)
    - **Adjustment** — start equal, apply +/- per person
  - Participant selector (checkbox list of group members or friends)
  - Notes field
  - Receipt image upload (Vercel Blob)
  - Recurring toggle: weekly / biweekly / monthly / yearly
- **Edit expense** — same form, pre-filled; generates system comment with diff
- **Delete expense** — soft delete with confirmation; restorable
- **Expense detail page** — full info, participant breakdown, comments, edit history, receipt viewer

#### 1.5 Settlements (Settle Up)
- Record a manual payment between two users
- Amount defaults to the full balance owed (editable for partial)
- Settlement stored as an expense with `is_payment = true`
- Shows in activity feed and expense list with distinct styling
- Group context or friend context

#### 1.6 Balance Calculation Engine
This is the **critical business logic**. Implement as a server-side utility.

```
calculateBalances(groupId | friendshipId):
  1. Fetch all non-deleted expenses (including settlements)
  2. For each expense, for each participant:
     net = paid_share - owed_share
  3. Aggregate per user-pair per currency → raw debts
  4. If simplify_debts enabled:
     a. Compute net balance per user per currency
     b. Separate into creditors (positive) and debtors (negative)
     c. Greedily match largest debtor to largest creditor
     d. Repeat until all settled
     e. Return simplified debt list
  5. Return: { balances: [...], debts: [...], simplified_debts: [...] }
```

**Simplify Debts Algorithm** (greedy approach):
```
function simplifyDebts(netBalances: Map<userId, amount>):
  creditors = sorted list of users with positive balance (descending)
  debtors = sorted list of users with negative balance (ascending by abs)
  result = []
  while creditors and debtors not empty:
    creditor = creditors[0]
    debtor = debtors[0]
    transfer = min(creditor.amount, abs(debtor.amount))
    result.push({ from: debtor.id, to: creditor.id, amount: transfer })
    creditor.amount -= transfer
    debtor.amount += transfer
    if creditor.amount == 0: remove from creditors
    if debtor.amount == 0: remove from debtors
  return result
```

---

### Phase 2: Enhanced Features

#### 2.1 Activity Feed
- Dashboard shows chronological activity feed
- Each entry: actor, action description, timestamp, link to entity
- Filter by group
- "Since you last visited" separator
- Real-time updates via Pusher/Ably websocket channel

#### 2.2 Comments
- Comment thread on each expense
- System comments auto-generated on edit (show diff: "Alice changed the amount from $50 to $60")
- Real-time comment updates
- Email notification for new comments (configurable)

#### 2.3 Notifications System
- In-app notification bell with unread count
- Notification types: expense added/edited/deleted, payment recorded, added to group, comment on your expense, friend request
- Mark as read (individual + mark all)
- Email notifications via Resend (configurable per type in user settings)
- Push notification support (web push via service worker, optional)

#### 2.4 Recurring Expenses
- Vercel Cron job runs daily at midnight UTC
- Queries expenses where `repeat_interval != 'none'` AND `next_repeat_date <= today`
- Clones the expense with new date
- Updates `next_repeat_date` based on interval
- Sends email reminder if configured (N days before)

#### 2.5 Multi-Currency Support
- Seed `currencies` table with 100+ currencies (code, name, symbol)
- Each expense stores its own `currency_code`
- Balances computed and displayed per currency
- Currency conversion feature:
  - Fetch live rates from Open Exchange Rates API (or exchangerate.host free tier)
  - Convert all balances in a group/friendship to user's default currency
  - Store as a one-time snapshot (don't auto-update)

#### 2.6 Expense Categories
- Seed database with Splitwise-compatible category tree:
  - Entertainment → Games, Movies, Music, Sports, Other
  - Food & Drink → Groceries, Dining out, Liquor, Other
  - Home → Rent, Mortgage, Household supplies, Furniture, Maintenance, Pets, Services, Other
  - Life → Childcare, Clothing, Education, Gifts, Medical, Insurance, Taxes, Other
  - Transportation → Parking, Gas/fuel, Car, Bus/train, Bicycle, Hotel, Taxi, Other
  - Utilities → Electric, Water, Gas, Internet, Trash, Phone, TV, Other
  - General → Uncategorized
- Category icon for each (use Lucide icons)
- Category picker in expense form (grouped dropdown)

---

### Phase 3: Premium / Advanced Features

#### 3.1 Receipt Scanning (OCR)
- Upload receipt photo → send to OCR service (Tesseract.js client-side, or Google Cloud Vision API, or OpenAI Vision)
- Extract: total amount, line items with prices
- Auto-fill expense amount
- Generate `expense_items` for itemization

#### 3.2 Itemization
- After receipt scan or manual entry, show line-item list
- Each item: description, amount
- Assign each item to one or more participants
- Auto-calculate `owed_share` per participant from assigned items
- UI: drag-and-drop items onto participant avatars, or checkbox grid

#### 3.3 Charts & Reports
- **Spending by category** — pie/donut chart (use Recharts or Chart.js)
- **Spending over time** — line/bar chart (monthly aggregation)
- **Balance trend** — how your net balance changes over time
- **Group breakdown** — spending per group
- Filter by date range, group, category
- CSV export of all expenses

#### 3.4 Expense Search
- Full-text search across expense descriptions and notes
- Filter by: date range, category, group, amount range, currency
- Powered by PostgreSQL `tsvector` full-text search or pg_trgm

#### 3.5 Default Splits
- Per-group setting: define a default split ratio for all members
- When adding expense to that group, auto-apply the ratio
- Stored in `default_splits` table

#### 3.6 Invite System
- Share group invite link (auto-generated, resettable)
- Email invite for non-registered users (creates placeholder account)
- When invitee registers, merge placeholder into real account
- QR code generation for invite link (use `qrcode` npm package)

---

## Page & Route Structure (App Router)

```
app/
├── (auth)/
│   ├── login/page.tsx              — Sign in (magic link + OAuth)
│   ├── register/page.tsx           — Sign up
│   └── verify/page.tsx             — Magic link verification
├── (app)/                          — Authenticated layout with sidebar
│   ├── layout.tsx                  — Sidebar nav + top bar + notification bell
│   ├── dashboard/page.tsx          — Activity feed + overall balances summary
│   ├── groups/
│   │   ├── page.tsx                — List all groups
│   │   ├── new/page.tsx            — Create group form
│   │   └── [groupId]/
│   │       ├── page.tsx            — Group dashboard (expenses, balances, members)
│   │       ├── expenses/
│   │       │   ├── new/page.tsx    — Add expense to group
│   │       │   └── [expenseId]/
│   │       │       ├── page.tsx    — Expense detail + comments
│   │       │       └── edit/page.tsx
│   │       ├── settle/page.tsx     — Settle up within group
│   │       ├── balances/page.tsx   — Detailed balances view
│   │       ├── whiteboard/page.tsx — Shared group notepad
│   │       └── settings/page.tsx   — Group settings
│   ├── friends/
│   │   ├── page.tsx                — Friends list with balances
│   │   ├── add/page.tsx            — Add friend form
│   │   └── [friendId]/
│   │       ├── page.tsx            — Friend detail (shared expenses, balance)
│   │       └── settle/page.tsx     — Settle up with friend
│   ├── expenses/
│   │   ├── page.tsx                — All expenses (searchable, filterable)
│   │   └── new/page.tsx            — Add non-group expense
│   ├── activity/page.tsx           — Full activity feed
│   ├── notifications/page.tsx      — All notifications
│   ├── reports/page.tsx            — Charts and reports
│   ├── settings/
│   │   ├── page.tsx                — Account settings
│   │   ├── profile/page.tsx        — Edit profile
│   │   └── notifications/page.tsx  — Notification preferences
│   └── invite/[code]/page.tsx      — Accept group invite
├── api/
│   ├── auth/[...nextauth]/route.ts — Auth.js handler
│   ├── expenses/route.ts           — CRUD expenses
│   ├── expenses/[id]/route.ts
│   ├── expenses/[id]/comments/route.ts
│   ├── groups/route.ts
│   ├── groups/[id]/route.ts
│   ├── groups/[id]/balances/route.ts
│   ├── groups/[id]/simplify/route.ts
│   ├── friends/route.ts
│   ├── friends/[id]/route.ts
│   ├── settlements/route.ts
│   ├── notifications/route.ts
│   ├── categories/route.ts
│   ├── currencies/route.ts
│   ├── upload/route.ts             — Receipt upload
│   ├── ocr/route.ts                — Receipt OCR processing
│   └── cron/
│       └── recurring-expenses/route.ts — Vercel Cron handler
└── globals.css
```

---

## Key UI Components

```
components/
├── layout/
│   ├── Sidebar.tsx                 — Nav: Dashboard, Groups, Friends, Activity, Reports
│   ├── TopBar.tsx                  — Search, notification bell, user menu
│   └── MobileNav.tsx               — Bottom tab bar for mobile
├── expenses/
│   ├── ExpenseForm.tsx             — Full add/edit expense form
│   ├── SplitMethodSelector.tsx     — Tabs: Equal, Exact, Percentage, Shares, Adjustment
│   ├── ParticipantSelector.tsx     — Checkbox list with amounts
│   ├── PayerSelector.tsx           — Who paid (single or multi-payer)
│   ├── CategoryPicker.tsx          — Two-level grouped dropdown
│   ├── CurrencyPicker.tsx          — Searchable currency dropdown
│   ├── ExpenseCard.tsx             — Compact expense summary for lists
│   ├── ExpenseDetail.tsx           — Full expense view
│   ├── ReceiptUploader.tsx         — Drag-drop / camera upload
│   ├── ItemizationEditor.tsx       — Line item assignment UI
│   └── RecurringToggle.tsx         — Repeat interval selector
├── groups/
│   ├── GroupCard.tsx               — Group summary card for list
│   ├── GroupHeader.tsx             — Group name, image, member count
│   ├── MemberList.tsx              — Members with balances
│   ├── BalanceSummary.tsx          — Who owes whom (with simplify toggle)
│   ├── DebtGraph.tsx               — Visual debt arrows between members
│   └── Whiteboard.tsx              — Shared notepad editor
├── friends/
│   ├── FriendCard.tsx              — Friend with balance
│   ├── AddFriendForm.tsx           — Email/search to add friend
│   └── FriendBalanceDetail.tsx     — Per-currency breakdown
├── settlements/
│   ├── SettleUpForm.tsx            — Amount input, confirm payment
│   └── SettlementCard.tsx          — Settlement display in list
├── activity/
│   ├── ActivityFeed.tsx            — Chronological event list
│   └── ActivityItem.tsx            — Single activity entry
├── notifications/
│   ├── NotificationBell.tsx        — Icon with unread count badge
│   └── NotificationList.tsx        — Dropdown or page list
├── reports/
│   ├── SpendingByCategory.tsx      — Pie/donut chart
│   ├── SpendingOverTime.tsx        — Line/bar chart
│   └── ExportCSV.tsx               — Download button
├── comments/
│   ├── CommentThread.tsx           — List of comments
│   └── CommentInput.tsx            — New comment form
└── ui/                             — shadcn/ui components (Button, Dialog, Input, etc.)
```

---

## Server Actions & API Design

Use **Next.js Server Actions** for mutations and **Route Handlers** for the public-facing REST API.

### Server Actions (app/actions/)

```typescript
// expenses.ts
createExpense(data: CreateExpenseInput): Promise<Expense>
updateExpense(id: string, data: UpdateExpenseInput): Promise<Expense>
deleteExpense(id: string): Promise<void>
restoreExpense(id: string): Promise<Expense>

// groups.ts
createGroup(data: CreateGroupInput): Promise<Group>
updateGroup(id: string, data: UpdateGroupInput): Promise<Group>
deleteGroup(id: string): Promise<void>
addMemberToGroup(groupId: string, userId: string): Promise<void>
removeMemberFromGroup(groupId: string, userId: string): Promise<void>

// friends.ts
sendFriendRequest(email: string): Promise<Friendship>
acceptFriendRequest(id: string): Promise<void>
rejectFriendRequest(id: string): Promise<void>

// settlements.ts
recordSettlement(data: SettlementInput): Promise<Expense>

// comments.ts
addComment(expenseId: string, content: string): Promise<Comment>
deleteComment(id: string): Promise<void>

// balances.ts
getGroupBalances(groupId: string): Promise<BalanceResult>
getFriendBalance(friendId: string): Promise<BalanceResult>
getOverallBalances(): Promise<OverallBalances>
```

---

## Implementation Order

### Sprint 1: Foundation
1. Initialize Next.js 14 project with TypeScript, Tailwind, shadcn/ui
2. Set up Drizzle ORM + PostgreSQL schema + migrations
3. Implement Auth.js (Google + GitHub + Email magic link)
4. Seed currencies and categories tables
5. User profile page + settings

### Sprint 2: Core Expense Flow
1. Friends system (add, accept, list, balances)
2. Groups CRUD + member management
3. Expense form (all split methods)
4. Balance calculation engine
5. Simplify debts algorithm
6. Settle up flow

### Sprint 3: Social & Communication
1. Activity feed + activity logging
2. Comment system on expenses
3. Notification system (in-app + email via Resend)
4. Real-time updates (Pusher/Ably)

### Sprint 4: Enhanced Features
1. Recurring expenses + Vercel Cron
2. Multi-currency support + conversion
3. Receipt upload + OCR
4. Itemization
5. Expense search

### Sprint 5: Reports & Polish
1. Charts and reports (Recharts)
2. CSV export
3. Invite links + QR codes
4. Group whiteboard
5. Default splits
6. Mobile-responsive polish
7. PWA support (optional)

---

## Environment Variables

```env
# Database
DATABASE_URL=postgresql://...

# Auth
NEXTAUTH_URL=http://localhost:3000
NEXTAUTH_SECRET=...
GOOGLE_CLIENT_ID=...
GOOGLE_CLIENT_SECRET=...
GITHUB_CLIENT_ID=...
GITHUB_CLIENT_SECRET=...

# Email
RESEND_API_KEY=...

# File Storage
BLOB_READ_WRITE_TOKEN=...  (Vercel Blob)

# Real-time
PUSHER_APP_ID=...
PUSHER_KEY=...
PUSHER_SECRET=...
PUSHER_CLUSTER=...
NEXT_PUBLIC_PUSHER_KEY=...
NEXT_PUBLIC_PUSHER_CLUSTER=...

# Currency Rates
EXCHANGE_RATE_API_KEY=...

# OCR (optional)
GOOGLE_CLOUD_VISION_KEY=...

# Cron Secret
CRON_SECRET=...

# Rate Limiting (Upstash Redis)
UPSTASH_REDIS_REST_URL=...
UPSTASH_REDIS_REST_TOKEN=...

# Error Monitoring (Sentry)
SENTRY_DSN=...
SENTRY_AUTH_TOKEN=...

# Analytics (PostHog)
NEXT_PUBLIC_POSTHOG_KEY=...
NEXT_PUBLIC_POSTHOG_HOST=...
```

---

## Seed Data

Seed the database with:
- **100+ currencies** (USD, EUR, GBP, JPY, INR, CAD, AUD, CHF, CNY, etc.)
- **Category tree** (7 parent categories, 40+ subcategories as listed above)
- **Demo users** (for development): Alice, Bob, Charlie with sample expenses and groups

---

## Non-Functional Requirements

- **Performance**: Server components by default; client components only for interactivity. Use React Suspense + streaming for data-heavy pages. See [Performance Benchmarks](#performance-benchmarks) for latency targets and caching strategy.
- **Mobile-first**: Responsive design with bottom tab navigation on mobile, sidebar on desktop.
- **Accessibility**: ARIA labels, keyboard navigation, focus management, color contrast AA compliant. See [Accessibility Audit](#accessibility-audit) for full compliance plan.
- **Security**: CSRF protection (Auth.js built-in), input validation (Zod schemas), parameterized queries (Drizzle), rate limiting on API routes. See [Security Hardening](#security-hardening) and [Rate Limiting](#rate-limiting) for details.
- **Testing**: Vitest for unit tests (balance calculation, simplify debts), Playwright for E2E. See [Testing Strategy](#testing-strategy) for coverage targets and test plans.
- **Error handling**: Global error boundaries, toast notifications for action feedback, optimistic updates with rollback. See [Error Monitoring & Observability](#error-monitoring--observability) for production monitoring.
- **Compliance**: GDPR data export/deletion, cookie consent, data retention policies. See [Compliance & Legal](#compliance--legal).
- **CI/CD**: Automated lint, type-check, test, and build pipeline. See [CI/CD Pipeline](#cicd-pipeline).
- **Observability**: Structured logging, error tracking, uptime monitoring. See [Error Monitoring & Observability](#error-monitoring--observability).
- **Backup**: Automated database backups with point-in-time recovery. See [Backup & Disaster Recovery](#backup--disaster-recovery).

---

## Key Algorithms to Implement

### 1. Balance Calculator
Input: List of expenses with participants
Output: Per-pair debts, per-user net balances, simplified debts

### 2. Debt Simplification (Greedy)
Input: Map of user → net balance
Output: Minimal set of transfers

### 3. Multi-Payer Split Resolution
Input: Multiple payers with amounts, multiple owees with shares
Output: `expense_participants` rows with correct `paid_share` and `owed_share`

### 4. Recurring Expense Cloner
Input: Expense with repeat_interval
Output: New expense with updated date, updated next_repeat_date on original

---

## Testing Strategy

### Unit Tests (Vitest)
- **Coverage target**: 80%+ for business logic, 60%+ overall
- **Critical paths to test**:
  - Balance calculation engine (equal splits, exact amounts, percentages, shares, adjustments)
  - Debt simplification algorithm (2-person, 3-person, N-person, single currency, multi-currency)
  - Multi-payer split resolution (edge cases: rounding, zero amounts, single participant)
  - Recurring expense cloner (weekly, biweekly, monthly, yearly interval advancement)
  - Currency rounding edge cases (e.g., $10 split 3 ways → $3.34, $3.33, $3.33)
- **Test data factories**: Use a `createTestExpense()`, `createTestGroup()`, `createTestUser()` factory pattern with sensible defaults and overrides via partial params
- **Mocking strategy**: Mock database calls with Drizzle's type-safe query builder; mock external services (Pusher, Resend, OCR) with Vitest `vi.mock()`

### Integration Tests (Vitest + test database)
- Spin up a test PostgreSQL database (use `pg-mem` or a Docker container via `testcontainers`)
- Test full server action flows: create expense → verify balances → settle up → verify zero balances
- Test auth flows with mocked Auth.js sessions
- Test Drizzle migrations apply cleanly on a fresh database

### E2E Tests (Playwright)
- **Core flows**:
  - Sign up → create group → add members → add expense → verify balances → settle up
  - Send friend request → accept → add expense between friends
  - Edit expense → verify system comment generated
  - Delete expense → verify balance recalculation
  - Recurring expense creation → verify cron cloning (mock time)
- **Visual regression**: Capture screenshots of key pages for regression comparison
- Run against a seeded test database with predictable data

### Test Fixtures
- Seed scripts for test environments with deterministic UUIDs
- Fixture sets: "empty group," "group with 5 expenses," "group with simplified debts," "multi-currency group"

---

## CI/CD Pipeline

### GitHub Actions Workflow

```yaml
# .github/workflows/ci.yml
name: CI
on: [push, pull_request]
jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20 }
      - run: npm ci
      - run: npm run lint          # ESLint
      - run: npm run type-check    # tsc --noEmit
      - run: npm run test          # Vitest unit + integration
      - run: npm run build         # Next.js production build

  e2e:
    runs-on: ubuntu-latest
    needs: quality
    services:
      postgres:
        image: postgres:16
        env: { POSTGRES_DB: monkeysplit_test, POSTGRES_PASSWORD: test }
        ports: ['5432:5432']
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20 }
      - run: npm ci
      - run: npx playwright install --with-deps
      - run: npm run db:migrate
        env: { DATABASE_URL: postgresql://postgres:test@localhost:5432/monkeysplit_test }
      - run: npm run db:seed
      - run: npx playwright test
```

### Deployment Strategy
- **Preview deployments**: Vercel auto-deploys every PR to a unique preview URL — enable GitHub integration and require passing CI checks before merge
- **Production deployments**: Merge to `main` triggers production deploy on Vercel
- **Database migrations**: Run `drizzle-kit push` as a build step on Vercel (add to `vercel.json` build command) or use a separate migration job
- **Environment variable management**: Use Vercel Environment Variables UI with three scopes: Development, Preview, Production. Never commit `.env` files. Use `.env.example` as a template.

---

## Error Monitoring & Observability

### Error Tracking — Sentry
- Install `@sentry/nextjs` and configure for both client and server
- Capture unhandled exceptions, rejected promises, and server action errors
- Tag errors with user ID, group ID, and action context
- Set up Sentry alerts for error rate spikes (> 5 errors/min)

### Structured Logging — Pino
- Use `pino` for server-side structured JSON logging
- Log levels: `error` for failures, `warn` for degraded paths, `info` for key actions (expense created, settlement recorded), `debug` for development
- Include correlation IDs (request ID) across log entries
- In production, ship logs to Vercel's built-in log drain or connect to Datadog/Logflare

### Uptime Monitoring
- Use Vercel's built-in cron monitoring to verify `recurring-expenses` cron runs successfully
- Set up an external uptime monitor (e.g., BetterUptime or UptimeRobot) for the production URL
- Health check endpoint: `GET /api/health` → returns `{ status: 'ok', db: 'connected', timestamp }` by querying the database with a simple `SELECT 1`

### Performance Monitoring
- Track Core Web Vitals via Vercel Analytics (built-in) or Sentry Performance
- Monitor server action durations with custom Sentry spans
- Set alert thresholds: LCP < 2.5s, FID < 100ms, CLS < 0.1

---

## Rate Limiting

### Strategy
Use **Upstash Redis** with the `@upstash/ratelimit` package for serverless-compatible rate limiting.

### Rate Limits by Route

| Route Pattern | Limit | Window | Notes |
|--------------|-------|--------|-------|
| `POST /api/auth/*` | 10 requests | 1 minute | Prevent brute force |
| `POST /api/expenses` | 30 requests | 1 minute | Per authenticated user |
| `POST /api/settlements` | 10 requests | 1 minute | Per authenticated user |
| `POST /api/friends` | 20 requests | 1 minute | Prevent spam invites |
| `POST /api/upload` | 5 requests | 1 minute | Limit file uploads |
| `POST /api/ocr` | 3 requests | 1 minute | Expensive operation |
| `GET /api/*` | 100 requests | 1 minute | General read endpoints |

### Implementation
- Apply rate limiting in Next.js middleware (`middleware.ts`) for API routes
- Use sliding window algorithm via `@upstash/ratelimit`
- Return `429 Too Many Requests` with `Retry-After` header
- Environment variables: `UPSTASH_REDIS_REST_URL`, `UPSTASH_REDIS_REST_TOKEN`

---

## Data Migration & Schema Evolution

### Drizzle Migration Workflow
- Generate migrations: `npx drizzle-kit generate:pg` after schema changes
- Apply migrations: `npx drizzle-kit push:pg` (development) or `npx drizzle-kit migrate` (production with migration files)
- Store migration files in `drizzle/migrations/` and commit to version control
- Review generated SQL before applying to production

### Rollback Procedures
- Keep a `drizzle/rollbacks/` directory with manual rollback SQL for each migration
- For additive changes (new columns, tables): rollback by dropping the addition
- For destructive changes (column removal, type changes): use a two-phase approach:
  1. Deploy code that handles both old and new schema
  2. Run migration
  3. Deploy code that only uses new schema
- Test migrations against a staging database before production

### Zero-Downtime Migrations
- Prefer additive migrations (add nullable columns, new tables)
- Never rename or drop columns directly — use add → migrate data → remove pattern
- Use `DEFAULT` values for new non-nullable columns
- Run migrations during low-traffic periods as a precaution

---

## Backup & Disaster Recovery

### Database Backups
- **Neon**: Automatic daily backups with 7-day retention on Pro plan; point-in-time recovery (PITR) to any second within the retention window
- **Supabase**: Daily backups with 7-day retention (Pro); point-in-time recovery available on Pro plan
- Document which provider is in use and the specific backup configuration

### Recovery Strategy
- **Point-in-time recovery**: Use provider's PITR to restore to a moment before data corruption
- **Branch-based recovery** (Neon): Create a database branch from a past state, verify data, then promote or copy data back
- **RTO target**: < 1 hour for full recovery
- **RPO target**: < 1 minute (with PITR)

### Data Export & Portability
- Admin endpoint: `GET /api/admin/export` — exports all user data as JSON (for GDPR and portability)
- User-facing: "Export my data" button in settings → generates a ZIP with expenses, groups, balances as CSV/JSON
- Regular automated export to cloud storage (optional, for additional safety)

---

## Security Hardening

### Input Sanitization
- Use Zod schemas for all input validation on server actions and API routes
- Sanitize user-generated content (descriptions, notes, comments, whiteboard) with `DOMPurify` or `sanitize-html` before rendering to prevent stored XSS
- Escape HTML in all user-facing text rendered outside of React's auto-escaping (e.g., email templates)

### Security Headers
Configure in `next.config.js` or `vercel.json`:
```
Content-Security-Policy: default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'; img-src 'self' blob: data: https:;
Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(), geolocation=()
```

### Dependency Vulnerability Scanning
- Enable **Dependabot** on the GitHub repo for automatic dependency update PRs
- Run `npm audit` in CI pipeline; fail the build on critical/high vulnerabilities
- Consider **Snyk** for deeper scanning and license compliance

### Session Management
- Auth.js session strategy: JWT with 30-day expiry
- Rotate session tokens on privilege escalation (e.g., changing email, enabling app lock)
- Invalidate all sessions on password change
- Implement CSRF tokens for all state-changing operations (Auth.js provides this by default)
- Secure cookie settings: `httpOnly`, `secure`, `sameSite: 'lax'`

---

## Performance Benchmarks

### Latency Targets

| Operation | Target | Notes |
|-----------|--------|-------|
| Balance calculation (group of 20) | < 200ms | Server-side, cached |
| Simplify debts (group of 20) | < 100ms | In-memory algorithm |
| Expense list page load | < 500ms | With pagination (20 items) |
| Dashboard load | < 800ms | Activity feed + balances |
| Expense creation (server action) | < 300ms | Including balance recalculation |

### Database Optimization
- **Indexes**: Add indexes on `expenses(group_id, deleted_at)`, `expenses(friendship_id)`, `expense_participants(user_id)`, `activity_log(group_id, created_at)`, `notifications(user_id, read)`
- **Query optimization**: Use `EXPLAIN ANALYZE` on critical queries during development; aim for index scans over sequential scans
- **Connection pooling**: Use Neon's built-in connection pooler or PgBouncer; configure Drizzle with `max: 10` connections for serverless

### Caching Strategy
- **React `cache()`**: Wrap expensive balance calculations for request-level deduplication
- **ISR (Incremental Static Regeneration)**: Use for semi-static pages like category lists, currency lists (revalidate every 24h)
- **TanStack Query**: Client-side cache with `staleTime: 30s` for balance data, `staleTime: 5m` for user profile
- **Redis cache** (Upstash): Cache exchange rates (TTL: 1 hour), group balances (TTL: 30s, invalidated on expense mutation)

---

## Accessibility Audit

### WCAG AA Compliance
- Run **axe-core** automated checks in CI (via `@axe-core/playwright` in E2E tests)
- Run Lighthouse accessibility audit; target score > 90
- Manual audit checklist for each major feature before release

### Screen Reader Testing
- Test with VoiceOver (macOS/iOS) and NVDA (Windows) on key flows:
  - Creating an expense (form navigation, split method switching)
  - Viewing balances (tabular data, debt summaries)
  - Notification list (unread counts, marking as read)
- Ensure all images have `alt` text; decorative icons use `aria-hidden="true"`
- Use `aria-live="polite"` regions for real-time updates (new expenses, balance changes)

### Keyboard Navigation
- All interactive elements reachable via Tab
- Expense form: Tab through fields in logical order, Enter to submit
- Category picker: Arrow keys to navigate, Enter to select
- Itemization drag-and-drop: Provide keyboard alternative (arrow keys to reorder, Space to assign)
- Modal dialogs: Focus trap, Escape to close
- Skip navigation link at top of page

---

## Compliance & Legal

### GDPR Compliance
- **Data export**: "Download my data" feature in user settings → exports all personal data as JSON/CSV ZIP
- **Data deletion**: "Delete my account" flow:
  1. Warn about unsettled balances
  2. Anonymize expense participation (replace name/email with "Deleted User")
  3. Delete user record, sessions, notifications
  4. Retain anonymized financial records for other users' balance integrity
- **Right to rectification**: Users can edit their profile data at any time

### Legal Pages
- `/terms` — Terms of Service page (static MDX)
- `/privacy` — Privacy Policy page (static MDX)
- Link to both in the footer and during registration

### Cookie Consent
- MonkeySplit uses only essential cookies (session, CSRF) — no tracking cookies in base deployment
- If analytics are added (PostHog, etc.), implement a cookie consent banner using a lightweight library (e.g., `cookie-consent-banner`)
- Store consent preference in a cookie; only load analytics scripts after consent

### Data Retention
- Active user data: retained indefinitely while account is active
- Soft-deleted expenses: permanently purged after 90 days via a scheduled cron job
- Deleted accounts: personal data removed within 30 days; anonymized financial records retained
- Activity logs: retained for 1 year, then archived or purged

---

## Analytics

### Product Analytics — PostHog
- Self-hosted or cloud PostHog instance
- Install `posthog-js` and initialize in the app layout (only after cookie consent if required)
- Server-side event tracking via PostHog Node SDK for critical backend events

### Key Metrics to Track

| Metric | Event | Purpose |
|--------|-------|---------|
| DAU / MAU | `page_view` | User engagement |
| Expenses created | `expense_created` | Core feature adoption |
| Settlement rate | `settlement_recorded` / `expense_created` | Feature completion |
| Group creation | `group_created` | Collaboration adoption |
| Invite acceptance rate | `invite_accepted` / `invite_sent` | Growth funnel |
| OCR usage | `ocr_scanned` | Premium feature adoption |
| Retention (D1, D7, D30) | Session tracking | User stickiness |

### Feature Flags
- Use PostHog feature flags to gate Phase 3 features (OCR, Stripe Connect, charts)
- Roll out new features gradually with percentage-based rollouts

---

## Edge Case Documentation

### Placeholder Account Merging
When a non-registered user is invited to a group or added as a friend:
1. Create a placeholder user record with `email` only (no password, no OAuth)
2. Associate expenses and group memberships with the placeholder
3. When the user registers with that email:
   - Match by email address
   - Merge placeholder into the new real account (update `user_id` FK references)
   - Transfer all group memberships, expense participations, and friend connections
   - Delete the placeholder record
4. Edge case: if user registers with a different email, allow manual claim via invite link token

### Concurrent Expense Edits
- Use **optimistic locking** via an `updated_at` timestamp
- When saving an edit, check that `updated_at` matches the value when the form was loaded
- If mismatch: reject the save with a "This expense was modified by another user" error and show the latest version
- Real-time: push edit notifications via Pusher so other viewers see changes live

### Currency Rounding Edge Cases
- **Splitting $10.00 three ways**: $3.34 + $3.33 + $3.33 = $10.00
  - Assign the extra cent to the first participant in alphabetical order (deterministic)
- **General rule**: Calculate each share as `floor(amount * 100 / N) / 100`, then distribute the remainder cents one-by-one to participants
- **Multi-decimal currencies** (e.g., KWD with 3 decimal places): Respect the `decimal_places` field from the `currencies` table
- **Zero-amount shares**: Allow $0.00 shares for participants who are "involved" but not paying/owing (e.g., a child in a family group)

### Group Deletion with Unsettled Balances
- **Prevent hard deletion** if any non-zero balances exist between group members
- **Soft delete**: Mark group as deleted, hide from UI, but preserve for balance resolution
- Show a warning: "This group has unsettled balances. Members can still access it to settle up."
- Once all balances are $0.00, allow permanent deletion (or auto-purge after 90 days)

### Additional Edge Cases
- **Self-expense**: Prevent adding an expense where the same user is the only payer and only participant
- **Empty groups**: Allow groups with no expenses; show helpful onboarding prompts
- **Large groups** (50+ members): Paginate member lists; optimize balance calculation with caching
- **Timezone handling**: Store all timestamps in UTC; display in user's locale timezone
- **Network failures**: Optimistic UI updates with rollback on server error; queue failed actions for retry

---

*This prompt serves as the complete specification for building MonkeySplit. Each phase can be implemented incrementally, with Phase 1+2 forming the MVP.*
