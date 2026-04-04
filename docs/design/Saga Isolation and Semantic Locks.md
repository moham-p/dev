In [the previous post](../From 2PC to Sagas, Managing Transactions in Distributed Systems), we established that Sagas replace **XA/2PC-style global transactions** with **local transactions + compensation**. But that leaves a critical gap:

> What prevents multiple sagas from interfering with each other?

In traditional systems built on **Two-Phase Commit** (2PC) and **XA specification**, isolation is handled by **database locks**.
In Saga-based systems, there are no such locks across services.

Instead, we rely on **semantic locks**—explicit, business-level state transitions such as `PENDING`, `IN_PROGRESS`, or `RESERVED`.

A **pending state** is the most common example:

* It signals that a business process is ongoing
* It prevents conflicting operations
* It encodes isolation directly into the domain model

This is not a technical mechanism—it is a **modeling decision**.


## Database Locks vs Semantic Locks

Here is the precise comparison:

| Dimension        | Database Locks (2PC/XA)                   | Semantic Locks (Saga)          |
| ---------------- | ----------------------------------------- | ------------------------------ |
| Enforcement      | Infrastructure (DB engine)                | Application / domain logic     |
| Visibility       | Hidden from business logic                | Explicit in domain model       |
| Blocking         | Yes (transactions wait)                   | No (fail fast or branch logic) |
| Scope            | Single database / tightly coupled systems | Cross-service, distributed     |
| Duration         | Short-lived                               | Can be long-running            |
| Failure handling | Automatic rollback                        | Explicit compensation          |
| Scalability      | Limited                                   | High (non-blocking)            |

The key shift:

> You move from **implicit isolation enforced by the database** to **explicit isolation encoded in business state**.


## Interaction with Choreography vs Orchestration

Semantic locks behave differently depending on how the Saga is implemented.

### 1. Choreography (Event-driven Sagas)

* Each service reacts to events independently
* Each service must:

    * Check semantic state (e.g., `PENDING`)
    * Decide whether to proceed or reject

Implication:

* Isolation logic is **distributed**
* Every service must consistently interpret state transitions
* Risk of divergence if rules are not aligned

Typical pattern:

* Service A sets `ORDER_PENDING`
* Service B receives event → validates state before acting
* Service C ignores or compensates if state is incompatible


### 2. Orchestration (Centralized Sagas)

* A central orchestrator controls the workflow
* It enforces ordering and guards transitions

Implication:

* Isolation logic is **centralized**
* Easier to enforce consistent rules
* The orchestrator becomes the authority on valid transitions

Typical pattern:

* Orchestrator sets `PENDING`
* Calls downstream services only when state is valid
* Triggers compensation if a step fails


### Key Difference

| Aspect                | Choreography | Orchestration |
| --------------------- | ------------ | ------------- |
| Isolation enforcement | Distributed  | Centralized   |
| Risk of inconsistency | Higher       | Lower         |
| Flexibility           | Higher       | Lower         |
| Observability         | Harder       | Easier        |


## Real Production Failure Scenarios

This is where semantic locks become critical—not theoretical.

### 1. Double Booking / Resource Conflict

**Scenario**

* Two sagas attempt to reserve the same inventory item

Without semantic lock:

* Both succeed → overselling

With semantic lock:

* First saga sets `RESERVED_PENDING`
* Second saga sees state → fails or retries


### 2. Concurrent Cancel vs Confirm

**Scenario**

* User cancels an order while payment is being processed

Without semantic lock:

* Payment succeeds
* Order is already canceled → inconsistent state

With semantic lock:

* `ORDER_PENDING` prevents cancellation
* Or cancellation triggers compensation (refund)


### 3. Retry Storms and Duplicate Processing

**Scenario**

* A service retries due to timeout
* Same Saga step executes multiple times

Without semantic lock:

* Duplicate side effects (e.g., double charge)

With semantic lock:

* State check (`PAYMENT_COMPLETED`) prevents re-execution


### 4. Cross-Saga Interference

**Scenario**

* One Saga updates an entity while another Saga is mid-flight

Without semantic lock:

* Interleaved updates corrupt business invariants

With semantic lock:

* Entity marked `IN_PROGRESS`
* Competing Saga rejects or queues operation


## Final Takeaway

The evolution now becomes complete:

* **2PC/XA** → atomicity + isolation via infrastructure
* **Outbox** → reliable communication
* **Saga** → distributed consistency via compensation
* **Semantic locks** → isolation via business semantics

> In distributed systems, correctness is no longer enforced by the database—it is encoded in your domain model.

That is the real shift.

---

Happy coding! 💻

