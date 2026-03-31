# Session State Template

This template is rendered inline in the conversation before every AI action.
Do NOT create a separate file — render it as a markdown block.

---

## Full State Block (copy and fill in)

```markdown
---
## 🏓 Session State
| Field        | Value |
|--------------|-------|
| Story        | [STORY-ID]: [story title] |
| Stack        | [TypeScript+React / Kotlin+Spring / Java+Spring / Python+FastAPI] |
| Test FW      | [Vitest / JUnit5+MockK / JUnit5+Mockito / pytest] |
| E2E FW       | [Playwright / RestAssured / RestAssured / httpx+playwright / none] |
| Conventions  | [new project / existing project — key patterns summary] |
| Phase        | [PLANNING / RED / GREEN / REFACTOR] |
| Current Task | [N] of [total]: [task title] |
| Turn         | [DEV / AI] |
| Next Action  | [one-line description of what happens next] |
---
```

---

## Task Progress Block (show alongside state in Phase 3)

```markdown
### Task Progress
| # | Title | Type | Status |
|---|-------|------|--------|
| 1 | [title] | unit | ✅ done / 🔄 active / ⏳ pending |
| 2 | [title] | integration | ⏳ pending |
| 3 | [title] | e2e | ⏳ pending |
```

---

## State Values Reference

### Phase values
- `PLANNING` — Phase 2, working on task breakdown
- `RED` — Writing a failing test
- `GREEN` — Writing minimum implementation
- `REFACTOR` — Reviewing and improving code quality

### Turn values
- `DEV` — Developer's turn (AI is waiting)
- `AI` — AI's turn (AI is acting)

### Status values (task progress)
- `⏳ pending` — Not started
- `🔄 active` — Currently in progress
- `✅ done` — All tests pass, refactoring complete

---

## Example: Filled State Block

```markdown
---
## 🏓 Session State
| Field        | Value |
|--------------|-------|
| Story        | PROJ-123: User profile update |
| Stack        | TypeScript + React |
| Test FW      | Vitest + @testing-library/react |
| E2E FW       | Playwright |
| Conventions  | existing project — co-located *.test.ts, describe/it, vi.mock, PascalCase+Service |
| Phase        | RED |
| Current Task | 2 of 4: UserService.updateProfile |
| Turn         | AI |
| Next Action  | AI writes failing unit test for updateProfile method |
---

### Task Progress
| # | Title | Type | Status |
|---|-------|------|--------|
| 1 | UserRepository.save | unit | ✅ done |
| 2 | UserService.updateProfile | unit | 🔄 active |
| 3 | UserController PATCH /users/:id | integration | ⏳ pending |
| 4 | E2E: profile update flow | e2e | ⏳ pending |
```
