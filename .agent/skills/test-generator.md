# Skill: Test Generator

## Purpose
Generate unit tests, integration tests, and API tests for any function, service, or endpoint — covering the happy path, edge cases, and error paths — for any stack (TypeScript/Jest, Python/pytest, Java/JUnit).

## When to Use
- Adding tests to untested code
- Expanding coverage before a production release
- Test-driving a new feature (write tests first)
- After a bug fix (add regression test)
- CI pipeline requires coverage > threshold

---

## Universal Agent Rules (Follow for Every Task)
1. **Read code first** — read the function/service/route being tested before writing any test.
2. **Identify root cause** — identify what can go wrong in the code to write meaningful edge cases.
3. **Implement minimal safe fix** — write tests that cover the code as it IS, not as you wish it were.
4. **Run tests / typecheck / build** — all generated tests must pass on first run (no broken tests committed).
5. **Explain files changed** — list every new test file and the coverage it adds.

---

## Skill-Specific Rules
- Tests must test behavior, not implementation details. If you rename a private function, tests should not break.
- One assertion per test where possible — it's clearer what failed.
- Test names must describe the scenario: `it("returns 403 when user does not own the resume")`
- Mock only external dependencies (HTTP clients, DB, 3rd party APIs), not internal logic.
- Never mock the database in integration tests — use a real test DB or in-memory equivalent.
- Test files live next to the code they test: `resume.service.test.ts` beside `resume.service.ts`.

---

## Test Types

### Unit Tests
```
What: Test a single function in complete isolation.
Mock: everything external (DB, HTTP, filesystem)
Speed: < 100ms per test
When: business logic, utility functions, transformations, calculations
```

### Integration Tests
```
What: Test a service or module with real dependencies (real DB, real cache)
Mock: only external HTTP services (use test doubles/mock servers)
Speed: < 2 seconds per test
When: service layer, DB queries, auth flows
```

### API Tests (End-to-End)
```
What: Test the full HTTP request-response cycle
Mock: none (or only external payment/email providers)
Speed: < 5 seconds per test
When: all public API endpoints, critical user flows
Use: supertest (Node), TestClient (FastAPI), MockMvc (Spring Boot)
```

---

## Step-by-Step Workflow

### Step 1 — Read the Code
```
1. Read the function/route/service in full.
2. List all inputs and their possible values (null, empty, invalid, valid).
3. List all possible code paths (if/else branches, error throws).
4. List all external dependencies (DB calls, API calls, events).
5. List the expected outputs for each path.
```

### Step 2 — Design Test Cases
```
For every function, cover:
1. Happy path — valid input, expected output
2. Edge cases — null, undefined, empty string, 0, empty array
3. Error paths — what happens when DB throws, API returns 500
4. Business rule cases — specific domain rules that must always hold
5. Security cases — auth checks, authorization boundaries (IDOR)
```

### Step 3 — Write Tests

#### TypeScript / Jest / Next.js
```typescript
// resume.service.test.ts
import { describe, it, expect, vi, beforeEach } from 'vitest'
import { getResume } from './resume.service'
import { prisma } from '../lib/db'

vi.mock('../lib/db', () => ({
  prisma: { resume: { findUnique: vi.fn() } }
}))

describe('getResume', () => {
  beforeEach(() => vi.clearAllMocks())

  it('returns resume when user is owner', async () => {
    const mockResume = { id: '1', userId: 'user1', title: 'My Resume' }
    vi.mocked(prisma.resume.findUnique).mockResolvedValue(mockResume)

    const result = await getResume('1', 'user1')
    expect(result).toEqual(mockResume)
  })

  it('returns null when user is not owner', async () => {
    const mockResume = { id: '1', userId: 'user2', title: 'Other Resume' }
    vi.mocked(prisma.resume.findUnique).mockResolvedValue(mockResume)

    const result = await getResume('1', 'user1')
    expect(result).toBeNull()
  })

  it('returns null when resume does not exist', async () => {
    vi.mocked(prisma.resume.findUnique).mockResolvedValue(null)

    const result = await getResume('nonexistent', 'user1')
    expect(result).toBeNull()
  })

  it('throws when DB connection fails', async () => {
    vi.mocked(prisma.resume.findUnique).mockRejectedValue(new Error('DB down'))

    await expect(getResume('1', 'user1')).rejects.toThrow('DB down')
  })
})
```

#### API Test (Next.js / Supertest)
```typescript
// resume.api.test.ts
import { createMocks } from 'node-mocks-http'
import { GET } from '../app/api/resume/[id]/route'

describe('GET /api/resume/[id]', () => {
  it('returns 200 with resume for owner', async () => {
    const { req } = createMocks({ method: 'GET' })
    // setup session mock, prisma mock...
    const response = await GET(req, { params: { id: 'resume1' } })
    expect(response.status).toBe(200)
    const body = await response.json()
    expect(body.id).toBe('resume1')
  })

  it('returns 403 for non-owner', async () => {
    // setup different user session
    const response = await GET(req, { params: { id: 'resume1' } })
    expect(response.status).toBe(403)
  })

  it('returns 404 for missing resume', async () => {
    // setup null return from prisma
    const response = await GET(req, { params: { id: 'nonexistent' } })
    expect(response.status).toBe(404)
  })

  it('returns 401 when not authenticated', async () => {
    // no session
    const response = await GET(req, { params: { id: 'resume1' } })
    expect(response.status).toBe(401)
  })
})
```

#### Python / pytest
```python
# test_resume_service.py
import pytest
from unittest.mock import AsyncMock, patch
from app.services.resume_service import get_resume

@pytest.mark.asyncio
async def test_get_resume_returns_resume_for_owner():
    mock_resume = {"id": "1", "user_id": "user1", "title": "My Resume"}
    with patch("app.services.resume_service.db.resume.find_unique",
               return_value=mock_resume):
        result = await get_resume("1", "user1")
        assert result == mock_resume

@pytest.mark.asyncio
async def test_get_resume_returns_none_for_non_owner():
    mock_resume = {"id": "1", "user_id": "user2", "title": "Other"}
    with patch("app.services.resume_service.db.resume.find_unique",
               return_value=mock_resume):
        result = await get_resume("1", "user1")
        assert result is None

@pytest.mark.asyncio
async def test_get_resume_returns_none_when_not_found():
    with patch("app.services.resume_service.db.resume.find_unique",
               return_value=None):
        result = await get_resume("nonexistent", "user1")
        assert result is None
```

#### Java / Spring Boot / JUnit 5
```java
// ResumeServiceTest.java
@ExtendWith(MockitoExtension.class)
class ResumeServiceTest {

    @Mock private ResumeRepository resumeRepository;
    @InjectMocks private ResumeService resumeService;

    @Test
    void getResume_returnsResume_whenUserIsOwner() {
        Resume resume = new Resume("1", "user1", "My Resume");
        when(resumeRepository.findById("1")).thenReturn(Optional.of(resume));

        Resume result = resumeService.getResume("1", "user1");

        assertThat(result).isEqualTo(resume);
    }

    @Test
    void getResume_throwsForbidden_whenUserIsNotOwner() {
        Resume resume = new Resume("1", "user2", "Other Resume");
        when(resumeRepository.findById("1")).thenReturn(Optional.of(resume));

        assertThrows(ForbiddenException.class,
            () -> resumeService.getResume("1", "user1"));
    }

    @Test
    void getResume_throwsNotFound_whenResumeDoesNotExist() {
        when(resumeRepository.findById("nonexistent"))
            .thenReturn(Optional.empty());

        assertThrows(NotFoundException.class,
            () -> resumeService.getResume("nonexistent", "user1"));
    }
}
```

---

## Commands to Run

```bash
# Run tests (Node.js)
npm test -- --coverage --coverage-threshold='{"global":{"lines":80}}'

# Run specific test file
npx vitest run src/services/resume.service.test.ts

# Watch mode (TDD)
npx vitest --watch

# Python
pytest tests/ -x -v --cov=app --cov-report=term-missing

# Java / Spring Boot
./mvnw test -Dtest=ResumeServiceTest
```

---

## Test Coverage Targets by Layer

| Layer | Target | Reason |
|-------|--------|--------|
| Business logic (services) | ≥ 90% | Most likely to have bugs |
| API routes | ≥ 80% | Every endpoint needs a test |
| Utilities / helpers | ≥ 80% | Used everywhere |
| UI components | ≥ 60% | Rendering + interaction |
| DB migrations | Review only | Don't test SQL with SQL |

---

## Validation Checklist
- [ ] Every generated test passes on first run
- [ ] Happy path covered
- [ ] At least 2 edge cases covered per function
- [ ] Error/failure paths covered
- [ ] Security case tested (auth/IDOR) if applicable
- [ ] No mocking of internal business logic
- [ ] Test names describe the scenario clearly
- [ ] Tests are deterministic (no time-dependent assertions)

---

## Final Response Format

```
## Tests Generated

**File:** [path/to/test.file]
**Tests Written:** [n]
**Estimated Coverage Gain:** +X%

| Test | Type | Scenario |
|------|------|----------|
| it("...") | unit | Happy path |
| it("...") | unit | Edge case — null input |
| it("...") | api | 403 unauthorized |

**Run:** `npm test -- src/path/to/test.file`

**Coverage Before/After:**
- Before: X%
- After (estimated): Y%
```
