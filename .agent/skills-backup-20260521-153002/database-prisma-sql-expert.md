# Skill: Database / Prisma / SQL Expert

## Purpose
Design, migrate, query, and optimize relational databases (PostgreSQL, MySQL, SQLite) using raw SQL or an ORM (Prisma, TypeORM, SQLAlchemy, Hibernate). Fix schema issues, write efficient queries, and build safe migrations.

## When to Use
- Designing or modifying a database schema
- Writing complex queries (joins, aggregations, CTEs, window functions)
- Fixing slow queries or adding indexes
- Writing Prisma schema and migrations
- Seeding a database or writing data scripts
- Resolving migration conflicts or drift

---

## Universal Agent Rules (Follow for Every Task)
1. **Read code first** — read existing Prisma schema, migration history, and query patterns before changes.
2. **Identify root cause** — for slow queries, run EXPLAIN ANALYZE and read the output before adding indexes.
3. **Implement minimal safe fix** — add only required columns/indexes. No schema restructuring without explicit request.
4. **Run tests / typecheck / build** — run `prisma migrate dev` and `prisma generate` and confirm no errors.
5. **Explain files changed** — list every migration file and schema change with its purpose.

---

## Skill-Specific Rules
- Every schema change must go through a migration file — never alter production DB by hand.
- All migrations must be reversible (include down migration) unless data destruction is intentional and confirmed.
- Never use SELECT * in application queries — always project needed columns.
- Always use parameterized queries / ORM — never string-concatenate SQL from user input.
- Foreign keys must have explicit ON DELETE behavior (CASCADE, RESTRICT, SET NULL).
- Soft deletes need a `deletedAt` column, not `isDeleted` boolean.

---

## Step-by-Step Workflow

### Step 1 — Schema Design
```
1. Identify entities, their attributes, and cardinality of relationships.
2. Normalize to 3NF: remove repeating groups, eliminate transitive dependencies.
3. Add standard audit fields to every table:
   - id (UUID or auto-increment bigint)
   - createdAt TIMESTAMP DEFAULT NOW()
   - updatedAt TIMESTAMP DEFAULT NOW() (trigger or ORM)
   - deletedAt TIMESTAMP NULL (if soft delete needed)
4. Add appropriate constraints: NOT NULL, UNIQUE, CHECK, FK.
5. Choose index strategy: see Step 4.
```

### Step 2 — Prisma Schema Patterns

#### Naming Convention
```prisma
// Models: PascalCase singular
// Fields: camelCase
// Relations: use explicit foreign key + relation name

model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String?
  posts     Post[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

model Post {
  id        String   @id @default(cuid())
  title     String
  content   String?
  published Boolean  @default(false)
  authorId  String
  author    User     @relation(fields: [authorId], references: [id], onDelete: Cascade)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([authorId])
  @@index([published, createdAt(sort: Desc)])
}
```

#### Enums in Prisma
```prisma
enum Role {
  USER
  ADMIN
  MODERATOR
}

model User {
  role Role @default(USER)
}
```

### Step 3 — Migration Workflow

#### Prisma Migrations
```bash
# Create new migration (development)
npx prisma migrate dev --name add_user_role

# Apply migrations in production
npx prisma migrate deploy

# Check migration status
npx prisma migrate status

# Reset dev database (dangerous — dev only)
npx prisma migrate reset

# Generate Prisma client after schema change
npx prisma generate
```

#### Raw SQL Migration Template
```sql
-- up.sql
BEGIN;

ALTER TABLE users ADD COLUMN role VARCHAR(20) NOT NULL DEFAULT 'USER';
CREATE INDEX idx_users_role ON users(role);

COMMIT;

-- down.sql
BEGIN;

DROP INDEX IF EXISTS idx_users_role;
ALTER TABLE users DROP COLUMN role;

COMMIT;
```

### Step 4 — Indexing Strategy
```
Always index:
- Primary keys (automatic)
- Foreign keys (Prisma does NOT auto-create — add @@index manually)
- Columns used in WHERE clauses with high cardinality
- Columns used in ORDER BY on large tables
- Composite index for common multi-column WHERE + ORDER BY patterns

Never index:
- Low-cardinality columns (boolean, status with 2-3 values) unless part of composite
- Tables with < 1000 rows (sequential scan is faster)
- Columns rarely used in queries
```

### Step 5 — Query Patterns

#### Efficient Pagination (Cursor-based)
```typescript
// Cursor-based (scalable for large datasets)
const posts = await prisma.post.findMany({
  take: 20,
  skip: cursor ? 1 : 0,
  cursor: cursor ? { id: cursor } : undefined,
  orderBy: { createdAt: 'desc' },
})

// Offset-based (simple, fine for admin/small datasets)
const posts = await prisma.post.findMany({
  skip: (page - 1) * pageSize,
  take: pageSize,
})
```

#### Avoiding N+1
```typescript
// BAD — N+1 query
const users = await prisma.user.findMany()
for (const user of users) {
  const posts = await prisma.post.findMany({ where: { authorId: user.id } })
}

// GOOD — single query with include
const users = await prisma.user.findMany({
  include: { posts: { select: { id: true, title: true } } }
})
```

#### Aggregations
```typescript
// Count with groupBy
const stats = await prisma.post.groupBy({
  by: ['authorId'],
  _count: { id: true },
  having: { id: { _count: { gt: 5 } } },
})

// Raw SQL for complex aggregations
const result = await prisma.$queryRaw`
  SELECT 
    date_trunc('day', created_at) AS day,
    COUNT(*) AS count
  FROM posts
  WHERE created_at >= NOW() - INTERVAL '30 days'
  GROUP BY 1
  ORDER BY 1
`
```

#### Transactions
```typescript
// Use $transaction for atomic multi-step operations
const [user, profile] = await prisma.$transaction([
  prisma.user.create({ data: { email } }),
  prisma.profile.create({ data: { bio } }),
])

// Interactive transaction (for conditional logic)
await prisma.$transaction(async (tx) => {
  const account = await tx.account.findUniqueOrThrow({ where: { id } })
  if (account.balance < amount) throw new Error('Insufficient funds')
  await tx.account.update({ where: { id }, data: { balance: { decrement: amount } } })
})
```

---

## Commands to Run

```bash
# Introspect existing DB (generate schema from existing DB)
npx prisma db pull

# Open Prisma Studio (DB GUI)
npx prisma studio

# Format Prisma schema
npx prisma format

# Validate schema
npx prisma validate

# Check for slow queries (PostgreSQL)
# In psql:
# SELECT query, mean_exec_time, calls FROM pg_stat_statements ORDER BY mean_exec_time DESC LIMIT 10;

# Explain a query
# EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT) SELECT ...;

# Check indexes
# SELECT indexname, indexdef FROM pg_indexes WHERE tablename = 'users';

# Check table sizes
# SELECT relname, pg_size_pretty(pg_total_relation_size(relid)) FROM pg_catalog.pg_statio_user_tables ORDER BY pg_total_relation_size(relid) DESC;
```

---

## Validation Checklist
- [ ] Schema is normalized (no repeating groups, no transitive dependencies)
- [ ] Every FK has a corresponding index (@@index in Prisma)
- [ ] All list queries have LIMIT/pagination
- [ ] No N+1 queries (use include/join)
- [ ] All migrations are in version control
- [ ] Migrations are reversible (down migration exists)
- [ ] No raw string SQL from user input
- [ ] Transactions used for multi-step atomic operations
- [ ] Soft deletes use deletedAt column
- [ ] Sensitive fields (passwords) are not in the DB as plaintext

---

## Final Response Format

```
## Database Changes Summary

**Schema Changes:**
- [table] — [what changed] — [reason]

**Migrations Created:**
- [migration_name] — [what it does]

**Indexes Added:**
- [table].[column] — [query pattern it supports]

**Query Optimizations:**
- [query] — [before: Xms] — [after: Xms] — [what was changed]

**Data Safety Notes:**
- [anything that could cause data loss or lock tables]
```
