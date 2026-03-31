# Commit Conventions

Ping-pong TDD sessions use Conventional Commits format for structured, readable history.

---

## Format

```
<type>(<scope>): <short description>

<body>

<footer>
```

### Rules

- **First line:** max 72 characters, imperative mood ("add" not "added")
- **Blank line** between subject, body, and footer
- **Body:** explain WHAT was done and why (not how)
- **Footer:** optional metadata

---

## Types for TDD Sessions

| Type | When to use |
|------|------------|
| `feat` | New feature implemented via TDD (most common) |
| `fix` | Bug found and fixed during the session |
| `refactor` | Refactoring-only session (no new behavior) |
| `test` | Test-only changes (rare in ping-pong TDD) |
| `chore` | Setup, config, build changes touched during session |

---

## Scope

Use the feature/module name as scope:

| Context | Example scope |
|---------|-------------|
| TypeScript React component | `user-profile`, `cart`, `auth` |
| Kotlin/Java Spring service | `user-service`, `payment`, `order` |
| Python FastAPI endpoint | `users`, `products`, `auth` |
| Cross-cutting | `api`, `domain` |

---

## Session Commit Template

### Single story, multiple tasks

```
feat(user-profile): add profile update functionality

Implements PROJ-123: User Profile Update

Tasks completed:
- UserService.updateProfile: validates and persists name/email changes
- UserController.patchUser: PATCH /api/users/:id endpoint
- ProfileForm component: edit form with validation feedback
- E2E: profile update flow from form to confirmation message

TDD: ping-pong pair programming session
```

### Bug fix session

```
fix(payment): handle currency rounding in order total calculation

Fixes PROJ-456: Payment amount mismatch due to decimal rounding error

Root cause: floating-point arithmetic used instead of integer cents
Solution: convert all monetary values to cents before calculation

Tasks completed:
- OrderCalculator.calculateTotal: use integer arithmetic in cents
- OrderCalculatorTest: added edge cases for rounding scenarios

TDD: ping-pong pair programming session
```

### Refactoring session

```
refactor(user-service): extract validation logic to UserValidator

No behavioral changes — all tests pass before and after refactor.

Motivation: UserService.updateProfile was > 40 lines; validation logic
was duplicated in UserService and OrderService.

TDD: ping-pong pair programming session
```

---

## Files to Stage and NOT Stage

### Always stage

- Source files modified or created during the session
- Test files created during the session
- Configuration files directly needed by new code

### Never stage

- `.env`, `.env.local`, `.env.production`
- Files matching `.gitignore`
- Build artifacts (`target/`, `build/`, `dist/`, `__pycache__/`, `.gradle/`)
- IDE files (`.idea/`, `.vscode/` — unless the project already tracks them)

### Ask before staging

- Database migration files (they may need review before commit)
- `package-lock.json` / `yarn.lock` / `Pipfile.lock` — stage if dependencies changed, skip if not

---

## Multi-Session Strategy

When multiple ping-pong TDD sessions contribute to one story:

**Option A: One commit per session** (preferred when sessions are a day apart)
```
feat(user-profile): add profile read endpoint (session 1/3)
feat(user-profile): add profile update endpoint (session 2/3)
feat(user-profile): add profile delete and E2E tests (session 3/3)
```

**Option B: Squash all sessions** (preferred for a clean history before PR)
- Squash when the full story is done, before the PR review
- Use the full story commit template above

---

## Confirming Before Commit

Always display the proposed commit message and the list of files to be staged before executing:

```markdown
## Commit Preview

**Commit message:**
```
feat(user-profile): add profile update functionality

Implements PROJ-123: User Profile Update

Tasks completed:
- ...

TDD: ping-pong pair programming session
```

**Files to stage:**
- src/services/UserService.ts
- src/services/UserService.test.ts
- src/components/ProfileForm.tsx
- src/components/ProfileForm.test.tsx
- e2e/user-profile.spec.ts

Ready to commit?
→ **"commit"** or **"ship it"** — execute the commit
→ To edit the message, type the revised message directly
```
