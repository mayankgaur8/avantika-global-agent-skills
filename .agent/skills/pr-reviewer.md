# Skill: PR Review Assistant

## Purpose
Review pull requests with the depth and consistency of a senior engineer — covering code correctness, security, test coverage, performance implications, breaking changes, and conventional commit quality — and produce structured, actionable feedback.

## When to Use
- Reviewing a PR before merge to main
- Self-reviewing your own PR before requesting a human review
- Setting up PR review standards for a team
- Automated pre-merge quality gate

---

## Universal Agent Rules (Follow for Every Task)
1. **Read code first** — read the full diff, not just the summary. Read the files being modified.
2. **Identify root cause** — for issues found, explain WHY it's a problem with a specific example.
3. **Implement minimal safe fix** — suggest the smallest fix that resolves each issue. No scope creep in reviews.
4. **Run tests / typecheck / build** — confirm build passes and no new TS errors introduced.
5. **Explain files changed** — reference exact file:line in every comment.

---

## Skill-Specific Rules
- Give actionable feedback: "Rename X to Y because..." not "naming could be improved".
- Distinguish blocking from non-blocking: BLOCK / WARN / SUGGEST.
- Do not request changes for style preferences — only for correctness, security, and quality gates.
- Always acknowledge what's done well (strength section in output).
- Check for unintended files in the diff (debug code, .env, IDE config).

---

## Review Dimensions (in order of priority)

### 1. Correctness (CRITICAL)
```
- Does the code do what the PR description says?
- Are there edge cases not handled? (null, empty, boundary values)
- Is error handling present on async operations?
- Are race conditions possible in the implementation?
- Does the implementation match the spec / ticket requirements?
```

### 2. Security (CRITICAL)
```
- Does the PR introduce any of the OWASP Top 10?
- Is user input validated server-side?
- Is authorization checked at the route level?
- Are there any secret values hardcoded?
- Does the diff include changes to auth/session middleware?
```

### 3. Breaking Changes
```
- Does this PR change any public API contract (routes, request shape, response shape)?
- Does it rename, remove, or change the behavior of exported functions?
- Does it change DB schema in a way that requires migration downtime?
- Is there a migration path for existing users/data?
```

### 4. Test Coverage
```
- Are the critical paths covered by tests?
- Are edge cases tested?
- Are new API routes tested with at least a success + error case?
- Do existing tests still pass? (no skipped/modified tests without reason)
```

### 5. Performance
```
- Does this introduce N+1 queries?
- Are there synchronous operations that should be async?
- Is there unnecessary data fetching (select * vs. select specific fields)?
- Does this add blocking operations to the hot path?
```

### 6. Code Quality
```
- Are functions short and single-purpose?
- Are names descriptive?
- Is there unnecessary complexity?
- Is there dead code or debug console.logs?
- Are there TODO comments that should be issues/tickets instead?
```

### 7. Commit Quality
```
- Do commits follow Conventional Commits format?
- Is each commit atomic (one logical change)?
- Are commit messages descriptive?
- Is there a squash needed before merge?
```

---

## Step-by-Step Workflow

### Step 1 — PR Overview
```bash
# Get the PR diff
gh pr diff <number>
# OR
git diff main...HEAD

# PR metadata
gh pr view <number>
```

### Step 2 — Read the Diff
```
1. Read the PR description — what is this supposed to do?
2. Read each changed file in full (not just the diff hunk)
3. Map the change: what layer is affected? (route / service / DB / UI)
4. Identify the "critical path" — what's the most important thing to get right?
```

### Step 3 — Check for Unintended Files
```bash
gh pr diff <number> --name-only
# Flag any: .env, *.log, package-lock.json (if unrelated), IDE config, test fixtures
```

### Step 4 — Run Automated Checks
```bash
git checkout <branch>
npm run build        # must pass
npx tsc --noEmit     # must pass
npm test             # must pass
npm run lint         # must pass
npm audit --audit-level=high  # no critical CVEs introduced
```

### Step 5 — Write Review Comments
```
Format for each comment:
[BLOCK] / [WARN] / [SUGGEST] — short title
File: path/to/file.ts:42
Issue: [what is wrong]
Reason: [why this is a problem]
Fix: [concrete suggestion]
```

---

## Review Comment Templates

```
[BLOCK] Missing authorization check
File: src/app/api/resume/[id]/route.ts:12
Issue: The GET handler does not verify that the requesting user owns the resume.
Reason: Any authenticated user can access any resume by guessing the ID (IDOR).
Fix: Add `if (resume.userId !== session.user.id) return NextResponse.json({}, { status: 403 })`

[BLOCK] Unhandled promise rejection
File: src/services/ai.service.ts:87
Issue: `await anthropic.messages.create(...)` is not wrapped in try/catch.
Reason: If Anthropic API returns a 500, the route handler will throw an unhandled exception.
Fix: Wrap in try/catch and return a 503 with a user-friendly message.

[WARN] N+1 query pattern
File: src/app/api/dashboard/route.ts:34
Issue: `prisma.resume.findMany()` inside a loop fetches sections for each resume separately.
Reason: For a user with 10 resumes, this fires 11 DB queries instead of 1.
Fix: Use `include: { sections: true }` in the outer findMany call.

[SUGGEST] Consider memoizing this computation
File: src/components/ScoreChart.tsx:56
Issue: `calculatePercentile(scores)` is called on every render.
Reason: This could cause jank on slow devices if scores array is large.
Fix: Wrap in useMemo(() => calculatePercentile(scores), [scores])
```

---

## PR Checklist (Self-Review)

Before requesting review, confirm:
- [ ] PR description explains WHY (not just what changed)
- [ ] Linked to issue/ticket
- [ ] No debug console.logs
- [ ] No hardcoded values that should be env vars
- [ ] Tests added for new behavior
- [ ] TypeScript errors: 0
- [ ] Lint errors: 0
- [ ] No sensitive files in diff
- [ ] No BREAKING CHANGE without documentation
- [ ] Commits are clean and follow Conventional Commits

---

## Validation Checklist
- [ ] All 7 review dimensions covered
- [ ] BLOCK items listed first
- [ ] Every comment references specific file:line
- [ ] Build + tests + lint confirmed passing (or failing, if blocking)
- [ ] Unintended files checked
- [ ] Breaking changes documented
- [ ] Positive feedback included

---

## Final Response Format

```
## PR Review: [PR Title] (#number)

**Author:** [name]
**Branch:** [branch → main]
**Files Changed:** [n]

**Decision:** APPROVE / REQUEST CHANGES / COMMENT

### ❌ Blocking Issues ([n])
1. [BLOCK] [issue] — [file:line]

### ⚠️ Warnings ([n])
1. [WARN] [issue] — [file:line]

### 💡 Suggestions ([n])
1. [SUGGEST] [suggestion] — [file:line]

### ✅ What's Done Well
- [strength]

### Automated Check Results
- Build: PASS / FAIL
- Tests: PASS / FAIL (n passing, n failing)
- Types: PASS / FAIL
- Lint: PASS / FAIL
- Security: PASS / ISSUES FOUND
```
