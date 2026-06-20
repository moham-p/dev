Imagine this flow:

```text
Browser -> Backend A -> Backend B
```

At first glance, it may look like Backend A is simply **forwarding** the request to Backend B.

But in Spring applications, the word **forward** has a very specific meaning. And when Backend A uses `FeignClient` to call Backend B, that is not technically a forward.

It is a new HTTP call.

---

## What Is a Real HTTP Forward?

A real HTTP forward happens inside the same web application.

For example, in Spring MVC:

```java
return "forward:/internal-endpoint";
```

The flow is:

```text
Browser -> /start
Server internally forwards to /internal-endpoint
Browser still sees /start
```

No new browser request is created. The same `HttpServletRequest` continues inside the server.

That means request attributes are preserved:

```java
request.setAttribute("source", "from-start");
return "forward:/target";
```

The `/target` endpoint can still read that attribute because it is the same request.

---

## What Happens with FeignClient?

FeignClient works differently.

Example:

```java
@FeignClient(name = "patient-service")
public interface PatientClient {

    @GetMapping("/patients/{id}")
    PatientDto getPatient(@PathVariable Long id);
}
```

When a controller uses this client:

```java
@GetMapping("/patients/{id}")
public PatientDto getPatient(@PathVariable Long id) {
    return patientClient.getPatient(id);
}
```

The flow is:

```text
Browser -> Backend A
Backend A -> new HTTP request -> Backend B
Backend B -> response -> Backend A
Backend A -> response -> Browser
```

So Backend A is not forwarding the same request. It is receiving one request and creating another one.

---

## The Main Difference

```text
Forward     = same request, same application
FeignClient = new request, another service
```

| Aspect                            | HTTP Forward                   | FeignClient             |
| --------------------------------- | ------------------------------ | ----------------------- |
| Request                           | Same request                   | New request             |
| Target                            | Same app/server                | Another API/service     |
| Browser URL changes?              | No                             | No                      |
| Request attributes preserved?     | Yes                            | No                      |
| Headers propagated automatically? | Same request headers available | Must be copied manually |
| Network call?                     | No external call               | Yes                     |

---

## Why This Matters

With a real forward, everything stays inside one application.

With FeignClient, we enter service-to-service communication. That means we must think about:

```text
- Timeouts
- Retries
- Authentication
- Header propagation
- Correlation IDs
- Logging
- Distributed tracing
- Downstream failures
```

For example, if the original request contains:

```text
Authorization: Bearer token
X-Correlation-Id: abc-123
```

Backend B will not automatically receive those headers. We need to explicitly propagate them, usually with a Feign `RequestInterceptor`.

---

## Better Wording

Instead of saying:

```text
Backend A forwards the request to Backend B using FeignClient.
```

A more accurate sentence is:

```text
Backend A acts as a proxy: it receives the client request and creates a new HTTP request to Backend B using FeignClient.
```

---

Happy coding! 💻
