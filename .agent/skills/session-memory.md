# Skill: Session Memory

## Purpose
Persist project context, architectural decisions, known issues, in-progress work, and team preferences between agent sessions — so every new conversation starts with full awareness of the project's state.

## When to Use
- Starting a new Claude Code session on an existing project
- After a gap of days/weeks returning to a project
- Before handing off a project to another developer
- When context about past decisions needs to be preserved
- Explicitly asked to "remember" something for future sessions

---

## Universal Agent Rules (Follow for Every Task)
1. **Read code first** — read `.agent/memory/` before proposing anything.
2. **Identify root cause** — for stale memories, verify against current code before applying.
3. **Implement minimal safe fix** — save only non-obvious facts; never duplicate what's in code.
4. **Run tests / typecheck / build** — not applicable; validate memories against current project state.
5. **Explain files changed** — list every memory file created or updated with a reason.

---

## Skill-Specific Rules
- Never store information derivable from reading the code (file paths, function names, schema).
- Never store secrets, credentials, API keys, or tokens in memory.
- Memory decays — always verify facts against the current codebase before acting on them.
- One memory file per topic. Update existing files; do not duplicate.
- If a memory contradicts current code, trust the code and update the memory.

---

## Memory File Structure

```
.agent/
└── memory/
    ├── MEMORY-INDEX.md           ← index of all memories (auto-generated)
    ├── architecture.md           ← system design decisions + ADRs
    ├── known-issues.md           ← bugs in progress, workarounds in place
    ├── progress.md               ← in-progress features, completed milestones
    ├── preferences.md            ← team coding conventions, agent behavior preferences
    ├── integrations.md           ← external APIs, services, credentials location
    └── project-context.md        ← tech stack, team, deploy targets, project goals
```

---

## Step-by-Step Workflow

### Step 1 — Session Start (Restore Context)
```
1. Read .agent/memory/MEMORY-INDEX.md — what memories exist?
2. Read .agent/memory/project-context.md — stack, goals, deploy targets.
3. Read .agent/memory/known-issues.md — what's broken or in-progress?
4. Read .agent/memory/progress.md — what was last being worked on?
5. Cross-check: verify key facts in memories against current code state.
6. Report: "I've loaded context. Here's what I know about this project: ..."
```

### Step 2 — During Session (Capture Decisions)
```
Capture to memory when:
- An architectural decision is made ("we chose X over Y because...")
- A non-obvious workaround is implemented
- A known issue is discovered but not yet fixed
- A preference is stated ("always use X pattern in this project")
- An integration is added or configured
- A milestone is reached or a feature is completed
```

### Step 3 — Session End (Save State)
```
1. Update .agent/memory/progress.md — what was completed, what's next
2. Update .agent/memory/known-issues.md — new bugs found, issues closed
3. Update .agent/memory/architecture.md — any new ADRs
4. Regenerate MEMORY-INDEX.md with all current memory files
5. Report: "Session saved. Next session will resume from: [state summary]"
```

---

## Memory File Templates

### MEMORY-INDEX.md
```markdown
# Memory Index — [Project Name]
Last updated: [date]

| File | Topic | Last Updated |
|------|-------|-------------|
| architecture.md | System design decisions | [date] |
| known-issues.md | Active bugs + workarounds | [date] |
| progress.md | Feature status | [date] |
| preferences.md | Team conventions | [date] |
| integrations.md | External services | [date] |
| project-context.md | Stack + goals | [date] |
```

### architecture.md
```markdown
# Architecture Decisions

## ADR-001: [Decision Title]
**Date:** [date]
**Status:** Accepted
**Context:** [why the decision was needed]
**Decision:** [what was decided]
**Consequences:** [trade-offs]

## Known Anti-Patterns (do not repeat)
- [pattern] — [why it failed]
```

### known-issues.md
```markdown
# Known Issues

## Active Bugs
| ID | Description | Severity | File | Status |
|----|-------------|----------|------|--------|
| B001 | [description] | High | [file:line] | In Progress |

## Active Workarounds
- [workaround description] — [file where it lives] — [when to remove it]

## Resolved (keep for 30 days)
| ID | Description | Fixed In | Date |
|----|-------------|----------|------|
```

### progress.md
```markdown
# Project Progress

## Current Sprint / Active Work
- [feature] — [status: In Progress / Blocked / Review] — [owner]

## Completed Milestones
- [date] — [milestone description]

## Next Up
1. [next priority task]
2. [second priority]

## Blocked Items
- [blocker] — [what's needed to unblock]
```

### preferences.md
```markdown
# Team Preferences & Conventions

## Code Style
- [convention] — [reason]

## Agent Behavior
- [what the agent should always/never do in this project]

## Review Standards
- [PR standards, commit message format, etc.]
```

---

## Commands to Run

```bash
# Initialize memory for a new project
mkdir -p .agent/memory

# View current memory state
ls -la .agent/memory/

# Check memory freshness (how old are files)
find .agent/memory -name "*.md" -exec ls -lt {} + | head -10

# Grep for a specific decision or fact
grep -r "decided\|chose\|because\|workaround" .agent/memory/
```

---

## Validation Checklist
- [ ] MEMORY-INDEX.md exists and lists all memory files
- [ ] project-context.md has stack, team, deploy target
- [ ] known-issues.md has current active bugs
- [ ] progress.md reflects current state (not stale)
- [ ] No secrets in any memory file
- [ ] All architecture decisions have a reason ("because...")
- [ ] Memories verified against current code (not stale)

---

## Final Response Format

```
## Session Memory Report

**Action:** [Session Start / Session End / Memory Update]
**Project:** [project name]

**Loaded Context:**
- Architecture: [key decisions loaded]
- Known Issues: [active bugs]
- In Progress: [active work]

**Saved This Session:**
- [memory file] — [what was saved]

**Next Session Resume Point:**
[one sentence — where to pick up from]
```
