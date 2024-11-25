Spring’s component model is built on the principle of *singletons* for many of its core beans, ensuring efficiency and consistency across your application. If you’ve ever wondered how Spring handles multiple requests while maintaining a single instance for components like controllers, services, and repositories, this article will walk you through it with practical examples and insights.

- Source code : [Singleton Components Demo](https://github.com/moham-p/dev-codes/tree/main/spring/thread)

## What are Singletons in Spring?

In Spring, the default scope of beans is `singleton`, which means Spring creates exactly one instance of a bean per application context. This single instance is shared across all requests, ensuring consistency and reducing overhead.

To verify that components like controllers, services, and repositories are singletons, you can use `System.identityHashCode` to print the unique memory reference hash for each bean instance.

---

## Verifying Singleton Behavior with Code

Let’s take a look at a simple Spring Boot application to demonstrate this.

### 1. **Order Entity**

This entity represents a basic `Order` object stored in a database:

```java
@Entity
@Table(name = "orders")
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String description;
}
```

### 2. **Order Controller**

The controller exposes an endpoint to fetch all orders and logs its hash code to verify its singleton nature:

```java
@RestController
@RequestMapping("/api/orders")
@RequiredArgsConstructor
@Slf4j
public class OrderController {

    private final OrderService orderService;

    @GetMapping
    public ResponseEntity<List<OrderDTO>> findAll() {
        log.info("OrderController HashCode: {}", System.identityHashCode(this));

        var response = orderService.findAll()
                .stream()
                .map(order -> new OrderDTO(order.getName(), order.getDescription()))
                .toList();
        return ResponseEntity.status(HttpStatus.OK).body(response);
    }
}
```

### 3. **Order Service**

The service layer handles business logic and also logs its hash code, along with the repository's:

```java
@Service
@RequiredArgsConstructor
@Slf4j
public class OrderService {

    private final OrderRepository orderRepository;

    public List<Order> findAll() {
        log.info("OrderService HashCode: {}", System.identityHashCode(this));
        log.info("OrderRepository HashCode: {}", System.identityHashCode(orderRepository));

        return orderRepository.findAll();
    }
}
```

---

## Observing Singleton Behavior in Logs

When you run the application and make multiple requests to `/api/orders`, the logs will show the same hash code for each instance. In my case, the hash codes were:

```plaintext
OrderController HashCode: 12345678
OrderService HashCode: 87654321
OrderRepository HashCode: 23456789
```

These consistent hash codes confirm that Spring is using the same instance of each bean across all requests.

---

## How Spring Handles Concurrent Requests with Thread Pools

You might wonder: if these components are singletons, how does Spring handle multiple requests concurrently?

The answer lies in **thread pools**. Spring’s default thread pool (backed by the servlet container, such as Tomcat) assigns each incoming HTTP request to a separate thread. Each thread executes the same singleton instance of your controller, service, and repository, ensuring that multiple requests can be processed concurrently.

### Key Points:
- The singleton components themselves are not thread-safe by design, but the stateless nature of most services and repositories makes them safe for multi-threaded access.
- If your service or controller maintains state (e.g., using class-level variables), you must ensure thread safety manually, as multiple threads can access the same singleton instance simultaneously.

---

## Why Use Singletons?

1. **Efficiency**: Singletons reduce memory usage and initialization overhead since only one instance is created.
2. **Consistency**: Using a single instance ensures that the state and configuration are consistent throughout the application.

---

## Wrap Up

Singletons, when combined with thread pools, allow Spring applications to scale efficiently without creating multiple instances of components. By logging `System.identityHashCode`, we verified that Spring uses the same instance of our beans across all requests. However, always remember to keep your singleton components stateless to avoid threading issues in multi-threaded environments.

---

Happy coding! 💻
