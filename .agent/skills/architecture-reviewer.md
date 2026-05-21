# Skill: Architecture Reviewer

## Purpose
Review an existing system's architecture for anti-patterns, scalability bottlenecks, maintainability issues, and dependency problems — and produce a prioritized improvement roadmap with specific, actionable recommendations.

## When to Use
- Codebase has grown messy after 3+ months of rapid development
- Onboarding a new developer who needs to understand the system
- Before a major new feature that requires structural changes
- Planning a migration (monolith → services, REST → GraphQL, etc.)
- Quarterly architecture health review

---

## Universal Agent Rules (Follow for Every Task)
1. **Read code first** — read the actual directory structure, key files, and dependencies before reviewing.
2. **Identify root cause** — for each finding, explain WHY it's a problem, not just what it is.
3. **Implement minimal safe fix** — recommendations must be incremental. Never suggest "rewrite everything."
4. **Run tests / typecheck / build** — confirm the app actually builds and tests pass before reviewing.
5. **Explain files changed** — for each recommendation, list which files would change.

---

## Skill-Specific Rules
- Review what exists, not what you would have done differently.
- Every finding needs a severity: Critical / High / Medium / Low.
- Every recommendation must specify the migration path (not just "use X instead").
- Never recommend a technology change without a cost/benefit analysis.
- Separate "must fix" from "nice to have" clearly.

---

## Step-by-Step Workflow

### Step 1 — Map the System
```
1. Read the directory structure: ls -la src/ (or equivalent)
2. Read package.json / pom.xml / requirements.txt — what dependencies exist?
3. Read the main entry points: app.ts / main.java / main.py
4. Read the DB schema: prisma/schema.prisma or migrations/
5. Read API routes: identify all public endpoints
6. Draw a mental model: Client → API → Service → DB → External
```

### Step 2 — Layering Review
```
Check separation of concerns:
1. Are HTTP concerns in routes/controllers ONLY? (no business logic in routes)
2. Is business logic in service layer? (not in DB layer, not in UI)
3. Is DB access in repository/DAO layer? (no raw queries in services)
4. Are cross-cutting concerns (auth, logging, errors) in middleware?

Anti-patterns to flag:
- Business logic in React components (should be in hooks or services)
- Direct DB queries in API routes (should go through service layer)
- Circular imports / circular dependencies
- God files (> 500 lines doing too many things)
- Magic numbers / strings (should be constants)
```

### Step 3 — Dependency Review
```
1. Check for circular dependencies: npx madge --circular src/
2. Identify tightly-coupled modules (changing one breaks many others)
3. Look for inappropriate cross-domain imports (auth module importing billing)
4. Check if interfaces/contracts are defined (or is everything concrete?)
5. Identify shared utilities that are actually doing too much
```

### Step 4 — Scalability Review
```
1. State management: Is shared state global (Zustand/Redux) when it should be local?
2. Caching: Are expensive operations cached or recomputed every request?
3. DB: Are queries paginated? Are there N+1 patterns?
4. API: Are endpoints idempotent? Is there pagination on all lists?
5. Background work: Are slow operations async (queued) or blocking HTTP threads?
6. File handling: Is storage on disk (bad) or object storage (good)?
```

### Step 5 — Maintainability Review
```
1. Naming: Are functions/files named for what they DO, not what they ARE?
2. Size: Are files/functions short enough to understand in < 1 minute?
3. Duplication: Is the same logic written in > 2 places?
4. Test coverage: Are the most critical paths tested?
5. Error handling: Are errors caught, logged, and surfaced correctly?
6. Dead code: Are there unused files, commented-out blocks, or TODO graveyards?
```

### Step 6 — Build Migration Path
```
For each finding, specify:
1. Current state: [what exists]
2. Target state: [what it should be]
3. Migration steps: [how to get there incrementally]
4. Effort: [hours/days]
5. Risk: [what could break during migration]
```

---

## Architecture Review Dimensions

```
LAYERING       — separation of concerns, layer isolation
COUPLING       — module dependencies, circular imports
COHESION       — single responsibility, file size, god objects
SCALABILITY    — caching, pagination, async, stateless
SECURITY       — auth at correct layer, no secrets in code
TESTABILITY    — dependency injection, mock-friendliness
OBSERVABILITY  — logging, tracing, error surfacing
CONSISTENCY    — same patterns used throughout, no one-offs
```

---

## Commands to Run

```bash
# Map directory structure
find src -type f -name "*.ts" | head -50

# Check circular dependencies
npx madge --circular src/

# Find large files (God files)
find src -name "*.ts" -exec wc -l {} + | sort -rn | head -20

# Find duplicate code patterns
npx jscpd src/ --threshold 10

# Check test coverage
npm test -- --coverage --coverageReporters=text | tail -30

# Count TODO/FIXME/HACK debt
grep -rn "TODO\|FIXME\|HACK\|XXX" src/ | wc -l

# Find dead exports (unused code)
npx ts-prune

# Dependency graph (Spring Boot)
./mvnw dependency:tree | head -100
```

---

## Validation Checklist
- [ ] Directory structure fully mapped
- [ ] All 8 review dimensions covered
- [ ] Every finding has severity + migration path
- [ ] Circular dependencies checked
- [ ] Large/god files identified
- [ ] Coupling between modules assessed
- [ ] Recommendations are incremental (not "rewrite")
- [ ] Effort estimates given for top recommendations

---

## Final Response Format

```
## Architecture Review Report

**Project:** [name]
**Stack:** [stack]
**Review Date:** [date]
**Overall Health:** Excellent / Good / Fair / Needs Work / Critical

### Architecture Map
[current: Client → API → Service → DB diagram]

### Findings by Severity

**Critical**
1. [finding] — [why it's a problem] — [migration path]

**High**
1. [finding] — [why it's a problem] — [migration path]

**Medium**
1. [finding] — [recommendation]

**Low / Nice to Have**
1. [recommendation]

### Top 3 Recommended Actions (in order)
1. [action] — [effort: Xh] — [impact: High/Med/Low]
2. [action] — [effort: Xh]
3. [action] — [effort: Xh]

### What's Working Well
- [strength] — keep this pattern
```
