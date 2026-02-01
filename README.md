# MonkeySplit

Open-source expense-splitting app — a Splitwise alternative built on the Vercel stack.

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Framework | Next.js 14+ (App Router) |
| Language | TypeScript (strict) |
| Styling | Tailwind CSS + shadcn/ui |
| Database | PostgreSQL (Neon) + Drizzle ORM |
| Auth | Auth.js v5 (Email magic link) |

## Quick Start

```bash
# Clone and install
git clone https://github.com/pavanpaik/MonkeySplit.git
cd MonkeySplit
npm install

# Configure environment
cp .env.example .env.local
# Edit .env.local with your Neon and Resend credentials

# Run development server
npm run dev
```

See [GETTING_STARTED.md](GETTING_STARTED.md) for detailed setup instructions.

---

## AI-Powered Development

This project includes Claude Code agents and skills for AI-assisted development.

### Agents (`.claude/agents/`)
Reusable, project-agnostic specialists:

| Agent | Purpose |
|-------|---------|
| `scaffolder` | Project initialization |
| `database-architect` | Schema design |
| `auth-implementer` | Authentication setup |
| `tester` | Unit and E2E tests |
| `reviewer` | Code review (read-only) |
| `debugger` | Root cause analysis |
| `feature-builder` | End-to-end features |

### Skills (`.agent/skills/`)
Pattern libraries for specific technologies:
- `nextjs-app-router` — App Router patterns
- `drizzle-orm` — Database operations
- `authjs-magic-link` — Auth.js setup
- `balance-calculation` — Expense splitting algorithm
- `vitest-testing` — Testing patterns
- `design-system` — UI guidelines

### Workstreams (`.agent/subagents/`)
Project-specific task definitions for Alpha 0.1.0 release.

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for development guidelines.

## License

MIT
