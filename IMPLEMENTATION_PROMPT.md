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
```

---

## Seed Data

Seed the database with:
- **100+ currencies** (USD, EUR, GBP, JPY, INR, CAD, AUD, CHF, CNY, etc.)
- **Category tree** (7 parent categories, 40+ subcategories as listed above)
- **Demo users** (for development): Alice, Bob, Charlie with sample expenses and groups

---

## Non-Functional Requirements

- **Performance**: Server components by default; client components only for interactivity. Use React Suspense + streaming for data-heavy pages.
- **Mobile-first**: Responsive design with bottom tab navigation on mobile, sidebar on desktop.
- **Accessibility**: ARIA labels, keyboard navigation, focus management, color contrast AA compliant.
- **Security**: CSRF protection (Auth.js built-in), input validation (Zod schemas), parameterized queries (Drizzle), rate limiting on API routes.
- **Testing**: Vitest for unit tests (balance calculation, simplify debts), Playwright for E2E.
- **Error handling**: Global error boundaries, toast notifications for action feedback, optimistic updates with rollback.

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

*This prompt serves as the complete specification for building MonkeySplit. Each phase can be implemented incrementally, with Phase 1+2 forming the MVP.*
