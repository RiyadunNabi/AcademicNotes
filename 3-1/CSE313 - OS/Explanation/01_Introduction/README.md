# CSE313 — Operating Systems, Lecture 1: Introduction
### Comprehensive study notes (slide + Tanenbaum Ch.1, explained from zero)

---

## 0. Why this document exists

Your teacher's slide deck is a *summary* — a set of pointers to remember. The book (Tanenbaum, *Modern Operating Systems*) has the full explanation, but Chapter 1 alone is ~85 pages, and you don't have time to read all of it right now. This document takes every bullet point on the slide, expands it with intuition, analogies, and just enough book-level depth so you actually *understand* it instead of memorizing it — assuming zero prior OS knowledge. Where useful, I've added a bit of extra context beyond the slide that will help later chapters make sense.

---

## 1. The problem an OS solves

### 1.1 A modern computer is a mess of hardware

Look at the slide's "Modern Computer System" diagram: CPU, disk controller, USB controller, graphics adapter, memory — all wired together, each with its own quirks, its own way of being "talked to," its own timing behavior.

Every running program needs some mix of these:
- **CPU time** — to actually execute its instructions
- **Memory** — to hold its code and data while running
- **I/O devices** — disk, keyboard, mouse, network, printer, etc.

If every application programmer had to know the exact electrical/register-level protocol for talking to a SATA hard disk, a USB controller, a GPU — **nobody would ever finish writing an application**. It's not just hard, it's a different kind of hard for every single device model.

**Analogy:** Imagine if, to make a phone call, you had to personally understand how radio towers route signals, how the telecom switching network works, and how to physically modulate an electromagnetic wave. Instead, you press a button labeled "Call." Someone built a layer between you and that complexity. That's what an OS is to a programmer.

### 1.2 Two jobs, one piece of software

The book (and slide) converge on this: an operating system exists to do two related but distinct jobs:

1. **Understand, manage, and allocate hardware resources efficiently.** ("Resource manager" view — someone has to decide *which* program gets the CPU next, *where* in memory a program's data lives, *which* process gets the disk next.)
2. **Give programmers a clean, simple, abstract machine to program against instead of the messy real hardware.** ("Extended machine" / "abstraction provider" view — you call `read()` to get bytes from a file; you never touch a disk controller register.)

Depending on who's explaining OS concepts, you'll hear more of one framing or the other — they're two views of the same software, not two different systems.

**Result:** Computers ship with a special layer of software — **the Operating System** — sitting between raw hardware and every other program, doing exactly these two jobs.

---

## 2. What is an OS? (Definitions + placement)

### 2.1 Three equivalent-ish definitions (from the slide)

- The program that gets loaded first at boot time and manages every other program afterward.
- System software that manages hardware and software resources and provides common services to programs.
- The fundamental interface between the hardware and the user.

None of these is *the* perfect definition (the book admits OS is genuinely hard to pin down precisely), but together they capture it: **it's the software, running with special privilege, that sits at the base of the software stack and manages everything else.**

### 2.2 Where it sits — "Placement of OS" diagram

```
        User 1   User 2   ...   User n
           |        |             |
        ┌──────────────────────────────┐
        │           Software            │
        │  ┌───────────┐ ┌────────────┐ │
        │  │  System    │ │Application │ │
        │  │  Software  │ │ Software   │ │
        │  └─────┬──────┘ └─────┬──────┘ │
        │        └───────┬──────┘        │
        │          Operating System      │
        └────────────────┬───────────────┘
        ┌────────────────┴───────────────┐
        │            Hardware             │
        │   CPU     RAM     I/O devices   │
        └──────────────────────────────────┘
```

The OS sits **directly on top of the bare hardware**. Everything else — your web browser, your compiler, your music player, even other "system" utilities — sits on top of the OS and relies on it.

### 2.3 "Scope of Interaction" diagram — who talks to whom

```
   Human users
       ↕
Application software   (browser, editor, games...)
       ↕
   Operating system
       ↕
Other system software  (drivers not in kernel, some libraries)
       ↕
     Hardware
```

Notice the double arrows on the *right* side going straight from Human users down to Hardware and Other system software — that just represents that, indirectly, everything a human does eventually reaches the hardware, but it always routes *through* the OS layer for anything that matters (resource access).

### 2.4 An important subtlety: the shell/GUI is NOT the OS

This trips people up. When you see the Windows desktop, or GNOME on your Pop!_OS, or a terminal prompt — that's the **shell** (text-based) or **GUI** (icon-based). It is a **user-mode program** that uses the OS to get things done, but it is not itself part of the OS.

- You can delete your email client and install a different one — no restriction.
- You **cannot** rewrite the low-level clock interrupt handler or memory manager as a regular user program — that's protected, privileged OS code.

This is actually something you're living right now: you switched from COSMIC to GNOME as your desktop environment on the *same* Pop!_OS kernel/OS underneath. The shell/GUI is swappable; the OS core is not (without deep, privileged changes).

---

## 3. Booting the computer

This wasn't emphasized heavily on the slide but is essential background: **how does the OS even get into memory in the first place, given that the OS is the thing that's supposed to be managing memory?** It's a bit of a chicken-and-egg problem, solved by hardware.

**Step by step:**

1. **Power on.** The CPU is hardwired to start executing instructions from a fixed address that lives in a small, non-volatile chip on the motherboard called **ROM/Flash** — containing the **BIOS** (Basic Input/Output System), or on modern machines, **UEFI** (same idea, more modern).
2. **BIOS runs diagnostics** ("POST" — Power-On Self Test): checks how much RAM is installed, checks that keyboard and basic devices respond. This is why you sometimes see a black screen with hardware info flash by before your OS logo appears.
3. **BIOS determines the boot device** — tries a list of devices (hard drive, USB, CD-ROM/DVD) stored in a tiny battery-backed memory called **CMOS**. (Fun distinction: *BIOS* is the program; *CMOS* is where BIOS stores settings like boot order, date/time.)
4. **First sector of the boot device is loaded into RAM and executed.** This tiny program reads the partition table to find which partition is "active," then loads a **secondary boot loader** from that partition (on Linux systems, this is commonly **GRUB** — you've probably seen the GRUB menu on your dual-boot HP Victus when choosing between Pop!_OS and Windows!).
5. **The boot loader loads the actual OS kernel into RAM and transfers control to it.**
6. **The OS initializes itself** — sets up data structures, starts essential background processes, and finally starts a login prompt or a GUI (like your GNOME session).

**Why this matters conceptually:** the OS *has* to be loaded into RAM before anything else can run, because it's the thing that will subsequently load and manage everything else. Hardware (BIOS/UEFI) bootstraps this from nothing — it's the one part of "loading a program" that isn't done by an OS, because at that point, no OS is running yet.

---

## 4. The Kernel

The slide gives a compact definition: **the kernel is the most fundamental part of the OS** — the piece that:
- Is the *first* thing loaded into memory when the OS starts,
- Runs at all times while the computer is on,
- Handles the core jobs: memory management, process management, disk management, etc.
- Is often used loosely as a synonym for "the OS" itself, though technically the OS is a broader concept (kernel + system libraries + utilities etc.).

**Analogy:** If the OS is a company, the kernel is upper management sitting in the head office with all the keys and authority — always present, always running the core operations, while other departments (drivers, GUI, utilities) come and go around it.

---

## 5. Dual-Mode Operation (Kernel Mode vs User Mode)

This is arguably the single most important concept in this whole lecture — it recurs throughout the entire OS course.

### 5.1 The core idea

The CPU itself (hardware!) supports **two modes of operation**:

| | **Kernel Mode** | **User Mode** |
|---|---|---|
| Also called | Supervisor mode, master mode, privileged mode, system mode | Slave mode, unprivileged mode, restricted mode |
| Who runs here | The OS / kernel | Ordinary application programs (browser, your C++ programs, etc.) |
| Instruction access | **All** machine instructions available | Only a **subset** — "privileged" instructions (direct hardware access, I/O control) are forbidden |
| Hardware/RAM access | Direct access | No direct access — must go through the OS via a system call |
| Mode bit | 0 | 1 |
| A crash here... | ...can bring down the **entire system** | ...usually only kills that one process; system keeps running |
| Address space | All kernel-mode code typically shares one address space | Each user process gets its **own, separate virtual address space** — isolated from others |

**Why does this distinction exist at all?** Protection. If any random application could directly write to arbitrary memory or directly command the disk controller, one buggy or malicious program could corrupt the whole system, read other users' private data, or crash everything. By making a **hardware-enforced** boundary (there's literally a "mode bit" flipped by hardware, not something software can just lie about), the OS protects itself and other programs from misbehaving user code.

**Analogy from the slide, and it's a good one:** *the OS is the Boss, applications are laborers.* The boss has the master keys to every room in the building (kernel mode = full hardware access). Laborers can only enter the rooms they're authorized for (user mode = restricted instruction set), and if a laborer needs something from a locked room, they have to *formally request it from the boss* rather than break in themselves. That formal request is exactly what a **system call** is (Section 7).

### 5.2 When does the CPU switch INTO kernel mode?

Three triggers (from the slide):

**A. System boot** — as we saw in Section 3, the very first thing that runs after power-on is inherently privileged; the machine starts in kernel mode.

**B. Hardware interrupt** — a hardware device (keyboard, disk, network card, timer/clock) signals the CPU "I need attention" (e.g., "a key was pressed," "disk read finished," "a clock tick occurred"). The CPU automatically jumps into kernel mode to let the OS handle it, because responding to hardware requires privileged instructions.

**C. Trap** — a *software-generated* interrupt. Two flavors:
  - An **error condition**: division by zero, invalid memory access (segmentation fault), illegal instruction, etc.
  - A **deliberate request**: a user program executing a `TRAP` instruction on purpose, to ask the OS to perform a service. This deliberate case is exactly what a **system call** is.

---

## 6. Switching modes / "Trapping into kernel is costly"

When a user program wants an OS service, it can't just "call a function" the normal way (that would require directly jumping into kernel code with kernel privileges — not allowed). Instead, it uses the special **TRAP instruction**.

**The sequence** (as shown on your slide's diagram):

```
USER MODE (mode bit = 1)
   User process ──► System Call
                        │
                    [TRAP instruction]
                        ▼
KERNEL MODE (mode bit = 0)
   Save caller's state ──► Execute system call ──► Restore state
                        │
                    [RETURN, mode bit flips back to 1]
                        ▼
USER MODE (mode bit = 1)
   Resume process
```

Concretely, what happens on a trap:

1. **Hardware saves minimal CPU state** — program counter (PC), stack pointer (sp), and a few other registers — automatically, so execution can be resumed later exactly where it left off.
2. **CPU switches the mode bit** to kernel mode.
3. **The kernel figures out which system call was requested** (there's a numbered table — every system call has an ID number).
4. **The kernel validates the parameters** — is this file descriptor valid? Is this pointer legal? (User programs are not trusted, remember — that's the whole point of the mode separation.)
5. **The kernel does the actual work** (e.g., reads data from disk into a buffer).
6. **State is restored, mode bit flips back to user mode**, and control returns to the instruction right after the trap.

**Why "costly"?** All this saving/restoring of state, validating parameters, and mode-switching is pure overhead compared to a normal function call within the same program — it doesn't do "useful work" by itself, it's the *cost of crossing the protection boundary safely*. This is why, in real systems, minimizing unnecessary system calls (e.g., reading a file in large chunks instead of one byte at a time) genuinely improves performance — you're the person writing OS-adjacent code later in this course, so this matters.

---

## 7. System Calls

### 7.1 What they are

A **system call** is the *only* legitimate doorway from user-mode code into kernel-mode services. It's how a program asks the OS to:
- read/write a file,
- create/destroy a process,
- allocate memory,
- talk to a device,
- etc.

They're normally *not* invoked with a raw TRAP instruction by the programmer directly (too low-level, too machine-specific). Instead, the OS provides a **library** with ordinary-looking C functions (e.g., `read()`, `write()`, `fork()`) that internally execute the TRAP instruction for you. From your C code's perspective, calling `read()` looks exactly like calling any other function — the trap-and-privilege-switch machinery is hidden inside the library implementation.

```c
count = read(fd, buffer, nbytes);
```

This single line, when it runs, secretly does all 11 steps below.

### 7.2 The 11 steps of making a system call (from the slide's diagram — `read` example)

```
User program calling read:
  1. Push nbytes
  2. Push &buffer
  3. Push fd
  4. Call read           ──► (library procedure "read" starts)
  5. Put code for read in register
  6. TRAP instruction executed  ───► mode switches user → kernel
  7. Dispatch (kernel looks up which syscall this is, via a table)
  8. Sys call handler runs (actual work happens here)
  9. Return (mode switches kernel → user)
 10. Library procedure "read" returns to caller
 11. Increment SP (stack cleanup)
```

The key insight: **step 6 is the only point where privilege actually changes.** Everything before it (1–5) is ordinary user-mode code (the library wrapper preparing arguments); everything from 9 onward is back in user mode.

### 7.3 Windows vs UNIX system calls (from your slide's table)

| UNIX | Win32 | What it does |
|---|---|---|
| `fork` | `CreateProcess` | Create a new process |
| `waitpid` | `WaitForSingleObject` | Wait for a process to finish |
| `execve` | *(none — folded into CreateProcess)* | Replace a process's program image |
| `exit` | `ExitProcess` | Terminate execution |
| `open` | `CreateFile` | Create/open a file |
| `close` | `CloseHandle` | Close a file |
| `read` | `ReadFile` | Read data |
| `write` | `WriteFile` | Write data |
| `mkdir` | `CreateDirectory` | Create a directory |
| `kill` | *(none)* | Win32 has no UNIX-style signals |

Two things worth internalizing:
- UNIX's model treats `fork` + `execve` as **two separate steps** (duplicate the process, then replace its program) — Windows' `CreateProcess` does both in one call. This split is a genuinely elegant UNIX design decision that pays off in flexibility (see the `fork()` discussion below).
- Not everything maps 1:1 — Windows lacks direct equivalents for UNIX file *links* or *signals* (`kill`), reflecting real design differences between the two systems, not just naming differences.

---

## 8. Core OS Components / Services (the "5 pillars")

The slide lists five major OS responsibilities. You'll spend entire chapters (and possibly entire course units in later semesters) on each:

1. **Process Management** — creating, scheduling, terminating processes; deciding who runs on the CPU and when.
2. **Memory Management** — allocating RAM to processes, keeping them isolated from each other, handling virtual memory.
3. **I/O Management** — mediating access to devices via drivers, handling interrupts from hardware.
4. **Deadlock Management** — detecting/preventing situations where processes are stuck waiting on each other forever (classic example: two processes each holding a resource the other needs).
5. **File System** — organizing persistent storage into files and directories, translating "open this file" into raw disk operations.

This lecture only *names* these; each gets its own deep dive later in CSE313/314.

---

## 9. UNIX System Calls for Process Management: `fork()` in depth

This is the trickiest and most important concrete mechanism from the slide, so it deserves the most space.

### 9.1 The four core process-management calls

| Call | Purpose |
|---|---|
| `pid = fork()` | Create a child process — an **exact duplicate** of the calling process |
| `pid = waitpid(pid, &statloc, options)` | Parent waits for a child to terminate |
| `s = execve(name, argv, environp)` | **Replace** the calling process's program with a different one |
| `exit(status)` | Terminate the current process, returning a status code |

### 9.2 What `fork()` actually does — step by step

When process **A** calls `fork()`:

1. Control switches to the kernel (it's a system call, so — trap!).
2. The kernel creates a brand-new process **B** that is an **exact duplicate** of A: same code, same data, same open files, same position in the program (the same value of the program counter).
3. Now there are **two identical, independent processes**, both about to resume execution from the instruction *right after* the `fork()` call.
4. The only difference between them: **the return value of `fork()` itself.**
   - In the **child** (B): `fork()` returns `0`.
   - In the **parent** (A): `fork()` returns the child's **PID** (a positive integer).
   - (If fork fails, it returns `-1` to the parent, and no child is created.)

This return-value trick is the *only* way your code can tell, after the fork, "am I the parent or the child?" — because otherwise the two processes are running literally identical code.

```c
pid_t pid = fork();
if (pid == 0) {
    // This block runs ONLY in the child
} else if (pid > 0) {
    // This block runs ONLY in the parent; pid = child's PID here
} else {
    // fork failed
}
```

### 9.3 Example 1 — from the slide

```c
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
int main()
{
    fork();               // makes two processes running the same program from here on
    printf("Hello world!\n");
    return 0;
}
```

**What prints?** `"Hello world!"` gets printed **twice** — once by the parent, once by the child — because after the `fork()` line, *both* processes independently continue executing the rest of `main()`, including the `printf`.

### 9.4 Example 2 — the fork "tree" (three forks in a row)

```c
#include <stdio.h>
#include <sys/types.h>
int main()
{
    fork();
    fork();
    fork();
    printf("hello\n");
    return 0;
}
```

**How to reason about this:** each `fork()` call **doubles** the number of currently-existing processes running from that point onward, because *every* existing process (parent and every child created so far) independently executes each subsequent line, including the next `fork()`.

- Before any fork: 1 process (P0)
- After fork #1: 2 processes (P0's child, call it P1, is created; P0 continues too)
- After fork #2: each of the 2 existing processes forks → 4 processes total
- After fork #3: each of the 4 existing processes forks → **8 processes total**

So `"hello\n"` gets printed **8 times** (2³ = 8), once by each of the 8 final processes. Your slide's diagram shows exactly this branching tree (P0 → P1, P2 → P3, P4, P5, P6 → P7, etc.) — try drawing it yourself with pen and paper; it's the single best exercise for internalizing `fork()`. It's a genuinely common exam/interview question, and the trick is always: **count how many processes exist at each fork, not "how many times fork() was called."**

**A subtlety to notice:** the *order* in which these 8 processes actually run (and hence the order the 8 "hello"s appear on screen) is **not guaranteed** — the OS scheduler decides. This is your first taste of **nondeterminism** in concurrent/parallel systems, a theme that will follow you through the rest of this course (race conditions, synchronization, etc.).

### 9.5 Why is `fork()` + `exec()` split into two calls?

This is a very "why does UNIX do it this way" question worth understanding, even though it's slightly beyond the slide. Splitting process creation (`fork`) from program replacement (`exec`) means a program can insert extra logic **between** the two — e.g., a shell can redirect the child's input/output to a file, or change its environment variables, *before* the child's memory image is replaced by the new program. Windows bundles this into one `CreateProcess` call and has to pass all that configuration as parameters instead — less flexible, but a single call.

---

## 10. OS Architecture: Monolithic Kernel vs Microkernel

This is about *how the OS's own internals are organized* — not how it talks to applications, but how the OS is built internally.

### 10.1 Monolithic Kernel

```
 User Space:   Applications
               Libraries
 ─────────────────────────────────────
 Kernel Space: File Systems
               Inter-Process Communication (IPC)
               I/O and Device Management
               Fundamental Process Management
 ─────────────────────────────────────
 Hardware
```

- **The entire OS** — process management, memory management, file systems, device drivers, IPC, everything — runs as **one big program**, all in kernel mode, all sharing the same address space.
- Any procedure inside the kernel can call any other procedure directly (no protection boundary between them).

**Advantages:**
- **Fast.** Internal components call each other directly — no need to cross the expensive user/kernel boundary (Section 6) between, say, the file system and the disk driver, because both are already in kernel space.

**Disadvantages:**
- **Huge, complex codebase** (Linux's kernel alone is on the order of tens of millions of lines).
- **Hard to understand/maintain** — thousands of procedures freely calling each other with no isolation.
- **Fragile:** since everything shares kernel-mode privilege, **a bug in any single component** (say, a buggy audio driver) **can crash the entire system** — there's no wall between "the audio driver" and "the memory manager" once you're inside the kernel.

Examples: traditional UNIX, Linux, most of Windows.

### 10.2 Microkernel

```
 User Space:   Applications
               Libraries
               File System | Process Server | Pagers | Drivers | ...  ← "servers"
 ─────────────────────────────────────
 Kernel Space: MicroKernel  (just: IPC, basic process comm, I/O control)
 ─────────────────────────────────────
 Hardware
```

- The kernel is stripped down to the **bare minimum**: basic process/thread scheduling, inter-process communication (IPC), and low-level I/O control.
- Everything else that used to be "in the kernel" — file systems, device drivers, network stacks — is pulled **out into user space**, running as ordinary, unprivileged processes called **servers**.
- If a user program (client) wants a file read, it doesn't get it directly from the kernel; it sends a **message** to the file server, which does the work and messages back a reply. The microkernel's main job is just to reliably deliver these messages between clients and servers.

**Advantages:**
- **More secure/reliable.** Since most services run as ordinary user processes, a crash in one server (e.g., the audio driver process crashes) does **not** bring down the entire OS — the rest of the system, including the kernel, remains untouched. Compare directly to the monolithic case above — this is literally the same "buggy audio driver" example, with the opposite outcome.
- **Easier to extend.** Adding a new service just means adding a new user-space server — no need to touch or recompile the kernel itself.

**Disadvantages:**
- **Performance overhead.** Every service request now involves message-passing between user-space processes (via the kernel) instead of a direct in-kernel function call — and remember from Section 6, crossing between user mode and kernel mode is expensive. A microkernel design does this *more often* than a monolithic one.

Examples: MINIX 3, QNX, L4 — and notably, **macOS/Mac OS X's kernel (XNU) is derived from the Mach microkernel**, though in practice modern macOS blends monolithic and microkernel ideas (a "hybrid kernel").

### 10.3 The exam-style example, worked out

> *"What is the impact of a severe bug in the audio driver, in a monolithic OS vs. a microkernel OS?"*

- **Monolithic:** the audio driver runs *inside* the kernel, sharing its address space and privilege with everything else. A severe bug (e.g., writing to an invalid memory address) can corrupt kernel data structures belonging to *completely unrelated* subsystems (memory manager, file system, scheduler) and bring the **entire machine down** — you'd need to reboot.
- **Microkernel:** the audio driver is just another user-space process talking to the microkernel via well-defined, checked messages. A severe bug in it might crash *that one process* — you'd lose sound, maybe get an error, but the file system, scheduler, and every other part of the OS keeps running fine.

---

## 11. Multiuser OS

- Allows **multiple users to access the underlying hardware resources concurrently** (at the "same time," from each user's perspective).
- Users may be logged in locally, or **remotely over a network**.
- The OS/system administrator allocates resources (CPU time slices, memory, storage quota) fairly across users.
- Examples of categories: distributed systems, time-sliced (timesharing) systems, multiprocessor systems.

**Intuition:** think of a university's central Linux server that many students SSH into simultaneously to compile and run code — the OS is juggling all of your processes, keeping you isolated from each other's memory and files, while sharing one physical machine.

---

## 12. Multiprocessor OS

- Built for systems with **more than one physical CPU/core** working together.
- All processors typically **share a common memory**, and the OS's job is to allocate/schedule work across all of them (deciding which core runs which process/thread at any given moment).
- Improves overall throughput/performance since multiple things can genuinely execute *simultaneously* (not just interleaved, like on a single core).
- Modern relevance: **every laptop today, including your HP Victus 15 (i5-13420H — a multi-core CPU), is technically a multiprocessor system** from the OS's point of view; Linux, Windows, and macOS are all multiprocessor operating systems in this sense — this isn't some exotic server-only category anymore.

---

## 13. Quick self-check (try answering without looking back)

1. Why can't the shell/GUI be considered "part of the OS"?
2. Name the three triggers that cause the CPU to switch into kernel mode.
3. In the `read(fd, buffer, nbytes)` example, at exactly which step does the mode bit actually flip? Why is everything before that step still "cheap"?
4. If a program calls `fork()` **four** times in a row (unconditionally, like the slide's example but with one more call), how many total processes exist by the end, and how many times will a `printf` after the last fork execute?
5. Give one advantage and one disadvantage each for monolithic and microkernel designs — and explain the audio-driver-bug example for both.
6. What's the difference between the `read` **system call** and the `read` **library procedure**, and why does UNIX bother separating these?

*(Answers are embedded in the sections above — if you can answer all six confidently without re-reading, this lecture is solid.)*

---

## 14. What's coming next (per the book's chapter flow)

This lecture (Ch. 1 intro) sets up vocabulary that the rest of the OS course builds on directly:
- **Process management** (Ch. 2) — will formalize what a "process" actually *is*, how `fork()`'s child gets scheduled, context switching, threads.
- **Memory management** (Ch. 3) — virtual memory, paging — how each process gets "its own" address space despite physically sharing RAM.
- **File systems** (Ch. 4) — the abstraction layer over raw disk blocks.
- **I/O** (Ch. 5) and **Deadlocks** (Ch. 6) round out the "5 pillars" from Section 8.

Given you're also deep in assembly language (CSE315/316) right now, it's worth noticing how much overlap there is: the **TRAP instruction**, **mode bit**, and **PSW (Program Status Word)** concepts from this lecture are literally the same hardware mechanisms your assembly course will eventually touch from the "below the OS" side. Seeing the same concept from two angles (OS course: "why do we trap into the kernel," assembly course: "here's the actual instruction and register behavior") should reinforce both.