# Chapter 17 — Recursion

---

## Overview

Riyad, this chapter answers a question you've probably already wondered about ever since you started writing recursive functions in C++/Java for DSA: **"When I call a function on itself, how does the computer actually keep track of all those in-progress calls without getting confused?"**

In a high-level language, this is completely invisible — the compiler quietly handles it for you. In assembly, there's no such magic. *You* are the compiler now. This chapter shows you exactly what machinery is needed to make recursion work, and — spoiler — it's built entirely out of things you already know from Chapter 8: the **stack**, **PUSH/POP**, and **CALL/RET**. Nothing new is invented; recursion is just a clever, disciplined way of reusing those same tools.

The chapter is organized like this:

- **17.1–17.2** build up the *idea* of recursion from scratch (in case your DSA background with recursion feels shaky), using a binary-tree example and then the classic factorial/find-max examples, but described only in algorithm form (not yet assembly).
- **17.3** introduces a prerequisite skill: how a procedure can receive input parameters *via the stack* (rather than via registers, which was the trick used in Chapter 8). This matters because recursive calls need a *fresh* copy of parameters for every call, and registers can only hold one value at a time — the stack, being able to grow, is the only place with room for many nested copies.
- **17.4** introduces the key new concept of the chapter: the **activation record** — the bundle of information (saved registers, parameters, return address) that must be preserved for *each* in-progress call, stacked one on top of another.
- **17.5** puts it all together and shows fully worked, traced-through assembly implementations of FACTORIAL and FIND_MAX as *actual recursive procedures*.
- **17.6** extends the idea to procedures that make **two** recursive calls in the same invocation (not just one), using the binomial coefficient (Pascal's Triangle) as the worked example.

---

## 17.1 The Idea of Recursion

**What does "recursive" mean?** A process is called **recursive** if it is *defined in terms of itself*. That sounds circular and confusing at first, so let's ground it with the book's example: a **binary tree**.

> A binary tree is either empty, or consists of a single element called the **root**, and whose remaining elements are partitioned into two disjoint subsets (the left and right subtrees), each of which is a binary tree.

Notice the sneaky, self-referential trick here: the definition of "binary tree" uses the phrase "binary tree" *inside itself* ("...each of which is a binary tree"). This is exactly what makes it recursive — to know what a binary tree is, you need to already know what a binary tree is, just applied to a smaller piece of the structure.

**Applying the definition to a concrete tree.** The book uses this tree:

```
        A
       / \
      B   C
     / \
    D   E
```

To *prove* this is a binary tree using the recursive definition, you peel it apart layer by layer:

1. Pick **A** as the root. Its left subtree is **T1** = {B, D, E}, and its right subtree is **T2** = {C}. For the whole thing to be a binary tree, *both* T1 and T2 must themselves be binary trees.
2. Look at **T1**. Pick **B** as its root. Its left subtree is **T1a** = {D}, and its right subtree is **T1b** = {E}.
3. Look at **T1a** = {D}. D has no children, so its left and right subtrees are *empty*. An empty tree is a binary tree by definition (this is the base case!). So T1a is a binary tree. By identical reasoning, T1b is a binary tree too.
4. Since both T1a and T1b are binary trees, **T1** (which is built from them) is a binary tree.
5. Now look at **T2** = {C}. Same story: C's subtrees are empty, so T2 is a binary tree.
6. Since **T1** and **T2** are both binary trees, the whole tree **T** is a binary tree. ∎

**Why walk through this slow, almost pedantic proof?** Because it reveals the *three defining ingredients* of every recursive process — and these three ingredients are the mental checklist you should run through any time you design a recursive algorithm, whether in C++ or in assembly:

1. **The main problem breaks down into smaller versions of the exact same problem.** ("Is T a binary tree?" breaks down into "Is T1 a binary tree?" and "Is T2 a binary tree?" — the *same question*, just asked about smaller pieces.)
2. **There must be an escape case (base case) that lets the recursion actually terminate.** Here, that's the empty tree — you don't need to ask "is this a binary tree?" recursively anymore once you hit nothing. Without an escape case, the recursion would try to go on forever (and in a real program, this crashes — more on that later in the chapter with stack overflow implications).
3. **Once a subproblem is solved, work resumes on the next part of the original problem.** After confirming T1 is a binary tree, you don't just stop — you go back and also check T2. This "resuming where you left off" detail is exactly the part that requires careful bookkeeping in assembly, and it's the whole point of this chapter.

---

## 17.2 Recursive Procedures

A **recursive procedure** is simply a procedure that calls **itself**. Let's build intuition with the two workhorse examples of this chapter.

### Example 1 — Factorial

You already know factorial: `N! = N × (N-1) × (N-2) × ... × 2 × 1`. The book first writes this the "obvious" non-recursive way:

```
FACTORIAL(1) = 1
FACTORIAL(N) = N × (N-1) × (N-2) × ... × 2 × 1     if N > 1
```

Now here's the clever rewrite. Notice that the tail end of that big product, `(N-1) × (N-2) × ... × 2 × 1`, is *exactly* the definition of `FACTORIAL(N-1)`. So we can substitute:

```
FACTORIAL(1) = 1
FACTORIAL(N) = N × FACTORIAL(N-1)     if N > 1
```

This is the recursive definition — "factorial" now appears on both sides. **Intuition:** to compute 4!, you don't need to know how to multiply four numbers together in one shot — you just need to know 3! (someone else's job, i.e., a smaller copy of your own job) and then multiply the answer by 4. And to know 3!, you need 2!, and so on, until you hit 1! = 1, which you just *know* outright (the escape case).

**Turning this into an algorithm** (pseudocode, not yet assembly):

```
1: PROCEDURE FACTORIAL (input: N, output: RESULT)
2: IF N = 1
3:    THEN
4:       RESULT = 1
5:    ELSE
6:       call FACTORIAL (input: N - 1, output: RESULT)
7:       RESULT = N × RESULT
8: END_IF
9: RETURN
```

**Reading line 7 carefully:** the `RESULT` on the *right-hand side* of `RESULT = N × RESULT` is *not* the same as the `RESULT` being computed on this line — it's whatever value the recursive call at line 6 handed back. This subtlety (the same variable name meaning "the answer from the call I just made" on the right side, and "the answer I'm producing now" on the left side) is exactly the kind of bookkeeping that assembly will force you to make explicit, since assembly has no automatic per-call variable scoping.

**Tracing FACTORIAL(4) by hand.** The book writes out each nested call:

```
call FACTORIAL(4, RESULT)   /* begin first call */
call FACTORIAL(3, RESULT)   /* begin second call */
call FACTORIAL(2, RESULT)   /* begin third call */
call FACTORIAL(1, RESULT)   /* begin fourth call */
RESULT = 1                  /* the escape case */
RETURN                      /* end fourth call */
```

Picture this as a stack of open, *unfinished* function calls piling up — the first call opens, but before it can finish, it needs to open a second call, which needs to open a third, and so on. **None of these calls can complete until the innermost one (the escape case) finishes first.** This is the recursive equivalent of Russian nesting dolls — you can't close the outer doll until you've closed every doll inside it, starting from the smallest.

Once the fourth call hits `N = 1`, it takes the escape branch, sets `RESULT = 1`, and returns. Now, unwinding back out:

- **Third call resumes** at line 7. Here `N = 2`, and `RESULT` (from the call it just made) is `1`. So it computes `RESULT = 2 × 1 = 2`. This call now finishes.
- **Second call resumes** at line 7. Here `N = 3`, and `RESULT` (from the call it just made) is `2`. So it computes `RESULT = 3 × 2 = 6`. This call finishes.
- **First call resumes** at line 7. Here `N = 4`, and `RESULT` is `6`. So it computes `RESULT = 4 × 6 = 24`. Done — `24` is handed back to whoever originally called `FACTORIAL(4)`.

**Notice the pattern of the three ingredients from 17.1:** the problem broke into smaller copies of itself (each call handles `N-1`), there was an escape case (`N=1`), and after each subproblem finished, work resumed exactly where it left off in the *calling* invocation.

### Example 2 — FIND_MAX (largest element of an array)

Same idea, applied to a different problem: find the largest of `N` integers in array `A`.

**The recursive insight:** if there's only one element, that one element *is* the max (escape case). Otherwise, the max of the whole array is either the *last* element `A[N]`, or the max of *everything before it*, `A[1] ... A[N-1]` — whichever is bigger. And "the max of `A[1]...A[N-1]`" is just... the same problem, on a smaller array.

```
1:  PROCEDURE FIND_MAX (input: N, output: MAX)
2:  IF N = 1
3:     THEN
4:        MAX = A[1]
5:     ELSE
6:        call FIND_MAX(N-1, MAX)
7:        IF A[N] > MAX
8:           THEN
9:              MAX = A[N]
10:          ELSE
11:             MAX = MAX     /* value returned by call at line 6 */
12:          END_IF
13: RETURN
```

**Tracing with A = {10, 50, 20, 4}:**

```
call FIND_MAX(4, MAX)   /* first call */
call FIND_MAX(3, MAX)   /* second call */
call FIND_MAX(2, MAX)   /* third call */
call FIND_MAX(1, MAX)   /* fourth call */
```

- **Fourth call (escape case):** `N=1`, so `MAX = A[1] = 10`. Returns.
- **Third call resumes** at line 7: is `A[2]` (=50) `>` `MAX` (=10)? Yes → `MAX = 50`.
- **Second call resumes** at line 7: is `A[3]` (=20) `>` `MAX` (=50)? No → `MAX` stays `50`.
- **First call resumes** at line 7: is `A[4]` (=4) `>` `MAX` (=50)? No → `MAX` stays `50`.

Final answer: **50** — correctly the largest of {10, 50, 20, 4}. Notice how the comparisons happen in *reverse* order of the array (checking `A[4]` last, even though it's the last element of the array) — this reverse unwinding order is a hallmark of recursion, and you'll see it again as a stack-related theme throughout this chapter (recall Chapter 8's reversal trick — recursion has a similar "LIFO flavor" baked into it, precisely *because* it's implemented using a stack).

---

## 17.3 Passing Parameters on the Stack

Before we can write recursion in assembly, we need a new skill: **how to hand parameters to a procedure using the stack**, instead of using registers like Chapter 8's `MULTIPLY` procedure did (which used `AX` and `BX` directly).

**Why can't we just keep using registers for recursion?** Because a recursive call is going to be *nested inside another call to the same procedure* — and there are only so many registers. If call #1 is using `AX` to hold its value of `N`, and it then calls itself again, call #2 would need to also use `AX` for *its* `N`... except that would overwrite call #1's value before call #1 is done needing it! You'd lose data. The stack, on the other hand, can grow arbitrarily deep (up to the size of the stack segment) — every nested call gets its *own fresh slot* on the stack, so nothing gets overwritten.

### Worked Example — PGM17_1.ASM (ADD_WORDS)

This example doesn't use recursion yet — it's purely there to teach you the *mechanics* of stack-based parameter passing, which recursion will then build on top of.

```asm
TITLE PGM17_1: ADD WORDS
.MODEL SMALL
.STACK 100H
.DATA
WORD1   DW   2
WORD2   DW   5
.CODE
MAIN    PROC
        MOV     AX, @DATA
        MOV     DS, AX
        PUSH    WORD1
        PUSH    WORD2
        CALL    ADD_WORDS
        MOV     AH, 4CH
        INT     21H
MAIN    ENDP
ADD_WORDS       PROC        NEAR
; adds two memory words
; stack on entry:  return addr.(top), word2, word1
; output: AX = sum
        PUSH    BP              ; save BP
        MOV     BP, SP          ; BP pts to stacktop
        MOV     AX, [BP+6]      ; AX gets WORD1
        ADD     AX, [BP+4]      ; AX has sum
        POP     BP              ; restore BP
        RET     4               ; exit
ADD_WORDS       ENDP
        END     MAIN
```

**Walking through this step by step, Riyad:**

**Step 1 — MAIN pushes the parameters.** `PUSH WORD1` then `PUSH WORD2` puts both values on the stack *before* calling the procedure. Since `WORD1 = 2` and `WORD2 = 5`, and recall from Chapter 8 that PUSH makes SP decrease and the most-recently-pushed value ends up on top: after both pushes, the stack (top to bottom) holds `5` (WORD2, pushed last, on top), then `2` (WORD1, underneath it).

**Step 2 — CALL pushes the return address too.** Recall from Chapter 8's mechanics: `CALL ADD_WORDS` pushes the offset of the instruction right after the call (`MOV AH,4CH`) onto the stack, *then* jumps to `ADD_WORDS`. So right when `ADD_WORDS` starts executing, the stack looks like this (SP points to the top):

```
SP → | return_address (offset of line after CALL) |
     | 5   (WORD2 content)                          |
     | 2   (WORD1 content)                           |
```

**Step 3 — Why we can't just use SP to reach into the stack.** You might think: "well, SP already points near this data, can't I just use `[SP+2]` or similar to grab WORD1?" The book flags an important 8086 restriction here: **SP itself cannot be used as a base register in indirect/based addressing** (i.e., you can't write `MOV AX,[SP+6]`). This is simply a hardware limitation of the 8086's addressing modes — SP is special-purpose and wasn't designed to double as a general "point into memory" register.

**Step 4 — Enter BP.** This is why the procedure does:
```asm
PUSH BP
MOV BP, SP
```
`BP` (Base Pointer) *can* legally be used in based addressing (`[BP+offset]`), and — bonus detail worth remembering — **whenever BP is used in an addressing expression, the CPU automatically assumes the segment is SS** (the stack segment), exactly the segment we need to reach into here. This is a hardware convention, not something you configure — it's simply how BP behaves differently from other general registers like BX, which default to DS.

So: first we `PUSH BP` to save whatever the *calling* program was using BP for (defensive save, exactly like Chapter 8's `PUSH AX`/`PUSH BX` habit), and *then* we do `MOV BP, SP`, which copies the current stack-top offset into BP. Now BP is "anchored" at this position, and — crucially — even though SP itself will change value later as the procedure does other pushes/pops internally, **BP stays fixed at this anchor point**, giving us a stable reference point to reach into the stack data. After these two lines, the stack looks like:

```
SP → | original BP value                             |  ← BP now points here too
     | return address                                  |
     | 5                                                |
     | 2                                                |
```

**Step 5 — Computing the offsets.** Each stack slot is a word (2 bytes), and BP currently points at the topmost slot (the saved BP value itself, at offset `BP+0`). Counting upward from there in 2-byte steps:
- `BP+0` → saved BP value
- `BP+2` → return address
- `BP+4` → WORD2's content (5) — pushed *last*, but it's the *second* word up from BP because the saved BP + return address sit above it
- `BP+6` → WORD1's content (2) — pushed *first*, so it's furthest from the top

This exactly matches the code:
```asm
MOV     AX, [BP+6]      ; AX gets WORD1 (the value 2)
ADD     AX, [BP+4]      ; AX has sum   (adds WORD2, the value 5) → AX = 7
```

**Step 6 — Cleaning up and returning.** `POP BP` restores BP to whatever it held before this procedure started (undoing the earlier `PUSH BP`), and the stack shrinks back to:
```
SP → | return address |
     | 5                |
     | 2                |
```
Then comes `RET 4`. Recall the `RET n` variant from Chapter 8: it pops the return address into IP (sending control back to MAIN), **and then additionally adds `4` to SP**, which discards 4 more bytes — exactly enough to throw away the two pushed parameter words (`5` and `2`) without needing separate POP instructions for them. After `RET 4`, the stack is completely back to its pre-call state (`SP = 0100h` again), and AX holds the answer, `7`.

**Why `RET 4` and not `RET 2` or something else?** Because *two words* (4 bytes total) were pushed as parameters (`WORD1` and `WORD2`), and it's the *caller's* job to have pushed them, but it's conventionally the *callee's* (procedure's) job to clean them off the stack before returning, using the `RET n` shortcut — this keeps MAIN's code simpler, since it doesn't need extra `ADD SP,4` cleanup instructions of its own after the `CALL`.

---

## 17.4 The Activation Record

Now we get to the real conceptual heart of the chapter.

**The problem this section solves:** Every time a recursive procedure calls itself, it needs a **completely separate, private copy** of its parameters and local variables for that particular call — because each nested call is working on a different piece of the problem (recall `FACTORIAL(4)` calling `FACTORIAL(3)` calling `FACTORIAL(2)`... each with a *different* value of N). Also, each call needs to remember exactly which line of code to resume at once the call it made returns.

This bundle — *"the saved BP, the return address, and the parameters/local variables belonging to one specific call"* — is called the **activation record** of that call. Think of it as a labeled folder that gets created fresh every time a call begins, and destroyed the moment that call finishes. Because the stack is LIFO, and because calls nest inside each other (the newest call is always the "most recently started, not-yet-finished" one), **the stack is a perfect natural fit for storing a growing pile of activation records** — the most recent call's activation record always sits on top, exactly where you'd need to look first to resume it.

**Building the picture with an example: a procedure called once by MAIN, which then calls itself twice more (so, 3 total invocations).**

**After the first call begins** (MAIN placed the initial parameters on the stack, then called the procedure, which does its usual `PUSH BP` / `MOV BP,SP`):

```
SP and BP → | original BP value                      |  ⎫
            | return address (in main procedure)      |  ⎬ activation record — first call
            | parameters                               |  ⎭
```

**Before making its recursive call, the procedure pushes a fresh activation record for the *next* call** (new parameters, then CALL — which pushes a new return address — then inside the new invocation, `PUSH BP` / `MOV BP,SP` again). The stack now becomes:

```
SP and BP → | saved BP value (from first call)          |  ⎫
            | return address (in first call)             |  ⎬ activation record — second call
            | parameters and local variables             |  ⎭
            |------------------------------------------- |
            | original BP                                 |  ⎫
            | return address (in main procedure)          |  ⎬ activation record — first call
            | parameters                                   |  ⎭
```

**One more recursive call, and a third activation record stacks on top:**

```
SP and BP → | saved BP value (from second call)          |  ⎫
            | return addr (in second call)                |  ⎬ activation record — third call
            | parameters and local variables              |  ⎭
            |------------------------------------------- |
            | original BP value (from first call)          |  ⎫
            | return addr (in first procedure)              |  ⎬ activation record — second call
            | parameters and local variables                |  ⎭
            |------------------------------------------- |
            | original BP                                    |  ⎫
            | return addr (in main procedure)                |  ⎬ activation record — first call
            | parameters                                       |  ⎭
```

Notice the structure: it's literally a **stack of stack frames** — each activation record is itself a mini stack-frame, and they pile on top of each other exactly like PUSHed items would, just each "item" here happens to be a multi-word bundle instead of a single word.

**Now suppose the third call is the escape case.** It computes its answer, places it somewhere accessible (a register, per our convention from Chapter 8's documentation style), and returns:

1. **POP BP** restores BP to what it was for the *second* call (undoing the third call's `PUSH BP`).
2. A **RET** (with the appropriate `n` to discard that call's parameters/locals) pops the saved return address into IP, sending execution back into the *second* call, exactly at the point right after where it made its recursive call.
3. As part of `RET n`, the third call's parameters/local variables are also discarded from the stack.

The stack shrinks back down to just the second and first calls' activation records:

```
SP and BP → | saved BP value (from first call)          |  ⎫
            | return addr (in first call)                 |  ⎬ activation record — second call
            | parameters and local variables               |  ⎭
            |------------------------------------------- |
            | original BP                                    |  ⎫
            | return addr (in main procedure)                 |  ⎬ activation record — first call
            | parameters                                        |  ⎭
```

The **second call now resumes**, picks up the result from the third call, finishes its own work, and returns in exactly the same fashion — popping BP, returning via RET, discarding its own frame. This leaves only the first call's frame:

```
SP and BP → | original BP value            |  ⎫
            | return address (in main procedure) | ⎬ activation record — first call
            | parameters                          | ⎭
```

Finally, the **first call resumes**, finishes, restores BP one last time, and control passes back to MAIN — with the stack now completely back to empty, exactly as it started.

**The big-picture takeaway, Riyad:** at *any* single moment during a chain of recursive calls, the stack literally contains a visual, ordered history of "who called whom" — the deepest, most-recently-started call is always on top, and unwinding proceeds strictly in reverse order of how the calls were made (again, that LIFO signature you first met in Chapter 8's stack-reversal trick). This is *precisely* how your compiler implements recursion in C++/Java under the hood too — the only difference is that there, it's invisible; here, you're writing every `PUSH BP` / `MOV BP,SP` / `POP BP` / `RET n` yourself.

---

## 17.5 Implementation of Recursive Procedures

Time to combine everything: 17.2's algorithms, 17.3's stack-parameter-passing mechanics, and 17.4's activation-record bookkeeping, into real, runnable assembly.

### Example 17.1 — Coding FACTORIAL

**Goal:** implement the FACTORIAL procedure from 17.2, and test it computing 3!.

```asm
TITLE PGM17_2: FACTORIAL PROGRAM
.MODEL SMALL
.STACK 100H
.CODE
MAIN     PROC
        MOV     AX, 3           ; N = 3
        PUSH    AX              ; N on stack
        CALL    FACTORIAL       ; AX has 3 factorial
        MOV     AH, 4CH
        INT     21H             ; dos return
MAIN     ENDP
FACTORIAL       PROC        NEAR
; computes N factorial
; input: stack on entry - ret. addr.(top), N
; output AX
        PUSH    BP              ; save BP
        MOV     BP, SP          ; BP pts to stacktop
; if
        CMP     WORD PTR [BP+4], 1  ; N = 1?
        JG      END_IF          ; no, N>1
; then
        MOV     AX, 1           ; result = 1
        JMP     RETURN          ; go to return
END_IF:
        MOV     CX, [BP+4]      ; get N
        DEC     CX              ; N-1
        PUSH    CX              ; save on stack
        CALL    FACTORIAL       ; recursive call
        MUL     WORD PTR [BP+4] ; RESULT = N*RESULT
RETURN:
        POP     BP              ; restore BP
        RET     2               ; return and discard N
FACTORIAL       ENDP
        END     MAIN
```

**Walking through line by line:**

- **MAIN:** simply loads `3` into AX, pushes it onto the stack (this is `N`, our parameter), and calls `FACTORIAL`. After the procedure returns, AX will hold the answer.

- **`PUSH BP` / `MOV BP,SP`:** identical pattern to 17.3's `ADD_WORDS` — save the caller's BP, then anchor our own BP at the current stack top so we can address our parameter.

- **`CMP WORD PTR [BP+4], 1`:** this checks whether N equals 1 (our escape case). Notice the offset is `BP+4`, not `BP+6` like in the earlier ADD_WORDS example — that's because here we only pushed *one* parameter (N), not two, so counting up from BP: `BP+0` = saved BP, `BP+2` = return address, `BP+4` = N. **Why `WORD PTR`?** The book explains this is required because the assembler can't tell, just from the bare number `1` on the right side of the comparison, whether you mean to compare a byte or a word at that memory location — `WORD PTR` removes the ambiguity by explicitly telling the assembler "treat this memory location as a 16-bit word."

- **`JG END_IF`:** "jump if greater" — if N > 1 (i.e., the comparison found N is *not* equal to 1, and specifically bigger), skip past the escape-case code, since this call needs to recurse instead.

- **Escape case (N = 1):** `MOV AX,1` sets the result to 1, and `JMP RETURN` skips straight to the cleanup/return code, bypassing the recursive-call logic entirely.

- **`END_IF:` — the recursive branch (N > 1):**
  ```asm
  MOV   CX, [BP+4]   ; get N
  DEC   CX           ; N - 1
  PUSH  CX           ; save N-1 on stack — this is the parameter for the NEXT call
  CALL  FACTORIAL    ; recursive call!
  MUL   WORD PTR [BP+4]   ; RESULT = N * RESULT
  ```
  This is the heart of the whole chapter, Riyad — look closely. We fetch our *own* N (via `[BP+4]`), decrement it to get `N-1`, push *that* onto the stack as the parameter for the next call, and then `CALL FACTORIAL` **again** — the procedure literally invoking itself. When that inner call eventually returns, by convention (documented at the top: "output AX") its answer sits in AX. The line `MUL WORD PTR [BP+4]` then multiplies AX by *our own* N (again fetched via `[BP+4]`, which is still valid because BP hasn't moved!) — implementing exactly `RESULT = N × RESULT` from the algorithm. `MUL` here is the actual multiply instruction (you'll meet it properly in Chapter 9) — for now just know `MUL WORD PTR[BP+4]` multiplies AX by the word at that memory address, leaving the product in AX (technically DX:AX for a full 32-bit product, but for these small values it fits in AX alone).

  **Crucial subtlety worth sitting with:** even though this procedure just called *itself*, and that inner call also did `PUSH BP`/`MOV BP,SP` and used its *own* `[BP+4]`, our *outer* call's BP is completely undisturbed by all of that — because the inner call's `PUSH BP` saved a *copy* of our BP value on the stack, and when the inner call finishes with `POP BP`, it puts that exact value back. Our own BP is safe throughout, still correctly anchored on our own activation record. This is exactly what makes recursion *work* — every level of nesting gets its own private, uncorrupted "view" of its own data.

- **`RETURN:` — cleanup:**
  ```asm
  POP  BP     ; restore BP
  RET  2      ; return and discard N
  ```
  `POP BP` undoes our `PUSH BP`. `RET 2` returns control to the caller and discards 2 bytes (our one pushed parameter, N) from the stack — leaving everything perfectly balanced.

**Tracing FACTORIAL(3) step by step, exactly as the book walks through it:**

The test program pushes `3` and calls FACTORIAL. Inside, `PUSH BP`/`MOV BP,SP` runs, giving:

```
SP → | (original BP)                |  ← BP (first call)
     | return addr (line 8, in MAIN) |
     | 3  (value of N)                 |
```

At `CMP WORD PTR[BP+4],1`, since N=3≠1, we fall into the recursive branch: fetch N (3), decrement to 2, push `2`, and `CALL FACTORIAL` again. Inside this **second call**, `PUSH BP`/`MOV BP,SP` runs again:

```
SP → | BP (first call)                  |  ← BP (second call)
     | return addr (line 28, inside first call's MUL instruction) |
     | 2                                  |
     |------------------------------------ |
     | BP (original)                        |
     | return addr (line 8)                  |
     | 3                                       |
```

Since N=2≠1 in the second call, it also recurses: fetch N (2), decrement to 1, push `1`, and `CALL FACTORIAL` a **third time**:

```
SP → | BP (second call)                  |  ← BP (third call)
     | return addr (line 28)                |
     | 1                                      |
     |------------------------------------ |
     | BP (first call)                        |
     | return addr (line 28)                    |
     | 2                                          |
     |------------------------------------ |
     | BP (original)                              |
     | return addr (line 8)                          |
     | 3                                               |
```

**Now, in the third call, N=1 — the escape case fires!** `MOV AX,1` sets AX=1, and we jump to `RETURN`. `POP BP` restores BP to the **second call's** BP value, and `RET 2` pops the saved return address (which points to the `MUL WORD PTR[BP+4]` line, back inside the *second* call) into IP, and discards the pushed `1` (N for the third call) from the stack. The stack is now back to just two frames:

```
SP → | BP (first call)              |  ← BP (second call)
     | return addr (line 28)          |
     | 2                                 |
     |---------------------------- |
     | BP (original)                     |
     | return addr (line 8)                |
     | 3                                     |
```

**Execution resumes in the second call, right at `MUL WORD PTR[BP+4]`.** AX currently holds `1` (returned from the third call). `[BP+4]` here refers to the *second call's* N, which is `2`. So `AX = AX × 2 = 1 × 2 = 2`. The second call then falls through to `RETURN:`, does `POP BP` (restoring to the **first call's** BP) and `RET 2` (returning to the line right after the *first* call's `CALL FACTORIAL`, which is the `MUL WORD PTR[BP+4]` line inside the first call, and discarding the second call's pushed `2`). Stack now:

```
SP → | BP (original)         |  ← BP (first call)
     | return addr (line 8)     |
     | 3  (value of N)             |
```

**Execution resumes in the first call, at `MUL WORD PTR[BP+4]`.** AX = 2 (from the second call). `[BP+4]` here is the *first call's* N = `3`. So `AX = 2 × 3 = 6`. The first call finishes, restores BP to its original (pre-call) value, and `RET 2` sends control all the way back into MAIN, right after `CALL FACTORIAL`, discarding the last pushed parameter. **Final result: AX = 6 = 3!** — exactly correct.

**Notice the multiplication order:** `1 × 2 = 2`, then `2 × 3 = 6` — this is precisely `1 × 2 × 3`, matching the definition of 3!, but computed by *unwinding* the recursion from the innermost call outward, exactly mirroring the hand-trace from section 17.2.

### Example 17.2 — Coding FIND_MAX

```asm
TITLE PGM17_3: FIND_MAX
.MODEL SMALL
.STACK 100H
.DATA
A                DW      10, 50, 20, 4
.CODE
MAIN     PROC
        MOV     AX, @DATA
        MOV     DS, AX          ; initialize DS
        MOV     AX, 4           ; no. of elts in array
        PUSH    AX              ; parameter on stack
        CALL    FIND_MAX        ; returns MAX in AX
        MOV     AH, 4CH
        INT     21H             ; dos exit
MAIN     ENDP
FIND_MAX         PROC       NEAR
; finds the largest element in array A of N elements
; input: stack on entry - ret. addr.(top), N
; output: AX largest element
        PUSH    BP              ; save BP
        MOV     BP, SP          ; BP pts to stacktop
; if
        CMP     WORD PTR [BP+4], 1  ; N=1?
        JG      ELSE_           ; no, go to set up next call
; then
        MOV     AX, A           ; MAX = A[1]
        JMP     END_IF1
ELSE_:
        MOV     CX, [BP+4]      ; get N
        DEC     CX              ; N-1
        PUSH    CX              ; save on stack
        CALL    FIND_MAX        ; returns MAX in AX
; if
        MOV     BX, [BP+4]      ; get N
        SHL     BX, 1           ; 2N
        SUB     BX, 2           ; 2(N-1)
        CMP     A[BX], AX       ; A[N] > MAX?
        JLE     END_IF1         ; no, go to return
; then
        MOV     AX, A[BX]       ; yes, set MAX = A[N]
END_IF1:
        POP     BP              ; restore BP
        RET     2               ; return and discard N
FIND_MAX         ENDP
        END     MAIN
```

**What's new here compared to FACTORIAL:**

- The escape case (`N=1`) sets `AX = A` (i.e., `A[1]`, since `A` alone refers to the first word of the array — a convention from Chapter 10 that you should already be comfortable with).

- The recursive branch fetches N, decrements it, pushes it, and calls `FIND_MAX` recursively — same pattern as FACTORIAL.

- **The tricky part is computing the memory address of `A[N]`.** Recall from Chapter 10: for a **word** array, the offset of the Nth element (1-indexed) is `A + 2×(N-1)` — you multiply by 2 because each element is a word (2 bytes), and you subtract 1 because arrays here are conventionally 1-indexed but memory offsets are 0-indexed from the array's base. The code computes this as:
  ```asm
  MOV   BX, [BP+4]   ; get N
  SHL   BX, 1        ; BX = 2×N   (shift-left-by-1 = multiply by 2, from Ch.7)
  SUB   BX, 2        ; BX = 2×N - 2 = 2×(N-1)
  ```
  Then `A[BX]` uses **based addressing** — recall from your addressing-modes material that `A[BX]` means "the word at address `A + BX`", i.e., exactly `A + 2×(N-1)`, which is precisely the offset formula. This lets `CMP A[BX], AX` correctly compare the *current* array element `A[N]` against `MAX` (which is sitting in AX, having just come back from the recursive call).

- **`JLE END_IF1`:** "jump if less than or equal" — if `A[N] ≤ MAX`, there's nothing to do (AX already holds the correct MAX from the recursive call), so we skip straight to cleanup. **Notice the book's comment: "the ELSE statement at line 11 of the algorithm need not be coded"** — this is a nice practical simplification: since the algorithm's `ELSE` branch just says `MAX = MAX` (i.e., "do nothing, keep the value you already have"), and AX already holds that value from the CALL, there's genuinely no code needed for that branch — a good example of translating pseudocode into leaner real assembly by recognizing redundant branches.

- If `A[N] > MAX`, we fall through to `MOV AX, A[BX]`, updating AX to the new, larger value.

**The stacking behavior during the recursive calls is exactly analogous to Example 17.1** — the book notes this explicitly and leaves it as an exercise to trace fully, since by this point the pattern (nested activation records, each with its own N, resolved from the innermost escape case outward) should already feel familiar to you.

---

## 17.6 More Complex Recursion

So far, every recursive procedure we've written makes **exactly one** recursive call per invocation (FACTORIAL calls itself once; FIND_MAX calls itself once). But recursion isn't limited to that — a procedure can make **multiple** recursive calls within a single invocation. This section's example: computing **binomial coefficients**, i.e., the numbers that show up in Pascal's Triangle.

### The Math Background

You may recognize `C(n, k)` from combinatorics — it's "n choose k," the coefficients that appear when you expand `(x + y)ⁿ`:

```
(x+y)ⁿ = C(n,0)xⁿy⁰ + C(n,1)xⁿ⁻¹y¹ + C(n,2)xⁿ⁻²y² + ... + C(n,n-1)x¹yⁿ⁻¹ + C(n,n)x⁰yⁿ
```

These same numbers form **Pascal's Triangle**:

```
             C(0,0)
          C(1,0)  C(1,1)
       C(2,0)  C(2,1)  C(2,2)
    C(3,0)  C(3,1)  C(3,2)  C(3,3)
 C(4,0)  C(4,1)  C(4,2)  C(4,3)  C(4,4)
```

which evaluates numerically to the familiar triangle:

```
        1
       1 1
      1 2 1
     1 3 3 1
    1 4 6 4 1
```

**The recursive relation** that generates this triangle:

```
C(n, n) = C(n, 0) = 1                                    (the edges are always 1)
C(n, k) = C(n-1, k) + C(n-1, k-1)     if n > k > 0        (interior entries)
```

**Intuition, Riyad — this is worth really sitting with:** every interior number in Pascal's triangle is the sum of the two numbers *directly above it* (upper-left and upper-right). Look at the `3` in the row `1 3 3 1` — it sits below the `1` and `2` in the row above it, and `1+2=3`. That's *exactly* what `C(n,k) = C(n-1,k) + C(n-1,k-1)` says algebraically.

**Worked hand-trace of C(3,2)** (the book's example):

```
C(3,2) = C(2,2) + C(2,1)
C(2,2) = 1                          (edge case: k=n)
C(2,1) = C(1,1) + C(1,0)
C(1,1) = 1                          (edge case: k=n)
C(1,0) = 1                          (edge case: k=0)
So C(2,1) = 1 + 1 = 2
And C(3,2) = 1 + 2 = 3
```

**The algorithm:**

```
PROCEDURE BINOMIAL (input: N, K; output: RESULT)
IF (K = N) OR (K = 0)
   THEN
      RESULT = 1
   ELSE
      CALL BINOMIAL(N-1, K,   RESULT1)
      CALL BINOMIAL(N-1, K-1, RESULT2)
      RESULT = RESULT1 + RESULT2
RETURN
```

**Two things distinguish BINOMIAL from FACTORIAL/FIND_MAX, and the book calls both out explicitly:**

1. **There are *two* escape cases** (`K=N` or `K=0`), not just one — both correspond to the "edges" of Pascal's triangle, which are always 1.
2. **The general (non-escape) case makes *two* separate recursive calls**, not one — computing `C(n-1,k)` and `C(n-1,k-1)` independently, then adding their results together.

### Example 17.3 — Coding BINOMIAL

```asm
TITLE PGM17_4: BINOMIAL COEFFICIENTS
.MODEL SMALL
.STACK 100H
.CODE
MAIN     PROC
        MOV     AX, 2           ; K=2
        PUSH    AX
        MOV     AX, 3           ; N=3
        PUSH    AX
        CALL    BINOMIAL        ; AX = RESULT
        MOV     AH, 4CH
        INT     21H             ; DOS EXIT
MAIN     ENDP
BINOMIAL         PROC       NEAR
        PUSH    BP
        MOV     BP, SP
        MOV     AX, [BP+6]      ; get K
; if
        CMP     AX, [BP+4]      ; K=N?
        JE      THEN            ; yes, nonrecursive case
        CMP     AX, 0           ; K=0?
        JNE     ELSE_           ; no, recursive case
THEN:
        MOV     AX, 1           ; RESULT = 1
        JMP     RETURN
ELSE_:
; compute C(N-1,K)
        PUSH    [BP+6]          ; save K
        MOV     CX, [BP+4]      ; get N
        DEC     CX              ; N-1
        PUSH    CX              ; save N-1
        CALL    BINOMIAL        ; RESULT1 in AX
        PUSH    AX              ; save RESULT1
; compute C(N-1,K-1)
        MOV     CX, [BP+6]      ; get K
        DEC     CX              ; K-1
        PUSH    CX              ; save K-1
        MOV     CX, [BP+4]      ; get N
        DEC     CX              ; N-1
        PUSH    CX              ; save N-1
        CALL    BINOMIAL        ; RESULT2 in AX
; compute C(N,K)
        POP     BX              ; get RESULT1
        ADD     AX, BX          ; RESULT = RESULT1 + RESULT2
RETURN:
        POP     BP              ; restore BP
        RET     4               ; return and discard N and K
BINOMIAL         ENDP
        END     MAIN
```

**Walking through the setup:** MAIN needs to push **two** parameters this time (K and N), and — pay close attention to the *order* — it pushes `K` first, then `N`. This determines the stack layout inside BINOMIAL: since N was pushed *last* (closer to the top), it ends up at the *smaller* offset from BP. Counting up: `BP+0`=saved BP, `BP+2`=return address, `BP+4`=N (pushed second/last), `BP+6`=K (pushed first). That's why the code reads `[BP+6]` for K and `[BP+4]` for N.

**The IF checking both escape conditions:**
```asm
MOV   AX, [BP+6]      ; get K
CMP   AX, [BP+4]      ; K=N?
JE    THEN             ; yes, nonrecursive case
CMP   AX, 0             ; K=0?
JNE   ELSE_               ; no, recursive case
```
This first checks `K == N`; if so, jump straight to the escape branch (`THEN:`). If not, it checks `K == 0`; if that's also *not* true (`JNE`, jump if not equal), we go to the general recursive case (`ELSE_:`). Falling through past both jumps (i.e., if `K==0` *was* true) also lands at `THEN:` — so both escape conditions correctly funnel into the same `RESULT=1` code. This is a clean way to implement an `OR` condition (`K=N OR K=0`) using two sequential comparisons, a pattern worth remembering for your own conditional logic in CSE314/318 work too.

**The recursive branch — two calls, carefully sequenced:**

*First recursive call — computing C(N-1, K):*
```asm
PUSH    [BP+6]      ; save K  (unchanged — same K for the sub-problem)
MOV     CX, [BP+4]  ; get N
DEC     CX           ; N-1
PUSH    CX             ; save N-1
CALL    BINOMIAL         ; RESULT1 in AX
PUSH    AX                 ; save RESULT1
```
Notice the push order here matches MAIN's original convention: push `K` first, then push `N-1` — exactly mirroring "push K, then push N" from the outer call, so the recursive call's own `[BP+6]`/`[BP+4]` addressing works identically inside the nested invocation. After the call returns with `RESULT1` in AX, we immediately `PUSH AX` — **this is essential**, because we're about to make a *second* recursive call, which will itself use AX for its own return value, overwriting whatever's currently there. Stashing `RESULT1` safely on the stack (rather than leaving it in a register) is exactly the kind of defensive habit Chapter 8 taught you, now put to real use.

*Second recursive call — computing C(N-1, K-1):*
```asm
MOV     CX, [BP+6]  ; get K
DEC     CX            ; K-1
PUSH    CX              ; save K-1
MOV     CX, [BP+4]        ; get N
DEC     CX                  ; N-1
PUSH    CX                    ; save N-1
CALL    BINOMIAL                ; RESULT2 in AX
```
Same pattern, but this time both parameters change from the *original* call's values: K becomes `K-1`, and N becomes `N-1`. After this returns, `RESULT2` is in AX.

*Combining the two results:*
```asm
POP     BX          ; get RESULT1 (the value we stashed earlier)
ADD     AX, BX       ; RESULT = RESULT1 + RESULT2
```
Since AX currently holds RESULT2, and we pop our earlier-saved RESULT1 into BX, adding them gives the final answer in AX — implementing `RESULT = RESULT1 + RESULT2` from the algorithm.

**Cleanup:** `POP BP` restores BP, and `RET 4` discards **4 bytes** (both pushed parameters, K and N — 2 bytes each) as it returns, since this procedure's caller pushed two words as input.

**Tracing C(3,2):** MAIN pushes K=2, then N=3, then calls BINOMIAL. Since K(2) ≠ N(3) and K(2) ≠ 0, we take the recursive branch. First call computes `C(2,2)` → escape case (K=N) → returns `1`. This gets pushed as RESULT1. Second call computes `C(2,1)`, which is itself *not* an escape case (K=1, N=2, neither K=N nor K=0), so *it* recursively computes `C(1,1)` (escape, =1) and `C(1,0)` (escape, =1), summing to `2`. Back in the outer call: `RESULT1 (1) + RESULT2 (2) = 3` — **matching the hand-trace exactly.** The book encourages you to fully diagram the stack through this trace yourself (as practice) the same way Example 17.1 was diagrammed step-by-step above — a great exercise to actually cement this in your head, Riyad, especially since two *interleaved* recursive calls (rather than just one) is the trickiest part of this whole chapter.

---

## Summary (from the book)

- Recursive problem solving has three characteristics: (1) the main problem breaks down into simpler versions of the same problem; (2) there's a non-recursive escape case; and (3) once a subproblem is solved, work proceeds to the next step of the original problem.
- In assembly, recursive procedures are implemented as follows: the calling program places the activation record for the first call on the stack and calls the procedure. The procedure uses **BP** to access the data it needs from the stack. Before initiating a recursive call, a procedure places the activation record for that call on the stack and calls itself. When a call completes, **BP is restored**, the **return address is popped into IP**, and the data for the completed call is popped off the stack.
- The code for a procedure may involve more than one recursive call (as with BINOMIAL). Intermediate results may be saved on the stack and retrieved when the original call resumes.

---

## Glossary

| Term | Meaning |
|---|---|
| **activation record** | The values of the parameters, local variables, and return address belonging to one specific procedure call |
| **recursive process** | A process that is defined in terms of itself |

---

## Tying It Back Together — Why This Chapter Matters

Riyad, the single biggest payoff of this chapter is realizing that **recursion in a high-level language isn't magic — it's exactly this**, just hidden behind the compiler:

- Every time C++/Java/Python calls a function, it's silently doing what `CALL` does: pushing a return address.
- Every local variable and parameter you declare inside a recursive function lives in what's effectively an *activation record* — the compiler manages a "stack frame" per call, using its own hidden equivalent of BP (often literally called the **frame pointer** in compiler/OS terminology — the exact same concept as your `BP` here).
- **Stack overflow** errors you've probably hit before in deeply-recursive C++ code (e.g., a bad base case that never triggers) are now completely demystified: it's the literal stack segment (`.STACK 100H` — a *finite* block of memory!) filling up with activation record after activation record until there's no more room to push.

This chapter is a great one to have completed before your OS course (CSE313/314) gets deeper into how the OS manages a process's call stack, and it also directly explains *why* your DSA recursive solutions sometimes need `sys.setrecursionlimit()` in Python or hit `StackOverflowError` in Java — you've now seen, at the byte level, exactly where that limit comes from.

---

## Exercises (from the book, for your own practice)

1. Write a recursive definition of `aⁿ`, where `n` is a nonnegative integer.
2. Ackermann's function is defined for nonnegative integers `m` and `n` as:
   ```
   A(0, n) = n + 1
   A(m, 0) = A(m-1, 1)
   A(m, n) = A(m-1, A(m, n-1))     if m,n ≠ 0
   ```
   Use the definition to show that `A(2,2) = 7`.
3. Trace the stack at several points during `PGM17_3.ASM` (FIND_MAX) — at the moment BP is set in each of the four nested calls, and again at the moment each call completes (after its `RET 2`), noting AX each time.
4. **(Programming)** Write a recursive assembly procedure to sum the elements of a word array, and test it on a four-element array.
5. **(Programming)** The Fibonacci sequence `1, 1, 2, 3, 5, 8, 13, 21, 34, 55, ...` is defined recursively as:
   ```
   F(0) = F(1) = 1
   F(n) = F(n-1) + F(n-2)     if n > 1
   ```
   Write a recursive assembly procedure to compute `F(n)`, and call it in a test program to compute `F(7)`.

*(These are great candidates for your own practice runs in DEBUG, tracing the stack exactly the way we did for FACTORIAL in 17.5 — especially exercise 5, since Fibonacci makes two recursive calls per invocation just like BINOMIAL did, so it's excellent practice for the trickiest part of this chapter.)*
