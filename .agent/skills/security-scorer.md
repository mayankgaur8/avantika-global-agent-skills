# Skill: Security Severity Scorer

## Purpose
Apply a quantitative CVSS-inspired scoring model to every security finding, prioritize by exploitability × impact, and produce a risk-ranked remediation plan with effort estimates — so security work can be planned into sprints like any other engineering work.

## When to Use
- After running security-reviewer and needing to prioritize findings
- Security audit has too many findings to fix at once
- Sprint planning — need to decide which security issues go in this sprint
- Stakeholder reporting — need numbers, not just "medium risk"
- Bug bounty triage — need consistent scoring across reports

---

## Universal Agent Rules (Follow for Every Task)
1. **Read code first** — read the vulnerable code before scoring; never score from a description alone.
2. **Identify root cause** — score the root vulnerability, not the symptom.
3. **Implement minimal safe fix** — fix in severity order: Critical → High → Medium → Low.
4. **Run tests / typecheck / build** — validate fixes don't break existing tests.
5. **Explain files changed** — list every file modified for each fix.

---

## Skill-Specific Rules
- Score every finding consistently using the same model.
- A Critical finding (CVSS ≥ 9.0) must be fixed before any deployment.
- Never ship a known Critical or High finding intentionally — document it as a known risk if you must.
- Scoring is context-dependent: a hardcoded secret in a public repo is Critical; in a local dev-only script it's Low.

---

## Scoring Model (CVSS-Inspired, Simplified)

### Base Score Components

**Attack Vector (AV)**
```
Network (N)    = 0.85  — exploitable remotely over internet
Adjacent (A)   = 0.62  — exploitable on same network
Local (L)      = 0.55  — requires local access
Physical (P)   = 0.20  — requires physical device access
```

**Attack Complexity (AC)**
```
Low (L)        = 0.77  — no special conditions required
High (H)       = 0.44  — requires specific conditions
```

**Privileges Required (PR)**
```
None (N)       = 0.85  — no authentication needed
Low (L)        = 0.62  — low-privilege account needed
High (H)       = 0.27  — admin account needed
```

**User Interaction (UI)**
```
None (N)       = 0.85  — no user action needed
Required (R)   = 0.62  — requires victim to click/act
```

**Impact (CIA)**
```
For each of Confidentiality, Integrity, Availability:
High   = 0.56
Low    = 0.22
None   = 0.00
```

### Score Calculation
```
Exploitability = 8.22 × AV × AC × PR × UI
Impact = 1 - ((1 - C) × (1 - I) × (1 - A))

If Impact ≤ 0: Score = 0
If Impact > 0: Score = min(10, 1.08 × (Impact + Exploitability))

Rounded to 1 decimal place.
```

### Score → Severity Mapping
```
9.0–10.0 = CRITICAL  — fix immediately, do not deploy
7.0–8.9  = HIGH      — fix before next production deploy
4.0–6.9  = MEDIUM    — fix within current sprint
0.1–3.9  = LOW       — fix when convenient, track in backlog
0.0      = INFO      — informational, no immediate action
```

---

## Step-by-Step Workflow

### Step 1 — Input Findings
```
For each finding from security-reviewer:
1. Name the vulnerability (e.g., "IDOR on /api/resume/[id]")
2. Set Attack Vector (N/A/L/P)
3. Set Attack Complexity (L/H)
4. Set Privileges Required (N/L/H)
5. Set User Interaction (N/R)
6. Set Confidentiality Impact (H/L/N)
7. Set Integrity Impact (H/L/N)
8. Set Availability Impact (H/L/N)
```

### Step 2 — Calculate Scores
```
Apply the formula above to get a numeric score.
Map to severity band (Critical/High/Medium/Low/Info).
Add contextual multiplier:
- Public-facing production endpoint: ×1.0 (no adjustment)
- Dev/staging only: ×0.5
- Already mitigated by WAF/CDN: ×0.7
```

### Step 3 — Build Risk Register
```
Sort all findings by final score (highest first).
Group into: Critical, High, Medium, Low.
For each group, estimate fix effort in hours.
Calculate risk-adjusted priority: Score / Fix Effort (hours) = ROI.
Fix high-ROI items first.
```

### Step 4 — Generate Sprint Plan
```
Critical: all must be in next sprint (or hotfix immediately)
High: all must be in next sprint
Medium: 50% in current sprint, 50% in next
Low: add to backlog, address in hardening sprints
```

---

## Common Vulnerability Scores (Reference)

| Vulnerability | Score | Severity | Typical Fix Effort |
|---------------|-------|----------|-------------------|
| IDOR — user accesses other user's data | 8.1 | HIGH | 1–2h |
| SQL injection via string concat | 9.8 | CRITICAL | 2–4h |
| Hardcoded API key in source code | 9.1 | CRITICAL | 30m + key rotation |
| Missing auth on admin endpoint | 9.3 | CRITICAL | 1h |
| JWT: `alg: none` accepted | 9.8 | CRITICAL | 30m |
| Stored XSS via user content | 7.5 | HIGH | 1–3h |
| CSRF on state-changing endpoint | 6.5 | MEDIUM | 2h |
| Dependency with known CVE (critical) | 7.2 | HIGH | 30m (upgrade) |
| Open CORS policy on auth API | 6.3 | MEDIUM | 30m |
| Missing rate limit on login | 5.3 | MEDIUM | 2–3h |
| Error response reveals stack trace | 4.3 | MEDIUM | 30m |
| Self-XSS only, no stored vector | 3.0 | LOW | 1h |
| Missing security header (non-CSP) | 2.0 | LOW | 15m |

---

## Project-Specific Risk Profiles

### GlobalResumeAI
```
Highest risk areas: /api/resume/[id] (IDOR), AI API key exposure, PDF generation SSRF
Critical threshold: user A accessing user B's resume = 8.1 (HIGH, near Critical)
Compensating controls: Prisma session check reduces AV from Network to... still Network
```

### CAT Prep App
```
Highest risk areas: test score manipulation (Integrity HIGH), answer exposure (Confidentiality HIGH)
Scoring note: wrong test score = Integrity = HIGH → typically CVSS 7.5+
Fix priority: server-side scoring must be verified before every deploy
```

### Spring Boot Microservices
```
Highest risk areas: inter-service auth, JWT validation, actuator exposure
CRITICAL watch: actuator/env endpoint exposed publicly = credentials exposure = 9.1
```

### AI SaaS Platform
```
Highest risk areas: BYOK key exposure, cross-tenant AI history access, Stripe webhook forgery
BYOK key in browser network tab = Confidentiality HIGH + Network AV = typically 8.5 (HIGH)
```

---

## Commands to Run

```bash
# Get dependency CVE scores
npm audit --json | python3 -c "
import json,sys
data=json.load(sys.stdin)
for v in data.get('vulnerabilities',{}).values():
    print(v.get('severity','?').upper(), v.get('name'), v.get('url',''))
" 2>/dev/null

# Check for critical OWASP patterns
grep -rn "dangerouslySetInnerHTML\|innerHTML\|eval(" src/ | head -10
grep -rn "query.*\+\|execute.*\+" src/ | head -10
```

---

## Validation Checklist
- [ ] Every finding has a numeric CVSS-inspired score
- [ ] All Critical findings (≥ 9.0) have immediate fix plan
- [ ] Risk register sorted by score (not by order found)
- [ ] Fix effort estimated for each finding
- [ ] Sprint allocation made for Critical + High
- [ ] Risk register saved to .agent/memory/

---

## Final Response Format

```
## Security Risk Register

**Total Findings:** [n]
**Critical:** [n] | **High:** [n] | **Medium:** [n] | **Low:** [n]
**Deploy Blocked:** [YES if any Critical or High exists / NO]

| # | Vulnerability | Score | Severity | Fix Effort | Sprint |
|---|---------------|-------|----------|-----------|--------|
| 1 | [vuln] | 9.8 | CRITICAL | 2h | Immediate |
| 2 | [vuln] | 7.5 | HIGH | 1h | This sprint |

**Blocked Deploy Until Fixed:**
- [list Critical + High findings]

**Total Remediation Effort:**
- Critical: [Xh]
- High: [Xh]
- Medium: [Xh]
```
