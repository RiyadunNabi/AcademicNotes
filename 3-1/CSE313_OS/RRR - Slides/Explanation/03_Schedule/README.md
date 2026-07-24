# CPU Scheduling — Complete Study Guide
### (Based on Tanenbaum Ch. 2.4, scoped to your teacher's slides)

---

## 0. Why does scheduling even exist?

Imagine you have **one CPU** but **many processes** that are all "ready" to run (not blocked on I/O, not waiting for anything — just sitting there wanting CPU time). Someone has to decide: *which one runs next?*

That decision-maker is called the **scheduler**. The rule/algorithm it uses to decide is the **scheduling algorithm**.

This matters because switching between processes isn't free. A **process switch** (context switch) costs real time:

1. Trap from user mode → kernel mode.
2. Save the current process's state (registers etc.) into the process table.
3. Possibly save memory-map info (page table reference bits).
4. Run the scheduling algorithm to pick the next process.
5. Reload the MMU (Memory Management Unit) with the new process's memory map.
6. Start the new process.
7. Bonus pain: the CPU cache often gets invalidated, so the new process runs slow at first because everything has to be re-fetched from RAM.

So: too many switches per second = CPU wastes time doing "admin work" instead of actual computation. Too few switches = poor responsiveness. Scheduling is a balancing act.

**Why it matters practically:** a good scheduler affects CPU utilization, how "snappy" a system feels to a user, and whether important tasks (e.g., a nuclear reactor's safety monitor) get attention when they need it.

---

## 1. Process Behavior: CPU Bursts and I/O Bursts

Almost every process alternates between two phases:

- **CPU burst**: a period where the process is actually computing (using the CPU), with no I/O.
- **I/O burst**: a period where the process is blocked, waiting for I/O (disk read, network response, etc.) to complete.

A process's life looks like: `compute → I/O wait → compute → I/O wait → ...`

> **Subtlety:** if the CPU is copying pixels to video RAM to update your screen, that is *still* CPU computation, not I/O — because the CPU is actively working. "I/O" specifically means the process is **blocked**, waiting on an external device.

### CPU-bound vs. I/O-bound processes

| Type | Behavior | Example |
|---|---|---|
| **CPU-bound** (compute-bound) | Long CPU bursts, infrequent I/O waits | Video encoding, encryption, matrix multiplication |
| **I/O-bound** | Short CPU bursts, frequent I/O waits | A shell waiting for you to type a command, a text editor |

**Key insight (the book stresses this, and it's the kind of thing that shows up in exams):** what makes a process "I/O-bound" is *not* that its I/O takes a long time. It's that it does *very little computing between* I/O requests. Whether a disk read takes 1 ms or 100 ms, issuing the *request* takes the same tiny amount of CPU time. I/O-bound processes just don't compute much before asking for I/O again.

**Trend to know:** as CPUs keep getting faster (much faster than disks), *more and more processes become effectively I/O-bound* relative to the CPU's speed. This is why scheduling I/O-bound processes well (giving them the CPU quickly so they can fire off their I/O request and get the disk/network busy) becomes increasingly important over time.

---

## 2. When Does the Scheduler Actually Get Invoked?

Four/five natural trigger points:

1. **A new process is created** — now both parent and child are "ready." Scheduler decides who runs: parent or child. Either is a legitimate choice.
2. **A process exits** — it's gone, so someone else from the ready queue must be picked (or, if nobody's ready, an idle process runs).
3. **A process blocks** — e.g., it calls `read()` and needs to wait for a disk, or it's waiting on a semaphore. Since it can't run now, someone else must be chosen.
4. **An I/O interrupt occurs** — some device finished its work, so whatever process was waiting on it may now be ready. The scheduler must decide: keep running the current process, switch to the newly-ready one, or run something else entirely.
5. **A clock/timer interrupt occurs** (if the hardware has a periodic clock) — this is what makes *preemption* possible (see below). The scheduler can be invoked at every tick or every k-th tick.

---

## 3. Preemptive vs. Nonpreemptive Scheduling

This classification is about **what happens at a clock interrupt.**

- **Nonpreemptive**: once a process starts running, it keeps the CPU until it *voluntarily* gives it up — either by blocking (I/O, waiting on something) or by finishing. Even if it runs for hours, nobody forces it off. Clock interrupts happen, but they don't trigger a scheduling decision — after handling the interrupt, control just goes back to whoever was running.
- **Preemptive**: a process is given a maximum time slice. If it's still running when that slice ends, it gets **forcibly suspended** and the scheduler picks someone else (possibly the same process again, if nothing else is ready). This *requires* a hardware clock that interrupts at the end of each interval — without a clock, preemptive scheduling is impossible.

**Intuition:** nonpreemptive = "I trust you to be polite and hand back the mic." Preemptive = "I set a timer, and when it beeps, I take the mic back whether you like it or not."

---

## 4. Three Environments, Three Sets of Priorities

Different systems care about different things:

### Batch systems
No interactive users sitting at terminals. Jobs (payroll, insurance claims, bank interest calc) are submitted and run when convenient. Since nobody is impatiently waiting, **nonpreemptive** algorithms (or preemptive with *long* time slices) are fine — this reduces the number of process switches and improves overall efficiency.

### Interactive systems
Real users (or remote clients hitting a server) are waiting for responses *right now*. **Preemption is essential** — otherwise one buggy or greedy process could hog the CPU forever and starve everyone else.

### Real-time systems
Processes have **deadlines**. Interestingly, preemption is *sometimes not even necessary* here — because in a well-designed real-time system, every process is cooperative by construction (it knows it must do its work quickly and yield). This is the key difference from interactive systems: interactive systems must assume processes might be uncooperative, buggy, or even malicious; real-time systems only run trusted, purpose-built code.

---

## 5. Goals of a Scheduling Algorithm

*(This is literally Figure 2-40 in the book — good to memorize the categories)*

**All systems, always:**
- **Fairness** — comparable processes get comparable CPU share.
- **Policy enforcement** — whatever the stated policy is (e.g., "safety-control always wins"), the scheduler must actually enforce it.
- **Balance** — try to keep *all* parts of the system busy (CPU **and** I/O devices), not just the CPU. If you run all CPU-bound jobs first and all I/O-bound jobs after, you waste time — first the disk sits idle, then later the CPU sits idle. Mixing job types keeps everything busy simultaneously.

**Batch systems specifically:**
- **Throughput** — jobs completed per hour. Bigger is better.
- **Turnaround time** — average time from job submission to job completion. Smaller is better.
- **CPU utilization** — percentage of time CPU is busy. (Tanenbaum actually calls this a *weak* metric — it's like judging a car by how many times per hour the engine turns over, rather than by how far it actually gets you. Still useful as a signal for "do we need more hardware?")

> Important warning: maximizing throughput does **not** automatically minimize turnaround time. A scheduler that always runs short jobs first can get amazing throughput while long jobs starve forever, making *average* turnaround terrible even though throughput looks great.

**Interactive systems specifically:**
- **Response time** — time between issuing a command and getting a result. Minimize this.
- **Proportionality** — meeting the user's *psychological* expectation. Users expect big tasks (uploading a 500 MB file) to take a while, and get impatient if a "simple-looking" task (like disconnecting) takes long, even if the scheduler technically can't do anything about it.

**Real-time systems specifically:**
- **Meeting deadlines** — missing a deadline can mean lost data or worse.
- **Predictability** — especially for multimedia; occasional missed deadlines might be tolerable, but erratic/jittery behavior degrades audio/video quality. (Ears are more sensitive to jitter than eyes, interestingly.)

---

## 6. Batch Scheduling Algorithms

These assume no interactive users, so no urgency about response time — but turnaround and throughput matter.

### 6.1 First-Come, First-Served (FCFS / FIFO)

**The idea:** simplest possible algorithm. Maintain a single queue. Whoever arrives first, runs first, and runs to completion (or until it blocks) without interruption. New arrivals go to the tail of the queue.

- **Nonpreemptive.**
- **Real-life analogy:** standing in line at a bank counter (your Sonali Bank example from the slide) — first in line, first served, no cutting.

**Implementation:** trivially a FIFO linked list. Pick from head, insert at tail. `O(1)` operations both ways.

#### Worked Example 1 (all arrive at time 0):

| Process | Duration | Order |
|---|---|---|
| P1 | 24 | 1st |
| P2 | 3 | 2nd |
| P3 | 4 | 3rd |

Timeline: `P1 [0–24] → P2 [24–27] → P3 [27–31]`

Turnaround time = completion time − arrival time (arrival = 0 for all here, so turnaround = completion time):
- P1: 24, P2: 27, P3: 31
- **Average turnaround = (24+27+31)/3 = 27.33**

#### Worked Example 2 (same jobs, different arrival *order* in queue):

If instead P2 (duration 3) enters the queue first, then P3 (4), then P1 (24):

Timeline: `P2 [0–3] → P3 [3–7] → P1 [7–31]`

- P2: 3, P3: 7, P1: 31
- **Average turnaround = (3+7+31)/3 = 13.67**

**Notice:** same total work, same total completion time (31), but wildly different *average* turnaround depending on the *order*. This is the seed of the idea behind Shortest-Job-First — order matters a lot for average turnaround, even though it doesn't matter for total throughput.

**Advantages:** dead simple to understand and implement; feels "fair" in the naive sense (first come, first served, like a queue for concert tickets).

**Major problem — the Convoy Effect:**

Imagine one CPU-bound process (CPUB) and several I/O-bound processes (IOBs) in the ready queue.

1. CPUB runs for a long CPU burst while all the IOBs sit waiting in the ready queue — meanwhile, the **I/O devices sit idle** because none of the IOBs got a turn yet to issue their I/O requests.
2. Eventually CPUB finishes its burst and goes off to do its own I/O. Now all the IOBs get to run — but each one only needs a *tiny* CPU burst before immediately going back to wait for I/O again. They zip through the CPU very fast.
3. But this means the **CPU** now sits mostly **idle**, because everyone who's runnable is momentarily blocked on I/O, and CPUB is still off doing its own I/O.
4. The cycle repeats: CPU busy/I/O idle, then CPU idle/I/O busy, alternating — very poor overall utilization of *both* resources simultaneously.

This is called the **convoy effect**: a few slow-moving processes force everyone else to bunch up behind them, like slow trucks blocking a highway convoy. FCFS's rigid non-preemptive nature is the root cause — nothing can interrupt CPUB even briefly to let a quick I/O-bound job slip through.

---

### 6.2 Shortest Job First (SJF)

**The idea:** among jobs *currently waiting*, always run the one with the smallest total run time next. Nonpreemptive — once started, a job runs to completion.

**Requirement:** you must know (or estimate) each job's run time *in advance*. (Realistic in batch environments like an insurance company running the same kind of claim-processing job every day — historical averages are a decent predictor.)

#### Worked example (all arrive at t=0):

| Process | Duration | SJF Order |
|---|---|---|
| P4 | 3 | 1st |
| P1 | 6 | 2nd |
| P3 | 7 | 3rd |
| P2 | 8 | 4th |

Timeline: `P4[0–3] → P1[3–9] → P3[9–16] → P2[16–24]`
- Turnarounds: 3, 9, 16, 24 → **Average = 13**

Compare to FCFS on the *same* jobs in original arrival order (P1,P2,P3,P4):
Timeline: `P1[0–6] → P2[6–14] → P3[14–21] → P4[21–24]`
- Turnarounds: 6, 14, 21, 24 → **Average = 16.25**

**Total completion time is identical (24) in both cases** — SJF doesn't change total throughput at all. What it changes is the **average turnaround time**, by making short jobs finish quickly instead of waiting behind long ones.

#### Why is SJF mathematically optimal (when all jobs arrive together)?

Say four jobs have durations $a, b, c, d$ and you run them in that order. Then:
- Job 1 finishes at time $a$
- Job 2 finishes at time $a+b$
- Job 3 finishes at time $a+b+c$
- Job 4 finishes at time $a+b+c+d$

Mean turnaround $= \dfrac{4a + 3b + 2c + d}{4}$

Notice $a$ is multiplied by 4 (it "contributes" to every later job's wait too), $b$ by 3, $c$ by 2, $d$ by only 1. So to minimize the sum, put the **smallest** value where it gets multiplied the **most** — i.e., shortest job first, then next shortest, etc. This generalizes to any number of jobs. That's the proof sketch — nice one to remember for exams.

#### ⚠️ SJF is only optimal if all jobs are available (arrived) at the same time!

Counter-example from the book: Jobs A(2), B(4), C(1), D(1), E(1), with A,B arriving at t=0 and C,D,E arriving at t=3.

At t=0 only A or B can be chosen (others haven't arrived). If you greedily apply SJF at each decision point, you'd run order A,B,C,D,E → average wait 4.6. But running B,C,D,E,A gives a *better* average wait of 4.4 — because holding off on the short jobs C,D,E (which arrive slightly later) to run the long job B "for free" while nothing else is available turns out worse than a smarter ordering. Point: naive/greedy SJF can be **suboptimal** once arrival times differ.

---

### 6.3 Shortest Remaining Time Next (SRTN) — the preemptive version of SJF

**The idea:** same as SJF, but now **preemptive**. The scheduler always runs whichever ready process has the *least remaining* time to finish. When a **new** job arrives, compare its *total* time against the *remaining* time of the currently running job — if the newcomer needs less time, **preempt** the current job and switch to the new one immediately.

#### Worked example:

| Process | Duration | Arrival |
|---|---|---|
| P1 | 10 | 0 |
| P2 | 2 | 2 |

- t=0: only P1 is ready → run P1.
- t=2: P2 arrives needing only 2, but P1 has 8 remaining → P2 wins → **preempt P1**, run P2.
- t=4: P2 finishes (it only needed 2). Resume P1 with its remaining 8.
- t=12: P1 finishes.

Turnarounds: P1 = 12 − 0 = 12, P2 = 4 − 2 = 2 → **Average = (12+2)/2 = 7**

(Compare: if this had been plain non-preemptive SJF/FCFS running P1 fully first, P1=10, P2 = 10 (it has to wait until P1 finishes at 10, then runs 2 more, finishing at 12, so turnaround = 12−2=10) → average = 10. So SRTN's 7 is clearly better here.)

#### ⚠️ Problem: Starvation

If process A needs 1 hour of CPU time, but a fresh 1-minute job keeps arriving *every single minute forever*, A will **never** get to run — it's always "not the shortest" compared to the freshly-arrived tiny job. This is called **starvation**: a process is technically still runnable, but it never actually gets its turn, essentially forever.

---

## 7. Interactive Scheduling Algorithms

Interactive systems (including servers handling many remote users) care most about **response time**. Algorithms here are almost always **preemptive**, and time is chopped into fixed intervals called **quanta**. A scheduling decision can be made at the start of every quantum.

### 7.1 Round Robin (RR)

**The idea:** the simplest fair preemptive algorithm. Every process gets a fixed time slice (the **quantum**). It runs until either (a) the quantum expires, or (b) it blocks/finishes on its own — whichever comes first. If the quantum expires while it's still running, it's preempted and sent to the **back of the ready queue**.

**Implementation:** ready queue = simple FIFO list.
- Scheduler pops the process at the head, sets a timer for one quantum, runs it.
- If the quantum expires and the process is still running: move it to the tail of the queue, pick the new head.

#### Worked example — quantum = 20, all arrive at t=0:

| Process | Run time |
|---|---|
| P1 | 53 |
| P2 | 17 |
| P3 | 68 |
| P4 | 24 |

Gantt chart:
```
P1  P2  P3  P4  P1  P3  P4  P1  P3  P3
0   20  37  57  77  97  117 121 134 154 162
```
Walking through it: P1 runs 20 (33 left), P2 runs its full 17 (done, only needed 17 < quantum), P3 runs 20 (48 left), P4 runs 20 (4 left), P1 runs its remaining 33... wait — actually P1 gets another 20 here (13 left), P3 runs 20 more (28 left), P4 finishes its remaining 4, P1 finishes its remaining 13, P3 runs 20 more (8 left), P3 finishes its remaining 8.

**Note on RR's trade-off:** Round Robin generally gives a *higher average turnaround time* than SJF (because it doesn't prioritize short jobs), but it gives dramatically **better response time** — nobody waits an unbounded amount of time before getting *some* CPU attention, unlike FCFS where a job stuck behind a huge job just waits and waits.

### 7.2 Choosing the Quantum Size — a genuinely important design question

This is the *one* subtle, interesting design decision in RR (per the book, verbatim sentiment).

- **Quantum too short** → too many context switches → wasted CPU time on overhead. Concretely: if a context switch costs 1 ms and quantum = 4 ms, then 1 out of every 5 ms (20%!) is *pure overhead*, not useful work.
- **Quantum too long** → RR degenerates toward FCFS-like behavior (each process just finishes or blocks before the quantum even matters) → but if the quantum is long and *lots* of processes are waiting, some unlucky process near the back of a 50-process queue might wait several **seconds** before its first turn → feels sluggish to interactive users, especially bad if that unlucky process only needed a few ms of CPU anyway.

**Rule of thumb (memorize this number, it shows up in questions):** a quantum around **20–50 ms** is usually a reasonable compromise. Also: if the quantum is set *longer* than the typical CPU burst length of most processes, most processes will block naturally before the quantum runs out anyway — so genuine preemption (forced switch mid-burst) becomes rare, and process switches only happen when logically necessary (a process blocking), which is actually *good* for efficiency.

### 7.3 Priority Scheduling

**The idea:** RR implicitly assumes every process is equally important — often false. Instead, assign each process a **priority**; always run the highest-priority *runnable* process.

**Problem:** low-priority processes can **starve** forever if high-priority ones keep showing up.
**Fix:** decrease a running process's priority over time (e.g., at every clock tick), so eventually even a high-priority process's priority drops enough that something else gets a turn. Or: give each process a maximum quantum regardless of priority.

#### How are priorities assigned?

- **Static** — fixed in advance. Works when application behavior is well-known/predictable (e.g., a strict military hierarchy: general=100, colonel=90, etc., or paying more money for higher priority: $100/hr = high priority, $75/hr = medium, etc.)
- **Dynamic** — computed and adjusted by the system on the fly, based on current behavior. This is more common/flexible for general-purpose systems.

**A concrete dynamic-priority formula (from the book, worth memorizing):**
$$\text{priority} = \frac{1}{f}$$
where $f$ = fraction of the *last* quantum actually used by the process before it blocked.

- Used only 1 ms of a 50 ms quantum before blocking (very I/O-bound behavior) → $f = 1/50$ → priority $= 50$ (**high** priority).
- Used 25 ms of a 50 ms quantum → $f = 1/2$ → priority $= 2$ (**lower** priority).
- Used the *entire* quantum (very CPU-bound) → $f=1$ → priority $=1$ (**lowest**).

**Why reward I/O-bound processes with higher priority?** Because an I/O-bound process that gets the CPU quickly can immediately fire off its next I/O request, letting the I/O device start working *in parallel* with some other process's computation. Making it wait pointlessly delays the I/O device from getting started, wasting overall system throughput. This connects directly back to the "Balance" scheduling goal from Section 5.

### 7.4 Priority Classes (grouping priorities)

Rather than a huge number of individual priority levels, it's often more practical to group processes into a handful of **priority classes**, and:
- Use **strict priority** *between* classes (never touch class 3 while anything in class 4 is runnable).
- Use **Round Robin** *within* each class.

Typical ordering (highest → lowest), as shown in the slides:
```
System processes  >  Interactive processes  >  Interactive editing  >  Batch processes  >  Student processes
```

**Danger:** if priorities are never adjusted, a lower class can **starve completely** — e.g., "student processes" might simply never run if there's always something in a higher class ready.

### 7.5 Multiple Queues (CTSS — historical but instructive)

One of the earliest priority schedulers, from MIT's CTSS system on the IBM 7094. Context: process switching was *very* expensive back then (had to swap the entire process to/from disk, since only one process fit in memory at a time).

**Idea:** give CPU-bound processes progressively **larger** quanta over time instead of many small ones (to reduce the number of expensive swaps):
- 1st time a process runs: quantum = 1.
- If it uses the whole quantum and needs more: next time it gets quantum = 2.
- Then 4, then 8, then 16, then 32, then 64...
- Whenever a process fully consumes its allotted quanta, it drops down one priority class (gets bigger quanta, but runs less *frequently*).

**Worked example (matches slide Q46):** a process needing 30 quanta total to finish.
- Run 1: gets 1 quantum (1 used, 29 remain, swapped out)
- Run 2: gets 2 (3 used total, 27 remain)
- Run 3: gets 4 (7 used, 23 remain)
- Run 4: gets 8 (15 used, 15 remain)
- Run 5: gets 16, but only 15 are actually needed → **finishes partway through this run**.

So it needed **5 swaps total** (including the very first load) — compare to needing 30 swaps under plain Round-Robin with quantum=1! Massive reduction in expensive process-switch overhead.

**A cute real gotcha from CTSS (good "fun fact" for understanding *why* real-world scheduling is hard):** CTSS also had a rule that pressing Enter/carriage-return bumped a process back up to the highest-priority class (assuming the user was about to interact). Some clever user discovered that if his CPU-bound job just pressed Enter randomly every few seconds, it kept getting bumped to high priority and got great response time — completely gaming the scheduler. **Moral: getting scheduling right in *practice* is much harder than getting it right in *theory*.**

### 7.6 Shortest Process Next

Same spirit as SJF, but applied to *interactive* command execution instead of batch jobs. Interactive use tends to follow: `wait for command → execute → wait for command → execute...`. If you treat each command execution as a "job," you can try to run the shortest-estimated one next.

**Problem:** you don't know the future — how do you estimate how long the *next* command will take?

**Solution: Aging** — predict the next run time using a weighted average of the previous estimate and the most recently observed actual time:
$$\text{new estimate} = a \cdot T_{old} + (1-a) \cdot T_{measured}$$

With $a = 1/2$, this is especially easy to compute: just add old estimate + new measurement and divide by 2 (a single right bit-shift in binary!). Choosing $a$ controls memory: values closer to 1 "remember" old history longer; values closer to 0 forget old runs fast and react quickly to recent behavior. After a few updates, the weight of very old measurements decays exponentially — e.g., after 3 updates with $a=1/2$, the original $T_0$'s influence has dropped to $1/8$.

### 7.7 Guaranteed Scheduling

**The idea:** make an explicit, honest promise to users, and then keep it. E.g.: "if $n$ users are logged in, you'll get roughly $1/n$ of the CPU."

**Mechanism:** track, per process, (a) time since creation, (b) actual CPU time received so far. Compute:
$$\text{ratio} = \frac{\text{actual CPU time received}}{\text{CPU time entitled to } (= \text{time since creation} / n)}$$

- ratio $< 1$ → process has gotten *less* than its fair share.
- ratio $> 1$ → process has gotten *more* than fair share.

**Rule:** always run whichever ready process currently has the **lowest** ratio, until its ratio rises above its closest competitor's — then switch.

### 7.8 Lottery Scheduling

A cleverly simple way to achieve *similar* fairness guarantees to Guaranteed Scheduling, without the bookkeeping complexity.

**The idea:** give each process some number of "lottery tickets." At every scheduling decision, draw a ticket at random; whoever holds it gets to run (e.g., "hold a lottery 50 times a second, winner gets 20 ms of CPU").

- More important processes simply get **more tickets** → proportionally higher chance of winning.
- If there are 100 total tickets and a process holds 20 of them, it has a 20% chance each lottery → in the long run, it gets **about 20%** of total CPU time. Clean, intuitive, and *statistically* self-correcting.

**Nice properties:**
- **Highly responsive**: a brand-new process, as soon as it's granted tickets, has a fair shot at winning at the *very next* lottery — no ramp-up delay.
- **Ticket exchange between cooperating processes**: e.g., a client sends a request to a server and then blocks — it can temporarily *give all its tickets to the server*, boosting the server's chance of running next (so it finishes the client's request faster). The server returns the tickets afterward.
- **Solves proportional-rate problems elegantly**: e.g., three video-streaming processes needing 10, 20, and 25 frames/sec respectively — just give them 10, 20, and 25 tickets. The CPU then naturally divides itself in roughly the ratio 10:20:25, automatically.

### 7.9 Fair-Share Scheduling

**Problem it solves:** ordinary per-*process* fairness can still be unfair per-*user*. If user 1 starts 9 processes and user 2 starts only 1, plain Round Robin gives user 1 about 90% of the CPU (9 processes × equal shares) and user 2 only 10% — even though intuitively both users "deserve" 50/50.

**The idea:** track ownership by *user*, and enforce fairness at the *user* level, not the process level. If user 1 (4 processes: A,B,C,D) and user 2 (1 process: E) are each promised 50% of the CPU:

- If truly 50/50: schedule so E gets picked as often as *all four* of user 1's processes combined, e.g.:
  `A E B E C E D E A E B E C E D E ...`
- If user 1 is promised **twice** as much CPU as user 2 instead:
  `A B E C D E A B E C D E ...`

The exact interleaving can vary, but the *constraint being enforced* is per-user share, not per-process share.

---

## 8. Real-Time Scheduling

A **real-time system** is one where *correctness depends on timing*, not just on getting the right answer. Getting the right answer *too late* is often as bad as not getting it at all (e.g., a CD player's audio decoder, a hospital patient monitor, an aircraft autopilot, factory robot control).

### Hard vs. Soft real-time

- **Hard real-time**: deadlines are absolute. Missing one is a system failure (imagine an airbag controller).
- **Soft real-time**: missing an occasional deadline is undesirable but tolerable (imagine a video player dropping one frame occasionally — annoying, not catastrophic).

### Periodic vs. Aperiodic events

- **Periodic**: events that occur at regular, predictable intervals (e.g., a sensor reading every 100 ms).
- **Aperiodic**: events that occur unpredictably (e.g., a user pressing an emergency stop button).

### Schedulability test

Given $m$ periodic events, where event $i$ occurs with period $P_i$ and requires $C_i$ seconds of CPU time to service each time it occurs, the system is **schedulable** (i.e., it is *possible* in principle to meet all deadlines) only if:

$$\sum_{i=1}^{m} \frac{C_i}{P_i} \leq 1$$

Intuition: $C_i / P_i$ is "the fraction of CPU time this one event stream demands, on average." If the *sum* of all such demanded fractions exceeds 1 (i.e., you'd need more than 100% of the CPU), it's mathematically impossible to satisfy everyone — no scheduling algorithm, however clever, can save you. This assumes negligible context-switch overhead.

#### Worked example (from the slide/book):
Three periodic events: periods 100, 200, 500 ms; CPU needs 50, 30, 100 ms respectively.
$$\frac{50}{100} + \frac{30}{200} + \frac{100}{500} = 0.5 + 0.15 + 0.2 = 0.85 \leq 1$$
✅ **Schedulable.**

Now add a 4th event with period 1000 ms (1 sec). It remains schedulable as long as its own $C/P$ doesn't push the total over 1:
$$1 - 0.85 = 0.15 \Rightarrow C_4 \leq 0.15 \times 1000 = 150 \text{ ms}$$
So the new event can demand **up to 150 ms** of CPU time per occurrence and the system stays schedulable.

### Static vs. Dynamic real-time scheduling

- **Static**: all scheduling decisions are made *before* the system starts running (requires perfect advance knowledge of every job and every deadline).
- **Dynamic**: scheduling decisions are made *at run time*, adapting as things happen. More flexible, doesn't require perfect foreknowledge.

---

## 9. Policy vs. Mechanism (a design philosophy, brief but important)

Normally schedulers treat all processes as independent competitors. But sometimes one process has many *children* it understands intimately (e.g., a database server spawning child processes to handle different queries) — and *it* knows better than the OS which of its children matter most right now.

**The fix:** separate the **scheduling mechanism** (the actual machinery in the kernel that picks who runs) from the **scheduling policy** (the parameters/priorities that decide *how* it picks). Concretely: the kernel might run a plain priority scheduler (mechanism), but expose a system call letting a parent process *set the priorities of its own children* (policy). The kernel still does the mechanical work, but an informed user-level process supplies the intelligence about what matters.

This is a recurring theme in OS design generally — separating "the engine" from "the steering," so the engine stays simple and general while intelligence can live at a higher, more informed layer.

---

## 10. Thread Scheduling

Recall (from your earlier threads material): threads within a process share an address space but have their own registers/stack. When a system has multiple *processes*, each with multiple *threads*, there are now **two levels of scheduling decision**: which process, and which thread within it. This differs a lot depending on whether threads are implemented at **user level** or **kernel level**.

### 10.1 User-level threads

The **kernel doesn't know threads exist at all** — as far as the kernel is concerned, it just sees one process. The kernel picks a *process* to run (using whatever process-scheduling algorithm), and then a **thread scheduler living inside that process** (part of the runtime library) decides which of *its own* threads gets to run during that process's turn.

**Consequences:**
- Thread switching is **extremely cheap** — just a few machine instructions, since there's no trap into the kernel, no memory-map reload, no cache flush.
- **No preemption between threads** unless the thread library builds it itself, since there's no clock interrupt dedicated to threads — a thread can hog the whole process's CPU time if it never voluntarily yields.
- Example: 50 ms process quantum, threads that run ~5 ms per burst before yielding → possible order: `A1, A2, A3, A1, A2, A3, ...` (all interleaving happens *within* process A's turns). **Not possible**: `A1, B1, A2, B2, ...` — because the kernel isn't scheduling individual threads, it schedules whole processes, so you never see two *different processes'* threads interleaved thread-by-thread; process A gets its full quantum before B ever gets a look-in.
- **Big advantage:** the application can use an **application-specific thread scheduler**, tuned to its own logic. E.g., a web server has a "dispatcher" thread and several "worker" threads; when a worker blocks on disk I/O, the runtime — knowing exactly what each thread does — can intelligently pick the dispatcher to run next (so it can spin up another worker), maximizing parallelism. The *kernel* could never make this smart a decision because it has no idea what each thread's *purpose* is.

### 10.2 Kernel-level threads

The **kernel is directly aware of every thread** and schedules them individually — it doesn't have to treat "the process" as an atomic scheduling unit at all (though it *can* take process-membership into account if it wants to).

**Consequences:**
- Thread switches are **expensive** — a full context switch (memory map change, cache invalidation), just like switching between processes.
- **Threads can be forcibly preempted mid-burst**, since the kernel has a proper clock-driven mechanism for threads, not just processes.
- Interleaving across *different* processes' threads is now possible: `A1, B1, A2, B2, A3, B3` — something that could never happen with user-level threads.
- **Upside kernel-level threads have over user-level:** if one thread in process A blocks on I/O, *only that thread* blocks — the kernel can freely schedule another ready thread (even from the same process) in the meantime. With user-level threads, if the thread scheduler isn't careful (or if the blocking call is a genuine kernel-level blocking syscall), the *entire process* — and hence *all* its threads — can end up blocked, because the kernel only sees "one process," not the individual thread that's actually stuck.
- The kernel can also make *smarter* trade-off decisions using its knowledge of switch cost: e.g., given two equally-important ready threads, one belonging to the same process as the thread that just blocked, and one belonging to a different process, the kernel might prefer the *same-process* one — because switching within a process avoids the expensive memory-map reload and cache flush that switching to a different process would cause.

### Quick comparison table

| | User-level threads | Kernel-level threads |
|---|---|---|
| Kernel aware of threads? | No | Yes |
| Thread switch cost | Very cheap (few instructions) | Expensive (full context switch) |
| Can preempt a runaway thread? | No (cooperative only, unless library adds its own timer) | Yes |
| One thread blocking on I/O blocks the whole process? | Yes (typically) | No — only that thread blocks |
| Cross-process thread interleaving possible? | No | Yes |
| Can use app-specific scheduling logic? | Yes, easily | Harder — kernel doesn't know app semantics |

---

## 11. Quick-Reference Summary Table (all algorithms)

| Algorithm | Preemptive? | Environment | Needs advance knowledge of run time? | Main strength | Main weakness |
|---|---|---|---|---|---|
| FCFS | No | Batch | No | Trivial to implement, "fair" in naive sense | Convoy effect; poor avg. turnaround |
| SJF | No | Batch | Yes | Provably minimal avg. turnaround (if all arrive together) | Needs foreknowledge; unfair to long jobs; not optimal if arrivals staggered |
| SRTN | Yes | Batch | Yes | Even better avg. turnaround than SJF | Starvation possible for long jobs |
| Round Robin | Yes | Interactive | No | Great response time, simple, fair | Ignores importance/priority; quantum tuning is tricky |
| Priority Scheduling | Usually yes | Interactive/general | No (but priority must be assigned) | Reflects real importance | Starvation of low priority unless aged |
| Multiple Queues (CTSS-style) | Yes | Interactive/batch hybrid | No | Cuts down expensive swaps for long CPU-bound jobs | Can be gamed (carriage-return trick!) |
| Shortest Process Next | Not inherently | Interactive | Estimated via aging | Approximates SJF benefits interactively | Estimation can be wrong |
| Guaranteed Scheduling | Yes | Interactive | No | Explicit, provable fairness promise | Bookkeeping overhead |
| Lottery Scheduling | Yes | Interactive | No | Simple, statistically fair, flexible via ticket exchange | Randomness → short-term variance |
| Fair-Share | Yes | Interactive, multi-user | No | Fair *per user*, not just per process | More bookkeeping |
| Real-time (deadline-based) | Either | Real-time | Yes (periods & CPU needs) | Meets hard guarantees if schedulable | Not schedulable if demand > 100% CPU |

---

## 12. Common Exam-Style Pitfalls to Watch For

1. **"Turnaround time" vs "waiting time" vs "response time"** — these are different! Turnaround = completion − arrival. Waiting time = turnaround − actual run time (i.e., time spent *not* running while ready). Response time (interactive) = time until first response, not full completion.
2. When computing FCFS/SJF/SRTN by hand, **always double-check whether all jobs arrive at t=0** — if not, you cannot apply "always pick shortest job" blindly; you must respect arrival order and only choose among what has *actually arrived*.
3. Convoy effect explanation: don't just say "FCFS is bad" — explain *why*: nonpreemptive + one long job blocking many short ones → alternating idle CPU / idle I/O device pattern.
4. Schedulability formula $\sum C_i/P_i \le 1$ — remember it's a **necessary** condition for schedulability, not always sufficient by itself in more complex real systems, but it's the standard test used at this course level.
5. Distinguish clearly: **preemptive vs nonpreemptive** is about *forcing* a process off the CPU at a timer tick. Don't confuse this with "does the scheduler ever switch processes" (it always can, at blocking/exit/new-process events, even under nonpreemptive scheduling — nonpreemptive just means *the clock alone* can't trigger a switch).
6. User-level vs kernel-level thread scheduling is a favorite short-answer/labeling question — the key discriminator is always: **does the kernel know threads exist as separate schedulable entities?**

---

*Good luck with your exam prep, bhai. If you want, I can also generate practice numericals (Gantt-chart style, similar to the P1–P4 examples above) with worked solutions so you can self-test before the exam.*
