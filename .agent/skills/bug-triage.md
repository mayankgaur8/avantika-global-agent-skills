# Skill: Bug Triage

## Purpose
Systematically classify, prioritize, and route incoming bugs across a project — determining severity, affected users, root cause category, and which skill(s) to invoke — before any fix is attempted.

## When to Use
- Multiple bugs reported at once (triage queue)
- A production incident with unclear scope
- QA returns a list of issues before a release
- You want to prioritize what to fix first
- An error monitoring alert fires (Sentry, Datadog, etc.)

---

## Universal Agent Rules (Follow for Every Task)
1. **Read code first** — read the error logs, stack traces, and relevant code before classifying.
2. **Identify root cause** — classify by ROOT CAUSE, not by symptom.
3. **Implement minimal safe fix** — triage only; do not fix during triage unless it's a one-liner.
4. **Run tests / typecheck / build** — if reproducing a bug, run in isolation first.
5. **Explain files changed** — list every file where the bug was traced.

---

## Skill-Specific Rules
- One triage entry per bug. Do not merge separate bugs.
- Severity must be assigned before routing. Never skip severity.
- A Critical bug (data loss, security, auth bypass) must be escalated immediately — do not batch with other bugs.
- Triage is read-only — do not start fixing until the triage report is complete.
- Every bug needs a reproduction path before it can be triaged as confirmed.

---

## Severity Scale

| Severity | Definition | Response Time | Example |
|----------|-----------|---------------|---------|
| **P0 — Critical** | Data loss, security breach, auth bypass, full outage | Fix in < 2 hours | Users can access each other's data |
| **P1 — High** | Major feature broken for all users, no workaround | Fix in < 24 hours | PDF export returns blank file |
| **P2 — Medium** | Feature broken for some users or with workaround | Fix in next sprint | Timer resets only on Firefox |
| **P3 — Low** | Minor visual issue, non-blocking, rare edge case | Fix when convenient | Button label truncates at 320px |
| **P4 — Enhancement** | Not a bug, improvement request | Add to backlog | "Would be nice if..." |

---

## Root Cause Categories

```
AUTH       — authentication/authorization failure
DATA       — data corruption, wrong calculation, bad state
INFRA      — environment, config, deployment, network
PERF       — performance, timeout, memory leak
UI         — visual rendering, layout, accessibility
ASYNC      — race condition, missing await, event ordering
SCHEMA     — wrong data type, missing field, migration
EXTERNAL   — third-party API failure, webhook, payment
AI         — LLM non-deterministic output, prompt issue
SECURITY   — vulnerability, injection, exposure
```

---

## Step-by-Step Workflow

### Step 1 — Collect
```
1. Gather all bug reports: error logs, user reports, monitoring alerts.
2. For each bug: capture title, error message, stack trace, affected user count.
3. Attempt to reproduce each bug locally (if possible).
4. Note: browser, device, user role, data state at time of failure.
```

### Step 2 — Classify
```
For each bug:
1. Assign severity (P0–P4) based on: user impact × data risk × workaround availability.
2. Assign root cause category (AUTH/DATA/INFRA/PERF/UI/ASYNC/SCHEMA/EXTERNAL/AI/SECURITY).
3. Map to affected component: [service] / [page] / [API route] / [DB table].
4. Identify which skill handles the fix (bug-fixer / security-reviewer / performance-optimizer / etc.).
```

### Step 3 — Prioritize
```
Sort triage list:
1. P0 first — fix immediately, notify team
2. P1 second — fix before next deploy
3. P2 in current sprint
4. P3/P4 in backlog

Within the same severity, prioritize by:
- Number of affected users (higher = more urgent)
- Data risk (data loss > data display error)
- Reversibility (harder to undo = fix sooner)
```

### Step 4 — Route
```
For each confirmed bug, assign to the correct skill:

P0 SECURITY bugs      → security-reviewer immediately
P0 DATA bugs          → database-prisma-sql-expert + bug-fixer
P1 PERF bugs          → performance-optimizer
P1+ AUTH bugs         → security-reviewer
UI bugs (any P)       → bug-fixer → ui-ux-polisher
SCHEMA bugs           → database-prisma-sql-expert
AI bugs               → ai-saas-builder
General logic bugs    → bug-fixer
```

### Step 5 — Document
```
Save triage to .agent/memory/known-issues.md with:
- Bug ID (BUG-001, BUG-002...)
- Severity + category
- Assigned skill
- Reproduction steps
- Current status
```

---

## Triage Report Template

```markdown
# Bug Triage Report
**Date:** [date]
**Project:** [project]
**Total Bugs:** [n]
**P0 Count:** [n] — [immediate action if > 0]

## P0 — Critical (Fix Immediately)
| ID | Title | Error | Root Cause | File | Action |
|----|-------|-------|------------|------|--------|
| BUG-001 | [title] | [error snippet] | SECURITY | [file:line] | security-reviewer NOW |

## P1 — High (Fix Before Next Deploy)
| ID | Title | Error | Root Cause | File | Skill | Users Affected |
|----|-------|-------|------------|------|-------|----------------|

## P2 — Medium (Current Sprint)
[table]

## P3/P4 — Low / Backlog
[table]

## Won't Fix
| ID | Title | Reason |
|----|-------|--------|

## Recommended Fix Order
1. BUG-001 — [why first]
2. BUG-003 — [why second]
```

---

## Project-Specific Triage Patterns

### GlobalResumeAI
```
P0 triggers: resume data visible to wrong user, AI API key in browser response
P1 triggers: PDF export fails, resume save fails, AI tailoring returns error for all users
Common P2: formatting breaks on specific template, slow AI response on large resumes
Route AI bugs → ai-saas-builder; data bugs → database-prisma-sql-expert
```

### CAT Prep App
```
P0 triggers: test score calculation wrong, user can access another's test answers
P1 triggers: timer stops completely, test submission fails, results page crashes
Common P2: timer visual glitch on specific browser, question image doesn't load
Route score bugs → database-prisma-sql-expert (server-side calculation)
```

### Spring Boot Microservices
```
P0 triggers: JWT not validated, unauthorized endpoint access, data leaked across tenants
P1 triggers: service returns 500 for valid requests, DB connection pool exhausted
Common P2: slow endpoint for large datasets, Hibernate N+1 on specific query
Route P0 → security-reviewer immediately; P1 infra → deployment-engineer
```

---

## Commands to Run

```bash
# Pull recent errors from logs (Next.js / Node)
grep -E "ERROR|FATAL|500" .next/server/*.log | tail -50

# Pull errors from git log commit messages
git log --oneline --since="1 week ago" | grep -i "fix\|bug\|error\|crash"

# Check Sentry or monitoring (if configured)
# open https://sentry.io/organizations/[org]/issues/

# Count errors by type in log file
grep -oP "(?<=Error: ).*" server.log | sort | uniq -c | sort -rn | head -20
```

---

## Validation Checklist
- [ ] Every bug has a severity (P0–P4)
- [ ] Every bug has a root cause category
- [ ] Every P0 bug has an immediate action assigned
- [ ] Triage saved to .agent/memory/known-issues.md
- [ ] No P0 bugs left without assignment
- [ ] Fix order is clear and documented
- [ ] Won't Fix list has explicit reasons

---

## Final Response Format

```
## Bug Triage Report

**Total:** [n bugs] — P0: [n] | P1: [n] | P2: [n] | P3/P4: [n]

### Immediate Actions Required
[any P0 bugs with explicit next step]

### Triage Summary
| ID | Title | Severity | Category | Skill | ETA |
|----|-------|----------|----------|-------|-----|
| BUG-001 | [title] | P0 | SECURITY | security-reviewer | 2h |

### Fix Order
1. [BUG-ID]: [reason for priority]

### Saved To
.agent/memory/known-issues.md
```
