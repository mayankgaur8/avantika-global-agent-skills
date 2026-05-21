# Skill: Cloud Cost Optimizer

## Purpose
Analyze cloud spending on Vercel, AWS, and Azure — identify waste, right-size resources, and implement cost controls — while maintaining or improving performance and reliability.

## When to Use
- Monthly cloud bill is higher than expected
- Scaling up and want to control costs proactively
- Preparing a cost estimate for a new feature
- AI token costs are growing faster than revenue
- Moving from startup to scale-up (optimizing for margin)

---

## Universal Agent Rules (Follow for Every Task)
1. **Read code first** — read infrastructure config, usage logs, and billing dashboards before recommending changes.
2. **Identify root cause** — find the specific service/resource driving the bill, not just the total.
3. **Implement minimal safe fix** — cost optimizations must not reduce reliability. Test in staging first.
4. **Run tests / typecheck / build** — verify performance is maintained after optimizations.
5. **Explain files changed** — list every config/code file changed and the expected cost impact.

---

## Skill-Specific Rules
- Never optimize for cost at the expense of data integrity or security.
- Get a baseline cost (this month's bill) before making any changes.
- Every optimization must have an estimated savings figure.
- Validate that optimization doesn't create a performance regression.
- Set up budget alerts BEFORE implementing changes (cost spikes can happen during migration).

---

## Part 1: Vercel Cost Optimization

### Common Cost Drivers
```
1. Serverless function invocations — charged per execution + duration
2. Edge function invocations — cheaper but limited runtime
3. Bandwidth — data transferred out
4. Image optimization — Next.js /images/* transformations
5. Build minutes — CI builds per month
```

### Optimization Strategies

**Reduce Function Invocations**
```typescript
// Cache static API responses with SWR / React Query
// Instead of fetching on every page load:
const { data } = useSWR('/api/categories', fetcher, {
  revalidateOnFocus: false,
  dedupingInterval: 300000, // 5 min cache
})

// Add HTTP cache headers on stable data
return NextResponse.json(data, {
  headers: { 'Cache-Control': 'public, s-maxage=300, stale-while-revalidate=600' }
})
```

**Move to Edge Runtime (3× cheaper than Node.js serverless)**
```typescript
// app/api/fast-route/route.ts
export const runtime = 'edge'  // ~$0.50/million vs ~$1.50/million for Node.js

// Note: Edge runtime has limitations — no Node.js APIs, no Prisma ORM
// Use for: auth middleware, lightweight transformations, A/B testing
// Keep Node.js for: DB queries, file handling, AI streaming
```

**Reduce Bandwidth**
```typescript
// Compress API responses
return NextResponse.json(data, {
  headers: { 'Content-Encoding': 'gzip' }  // or use Vercel's automatic compression
})

// Paginate large responses (never return more than 100 items)
const items = await prisma.resume.findMany({ take: 20, skip: offset })
```

**Image Optimization**
```typescript
// Use next/image with explicit sizes (avoids generating unnecessary variants)
<Image src={url} width={800} height={600} sizes="(max-width: 768px) 100vw, 800px" />

// For user avatars: limit to specific sizes
// next.config.js
images: {
  imageSizes: [32, 64, 128],      // only these sizes generated
  deviceSizes: [640, 1280],        // not all possible device widths
}
```

**Reduce Build Minutes**
```yaml
# .github/workflows/deploy.yml
# Only trigger builds when relevant files change
on:
  push:
    paths:
      - 'src/**'
      - 'prisma/**'
      - 'package.json'
      - '!**.md'         # skip docs-only changes
```

---

## Part 2: AWS Cost Optimization

### Common Cost Drivers
```
1. EC2/ECS — over-provisioned instances running 24/7
2. RDS — multi-AZ + large instance for small workload
3. S3 + CloudFront — bandwidth and request charges
4. Lambda — invocation + duration (mostly cheap unless misused)
5. Data transfer — between AZs or to internet
6. NAT Gateway — can be expensive at scale
```

### Optimization Strategies

**Right-size EC2/ECS**
```bash
# Check CPU utilization (should be > 40% average)
aws cloudwatch get-metric-statistics \
  --namespace AWS/EC2 \
  --metric-name CPUUtilization \
  --period 86400 \
  --statistics Average \
  --start-time $(date -d '7 days ago' -u +%Y-%m-%dT%H:%M:%S) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%S)

# < 20% average CPU → downsize by one tier (e.g., t3.large → t3.medium saves ~50%)
```

**RDS Cost Reduction**
```
- Use Aurora Serverless v2 for dev/staging (scales to 0 when idle)
- For production: reserved instance (1-year commit = ~40% savings)
- Enable automated snapshots to S3 (cheaper than RDS automated backups)
- Move read replicas to smaller instance if read load is low
- Enable Performance Insights only in production (not dev)
```

**S3 Lifecycle Policies**
```json
{
  "Rules": [{
    "Id": "move-old-to-glacier",
    "Status": "Enabled",
    "Transitions": [
      { "Days": 90, "StorageClass": "STANDARD_IA" },
      { "Days": 365, "StorageClass": "GLACIER_IR" }
    ],
    "Expiration": { "Days": 2555 }
  }]
}
```

**Lambda Optimization**
```python
# Reduce cold starts: keep functions small, use Lambda SnapStart (Java)
# Right-size memory: 128MB → 256MB often halves duration (net cost neutral or cheaper)
# Use Graviton2 (arm64): same performance, 20% cheaper
# Configure: runtime: python3.12, architecture: arm64
```

---

## Part 3: AI Cost Optimization

### Token Cost Management
```
Claude claude-sonnet-4-6:  $3.00/M input, $15.00/M output
Claude claude-haiku-4-5:   $0.25/M input, $1.25/M output
GPT-4o:               $2.50/M input, $10.00/M output
GPT-4o-mini:          $0.15/M input, $0.60/M output

Rule: Use the cheapest model that produces acceptable quality for the task.
- Classification/extraction: Haiku / gpt-4o-mini
- Complex reasoning/writing: Sonnet / gpt-4o
- Never: use Opus/GPT-4 for tasks that Haiku can handle
```

**Prompt Caching (Anthropic — up to 90% cost reduction)**
```typescript
// Use cache_control on long system prompts that don't change
const response = await anthropic.messages.create({
  model: 'claude-sonnet-4-6',
  system: [
    {
      type: 'text',
      text: LONG_SYSTEM_PROMPT,  // 1000+ tokens
      cache_control: { type: 'ephemeral' }  // cached for 5 minutes
    }
  ],
  messages: [{ role: 'user', content: userMessage }]
})
// First call: full cost. Subsequent calls: 90% discount on cached tokens.
```

**Output Token Limits**
```typescript
// Always set max_tokens to the minimum needed
max_tokens: 500,   // for short answers (not 4096 by default)
max_tokens: 2048,  // for documents
max_tokens: 100,   // for classification tasks
```

**Model Routing**
```typescript
// Route to cheapest model based on task complexity
function selectModel(task: 'classify' | 'extract' | 'write' | 'reason'): string {
  const routing = {
    classify: 'claude-haiku-4-5-20251001',   // $0.25/M
    extract:  'claude-haiku-4-5-20251001',   // $0.25/M
    write:    'claude-sonnet-4-6',           // $3.00/M
    reason:   'claude-sonnet-4-6',           // $3.00/M
  }
  return routing[task]
}
```

---

## Cost Dashboard Queries

```bash
# AWS: current month cost by service
aws ce get-cost-and-usage \
  --time-period Start=$(date +%Y-%m-01),End=$(date +%Y-%m-%d) \
  --granularity MONTHLY \
  --metrics BlendedCost \
  --group-by Type=DIMENSION,Key=SERVICE

# Vercel: check bandwidth usage
# Dashboard → Settings → Usage → Bandwidth

# Azure: cost by resource group
az consumption usage list \
  --start-date $(date -d '30 days ago' +%Y-%m-%d) \
  --end-date $(date +%Y-%m-%d) \
  --query "[].{resource:instanceName, cost:pretaxCost}" \
  --output table | sort -k2 -rn | head -20
```

---

## Validation Checklist
- [ ] Baseline cost documented (this month's bill)
- [ ] Top 3 cost drivers identified
- [ ] Estimated savings per optimization
- [ ] Optimizations tested in staging first
- [ ] Budget alerts configured (alert at 80% and 100% of budget)
- [ ] Performance metrics unchanged after optimization
- [ ] AI model routing implemented (Haiku for simple tasks)
- [ ] Prompt caching enabled for long system prompts

---

## Final Response Format

```
## Cost Optimization Report

**Platform:** [Vercel / AWS / Azure / Multi-cloud]
**Current Monthly Cost:** $[X]
**Target Monthly Cost:** $[Y]
**Estimated Savings:** $[Z] ([X]%)

### Top Cost Drivers
1. [service] — $[cost]/mo — [optimization]

### Changes Made
| Change | File | Est. Savings |
|--------|------|-------------|
| [change] | [file] | $X/mo |

### Budget Alerts Configured
- 80% threshold: alert to [email/Slack]
- 100% threshold: alert to [email/Slack]

**Implementation Timeline:** [immediate / this week / this sprint]
```
