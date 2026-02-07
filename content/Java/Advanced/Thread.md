# 1. Definition

[Definition from Wikipedia](<https://en.wikipedia.org/wiki/Thread_(computing)>): a thread of execution is the smallest sequence of programmed instructions that can be managed independently by a scheduler, which is typically a part of the operating system. In many cases, a thread is a component of a process.

> A thread is an execution state, as in: a combination of CPU registers, stack, the lot. The OS creates a "process" by reserving some resources to it, and starting a "main" thread. That thread then can spawn more threads.

> NOTE: The term `Thread` in hardware does mean how many threads the CPU can process at the same time. For example: an intel i5 1240p has 16 threads (4 performance-cores which use `Hyper-Threading` can handle 2 threads at once and 8 efficient-cores can handle 1 thread) mean at the very same time (in a few nanoseconds) only 16 threads can be processed by CPU. We can think of this as having exactly 16 "chairs" and 16 workers can use those chairs to do their work simultaneously.

The context switching flow:

1. **The Trigger (The Whistle Blows)**

- A thread is running happily on the CPU. Suddenly, one of two things happens:
- Involuntary (Preemption): The hardware timer fires (e.g., "Your 10ms timeslice is up!").
- Voluntary (Blocking): The thread asks for something slow (e.g., "Read this file from disk"). The thread cannot proceed until the data arrives.

2. **Mode Switch (User to Kernel)**

- The CPU cannot trust the user's code to switch itself. It must hand control to the "Dictator" (The OS Kernel).
- The CPU switches from User Mode (low privilege) to Kernel Mode (high privilege).
- Control jumps to a specific address in memory called the Interrupt Handler.

3. **Save the State (The "Snapshot")**

- The OS needs to save the "Context" of the current thread (Thread A) so it can resume later exactly where it left off. It saves the specific CPU Registers into a data structure in RAM called the TCB (Thread Control Block).
- Program Counter (PC): "Which line of code was I executing?"
- Stack Pointer (SP): "Where is my stack memory?"
- General Registers: The temporary variables currently held in the CPU (e.g., AX, BX, R1, R2).

4. **The Scheduler Decisions**

- Now that Thread A is safely saved (parked), the OS Scheduler code runs.
- It looks at the queue of waiting threads.
- It picks the "winner" (Thread B) based on priority or fairness algorithms.

5. **Restore the State (The "Load Game")**

- The OS grabs the TCB (Thread Control Block) for Thread B.
- It takes the values stored in RAM for Thread B.
- It effectively "pastes" them back into the physical CPU Registers.
- It points the Program Counter to the last instruction Thread B executed before it was paused.

6. **Mode Switch (Kernel to User)**

- The OS steps away.
- The CPU switches back from Kernel Mode to User Mode.
- The CPU jumps to the address in the Program Counter.

7. **Execution Resumes**

- Thread B starts running. To Thread B, it feels like no time has passed at all. It doesn't know it was asleep.

# 2. Thread Lifecycle (The States)

Understanding thread states is crucial for debugging. When you look at a thread dump (e.g., using VisualVM or `jstack`), you will see these states.

In Java, a thread can be in one of 6 states (defined in Thread.State enum):

1. **NEW**: The thread is created but not yet started.

- Code: `Thread t = new Thread();`

2. **RUNNABLE**: The thread is executing in the JVM.

- *Note*: It might be actually running on the CPU, or waiting for the OS scheduler to give it CPU time. Java doesn't distinguish between "Running" and "Ready".

3. **BLOCKED**: The thread is blocked waiting for a monitor lock.

- *Scenario*: Trying to enter a synchronized block that is already held by another thread.

4. **WAITING**: The thread is waiting indefinitely for another thread to perform a particular action.

- *Scenario*: Calling `Object.wait()`, `Thread.join()`, or `LockSupport.park()`.

5. **TIMED_WAITING**: The thread is waiting for another thread for a specified waiting time.

- *Scenario*: Calling `Thread.sleep(1000)`, `Object.wait(1000)`, or `Thread.join(1000)`.

6. **TERMINATED**: The thread has completed execution.

**Key Takeaway for Debugging:**

If you see many threads in **BLOCKED** state, you likely have a synchronization bottleneck or a Deadlock.

If you see threads in **WAITING**, they are idle, waiting for a signal.

# 3. Daemon vs. User Threads 

Java distinguishes between two types of threads based on how they affect the JVM shutdown.

## 3.1 User Thread (Non-Daemon)
- **Default:** Any thread you create is a User Thread by default.
- **Rule:** The JVM will not exit as long as there is at least one User Thread running. Even if `main()` finishes, the JVM stays alive if another User Thread is working.
- **Use case:** Critical tasks that must finish (e.g., processing a payment, writing a file).

## 3.2 Daemon Thread
- **Definition:** A service thread that runs in the background to support User Threads.
- **Rule:** The JVM exits immediately when all User Threads have finished, killing all Daemon Threads instantly (even if they are in the middle of code blocks like `finally`).
- **Use case:** Garbage Collector (GC), removing expired cache entries, sending heartbeats.

```java
public class DaemonDemo {
    public static void main(String[] args) {
        Thread daemonThread = new Thread(() -> {
            while (true) {
                System.out.println("Daemon is working...");
                try { Thread.sleep(1000); } catch (InterruptedException e) {}
            }
        });

        // Mark as daemon BEFORE starting
        daemonThread.setDaemon(true); 
        daemonThread.start();

        System.out.println("Main thread is finishing...");
        // JVM will exit here. The Daemon thread will be killed immediately,
        // likely printing "Daemon is working..." only once or zero times.
    }
}
```

# 4. Implementation

## 4.1 Platform Thread (The Heavyweight Wrapper)

In Java versions prior to 21, what we simply called a `Thread` is actually a **Platform Thread**.

**Implementation:** It is a thin wrapper around an OS Thread (Kernel Thread). This implies a **1:1 mapping**.

**The Cost:** Because it maps directly to the OS, every time a Platform Thread blocks (e.g., waiting for a database response), the OS must perform the entire **Context Switch Flow** described above (Steps 1 through 7).

**Consequence:** This is expensive (consumes ~2MB memory per thread) and slow (context switching takes microseconds). You cannot create millions of them; usually, a few thousand is the limit before `OutOfMemoryError`.

```java
public class PlatformThreadDemo {
    public static void main(String[] args) {
        // Using CachedThreadPool (creates new OS threads as needed)
        // WARNING: This might crash the JVM due to OutOfMemoryError on many machines!
        try (var executor = Executors.newCachedThreadPool()) {
            for (int i = 0; i < 100_000; i++) {
                executor.submit(() -> {
                    try {
                        // Simulate a blocking I/O operation (e.g., fetching data from DB)
                        // In Platform threads, this BLOCKS the actual OS thread.
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        // handle exception
                    }
                });
            }
        }
    }
}
```

## 4.2 Virtual Thread (The Game Changer - Java 21+)

Virtual Threads introduce an **M:N mapping** (Many Virtual Threads mapped to Few OS Threads).

**How it works:** The JVM acts as a scheduler. When a Virtual Thread hits a blocking operation (like `Thread.sleep` or I/O), it does **NOT** trigger the heavy OS Context Switch.

**The "Cheat":** Instead of Step 2 (Mode Switch to Kernel), the JVM simply "unmounts" the Virtual Thread from the carrier OS thread and puts it aside in the heap (RAM). The OS thread is then free to execute another Virtual Thread immediately.

**Benefit:**
- No expensive kernel mode switch.
- Extremely lightweight (variable size, starting at few hundred bytes).
- You can create **millions** of virtual threads without crashing the system.

```java
public class VirtualThreadDemo {
    public static void main(String[] args) {
        // Java 21+: newVirtualThreadPerTaskExecutor
        // This creates a lightweight virtual thread for every single task.
        try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
            for (int i = 0; i < 100_000; i++) {
                executor.submit(() -> {
                    try {
                        // When this sleeps, the underlying OS thread is RELEASED
                        // to handle other tasks. No blocking!
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        // handle exception
                    }
                });
            }
        }
    }
}
```

> **Analogy:**
>
> - **Platform Thread:** Hiring a taxi for every single passenger. Even if the passenger stops to buy coffee, the taxi waits and the meter runs.
> - **Virtual Thread:** A bus system. When a passenger gets off to buy coffee, the bus picks up someone else. The bus (OS Thread) never stops working.

## 4.3 Thread Pool (The Manger)

Since creating a Platform Thread is expensive (involving OS interaction and memory allocation), we cannot just create and destroy them frequently. That would kill the performance.

To solve this, Java uses the Thread Pool pattern.

Analogy: Imagine a restaurant.

- Without Pool: Every time a customer arrives, you hire a new waiter. When they finish eating, you fire the waiter. This is slow and expensive (hiring costs).
- With Pool: You hire 10 waiters permanently. They take turns serving customers. If all 10 are busy, new customers must wait in a queue.

### 4.3.1 Pooling Platform Threads (The Classic Approach)

For traditional threads, we use `ExecutorService` to manage a pool of reusable threads.

Common Types:

- **FixedThreadPool**: A fixed number of threads (e.g., 10). Good for predictable loads.
- **CachedThreadPool**: Creates new threads as needed but reuses old ones when available. Good for short-lived asynchronous tasks.

**Pros**:

- **Resource Control**: Prevents the system from crashing by limiting the number of active threads (e.g., preventing CPU saturation).
- **Performance**: Reusing threads saves the initialization cost.

**Cons**:

- **Complexity**: Tuning the pool size is hard. Too small = high latency (queue builds up). Too big = resource exhaustion.
- **Deadlock Risk**: If threads in the pool are waiting for other threads in the same pool, the system hangs.

```java
try (var executor = Executors.newFixedThreadPool(10)) {
    for (int i = 0; i < 100; i++) {
        executor.submit(() -> {
            // This task runs on one of the 10 reusable threads
            System.out.println("Running on: " + Thread.currentThread());
        });
    }
}
```

### 4.3.2 Pooling Virtual Threads (The Anti-Pattern)

With Java 21+, the rules have changed completely.

**DO NOT** Pool Virtual Threads.

Since Virtual Threads are designed to be disposable and extremely cheap to create, pooling them actually adds unnecessary overhead without any benefit.

- **The Modern Approach**: Use Executors.newVirtualThreadPerTaskExecutor().
- **Behavior**: It creates a new Virtual Thread for every single task. It never reuses them.

```java
// Example: Virtual Thread Executor (Note: It is NOT a pool in the traditional sense)
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    // If you submit 10,000 tasks, it creates 10,000 virtual threads instantly.
    for (int i = 0; i < 10_000; i++) {
        executor.submit(() -> {
            System.out.println("Running on: " + Thread.currentThread());
        });
    }
}
```

### 4.3.3 Summary: To Pool or Not to Pool?
| Feature | Platform Thread | Virtual Thread |
| --------------- | --------------- | --------------- |
| Stategy | **MUST Pool** | **NEVER Pool** |
| Why | Creation is slow & expensive | Creation is fast & cheap |
| Resource Limit | Limit the _Threads_ (e.g., max 200 threads) | Limit the _Resource_ (e.g., max 50 DB connections), not the thread |
| Executor | `newFixedThreadPool`, `newCachedThreadPool` | `newVirtualThreadPerTaskExecutor` |

>NOTE: In **Platform Thread** we limit the number of threads (e.g., 100 threads) to protect the database - the platform threads work as a regulating valve. In **Virtual Thread** we can let the threads spawn freely (but the database can only handle up to 100 connections) so we use `Semaphore` or database's `Connection Pool` to limit.
