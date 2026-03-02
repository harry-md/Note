---
title: Generics
tag: [Java, Generics, Collection, Datastructure]
---

# 1. Generic Programming

Generics allow you to write classes, interfaces, and methods that work with different data types, without having to specify the exact type in advance.

This makes your code more flexible, reusable, and type-safe.

## 1.1 Why Use Generics?
- **Code Reusability:** Write one class or method that works with different data types.
- **Type Safety:** Catch type errors at compile time instead of runtime.
- **Cleaner Code:** No need for casting when retrieving objects.

## 1.2 Generic Method Example

```java
public class Main {
  // Generic method: works with any type T
  public static <T> void printArray(T[] array) {
    for (T item : array) {
      System.out.println(item);
    }
  }

  public static void main(String[] args) {
    String[] names = {"Jenny", "Liam"};

    Integer[] numbers = {1, 2, 3};

    printArray(names);
    printArray(numbers);
  }
}
```

>**Explanation:** `<T>` is a generic type parameter - it means the method can work with any referenced type: `String`, `Integer`, `Double`, etc.
>The method `printArray()` takes an array of type `T` and prints every element.
>When you call the method, Java figures out what `T` should be based on the argument you pass in.

## 1.3 Bounded Types

You can use the extends keyword to limit the types a generic class or method can accept.

For example, you can require that the type must be a subclass of Number:

```java
// can extends multiple class with &
// class Stats<T extends Number & Serializable> {
class Stats<T extends Number> {
  T[] nums;
  Stats(T[] nums) {
    this.nums = nums;
  }
  double average() {
    double sum = 0;
    for (T num : nums) {
      sum += num.doubleValue();
    }
    return sum / nums.length;
  }
}
public class Main {
  public static void main(String[] args) {
    Integer[] intNums = {10, 20, 30, 40};
    Stats<Integer> intStats = new Stats<>(intNums);
    System.out.println("Integer average: " + intStats.average());

    Double[] doubleNums = {1.5, 2.5, 3.5};
    Stats<Double> doubleStats = new Stats<>(doubleNums);
    System.out.println("Double average: " + doubleStats.average());
  }
}
```

Even though int values are used in the first case, the `.doubleValue()` method converts them to double, so the result is shown with a decimal point.

>**Explanation:** `<T extends Number>`: Restricts `T` to only work with numeric types like `Integer`, `Double`, or `Float`.
`.doubleValue()`: Converts any number to a `double` for calculation.
Works for any array of numbers as long as the elements are subclasses of `Number`.

## 1.4 Wildcards

In Java Generics, wildcards are used when you don’t know the exact type. They let you write flexible and reusable code. The wildcard is represented by a `?` (question mark). Wildcards are mainly used in method parameters to accept different generic types safely.

### 1.4.1 Upper Bounded Wildcards

These wildcards can be used when you want to relax the restrictions on a variable. For example, say you want to write a method that works on `List<Integer>`, `List<Double>` and `List<Number>`, you can do this using an upper bounded wildcard.

```java
private static double sum(List<? extends Number> list) {
    double sum = 0.0;
    for (Number i : list) {
        sum += i.doubleValue();
    }
    return sum;
}

public static void main(String[] args) {
    List<Integer> list1 = Arrays.asList(4, 5, 6, 7);
    System.out.println("Total sum is:" + sum(list1));

    List<Double> list2 = Arrays.asList(4.1, 5.1, 6.1);
    System.out.print("Total sum is:" + sum(list2));
}
```

**Output:**

```
Total sum is:22.0
Total sum is:15.299999999999999
```

>**Explanation:** In the above program, `list1` holds `Integer` values and `list2` holds `Double` values. Both are passed to the sum method, which uses a wildcard `<? extends Number>`. This means it can accept any list of a type that is a subclass of `Number`, like `Integer` or `Double`.

### 1.4.2 Lower Bounded Wildcards

It is expressed using the wildcard character (`?`), followed by the super keyword, followed by its lower bound: `<? super A>`.

```java
public static void printOnlyIntegerClassorSuperClass(List<? super Integer> list) {
    System.out.println(list);
}

public static void main(String[] args) {
    List<Integer> list1 = Arrays.asList(4, 5, 6, 7);
    printOnlyIntegerClassorSuperClass(list1);

    List<Number> list2 = Arrays.asList(4, 5, 6, 7);
    printOnlyIntegerClassorSuperClass(list2);
}
```

**Output:**
```
[4, 5, 6, 7]
[4, 5, 6, 7]
```

>**Explanation:** Here, the method `printOnlyIntegerClassorSuperClass` accepts only `Integer` or its superclasses (like `Number`). If you try to pass a list of `Double`, it gives a compile-time error because `Double` is not a superclass of `Integer`.

>**NOTE:** Use extend wildcard when you want to get values out of a structure and super wildcard when you put values in a structure. Don’t use wildcard when you get and put values in a structure. You can specify an upper bound for a wildcard or you can specify a lower bound, but you cannot specify both.

### 1.4.3 Unbounded Wildcard

This wildcard type is specified using the wildcard character (`?`), for example, `List`. This is called a list of unknown types. These are useful in the following cases:
- When writing a method that can be employed using functionality provided in `Object` class.
- When the code is using methods in the generic class that doesn't depend on the type parameter

```java
private static void printlist(List<?> list) {
    System.out.println(list);
}

public static void main(String[] args) {
    List<Integer> list1 = Arrays.asList(1, 2, 3);
    List<Double> list2 = Arrays.asList(1.1, 2.2, 3.3);

    printlist(list1);
    printlist(list2);
}
```

## 1.5 Java Collection Framework (`java.util.*`)

![[collection.webp]]

### 1.5.1 Collection

Store single value set (e.g., `List`, `Set`, ...)

**Static Methods in `Collection`:** `sort()`, `reverse()`, `shuffle()`, `max()`, `min()`, `frequency()`

<h4>1.5.1.1 List</h4>

**Array List:**
- Store elements sequentially in an array.
- If add a new element exceed its capacity, create a new array and copy all the elements from the previous array to the new array.

**Time Complexity**

| Operation | Best | Worst | Average |
| --------------- | --------------- | --------------- | --------------- |
| Read | O(1) | O(1) | O(1) |
| Add | O(1) (at the end) | O(n) (at a specific index) | O(n) |
| Remove | O(1) (at the end) | O(n) (all elements have to shift to the left) | O(n) |


**Linked List:**
- Storing elements in a linked list style.

**Time Complexity**

| Operation | Best | Worst | Average |
| --------------- | --------------- | --------------- | --------------- |
| Read | O(n) (have to walk node by node)| O(n) | O(n)  |
| Add | O(1) (at the begining/end) | O(n) (finding the node is the expensive part) | O(n) |
| Remove | O(1) (at the begining/end) | O(n) (finding the node is the expensive part) | O(n) |

<h4>1.5.1.2. Set</h4>

- Structure to store unique elements (there is no two elements with the same value in set).
- Implementation: `HashSet`, `LinkedHashSet`, `TreeSet`
- Elements in `HashSet` won't be ordered when inserting, `LinkedHashSet`'s elements will be ordered when inserting

### 1.5.2 Map
- The `Map` interface is a part of the Java Collections Framework and is used to store key-value pairs. Each key must be unique, but values can be duplicated.
- A Map is useful when you want to associate a key (like a name or ID) with a value (like an age or description).
- Common classes that implement Map:
    - HashMap - fast and unordered
    - TreeMap - sorted by key
    - LinkedHashMap - ordered by insertion
