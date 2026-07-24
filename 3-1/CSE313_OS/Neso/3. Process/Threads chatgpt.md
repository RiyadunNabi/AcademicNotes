Awesome! 😄 Let's continue.

Now we start the **Threads** part of the chapter. This is one of the most asked topics in Operating Systems. 

---

# 📖 Lesson 8 — Threads (From Scratch)

---

# First, Let's Remember

Earlier we learned:

> A **process** is a program in execution.

Example:

```text
Chrome
```

When you open Chrome,

it becomes a process.

Question:

**Does Chrome do only one thing?**

No.

Chrome can:

* Play YouTube 🎥
* Download files 📥
* Open many tabs 🌐
* Play music 🎵
* Check spelling
* Load images

Can one process really do all these at the same time?

Yes!

How?

Using **Threads**.

---

# Imagine This

Suppose you own a restaurant.

The restaurant is one business.

Inside the restaurant,

there are many workers.

```text
Restaurant

├── Cook
├── Waiter
├── Cashier
└── Cleaner
```

Everyone works together.

The restaurant is like a **Process**.

Each worker is like a **Thread**.

---

# What is a Thread?

### Definition

> A **thread** is the smallest unit of CPU execution inside a process.

Don't memorize yet.

Let's understand.

---

# Simple Definition

A **thread** is simply a **worker inside a process**.

One process

↓

Can contain

↓

One or more threads.

---

# Visual

```text
Process

│

├── Thread 1

├── Thread 2

├── Thread 3
```

Each thread performs a different task.

---

# Example 1 — Chrome

Suppose Chrome has three tabs.

```text
Chrome Process

│

├── Thread 1 → YouTube

├── Thread 2 → Facebook

└── Thread 3 → Gmail
```

Each thread works independently.

---

# Example 2 — Microsoft Word

When you type,

Word is doing many things.

```text
Word Process

│

├── Typing

├── Auto Save

├── Spell Check

└── Printing
```

Imagine if Word had only one thread.

It might stop typing while it checks spelling.

That would feel very slow.

Instead,

different threads handle different tasks.

---

# Process vs Thread

Students often confuse these.

Let's compare.

| Process             | Thread                 |
| ------------------- | ---------------------- |
| Large unit          | Small unit             |
| Own memory          | Shares process memory  |
| Heavy               | Lightweight            |
| Expensive to create | Faster to create       |
| Contains threads    | Lives inside a process |

---

# Easy Analogy

Imagine a house.

```text
House

↓

Process
```

People living inside

```text
People

↓

Threads
```

The people share:

* Kitchen
* Bathroom
* Living room

Similarly,

threads share the process's resources.

---

# Memory of a Process

Suppose we have

```text
Chrome Process
```

Its memory looks like this.

```text
+----------------------+
| Code                 |
| Heap                 |
| Global Variables     |
+----------------------+
```

Now three threads are created.

```text
Chrome

│

├── Thread 1

├── Thread 2

└── Thread 3
```

Do they each get a separate copy of Chrome's code?

❌ No.

They all share it.

---

# Shared Resources

Threads share:

✅ Code

✅ Heap

✅ Global Variables

✅ Open Files

Example

```text
Chrome

↓

Shared Memory

↑

Thread 1

Thread 2

Thread 3
```

All threads use the same memory.

---

# Private Resources

Each thread has its own:

* Program Counter
* Registers
* Stack

Why?

Because each thread may be executing a different function.

Example:

Thread 1

```cpp
downloadFile();
```

Thread 2

```cpp
playVideo();
```

They are at different points in the code, so each needs its own execution state.

---

# Why Do We Need Threads?

Imagine downloading a large file.

Without threads:

```text
Download

↓

Cannot use browser
```

Everything freezes.

With threads:

```text
Thread 1

↓

Download

Thread 2

↓

Scroll Page

Thread 3

↓

Play Video
```

Everything works smoothly.

---

# Benefits of Threads

### 1. Faster Execution

Many tasks happen at the same time.

Example:

Chrome loads:

* Images
* Text
* Ads

using different threads.

---

### 2. Better CPU Utilization

If one thread is waiting for the Internet,

another thread can continue working.

The CPU doesn't stay idle.

---

### 3. Easy Communication

Remember IPC?

Processes need special methods like shared memory or message passing.

Threads are easier.

Why?

Because they already share memory.

Example

Thread 1 writes

```text
count = 10
```

Thread 2 can immediately read

```text
count
```

No IPC is needed.

---

### 4. Lower Cost

Creating a new process takes time.

The OS must create:

* PCB
* Memory
* Resources

Creating a thread is much cheaper.

---

# Disadvantage of Threads

Since threads share memory,

they can accidentally change each other's data.

Example

Thread 1

```cpp
balance = balance + 100;
```

Thread 2

```cpp
balance = balance - 50;
```

If both run at the same time,

the result may become incorrect.

This is called a **Race Condition**.

We'll learn how to solve it later using synchronization.

---

# Single-Threaded vs Multi-Threaded

## Single Thread

```text
Process

↓

Thread

↓

Task 1

↓

Task 2

↓

Task 3
```

Everything happens one after another.

---

## Multi-Threaded

```text
Process

│

├── Thread 1

├── Thread 2

└── Thread 3
```

Multiple tasks can make progress concurrently.

---

# Real-Life Example

Suppose you're cooking.

Single-threaded:

```text
Boil Rice

↓

Cook Chicken

↓

Make Salad
```

Total time is longer because you do everything one by one.

Multi-threaded:

```text
Boil Rice

Cook Chicken

Make Salad
```

You overlap tasks, so the meal is ready sooner.

---

# Process vs Thread (Memory)

## Two Processes

```text
Process A

Memory A


Process B

Memory B
```

Memory is completely separate.

---

## One Process with Threads

```text
Process

↓

Shared Memory

↑

Thread 1

Thread 2

Thread 3
```

Threads share the process's memory.

---

# Exam Questions

### Q1. What is a thread?

**Answer:**

A thread is the smallest unit of CPU execution within a process.

---

### Q2. Can one process have multiple threads?

✅ Yes.

---

### Q3. What do threads share?

* Code
* Heap
* Global Variables
* Open Files

---

### Q4. What does each thread have separately?

* Program Counter
* Registers
* Stack

---

### Q5. Why are threads faster than processes?

Because creating and switching between threads requires less overhead, and threads share the same process resources.

---

# 📝 Lesson 8 Summary

* A **thread** is the smallest unit of execution inside a process.
* A process can have **one or many threads**.
* Threads **share** the process's code, memory, and open files.
* Each thread has its own **program counter, registers, and stack**.
* Threads improve responsiveness and efficiency but require synchronization because they share data.

---

## 📚 Next Lesson (Lesson 9)

We'll learn **Types of Threads**:

* User-Level Threads (ULT)
* Kernel-Level Threads (KLT)
* Differences between them
* Advantages and disadvantages
* Easy diagrams and real-life examples

This is another very common exam topic and builds directly on today's lesson.
----
----
Awesome bro! 😄 Now we continue with one of the **most confusing topics** for beginners. Don't worry—by the end of this lesson, you'll understand it clearly.

---

# 📖 Lesson 9 — Types of Threads

In the previous lesson, we learned:

* A **process** can have many **threads**.
* Threads execute different tasks.
* Threads share the process's memory.

Now the question is:

> **Who manages these threads?**

There are two possibilities:

1. The **User Program (Library)**
2. The **Operating System (Kernel)**

That's why there are two main types of threads.

```text
Threads

│

├── User-Level Threads (ULT)

└── Kernel-Level Threads (KLT)
```

---

# Before Learning the Types...

First understand two words.

## What is User Space?

User Space is where your applications run.

Examples:

* Chrome
* VS Code
* Spotify
* Calculator

Applications cannot directly control the CPU or hardware.

---

## What is Kernel Space?

Kernel Space is where the Operating System runs.

The kernel can:

* Control the CPU
* Allocate memory
* Access hardware
* Create processes
* Schedule threads

Think of it as the **boss** of the computer.

---

# Real-Life Analogy

Imagine a university.

### Students

```text
Students
```

They represent **User Space**.

---

### Principal

```text
Principal
```

Represents the **Kernel**.

Students cannot directly make university rules.

Only the principal can.

---

# User-Level Threads (ULT)

Let's start with the easier one.

## Definition

A **User-Level Thread** is managed by a **thread library**, not by the operating system.

---

# What does this mean?

Suppose Chrome creates four threads.

```text
Chrome

│

├── Thread A

├── Thread B

├── Thread C

└── Thread D
```

The operating system doesn't know there are four threads.

The OS thinks:

```text
One Process

↓

Chrome
```

Inside Chrome, a thread library manages the threads.

---

# Visual

```text
Application

↓

Thread Library

↓

Thread A

Thread B

Thread C

Thread D



Operating System

↓

Chrome Process
```

Notice:

The OS only sees **one process**.

---

# Example

Imagine a classroom.

The teacher sees:

```text
Group 1
```

But inside Group 1,

the students divide the work themselves.

The teacher doesn't know who is doing what.

That's exactly how User-Level Threads work.

---

# Advantages of ULT

## 1. Very Fast

Why?

Because the operating system is **not involved**.

Creating a new thread is just a library operation.

---

## Example

Thread Library says

```text
Create Thread
```

Done.

No system call.

Very fast.

---

## 2. Easy to Create

No kernel interaction.

Everything happens inside the application.

---

## 3. Less Overhead

Since the OS is not involved,

switching between User-Level Threads is very quick.

---

# Disadvantage

Here's the biggest problem.

Suppose

Thread A

needs to read from the disk.

```text
Thread A

↓

Disk Read
```

Disk access is slow.

Thread A becomes blocked.

Question:

Can Thread B continue?

❌ No.

Why?

Because the operating system only knows about **one process**.

So it blocks the **entire process**.

```text
Thread A → Waiting

↓

Whole Process Stops

↓

Thread B Stops

Thread C Stops

Thread D Stops
```

This is the biggest disadvantage of ULT.

---

# Kernel-Level Threads (KLT)

Now let's learn the second type.

## Definition

A **Kernel-Level Thread** is managed directly by the operating system.

---

# What does this mean?

Suppose Chrome creates four threads.

```text
Chrome

│

├── Thread A

├── Thread B

├── Thread C

└── Thread D
```

This time,

the operating system knows about **every thread**.

---

# Visual

```text
Kernel

↓

Thread A

Thread B

Thread C

Thread D
```

Each thread has its own scheduling information.

The scheduler can choose any thread.

---

# Example

Imagine a teacher.

Instead of seeing only the group,

the teacher knows every student.

```text
Teacher

↓

Student A

Student B

Student C

Student D
```

The teacher can directly choose who should work.

---

# Advantage

Suppose Thread A waits for the disk.

```text
Thread A

↓

Waiting
```

Can Thread B continue?

✅ Yes.

Because the OS knows every thread separately.

```text
Thread A → Waiting

Thread B → Running

Thread C → Ready

Thread D → Ready
```

The entire process does **not** stop.

---

# Another Advantage

If your computer has multiple CPU cores,

different kernel threads can run on different cores.

Example:

```text
CPU Core 1

↓

Thread A



CPU Core 2

↓

Thread B
```

This improves performance.

---

# Disadvantage

Kernel-Level Threads are slower to manage.

Why?

Every thread operation requires help from the operating system.

Example:

Creating a thread

↓

System Call

↓

Kernel

↓

Create Thread

This takes more time than User-Level Threads.

---

# User vs Kernel Threads

| User-Level Threads                 | Kernel-Level Threads          |
| ---------------------------------- | ----------------------------- |
| Managed by thread library          | Managed by OS kernel          |
| OS doesn't know individual threads | OS knows every thread         |
| Faster to create                   | Slower to create              |
| Lower overhead                     | Higher overhead               |
| If one thread blocks, all block    | Only blocked thread stops     |
| Limited parallelism                | Can run on multiple CPU cores |

---

# Easy Analogy

Imagine a football coach.

### User-Level Threads

Coach only knows:

```text
Team A
```

Inside Team A,

players organize themselves.

Coach doesn't know each player.

---

### Kernel-Level Threads

Coach knows every player.

```text
Coach

↓

Player A

Player B

Player C
```

Coach can substitute one player without stopping the whole team.

---

# Which One is Better?

There is no universal answer.

### User-Level Threads

✅ Faster

✅ Simpler

❌ Blocking affects all threads

---

### Kernel-Level Threads

✅ Better multitasking

✅ Better multicore support

❌ More overhead

Modern operating systems like **Linux**, **Windows**, and **macOS** mainly use **Kernel-Level Threads** because they support true parallel execution on multicore processors.

---

# Exam Questions

### Q1. What is a User-Level Thread?

A thread managed by a thread library in user space rather than by the operating system.

---

### Q2. What is a Kernel-Level Thread?

A thread managed directly by the operating system kernel.

---

### Q3. What happens if a User-Level Thread blocks?

The operating system blocks the **entire process**, so the other user-level threads cannot run.

---

### Q4. What happens if a Kernel-Level Thread blocks?

Only that thread blocks. Other threads in the same process can continue running.

---

### Q5. Which type supports multicore processors better?

✅ **Kernel-Level Threads**, because the kernel can schedule different threads on different CPU cores.

---

# 📝 Lesson 9 Summary

* There are two main thread types:

  * **User-Level Threads (ULT)**: Managed by a user-space thread library.
  * **Kernel-Level Threads (KLT)**: Managed by the operating system.
* **ULTs** are fast but have a major drawback: if one thread blocks, the whole process blocks.
* **KLTs** have more overhead but support better scheduling, blocking behavior, and multicore execution.
* Modern operating systems generally prefer **Kernel-Level Threads**.

---

## 📚 Next Lesson (Lesson 10)

We'll learn **Multithreading Models**, including:

* Many-to-One
* One-to-One
* Many-to-Many
* Two-Level Model

These models explain **how user-level threads are mapped to kernel-level threads**, which is one of the most important concepts in the Threads chapter.
---
---
Awesome bro! 🔥 Now we reach one of the **most confusing topics** in OS.

Many students memorize these models and forget them.

**We're NOT going to memorize.**

We'll understand them using simple pictures and real-life examples.

---

# 📖 Lesson 10 — Multithreading Models

---

# First, Let's Recall

In Lesson 9, we learned:

There are two types of threads.

```text
User-Level Threads (ULT)

Kernel-Level Threads (KLT)
```

Now a question arises.

Suppose Chrome creates

```text
Thread 1

Thread 2

Thread 3
```

How are these connected to the kernel?

Does

* one user thread become one kernel thread?

or

* many user threads become one kernel thread?

or something else?

These relationships are called

# Multithreading Models.

---

# Imagine This

Suppose you own a company.

You have workers.

```text
Worker A

Worker B

Worker C
```

Now,

who reports to the manager?

There are several possibilities.

Exactly the same happens with threads.

---

# There are Four Models

```text
1. Many-to-One

2. One-to-One

3. Many-to-Many

4. Two-Level Model
```

We'll learn them one by one.

---

# 1. Many-to-One Model

Let's start with the easiest.

---

## Picture

```text
User Threads

Thread A

Thread B

Thread C

      │

      │

      ▼

One Kernel Thread
```

Notice.

Many user threads

↓

One kernel thread.

That's why it is called

**Many-to-One**.

---

# What Happens?

Suppose Chrome has

```text
Thread A

Thread B

Thread C
```

The operating system sees only

```text
One Kernel Thread
```

All three user threads use the same kernel thread.

---

# Example

Imagine three students.

```text
Student A

Student B

Student C
```

Only one representative talks to the teacher.

```text
Teacher

↑

Representative
```

Teacher never sees the other students.

Similarly,

the kernel only sees one thread.

---

# Advantage

Very fast.

Why?

Because thread management happens in user space.

No kernel involvement.

---

# Disadvantage

Huge problem.

Suppose

Thread A

waits for disk I/O.

```text
Thread A

↓

Waiting
```

Since all threads share one kernel thread,

the whole process blocks.

```text
Thread A

Waiting

↓

Thread B Stops

↓

Thread C Stops
```

Nobody can continue.

---

# Another Problem

Can two threads run on two CPU cores?

Example:

```text
Core 1

?

Core 2

?
```

No.

Because there is only

ONE kernel thread.

Only one CPU core can execute it.

---

# Summary

```text
Many User Threads

↓

One Kernel Thread

↓

Fast

↓

Poor Parallelism
```

---

# 2. One-to-One Model

Now let's improve it.

---

## Picture

```text
User Thread A

↓

Kernel Thread A


User Thread B

↓

Kernel Thread B


User Thread C

↓

Kernel Thread C
```

Each user thread gets its own kernel thread.

---

# Example

Imagine

three students.

Each talks directly to the teacher.

```text
Teacher

↑

Student A

Student B

Student C
```

Nobody shares.

---

# Advantage

Suppose Thread A waits.

```text
Thread A

↓

Waiting
```

Can Thread B continue?

YES.

```text
Thread B

↓

Running
```

The kernel knows every thread.

---

# Another Advantage

Suppose

you have

two CPU cores.

```text
Core 1

↓

Thread A


Core 2

↓

Thread B
```

Now two threads can run simultaneously.

Great performance.

---

# Disadvantage

Kernel threads are expensive.

Suppose

you create

```text
5000 Threads
```

The operating system must create

```text
5000 Kernel Threads
```

That uses lots of memory and CPU time.

---

# Summary

```text
One User Thread

↓

One Kernel Thread

↓

Excellent Parallelism

↓

Higher Overhead
```

---

# 3. Many-to-Many Model

Now comes the smartest model.

---

## Picture

```text
User Threads

T1

T2

T3

T4

T5

      │

      ▼

Kernel Threads

K1

K2

K3
```

Notice.

Five user threads.

Only three kernel threads.

Not one.

Not five.

Somewhere in between.

---

# What Happens?

The thread library maps

many user threads

onto

many kernel threads.

Example

```text
User

T1

↓

K1


T2

↓

K2


T3

↓

K2


T4

↓

K3


T5

↓

K1
```

The mapping can change.

---

# Why is This Good?

Suppose Thread 1 blocks.

Other kernel threads still work.

Good.

Suppose

100 user threads exist.

The OS doesn't need

100 kernel threads.

Less overhead.

---

# Real-Life Example

Imagine

100 students.

Only

10 teachers.

Teachers help students.

Students don't each need their own teacher.

---

# Advantage

Best balance.

* Good performance
* Less overhead
* Better parallelism

---

# Disadvantage

Very difficult to implement.

---

# 4. Two-Level Model

This is almost the same as

Many-to-Many.

But with one extra feature.

---

## Picture

```text
User Threads

T1

↓

K1

(Locked)


T2

↓

K2


T3

↓

K2
```

Normally,

threads are mapped like Many-to-Many.

But,

if needed,

a user thread can be permanently connected

to one kernel thread.

---

# Why?

Suppose

one thread is extremely important.

Example

```text
Video Playback
```

You don't want delays.

So you permanently assign

one kernel thread to it.

---

# Easy Analogy

Hospital.

Many patients.

Many doctors.

Normally,

patients see any doctor.

But

VIP patient

↓

Personal Doctor

Always.

That is the

Two-Level Model.

---

# Comparison Table

| Model        | Mapping                                    | Advantage                                  | Disadvantage                                        |
| ------------ | ------------------------------------------ | ------------------------------------------ | --------------------------------------------------- |
| Many-to-One  | Many ULT → One KLT                         | Very fast                                  | No true parallelism; one blocking thread blocks all |
| One-to-One   | One ULT → One KLT                          | Excellent performance on multicore systems | High overhead                                       |
| Many-to-Many | Many ULT → Many KLT                        | Balanced performance and overhead          | More complex                                        |
| Two-Level    | Many-to-Many + optional dedicated mappings | Flexible                                   | Very complex                                        |

---

# Memory Trick 🎯

Imagine buses.

### Many-to-One

```text
100 Students

↓

1 Bus
```

Cheap.

But if the bus breaks,

everyone stops.

---

### One-to-One

```text
100 Students

↓

100 Cars
```

Fast.

But expensive.

---

### Many-to-Many

```text
100 Students

↓

10 Buses
```

Balanced.

---

### Two-Level

```text
100 Students

↓

10 Buses

+

VIP Student

↓

Private Car
```

Best flexibility.

---

# Which Model Do Modern Operating Systems Use?

Modern systems such as **Linux** and **Windows** generally use the **One-to-One** model.

Why?

Because:

* It supports multicore processors well.
* If one thread blocks, others continue.
* Modern computers have enough resources to handle the extra kernel threads.

---

# Exam Questions

### Q1. What is the Many-to-One model?

Many user-level threads are mapped to a single kernel thread.

---

### Q2. Which model provides the best parallelism?

✅ **One-to-One**, because each user thread has its own kernel thread.

---

### Q3. Which model has the lowest overhead?

✅ **Many-to-One**, since only one kernel thread is used.

---

### Q4. Which model is the most flexible?

✅ **Two-Level Model**, because it combines Many-to-Many mapping with the option to bind specific user threads to dedicated kernel threads.

---

# 📝 Lesson 10 Summary

* **Many-to-One**: Fast but poor parallelism.
* **One-to-One**: Best parallelism, but more overhead.
* **Many-to-Many**: A compromise between speed and flexibility.
* **Two-Level**: Like Many-to-Many, but allows certain user threads to be permanently bound to kernel threads.
* Modern operating systems typically use the **One-to-One** model.

---

# 📚 Next Lesson (Lesson 11)

We'll study **Thread Libraries**, including:

* What is a Thread Library?
* POSIX Threads (Pthreads)
* Windows Threads
* Java Threads
* How programmers actually create and manage threads

These are the APIs you'll use when writing multithreaded programs. 
---
---
Awesome bro! 😄 Let's continue.

Today we'll learn **Thread Libraries**.

This lesson is easier than the previous one because now we'll see **how programmers actually create threads**.

---

# 📖 Lesson 11 — Thread Libraries

---

# First, What is a Library?

Before learning **Thread Library**, let's understand what a **library** is.

Suppose you are writing a C++ program.

```cpp
#include <iostream>
using namespace std;

int main()
{
    cout << "Hello";
}
```

Question:

Where did `cout` come from?

Did you write it?

❌ No.

It came from the **iostream library**.

A library is simply a collection of **pre-written code**.

---

## Real-Life Analogy

Imagine you're building a house.

Do you make your own:

* Bricks?
* Doors?
* Windows?

No.

You buy them ready-made.

A **library** is like a shop that provides ready-made tools.

---

# What is a Thread Library?

Now suppose you want to create threads.

Could you write all the low-level OS code yourself?

No.

Instead, programmers use a **Thread Library**.

### Definition

> A **Thread Library** is a collection of functions that allows programmers to create and manage threads easily.

Instead of writing hundreds of lines of OS code,

you simply call a function.

---

# Example

Instead of saying

```text
Create a thread manually...
Allocate stack...
Initialize registers...
...
```

You simply write something like:

```cpp
createThread();
```

The library does the hard work.

---

# Main Thread Libraries

Most Operating Systems textbooks discuss three major thread libraries.

```text
1. POSIX Threads (Pthreads)

2. Windows Threads

3. Java Threads
```

Let's learn each one.

---

# 1. POSIX Threads (Pthreads)

This is the most common thread library in **Linux** and **Unix**.

"Pthread" means

**POSIX Thread**.

POSIX is a standard that Unix-like operating systems follow.

---

## Example

Creating a thread in C using Pthreads:

```cpp
pthread_create();
```

Don't worry about the syntax now.

Just remember:

`pthread_create()`

↓

Creates a new thread.

---

## Other Common Functions

```cpp
pthread_create()
```

Creates a thread.

---

```cpp
pthread_join()
```

Waits until a thread finishes.

---

```cpp
pthread_exit()
```

Ends a thread.

---

### Easy Analogy

Imagine a manager.

```text
Manager

↓

Create Worker

↓

Wait for Worker

↓

Worker Finishes
```

Exactly like

```cpp
pthread_create()

pthread_join()

pthread_exit()
```

---

# 2. Windows Threads

Windows has its own thread library.

Instead of

```cpp
pthread_create()
```

Windows uses

```cpp
CreateThread()
```

Notice

Linux

↓

```cpp
pthread_create()
```

Windows

↓

```cpp
CreateThread()
```

Different names,

same idea.

Both create threads.

---

# Example

Windows program

↓

```cpp
CreateThread()
```

↓

Operating System

↓

New Thread

---

# 3. Java Threads

Java is different.

Java has built-in support for threads.

There are two common ways.

---

## Method 1

Extend the `Thread` class.

Example

```java
class MyThread extends Thread
{
    public void run()
    {
        System.out.println("Hello");
    }
}
```

When you call

```java
start();
```

Java creates a new thread.

---

## Method 2 (Preferred)

Implement the `Runnable` interface.

Example

```java
class MyTask implements Runnable
{
    public void run()
    {
        System.out.println("Hello");
    }
}
```

Again,

calling

```java
start();
```

creates a thread.

---

# Why Do We Need Thread Libraries?

Suppose you want 100 threads.

Without a library,

you would have to:

* Allocate memory
* Create stacks
* Initialize registers
* Set scheduling information
* Handle errors

Very difficult!

Instead,

the library provides simple functions.

Example:

```cpp
pthread_create();
```

Done.

---

# Thread Library vs Operating System

Students often confuse these.

Let's compare.

## Thread Library

Lives in your program.

Provides functions.

Example

```cpp
pthread_create()
```

---

## Operating System

Actually creates and schedules the thread.

Think of it this way:

You order food using an app.

The app

↓

Thread Library

Restaurant

↓

Operating System

The app receives your request,

but the restaurant cooks the food.

---

# Thread Lifecycle

Suppose you call

```cpp
pthread_create();
```

What happens?

Step 1

Program requests a new thread.

↓

Step 2

Thread library processes the request.

↓

Step 3

Operating system creates the thread (or the library manages it, depending on the implementation).

↓

Step 4

Thread begins executing.

---

# Thread Synchronization (Preview)

Suppose

Thread A

and

Thread B

both change

```cpp
balance
```

at the same time.

Problem!

We need synchronization.

Example

```cpp
balance = balance + 100;
```

and

```cpp
balance = balance - 50;
```

If both execute simultaneously,

the final value may be incorrect.

We'll study synchronization in a later chapter.

---

# Comparison

| Library         | Used In      | Thread Creation Function |
| --------------- | ------------ | ------------------------ |
| Pthreads        | Linux / Unix | `pthread_create()`       |
| Windows Threads | Windows      | `CreateThread()`         |
| Java Threads    | Java         | `start()`                |

---

# Memory Trick 🎯

Imagine different countries.

### Bangladesh

Emergency Number

```text
999
```

---

### USA

Emergency Number

```text
911
```

Different numbers.

Same purpose.

Similarly,

Linux

↓

```cpp
pthread_create()
```

Windows

↓

```cpp
CreateThread()
```

Java

↓

```java
start()
```

Different APIs.

Same purpose:

**Create a thread.**

---

# Exam Questions

### Q1. What is a Thread Library?

A collection of functions that helps programmers create and manage threads.

---

### Q2. Name three thread libraries.

* POSIX Threads (Pthreads)
* Windows Threads
* Java Threads

---

### Q3. Which thread library is commonly used in Linux?

✅ **Pthreads**

---

### Q4. Which function creates a thread in Pthreads?

✅ `pthread_create()`

---

### Q5. Which function creates a thread in Windows?

✅ `CreateThread()`

---

### Q6. How are threads commonly started in Java?

By calling the `start()` method on a `Thread` object (or a class implementing `Runnable`).

---

# 📝 Lesson 11 Summary

* A **Thread Library** provides ready-made functions for creating and managing threads.
* The three major thread libraries are:

  * **Pthreads** (Linux/Unix)
  * **Windows Threads**
  * **Java Threads**
* They have different function names but all serve the same purpose: simplifying thread creation and management.
* The library provides the programming interface, while the operating system ultimately executes and schedules threads.

---

# 📚 Next Lesson (Lesson 12)

We'll learn **Multithreading Issues**, including:

* `fork()` and `exec()` with threads
* Signal handling
* Thread cancellation
* Thread-local storage
* Scheduler activations

These are more advanced topics, but I'll explain them from **absolute beginner level** with simple examples. 
---
---
Awesome bro! 🔥 We are now at the last advanced topic of the **Threads** chapter.

Some books skip these topics, but many university exams (including BUET) ask theory questions from them.

---

# 📖 Lesson 12 — Multithreading Issues

This lesson covers several independent concepts.

We'll study them one by one.

---

# Topics

```text
1. fork() and exec()

2. Signal Handling

3. Thread Cancellation

4. Thread-Local Storage (TLS)

5. Scheduler Activations
```

Don't worry.

They're much easier than they sound.

---

# Part 1 — fork() and Threads

Remember Lesson 6?

We learned

```text
fork()
```

creates a new process.

Example

```
Parent Process

↓

fork()

↓

Parent        Child
```

Easy.

---

## Now Imagine

Suppose Chrome has

```text
Chrome Process

│

├── Thread A

├── Thread B

└── Thread C
```

Now Chrome calls

```text
fork()
```

Question:

**What happens to the threads?**

---

There are **two possibilities**.

---

## Option 1

The child gets

ALL threads.

```
Parent

Thread A

Thread B

Thread C



↓

fork()



Child

Thread A

Thread B

Thread C
```

Everything is copied.

---

## Option 2

The child gets

ONLY the thread that called

```text
fork()
```

Suppose

Thread B calls

```text
fork()
```

Then

```
Parent

A

B

C



↓

fork()



Child

B only
```

No Thread A.

No Thread C.

---

## Which One is Used?

Most modern systems (such as POSIX threads on Linux) use:

✅ **Only the calling thread is duplicated in the child process.**

Why?

Imagine Thread A is printing.

Thread B is downloading.

Thread C is editing memory.

Copying all threads may leave shared data in an inconsistent state.

Copying only the calling thread is much safer.

---

# What About exec()?

Remember

```text
exec()
```

replaces the current program.

Suppose

```
Chrome

↓

exec()

↓

Firefox
```

After `exec()`,

everything from Chrome is gone.

Only Firefox exists.

The old threads also disappear.

The new program starts with its own initial thread.

---

# Memory Trick

```
fork()

↓

Copy Process


exec()

↓

Replace Process
```

---

# Part 2 — Signal Handling

First,

what is a signal?

---

Imagine

your phone rings.

```
📞

Incoming Call
```

You immediately stop what you're doing.

Signals work like that.

---

## Definition

A **signal** is a notification sent to a process or thread that a particular event has occurred.

---

## Examples

Keyboard

You press

```
Ctrl + C
```

Program stops.

Why?

Because the OS sent a signal.

---

Another example

Timer finishes.

```
Alarm

↓

Signal
```

---

# Signal Handling in Threads

Suppose

```
Thread A

Thread B

Thread C
```

A signal arrives.

Question:

Which thread receives it?

There are different rules depending on the signal type and OS.

In general:

* Some signals are directed to a **specific thread**.
* Some signals are delivered to the **process**, and the kernel chooses an appropriate thread to handle them.

For beginners, remember:

> Signals notify a process or thread that an event has occurred.

---

# Part 3 — Thread Cancellation

Suppose

Thread A

is downloading

a 10 GB file.

Suddenly,

the user presses

```
Cancel Download
```

The thread must stop.

This is called

# Thread Cancellation

---

## Definition

Stopping a thread before it finishes.

---

There are two methods.

---

## 1. Asynchronous Cancellation

Immediately stop the thread.

```
Running

↓

STOP
```

No warning.

---

Problem?

Suppose

the thread is writing a file.

```
Writing...

↓

STOP
```

The file may become corrupted.

Dangerous.

---

## 2. Deferred Cancellation

Instead of stopping immediately,

the thread checks

```
Should I stop?
```

If yes,

it exits safely.

```
Running

↓

Check

↓

Exit
```

Much safer.

This is the preferred method.

---

# Part 4 — Thread-Local Storage (TLS)

Remember

threads share memory.

```
Process

↓

Shared Memory

↑

Thread A

Thread B
```

Suppose

both threads have

```cpp
count
```

Thread A changes

```cpp
count = 5;
```

Thread B changes

```cpp
count = 100;
```

Problem!

They overwrite each other's values.

---

## Solution

Each thread gets its own private variable.

```
Thread A

count = 5


Thread B

count = 100
```

Now

they don't interfere.

This is called

# Thread-Local Storage

---

### Easy Analogy

Imagine

two students.

Instead of sharing one notebook,

each has

their own notebook.

No confusion.

---

# Part 5 — Scheduler Activations

This is the hardest topic,

but your teacher usually asks only the basic idea.

---

Remember

User Threads

and

Kernel Threads?

Sometimes

the operating system

needs to communicate

with the thread library.

Example

```
OS

↓

Thread Library
```

The OS informs the thread library about important events, such as a thread blocking or becoming runnable, so the library can schedule user threads efficiently.

This cooperation is called

# Scheduler Activations

---

### Easy Analogy

Imagine

Teacher

and

Class Monitor.

Teacher says

```
Ali is absent.

Choose another student.
```

The monitor immediately reorganizes the class.

Similarly,

the OS informs

the thread library,

and

the library adjusts which user thread should run.

---

# Summary Table

| Topic                 | Simple Meaning                                    |
| --------------------- | ------------------------------------------------- |
| `fork()`              | Creates a new process                             |
| `exec()`              | Replaces the current program                      |
| Signal                | Notification from OS                              |
| Thread Cancellation   | Stopping a thread                                 |
| Thread-Local Storage  | Private data for each thread                      |
| Scheduler Activations | Cooperation between the OS and the thread library |

---

# Memory Trick 🎯

Imagine a company.

* **fork()** → Open a new branch office.
* **exec()** → Replace the entire company with a new one.
* **Signal** → Manager sends an urgent announcement.
* **Thread Cancellation** → Fire one employee.
* **TLS** → Every employee has their own locker.
* **Scheduler Activations** → Manager and team leader coordinate work.

---

# Exam Questions

### Q1. What happens after `fork()` in a multithreaded process?

In POSIX systems, the child process contains **only the thread that called `fork()`**.

---

### Q2. What does `exec()` do?

It replaces the current process image with a new program. The old threads are replaced, and the new program starts with its own initial thread.

---

### Q3. What is a signal?

A notification sent by the operating system to inform a process or thread that an event has occurred.

---

### Q4. What is Thread Cancellation?

The process of terminating a thread before it completes execution.

---

### Q5. What is Thread-Local Storage (TLS)?

A mechanism that gives each thread its own private copy of a variable, preventing interference between threads.

---

# 📝 Lesson 12 Summary

* `fork()` creates a new process; in multithreaded POSIX programs, only the calling thread is copied.
* `exec()` replaces the current program and its threads.
* Signals notify processes or threads about events.
* Thread cancellation can be **asynchronous** or **deferred**, with deferred cancellation being safer.
* Thread-Local Storage gives each thread its own private data.
* Scheduler activations allow cooperation between the operating system and the thread library.

---

# 🎉 Congratulations!

If your PDF follows the standard Operating Systems "Processes and Threads" chapter, you've now covered the complete sequence:

1. ✅ Processes
2. ✅ Process States
3. ✅ PCB
4. ✅ Scheduling & Queues
5. ✅ Context Switching
6. ✅ Process Creation & Termination
7. ✅ IPC
8. ✅ Threads
9. ✅ Types of Threads
10. ✅ Multithreading Models
11. ✅ Thread Libraries
12. ✅ Multithreading Issues

This gives you a solid foundation for the next major OS topics, such as **CPU Scheduling Algorithms**, **Synchronization**, and **Deadlocks**. 
