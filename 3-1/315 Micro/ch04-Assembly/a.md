Let's go through the `MAIN` procedure **line by line**, explaining **what** it does and **why** it is necessary.

```asm
MAIN PROC
```

* `PROC` means **procedure starts here**.
* `MAIN` is the name of the procedure.
* The CPU doesn't care about this name—it's mainly for the assembler and programmer.

---

```asm
MOV AX, @DATA
```

### What does `@DATA` mean?

`@DATA` is **not the actual data**. It is the **segment address** of the `.DATA` section.

Imagine memory like this:

```
Memory
+-------------------+
| Code Segment      |
+-------------------+
| Data Segment      |  <-- @DATA points here
+-------------------+
| Stack Segment     |
+-------------------+
```

Suppose (just as an example):

```
Data Segment starts at 2A30h
```

Then the assembler replaces

```asm
MOV AX, @DATA
```

with something like

```asm
MOV AX, 2A30h
```

Now

```
AX = 2A30h
```

---

```asm
MOV DS, AX
```

### Why?

All your variables

```asm
ALPHA
BYT
WRD
SUM
B_ARRAY
...
```

are stored in the **data segment**.

The CPU accesses variables like this:

```
Physical Address = DS × 16 + Offset
```

If `DS` is wrong, every variable points to the wrong place.

So we copy

```
AX = Data Segment Address
```

into

```
DS
```

Now

```
DS = Data Segment
```

and all variables become accessible.

---

### Why can't we simply do

```asm
MOV DS, @DATA
```

Because the 8086 **doesn't allow** loading a segment register directly from an immediate value.

This is illegal:

```asm
MOV DS, 1234h
```

The only legal way is

```asm
MOV AX, @DATA
MOV DS, AX
```

---

```asm
MOV DL, LF
```

Earlier,

```asm
LF EQU 0AH
```

`EQU` creates a **named constant**.

So the assembler simply replaces

```asm
LF
```

with

```asm
0AH
```

Therefore,

```asm
MOV DL, LF
```

becomes

```asm
MOV DL, 0AH
```

So after execution,

```
DL = 0AH
```

(Line Feed character in ASCII.)

---

### Why store it in `DL`?

In this program...

**No useful reason.**

The program never uses `DL` afterward.

It's just demonstrating that an `EQU` constant behaves exactly like writing the number directly.

---

```asm
MOV AH, 4CH
```

DOS uses **INT 21h** for many services.

Which service you want depends on **AH**.

Here

```
AH = 4Ch
```

means

> "Terminate the program."

---

Think of it like calling a function:

```
INT 21h

AH tells DOS:
"What function do you want?"
```

Examples:

```
AH = 01h   Read keyboard

AH = 02h   Print character

AH = 09h   Print string

AH = 4Ch   Exit program
```

---

```asm
INT 21H
```

This tells the CPU:

> Stop executing your own code and ask DOS to perform the requested service.

Since

```
AH = 4Ch
```

DOS understands:

```
Terminate this program
Return control to DOS
```

The program ends here.

---

```asm
MAIN ENDP
```

Marks the end of the `MAIN` procedure.

Like a closing brace `}` in C.

---

```asm
END MAIN
```

This is for the **assembler**, not the CPU.

It means:

1. The source file ends here.
2. Execution should begin at `MAIN`.

Equivalent idea in C:

```c
int main()
{
    ...
}
```

The assembler records that `MAIN` is the program's entry point.

---

## Overall flow

```
Program starts
      │
      ▼
Load data segment address into AX
      │
      ▼
Copy AX into DS
(now variables can be accessed)
      │
      ▼
Put 0AH into DL
(just a demonstration of EQU)
      │
      ▼
Set AH = 4Ch
(request DOS exit service)
      │
      ▼
INT 21h
(DOS terminates the program)
      │
      ▼
Program ends
```

The two most important lines are:

```asm
MOV AX, @DATA
MOV DS, AX
```

Without them, your program would not be able to correctly access variables in the `.DATA` segment because `DS` would not point to the right segment.
