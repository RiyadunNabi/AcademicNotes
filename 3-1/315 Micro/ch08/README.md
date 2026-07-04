Full beginner-friendly breakdown of Chapter 8, Riyad — this is a big one: it introduces the **stack** (a concept you'll recognize from DSA, but now seen from the hardware's point of view) and **procedures** (assembly's version of functions), plus your first real session using the **DEBUG** tool to watch a program execute one instruction at a time.

---

# Chapter 8 — The Stack and Introduction to Procedures

---

## Overview

Every program needs some kind of scratch space — a place to temporarily stash data or addresses that it'll need again shortly. In 8086 assembly, that scratch space is called the **stack**, and it lives in its own dedicated segment (recall `.STACK 100H` from your very first programs — you've been silently using a stack the whole time without knowing exactly how it worked under the hood).

This chapter has two big halves:

- **Sections 8.1–8.2** introduce the stack itself: how to add data to it (`PUSH`), remove data from it (`POP`), and one clever application — reversing a sequence of characters — that falls directly out of how a stack naturally behaves.
- **Sections 8.3–8.5** introduce **procedures** — assembly's equivalent of functions/subroutines from high-level languages. You'll learn the exact mechanics of how a procedure is *called* and how it *returns* — mechanics that high-level languages hide from you completely, but which the stack (from the first half of the chapter!) makes possible. Section 8.5 ties everything together with a full worked example: a procedure that multiplies two numbers using nothing but shifting and addition (a nice callback to Chapter 7), traced live in the DEBUG program so you can watch the stack and registers change in real time.

---

## 8.1 The Stack

**What is a stack?** A stack is a **one-dimensional data structure** where you can only add or remove items from *one end*. This is called **last-in, first-out** (LIFO) behavior: whatever item was added most recently is the very first one that comes back off. The book's analogy: a stack of dishes. You always place a new dish on *top* of the pile, and when you need a dish, you take the *top* one — you can't easily grab a dish from the middle or bottom without disturbing everything above it. The most recent addition is called the **top of the stack**.

*(If this sounds exactly like the "Stack" abstract data type you've studied in DSA — it is. This chapter shows you that a real stack data structure isn't just a nice abstraction; it's implemented directly in hardware via a dedicated memory segment and two special-purpose CPU registers.)*

**Setting aside memory for the stack.** Every program needs to reserve a block of memory to actually hold the stack, and you've been doing this since your very first programs, using the directive:
```asm
.STACK 100H
```
This tells the assembler "reserve `100h` (256 decimal) bytes of memory as the stack segment."

**The two registers that manage the stack:**
- **SS** (Stack Segment) — holds the *segment number* of the stack segment. Once the program is assembled and loaded into memory, SS gets set automatically; you don't set it yourself.
- **SP** (Stack Pointer) — holds the *offset address* of the current top of the stack, within the stack segment.

**Initial value of SP.** For a stack declared as `.STACK 100H`, SP starts out equal to `100h` — this represents the **empty stack** position (i.e., "one byte past the very last usable byte of the stack segment," since a 100h-byte segment has valid offsets `0000h` through `00FFh`, and `0100h` sits just beyond that). Once the stack actually contains data, SP holds the offset of whatever is currently on top.

**Figure 8.1A — Empty Stack.** Picture the stack segment as a column of memory addresses, drawn with the *highest* offset at the bottom and the *lowest* offset at the top (this upside-down-looking layout is intentional — you'll see why in a moment). With SP = `0100h`, the arrow marking "top of stack" points just past the bottom of the segment, at offset `0100h`, since nothing has been pushed yet.

### PUSH and PUSHF

To add a new word onto the stack, you **PUSH** it. Syntax:
```asm
PUSH source
```
where `source` must be a **16-bit register or memory word** — note that PUSH only ever works with *whole words* (2 bytes), never single bytes. For example:
```asm
PUSH AX
```

**Executing PUSH causes exactly two things to happen, in this order:**
1. **SP is decreased by 2.**
2. **A copy of the source's content is moved to the memory address specified by SS:SP.** (The source register/memory location itself is left completely unchanged — PUSH only *copies* the value onto the stack.)

**Why does SP decrease, rather than increase, when you add something?** This is the detail that makes the earlier upside-down memory diagram make sense. The stack segment's memory is physically laid out with low addresses at the top and high addresses at the bottom of the diagram, but the stack itself **grows toward the beginning of memory** (toward lower offset numbers) as you push more data onto it. So "pushing a new item" and "moving toward lower addresses" are the same thing. This is a purely historical/hardware design decision in the 8086 — just something to memorize: **stacks grow downward in memory** (toward offset 0), not upward.

**PUSHF** is a special variant with **no operands** — it simply pushes the entire contents of the **FLAGS register** onto the stack in one shot (rather than a general-purpose register like AX or BX). This is handy for temporarily saving the current flag state (e.g., before a subroutine that might disturb the flags, if you need to check them again afterward).

**Walking through Figure 8.1 step by step** (starting from the empty stack in Figure 8.1A, with AX = `1234h` and BX = `5678h`):

**Figure 8.1B — After `PUSH AX`:**
- SP decreases from `0100h` to `00FEh` (a decrease of exactly 2, since we pushed one word).
- The value `1234h` (AX's content) gets copied into memory at offset `00FEh` (which is now `SS:SP`).
- AX itself still holds `1234h` — untouched.

**8.1C — After `PUSH BX`:**
- SP decreases again, from `00FEh` to `00FCh`.
- The value `5678h` (BX's content) gets copied into memory at the new SS:SP, offset `00FCh`.
- Now the stack holds two words: `5678h` at the (new) top (offset `00FCh`), and `1234h` sitting just below it (offset `00FEh`) — exactly reflecting LIFO order, since BX was pushed *after* AX, so it now sits *on top of* AX's pushed value.

### POP and POPF

To remove the top item from the stack, you **POP** it. Syntax:
```asm
POP destination
```
where `destination` must be a **16-bit register (except IP)** or memory word. For example:
```asm
POP BX
```

**Executing POP causes exactly two things to happen, in this order:**
1. **The content currently at SS:SP (the top of the stack) is moved into the destination.**
2. **SP is increased by 2.**

This is exactly the reverse recipe of PUSH — makes sense, since POP is literally "undoing" a push.

**Figure 8.2 walks through this** using a fresh example: suppose the stack currently holds `5678h` on top (at offset `00FCh`) and `1234h` below it (at offset `00FEh`), with SP = `00FCh` (Figure 8.2A).

**8.2B — After `POP CX`:**
- The top-of-stack value, `5678h`, gets copied into CX.
- SP increases from `00FCh` to `00FEh` — now pointing at `1234h`, which has become the new top of the stack (even though we didn't erase or move the old data — the stack "forgets" about a popped value simply by moving SP past it; the old bytes are still physically sitting in memory, but they're no longer considered part of the stack).

**8.2C — After `POP DX`:**
- The (new) top-of-stack value, `1234h`, gets copied into DX.
- SP increases again, from `00FEh` back to `0100h` — the stack is now empty once more, exactly matching its starting state in Figure 8.1A.

**POPF** is the mirror image of PUSHF: it pops the top word of the stack directly into the **FLAGS register**, restoring whatever flag state had previously been saved there.

**Important — no flag effects.** None of these four instructions (`PUSH`, `PUSHF`, `POP`, `POPF`) has any effect on the status flags themselves (except, of course, `POPF`'s entire *purpose* is to overwrite the flags register — but that's the instruction's job, not a side effect of some other computation).

**Two illegal forms to watch out for:**

1. **You cannot PUSH or POP a single byte.** Since PUSH/POP are strictly *word* operations, something like
   ```asm
   Illegal: PUSH DL
   ```
   is not allowed — DL is only 8 bits, but PUSH always needs a full 16-bit source.

2. **You cannot PUSH an immediate (constant) value on the 8086.** Something like
   ```asm
   Illegal: PUSH 2
   ```
   is not allowed on the processor this course targets. *(Side note from the book: this restriction is lifted on the more advanced 80186/80486 processors, which do allow pushing an immediate constant directly — but that's beyond the scope of this course; Chapter 20 covers those processors if you're curious later.)*

### The Stack Isn't Just for You — DOS Uses It Too

Here's a subtlety worth knowing: your program isn't the *only* thing using the stack. The **operating system itself** borrows the stack for its own temporary bookkeeping. For example, whenever you call an `INT 21h` function, DOS internally needs to save the values of whatever registers *it's* about to use, so that it can restore them before handing control back to you — and it does this by pushing them onto the very same stack your program has been using.

**Why doesn't this cause a problem for your program?** Because DOS is disciplined about it: whatever it pushes on entry to the interrupt routine, it faithfully pops back off again before returning control to your program. So by the time execution resumes in your code right after the `INT 21h` line, the stack is back exactly as it was before the interrupt — as if DOS's activity never happened, from your program's point of view.

---

## 8.2 A Stack Application

Now for a genuinely useful trick that falls directly out of the stack's LIFO nature: **because the order things come *off* a stack is always the reverse of the order they went *on*, you can use a stack to reverse any sequence of data** — in this case, a line of typed characters.

### Algorithm to Reverse Input

```
Display a '?'
Initialize count to 0
Read a character
WHILE character is not a carriage return DO
   push character onto the stack
   increment count
   read a character
END_WHILE;
Go to a new line
FOR count times DO
   pop a character from the stack;
   display it;
END_FOR
```

**The core idea in plain English:** as the user types characters one at a time, push each one onto the stack (instead of, say, storing them in an array in the order typed). Once the user hits Enter, every character the user typed is now sitting on the stack — but in reverse order of typing, because the *last* character typed is sitting right on *top*. So when you then simply pop everything back off, one at a time, and display each one as you pop it, the characters come out in exactly the reverse of their original order — with **zero extra reversal logic needed**. The reversal is a free side effect of how a stack naturally behaves.

**Why keep a `count`?** Since we don't know in advance how many characters the user will type, we need some way to know how many times to pop in the second half of the algorithm. Counting each push as it happens (and later using that same count to drive the popping FOR loop) solves this cleanly.

### Program Listing PGM8_1.ASM

```asm
TITLE PGM8_1: REVERSE INPUT
.MODEL SMALL
.STACK 100H
.CODE
MAIN    PROC
; display user prompt
        MOV     AH, 2           ; prepare to display
        MOV     DL, '?'         ; char to display
        INT     21H             ; display '?'
; initialize character count
        XOR     CX, CX          ; count = 0
; read a character
        MOV     AH, 1           ; prepare to read
        INT     21H             ; read a char
; while character is not a carriage return do
WHILE_:
        CMP     AL, 0DH         ; CR?
        JE      END_WHILE       ; yes, exit loop
        ; save character on the stack and increment count
        PUSH    AX              ; push it on stack
        INC     CX              ; count = count + 1
        ; read a character
        INT     21H             ; read a char
        JMP     WHILE_          ; loop back
END_WHILE:
; go to a new line
        MOV     AH, 2           ; display char fcn
        MOV     DL, 0DH         ; CR
        INT     21H             ; execute
        MOV     DL, 0AH         ; LF
        INT     21H             ; execute
        JCXZ    EXIT            ; exit if no characters read
; for count times do
TOP:
        ; pop a character from the stack
        POP     DX              ; get a char from stack
        ; display it
        INT     21H             ; display it
        LOOP    TOP
; end_for
EXIT:
        MOV     AH, 4CH
        INT     21H
MAIN    ENDP
        END     MAIN
```

**Walking through the code, step by step:**

- **Lines displaying `'?'`** — a simple prompt, exactly like patterns you've seen since Chapter 4.
- **`XOR CX, CX`** — initializes the character counter to 0 (recall this trick from Chapter 7's masking section — XORing a register with itself always zeroes it, more efficiently than `MOV CX, 0`).
- **The WHILE loop (`WHILE_:` through `JMP WHILE_`)** — reads characters one at a time. For each character that *isn't* a carriage return:
  - `PUSH AX` — pushes the character onto the stack. **Important detail:** the character itself is only in AL (the low byte of AX), but PUSH can *only* push a full word — so the *entire* AX register gets pushed, not just AL. This means the stack ends up holding the whole 16-bit AX value for each character, even though only its low byte (AL) actually matters. It's a little wasteful of stack space, but it's the only option, since byte-pushes are illegal (as we just learned in 8.1).
  - `INC CX` — bumps the running count of characters read.
  - Reads the next character and loops back to check it.
- **Exiting the WHILE loop (label `END_WHILE:`)** — by this point, every character the user typed is sitting on the stack, with the **most recently typed character on top**. AL still holds the ASCII code of the carriage return itself (the character that broke the loop) — but notice that carriage return itself was *never* pushed onto the stack (the `CMP`/`JE` check happens *before* the `PUSH`), so only the actual typed characters made it onto the stack, not the terminating Enter key.
- **Moving to a new line** — two `INT 21H` calls display a carriage return (`0Dh`) followed by a line feed (`0Ah`), the standard two-character combination needed to actually move the cursor to a fresh new line in a DOS console (a carriage return alone just returns the cursor to the start of the *same* line, which is why both bytes are needed together).
- **`JCXZ EXIT`** — recall from Chapter 6, `JCXZ` = "**J**ump if **CX** is **Z**ero." If the user typed absolutely nothing before pressing Enter (i.e., count = 0), there's nothing on the stack to pop, so the program jumps straight past the FOR loop to the DOS exit — this guards against trying to POP from an empty stack, which would be a bug.
- **The FOR loop (`TOP:` through `LOOP TOP`)** — pops each character off the stack, one at a time, and displays it:
  - `POP DX` — pops the top-of-stack word into DX. Only DL actually matters (since DOS's "display character" function, `AH=2`, reads the character from DL) — but again, since POP only works on full words, the whole word gets popped even though we only care about its low byte.
  - `INT 21H` — displays whatever character ended up in DL.
  - `LOOP TOP` — recall from Chapter 6: `LOOP` automatically decrements CX and jumps back to `TOP` as long as CX isn't yet 0. Since CX was set to the exact character count during the WHILE loop, this FOR loop runs exactly that many times — popping and displaying every character that was pushed, and stopping precisely when the stack (of typed characters) is empty again.

**Sample executions from the book:**

```
C>PGM8_1
?THIS IS A TEST
TSET A SI SIHT

C>PGM8_1
?A
A

C>PGM8_1
?     (only carriage return typed)
      (no output)
C>
```

Notice in the first example: `"THIS IS A TEST"` comes back as `"TSET A SI SIHT"` — the entire line reversed character-by-character (including where the spaces land, since spaces are just ordinary characters that get pushed and popped exactly like letters). This is why the *word order* also appears flipped — the algorithm doesn't know or care about word boundaries; it's blindly reversing the raw character sequence, and reversing a whole sentence character-by-character happens to also flip the order of the words within it, as a side effect.

The third example confirms the `JCXZ EXIT` safety check works correctly: typing nothing but Enter produces no output at all (rather than crashing or popping garbage), since the program correctly detects `count = 0` and skips the FOR loop entirely.

---

## 8.3 Terminology of Procedures

Back in Chapter 6, you were introduced to the idea of **top-down program design**: instead of tackling one giant problem all at once, break it into smaller, easier-to-solve subproblems. High-level languages give you *functions* or *procedures* to implement each subproblem as a self-contained, reusable, nameable chunk of code — and assembly language gives you exactly the same tool, also called a **procedure**.

**Structure of an assembly program as a collection of procedures.** One of the procedures in your program is always the **main procedure** — this is the one that contains the program's actual entry point (the very first instruction that runs when the program starts). To accomplish some sub-task, the main procedure **calls** one of the other procedures in the program. Procedures aren't limited to being called only by main, either — any procedure can call any other procedure, procedures can call each other back and forth, and a procedure can even call **itself** (this is how you'd implement recursion in assembly, a topic explored further later in the course).

**The call/return cycle.** When procedure A calls procedure B, execution control transfers over to B, and B's instructions run. Once B finishes its job, it typically **returns** control back to A — specifically, back to the very next instruction in A that comes right after the original call statement (see **Figure 8.3**). In a high-level language, the compiler generates all the machinery needed to make this call-then-return-to-exactly-the-right-spot behavior work, completely invisibly to you. In assembly, we get to see and control that machinery ourselves — which is exactly what section 8.4 unpacks.

### Procedure Declaration

The syntax for declaring (writing) a procedure:
```asm
name    PROC    type
;body of the procedure
        RET
name    ENDP
```

- **`name`** — a user-chosen identifier for the procedure (just like a variable name, but for a block of code).
- **`type`** — an *optional* operand, either **NEAR** or **FAR**. If you omit it, **NEAR is assumed by default**.
  - **NEAR** means the instruction that calls this procedure lives in the *same* code segment as the procedure itself.
  - **FAR** means the calling instruction is in a *different* segment than the procedure.

*(The book explicitly narrows scope here: for the remainder of this chapter, and indeed for a while yet, we'll assume every procedure is NEAR — since our small programs so far only use one code segment anyway, and FAR procedures don't become relevant until multi-segment or multi-module programs, which is a Chapter 14 topic.)*

**Figure 8.3 — Procedure Call and Return**, in outline form:
```
MAIN PROC
    CALL PROC1
    next instruction
    ...

PROC1 PROC
    first instruction
    ...
    RET
```
The arrow-based diagram in the book shows control flowing from `CALL PROC1` down into `PROC1`'s first instruction, and then — once `PROC1` reaches its `RET` — flowing back up to the "next instruction" line right after the original `CALL`. This is the whole call/return cycle in a nutshell, visually.

#### RET

The **RET** (return) instruction is what makes a procedure hand control back to whoever called it. **Every procedure — except the main procedure itself — should contain a `RET` somewhere**, and by convention it's almost always the very last statement in the procedure's body (since once you're done with a procedure's job, there's nothing left to do but return).

*(Why doesn't the main procedure need a RET? Because main isn't "called" by anything within your program — it's the entry point invoked by the operating system itself when your program starts. Main "returns" not via RET, but via the DOS-exit interrupt call, `MOV AH,4CH` / `INT 21H`, which you've been using since Chapter 4 to end every program.)*

#### Communication Between Procedures

Here's a real difference from high-level languages that's worth sitting with, Riyad: **assembly language procedures have no parameter lists.** In a language like C++ or Java, a function signature explicitly declares what inputs it takes and what it returns — the compiler handles passing that data around for you automatically. In assembly, there's no such built-in mechanism at all. **It's entirely up to you, the programmer, to invent a scheme for how a procedure receives its inputs and how it hands back its results.**

For simple cases with just a few input/output values, the natural and common approach — which you'll see in this chapter's worked example — is to simply **agree on a convention: certain registers will hold certain inputs, and certain registers will hold certain outputs**, and document that convention clearly (see the next subsection) so that anyone using your procedure (including future-you) knows the rules. *(The book notes that the fuller, more general issue of procedure communication — including handling many parameters, or parameters too large to fit in registers — is picked up properly in Chapter 14.)*

#### Procedure Documentation

Since a procedure's inputs/outputs are just an informal convention (not something the assembler can check or enforce for you), it becomes really important to **write it down clearly** as a comment, so nobody has to reverse-engineer your procedure's behavior by reading through its entire body. The book's standard comment-block template, which you should adopt as a habit for every procedure you write from here on:

```asm
; (describe what the procedure does)
; input:  (where it receives information from
;          the calling program)
; output: (where it delivers results to
;          the calling program)
; uses:   (a list of procedures that it calls)
```

You'll see this exact template used in the worked example of section 8.5 below.

---

## 8.4 CALL and RET

### The CALL Instruction

To actually invoke (call) a procedure, you use the **CALL** instruction. There are two flavors:

**Direct call:**
```asm
CALL name
```
where `name` is literally the name of a procedure you've declared — this is by far the more common form, and the only one you'll typically need for now.

**Indirect call:**
```asm
CALL address_expression
```
where `address_expression` specifies a register or memory location that itself *contains* the address of a procedure (rather than naming the procedure directly). This is useful when you don't know in advance which of several procedures you want to call — you can compute or look up an address at runtime and call through it. *(This is conceptually similar to function pointers in C — a topic that will likely come up again later in your course.)*

### What Actually Happens When CALL Executes

This is the heart of the whole chapter — the exact mechanics that make procedure calls work, laid completely bare (something a high-level language would never show you):

**Executing a CALL instruction causes exactly two things to happen, in this order:**

1. **The return address is saved on the stack.** Specifically, this is the **offset** of the very next instruction after the CALL statement — i.e., "come back to right here once you're done." At the moment CALL executes, this address is sitting in the CS:IP register pair (IP specifically holding the offset part, since IP always tracks "the next instruction to fetch"). CALL pushes this IP value onto the stack, exactly like an ordinary `PUSH` would.

2. **IP gets overwritten with the offset address of the procedure's first instruction.** This is what actually **transfers control** to the procedure — the very next fetch-execute cycle will now grab its instruction from the procedure's starting address, instead of continuing on with whatever would have come next in the calling code.

**Figures 8.4A and 8.4B walk through this concretely.** Imagine `MAIN` contains `CALL PROC1` at offset `0010h`, with the very next instruction sitting at offset `0012h` (so the CALL instruction itself must be 2 bytes long, encoding at `0010h`–`0011h`). `PROC1` itself begins at offset `0200h`.

**8.4A — Before CALL:** IP points at `0012h` (about to fetch `CALL PROC1` — wait, actually the book's figure shows IP pointing at `0012h` meaning the CALL at `0010h` has just been fetched and IP has already advanced to point at what *would* be the next instruction, `0012h`, before CALL's own effects execute). The stack is untouched — SP still points at `0100h` (empty).

**8.4B — After CALL:**
- The value `0012h` (the return address — offset of the instruction right after CALL) has been **pushed onto the stack**. SP has decreased by 2, from `0100h` to `00FEh`, and the stack now holds `0012h` at that new top-of-stack location.
- **IP has been changed to `0200h`** — the offset of `PROC1`'s first instruction. Control has now genuinely transferred to `PROC1`; the next instruction the CPU fetches will be `PROC1`'s very first one.

### What Actually Happens When RET Executes

To return from a procedure, you execute:
```asm
RET pop_value
```
Here, `pop_value` is an **optional integer argument**.

**For a NEAR procedure** (which, remember, is the only kind we're dealing with for now), executing `RET` (with no argument) causes:
- **The top word of the stack is popped directly into IP.**

Since that top-of-stack word is exactly the return address that CALL saved there earlier, this single action is enough to send execution right back to the instruction immediately following the original CALL — CS:IP now correctly points at "the next instruction after CALL," and the program simply continues on from there, exactly as if the procedure call had never interrupted the flow.

**If a `pop_value` N is specified:** after popping IP as above, N is *additionally* **added to SP** — which has the effect of discarding N further bytes from the stack (skipping past them without reading their values into anything). *(This variant is genuinely useful once procedures start receiving parameters that were pushed onto the stack by the caller — RET N can clean up those parameter bytes off the stack in the same instruction that returns control. This will matter more once Chapter 14 gets into fuller parameter-passing conventions.)*

**Figures 8.5A and 8.5B illustrate this**, continuing the same running example: suppose `PROC1`'s `RET` instruction sits at offset `0300h`, with the return address `0012h` still sitting on top of the stack (SP = `00FEh`).

**8.5A — Before RET:** IP = `0300h` (about to execute the RET instruction itself). Stack top still holds `0012h`, SP = `00FEh`.

**8.5B — After RET:**
- IP has been popped from the stack, so **IP now = `0012h`** — control has jumped back to exactly the instruction right after the original CALL, inside MAIN.
- SP has increased back to `0100h` — the stack is empty again, exactly as it was before the CALL happened. The stack has been perfectly restored to its pre-call state, with zero leftover residue — this is exactly the "balanced" behavior any well-written procedure should exhibit.

**The big-picture takeaway:** CALL and RET are really just a `PUSH`+`jump` and a `POP`+`jump` in disguise, using the *stack itself* as the mechanism for remembering "where do I go back to." This is precisely why the stack (from sections 8.1–8.2) had to be introduced first — procedures literally could not work without it.

---

## 8.5 An Example of a Procedure

Now for a complete, worked example that pulls together everything from this chapter (and calls back to Chapter 7's shift instructions): a procedure that computes the product of two positive integers **using only shifting and addition** — no multiplication instruction involved at all (you'll meet the actual `MUL`/`IMUL` instructions in Chapter 9; this is a demonstration of *how* multiplication can be built from more primitive operations, which is worth understanding even once you have a dedicated MUL instruction available).

### The Multiplication Algorithm

```
Product = 0
REPEAT
   IF lsb of B is 1     (Recall lsb = least
                          significant bit)
   THEN
      Product = Product + A
   END_IF
   Shift left A
   Shift right B
UNTIL B = 0
```

**Why does this work?** This is essentially the same process you'd use for manual long multiplication in binary — think of it as "binary grade-school multiplication." Recall from Chapter 7 that:
- **Shifting A left** doubles A each time — this mirrors how, in decimal long multiplication, each successive partial product is shifted one place further left (representing "×10 more") for each digit position of the multiplier you move past.
- **Shifting B right** examines B's bits one at a time, from the least significant bit upward, each time discarding the bit just examined — this is how the algorithm "moves along" the multiplier bit by bit.
- **Checking the lsb of B** replicates exactly the "is this multiplier digit a 1 or a 0?" check you make at each step of manual binary long multiplication — a 1-digit means "add in this partial product," a 0-digit means "skip this position."

Every time through the loop, A gets bigger (doubled) while B gets smaller (halved, discarding the bit you just used) — and the loop naturally terminates once B has been shifted all the way down to 0, meaning every one of its original bits has been examined.

### Worked Example: A = 111b (7), B = 1101b (13)

The book traces through the algorithm by hand, step by step:

```
Product = 0
Since lsb of B is 1, Product = 0 + 111b = 111b
Shift left A: A = 1110b
Shift right B: B = 110b

Since lsb of B is 0,
Shift left A: A = 11100b
Shift right B: B = 11b

Since lsb of B is 1
Product = 111b + 11100b = 100011b
Shift left A: A = 111000b
Shift right B: B = 1

Since lsb of B is 1
Product = 100011b + 111000b = 1011011b
Shift left A: A = 1110000b
Shift right B: B = 0

Since lsb of B = 0
Return Product = 1011011b = 91d
```

**Walking through this trace, iteration by iteration:**

- **Iteration 1:** B = `1101b`, lsb = 1 → add A (`111b`) into Product: `0 + 111b = 111b`. Then double A → `1110b`, and halve B (discarding that lsb) → `110b`.
- **Iteration 2:** B = `110b`, lsb = 0 → skip the addition. Double A → `11100b`, halve B → `11b`.
- **Iteration 3:** B = `11b`, lsb = 1 → add A (`11100b`) into Product: `111b + 11100b = 100011b`. Double A → `111000b`, halve B → `1b`.
- **Iteration 4:** B = `1b`, lsb = 1 → add A (`111000b`) into Product: `100011b + 111000b = 1011011b`. Double A → `1110000b`, halve B → `0`.
- **Loop ends** (B = 0). Final Product = `1011011b`, which in decimal is 91.

**Sanity check via ordinary decimal multiplication:** 7 × 13 should equal 91 — and indeed `1011011b = 64+16+8+2+1 = 91` decimal. ✓ The algorithm checks out.

**The book also demonstrates this matches ordinary binary long multiplication done by hand:**
```
    111b
  × 1101b
  ------
    111
   000
  111
 111
 -------
1011011b
```
This is exactly the same process you'd use for decimal long multiplication (multiply the top number by each digit of the bottom number, shifting each partial product one place further left, then add all the partial products together) — just carried out in base 2. Notice each partial product row corresponds exactly to one iteration of the algorithm's loop (a row of `111` for each `1`-bit in the multiplier `1101b`, appropriately shifted, and nothing contributed for the middle `0`-bit) — which is a nice concrete confirmation that the shift-and-add algorithm really is just mechanized long multiplication.

### Program Listing PGM8_2.ASM

```asm
TITLE PGM8_2: MULTIPLICATION BY ADD AND SHIFT
.MODEL SMALL
.STACK 100H
.CODE
MAIN     PROC
; execute in DEBUG. Place A in AX and B in BX
        CALL    MULTIPLY
; DX will contain the product
        MOV     AH, 4CH
        INT     21H
MAIN     ENDP

MULTIPLY PROC
; multiplies two nos. A and B by shifting and addition
; input:  AX = A, BX = B. Nos. in range 0 - FFh
; output: DX = product
        PUSH    AX
        PUSH    BX
        XOR     DX, DX          ; product = 0
REPEAT:
; if B is odd
        TEST    BX, 1           ; is B odd?
        JZ      END_IF          ; no, even
; then
        ADD     DX, AX          ; prod = prod + A
END_IF:
        SHL     AX, 1           ; shift left A
        SHR     BX, 1           ; Shift right B
; until
        JNZ     REPEAT
        POP     BX
        POP     AX
        RET
MULTIPLY ENDP
        END     MAIN
```

**Walking through the code:**

- **`MAIN`** is deliberately minimal — it has **no input or output of its own coded into it**. Instead, the book's plan is to use **DEBUG** (a program that lets you inspect and control execution manually) to *manually* place test values into AX and BX before running, and then *manually* inspect DX afterward to see the result. This is explicitly called out as a teaching device: it gives us an excuse to properly learn how to use DEBUG, which is an essential skill for actually understanding what your assembly code is doing at the machine level (rather than just trusting it blindly).
- **`CALL MULTIPLY`** — invokes the procedure. Recall from 8.4 exactly what happens here: the return address (the offset of `MOV AH,4CH`, the next line) gets pushed onto the stack, and control jumps to `MULTIPLY`'s first instruction.
- **Procedure documentation** — notice `MULTIPLY` follows the exact comment template from 8.3: it states what it does, that its inputs come in via AX and BX (restricted to the range 0–FFh to avoid overflow, since DX is only 16 bits and could overflow with larger inputs), and that its output (the product) comes back in DX.
- **`PUSH AX` / `PUSH BX`** — this is the book's demonstration of **defensive register-saving**, a very important habit: since the procedure is about to modify AX and BX internally (as part of the shift-and-add algorithm), and the *calling* program might still need its original AX/BX values intact after the procedure returns, `MULTIPLY` saves copies of both registers on the stack *before* touching them. The book is candid that, in this *particular* small program, this precaution isn't strictly necessary (since MAIN doesn't do anything with AX/BX after the call) — but it demonstrates the **general best practice**: a well-behaved procedure shouldn't cause unwanted side effects on registers the caller cares about, unless those registers are explicitly documented as part of the procedure's *output*.
- **`XOR DX, DX`** — clears DX to 0, ready to accumulate the product (the same "clear a register" trick from Chapter 7).
- **The REPEAT loop (`REPEAT:` through `JNZ REPEAT`)** — directly implements the shift-and-add algorithm:
  - `TEST BX, 1` — recall from Chapter 7: this checks bit 0 (the lsb) of BX without modifying BX, setting ZF = 1 if that bit is 0 (even) and ZF = 0 if that bit is 1 (odd).
  - `JZ END_IF` — if BX's lsb is 0 (even, ZF=1), skip straight past the addition.
  - `ADD DX, AX` — (only reached when BX's lsb was 1) adds the current value of A (in AX) into the running product (in DX) — implementing the algorithm's `Product = Product + A` step.
  - `SHL AX, 1` — doubles A (Chapter 7's left-shift-as-multiply-by-2 trick).
  - `SHR BX, 1` — halves B, discarding the lsb we just examined (Chapter 7's right-shift-as-divide-by-2 trick — and since B here is meant as *unsigned*, `SHR` rather than `SAR` is the correct choice, exactly per the rule from 7.2.2).
  - `JNZ REPEAT` — this is the "**UNTIL B = 0**" loop-exit test, but written as its logical opposite ("keep looping *while* not zero"): since `SHR BX,1` was the most recently executed instruction affecting the flags, `JNZ` checks whether that shift's result (the new value of BX) was nonzero. If BX still isn't 0, loop back to `REPEAT` for another pass; once BX has finally reached 0, the jump fails and execution falls through to the next lines.
- **`POP BX` / `POP AX`** — restores AX and BX to their original values, undoing the earlier pushes. **Crucial detail, and worth memorizing as a general rule:** the pops happen in the **exact reverse order** of the pushes (`PUSH AX` then `PUSH BX`, but `POP BX` then `POP AX`). This is *required*, not just stylistic — because the stack is LIFO, whatever was pushed *last* (BX) is sitting right on *top*, so it must be popped *first* to correctly unwind back to AX underneath it. Getting this order backwards would swap the two registers' values instead of restoring them correctly.
- **`RET`** — returns control to MAIN, right after the original `CALL MULTIPLY` line, exactly per the mechanics from section 8.4.

---

### Using DEBUG to Trace the Program

This is the chapter's big practical payoff: actually *watching* everything from sections 8.1–8.4 happen live, one instruction at a time, using Microsoft's **DEBUG** program (a low-level debugging tool that comes with DOS). This is genuinely valuable, Riyad — it turns everything we've discussed abstractly (SP changing, IP jumping, the stack filling and emptying) into something you can literally watch happen in front of you.

**Starting DEBUG on the compiled program:**
```
C> DEBUG PGM8_2.EXE
-
```
DEBUG responds with its own command-prompt symbol, a simple hyphen (`-`), and waits for commands. (In the transcript below, whatever the user actually types is shown in **bold**, following the book's convention.)

#### The U (Unassemble) Command

```
-U
177F:0000 E80400        CALL    0007
177F:0003 B44C          MOV     AH,4C
177F:0005 CD21          INT     21
177F:0007 50            PUSH    AX
177F:0008 53            PUSH    BX
177F:0009 33D2          XOR     DX,DX
177F:000B F7C30100      TEST    BX,0001
177F:000F 7402          JZ      0013
177F:0011 03D0          ADD     DX,AX
177F:0013 D1E0          SHL     AX,1
177F:0015 D1EB          SHR     BX,1
177F:0017 75F2          JNZ     000B
177F:0019 5B            POP     BX
177F:001A 58            POP     AX
177F:001B C3            RET
177F:001C E3D1          JCXZ    FFEF
177F:001E E38B          JCXZ    FFAB
```

**What U does:** it tells DEBUG to interpret the raw bytes sitting in memory as machine-language instructions, and display them in a human-readable disassembled form. Each line shows: the **segment:offset** address of that instruction, the raw **machine code bytes**, and the corresponding **assembly instruction**. Everything is shown in **hex**.

**Reading the output:** from the very first line, `CALL 0007`, we can immediately deduce that `MAIN` occupies offsets `0000h` through `0005h` (three instructions: the CALL itself, plus `MOV AH,4CH` and `INT 21H`), and `MULTIPLY` begins right at offset `0007h` (exactly matching the CALL's target) and runs through to `001Bh`, where its `RET` instruction lives. Everything listed *after* offset `001Bh` (the two `JCXZ` lines) is simply **garbage** — leftover bytes in memory that happen to *look like* valid instructions when DEBUG tries to disassemble them, but they're not actually part of our program; they're just whatever data happened to be sitting in memory past the end of our code.

#### The R (Register) Command

```
-R
AX=0000  BX=0000  CX=001C  DX=0000  SP=0100  BP=0000  SI=0000  DI=0000
DS=176F  ES=176F  SS=1781  CS=177F  IP=0000   NV UP EI PL NZ NA PO NC
177F:0000 E80400     CALL    0007
```

**What R does:** displays the current contents of every register, plus a symbolic readout of the flags (`NV UP EI PL NZ NA PO NC` — each pair of letters represents one flag's current state; you won't need to memorize every abbreviation right now, but it's useful to know this line exists), followed by the next instruction about to execute.

Notice **SP = `0100h`** right from the start — confirming exactly what section 8.1 told us: the stack begins in its "empty" state with SP set to the size of the reserved stack segment.

#### The D (Dump Memory) Command

```
-DSS:F0 FF
1781:00F0  00 00 00 00 00 00 6F 17-A4 13 07 00 6F 17 00 00
```

**What D does:** displays the raw contents of a specified range of memory, byte by byte, in hex. The command `DSS:F0 FF` specifically means "show me the bytes from `SS:F0` through `SS:FF`" — i.e., the last 16 bytes of the stack segment (which, since our stack was declared as `100h` bytes = offsets `00h`–`FFh`, is the entire usable stack region near its "bottom," close to where SP currently points).

Since the stack is completely empty at this point, every byte shown here is simply **garbage** — leftover values from whatever happened to occupy this memory before our program was loaded; none of it has any meaning yet.

#### The R command to set a register value

Before we can actually watch the multiplication happen, we need to load our test values (A and B) into AX and BX:

```
-RAX
AX 0000:7
```
Typing `RAX` tells DEBUG "I want to change the content of AX." DEBUG responds by showing the *current* value (`0000`), followed by a colon, and then waits for you to type the *new* value — here, `7` (i.e., A = 7).

```
-RBX
BX 0000:D
```
Same idea for BX — setting B = `Dh` = 13 decimal.

Checking the registers again confirms both values took effect:
```
-R
AX=0007  BX=000D  CX=001C  DX=0000  SP=0100  BP=0000  SI=0000  DI=0000
DS=176F  ES=176F  SS=1781  CS=177F  IP=0000   NV UP EI PL NZ NA PO NC
177F:0000 E80400     CALL    0007
```

#### The T (Trace) Command

```
-T
AX=0007  BX=000D  CX=001C  DX=0000  SP=00FE  BP=0000  SI=0000  DI=0000
DS=176F  ES=176F  SS=1781  CS=177F  IP=0007   NV UP EI PL NZ NA PO NC
177F:0007 50     PUSH    AX
```

**What T does:** executes exactly **one single instruction** and then displays the updated registers — letting you watch the effect of each individual instruction in complete isolation. This is the single most useful command for genuinely understanding what's happening step by step.

**Two changes to notice immediately** after tracing through the `CALL 0007` instruction:
1. **IP now = `0007h`** — exactly PROC1's (MULTIPLY's) starting offset, confirming CALL did transfer control correctly, per section 8.4's mechanics.
2. **SP decreased from `0100h` to `00FEh`** — a drop of exactly 2, confirming CALL pushed the (2-byte) return address onto the stack, exactly as described earlier.

Dumping the stack again confirms this concretely:
```
-DSS:F0 FF
1781:00F0  00 00 00 00 07 00 00 00-07 00 7F 17 A4 13 03 00
```
**Important gotcha the book flags explicitly:** the return address is actually `0003h` — but it's *displayed* in the byte dump as `03 00` (low byte first, then high byte). This is because of the 8086's **little-endian** byte ordering: multi-byte values are always stored in memory with their *least significant byte first*. This is a detail you'll want to keep in the back of your mind any time you're reading a raw hex memory dump — the bytes won't appear in the "obvious" left-to-right reading order you might expect.

#### The G (Go) Command

```
-G7
```
Wait — the book actually uses `G` followed by an offset to run *until* a specific point, rather than tracing one instruction at a time. Syntax:
```
G offset
```
This tells DEBUG to execute instructions continuously (much faster than tracing one at a time) and automatically **stop right before** the instruction at the given offset. This is extremely useful for skipping quickly past a bunch of instructions you already understand, so you can slow down and trace carefully only at the point you're actually interested in.

The book uses this to run past the initial `PUSH AX`, `PUSH BX`, and `XOR DX,DX` lines in one shot, stopping right at offset `000Bh` (where `TEST BX,0001` sits, the very first instruction of the REPEAT loop):
```
-GB
AX=0007  BX=000D  CX=001C  DX=0000  SP=00FA  BP=0000  SI=0000  DI=0000
DS=176F  ES=176F  SS=1781  CS=177F  IP=000B   NV UP EI PL ZR NA PE NC
177F:000B F7C30100     TEST    BX,0001
```
*(Note: the command as typed is `GB`, meaning "go, stopping at hex offset `B`" — DEBUG interprets bare numbers/letters after G as hex offsets directly, no leading zeros needed.)*

Notice **SP has dropped further, from `00FEh` to `00FAh`** — a further decrease of 4, exactly matching the two additional pushes (`PUSH AX` and `PUSH BX` inside MULTIPLY, 2 bytes each = 4 bytes total). Dumping the stack now:
```
-DSS:F0 FF
1781:00F0  00 00 00 00 07 00 00 17-A4 13 0D 00 07 00 03 00
```
The stack now holds **three words**: BX's saved value (`000Dh`), AX's saved value (`0007h`), and — underneath both of those — the original return address (`0003h`) that CALL pushed first. Reading right-to-left in pairs (remembering the little-endian byte order from before): `0D 00` = `000Dh` (BX), `07 00` = `0007h` (AX), `03 00` = `0003h` (return address) — exactly matching the order they were pushed (return address first via CALL, then AX, then BX), with the most recently pushed value (BX) naturally ending up as the current top of the stack.

#### Watching the Loop in Action

The book now alternates between **G17** (run until offset `0017h`, the bottom of the REPEAT loop, right at the `JNZ REPEAT` instruction) and **T** (single-step trace), to watch every iteration of the multiplication algorithm unfold:

```
-G17
AX=000E  BX=0006  CX=001C  DX=0007  SP=00FA  BP=0000  SI=0000  DI=0000
DS=176F  ES=176F  SS=1781  CS=177F  IP=0017   NV UP EI PL NZ AC PE CY
177F:0017 75F2     JNZ     000B
```

**Walking through this first pass:** the initial value of B (in BX) was `0Dh = 1101b`, whose lsb is 1 — so `TEST BX,1` correctly detected an odd number, the `ADD DX,AX` executed, giving DX = `0 + 7 = 0007h`. Then `SHL AX,1` doubled A from `7` to `14d = 000Eh`, and `SHR BX,1` halved B (rounding down, discarding the lsb we just used) from `13` to `6 = 0110b`. This exactly matches the hand-worked trace from earlier in the section (iteration 1: "Product = 111b", i.e., 7 decimal — consistent with DX = `0007h` here).

Running the `T` command to step to the top of the loop, then `G17` again to reach the bottom:
```
-T
AX=000E  BX=0006  CX=001C  DX=0007  SP=00FA  BP=0000  SI=0000  DI=0000
DS=176F  ES=176F  SS=1781  CS=177F  IP=000B   NV UP EI PL NZ AC PE CY
177F:000B F7C30100     TEST    BX,0001

-G17
AX=001C  BX=0003  CX=001C  DX=0007  SP=00FA  BP=0000  SI=0000  DI=0000
DS=176F  ES=176F  SS=1781  CS=177F  IP=0017   NV UP EI PL NZ AC PE NC
177F:0017 75F2     JNZ     000B
```
Since BX had become `0006h = 110b` (lsb = 0, even), the `ADD` step was skipped this time — **DX stayed at `0007h`, unchanged**. AX doubled again, from `000Eh` to `11100b = 001Ch`, and BX halved again, from `6` to `11b = 3h`. This matches the hand trace's iteration 2 exactly ("Since lsb of B is 0" → Product unchanged).

Two more trips through the loop, watching AX, BX, and DX evolve:
```
-T
AX=001C  BX=0003  CX=001C  DX=0007  SP=00FA  BP=0000  SI=0000  DI=0000
DS=176F  ES=176F  SS=1781  CS=177F  IP=000B   NV UP EI PL NZ AC PE NC
177F:000B F7C30100     TEST    BX,0001

-G17
AX=0038  BX=0001  CX=001C  DX=0023  SP=00FA  BP=0000  SI=0000  DI=0000
DS=176F  ES=176F  SS=1781  CS=177F  IP=0017   NV UP EI PL NZ AC PO CY
177F:0017 75F2     JNZ     000B
```
Here, BX was `3 = 11b` (lsb = 1, odd) → DX gets updated: `0007h + 001Ch = 0023h` (hex addition: `7 + 1C = 23h`, which checks out — matching the hand trace's iteration 3, `100011b = 23h` decimal... let's just double check: `100011b` in decimal is 32+2+1=35, and `23h` in decimal is also 2×16+3=35 ✓). AX doubled to `38h`, BX halved to `1`.

```
-T
AX=0038  BX=0001  CX=001C  DX=0023  SP=00FA  BP=0000  SI=0000  DI=0000
DS=176F  ES=176F  SS=1781  CS=177F  IP=000B   NV UP EI PL NZ AC PO CY
177F:000B F7C30100     TEST    BX,0001

-G17
AX=0070  BX=0000  CX=001C  DX=005B  SP=00FA  BP=0000  SI=0000  DI=0000
DS=176F  ES=176F  SS=1781  CS=177F  IP=0017   NV UP EI PL ZR AC PE CY
177F:0017 75F2     JNZ     000B
```
Final iteration: BX = `1` (lsb = 1, odd) → DX updated one last time: `0023h + 0038h = 005Bh`. Converting to decimal: `05Bh = 5×16+11 = 91` — **exactly matching our hand-computed answer of 91!** ✓ AX doubled to `70h`, and crucially, **BX's right-shift produced 0** this time — notice the flags now show `ZR` (zero flag set), confirming BX genuinely reached zero.

**Why does the loop correctly stop here?** Because `JNZ` (jump if *not* zero) checks the flags left over from the most recent operation affecting them — which was `SHR BX,1`. Since that shift's result was 0, ZF = 1, so the "not zero" condition for `JNZ` is false, and the jump is **skipped** — execution simply falls through past the loop, exactly implementing the algorithm's "UNTIL B = 0" exit condition.

#### Finishing the Trace: POP, POP, and RET

```
-T
AX=0070  BX=0000  CX=001C  DX=005B  SP=00FA  BP=0000  SI=0000  DI=0000
DS=176F  ES=176F  SS=1781  CS=177F  IP=0019   NV UP EI PL ZR AC PE CY
177F:0019 5B     POP     BX

-T
AX=0070  BX=000D  CX=001C  DX=005B  SP=00FC  BP=0000  SI=0000  DI=0000
DS=176F  ES=176F  SS=1781  CS=177F  IP=001A   NV UP EI PL ZR AC PE CY
177F:001A 58     POP     AX

-T
AX=0007  BX=000D  CX=001C  DX=005B  SP=00FE  BP=0000  SI=0000  DI=0000
DS=176F  ES=176F  SS=1781  CS=177F  IP=001B   NV UP EI PL ZR AC PE CY
177F:001B C3     RET
```

Watch BX and AX get **correctly restored** to their original input values — `BX = 000Dh` (13) and `AX = 0007h` (7), exactly what we manually loaded before the procedure started — even though both registers had been completely overwritten by the shifting algorithm along the way. This is the whole point of the earlier `PUSH AX`/`PUSH BX` at the start of MULTIPLY: the procedure faithfully returns these registers to the caller in the same state it found them, regardless of whatever internal scratch work it needed to do with them. Also notice **SP climbing back up**, from `00FAh` → `00FCh` → `00FEh`, exactly 2 bytes per POP, as expected.

**A subtle observation from the book:** dumping the stack at this point,
```
-DSS:F0 FF
1781:00F0  70 00 70 00 07 00 00 00-1B 00 7F 17 A4 13 03 00
```
you'll notice the specific values `000Dh` and `0007h` (BX's and AX's saved copies) are **no longer visible** in this dump, even though we know they were sitting right there just a moment ago. The book is careful to clarify: **this is not because POP erased them.** POP never actually erases the bytes it reads — it simply moves SP past them, and the *old* bytes remain sitting untouched in memory (as garbage, no longer considered "on the stack"). What actually happened here is that **DEBUG itself has been using this same stack space** for its own internal bookkeeping while we've been typing commands, and its own activity has overwritten those particular bytes in the interim. It's a nice reminder that the "stack" the CPU tracks via SP is really just an *interpretation* of ordinary memory — nothing more, nothing magically protected about it.

Finally, tracing the `RET`:
```
-T
AX=0007  BX=000D  CX=001C  DX=005B  SP=0100  BP=0000  SI=0000  DI=0000
DS=176F  ES=176F  SS=1781  CS=177F  IP=0003   NV UP EI PL ZR AC PE CY
177F:0003 B44C     MOV     AH,4C
```
**IP correctly becomes `0003h`** — exactly the return address, sending us right back into MAIN, to the `MOV AH,4CH` line immediately following the original `CALL MULTIPLY`. **SP has returned all the way back to `0100h`** — the stack is completely empty again, exactly matching its state before the call ever happened. Everything has balanced perfectly.

#### Finishing Up

To let the rest of the program (the DOS-exit sequence) run to completion:
```
-G
Program terminated normally
```
Typing a bare `G` (with no offset) simply runs the program to completion.

And finally, to leave DEBUG and return to the DOS prompt:
```
-Q
>C
```
`Q` = **quit**.

---

## Summary (from the book)

- The stack is a **temporary storage area** used by both application programs and the operating system.
- The stack is a **last-in, first-out** data structure. **SS:SP** points to the top of the stack.
- The stack-altering instructions are **PUSH, PUSHF, POP, and POPF.** PUSH adds a new top word to the stack, and POP removes the top word. PUSHF saves the FLAGS register on the stack, and POPF puts the stack top into the FLAGS register.
- **SP decreases by 2** when PUSH or PUSHF is executed, and it **increases by 2** when POP or POPF is executed. SP is initialized to the first word after the stack segment when the program is loaded.
- A **procedure** is a subprogram. Assembly language programs are typically broken into two (or more) procedures. One of the procedures is the **main procedure**, which contains the entry point to the program. Procedures may call other procedures, or themselves.
- There are two kinds of procedures, **NEAR** and **FAR**. A NEAR procedure is in the same code segment as the calling program, and a FAR procedure is in a different segment.
- The **CALL** instruction is used to invoke a procedure. For a NEAR procedure, execution of CALL causes the offset address of the next instruction in line after the CALL to be saved on the stack, and the IP gets the offset of the first instruction in the procedure.
- Procedures end with a **RET** instruction. Its execution causes the stack to be popped into IP, and control returns to the calling program. In order for the return address to be accessible, the procedure must ensure that it is at the top of the stack when RET is executed.
- In assembly language, procedures often pass data through **registers**.

---

## Glossary

| Term | Meaning |
|---|---|
| **direct procedure call** | A procedure call of the form `CALL name` |
| **FAR procedure** | A procedure that can be called by procedures residing in any segment |
| **indirect procedure call** | A procedure call of the form `CALL addr_exp` |
| **NEAR procedure** | A procedure that can only be called by another procedure residing in the same segment |
| **top of the stack** | The last word of data added to the stack |

## New Instructions Introduced This Chapter

`CALL`, `POP`, `POPF`, `PUSH`, `PUSHF`, `RET`

---

That's the complete Chapter 8, Riyad. This chapter closes an important loop: you now know exactly how the stack works mechanically, and — more importantly — you've seen that "procedures," which feel like a totally ordinary, almost invisible feature in every high-level language you've used, are actually built out of nothing more than a stack, a couple of PUSH/POP-like operations (CALL and RET), and the IP register being redirected. There's no hidden magic. This is also a great point to get comfortable with DEBUG as a tool — being able to single-step through your own code, watch registers change in real time, and peek at raw memory is a skill that will keep paying off for the rest of this course (and arguably, for any low-level debugging you ever do in your career, even outside assembly). Ready for Chapter 9 — actual MUL/DIV instructions — whenever you upload it.