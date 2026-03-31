# TDD Test Writing Guide

Detailed test patterns for each supported stack. A good failing test:
1. Compiles/parses without syntax errors
2. Fails at runtime for the **right reason** (assertion failure or missing implementation — NOT import error or wrong test setup)
3. Has a descriptive name readable as a specification sentence
4. Is focused on a single behavior

---

## TypeScript + React

### Frameworks
- **Unit/Component:** Vitest + @testing-library/react OR Jest + @testing-library/react
- **E2E:** Playwright OR Cypress

### File Location Conventions
```
src/
  components/
    UserProfile/
      UserProfile.tsx
      UserProfile.test.tsx   ← co-located test
  services/
    userService.ts
    userService.test.ts
e2e/
  user-profile.spec.ts       ← Playwright E2E
```

### Unit Test Anatomy (Vitest)

```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest'
import { render, screen, waitFor } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import { UserProfile } from './UserProfile'
import { userService } from '../services/userService'

vi.mock('../services/userService')

describe('UserProfile', () => {
  beforeEach(() => {
    vi.clearAllMocks()
  })

  it('should display user name when profile loads successfully', async () => {
    // Arrange
    vi.mocked(userService.getUser).mockResolvedValue({ name: 'Alice', email: 'alice@example.com' })

    // Act
    render(<UserProfile userId="123" />)

    // Assert
    await waitFor(() => {
      expect(screen.getByText('Alice')).toBeInTheDocument()
    })
  })

  it('should show error message when profile fetch fails', async () => {
    // Arrange
    vi.mocked(userService.getUser).mockRejectedValue(new Error('Not found'))

    // Act
    render(<UserProfile userId="invalid" />)

    // Assert
    await waitFor(() => {
      expect(screen.getByRole('alert')).toHaveTextContent('Failed to load profile')
    })
  })
})
```

### Service Unit Test (Vitest)

```typescript
import { describe, it, expect, vi } from 'vitest'
import { UserService } from './UserService'
import { UserRepository } from './UserRepository'

describe('UserService', () => {
  it('should return user when user exists', async () => {
    // Arrange
    const mockRepo = {
      findById: vi.fn().mockResolvedValue({ id: '1', name: 'Alice' })
    } as unknown as UserRepository
    const sut = new UserService(mockRepo)

    // Act
    const result = await sut.getUser('1')

    // Assert
    expect(result).toEqual({ id: '1', name: 'Alice' })
    expect(mockRepo.findById).toHaveBeenCalledWith('1')
  })
})
```

### Naming Conventions
- `it('should [expected behavior] when [context]', ...)`
- Use `describe` for the component/service name, `it` for behaviors
- Avoid "test that", "verify that" — start directly with "should"

### What a Correct RED Failure Looks Like
```
FAIL src/services/UserService.test.ts
  UserService > should return user when user exists
    TypeError: sut.getUser is not a function
    ← method does not exist yet → correct RED
```
NOT this (wrong reason):
```
    SyntaxError: Cannot find module './UserService'
    ← file doesn't exist → fix the import, not the test
```

### E2E Test (Playwright)

```typescript
// e2e/user-profile.spec.ts
import { test, expect } from '@playwright/test'

test.describe('User Profile', () => {
  test('should display updated name after profile edit', async ({ page }) => {
    // Arrange — navigate to page
    await page.goto('/profile/123')
    await expect(page.getByRole('heading')).toHaveText('Alice')

    // Act
    await page.getByRole('button', { name: 'Edit Profile' }).click()
    await page.getByLabel('Name').fill('Alice Updated')
    await page.getByRole('button', { name: 'Save' }).click()

    // Assert
    await expect(page.getByRole('heading')).toHaveText('Alice Updated')
    await expect(page.getByRole('status')).toHaveText('Profile updated successfully')
  })
})
```

---

## Kotlin + Spring

### Frameworks
- **Unit:** JUnit5 + MockK + AssertJ (or Kotest)
- **Integration:** @SpringBootTest + MockMvc or Testcontainers
- **E2E:** RestAssured

### File Location Conventions
```
src/
  main/kotlin/com/example/
    domain/
      User.kt
      UserService.kt
    web/
      UserController.kt
  test/kotlin/com/example/
    domain/
      UserServiceTest.kt
    web/
      UserControllerTest.kt
      UserApiTest.kt           ← RestAssured E2E
```

### Unit Test Anatomy

```kotlin
import io.mockk.*
import org.assertj.core.api.Assertions.assertThat
import org.junit.jupiter.api.BeforeEach
import org.junit.jupiter.api.DisplayName
import org.junit.jupiter.api.Nested
import org.junit.jupiter.api.Test
import org.junit.jupiter.api.assertThrows

@DisplayName("UserService")
class UserServiceTest {

    private val userRepository = mockk<UserRepository>()
    private val sut = UserService(userRepository)

    @BeforeEach
    fun setUp() {
        clearAllMocks()
    }

    @Nested
    @DisplayName("getUser")
    inner class GetUser {

        @Test
        @DisplayName("should return user when user exists")
        fun `should return user when user exists`() {
            // Arrange
            val userId = UserId("user-123")
            val expectedUser = User(id = userId, name = "Alice", email = "alice@example.com")
            every { userRepository.findById(userId) } returns expectedUser

            // Act
            val result = sut.getUser(userId)

            // Assert
            assertThat(result).isEqualTo(expectedUser)
            verify(exactly = 1) { userRepository.findById(userId) }
        }

        @Test
        @DisplayName("should throw UserNotFoundException when user does not exist")
        fun `should throw UserNotFoundException when user does not exist`() {
            // Arrange
            val userId = UserId("nonexistent")
            every { userRepository.findById(userId) } returns null

            // Act & Assert
            assertThrows<UserNotFoundException> {
                sut.getUser(userId)
            }
        }
    }
}
```

### Controller Integration Test (@WebMvcTest)

```kotlin
@WebMvcTest(UserController::class)
@DisplayName("UserController")
class UserControllerTest {

    @Autowired private lateinit var mockMvc: MockMvc
    @MockkBean private lateinit var userService: UserService

    @Test
    @DisplayName("should return 200 with user when user exists")
    fun `should return 200 with user when user exists`() {
        // Arrange
        val userId = "user-123"
        every { userService.getUser(UserId(userId)) } returns
            User(id = UserId(userId), name = "Alice", email = "alice@example.com")

        // Act & Assert
        mockMvc.get("/api/users/$userId")
            .andExpect {
                status { isOk() }
                jsonPath("$.name") { value("Alice") }
                jsonPath("$.email") { value("alice@example.com") }
            }
    }

    @Test
    @DisplayName("should return 404 when user does not exist")
    fun `should return 404 when user does not exist`() {
        // Arrange
        every { userService.getUser(any()) } throws UserNotFoundException("not found")

        // Act & Assert
        mockMvc.get("/api/users/invalid")
            .andExpect { status { isNotFound() } }
    }
}
```

### E2E Test (RestAssured)

```kotlin
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@DisplayName("User API")
class UserApiTest {

    @LocalServerPort private var port: Int = 0

    @BeforeEach
    fun setUp() {
        RestAssured.port = port
    }

    @Test
    @DisplayName("should update user name and return 200")
    fun `should update user name and return 200`() {
        // Arrange — create user first
        val userId = createUser("Alice")

        // Act
        given()
            .contentType(ContentType.JSON)
            .body("""{"name": "Alice Updated"}""")
        .`when`()
            .put("/api/users/$userId")
        .then()
            .statusCode(200)
            .body("name", equalTo("Alice Updated"))
    }
}
```

### Naming Conventions
- `@DisplayName("should [behavior] when [context]")`
- Backtick function names in Kotlin for readability: `` `should return user when user exists` ``
- Use `@Nested` with `@DisplayName` to group by method/feature

### What a Correct RED Failure Looks Like
```
UserServiceTest > getUser > should return user when user exists FAILED
  io.mockk.MockKException: no answer found for: UserRepository(#1).findById(UserId(value=user-123))
  ← method doesn't exist in UserService yet → correct RED
```

---

## Java + Spring

### Frameworks
- **Unit:** JUnit5 + Mockito + AssertJ
- **Integration:** @SpringBootTest + MockMvc or Testcontainers
- **E2E:** RestAssured

### File Location Conventions
```
src/
  main/java/com/example/
    domain/UserService.java
    web/UserController.java
  test/java/com/example/
    domain/UserServiceTest.java
    web/UserControllerTest.java
    web/UserApiTest.java
```

### Unit Test Anatomy

```java
import org.junit.jupiter.api.*;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.*;
import org.mockito.junit.jupiter.MockitoExtension;
import static org.assertj.core.api.Assertions.*;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
@DisplayName("UserService")
class UserServiceTest {

    @Mock private UserRepository userRepository;
    @InjectMocks private UserService sut;

    @Nested
    @DisplayName("getUser")
    class GetUser {

        @Test
        @DisplayName("should return user when user exists")
        void shouldReturnUserWhenUserExists() {
            // Arrange
            var userId = new UserId("user-123");
            var expectedUser = new User(userId, "Alice", "alice@example.com");
            when(userRepository.findById(userId)).thenReturn(Optional.of(expectedUser));

            // Act
            var result = sut.getUser(userId);

            // Assert
            assertThat(result).isEqualTo(expectedUser);
            verify(userRepository).findById(userId);
        }

        @Test
        @DisplayName("should throw UserNotFoundException when user does not exist")
        void shouldThrowWhenUserDoesNotExist() {
            // Arrange
            var userId = new UserId("nonexistent");
            when(userRepository.findById(userId)).thenReturn(Optional.empty());

            // Act & Assert
            assertThatThrownBy(() -> sut.getUser(userId))
                .isInstanceOf(UserNotFoundException.class);
        }
    }
}
```

### Controller Integration Test

```java
@WebMvcTest(UserController.class)
@DisplayName("UserController")
class UserControllerTest {

    @Autowired private MockMvc mockMvc;
    @MockBean private UserService userService;

    @Test
    @DisplayName("should return 200 with user data when user exists")
    void shouldReturn200WithUserData() throws Exception {
        // Arrange
        var userId = "user-123";
        when(userService.getUser(new UserId(userId)))
            .thenReturn(new User(new UserId(userId), "Alice", "alice@example.com"));

        // Act & Assert
        mockMvc.perform(get("/api/users/" + userId))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.name").value("Alice"))
            .andExpect(jsonPath("$.email").value("alice@example.com"));
    }
}
```

### Naming Conventions
- `@DisplayName("should [behavior] when [context]")` for test methods
- Method names in camelCase: `shouldReturnUserWhenUserExists`
- Use `@Nested` to group related tests

---

## Python + FastAPI

### Frameworks
- **Unit:** pytest + unittest.mock
- **Integration:** pytest + httpx (AsyncClient with ASGITransport)
- **E2E:** pytest + playwright (optional)

### File Location Conventions
```
app/
  domain/
    user_service.py
  api/
    users.py
    main.py
tests/
  unit/
    test_user_service.py
  integration/
    test_users_api.py
  e2e/
    test_user_profile.py     ← Playwright E2E
```

### Unit Test Anatomy

```python
# tests/unit/test_user_service.py
import pytest
from unittest.mock import AsyncMock, MagicMock, patch
from app.domain.user_service import UserService
from app.domain.exceptions import UserNotFoundException


class TestUserServiceGetUser:
    def setup_method(self):
        self.mock_repository = MagicMock()
        self.sut = UserService(repository=self.mock_repository)

    def test_should_return_user_when_user_exists(self):
        # Arrange
        expected_user = {"id": "user-123", "name": "Alice", "email": "alice@example.com"}
        self.mock_repository.find_by_id.return_value = expected_user

        # Act
        result = self.sut.get_user("user-123")

        # Assert
        assert result == expected_user
        self.mock_repository.find_by_id.assert_called_once_with("user-123")

    def test_should_raise_not_found_when_user_does_not_exist(self):
        # Arrange
        self.mock_repository.find_by_id.return_value = None

        # Act & Assert
        with pytest.raises(UserNotFoundException):
            self.sut.get_user("nonexistent")
```

### Async Unit Test

```python
import pytest
from unittest.mock import AsyncMock

class TestAsyncUserService:
    @pytest.mark.asyncio
    async def test_should_return_user_when_user_exists(self):
        # Arrange
        mock_repo = AsyncMock()
        mock_repo.find_by_id.return_value = {"id": "1", "name": "Alice"}
        sut = AsyncUserService(repository=mock_repo)

        # Act
        result = await sut.get_user("1")

        # Assert
        assert result["name"] == "Alice"
```

### Integration Test (httpx + FastAPI)

```python
# tests/integration/test_users_api.py
import pytest
from httpx import AsyncClient, ASGITransport
from app.main import app


@pytest.mark.asyncio
async def test_should_return_200_with_user_when_user_exists(test_user):
    # test_user is a pytest fixture that creates a user in the test DB
    async with AsyncClient(transport=ASGITransport(app=app), base_url="http://test") as client:
        response = await client.get(f"/api/users/{test_user['id']}")

    assert response.status_code == 200
    assert response.json()["name"] == test_user["name"]


@pytest.mark.asyncio
async def test_should_return_404_when_user_does_not_exist():
    async with AsyncClient(transport=ASGITransport(app=app), base_url="http://test") as client:
        response = await client.get("/api/users/nonexistent-id")

    assert response.status_code == 404
    assert "not found" in response.json()["detail"].lower()
```

### E2E Test (Playwright Python)

```python
# tests/e2e/test_user_profile.py
import pytest
from playwright.async_api import async_playwright

@pytest.mark.asyncio
async def test_should_display_updated_name_after_profile_edit():
    async with async_playwright() as p:
        browser = await p.chromium.launch()
        page = await browser.new_page()

        # Arrange
        await page.goto("http://localhost:3000/profile/123")
        await page.wait_for_selector("h1:has-text('Alice')")

        # Act
        await page.click("button:has-text('Edit Profile')")
        await page.fill("input[name='name']", "Alice Updated")
        await page.click("button:has-text('Save')")

        # Assert
        await page.wait_for_selector("h1:has-text('Alice Updated')")
        assert await page.locator("h1").inner_text() == "Alice Updated"

        await browser.close()
```

### Naming Conventions
- Function names: `test_should_[behavior]_when_[context]`
- Class names: `TestServiceNameMethodName`
- Use `pytest.mark.asyncio` for async tests
- Use `conftest.py` for shared fixtures

### What a Correct RED Failure Looks Like
```
FAILED tests/unit/test_user_service.py::TestUserServiceGetUser::test_should_return_user_when_user_exists
  AttributeError: 'UserService' object has no attribute 'get_user'
  ← method doesn't exist yet → correct RED
```
NOT this (wrong reason):
```
  ImportError: cannot import name 'UserService' from 'app.domain.user_service'
  ← module doesn't exist → create the module first
```

---

## Common Antipatterns to Avoid

| Antipattern | Problem | Fix |
|-------------|---------|-----|
| Testing implementation details | Test breaks on refactor | Test behavior, not method calls |
| Too many assertions in one test | Hard to diagnose failure | One behavior per test |
| Mocking what you own | Hides integration bugs | Mock external deps only |
| Test names like `test1`, `testGetUser` | Not readable as spec | Use full sentence: `should return user when exists` |
| Setup in test body instead of fixture | Duplication | Use `beforeEach` / `setUp` / pytest fixture |
| Asserting on mock call count unnecessarily | Brittle | Assert on outcome, not internals |
