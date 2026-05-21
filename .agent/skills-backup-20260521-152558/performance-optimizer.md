# Skill: Performance Optimizer

## Purpose
Identify and fix performance bottlenecks across frontend (load time, rendering), backend (response time, throughput), and database (query speed, connection efficiency) for any tech stack.

## When to Use
- Pages are slow to load (LCP > 2.5s)
- API endpoints are taking > 500ms
- Database queries are slow (> 100ms)
- High memory or CPU usage in production
- App feels sluggish or unresponsive under load

---

## Rules
- Measure before you optimize. Never optimize by intuition alone.
- Fix the biggest bottleneck first (Amdahl's Law).
- Every optimization must have a before/after measurement.
- Do not introduce complexity for micro-optimizations (< 5% gain).
- Caching is a last resort after the underlying operation is optimized.

---

## Step-by-Step Workflow

### Step 1 — Measure & Baseline
```
1. Record current performance metrics before touching anything:
   - Frontend: LCP, FID/INP, CLS via Lighthouse or WebPageTest
   - API: average and p99 response time via logs or APM
   - DB: slow query log or EXPLAIN ANALYZE on key queries
2. Identify the top 3 slowest operations — focus only on those.
3. Profile, don't guess: use flame graphs, query analyzers, bundle analyzers.
```

### Step 2 — Frontend Performance

#### Bundle Optimization
```
1. Run bundle analyzer: ANALYZE=true npm run build
2. Find packages > 100KB — check if tree-shakeable or replaceable.
3. Lazy-load routes and heavy components: React.lazy + Suspense.
4. Split vendor chunk from app chunk in webpack/vite config.
5. Remove unused dependencies: npx depcheck.
```

#### Rendering Optimization
```
1. Check for unnecessary re-renders: React DevTools Profiler.
2. Memoize expensive computations: useMemo (for computed values), useCallback (for stable function refs passed to children).
3. Virtualize long lists: react-window or @tanstack/virtual.
4. Move static data out of components (defined outside the component function).
5. Use React.memo only on components that render frequently with stable props.
```

#### Asset Optimization
```
1. Images: use next/image or lazy loading. Convert PNG/JPG to WebP.
2. Fonts: use font-display: swap. Preload critical fonts.
3. Static assets: confirm CDN caching with Cache-Control: max-age=31536000, immutable.
4. CSS: remove unused styles (PurgeCSS for Tailwind projects).
```

### Step 3 — API / Backend Performance

#### Response Time
```
1. Identify endpoints with p99 > 500ms via logs.
2. Add timing instrumentation to pinpoint: DB, external API, or compute.
3. Async everything: don't block the thread on I/O.
4. Use streaming for large responses instead of buffering.
5. Implement response compression (gzip/brotli) at server or reverse proxy level.
```

#### Caching Strategy
```
1. In-memory cache (Redis/Memcached) for: session data, hot lookup tables, rate limit counters.
2. HTTP caching: set Cache-Control on public, semi-static API responses.
3. Query result caching: cache expensive DB query results with appropriate TTL.
4. Invalidate on write — don't serve stale data for user-specific resources.
```

#### Concurrency
```
1. Use connection pooling for DB (pgBouncer, HikariCP, SQLAlchemy pool).
2. Process independent operations concurrently: Promise.all, asyncio.gather, CompletableFuture.
3. Move CPU-heavy work to worker threads / background jobs.
4. Set appropriate thread pool sizes for your workload.
```

### Step 4 — Database Performance

#### Query Optimization
```
1. Run EXPLAIN ANALYZE on every slow query.
2. Add indexes on: foreign keys, columns in WHERE, ORDER BY, GROUP BY clauses.
3. Fix N+1 queries: use JOIN or include/eager-load relations.
4. Use projections: SELECT only the columns you need, not SELECT *.
5. Paginate all list queries: LIMIT + OFFSET or cursor-based.
6. Avoid full table scans: check for missing WHERE clauses.
```

#### Connection & Pool Health
```
1. Verify pool size matches your workload (start with 10, tune up).
2. Check for connection leaks: connections that open and never close.
3. Use read replicas for reporting/analytics queries.
4. Partition large tables by date if > 100M rows.
```

#### Schema Optimization
```
1. Store enums as enum type or smallint, not varchar.
2. Use appropriate column types (int vs bigint, text vs varchar(n)).
3. Archive or delete old data — don't let tables grow unbounded.
```

---

## Commands to Run

```bash
# Frontend bundle analysis (Next.js)
ANALYZE=true npm run build

# Lighthouse CI
npx lighthouse https://localhost:3000 --output=json --output-path=./lighthouse.json

# Find N+1 queries (look for repeated similar queries in logs)
# Enable query logging in Prisma:
# DEBUG="prisma:query" npm start

# PostgreSQL slow query analysis
# In psql: EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT) SELECT ...;

# Check DB indexes
# SELECT indexname, indexdef FROM pg_indexes WHERE tablename = 'your_table';

# Node.js heap profiling
node --inspect server.js
# Then connect Chrome DevTools → Memory → Take heap snapshot

# Check API response times
ab -n 1000 -c 10 http://localhost:3000/api/endpoint
wrk -t12 -c400 -d30s http://localhost:3000/api/endpoint
```

---

## Validation Checklist
- [ ] Lighthouse performance score ≥ 90 on key pages
- [ ] LCP < 2.5s, CLS < 0.1, INP < 200ms
- [ ] No single JS bundle > 250KB (gzipped)
- [ ] API p99 response time < 500ms
- [ ] No N+1 queries in ORM usage
- [ ] Database indexes exist for all frequent filter columns
- [ ] Long lists are virtualized or paginated
- [ ] Images are served in WebP with lazy loading
- [ ] Redis/caching in place for hot data
- [ ] Connection pooling configured for DB

---

## Final Response Format

```
## Performance Optimization Report

### Baseline Metrics
| Metric | Before | After | Improvement |
|--------|--------|-------|-------------|
| LCP | Xs | Xs | -X% |
| Bundle size | XKB | XKB | -X% |
| API p99 | Xms | Xms | -X% |
| DB query | Xms | Xms | -X% |

### Changes Made
1. [change] — [file] — [X% improvement]
2. [change] — [file] — [X% improvement]

### Remaining Bottlenecks
- [bottleneck] — [estimated effort to fix] — [priority]

### Recommended Next Steps
1. [action]
```
