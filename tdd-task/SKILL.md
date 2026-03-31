---
name: tdd-task
description: >
  Runs a single task's RED-GREEN-REFACTOR TDD loop in a ping-pong style.
  Reads the active session file from .tdd-sessions/ to find the current task,
  drives the full cycle with the developer (alternating test writing and
  implementation), then updates the session file on completion.
  Run /tdd-plan first to create the session file.
version: 1.0.0
category: engineering
tags: [tdd, pair-programming, ping-pong, red-green-refactor, testing]
triggers:
  - tdd task
  - next tdd task
  - run tdd
---

# TDD Task

## Invocation

```
/tdd-task [story-id]
```

- **No argument** — auto-detect the active session file from `.tdd-sessions/`
- **Story ID provided** — use `.tdd-sessions/{story-id}.md` directly

---

## On Start: Locate Session File

### Without an argument

1. List files in `.tdd-sessions/`
2. **If the directory does not exist or is empty:**
   ```
   "No active TDD session found. Run /tdd-plan first to create a session."
   ```
   STOP.
3. **If exactly one file exists:** use it
4. **If multiple files exist:**
   ```
   "Multiple sessions found. Which story are you working on?
   [1] 12345678.md — User Profile Update
   [2] user-profile-update-2026-03-31.md — (no ID)
   Type the number or story ID."
   ```
   PAUSE and wait for selection.

### After locating the file

Read the session file. **If all tasks are already ✅ done:**
```
"All tasks in this session are complete. Run /tdd-commit to review and commit."
```
STOP.

Find the first task with status `⏳ pending` and mark it `🔄 active` in the file.

---

## Session State Block

Render this before every AI action. Read all values from `.tdd-session.md`.

```markdown
---
## 🏓 Session State
| Field        | Value |
|--------------|-------|
| Story        | [ID]: [title] |
| Stack        | [stack] |
| Test FW      | [framework] |
| E2E FW       | [framework] |
| Conventions  | [conventions summary] |
| Phase        | RED / GREEN / REFACTOR |
| Current Task | [N] of [total]: [task title] |
| Turn         | DEV / AI |
| Next Action  | [one-line description] |
---

### Task Progress
| # | Title | Type | Status |
|---|-------|------|--------|
| 1 | [title] | unit | ✅ done / 🔄 active / ⏳ pending |
...
```

---

## Turn Setup

Display the current task details, then ask:

```
Task [N]: [title]
Type: [unit/integration/e2e]
The first test will assert: [assertion description]

Who writes the first failing test?
→ **"me"** — I'll write it
→ **"you"** — AI writes it
```

PAUSE and wait.

---

## Escape Hatches (always available)

| Developer types | Effect |
|----------------|--------|
| `skip` | AI takes the current turn instead |
| `next task` | Mark current task done, move to next |
| `abort` | Stop session, run /tdd-commit to commit what's done |
| `restart task` | Reset current task, start from RED |

---

## RED Phase

### If AI's turn

1. Render session state block (Phase: RED, Turn: AI)
2. **Before writing, recall the detected conventions from the session file:**
   - Test file location (co-located vs `test/` directory)
   - File naming suffix
   - describe/test/it structure and nesting depth
   - Assertion library and style
   - Mock pattern (vi.mock, vi.fn injection, mockk, MagicMock, etc.)
   - Test method naming convention
3. Write a failing test that:
   - Follows the project's detected conventions (not generic defaults)
   - Has a clear, behavior-describing name
   - References a function/class/endpoint that does not exist yet or behaves incorrectly
   - Will fail for the **right reason**: assertion failure or missing symbol — NOT syntax/import error
4. Display the test in a code block with the **exact file path** matching the project convention
5. Describe the expected failure output
6. Display:
   ```
   Run this test and confirm it is RED (failing for the right reason).
   Type **"red confirmed"** or **"failing"** to continue.
   (Or type "skip" to have AI write the implementation instead)
   ```
7. PAUSE and wait

### If developer's turn

1. Render session state block (Phase: RED, Turn: DEV)
2. Display:
   ```
   ## 🔴 RED — Your Turn

   Task [N]: [task title]
   Write a failing test that asserts: [assertion description]

   Paste your test code here.
   (Or type "skip" to have AI write it instead)
   ```
3. PAUSE and wait
4. When developer pastes code: acknowledge, note file path, proceed to GREEN

### What "right reason" means

```
✅ Correct RED                              ❌ Wrong RED (fix first)
─────────────────────────────────────────────────────────────────
TypeError: method is not a function         SyntaxError: Cannot find module
AssertionError: expected null to equal {...} ImportError: cannot import name
MockKException: no answer found             ClassNotFoundException
```

If the failure is wrong: create an empty stub (no behavior) to resolve the compile/import error, re-run, then confirm RED.

---

## GREEN Phase

### If AI's turn

1. Render session state block (Phase: GREEN, Turn: AI)
2. **Before writing, recall the detected conventions from the session file:**
   - Directory structure (feature-based vs layer-based)
   - Class/function naming convention
   - Dependency injection pattern
   - Annotation/decorator style
   - Error handling approach
3. Write the **minimum code** to make the failing test pass — nothing more
4. Display code with exact file path
5. Display:
   ```
   Add or modify this code, then run all tests.
   Type **"green confirmed"** or **"passing"** when all tests pass.
   (Paste error output if tests still fail)
   ```
6. PAUSE and wait
7. If developer pastes errors: diagnose the root cause, provide a targeted fix, PAUSE again

### If developer's turn

1. Render session state block (Phase: GREEN, Turn: DEV)
2. Display:
   ```
   ## 🟢 GREEN — Your Turn

   Write the minimum code to make the failing test pass.
   No extra features — pass the test only.

   Type **"done"** or **"green confirmed"** when all tests pass.
   (Or type "skip" to have AI implement it instead)
   ```
3. PAUSE and wait

### Minimum implementation principle

```
// ✅ Minimum — only what the current test needs
async getUser(id: string) {
  return this.repository.findById(id)
}

// ❌ Over-engineering during GREEN
async getUser(id: string) {
  const cached = this.cache.get(id)   // no test for this yet
  if (!cached) throw new Error(...)    // no test for this yet
  return cached
}
```

Add caching, error handling, etc. only when a test requires it.

---

## REFACTOR Phase

1. Render session state block (Phase: REFACTOR)
2. Review the test + implementation for:
   - **Duplication** — same logic in test and implementation?
   - **Naming** — does the name reveal intent without a comment?
   - **Magic values** — extract to named constants?
   - **Method length** — over 10 lines? Consider extraction.
   - **Framework idioms** — Spring beans, React hooks, pytest fixtures
   - **Convention consistency** — matches patterns detected in session file?
3. For each suggestion, display and PAUSE:
   ```
   **Refactoring suggestion [N]:** [one-line title]

   Reason: [why this matters]

   Before:
   [code block]

   After:
   [code block]

   Apply this refactor? (yes / no / modify)
   ```
4. If nothing to refactor: "Code looks clean. No refactoring needed this cycle."
5. After all suggestions resolved:
   ```
   REFACTOR complete. Are all tests still GREEN?
   Type **"green"** to continue to the next test.
   ```
6. PAUSE and wait

---

## Cycle Completion → Role Swap

After REFACTOR:
- Swap roles (AI ↔ DEV) for the next test in the same task
- Return to RED with the swapped roles

---

## Task Completion

When the developer indicates the task's tests are done (no more tests to write for this task), or when both parties agree the task is complete:

1. Update `.tdd-sessions/{story-id}.md`:
   - Mark current task as `✅ done`
   - If a next task exists, the next `/tdd-task` invocation will pick it up automatically

2. Display:
   ```
   ✅ Task [N] complete: [task title]

   Tasks remaining: [list of pending tasks]

   → Run **/tdd-task** to continue with Task [N+1]: [title]
   → Run **/tdd-commit** if you want to commit what's done so far
   ```

---

## Reference Files

| File | When to read |
|------|-------------|
| `.agents/skills/ping-pong-tdd/references/tdd-test-writing-guide.md` | Full test patterns per stack |
| `.agents/skills/ping-pong-tdd/references/convention-detection.md` | Per-stack convention extraction |
| `.agents/skills/ping-pong-tdd/references/red-green-refactor-guide.md` | Detailed phase completion criteria |
