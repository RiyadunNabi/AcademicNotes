Absolutely bro. 👍 That's actually the best way to learn Operating Systems.

We'll treat this like a university class, **lesson by lesson**, not page by page.

# Chapter 3: Processes

We'll divide it into these lessons:

### Lesson 1 — Introduction to Processes

* What is a Program?
* What is a Process?
* Program vs Process
* Why do we need processes?

### Lesson 2 — Process States

* New
* Ready
* Running
* Waiting
* Terminated
* State transition diagram

### Lesson 3 — Process Control Block (PCB)

* What is PCB?
* Why OS needs PCB?
* Every field explained

### Lesson 4 — Process Scheduling

* Ready Queue
* Waiting Queue
* CPU Scheduler

### Lesson 5 — Operations on Processes

* Process Creation
* Process Termination

### Lesson 6 — Interprocess Communication (IPC)

* Why processes communicate
* Shared Memory
* Message Passing

### Lesson 7 — Threads

* What is a thread?
* User vs Kernel threads
* Multithreading

---

# LESSON 1 — Introduction to Processes

We'll start from **zero knowledge**.

---

# Imagine Your Computer

Suppose your laptop is OFF.

You turn it ON.

After Windows/Linux starts, you open:

* Chrome
* VS Code
* Spotify
* Calculator

Question:

**Are these just files?**

No.

They are **running**.

The OS is managing them.

Each running application is called a **Process**.

---

# Step 1 — What is a Program?

A **program** is simply a collection of instructions stored in a file.

Example

```
chrome.exe
```

or

```
calculator.exe
```

or

```
code.exe
```

These are just files stored on your SSD/HDD.

Nothing is happening.

They are sleeping.

Think of them like a movie stored on your computer.

The movie exists.

But until you press Play...

Nothing happens.

---

## Another Example

Suppose you wrote this C++ code.

```cpp
#include <iostream>
using namespace std;

int main()
{
    cout << "Hello";
}
```

After compiling:

```
hello.exe
```

This **hello.exe** is a **program**.

It is just sitting inside your computer.

No CPU.

No RAM.

No execution.

---

# Important Definition

> **Program = A passive collection of instructions stored on disk.**

Passive means

"It is not doing anything."

---

# Step 2 — What is a Process?

Now you double-click

```
hello.exe
```

Immediately,

Windows/Linux loads it into RAM.

CPU starts executing instructions.

Now

```
hello.exe
```

becomes

```
Running hello.exe
```

This is called a **Process**.

So,

> **A process is a program that is currently executing.**

This is one of the most important definitions in Operating Systems.

---

# Visual

```
Program (Disk)

↓

Double Click

↓

Loaded into RAM

↓

CPU Executes

↓

Process
```

---

# Example

Before opening Chrome

```
chrome.exe
```

Stored on disk.

Nothing is happening.

---

After opening Chrome

```
chrome.exe

↓

Running

↓

Chrome Process
```

Now it has

* Memory
* CPU time
* Variables
* Open files
* Browser tabs
* Program counter
* Stack

Now it is called a **process**.

---

# Real-Life Analogy

Imagine a recipe book.

The recipe book contains instructions.

But nobody is cooking.

```
Recipe Book

↓

Program
```

Now your mother starts cooking.

She follows every instruction.

```
Cooking

↓

Process
```

When cooking finishes,

the process ends.

---

# Another Analogy

Blueprint

↓

Program

Workers building a house

↓

Process

Finished house

↓

Process terminated

---

# Program vs Process

| Program        | Process         |
| -------------- | --------------- |
| Stored on disk | Loaded into RAM |
| Passive        | Active          |
| Doesn't run    | Runs            |
| No CPU         | Uses CPU        |
| Static         | Dynamic         |

A simple way to remember:

* **Program = Recipe**
* **Process = Cooking**

---

# Why Do We Need Processes?

Imagine there were **no processes**.

You open Chrome.

The OS doesn't know:

* where Chrome's memory is,
* what it's doing,
* which files it's using,
* whether it's running or waiting.

That would be chaos.

So the OS creates a **process** for every running program.

A process gives the OS all the information it needs to manage that running program.

---

# Can One Program Create Multiple Processes?

Yes!

Example:

You open Calculator.

```
Calculator

↓

1 Process
```

Now open another Calculator.

```
Calculator

↓

Process 1

Process 2
```

Both came from the **same program**, but they are **different processes**.

Each has its own memory, variables, and execution.

---

# Example from Your Own Experience

Suppose you open:

* Firefox
* VS Code
* Terminal
* Discord

The OS creates something like:

```
Firefox Process

VS Code Process

Terminal Process

Discord Process
```

The CPU quickly switches between them, making it look like everything runs at the same time.

---

# Key Points to Remember

✅ A **program** is a file stored on disk.

✅ When a program starts running, it becomes a **process**.

✅ A process uses **CPU**, **RAM**, and other system resources.

✅ The operating system manages **processes**, not just program files.

---

## Quick Quiz 🧠

**Q1.** What is a program?

**Q2.** What is a process?

**Q3.** Which one is active: program or process?

**Q4.** Can one program create multiple processes?

**Q5.** Why does the operating system create processes?

---

In **Lesson 2**, we'll learn about the **Life Cycle of a Process (Process States)**—how a process moves through **New → Ready → Running → Waiting → Terminated**, with easy animations and examples.
---
---
Awesome! Let's continue.

# **Lesson 2 — Process States (Life Cycle of a Process)**

This is one of the **most important topics** in Operating Systems. Almost every OS exam asks about it.

---

# What is a Process State?

A process is **not always running**.

Think about yourself.

Are you always studying?

No.

Sometimes you are:

* Sleeping 😴
* Eating 🍔
* Waiting for a friend 🚶
* Studying 📚

Similarly, a process also changes its state.

The operating system keeps track of its current state. 

---

# The 5 Basic States

```
        +------+
        | New  |
        +------+
            |
            v
        +-------+
        | Ready |
        +-------+
            |
            v
       +---------+
       | Running |
       +---------+
        /       \
       /         \
      v           v
+----------+   +-----------+
| Waiting  |   |Terminated |
+----------+   +-----------+
      |
      |
      v
   +-------+
   | Ready |
   +-------+
```

There are **5 states**:

1. New
2. Ready
3. Running
4. Waiting (Blocked)
5. Terminated

We'll learn each one.

---

# State 1 — New

Imagine you double-click Chrome.

```
Chrome.exe
```

Windows receives your request.

The operating system starts creating a process.

At this moment,

the process is in the **New** state.

### What happens here?

The OS:

* creates a Process ID (PID)
* allocates memory
* creates a PCB (we'll learn PCB later)
* prepares the process

The process is **being created**.

It has **not started running yet**.

---

## Real-Life Example

Suppose you apply for BUET admission.

You submit your application.

Your application exists,

but classes haven't started.

That is like the **New** state.

---

# State 2 — Ready

After creation,

the process moves to **Ready**.

Now the process has everything it needs.

It only needs one thing:

**CPU**.

Imagine there are 20 students waiting outside a teacher's room.

Everyone is ready.

But only one student can enter.

The students waiting outside are like the **Ready Queue**.

---

## Example

Suppose you open:

* Chrome
* VS Code
* Spotify

CPU can execute only one process at a time (assuming one CPU core).

The others wait.

```
Ready Queue

Chrome

VS Code

Spotify
```

All are ready.

None is running.

---

# State 3 — Running

The CPU chooses one process.

Now it starts executing.

This state is called **Running**.

Example:

```
CPU

↓

Chrome
```

Chrome is using the CPU.

It is executing instructions.

---

## Real-Life Example

Students waiting outside:

```
Student A

Student B

Student C
```

Teacher calls Student A.

Student A enters.

Now Student A is **Running**.

The others remain **Ready**.

---

# Important Question

Can many processes be in the Ready state?

✅ Yes.

Can many processes be in the Running state?

For **one CPU core**, ❌ No.

Only one process can use the CPU at one time.

*(On multi-core CPUs, each core can run one process at the same time.)*

---

# State 4 — Waiting (Blocked)

This is the state beginners usually find confusing.

Let's make it simple.

Sometimes a process **cannot continue**, even if it has the CPU.

Why?

Because it is waiting for something.

Examples:

* Waiting for keyboard input
* Waiting for mouse click
* Waiting for a file to load
* Waiting for internet data
* Waiting for disk read

---

## Example

Suppose Chrome wants to download a webpage.

It sends a request.

Now it must wait for the server's response.

During this waiting time,

Chrome **doesn't need the CPU**.

So the OS moves it to the **Waiting** state.

```
Chrome

↓

Waiting for Internet

↓

Waiting State
```

When the webpage arrives,

Chrome becomes **Ready** again.

---

## Real-Life Example

Suppose you're cooking noodles.

You boil water.

Now you wait for 5 minutes.

Are you cooking?

No.

You're waiting.

Similarly, a process waits for events to happen.

---

# State 5 — Terminated

Eventually, the process finishes.

Example:

Calculator.

You close it.

The operating system removes the process.

Its memory is released.

CPU is freed.

Files are closed.

The process enters the **Terminated** state.

---

## Real-Life Example

An exam ends.

Students leave the classroom.

The process is finished.

---

# Complete Journey

Let's see the entire life cycle.

```
1. Double-click Chrome

↓

New

↓

Ready

↓

Running

↓

Waiting
(Downloading webpage)

↓

Ready

↓

Running

↓

Terminated
(User closes Chrome)
```

This cycle can happen many times.

A process may go between **Ready**, **Running**, and **Waiting** repeatedly before it finally terminates.

---

# Why Does a Running Process Go Back to Ready?

Suppose Chrome has been running for a while.

The OS wants to give VS Code a chance to run.

So after Chrome uses its allotted CPU time (called a **time slice**),

the OS interrupts it.

Chrome goes back to **Ready**.

VS Code starts **Running**.

This is called **CPU scheduling**.

We'll study it in detail later.

---

# State Transition Summary

| From    | To         | Why?                           |
| ------- | ---------- | ------------------------------ |
| New     | Ready      | Process creation finished      |
| Ready   | Running    | CPU assigned                   |
| Running | Waiting    | Waiting for I/O or an event    |
| Waiting | Ready      | Event completed                |
| Running | Ready      | CPU time expired (preemption)  |
| Running | Terminated | Process finished or was killed |

---

# Memory Trick 🎯

Think of a student:

* **New** → Admission completed.
* **Ready** → Waiting outside the classroom.
* **Running** → Attending the class.
* **Waiting** → Went to the library to get a book.
* **Ready** → Returned and waiting to enter again.
* **Terminated** → Graduated.

---

# Exam Questions

### Q1. What is the difference between Ready and Waiting?

**Ready:**

* Waiting only for the CPU.
* Can run as soon as the CPU is free.

**Waiting (Blocked):**

* Waiting for an external event (like disk, keyboard, network, etc.).
* Even if the CPU is free, it **cannot** run until the event finishes.

---

### Q2. Can a process go directly from Waiting to Running?

❌ No.

It must first return to **Ready**.

```
Waiting

↓

Ready

↓

Running
```

---

### Q3. Can a process go directly from New to Running?

❌ No.

It first enters the **Ready** state.

```
New

↓

Ready

↓

Running
```

---

# Lesson 2 Summary

* **New:** The process is being created.
* **Ready:** The process is prepared and waiting for the CPU.
* **Running:** The CPU is executing the process.
* **Waiting (Blocked):** The process is waiting for an event or I/O.
* **Terminated:** The process has finished execution.

Understanding these five states makes the rest of the Processes chapter much easier. In **Lesson 3**, we'll learn about the **Process Control Block (PCB)**—the data structure the operating system uses to remember everything about each process.
---
---
Great! Now we reach one of the **most important concepts** in Operating Systems.

---

# 📖 Lesson 3 — Process Control Block (PCB)

> **This is the OS's notebook about a process.**

If you understand this lesson, you'll never forget PCB.

---

# First, Let's Think...

Suppose you open three programs:

```
Chrome
VS Code
Spotify
```

Now imagine the CPU is running **Chrome**.

After **10 milliseconds**, the operating system stops Chrome and gives the CPU to VS Code.

Question:

👉 **When Chrome runs again later, how does it know where to continue?**

Does it start from the beginning?

```
No!
```

Imagine watching a movie.

You stop at **1 hour 15 minutes**.

Tomorrow you continue.

How does Netflix know where you stopped?

Because it **saved your progress**.

The operating system does the same thing.

It saves the process's progress.

Where?

Inside the **PCB**.

---

# What is PCB?

PCB stands for

> **Process Control Block**

It is a **data structure** maintained by the operating system for **every process**.

---

## Simple Definition

> **PCB is a record that stores all information about a process.**

Think of it as the **identity card** of a process.

Every process has **one PCB**.

---

# Real-Life Analogy

Imagine a school.

Every student has a file.

```
Student File

Name

Roll

Department

CGPA

Attendance

Semester
```

The school uses this file to manage students.

Similarly,

```
PCB

PID

State

Memory

CPU Registers

Program Counter

Files

Scheduling Info
```

The OS uses the PCB to manage processes.

---

# Why Do We Need PCB?

Imagine there is **no PCB**.

The OS would not know:

* Which process is running.
* Where the process stopped.
* Which memory belongs to it.
* Which files it opened.
* Which CPU registers belong to it.

Everything would become mixed up.

So every process gets its own PCB.

---

# Where is PCB Stored?

Many beginners ask this.

PCB is stored **inside the operating system's memory (Kernel Space)**.

Not inside your program.

```
RAM

+------------------------+
| Operating System       |
|                        |
|  PCB of Chrome         |
|  PCB of VS Code        |
|  PCB of Spotify        |
+------------------------+

+------------------------+
| User Programs          |
| Chrome                 |
| VS Code                |
| Spotify                |
+------------------------+
```

The OS owns the PCB.

Applications cannot directly change it.

---

# What's Inside a PCB?

A PCB stores many pieces of information.

Let's learn them one by one.

---

# 1. Process ID (PID)

Every process has a unique number.

Example

```
Chrome

PID = 2310
```

```
VS Code

PID = 4532
```

```
Spotify

PID = 1872
```

The operating system uses the PID to identify processes.

Think of it like your student ID.

You may have the same name as another person,

but your student ID is unique.

---

# 2. Process State

Remember Lesson 2?

```
New

Ready

Running

Waiting

Terminated
```

The PCB stores the current state.

Example

```
Chrome

State = Running
```

Later,

```
State = Waiting
```

The OS updates it whenever the state changes.

---

# 3. Program Counter (PC)

This is one of the **most important fields**.

Suppose your program is

```cpp
cout<<"A";

cout<<"B";

cout<<"C";
```

The CPU has already executed

```cpp
cout<<"A";
```

Now it is about to execute

```cpp
cout<<"B";
```

The Program Counter stores the address of the **next instruction**.

Example

```
Program

Line 1

Line 2  ← Next

Line 3
```

If the process stops now,

later it resumes from Line 2,

not from Line 1.

---

## Analogy

Bookmark in a book.

You stop reading page 50.

The bookmark remembers your position.

Program Counter is that bookmark.

---

# 4. CPU Registers

CPU registers are very small, very fast memory inside the CPU.

Example:

```
AX

BX

CX

DX
```

(Names vary depending on the CPU architecture.)

Suppose Chrome is running.

The CPU registers contain Chrome's values.

Now the OS switches to VS Code.

Before switching,

it saves Chrome's register values into Chrome's PCB.

Later,

when Chrome runs again,

the OS restores those values back into the CPU.

This is how the process continues exactly where it left off.

---

# 5. Memory Information

Every process has its own memory.

The PCB stores information about that memory.

Example

```
Code

Stack

Heap

Global Variables
```

The OS uses this to know which memory belongs to which process.

Without this,

Chrome could accidentally overwrite VS Code's memory!

---

# 6. CPU Scheduling Information

Suppose three processes exist.

```
Chrome

VS Code

Spotify
```

Which one should run next?

The scheduler decides.

For that,

the PCB stores scheduling information such as:

* Priority
* Queue position
* Time slice
* Scheduling parameters

We'll study scheduling in the next lesson.

---

# 7. I/O Status Information

Suppose Chrome opens

```
photo.jpg

music.mp3

report.pdf
```

The PCB keeps track of:

* Open files
* Printers
* USB devices
* Network connections

This helps the OS manage input/output correctly.

---

# Complete PCB Diagram

```
+---------------------------+
| Process Control Block     |
+---------------------------+
| Process ID (PID)          |
| Process State             |
| Program Counter           |
| CPU Registers             |
| Memory Information        |
| Scheduling Information    |
| I/O Status                |
+---------------------------+
```

You don't need to memorize the order, but you should know **what each field is used for**.

---

# Example: CPU Switching (Context Switch)

Imagine:

```
Chrome is Running.
```

The CPU has:

```
PC = Line 100

Registers = Some values
```

Now the OS wants to run VS Code.

### Step 1

Save Chrome's information into **Chrome's PCB**.

```
Chrome PCB

PC = Line 100

Registers = Saved
```

### Step 2

Load VS Code's information from **VS Code's PCB**.

```
VS Code PCB

PC = Line 250

Registers = Loaded
```

Now VS Code continues from **Line 250**, exactly where it previously stopped.

This saving and restoring process is called a **Context Switch**.

We'll study it in detail later.

---

# Easy Memory Trick

Think of a video game.

When you save the game:

```
Health = 90

Coins = 150

Position = Level 4

Weapon = Sword
```

Later,

you load the save and continue from the same point.

A PCB is like the **save file** for a process.

---

# Exam Questions

### Q1. What is PCB?

**Answer:**
A PCB (Process Control Block) is a data structure maintained by the operating system that stores all important information about a process.

---

### Q2. Why is PCB needed?

Because the operating system uses it to:

* identify a process,
* manage its execution,
* save its state during context switching,
* and resume it later.

---

### Q3. Does every process have a PCB?

✅ **Yes.** Every process has its own PCB.

---

# 📝 Lesson 3 Summary

* **PCB = Process Control Block**
* It is maintained by the **operating system**.
* Every process has **one PCB**.
* It stores:

  * PID
  * Process State
  * Program Counter
  * CPU Registers
  * Memory Information
  * Scheduling Information
  * I/O Status
* During a **context switch**, the OS saves the current process's information into its PCB and restores another process's information from its PCB.

---

➡️ **Lesson 4** will cover **Process Scheduling and Queues**:

* Job Queue
* Ready Queue
* Device Queue (Waiting Queue)
* CPU Scheduler
* How the OS decides which process runs next

These concepts build directly on the PCB and process states you've just learned.
---
---
Awesome! You're doing great. Now we understand:

* ✅ What a Process is
* ✅ Process States
* ✅ PCB

Now let's learn **how the Operating System manages hundreds of processes**.

---

# 📖 Lesson 4 — Process Scheduling & Queues

This lesson answers one big question:

> **If 100 processes are ready, who gets the CPU first?**

The answer is **Process Scheduling**.

---

# First, Let's Imagine...

Suppose you open these applications:

```text
Chrome
VS Code
Spotify
Discord
Calculator
```

Now you have **5 processes**.

But...

You only have **1 CPU core**.

Can the CPU execute all 5 processes at the exact same time?

❌ No.

It can execute **only one process at a time**.

So...

Where do the other four wait?

The answer:

**Queues.**

---

# What is a Queue?

Before learning OS queues, let's understand a normal queue.

Suppose you're at a bank.

```text
Cash Counter

↓

Person A

Person B

Person C

Person D
```

Only one person is being served.

Everyone else waits in line.

A process does exactly the same thing.

---

# Process Scheduling

The Operating System decides

> "Who should use the CPU next?"

This decision-making process is called

# **Process Scheduling**

The component that makes this decision is called the

# **CPU Scheduler**

---

# Real-Life Example

Imagine one teacher.

Five students need help.

```text
Teacher

↓

Student A

Student B

Student C

Student D

Student E
```

Teacher can help only one student at a time.

Teacher chooses one student.

CPU Scheduler works exactly like that teacher.

---

# There are Three Important Queues

Most beginner books talk about three queues.

```text
1. Job Queue

2. Ready Queue

3. Device Queue (Waiting Queue)
```

We'll learn them one by one.

---

# 1. Job Queue

Imagine you turn on your computer.

Many programs exist.

```text
Chrome

VS Code

Spotify

Steam

Discord
```

Some are already running.

Some are waiting to be loaded.

The collection of all processes in the system is called the **Job Queue**.

Think of it as the **master list** of processes.

---

### Analogy

Imagine a hospital.

Everyone who enters the hospital is registered.

Not everyone is currently seeing a doctor.

But everyone is in the hospital's record.

That record is like the **Job Queue**.

---

# 2. Ready Queue ⭐ (Very Important)

This is the most important queue.

A process enters the Ready Queue when:

* It has memory.
* It has all required resources.
* It is **only waiting for the CPU**.

Example

```text
Ready Queue

Chrome

VS Code

Spotify
```

All three are ready.

Only one can run.

---

### Visual

```text
Ready Queue

↓

Chrome

↓

VS Code

↓

Spotify

↓

CPU
```

CPU picks one.

The others continue waiting.

---

# Example

Suppose Chrome is running.

```text
Running

↓

Chrome
```

VS Code is waiting.

```text
Ready Queue

↓

VS Code
```

Spotify is waiting.

```text
Ready Queue

↓

Spotify
```

When Chrome's time finishes,

VS Code moves to Running.

---

# 3. Device Queue (Waiting Queue)

Sometimes a process is **not waiting for the CPU**.

Instead,

it is waiting for an I/O device.

Examples:

* Keyboard
* Mouse
* Printer
* Disk
* Network

---

Suppose Chrome wants to download a webpage.

```text
Chrome

↓

Waiting for Internet
```

Chrome goes to the Device Queue.

Another example:

```text
Word

↓

Printing Document
```

Word waits for the printer.

---

### Visual

```text
Printer Queue

↓

Word

↓

PDF Reader

↓

Excel
```

All are waiting for the printer.

---

# Difference Between Ready Queue and Device Queue

This question is very common in exams.

| Ready Queue                             | Device Queue                      |
| --------------------------------------- | --------------------------------- |
| Waiting for CPU                         | Waiting for I/O                   |
| Can run immediately if CPU becomes free | Cannot run until the I/O finishes |
| State = Ready                           | State = Waiting (Blocked)         |

---

# How Scheduling Works

Let's watch a complete example.

Suppose three processes exist.

```text
Chrome

VS Code

Spotify
```

Initially,

```text
Ready Queue

Chrome

VS Code

Spotify
```

---

## Step 1

CPU Scheduler selects Chrome.

```text
CPU

↓

Chrome
```

Chrome becomes **Running**.

---

## Step 2

Chrome starts downloading a webpage.

Now Chrome needs the Internet.

```text
Chrome

↓

Waiting
```

Chrome leaves the CPU.

---

## Step 3

CPU becomes free.

Scheduler chooses VS Code.

```text
CPU

↓

VS Code
```

---

## Step 4

Internet data arrives.

Chrome is no longer waiting.

It returns to

```text
Ready Queue
```

Notice:

It **does not** go directly to Running.

It must wait its turn again.

---

# Complete Flow

```text
New

↓

Ready Queue

↓

Running

↓

Waiting (Device Queue)

↓

Ready Queue

↓

Running

↓

Terminated
```

This cycle may repeat many times before the process finishes.

---

# CPU Scheduler

The CPU Scheduler is part of the Operating System.

Its job is:

* choose the next process,
* remove it from the Ready Queue,
* give it the CPU.

Simple diagram:

```text
Ready Queue

↓

CPU Scheduler

↓

CPU

↓

Running Process
```

---

# Dispatcher

Many books also mention a **Dispatcher**.

Students often confuse it with the Scheduler.

### CPU Scheduler

Decides **who** should run.

### Dispatcher

Actually gives the CPU to that process.

Think of it like this:

Teacher:

> "Student A, come in."

That decision is the Scheduler.

Opening the door and seating Student A is the Dispatcher's job.

---

# Example

Suppose:

```text
Ready Queue

Chrome

VS Code

Spotify
```

Scheduler says:

```text
Run VS Code.
```

Dispatcher performs the switch.

Now

```text
Running

↓

VS Code
```

---

# Why Scheduling is Needed

Imagine there were no scheduler.

All processes would try to use the CPU at the same time.

Chaos!

Scheduling ensures:

* fairness,
* efficient CPU usage,
* fast response,
* smooth multitasking.

---

# Exam Questions

### Q1. What is Process Scheduling?

**Answer:**
Process Scheduling is the mechanism by which the operating system selects one process from the Ready Queue and allocates the CPU to it.

---

### Q2. What is the Ready Queue?

A queue containing processes that are ready to execute and are waiting only for CPU time.

---

### Q3. What is the Device Queue?

A queue containing processes waiting for I/O operations or other events.

---

### Q4. What is the difference between Scheduler and Dispatcher?

| Scheduler                | Dispatcher                          |
| ------------------------ | ----------------------------------- |
| Chooses the next process | Gives the CPU to the chosen process |
| Makes the decision       | Performs the context switch         |

---

# 📝 Lesson 4 Summary

* The CPU can run only one process at a time (per core).
* The **CPU Scheduler** chooses which ready process gets the CPU.
* The **Ready Queue** contains processes waiting only for CPU time.
* The **Device Queue (Waiting Queue)** contains processes waiting for I/O or other events.
* The **Dispatcher** performs the actual switch from one process to another.
* After an I/O operation completes, a process moves from the **Waiting** state back to the **Ready** state, not directly to **Running**.

---

➡️ **Next: Lesson 5 — Context Switching.**

This is one of the most important OS concepts. You'll learn:

* What a **context switch** is,
* Why it's necessary,
* What exactly gets saved and restored,
* Why context switching has an overhead,
* A complete step-by-step example with PCB, CPU registers, and the Program Counter.
---
---
Awesome! 🔥 Now we'll learn one of the **most important concepts** in Operating Systems.

If you understand this lesson, you'll understand **how multitasking actually works**.

---

# 📖 Lesson 5 — Context Switching

---

# First, Imagine This Situation

Suppose you have opened:

```text
Chrome
VS Code
Spotify
```

Only **Chrome** is running.

```text
CPU
 ↓
Chrome
```

Suddenly, the operating system decides:

> "Now VS Code should run."

Question:

👉 **How can Chrome stop and later continue from exactly the same place?**

Does it restart from the beginning?

```text
No!
```

Then how?

The answer is

# **Context Switching**

---

# What is Context?

Before learning Context Switching,

we need to know what **Context** means.

Context means

> **Everything the CPU needs to continue a process exactly where it stopped.**

Imagine you're writing an assignment.

```text
Page 15

Paragraph 3

Sentence 2
```

Your friend calls you.

You stop writing.

Later you continue from

```text
Page 15

Paragraph 3

Sentence 2
```

How?

Because you remembered where you stopped.

That information is your **context**.

---

# What Does Context Include?

For a process,

Context includes:

* Program Counter (next instruction)
* CPU Registers
* Process State
* Stack Pointer
* CPU Flags
* Scheduling Information

Most of these are stored in the **PCB**.

---

# What is Context Switching?

## Definition

> **Context Switching is the process of saving the current process's context and loading another process's context so the CPU can switch between processes.**

Don't worry if it sounds difficult.

Let's learn step by step.

---

# Example

Suppose:

```text
CPU

↓

Chrome
```

Chrome is executing.

Its PCB contains

```text
Program Counter = Line 250

Registers

Stack Pointer

State = Running
```

Now the OS wants VS Code to run.

What happens?

---

# Step 1 — Stop Chrome

The OS interrupts Chrome.

Chrome stops executing.

```text
CPU

↓

Stopped
```

---

# Step 2 — Save Chrome's Context

The OS saves Chrome's information into Chrome's PCB.

```text
Chrome PCB

Program Counter = Line 250

Registers = Saved

Stack Pointer = Saved

State = Ready
```

Now Chrome can safely sleep.

---

# Step 3 — Select Another Process

Scheduler chooses

```text
VS Code
```

---

# Step 4 — Load VS Code's Context

VS Code already has a PCB.

```text
VS Code PCB

Program Counter = Line 840

Registers

Stack Pointer
```

The OS copies these values into the CPU.

---

# Step 5 — Continue VS Code

Now

```text
CPU

↓

VS Code
```

VS Code resumes from

```text
Line 840
```

It does **NOT** start from Line 1.

---

# Entire Process

```text
Chrome Running

↓

Save Chrome Context

↓

Load VS Code Context

↓

VS Code Running
```

This entire operation is called

# Context Switch

---

# Visual Diagram

```text
Before

CPU

↓

Chrome



↓

Save Context



Chrome PCB



↓

Load Context



VS Code PCB



↓

CPU

↓

VS Code
```

---

# Where is Context Stored?

Remember Lesson 3?

Each process has a PCB.

```text
PCB

Program Counter

Registers

Stack Pointer

State
```

The operating system saves the context into the PCB.

---

# Real-Life Analogy

Imagine three students sharing one desk.

Student A is writing.

Teacher says

> Stop!

Student A writes

```text
Page 40

Question 5

Line 8
```

inside his notebook.

Now Student B starts writing.

Later,

Student A returns,

opens his notebook,

and continues from

```text
Page 40

Question 5

Line 8
```

Exactly.

That notebook is like the **PCB**.

---

# Another Example

Suppose Chrome is downloading.

```text
Running

↓

Waiting for Internet
```

The CPU should not remain idle.

The OS performs a context switch.

Now

```text
CPU

↓

Spotify
```

Spotify runs while Chrome waits.

When the Internet finishes,

Chrome returns to the Ready Queue.

Later,

another context switch happens.

Chrome resumes.

---

# Why Context Switching is Needed

Without context switching,

only one program could run until it finished.

Imagine this:

```text
Open Chrome

↓

Wait until Chrome closes

↓

Open VS Code

↓

Wait until VS Code closes
```

That would be terrible.

Instead,

your computer rapidly switches between processes.

It feels like everything runs at the same time.

---

# Does Context Switching Take Time?

Yes.

This is called

# Context Switch Overhead

---

Why?

Because the OS has to

* stop one process,
* save registers,
* save the Program Counter,
* load another PCB,
* restore registers,
* restart the CPU.

All this takes time.

During this time,

the CPU is **not doing useful work** for user programs.

---

# Example

Suppose

```text
Chrome runs

10 ms
```

Context switch

```text
0.2 ms
```

VS Code runs

```text
10 ms
```

Context switch

```text
0.2 ms
```

These **0.2 ms** are overhead.

Too many context switches reduce performance.

---

# Important Point

A context switch **does not execute your program**.

It only prepares the CPU to execute another process.

So,

Context Switching is considered **overhead**.

---

# When Does Context Switching Happen?

Several situations can trigger it:

### 1. Time Slice Ends

Example:

Chrome uses its allotted CPU time.

The scheduler switches to VS Code.

---

### 2. Process Waits for I/O

Chrome starts downloading.

It cannot continue.

The OS switches to another process.

---

### 3. Higher Priority Process Arrives

Suppose a high-priority antivirus process becomes ready.

The OS may stop Chrome and run the antivirus immediately.

---

### 4. Process Finishes

Calculator exits.

The scheduler selects another process.

---

# Context Switch vs Process Switch

Students often ask this.

Usually,

these terms are used interchangeably in introductory OS courses.

A context switch changes the CPU from one execution context to another (often from one process to another).

---

# Complete Example

Suppose we have:

```text
Chrome
VS Code
Spotify
```

### Initial

```text
Running

↓

Chrome
```

### Time slice expires

```text
Save Chrome PCB
```

### Scheduler chooses

```text
VS Code
```

### Restore VS Code PCB

```text
Program Counter = 850

Registers = Loaded
```

### CPU

```text
Running

↓

VS Code
```

Later,

VS Code waits for disk I/O.

```text
Waiting
```

Scheduler chooses Spotify.

Another context switch happens.

---

# Memory Trick 🎯

Imagine Netflix.

You're watching a movie.

You pause at

```text
1 hour 22 minutes
```

Netflix remembers your position.

Tomorrow,

you continue from

```text
1 hour 22 minutes
```

The saved position is like the **context**.

Loading it again is like a **context switch**.

---

# Exam Questions

### Q1. What is Context Switching?

**Answer:**
Context switching is the process of saving the current process's context into its PCB and loading another process's context so the CPU can execute a different process.

---

### Q2. Why is Context Switching needed?

* To allow multitasking.
* To share the CPU among multiple processes.
* To switch when a process waits for I/O or its time slice expires.

---

### Q3. What information is saved during a Context Switch?

Typically:

* Program Counter (PC)
* CPU Registers
* Stack Pointer
* Process State
* Other CPU-related information stored in the PCB

---

### Q4. Why is Context Switching called overhead?

Because the CPU spends time saving and restoring process information instead of executing user programs.

---

# 📝 Lesson 5 Summary

* **Context** = the information needed to resume a process.
* **Context Switching** = saving one process's context and loading another's.
* The **PCB** stores the saved context.
* Context switching enables **multitasking**.
* It introduces **overhead**, since the CPU spends time switching instead of executing user code.

---

## 📚 What's Next?

**Lesson 6 — Operations on Processes**

We'll learn:

* How a process is **created**
* How a process is **terminated**
* Parent and child processes
* `fork()`, `exec()`, and `wait()` (very important in Linux/Unix)
* Zombies and orphan processes (explained from scratch with examples)

These are fundamental concepts you'll use in both Operating Systems and systems programming.

---
---
Awesome! 🔥 Now we're entering one of the most practical topics in Operating Systems.

Until now, we've learned:

* ✅ What a process is
* ✅ Process states
* ✅ PCB
* ✅ Scheduling
* ✅ Context Switching

Now the question is:

> **How does a process start?**
>
> **How does it end?**

That's what we'll learn today.

---

# 📖 Lesson 6 — Process Operations (Creation & Termination)

---

# Part 1 — Process Creation

## Imagine This

You turn on your computer.

What is the first process?

Is it Chrome?

No.

Is it VS Code?

No.

The **Operating System** creates the very first process.

This is called the **initial process** (or init/system process, depending on the OS).

After that, this process creates other processes.

---

# Real-Life Analogy

Imagine a family.

👨 Father

↓

👦 Son

↓

👧 Daughter

↓

👶 Grandchild

Each child comes from a parent.

Processes work the same way.

---

# Parent and Child Process

When one process creates another process:

* The original process = **Parent Process**
* The new process = **Child Process**

Example:

```text
Terminal

↓

Runs Chrome

↓

Chrome Process
```

Here,

```text
Terminal
```

is the **Parent Process**.

```text
Chrome
```

is the **Child Process**.

---

# Another Example

Suppose you open VS Code.

Inside VS Code you press

```text
Run C++ Program
```

Now VS Code creates another process.

```text
VS Code

↓

g++

↓

a.exe
```

So

```text
VS Code
```

is the parent,

and

```text
a.exe
```

is the child.

---

# Process Tree

Processes often form a tree.

Example:

```text
System

│

├── Chrome

│      ├── Tab 1

│      ├── Tab 2

│      └── Tab 3

│

├── VS Code

│      ├── Terminal

│      └── Compiler

│

└── Spotify
```

Notice:

One process can create many child processes.

---

# What Happens During Process Creation?

Suppose you double-click Chrome.

The operating system performs several steps.

---

## Step 1

Create a new PCB.

```text
Chrome PCB
```

---

## Step 2

Assign a unique PID.

Example

```text
PID = 4215
```

---

## Step 3

Allocate memory.

The OS gives memory for:

* Program Code
* Stack
* Heap
* Global Variables

---

## Step 4

Load the executable into memory.

```text
chrome.exe

↓

RAM
```

---

## Step 5

Set the Program Counter.

Initially,

it points to the first instruction.

---

## Step 6

Put the process into the Ready Queue.

Now the process waits for the CPU.

---

# Complete Creation Flow

```text
Double Click

↓

OS Creates PCB

↓

Assign PID

↓

Allocate Memory

↓

Load Program

↓

Ready Queue

↓

Running
```

---

# Parent and Child Relationship

The child process may receive resources from the parent.

Example:

Parent

```text
Terminal
```

Child

```text
Python Program
```

The child may inherit:

* Environment variables
* Current directory
* Open files
* User permissions

Think of it like a child inheriting some things from their parents.

---

# Process Termination

Every process eventually ends.

This is called

# Process Termination

---

# When Does a Process Terminate?

There are several reasons.

---

## 1. Normal Completion

The program finishes successfully.

Example

```cpp
int main()
{
    cout<<"Hello";
}
```

After printing

```text
Hello
```

the program ends.

Normal termination.

---

## 2. User Stops It

Example

You close Chrome.

```text
X

↓

Chrome Closed
```

The operating system terminates the process.

---

## 3. Fatal Error

Suppose a program crashes.

Example:

```cpp
int* p = nullptr;

*p = 10;
```

This causes an invalid memory access.

The operating system terminates the process.

---

## 4. Parent Terminates Child

Sometimes a parent process tells the OS:

```text
Stop my child.
```

The child process is terminated.

---

# What Happens During Termination?

Suppose Chrome exits.

The OS performs several tasks.

---

## Step 1

Close open files.

Example

```text
photo.jpg

music.mp3
```

are closed.

---

## Step 2

Release memory.

The RAM used by Chrome is freed.

---

## Step 3

Release I/O devices.

Printer

USB

Network

All resources are released.

---

## Step 4

Delete PCB.

The process no longer exists.

---

# Complete Termination Flow

```text
Running

↓

Finish

↓

Close Files

↓

Free Memory

↓

Delete PCB

↓

Terminated
```

---

# Important Concept — `fork()`

In Linux/Unix, a process usually creates another process using the **`fork()`** system call.

Imagine:

```text
Parent Process

↓

fork()

↓

Parent

Child
```

Now there are **two processes**.

At first, they are almost identical.

Each has:

* its own PCB,
* its own PID,
* its own memory space.

They continue executing independently.

---

# Example

Suppose Process A calls `fork()`.

Before:

```text
Process A
```

After:

```text
Process A

↓

fork()

↓

Parent

Child
```

Now the scheduler can run either one.

---

# What is `exec()`?

Suppose the child process wants to run another program.

It uses **`exec()`**.

Example:

```text
Terminal

↓

fork()

↓

Child

↓

exec()

↓

Chrome
```

`exec()` replaces the current program with a new one.

Think of it like changing the script an actor is performing while keeping the same stage.

---

# What is `wait()`?

Suppose a parent creates a child.

Sometimes the parent wants to wait until the child finishes.

It uses **`wait()`**.

Example:

```text
Parent

↓

Creates Child

↓

wait()

↓

Child Finishes

↓

Parent Continues
```

This helps synchronize parent and child.

---

# Quick Summary of Linux System Calls

| System Call | Purpose                                  |
| ----------- | ---------------------------------------- |
| `fork()`    | Create a child process                   |
| `exec()`    | Replace the current program with another |
| `wait()`    | Wait for a child process to finish       |

---

# Memory Trick 🎯

Think of a restaurant.

Manager (Parent)

↓

Hires a Worker (Child)

↓

Worker does the job

↓

Worker leaves (Termination)

The manager continues working.

---

# Exam Questions

### Q1. What is a parent process?

A process that creates another process.

---

### Q2. What is a child process?

A process created by another process.

---

### Q3. What happens during process creation?

* PCB is created.
* PID is assigned.
* Memory is allocated.
* Program is loaded.
* Process enters the Ready Queue.

---

### Q4. What happens during process termination?

* Open files are closed.
* Memory is released.
* Resources are freed.
* PCB is deleted.

---

### Q5. What do `fork()`, `exec()`, and `wait()` do?

* `fork()` → Creates a child process.
* `exec()` → Replaces the current program with a new one.
* `wait()` → Makes the parent wait for its child to finish.

---

# 📝 Lesson 6 Summary

* A **parent process** can create one or more **child processes**.
* During **process creation**, the OS creates a PCB, assigns a PID, allocates memory, and places the process in the Ready Queue.
* During **process termination**, the OS closes files, frees memory, releases resources, and removes the PCB.
* In Linux/Unix:

  * `fork()` creates a child process.
  * `exec()` loads a new program into a process.
  * `wait()` lets a parent wait for its child to finish.

---

## ➡️ Next: Lesson 7 — Interprocess Communication (IPC)

This is where you'll learn **how two different processes communicate** with each other using:

* Shared Memory
* Message Passing
* Pipes
* Producer–Consumer examples

It's one of the most interesting and commonly tested topics in Operating Systems.
---
---
