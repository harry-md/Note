---
title: Spring AOP
tag: [Spring Framework, AOP, Aspect, OOP]
---
# Definition

Aspect-oriented Programming (AOP) complements Object-oriented Programming (OOP) by providing another way of thinking about program structure. The key unit of modularity in OOP is the **class**, whereas in AOP the unit of modularity is the **aspect**. Aspects enable the modularization of concerns (such as transaction management) that cut across multiple types and objects. (Such concerns are often termed "crosscutting" concerns in AOP literature.)

One of the key components of Spring is the AOP framework. While the Spring IoC container does not depend on AOP (meaning you do not need to use AOP if you don’t want to), AOP complements Spring IoC to provide a very capable **middleware solution**.

# Terminology

1. **Aspect**: A module that encapsulates a cross-cutting concern. It's like a class that defines **what** to do (**advice**) and **where** to do it (**pointcut**). In Spring, aspects can be regular Java classes annotated with `@Aspect`.

2. **Join Point**: A specific point in the program's execution where an aspect can be applied. **In Spring AOP, join points are always method executions (e.g., when a method is called)*. Note: Spring doesn't support field-level or constructor join points like full AspectJ does.

3. **Advice**: The actual code that runs at a join point. It's the **"WHAT"** part of the aspect. Spring supports five types of advice:
    - **Before**: Runs before the join point (method execution). **Useful for setup or validation**.
    - **After**: Runs after the join point, **regardless of success or failure. Like a finally block**.
    - **After Returning**: Runs after the join point if it completes successfully (no exception thrown). **You can access the return value**.
    - **After Throwing**: Runs after the join point if an exception is thrown. **You can access the exception**.
    - **Around**: The most powerful; it wraps the join point, allowing you to control if/when the method proceeds, modify arguments/return values, or handle exceptions. It's like **a try-catch-finally around the method call**.

>Around advice is the most general kind of advice. Since Spring AOP, like AspectJ, provides a full range of advice types, we recommend that you use the least powerful advice type that can implement the required behavior. For example, if you need only to update a cache with the return value of a method, you are better off implementing an after returning advice than an around advice, although an around advice can accomplish the same thing. Using the most specific advice type provides a simpler programming model with less potential for errors. For example, you do not need to invoke the `proceed()` method on the `JoinPoint` used for around advice, and, hence, you cannot fail to invoke it.

4. **Pointcut**: Defines **"WHERE"** the advice should be applied. It's a expression that matches join points. Spring uses AspectJ's pointcut expression language (e.g., execution(* com.example.service.*.*(..)) means "any method in any class under com.example.service package").

5. **Introduction (or Inter-type Declaration)**: Allows an aspect to add new methods or fields to existing classes. This is less common but powerful for mixins.

6. **Target Object**: The original object (bean) that the aspect is advising.

7. **AOP Proxy**: Spring creates a proxy object that wraps the target object. Calls to the target go through this proxy, which applies the aspects. Spring uses:

8. **JDK Dynamic Proxies**: For interfaces (default if the bean implements an interface).
CGLIB Proxies: For classes without interfaces (requires CGLIB library, included in Spring).

9. **Weaving**: The process of applying aspects to the target objects. In Spring, this happens at runtime via proxies (not at compile-time like AspectJ).
