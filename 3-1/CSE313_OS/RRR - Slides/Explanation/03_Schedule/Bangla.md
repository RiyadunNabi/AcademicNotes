# CPU Scheduling — সম্পূর্ণ Study Guide (বাংলা ব্যাখ্যা সহ)
### (Tanenbaum Ch. 2.4 + তোমার Teacher-এর Slide অনুযায়ী)

---

## 0. Scheduling কেন লাগে?

ধরো তোমার একটাই CPU আছে, কিন্তু অনেকগুলো process "ready" অবস্থায় আছে (মানে কোনো I/O-র জন্য wait করছে না, শুধু CPU পাওয়ার অপেক্ষায় বসে আছে)। তাহলে কেউ একজনকে decide করতে হবে — **কোনটা এখন চলবে?**

এই decision যে নেয়, তাকে বলে **scheduler**। আর যে rule/algorithm দিয়ে ও decide করে, সেটাই **scheduling algorithm**।

এটা কেন matter করে? কারণ এক process থেকে আরেক process-এ switch করাটা **free না**, এতে সময় লাগে। একে বলে **process switch (context switch)**। এতে যা যা হয়:

1. User mode থেকে kernel mode-এ trap করতে হয়।
2. Current process-এর state (registers ইত্যাদি) save করতে হয় process table-এ।
3. কখনো কখনো memory map (page table info)-ও save করতে হয়।
4. Scheduling algorithm চালিয়ে পরবর্তী process বাছাই করতে হয়।
5. MMU (Memory Management Unit)-কে নতুন process-এর memory map দিয়ে reload করতে হয়।
6. নতুন process start করতে হয়।
7. এক্সট্রা ঝামেলা: প্রায়ই CPU cache invalidate হয়ে যায়, তাই নতুন process প্রথমে একটু slow চলে, কারণ সবকিছু আবার RAM থেকে fetch করতে হয়।

তাহলে বুঝলে — বেশি বেশি switch করলে CPU-র সময় "administration"-এ চলে যায়, actual কাজ কম হয়। আবার কম switch করলে system slow feel হয় (responsiveness কমে)। Scheduling মানে হলো এই balance রক্ষা করা।

**Practically কেন matter করে:** ভালো scheduler থাকলে CPU utilization ভালো হয়, system ব্যবহার করতে "snappy" লাগে, আর জরুরি কাজ (যেমন একটা nuclear reactor-এর safety monitor) সময়মতো attention পায়।

---

## 1. Process Behavior: CPU Burst আর I/O Burst

প্রায় সব process দুইটা phase-এর মধ্যে alternate করে:

- **CPU burst**: এই সময়টায় process আসলেই compute করছে (CPU ব্যবহার করছে), কোনো I/O নেই।
- **I/O burst**: এই সময়টায় process blocked, কোনো external device (disk read, network response ইত্যাদি) থেকে response-এর অপেক্ষায় আছে।

একটা process-এর জীবন এইরকম: `compute → I/O wait → compute → I/O wait → ...`

> **সূক্ষ্ম ব্যাপার:** CPU যদি screen update করার জন্য video RAM-এ pixel copy করে, সেটাও compute — I/O না। কারণ CPU তখনও active কাজ করছে। "I/O" মানে specifically হলো — process **blocked**, external device-এর জন্য অপেক্ষা করছে।

### CPU-bound vs I/O-bound processes

| Type | Behavior | উদাহরণ |
|---|---|---|
| **CPU-bound** (compute-bound) | Long CPU burst, কম কম I/O wait | Video encoding, encryption, matrix multiplication |
| **I/O-bound** | Short CPU burst, ঘন ঘন I/O wait | Shell যেটা তোমার command-এর অপেক্ষায় আছে, text editor |

**Key insight (এটা exam-এ আসার মতো point, মনে রাখবে):** কোনো process "I/O-bound" হওয়ার মানে এই না যে তার I/O করতে বেশি সময় লাগে। এর মানে হলো — process টা তার দুই I/O request-এর মাঝে **খুবই কম compute** করে। Disk read করতে 1 ms লাগুক বা 100 ms লাগুক, request পাঠাতে CPU-র লাগে একই পরিমাণ (tiny) সময়। I/O-bound process মূলত I/O-bound এই কারণে যে সে বারবার তাড়াতাড়ি I/O চায়, তার I/O নিজেই দীর্ঘ বলে না।

**একটা trend মনে রাখার মতো:** CPU যত fast হচ্ছে (disk-এর তুলনায়), তত বেশি process ধীরে ধীরে *effectively I/O-bound* হয়ে যাচ্ছে। এই কারণেই I/O-bound process-কে ভালোভাবে schedule করা (মানে তাড়াতাড়ি CPU দিয়ে দেওয়া, যেন সে দ্রুত disk/network request পাঠিয়ে দিতে পারে) ভবিষ্যতে আরও গুরুত্বপূর্ণ হবে।

---

## 2. Scheduler কখন call হয়?

মোটামুটি ৪-৫টা natural trigger point আছে:

1. **নতুন process তৈরি হলে** — এখন parent আর child দুইটাই "ready"। কে আগে চলবে — parent নাকি child? দুইটাই valid choice।
2. **Process exit করলে** — সেটা তো আর নাই, তাই ready queue থেকে অন্য কাউকে বাছতে হবে (কেউ ready না থাকলে idle process চলে)।
3. **Process block করলে** — যেমন `read()` call করে disk-এর জন্য wait করছে, বা কোনো semaphore-এর জন্য wait করছে। তখন সে এখন চলতে পারবে না, তাই অন্য কাউকে বাছতে হবে।
4. **I/O interrupt আসলে** — কোনো device তার কাজ শেষ করে ফেললে, যে process সেই device-এর জন্য blocked ছিল, সে হয়তো এখন ready হয়ে গেছে। Scheduler এখন decide করবে: current process-কেই চালিয়ে যাবে, নাকি নতুন ready হওয়া process-কে চালাবে, নাকি অন্য কাউকে।
5. **Clock/timer interrupt আসলে** (যদি hardware-এ periodic clock থাকে) — এটাই **preemption**-কে সম্ভব করে (নিচে দেখো)। প্রতি tick-এ বা প্রতি k-তম tick-এ scheduler invoke করা যায়।

---

## 3. Preemptive vs Nonpreemptive Scheduling

এই classification-টা মূলত clock interrupt-এ **কী ঘটে** তার ওপর ভিত্তি করে।

- **Nonpreemptive**: একবার কোনো process চলা শুরু করলে, সে **নিজে থেকে** CPU না ছাড়া পর্যন্ত সে-ই চলতে থাকে — হয় block হয়ে (I/O, কিছুর জন্য wait) নয় শেষ হয়ে। কয়েক ঘণ্টা চললেও কেউ ওকে জোর করে থামায় না। Clock interrupt আসতে থাকে ঠিকই, কিন্তু সেটা কোনো scheduling decision trigger করে না — interrupt handle হওয়ার পর যে চলছিল সে-ই আবার চলতে থাকে।
- **Preemptive**: প্রতিটা process-কে একটা maximum time slice দেওয়া হয়। সেই সময় শেষ হয়ে গেলেও process যদি চলতেই থাকে, তাকে **জোর করে suspend** করা হয় এবং scheduler অন্য কাউকে বাছে (কিছু ready না থাকলে হয়তো একই process আবার)। এর জন্য অবশ্যই একটা hardware clock দরকার যেটা প্রতি interval শেষে interrupt দেয় — clock না থাকলে preemptive scheduling সম্ভবই না।

**Intuition:** Nonpreemptive = "আমি তোমাকে বিশ্বাস করি, তুমি নিজে থেকে mic ছেড়ে দিবা।" Preemptive = "আমি একটা timer সেট করে রাখলাম, বেজে উঠলেই mic কেড়ে নিব, তুমি চাও বা না চাও।"

---

## 4. তিনটা Environment, তিন রকমের Priority

Different system-এ different জিনিস matter করে:

### Batch systems
কোনো interactive user terminal-এ বসে অপেক্ষা করছে না। Job (payroll, insurance claim, bank interest calculation) submit করা হয় আর সুবিধামতো সময়ে চলে। যেহেতু কেউ অধৈর্য হয়ে বসে নেই, তাই **nonpreemptive** algorithm (বা লম্বা time slice-এর preemptive) ঠিকই আছে — এতে process switch কমে, overall efficiency বাড়ে।

### Interactive systems
Real user (বা remote client) *এখনই* response চায়। এখানে **preemption must**, নাহলে একটা greedy বা buggy process পুরো CPU দখল করে বাকি সবাইকে starve করে ফেলতে পারে।

### Real-time systems
Process-গুলোর **deadline** আছে। মজার ব্যাপার হলো — এখানে অনেক সময় preemption-এর দরকারই পড়ে না, কারণ real-time system-এ সব process ডিজাইন করাই থাকে cooperative হিসেবে (নিজেরাই দ্রুত কাজ শেষ করে CPU ছেড়ে দেয়)। Interactive system থেকে এইটাই মূল পার্থক্য: interactive system-কে ধরে নিতে হয় process uncooperative, buggy, এমনকি malicious-ও হতে পারে; real-time system-এ শুধু trusted, purpose-built code-ই চলে।

---

## 5. Scheduling Algorithm-এর Goals

*(এটা বইয়ের Figure 2-40, ক্যাটাগরি অনুযায়ী মনে রাখা ভালো)*

**সব system-এ, সবসময়:**
- **Fairness** — একরকম process-দের একরকম CPU share দিতে হবে।
- **Policy enforcement** — যা policy বলা আছে (যেমন "safety-control সবসময় আগে"), scheduler-কে সেটা মেনে চলতে হবে।
- **Balance** — চেষ্টা করতে হবে system-এর **সব অংশ** (CPU **এবং** I/O device) ব্যস্ত রাখতে, শুধু CPU না। সব CPU-bound job আগে চালিয়ে পরে সব I/O-bound job চালালে সময় নষ্ট হয় — প্রথমে disk বসে থাকে, পরে CPU বসে থাকে। বিভিন্ন type job মিশিয়ে চালালে একসাথে সবকিছু ব্যস্ত রাখা যায়।

**শুধু Batch system-এর জন্য:**
- **Throughput** — প্রতি ঘণ্টায় কয়টা job শেষ হলো। বেশি ভালো।
- **Turnaround time** — job submit করা থেকে শেষ হওয়া পর্যন্ত গড় সময়। কম ভালো।
- **CPU utilization** — CPU কতক্ষণ busy ছিল, percentage-এ। (Tanenbaum আসলে বলছে এটা একটা *দুর্বল* metric — এটা অনেকটা গাড়ির engine ঘণ্টায় কতবার ঘুরল সেটা দিয়ে গাড়ির quality বিচার করার মতো। তবুও এটা useful signal — "আমাদের কি আরও hardware দরকার?" বোঝার জন্য।)

> Important warning: throughput maximize করলেই যে turnaround time কম হবে, তা না। এমন scheduler যেটা সবসময় short job আগে চালায়, সে amazing throughput পেতে পারে, কিন্তু long job-গুলো forever starve করতে পারে — ফলে *average* turnaround খুবই খারাপ হবে, throughput ভালো দেখালেও।

**শুধু Interactive system-এর জন্য:**
- **Response time** — command দেওয়া থেকে result পাওয়া পর্যন্ত সময়। এটা কমাতে হবে।
- **Proportionality** — user-এর *psychological* expectation পূরণ করা। User আশা করে বড় কাজ (500 MB file upload) সময় নিবে, আর "সহজ" মনে হওয়া কাজ (যেমন disconnect করা) দেরি হলে বিরক্ত হয়, যদিও scheduler হয়তো সেখানে কিছুই করতে পারে না।

**শুধু Real-time system-এর জন্য:**
- **Meeting deadlines** — deadline miss হলে data loss বা তার চেয়ে খারাপ কিছু হতে পারে।
- **Predictability** — বিশেষত multimedia-তে; মাঝে মাঝে deadline miss হলে হয়তো সহ্য করা যায়, কিন্তু erratic/jittery behavior audio/video quality নষ্ট করে দেয়। (মজার ব্যাপার — কান চোখের চেয়ে jitter-এ বেশি sensitive।)

---

## 6. Batch Scheduling Algorithms

এখানে ধরে নেওয়া হয় কোনো interactive user নেই, তাই response time নিয়ে urgency নেই — কিন্তু turnaround আর throughput matter করে।

### 6.1 First-Come, First-Served (FCFS / FIFO)

**Idea:** সবচেয়ে simple algorithm। একটাই queue। যে আগে এসেছে, সে আগে চলবে, এবং শেষ পর্যন্ত চলবে (বা block হওয়া পর্যন্ত), মাঝে interrupt হবে না। নতুন যারা আসে, তারা queue-এর tail-এ যায়।

- **Nonpreemptive।**
- **Real-life analogy:** Sonali Bank-এ line-এ দাঁড়ানোর মতো — যে আগে এসেছে, তার আগে service, কেউ line কাটতে পারবে না।

**Implementation:** সাধারণ FIFO linked list। Head থেকে pick, tail-এ insert। দুটোই `O(1)`।

#### Worked Example 1 (সবাই t=0 তে আসে):

| Process | Duration | Order |
|---|---|---|
| P1 | 24 | 1st |
| P2 | 3 | 2nd |
| P3 | 4 | 3rd |

Timeline: `P1 [0–24] → P2 [24–27] → P3 [27–31]`

Turnaround time = completion time − arrival time (এখানে arrival সবার 0, তাই turnaround = completion time):
- P1: 24, P2: 27, P3: 31
- **Average turnaround = (24+27+31)/3 = 27.33**

#### Worked Example 2 (একই job, কিন্তু queue-এ ঢোকার *order* আলাদা):

যদি P2 (duration 3) আগে queue-এ ঢোকে, তারপর P3 (4), তারপর P1 (24):

Timeline: `P2 [0–3] → P3 [3–7] → P1 [7–31]`

- P2: 3, P3: 7, P1: 31
- **Average turnaround = (3+7+31)/3 = 13.67**

**খেয়াল করো:** একই total কাজ, একই total completion time (31), কিন্তু *order* অনুযায়ী average turnaround অনেক আলাদা। এখান থেকেই Shortest-Job-First-এর idea আসে — order-টা average turnaround-এর জন্য অনেক matter করে, যদিও total throughput-এর জন্য করে না।

**সুবিধা:** বুঝতে ও implement করতে খুবই সহজ; naive অর্থে "fair" মনে হয় (concert ticket-এর line-এর মতো, প্রথমে আসো প্রথমে পাও)।

**বড় সমস্যা — Convoy Effect:**

ধরো একটা CPU-bound process (CPUB) আছে আর অনেকগুলো I/O-bound process (IOB) আছে ready queue-এ।

1. CPUB লম্বা সময় ধরে CPU burst চালায়, এই সময়ে সব IOB ready queue-এ বসে থাকে — এদিকে **I/O device গুলা idle বসে থাকে**, কারণ কোনো IOB-ই এখনো turn পায়নি তাদের I/O request পাঠানোর জন্য।
2. অবশেষে CPUB তার burst শেষ করে নিজের I/O করতে চলে যায়। এখন সব IOB চলার সুযোগ পায় — কিন্তু প্রত্যেকের খুবই ছোট CPU burst দরকার, তারপরই তারা আবার I/O-র জন্য wait করতে চলে যায়। তারা CPU-তে খুব দ্রুত ঘুরে যায়।
3. কিন্তু এর মানে **CPU** এখন প্রায় **idle** হয়ে যায়, কারণ সবাই যারা runnable ছিল তারা মুহূর্তের মধ্যেই I/O-তে blocked, আর CPUB এখনও তার নিজের I/O করছে।
4. Cycle বারবার repeat হয়: CPU busy/I/O idle, তারপর CPU idle/I/O busy — দুইটা resource একসাথে ভালোভাবে ব্যবহার হয় না।

একে বলে **convoy effect**: কয়েকটা slow process বাকি সবাইকে তাদের পিছনে বসিয়ে রাখে, ঠিক যেমন highway-তে কয়েকটা slow truck পুরো convoy আটকে রাখে। FCFS-এর rigid non-preemptive স্বভাবই এর মূল কারণ — কিছুই CPUB-কে মাঝপথে একটু থামিয়ে কোনো দ্রুত I/O-bound job-কে ফাঁক দিয়ে ঢুকতে দিতে পারে না।

---

### 6.2 Shortest Job First (SJF)

**Idea:** বর্তমানে যারা *wait* করছে তাদের মধ্যে থেকে, সবসময় সবচেয়ে ছোট total run time-এর job-টা আগে চালাও। Nonpreemptive — একবার শুরু হলে শেষ পর্যন্ত চলবে।

**Requirement:** প্রতিটা job-এর run time আগে থেকে জানা (বা estimate করা) থাকতে হবে। (Batch environment-এ এটা realistic — যেমন insurance company প্রতিদিন একই ধরনের claim-processing job চালায়, historical average দিয়ে ভালোই predict করা যায়।)

#### Worked example (সবাই t=0 তে আসে):

| Process | Duration | SJF Order |
|---|---|---|
| P4 | 3 | 1st |
| P1 | 6 | 2nd |
| P3 | 7 | 3rd |
| P2 | 8 | 4th |

Timeline: `P4[0–3] → P1[3–9] → P3[9–16] → P2[16–24]`
- Turnaround: 3, 9, 16, 24 → **Average = 13**

একই job-গুলো FCFS-এ original arrival order-এ (P1,P2,P3,P4) চালালে compare করি:
Timeline: `P1[0–6] → P2[6–14] → P3[14–21] → P4[21–24]`
- Turnaround: 6, 14, 21, 24 → **Average = 16.25**

**Total completion time দুই ক্ষেত্রেই একই (24) — SJF total throughput একেবারেই বদলায় না।** যেটা বদলায় সেটা হলো **average turnaround time** — ছোট job-গুলোকে বড় job-এর পিছনে না ফেলে আগে শেষ করিয়ে।

#### SJF mathematically optimal কেন (যখন সব job একসাথে আসে)?

ধরো চারটা job-এর duration $a, b, c, d$, আর এই order-এই চালানো হচ্ছে। তাহলে:
- Job 1 শেষ হয় $a$ সময়ে
- Job 2 শেষ হয় $a+b$ সময়ে
- Job 3 শেষ হয় $a+b+c$ সময়ে
- Job 4 শেষ হয় $a+b+c+d$ সময়ে

Mean turnaround $= \dfrac{4a + 3b + 2c + d}{4}$

খেয়াল করো $a$-কে 4 দিয়ে multiply করা হচ্ছে (এটা পরের সব job-এর wait-এও "contribute" করে), $b$-কে 3 দিয়ে, $c$-কে 2 দিয়ে, $d$-কে শুধু 1 দিয়ে। তাহলে sum minimize করতে হলে সবচেয়ে ছোট value-টাকে সবচেয়ে বেশি multiply হওয়া জায়গায় বসাতে হবে — মানে shortest job first, তারপর তার পরেরটা shortest, ইত্যাদি। এটা যেকোনো সংখ্যক job-এর জন্যই সত্যি। Exam-এর জন্য মনে রাখার মতো একটা proof sketch।

#### ⚠️ SJF optimal শুধু তখনই যখন সব job একই সময়ে available (arrive) করে!

বই থেকে counter-example: Job A(2), B(4), C(1), D(1), E(1) — A, B আসে t=0-তে, আর C, D, E আসে t=3-তে।

t=0-তে শুধু A বা B বাছা যায় (বাকিরা এখনও আসেনি)। প্রতিটা decision point-এ greedy ভাবে SJF apply করলে order হবে A,B,C,D,E → average wait 4.6। কিন্তু order B,C,D,E,A দিলে average wait ভালো, 4.4 — কারণ ছোট job C,D,E-দের (যারা একটু পরে আসছে) জন্য অপেক্ষা না করে, বড় job B-কে "ফাও" চালিয়ে ফেলাটা একটা smarter ordering-এর চেয়ে খারাপ হয়ে যায়। মূল কথা: naive/greedy SJF arrival time আলাদা হলে **suboptimal** হতে পারে।

---

### 6.3 Shortest Remaining Time Next (SRTN) — SJF-এর Preemptive version

**Idea:** SJF-এর মতোই, কিন্তু এখন **preemptive**। Scheduler সবসময় ready process-দের মধ্যে যার *remaining* time সবচেয়ে কম, তাকে চালায়। যখন একটা **নতুন** job আসে, তার *total* time-কে current running process-এর *remaining* time-এর সাথে compare করা হয় — নতুন job যদি কম সময়ে শেষ হয়, তাহলে current process-কে **preempt** করে নতুনটা চালানো হয়।

#### Worked example:

| Process | Duration | Arrival |
|---|---|---|
| P1 | 10 | 0 |
| P2 | 2 | 2 |

- t=0: শুধু P1 ready → P1 চলে।
- t=2: P2 আসে, তার শুধু 2 লাগবে, কিন্তু P1-এর বাকি আছে 8 → P2 জিতে যায় → **P1 preempt**, P2 চলে।
- t=4: P2 শেষ (তার শুধু 2-ই লাগত)। P1 আবার resume করে তার বাকি থাকা 8 নিয়ে।
- t=12: P1 শেষ।

Turnaround: P1 = 12 − 0 = 12, P2 = 4 − 2 = 2 → **Average = (12+2)/2 = 7**

(তুলনা করো: এইটা যদি plain non-preemptive হতো (P1 আগে পুরোটা চলত), তাহলে P1=10, P2 = t=10-এ শুরু করে t=12-এ শেষ, turnaround = 12−2=10 → average = 10। তাহলে SRTN-এর 7 এখানে স্পষ্ট ভালো।)

#### ⚠️ সমস্যা: Starvation

Process A-র যদি 1 ঘণ্টা CPU time লাগে, কিন্তু প্রতি মিনিটে একটা নতুন 1-মিনিটের job আসতেই থাকে *forever*, তাহলে A **কখনোই** চলার সুযোগ পাবে না — সবসময়ই নতুন আসা tiny job-এর তুলনায় সে "shortest না"। একে বলে **starvation**: একটা process technically এখনও runnable, কিন্তু সে কখনোই আসলে turn পায় না, কার্যত forever।

---

## 7. Interactive Scheduling Algorithms

Interactive system-এ (remote user handle করা server-সহ) সবচেয়ে বেশি matter করে **response time**। এখানকার algorithm প্রায় সবসময়ই **preemptive**, আর সময়কে ভাগ করা হয় fixed interval-এ, একে বলে **quantum**। প্রতি quantum-এর শুরুতে scheduling decision নেওয়া যায়।

### 7.1 Round Robin (RR)

**Idea:** সবচেয়ে simple fair preemptive algorithm। প্রতিটা process একটা fixed time slice (**quantum**) পায়। সে চলতে থাকবে যতক্ষণ না (a) quantum শেষ হয়, বা (b) সে নিজে থেকে block/finish করে — যেটা আগে হয়। Quantum শেষ হয়ে গেলে যদি process তখনও চলতে থাকে, তাকে preempt করে ready queue-এর **শেষে (tail)** পাঠানো হয়।

**Implementation:** ready queue = simple FIFO list।
- Scheduler head থেকে process pop করে, এক quantum-এর জন্য timer সেট করে, চালায়।
- Quantum শেষ হয়ে গেলে যদি process তখনও চলে: এটাকে queue-এর tail-এ পাঠায়, নতুন head পিক করে।

#### Worked example — quantum = 20, সবাই t=0-তে আসে:

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
Step by step: P1 চলে 20 (বাকি 33), P2 তার পুরো 17 চলে (শেষ, কারণ 17 < quantum), P3 চলে 20 (বাকি 48), P4 চলে 20 (বাকি 4), P1 আবার 20 (বাকি 13), P3 আবার 20 (বাকি 28), P4 তার বাকি 4 শেষ করে, P1 তার বাকি 13 শেষ করে, P3 আবার 20 (বাকি 8), P3 তার বাকি 8 শেষ করে।

**RR-এর trade-off মনে রাখার মতো:** Round Robin সাধারণত SJF-এর চেয়ে *বেশি* average turnaround time দেয় (কারণ ছোট job-কে priority দেয় না), কিন্তু response time দিক দিয়ে **অনেক ভালো** — কেউই একটা বড় job-এর পিছনে fixed অসীম সময় আটকে থাকে না, FCFS-এর মতো যেখানে বড় job-এর পিছনে পড়া job শুধুই wait করতে থাকে।

### 7.2 Quantum Size বাছা — গুরুত্বপূর্ণ Design Decision

এটা RR-এর একমাত্র (বই অনুযায়ী) genuinely subtle এবং interesting design decision।

- **Quantum খুব ছোট হলে** → অনেক বেশি context switch → overhead-এ CPU-র সময় নষ্ট। যেমন: context switch-এর খরচ যদি 1 ms হয় আর quantum = 4 ms, তাহলে প্রতি 5 ms-এর মধ্যে 1 ms (20%!) শুধুই *overhead*, কোনো কাজে লাগছে না।
- **Quantum খুব বড় হলে** → RR প্রায় FCFS-এর মতো আচরণ করা শুরু করে (বেশিরভাগ process quantum শেষ হওয়ার আগেই finish বা block হয়ে যায়) → কিন্তু quantum বড় হলে আর অনেক process wait করলে, 50-টা process-এর queue-এর পিছনের কেউ হয়তো তার প্রথম turn পেতে কয়েক **সেকেন্ড** wait করতে পারে → interactive user-এর কাছে sluggish লাগে, বিশেষত খারাপ যদি সেই unlucky process-এর আসলে অল্প কিছু ms-ই দরকার ছিল।

**Rule of thumb (এই সংখ্যাটা exam-এ আসতে পারে, মনে রাখো):** সাধারণত **20–50 ms** quantum একটা reasonable compromise। আরেকটা কথা: যদি quantum বেশিরভাগ process-এর typical CPU burst length থেকে *বড়* হয়, তাহলে বেশিরভাগ process quantum শেষ হওয়ার আগেই এমনিতেই block হয়ে যাবে — তাই আসল preemption (burst-এর মাঝে জোর করে থামানো) কম হয়, process switch তখনই হয় যখন logically দরকার (কোনো process block করলে) — এটা efficiency-র জন্য ভালো।

### 7.3 Priority Scheduling

**Idea:** RR ধরে নেয় সব process সমান গুরুত্বপূর্ণ — প্রায়ই এটা সত্যি না। এর বদলে প্রতিটা process-কে একটা **priority** দাও; সবসময় সবচেয়ে বেশি priority-র *runnable* process চালাও।

**সমস্যা:** নতুন নতুন উচ্চ priority-র process আসতেই থাকলে low-priority process **starve** করতে পারে forever।
**Fix:** সময়ের সাথে সাথে running process-এর priority কমাতে থাকো (যেমন প্রতি clock tick-এ), যেন eventually এমনকি high-priority process-এর priority-ও এতটা কমে যায় যে অন্য কেউ turn পায়। অথবা: priority নির্বিশেষে প্রতিটা process-কে একটা maximum quantum দাও।

#### Priority কীভাবে assign করা হয়?

- **Static** — আগে থেকেই fix করা। যখন application behavior well-known/predictable হয় তখন ভালো কাজ করে (যেমন strict military hierarchy: general=100, colonel=90, ইত্যাদি, অথবা বেশি টাকা দিয়ে বেশি priority: $100/hr = high priority, $75/hr = medium, ইত্যাদি)।
- **Dynamic** — system নিজে on-the-fly compute ও adjust করে। General-purpose system-এর জন্য এটা বেশি flexible/common।

**একটা concrete dynamic-priority formula (বই থেকে, মনে রাখার মতো):**
$$\text{priority} = \frac{1}{f}$$
যেখানে $f$ = block হওয়ার আগে process তার *শেষ* quantum-এর কতটুকু আসলে ব্যবহার করেছে, সেই fraction।

- 50 ms quantum-এর মধ্যে শুধু 1 ms ব্যবহার করে block হয়ে গেল (খুবই I/O-bound behavior) → $f = 1/50$ → priority $= 50$ (**উচ্চ** priority)।
- 50 ms quantum-এর 25 ms ব্যবহার করল → $f = 1/2$ → priority $= 2$ (**কম** priority)।
- পুরো quantum ব্যবহার করল (খুব CPU-bound) → $f=1$ → priority $=1$ (**সবচেয়ে কম**)।

**I/O-bound process-কে বেশি priority দেওয়া হয় কেন?** কারণ যে I/O-bound process দ্রুত CPU পায়, সে সাথে সাথে তার পরের I/O request পাঠিয়ে দিতে পারে, যার ফলে I/O device অন্য কোনো process-এর computation-এর সাথে *parallel*-এ কাজ শুরু করতে পারে। তাকে বৃথা wait করালে I/O device শুরু হতে দেরি হয়, overall system throughput নষ্ট হয়। এটা সরাসরি Section 5-এর "Balance" goal-এর সাথে সম্পর্কিত।

### 7.4 Priority Classes (Priority-দের Group করা)

অনেক individual priority level রাখার চেয়ে, process-গুলোকে কয়েকটা **priority class**-এ group করা বেশি practical, এবং:
- Class-দের *মধ্যে* strict priority ব্যবহার করা হয় (class 4-এ কিছু runnable থাকতে class 3-কে touch করা হয় না)।
- প্রতিটা class-এর *ভিতরে* Round Robin ব্যবহার করা হয়।

Typical ordering (highest → lowest), slide অনুযায়ী:
```
System processes  >  Interactive processes  >  Interactive editing  >  Batch processes  >  Student processes
```

**বিপদ:** priority কখনো adjust না করলে, নিচের class সম্পূর্ণভাবে **starve** করতে পারে — যেমন "student processes" হয়তো কখনোই চলবে না যদি সবসময় উপরের কোনো class-এ কিছু ready থাকে।

### 7.5 Multiple Queues (CTSS — পুরনো কিন্তু শিক্ষণীয়)

সবচেয়ে পুরনো priority scheduler-দের একটা, MIT-এর CTSS system থেকে, IBM 7094-তে চলত। Context: তখন process switch খুবই expensive ছিল (একবারে শুধু একটা process মেমরিতে থাকতে পারত, তাই পুরো process disk-এ swap করে আরেকটা load করতে হতো)।

**Idea:** CPU-bound process-দের ছোট ছোট অনেক quantum না দিয়ে ধীরে ধীরে বড় বড় quantum দাও (expensive swap কমানোর জন্য):
- প্রথমবার চললে: quantum = 1।
- পুরো quantum ব্যবহার করে আরও দরকার হলে: পরের বার quantum = 2।
- তারপর 4, তারপর 8, 16, 32, 64...
- একটা process তার সব allocated quanta পুরোপুরি ব্যবহার করে ফেললে, সে এক priority class নিচে নেমে যায় (বড় quanta পায়, কিন্তু কম *ঘন ঘন* চলে)।

**Worked example (slide Q46 অনুযায়ী):** একটা process-এর মোট 30 quanta দরকার শেষ করতে।
- Run 1: quantum 1 পায় (1 use, বাকি 29, swap out)
- Run 2: quantum 2 পায় (মোট 3 use, বাকি 27)
- Run 3: quantum 4 পায় (7 use, বাকি 23)
- Run 4: quantum 8 পায় (15 use, বাকি 15)
- Run 5: quantum 16 পায়, কিন্তু আসলে শুধু 15 দরকার → **এই run-এর মাঝেই শেষ হয়ে যায়**।

তাহলে মোট **5 বার swap** লাগল (প্রথম load সহ) — plain Round-Robin quantum=1 দিয়ে 30 বার swap লাগত তার তুলনায়! Expensive process-switch overhead-এ বিরাট কমতি।

**CTSS-এর একটা মজার gotcha (real-world scheduling কেন কঠিন সেটা বোঝার জন্য ভালো "fun fact"):** CTSS-এ একটা rule ছিল — Enter/carriage-return চাপলে সেই process-কে সাথে সাথে highest-priority class-এ পাঠিয়ে দেওয়া হতো (ধরে নেওয়া হতো user এখন interact করতে যাচ্ছে)। কোনো এক clever user আবিষ্কার করে ফেলল, তার CPU-bound job যদি প্রতি কয়েক সেকেন্ডে randomly Enter চাপতে থাকে, তাহলে সে বারবার high priority পেয়ে যায় আর দারুণ response time পায় — একেবারে scheduler-কে game করে ফেলল। **শিক্ষা: Theory-তে scheduling ঠিকভাবে করা আর practice-এ ঠিকভাবে করা — দুইটা সম্পূর্ণ আলাদা জিনিস।**

### 7.6 Shortest Process Next

SJF-এর মতোই spirit, কিন্তু batch job-এর বদলে *interactive* command execution-এ apply করা। Interactive use সাধারণত হয়: `command-এর জন্য wait → execute → command-এর জন্য wait → execute...`। প্রতিটা command execution-কে একটা "job" ধরলে, সবচেয়ে ছোট-estimated job আগে চালানোর চেষ্টা করা যায়।

**সমস্যা:** ভবিষ্যত তো জানা নেই — পরের command কতক্ষণ লাগবে সেটা কীভাবে estimate করবে?

**Solution: Aging** — আগের estimate আর সদ্য observed actual time-এর weighted average নিয়ে পরের run time predict করা:
$$\text{নতুন estimate} = a \cdot T_{old} + (1-a) \cdot T_{measured}$$

$a = 1/2$ দিলে এটা compute করা খুবই সহজ — শুধু old estimate + নতুন measurement যোগ করে 2 দিয়ে ভাগ (binary-তে এক bit right-shift!)। $a$-এর মান memory control করে: 1-এর কাছাকাছি হলে পুরনো history বেশি মনে রাখে; 0-এর কাছাকাছি হলে পুরনো run দ্রুত ভুলে যায়, সাম্প্রতিক behavior-এ দ্রুত react করে। কয়েকবার update-এর পর অনেক পুরনো measurement-এর প্রভাব exponentially কমে যায় — যেমন $a=1/2$-এ 3 বার update-এর পর, original $T_0$-এর weight $1/8$-এ নেমে যায়।

### 7.7 Guaranteed Scheduling

**Idea:** user-দের কাছে একটা explicit, সৎ promise করো, তারপর সেটা রক্ষা করো। যেমন: "$n$ জন user login করা থাকলে, তুমি মোটামুটি CPU-র $1/n$ পাবে।"

**Mechanism:** প্রতিটা process-এর জন্য track করো (a) কতক্ষণ আগে তৈরি হয়েছে, (b) এখন পর্যন্ত সে আসলে কতটুকু CPU পেয়েছে। Compute করো:
$$\text{ratio} = \frac{\text{আসলে পাওয়া CPU time}}{\text{প্রাপ্য CPU time} (= \text{তৈরি হওয়ার পর সময়} / n)}$$

- ratio $< 1$ → process তার fair share-এর *কম* পেয়েছে।
- ratio $> 1$ → process fair share-এর *বেশি* পেয়েছে।

**Rule:** যে ready process-এর ratio সবচেয়ে **কম**, তাকে সবসময় চালাও, যতক্ষণ না তার ratio তার নিকটতম প্রতিযোগীর চেয়ে বেশি হয়ে যায় — তখন switch করো।

### 7.8 Lottery Scheduling

Guaranteed Scheduling-এর মতোই fairness guarantee দেওয়ার একটা clever সহজ পদ্ধতি, কিন্তু জটিল bookkeeping ছাড়াই।

**Idea:** প্রতিটা process-কে কিছু "lottery ticket" দাও। প্রতিটা scheduling decision-এ randomly একটা ticket draw করা হয়; যে সেই ticket ধরে আছে, সে চলার সুযোগ পায় (যেমন: "প্রতি সেকেন্ডে 50 বার lottery হয়, বিজয়ী 20 ms CPU পায়")।

- বেশি গুরুত্বপূর্ণ process-কে সহজেই **বেশি ticket** দাও → জেতার সম্ভাবনা proportionally বেশি হয়।
- মোট 100 ticket থাকলে, একটা process যদি 20-টা ধরে রাখে, প্রতি lottery-তে তার জেতার সম্ভাবনা 20% → দীর্ঘমেয়াদে সে total CPU-র **প্রায় 20%** পাবে। Clean, intuitive, আর statistically self-correcting।

**সুন্দর কিছু property:**
- **অনেক responsive**: একটা একদম নতুন process, ticket পাওয়ার সাথে সাথেই *পরের* lottery-তে জেতার fair সুযোগ পায় — কোনো ramp-up delay নেই।
- **Cooperating process-দের মধ্যে ticket exchange**: যেমন, একটা client server-এ request পাঠিয়ে block হয়ে গেল — এটা temporarily তার সব ticket **server-কে দিয়ে দিতে পারে**, যাতে server পরে চলার সুযোগ বেশি পায় (client-এর request দ্রুত শেষ করার জন্য)। কাজ শেষে server ticket ফেরত দেয়।
- **Proportional-rate সমস্যা elegantly solve করে**: যেমন তিনটা video-streaming process-এর যথাক্রমে 10, 20, আর 25 frame/sec দরকার — শুধু 10, 20, 25 ticket দিয়ে দাও। CPU নিজে থেকেই প্রায় 10:20:25 ratio-তে ভাগ হয়ে যায়, automatically।

### 7.9 Fair-Share Scheduling

**যে সমস্যা এটা সমাধান করে:** সাধারণ per-*process* fairness তবুও per-*user*-এর জন্য unfair হতে পারে। User 1 যদি 9-টা process চালু করে আর user 2 শুধু 1-টা, plain Round Robin দিয়ে user 1 প্রায় CPU-র 90% পাবে (9 process × সমান share) আর user 2 মাত্র 10% — যদিও intuitively দুই user-এর 50/50 পাওয়া "উচিত"।

**Idea:** *user* অনুযায়ী ownership track করো, আর *user* level-এ fairness enforce করো, process level-এ না। User 1 (4 process: A,B,C,D) আর user 2 (1 process: E) যদি প্রত্যেকে CPU-র 50% পাওয়ার promise পায়:

- সত্যিকারের 50/50 হলে: schedule এমনভাবে করো যেন E যতবার চলে, user 1-এর সব চারটা process মিলে ততবার চলে, যেমন:
  `A E B E C E D E A E B E C E D E ...`
- User 1 যদি user 2-এর তুলনায় দ্বিগুণ CPU পাওয়ার promise পায়:
  `A B E C D E A B E C D E ...`

Exact interleaving বিভিন্ন হতে পারে, কিন্তু enforce করা constraint সবসময় per-user share, per-process না।

---

## 8. Real-Time Scheduling

**Real-time system** এমন একটা system যেখানে *correctness* শুধু সঠিক answer পাওয়ার ওপর না, timing-এর ওপরও নির্ভর করে। সঠিক answer *দেরিতে* পাওয়া প্রায়ই একদমই না পাওয়ার মতোই খারাপ (যেমন CD player-এর audio decoder, হাসপাতালের patient monitor, aircraft autopilot, factory robot control)।

### Hard vs Soft real-time

- **Hard real-time**: deadline একদম absolute। একটা miss হলেই system failure (যেমন airbag controller)।
- **Soft real-time**: মাঝে মাঝে deadline miss হওয়া অনাকাঙ্ক্ষিত কিন্তু সহনীয়। (যেমন video player মাঝে মাঝে একটা frame drop করলে — বিরক্তিকর, catastrophic না।)

### Periodic vs Aperiodic events

- **Periodic**: নিয়মিত, predictable interval-এ ঘটা event (যেমন প্রতি 100 ms-এ sensor reading)।
- **Aperiodic**: unpredictably ঘটা event (যেমন user emergency stop button চাপা)।

### Schedulability Test

$m$-টা periodic event থাকলে, যেখানে event $i$ ঘটে period $P_i$-তে আর প্রতিবার handle করতে $C_i$ সেকেন্ড CPU time লাগে, তাহলে system **schedulable** (মানে সব deadline meet করা principle-এ সম্ভব) হবে শুধু যদি:

$$\sum_{i=1}^{m} \frac{C_i}{P_i} \leq 1$$

Intuition: $C_i / P_i$ মানে হলো "এই একটা event stream গড়ে CPU-র কত fraction চায়।" সব event stream-এর demand করা fraction-এর *sum* যদি 1-এর বেশি হয়ে যায় (মানে 100%-এর বেশি CPU দরকার হয়), তাহলে mathematically সবাইকে সন্তুষ্ট করা impossible — যত ভালো algorithমই হোক না কেন, কিছু করার নেই। এখানে ধরে নেওয়া হয়েছে context-switch overhead negligible।

#### Worked example (slide/বই থেকে):
তিনটা periodic event: period 100, 200, 500 ms; CPU দরকার যথাক্রমে 50, 30, 100 ms।
$$\frac{50}{100} + \frac{30}{200} + \frac{100}{500} = 0.5 + 0.15 + 0.2 = 0.85 \leq 1$$
✅ **Schedulable।**

এখন 4র্থ একটা event যোগ হলো, period 1000 ms (1 sec)। এটা schedulable থাকবে যদি এর নিজের $C/P$ total-কে 1-এর বেশি না নিয়ে যায়:
$$1 - 0.85 = 0.15 \Rightarrow C_4 \leq 0.15 \times 1000 = 150 \text{ ms}$$
তাহলে নতুন event প্রতিবার **সর্বোচ্চ 150 ms** CPU time demand করতে পারে এবং system তাও schedulable থাকবে।

### Static vs Dynamic real-time scheduling

- **Static**: সব scheduling decision system চালু হওয়ার *আগেই* নেওয়া হয় (প্রতিটা job আর deadline সম্পর্কে আগে থেকে পুরোপুরি জানা লাগে)।
- **Dynamic**: scheduling decision *run time*-এ নেওয়া হয়, ঘটনা অনুযায়ী adapt করে। বেশি flexible, আগে থেকে সবকিছু জানার দরকার নেই।

---

## 9. Policy vs Mechanism (একটা Design Philosophy, সংক্ষেপে কিন্তু গুরুত্বপূর্ণ)

সাধারণত scheduler সব process-কে independent competitor হিসেবে treat করে। কিন্তু মাঝে মাঝে একটা process-এর অনেকগুলো *child* থাকে যাদের সে ভালোভাবে চেনে (যেমন একটা database server অনেকগুলো child process spawn করে বিভিন্ন query handle করতে) — আর *সেই process* OS-এর চেয়ে ভালো জানে তার কোন child এখন বেশি গুরুত্বপূর্ণ।

**সমাধান:** **scheduling mechanism** (kernel-এর ভিতরের আসল machinery যেটা actually কে চলবে বাছে) থেকে **scheduling policy** (কীভাবে বাছবে সেটা ঠিক করা parameter/priority) আলাদা করে ফেলো। Concretely: kernel হয়তো একটা plain priority scheduler চালায় (mechanism), কিন্তু একটা system call expose করে যা দিয়ে parent process নিজের child-দের priority set করতে পারে (policy)। Kernel এখনও mechanical কাজটা করে, কিন্তু একটা informed user-level process intelligence যোগ করে।

এটা OS design-এ বারবার ফিরে আসা একটা theme — "engine" আর "steering"-কে আলাদা রাখা, যাতে engine সহজ ও general থাকে, আর intelligence উঁচু, বেশি informed layer-এ থাকতে পারে।

---

## 10. Thread Scheduling

মনে করো (আগের threads material থেকে): একটা process-এর ভিতরের thread-গুলো address space share করে কিন্তু নিজেদের registers/stack রাখে। একটা system-এ যদি অনেকগুলো *process* থাকে, প্রতিটার আবার অনেকগুলো *thread* থাকে, তাহলে scheduling decision-এর এখন **দুই স্তর**: কোন process, আর তার ভিতরে কোন thread। এটা অনেকটাই depend করে thread implementation **user level**-এ নাকি **kernel level**-এ হচ্ছে তার ওপর।

### 10.1 User-level threads

**Kernel জানেই না thread বলে কিছু আছে** — kernel-এর কাছে শুধু একটা process দেখা যায়। Kernel একটা *process* বাছে (তার নিজের process-scheduling algorithm দিয়ে), তারপর সেই process-এর ভিতরে থাকা **thread scheduler** (runtime library-এর অংশ) decide করে তার *নিজের* কোন thread এই turn-এ চলবে।

**ফলাফল:**
- Thread switching **অত্যন্ত সস্তা** — মাত্র কয়েকটা machine instruction, কারণ kernel-এ trap নেই, memory-map reload নেই, cache flush নেই।
- **Thread-দের মধ্যে কোনো preemption হয় না** যদি না thread library নিজে সেটা তৈরি করে, কারণ thread-দের জন্য কোনো নিজস্ব clock interrupt নেই — একটা thread স্বেচ্ছায় CPU না ছাড়লে সে পুরো process-এর সব সময় নিয়ে নিতে পারে।
- উদাহরণ: 50 ms process quantum, thread-গুলো প্রতি burst-এ ~5 ms চলে yield করার আগে → সম্ভব order: `A1, A2, A3, A1, A2, A3, ...` (সব interleaving process A-এর turn-এর *ভিতরেই* ঘটছে)। **সম্ভব না**: `A1, B1, A2, B2, ...` — কারণ kernel individual thread schedule করছে না, সে পুরো process schedule করছে, তাই দুইটা *ভিন্ন* process-এর thread কখনো thread-by-thread interleave হতে দেখবে না; process A তার পুরো quantum পায় B-এর কোনো সুযোগ পাওয়ার আগেই।
- **বড় সুবিধা:** application একটা **application-specific thread scheduler** ব্যবহার করতে পারে, নিজের logic অনুযায়ী tune করা। যেমন, একটা web server-এর একটা "dispatcher" thread আর কয়েকটা "worker" thread আছে; একটা worker disk I/O-তে block করলে, runtime — প্রতিটা thread কী করে সেটা ভালোভাবে জেনে — বুদ্ধি করে dispatcher-কে পরে চালাতে পারে (যেন সে আরেকটা worker spin up করতে পারে), parallelism maximize হয়। *Kernel* এতটা smart decision কখনো নিতে পারবে না, কারণ তার প্রতিটা thread-এর *purpose* সম্পর্কে কোনো ধারণা নেই।

### 10.2 Kernel-level threads

**Kernel সরাসরি প্রতিটা thread সম্পর্কে জানে** এবং তাদের individually schedule করে — সে "process"-কে atomic scheduling unit হিসেবে treat করতে বাধ্য না (যদিও চাইলে process membership হিসেবে নিয়ে বিবেচনা করতে পারে)।

**ফলাফল:**
- Thread switch **expensive** — একদম process switch-এর মতোই পুরো context switch (memory map বদল, cache invalidation)।
- **Thread burst-এর মাঝেই forcibly preempt** হতে পারে, কারণ kernel-এর thread-এর জন্য proper clock-driven mechanism আছে, শুধু process-এর জন্য না।
- ভিন্ন process-এর thread-দের মধ্যে interleaving এখন সম্ভব: `A1, B1, A2, B2, A3, B3` — এটা user-level threads দিয়ে কখনো সম্ভব ছিল না।
- **Kernel-level threads-এর একটা upside user-level-এর তুলনায়:** process A-র একটা thread যদি I/O-তে block করে, *শুধু সেই thread*-ই block হয় — kernel অন্য কোনো ready thread (এমনকি একই process-এর) কে freely schedule করতে পারে ইতিমধ্যে। User-level threads-এ, thread scheduler সতর্ক না থাকলে (বা যদি সেই blocking call একটা genuine kernel-level blocking syscall হয়), *পুরো process* — মানে তার *সব* thread — block হয়ে যেতে পারে, কারণ kernel শুধু "একটা process" দেখে, যে thread টা আসলে আটকা পড়েছে সেটা না।
- Kernel তার switch cost সম্পর্কে জ্ঞান ব্যবহার করে smarter trade-off decision নিতেও পারে: যেমন সমান গুরুত্বপূর্ণ দুইটা ready thread থাকলে, একটা একই process-এর যেই thread টা এইমাত্র block হলো তার, আর একটা ভিন্ন process-এর — kernel হয়তো *একই process*-এরটাকে preference দিবে — কারণ একই process-এর মধ্যে switch করলে expensive memory-map reload আর cache flush এড়ানো যায়, যা ভিন্ন process-এ switch করলে হতোই।

### Quick comparison table

| | User-level threads | Kernel-level threads |
|---|---|---|
| Kernel thread সম্পর্কে জানে? | না | হ্যাঁ |
| Thread switch cost | খুব সস্তা (কয়েক instruction) | Expensive (পুরো context switch) |
| Runaway thread-কে preempt করা যায়? | না (cooperative only, যদি না library নিজের timer বানায়) | হ্যাঁ |
| একটা thread I/O-তে block করলে পুরো process block হয়? | হ্যাঁ (সাধারণত) | না — শুধু সেই thread block হয় |
| ভিন্ন process-এর thread interleaving সম্ভব? | না | হ্যাঁ |
| App-specific scheduling logic ব্যবহার করা যায়? | হ্যাঁ, সহজেই | কঠিন — kernel app-এর semantics জানে না |

---

## 11. সব Algorithm-এর Quick-Reference Summary Table

| Algorithm | Preemptive? | Environment | Run time আগে জানা লাগে? | Main strength | Main weakness |
|---|---|---|---|---|---|
| FCFS | না | Batch | না | Implement করা trivial, naive অর্থে "fair" | Convoy effect; খারাপ avg. turnaround |
| SJF | না | Batch | হ্যাঁ | Provably minimal avg. turnaround (সব একসাথে arrive করলে) | Foreknowledge লাগে; long job-এর প্রতি unfair; arrival staggered হলে optimal না |
| SRTN | হ্যাঁ | Batch | হ্যাঁ | SJF-এর চেয়েও ভালো avg. turnaround | Long job-এর starvation সম্ভব |
| Round Robin | হ্যাঁ | Interactive | না | দারুণ response time, simple, fair | Importance/priority ignore করে; quantum tune করা কঠিন |
| Priority Scheduling | সাধারণত হ্যাঁ | Interactive/general | না (কিন্তু priority assign করতে হয়) | Real importance reflect করে | Aging না করলে low priority starve করে |
| Multiple Queues (CTSS-style) | হ্যাঁ | Interactive/batch hybrid | না | Long CPU-bound job-এর জন্য expensive swap কমায় | Game করা যায় (carriage-return trick!) |
| Shortest Process Next | নিজে থেকে না | Interactive | Aging দিয়ে estimate | Interactively SJF-এর benefit approximate করে | Estimation ভুল হতে পারে |
| Guaranteed Scheduling | হ্যাঁ | Interactive | না | Explicit, provable fairness promise | Bookkeeping overhead |
| Lottery Scheduling | হ্যাঁ | Interactive | না | Simple, statistically fair, ticket exchange দিয়ে flexible | Randomness → short-term variance |
| Fair-Share | হ্যাঁ | Interactive, multi-user | না | *User*-ভিত্তিক fair, শুধু process না | বেশি bookkeeping |
| Real-time (deadline-based) | যেকোনোটাই | Real-time | হ্যাঁ (period ও CPU need) | Schedulable হলে hard guarantee meet করে | Demand > 100% CPU হলে schedulable না |

---

## 12. Exam-এ সাধারণ যেসব ভুল হয়, খেয়াল রাখবে

1. **"Turnaround time" vs "waiting time" vs "response time"** — এগুলা আলাদা! Turnaround = completion − arrival। Waiting time = turnaround − actual run time (মানে ready থেকেও *না* চলার সময়)। Response time (interactive) = প্রথম response আসা পর্যন্ত সময়, পুরো completion না।
2. FCFS/SJF/SRTN হাতে-কলমে করার সময়, সবসময় **check করো সব job t=0-তে আসছে কিনা** — না আসলে "সবসময় shortest job বাছো" এভাবে blindly apply করা যাবে না; arrival order মেনে চলতে হবে, শুধু যা *actually* arrive করেছে তার মধ্যে থেকে বাছতে হবে।
3. Convoy effect explain করার সময়, শুধু "FCFS খারাপ" বলো না — *কেন* সেটা বলো: nonpreemptive + একটা long job অনেক short job-কে block করছে → CPU idle / I/O device idle-এর alternating pattern।
4. Schedulability formula $\sum C_i/P_i \le 1$ — মনে রাখো এটা schedulability-র জন্য একটা **necessary** condition, বাস্তব জটিল system-এ সবসময় sufficient না, কিন্তু এই course-এর level-এ এটাই standard test।
5. Clearly distinguish করো: **preemptive vs nonpreemptive** ব্যাপারটা timer tick-এ process-কে *জোর করে* CPU থেকে সরানো নিয়ে। এটাকে "scheduler কি কখনো process switch করে" (সেটা তো nonpreemptive-এও হয়, block/exit/new-process event-এ; nonpreemptive মানে শুধু *clock একাই* switch trigger করতে পারে না) — এর সাথে গুলিয়ে ফেলো না।
6. User-level vs kernel-level thread scheduling short-answer/labeling question-এ প্রিয় একটা topic — key discriminator সবসময়: **kernel কি thread-দের আলাদা schedulable entity হিসেবে জানে?**

---

*Exam-এর জন্য শুভকামনা, ভাই! যদি চাও, আমি আরও practice numerical (উপরের P1–P4 example-এর মতো Gantt-chart style) worked solutions সহ বানিয়ে দিতে পারি, exam-এর আগে নিজে test করার জন্য।*
