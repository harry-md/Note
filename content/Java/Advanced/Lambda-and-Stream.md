---
title: Lambda & Stream
---

# 1. Lambda Expression

## 1.1 Definition & Syntax

A **lambda expression** is a short block of code that takes in parameters and returns a value. Lambdas look similar to methods, but they do not need a name, and they can be written right inside a method body.

**Syntax:**
```
parameter -> expression

(parameter1, parameter2) -> expression

(parameter1, parameter2) -> {
  // code block
  return result;
}
```

**Why Use Lambda Expressions?**
- **Concise Code:** Reduce boilerplate compared to anonymous classes.
- **Functional Programming:** Treat functions as first-class citizens.
- **Improved Readability:** Code is easier to read and maintain.
- **Enhanced Collections and Streams:** Simplify operations like filtering, mapping, and iterating.

## 1.2 Using Lambda Expressions

Lambdas are often passed as arguments to methods. For example, you can use a lambda in the `forEach()` method of an `ArrayList`:

```java
public static void main(String[] args) {
    ArrayList<Integer> numbers = Arrays.asList(1, 2, 3, 4);
    numbers.forEach(n -> System.out.println(n));
}
```

## 1.3 Lambdas in Variables

A lambda expression can be stored in a variable. The variable's type must be an interface with exactly one method (a **functional interface**). The lambda must match that method's parameters and return type.

Java includes many built-in functional interfaces, such as `Consumer` (from the `java.util package`) used with lists.

```java
public static void main(String[] args) {
    ArrayList<Integer> numbers = Arrays.asList(1, 2, 3, 4);
    Consumer<Integer> method = (n) -> { System.out.println(n); };
    numbers.forEach(method);
}
```

## 1.4 Lambdas as Method Parameters

You can also pass a lambda expression to a method. The method's parameter must be a functional interface. Calling the interface's method will then run the lambda expression:

```java
interface StringFunction {
    String run(String str);
}

public class Main {
    public static void main(String[] args) {
        StringFunction exclaim = (s) -> s + "!";
        StringFunction ask = (s) -> s + "?";
        printFormatted("Hello", exclaim);
        printFormatted("Hello", ask);
    }

    public static void printFormatted(String str, StringFunction format) {
        String result = format.run(str);
        System.out.println(result);
    }
}
```

## 1.5 Anonymous Class vs. Lambda Expression

Anonymous Class

```java
interface Greeting {
    void sayHello();
}

public class Main {
    public static void main(String[] args) {
        Greeting g = new Greeting() {
            public void sayHello() {
            System.out.println("Hello from anonymous class");
            }
        }; 
        g.sayHello();
    }
}
```
Lambda Expression (Functional Interface)

```java
// interface has exactly one abstract method, lambda expression will provide its implementation
@FunctionalInterface // Optional but recommended
interface Greeting {
    void sayHello();
}

public class Main {
    public static void main(String[] args) {
        Greeting g = () -> System.out.println("Hello from lambda");
        g.sayHello();
    }
}
```

>**Rule of thumb:** Use a `lambda` for short, single-method interfaces. Use an anonymous class when you need to override multiple methods, add fields, or extend a class.

## 1.6 Common built-in functional interfaces

| Interface | Method | Purpose |
| --------------- | --------------- | --------------- |
| `Predicate` | `boolean test(T t)` | Tests a given condition and returns true or false. |
| `Consumer` | `void acept(T t)` | Performs an action on the given argument without returning a result. |
| `Supplier` | `T get()` | Supplies or generates a result without taking any input. |
| `Comparator<T>` | `int compare(T o1, T o2)` | Compares two objects to determine their order. |
| `Comparable<T>` | `int compareTo(T o)` | Defines the natural ordering for objects of a class. |


# 2. Stream

## 2.1 Sequential Stream

Stream was introduced in Java 8, the Stream API is used to process collections of objects. A stream in Java is a sequence of objects that supports various methods that can be pipelined to produce the desired result. 


## 2.2 Parallel Stream
