# CSE313 — Operating Systems, Lecture 1: Introduction
### বাংলায় সহজ ব্যাখ্যা (সব OS টার্ম ইংরেজিতেই থাকবে)

---

## 0. এই ডকুমেন্ট কেন

স্লাইড হলো summary — মনে রাখার জন্য কিছু পয়েন্ট। বই (Tanenbaum) এ পুরা ব্যাখ্যা আছে, কিন্তু এখন ওইটা পড়ার সময় নাই। তাই প্রতিটা পয়েন্ট বাংলায়, সহজ উদাহরণ দিয়ে বুঝায় দিচ্ছি, যাতে exam হলে গিয়ে মুখস্থ না করে বুঝে লিখতে পারিস।

---

## 1. OS কোন সমস্যা সমাধান করে

### 1.1 কম্পিউটারের হার্ডওয়্যার আসলে অনেক এলোমেলো জিনিসের সমষ্টি

তোর কম্পিউটারে CPU, disk controller, USB controller, graphics adapter, memory — সব আলাদা আলাদা hardware, প্রত্যেকটার কথা বলার নিয়ম (protocol) আলাদা।

প্রতিটা program চালাতে দরকার হয়:
- **CPU time** — instruction execute করার জন্য
- **Memory** — code আর data রাখার জন্য
- **I/O devices** — disk, keyboard, mouse, network ইত্যাদি

এখন যদি প্রতিটা application লেখা programmer-কে জানতে হতো একটা নির্দিষ্ট SATA hard disk এর register-level protocol কীভাবে কাজ করে, USB controller কীভাবে কথা বলে — **তাহলে কেউ কোনোদিন কোনো application শেষ করতে পারতো না।** কারণ প্রতিটা hardware model এর জন্য এটা আলাদা রকম কঠিন।

**উদাহরণ:** ধর, ফোন কল করতে যদি তোকে radio tower কীভাবে signal route করে, telecom switching network কীভাবে কাজ করে — এসব জানতে হতো! তার বদলে তুই শুধু "Call" বাটনে চাপ দিস। মাঝখানে একটা layer বসানো আছে যেটা সব জটিলতা সামলায় — এটাই OS, programmer এর জন্য।

### 1.2 একটা software, দুইটা কাজ

বই আর স্লাইড দুইটাতেই বলে, OS এর দুইটা কাজ থাকে — আলাদা কিন্তু একসাথে জড়িত:

1. **Hardware resources গুলো বুঝে, manage করে, efficiently allocate করা।** ("Resource manager" view — কোন program এখন CPU পাবে, memory তে কোথায় জায়গা পাবে, disk এর turn কার — এসব ঠিক করা।)
2. **Programmer-কে একটা পরিষ্কার, সহজ, abstract machine দেওয়া** — messy আসল hardware এর বদলে। ("Extended machine" / abstraction provider view — তুই শুধু `read()` কল করিস file থেকে data পাওয়ার জন্য, disk controller এর সাথে সরাসরি কথা বলা লাগে না।)

কে কীভাবে ব্যাখ্যা করছে তার উপর নির্ভর করে এক framework বেশি emphasize হয়, কিন্তু আসলে এগুলা একই software এর দুইটা দিক মাত্র।

**ফলাফল:** কম্পিউটারে একটা special software layer থাকে — **Operating System** — যেটা raw hardware আর বাকি সব program এর মাঝখানে বসে এই দুইটা কাজই করে।

---

## 2. OS কী? (Definition + কোথায় বসে)

### 2.1 তিনটা প্রায়-সমান definition (স্লাইড থেকে)

- যেটা boot এর সময় সবার আগে load হয় আর পরে বাকি সব program manage করে।
- System software যেটা hardware আর software resource manage করে, এবং program-কে common service দেয়।
- Hardware আর user এর মাঝখানের fundamental interface।

এর কোনোটাই একেবারে perfect definition না (বইও স্বীকার করে OS-কে সঠিকভাবে define করা কঠিন), কিন্তু একসাথে দেখলে বোঝা যায়: **এটা special privilege নিয়ে চলা software, যেটা software stack এর একদম নিচে বসে বাকি সব manage করে।**

### 2.2 OS কোথায় বসে — "Placement of OS" diagram

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

OS একদম bare hardware এর উপরে সরাসরি বসে। বাকি সবকিছু — browser, compiler, music player, এমনকি অন্য "system" utility — OS এর উপরে বসে, আর OS এর উপর নির্ভর করে।

### 2.3 "Scope of Interaction" diagram — কে কার সাথে কথা বলে

```
   Human users
       ↕
Application software   (browser, editor, games...)
       ↕
   Operating system
       ↕
Other system software  (kernel এর বাইরের driver, কিছু library)
       ↕
     Hardware
```

খেয়াল কর, ডানপাশে Human users থেকে সরাসরি Hardware আর Other system software পর্যন্ত double arrow আছে — এর মানে হলো, ঘুরিয়ে-ফিরিয়ে, একজন human যা-ই করুক না কেন শেষমেশ hardware পর্যন্ত পৌঁছায়, কিন্তু যেকোনো resource access এর জন্য সবসময় এটা OS layer দিয়েই যায়।

### 2.4 একটা গুরুত্বপূর্ণ সূক্ষ্ম ব্যাপার: shell/GUI, OS না

এইখানে অনেকে ভুল করে। তুই যখন Windows desktop, বা GNOME (তোর Pop!_OS এ), বা একটা terminal prompt দেখিস — এটা **shell** (text-based) অথবা **GUI** (icon-based)। এটা একটা **user-mode program** যেটা OS ব্যবহার করে কাজ করায়, কিন্তু এটা নিজে OS এর অংশ না।

- তুই তোর email client delete করে অন্যটা install করতে পারিস — কোনো বাধা নাই।
- কিন্তু তুই একজন সাধারণ user হয়ে low-level clock interrupt handler বা memory manager নিজে rewrite করতে পারবি না — ওইটা protected, privileged OS code।

এটা আসলে তুই নিজেই করেছিস: একই Pop!_OS kernel/OS এর উপরে থেকে COSMIC থেকে GNOME এ shift করেছিস। মানে shell/GUI বদলানো যায়, কিন্তু OS core বদলানো যায় না (deep, privileged পরিবর্তন ছাড়া)।

---

## 3. কম্পিউটার Boot হওয়া

স্লাইডে এটা তেমন emphasize করা হয়নি, কিন্তু জানা দরকার: **OS নিজেই তো memory manage করার কথা, তাহলে OS নিজে memory তে আসে কীভাবে?** এটা একটা chicken-and-egg সমস্যা, hardware দিয়ে সমাধান করা হয়।

**ধাপে ধাপে:**

1. **Power on হলো।** CPU hardware-ভাবেই একটা fixed address থেকে instruction execute করা শুরু করে, যেটা motherboard এর একটা ছোট non-volatile chip এ থাকে — এটাকে বলে **ROM/Flash**, এতে থাকে **BIOS** (Basic Input/Output System), বা আধুনিক মেশিনে **UEFI** (একই idea, আরও modern version)।
2. **BIOS diagnostics চালায়** ("POST" — Power-On Self Test): কত RAM লাগানো আছে check করে, keyboard আর basic device ঠিকমতো সাড়া দিচ্ছে কিনা check করে। এজন্যই OS logo আসার আগে একটা কালো screen এ hardware info flash করে দেখায়।
3. **BIOS ঠিক করে কোন device থেকে boot হবে** — একটা list অনুযায়ী try করে (hard drive, USB, CD-ROM), যেটা একটা ছোট battery-backed memory তে সংরক্ষিত থাকে যার নাম **CMOS**। (মজার তফাত: *BIOS* হলো program, *CMOS* হলো যেখানে BIOS তার settings রাখে — boot order, date/time ইত্যাদি।)
4. **Boot device এর প্রথম sector RAM এ load হয়ে execute হয়।** এই ছোট program partition table পড়ে দেখে কোন partition "active", তারপর সেই partition থেকে একটা **secondary boot loader** load করে (Linux এ এটা সাধারণত **GRUB** — তুই নিশ্চয়ই তোর HP Victus এ dual-boot এর সময় Pop!_OS আর Windows এর মধ্যে choose করার সময় GRUB menu দেখেছিস!)।
5. **Boot loader আসল OS kernel RAM এ load করে ওখানে control পাঠিয়ে দেয়।**
6. **OS নিজেকে initialize করে** — data structure setup করে, essential background process চালু করে, শেষে login prompt বা GUI (যেমন তোর GNOME session) চালু করে।

**কেন এটা গুরুত্বপূর্ণ:** OS-কে অবশ্যই RAM এ load হতে হবে বাকি সবকিছু চালানোর আগে, কারণ পরে ও-ই তো বাকি সব load ও manage করবে। Hardware (BIOS/UEFI) এই পুরো ব্যাপারটা "শূন্য থেকে" শুরু করায় — এটাই একমাত্র জায়গা যেখানে "program load করা" এর কাজটা OS করে না, কারণ ঐ মুহূর্তে কোনো OS-ই এখনো চলছে না।

---

## 4. Kernel

স্লাইডের সংক্ষিপ্ত definition: **kernel হলো OS এর সবচেয়ে fundamental অংশ** — যেটা:
- OS চালু হওয়ার সময় *সবার আগে* memory তে load হয়,
- কম্পিউটার চালু থাকা অবস্থায় সবসময় চলতে থাকে,
- মূল কাজগুলো করে: memory management, process management, disk management ইত্যাদি।
- অনেক সময় ঢিলেঢালাভাবে পুরা "OS" এর সমার্থক শব্দ হিসেবে ব্যবহার হয়, যদিও technically OS একটা বড় concept (kernel + system library + utility ইত্যাদি সব মিলিয়ে)।

**উদাহরণ:** OS যদি একটা company হয়, তাহলে kernel হলো head office এর upper management, যাদের কাছে সব চাবি ও ক্ষমতা আছে — সবসময় উপস্থিত, core operation চালায়, আর বাকি department (driver, GUI, utility) আসে-যায় ওদের চারপাশে।

---

## 5. Dual-Mode Operation (Kernel Mode vs User Mode)

এই lecture এর সবচেয়ে গুরুত্বপূর্ণ concept এইটা — পুরা OS course জুড়েই বারবার আসবে।

### 5.1 মূল idea

CPU নিজেই (hardware!) দুইটা mode support করে:

| | **Kernel Mode** | **User Mode** |
|---|---|---|
| আরও যেসব নামে ডাকা হয় | Supervisor mode, master mode, privileged mode, system mode | Slave mode, unprivileged mode, restricted mode |
| এখানে কে চলে | OS / kernel | সাধারণ application program (browser, তোর C++ program ইত্যাদি) |
| Instruction access | **সব** machine instruction ব্যবহার করা যায় | শুধু একটা **subset** — "privileged" instruction (সরাসরি hardware access, I/O control) নিষিদ্ধ |
| Hardware/RAM access | Direct access আছে | Direct access নাই — system call দিয়ে OS এর মাধ্যমে যেতে হয় |
| Mode bit | 0 | 1 |
| এখানে crash হলে... | ...পুরা system down হয়ে যেতে পারে | ...সাধারণত শুধু ওই process টাই মরে, system চলতে থাকে |
| Address space | সব kernel-mode code সাধারণত একই address space শেয়ার করে | প্রতিটা user process নিজের **আলাদা virtual address space** পায় — অন্যদের থেকে isolated |

**এই আলাদা করার কারণ কী?** Protection। যদি যেকোনো application সরাসরি যেকোনো memory তে লিখতে পারতো, বা disk controller-কে সরাসরি order দিতে পারতো, তাহলে একটা buggy বা malicious program পুরা system নষ্ট করে দিতে পারতো, অন্য user এর private data পড়ে ফেলতে পারতো, বা সব crash করে দিতে পারতো। একটা **hardware-enforced** boundary বানিয়ে (একটা literal "mode bit" আছে যেটা hardware flip করে, software মিথ্যা বলতে পারে না), OS নিজেকে আর অন্য program-কে misbehaving user code থেকে রক্ষা করে।

**স্লাইডের উদাহরণ, বেশ ভালো একটা:** *OS হলো Boss, application গুলো labourer।* Boss এর কাছে building এর সব রুমের master key আছে (kernel mode = full hardware access)। Labourer রা শুধু নিজেদের অনুমোদিত রুমেই ঢুকতে পারে (user mode = restricted instruction set), আর যদি কোনো লকড রুম থেকে কিছু দরকার হয়, তাহলে ভেঙে ঢোকার বদলে boss কে formally request করতে হয়। এই formal request-ই হলো **system call** (দেখ Section 7)।

### 5.2 CPU কখন Kernel Mode এ চলে যায়?

তিনটা trigger (স্লাইড থেকে):

**A. System boot** — Section 3 তে দেখেছি, power-on এর পর প্রথম যেটা চলে সেটা inherently privileged; machine kernel mode এ শুরু হয়।

**B. Hardware interrupt** — একটা hardware device (keyboard, disk, network card, timer/clock) CPU-কে signal দেয় "আমার attention দরকার" (যেমন: "একটা key press হয়েছে," "disk read শেষ হয়েছে," "একটা clock tick হয়েছে")। CPU automatically kernel mode এ চলে যায় যাতে OS এটা handle করতে পারে, কারণ hardware এর response দিতে privileged instruction লাগে।

**C. Trap** — এটা *software দ্বারা তৈরি* interrupt। দুই ধরনের:
  - একটা **error condition**: division by zero, invalid memory access (segmentation fault), illegal instruction ইত্যাদি।
  - একটা **ইচ্ছাকৃত request**: user program ইচ্ছা করে একটা `TRAP` instruction চালায়, OS কে service করার জন্য request করতে। এই ইচ্ছাকৃত case-ই হলো **system call**।

---

## 6. Mode Switch করা / "Kernel এ Trap করা costly"

User program যখন OS এর কোনো service চায়, ও normal ভাবে "function call" করতে পারে না (কারণ তাহলে সরাসরি kernel privilege সহ kernel code এ ঢুকে যেতে হবে — allowed না)। এর বদলে একটা special **TRAP instruction** ব্যবহার হয়।

**যেভাবে ঘটে** (তোর স্লাইডের diagram অনুযায়ী):

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

Trap হলে exactly যা ঘটে:

1. **Hardware minimal CPU state save করে** — program counter (PC), stack pointer (sp), আর আরও কিছু register — automatically, যাতে পরে ঠিক যেখানে ছিল সেখান থেকেই resume করা যায়।
2. **CPU mode bit switch করে** kernel mode এ।
3. **Kernel বের করে কোন system call request করা হয়েছে** (একটা numbered table থাকে — প্রতিটা system call এর একটা ID number থাকে)।
4. **Kernel parameters validate করে** — এই file descriptor কি valid? এই pointer কি legal? (User program বিশ্বাস করা যায় না, মনে রাখিস — mode separation এর পুরা পয়েন্টই এটা।)
5. **Kernel আসল কাজটা করে** (যেমন disk থেকে buffer এ data পড়ে)।
6. **State restore হয়, mode bit আবার user mode এ ফিরে যায়**, আর control ফিরে যায় ঠিক trap এর পরের instruction এ।

**কেন এটা "costly"?** এই সব state save/restore করা, parameter validate করা, mode-switch করা — একই program এর মধ্যে normal function call এর তুলনায় pure overhead — এটা নিজে কোনো "useful work" করে না, এটা হলো protection boundary নিরাপদে পার হওয়ার "খরচ"। এজন্যই real system এ অপ্রয়োজনীয় system call কমানো (যেমন এক byte এক byte করে না পড়ে বড় chunk এ file পড়া) performance আসলেই বাড়ায় — তুই পরে এই course এই OS-adjacent code লিখবি, তাই এটা মনে রাখা জরুরি।

---

## 7. System Calls

### 7.1 এগুলো কী

**System call** হলো user-mode code থেকে kernel-mode service এ ঢোকার *একমাত্র* বৈধ দরজা। এটা দিয়ে একটা program OS কে বলে:
- একটা file read/write করতে,
- একটা process create/destroy করতে,
- memory allocate করতে,
- একটা device এর সাথে কথা বলতে,
- ইত্যাদি।

সাধারণত programmer নিজে raw TRAP instruction দিয়ে এগুলো কল করে না (এটা খুবই low-level, machine-specific)। এর বদলে OS একটা **library** দেয় যেটাতে normal দেখতে C function থাকে (যেমন `read()`, `write()`, `fork()`) — এগুলো ভেতরে ভেতরে TRAP instruction execute করে দেয়। তোর C code এর দিক থেকে, `read()` কল করা মানে অন্য যেকোনো function কল করার মতোই দেখতে — trap-and-privilege-switch এর machinery library implementation এর ভেতরে লুকানো থাকে।

```c
count = read(fd, buffer, nbytes);
```

এই এক লাইন চলার সময় নিচের 11 step গোপনে সবগুলো করে।

### 7.2 System Call করার 11 টা step (স্লাইডের diagram থেকে — `read` example)

```
User program calling read:
  1. Push nbytes
  2. Push &buffer
  3. Push fd
  4. Call read           ──► (library procedure "read" শুরু হয়)
  5. read এর জন্য code register এ রাখা হয়
  6. TRAP instruction execute হয়  ───► mode switch user → kernel
  7. Dispatch (kernel table দেখে বের করে এটা কোন syscall)
  8. Sys call handler চলে (আসল কাজ এখানেই হয়)
  9. Return (mode switch kernel → user)
 10. Library procedure "read" caller এর কাছে ফিরে যায়
 11. Increment SP (stack cleanup)
```

মূল insight: **step 6 হলো একমাত্র জায়গা যেখানে actually privilege পরিবর্তন হয়।** এর আগে (1–5) সবটাই ordinary user-mode code (library wrapper argument সাজাচ্ছে); step 9 থেকে আবার user mode এ ফিরে আসে।

### 7.3 Windows vs UNIX system calls (তোর স্লাইডের table থেকে)

| UNIX | Win32 | কী করে |
|---|---|---|
| `fork` | `CreateProcess` | নতুন process তৈরি করে |
| `waitpid` | `WaitForSingleObject` | একটা process শেষ হওয়ার জন্য অপেক্ষা করে |
| `execve` | *(নাই — CreateProcess এর মধ্যেই আছে)* | Process এর program image replace করে |
| `exit` | `ExitProcess` | Execution terminate করে |
| `open` | `CreateFile` | File create/open করে |
| `close` | `CloseHandle` | File close করে |
| `read` | `ReadFile` | Data read করে |
| `write` | `WriteFile` | Data write করে |
| `mkdir` | `CreateDirectory` | Directory create করে |
| `kill` | *(নাই)* | Win32 তে UNIX-style signal নাই |

দুইটা জিনিস মনে রাখার মতো:
- UNIX এর model `fork` + `execve` কে **দুইটা আলাদা step** হিসেবে treat করে (আগে process duplicate করে, তারপর program replace করে) — Windows এর `CreateProcess` একই কল এ দুইটাই করে ফেলে। এই আলাদা করাটা UNIX এর একটা genuinely elegant design decision, যেটার সুবিধা flexibility তে পাওয়া যায় (নিচে `fork()` discussion দেখ)।
- সব কিছু 1:1 map হয় না — Windows এ UNIX file *link* বা *signal* (`kill`) এর সরাসরি equivalent নাই, এটা শুধু নামের পার্থক্য না, real design পার্থক্য।

---

## 8. Core OS Components / Services ("5 pillars")

স্লাইডে পাঁচটা major OS responsibility list করা আছে। এদের প্রতিটার উপর তুই পুরা chapter (এমনকি পরের সেমিস্টারে পুরা course unit) খরচ করবি:

1. **Process Management** — process create, schedule, terminate করা; কে CPU তে কখন চলবে সেটা ঠিক করা।
2. **Memory Management** — process দের RAM allocate করা, একজন থেকে আরেকজনকে isolated রাখা, virtual memory handle করা।
3. **I/O Management** — driver দিয়ে device access control করা, hardware থেকে আসা interrupt handle করা।
4. **Deadlock Management** — এমন পরিস্থিতি detect/prevent করা যেখানে process গুলো একে অপরের জন্য চিরকাল অপেক্ষা করছে (classic example: দুইটা process, প্রত্যেকের কাছে এমন resource আছে যেটা অন্যজনের দরকার)।
5. **File System** — persistent storage কে file আর directory তে organize করা, "এই file টা open কর" কে raw disk operation এ translate করা।

এই lecture শুধু এগুলোর *নাম* বলে দিছে; প্রতিটার deep dive পরে CSE313/314 তে হবে।

---

## 9. UNIX Process Management System Calls: `fork()` বিস্তারিত

স্লাইডের এটাই সবচেয়ে tricky আর গুরুত্বপূর্ণ concrete mechanism, তাই সবচেয়ে বেশি space এখানে দিচ্ছি।

### 9.1 চারটা মূল process-management call

| Call | কাজ |
|---|---|
| `pid = fork()` | একটা child process তৈরি করে — calling process এর **exact duplicate** |
| `pid = waitpid(pid, &statloc, options)` | Parent একটা child terminate হওয়ার জন্য অপেক্ষা করে |
| `s = execve(name, argv, environp)` | Calling process এর program **replace** করে অন্য একটা দিয়ে |
| `exit(status)` | বর্তমান process terminate করে, একটা status code return করে |

### 9.2 `fork()` আসলে কী করে — step by step

Process **A** যখন `fork()` কল করে:

1. Control kernel এ চলে যায় (এটা একটা system call, তাই — trap!)।
2. Kernel একটা একদম নতুন process **B** তৈরি করে যেটা A এর **exact duplicate**: same code, same data, same খোলা file, program এ same position (program counter এর same value)।
3. এখন দুইটা identical, independent process আছে, দুইটাই `fork()` কল এর *ঠিক পরের* instruction থেকে resume করার জন্য প্রস্তুত।
4. দুইটার মধ্যে একমাত্র পার্থক্য: **`fork()` এর নিজের return value.**
   - **Child** (B) এ: `fork()` return করে `0`।
   - **Parent** (A) এ: `fork()` return করে child এর **PID** (একটা positive integer)।
   - (Fork ব্যর্থ হলে parent এ `-1` return হয়, আর কোনো child তৈরি হয় না।)

এই return-value trick-ই তোর code এর একমাত্র উপায় বোঝার যে fork এর পরে "আমি কি parent নাকি child?" — কারণ নাহলে দুইটা process literally identical code চালাচ্ছে।

```c
pid_t pid = fork();
if (pid == 0) {
    // এই block শুধু child এ চলবে
} else if (pid > 0) {
    // এই block শুধু parent এ চলবে; pid = এখানে child এর PID
} else {
    // fork ব্যর্থ হয়েছে
}
```

### 9.3 Example 1 — স্লাইড থেকে

```c
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
int main()
{
    fork();               // এখান থেকে একই program চালানো দুইটা process তৈরি হয়
    printf("Hello world!\n");
    return 0;
}
```

**কী print হবে?** `"Hello world!"` **দুইবার** print হবে — একবার parent থেকে, একবার child থেকে — কারণ `fork()` লাইনের পরে, দুইটা process-ই independently `main()` এর বাকি অংশ চালায়, printf সহ।

### 9.4 Example 2 — fork "tree" (একটানা তিনটা fork)

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

**কীভাবে reason করবি:** প্রতিটা `fork()` কল সেই মুহূর্ত থেকে চলা process এর সংখ্যা **দ্বিগুণ** করে দেয়, কারণ *প্রতিটা* existing process (parent এবং এখন পর্যন্ত তৈরি হওয়া প্রতিটা child) independently পরের প্রতিটা লাইন চালায়, তার মধ্যে পরের `fork()` ও থাকে।

- প্রথম fork এর আগে: 1 টা process (P0)
- fork #1 এর পরে: 2 টা process (P0 এর child, বলি P1, তৈরি হলো; P0 ও চলতে থাকলো)
- fork #2 এর পরে: existing 2 টা process এর প্রত্যেকটা fork করে → মোট 4 টা process
- fork #3 এর পরে: existing 4 টা process এর প্রত্যেকটা fork করে → **মোট 8 টা process**

তাই `"hello\n"` **8 বার** print হবে (2³ = 8), শেষের 8 টা process এর প্রতিটা থেকে একবার করে। তোর স্লাইডের diagram ঠিক এই branching tree টাই দেখায় (P0 → P1, P2 → P3, P4, P5, P6 → P7 ইত্যাদি) — নিজে কাগজে-কলমে এঁকে দেখ, এটা `fork()` বোঝার সবচেয়ে ভালো exercise। এটা genuinely common exam/interview question, আর trick সবসময় একই: **প্রতিটা fork এ কয়টা process আছে সেটা গুনতে হবে, "fork() কতবার কল হয়েছে" সেটা না।**

**একটা সূক্ষ্ম ব্যাপার:** এই 8 টা process আসলে কোন order এ চলে (এবং তাই screen এ 8 টা "hello" কোন order এ আসে) সেটা **guaranteed না** — OS scheduler ঠিক করে। এটাই তোর concurrent/parallel system এ **nondeterminism** এর প্রথম স্বাদ, একটা theme যেটা এই course এর বাকি অংশ জুড়ে চলবে (race conditions, synchronization ইত্যাদি)।

### 9.5 `fork()` আর `exec()` আলাদা দুই কলে কেন করা হলো?

এটা একটা "UNIX কেন এভাবে করলো" প্রশ্ন, স্লাইড থেকে একটু বেশি হলেও বোঝা দরকার। Process creation (`fork`) কে program replacement (`exec`) থেকে আলাদা করার মানে হলো একটা program দুইটার *মাঝখানে* extra logic ঢুকাতে পারে — যেমন, একটা shell child এর input/output একটা file এ redirect করতে পারে, বা environment variable বদলাতে পারে, child এর memory image নতুন program দিয়ে replace হওয়ার *আগেই*। Windows এটা একটা `CreateProcess` কলে বান্ডেল করে ফেলে, আর ওই সব configuration parameter হিসেবে পাস করতে হয় — flexibility কম, কিন্তু single call.

---

## 10. OS Architecture: Monolithic Kernel vs Microkernel

এটা *OS এর নিজের internal structure* নিয়ে — application দের সাথে কথা বলার নিয়ম না, বরং OS টা ভেতরে ভেতরে কীভাবে বানানো সেটা।

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

- **পুরা OS** — process management, memory management, file system, device driver, IPC, সবকিছু — **একটা বড় program** হিসেবে চলে, সবটাই kernel mode এ, same address space শেয়ার করে।
- Kernel এর ভেতরের যেকোনো procedure অন্য যেকোনো procedure-কে সরাসরি call করতে পারে (মাঝে কোনো protection boundary নাই)।

**সুবিধা:**
- **Fast।** Internal component রা সরাসরি একে অপরকে call করে — file system আর disk driver এর মধ্যে expensive user/kernel boundary (Section 6) পার হওয়া লাগে না, কারণ দুইটাই already kernel space এ।

**অসুবিধা:**
- **বিশাল, জটিল codebase** (শুধু Linux kernel এর size কোটি কোটি লাইনের কাছাকাছি)।
- **বুঝা/maintain করা কঠিন** — হাজার হাজার procedure কোনো isolation ছাড়াই একে অপরকে call করছে।
- **ভঙ্গুর (fragile):** যেহেতু সবাই kernel-mode privilege শেয়ার করে, **যেকোনো একটা component এর bug** (ধর, একটা buggy audio driver) **পুরা system crash করে দিতে পারে** — kernel এর ভেতরে একবার ঢুকে গেলে "audio driver" আর "memory manager" এর মধ্যে কোনো দেয়াল নাই।

উদাহরণ: traditional UNIX, Linux, বেশিরভাগ Windows।

### 10.2 Microkernel

```
 User Space:   Applications
               Libraries
               File System | Process Server | Pagers | Drivers | ...  ← "servers"
 ─────────────────────────────────────
 Kernel Space: MicroKernel  (শুধু: IPC, basic process communication, I/O control)
 ─────────────────────────────────────
 Hardware
```

- Kernel কে একদম **bare minimum** এ নামিয়ে আনা হয়: basic process/thread scheduling, inter-process communication (IPC), আর low-level I/O control।
- বাকি সবকিছু যেটা আগে "kernel এর ভেতরে" ছিল — file system, device driver, network stack — সেগুলা **user space এ** বের করে দেওয়া হয়, সাধারণ, unprivileged process হিসেবে চলে, এদের বলে **servers**।
- একটা user program (client) যদি file read করতে চায়, ও সরাসরি kernel থেকে পায় না; ও file server-কে একটা **message** পাঠায়, server কাজ করে message দিয়ে reply পাঠায়। Microkernel এর মূল কাজ শুধু এই message গুলো client আর server এর মধ্যে reliably পৌঁছে দেওয়া।

**সুবিধা:**
- **বেশি secure/reliable।** যেহেতু বেশিরভাগ service সাধারণ user process হিসেবে চলে, একটা server এ crash হলে (যেমন audio driver process crash করলো) **পুরা OS down হয় না** — বাকি system, kernel সহ, ঠিকঠাক চলতে থাকে। এই একই "buggy audio driver" example এর সাথে তুলনা কর — এবার ফলাফল উল্টা।
- **সহজে extend করা যায়।** নতুন service যোগ করা মানে শুধু একটা নতুন user-space server যোগ করা — kernel touch বা recompile করা লাগে না।

**অসুবিধা:**
- **Performance overhead।** প্রতিটা service request এখন user-space process দের মধ্যে message-passing এর মাধ্যমে হয় (kernel এর মাধ্যমে) সরাসরি in-kernel function call এর বদলে — আর Section 6 থেকে মনে কর, user mode আর kernel mode এর মধ্যে যাওয়া-আসা expensive। Microkernel design এটা monolithic এর চেয়ে *বেশিবার* করে।

উদাহরণ: MINIX 3, QNX, L4 — আর বিশেষভাবে, **macOS/Mac OS X এর kernel (XNU) Mach microkernel থেকে derived**, যদিও বাস্তবে modern macOS monolithic আর microkernel এর idea মিশিয়ে ব্যবহার করে (একটা "hybrid kernel")।

### 10.3 Exam-এ আসতে পারে এমন worked example

> *"Audio driver তে একটা severe bug হলে monolithic OS vs microkernel OS এ এর প্রভাব কী?"*

- **Monolithic:** audio driver kernel এর *ভেতরে* চলে, address space আর privilege সবার সাথে শেয়ার করে। একটা severe bug (যেমন invalid memory address এ write করা) *সম্পূর্ণ unrelated* subsystem (memory manager, file system, scheduler) এর kernel data structure নষ্ট করে দিতে পারে, আর **পুরা machine down** করে দিতে পারে — reboot দিতে হবে।
- **Microkernel:** audio driver হলো শুধু আরেকটা user-space process, যেটা well-defined, checked message দিয়ে microkernel এর সাথে কথা বলে। এতে একটা severe bug হলে হয়তো *শুধু ওই process* crash করবে — তুই sound হারাবি, হয়তো error পাবি, কিন্তু file system, scheduler, আর বাকি সব OS ঠিকমতোই চলতে থাকবে।

---

## 11. Multiuser OS

- **একসাথে একাধিক user** একই hardware resource access করতে পারে (প্রত্যেকের দৃষ্টিতে "একই সময়ে")।
- User রা local ভাবে login করতে পারে, বা **network এর মাধ্যমে remote ভাবে**।
- OS/system administrator resource (CPU time slice, memory, storage quota) সবার মধ্যে fairly ভাগ করে দেয়।
- Category গুলোর উদাহরণ: distributed system, time-sliced (timesharing) system, multiprocessor system।

**Intuition:** ধর একটা university এর central Linux server, যেখানে অনেক student একসাথে SSH করে code compile আর run করে — OS তোদের সবার process একসাথে সামলায়, একে অপরের memory আর file থেকে isolated রাখে, একটাই physical machine share করার পরেও।

---

## 12. Multiprocessor OS

- এমন system এর জন্য যেখানে **একের বেশি physical CPU/core** একসাথে কাজ করে।
- সব processor সাধারণত একটা **common memory শেয়ার করে**, আর OS এর কাজ হলো সব processor এর মধ্যে কাজ ভাগ করে দেওয়া (কোন core কোন process/thread চালাবে সেটা ঠিক করা)।
- Overall throughput/performance বাড়ায় কারণ একাধিক জিনিস genuinely *একইসাথে* execute হতে পারে (শুধু interleaved না, যেমনটা single core এ হয়)।
- Modern relevance: **আজকের যেকোনো laptop, তোর HP Victus 15 (i5-13420H — একটা multi-core CPU) সহ, technically OS এর দৃষ্টিতে একটা multiprocessor system**; Linux, Windows, macOS সবগুলাই এই অর্থে multiprocessor operating system — এটা আর কোনো exotic server-only category না।

---

## 13. Quick self-check (না দেখে উত্তর দেওয়ার চেষ্টা কর)

1. Shell/GUI কে কেন "OS এর অংশ" ধরা যায় না?
2. তিনটা trigger এর নাম বল যেগুলা CPU কে kernel mode এ switch করায়।
3. `read(fd, buffer, nbytes)` example এ, ঠিক কোন step এ mode bit actually flip হয়? এর আগের সব step কেন "cheap"?
4. একটা program যদি একটানা **চারবার** `fork()` কল করে (unconditionally, স্লাইডের example এর মতো কিন্তু একটা call বেশি), তাহলে শেষে মোট কতগুলা process থাকবে, আর শেষ fork এর পরের `printf` কতবার execute হবে?
5. Monolithic আর microkernel design এর একটা করে সুবিধা আর অসুবিধা বল — আর audio-driver-bug example দুইটার জন্যই ব্যাখ্যা কর।
6. `read` **system call** আর `read` **library procedure** এর মধ্যে পার্থক্য কী, আর UNIX কেন এগুলা আলাদা করে রাখলো?

*(উত্তর উপরের section গুলাতেই আছে — যদি সব ছয়টা re-read ছাড়া confidently answer করতে পারিস, এই lecture টা solid।)*

---

## 14. পরে কী আসবে (বইয়ের chapter flow অনুযায়ী)

এই lecture (Ch. 1 intro) যেই vocabulary সেট আপ করে দিলো, বাকি পুরা OS course সরাসরি সেটার উপর বানানো:
- **Process management** (Ch. 2) — formalize করবে "process" আসলে *কী*, `fork()` এর child কীভাবে schedule হয়, context switching, thread।
- **Memory management** (Ch. 3) — virtual memory, paging — কীভাবে প্রতিটা process physically RAM share করার পরেও "নিজের" address space পায়।
- **File systems** (Ch. 4) — raw disk block এর উপরের abstraction layer।
- **I/O** (Ch. 5) আর **Deadlocks** (Ch. 6) — Section 8 এর "5 pillars" এর বাকি অংশ পূর্ণ করে।

তুই এখন assembly language (CSE315/316) নিয়েও কাজ করছিস, তাই খেয়াল কর কত overlap আছে: এই lecture এর **TRAP instruction**, **mode bit**, আর **PSW (Program Status Word)** concept গুলা literally একই hardware mechanism যেটা তোর assembly course "OS এর নিচের দিক" থেকে touch করবে। একই concept দুই angle থেকে দেখা (OS course: "কেন আমরা kernel এ trap করি," assembly course: "এই তো actual instruction আর register behavior") দুইটাকেই ভালো করে মাথায় বসিয়ে দেবে।
