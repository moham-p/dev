Understanding transactions is a foundational skill in database systems. Transactions are critical for ensuring database reliability and consistency. By exploring transactions, you can:

- Build robust, error-resilient applications.
- Debug database behaviors with confidence.
- Design systems that handle concurrency and consistency gracefully.

Curiosity about transactions is a gateway to mastering how databases work behind the scenes. PostgreSQL’s handling of transactions, even for simple queries, offers insights into how databases maintain ACID properties (Atomicity, Consistency, Isolation, Durability).

- Source code : [Spring boot / JPA / PostgreSQL](https://github.com/moham-p/dev-codes/tree/main/database/trans)

## Exploring PostgreSQL Transactions with a Simple SELECT Query in Spring Boot

When we think about database transactions, the focus is often on `INSERT`, `UPDATE`, or `DELETE` queries. But did you know that even a simple `SELECT` query in PostgreSQL is part of a transaction? Let's build a simple demo project to verify this behavior.

### Code Setup: A Minimal Spring Boot Example

Here’s a simple setup to illustrate the concept:

#### Entity
```java
@Entity
@Data
public class AccountEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private int balance;
}
```

#### Repository
```java
public interface AccountRepository extends JpaRepository<AccountEntity, Long> {
}
```

#### Service
```java
@Service
@AllArgsConstructor
public class AccountService {

    private final AccountRepository accountRepository;

    public List<AccountEntity> getAllAccounts() {
        return accountRepository.findAll();
    }
}
```

#### Controller
```java
@RestController
@AllArgsConstructor
public class AccountController {

    private final AccountService accountService;

    @GetMapping("/accounts")
    public List<AccountEntity> getAllAccounts() {
        return accountService.getAllAccounts();
    }
}
```

To observe transactions in action, enable logging in `postgresql.conf`:

```plaintext
log_statement = 'all'
```

Now, make a request to:
```plaintext
GET localhost:8080/accounts
```

Here’s what you’ll see in the PostgreSQL logs:

```plaintext
execute <unnamed>: BEGIN READ ONLY
execute <unnamed>: select ae1_0.id,ae1_0.balance,ae1_0.name from account_entity ae1_0
execute <unnamed>: COMMIT
```
Notice that the query is executed between BEGIN READ ONLY and COMMIT, which marks the boundaries of the transaction.

## PostgreSQL Transactional Nature

In PostgreSQL, every query is executed in a transaction, whether explicitly declared or not. We didn’t explicitly define `@Transactional` in our service method `getAllAccounts`. However, PostgreSQL’s default **autocommit mode** implicitly wraps the query in its own transaction and commits it immediately after the query finishes.

If you want to explicitly define transactions in your service methods that deal with reading data, you can do so by using the annotation `@Transactional(readOnly = true)`.

### Why Autocommit Exists
- PostgreSQL ensures that all queries are atomic, consistent, isolated, and durable (ACID properties).
- Even a `SELECT` benefits from this, as it guarantees the query operates on a consistent snapshot of the database.

## Why Consistency Matters for SELECT Queries

A **snapshot** in a transaction refers to the consistent view of the database that a query or set of queries operates on. Don’t forget that `SELECT` operations can include complex joins across many tables. 
These operations are executed as a single `SELECT` query (PostgreSQL does not split a complex join into subqueries).

A complex join involves multiple tables and potentially millions of rows. Without a consistent snapshot, intermediate states (caused by concurrent updates, inserts, or deletes) could lead to:

- **Inconsistent Results**: The query might see partial updates from other transactions.
- **Phantom Reads**: Rows that didn’t exist at the start of the query might suddenly appear due to concurrent inserts.

Therefore, Having a consistent snapshot of the database during execution is critical for maintaining data accuracy and consistency, especially in environments with concurrent transactions.

In the next part, we continue our curiosity-driven journey by exploring the behavior of update queries.

---

Happy coding! 💻
