# Skill: Bug Fixer

## Purpose
Systematically diagnose and resolve bugs across any tech stack (React, Next.js, Spring Boot, Python, Node.js, etc.) without introducing regressions or side effects.

## When to Use
- Runtime errors, exceptions, or crashes
- Build or compilation failures
- Logic errors producing wrong output
- Broken UI behavior or API responses
- Test failures you can't immediately explain

---

## Universal Agent Rules (Follow for Every Task)
1. **Read code first** — read the relevant files before proposing any change. Never assume.
2. **Identify root cause** — trace the error to its actual source, not the symptom.
3. **Implement minimal safe fix** — smallest change that resolves the problem. No scope creep.
4. **Run tests / typecheck / build** — validate before reporting complete.
5. **Explain files changed** — list every modified file with a one-line reason.

---

## Skill-Specific Rules
- Never guess a fix. Trace the error to its root cause first.
- Fix only what is broken. Do not refactor unrelated code.
- Do not introduce new dependencies to solve a simple bug.
- Preserve the existing code style and patterns.
- If a fix touches shared utilities, verify no other callers break.
- Write the minimal change that eliminates the bug.

---

## Step-by-Step Workflow

### Step 1 — Reproduce
```
1. Read the full error message and stack trace carefully.
2. Identify the file, line number, and failing function.
3. Understand what the code is supposed to do vs. what it actually does.
```

### Step 2 — Trace the Root Cause
```
1. Follow the call chain upward from the error site.
2. Check: wrong type, null/undefined, off-by-one, missing await, bad state.
3. Search for recent changes in git that may have introduced the bug.
4. Check environment variables and config if behavior differs by environment.
```

### Step 3 — Fix
```
1. Make the smallest possible change to resolve the root cause.
2. Do not change function signatures unless absolutely necessary.
3. If the fix needs a helper, keep it local to the file.
```

### Step 4 — Verify
```
1. Re-run the failing test or reproduce the original error scenario.
2. Run the full test suite for the affected module.
3. Grep for other usages of the fixed function/component.
4. Check that the fix works in both dev and prod builds.
```

---

## Commands to Run

### JavaScript / TypeScript
```bash
npm run build            # catch compile errors
npm test -- --watch      # run tests for changed files
npx tsc --noEmit         # type-check without building
```

### Python
```bash
python -m pytest tests/ -x -v    # stop at first failure
python -m mypy src/               # type check
```

### Java / Spring Boot
```bash
./mvnw test -pl <module>          # run module tests
./mvnw compile                    # catch compile errors
```

### General
```bash
git diff HEAD~1                   # see what changed recently
grep -rn "functionName" src/      # find all usages
```

---

## Validation Checklist
- [ ] The original error no longer occurs
- [ ] All existing tests still pass
- [ ] No TypeScript / linting errors introduced
- [ ] No unrelated files modified
- [ ] Fix is understandable without comments (rename if needed)
- [ ] Behavior is correct in edge cases (null, empty, boundary values)

---

## Final Response Format

```
## Bug Fix Summary

**Root Cause:** [One sentence — what was actually wrong]

**Fix Applied:** [What was changed and why it resolves the root cause]

**Files Changed:**
- path/to/file.ts — [what changed]

**Verified By:**
- [test run output or repro steps confirmed fixed]

**Watch Out For:**
- [any edge cases or related areas to monitor]
```
