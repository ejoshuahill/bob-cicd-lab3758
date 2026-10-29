# Java Unit Test Best Practices

## Naming Convention

Use the three-part snake-in-camel pattern:

```
methodName_condition_expectedBehavior()
```

Examples from this repo:

| ✅ Good | ❌ Bad |
|---|---|
| `getOrderById_existingId_returnsOrder` | `testGetOrderById` |
| `updateOrderStatus_invalidTransition_throwsException` | `shouldThrowWhenTransitionIsInvalid` |
| `createOrder_setsDefaultStatus` | `createOrderTest` |

---

## Assertions — AssertJ Only

Always import from `org.assertj.core.api.Assertions`:

```java
import static org.assertj.core.api.Assertions.assertThat;
import static org.assertj.core.api.Assertions.assertThatThrownBy;
```

**Do not use:**
- `org.junit.jupiter.api.Assertions.assertEquals` (JUnit built-ins) — use AssertJ instead
- `org.hamcrest.MatcherAssert.assertThat` — Hamcrest is not in this project

AssertJ patterns used in this repo:

```java
// Value assertions
assertThat(result).isEqualTo("PENDING");
assertThat(result).isPresent();
assertThat(result).isEmpty();
assertThat(list).hasSize(1);

// Exception assertions
assertThatThrownBy(() -> service.updateOrderStatus(1L, "DELIVERED"))
        .isInstanceOf(IllegalStateException.class)
        .hasMessageContaining("Cannot transition");
```

---

## Mocking — Mockito

### Service tests — pure Mockito

```java
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {

    @Mock
    private OrderRepository orderRepository;   // collaborator

    @InjectMocks
    private OrderService orderService;         // class under test
}
```

### Controller tests — Spring slice

```java
@WebMvcTest(OrderController.class)
class OrderControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean                                  // replaces the bean in the app context
    private OrderService orderService;
}
```

**Never** mix `@Mock` + `@InjectMocks` inside a `@WebMvcTest` class, or `@MockBean`
inside a plain `@ExtendWith(MockitoExtension.class)` class.

---

## Fixture Setup

Build test fixtures with plain setters in a `@BeforeEach` method:

```java
@BeforeEach
void setUp() {
    testOrder = new Order();
    testOrder.setId(1L);
    testOrder.setCustomerName("John Doe");
    testOrder.setProduct("Widget");
    testOrder.setAmount(new BigDecimal("29.99"));
    testOrder.setStatus("PENDING");
}
```

- Name the method `setUp()` (matches repo convention)
- Use the fixture object in tests; override individual fields per test when needed
- Do **not** use Lombok builders or factory helpers — the repo does not use them

---

## Static Import Order

Group static imports after regular imports, in this order:

```java
import static org.assertj.core.api.Assertions.assertThat;
import static org.assertj.core.api.Assertions.assertThatThrownBy;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.Mockito.verify;
import static org.mockito.Mockito.when;
```

---

## Common Pitfalls

| Pitfall | What to do instead |
|---|---|
| Using `Assertions.assertEquals` | Use `assertThat(a).isEqualTo(b)` |
| Stubbing calls that are never invoked | Remove unnecessary stubs; Mockito strict mode will fail |
| `verify` without meaning | Only verify side-effects (saves, deletes) — not reads |
| `any()` on a primitive | Use `anyLong()`, `anyInt()`, etc. |
| Missing `throws Exception` on MockMvc tests | Add it — `perform` throws a checked exception |
