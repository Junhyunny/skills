# Ping-Pong TDD — Shared Reference Library

This directory contains the shared reference files used by the three ping-pong TDD skills.
The `ping-pong-tdd` skill itself has been split into focused skills for better reliability.

---

## Use These Skills Instead

| Skill | Command | Purpose |
|-------|---------|---------|
| tdd-plan | `/tdd-plan` | Load story, detect stack & conventions, plan tasks |
| tdd-task | `/tdd-task` | Run one task's RED-GREEN-REFACTOR TDD cycle |
| tdd-commit | `/tdd-commit` | Commit session work with a structured message |

## Typical Session Flow

```
/tdd-plan 12345678         # 1. Load story from TrackerBoot, plan tasks
                           #    → creates .tdd-session.md

/tdd-task                  # 2. Run Task 1 (RED → GREEN → REFACTOR)
                           #    → updates .tdd-session.md

/tdd-task                  # 3. Run Task 2 (repeat for each task)

/tdd-commit                # 4. Commit, clean up .tdd-session.md
```

### Providing story content directly

```
/tdd-plan "As a customer, I want to initiate a payment for my order
so I can complete my purchase. The system creates a payment intent
and returns a client secret for the frontend."
```

---

## Session State Files: `.tdd-sessions/`

Created by `/tdd-plan`, updated by `/tdd-task`, cleaned up by `/tdd-commit`.

| Situation | File path |
|-----------|-----------|
| Story ID available | `.tdd-sessions/12345678.md` |
| No ID (pasted content) | `.tdd-sessions/user-profile-update-2026-03-31.md` |

- `/tdd-plan` automatically adds `.tdd-sessions/` to `.gitignore`
- Multiple sessions can coexist — `/tdd-task` and `/tdd-commit` auto-detect or ask which one to use
- Each file stores: story info, stack, conventions, task list with statuses, full task details

---

## Reference Files (used by all three skills)

| File | Purpose |
|------|---------|
| `references/tdd-test-writing-guide.md` | Test patterns per stack (TypeScript, Kotlin, Java, Python) |
| `references/tech-stack-detection.md` | Stack auto-detection from build files |
| `references/convention-detection.md` | Reading existing project conventions |
| `references/mcp-story-fetching.md` | TrackerBoot MCP tool patterns and field extraction |
| `references/red-green-refactor-guide.md` | Phase completion criteria and common failure modes |
| `references/commit-conventions.md` | Conventional Commits format and multi-session strategy |

---

## Supported Stacks

| Stack | Unit Tests | E2E Tests |
|-------|-----------|-----------|
| TypeScript + React | Vitest / Jest + @testing-library/react | Playwright / Cypress |
| Kotlin + Spring | JUnit5 + MockK + Testcontainers | RestAssured |
| Java + Spring | JUnit5 + Mockito + Testcontainers | RestAssured |
| Python + FastAPI | pytest + httpx + pytest-asyncio | Playwright (Python) |
