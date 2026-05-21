# Skill: Master Agent Orchestrator

## Purpose
Analyze any engineering task and automatically route it to the correct skill (or workflow) — in the right order, with the right context. This is the entry point for any task spanning multiple domains, and the dispatch center for the full 26-skill AI Engineering Platform.

## When to Use
- You have a task but aren't sure which skill applies
- The task spans multiple areas ("make this app production-ready")
- You want to run a full multi-skill workflow automatically
- Starting a new feature, fixing an incident, or launching a release

---

## Universal Agent Rules (Apply Before Every Skill)

1. **Read code first** — never assume. Read the relevant files before any change.
2. **Identify root cause** — trace errors and findings to their actual source.
3. **Implement minimal safe fix** — smallest change that solves the problem.
4. **Run tests / typecheck / build** — validate before reporting complete.
5. **Explain files changed** — list every modified file with a one-line reason.

---

## Routing Decision Tree

```
What is your task?
│
├── [MEMORY/CONTEXT]
│   ├── "start a session" / "load context"         → session-memory
│   ├── "what's the project stack?"                → auto-detector
│   └── "plan this task"                           → task-planner
│
├── [BUGS & INCIDENTS]
│   ├── One bug, known error                       → bug-fixer
│   ├── Multiple bugs / production incident        → bug-triage → bug-fixer
│   └── P0 security bug                            → bug-triage → security-reviewer
│
├── [ARCHITECTURE & DESIGN]
│   ├── Design a new feature from scratch          → fullstack-architect
│   ├── Review existing architecture               → architecture-reviewer
│   └── Plan a complex task                        → task-planner
│
├── [DATABASE]
│   └── Schema / query / migration                 → database-prisma-sql-expert
│
├── [FRONTEND & UI]
│   └── Visual / accessibility / responsive        → ui-ux-polisher
│
├── [AI FEATURES]
│   └── LLM, streaming, rate limits, billing       → ai-saas-builder
│
├── [SECURITY]
│   ├── Find security issues                       → security-reviewer
│   ├── Prioritize security findings               → security-scorer
│   └── Pre-launch security gate                   → security-reviewer → security-scorer
│
├── [QUALITY]
│   ├── Score code quality                         → code-quality-scorer
│   ├── Review a PR                                → pr-reviewer
│   └── Add tests                                  → test-generator
│
├── [PERFORMANCE]
│   └── Slow page / API / query                    → performance-optimizer
│
├── [DEPLOY & INFRA]
│   ├── Deploy to Vercel/Azure/AWS                 → deployment-engineer
│   ├── Audit CI/CD pipeline                       → cicd-auditor
│   ├── Add logging/tracing/metrics                → observability-engineer
│   └── Reduce cloud costs                         → cost-optimizer
│
├── [RELEASE]
│   ├── Generate changelog                         → changelog-generator
│   ├── Full release workflow                      → release-manager
│   └── Write/update documentation                → docs-generator
│
├── [GIT]
│   └── Commit / stage / message                   → git-commit-assistant
│
└── [MULTI-SKILL WORKFLOWS]
    ├── "launch" / "go live"                       → agent-collaboration WORKFLOW-01
    ├── "close the sprint"                         → agent-collaboration WORKFLOW-02
    ├── "ship [feature] from scratch"              → agent-collaboration WORKFLOW-03
    ├── "production is broken" / incident          → agent-collaboration WORKFLOW-04
    ├── "monthly health check"                     → agent-collaboration WORKFLOW-05
    ├── "harden before deploy"                     → agent-collaboration WORKFLOW-06
    └── "add AI feature end to end"                → agent-collaboration WORKFLOW-07
```

---

## Pre-Built Workflow Shortcuts

Say any of these and the orchestrator routes automatically:

| Trigger Phrase | Workflow | Skills Involved |
|----------------|----------|-----------------|
| "ready to go live" | WORKFLOW-01: Full Launch | 11 skills |
| "close the sprint" | WORKFLOW-02: Sprint Close | 6 skills |
| "ship [feature] from scratch" | WORKFLOW-03: Feature Ship | 11 skills |
| "production is broken" | WORKFLOW-04: Incident | 8 skills |
| "monthly review" | WORKFLOW-05: Health Check | 6 skills |
| "harden before deploy" | WORKFLOW-06: Pre-Deploy | 5 skills |
| "add AI feature end to end" | WORKFLOW-07: AI Feature | 9 skills |

---

## Composite Workflows (Manual)

### "Ship a new feature end-to-end"
```
1. task-planner              → phased plan
2. fullstack-architect       → data model + API contract
3. database-prisma-sql-expert → schema + migration
4. [implement]
5. test-generator            → unit + integration tests
6. ui-ux-polisher            → polish UI components
7. security-reviewer         → check new endpoints
8. pr-reviewer               → self-review before PR
9. git-commit-assistant      → clean conventional commits
```

### "Harden and deploy"
```
1. code-quality-scorer       → baseline quality report
2. production-readiness      → full pre-launch audit
3. security-reviewer         → fix Critical + High findings
4. security-scorer           → risk-rank all findings
5. performance-optimizer     → fix top 3 bottlenecks
6. cicd-auditor              → verify pipeline
7. deployment-engineer       → configure + deploy
8. changelog-generator       → release notes
9. release-manager           → tag + GitHub release
```

### "Fix a production bug"
```
1. bug-triage                → classify severity
2. bug-fixer                 → root cause + fix
3. test-generator            → add regression test
4. git-commit-assistant      → hotfix commit
```

### "Add AI feature"
```
1. task-planner              → plan the AI feature
2. ai-saas-builder           → LLM + streaming + rate limits
3. database-prisma-sql-expert → usage tracking schema
4. cost-optimizer            → model routing + prompt caching
5. security-reviewer         → API key safety, prompt injection
6. test-generator            → structured output validation tests
7. observability-engineer    → AI metrics (tokens, latency, errors)
8. git-commit-assistant      → commit
```

### "Monthly health check"
```
1. code-quality-scorer       → quality score trend
2. security-reviewer         → dependency CVE scan
3. performance-optimizer     → current p99 latencies
4. cost-optimizer            → cloud cost review
5. architecture-reviewer     → new anti-patterns
6. session-memory            → save monthly report
```

---

## Skill → Category Map

| Skill | Category |
|-------|----------|
| session-memory | Intelligence |
| auto-detector | Intelligence |
| task-planner | Intelligence |
| bug-triage | Intelligence |
| bug-fixer | Core |
| production-readiness | Core |
| ui-ux-polisher | Core |
| fullstack-architect | Core |
| deployment-engineer | Core |
| security-reviewer | Core |
| performance-optimizer | Core |
| database-prisma-sql-expert | Core |
| ai-saas-builder | Core |
| git-commit-assistant | Core |
| architecture-reviewer | Core |
| code-quality-scorer | Quality Gate |
| security-scorer | Quality Gate |
| pr-reviewer | Quality Gate |
| test-generator | Quality Gate |
| changelog-generator | Release |
| release-manager | Release |
| cicd-auditor | Release |
| docs-generator | Release |
| observability-engineer | Infrastructure |
| cost-optimizer | Infrastructure |
| agent-collaboration | Orchestration |

---

## Task Classification Examples

| Task | Skill(s) |
|------|----------|
| "Fix: TypeError at line 42" | bug-fixer |
| "5 bugs in QA — prioritize" | bug-triage → bug-fixer |
| "Audit before Vercel deploy" | production-readiness → deployment-engineer |
| "Dashboard slow on mobile" | performance-optimizer + ui-ux-polisher |
| "Add JWT refresh token" | security-reviewer → bug-fixer |
| "Design multi-tenant schema" | fullstack-architect → database-prisma-sql-expert |
| "Add Claude streaming chat" | ai-saas-builder → cost-optimizer |
| "What quality score is this code?" | code-quality-scorer |
| "Review my PR before merge" | pr-reviewer |
| "How much does our AI cost?" | cost-optimizer |
| "Set up structured logging" | observability-engineer |
| "Write tests for this service" | test-generator |
| "Generate CHANGELOG since v1.2" | changelog-generator |
| "Ship v2.0" | release-manager (orchestrates all) |
| "Audit our GitHub Actions" | cicd-auditor |
| "Write API docs" | docs-generator |
| "Score security findings" | security-scorer |
| "Plan this 2-week feature" | task-planner |
| "Save project context" | session-memory |
| "What stack is this project?" | auto-detector |

---

## How to Invoke

### Single skill
```
Use the bug-fixer skill to fix: [paste error]
```

### Composite workflow (named)
```
Use the master-agent-orchestrator skill.
Task: GlobalResumeAI is ready to go live.
Run WORKFLOW-01 (Full Launch Pipeline).
Stack: Next.js 15, Prisma, PostgreSQL, Anthropic, Vercel.
```

### Composite workflow (described)
```
Use the master-agent-orchestrator skill.
Task: Ship the AI grammar correction feature for Avantika English Coach end to end.
Stack: Next.js 15, Anthropic claude-sonnet-4-6, Prisma.
Start from design and finish with a clean commit.
```

### First session on a new project
```
Use the master-agent-orchestrator skill.
Action: Initialize context for this project.
Run: auto-detector → session-memory → task-planner.
Goal: understand the stack, load any existing context, plan next 3 tasks.
```

---

## Output Format

```
## Orchestration Plan

**Task:** [what you described]
**Complexity:** Simple / Multi-step / Full Workflow
**Estimated Duration:** [X hours]

**Skills to Run (in order):**
1. [skill-name] — [why + what it will do]
2. [skill-name] — [why + what it will do]

**Stop Conditions:**
- [condition that pauses the workflow for human input]

**Starting now with:** [first skill]

---
[execution begins]
```
