Modern Java applications often need to perform multiple tasks asynchronously without blocking threads or slowing down the system. One of the essential tools introduced in Java 8 to address this need is the **`CompletableFuture`**.

This blog post focuses on how `CompletableFuture` makes asynchronous programming easier, compared with JavaScript’s Promise concept, and demonstrates its real-world use in a Java application.

## How CompletableFuture Compares to JavaScript's Promise

If you're familiar with JavaScript Promises, understanding `CompletableFuture` becomes much easier:

| Aspect                    | JavaScript Promise                                   | Java CompletableFuture                     |
|----------------------------|------------------------------------------------------|--------------------------------------------|
| Creating Async Operation   | `new Promise((resolve, reject) => {...})`            | `CompletableFuture.supplyAsync(() -> {...})` |
| Handling Result            | `.then(value => {...})`                              | `.thenAccept(result -> {...})`              |
| Handling Errors            | `.catch(error => {...})`                             | `.exceptionally(error -> {...})`            |
| Execution Context          | Event loop, non-blocking                             | Separate thread pool (e.g., ForkJoinPool or ExecutorService) |
| Blocking Behavior          | Non-blocking by design                               | Supports both non-blocking and blocking (`get()`) |

✅ `CompletableFuture` brings Promise-style async composition into Java, with even more control over threading.

---

## Basic Examples

Let's look at two simple cases: `runAsync` and `supplyAsync`. Use `runAsync` when you don't need a result, and `supplyAsync` when you do.

### Example 1: `runAsync` — Task without a return value

```java
import java.util.concurrent.CompletableFuture;

public class RunAsyncExample {
    public static void main(String[] args) {
        System.out.println("Main thread: " + Thread.currentThread().getName());

        CompletableFuture<Void> future = CompletableFuture.runAsync(() -> 
                System.out.println("Running async task on: " + Thread.currentThread().getName()));

        future.join();
    }
}
```
Output might look like:
```
Main thread: main
Running async task on: ForkJoinPool.commonPool-worker-1
```

### Example 2: `supplyAsync` — Task with a return value

```java
import java.util.concurrent.CompletableFuture;

public class SupplyAsyncExample {
    public static void main(String[] args) {
        CompletableFuture<String> future = CompletableFuture.supplyAsync(() ->
                "Hello from " + Thread.currentThread().getName());

        String result = future.join();
        System.out.println("Result: " + result);
    }
}
```
Output might look like:
```
Result: Hello from ForkJoinPool.commonPool-worker-1
```

## Real-World Usage: Asynchronous File Import

Let's see a real-world example: handling **asynchronous file imports** without blocking HTTP request threads.

> **Note:** The async behavior here is achieved using `@Async`, which runs on **Spring's default async executor** (like `SimpleAsyncTaskExecutor`) — **not** on `ForkJoinPool.commonPool()`.
The method returns `CompletableFuture<String>`, meaning it **behaves similarly to `supplyAsync`**, where a result (`jobId`) is asynchronously produced and returned.

### Service Layer: ImportService

```java
@Async
public CompletableFuture<String> importFile(String jobId, String filePath) {
    try {
        HSSFWorkbook workbook = new HSSFWorkbook(new FileInputStream(filePath));
        HSSFSheet sheet = workbook.getSheetAt(0);

        if (sheet.getPhysicalNumberOfRows() > 1) {
            processSheetRows(sheet, jobId);
        }

        return CompletableFuture.completedFuture(jobId);

    } catch (Exception e) {
        throw new JobException(jobId, e.getMessage());
    }
}
```

- `@Async` ensures the method runs in a background thread.
- Returns `CompletableFuture<String>`, behaving like a `supplyAsync`.
- Uses Spring Boot’s default async executor, **not ForkJoinPool**.

### Controller Layer: FileImportController

```java
@PostMapping
public ResponseEntity<JobIdResponse> importFile(@RequestParam("file") MultipartFile file) throws IOException {
    File uploadedFile = fileStorageService.saveFile(file);
    String filePath = uploadedFile.getAbsolutePath();
    Job newJob = jobService.saveNewFileJob(JobType.IMPORT);

    importService.importFile(newJob.getId(), filePath)
                 .whenComplete(fileImportServiceCallback);

    return ResponseEntity.status(HttpStatus.CREATED).body(JobMapper.toJobId(newJob));
}
```

- Saves the uploaded file.
- Triggers `importFile` asynchronously.
- Registers a `whenComplete` callback to react to completion.
- Returns HTTP 201 (Created) immediately, without waiting for import to finish.

### Callback Handling: FileServiceCallback

```java
@Component
@RequiredArgsConstructor
public class FileServiceCallback implements BiConsumer<String, Throwable> {

    private final JobService jobService;

    @Override
    public void accept(String jobId, Throwable ex) {
        if (ex == null) {
            jobService.updateJobState(jobId, JobState.DONE);
        } else {
            jobService.updateJobState(jobId, JobState.ERROR);
        }
    }
}
```

- If the import is successful → mark job as `DONE`.
- If it fails → mark job as `ERROR`.

---

## Key Takeaways

- `CompletableFuture` enables easy asynchronous programming.
- `runAsync` is for tasks without a return; `supplyAsync` is for tasks with a return.
- Returning `CompletableFuture<T>` manually (like with `completedFuture`) is a common pattern when you already have a value.
- Using `@Async` with `CompletableFuture` provides clean, scalable non-blocking designs.
- In Spring, `@Async` uses its own executor, not the ForkJoinPool by default.

---

Happy coding! 💻
