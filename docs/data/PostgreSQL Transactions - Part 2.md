In [part one](../PostgreSQL Transactions - Part 1), we explained how **SELECT** queries benefit from running as a transaction. In this part, let’s explore why transactions are essential for **UPDATE** statements.

We mentioned earlier that **autocommit mode** is the default in PostgreSQL. This means that if you do not explicitly use **BEGIN** to start a transaction, each individual SQL statement (**SELECT**, **INSERT**, **UPDATE**, **DELETE**, etc.) is treated as a separate transaction. PostgreSQL will automatically commit the statement immediately after its execution.

## Importance of Transactions for UPDATE Statements

Running **UPDATE** statements as transactions is essential because modifications to a database require ACID compliance to ensure reliability and data integrity.

When using **UPDATE** statements, we need to apply changes as a set of units in an all-or-nothing fashion. This is achieved through the **Atomicity** property of transactions. Additionally, if something goes wrong during the update, it is critical to restore the database to a valid state, ensuring data integrity. This is made possible by the **Consistency** property of transactions. The rollback mechanism in transactions prevents the database from being left in an invalid state due to partially completed operations.

Moreover, we rely on the **Isolation** property provided by transactions. Isolation ensures that **SELECT** statements (and other queries) operate on a consistent view of the database, often referred to as a "snapshot."

## PostgreSQL Isolation Levels

The default isolation level in PostgreSQL is **Read Committed**, which ensures that queries only see data that has been committed. However, changes made within a transaction are not visible to other transactions until the transaction is committed. This isolation level is sufficient for most applications.

For use cases that require stricter isolation, PostgreSQL offers two higher levels:

1. **Repeatable Read**: Provides a consistent snapshot of data for the entire duration of a transaction, preventing **non-repeatable reads** (when the same query returns different results due to concurrent updates).
2. **Serializable**: The strictest isolation level, simulating sequential transaction execution. It prevents all concurrency anomalies, including **phantom reads**.

## Phantom Reads and Isolation Challenges

**Phantom Reads** exemplify isolation challenges:
- A transaction retrieves rows with `SELECT * FROM person WHERE age > 30;`.
- If another transaction inserts a row with `age = 35` and commits, rerunning the query would include the new row, creating a "phantom" effect.

While **Repeatable Read** prevents some anomalies, it does not eliminate phantoms. **Serializable isolation**, however, ensures that even these are addressed by effectively serializing transaction execution.

Isolation is particularly important in multi-user environments, where multiple transactions interact and can lead to complex concurrency issues.

## Locking Mechanisms Outside of Transactions

Locks can also be used outside of transactions, providing manual control over database resources. For example, the **LOCK TABLE** statement allows users to explicitly lock a table or specific rows to manage custom workflows, independent of transactional logic.

### Examples of Row-Level Locks:
1. **`SELECT FOR UPDATE`**  
   Locks selected rows to prevent concurrent modifications. Other transactions are blocked from updating or deleting these rows until the lock is released.

2. **`SELECT FOR SHARE`**  
   Acquires a less restrictive lock on rows, allowing other transactions to read the rows but preventing modifications. This is useful when you need to ensure rows are not updated or deleted while being processed, but without blocking other readers.

These locking mechanisms are particularly valuable for fine-tuned control over database operations in complex workflows.

## `SELECT FOR UPDATE` vs. Optimistic Locking

### **`SELECT FOR UPDATE` (Pessimistic Locking)**
This approach is ideal for high-conflict scenarios, such as avoiding double-booking issues. However, it can degrade performance due to contention, as rows are locked and other transactions are blocked from modifying them until the lock is released.

To address these limitations, **optimistic locking** is an alternative strategy that assumes conflicts are rare.

## Optimistic Locking

Optimistic locking avoids locking rows during read operations, allowing other transactions to read or write to the same rows concurrently. It relies on versioning to detect conflicts at the time of committing changes.

### How It Works:
1. A **version column** (or timestamp) is added to the table to track changes to rows.
2. During an **UPDATE**, the query checks if the version column matches the expected value:
    - **If the version matches**, the update proceeds, and the version column is incremented.
    - **If the version does not match**, the update fails, signaling that another transaction has modified the row, resulting in a concurrency exception.

### Example:
**Table Setup with a Version Column**
```sql
CREATE TABLE account (
    id SERIAL PRIMARY KEY,
    balance INT NOT NULL,
    version INT NOT NULL
);
```

**Optimistic Update Query**
```sql
UPDATE account
SET balance = 500, version = version + 1
WHERE id = 1 AND version = 58;
```

If the condition `version = 58` fails (e.g., another transaction has already updated the row and incremented the version), the update will not proceed. This approach is lightweight and works best in scenarios where conflicts are infrequent, as it avoids the overhead of maintaining locks.

## Practical Tips for Transaction Management

1. **Choose the Right Isolation Level**
    - Use **Read Committed** for general-purpose applications, as it balances performance and consistency.
    - Opt for **Repeatable Read** when consistent snapshots are needed, particularly in read-heavy operations.
    - Choose **Serializable** for scenarios requiring the highest level of data integrity, such as financial transactions, where avoiding anomalies is critical.

2. **Leverage Locking Strategically**
    - Use locks judiciously to maintain a balance between performance and consistency.
    - For example, prefer `SELECT FOR UPDATE` in high-conflict situations to prevent double-booking or simultaneous modifications.

3. **Optimize for Concurrency**
    - In high-concurrency environments, favor **optimistic locking** to reduce contention and improve performance.
    - This approach minimizes the overhead of holding locks while still ensuring consistency by checking for conflicts at commit time.

---

Happy coding! 💻
