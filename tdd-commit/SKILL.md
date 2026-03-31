---
name: tdd-commit
description: >
  Final step of a ping-pong TDD session. Reads the active session file from
  .tdd-sessions/ to summarize completed tasks, proposes a Conventional Commits
  message, executes the commit, and cleans up the session file.
  Run after all /tdd-task invocations are done.
version: 1.0.0
category: engineering
tags: [tdd, pair-programming, ping-pong, commit, git]
triggers:
  - tdd commit
  - commit tdd
  - finish tdd session
---

# TDD Commit

## Invocation

```
/tdd-commit [story-id]
```

- **No argument** — auto-detect the active session file from `.tdd-sessions/`
- **Story ID provided** — use `.tdd-sessions/{story-id}.md` directly

---

## Step 1: Locate Session File

1. List files in `.tdd-sessions/`
2. **If the directory does not exist or is empty:**
   ```
   "No active TDD session found. Run /tdd-plan to start a new session."
   ```
   STOP.
3. **If exactly one file exists:** use it
4. **If multiple files exist:**
   ```
   "Multiple sessions found. Which story are you committing?
   [1] 12345678.md — User Profile Update
   [2] payment-intent-2026-03-31.md — (no ID)
   Type the number or story ID."
   ```
   PAUSE and wait for selection.

---

## Step 2: Display Changes Summary

```markdown
## Changes Summary

**Story:** [ID]: [title]
**Stack:** [stack]

### Completed Tasks
- ✅ Task 1: [title] — [type]
- ✅ Task 2: [title] — [type]
- ⏳ Task 3: [title] — [type] ← skipped / not started

### Pending Tasks (not committed)
- [list any ⏳ pending tasks, or "none — all tasks complete"]
```

If there are pending tasks, note: "These tasks were not completed and will not be included in this commit."

---

## Step 3: Propose Commit Message

Use Conventional Commits format:

```
feat(<scope>): <short description under 72 chars>

Implements [STORY-ID]: [story title]

Tasks completed:
- [task 1 title]: [one-line description of what was implemented]
- [task 2 title]: [one-line description]
- [task N title]: [one-line description]

TDD: ping-pong pair programming session
```

### Scope selection

Use the feature/domain name — not the technical layer:

| Context | Example scope |
|---------|-------------|
| TypeScript React | `user-profile`, `cart`, `checkout`, `auth` |
| Kotlin/Java Spring | `payment`, `order`, `user-service`, `notification` |
| Python FastAPI | `users`, `products`, `orders`, `auth` |

### Type selection

| Situation | Type |
|-----------|------|
| New feature (most common) | `feat` |
| Bug fix | `fix` |
| Refactoring only, no new behavior | `refactor` |

---

## Step 4: Show Commit Preview and PAUSE

```markdown
## Commit Preview

**Message:**
```
[full commit message]
```

**Files to stage:**
[list source + test files created or modified during the session]
(Build artifacts, .env files, and IDE folders will NOT be staged)

Ready to commit?
→ Type **"commit"** or **"ship it"** to execute
→ Type your preferred message to override it
→ Type **"cancel"** to exit without committing
```

PAUSE and wait.

---

## Step 5: Execute Commit

When developer confirms:

1. Stage all source and test files from the session (never stage: `.env`, `.env.*`, build artifacts like `target/`, `build/`, `dist/`, `__pycache__/`, `.gradle/`)
2. Run `git add [files]` then `git commit -m "[message]"`
3. Display the resulting commit hash:
   ```
   ✅ Committed: abc1234 feat(payment): add payment intent creation endpoint
   ```

---

## Step 6: Clean Up Session File

```
Delete `.tdd-sessions/[filename]`? (yes / no)
→ "yes" — delete the file
→ "no" — keep it (useful if the story spans multiple sessions)
```

PAUSE and wait. Execute accordingly.

If `.tdd-sessions/` is now empty after deletion, it can be left in place — it is already gitignored.

---

## Reference Files

| File | When to read |
|------|-------------|
| `.agents/skills/ping-pong-tdd/references/commit-conventions.md` | Full commit format rules and multi-session strategy |
