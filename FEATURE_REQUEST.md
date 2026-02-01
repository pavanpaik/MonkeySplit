# Feature Request: Update IMPLEMENTATION_PROMPT.md with Production Readiness Gaps

## Summary

The current `IMPLEMENTATION_PROMPT.md` is a comprehensive blueprint but lacks several details needed for a truly production-ready system. Based on a confidence evaluation (~70-80% overall), the following areas should be added or expanded to bring production confidence to ~90%.

---

## Proposed Additions

### 1. Testing Strategy
- Define a concrete test plan: unit test coverage targets, integration test scope, E2E test scenarios
- Specify what to test with Vitest (balance calculation, debt simplification, split resolution, edge cases)
- Define Playwright E2E flows (auth, create expense, settle up, group invite)
- Add test data factories / fixtures strategy

### 2. CI/CD Pipeline
- GitHub Actions workflow for lint, type-check, test, build
- Preview deployments on PRs (Vercel handles this, but should be documented)
- Database migration strategy for staging/production environments
- Environment variable management across environments

### 3. Error Monitoring & Observability
- Add Sentry (or equivalent) for error tracking
- Structured logging strategy (e.g., Pino or Winston)
- Uptime monitoring for cron jobs and critical paths
- Performance monitoring / Web Vitals tracking

### 4. Rate Limiting Implementation
- The spec mentions rate limiting but provides no details
- Define rate limiting strategy per API route (e.g., auth endpoints, expense creation)
- Specify tooling: Upstash Redis rate limiter, Vercel Edge middleware, or similar

### 5. Data Migration & Schema Evolution
- Strategy for handling schema changes over time with Drizzle migrations
- Rollback procedures for failed migrations
- Zero-downtime migration approach

### 6. Backup & Disaster Recovery
- Database backup schedule and retention policy
- Point-in-time recovery strategy (Neon/Supabase provide this, but should be documented)
- Data export / portability plan

### 7. Security Hardening
- Input sanitization beyond Zod validation (XSS prevention in user-generated content)
- Security headers (CSP, HSTS — partially Vercel-handled but should be explicit)
- Dependency vulnerability scanning (Dependabot / Snyk)
- Authentication session management details (token rotation, session expiry)

### 8. Performance Benchmarks
- Define acceptable latency targets (e.g., balance calculation < 200ms for groups of 20)
- Database query optimization guidelines (indexes, query plans)
- Connection pooling configuration
- Caching strategy (React cache, ISR, Redis)

### 9. Accessibility Audit
- WCAG AA compliance testing plan (axe-core, Lighthouse)
- Screen reader testing checklist
- Keyboard navigation testing for complex components (expense form, itemization drag-and-drop)

### 10. Compliance & Legal
- GDPR data deletion / export capabilities
- Terms of service and privacy policy pages
- Cookie consent handling
- Data retention policies

### 11. Analytics
- Product analytics integration (PostHog, Mixpanel, or similar)
- Key metrics to track (DAU, expenses created, settlement rate)

### 12. Edge Case Documentation
- Placeholder account merging when invited users register
- Handling concurrent expense edits
- Currency rounding edge cases (splitting $10 three ways)
- Group deletion with unsettled balances

---

## Confidence Breakdown

| Area | Confidence | Notes |
|------|-----------|-------|
| Project scaffolding, DB schema, Auth | 90%+ | Standard, well-documented stack |
| Balance engine, debt simplification | 90%+ | Algorithms explicitly specified |
| CRUD operations, routing, UI | 90%+ | Standard Next.js patterns |
| Real-time (Pusher/Ably) | 60-75% | External service dependency |
| Multi-currency, OCR, recurring | 60-75% | External APIs, edge cases |
| Stripe Connect | 50-60% | Complex onboarding flow |
| Full production readiness | ~70% | Missing operational concerns listed above |

## Impact

Adding these sections would close the gap between "feature complete" and "production hardened," primarily affecting Phase 3 and operational readiness.
