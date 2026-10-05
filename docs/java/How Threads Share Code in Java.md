---
tags:
  - Concurrency
  - Language Internals
---

In [a previous post](../../spring/How Singleton Components Work with Thread Pools) we explored how Spring Boot handles multiple requests using thread pools, 
and how singleton-scoped components (like controllers and services) work in such an environment. But an equally important question remains:
*“If all threads share the same singleton, do they each copy the code into their stack?”*

## Short Answer

**No**, each thread does **not** copy the code of the singleton class into its own stack.

Instead:

* The code is shared
* Each thread gets its own stack frame and local variables when invoking a method


## How Code and Threads Work in Java

### What Is the Stack?

Each thread in Java has its own call stack. It stores:

* Method call frames
* Local variables
* Return addresses

Each time a method is called, a new stack frame is pushed onto the thread’s private stack memory.

---

### Where Does the Code Live?

The compiled method code (bytecode) lives in the JVM's method area, also known as:

* **Metaspace** (in modern JVMs)
* **Code cache**

This area is **shared** by all threads. There's only **one copy of the method**, no matter how many threads invoke it.

---

### What Happens When Multiple Threads Use a Singleton?

Consider this singleton class:

```java
public class MySingleton {
    private static final MySingleton instance = new MySingleton();

    private MySingleton() {}

    public static MySingleton getInstance() {
        return instance;
    }

    public void doSomething() {
        // some logic
    }
}
```

Now multiple threads run:

```java
MySingleton.getInstance().doSomething();
```

Each thread:

* Calls `getInstance()` and receives the **same singleton instance**
* Invokes `doSomething()`, which:

    * Pushes a stack frame onto **that thread’s stack**
    * Allocates **method-local variables**
* Executes the **shared code** independently

The code for `doSomething()` is not copied — it is **shared**, but executed in separate thread contexts.

---

## Why Singleton Components Are Safe in Spring — If Stateless

Spring Boot creates most beans as **singletons** by default:

* One instance per application context
* Shared across all requests and threads

This is safe **only if the bean is stateless**.

---

### ⚠️ Avoid Shared Mutable State

Here’s a dangerous anti-pattern:

```java
@Service
public class DangerousService {
    private List<String> names = new ArrayList<>(); // ❌ shared mutable state!
}
```

This `names` list is shared by all threads. Concurrent access can lead to:

* Race conditions
* `ConcurrentModificationException`
* Corrupted data

---

### ✅ Use Method-Local Variables Instead

Here’s a safer version:

```java
@Service
public class SafeService {
    public void handleRequest(String name) {
        List<String> names = new ArrayList<>(); // ✅ local to this method
        names.add(name);
        // process names...
    }
}
```

Every request gets its own list, allocated in that thread’s stack frame. No interference, no bugs.

---

## Real-World Analogy: Shared Code, Private Data

Imagine the method as a set of instructions written on a whiteboard in a room. Each thread walks into the room, 
reads the instructions, and follows them using its own notebook (stack frame). Everyone reads the same board, but works with their own materials.

---

## Wrap-Up

Threads don’t copy code — they just execute shared code in their own private space. That’s why singleton beans in Spring are safe, as long as you keep them stateless.

If your method uses only local variables and doesn’t touch shared state, you're good.

---

Happy coding! 💻
