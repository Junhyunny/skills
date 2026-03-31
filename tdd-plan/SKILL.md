---
name: tdd-plan
description: >
  First step of a ping-pong TDD session. Fetches a story from TrackerBoot MCP
  or accepts pasted story content, detects the project stack and conventions,
  collaboratively plans a task breakdown, and writes a .tdd-session.md file
  that /tdd-task and /tdd-commit will use.
version: 1.0.0
category: engineering
tags: [tdd, pair-programming, planning, ping-pong, agile]
triggers:
  - tdd plan
  - plan tdd
  - start tdd session
---

# TDD Plan

## Invocation

```
/tdd-plan [story-id | "full story content"]
```

- **TrackerBoot story ID** (numeric, e.g. `12345678` or `#12345678`) → fetch via MCP
- **Pasted story text** → use directly
- **No argument** → display error and stop: "A story is required. Please provide a TrackerBoot story ID or paste the story content."

---

## Step 1: Load Story

### If story ID provided

Scan available MCP tools for TrackerBoot patterns:
```
mcp__trackerboot__get_story
mcp__trackerboot__fetch_story
mcp__trackerboot__story
mcp__pivotal__get_story
mcp__tracker__get_story
```

Strip a leading `#` before calling (`#12345678` → `12345678`).

Extract from response:
- **Title:** `name` field
- **Description:** `description` field
- **Acceptance Criteria:** `tasks[]` array (preferred) or parse "Acceptance Criteria:" section in description
- **Status:** `current_state` field

**If MCP tool not found or call fails:**
```
"Could not fetch story [ID] from TrackerBoot.
Please check that the TrackerBoot MCP server is configured, then try again,
or paste the story content directly."
```
**STOP — do not proceed.**

### If story content pasted

Parse directly — extract title, description, acceptance criteria from the text.

### Confirm with developer

Display the story and PAUSE:

```markdown
## Story

**ID:** [id or "—"]
**Title:** [title]

**Description:**
[description]

**Acceptance Criteria:**
- [ ] [AC 1]
- [ ] [AC 2]
...

---
Does this look correct? Type "ok" to continue, or let me know what to fix.
```

Wait for "ok" / "yes" / "looks good". Apply any corrections and re-display if requested.

---

## Step 2: Detect Tech Stack

Read project root files and apply these rules in order:

| File present | Stack |
|-------------|-------|
| `package.json` with `"react"` in deps | TypeScript + React |
| `build.gradle.kts` with kotlin plugin | Kotlin + Spring |
| `build.gradle` (not .kts) + `src/main/java/` | Java + Spring |
| `pom.xml` + `src/main/java/` | Java + Spring (Maven) |
| `pyproject.toml` or `requirements.txt` with `fastapi` | Python + FastAPI |

For TypeScript, also check devDependencies:
- `"vitest"` → unit FW: Vitest
- `"jest"` → unit FW: Jest
- `"@playwright/test"` or `playwright.config.*` → E2E: Playwright
- `"cypress"` → E2E: Cypress

If ambiguous: "Detected candidates: [list]. Which stack are we working with?"
If nothing detected: "Could not detect the stack. Please specify: typescript-react / kotlin-spring / java-spring / python-fastapi"

For full detection rules see `.agents/skills/ping-pong-tdd/references/tech-stack-detection.md`.

---

## Step 3: Detect Project Conventions

**Skip this step if no existing test or source files are found (new project).**

Read 2–3 test files + 1–2 source files from the same area as the story. Prefer recently modified files.

Extract:
- Test file location (co-located vs `test/` directory)
- Test file naming suffix (`*.test.ts`, `*Test.kt`, `test_*.py`)
- Test structure (describe/it nesting, `@Nested`, class-based)
- Assertion library and style
- Mock pattern (vi.mock, vi.fn injection, mockk, @MockBean, MagicMock)
- Dependency injection pattern in source files
- Naming conventions (suffixes, casing)

Display a brief summary (no PAUSE — informational only):

```markdown
## Project Conventions Detected

Files read: [file list]

- **Test location:** [co-located / test/ directory]
- **Test naming:** [pattern]
- **Test structure:** [describe/it / @Nested / class-based]
- **Assertions:** [expect().toEqual / assertThat / assert ==]
- **Mocks:** [vi.mock / vi.fn injection / mockk / MagicMock]
- **Source structure:** [feature-based / layer-based]

All new code will follow these patterns.
```

For full convention rules see `.agents/skills/ping-pong-tdd/references/convention-detection.md`.

---

## Step 4: Propose Task Breakdown

Display session state, then propose tasks:

```markdown
---
## 🏓 Session State
| Field | Value |
|-------|-------|
| Story | [ID]: [title] |
| Stack | [stack] |
| Test FW | [test framework] |
| E2E FW | [e2e framework] |
| Conventions | [new project / existing — key patterns] |
| Phase | PLANNING |
---

## Task Plan (Draft)

### Task List

#### Task 1: [title]
- **Type:** unit | integration | e2e
- **The test will assert:** [one sentence]
- **Implementation scope:** [what code will be written]
- **Acceptance criteria link:** [which ACs]
- **Dependencies:** none

#### Task 2: [title]
...

#### Task [N] (E2E): [title]
- **Type:** e2e
- **The test will assert:** [full user flow]
- **Implementation scope:** E2E test only
- **Acceptance criteria link:** Full AC coverage check
- **Dependencies:** Task [N-1]

---
**Questions for clarification:**
1. [ambiguity or scope question]

---
Please review. You can add, remove, reorder, or adjust tasks.
Type **"ready"**, **"go"**, or **"approved"** when satisfied.
```

### Task ordering heuristics
1. Domain/service layer first (pure logic, easiest to unit test)
2. Repository/data layer second (persistence)
3. API/controller layer third (HTTP interface)
4. E2E last (full flow verification)

### After feedback

Update the task list and re-display. Repeat until a trigger phrase is received:
`ready` / `go` / `approved` / `looks good`

---

## Step 5: Write Session File

### Determine the session file path

Session files live in `.tdd-sessions/` in the project root. The filename is based on the story ID:

| Situation | Filename |
|-----------|----------|
| Story ID available (e.g. `12345678`) | `.tdd-sessions/12345678.md` |
| No story ID (pasted content) | `.tdd-sessions/{title-slug}-{YYYY-MM-DD}.md` |

**Title slug:** lowercase, spaces and special characters replaced with hyphens, max 40 chars.
Example: "User Profile Update" → `user-profile-update` → `.tdd-sessions/user-profile-update-2026-03-31.md`

### Add `.tdd-sessions/` to `.gitignore`

Before writing the session file:

1. Check if `.gitignore` exists in the project root
2. Check if `.tdd-sessions/` (or `.tdd-sessions`) is already listed
3. If not listed:
   - If `.gitignore` exists: append a line `.tdd-sessions/` to the file
   - If `.gitignore` does not exist: create it with the single entry `.tdd-sessions/`
4. Display: "Added `.tdd-sessions/` to `.gitignore`" (or "`.tdd-sessions/` already in `.gitignore`")

### Write the session file

Create `.tdd-sessions/` directory if it does not exist, then write the file:

```markdown
# TDD Session

## Meta
- **Story ID:** [id or "—"]
- **Story Title:** [title]
- **Stack:** [stack]
- **Test FW:** [framework]
- **E2E FW:** [framework]
- **Conventions:** [summary line]

## Tasks
| # | Title | Type | Status |
|---|-------|------|--------|
| 1 | [title] | unit | ⏳ pending |
| 2 | [title] | integration | ⏳ pending |
| 3 | [title] | e2e | ⏳ pending |

## Task Details

### Task 1: [title]
- **Type:** unit
- **The test will assert:** [assertion description]
- **Implementation scope:** [scope]
- **Acceptance criteria link:** [AC reference]
- **Dependencies:** none
- **Status:** ⏳ pending

### Task 2: [title]
...
```

Then display:

```markdown
## ✅ Session Ready

`.tdd-sessions/[filename]` created with [N] tasks.

Run **/tdd-task** to start Task 1.
```
