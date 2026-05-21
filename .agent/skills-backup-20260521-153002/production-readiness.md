# Skill: Production Readiness Auditor

## Purpose
Audit any project end-to-end — code quality, security, performance, config, observability, and deployment hygiene — and produce a prioritized action plan before going live.

## When to Use
- Before deploying to production for the first time
- Before a major release or feature launch
- After a period of rapid development ("we built fast, now let's harden it")
- When onboarding a new app to a CI/CD pipeline

---

## Universal Agent Rules (Follow for Every Task)
1. **Read code first** — read the relevant files before proposing any change. Never assume.
2. **Identify root cause** — for each finding, trace to the actual source in code.
3. **Implement minimal safe fix** — fix what is broken; do not refactor unrelated code.
4. **Run tests / typecheck / build** — validate all findings with evidence (commands run + output).
5. **Explain files changed** — list every modified file with a one-line reason.

---

## Skill-Specific Rules
- Do not change code during audit — only report and recommend.
- Categorize every finding as Critical / High / Medium / Low.
- Be specific: reference file names and line numbers, not vague concerns.
- Do not invent problems. Only flag things that are actually present.
- Recommendations must be actionable, not just theoretical.

---

## Step-by-Step Workflow

### Step 1 — Environment & Config
```
1. Check .env.example vs actual env vars — are secrets committed?
2. Confirm NODE_ENV / SPRING_PROFILES_ACTIVE / PYTHON_ENV is set correctly.
3. Verify no hardcoded credentials, API keys, or connection strings in source.
4. Check CORS config — is it open (*) in production?
5. Review rate limiting configuration.
```

### Step 2 — Security Baseline
```
1. Check authentication middleware is applied on all protected routes.
2. Verify JWT/session expiry is configured.
3. Look for SQL injection, XSS, and input validation gaps.
4. Check dependency vulnerabilities (npm audit / pip-audit / mvn dependency-check).
5. Confirm HTTPS is enforced and HTTP redirects are in place.
```

### Step 3 — Error Handling & Observability
```
1. Confirm unhandled promise rejections / exceptions are caught globally.
2. Check that error responses never leak stack traces to clients.
3. Verify logging is structured (JSON) and goes to stdout.
4. Confirm health check endpoint exists (GET /health or /actuator/health).
5. Check that critical operations have correlation IDs for tracing.
```

### Step 4 — Performance & Scalability
```
1. Look for N+1 queries (ORM relations loaded inside loops).
2. Check database indexes on foreign keys and common filter columns.
3. Confirm pagination is implemented on all list endpoints.
4. Check for unbounded file uploads or memory buffers.
5. Verify caching headers on static assets.
```

### Step 5 — Build & Deployment
```
1. Confirm build output is deterministic (lock files committed).
2. Check Docker image — does it run as non-root? Is .dockerignore present?
3. Verify CI runs lint + tests before deploy.
4. Confirm rollback strategy exists.
5. Check resource limits (memory/CPU) are set in deployment config.
```

### Step 6 — Database & Data
```
1. Confirm migrations are versioned and auto-run on deploy.
2. Check no raw DROP/DELETE without WHERE in migrations.
3. Verify database backups are configured.
4. Confirm sensitive fields (passwords, PII) are never logged.
```

---

## Commands to Run

```bash
npm audit --audit-level=high         # JS vulnerability scan
npx depcheck                         # unused dependencies
pip-audit                            # Python vulnerabilities
./mvnw dependency:check              # Java OWASP check
grep -rn "console.log" src/          # debug logs left in
grep -rn "TODO\|FIXME\|HACK" src/    # tech debt markers
grep -rn "process.env" src/          # env var usage audit
docker scout cves <image>            # container CVE scan
```

---

## Validation Checklist
- [ ] No secrets in source code or git history
- [ ] All critical and high findings addressed
- [ ] Health check endpoint returns 200
- [ ] Dependencies have no critical CVEs
- [ ] Error responses don't expose internals
- [ ] Structured logging in place
- [ ] Database migrations are idempotent
- [ ] CI/CD pipeline runs before deploy
- [ ] Rollback plan documented

---

## Final Response Format

```
## Production Readiness Audit

**Overall Status:** READY / NEEDS WORK / BLOCKED

### Critical (must fix before deploy)
1. [finding] — [file:line] — [recommended fix]

### High (fix within 24h of deploy)
1. [finding] — [file:line] — [recommended fix]

### Medium (fix within sprint)
1. [finding] — [recommended fix]

### Low / Nice to Have
1. [finding] — [recommendation]

**Estimated Fix Time:** [X hours / days]
**Recommended Deploy Date:** [when it'll be ready]
```
