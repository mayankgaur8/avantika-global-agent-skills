# Skill: Master Agent Orchestrator

## Purpose
Analyze any engineering task and automatically route it to the correct skill (or combination of skills), in the right order, with the right context. This is the entry point when you're unsure which skill to use, or when a task spans multiple domains.

## When to Use
- You have a task but aren't sure which skill applies
- The task is large and spans multiple areas (e.g., "make this app production-ready")
- You want the agent to self-direct across a full workflow
- Starting a new feature from scratch end-to-end

---

## Agent Rules (Apply to Every Skill)
Before executing any skill workflow, the agent MUST:

1. **Read code first** — never assume. Read the relevant files before proposing any change.
2. **Identify root cause** — for bugs/issues, trace to the actual source, not the symptom.
3. **Implement minimal safe fix** — smallest change that solves the problem. No refactoring scope creep.
4. **Run tests / typecheck / build** — validate the change before reporting completion.
5. **Explain files changed** — always list every file modified with a one-line reason.

---

## Routing Decision Tree

```
What is your task?
│
├── Something is broken / error in console or logs
│   └── → USE: bug-fixer
│
├── App needs to go live / pre-deploy review
│   └── → USE: production-readiness → security-reviewer → deployment-engineer
│
├── UI looks rough / accessibility issues / mobile layout broken
│   └── → USE: ui-ux-polisher
│
├── Designing a new feature or service from scratch
│   └── → USE: fullstack-architect → (implement) → git-commit-assistant
│
├── Deploying to Vercel / Railway / Azure / Docker
│   └── → USE: deployment-engineer
│
├── Security concerns / auth review / vulnerability scan
│   └── → USE: security-reviewer
│
├── App is slow / high latency / large bundle
│   └── → USE: performance-optimizer
│
├── Database schema change / migration / slow query
│   └── → USE: database-prisma-sql-expert
│
├── Adding AI features / LLM integration / streaming
│   └── → USE: ai-saas-builder
│
└── Committing code / writing commit messages
    └── → USE: git-commit-assistant
```

---

## Composite Workflows

### "Ship a new feature end-to-end"
```
1. fullstack-architect    → design the data model + API contract
2. database-prisma-sql-expert → write the schema + migration
3. (implement feature)
4. ui-ux-polisher         → polish the UI components
5. security-reviewer      → check the new endpoints
6. git-commit-assistant   → commit cleanly with conventional message
```

### "Harden and deploy an existing app"
```
1. production-readiness   → full audit, get the findings list
2. security-reviewer      → fix Critical + High security findings
3. performance-optimizer  → fix the top 3 perf bottlenecks
4. deployment-engineer    → configure and deploy
5. git-commit-assistant   → final clean commit
```

### "Diagnose and fix a reported bug"
```
1. bug-fixer              → diagnose root cause and apply fix
2. (if fix touches auth)  → security-reviewer on the changed files
3. git-commit-assistant   → commit with conventional message
```

### "Start an AI SaaS from scratch"
```
1. fullstack-architect    → define architecture and tech stack
2. database-prisma-sql-expert → design user + subscription + usage schema
3. ai-saas-builder        → implement LLM integration + streaming + rate limits
4. security-reviewer      → audit before first deploy
5. deployment-engineer    → deploy to Vercel + configure env vars
6. git-commit-assistant   → structured commit history
```

### "Performance crisis — app is slow"
```
1. performance-optimizer  → measure first, identify top bottlenecks
2. database-prisma-sql-expert → fix N+1 queries and add indexes
3. production-readiness   → confirm nothing else is broken
4. git-commit-assistant   → commit perf fixes
```

---

## Skill Dependency Map

```
fullstack-architect
    └── database-prisma-sql-expert (schema + queries)
    └── ai-saas-builder (if AI features)
    └── security-reviewer (API auth boundaries)

production-readiness
    └── security-reviewer (security section)
    └── performance-optimizer (perf section)
    └── deployment-engineer (deploy config section)

deployment-engineer
    └── git-commit-assistant (final commit before push)
```

---

## How to Invoke

### Single skill
```
Use the bug-fixer skill to fix this error: [paste error]
```

### Composite workflow
```
Use the master-agent-orchestrator skill. My task is:
"This app is ready for its first production deploy to Vercel.
Run a full pre-deploy workflow."
```

### With project context
```
Use the master-agent-orchestrator skill for GlobalResumeAI.
The task is: add AI-powered job description tailoring to the resume builder.
Tech stack: Next.js 15, Prisma, PostgreSQL, Anthropic Claude.
```

---

## Task Classification Examples

| Task | Skill(s) |
|------|----------|
| "Fix: TypeError at line 42" | bug-fixer |
| "Audit before Vercel deploy" | production-readiness → deployment-engineer |
| "Dashboard is slow on mobile" | ui-ux-polisher + performance-optimizer |
| "Add JWT refresh token rotation" | security-reviewer → bug-fixer (if broken) |
| "Design multi-tenant schema" | fullstack-architect → database-prisma-sql-expert |
| "Add Claude streaming chat" | ai-saas-builder |
| "Commit my feature work" | git-commit-assistant |
| "We got a security report" | security-reviewer (priority: Critical first) |
| "Migrate from MySQL to PostgreSQL" | database-prisma-sql-expert |
| "Dockerize and deploy to Azure" | deployment-engineer |

---

## Output Format

```
## Orchestration Plan

**Task:** [what you described]
**Complexity:** Simple / Multi-step / Full workflow

**Skills to Run (in order):**
1. [skill-name] — [why this skill, what it will do]
2. [skill-name] — [why this skill, what it will do]

**Starting with:** [skill-name]

---
[skill execution begins here]
```
