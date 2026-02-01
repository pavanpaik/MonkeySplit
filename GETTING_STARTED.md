# Getting Started with MonkeySplit

This guide helps you set up the development environment for MonkeySplit.

## Prerequisites

### Required Accounts

| Service | Purpose | Sign Up |
|---------|---------|---------|
| **Neon** | PostgreSQL database | [neon.tech](https://neon.tech) (free tier) |
| **Resend** | Email magic links | [resend.com](https://resend.com) (free tier) |
| **Vercel** | Deployment | [vercel.com](https://vercel.com) |
| **GitHub** | Code hosting | [github.com](https://github.com) |

### Local Tools

Verify you have these installed:
```bash
node -v   # Should be 18+
npm -v    # Should be 9+
git -v
```

## Setup Steps

### 1. Clone the Repository

```bash
git clone https://github.com/pavanpaik/MonkeySplit.git
cd MonkeySplit
```

### 2. Create Neon Database

1. Go to [neon.tech](https://neon.tech) → Create Project
2. Name it `monkeysplit`
3. Copy the connection string (with pooler)

### 3. Create Resend API Key

1. Go to [resend.com](https://resend.com) → API Keys
2. Create a new API key
3. For local dev, use their test domain or add your own

### 4. Configure Environment Variables

```bash
cp .env.example .env.local
```

Edit `.env.local`:
```bash
# Database (from Neon dashboard)
DATABASE_URL=postgres://user:pass@ep-xxx.us-east-2.aws.neon.tech/monkeysplit?sslmode=require

# Auth
NEXTAUTH_URL=http://localhost:3000
NEXTAUTH_SECRET=your-secret-here  # Generate: openssl rand -base64 32
RESEND_API_KEY=re_xxxxx

# Optional: MCP integrations (for Claude Code)
NEON_API_KEY=
GITHUB_TOKEN=
```

### 5. Install Dependencies

```bash
npm install
```

### 6. Initialize Database

```bash
npx drizzle-kit push:pg
```

### 7. Start Development Server

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000)

---

## 🤖 First-Time Bootstrap (AI-Powered)

For a **completely autonomous** project setup, copy this prompt into Claude Code:

```
You are bootstrapping MonkeySplit from scratch. Execute all Alpha 0.1.0 workstreams in order:

1. Read `.agent/context/core.md` for project standards
2. Execute each workstream in `.agent/subagents/` following their instructions:
   - WS-0.1-A: Scaffolding (create Next.js app, shadcn/ui)
   - WS-0.1-B: Database (Drizzle schema, push to Neon)
   - WS-0.1-C: Auth (Auth.js magic link setup)
   - WS-0.1-D: Balance Engine (calculation algorithm + tests)
   - WS-0.1-E: Groups (CRUD, member management)
   - WS-0.1-F: Expenses (form, equal split)
   - WS-0.1-G: Integration (layout, navigation, E2E test)

For each workstream:
- Read the subagent file for instructions
- Read referenced skill files for patterns
- Create deliverables listed in the workstream
- Run verification commands before proceeding
- Create handoff report in handoffs/WS-X.Y-Z.md

Prerequisites already configured:
- DATABASE_URL in .env.local
- RESEND_API_KEY in .env.local
- NEXTAUTH_SECRET in .env.local

Start with WS-0.1-A and work through to WS-0.1-G. Stop and report after each workstream completes.
```

This will scaffold the entire Alpha 0.1.0 release autonomously.

---

## Optional: Claude Code with MCP

To use natural language database operations:

```bash
# Set MCP env vars
export NEON_API_KEY=your-neon-api-key
export GITHUB_TOKEN=your-github-token

# Start Claude Code with MCP config
claude --mcp-config .mcp/config.json
```

## Troubleshooting

### Database Connection Errors
- Ensure `?sslmode=require` is in your DATABASE_URL
- Check Neon dashboard for connection limits

### Email Not Sending
- Verify RESEND_API_KEY is correct
- Check Resend dashboard for delivery logs
- For local testing, check console for magic link URL

## Next Steps

- Read `IMPLEMENTATION_PROMPT.md` for full specifications
- See `CLAUDE.md` for coding standards
- Check `.agent/subagents/` for workstream definitions
