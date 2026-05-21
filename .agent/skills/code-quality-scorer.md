# Skill: Code Quality Scorer

## Purpose
Score any codebase or pull request on 8 quality dimensions, identify the top improvements, and produce an actionable report with file-level findings — making quality measurable and improvable over time.

## When to Use
- Quarterly codebase health check
- Before a major feature merge to main
- When onboarding to a new codebase
- To establish quality baseline before a refactor
- As part of PR review to give structured feedback

---

## Universal Agent Rules (Follow for Every Task)
1. **Read code first** — read actual source files before scoring; never score from filename alone.
2. **Identify root cause** — for low scores, identify the specific files driving the score down.
3. **Implement minimal safe fix** — suggest the highest-impact fix per dimension, not everything at once.
4. **Run tests / typecheck / build** — run linters and type checkers as part of scoring.
5. **Explain files changed** — for each recommendation, name the specific file to change.

---

## Skill-Specific Rules
- Scores are 0–10 per dimension. Explain the score; never just state a number.
- A score of 5+ means "acceptable"; < 5 means "needs attention"; < 3 means "blocking".
- Focus findings on the files that most affect the overall score.
- Do not penalize intentional trade-offs (e.g., skipping tests on seeding scripts).

---

## The 8 Quality Dimensions

### 1. Type Safety (0–10)
```
10: 100% TypeScript strict mode, no `any`, all return types explicit
7: TypeScript used, occasional `any`, most types inferred correctly
5: TypeScript present, significant `any` usage, some missing types
3: TypeScript in name only — mostly `any` or implicit `any`
0: No typing, plain JS

Check:
- npx tsc --noEmit (0 errors = good)
- grep -rn ": any" src/ | wc -l
- tsconfig.json has "strict": true
```

### 2. Test Coverage (0–10)
```
10: > 90% coverage, meaningful tests (not just coverage-chasing)
7: 70–90% coverage, critical paths well tested
5: 50–70% coverage, happy paths tested
3: < 50% coverage, or tests that don't assert anything real
0: No tests

Check:
- npm test -- --coverage
- Focus on: business logic coverage, not just line coverage
- Look for: tests that test the right things, not just any code
```

### 3. Complexity (0–10)
```
10: Functions < 20 lines, cyclomatic complexity < 5, single responsibility
7: Most functions focused, a few large ones
5: Some god functions/files (> 100 lines), nested conditions
3: Regular god files (> 300 lines), deeply nested logic
0: Monolithic files, everything mixed together

Check:
- find src -name "*.ts" -exec wc -l {} + | sort -rn | head -10
- grep -rn "if\|else\|switch\|catch" src/ | awk -F: '{print $1}' | sort | uniq -c | sort -rn | head -10
```

### 4. Duplication (0–10)
```
10: DRY — no repeated logic, shared utilities used consistently
7: Minor duplication, < 5% duplicated blocks
5: Noticeable duplication, same logic in 3+ places
3: Significant copy-paste, major refactoring needed
0: Same code block appears everywhere

Check:
- npx jscpd src/ --threshold 5
```

### 5. Naming Quality (0–10)
```
10: Function/variable names describe exactly what they do, no abbreviations
7: Generally clear names, occasional unclear ones
5: Mix of clear and cryptic names (x, temp, data, obj)
3: Single-letter variables, non-descriptive names throughout
0: Obfuscated or auto-generated names

Check:
- grep -rn "\bx\b\|\btemp\b\|\bdata\b\|\bobj\b\|\bres\b\|\bval\b" src/ | wc -l
```

### 6. Error Handling (0–10)
```
10: All async operations wrapped, errors typed, user-friendly messages, nothing swallowed
7: Most errors handled, occasional unhandled promise rejection
5: try/catch present but catch blocks are empty or just console.log
3: Multiple unhandled promise rejections, raw errors exposed to users
0: No error handling

Check:
- grep -rn "catch.*{}" src/          # empty catch blocks
- grep -rn "console.error" src/       # basic error logging only
- grep -rn "res.status(500)" src/     # raw error responses
```

### 7. Documentation (0–10)
```
10: Complex functions have 1-line WHY comments, public APIs have JSDoc
7: Key public functions documented, complex logic has context
5: Some comments, mostly outdated or obvious
3: No documentation on public APIs
0: No comments anywhere

Check:
- grep -rn "\/\*\*\|\/\/ " src/ | wc -l
- Note: score LOWER for obvious comments ("// increment counter") and HIGHER for why-comments
```

### 8. Consistency (0–10)
```
10: Same patterns used throughout — no one-offs, no mixing paradigms
7: Mostly consistent, 1-2 legacy patterns
5: Inconsistent patterns in multiple areas
3: Mix of different frameworks, naming styles, patterns
0: Every file looks like it was written by someone different

Check:
- Are imports using the same style (named vs default, path aliases)?
- Same error handling pattern in all routes?
- Same async pattern (async/await vs .then)?
```

---

## Step-by-Step Workflow

### Step 1 — Setup
```
1. Check build: npm run build / ./mvnw compile
2. Check lint: npm run lint
3. Check types: npx tsc --noEmit
4. Check tests: npm test -- --coverage
5. Check circular deps: npx madge --circular src/
```

### Step 2 — Score Each Dimension
```
For each of the 8 dimensions:
1. Run the check command
2. Assign score 0–10
3. Note the top 3 files driving the score
4. Write one actionable recommendation
```

### Step 3 — Calculate Overall Score
```
Overall = average of all 8 dimensions
Weight heavier: Type Safety (×1.5) + Test Coverage (×1.5) + Error Handling (×1.2)

Grade:
- 8.5–10: A — Production-grade
- 7.0–8.4: B — Good, minor improvements needed
- 5.5–6.9: C — Acceptable, planned improvements needed
- 4.0–5.4: D — Below standard, improvements required before next release
- 0–3.9:  F — Critical, must improve before production
```

---

## Commands to Run

```bash
npm run build && npx tsc --noEmit
npm test -- --coverage --coverageReporters=text
npx eslint src/ --max-warnings=0
npx jscpd src/ --threshold 5 2>/dev/null
npx madge --circular src/ 2>/dev/null
find src -name "*.ts" -exec wc -l {} + | sort -rn | head -15
grep -rn ": any" src/ | wc -l
grep -rn "catch.*{\s*}" src/ | head -10
```

---

## Validation Checklist
- [ ] All 8 dimensions scored
- [ ] Top 3 files identified per low-scoring dimension
- [ ] One actionable fix per dimension below 7
- [ ] Overall grade calculated
- [ ] Score saved to .agent/memory/ for trend tracking
- [ ] Previous score compared (if available)

---

## Final Response Format

```
## Code Quality Report

**Project:** [name]
**Date:** [date]
**Overall Score:** [X.X/10] — Grade: [A/B/C/D/F]

| Dimension | Score | Top Issue | Recommendation |
|-----------|-------|-----------|----------------|
| Type Safety | X/10 | [issue] | [fix] |
| Test Coverage | X/10 | [issue] | [fix] |
| Complexity | X/10 | [issue] | [fix] |
| Duplication | X/10 | [issue] | [fix] |
| Naming | X/10 | [issue] | [fix] |
| Error Handling | X/10 | [issue] | [fix] |
| Documentation | X/10 | [issue] | [fix] |
| Consistency | X/10 | [issue] | [fix] |

**Top 3 Files to Improve:**
1. [file] — [score impact] — [what to fix]

**Priority Improvements (highest ROI):**
1. [improvement] — [score gain] — [effort]
```
