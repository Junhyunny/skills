# Tech Stack Detection Guide

Phase 2 identifies the project in two steps.

**Step 1 — Stack detection (this file):** Determine the tech stack and frameworks from build files and dependencies.
**Step 2 — Convention detection (`convention-detection.md`):** Read existing source/test files to understand coding style.

Stack detection always runs. Convention detection only runs when existing code files are present.

---

Detect the tech stack at the start of Phase 2 by reading project root files. Apply rules in this order — higher rules have higher confidence.

---

## Detection Order

### 1. TypeScript + React

**Primary signal:** `package.json` exists.

Read `package.json` and check `dependencies` and `devDependencies`:

| Field presence | Meaning |
|----------------|---------|
| `"react"` in deps | React project |
| `"typescript"` or `.ts`/`.tsx` files exist | TypeScript confirmed |
| `"vitest"` in devDeps | Unit test FW: Vitest |
| `"jest"` in devDeps | Unit test FW: Jest |
| `"@testing-library/react"` | Component testing: RTL |
| `"@playwright/test"` or `playwright.config.*` | E2E: Playwright |
| `"cypress"` or `cypress.config.*` | E2E: Cypress |
| `"next"` in deps | Next.js (React) |
| `"vite"` in devDeps | Vite toolchain |

**Confirmed stack:** TypeScript + React
**Unit FW:** Vitest (if present) else Jest
**E2E FW:** Playwright (if present) else Cypress (if present) else none detected

---

### 2. Kotlin + Spring

**Primary signal:** `build.gradle.kts` exists at project root.

Read the file and check:

| Signal | Meaning |
|--------|---------|
| `kotlin("jvm")` or `id("org.jetbrains.kotlin.jvm")` | Kotlin confirmed |
| `org.springframework.boot` plugin | Spring Boot |
| `io.mockk:mockk` in test deps | MockK |
| `org.testcontainers` in deps | Testcontainers available |
| `io.rest-assured` or `io.restassured` | RestAssured |
| `src/main/kotlin/` directory exists | Kotlin source root |
| `src/test/kotlin/` directory exists | Kotlin test root |

**Confirmed stack:** Kotlin + Spring
**Unit FW:** JUnit5 + MockK
**E2E FW:** RestAssured (or Spring MockMvc for controller tests)

---

### 3. Java + Spring

**Primary signals:** `build.gradle` (not `.kts`) + `src/main/java/` directory, OR `pom.xml` + Java source.

**Gradle signals (`build.gradle`):**

| Signal | Meaning |
|--------|---------|
| `org.springframework.boot` plugin | Spring Boot |
| `org.mockito` in test deps | Mockito |
| `org.testcontainers` in deps | Testcontainers |
| `io.rest-assured` or `io.restassured` | RestAssured |
| `src/main/java/` exists | Java confirmed |

**Maven signals (`pom.xml`):**

| Signal | Meaning |
|--------|---------|
| `spring-boot-starter-*` dependency | Spring Boot |
| `mockito-core` dependency | Mockito |
| `testcontainers` dependency | Testcontainers |
| `rest-assured` dependency | RestAssured |
| `src/main/java/` exists | Java confirmed |

**Confirmed stack:** Java + Spring
**Unit FW:** JUnit5 + Mockito
**E2E FW:** RestAssured

---

### 4. Python + FastAPI

**Primary signals:** `pyproject.toml` or `requirements.txt` or `requirements-dev.txt`.

Read the file and check:

| Signal | Meaning |
|--------|---------|
| `fastapi` dependency | FastAPI confirmed |
| `pytest` dependency | Test FW: pytest |
| `httpx` dependency | HTTP test client |
| `pytest-asyncio` dependency | Async test support |
| `anyio` dependency | Async runtime |
| `playwright` (Python) | E2E: Playwright |
| `uvicorn` dependency | ASGI server |

**Confirmed stack:** Python + FastAPI
**Unit FW:** pytest + httpx (+ pytest-asyncio if async)
**E2E FW:** Playwright (Python) if present

---

## Ambiguity Resolution

### Multiple frameworks detected

If both `build.gradle.kts` (Kotlin) and `package.json` (React) exist:
→ This is a monorepo. Ask: "This looks like a monorepo. Which service are we working on? (frontend / backend)"

If both `build.gradle` and `pom.xml` exist:
→ Unusual. Ask: "Both Gradle and Maven build files found. Which build tool is in use? (gradle / maven)"

### No signals found

If none of the above primary signals are detected:
→ Ask: "Could not detect the project stack. Please specify:
   1. TypeScript + React
   2. Kotlin + Spring
   3. Java + Spring
   4. Python + FastAPI
   5. Other (describe)"

### Developer override

At any point, the developer can type `stack: [name]` to override detection:
- `stack: typescript-react` → TypeScript + React
- `stack: kotlin-spring` → Kotlin + Spring
- `stack: java-spring` → Java + Spring
- `stack: python-fastapi` → Python + FastAPI

---

## Display Format

After detection, display in the planning proposal:

```
**Detected stack:** TypeScript + React
**Unit test framework:** Vitest + @testing-library/react
**E2E test framework:** Playwright
```

If uncertain:
```
**Detected stack:** TypeScript + React (inferred — found `"vitest"` and `"react"`)
Type `stack: [name]` to override if this is incorrect.
```
