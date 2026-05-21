# Skill: Documentation Generator

## Purpose
Generate production-quality technical documentation — README files, API references, architecture docs, onboarding guides, and multilingual user documentation — from the actual source code and project structure.

## When to Use
- Project has no README or an outdated one
- Setting up a public API that developers will consume
- Onboarding a new team member
- Generating multilingual user-facing docs (EN, HI, FR, etc.)
- Creating architecture docs before a design review

---

## Universal Agent Rules (Follow for Every Task)
1. **Read code first** — read the actual source, endpoints, and types before writing any docs.
2. **Identify root cause** — for missing docs, identify the highest-impact gap first (README > API > architecture).
3. **Implement minimal safe fix** — generate docs for what exists today; don't document future plans as if they're real.
4. **Run tests / typecheck / build** — API docs must be validated against actual route signatures.
5. **Explain files changed** — list every doc file created or updated.

---

## Skill-Specific Rules
- Docs are code — they must be accurate, or they cause more harm than no docs.
- Never document internal implementation details in public-facing docs.
- Every code example in docs must be copy-paste-runnable.
- Multilingual docs: translate meaning, not words (context-aware translation).
- Docs live with the code they describe — co-locate, don't separate.

---

## Documentation Types

### 1. README.md (project entry point)
```markdown
# Project Name

One-sentence description.

## Quick Start
[3 commands to get it running]

## What It Does
[2 paragraphs, non-technical]

## Tech Stack
[simple list with versions]

## Development
[how to run locally, env setup]

## Deployment
[how to deploy, where it lives]

## Contributing
[how to contribute, PR process]
```

### 2. API Reference
```markdown
## POST /api/resume/tailor

Tailors a resume's bullet points to match a job description using Claude AI.

### Authentication
Bearer token required (NextAuth session).

### Request
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| resumeId | string | Yes | UUID of the resume to tailor |
| jobDescription | string | Yes | Full job posting text (max 10,000 chars) |

### Response (200 OK)
| Field | Type | Description |
|-------|------|-------------|
| suggestions | Suggestion[] | List of rewrite suggestions |
| tokensUsed | number | AI tokens consumed |

### Suggestion Object
| Field | Type | Description |
|-------|------|-------------|
| sectionId | string | ID of the section being rewritten |
| original | string | Original bullet point |
| suggested | string | AI-suggested rewrite |
| reasoning | string | Why this change improves matching |

### Error Responses
| Status | Code | Meaning |
|--------|------|---------|
| 401 | UNAUTHORIZED | No valid session |
| 404 | NOT_FOUND | Resume not found |
| 429 | RATE_LIMITED | Too many AI requests |
| 503 | AI_UNAVAILABLE | Anthropic API unavailable |

### Example
```bash
curl -X POST https://api.globalresumeai.com/api/resume/tailor \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"resumeId": "res_01HX...", "jobDescription": "We are looking for..."}'
```
```

### 3. Architecture Document
```markdown
# Architecture Overview

## System Diagram
[Mermaid diagram]

## Components
| Component | Technology | Purpose |
|-----------|-----------|---------|
| Frontend | Next.js 15 | SSR + App Router |
| API Layer | Next.js Route Handlers | REST API |
| Database | PostgreSQL (Supabase) | Primary data store |
| AI | Anthropic Claude | Resume tailoring |
| Cache | Upstash Redis | Rate limiting, session |
| Storage | Vercel Blob | Resume exports (PDF) |

## Key Design Decisions
| Decision | Choice | Reason |
|----------|--------|--------|
| ORM | Prisma | Type safety, migrations |
| Auth | NextAuth | Google/GitHub OAuth + Credentials |
| State | TanStack Query | Server state management |
```

### 4. Onboarding Guide (for new developers)
```markdown
# Developer Onboarding

## Day 1 Checklist
- [ ] Clone repo and run locally
- [ ] Read this architecture overview
- [ ] Run existing tests
- [ ] Make a trivial change and open a PR

## Local Setup (10 minutes)
[step by step with exact commands]

## How the Codebase Is Organized
[directory structure with explanations]

## Key Patterns Used
[2-3 most important patterns with examples]

## Where to Ask for Help
[team contact, docs links]
```

---

## Multilingual Documentation

### Supported Languages
```
en — English (default)
hi — Hindi (for Indian user-facing apps)
fr — French
de — German
es — Spanish
ja — Japanese
zh — Simplified Chinese
```

### Multilingual Strategy
```
1. Write master doc in English first.
2. Identify which sections need translation:
   - User-facing: EVERYTHING
   - API reference: parameter names stay in English; descriptions translated
   - Code comments: keep in English
3. Translation principles:
   - Translate meaning, not words
   - Use local idioms where natural
   - Keep technical terms (API, JSON, OAuth) in English
   - Use formal language for professional tools (not casual)
```

### Hindi Documentation Example (GlobalResumeAI)
```markdown
# GlobalResumeAI — त्वरित शुरुआत

## यह क्या करता है?
GlobalResumeAI आपके resume को job description के हिसाब से automatically customize करता है।
AI आपके bullet points को rewrite करके keywords match करता है।

## शुरू करें
1. अपना Google account से login करें
2. अपना resume upload करें या नया बनाएं
3. Job description paste करें
4. "AI से Tailor करें" button क्लिक करें

## AI Suggestions का उपयोग कैसे करें
- हरे checkmark (✓) से suggestion accept करें
- लाल X (✗) से reject करें
- अपने words में edit करने के लिए ✏️ icon click करें
```

---

## Step-by-Step Workflow

### Step 1 — Audit Current State
```bash
# Check what docs already exist
find . -name "*.md" -not -path "*/node_modules/*" | head -20

# Check README freshness
git log --oneline README.md | head -5  # when was it last updated?

# Check API routes (to document)
find src/app/api -name "route.ts" | head -30
```

### Step 2 — Prioritize What to Write
```
Priority order:
1. README.md — gets read first by everyone
2. API reference — needed for any integration
3. Onboarding guide — needed when team grows
4. Architecture doc — needed before design reviews
5. Multilingual user docs — needed for non-English users
```

### Step 3 — Generate from Source
```
For API docs:
1. Read each src/app/api/*/route.ts
2. Extract: method, path, auth required, params, response shape
3. Write Markdown table for request + response
4. Add a working curl example

For README:
1. Read package.json (name, description, dependencies)
2. Read the entry point (app/page.tsx or server.ts)
3. Read .env.example (what env vars needed)
4. Read existing CI/CD config (.github/workflows)
```

---

## Commands to Run

```bash
# Generate API docs from TypeScript types (if using tRPC or OpenAPI)
npx trpc-openapi generate

# Generate from Swagger annotations (Spring Boot)
./mvnw spring-boot:run  # then visit /swagger-ui.html

# FastAPI auto-docs
# Already built-in at /docs and /redoc

# Extract all API routes (Next.js)
find src/app/api -name "route.ts" -exec grep -l "export async function" {} \;

# Check doc coverage (% of routes documented)
# Manual: count route files vs documented endpoints
```

---

## Validation Checklist
- [ ] README has a < 1-minute Quick Start section
- [ ] Every public API endpoint is documented
- [ ] Every code example is copy-paste runnable
- [ ] Architecture diagram exists (Mermaid or image)
- [ ] Onboarding guide includes local setup steps
- [ ] No future features documented as if they exist
- [ ] Multilingual docs use translated meaning (not word-for-word)
- [ ] Docs are co-located with the code they describe

---

## Final Response Format

```
## Documentation Generated

**Documents Created/Updated:**
- [filename] — [doc type] — [language]

**API Coverage:**
- Routes documented: [n/total]
- Request shapes: [complete / partial]
- Error codes: [complete / partial]

**Multilingual:**
- Languages: [list]
- Sections translated: [list]

**Files Changed:**
- [file] — [what was written]

**Estimated Reading Time:**
- README: [Xm]
- API Reference: [Xm]
- Architecture: [Xm]
```
