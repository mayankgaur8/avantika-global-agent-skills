# Skill: Deployment Engineer

## Purpose
Prepare any application for production deployment on modern platforms (Vercel, Railway, Render, AWS, Azure, GCP, Docker/K8s) with correct config, CI/CD pipelines, and zero-downtime deploy strategy.

## When to Use
- First-time deployment of a new app
- Migrating from one hosting platform to another
- Setting up CI/CD pipeline from scratch
- Dockerizing an application
- Configuring environment variables and secrets for production

---

## Rules
- Never commit secrets to source control — use platform secret managers or .env injection.
- Production build must be tested locally before deploying.
- Every deploy must have a rollback path.
- Health checks must pass before routing traffic to new instances.
- Lock file (package-lock.json, yarn.lock, poetry.lock) must be committed.

---

## Step-by-Step Workflow

### Step 1 — Pre-Deploy Checklist
```
1. Run production build locally: npm run build / ./mvnw package / python -m build
2. Confirm all environment variables are documented in .env.example
3. Verify no secrets are hardcoded in source
4. Run npm audit / pip-audit and resolve critical/high CVEs
5. Confirm database migrations are ready and reversible
```

### Step 2 — Dockerfile (if containerizing)
```dockerfile
# Multi-stage build — keep image small

# Stage 1: Build
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

# Stage 2: Runtime
FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
COPY --from=builder /app/.next ./.next
COPY --from=builder /app/public ./public
COPY --from=builder /app/package*.json ./
COPY --from=builder /app/node_modules ./node_modules
USER appuser
EXPOSE 3000
CMD ["node", "server.js"]
```

Rules for a good Dockerfile:
- Multi-stage build to keep image small
- Run as non-root user
- .dockerignore must exclude: node_modules, .git, .env, coverage, dist (if included in source)

### Step 3 — Platform-Specific Setup

#### Vercel (Next.js / Node)
```
1. Connect GitHub repo to Vercel.
2. Set Framework Preset: Next.js.
3. Add env vars in Vercel Dashboard → Settings → Environment Variables.
4. Set NEXTAUTH_URL = https://yourdomain.com in production.
5. Configure custom domain and enable HTTPS.
6. Set up preview deployments for pull requests.
```

#### Railway / Render (Node, Python, Java)
```
1. Connect repo. Set build command and start command.
2. Add env vars in platform dashboard.
3. Enable health check on /health endpoint.
4. Set auto-deploy on push to main.
5. Configure DB as a managed service on the same platform.
```

#### Azure App Service
```
1. Create App Service Plan (B1 minimum for production).
2. Deploy via GitHub Actions or Azure CLI.
3. Set Application Settings (env vars) in Configuration blade.
4. Enable managed identity for Key Vault access.
5. Configure deployment slots for blue-green deployments.
6. Set up Application Insights for monitoring.
```

#### AWS (ECS / Elastic Beanstalk)
```
1. Push Docker image to ECR.
2. Create ECS task definition with resource limits.
3. Set up Application Load Balancer with target group.
4. Store secrets in AWS Secrets Manager; inject via task definition.
5. Configure auto-scaling policy.
6. Set up CloudWatch alarms on error rate and latency.
```

### Step 4 — CI/CD Pipeline (GitHub Actions)
```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20', cache: 'npm' }
      - run: npm ci
      - run: npm run lint
      - run: npm test -- --ci

  deploy:
    needs: test
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4
      - name: Deploy to Vercel
        run: npx vercel --prod --token=${{ secrets.VERCEL_TOKEN }}
```

### Step 5 — Post-Deploy Verification
```
1. Hit the health check endpoint: curl https://yourdomain.com/health
2. Run smoke tests against production URL.
3. Check error rate in monitoring dashboard for 10 minutes.
4. Verify environment variables are loaded correctly (check /health response).
5. Test the critical user path (login → core action → logout).
```

---

## Commands to Run

```bash
# Local production build test
npm run build && npm start

# Docker build and test locally
docker build -t myapp:latest .
docker run -p 3000:3000 --env-file .env.production myapp:latest

# Check image size
docker images myapp:latest

# Scan image for vulnerabilities
docker scout cves myapp:latest

# Verify .dockerignore
cat .dockerignore

# Check for exposed secrets in git history
git log --all --full-history -- .env
```

---

## .dockerignore Template
```
node_modules
.git
.env
.env.*
coverage
.next/cache
*.log
Dockerfile
.dockerignore
README.md
```

---

## Validation Checklist
- [ ] Production build succeeds locally
- [ ] No secrets in source code or committed .env files
- [ ] Dockerfile runs as non-root user
- [ ] Health check endpoint exists and returns 200
- [ ] CI pipeline runs tests before deploying
- [ ] Environment variables configured in platform dashboard
- [ ] Database migrations applied before app starts
- [ ] Rollback strategy documented
- [ ] Custom domain and HTTPS configured
- [ ] Monitoring/alerting in place

---

## Final Response Format

```
## Deployment Summary

**Platform:** [Vercel / Railway / Azure / AWS / etc.]
**Deploy Strategy:** [direct push / blue-green / canary]

**Config Files Created/Modified:**
- [file] — [purpose]

**Environment Variables Required:**
- VAR_NAME — [what it is, where to get it]

**Deploy Command:**
[exact command or pipeline trigger]

**Verification Steps:**
1. [step]
2. [step]

**Rollback Plan:**
[how to revert if something goes wrong]
```
