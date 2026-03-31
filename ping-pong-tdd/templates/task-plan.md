# Task Plan Template

Generated at the end of Phase 2 (Collaborative Planning). Display this in the conversation when presenting the initial task breakdown and after each revision.

---

## Task Plan Format

```markdown
## Task Plan

**Story:** [STORY-ID]: [title]
**Detected stack:** [stack]
**Test frameworks:** [unit FW] | [E2E FW]

---

### Story Summary
[2-3 sentences summarizing what the story is about and why it matters]

### Acceptance Criteria
- [ ] [AC 1]
- [ ] [AC 2]
- [ ] [AC 3]

---

### Task List

#### Task 1: [title — noun phrase describing what's being built]
- **Type:** unit | integration | e2e
- **The test will assert:** [one sentence — what the first failing test will assert in behavior terms]
- **Implementation scope:** [what production code will be written to make it pass]
- **Acceptance criteria link:** [which AC item(s) this covers]
- **Dependencies:** none

#### Task 2: [title]
- **Type:** unit | integration | e2e
- **The test will assert:** [one sentence]
- **Implementation scope:** [what code will be written]
- **Acceptance criteria link:** [AC reference]
- **Dependencies:** Task 1

#### Task 3: [title — often a controller/API layer]
- **Type:** integration
- **The test will assert:** [HTTP response behavior, status codes, response body]
- **Implementation scope:** [endpoint handler, routing, serialization]
- **Acceptance criteria link:** [AC reference]
- **Dependencies:** Task 1, Task 2

#### Task [N] (E2E): [title — user-facing flow]
- **Type:** e2e
- **The test will assert:** [full user journey from UI to confirmation, in user-perspective language]
- **Implementation scope:** E2E test only — feature already implemented by prior tasks
- **Acceptance criteria link:** Full AC coverage check
- **Dependencies:** Task [N-1]

---

### Questions for Developer
1. [Ambiguity or scope question]
2. [Edge case clarification]

---

Please review the plan.
- You can add, remove, reorder, or adjust the scope of any task
- When ready, type **"ready"**, **"go"**, or **"approved"**
```

---

## Filling in the Template — Guidelines

### Task titles
- Use noun phrases: "UserService.updateProfile", "PATCH /api/users/:id endpoint", "Profile edit E2E flow"
- Be specific enough to know when it's done
- Avoid vague titles like "Backend work" or "Tests"

### Task type selection
- `unit` — tests a single class/function in isolation with mocks
- `integration` — tests a component with real dependencies (DB, HTTP layer, frameworks)
- `e2e` — tests a full user flow through the UI or API boundary

### Task ordering heuristics
1. Domain/service layer first (pure business logic, most testable)
2. Repository/data layer second (persistence)
3. API/controller layer third (HTTP interface)
4. E2E last (verifies the full assembled system)

### Number of tasks
- Simple story (1-2 ACs): 2-3 tasks + 1 E2E
- Medium story (3-4 ACs): 3-5 tasks + 1 E2E
- Complex story: consider splitting the story before planning tasks

### E2E task scope
- One E2E task per story is usually enough
- The E2E test should cover the primary happy path from the story
- Edge cases are covered by unit/integration tests
