# Java Unit Test Writing Workflow

## Overview

Before writing any test, read the existing tests in this repository to understand
the established conventions. Never invent a style from scratch.

---

## Step 1 — Read Existing Tests First

Always start by reading the two canonical test files:

- `order-service/src/test/java/com/example/orders/service/OrderServiceTest.java`
- `order-service/src/test/java/com/example/orders/controller/OrderControllerTest.java`

Identify and note:
- Which assertion library is used and how (AssertJ static imports)
- Which mocking annotations are used (`@Mock`/`@InjectMocks` vs `@MockBean`)
- The test method naming pattern (`methodName_condition_expectedBehavior`)
- How fixtures are constructed in `@BeforeEach`
- Import style (static imports grouped at the bottom of imports)

---

## Step 2 — Read the Production Class Under Test

Read the source file you are about to test so you understand:
- All public methods and their signatures
- What the method delegates to (repositories, other services)
- Any exception paths or status-transition guards

---

## Step 3 — Plan the Test Cases

Before writing code, list the cases to cover:
- **Happy path** — valid input, expected return value
- **Not-found / empty** — repository returns empty Optional
- **Validation / exception path** — illegal state, bad argument
- **Side-effect verification** — confirm `verify(mock).method()` where relevant

---

## Step 4 — Write the Test Class

Follow the conventions observed in Step 1:

1. Same package as the class under test (in `src/test/java/`)
2. `@ExtendWith(MockitoExtension.class)` for pure service tests
3. `@WebMvcTest(Controller.class)` for controller slice tests
4. Fixtures built with plain setters in `@BeforeEach void setUp()`
5. One `@Test` method per case, named `methodName_condition_expectedBehavior`
6. AssertJ for all assertions — `assertThat(...)`, `assertThatThrownBy(...)`
7. Mockito stubbing with `when(...).thenReturn(...)` / `when(...).thenThrow(...)`
8. Interaction verification with `verify(mock).method(...)` where meaningful

---

## Step 5 — Validate

After writing, review the new test class and confirm:

- [ ] It compiles (imports are correct, no missing classes)
- [ ] It lives in the correct package directory
- [ ] Every test method has a clear assertion — no empty test bodies
- [ ] No JUnit 4 (`@RunWith`, `Assert.*`) or Hamcrest (`assertThat` from Hamcrest) used
- [ ] Static imports are used for AssertJ and Mockito helpers
