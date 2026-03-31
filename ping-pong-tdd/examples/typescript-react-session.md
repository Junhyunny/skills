# Example Session: TypeScript + React

A complete ping-pong TDD session for a user profile update story.

---

## Invocation

```
/ping-pong-tdd 11223344
```

---

## Phase 1: Story Fetch

**AI:** Fetching from TrackerBoot MCP...

```markdown
## Story

**ID:** 11223344
**Title:** User Profile Update

**Description:**
A logged-in user should be able to update their name and email address.
Changes take effect immediately, and a success message is shown on save.

**Acceptance Criteria:**
- [ ] User can update their name
- [ ] User can update their email
- [ ] An error message is shown when the email format is invalid
- [ ] A confirmation message "Profile updated successfully" is shown on save

**Status:** In Progress

---
Does this look correct? Type "ok" to continue, or let me know what to fix.
```

**Developer:** ok

---

## Phase 2: Collaborative Planning

**AI:**

```markdown
## Project Conventions Detected

Files read: `src/user/user.service.test.ts`, `src/cart/cart.service.test.ts`,
            `src/user/user.service.ts`

Detected patterns:
- **Test location:** Co-located with source files
- **Test file naming:** `*.test.ts`
- **Test structure:** `describe('ClassName') > it('should ...')`
- **Mock approach:** `vi.mock('../dependency')` + `vi.mocked(fn).mockReturnValue()`
- **Assertion:** `expect(...).toEqual(...)` (Vitest default)
- **Source structure:** Feature-based directories, Service classes use PascalCase + `Service` suffix

All new code will follow these patterns.

---
## 🏓 Session State
| Field        | Value |
|--------------|-------|
| Story        | 11223344: User Profile Update |
| Stack        | TypeScript + React |
| Test FW      | Vitest + @testing-library/react |
| E2E FW       | Playwright |
| Conventions  | existing project — co-located *.test.ts, describe/it, vi.mock, PascalCase+Service |
| Phase        | PLANNING |
| Current Task | — |
| Turn         | — |
| Next Action  | Propose task breakdown |
---

## Task Plan (Draft)

**Detected stack:** TypeScript + React
**Test frameworks:** Vitest + @testing-library/react | Playwright

#### Task 1: updateUserProfile service function
- **Type:** unit
- **The test will assert:** updateUserProfile({ name, email }) calls the API and returns the updated user
- **Implementation scope:** src/services/userService.ts — updateUserProfile function
- **Acceptance criteria link:** AC 1, AC 2
- **Dependencies:** none

#### Task 2: Email validation
- **Type:** unit
- **The test will assert:** validateEmail returns false for invalid email formats
- **Implementation scope:** src/utils/validation.ts — validateEmail function
- **Acceptance criteria link:** AC 3
- **Dependencies:** none

#### Task 3: ProfileForm component
- **Type:** unit (component)
- **The test will assert:** submitting with valid input calls updateUserProfile; invalid email shows error message
- **Implementation scope:** src/components/ProfileForm/ProfileForm.tsx
- **Acceptance criteria link:** AC 1, AC 2, AC 3, AC 4
- **Dependencies:** Task 1, Task 2

#### Task 4 (E2E): Profile update full flow
- **Type:** e2e
- **The test will assert:** User updates name and sees "Profile updated successfully" after saving
- **Implementation scope:** e2e/user-profile.spec.ts (E2E test only)
- **Acceptance criteria link:** Full AC coverage check
- **Dependencies:** Task 3

---
**Questions:**
1. Is the API endpoint PATCH /api/users/:id?
2. Are there minimum/maximum length constraints on the name field?

---
Please review. Type "ready" when satisfied.
```

**Developer:** No name length limits. API is PATCH /api/users/:id. ready

**AI:**

```markdown
## ✅ Plan Confirmed

| # | Task | Type |
|---|------|------|
| 1 | updateUserProfile service function | unit |
| 2 | Email validation | unit |
| 3 | ProfileForm component | unit (component) |
| 4 | Profile update full flow | e2e |

---
Starting Phase 3. Task 1: updateUserProfile service function

Who writes the first failing test?
→ **"me"** — I'll write it
→ **"you"** — AI writes it
```

**Developer:** you

---

## Phase 3: TDD Loop — Task 1

### Cycle 1 — AI writes test, Developer implements

**AI (RED):**

```markdown
---
## 🏓 Session State
| Field        | Value |
|--------------|-------|
| Story        | 11223344: User Profile Update |
| Stack        | TypeScript + React |
| Conventions  | existing project — co-located *.test.ts, describe/it, vi.mock |
| Phase        | RED |
| Current Task | 1 of 4: updateUserProfile service function |
| Turn         | AI |
| Next Action  | AI writes failing unit test for updateUserProfile |
---

## 🔴 RED — AI Writing

**File:** `src/services/userService.test.ts` (new)

```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest'
import { updateUserProfile } from './userService'

const mockFetch = vi.fn()
vi.stubGlobal('fetch', mockFetch)

describe('updateUserProfile', () => {
  beforeEach(() => {
    vi.clearAllMocks()
  })

  it('should call PATCH /api/users/:id with updated data and return the updated user', async () => {
    // Arrange
    const userId = 'user-123'
    const updateData = { name: 'Alice Updated', email: 'alice.new@example.com' }
    const updatedUser = { id: userId, ...updateData }

    mockFetch.mockResolvedValueOnce({
      ok: true,
      json: async () => updatedUser,
    })

    // Act
    const result = await updateUserProfile(userId, updateData)

    // Assert
    expect(mockFetch).toHaveBeenCalledWith(`/api/users/${userId}`, {
      method: 'PATCH',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(updateData),
    })
    expect(result).toEqual(updatedUser)
  })
})
```

**Expected failure:**
```
TypeError: updateUserProfile is not a function
```

Run this test and confirm it is RED.
Type **"red confirmed"** when the test is failing for the right reason.
```

**Developer:** red confirmed

**AI (GREEN):**

```markdown
---
## 🏓 Session State
| Phase | GREEN | Turn | DEV |
---

## 🟢 GREEN — Your Turn

Write the minimum code to make the failing test pass.
The goal is passing the test only — no extra features.

Type **"done"** when all tests pass.
```

**Developer:** done

**AI (REFACTOR):**

```markdown
---
## 🏓 Session State
| Phase | REFACTOR | Turn | — |
---

## 🔧 REFACTOR

One observation: the current implementation does not handle the case where `response.ok` is false.
Since there is no test for that yet, adding it now would violate the GREEN principle.
Recommend adding an error-case test in the next cycle.

The rest of the code is clean. No refactoring needed this cycle.

Are all tests still GREEN? Type **"green"** to continue.
```

**Developer:** green

### Cycle 2 — Developer writes test, AI implements (roles swapped)

**AI:**

```
## 🔴 RED — Your Turn

Task 1, second test:
Write a failing test to verify that updateUserProfile throws an error when the server returns a non-ok response.

Paste your test code here.
```

**Developer:**
```typescript
it('should throw an error when the server returns a non-ok response', async () => {
  mockFetch.mockResolvedValueOnce({ ok: false, status: 500 })

  await expect(
    updateUserProfile('user-123', { name: 'Alice', email: 'alice@example.com' })
  ).rejects.toThrow('Failed to update profile')
})
```

**AI (GREEN — AI implements):**

```markdown
---
## 🏓 Session State
| Phase | GREEN | Turn | AI |
---

## 🟢 GREEN — AI Writing

**File:** `src/services/userService.ts` (updated)

```typescript
export async function updateUserProfile(
  userId: string,
  data: { name: string; email: string }
) {
  const response = await fetch(`/api/users/${userId}`, {
    method: 'PATCH',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(data),
  })

  if (!response.ok) {
    throw new Error('Failed to update profile')
  }

  return response.json()
}
```

Run all tests. Type **"green confirmed"** when all pass.
```

**Developer:** green confirmed

**AI (REFACTOR → Task Complete):**

```markdown
## 🔧 REFACTOR

Code looks clean. No refactoring needed for this cycle.

---
✅ Task 1 complete: updateUserProfile service function

Moving to 2 of 4.

Task 2: Email validation
Who writes the first failing test?
→ "me" / "you"
```

*(Tasks 2 and 3 continue similarly — omitted for brevity)*

---

## Phase 4: Review & Commit

**AI:**

```markdown
## Changes Summary

### Completed Tasks
- ✅ Task 1: updateUserProfile service function — 2 tests, 1 implementation file
- ✅ Task 2: Email validation — 3 tests, 1 implementation file
- ✅ Task 3: ProfileForm component — 4 tests, 1 component file
- ✅ Task 4 (E2E): Profile update full flow — 1 E2E test

### Modified Files
- `src/services/userService.ts` (new)
- `src/services/userService.test.ts` (new)
- `src/utils/validation.ts` (new)
- `src/utils/validation.test.ts` (new)
- `src/components/ProfileForm/ProfileForm.tsx` (new)
- `src/components/ProfileForm/ProfileForm.test.tsx` (new)
- `e2e/user-profile.spec.ts` (new)

---
**Commit message:**
```
feat(user-profile): add profile name and email update

Implements 11223344: User Profile Update

Tasks completed:
- updateUserProfile: PATCH /api/users/:id with error handling
- validateEmail: format validation
- ProfileForm component: edit form with validation and success feedback
- E2E: profile update flow from form submission to confirmation

TDD: ping-pong pair programming session
```

Type **"commit"** to execute.
```

**Developer:** commit

**AI:** Committed. `abc1234 feat(user-profile): add profile name and email update`
