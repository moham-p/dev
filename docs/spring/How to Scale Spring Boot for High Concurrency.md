In [my previous post](../How Singleton Components Work with Thread Pools), we explored how Spring Boot uses singleton-scoped components (like controllers, services, and repositories) and how multiple threads can safely share these instances thanks to stateless design. We verified singleton behavior using `System.identityHashCode` and explained how Spring delegates concurrent request processing to the servlet container’s thread pool (e.g., Tomcat).

In this follow-up post, we go one level deeper — tackling questions like:

* What happens when concurrent user traffic increases?
* Should you increase the Tomcat thread pool size?
* How do you determine the right number of threads?
* How do you monitor and tune your Spring Boot app for concurrency?


## Spring Boot Uses Thread Pools for Concurrency

Every incoming HTTP request is assigned to a thread from a shared thread pool maintained by the embedded servlet container (e.g., Tomcat, Jetty, or Undertow). 
This makes your app capable of handling multiple users simultaneously, even though your beans are singleton-scoped.

But here’s the catch: the thread pool has limits.

If all threads are busy:

* Incoming requests are queued
* If the queue is full, then requests are rejected or delayed


## When Should You Increase the Thread Pool Size?

If your application starts experiencing, slow response times, request timeouts, or 5xx errors under load it might be time to look at 
your **Tomcat thread pool settings**.

You can configure them in `application.yml`:

```yaml
server:
  tomcat:
    threads:
      max: 200     # default is 200
      min-spare: 10
```


But how do you know what number is right?


## Rule of Thumb: I/O-Bound vs. CPU-Bound

Your thread pool sizing depends on the **nature of your workload**:

### For I/O-Bound Applications

*(e.g., database queries, external HTTP calls)*

* Threads often sit idle waiting for I/O
* You can afford to have **more threads than CPU cores**
* Start with: **2x to 4x the number of cores**

### For CPU-Bound Applications

*(e.g., heavy computation, encoding)*

* Threads are CPU-hungry
* More threads → more context switching → slower performance
* Match threads to number of CPU cores: **maxThreads ≈ CPU cores**

### Example

If your server has 8 cores:

* I/O-heavy service: try `maxThreads = 32–64`
* CPU-heavy service: try `maxThreads = 8`

Always monitor and adjust.

---

## Monitoring Thread Pool Usage

To make informed decisions, observe the actual load on your thread pool. You can do this using:

### Spring Boot Actuator + Micrometer

Enable actuator endpoints and integrate with:

* Prometheus
* Grafana
* CloudWatch, New Relic, etc.

Look for:

* `tomcat.threads.active`
* `tomcat.threads.max`
* `tomcat.threads.busy`

### JVisualVM or JFR

For local testing, tools like **VisualVM** or **Java Flight Recorder** help track:

* Thread activity
* Garbage collection
* Stack traces and bottlenecks

---

Happy coding! 💻
