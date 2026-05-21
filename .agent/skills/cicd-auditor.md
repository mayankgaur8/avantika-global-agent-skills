# Skill: CI/CD Auditor

## Purpose
Audit CI/CD pipelines (GitHub Actions, Azure DevOps, GitLab CI, Jenkins) for security vulnerabilities, inefficiencies, missing gates, and best-practice violations — and produce a hardened, optimized pipeline configuration.

## When to Use
- Setting up CI/CD for the first time
- Pipeline is slow (> 10 minutes for a standard build)
- Security audit flags CI/CD as a risk vector
- Secrets may have been exposed in pipeline logs
- Before enabling auto-deploy to production

---

## Universal Agent Rules (Follow for Every Task)
1. **Read code first** — read every .github/workflows/*.yml or equivalent before suggesting changes.
2. **Identify root cause** — for slow pipelines, profile each step before optimizing.
3. **Implement minimal safe fix** — improve the pipeline incrementally; one concern at a time.
4. **Run tests / typecheck / build** — validate pipeline changes in a branch before merging.
5. **Explain files changed** — list every workflow file modified and why.

---

## Skill-Specific Rules
- Never pin to `@main` or `@latest` for third-party actions — always use a commit SHA or exact version tag.
- Secrets must NEVER appear in pipeline logs — use `::add-mask::` or structured secret injection.
- Production deployments must require: tests pass + at least 1 reviewer approval.
- Pipelines must fail fast — cheapest checks first (lint before tests, tests before deploy).
- Every production deploy needs a manual approval gate or protected branch rule.

---

## Step-by-Step Workflow

### Step 1 — Inventory
```
1. List all workflow files: ls .github/workflows/
2. For each workflow, identify: trigger, jobs, steps, dependencies.
3. Map the pipeline stages: lint → test → build → deploy.
4. Note which jobs run in parallel vs series.
5. Check: does a deploy job have a dependency on the test job?
```

### Step 2 — Security Audit
```
1. Check secret usage: are secrets referenced as ${{ secrets.NAME }}? (correct)
   Red flag: hardcoded values, env vars set to literal API keys
2. Check action pinning: uses: actions/checkout@v4 (good) vs @main (bad)
3. Check permissions: does the workflow request only the minimum GitHub permissions?
   jobs.deploy.permissions: { contents: read, deployments: write } — not write-all
4. Check pull_request triggers: does on: pull_request run untrusted code with secrets?
   (Use pull_request_target carefully — it has elevated permissions)
5. Check artifact uploads: are secrets accidentally included in uploaded artifacts?
6. Check environment protection rules: is production environment protected?
```

### Step 3 — Correctness Audit
```
1. Does every deploy job have needs: [test] or equivalent?
2. Are database migrations run before the new app version starts?
3. Is there a health check after deploy (not just "deploy succeeded")?
4. Are rollback instructions documented or automated?
5. Is there a notification on deploy failure?
6. Does the pipeline run on the correct branches (main for prod, feature/* for PR)?
```

### Step 4 — Efficiency Audit
```
1. Identify slow steps: check job logs for steps > 2 minutes.
2. Check dependency caching: is npm ci / pip install cached?
3. Identify parallelizable jobs: lint and test can often run in parallel.
4. Check for unnecessary steps: do you re-install dependencies in every job?
5. Check build matrix: are you testing on more platforms than needed?
6. Check Docker layer caching: is --cache-from used for Docker builds?
```

### Step 5 — Gate Design
```
Required gates for a production pipeline:
- Gate 1: Lint + Type check (fast, < 2 min)
- Gate 2: Unit tests (< 5 min)
- Gate 3: Integration tests (< 10 min)
- Gate 4: Security scan (npm audit / SAST)
- Gate 5: Build succeeds
- Gate 6: [Manual approval for prod] or [Auto-deploy to staging]
- Gate 7: Deploy + health check
```

---

## Optimized GitHub Actions Template

```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

permissions:
  contents: read
  deployments: write

jobs:
  # Gate 1: Fast checks (run in parallel)
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4              # pinned version ✓
      - uses: actions/setup-node@v4
        with: { node-version: '20', cache: 'npm' }
      - run: npm ci
      - run: npm run lint
      - run: npx tsc --noEmit

  # Gate 2: Tests (run in parallel with lint)
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env: { POSTGRES_PASSWORD: test }
        options: --health-cmd pg_isready
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20', cache: 'npm' }
      - run: npm ci
      - run: npm test -- --ci --coverage
      - uses: actions/upload-artifact@v4
        with: { name: coverage, path: coverage/ }

  # Gate 3: Security scan
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm audit --audit-level=high

  # Gate 4: Build
  build:
    needs: [lint, test, security]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20', cache: 'npm' }
      - run: npm ci
      - run: npm run build
      - uses: actions/upload-artifact@v4
        with: { name: build, path: .next/ }

  # Gate 5: Deploy (only on main, after all checks pass)
  deploy:
    needs: [build]
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment: production           # requires manual approval if configured
    steps:
      - uses: actions/checkout@v4
      - name: Deploy to Vercel
        run: npx vercel --prod --token=${{ secrets.VERCEL_TOKEN }}
      - name: Health check
        run: |
          sleep 30
          curl -f https://yourdomain.com/api/health || exit 1
```

---

## Commands to Run

```bash
# List all workflow files
ls -la .github/workflows/

# Check for hardcoded secrets (red flag)
grep -rn "sk-\|api_key\|password\|secret" .github/workflows/

# Check action versions (should not be @main or @latest)
grep -rn "uses:" .github/workflows/ | grep -v "@v[0-9]\|@[a-f0-9]\{40\}"

# Check workflow permissions
grep -A5 "permissions:" .github/workflows/*.yml

# Check for missing needs: on deploy jobs
grep -B5 "deploy\|vercel\|azure" .github/workflows/*.yml | grep -v "needs:"

# Validate workflow YAML syntax
python3 -c "import yaml; yaml.safe_load(open('.github/workflows/deploy.yml'))"
```

---

## Validation Checklist
- [ ] All third-party actions pinned to version tag (not @main)
- [ ] No secrets hardcoded in workflow files
- [ ] Deploy job has `needs: [test]` dependency
- [ ] Production environment has protection rules (manual approval or branch protection)
- [ ] Dependencies are cached (npm ci / pip install)
- [ ] Health check runs after every deploy
- [ ] Pipeline fails fast (lint before tests, tests before deploy)
- [ ] Security scan in pipeline (npm audit / equivalent)
- [ ] No overly broad permissions (`contents: write-all`)
- [ ] Separate workflows for PR checks vs. production deploy

---

## Final Response Format

```
## CI/CD Audit Report

**Pipeline Files Reviewed:** [list]
**Overall Status:** Secure / Needs Work / Critical Issues

### Security Findings
1. [finding] — [file:line] — [fix]

### Correctness Issues
1. [issue] — [fix]

### Efficiency Improvements
1. [improvement] — [estimated time saved]

### Optimized Pipeline
[YAML snippet with fixes applied]

**Estimated Deploy Time After Optimization:** [Xm → Ym]
```
