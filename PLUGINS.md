# MonkeySplit Plugins & MCP Integrations

Recommended plugins and MCP servers for each release phase.

---

## Alpha 0.1.0 — Core Development

| Plugin | Type | Purpose | Install |
|--------|------|---------|---------|
| **Neon MCP** | MCP Server | Database queries, schema mgmt | `@neondatabase/mcp-server-neon` |
| **GitHub MCP** | MCP Server | PRs, issues, code search | `@modelcontextprotocol/server-github` |
| **Filesystem MCP** | MCP Server | File operations | `@modelcontextprotocol/server-filesystem` |
| **Drizzle Skill** | Skill | ORM patterns, migrations | mcpmarket.com/drizzle-orm |
| **Next.js-shadcn Skill** | Skill | App Router + UI patterns | mcpmarket.com/nextjs-shadcn |

---

## Release 0.2.0 — Split Methods & Settlements

| Plugin | Type | Purpose |
|--------|------|---------|
| **Postgres MCP** | MCP Server | Direct SQL queries for complex calculations |

---

## Release 0.3.0 — Social & Communication

| Plugin | Type | Purpose |
|--------|------|---------|
| **Resend MCP** | MCP Server | Email template testing, send logs |

---

## Release 0.4.0 — Multi-Currency & Real-time

| Plugin | Type | Purpose |
|--------|------|---------|
| **Pusher MCP** | MCP Server | Real-time event testing |

---

## Release 0.5.0 — Premium Features

| Plugin | Type | Purpose |
|--------|------|---------|
| **OpenAI Vision** | API | OCR for receipt scanning |

---

## Release 0.6.0 — Production Hardening

| Plugin | Type | Purpose |
|--------|------|---------|
| **Vercel Plugin** | Plugin | Deploy, logs, preview envs |
| **Sentry MCP** | MCP Server | Error tracking |
| **Playwright MCP** | MCP Server | E2E test automation |

---

## Configuration

MCP servers are configured in `.mcp/config.json`. Set required env vars:

```bash
export NEON_API_KEY=your-neon-api-key
export GITHUB_TOKEN=your-github-token
export DATABASE_URL=postgres://...
```

## Usage

With Claude Code:
```bash
# Start Claude Code with MCP servers
claude --mcp-config .mcp/config.json
```
