# Skill: Changelog Generator

## Purpose
Read git history since the last release tag and auto-generate a structured, human-readable CHANGELOG.md entry — grouped by type (features, fixes, performance, breaking changes) — suitable for release notes, GitHub releases, and user-facing announcements.

## When to Use
- Preparing a release
- After every sprint to document what shipped
- Generating release notes for GitHub/GitLab releases
- Communicating changes to non-technical stakeholders
- Before bumping the version in package.json / pom.xml

---

## Universal Agent Rules (Follow for Every Task)
1. **Read code first** — read git log and actual diffs; don't generate changelog from assumptions.
2. **Identify root cause** — group commits by their actual meaning, not just their type prefix.
3. **Implement minimal safe fix** — only include commits since the last tag; do not include unreleased work.
4. **Run tests / typecheck / build** — confirm build passes before tagging a release.
5. **Explain files changed** — list CHANGELOG.md as the only file modified.

---

## Skill-Specific Rules
- Only include commits that are user-visible (skip chore, ci, docs unless notable).
- Breaking changes MUST be at the top, clearly marked.
- Group commits by: Features, Bug Fixes, Performance, Security, Breaking Changes.
- Rewrite commit messages into user-friendly language (not technical jargon).
- Link to PR/issue numbers where available.
- Date format: ISO 8601 (YYYY-MM-DD).

---

## Commit Type → Changelog Section Mapping

```
feat:     → ## Features (new capabilities)
fix:      → ## Bug Fixes
perf:     → ## Performance
security: → ## Security
refactor: → (skip — internal, not user-visible)
test:     → (skip — internal)
chore:    → (skip — unless it's a dependency upgrade)
ci:       → (skip — unless it affected deployment)
docs:     → ## Documentation (if notable)
BREAKING: → ## Breaking Changes (always at top)
deps:     → ## Dependencies (if user-visible security fix)
```

---

## Step-by-Step Workflow

### Step 1 — Find Last Release
```bash
git tag --sort=-v:refname | head -5
# Find the most recent tag: e.g., v1.2.3

git log v1.2.3..HEAD --oneline
# This is everything to include in the changelog
```

### Step 2 — Collect Commits
```bash
git log v1.2.3..HEAD --pretty=format:"%h %s (%an)" --no-merges
# --no-merges skips merge commits
# Filter out: chore, ci, test, refactor (unless user-visible)
```

### Step 3 — Parse and Categorize
```
For each commit:
1. Extract type: feat/fix/perf/security/BREAKING
2. Extract scope: (auth)/(resume)/(db)/ etc.
3. Extract description
4. Rewrite as user-friendly sentence:
   - "feat(auth): add Google OAuth" → "Added Google OAuth sign-in"
   - "fix(resume): prevent crash on empty sections" → "Fixed crash when resume has empty sections"
   - "perf(db): add index on users.email" → "Improved login speed with email index"
5. Link to PR: "(#123)" if PR number in commit or log
```

### Step 4 — Write CHANGELOG Entry
```
## [vX.Y.Z] — YYYY-MM-DD

### ⚠️ Breaking Changes
- [description] — [migration instructions]

### ✨ New Features
- [description] (#PR)

### 🐛 Bug Fixes
- [description] (#PR)

### ⚡ Performance
- [description]

### 🔒 Security
- [description] (CVE-XXXX-XXXX if applicable)

### 📦 Dependencies
- Upgraded [package] from vX to vY (fixes CVE-...)
```

### Step 5 — Generate Audience Variants
```
Technical changelog: for developers (all groups, with file names)
User-facing release notes: only Features + Bug Fixes (plain English)
LinkedIn/social post: top 3 highlights in 2 sentences
```

---

## Commands to Run

```bash
# Get last tag
LAST_TAG=$(git tag --sort=-v:refname | head -1)
echo "Last tag: $LAST_TAG"

# All commits since last tag (no merges)
git log ${LAST_TAG}..HEAD --pretty=format:"%h | %s | %an | %ad" --date=short --no-merges

# Commits by type
git log ${LAST_TAG}..HEAD --oneline --no-merges | grep "^[a-f0-9]* feat"
git log ${LAST_TAG}..HEAD --oneline --no-merges | grep "^[a-f0-9]* fix"
git log ${LAST_TAG}..HEAD --oneline --no-merges | grep -i "BREAKING\|feat!:"

# PR numbers from merge commits
git log ${LAST_TAG}..HEAD --merges --pretty=format:"%s" | grep -oP "#\d+"

# Files changed since last tag
git diff ${LAST_TAG}..HEAD --stat | tail -3
```

---

## CHANGELOG.md Template

```markdown
# Changelog

All notable changes to this project are documented here.
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
This project adheres to [Semantic Versioning](https://semver.org/).

## [Unreleased]

## [1.3.0] — 2026-05-21

### ⚠️ Breaking Changes
- The `/api/resume` endpoint now requires `Content-Type: application/json`.
  Previously it accepted both JSON and form-data. (#145)

### ✨ New Features
- Added AI-powered resume tailoring — paste a job description to get tailored bullet points. (#142)
- Added PDF export with custom fonts and layout templates. (#138)
- Added resume version history (up to 10 versions per resume). (#135)

### 🐛 Bug Fixes
- Fixed crash when resume has empty work experience sections. (#148)
- Fixed timer resetting on question navigation in mock tests. (#147)
- Fixed PDF export failing for resumes with special characters. (#144)

### ⚡ Performance
- Improved AI tailoring response time from 4.2s to 1.1s with streaming. (#141)
- Added email index — login is now 3× faster for large user bases. (#139)

### 🔒 Security
- Fixed IDOR vulnerability: users can no longer access other users' resumes. (#149)

### 📦 Dependencies
- Upgraded Next.js from 14.2 to 15.1 (performance improvements, stability fixes).

[1.3.0]: https://github.com/org/repo/compare/v1.2.0...v1.3.0
[1.2.0]: https://github.com/org/repo/compare/v1.1.0...v1.2.0
```

---

## Validation Checklist
- [ ] All commits since last tag reviewed
- [ ] Breaking changes section present (if any)
- [ ] User-friendly language (not raw commit messages)
- [ ] PR/issue numbers linked where available
- [ ] Security fixes included
- [ ] CHANGELOG.md prepended (new entry at top)
- [ ] Version and date correct
- [ ] Compare link added at bottom

---

## Final Response Format

```
## Changelog Generated

**Version:** [vX.Y.Z]
**Date:** [YYYY-MM-DD]
**Commits Included:** [n]
**Previous Tag:** [vA.B.C]

**Summary:**
- Features: [n]
- Bug Fixes: [n]
- Breaking Changes: [n]
- Performance: [n]
- Security: [n]

**Changelog entry prepended to CHANGELOG.md.**

**Release Notes (user-facing summary):**
[2–3 sentence highlights for non-technical audience]
```
