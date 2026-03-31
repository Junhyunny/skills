# Commit Message Template

Use this template when generating the commit message in Phase 4.

---

## Standard Session Commit

```
feat(<scope>): <short description under 72 chars>

Implements <STORY-ID>: <story title>

Tasks completed:
- <task 1 title>: <one-line description of what was implemented>
- <task 2 title>: <one-line description>
- <task 3 title>: <one-line description>

TDD: ping-pong pair programming session
```

---

## Examples

### TypeScript + React feature

```
feat(user-profile): add profile name and email update

Implements PROJ-123: User Profile Update

Tasks completed:
- UserService.updateProfile: validates and persists name/email changes
- UserController PATCH /api/users/:id: request parsing and response formatting
- ProfileForm component: edit form with inline validation messages
- E2E: edit profile flow from form submission to success confirmation

TDD: ping-pong pair programming session
```

### Kotlin + Spring feature

```
feat(payment): add payment intent creation endpoint

Implements PROJ-456: Payment Intent Creation API

Tasks completed:
- PaymentService.createIntent: creates Stripe payment intent with amount validation
- PaymentRepository.save: persists intent with idempotency key
- POST /api/payments/intent: 201 response with client secret
- E2E: payment intent creation with valid card details returns client secret

TDD: ping-pong pair programming session
```

### Python + FastAPI bug fix

```
fix(orders): correct rounding error in order total calculation

Fixes PROJ-789: Payment amount decimal rounding error

Root cause: float arithmetic used for monetary values; replaced with
integer cents throughout the calculation pipeline.

Tasks completed:
- OrderCalculator.calculate_total: all values in integer cents
- OrderCalculator tests: added rounding edge cases

TDD: ping-pong pair programming session
```

---

## Scope Selection Guide

| Stack | Good scope examples |
|-------|-------------------|
| TypeScript + React | `user-profile`, `cart`, `checkout`, `auth`, `dashboard` |
| Kotlin + Spring | `user-service`, `payment`, `order`, `notification`, `auth` |
| Java + Spring | same as Kotlin |
| Python + FastAPI | `users`, `products`, `orders`, `auth`, `analytics` |

Use the feature/module name that best describes what was changed, not the technical layer.
