Modern distributed applications rarely resemble the monoliths we built a decade ago. Instead of a single codebase and database, today’s systems are composed of many independently deployed services, often communicating through an event bus such as Apache Kafka.
This architectural shift brings scalability and autonomy—but it fundamentally changes how transactions work.

In a monolith, transactions are straightforward. In distributed systems, they are not.

This post explains why traditional transactions break down, why the Outbox pattern is not enough, and how the Saga pattern addresses long-running, distributed business workflows.


## Life Was Simple in the Monolith

In a monolithic architecture:

* All modules run in a single process
* All data typically lives in a single database
* Business operations execute inside one transactional context

A checkout flow—charging a payment, reserving inventory, sending confirmation—can be wrapped in a single database transaction.
If anything fails, the database rolls everything back.

Two-phase commit (2PC) or even a single local transaction provides:

* Atomicity
* Consistency
* Clear failure semantics

Developers rarely have to think about partial success.


## The Distributed Reality

When applications are decomposed into services:

* Each service owns its own database
* Communication happens via events or remote calls
* Transactions may span multiple services and long time windows

At this point, 2PC becomes impractical:

* It couples services tightly
* It scales poorly
* It introduces blocking and failure amplification

More importantly, many real-world workflows are long-running. You cannot lock databases across services for minutes—or hours—waiting for a business process to complete.

Something has to change.


## The Dual Write Problem

A common challenge in distributed systems is the **dual write problem**:

1. A service updates its database
2. It publishes an event or calls another service
3. One succeeds while the other fails

The result is inconsistency.

For example, the database might successfully store a new order, but the event notifying downstream services might never be published. Those services would then remain unaware that the order exists.


## The Outbox Pattern (and Its Limits)

The **Transactional Outbox pattern** addresses this reliability problem.

Instead of publishing a message directly after updating the database, the service first writes the outgoing message to an **outbox table in the same database transaction** as the business data change. Because both writes occur in a single transaction, they either succeed or fail together.

Once the transaction commits, a separate process retrieves messages from the outbox table and publishes them to the message broker. This publishing step can be implemented in different ways, such as:

* A **polling publisher**, which periodically queries the outbox table for new messages
* **transaction log tailing**, where changes are detected directly from the database’s transaction log

This approach ensures that messages are **reliably delivered** without requiring distributed transactions.

However, it is important to understand the scope of the problem this pattern solves. The Outbox pattern guarantees **reliable communication between services**, but it does not guarantee **business-level consistency across multiple services**.


## When Reliable Messaging Is Not Enough

Consider a checkout process split across services:

* Order Service creates an order
* Inventory Service reserves stock
* Shipping Service creates a label
* Notification Service sends an email

All of these steps must succeed—or the system must undo the ones that already ran.

In a monolith, this is trivial.
In a distributed system, there is no shared transaction to roll back.

This is the class of problems the **Saga pattern** is designed to solve.


## What Is the Saga Pattern?

A **Saga** models a business transaction as a sequence of **local transactions**, each executed by a different service.

Key characteristics:

* Each service commits its own database transaction
* There is no global rollback
* Failures are handled through **compensating actions**

Instead of rolling back, the system explicitly undoes completed steps.

Example:

* Inventory reserved → later failure → inventory is released
* Payment captured → later failure → refund is issued

This shifts responsibility from the database to the application.


## Two Ways to Implement a Saga

### 1. Choreography (Event-Driven Sagas)

In choreography:

* There is no central coordinator
* Each service:

    * Executes its local transaction
    * Publishes an event
    * The next service reacts to that event

**Advantages**

* Highly decoupled
* Naturally fits event-driven architectures
* Scales well

**Trade-offs**

* Business flow is implicit and scattered
* Harder to trace and debug
* Failure handling logic is distributed


### 2. Orchestration (Centralized Control)

In orchestration:

* A **Saga Orchestrator** coordinates the process
* It tells services what to do and when
* It decides when to trigger compensating actions

Communication may be:

* Synchronous (HTTP, gRPC)
* Asynchronous (messaging, Kafka)

**Advantages**

* Business logic is explicit and centralized
* Easier to reason about complex workflows
* Clear visibility into process state

**Trade-offs**

* Additional component to manage
* Risk of creating a bottleneck if poorly designed


## The Core Mental Shift

The most important takeaway is conceptual:

**Distributed systems do not give you rollback for free.**

You must:

* Accept partial failure as normal
* Design compensating actions explicitly
* Treat business workflows as state machines, not transactions

The Saga pattern formalizes this approach.

For how Sagas handle isolation and prevent cross-saga interference, see [Saga Isolation and Semantic Locks](../Saga Isolation and Semantic Locks).

## When to Use What

* **Single service, simple consistency** → local transactions
* **Reliable event publication** → Outbox pattern
* **Multi-step, cross-service business processes** → Saga pattern

In practice, these patterns are often used together:

* **Outbox** ensures that state changes and events are published reliably
* **Saga** ensures that multi-service workflows remain consistent


## Closing Thoughts

Migrating from monoliths to distributed systems is not just a technical refactor—it is a **transactional paradigm shift**.

The Saga pattern is not a silver bullet, but it provides a practical and scalable way to manage long-running, distributed business processes in a world where traditional transactions no longer apply.

Understanding this shift is essential for designing systems that fail predictably—and recover correctly.

---

Happy coding! 💻
