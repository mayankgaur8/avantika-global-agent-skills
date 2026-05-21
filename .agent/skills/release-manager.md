# Skill: Release Manager

## Purpose
Orchestrate the full release process — from version bump through changelog generation, git tagging, GitHub release creation, deployment, and post-release verification — as a single repeatable workflow.

## When to Use
- Releasing a new version to production
- Creating a hotfix release
- Setting up a release process for the first time
- Before any version tag is created

---

## Universal Agent Rules (Follow for Every Task)
1. **Read code first** — read current version in package.json / pom.xml / pyproject.toml before bumping.
2. **Identify root cause** — confirm all tests pass and no Critical security issues before releasing.
3. **Implement minimal safe fix** — release only code that is on main/master and fully reviewed.
4. **Run tests / typecheck / build** — full build + test suite must pass before tagging.
5. **Explain files changed** — list every file touched during release (version, changelog, tag).

---

## Skill-Specific Rules
- Never tag a release on a branch other than main (or the designated release branch).
- Version bump must follow Semantic Versioning (MAJOR.MINOR.PATCH).
- CHANGELOG.md must be updated before tagging.
- Git tag must be annotated (not lightweight): `git tag -a v1.2.3 -m "Release v1.2.3"`.
- Never force-push a release tag.
- Hotfix releases: branch from the previous tag, patch, PR back to main, tag.

---

## Semantic Versioning Rules

```
MAJOR (X.0.0) — Breaking change: existing clients must update their code
MINOR (x.Y.0) — New feature, backward-compatible
PATCH (x.y.Z) — Bug fix, backward-compatible

Pre-release: 1.0.0-beta.1, 1.0.0-rc.1
Hotfix:      1.2.3 → 1.2.4 (patch from previous tag)
```

---

## Step-by-Step Workflow

### Step 1 — Pre-Release Gate
```bash
# 1. Confirm on correct branch
git branch --show-current   # must be: main

# 2. Confirm no uncommitted changes
git status                  # must be: clean

# 3. Pull latest
git pull origin main

# 4. Run full test suite
npm test && npm run build
# OR: ./mvnw clean verify

# 5. Check for P0/P1 known issues
cat .agent/memory/known-issues.md | grep "P0\|P1"
# Any open P0 = STOP. Do not release.

# 6. Security check
npm audit --audit-level=high
# Any critical CVEs = STOP.
```

### Step 2 — Determine Version
```
Checklist:
- Any BREAKING CHANGE in commits since last tag? → MAJOR bump
- Any new features (feat:) in commits? → MINOR bump
- Only bug fixes (fix:) in commits? → PATCH bump

# Current version
cat package.json | grep '"version"'
# Last tag
git tag --sort=-v:refname | head -1
```

### Step 3 — Update Version
```bash
# JavaScript / Node.js
npm version patch   # or minor / major
# This updates package.json, package-lock.json, and creates a git commit

# Python (pyproject.toml)
# Edit version manually in pyproject.toml
# Then: git add pyproject.toml && git commit -m "chore: bump version to X.Y.Z"

# Spring Boot (pom.xml)
./mvnw versions:set -DnewVersion=X.Y.Z
git add pom.xml && git commit -m "chore(release): bump version to X.Y.Z"
```

### Step 4 — Generate Changelog
```
→ Run: changelog-generator skill for this version

Update CHANGELOG.md with the new entry.
git add CHANGELOG.md
git commit -m "docs: update CHANGELOG for vX.Y.Z"
```

### Step 5 — Tag the Release
```bash
git tag -a vX.Y.Z -m "Release vX.Y.Z

[one-paragraph summary of what's in this release]"

git push origin main --tags
```

### Step 6 — Create GitHub Release
```bash
# Using GitHub CLI
gh release create vX.Y.Z \
  --title "Release vX.Y.Z — [release name]" \
  --notes "$(cat <<'EOF'
## What's New
[top 3 features]

## Bug Fixes
[top 3 fixes]

## Full Changelog
[CHANGELOG.md entry or link]
EOF
)"
```

### Step 7 — Deploy
```
→ Run: deployment-engineer skill
Trigger CI/CD pipeline for the release tag.
Confirm health check passes after deploy.
```

### Step 8 — Post-Release Verification
```bash
# Check deployed version matches tag
curl https://yourdomain.com/api/health | grep version

# Smoke test critical flows
[run smoke test script or manual test plan]

# Monitor error rate for 15 minutes
# Watch: Sentry / Datadog / CloudWatch / Application Insights
```

### Step 9 — Announce
```
→ If MINOR or MAJOR release:
- Update .agent/memory/progress.md with milestone
- Post to team Slack / Discord
- If public product: use changelog for social/blog post
- Run: docs-generator skill to update public docs
```

---

## Hotfix Workflow

```bash
# Branch from the previous tag (not from main if main has unreleased work)
git checkout -b hotfix/security-fix vX.Y.Z

# Apply the minimal fix
# ... fix the bug ...

# Test thoroughly
npm test

# PR back to main AND the release branch
# After merge, tag the hotfix
git tag -a vX.Y.Z+1 -m "Hotfix: [description]"
git push origin --tags
```

---

## Version Files by Stack

| Stack | File | Field |
|-------|------|-------|
| Node.js / Next.js | `package.json` | `"version"` |
| Python | `pyproject.toml` | `version = "x.y.z"` |
| Spring Boot | `pom.xml` | `<version>` |
| Docker | `Dockerfile` LABEL | `org.opencontainers.image.version` |
| GitHub | `package.json` → auto-synced with release tag |

---

## Validation Checklist
- [ ] On main branch with clean working tree
- [ ] All tests pass
- [ ] No Critical/High security issues open
- [ ] Version bumped following SemVer
- [ ] CHANGELOG.md updated with this version
- [ ] Git tag is annotated (not lightweight)
- [ ] Tag pushed to remote
- [ ] GitHub release created with release notes
- [ ] Deploy completed and health check passed
- [ ] Smoke tests passed
- [ ] Progress memory updated

---

## Final Response Format

```
## Release Complete

**Version:** [vX.Y.Z]
**Type:** Major / Minor / Patch / Hotfix
**Released:** [YYYY-MM-DD HH:MM UTC]

**Release Checklist:**
- [x] Tests passed
- [x] Version bumped
- [x] Changelog updated
- [x] Tag created: vX.Y.Z
- [x] GitHub release created
- [x] Deployed to production
- [x] Health check: PASS
- [x] Smoke tests: PASS

**GitHub Release URL:** [link]

**Next Steps:**
- Monitor error rate for 1 hour
- [any follow-up actions]
```
