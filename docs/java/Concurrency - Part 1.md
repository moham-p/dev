When building concurrent applications in Java, managing threads properly is crucial. Spawning raw threads (`new Thread(...)`) works for simple cases, but it's inefficient and hard to scale. Java’s concurrency package offers a more powerful approach: **thread pools**.

This post will walk you through the fundamentals of using `ExecutorService` and customizing threads with `ThreadFactory`. Then we’ll explore a special thread pool used behind the scenes—`ForkJoinPool.commonPool()`.

---

## Why Use Thread Pools?

Thread pools reuse threads instead of creating new ones for each task. This means:

- Better performance (less time creating/destroying threads)
- Controlled concurrency (limits how many threads run at once)
- Cleaner code (no manual thread lifecycle management)

---

## Using `ExecutorService`

`ExecutorService` is the most common way to use thread pools in Java:

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class PoolExample {
    public static void main(String[] args) {
        ExecutorService executor = Executors.newFixedThreadPool(2);

        executor.submit(() -> {
            System.out.println("Task 1 executed by " + Thread.currentThread().getName());
        });

        executor.submit(() -> {
            System.out.println("Task 2 executed by " + Thread.currentThread().getName());
        });

        executor.shutdown();
    }
}
```

☑️ You can choose from:

- `newFixedThreadPool(int n)`
- `newCachedThreadPool()`
- `newSingleThreadExecutor()`

Each comes with trade-offs depending on your workload (CPU-bound, I/O-bound, etc.).

---

## Customizing Threads with `ThreadFactory`

Need more control over thread naming or daemon settings? Use a `ThreadFactory`.

```java
import java.util.concurrent.ThreadFactory;

ThreadFactory factory = r -> {
    Thread t = new Thread(r);
    t.setName("worker-" + t.getId());
    return t;
};
```

You can pass this factory into your executor:

```java
ExecutorService executor = Executors.newFixedThreadPool(2, factory);
```

This is super helpful for debugging and log tracing.

## Concurrency vs. Parallelism  
Before we go further, it's important to understand the difference between **concurrency** and **parallelism**. 

- Parallelism means using multiple CPU cores to execute tasks at the same time. 
- Concurrency, on the other hand, is about handling multiple tasks at once—usually on a single core—by switching between 
them efficiently. It gives the impression of things happening simultaneously, even if they’re not.

Although you can achieve a degree of parallelism using an `ExecutorService` (for example, by creating a fixed thread pool 
with `Executors.newFixedThreadPool()` on a multi-core machine), its primary purpose is to manage task submission and 
scheduling. It was designed for general-purpose task execution (particularly well-suited for I/O-bound workloads). 

If you're aiming for fine-grained, CPU-bound parallelism (along with advanced features like task splitting and work-stealing), then `ForkJoinPool` is exactly what you're looking for.

## What is `ForkJoinPool.commonPool()`?

This is a special shared thread pool that is used by:

- `parallelStream()`
- `CompletableFuture.runAsync()` and `supplyAsync()` (when no custom executor is provided)
- Fork/Join Framework (e.g., `RecursiveTask`, `RecursiveAction`)

Although `ForkJoinPool.commonPool()` is primarily designed for parallelism, it also incorporates concurrency.

- **Parallelism:** `ForkJoinPool.commonPool()` utilizes multiple worker threads—typically backed by available CPU cores—to execute tasks in parallel. The default parallelism level is usually `Runtime.getRuntime().availableProcessors() - 1`, meaning it takes advantage of multiple cores to run tasks simultaneously.

- **Concurrency:** Beyond parallel execution, the pool handles task scheduling, load balancing through work-stealing, and efficient queuing. This dynamic task management is a hallmark of concurrency—coordinating multiple tasks over time, even if they're not all executing at once.

By default, the size of the common `ForkJoinPool` is set to `Runtime.getRuntime().availableProcessors() - 1`. This configuration reserves one core for the main thread and allows the remaining worker threads to fully utilize the system’s CPU capacity. Let’s see how this works on an 8-core machine in the example below:

```java
import java.util.Arrays;
import java.util.List;

public class ParallelStreamExample {
    public static void main(String[] args) {

        // Prints: 8
        System.out.println("Available processors (cores): " + Runtime.getRuntime().availableProcessors());

        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

        // Use parallelStream to process the list in parallel
        int sum = numbers.parallelStream()
                // This method is used to perform an action on each element of the stream as it is processed. 
                // It does not affect the result of the stream, but it's useful for debugging or logging.
                .peek(num -> System.out.println("Processing " + num + " on thread: " + Thread.currentThread().getName()))
                .mapToInt(Integer::intValue)
                .sum();

        System.out.println("Sum: " + sum);
    }
}
```

- It prints something like:

```
Processing 5 on thread: ForkJoinPool.commonPool-worker-5
Processing 7 on thread: main
Processing 2 on thread: ForkJoinPool.commonPool-worker-3
Processing 3 on thread: ForkJoinPool.commonPool-worker-1
Processing 9 on thread: ForkJoinPool.commonPool-worker-2
Processing 4 on thread: ForkJoinPool.commonPool-worker-6
Processing 6 on thread: ForkJoinPool.commonPool-worker-4
Processing 1 on thread: ForkJoinPool.commonPool-worker-7
Processing 8 on thread: ForkJoinPool.commonPool-worker-5
Processing 10 on thread: main
Sum: 55
```

## When to Avoid It

The `commonPool` works great for **non-blocking CPU-bound tasks**, but you **should not use it for blocking I/O or long-running tasks**, because:

- It has a limited number of threads
- If all are blocked, other tasks will starve

**Solution**: Use your own `ExecutorService` for blocking tasks:

```java
CompletableFuture.supplyAsync(() -> slowOperation(), customExecutor);
```

---

Happy coding! 💻
