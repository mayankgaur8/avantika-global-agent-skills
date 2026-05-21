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

### Emergency fix
```
Use the master-agent-orchestrator skill.
Task: URGENT — users can access each other's resumes.
Likely IDOR bug in /api/resume/[id] route.
Run: bug-fixer → security-reviewer → git-commit-assistant.
```
