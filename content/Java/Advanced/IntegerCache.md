---
title: Integer Cache
tag: [Java, Integer Cache, List]
---

I just found out something amazing about how Java handles numbers inside Lists, and I need to document it here so I don't forget. This is about memory, performance, and some clever optimizations by the JVM.

# 1. The Fast Path - Primitive Array `int[]`

First, let's look at a simple array of primitive integers.

```java
int[] myPrimitives = new int[]{1, 100, 2000};
```

How does this look in memory?

This is performance heaven for a CPU. The variable `myPrimitives` on the Stack holds the starting address of a block of memory on the Heap. Inside that block on the Heap, you find the actual values `1`, `100`, and `2000` placed right next to each other.

When you iterate through this array (e.g., `myPrimitives[0]`, then `myPrimitives[1]`), the CPU does one jump to the Heap and then simply reads the next memory slot. Modern CPUs are designed for this; they are super efficient at reading contiguous blocks of data. This is called spatial locality.

![[fastest.svg]]

# 2. The Slower Path - `ArrayList<Integer>`

Now, let's look at the object world. Java doesn't allow primitive types inside `ArrayLists`. It automatically wraps int into an object type called `Integer`. This is called autoboxing.

```java
ArrayList<Integer> myList = new ArrayList<>();
myList.add(1);
myList.add(100);
myList.add(2000);
```

## 2.1 What a "Reference Array" Really Is

A `ArrayList` is essentially a wrapper around an internal array of pointers, or more formally, references. When you call `myList`.`get(i)`, the JVM isn't giving you the value. It’s giving you the address where the value lives.

On modern JVMs running on 64-bit systems, these pointers are often compressed to save space. So, each slot in the internal array stores only **4 bytes**.

## 2.2 Enter Java’s "Integer Cache"

This is a cool trick Java uses to save RAM. Java keeps a pool (or "cache") of pre-created `Integer` objects for numbers within a small range, typically from **-128 to 127**.

When you add `myList.add(1)`, Java doesn’t create a new object. It just finds the existing object for 1 in its cache and adds a pointer to it in your list.

## 2.3 Visualizing a Cached ArrayList

1. CPU reads the address of the pointer array.
2. CPU reads slot 0 (`100`). This is a reference.
3. **THE SLOW JUMP**: CPU must now jump from the pointer array on the Heap to a completely different location on the Heap to read the real value `1` inside the Integer Object.
4. CPU returns to the array, reads slot 1 (`104`), and performs another jump.

![[fast.svg]]

# 3. The Fragmentation Trap

The situation gets even slower when your integers are outside the cache range. When you add `myList.add(2000)`, Java is forced to create a brand new object somewhere in Heap memory. The problem is that new objects aren’t guaranteed to be placed together in a tight block.

Let's visualize this fragmentation:

In this scenario, while the pointers themselves (those 4-byte slots) are placed neatly together, the objects they point to (`200`, `300`, `1000`) are scattered all over the Heap memory.

Now, the performance hit is severe. When you jump to read object `200`, your CPU Cache might load some "neighboring" data from that random area in memory. When you immediately jump to find object `300`, that "neighboring" data is useless.

This is a complete **CPU Cache Miss**. The CPU must waste precious time waiting for the RAM to find the memory page for object `300`, then again for `1000`. This is memory **fragmentation**.

![[slow.svg]]

# 4. Comparing performance for iteration

`int[]`: Contiguous values. Super fast iteration. Max CPU Cache utilization.

`ArrayList<Integer>` **with cache**: Contiguous pointers. Values are elsewhere. Slightly faster than non-cached lists because cached objects are often somewhat close to each other in memory. But still much slower than a primitive array.

`ArrayList<Integer>` **without cache** (Fragmentation): Contiguous pointers. Values are scattered everywhere. Extremely slow iteration due to high CPU Cache miss rate.


**Key Takeaway**
1. **Iterating primitive lists is always faster than iterating object lists** (this is why `List<int>` in C# so efficient). The extra memory jump kills performance.
2. The **Integer Cache** is optimized for **RAM usage** and correct == comparisons, not primarily for iteration speed.
3. Memory **fragmentation** turns your performance from slower (pointer chase) to abysmal (RAM wait). Avoid massive object arrays if you only care about fast iteration.
