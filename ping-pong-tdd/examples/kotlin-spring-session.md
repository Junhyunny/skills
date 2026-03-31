# Example Session: Kotlin + Spring

A ping-pong TDD session for a payment intent creation API endpoint.

---

## Invocation

```
/ping-pong-tdd "As a customer, I want to initiate a payment for my order so that I can complete my purchase. The system creates a payment intent and returns a client secret for the frontend to use."
```

(Full story content provided directly — skips Phase 1 MCP fetch)

---

## Phase 2: Collaborative Planning

**AI:**

```markdown
---
## 🏓 Session State
| Field        | Value |
|--------------|-------|
| Story        | Payment Intent Creation |
| Stack        | Kotlin + Spring |
| Test FW      | JUnit5 + MockK |
| E2E FW       | RestAssured |
| Conventions  | new project — default patterns |
| Phase        | PLANNING |
| Current Task | — |
| Turn         | — |
| Next Action  | Propose task breakdown |
---

## Task Plan (Draft)

### Story Summary
An API that creates a payment intent so the customer can initiate payment for an order.
Returns a client secret that the frontend uses to complete the payment.

### Task List

#### Task 1: PaymentService.createIntent
- **Type:** unit
- **The test will assert:** createIntent(orderId, amount) calls PaymentGateway and returns a PaymentIntent
- **Implementation scope:** PaymentIntent data class, PaymentService, PaymentGateway interface
- **Acceptance criteria link:** Core payment intent creation logic
- **Dependencies:** none

#### Task 2: PaymentRepository — save intent
- **Type:** unit
- **The test will assert:** save(intent) persists to DB and returns the saved entity
- **Implementation scope:** PaymentRepository, PaymentEntity JPA entity
- **Acceptance criteria link:** Payment state persistence
- **Dependencies:** Task 1

#### Task 3: POST /api/payments/intent endpoint
- **Type:** integration (@WebMvcTest)
- **The test will assert:** valid request returns 201 with clientSecret
- **Implementation scope:** PaymentController, CreatePaymentIntentRequest DTO, PaymentIntentResponse DTO
- **Acceptance criteria link:** API contract
- **Dependencies:** Task 1

#### Task 4 (E2E): Payment intent creation full flow
- **Type:** e2e (RestAssured + @SpringBootTest)
- **The test will assert:** POST request against the real app returns 201 with a valid clientSecret
- **Implementation scope:** E2E test only
- **Dependencies:** Task 3

---
Please review the plan.
- You can add, remove, reorder, or adjust the scope of any task
- When ready, type **"ready"**, **"go"**, or **"approved"**
```

**Developer:** Task 2 should be an integration test using Testcontainers.

**AI:**

```markdown
## Task Plan (Revised)

#### Task 2: PaymentRepository — save intent ← updated
- **Type:** integration (Testcontainers)
- **The test will assert:** save(intent) persists to real PostgreSQL and is retrievable
- **Implementation scope:** PaymentRepository (Spring Data), PaymentEntity, DB migration
- **Dependencies:** Task 1

All other tasks remain the same. When ready, type **"ready"**.
```

**Developer:** ready

---

## Phase 3: TDD Loop — Task 1

**AI:**

```
## ✅ Plan Confirmed

Starting Phase 3. Task 1: PaymentService.createIntent

Who writes the first failing test?
→ **"me"** — I'll write it
→ **"you"** — AI writes it
```

**Developer:** me

### Cycle 1 — Developer writes test, AI implements

**AI:**

```markdown
---
## 🏓 Session State
| Field        | Value |
|--------------|-------|
| Story        | Payment Intent Creation |
| Stack        | Kotlin + Spring |
| Test FW      | JUnit5 + MockK |
| E2E FW       | RestAssured |
| Conventions  | new project — default patterns |
| Phase        | RED |
| Current Task | 1 of 4: PaymentService.createIntent |
| Turn         | DEV |
| Next Action  | Developer writes a failing unit test for createIntent |
---

### Task Progress
| # | Title | Type | Status |
|---|-------|------|--------|
| 1 | PaymentService.createIntent | unit | 🔄 active |
| 2 | PaymentRepository — save intent | integration | ⏳ pending |
| 3 | POST /api/payments/intent | integration | ⏳ pending |
| 4 | Payment intent creation full flow | e2e | ⏳ pending |

## 🔴 RED — Your Turn

Task 1: Write a failing test that verifies PaymentService creates a payment intent via PaymentGateway.

Paste your test code here.
```

**Developer:**
```kotlin
@DisplayName("PaymentService")
class PaymentServiceTest {

    private val paymentGateway = mockk<PaymentGateway>()
    private val paymentRepository = mockk<PaymentRepository>()
    private val sut = PaymentService(paymentGateway, paymentRepository)

    @Test
    @DisplayName("should create payment intent and return client secret")
    fun `should create payment intent and return client secret`() {
        // Arrange
        val orderId = OrderId("order-123")
        val amount = Money(10000L, Currency.KRW)
        val expectedIntent = PaymentIntent(
            id = PaymentIntentId("pi_123"),
            orderId = orderId,
            amount = amount,
            clientSecret = "pi_123_secret_abc",
            status = PaymentStatus.PENDING
        )

        every { paymentGateway.createIntent(orderId, amount) } returns expectedIntent
        every { paymentRepository.save(expectedIntent) } returns expectedIntent

        // Act
        val result = sut.createIntent(orderId, amount)

        // Assert
        assertThat(result.clientSecret).isEqualTo("pi_123_secret_abc")
        assertThat(result.status).isEqualTo(PaymentStatus.PENDING)
        verify { paymentGateway.createIntent(orderId, amount) }
        verify { paymentRepository.save(expectedIntent) }
    }
}
```

**AI (GREEN — AI implements):**

```markdown
---
## 🏓 Session State
| Field        | Value |
|--------------|-------|
| Phase        | GREEN |
| Current Task | 1 of 4: PaymentService.createIntent |
| Turn         | AI |
| Next Action  | AI writes minimum implementation to pass the test |
---

## 🟢 GREEN — AI Writing

**Files:**

`src/main/kotlin/com/example/payment/domain/PaymentIntent.kt`
```kotlin
data class PaymentIntent(
    val id: PaymentIntentId,
    val orderId: OrderId,
    val amount: Money,
    val clientSecret: String,
    val status: PaymentStatus
)

enum class PaymentStatus { PENDING, COMPLETED, FAILED }
data class PaymentIntentId(val value: String)
data class OrderId(val value: String)
data class Money(val amount: Long, val currency: Currency)
enum class Currency { KRW, USD }
```

`src/main/kotlin/com/example/payment/domain/PaymentGateway.kt`
```kotlin
interface PaymentGateway {
    fun createIntent(orderId: OrderId, amount: Money): PaymentIntent
}
```

`src/main/kotlin/com/example/payment/domain/PaymentRepository.kt`
```kotlin
interface PaymentRepository {
    fun save(intent: PaymentIntent): PaymentIntent
}
```

`src/main/kotlin/com/example/payment/domain/PaymentService.kt`
```kotlin
class PaymentService(
    private val paymentGateway: PaymentGateway,
    private val paymentRepository: PaymentRepository
) {
    fun createIntent(orderId: OrderId, amount: Money): PaymentIntent {
        val intent = paymentGateway.createIntent(orderId, amount)
        return paymentRepository.save(intent)
    }
}
```

Run all tests. Type **"green confirmed"** when all pass.
```

**Developer:** green confirmed

**AI (REFACTOR):**

```markdown
---
## 🏓 Session State
| Phase | REFACTOR | Turn | — |
---

## 🔧 REFACTOR

Code reviewed.

**Observation:** PaymentService has two responsibilities — gateway delegation and persistence.
This is intentional (orchestration service pattern) and does not need refactoring.

Domain value objects are cleanly separated. ✅

Are all tests still GREEN? Type **"green"** to continue.
```

**Developer:** green

### Cycle 2 — AI writes test, Developer implements (roles swap)

**AI (RED):**

```markdown
---
## 🏓 Session State
| Field        | Value |
|--------------|-------|
| Phase        | RED |
| Current Task | 1 of 4: PaymentService.createIntent |
| Turn         | AI |
| Next Action  | AI writes failing test for gateway error handling |
---

## 🔴 RED — AI Writing

**File:** `src/test/kotlin/com/example/payment/domain/PaymentServiceTest.kt` (add)

```kotlin
@Test
@DisplayName("should throw PaymentException when gateway fails")
fun `should throw PaymentException when gateway fails`() {
    // Arrange
    val orderId = OrderId("order-999")
    val amount = Money(10000L, Currency.KRW)
    every { paymentGateway.createIntent(orderId, amount) } throws RuntimeException("Gateway timeout")

    // Act & Assert
    assertThrows<PaymentException> {
        sut.createIntent(orderId, amount)
    }
}
```

**Expected failure:**
```
class com.example.payment.domain.PaymentException not found
  ← PaymentException class does not exist yet
```

Create an empty `PaymentException` class first to resolve the compile error,
then run the test — the assertion failure is the correct RED state.

Type **"red confirmed"** when the test is failing for the right reason.
```

**Developer:** red confirmed

**AI:**

```markdown
---
## 🏓 Session State
| Phase | GREEN | Turn | DEV |
---

## 🟢 GREEN — Your Turn

Write the minimum code to make the test throw `PaymentException`.

Type **"done"** when all tests pass.
```

**Developer:** done

*(Session continues for Tasks 2, 3, 4 — omitted for brevity)*

---

## Phase 4: Review & Commit

**AI:**

```markdown
## Changes Summary

### Completed Tasks
- ✅ Task 1: PaymentService.createIntent — 2 tests, 4 implementation files
- ✅ Task 2: PaymentRepository (Testcontainers) — 2 tests, 2 files + migration
- ✅ Task 3: POST /api/payments/intent — 2 integration tests, 1 controller
- ✅ Task 4 (E2E): Payment intent creation full flow — 1 E2E test

### Modified Files
- `src/main/kotlin/com/example/payment/domain/PaymentIntent.kt` (new)
- `src/main/kotlin/com/example/payment/domain/PaymentService.kt` (new)
- `src/main/kotlin/com/example/payment/domain/PaymentGateway.kt` (new)
- `src/main/kotlin/com/example/payment/domain/PaymentRepository.kt` (new)
- `src/main/kotlin/com/example/payment/domain/PaymentException.kt` (new)
- `src/main/kotlin/com/example/payment/web/PaymentController.kt` (new)
- `src/test/kotlin/com/example/payment/domain/PaymentServiceTest.kt` (new)
- `src/test/kotlin/com/example/payment/web/PaymentControllerTest.kt` (new)
- `src/test/kotlin/com/example/payment/e2e/PaymentApiTest.kt` (new)

---
**Commit message:**
```
feat(payment): add payment intent creation endpoint

Tasks completed:
- PaymentService.createIntent: gateway delegation with error handling
- PaymentRepository (Testcontainers): persistence with PostgreSQL
- POST /api/payments/intent: 201 with clientSecret in response
- E2E: full payment intent creation flow verified with RestAssured

TDD: ping-pong pair programming session
```

Type **"commit"** to execute.
```

**Developer:** commit

**AI:** Committed. `def5678 feat(payment): add payment intent creation endpoint`
