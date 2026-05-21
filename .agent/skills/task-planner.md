# Skill: Autonomous Task Planner

## Purpose
Break any complex engineering task into a structured, sequenced plan with phases, dependencies, time estimates, risk flags, and skill assignments — before a single line of code is written.

## When to Use
- Starting any task estimated > 2 hours
- A feature touches more than 2 layers (frontend + backend + DB + AI)
- Multiple developers need to coordinate on the same feature
- You need to estimate a task for a sprint or roadmap
- The requirements are vague and need clarification before implementation

---

## Universal Agent Rules (Follow for Every Task)
1. **Read code first** — understand existing architecture before writing the plan.
2. **Identify root cause** — for vague tasks, ask clarifying questions before planning.
3. **Implement minimal safe fix** — plan the minimum viable implementation first; extensions go in Phase 2.
4. **Run tests / typecheck / build** — validate that phase completion criteria are testable.
5. **Explain files changed** — every phase must list the files it creates or modifies.

---

## Skill-Specific Rules
- Never start implementing before the plan is confirmed.
- Every phase must have: goal, steps, files affected, completion test, time estimate.
- Flag all external dependencies (third-party APIs, external teams) explicitly.
- If requirements are ambiguous, block the plan with a [QUESTION] before proceeding.
- A plan is a living document — update it as you learn during implementation.

---

## Step-by-Step Workflow

### Step 1 — Understand the Task
```
1. Read the task description fully.
2. Identify: What is the user-visible outcome?
3. Identify: What data changes are needed? (schema, state)
4. Identify: What API surface is needed? (new routes, modified routes)
5. Identify: What UI changes are needed?
6. Flag any ambiguities as [QUESTION] items — get answers before planning.
```

### Step 2 — Decompose into Phases
```
Group work into sequential phases:
- Phase 0: Foundation — schema, migrations, types
- Phase 1: Backend — API routes, service logic, validation
- Phase 2: Frontend — UI components, API integration, state
- Phase 3: AI/Integrations — third-party services, AI features
- Phase 4: Polish — tests, error handling, loading states
- Phase 5: Deploy — config, env vars, CI/CD

Not every task needs all phases. Skip phases that don't apply.
```

### Step 3 — Estimate Each Phase
```
For each phase:
- Time estimate: optimistic / realistic / pessimistic (use 3-point)
- Risk level: Low / Medium / High
- Dependencies: what must be done before this phase
- Blockers: what external dependency could delay this
- Completion test: how will you know this phase is done?
```

### Step 4 — Assign Skills
```
Map each phase to the appropriate skill:
- Phase 0 (schema)      → database-prisma-sql-expert
- Phase 1 (API)         → fullstack-architect, security-reviewer
- Phase 2 (UI)          → ui-ux-polisher
- Phase 3 (AI)          → ai-saas-builder
- Phase 4 (tests)       → test-generator
- Phase 5 (deploy)      → deployment-engineer
```

### Step 5 — Identify Risks
```
Common risk flags:
- [BREAKING] — this change breaks existing API contracts
- [MIGRATION] — requires a DB migration (potential downtime)
- [EXTERNAL] — depends on a third-party API (rate limits, availability)
- [PERF] — may impact performance under load
- [SECURITY] — touches auth/authorization boundary
- [AI] — AI output is non-deterministic (needs evaluation)
```

---

## Plan Template

```markdown
# Task Plan: [Task Name]
**Created:** [date]
**Estimated Total:** [X–Y hours / days]
**Complexity:** Simple / Medium / Complex / Epic

## Clarifying Questions (must resolve before starting)
- [QUESTION] [question text] → [answer when known]

## Architecture Decision
[one paragraph — approach chosen and why]

## Phase 0: Foundation
**Goal:** [what this phase achieves]
**Time:** [X hours]
**Risk:** Low / Medium / High
**Files Created/Modified:**
- `prisma/schema.prisma` — [what changes]
**Steps:**
1. [step]
2. [step]
**Completion Test:** [how to verify done]

## Phase 1: Backend
[same structure]

## Phase 2: Frontend
[same structure]

## Phase 3: Integrations / AI
[same structure]

## Phase 4: Tests & Error Handling
[same structure]

## Phase 5: Deploy
[same structure]

## Risk Register
| Risk | Phase | Severity | Mitigation |
|------|-------|----------|-----------|
| [risk] | [phase] | High/Med/Low | [mitigation] |

## Definition of Done
- [ ] All phases complete
- [ ] Tests pass
- [ ] No TypeScript errors
- [ ] Security review passed (if Phase 1 touched auth)
- [ ] PR created and reviewed
```

---

## Example Plans

### GlobalResumeAI — AI Resume Tailoring Feature
```
Phase 0 (1h): Add AISuggestion table to Prisma schema
Phase 1 (2h): POST /api/resume/[id]/tailor route + Anthropic integration
Phase 2 (3h): AISuggestions panel + accept/reject UI
Phase 3 (1h): Streaming response with loading state
Phase 4 (2h): Tests + error handling for AI failures
Phase 5 (1h): Env vars + Vercel deploy config

Total: 10h realistic
Risks: [AI] non-deterministic output, [MIGRATION] new table
```

### CAT Prep App — Timer Persistence Fix
```
Phase 0 (30m): Add checkpoint column to TestAttempt
Phase 1 (1h): POST /api/test/checkpoint saves timer state every 30s
Phase 2 (1h): Timer component reads from DB on mount, saves on interval
Phase 4 (30m): Tests for checkpoint save and restore

Total: 3h realistic
Risks: [MIGRATION] schema change, [PERF] 30s polling under load
```

---

## Commands to Run

```bash
# Save plan to .agent/memory/
mkdir -p .agent/memory
# Plan is saved as: .agent/memory/plan-[feature-name].md

# Track phase completion
grep -c "\- \[x\]" .agent/memory/plan-*.md   # count completed items

# Check for blocking questions
grep "\[QUESTION\]" .agent/memory/plan-*.md
```

---

## Validation Checklist
- [ ] All clarifying questions answered
- [ ] Every phase has a completion test
- [ ] Time estimates given (not just "TBD")
- [ ] Risk flags applied to phases that need them
- [ ] Skill assignments made for each phase
- [ ] Definition of Done listed
- [ ] Plan saved to .agent/memory/

---

## Final Response Format

```
## Task Plan

**Task:** [name]
**Total Estimate:** [X–Y hours]
**Phases:** [count]
**Risk Level:** [overall]

### Phase Summary
| Phase | Goal | Est. | Risk | Skill |
|-------|------|------|------|-------|
| 0 | Foundation | Xh | Low | database-prisma-sql-expert |
| 1 | Backend | Xh | Med | fullstack-architect |
| ... | | | | |

### Blocking Questions
[list any, or "None — ready to start"]

**Recommended First Action:**
[exactly what to do now]
```
