# Test Patterns — Copy-Paste Templates

These patterns match what is already in this repo. Pick the right skeleton and fill in
the blanks. Do not change the structural choices (annotation, import style, fixture approach).

---

## Pattern 1 — Service Unit Test (pure Mockito)

Use for any class annotated `@Service` that has repository dependencies.

```java
package com.example.orders.service;

import com.example.orders.model.Order;
import com.example.orders.repository.OrderRepository;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import java.math.BigDecimal;
import java.util.Optional;

import static org.assertj.core.api.Assertions.assertThat;
import static org.assertj.core.api.Assertions.assertThatThrownBy;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.Mockito.verify;
import static org.mockito.Mockito.when;

@ExtendWith(MockitoExtension.class)
class MyServiceTest {

    @Mock
    private OrderRepository orderRepository;   // replace with actual dependency

    @InjectMocks
    private MyService myService;               // class under test

    private Order testOrder;

    @BeforeEach
    void setUp() {
        testOrder = new Order();
        testOrder.setId(1L);
        testOrder.setCustomerName("John Doe");
        testOrder.setProduct("Widget");
        testOrder.setAmount(new BigDecimal("29.99"));
        testOrder.setStatus("PENDING");
    }

    @Test
    void methodName_happyPath_returnsExpected() {
        when(orderRepository.findById(1L)).thenReturn(Optional.of(testOrder));

        // call the method under test
        // assertThat(result)...
    }

    @Test
    void methodName_notFound_returnsEmpty() {
        when(orderRepository.findById(99L)).thenReturn(Optional.empty());

        // assertThat(result).isEmpty();
    }

    @Test
    void methodName_invalidInput_throwsException() {
        when(orderRepository.findById(1L)).thenReturn(Optional.of(testOrder));

        assertThatThrownBy(() -> myService.someMethod(1L, "BAD_VALUE"))
                .isInstanceOf(IllegalStateException.class)
                .hasMessageContaining("expected fragment");
    }

    @Test
    void methodName_sideEffect_callsRepository() {
        myService.deleteMethod(1L);
        verify(orderRepository).deleteById(1L);
    }
}
```

---

## Pattern 2 — Controller Slice Test (@WebMvcTest)

Use for `@RestController` classes.

```java
package com.example.orders.controller;

import com.example.orders.model.Order;
import com.example.orders.service.OrderService;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

import java.math.BigDecimal;
import java.util.Optional;

import static org.mockito.ArgumentMatchers.any;
import static org.mockito.Mockito.when;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@WebMvcTest(MyController.class)           // replace with actual controller
class MyControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private OrderService orderService;    // replace with actual service

    @Autowired
    private ObjectMapper objectMapper;

    @Test
    void getById_existingId_returns200() throws Exception {
        Order order = new Order();
        order.setId(1L);
        order.setCustomerName("Jane");
        order.setProduct("Gadget");
        order.setAmount(new BigDecimal("49.99"));
        order.setStatus("PENDING");

        when(orderService.getOrderById(1L)).thenReturn(Optional.of(order));

        mockMvc.perform(get("/api/orders/1"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.customerName").value("Jane"));
    }

    @Test
    void getById_nonExistingId_returns404() throws Exception {
        when(orderService.getOrderById(99L)).thenReturn(Optional.empty());

        mockMvc.perform(get("/api/orders/99"))
                .andExpect(status().isNotFound());
    }

    @Test
    void create_validPayload_returns201() throws Exception {
        Order order = new Order();
        order.setCustomerName("Bob");
        order.setProduct("Thing");
        order.setAmount(new BigDecimal("19.99"));
        order.setStatus("PENDING");

        when(orderService.createOrder(any(Order.class))).thenReturn(order);

        mockMvc.perform(post("/api/orders")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(objectMapper.writeValueAsString(order)))
                .andExpect(status().isCreated())
                .andExpect(jsonPath("$.customerName").value("Bob"));
    }

    @Test
    void create_missingFields_returns400() throws Exception {
        Order order = new Order(); // no required fields set

        mockMvc.perform(post("/api/orders")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(objectMapper.writeValueAsString(order)))
                .andExpect(status().isBadRequest());
    }
}
```

---

## Pattern 3 — Exception Path (assertThatThrownBy)

```java
assertThatThrownBy(() -> service.updateOrderStatus(1L, "DELIVERED"))
        .isInstanceOf(IllegalStateException.class)
        .hasMessageContaining("Cannot transition from PENDING to DELIVERED");
```

Chain `.isInstanceOf(...)` and `.hasMessageContaining(...)` on the same
`assertThatThrownBy` — do not split into separate assertions.
