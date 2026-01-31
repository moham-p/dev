Traditional persistence models—typically ORM-based and CRUD-oriented—are optimized for storing *current state*. Aggregates are mapped to tables, fields to columns, and instances to rows. While this approach is familiar and effective for basic data storage, it has a structural weakness that becomes increasingly visible in modern systems: **domain events are not a first-class concern**.

When an aggregate changes state—an order is placed, a payment is captured, an address is updated—those changes often need to be communicated to other parts of the system. In event-driven and microservice architectures, this communication happens through **domain events**. Yet in traditional persistence, event publication is usually *bolted on* to the business logic rather than being an inherent part of it.

### The “Bolted-On” Events Problem

In a typical ORM-based design, the flow looks like this:

1. Business logic updates an aggregate
2. The ORM persists the new state
3. A transaction commits
4. An event is published (if the developer remembered to do it)

The problem is that steps 2–4 are not intrinsically linked. ORMs may offer lifecycle callbacks, but they are technical hooks, not business-aware mechanisms. There is no built-in guarantee that event publication is atomic with state changes. As a result, systems can easily end up in inconsistent states:

* The database update succeeds but the event is never published
* The event is published even though the transaction rolls back
* The event contains incomplete or incorrect business data

In a microservice architecture, where downstream services rely on events for synchronization, workflows, and integration, this inconsistency leads to subtle but serious reliability issues.

### Event Sourcing: Persisting Aggregates as Events

Event sourcing addresses this mismatch by reversing the relationship between state and events. Instead of persisting aggregates as rows that represent their latest state, **event sourcing persists each aggregate as a sequence of domain events** in an event store.

For example, rather than storing an `Order` as a single row in an `ORDER` table, event sourcing stores a stream of events such as:

* `OrderCreated`
* `OrderApproved`
* `OrderShipped`

The aggregate’s current state is derived by replaying these events. In this model, events are not a side effect of state changes—they *are* the state changes.

### Reliable Event Publication and Built-In Auditing

A major benefit of event sourcing is that **domain events are reliably published whenever an aggregate’s state changes**. Event persistence and event publication are part of the same operation, eliminating the risk that business logic and integration logic drift apart.

This reliability makes event sourcing a strong foundation for event-driven microservice architectures. Because events can also record metadata—such as the identity of the user who made the change—the event stream doubles as an **accurate, guaranteed audit log**. The same stream can then be reused for notifications, inter-service integration, analytics, and monitoring without additional instrumentation.

### Snapshots and Aggregate Reconstruction

Replaying an entire event stream every time an aggregate is loaded can be inefficient for long-lived aggregates. To address this, event-sourced systems commonly use **snapshots**.

A snapshot captures the aggregate’s state at a specific point in time. When loading the aggregate, the system recreates the aggregate instance from the snapshot rather than constructing it using its default constructor and replaying every historical event. Only the events that occurred after the snapshot are applied, preserving correctness while significantly improving performance.

### The Challenge of Evolving Events

Event sourcing introduces its own complexities, particularly around **schema evolution**. Events and snapshots are stored forever, and their schemas will inevitably change over time. Aggregates may need to handle events from multiple schema versions, which can lead to bloated and fragile code.

A well-established solution is to **upgrade events to the latest schema version when they are loaded from the event store**. This keeps version-handling logic outside the aggregate, allowing aggregates to apply only the current version of events and remain focused on business behavior rather than compatibility concerns.

### Closing Thoughts

Traditional persistence treats event publication, auditing, and integration as secondary concerns—features that must be manually bolted on and carefully maintained. Event sourcing makes a different trade-off: it elevates events to the source of truth. In doing so, it aligns business state with system integration, provides reliable event publication, enables accurate auditing, and offers a scalable foundation for event-driven architectures—provided its additional complexity is handled with discipline.
