# Starting Development

This guide explains how to begin working on MonkeySplit.

## Prerequisites

### 1. Create Accounts
| Service | Purpose | Sign Up |
|---------|---------|---------|
| **Neon** | PostgreSQL database | [neon.tech](https://neon.tech) |
| **Resend** | Email magic links | [resend.com](https://resend.com) |

### 2. Configure Environment
```bash
cp .env.example .env.local
```

Edit `.env.local`:
```bash
DATABASE_URL=postgres://...@neon.tech/monkeysplit?sslmode=require
NEXTAUTH_SECRET=your-secret-here   # openssl rand -base64 32
RESEND_API_KEY=re_xxxxx
```

---

## Start Development

### Option A: AI-Assisted (Recommended)

Tell Claude:
```
Execute WS-0.1-A: Scaffolding
```

The scaffolder agent sets up Next.js, Tailwind, and shadcn/ui.

### Option B: Manual
```bash
npx create-next-app@latest . --typescript --tailwind --eslint --app --no-src-dir
npx shadcn@latest init
npm run dev
```

---

## Workstream Execution Order

| # | Command | Description |
|---|---------|-------------|
| 1 | `Execute WS-0.1-A` | Project scaffolding |
| 2 | `Execute WS-0.1-B` | Database schema |
| 3 | `Execute WS-0.1-C` | Authentication |
| 4 | `Execute WS-0.1-D` | Balance algorithm |
| 5 | `Execute WS-0.1-E` | Groups feature |
| 6 | `Execute WS-0.1-F` | Expenses feature |
| 7 | `Execute WS-0.1-G` | Integration & polish |

Each workstream creates a handoff report in `handoffs/WS-X.Y-Z.md` when complete.

---

## Using Claude Code Agents

View available agents:
```
/agents
```

Use a specific agent:
```
Use the scaffolder agent to set up the project
Use the database-architect agent to design the schema
Use the tester agent to write tests for the balance calculator
```

Agents automatically load relevant skills from `.agent/skills/`.
