Imagine two multi-threaded programs running on a Mac: one written in Java, the other in Go. Do they use different kinds of threads? The short answer: underneath, no. On top, yes.

## Same processes, same kernel threads

Each program runs as an ordinary macOS process with its own address space. At the kernel level, the threads inside both processes are the same type too: kernel threads, which on macOS come from the pthreads/Mach thread layer. The kernel schedules them the same way and has no idea which language created them.

The real difference is in the user-space layer each runtime builds on top of those threads.

## Java: platform threads and virtual threads

Java has two kinds of threads.

**Platform threads** (the classic kind): each `java.lang.Thread` maps 1:1 to an OS thread. If you create 500 platform threads, the JVM asks the kernel for 500 kernel threads.

**Virtual threads** (since Java 21): lightweight threads managed by the JVM, not the OS. The JVM runs many virtual threads on a small pool of OS threads called carrier threads (by default about one per CPU core).

When a virtual thread blocks, for example while waiting on I/O, the JVM unmounts it from its carrier thread, so the carrier can run another virtual thread. This is an M:N model, very close to how Go works.

## Go: many goroutines on a few OS threads

Goroutines are not OS threads. The Go runtime uses an M:N scheduler: thousands or even millions of goroutines share a small number of OS threads. By default this is roughly one per CPU core, set by `GOMAXPROCS`.

A goroutine starts with a small stack (about 2 KB) that grows when needed. The Go runtime switches between goroutines in user space, without involving the kernel.

## Summary

Both processes use the same kind of OS threads underneath. What differs is the unit of concurrency you write code with:

- In classic Java, that unit is an OS thread.
- In Go, it's a goroutine: a cheap, runtime-managed task running on top of a few OS threads.

This is why Go can easily run a million goroutines, while a million classic Java threads would run out of memory. From the kernel's point of view, the Go process might have about 8 threads, while the Java process could have thousands.

---

Happy coding! 💻