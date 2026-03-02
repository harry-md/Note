---
tag: [Java, Process, Parallelism, Concurrency]
---

# 1. Definition

[Definition from Wikipedia](https://en.wikipedia.org/wiki/Process_(computing)): In computing, a process is the instance of a computer program that is being executed by one or many threads. There are many different process models, some of which are light weight, but almost all processes (even entire virtual machines) are rooted in an OS process which comprises the program code, assigned system resources, physical and logical access permissions, and data structures to initiate, control and coordinate execution activity. Depending on the OS, a process may be made up of multiple threads of execution that execute instructions concurrently.


**In more simple term**: A process is like an abstract 'container' entity for running threads. Therefore, a process can't execute code because it just contains threads and threads are the ones doing the task. The OS assigns some system resources to the process when it is created.

![[Concepts-_Program_vs._Process_vs._Thread.jpg]]
*Overview of Process*

---

# 2. In Java context ☕️

You don't see people use `Process` much in Java (except some specific cases). The reason is in Java, you create a process equivalent to spin up entire JVM (which is a lot). So in Java world people prefer using [Thread](Thread.md) to achieve [concurrency and parallelism](Concurrency-and-Parallelism.md) instead of Process.

You're going to need to use `Process` when:
- Speed is less important than safety or isolation.
- Tasks should NOT see each other's memory (e.g., Chrome tabs - so one crashing tab doesn't kill the browser).
- The code is risky. If a process crashes, the OS cleans it up, and your main app stays alive.
- You need to run something outside Java (e.g., a Python script, a system command, or a legacy C++ app).

Example: We need to run some command from the Java application
```java
public class ProcessDemo {
    public static void main(String[] args) {
        try {
            // run the date command
            ProcessBuilder builder = new ProcessBuilder("date");
            
            Process process = builder.start();
            // we cannot just read variables, we have to read the OutputStream
            BufferedReader reader = new BufferedReader(
                new InputStreamReader(process.getInputStream())
            );

            String line;
            while ((line = reader.readLine()) != null) {
                System.out.println("The external process says: " + line);
            }
            
            int exitCode = process.waitFor();
            System.out.println("Process exited with code: " + exitCode);

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

