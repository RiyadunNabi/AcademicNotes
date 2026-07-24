# IPC — Exam Revision Notes (CT-1, Pages 1–33)

> Condensed from the full beginner-explanation file. Every topic, keyword, definition, formula/code and fact is kept — only the story padding/repetition is cut.

---

## 1. Process Basics

- **Program** = code stored on disk (not running).
- **Process** = a program currently running (loaded into RAM).
- Program → (run) → Process.
- Multiple processes can run "simultaneously" — CPU rapidly switches between them (context switching).
- **Independent processes**: don't affect / aren't affected by other processes (e.g., Calculator & Paint).
- **Cooperating processes**: can affect / be affected by others (e.g., Google Docs & Google Drive). PDF: processes may be independent or cooperating.

### Why processes cooperate (4 reasons)
1. **Information Sharing** – e.g., Chrome downloads photo.jpg → Photos app opens it.
2. **Computation Speedup** – split work (e.g., 1000 images → 4 processes × 250 images) → faster completion.
3. **Modularity** – divide a big system into parts (Login, Chat, Notification, Video processes in Facebook).
4. **Convenience** – doing multiple things at once (typing report while Spotify plays).

### Interprocess Communication (IPC)
- **Definition**: mechanism allowing cooperating processes to exchange data/information.
- Example: WhatsApp — Sender → Message → Receiver.

### Shell Pipeline Example (classic IPC example)
```bash
cat chapter1 chapter2 chapter3 | grep tree
```
- `cat` combines & prints all file contents (one process).
- `|` (**pipe**) sends output of first command directly as input to second command.
- `grep tree` filters matching lines (another process).
- **Key fact**: `cat` and `grep` are two different processes; the pipe is the IPC mechanism connecting them.

**Lesson 1 Summary**: Program vs Process; independent vs cooperating; IPC; pipe example (`cat file | grep word`) — output of one process becomes input of another.

---

## 2. IPC Issues (3 Core Problems)

Analogy: two people writing on the same whiteboard simultaneously → output gets garbled (e.g., "HEWOLLRLOD").

### Problem 1 — How does one process send info to another?
- Need a communication method. Answer differs for **processes** vs **threads**.
- Common methods: **Message Passing**, **Shared Memory**.

### Problem 2 — What if both processes use the same data? → Mutual Exclusion
- Bank account example: Balance = 1000 Tk; ATM1 & ATM2 both read 1000, both withdraw 500 "simultaneously" → both compute new balance 500 instead of correct 0. **Interference** occurs.
- **Mutual Exclusion**: only ONE process allowed to use a shared resource at a time (bathroom-with-one-key analogy).

### Problem 3 — Proper Sequencing → Synchronization
- Some processes depend on order (Cook → Serve → Eat, not Eat → Cook).
- Computer example: Download File (A) must finish before Open File (B). Wrong order = file doesn't exist yet.
- **Synchronization**: making processes execute in the correct order.

### Mutual Exclusion vs Synchronization (KEY TABLE — common exam Q)

| Concept | Meaning | Analogy |
|---|---|---|
| Mutual Exclusion | Prevent 2+ processes from using the same shared resource at the same time | One-lane bridge — only one car at a time |
| Synchronization | Make processes execute in the correct order | Traffic signal — Red → Yellow → Green |

### Big picture (Lesson 2)
```
Multiple Processes → Need to cooperate → 3 Problems:
 ├─ How to communicate? → Shared Memory / Message Passing
 ├─ How to avoid interfering? → Mutual Exclusion
 └─ How to maintain correct order? → Synchronization
```

---

## 3. Shared Memory & Spooling Example (Race Condition intro)

- **Shared Memory**: a region of RAM that multiple processes can read/write (like a shared notebook).
- Faster than copying data between processes — store once in shared memory, others read it.
- **Printer Queue example**: 3 people print → OS creates a **print queue** in shared memory; printer prints one by one.

### Spooling
- **Definition**: storing print jobs (or similar tasks) in a queue so they are processed one at a time (bakery line analogy).

### The `in` variable
- Points to the **next free slot** in the queue.
- Example: `in = 3` → next file goes into slot 3.

### Single process (correct) sequence
```cpp
next_free = in;          // read
slot[next_free] = File1; // write
in = next_free + 1;      // update
```

### Race condition demo — Two processes together
1. P1: `next_free = in;` → reads 3.
2. **CPU switches** to P2 before P1 finishes.
3. P2: `next_free = in;` → also reads 3 (both think slot 3 is theirs).
4. P1 writes `slot[3] = File1`.
5. P2 writes `slot[3] = File2` → **File1 is overwritten/lost**.
6. Both independently compute `in = 4`.
- **Result**: File1 disappears — this is the classic **race condition** demonstration.

### Why not just write `slot[in++] = file;`?
- **No** — doesn't fix it. `in++` is NOT a single machine instruction; it's actually:
  1. Load `in` from memory
  2. Increase it
  3. Store it back
- OS can switch process **between these steps** → not atomic → race condition still possible.

**Lesson 3 Summary**: Shared memory = RAM region accessible by multiple processes; Spooling = queued print jobs; `in` = next free slot; concurrent read/update of `in` → data loss; `in++` looks atomic in C++ but isn't at machine level.

---

## 4. Race Condition & Critical Region

### Race Condition — Formal Definition (IMPORTANT, exam quote-worthy)
> "Two or more processes are reading or writing shared data, and the final result depends on who runs precisely when."

- Called a "race" because processes race to access shared data first; outcome depends on **CPU scheduling, interrupts, timing**.
- Example: `count = 10`; two processes execute `count++` concurrently.
  - Correct (sequential) execution → count = 12.
  - Interleaved (racy) execution → both read 10, both write 11 → **count = 11** (wrong, lost update).
- Danger: **very hard to debug** — bug appears **randomly/intermittently** (works sometimes, fails other times — "heisenbugs").

### Critical Region (Critical Section)
- **Definition**: the part of a program that accesses shared memory/shared data.
- Example:
```cpp
int x = 5;
cin >> a;
x++;        // ← Critical Region (if x is shared)
cout << a;
```
- Goal: only ONE process should execute the critical region at a time (bathroom-key analogy). Want execution pattern `AAAA then BBBB`, not interleaved `ABAB`.

### 4 Requirements for a Correct Critical-Section Solution (VERY IMPORTANT — frequently asked)

| # | Requirement | Easy meaning |
|---|---|---|
| 1 | Mutual Exclusion | No two processes inside the critical region at the same time |
| 2 | No assumption on speed/CPUs | Solution must work regardless of CPU speed or number of CPUs |
| 3 | No unnecessary blocking | A process outside the critical region must not block others |
| 4 | No starvation / bounded waiting | No process should wait forever |

### Big Picture Flow (Lesson 4)
```
Processes → Need Shared Data → Race Condition → Critical Region →
Need Correct Solution → (1 at a time, CPU-independent, no unnecessary block, no starvation)
```

---

## 5. Mutual Exclusion with Busy Waiting — Attempted Solutions 1 & 2

Four historical solutions (slides order):
1. Disabling Interrupts
2. Lock Variables
3. Strict Alternation
4. Peterson's Solution ✅ (works, for 2 processes)

### Mutual = between two/more entities; Exclusion = keep others out.
> **Mutual Exclusion** = only one process may enter the critical region at a time.

### Solution 1: Disabling Interrupts
```cpp
disable_interrupts();
// critical region
enable_interrupts();
```
- **Interrupt**: signal that tells CPU "stop what you're doing, handle this event" (e.g., timer, keyboard).
- Idea: disabling interrupts prevents clock interrupt → prevents process switching → only one process runs without being pre-empted.
- **Two problems**:
  1. If a process forgets to re-enable interrupts → system may freeze (no further switching).
  2. **Multiprocessor systems**: disabling interrupts on CPU1 doesn't stop CPU2 — CPU2 can still access shared memory → race condition still possible.
- **Conclusion**: used only inside OS kernel for very short operations; not for user programs. Doesn't scale to multicore.

### Solution 2: Lock Variable
```cpp
int lock = 0;
while(lock);      // wait if locked
lock = 1;          // acquire lock
// Critical Region
lock = 0;          // release lock
```
- `lock = 0` → free (vacant); `lock = 1` → occupied.
- **Fails**: `while(lock);` (check) and `lock = 1;` (set) are TWO separate steps — a gap exists between them.
  - P1 checks `lock==0`, about to set lock=1, but CPU switches to P2.
  - P2 also checks `lock==0` (still true, P1 hasn't set it) → P2 sets lock=1 and enters.
  - CPU switches back to P1 → P1 also sets lock=1 and enters.
  - **Both P1 and P2 end up inside the critical region simultaneously** → Rule #1 (mutual exclusion) broken.
- **Root cause**: check-and-set is **not atomic**.

---

## 6. Strict Alternation, Busy Waiting, Peterson's Intro

### Strict Alternation
```cpp
int turn = 0;   // turn==0 → P0's turn; turn==1 → P1's turn
// P0: while(turn != 0);  ... turn = 1;
// P1: while(turn != 1);  ... turn = 0;
```
- Processes take turns strictly (TV remote — odd minutes to A, even to B).
- **Fails**: If P1 has nothing to do (not entering critical region) but it's P1's turn, **P0 is forced to wait** even though P1 isn't using the resource.
- **Violates Requirement 3** (a process outside the critical region should not block others).

### Busy Waiting
- **Definition**: repeatedly checking a condition in a loop instead of yielding/sleeping, e.g. `while(lock);`.
- **Problem**: wastes CPU time — CPU works hard but accomplishes nothing useful.

### Peterson's Solution — Structure (intro)
```cpp
while (true) {
    enter_region(process_id);
    // Critical Region
    leave_region(process_id);
    // Non-Critical Region
}
```
- `enter_region()`: like asking a guard — checks if someone's inside; if yes, wait; if no, enter.
- `leave_region()`: tell the guard "I'm leaving" — lets next process in.
- Peterson's algorithm = first **software-only** solution that **correctly solves the critical-section problem for 2 processes**.

### Summary Table (Lesson 6)
| Method | Pros | Cons |
|---|---|---|
| Strict Alternation | Simple | Forces turn-taking even when other process idle; violates Req 3 |
| Busy Waiting | Simple to implement | Wastes CPU time |
| Peterson's | Correct, fair, no unnecessary block | (still uses busy waiting internally) |

---

## 7. Peterson's Algorithm (Detailed — HIGH EXAM IMPORTANCE)

Uses **two shared variables**:
```cpp
bool interested[2];   // interested[i] = true → process i wants to enter
int turn;              // whose turn to wait if both want to enter
```
- `interested[]`: array of 2 booleans, initially both `false` (nobody wants to enter).
- `turn`: relevant **only** when both processes want to enter at the same time; decides who waits.

### Full Code
```cpp
void enter_region(int process) {
    int other = 1 - process;
    interested[process] = true;
    turn = process;
    while (turn == process && interested[other]) {
        // wait (busy wait)
    }
}

void leave_region(int process) {
    interested[process] = false;
}
```

### Line-by-line
- `int other = 1 - process;` → gives the OTHER process's index (0↔1 trick).
- `interested[process] = true;` → "I want to enter."
- `turn = process;` → "If both want in, let the other go first" (whoever writes to `turn` LAST ends up waiting).
- `while (turn == process && interested[other])` → loop continues **only if BOTH** conditions true:
  - Condition 1: `turn == process` (it's supposedly my process's turn to wait)
  - Condition 2: `interested[other]` (other process also wants in)
  - If either is false → proceed into critical region immediately.

### Two key cases (exam favorite)
**Case 1 – only P0 wants to enter**: `interested[1]=false` → P0's while-condition is false → P0 enters immediately. If P1 later wants to enter, it waits till P0 leaves.

**Case 2 – both want to enter simultaneously**: Say the last write to `turn` is `turn = 1`.
- P0 checks: `turn==0`? → FALSE → loop condition FALSE → **P0 enters immediately**.
- P1 checks: `turn==1`? TRUE, and `interested[0]`? TRUE → loop condition TRUE → **P1 waits**.
- When P0 calls `leave_region(0)` → `interested[0]=false` → P1's condition becomes FALSE → P1 enters.

### Why it works (guarantee)
- Exactly one process enters even if both start at the exact same time, because whichever process's write to `turn` happens **last** determines who waits.
- Satisfies all requirements for 2 processes: mutual exclusion ✅, immediate entry if only one wants in ✅, exactly one waits if both want in ✅, waiting process proceeds once the other leaves ✅.

### Easy memory trick
1. Raise hand: `interested[me] = true;`
2. Be polite: `turn = me;` ("you go first if we both want in")
3. Check: `while(...)` → wait or enter.

---

## 8. Problems with Busy Waiting: Priority Inversion; Sleep & Wakeup

### Why Busy Waiting is still bad (recap)
- Wastes CPU cycles — repeatedly checking a lock that's still held.

### Priority Inversion (IMPORTANT TERM)
- Setup: Low-priority process **L** enters critical region. High-priority process **H** becomes ready and wants to enter same critical region, but L is inside → H does `while(lock);` (busy-waits).
- **Disaster**: Since H has higher priority, CPU scheduler always picks H to run → **L never gets CPU time to finish and release the lock** → H waits forever, L never runs.
- **Definition**: Priority inversion = a high-priority process is effectively blocked indefinitely by a low-priority process because the low-priority process can't get CPU time to release the resource, while the high-priority process busy-waits.
- Real-life analogy: CEO (H) waits outside bathroom occupied by employee (L); security only attends to CEO, so employee never gets a chance to come out and free the bathroom.

### Solution: Sleep & Wakeup
- Instead of `while(lock);` (busy wait), process calls `sleep()` → "wake me up when the resource is available."
- **Process states**: Running → (needs resource) → **BLOCKED** (sleeping) → (resource free) → **READY** → Running again.
- While sleeping/BLOCKED, the CPU is free to run other processes — **no CPU time wasted**.

```cpp
sleep();          // suspend this process until woken
wakeup(process);  // make a specific sleeping process READY again
```

### Busy Waiting vs Sleep — Comparison Table
| Busy Waiting | Sleep |
|---|---|
| Continuously checks | Stops checking |
| Uses CPU | Doesn't use CPU while blocked |
| Wastes CPU time | CPU can run other processes |
| Can cause Priority Inversion | Avoids unnecessary spinning |

---

## 9. Producer–Consumer Problem (Bounded Buffer Problem)

- **Producer**: creates data and puts it into a shared **buffer**.
- **Consumer**: removes/uses data from the shared buffer.
- **Buffer**: temporary storage area (fixed number of slots).
- **Bounded** = limited size (e.g., only 4 slots max).

### Two edge-case problems
1. **Buffer Full**: Producer has nothing where to place a new item → Producer **sleeps** until Consumer removes ≥1 item.
2. **Buffer Empty**: Consumer has nothing to take → Consumer **sleeps** until Producer adds ≥1 item.

### Sleep & Wakeup applied (looks fine at first)
- Buffer full → producer `sleep()`; consumer removes item → `wakeup(producer)`.
- Buffer empty → consumer `sleep()`; producer adds item → `wakeup(consumer)`.

### The Lost Wakeup Problem (CRITICAL EXAM TOPIC)
Setup: `count` = number of items in buffer; initially `count = 0`.

**Step-by-step race**:
1. Consumer checks `count == 0` → true → **decides** to sleep, but has **not yet called `sleep()`**.
2. CPU switches to Producer **before** consumer actually sleeps.
3. Producer creates item A → `count = 1` → Producer thinks "consumer was waiting" → calls `wakeup(consumer)`.
4. **But consumer wasn't asleep yet** → the wakeup signal has nowhere to go → it is **lost** (does nothing).
5. CPU switches back to Consumer → Consumer now executes `sleep()` → goes to sleep.
6. Producer will not send another wakeup (it thinks it already woke the consumer).
7. **Result**: Consumer sleeps **forever**, even though the buffer has data (`count=1`).

- **Root cause**: the sequence "Check → Sleep" is **not atomic** — a gap exists where another process can change the situation and send a wakeup that arrives too early (and is thus lost).
- Real-life analogy: "I'll wake you for dinner" / "Dinner's ready!" (said before you actually fell asleep) / you go to sleep later and are never woken — mother thinks she already told you.

### Summary
- Sleep & Wakeup solves busy waiting ✅ but introduces Lost Wakeup ❌.
- Need a mechanism that avoids **both** busy waiting **and** lost wakeups → **Semaphores**.

---

## 10. Semaphores

### Motivation — 3 problems so far
1. **Race Condition** — shared data corrupted by concurrent access.
2. **Busy Waiting** — `while(lock);` wastes CPU.
3. **Sleep & Wakeup → Lost Wakeup** — wakeup signal can disappear.

### Definition
> A **semaphore** is a new variable type / a kind of generalized lock, introduced by **Dijkstra**.
- Acts as a "smart lock": tracks available resource count, makes waiting processes sleep, and wakes them automatically.
- Analogy: classroom with 3 chairs → `semaphore = 3` (free chairs). Each seated student decrements it; when 0, a new arriving student doesn't busy-wait — they **sleep** until a chair frees.
- Like a counter: usage decrements it; releasing increments it.

### Types of Semaphores
1. **Counting Semaphore**: can take many values (e.g., 0 to 100) — e.g., parking lot with 100 spaces.
2. **Binary Semaphore**: only values **0** or **1** — e.g., single-occupancy bathroom. Often called a **mutex lock** because it provides mutual exclusion.

### Two Operations (only 2, besides initialization)
| Operation | Aliases | Effect |
|---|---|---|
| **DOWN** | `P()`, `wait()` | Try to take a resource. If semaphore > 0, decrement and continue. If semaphore == 0, the process **sleeps** (does NOT go negative, does NOT busy-wait). |
| **UP** | (often `V()`, `signal()`) | Release a resource: increment semaphore. If a process was sleeping, OS wakes exactly **ONE** waiting process. |

- **Key fact**: semaphore value **never goes below zero**. If `down()` is called when semaphore = 0, the process sleeps instead of going to -1.
- `up()` wakes exactly one sleeping process (not all); that process resumes/completes its pending `down()`.

### Why Semaphores avoid Lost Wakeup
- Checking the value, changing it, and (if needed) sleeping are treated as **one atomic operation** — cannot be interrupted halfway.
- **Atomic** = an operation that cannot be interrupted partway through (like a full, indivisible signature).
- Without atomicity: two processes could both read semaphore=1 simultaneously, both decrement → 2 processes wrongly enter using 1 resource.
- With atomicity: OS guarantees strict order — e.g., Process A's `down()` completes (semaphore 1→0) fully before Process B's `down()` sees semaphore=0 and sleeps. No race condition.

### Complete worked example
```
Semaphore = 1
A: down() → 1→0 → A enters
B: down() → sees 0 → B sleeps
A: finishes → up() → 0→1 → OS wakes B
B: continues
```
Result: No busy waiting, no lost wakeup, no race condition.

### Hotel analogy
- `semaphore` = number of free rooms.
- Guest arrives → `down()`: room available → take it; no room → wait in lobby (sleep).
- Guest leaves → `up()`: free one room → reception immediately calls next waiting guest.

### Semaphore Summary Table
| Concept | Easy Meaning |
|---|---|
| Semaphore | Smart counter controlling access to shared resources |
| Counting Semaphore | Can have values 0,1,2,3,... |
| Binary Semaphore | Only 0 or 1 (a.k.a. mutex) |
| `down()` | Try to take a resource; sleep if none available |
| `up()` | Return a resource; wake one waiting process if needed |
| Atomic | Operation cannot be interrupted halfway |

### Entire Chapter Flow (Big Picture — good for last-minute revision)
```
Processes Need Shared Data
        │
        ▼
   Race Condition
        │
        ▼
   Critical Region
        │
        ▼
  Busy-Waiting Solutions
   ├── Disable Interrupts ❌ (fails on multicore / forgetting re-enable)
   ├── Lock Variable ❌ (check+set not atomic)
   ├── Strict Alternation ❌ (violates Req 3 — unnecessary blocking)
   └── Peterson's Algorithm ✅ (correct, but only for 2 processes; still busy-waits)
        │
        ▼
  Busy Waiting wastes CPU → Priority Inversion risk
        │
        ▼
   Sleep & Wakeup (fixes CPU waste)
        │
        ▼
   Lost Wakeup Problem (check→sleep not atomic)
        │
        ▼
      Semaphores ✅ (atomic down/up; no busy wait; no lost wakeup)
```

## Master Topic Checklist ✅
- Process vs Program; Independent vs Cooperating processes
- 4 reasons to cooperate (Info sharing, Speedup, Modularity, Convenience)
- IPC definition; shell pipe example (`cat | grep`)
- 3 IPC problems: communication method, Mutual Exclusion, Synchronization
- Mutual Exclusion vs Synchronization table
- Shared Memory; Spooling; the `in` variable; race demo; `in++` non-atomicity
- Race Condition (formal definition); Critical Region definition
- 4 Requirements of a correct CS solution
- Disabling Interrupts (+2 drawbacks)
- Lock Variable (+ why it fails)
- Strict Alternation (+ violates Req 3)
- Busy Waiting definition & drawback
- Peterson's Algorithm — variables, full code, line-by-line, 2 cases, why it works
- Priority Inversion (H/L scenario)
- Sleep & Wakeup; process states (Running/BLOCKED/READY)
- Producer–Consumer / Bounded Buffer problem; buffer full/empty handling
- Lost Wakeup Problem — full step-by-step trace
- Semaphores: definition (Dijkstra), types (Counting/Binary=mutex), down()/up(), atomicity, why it solves lost wakeup, worked example
