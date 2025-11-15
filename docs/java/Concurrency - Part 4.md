As applications demand higher scalability, non-blocking I/O, and minimal thread usage, these traditional models start to show limitations. This is where **reactive programming** enters the scene—not as a replacement, but as a fundamentally different approach to concurrency.

In this post, we’ll dive into reactive concurrency in Java with **Project Reactor** and explore how it transforms the way we handle asynchronous, concurrent tasks—especially in high-throughput systems.

---

## Reactive vs. Imperative Concurrency

| Feature           | Imperative Model (Threads, Executors)              | Reactive Model (Project Reactor)             |
| ----------------- | -------------------------------------------------- | -------------------------------------------- |
| Execution control | You manage when and how threads run                | You describe *what* should happen, not *how* |
| Blocking          | Often blocking (e.g., thread waits)                | Non-blocking, event-driven                   |
| Thread usage      | One thread per task (virtual threads improve this) | Small thread pool, I/O multiplexing          |
| Backpressure      | Manual, if at all                                  | Built-in flow control                        |

Reactive programming focuses on **declarative concurrency**—you define a pipeline of operations, and the system executes them asynchronously and concurrently *behind the scenes*.

## Project Reactor: A Reactive Concurrency Engine

**Project Reactor** brings [Reactive Streams](https://github.com/reactive-streams/reactive-streams-jvm/blob/v1.0.4/README.md) to Java. Instead of managing threads, queues, and futures, you work with two key abstractions:

* `Mono<T>`: A publisher of **zero or one** item.
* `Flux<T>`: A publisher of **zero to many** items (a reactive stream).

These types form the backbone of reactive pipelines, offering powerful operators and built-in support for concurrency and error handling.

## Threading in Reactor with Schedulers

In reactive programming, execution doesn't happen immediately in the calling thread. Instead, Reactor uses **Schedulers** to control where and how tasks run, allowing better separation of concerns and optimal use of system resources.

| Scheduler                     | Common Use Case                        |
| ----------------------------- | -------------------------------------- |
| `Schedulers.parallel()`       | CPU-bound tasks (e.g., data crunching) |
| `Schedulers.boundedElastic()` | Blocking I/O (e.g., file or DB access) |
| `Schedulers.single()`         | Work that must run on a single thread  |
| `Schedulers.immediate()`      | Execute directly on the current thread |

### Example: Parallel Processing without Threads

```java
Flux.range(1, 10)
    .flatMap(i -> Mono.just(i)
          .subscribeOn(Schedulers.parallel())
          .map(this::process))
    .collectList()
    .block();
```

Here, `flatMap` handles parallelism, and `subscribeOn` ensures each task runs concurrently—without you ever creating a thread.

## Built-in Backpressure = Controlled Concurrency

Reactive streams (and Project Reactor) support **backpressure**, a critical feature absent in models like `CompletableFuture`. Backpressure allows the **consumer to control the rate of data flow**, preventing memory overload or CPU exhaustion.

This makes reactive pipelines **self-regulating**, especially in systems that process data continuously or unevenly (e.g., stream ingestion, APIs, or message queues).

## Integrating Reactive Concurrency with Spring

To build fully non-blocking applications in the Spring ecosystem, you can combine **Project Reactor** with:

* **Spring WebFlux**: A reactive web framework.
* **Spring Data R2DBC**: A reactive relational database integration.

### Reactive Repository Interface:

```java
public interface PersonRepository extends ReactiveCrudRepository<Person, Long> {
    Flux<Person> findAll();
    Mono<Person> findById(Long id);
}
```

Unlike blocking repositories, this interface returns `Flux` and `Mono`, fully integrated with Reactor and WebFlux for end-to-end reactive pipelines.

## Project Reactor vs. Node.js: A Conceptual Parallel

Reactive programming isn’t unique to Java. Node.js has long used **event loops and non-blocking I/O**. Reactor brings similar principles to the JVM with stronger type safety and backpressure:

| Concept        | Project Reactor       | Node.js                         |
| -------------- | --------------------- | ------------------------------- |
| Async Type     | `Mono`, `Flux`        | Promises, Streams               |
| Execution      | Scheduler-based       | Event loop                      |
| Backpressure   | ✅ Built-in            | ⚠️ Rudimentary                  |
| Stream control | Functional operators  | Streams with `pause()/resume()` |
| Integration    | Spring WebFlux, R2DBC | Express, native modules         |

## When to Go Reactive

✅ Choose reactive concurrency when your application must:

- Handle a high volume of concurrent I/O (e.g., HTTP APIs, messaging)

- Support streaming or real-time data pipelines

- Scale with fewer threads and predictable resource usage

❌ Avoid it when:

- You need synchronous, CPU-heavy workflows

- Your team isn’t familiar with functional or stream-based programming

- You’re working with libraries that aren’t reactive-compatible

## How does the Project Reactor relate to CompletableFuture?

CompletableFuture:

- Part of Java’s standard library (introduced in Java 8).

- Represents a single future result of an asynchronous computation.

- Provides a straightforward API to handle asynchronous tasks and their results.

- Does not implement the [Reactive Streams specification](https://github.com/reactive-streams/reactive-streams-jvm/blob/v1.0.4/README.md#specification). It is more focused on individual, one-time asynchronous tasks
  rather than streams of data.

- Supports chaining of asynchronous operations through methods like thenApply, thenAccept, thenCompose, etc.

Project Reactor:

- A library that implements the [Reactive Streams specification](https://github.com/reactive-streams/reactive-streams-jvm/blob/v1.0.4/README.md#specification) and provides a powerful API for reactive programming.

- Offers two main types: Mono (for zero or one result) and Flux (for zero or more results).

- Designed for building non-blocking, event-driven applications.

- Supports backpressure, a critical feature for handling the flow of data in a robust and efficient manner.

## Final Thoughts

With Project Reactor, you gain a powerful model for asynchronous concurrency that eliminates many pitfalls of traditional thread-based programming.

It’s not about replacing threads—it’s about **thinking differently and shifting paradigms**. Let the system manage the “how,” while you focus on the “what.”

---

Happy coding! 💻
