# Skill: Security Reviewer

## Purpose
Perform a comprehensive security review of any application — covering OWASP Top 10, authentication, authorization, secrets management, input validation, and dependency vulnerabilities — and produce a prioritized remediation plan.

## When to Use
- Before any public launch or major release
- After adding new authentication/authorization code
- When handling user data, payments, or PII
- After a suspected or confirmed security incident
- Quarterly security review of a live app

---

## Universal Agent Rules (Follow for Every Task)
1. **Read code first** — read the actual auth, route, and validation code before reporting findings.
2. **Identify root cause** — show the exact vulnerable line, not just the category of vulnerability.
3. **Implement minimal safe fix** — provide a concrete fix for each finding. No vague recommendations.
4. **Run tests / typecheck / build** — run npm audit / pip-audit and confirm no new errors after fixes.
5. **Explain files changed** — list every modified file with a one-line explanation of the fix.

---

## Skill-Specific Rules
- Report every finding regardless of perceived severity — let the team triage.
- Never suppress findings because they "probably won't be exploited."
- Be specific: show the vulnerable code, not just a category name.
- Prioritize by exploitability and impact, not just CVSS score.
- Provide a concrete fix for every finding, not just a description of the problem.

---

## Step-by-Step Workflow

### Step 1 — Secrets & Configuration
```
1. Scan source code and git history for hardcoded secrets.
2. Verify .env files are in .gitignore.
3. Check that API keys / DB credentials are never logged.
4. Confirm secret rotation policy exists for third-party keys.
5. Verify different credentials are used for dev/staging/prod.
```

### Step 2 — Authentication
```
1. Check password hashing: must use bcrypt (cost ≥ 12), Argon2, or scrypt.
   Red flags: MD5, SHA1, SHA256 without salt for passwords.
2. Verify JWT: check signature algorithm (reject 'none'), check expiry.
3. Confirm refresh token rotation on use (prevent replay attacks).
4. Check rate limiting on login endpoint (prevent brute force).
5. Verify account lockout or CAPTCHA after N failed attempts.
6. Check password reset flow: tokens must be single-use and time-limited.
7. Verify MFA option exists for sensitive apps.
```

### Step 3 — Authorization
```
1. Check every endpoint: is auth middleware applied?
2. Verify object-level authorization (IDOR): can user A access user B's resource?
   Example: GET /api/orders/123 — does it check orders.userId === req.user.id?
3. Check role-based access: admin-only routes protected at both frontend AND backend.
4. Verify file downloads: generated URLs must not expose other users' files.
5. Check mass assignment: do API endpoints accept fields they shouldn't?
```

### Step 4 — Input Validation & Injection
```
1. Check every user input is validated server-side (not just frontend).
2. SQL injection: use parameterized queries or ORM exclusively. No string concatenation.
3. NoSQL injection: sanitize inputs before MongoDB/DynamoDB queries.
4. XSS: check that user-supplied content is escaped before rendering.
   In React: dangerouslySetInnerHTML is a red flag — audit every usage.
5. Command injection: never pass user input to shell commands.
6. Path traversal: never use user input in file system paths without sanitization.
7. SSRF: validate and allowlist URLs before making server-side HTTP requests.
```

### Step 5 — API Security
```
1. Check CORS: origin whitelist should NOT be *, especially for authenticated APIs.
2. Check CSRF protection on state-changing endpoints (SameSite cookie or CSRF token).
3. Verify rate limiting on all public endpoints.
4. Confirm sensitive operations require re-authentication (password change, delete account).
5. Check that error responses do not expose stack traces, DB errors, or internal paths.
6. Verify HTTP security headers: CSP, HSTS, X-Frame-Options, X-Content-Type-Options.
```

### Step 6 — Dependency Audit
```
1. Run vulnerability scanner on all dependencies.
2. Flag any dependency with a critical or high CVE.
3. Check for abandoned packages (last commit > 2 years, no maintainer response).
4. Verify no dev dependencies are included in the production bundle.
```

### Step 7 — Data Security
```
1. Confirm PII is encrypted at rest (DB-level or field-level encryption).
2. Verify sensitive data is not stored in browser localStorage (prefer httpOnly cookies).
3. Check that passwords/tokens are never returned in API responses.
4. Confirm logs do not contain PII, passwords, or tokens.
5. Verify data retention policy is implemented for old records.
```

---

## Commands to Run

```bash
# Secrets in git history
git log --all --full-history -p | grep -iE "(password|secret|api_key|token).*=.*['\"]"

# Hardcoded secrets in source
grep -rn --include="*.ts" --include="*.js" --include="*.py" \
  -E "(password|secret|api_key|token)\s*=\s*['\"][^'\"]{8,}" src/

# Dependency vulnerabilities
npm audit --audit-level=moderate
pip-audit
./mvnw dependency:check

# Check for dangerouslySetInnerHTML (XSS risk)
grep -rn "dangerouslySetInnerHTML" src/

# Check for raw SQL string concatenation
grep -rn "query.*\+.*req\." src/

# HTTP security headers check
curl -I https://yourdomain.com | grep -iE "(strict-transport|content-security|x-frame|x-content-type)"

# Check CORS config
grep -rn "cors\|Access-Control-Allow-Origin" src/
```

---

## Validation Checklist
- [ ] No secrets in source code or git history
- [ ] Passwords hashed with bcrypt/Argon2 (not MD5/SHA1)
- [ ] JWT signature algorithm is RS256 or HS256 (not 'none')
- [ ] Every protected endpoint has auth middleware
- [ ] Object-level auth checked (IDOR prevention)
- [ ] All inputs validated server-side
- [ ] No raw SQL string concatenation
- [ ] dangerouslySetInnerHTML usage reviewed
- [ ] CORS is not open (*) for authenticated APIs
- [ ] Rate limiting on login and sensitive endpoints
- [ ] No critical CVEs in dependencies
- [ ] Error responses don't leak stack traces
- [ ] HTTP security headers present
- [ ] PII not in logs or localStorage

---

## Final Response Format

```
## Security Review Report

**Date:** [date]
**Scope:** [what was reviewed]
**Overall Risk Level:** CRITICAL / HIGH / MEDIUM / LOW

### Critical Findings (fix immediately)
| # | Vulnerability | Location | Impact | Fix |
|---|---------------|----------|--------|-----|
| 1 | [type] | [file:line] | [impact] | [fix] |

### High Findings (fix before next deploy)
[same table format]

### Medium Findings (fix within sprint)
[same table format]

### Low / Informational
[brief list]

**Dependencies with CVEs:**
- [package@version] — CVE-XXXX-XXXX — [severity] — upgrade to [version]

**Remediation Priority Order:**
1. [most critical fix first]
```
