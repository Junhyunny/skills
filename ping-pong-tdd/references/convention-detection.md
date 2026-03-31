# Convention Detection Guide

When running a ping-pong TDD session in an existing project, the AI must first read the project's existing code so that new tests and implementation fit naturally into the surrounding style.

**Principle: The project's actual code takes precedence over default patterns.**

---

## When to Run

Execute at Phase 2 Step 2, immediately after stack detection.

```
No test/source files found → new project → use default patterns from tdd-test-writing-guide.md
Test/source files exist    → existing project → follow this guide to read conventions
```

---

## Universal: Files to Read

Check these regardless of stack.

### 1. Priority: Files from the Same Area as the Story

```
If the current story is about a "cart" feature:
  → Prefer: src/cart/, CartService*, cart.test.*, test_cart.py etc.
  → Fallback: the 2–3 most recently modified test files in the project
```

### 2. Code Style Config Files

| File | Stack | What to Check |
|------|-------|--------------|
| `.eslintrc*`, `eslint.config.*` | TypeScript | semicolons, quotes, import order |
| `prettier.config.*`, `.prettierrc` | TypeScript | indentation, line length |
| `.editorconfig` | universal | tab/space indentation, line endings |
| `detekt.yml`, `.detekt.yml` | Kotlin | function length, complexity thresholds |
| `pyproject.toml` `[tool.black]` | Python | line length |
| `pyproject.toml` `[tool.ruff]` | Python | lint rules |

---

## TypeScript + React

### Files to Read

```
1. 2–3 existing test files
   Prefer: *.test.ts, *.test.tsx, *.spec.ts, *.spec.tsx
   Location: co-located (next to source) vs __tests__/ vs src/test/

2. 1–2 existing source files
   Prefer: same layer as the story (service, hook, component, etc.)
```

### Conventions to Extract

**Test structure**

```typescript
// Pattern A: describe + it (most common)
describe('ServiceName', () => {
  describe('methodName', () => {
    it('should ...', () => { ... })
  })
})

// Pattern B: describe + test
describe('ServiceName', () => {
  test('should ...', () => { ... })
})

// Pattern C: top-level test only (no describe nesting)
test('ServiceName should ...', () => { ... })
```

**Identify:**
- Does the project use `it` or `test`?
- How many levels of `describe` nesting?
- Test name format: `'should X when Y'` vs `'X when Y should Z'`
- Where does `beforeEach` / `afterEach` live (inside or outside `describe`)?

**Mock patterns**

```typescript
// Pattern A: vi.mock (module-level)
vi.mock('../services/userService')
const mockGetUser = vi.mocked(userService.getUser)

// Pattern B: vi.fn() injected directly
const mockRepository = { findById: vi.fn() }
const sut = new UserService(mockRepository as UserRepository)

// Pattern C: jest.spyOn
jest.spyOn(userService, 'getUser').mockResolvedValue(...)
```

**Assertion patterns**

```typescript
// Vitest/Jest default
expect(result).toEqual(expected)
expect(result).toBe(expected)
expect(fn).toHaveBeenCalledWith(arg)

// @testing-library/jest-dom extensions
expect(element).toBeInTheDocument()
expect(element).toHaveTextContent('...')
```

**Source file conventions**

```typescript
// Class vs functional style
class UserService { ... }           // class-based
function createUserService() { }    // factory function
export const userService = { }      // object literal

// Dependency injection
constructor(private readonly repo: UserRepository) {}  // constructor injection
function getUser(repo = defaultRepo) {}                // default parameter

// Error handling
throw new UserNotFoundError(id)           // custom error class
return { ok: false, error: 'NOT_FOUND' } // Result pattern
```

**File/directory structure**

```
Feature-based:           Layer-based:
src/
  user/                  services/
    user.service.ts        user.service.ts
    user.service.test.ts components/
    user.types.ts           UserProfile.tsx
                         hooks/
                           useUser.ts
```

---

## Kotlin + Spring

### Files to Read

```
1. 2–3 test files
   Location: src/test/kotlin/.../**Test.kt or **Spec.kt

2. 1–2 source files
   Location: src/main/kotlin/.../
   Prefer: same layer as the story (domain, service, web, application, etc.)

3. Build file
   build.gradle.kts: check testImplementation dependencies
   (kotest vs JUnit5, mockk vs mockito-kotlin, etc.)
```

### Conventions to Extract

**Test class structure**

```kotlin
// Pattern A: JUnit5 + @Nested (hierarchical)
@DisplayName("UserService")
class UserServiceTest {
    @Nested @DisplayName("getUser") inner class GetUser {
        @Test @DisplayName("should ...") fun `...`() { }
    }
}

// Pattern B: JUnit5 flat structure
class UserServiceTest {
    @Test fun `should return user when exists`() { }
    @Test fun `should throw when not found`() { }
}

// Pattern C: Kotest (BehaviorSpec / DescribeSpec)
class UserServiceTest : BehaviorSpec({
    Given("an existing user") {
        When("getUser is called") {
            Then("returns the user") { }
        }
    }
})
```

**Mock patterns**

```kotlin
// MockK (most common)
private val repo = mockk<UserRepository>()
every { repo.findById(any()) } returns user
verify { repo.findById(userId) }

// Mockito-Kotlin
private val repo = mock<UserRepository>()
whenever(repo.findById(any())).thenReturn(user)
verify(repo).findById(userId)

// @MockkBean / @MockBean (Spring integration tests)
@MockkBean private lateinit var userService: UserService
@MockBean private lateinit var userService: UserService
```

**Assertion patterns**

```kotlin
// AssertJ (most common)
assertThat(result).isEqualTo(expected)
assertThat(result.name).isEqualTo("Alice")

// Kotest Matchers
result shouldBe expected
result.name shouldBe "Alice"

// JUnit5 built-in
assertEquals(expected, result)
assertThrows<UserNotFoundException> { sut.getUser(id) }
```

**Source file conventions**

```kotlin
// Package/directory structure — layer-based vs domain-based
com.example.user.UserService        // domain-based (hexagonal, etc.)
com.example.service.UserService     // layer-based

// Dependency injection — constructor vs field injection
class UserService(private val repo: UserRepository) {}                         // constructor (preferred)
@Service class UserService { @Autowired lateinit var repo: UserRepository }    // field injection

// Spring annotation patterns
@Service class UserService(...)             // service layer
@Repository interface UserRepository ...   // JPA repository
@RestController @RequestMapping("/api")    // controller
@Component                                 // generic bean

// Error handling patterns
throw UserNotFoundException("User $id not found")       // direct custom exception
throw NotFoundException(ErrorCode.USER_NOT_FOUND)       // shared exception + error code enum
return Result.failure(UserNotFoundError)                // Result type
```

**Package naming conventions**

```
domain/ vs entity/ vs model/           → domain objects
service/ vs application/               → business logic
repository/ vs port/out/               → persistence
controller/ vs web/ vs adapter/in/     → controllers
```

---

## Java + Spring

### Files to Read

```
1. 2–3 test files: src/test/java/.../**Test.java
2. 1–2 source files: src/main/java/.../
3. Build file: test dependencies in pom.xml or build.gradle
```

### Conventions to Extract

**Test class structure**

```java
// Pattern A: JUnit5 + @Nested
@DisplayName("UserService")
class UserServiceTest {
    @Nested @DisplayName("getUser")
    class GetUser {
        @Test @DisplayName("should return user when exists")
        void shouldReturnUserWhenExists() { }
    }
}

// Pattern B: flat structure (no nesting)
class UserServiceTest {
    @Test void shouldReturnUserWhenExists() { }
    @Test void shouldThrowWhenNotFound() { }
}
```

**Method naming**

```java
// camelCase verb form (most common)
void shouldReturnUserWhenExists()
void givenUserExists_whenGetUser_thenReturnUser()  // Given-When-Then style
void getUser_existingUser_returnsUser()             // underscore-separated
```

**Mock patterns**

```java
// @ExtendWith(MockitoExtension.class) + annotations
@Mock UserRepository userRepository;
@InjectMocks UserService sut;

// Direct construction
UserRepository mockRepo = mock(UserRepository.class);
UserService sut = new UserService(mockRepo);

// BDDMockito
given(repo.findById(id)).willReturn(Optional.of(user));
then(repo).should().findById(id);
```

**Source file conventions**

```java
// Constructor annotations
@RequiredArgsConstructor  // Lombok
public UserService(UserRepository userRepository) { }  // explicit constructor

// Immutability
private final UserRepository userRepository;  // final field
private UserRepository userRepository;        // mutable field

// Error code approach
throw new UserNotFoundException(id)                         // simple exception
throw new BusinessException(ErrorCode.USER_NOT_FOUND)       // error code enum
```

---

## Python + FastAPI

### Files to Read

```
1. 2–3 test files: tests/**test_*.py or tests/**/*_test.py
2. 1–2 source files: app/**/*.py
3. conftest.py: understand shared fixtures
4. pytest.ini or pyproject.toml [tool.pytest.ini_options]: test configuration
```

### Conventions to Extract

**Test structure**

```python
# Pattern A: class-based
class TestUserService:
    def setup_method(self):
        self.mock_repo = MagicMock()
        self.sut = UserService(repository=self.mock_repo)

    def test_should_return_user_when_exists(self):
        ...

# Pattern B: function-based (pytest style)
@pytest.fixture
def user_service(mock_repo):
    return UserService(repository=mock_repo)

def test_should_return_user_when_exists(user_service, mock_repo):
    ...

# Pattern C: async
@pytest.mark.asyncio
async def test_should_return_user_when_exists(async_client):
    ...
```

**Fixture patterns**

```python
# Check conftest.py for shared fixtures
@pytest.fixture
def mock_repo():
    return MagicMock(spec=UserRepository)

@pytest.fixture
def async_client(app):
    return AsyncClient(transport=ASGITransport(app=app), base_url="http://test")
```

**Mock patterns**

```python
# unittest.mock
from unittest.mock import MagicMock, AsyncMock, patch

mock_repo = MagicMock(spec=UserRepository)
mock_repo.find_by_id.return_value = user

# pytest-mock (mocker fixture)
def test_something(mocker):
    mock_fn = mocker.patch('app.services.some_function')
```

**Source file conventions**

```python
# Dependency injection

# Pattern A: FastAPI Depends
async def get_user(
    user_id: str,
    user_service: UserService = Depends(get_user_service)
):

# Pattern B: constructor injection (class-based)
class UserService:
    def __init__(self, repository: UserRepository):
        self._repository = repository

# Error handling
raise HTTPException(status_code=404, detail="User not found")  # direct
raise UserNotFoundException(user_id)                             # custom exception
return None  # return None, handle in router
```

**Naming conventions**

```python
# File names: snake_case
user_service.py, user_repository.py, test_user_service.py

# Class names: PascalCase
class UserService, class UserRepository

# Function/method names: snake_case
def get_user(self, user_id: str)
async def find_by_id(self, user_id: str)
```

---

## Convention Priority Rules

When existing code conflicts with the default guide:

```
Priority 1: Patterns directly observed in existing project code
Priority 2: Code style config files (.eslintrc, detekt, etc.)
Priority 3: Default patterns from tdd-test-writing-guide.md
```

**Example — default pattern vs project pattern conflict:**

```typescript
// tdd-test-writing-guide.md default: vi.mock() module approach
vi.mock('../services/userService')

// But if the project consistently uses constructor injection:
const mockRepo = { findById: vi.fn() }
const sut = new UserService(mockRepo as UserRepository)
// → Follow the project's approach
```

---

## Caveats

### Prefer recently modified files

- When selecting test files, prefer the most recently modified ones
- If legacy and modern patterns coexist, follow the more prevalent one
- When uncertain, ask the developer: "I see both pattern A and pattern B in the codebase. Which do you prefer for new code?"

### Fallback on detection failure

- If a file was read but a specific pattern cannot be determined: use the default pattern, then revisit in REFACTOR
- Never halt the session progress due to a detection failure

### When the developer writes code that differs from conventions

If the developer pastes a test in RED that does not match detected conventions:
- If the test intent is clear, proceed as-is (flag the convention difference in REFACTOR)
- In REFACTOR: "Existing tests use `vi.mock()` but this test uses direct injection. Would you like to align with the project convention?"
