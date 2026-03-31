# RED-GREEN-REFACTOR Guide

Defines what "done" means for each phase, common failure modes, and when to skip refactoring.

---

## RED Phase — Completion Criteria

The RED phase is complete when ALL of the following are true:

1. **Test compiles/parses** — no syntax errors, no missing imports that prevent the test runner from loading the file
2. **Test runs** — the test runner executes the test (doesn't crash on setup)
3. **Test fails** — the test runner reports a failure (exit code non-zero, test marked as FAILED)
4. **Fails for the right reason** — the failure message is an assertion failure or a "method not found / function not defined" error, NOT an import error or test setup crash

### What "Right Reason" Looks Like Per Stack

**TypeScript:**
```
✅ CORRECT
TypeError: sut.getUser is not a function
AssertionError: expected null to equal { name: 'Alice' }

❌ WRONG (fix before calling RED confirmed)
SyntaxError: Cannot find module './UserService'  → file doesn't exist, create the file first
TypeError: Cannot read properties of undefined (reading 'getUser')  → import is broken
```

**Kotlin:**
```
✅ CORRECT
io.mockk.MockKException: no answer found for: UserRepository(#1).findById(...)
org.opentest4j.AssertionFailedError: expected: <Alice> but was: <null>

❌ WRONG
error: unresolved reference: UserService  → class doesn't exist, create it first
```

**Java:**
```
✅ CORRECT
org.mockito.exceptions.base.MockitoException: Cannot mock final class UserService
java.lang.AssertionError: expected:<Alice> but was:<null>

❌ WRONG
java.lang.ClassNotFoundException: com.example.UserService  → class doesn't exist
```

**Python:**
```
✅ CORRECT
AttributeError: 'UserService' object has no attribute 'get_user'
AssertionError: assert None == {'name': 'Alice'}

❌ WRONG
ImportError: cannot import name 'UserService' from 'app.domain.user_service'  → module missing
ModuleNotFoundError: No module named 'app.domain'  → file missing
```

### Fixing Wrong RED Before Continuing

If the test fails for the wrong reason:
1. Create the minimum file/class/function stub needed so the import works
2. The stub should NOT implement the behavior — just be empty or return None/null/0
3. Run the test again — now it should fail for the right reason
4. THEN call "red confirmed"

---

## GREEN Phase — Completion Criteria

The GREEN phase is complete when ALL of the following are true:

1. **All tests pass** — exit code 0, all test markers show PASSED/OK
2. **No new tests were written** during implementation (tests and implementation are separate phases)
3. **Implementation is minimal** — only the code needed to pass the current failing test, nothing more

### Minimum Implementation Principle

Write the simplest code that makes the test pass:

```typescript
// Test asserts: getUser("1") returns { name: "Alice" }

// ✅ Correct minimum implementation
async getUser(id: string) {
  return this.repository.findById(id);
}

// ❌ Over-engineering during GREEN
async getUser(id: string) {
  const cached = this.cache.get(id);      // no test for this yet
  if (cached) return cached;
  const user = await this.repository.findById(id);
  if (!user) throw new UserNotFoundException(id);  // no test for this yet
  this.cache.set(id, user);
  return user;
}
```

Add caching, error handling, etc. only when a test drives you to do so.

### What To Do When GREEN Reveals Test Issues

If the implementation reveals that the test was written incorrectly:
1. **Do NOT silently fix the test** — PAUSE and discuss with the developer
2. Display: "Found a problem with the test during implementation: [description]. Should we fix the test, or is the test correct and the implementation needs to be adjusted?"
3. Wait for guidance

---

## REFACTOR Phase — What to Review

### Checklist (in order of impact)

**1. Duplication**
- Is the same logic in both the test and the implementation?
- Are there two functions/methods doing the same thing?
- Are the same magic values repeated in multiple places?

**2. Naming**
- Does the variable/function/class name reveal intent WITHOUT needing a comment?
- Is the name consistent with the rest of the codebase?
- Does the test method name accurately describe what it tests?

**3. Magic Values**
- Are there unexplained numbers or strings? Extract to named constants.
- Example: `if (attempts > 3)` → `if (attempts > MAX_LOGIN_ATTEMPTS)`

**4. Method Length and Complexity**
- Is any function/method over 10 lines? Consider extraction.
- Does any function do more than one thing?

**5. Framework Idioms**
- **Spring:** Are beans properly scoped? Is `@Transactional` placed correctly?
- **React:** Are hooks called correctly? Is state mutation avoided?
- **FastAPI:** Are dependency injection patterns followed? Are route handlers thin?
- **Kotlin:** Is idiomatic Kotlin used (data classes, extension functions, scope functions)?

**6. Test Quality**
- After refactoring implementation, does the test still test what it should?
- Are test setup and assertions still clear?
- Did the refactor accidentally make the test redundant?

### Suggesting a Refactor

For each suggestion, display:
```
**Refactoring suggestion [N/total]:** [one-line title]

Reason: [why this matters]

Before:
```[language]
[old code]
```

After:
```[language]
[new code]
```

Apply this change?
→ **yes** — apply
→ **no** — skip
→ **modify** — suggest a different approach
```

### When to Skip Refactoring

Skip immediately (don't suggest anything) when:
- The code is clearly clean, idiomatic, and readable
- The test has only one assertion and the implementation is a one-liner
- Both parties agree nothing needs changing

Display: "Code looks clean. No refactoring needed this cycle."

### After Refactoring

Always verify:
1. All tests still pass (run the test suite)
2. Display: "Refactoring complete. Are all tests still GREEN? Type **'green'** to move to the next test."

---

## Common Failure Modes

### RED → GREEN fails immediately (test was green already)

This means the behavior already existed. Options:
1. Delete the test and write a more specific failing test
2. Check if this test is actually new or a duplicate
3. The task may already be done — confirm with developer

### GREEN → all tests pass but coverage reveals gaps

After GREEN, run the full test suite (not just the new test). If other tests break:
1. PAUSE — show which tests broke and their error messages
2. Discuss: "This implementation broke existing tests: [list]. Should we fix the existing tests or change the implementation approach?"

### Refactor introduces a regression

If a suggested refactor causes tests to fail:
1. Immediately acknowledge: "This refactoring broke the tests."
2. Revert the refactor
3. Try a different, safer approach or skip this refactor

### Session runs too long on one task

If a task has been in RED-GREEN-REFACTOR for more than 3-4 cycles without the next test being written:
1. Suggest splitting the task: "This task feels larger than expected. Should we mark what's implemented so far as done and split the rest into a new task?"
2. PAUSE for developer decision
