---
title: Synchronization & Lock
---

# 1. The Problem: Race Condition

Before talking about locks, we must understand Race Condition.

**Definition**: A race condition occurs when two or more threads access shared data and try to change it at the same time. Because the thread scheduling algorithm can swap between threads at any time, you don't know the order in which the threads will attempt to access the shared data.

**Example:** Imagine a bank account with \$100.

1. Thread A checks balance: \$100.

2. Thread B checks balance: \$100.

3. Thread A withdraws \$10: New balance \$90.

4. Thread B withdraws \$10: It still thinks balance is \$100, so it calculates \$100 - \$10 = \$90. Result: You withdrew \$20 total, but the balance is \$90 instead of \$80. The bank lost money!

# 2. The `synchronized` Keyword (Intrinsic Lock)

This is the built-in locking mechanism in Java. Every object in Java has an internal entity called a Monitor (or Monitor Lock).

When a thread enters a `synchronized` block/method, it acquires the monitor. Other threads trying to enter must wait until the monitor is released.

## 2.1 Synchronized Method

This is the simplest way. It locks the entire method.

```java
public class Counter {
    private int count = 0;

    // Only one thread can execute this method at a time on the same instance
    public synchronized void increment() {
        count++;
    }
    // Equivalent to: synchronized(this) { count++ }
}
```

## 2.2 Synchronized Block (Fine-grained locking)

Instead of locking the whole method (*which can be slow*), we only lock the Critical Section (the specific lines of code that modify shared data).

```java
public class BankAccount {
    private double balance;
    private final Object lock = new Object(); // Better than locking 'this'

    public void withdraw(double amount) {
        // ... do some preparation (no locking needed here) ...
        System.out.println("Preparing to withdraw...");

        // Only lock this critical part
        synchronized (lock) {
            if (balance >= amount) {
                balance -= amount;
            }
        }
        // ... do logging or cleanup (no locking needed here) ...
    }
}
```

## 2.3 Static Synchronization

If the method is `static`, the lock is held on the **Class object** (`ClassName.class`), not the instance. This affects all instances of the class.

```java
public static synchronized void doSomething() {
    // Locks the whole class. No other thread can enter ANY static synchronized
    // method of this class.
}
```

# 3. ReentrantLock (Explicit Lock)

Introduced in Java 5 (`java.util.concurrent.locks`), `ReentrantLock` offers more flexibility than `synchronized`.

**Why use it over synchronized?**
- **Timeout:** synchronized waits forever. ReentrantLock can tryLock() (wait for a few seconds, then give up).
- **Fairness:** You can specify new ReentrantLock(true) to grant the lock to the longest-waiting thread (First-In-First-Out). synchronized is unfair (random).
- **Interruptible:** You can interrupt a thread waiting for a lock.

## 3.1 The Pattern (Try-Finally)

**CRITICAL:** You MUST unlock in a `finally` block. If code throws an exception and you don't unlock, the system hangs forever.

```java
public class ReentrantLockDemo {
    // false = non-fair (default, faster), true = fair (slower but equitable)
    private final Lock lock = new ReentrantLock();

    public void accessResource() {
        lock.lock(); // Acquire the lock
        try {
            // Critical section
            System.out.println(Thread.currentThread().getName() + " is processing");
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            // ALWAYS unlock in finally block
            lock.unlock(); 
        }
    }
    
    public void tryToAccess() {
        try {
            // Try to get lock for only 2 seconds
            if (lock.tryLock(2, TimeUnit.SECONDS)) {
                try {
                    System.out.println("Acquired lock!");
                } finally {
                    lock.unlock();
                }
            } else {
                System.out.println("Could not get lock, doing something else...");
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}
```

# 4. ReadWriteLock (Optimization)

Sometimes, you have a resource that is **read often** but **written rarely** (e.g., a Configuration object). Using a standard lock is inefficient because multiple threads should be able to *read* at the same time.

`ReadWriteLock` allows:
- **Multiple Readers** at the same time
- **Only One Writer** (exclusive access)

```java
public class Cache {
    private final ReadWriteLock rwLock = new ReentrantReadWriteLock();
    private String data = "Default";

    public String read() {
        rwLock.readLock().lock(); // Multiple threads can hold this
        try {
            return data;
        } finally {
            rwLock.readLock().unlock();
        }
    }

    public void write(String newData) {
        rwLock.writeLock().lock(); // Only ONE thread can hold this
        try {
            data = newData;
        } finally {
            rwLock.writeLock().unlock();
        }
    }
}
```

# 5. `volatile` keyword

To understand volatile, we must look at the hardware architecture.

**The Visibility Problem** In a multi-core CPU, each core has its own Cache (L1, L2).

1. Thread A (Core 1) reads variable `flag = true` from RAM into its Cache.

2. Thread A changes `flag` to `false`. **But it only updates the Cache, not the Main RAM immediately.**

3. Thread B (Core 2) reads `flag`. It might read the old value (`true`) from RAM because Core 1 hasn't flushed the change yet.

4. **Result:** Thread B runs forever because it never "sees" the change.

**The Solution:** volatile Declaring a variable as volatile tells the JVM and OS:

>"Do not cache this variable in CPU registers or local cache. **Always read/write it directly from Main Memory (RAM).**"

```java
public class VolatileDemo {
    // Without volatile, the reader thread might loop forever
    // even if the writer changes the flag to false.
    private volatile boolean running = true;

    public void startReader() {
        new Thread(() -> {
            while (running) {
                // Do some work...
            }
            System.out.println("Reader stopped!");
        }).start();
    }

    public void stop() {
        running = false; // Because of volatile, the change is instantly visible to Reader
        System.out.println("Stop signal sent!");
    }
}
```

## 5.1 `volatile` vs `synchronized`

| Feature | volatile | synchronized |
| --------------- | --------------- | --------------- |
| **Main Goal** | **Visibility** (Changes are seen immediately). | Atomicity (Exclusive access) & Visibility. |
| **Performance** | Very light (No blocking). | Heavy (Can block threads). |
| Usage | Simple flags, status variables. | Complex logic, compound actions (check-then-act). |
| Compound Opts | **Safe? NO. (count++ is NOT safe with volatile).** | **Safe? YES.** |

