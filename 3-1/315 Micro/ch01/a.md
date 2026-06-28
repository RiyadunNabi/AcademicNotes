Now I have the actual book. Let me redo the breakdown properly based exactly on what's in the PDF.

---

# Chapter 1 — Microcomputer Systems (Yu & Marut, exact book)

---

## Overview (Intro paragraph)

The chapter introduces microcomputer architecture — specifically the IBM PC. It covers:
- Main hardware: CPU, memory, peripherals
- What the computer does when executing an instruction
- Advantages and disadvantages of assembly language

---

## 1.1 The Components of a Microcomputer System

A typical microcomputer system has:
- **System unit** — the actual "computer", houses all circuit boards
- **Keyboard, display screen, disk drives** — called **I/O devices** (input/output devices), also called **peripheral devices** or **peripherals**

### How circuits work
Computer circuits are made of **IC (integrated circuit) chips** — each chip can contain hundreds or thousands of **transistors**.

These are called **digital circuits** because they operate on **discrete voltage levels** — specifically a high voltage and a low voltage.

We represent these as:
- `0` = low voltage
- `1` = high voltage

These are called **binary digits** or **bits**. All information in a computer is represented as strings of 0s and 1s — **bit strings**.

### Three functional parts of a computer
1. **CPU** — the brain, controls everything
2. **Memory circuits** — stores information
3. **I/O circuits** — communicates with I/O devices

In a microcomputer, the CPU is a single chip called a **microprocessor**.

### The System Board (Motherboard)
Inside the system unit is the **system board** (also called **motherboard**). It contains:
- The microprocessor
- Memory circuits
- **Expansion slots** — connectors for extra boards called **add-in boards** or **add-in cards**

I/O circuits are usually on these add-in cards.

---

## 1.1.1 Memory

### Bytes and Words

Memory stores information. A single memory element stores 1 bit. But memory is organized in groups of **8 bits = 1 byte**.

Each **memory byte** has a unique **address** — like a house number on a street. The first byte has address 0.

Critical distinction the book emphasizes:
- **Address** — fixed, unique identifier for each byte, never changes
- **Contents** — the data currently stored there, can change anytime
- **Value** — when contents are treated as a number

### How many bytes can be addressed?

Depends on how many bits the processor uses for addresses.

The 8086 uses **20-bit addresses** → 2²⁰ = **1,048,576** = **1 megabyte (1 MB)** of memory.
[See why?](b_1MB.md)

### Words

Two consecutive bytes form a **word** (16 bits). The **lower address** of the two bytes is the address of the word.

Example: memory word at address 2 = bytes at addresses 2 and 3.

The book uses **memory location** to mean either a byte or a word.

### Bit Position

Bits are numbered **right to left, starting from 0**.

In a **byte** (8 bits): positions 0–7
In a **word** (16 bits): positions 0–15
- Bits 0–7 = **low byte**
- Bits 8–15 = **high byte**

For a word stored in memory: low byte is at the lower address, high byte at the higher address.

### Memory Operations

The CPU can do only **two things** with memory:

- **Read (fetch)** — gets a *copy* of the contents. Original is unchanged.
- **Write (store)** — puts new data into a location. Original contents are **lost**.

### RAM and ROM

| | RAM | ROM |
|---|---|---|
| Full name | Random Access Memory | Read-Only Memory |
| Can be read? | Yes | Yes |
| Can be written? | Yes | No |
| Loses data on power off? | Yes | No |
| Used for | Your running programs & data | System startup programs (firmware) |

ROM-based system programs are called **firmware** — responsible for booting and self-testing the computer.

### Buses

Components communicate via **buses** — sets of wires connecting CPU, memory, and I/O.

Three buses:

| Bus | Purpose |
|---|---|
| **Address bus** | CPU puts the memory address here ("where?") |
| **Data bus** | Actual data travels here ("what?") |
| **Control bus** | Signals like read/write travel here ("do what?") |

Example — reading from memory:
1. CPU puts the address on the address bus
2. CPU sends a "read" signal on the control bus
3. Memory sends back the data on the data bus

---

## 1.1.2 The CPU

The CPU executes programs stored in memory. Every instruction it executes is a **bit string** — this is called **machine language**. For the 8086, instructions are **1 to 6 bytes long**.

The set of all instructions a CPU can execute is its **instruction set** — and it's unique to each CPU family.

Machine instructions are kept **simple by design** (add two numbers, move data). Complex tasks are just long sequences of these simple operations.

The 8086 CPU has **two main components**:

### Execution Unit (EU)

Does the actual work of executing instructions. Contains:
- **ALU (Arithmetic and Logic Unit)** — performs arithmetic (+, −, ×, ÷) and logic (AND, OR, NOT)
- **Registers** — fast storage circuits inside the CPU, referred to by name not address. The EU has 8 general registers: `AX, BX, CX, DX, SI, DI, BP, SP`
- **Temporary registers** — hold operands while the ALU is working
- **FLAGS register** — individual bits reflect the result of operations (e.g. was the result zero? was there overflow?)

### Bus Interface Unit (BIU)

Handles all communication between the EU and the outside world (memory, I/O). Contains:
- Registers: `CS, DS, ES, SS, IP`
- **IP (Instruction Pointer)** — always holds the address of the *next instruction* to be executed

### How EU and BIU work together — Prefetch

While the EU is executing one instruction, the BIU is already **fetching the next up to 6 bytes** from memory into the **instruction queue**. This is called **instruction prefetch** — it speeds up the CPU by keeping it busy.

If the EU needs memory access mid-execution, the BIU pauses prefetching and handles that first.

---

## 1.1.3 I/O Ports

I/O devices connect to the computer through **I/O circuits**. Each I/O circuit has registers called **I/O ports** — some for data, some for control commands.

I/O ports have **I/O addresses** (separate from memory addresses) — this is how the CPU tells the difference between accessing memory and accessing an I/O device.

I/O ports are **transfer points** between CPU and devices:
- Input: device sends data to port → CPU reads it
- Output: CPU writes data to port → I/O circuit sends it to device

### Serial and Parallel Ports

| Type | Transfer rate | Example devices |
|---|---|---|
| **Serial** | 1 bit at a time, slower | Keyboard |
| **Parallel** | 8 or 16 bits at a time, faster | Disk drive |

Some devices (like printers) can use either.

---

## 1.2 Instruction Execution

This is where the book explains what actually happens inside the CPU step by step.

A machine instruction has two parts:
- **Opcode** — specifies the *type* of operation (add? move? store?)
- **Operands** — the data or addresses to operate on

### The Fetch-Execute Cycle

The CPU repeats this loop forever:

**Fetch:**
1. Fetch the instruction from memory
2. Decode it (figure out what operation it is)
3. Fetch data from memory if needed

**Execute:**
4. Perform the operation
5. Store the result in memory if needed

### Detailed Example from the Book

The book traces through this specific instruction: *add the contents of register AX to the memory word at address 0, store result back*.

Machine code: `00000001 00000110 00000000 00000000`

Step by step:

1. **Fetch instruction** — BIU puts the instruction's address on the address bus, sends a read request on control bus. Memory sends back the 4-byte instruction on the data bus. Since 8086 reads a word at a time, this takes **two read operations**. IP is updated to point to the next instruction.

2. **Decode** — EU's decoder circuit figures out this is an ADD operation involving memory word at address 0.

3. **Fetch data** — EU tells BIU to get the contents of memory word 0. BIU sends address 0 on address bus, read request on control bus. Memory sends the data back on the data bus → stored in a **holding register**.

4. **Perform operation** — Holding register + AX register both go into the ALU, which adds them and holds the sum.

5. **Store result** — EU tells BIU to write the sum back to address 0. BIU sends write request on control bus, address 0 on address bus, the sum on the data bus. Previous contents of address 0 are overwritten.

Then the whole cycle repeats for whatever address is now in IP.

### Timing — Clock

All of this is synchronized by a **clock circuit** which generates **clock pulses** (regular on/off electrical signals).

- **Clock period** — time between two pulses
- **Clock rate / clock speed** — pulses per second, measured in **MHz** (megahertz; 1 MHz = 1 million cycles/second)

Original IBM PC: **4.77 MHz**. Each step in the fetch-execute cycle takes one or more clock periods. Memory read = 4 clock periods. Multiplication = 70+ clock periods.

Faster clock = faster CPU, but each CPU has a **maximum rated clock speed** it can't safely exceed.

---

## 1.3 I/O Devices

Primary I/O devices covered:

### Magnetic Disks
RAM loses everything on power off → disks provide **permanent storage**.

Two types:
- **Floppy disk (diskette)** — removable, portable, 360 KB to 1.44 MB capacity
- **Hard disk (fixed disk)** — sealed, non-removable, 20–100+ MB, much faster

The device that reads/writes a disk is a **disk drive**.

1 **kilobyte (KB)** = 2¹⁰ = 1024 bytes.

### Keyboard
Has its own microprocessor. Sends a coded signal when any key is pressed or released.

Important point from the book: **there is no direct connection between keyboard and screen**. The keyboard sends data to the running program. The *program* must explicitly send it to the screen to display it.

### Display Monitor
Standard output device. A circuit called the **video adapter** converts computer data into video signals. Can display text and graphics, some in color.

### Printers
Produce **hardcopy** (permanent printed output). Three kinds mentioned:
- **Daisy wheel** — typewriter-quality, can't do graphics
- **Dot matrix** — prints dots, supports different fonts and graphics
- **Laser** — 300 dots per inch, typewriter quality, quiet, expensive, essential for desktop publishing

---

## 1.4 Programming Languages

### Machine Language
The CPU can only execute machine language — pure binary bit strings. The book shows a real example:

```
10100001 00000000 00000000  → fetch memory word 0 into AX
00000101 00000100 00000000  → add 4 to AX
10100011 00000000 00000000  → store AX back to memory word 0
```

Writing programs this way is tedious and error-prone.

### Assembly Language
Uses **symbolic names** for operations, registers, and memory locations. Same program above in assembly:

```asm
MOV AX, A   ; fetch contents of location A into AX
ADD AX, 4   ; add 4 to AX
MOV A, AX   ; store AX back to location A
```

An **assembler** translates each assembly statement into exactly one machine language instruction.

### High-Level Languages
FORTRAN, Pascal, C, etc. Look more like natural language. Translated by a **compiler** — more complex than an assembler because one high-level statement becomes *many* machine instructions.

### Advantages of High-Level Languages
1. Closer to natural language — easier to read, write, and understand
2. Fewer statements needed for the same task
3. **Portable** — runs on any machine that has a compiler for that language

### Advantages of Assembly Language
1. **Efficiency** — produces faster, shorter machine code
2. Can access specific memory locations and I/O ports directly — impossible in some high-level languages
3. You can mix — write critical parts in assembly, the rest in a high-level language

Extra reason the book gives for learning assembly: it's the only way to truly understand how the computer "thinks" — what happens inside when programs run.

---

## 1.5 An Assembly Language Program

The book shows a complete real program — adds two numbers A and B, stores result in SUM:

```asm
TITLE PGM1_1: SAMPLE PROGRAM
.MODEL SMALL
.STACK 100H
.DATA
    A   DW  2
    B   DW  5
    SUM DW  ?
.CODE
MAIN PROC
    ; initialize DS
    MOV AX, @DATA
    MOV DS, AX
    ; add the numbers
    MOV AX, A      ; AX = A
    ADD AX, B      ; AX = A+B
    MOV SUM, AX    ; SUM = A+B
    ; exit to DOS
    MOV AX, 4C00H
    INT 21H
MAIN ENDP
END MAIN
```

### Three segments of every assembly program:

**1. Stack segment (`.STACK`)**
Used for temporary storage of addresses and data. Must always be declared — even if unused — or the assembler throws an error.

**2. Data segment (`.DATA`)**
Where variables are declared and given memory space. Examples:
- `A DW 2` → reserve a word (16 bits) called A, initialize to 2 (DW = Define Word)
- `B DW 5` → reserve a word called B, initialize to 5
- `SUM DW ?` → reserve a word called SUM, uninitialized

**3. Code segment (`.CODE`)**
Where actual instructions go. Organized into **procedures**. This program has one procedure called `MAIN` (starts with `MAIN PROC`, ends with `MAIN ENDP`).

### What the key instructions do:
- `MOV AX, @DATA` + `MOV DS, AX` — initialize the DS register (required boilerplate, explained in Chapter 4)
- `MOV AX, A` — copy contents of variable A into register AX
- `ADD AX, B` — add contents of B to AX (AX now = 7)
- `MOV SUM, AX` — copy AX into SUM
- `MOV AX, 4C00H` + `INT 21H` — exit back to DOS

Two types of statements in assembly:
- **Instructions** — executed when the program runs (`MOV AX, A`)
- **Directives** — commands for the assembler itself, not executed (`.MODEL SMALL`)
- **Comments** — anything after `;`, ignored by assembler

---

That's the entire Chapter 1 from your actual PDF, Riyad — every section, subsection, and sub-subsection covered. Ready for Chapter 2?