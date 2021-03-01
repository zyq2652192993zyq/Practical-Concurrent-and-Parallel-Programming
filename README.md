> # Practical-Concurrent-and-Parallel-Programming

## Why learn this course?

In this course we can learn how to write correct and efficient concurrent and parallel software, primarily using **Java**, on standard shared-memory multicore hardware. The course covers basic mechanisms such as threads, locks and shared memory as well as more advanced mechanisms such as parallel streams for bulk data, transactional memory, message passing, and lock-free data structures with compare-and-swap. It covers concepts such as atomicity, safety, liveness and deadlock. It covers how to measure and understand performance and scalability of parallel programs. It covers tools and methods to find bugs in concurrent programs.

## Course Information

* Website: http://www.itu.dk/people/sestoft/itu/PCPP/E2015/
* Textbook: Java Concurrency in Practice, Addison-Wesley 2006
* Programming Language: Java

## Exercises

| Subject                                                      | Notes             | Code            |
| ------------------------------------------------------------ | ----------------- | --------------- |
| Concurrent and parallel programming, why, what is so hard. Threads and locks in Java, shared mutable memory, mutual exclusion, visibility, volatile fields, atomic operations, avoiding sharing (thread confinement, stack confinement), immutability, final, safe publication. | Exercise 1 note   | Exercise 1 code |
| Threads and Locks: Designing thread-safe classes. Monitor pattern. Concurrent collections. Documenting thread-safety. | Exercise 2 note   | Exercise 2 code |
| Java 8 parallel streams for bulk data and parallel array prefix operations. Functional interfaces, lambda expressions, method reference expressions. | Exercise 3 note   | Exercise 1 code |
| Performance measurements.                                    | Exercise 4 note   | Exercise 1 code |
| Threads and Locks: Tasks and the Java executor framework. Concurrent pipelines, `wait()` and `notifyAll()`. | Exercise 5 note   | Exercise 1 code |
| Threads and Locks: Performance and scalability; concurrent hash map case study. | Exercise 6 note   | Exercise 1 code |
| Threads and Locks: GUI applications. Cache coherence and performance consequences. | Exercise 7 note   | Exercise 1 code |
| Threads and Locks: Testing concurrent programs.              | Exercise 8 note   | Exercise 1 code |
| Threads and Locks: Safety and liveness, absence of deadlock and livelock; the Java memory model. | Exercise 9 note   | Exercise 1 code |
| Transactional memory with Multiverse                         | Exercise 10 note  | Exercise 1 code |
| Optimistic concurrency, lock-free data structures, Treiber stack, compare-and-swap. | Exercise 11 note  | Exercise 1 code |
| The Michael and Scott queue, progress concepts, work-stealing queues. | Exercise 12 note | Exercise 1 code |

