# Skill: Observability Engineer

## Purpose
Instrument any application with production-grade structured logging, distributed tracing, and application metrics — so you can diagnose issues, monitor performance, and set up alerts without guesswork.

## When to Use
- App is in production but you can't diagnose issues from logs
- Setting up observability for the first time before launch
- Incidents take too long to diagnose (no correlation IDs, no traces)
- Need to add performance monitoring (p50/p99 latency, error rate)
- Setting up alerts for critical SLIs (Service Level Indicators)

---

## Universal Agent Rules (Follow for Every Task)
1. **Read code first** — read existing logging patterns before adding more. Avoid duplicating.
2. **Identify root cause** — for observability gaps, identify the specific scenario you can't debug today.
3. **Implement minimal safe fix** — add instrumentation without changing business logic.
4. **Run tests / typecheck / build** — confirm observability additions don't break existing tests.
5. **Explain files changed** — list every file where logging, tracing, or metrics were added.

---

## Skill-Specific Rules
- All logs must be structured JSON. No `console.log("User created: " + userId)`.
- Never log PII (email, name, address) or secrets (tokens, passwords) in log messages.
- Every HTTP request must have a correlation ID (trace ID) in the response header.
- Error logs must include: error message, error code, affected resource, correlation ID.
- Metrics must be named consistently: `[service].[entity].[operation].[unit]` format.

---

## The Three Pillars

### Pillar 1: Structured Logging

#### Format (JSON)
```json
{
  "timestamp": "2026-05-21T14:30:00.000Z",
  "level": "info",
  "service": "resume-service",
  "traceId": "abc123",
  "userId": "user_01HX...",
  "action": "resume.save",
  "resourceId": "resume_01HX...",
  "duration_ms": 45,
  "message": "Resume saved successfully"
}
```

#### Log Levels
```
error  — unexpected failure, action failed, requires investigation
warn   — expected failure, handled gracefully, potentially degraded state
info   — significant business event (user created, payment succeeded)
debug  — detailed technical info for debugging (not in production by default)
```

#### What to Log
```
ALWAYS log:
- Request start: method, path, user ID (not PII), trace ID
- Business events: user created, payment processed, resume saved
- Errors: all caught exceptions with stack traces
- External API calls: provider, duration, status code
- Database operations > 500ms: query (sanitized), duration

NEVER log:
- Passwords, tokens, API keys
- Credit card numbers, SSNs
- Full email addresses (use hashed or truncated: m***@gmail.com)
- Request bodies containing user PII
```

#### Implementation

**Next.js / TypeScript (with Pino)**
```typescript
// lib/logger.ts
import pino from 'pino'

export const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  base: { service: process.env.SERVICE_NAME || 'app' },
  timestamp: pino.stdTimeFunctions.isoTime,
  redact: ['body.password', 'body.token', 'headers.authorization'],
})

// middleware/logging.ts — add to every request
export function withLogging(handler: NextApiHandler): NextApiHandler {
  return async (req, res) => {
    const traceId = req.headers['x-trace-id'] || crypto.randomUUID()
    const start = Date.now()

    res.setHeader('x-trace-id', traceId)

    try {
      await handler(req, res)
      logger.info({
        traceId,
        method: req.method,
        path: req.url,
        status: res.statusCode,
        duration_ms: Date.now() - start,
      })
    } catch (err) {
      logger.error({ traceId, err, path: req.url }, 'Request failed')
      throw err
    }
  }
}
```

**Spring Boot (with structured logging)**
```yaml
# application.yml
logging:
  pattern:
    console: '{"timestamp":"%d{ISO8601}","level":"%p","service":"user-service","traceId":"%X{traceId}","logger":"%c{1}","message":"%m"}%n'
  level:
    root: INFO
    com.yourapp: DEBUG
```

**Python / FastAPI**
```python
# lib/logger.py
import structlog

logger = structlog.get_logger()
structlog.configure(
    processors=[
        structlog.processors.TimeStamper(fmt="iso"),
        structlog.stdlib.add_log_level,
        structlog.processors.JSONRenderer()
    ]
)

# middleware
@app.middleware("http")
async def logging_middleware(request: Request, call_next):
    trace_id = request.headers.get("x-trace-id", str(uuid4()))
    start = time.time()
    response = await call_next(request)
    logger.info("request", trace_id=trace_id, path=request.url.path,
                method=request.method, status=response.status_code,
                duration_ms=int((time.time() - start) * 1000))
    return response
```

---

### Pillar 2: Distributed Tracing

#### Setup (OpenTelemetry — works with Jaeger, Datadog, Azure Monitor)

```typescript
// instrumentation.ts (Next.js)
import { NodeSDK } from '@opentelemetry/sdk-node'
import { getNodeAutoInstrumentations } from '@opentelemetry/auto-instrumentations-node'
import { OTLPTraceExporter } from '@opentelemetry/exporter-trace-otlp-http'

const sdk = new NodeSDK({
  traceExporter: new OTLPTraceExporter({
    url: process.env.OTEL_EXPORTER_OTLP_ENDPOINT,
  }),
  instrumentations: [getNodeAutoInstrumentations()],
})

sdk.start()
```

```yaml
# docker-compose.yml — local Jaeger for development
jaeger:
  image: jaegertracing/all-in-one:1.53
  ports:
    - "16686:16686"  # UI
    - "4318:4318"    # OTLP HTTP
```

---

### Pillar 3: Metrics

#### Key Metrics to Track

```
HTTP Metrics:
- http_request_duration_ms (histogram) — latency by route
- http_requests_total (counter) — request rate by status code
- http_request_errors_total (counter) — error rate

Business Metrics:
- users_created_total — signups per day
- ai_requests_total — AI API calls (by model, by user plan)
- ai_tokens_used_total — token consumption (for billing)
- resume_saved_total — core action metric

Infrastructure Metrics:
- db_query_duration_ms — database latency
- cache_hit_rate — Redis cache effectiveness
- external_api_duration_ms — third-party API latency
```

#### Prometheus + Grafana (self-hosted)
```typescript
// lib/metrics.ts
import { register, Counter, Histogram } from 'prom-client'

export const httpRequestDuration = new Histogram({
  name: 'http_request_duration_ms',
  help: 'HTTP request duration in milliseconds',
  labelNames: ['method', 'route', 'status'],
  buckets: [50, 100, 200, 500, 1000, 2000, 5000],
})

export const aiRequestsTotal = new Counter({
  name: 'ai_requests_total',
  help: 'Total AI API requests',
  labelNames: ['model', 'plan', 'status'],
})

// GET /metrics endpoint for Prometheus scraping
export async function metricsHandler() {
  return new Response(await register.metrics(), {
    headers: { 'Content-Type': register.contentType }
  })
}
```

---

## SLI/SLO Definitions

```
Service Level Indicators (what to measure):
- Availability: % of requests that return non-5xx
- Latency: p99 response time
- Error rate: % of requests resulting in error

Service Level Objectives (targets):
- Availability: 99.5% over 30 days (≤ 3.6 hours downtime/month)
- Latency: p99 < 2000ms
- Error rate: < 0.5%

Alert when:
- Error rate > 1% for 5 minutes → page on-call
- p99 latency > 3s for 5 minutes → alert team
- Availability < 99% for 1 hour → page on-call
```

---

## Commands to Run

```bash
# Install Pino logger (Node.js)
npm install pino pino-pretty

# Install OpenTelemetry (Node.js)
npm install @opentelemetry/sdk-node @opentelemetry/auto-instrumentations-node

# Install structlog (Python)
pip install structlog

# Check current logging patterns
grep -rn "console.log\|console.error\|logger\." src/ | wc -l

# Find unstructured logs (should be 0)
grep -rn 'console\.log("[^{]' src/ | head -10

# Check for PII in logs
grep -rn "email\|password\|token" src/ | grep "console\|logger" | head -10
```

---

## Validation Checklist
- [ ] All logs are structured JSON (no string concatenation)
- [ ] No PII in any log message
- [ ] Every HTTP request has a correlation/trace ID
- [ ] Error logs include: message, code, resource ID, trace ID
- [ ] Health endpoint exists at /health or /actuator/health
- [ ] p99 latency metric instrumented on critical routes
- [ ] Business metrics tracked (signups, core actions)
- [ ] Alerts defined for: error rate, latency, availability
- [ ] Logs go to stdout (not files) in production

---

## Final Response Format

```
## Observability Setup

**Pillars Implemented:** Logging / Tracing / Metrics

### Logging
- Format: Structured JSON with Pino/structlog
- Files: [list files modified]
- PII redaction: [fields redacted]

### Tracing
- Provider: OpenTelemetry → [Jaeger / Datadog / Azure Monitor]
- Trace propagation: x-trace-id header on all requests
- Files: [list files modified]

### Metrics
- Key metrics added: [list]
- Scraping endpoint: /metrics
- Dashboard: [Grafana / Datadog / CloudWatch URL]

### Alerts Configured
- [alert name] — [condition] — [notification channel]

**Files Changed:**
- [file] — [what was added]
```
