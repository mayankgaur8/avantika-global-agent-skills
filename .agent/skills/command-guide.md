# Command Guide — Agent Skills Quick Reference

How to invoke every skill with real, copy-paste prompts for your projects.
Works with: Claude Code (VS Code), Cursor, any Claude agent context.

---

## Bug Fixing

### Generic
```
Use the bug-fixer skill to resolve this error:

[paste full error + stack trace here]
```

### With file context
```
Use the bug-fixer skill. Error in src/auth/refresh.ts:

TypeError: Cannot read properties of undefined (reading 'userId')
  at refreshToken (src/auth/refresh.ts:42:18)
  at POST /api/auth/refresh
```

### For GlobalResumeAI
```
Use the bug-fixer skill for GlobalResumeAI (Next.js + Prisma).
Error when saving resume: "Unique constraint failed on field: userId_resumeId"
File: src/app/api/resume/save/route.ts
```

### For CAT Prep App
```
Use the bug-fixer skill. The mock test timer resets when navigating between questions.
Component: src/components/MockTest/Timer.tsx
State management: Zustand store at src/store/testStore.ts
```

### For Spring Boot Microservices
```
Use the bug-fixer skill for a Spring Boot service.
Exception: org.springframework.dao.DataIntegrityViolationException
  at UserService.createUser(UserService.java:87)
Stack: user-service module, PostgreSQL backend
```

### For Avantika English Coach
```
Use the bug-fixer skill for Avantika English Coach.
Bug: Grammar correction returns malformed JSON — the UI fails to render error cards.
File: src/app/api/grammar-check/route.ts
Expected: { original, corrected, errors: [...] }
Actual: intermittent JSON parse error on Claude responses > 800 tokens
```

### For AI SaaS Platform
```
Use the bug-fixer skill for AI SaaS Platform.
Bug: Streaming stops mid-response for prompts > 2000 tokens.
File: src/app/api/chat/route.ts
Error in browser console: "ReadableStream cancelled: client disconnected"
Suspected: Vercel serverless function 10s timeout cutting the stream.
```

---

## Production Readiness Audit

### Before first deploy
```
Use the production-readiness skill.
Audit this entire project before we deploy to production.
Tech stack: [Next.js / Spring Boot / FastAPI / etc.]
Target: [Vercel / Azure / Railway / etc.]
```

### For GlobalResumeAI
```
Use the production-readiness skill for GlobalResumeAI.
We are deploying to Vercel + Supabase next week.
Stack: Next.js 15, Prisma, PostgreSQL (Supabase), NextAuth, Anthropic API.
Check: env vars, auth config, API security, DB connections, CORS, error handling.
```

### For Avantika English Coach
```
Use the production-readiness skill for Avantika English Coach.
Stack: Next.js, MongoDB, OpenAI.
Focus on: rate limiting on AI endpoints, user data privacy, error responses.
```

### For Spring Boot Microservices
```
Use the production-readiness skill on the user-service module.
Stack: Spring Boot 3, PostgreSQL, JWT auth, Docker.
Check: actuator endpoints security, secret management, Hibernate N+1 queries.
```

### For CAT Prep App
```
Use the production-readiness skill for CAT Prep App.
Deploy target: Vercel.
Priority checks:
- Server-side score calculation (no client trust)
- Timer state persists on page refresh (DB checkpoint every 30s)
- AI explanation endpoint has per-user rate limit
- Mock test data is isolated per user (no cross-user leakage)
```

### For AI SaaS Platform
```
Use the production-readiness skill for AI SaaS Platform.
Public launch target: Vercel + Upstash Redis + Supabase PostgreSQL.
Critical checks:
- Per-user AI rate limiting enforced (Upstash)
- Token usage tracked in DB for billing
- Stripe webhooks verified (signature check)
- Users cannot access each other's AI conversation history
- ANTHROPIC_API_KEY never reaches browser bundle
```

---

## UI/UX Polishing

### Generic
```
Use the ui-ux-polisher skill.
Pages to review: [list pages]
Priority: [mobile responsiveness / accessibility / visual consistency]
```

### For GlobalResumeAI
```
Use the ui-ux-polisher skill for GlobalResumeAI.
Review these pages: ResumeBuilder, Dashboard, AISuggestions, ExportPDF.
Priority issues:
- Resume preview doesn't scale on mobile
- Toolbar buttons have no visible focus state
- AI suggestion cards look inconsistent
```

### For CAT Prep App
```
Use the ui-ux-polisher skill for CAT Prep App.
Review: QuestionCard, MockTestTimer, ScoreReport, SubjectSelector.
Focus on:
- Question cards on 375px mobile
- Timer visibility under stress (large, high contrast)
- Score breakdown chart accessibility
```

### For Avantika English Coach
```
Use the ui-ux-polisher skill for Avantika English Coach.
Review: ConversationInterface, VocabularyExercise, ProgressDashboard.
Focus on: readable typography, clear input/response distinction, loading states on AI calls.
```

---

## Deployment

### Vercel (Next.js)
```
Use the deployment-engineer skill.
Deploy this Next.js app to Vercel.
Env vars needed: DATABASE_URL, NEXTAUTH_SECRET, NEXTAUTH_URL, ANTHROPIC_API_KEY.
Domain: [yourdomain.com]
```

### Railway (Full-stack)
```
Use the deployment-engineer skill.
Deploy to Railway: Next.js frontend + PostgreSQL DB.
Set up: auto-deploy from main branch, health check on /api/health, env injection.
```

### Azure App Service (Spring Boot)
```
Use the deployment-engineer skill.
Containerize and deploy this Spring Boot app to Azure App Service.
Stack: Java 21, Spring Boot 3.2, PostgreSQL (Azure Database).
Set up: GitHub Actions CI/CD, Application Insights, Key Vault for secrets.
```

### Docker (any stack)
```
Use the deployment-engineer skill.
Create a production-grade Dockerfile for this [Next.js / Python / Java] app.
Requirements: multi-stage build, non-root user, health check endpoint.
```

### For GlobalResumeAI
```
Use the deployment-engineer skill for GlobalResumeAI.
Platform: Vercel (frontend) + Supabase (DB).
Configure: preview deployments for PRs, production env vars, custom domain.
```

### For Avantika English Coach
```
Use the deployment-engineer skill for Avantika English Coach.
Platform: Vercel.
Env vars needed: DATABASE_URL, NEXTAUTH_SECRET, NEXTAUTH_URL, ANTHROPIC_API_KEY, UPSTASH_REDIS_URL.
Configure: streaming routes use Edge Runtime (not Node.js serverless, to avoid 10s timeout).
```

### For CAT Prep App
```
Use the deployment-engineer skill for CAT Prep App.
Platform: Vercel.
Special requirements:
- DB connection must support concurrent users taking simultaneous mock tests
- Confirm Supabase connection pooler (pgBouncer) is used for serverless
- Static question bank pages can use ISR (revalidate: 86400)
```

### For AI SaaS Platform
```
Use the deployment-engineer skill for AI SaaS Platform.
Platform: Vercel (app) + Supabase (DB) + Upstash (Redis rate limiting).
Configure:
- Edge Runtime on streaming API routes (no cold-start latency, no 10s timeout)
- Stripe webhook endpoint excluded from auth middleware
- Cron job for monthly usage reset (Vercel Cron)
- Preview env uses test Stripe keys, never production keys
```

---

## Security Review

### Full audit
```
Use the security-reviewer skill.
Perform a complete security audit of this project.
Focus areas: authentication, authorization (IDOR), input validation, dependencies.
```

### Auth-focused
```
Use the security-reviewer skill.
Focus on the authentication and authorization layer.
Files: src/auth/, src/middleware.ts, src/app/api/*
Check: JWT config, route protection, role enforcement, session security.
```

### For GlobalResumeAI (pre-launch)
```
Use the security-reviewer skill for GlobalResumeAI pre-launch.
Critical checks:
1. Can user A access user B's resumes? (IDOR on /api/resume/[id])
2. Are Anthropic API keys server-side only?
3. Is NextAuth configured securely (NEXTAUTH_SECRET, NEXTAUTH_URL)?
4. Are Prisma queries using parameterized inputs?
```

### Dependency scan
```
Use the security-reviewer skill — dependency audit only.
Run npm audit, check for critical/high CVEs, and suggest upgrade path.
```

### For CAT Prep App
```
Use the security-reviewer skill for CAT Prep App.
Focus: test integrity — can users manipulate their own scores?
Check:
1. Score calculation happens server-side, never trusting client payload
2. Timer cannot be fast-forwarded via API manipulation
3. Question answers are not exposed in the initial page payload
4. Users can only submit answers for their own active test attempt
```

### For Avantika English Coach
```
Use the security-reviewer skill for Avantika English Coach.
Focus: user data privacy and AI prompt injection.
Check:
1. User conversation history is isolated per user (no cross-user access)
2. Claude prompt wraps user input in XML tags to prevent injection
3. Grammar correction responses are validated as JSON before storing
4. User learning progress data is not exposed in public API endpoints
```

### For Spring Boot Microservices
```
Use the security-reviewer skill for Spring Boot microservices.
Check across all services:
1. Spring Security filter chain — are all routes explicitly permitted or denied?
2. JWT validation: algorithm pinned (not accepting 'none'), expiry enforced
3. @PreAuthorize annotations on admin endpoints
4. No secrets in application.yml — using environment injection or Azure Key Vault
5. Actuator endpoints: only /health and /info exposed publicly
```

### For AI SaaS Platform
```
Use the security-reviewer skill for AI SaaS Platform.
Critical pre-launch checks:
1. BYOK (bring-your-own-key): user API keys encrypted with AES-256 before DB storage
2. Platform API key absent from any client-side bundle (verify with build output scan)
3. Stripe webhook: signature verified on every event (no signature = reject)
4. Multi-tenant isolation: userId enforced on every AI history query
5. Prompt injection: user content isolated in <user_input> XML tags
6. Rate limit bypass: test that removing Authorization header still gets rate-limited by IP
```

---

## Performance Optimization

### Slow page load
```
Use the performance-optimizer skill.
Page: [page name/route]
Current LCP: [Xs] — target: under 2.5s
Suspected issues: [large bundle / slow API / unoptimized images]
```

### Slow API endpoint
```
Use the performance-optimizer skill.
Endpoint: POST /api/resume/analyze
Current p99: 4.2s — target: under 1s
Stack: Next.js API route → Prisma → PostgreSQL
```

### For CAT Prep App
```
Use the performance-optimizer skill for CAT Prep App.
Issue: Mock test results page takes 6s to load for tests with 100+ questions.
Suspected: N+1 query loading question + answer + explanation per row.
File: src/app/api/test-result/[id]/route.ts
```

### Bundle size
```
Use the performance-optimizer skill — bundle analysis.
Run ANALYZE=true npm run build and identify packages over 100KB.
Suggest: lazy loading, tree-shaking, or lighter alternatives.
```

### For GlobalResumeAI
```
Use the performance-optimizer skill for GlobalResumeAI.
Issue: ResumeBuilder page is 4.1s LCP — likely heavy PDF preview library in initial bundle.
Also check: AI suggestion polling causing unnecessary re-renders on every keystroke.
Measure: bundle analyzer + React DevTools profiler on ResumeBuilder component.
```

### For Spring Boot Microservices
```
Use the performance-optimizer skill for Spring Boot order-service.
Issue: GET /api/orders?userId=X takes 2.1s for users with 500+ orders.
Suspected: Hibernate N+1 on Order → OrderItem → Product fetch.
Profile with: p6spy SQL logger or Hibernate statistics enabled.
Fix target: under 200ms with eager join fetch + index on orders.user_id.
```

### For Avantika English Coach
```
Use the performance-optimizer skill for Avantika English Coach.
Issue: Grammar correction first response feels slow (1.5s before streaming starts).
Cause: cold-start on serverless function + model warm-up time.
Fix options: Edge Runtime migration, keep-alive ping, or optimistic UI (show "thinking..." immediately).
```

### For AI SaaS Platform
```
Use the performance-optimizer skill for AI SaaS Platform.
Issues to address:
1. Dashboard page re-fetches usage stats on every tab focus — add 5-minute SWR cache
2. Chat history query has no index on (userId, createdAt) — add composite index
3. Streaming latency: first token takes 800ms — investigate Edge vs Node.js runtime
Measure all three before changing anything.
```

---

## Database & Prisma

### New schema
```
Use the database-prisma-sql-expert skill.
Design a Prisma schema for: [describe entities and relationships]
Requirements: soft delete, audit fields (createdAt, updatedAt), proper indexes.
```

### Slow query fix
```
Use the database-prisma-sql-expert skill.
This query is taking 800ms on a table with 50k rows:
[paste the Prisma query or raw SQL]
Add appropriate indexes and optimize the query.
```

### Migration
```
Use the database-prisma-sql-expert skill.
I need to add [describe change] to the [table] table.
Write a safe migration: add column/index without locking the table.
```

### For GlobalResumeAI
```
Use the database-prisma-sql-expert skill for GlobalResumeAI.
Add a ResumeVersion table for tracking edit history (max 10 versions per resume).
Schema must support: restore to version, diff between versions, auto-cleanup of oldest.
```

### For CAT Prep App
```
Use the database-prisma-sql-expert skill for CAT Prep App.
Optimize the TestAttempt + QuestionAttempt query for the results page.
Current: loads all 100 question attempts with N+1 on Question + Answer.
Target: single query with include, select only needed fields, add index on testAttemptId.
Also: add composite index on (userId, completedAt) for "my test history" page.
```

### For Spring Boot Microservices
```
Use the database-prisma-sql-expert skill for Spring Boot.
Write a safe Flyway migration to add audit columns to 3 existing tables:
- users: add last_login_at TIMESTAMP NULL
- orders: add updated_at TIMESTAMP DEFAULT NOW()
- products: add deleted_at TIMESTAMP NULL (soft delete)
Migration must not lock tables — use ALTER TABLE ADD COLUMN with DEFAULT, then backfill.
```

### For Avantika English Coach
```
Use the database-prisma-sql-expert skill for Avantika English Coach.
Design the Progress tracking schema:
- Track per-user vocabulary score, grammar score, fluency score over time
- Support weekly trend queries (score progression over last 8 weeks)
- Store lesson completion with timestamp
- Support streak tracking (consecutive days of activity)
```

### For AI SaaS Platform
```
Use the database-prisma-sql-expert skill for AI SaaS Platform.
Design the billing schema:
- aiUsage: userId, model, inputTokens, outputTokens, costUsd, createdAt
- creditLedger: userId, delta (positive=purchase, negative=spend), reason, createdAt
- subscriptions: userId, stripeSubscriptionId, plan, status, currentPeriodEnd
Need: fast query for "how many tokens has this user used this billing period?"
Add: index on (userId, createdAt) for range queries.
```

---

## AI SaaS Features

### Add streaming chat
```
Use the ai-saas-builder skill.
Add a streaming AI chat feature using Anthropic Claude claude-sonnet-4-6.
Stack: Next.js 15 App Router.
Requirements: per-user rate limiting (20 req/min), token usage tracking in DB.
```

### For GlobalResumeAI
```
Use the ai-saas-builder skill for GlobalResumeAI.
Feature: AI-powered resume tailoring — user pastes job description, 
Claude suggests bullet point rewrites and keyword additions.
Model: claude-sonnet-4-6 (streaming).
Store: save AI suggestions per resume session for "accept/reject" UI.
```

### For Avantika English Coach
```
Use the ai-saas-builder skill for Avantika English Coach.
Feature: conversational grammar correction — user types a sentence,
Claude identifies errors, explains them, and provides corrected version.
Requirement: structured JSON response (not freeform text) for UI rendering.
```

### For CAT Prep App
```
Use the ai-saas-builder skill for CAT Prep App.
Feature: AI explanation for wrong answers — after each test,
Claude explains why the correct answer is right using the question context.
Optimization: batch explanations for all wrong answers in one API call.
```

### For Spring Boot Microservices
```
Use the ai-saas-builder skill for Spring Boot.
Add AI-powered search to the product catalog service.
Stack: Spring Boot 3, Anthropic Java SDK.
Feature: natural language search ("show me red running shoes under ₹3000") 
→ Claude extracts structured filters → JPA query.
Output format: { category, color, maxPrice, keywords[] } as JSON.
Use claude-haiku-4-5-20251001 (fast + cheap for extraction tasks).
```

### For AI SaaS Platform (core feature build)
```
Use the ai-saas-builder skill for AI SaaS Platform.
Build the complete AI credit + billing system:
1. Track token usage per request (inputTokens, outputTokens, model, costUsd)
2. Deduct credits from creditLedger on each AI call
3. Block AI calls when credits reach 0 (return 402 Payment Required)
4. Stripe metered billing: report usage to Stripe at end of billing period
5. Usage dashboard: show credits used, remaining, and cost breakdown by model
```

---

## Git Commits

### After completing a feature
```
Use the git-commit-assistant skill.
I've finished building: [describe what you built]
Files changed: [list files or let agent detect]
Create conventional commits and push to feature branch.
```

### For GlobalResumeAI
```
Use the git-commit-assistant skill for GlobalResumeAI.
Feature completed: AI resume tailoring with streaming Claude integration.
Changed files: src/app/api/ai-tailor/, src/components/AISuggestions.tsx, prisma/schema.prisma
Write commit(s) in conventional format and suggest PR title.
```

### Quick commit
```
Use the git-commit-assistant skill.
Stage and commit all changes to: src/auth/
Message should reflect: added refresh token rotation
```

### For CAT Prep App
```
Use the git-commit-assistant skill for CAT Prep App.
Completed: timer persistence fix — saves state to DB every 30s.
Changed: src/components/MockTest/Timer.tsx, src/store/testStore.ts, src/app/api/test/checkpoint/route.ts
Suggest conventional commit message.
```

### For Avantika English Coach
```
Use the git-commit-assistant skill for Avantika English Coach.
Completed: structured JSON grammar correction + streaming typing effect.
Changed: src/app/api/grammar-check/route.ts, src/components/Chat/GrammarResult.tsx, src/hooks/useGrammarStream.ts
Create commits per logical change, not one giant commit.
```

### For Spring Boot Microservices
```
Use the git-commit-assistant skill for Spring Boot user-service.
Completed: Flyway migration for audit columns + N+1 fix on order query.
Changed: src/main/resources/db/migration/V5__add_audit_columns.sql, 
         src/main/java/.../repository/OrderRepository.java
Stage migration and query fix as separate commits.
```

### For AI SaaS Platform
```
Use the git-commit-assistant skill for AI SaaS Platform.
Completed: full credit billing system — usage tracking, credit deduction, Stripe metered billing.
Changed: prisma/schema.prisma, src/lib/credits.ts, src/app/api/chat/route.ts, 
         src/app/api/billing/webhook/route.ts, src/components/UsageDashboard.tsx
Create logical commits: schema → credit logic → chat integration → billing webhook → UI.
```

---

## Master Orchestrator (multi-skill workflows)

### Full feature ship
```
Use the master-agent-orchestrator skill.
Task: Ship the AI resume tailoring feature for GlobalResumeAI end-to-end.
Stack: Next.js 15, Prisma, PostgreSQL, Anthropic Claude.
I need: architecture → DB schema → AI integration → security check → commit.
```

### Pre-deploy
```
Use the master-agent-orchestrator skill.
Task: GlobalResumeAI is ready for its first public launch on Vercel.
Run the complete pre-deploy workflow: production audit → security review → deploy config.
```

### CAT Prep App — new test module
```
Use the master-agent-orchestrator skill.
Task: Add a new "VARC" (Verbal Ability & Reading Comprehension) test module to CAT Prep App.
Stack: Next.js 15, Prisma, PostgreSQL.
I need: schema design → API routes → UI (question cards + timer) → AI explanations → commit.
```

### Avantika English Coach — new exercise type
```
Use the master-agent-orchestrator skill.
Task: Add "Fill in the Blank" exercise type to Avantika English Coach.
Claude generates a sentence with one blank, user fills it, Claude evaluates correctness.
Stack: Next.js 15, Anthropic claude-sonnet-4-6.
I need: schema → AI route (structured JSON) → exercise UI → progress tracking → commit.
```

### Spring Boot — new microservice
```
Use the master-agent-orchestrator skill.
Task: Create a new notification-service microservice in the Spring Boot mono-repo.
Handles: email (SendGrid) + push notifications.
Triggered via: async events from order-service and user-service.
I need: architecture design → Spring Boot scaffold → event listener → Docker config → commit.
```

### AI SaaS Platform — launch
```
Use the master-agent-orchestrator skill.
Task: AI SaaS Platform is feature-complete. Run the full production launch workflow.
Stack: Next.js 15, Prisma, PostgreSQL, Anthropic, Stripe, Upstash.
Run: production audit → security review → performance check → deploy config → final commit.
```

### Emergency fix
```
Use the master-agent-orchestrator skill.
Task: URGENT — users can access each other's resumes.
Likely IDOR bug in /api/resume/[id] route.
Run: bug-fixer → security-reviewer → git-commit-assistant.
```
