# Concurrency

[Definition from Wikipeadia](https://en.wikipedia.org/wiki/Concurrency_(computer_science)): The ability of a system to execute multiple tasks through simultaneous execution or time-sharing (context switching), sharing resources and managing interactions. Concurrency improves responsiveness, throughput, and scalability in modern computing

Concurrency is about structure. It is the ability of your program to deal with many things at once. It doesn't necessarily mean doing them at the exact same instant; it means managing multiple tasks by switching between them or overlapping their waiting times.

Imagine a kitchen with one chef (Single Core CPU).
1. The chef starts chopping onions.
2. Suddenly, the soup boils over! The chef stops chopping, turns down the heat on the soup (Context Switch), and then goes back to chopping onions.
3. Later, he puts a roast in the oven. While the roast is cooking (IO wait), he washes the dishes.

>This is **Concurrency**. The chef is making progress on the onions, the soup, and the roast "at the same time" by switching tasks efficiently. But physically, he is only doing one thing at a time.

# Paralellism

[Definition from Wikipeadia](https://en.wikipedia.org/wiki/Parallel_computing): Parallel computing is a type of computation in which many calculations or processes are carried out simultaneously. Large problems can often be divided into smaller ones, which can then be solved at the same time. There are several different forms of parallel computing: bit-level, instruction-level, data, and task parallelism. Parallelism has long been employed in high-performance computing, but has gained broader interest due to the physical constraints preventing frequency scaling. As power consumption (and consequently heat generation) by computers has become a concern in recent years, parallel computing has become the dominant paradigm in computer architecture, mainly in the form of multi-core processors.

Parallelism is about execution. It is the ability to do many things at the exact same instant. This requires hardware support (multiple CPU cores).


