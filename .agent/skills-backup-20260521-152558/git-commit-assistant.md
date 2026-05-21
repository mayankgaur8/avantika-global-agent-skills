# Skill: Git Commit Assistant

## Purpose
Stage, commit, and push code changes with clean, conventional commit messages that accurately describe what changed and why — consistently across all projects.

## When to Use
- Committing a completed feature or bug fix
- Committing a batch of related changes
- Writing a commit message that will appear in a PR or changelog
- Cleaning up messy staged files before committing
- Setting up a clean commit history before opening a PR

---

## Rules
- Commit only what belongs together. One logical change = one commit.
- Never commit broken code to main/master. Tests must pass.
- Never commit secrets (.env, API keys, credentials). Check before every commit.
- Use Conventional Commits format (enforced below).
- Do not use `git add .` blindly — review what you're staging.
- Write the commit message for the person reading the git log 6 months from now.

---

## Conventional Commits Format

```
<type>(<scope>): <short summary>

[optional body]

[optional footer: BREAKING CHANGE, Closes #issue]
```

### Types
| Type | When to Use |
|------|-------------|
| `feat` | New feature added |
| `fix` | Bug fix |
| `perf` | Performance improvement |
| `refactor` | Code restructure without behavior change |
| `style` | Formatting, whitespace (no logic change) |
| `test` | Adding or fixing tests |
| `docs` | Documentation only |
| `chore` | Build scripts, dependencies, config |
| `ci` | CI/CD pipeline changes |
| `revert` | Reverting a previous commit |

### Scope Examples
```
feat(auth): add Google OAuth login
fix(dashboard): prevent crash on empty data array
perf(api): add Redis cache to /users endpoint
refactor(payments): extract Stripe logic into service class
chore(deps): upgrade next.js to 15.1.0
```

### Good vs Bad Commit Messages
```
# BAD
git commit -m "fix stuff"
git commit -m "WIP"
git commit -m "changes"
git commit -m "updated files"

# GOOD
git commit -m "fix(auth): resolve JWT expiry not checked on refresh endpoint"
git commit -m "feat(resume): add PDF export with custom font support"
git commit -m "perf(db): add index on users.email to speed up login query"
```

---

## Step-by-Step Workflow

### Step 1 — Review Changes
```bash
git status                    # see all changed/untracked files
git diff                      # see unstaged changes in detail
git diff --staged             # see what's already staged
```

### Step 2 — Check for Secrets
```bash
# Never commit these
grep -rn "PRIVATE_KEY\|SECRET\|PASSWORD\|API_KEY" -- $(git diff --name-only)

# Verify .env is gitignored
cat .gitignore | grep "\.env"
```

### Step 3 — Stage Intentionally
```bash
# Stage specific files (preferred)
git add src/auth/login.ts src/auth/refresh.ts

# Stage a specific section of a file interactively
git add -p src/utils/helper.ts

# Unstage a file you accidentally staged
git restore --staged path/to/file.ts
```

### Step 4 — Write the Commit
```bash
# Short single-line commit
git commit -m "fix(auth): check JWT expiry on token refresh"

# Multi-line commit with body (for complex changes)
git commit -m "feat(billing): add Stripe subscription with usage-based pricing

- Add StripeService class with subscription create/cancel
- Webhook handler for invoice.paid and customer.subscription.deleted
- Track monthly token usage per org for Stripe metered billing

Closes #142"
```

### Step 5 — Pre-Push Checklist
```bash
git log --oneline -5          # review your last commits
git diff origin/main...HEAD   # see everything you're about to push
npm test                      # confirm tests pass
npm run build                 # confirm build works
```

### Step 6 — Push
```bash
# Push to current branch
git push origin HEAD

# Push new branch and set upstream
git push -u origin feature/my-branch

# Push and open PR (GitHub CLI)
gh pr create --title "feat(auth): add Google OAuth" --body "..."
```

---

## Multi-Commit Workflow (Feature Branch)

```bash
# Start feature branch
git checkout -b feat/resume-pdf-export

# Commit 1: data layer
git add src/services/pdf.service.ts
git commit -m "feat(resume): add PDF generation service using puppeteer"

# Commit 2: API endpoint
git add src/api/resume/export.ts
git commit -m "feat(resume): add POST /api/resume/:id/export endpoint"

# Commit 3: frontend
git add src/components/ResumeActions.tsx
git commit -m "feat(resume): add Export PDF button to resume actions"

# Squash if commits are too granular (before PR merge)
git rebase -i origin/main
```

---

## Commands to Run

```bash
# Full pre-commit routine
git status && git diff --stat && npm test && npm run build

# See what changed since last release tag
git log v1.2.0..HEAD --oneline

# Find commits by keyword
git log --oneline --all --grep="auth"

# Undo last commit (keep changes staged)
git reset --soft HEAD~1

# Amend last commit message (before pushing only)
git commit --amend -m "fix(auth): correct message"

# Check what's in .gitignore
git check-ignore -v path/to/file
```

---

## Validation Checklist
- [ ] No .env or secret files staged
- [ ] Commit message follows Conventional Commits format
- [ ] Only related changes in one commit (no unrelated files)
- [ ] Tests pass before commit
- [ ] Build passes before push
- [ ] Commit message explains WHY, not just what changed
- [ ] Branch name matches the feature/fix being worked on
- [ ] No `console.log` / debug statements in staged files
- [ ] No TODO/FIXME left in the committed code (unless intentional)

---

## Final Response Format

```
## Commit Summary

**Branch:** [branch name]
**Commit(s) Created:**

1. `<type>(<scope>): <message>`
   Files: [list of files]
   
**Pre-Commit Checks:**
- Tests: PASS / FAIL
- Build: PASS / FAIL
- Secrets scan: CLEAN / [flagged file]

**Push Status:** [pushed to origin/branch | not pushed — ready to push]

**Next Step:** [open PR / merge / continue working]
```
