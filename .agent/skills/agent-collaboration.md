# Skill: Agent Collaboration Workflow

## Purpose
Define and execute multi-skill workflows where multiple skills run in sequence automatically, with each skill's output feeding the next — enabling complex engineering work to complete without manual intervention between steps.

## When to Use
- Complex tasks spanning 3+ engineering domains
- Repeatable workflows (every sprint: quality score → changelog → release)
- Full pre-launch pipelines (audit → security → perf → deploy)
- Scheduled automated tasks (weekly quality reports, monthly cost reviews)
- "Run the full X workflow" requests

---

## Universal Agent Rules (Follow for Every Task)
1. **Read code first** — read the current state of the project before starting any workflow.
2. **Identify root cause** — if a step in the workflow fails, stop and diagnose before continuing.
3. **Implement minimal safe fix** — each skill in the chain does its job and only its job.
4. **Run tests / typecheck / build** — validate between workflow stages, not just at the end.
5. **Explain files changed** — at the end of each workflow, provide a combined list of all changes.

---

## Skill-Specific Rules
- Workflows execute skills in the defined order. Do not skip steps.
- If any step produces a BLOCKING finding (Critical bug, P0 security issue), STOP and report before continuing.
- Each skill in a workflow receives the output context of the previous skill.
- Workflows are defined here; execution happens step by step.
- If a step fails or is blocked, output the partial results before stopping.

---

## Pre-Built Workflows

### WORKFLOW-01: Full Launch Pipeline
**Trigger:** "We're ready to go live"
**Duration:** 2–4 hours
```
Step 1: session-memory       → load project context
Step 2: code-quality-scorer  → baseline quality report
Step 3: production-readiness → identify all pre-launch blockers
Step 4: security-reviewer    → find security issues
Step 5: security-scorer      → prioritize security findings
Step 6: performance-optimizer → fix top 3 performance issues
Step 7: cicd-auditor         → verify pipeline is production-ready
Step 8: deployment-engineer  → generate deploy config
Step 9: changelog-generator  → create launch changelog
Step 10: release-manager     → tag v1.0.0 and create GitHub release
Step 11: session-memory      → save post-launch state

STOP conditions:
- code-quality-scorer: Overall score < 5.0
- production-readiness: Any Critical finding
- security-scorer: Any CVSS ≥ 9.0 finding
- cicd-auditor: Secrets in pipeline
```

### WORKFLOW-02: Sprint Close
**Trigger:** "Close out this sprint"
**Duration:** 30–60 minutes
```
Step 1: bug-triage           → triage any open issues
Step 2: code-quality-scorer  → quality delta since last sprint
Step 3: git-commit-assistant → clean up any uncommitted work
Step 4: changelog-generator  → generate sprint changelog
Step 5: pr-reviewer          → review any open PRs
Step 6: session-memory       → save sprint completion state

STOP conditions:
- bug-triage: Any P0 open bugs → fix before closing sprint
```

### WORKFLOW-03: New Feature Ship
**Trigger:** "Ship [feature name] from scratch"
**Duration:** varies (planning + implementation)
```
Step 1: session-memory       → load current architecture context
Step 2: task-planner         → create phased implementation plan
Step 3: fullstack-architect  → design data model + API contract
Step 4: database-prisma-sql-expert → write schema + migration
Step 5: [IMPLEMENTATION PAUSE — human implements or agent codes]
Step 6: test-generator       → generate unit + integration tests
Step 7: security-reviewer    → audit new endpoints
Step 8: ui-ux-polisher       → polish any new UI
Step 9: pr-reviewer          → self-review before opening PR
Step 10: git-commit-assistant → structured commits
Step 11: session-memory      → save feature completion state
```

### WORKFLOW-04: Incident Response
**Trigger:** "Production is broken" / P0 incident
**Duration:** 15–60 minutes
```
Step 1: bug-triage           → classify and prioritize
Step 2: bug-fixer            → diagnose and fix root cause
Step 3: security-reviewer    → if SECURITY category: check for breach
Step 4: test-generator       → add regression test for this bug
Step 5: git-commit-assistant → hotfix commit
Step 6: deployment-engineer  → hotfix deploy
Step 7: session-memory       → log incident in known-issues.md
Step 8: changelog-generator  → generate hotfix release notes

STOP conditions:
- bug-triage P0 SECURITY: escalate to security-scorer immediately
```

### WORKFLOW-05: Monthly Health Check
**Trigger:** "Run the monthly review"
**Duration:** 1–2 hours
```
Step 1: code-quality-scorer  → quality score vs last month
Step 2: security-reviewer    → dependency CVE scan
Step 3: performance-optimizer → measure current p99 latencies
Step 4: cost-optimizer       → cloud cost review
Step 5: architecture-reviewer → any new anti-patterns introduced
Step 6: session-memory       → save monthly metrics

Output: Monthly Health Report with trends
```

### WORKFLOW-06: Pre-Deploy Hardening
**Trigger:** "Harden this app before next deploy"
**Duration:** 1–3 hours
```
Step 1: production-readiness → full audit
Step 2: security-scorer      → risk register
Step 3: cicd-auditor         → pipeline check
Step 4: observability-engineer → ensure logging/metrics in place
Step 5: deployment-engineer  → final deploy config
```

### WORKFLOW-07: AI Feature Pipeline
**Trigger:** "Add [AI feature] end to end"
**Duration:** 3–5 hours
```
Step 1: task-planner         → plan the AI feature implementation
Step 2: ai-saas-builder      → implement LLM integration + streaming
Step 3: database-prisma-sql-expert → usage tracking schema
Step 4: security-reviewer    → API key safety, prompt injection
Step 5: performance-optimizer → streaming latency, cold-start
Step 6: cost-optimizer       → token cost estimate + model routing
Step 7: test-generator       → AI feature tests (structured output validation)
Step 8: observability-engineer → AI metrics (tokens, latency, errors)
Step 9: git-commit-assistant → commit all AI feature work
```

---

## Workflow Output Format

Each workflow produces a Master Report:

```markdown
# Workflow Report: [WORKFLOW NAME]
**Date:** [date]
**Project:** [project]
**Duration:** [actual time]
**Status:** COMPLETE / BLOCKED / PARTIAL

## Step Results
| Step | Skill | Status | Key Finding |
|------|-------|--------|-------------|
| 1 | session-memory | ✅ | Context loaded: 3 ADRs, 2 known issues |
| 2 | code-quality-scorer | ✅ | Score: 7.2/10 (B) |
| 3 | production-readiness | ⚠️ | 2 High findings, 3 Medium |
| 4 | security-scorer | ❌ BLOCKED | CVSS 9.1 finding — fix required |

## Blocking Issues (must fix before continuing)
1. [issue] — [skill that found it] — [recommended fix]

## All Files Changed
- [file] — [skill] — [change]

## Next Recommended Action
[single sentence — what to do right now]
```

---

## How to Invoke a Workflow

```
Use the agent-collaboration skill.
Workflow: WORKFLOW-01 (Full Launch Pipeline)
Project: GlobalResumeAI
Deploy target: Vercel
Stack: Next.js 15, Prisma, PostgreSQL, Anthropic
```

```
Use the agent-collaboration skill.
Workflow: WORKFLOW-04 (Incident Response)
Issue: Users reporting 500 errors on resume save since 2:30pm
Suspected file: src/app/api/resume/save/route.ts
```

```
Use the agent-collaboration skill.
Workflow: WORKFLOW-07 (AI Feature Pipeline)
Feature: AI-powered grammar correction for Avantika English Coach
Model: claude-sonnet-4-6 (streaming + structured JSON output)
```

---

## Workflow Registry

| Workflow | Trigger Phrase | Steps | Duration |
|----------|---------------|-------|----------|
| WORKFLOW-01 | "ready to go live" / "launch" | 11 | 2–4h |
| WORKFLOW-02 | "close the sprint" / "sprint review" | 6 | 30–60m |
| WORKFLOW-03 | "ship [feature] from scratch" | 11 | varies |
| WORKFLOW-04 | "production is broken" / "incident" | 8 | 15–60m |
| WORKFLOW-05 | "monthly review" / "health check" | 6 | 1–2h |
| WORKFLOW-06 | "harden before deploy" | 5 | 1–3h |
| WORKFLOW-07 | "add AI feature end to end" | 9 | 3–5h |

---

## Validation Checklist
- [ ] Workflow selected from registry (or custom workflow defined)
- [ ] Project context loaded (session-memory)
- [ ] Stop conditions checked at each step
- [ ] Blocking issues reported before moving to next step
- [ ] Master Report generated at end
- [ ] All file changes listed
- [ ] session-memory updated with workflow results

---

## Final Response Format

```
## Agent Collaboration: [Workflow Name]

**Workflow:** [ID and name]
**Steps:** [n completed] / [total]
**Overall Status:** COMPLETE / BLOCKED at Step [n]

[Master Report as above]

**Total Changes:**
- Files Modified: [n]
- Issues Fixed: [n]
- Tests Added: [n]

**Recommended Next Action:**
[one sentence]
```
