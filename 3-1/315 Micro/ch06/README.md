Full beginner-friendly breakdown of Chapter 6, Riyad — this is one of the most important chapters since it's where your programs finally get "smart" (decisions and loops).

---

# Chapter 6 — Flow Control Instructions

---

## Overview

Up to now, your programs have just run top to bottom, one instruction after another — no decisions, no repetition. That's very limiting. Real programs need to:
- **Make decisions** ("if this, do that")
- **Repeat code** (loops)

This chapter introduces **jump instructions** — the assembly-level building blocks for both. A jump instruction tells the CPU: "don't go to the next instruction in line — go somewhere else instead."

Jumps can be:
- **Unconditional** — always jump, no question asked
- **Conditional** — jump only if a certain flag condition is true

---

## 6.1 An Example of a Jump

The book builds intuition with a real program: display all 256 characters in the IBM character set.

```asm
TITLE PGM6_1: IBM CHARACTER DISPLAY
.MODEL SMALL
.STACK 100H
.CODE
MAIN PROC
    MOV AH, 2       ; display char function
    MOV CX, 256     ; no. of chars to display
    MOV DL, 0       ; DL has ASCII code of null char
PRINT_LOOP:
    INT 21h         ; display a char
    INC DL          ; increment ASCII code
    DEC CX          ; decrement counter
    JNZ PRINT_LOOP  ; keep going if CX not 0
    ; DOS exit
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

### Walking through the logic

- The IBM character set has **256 characters**, codes 0–255. Codes 32–127 are standard ASCII (what you'd normally recognize as letters/symbols). Codes 0–31 and 128–255 are special graphics characters.
- **AH = 2** → prepares to use the "display single character" DOS function
- **CX = 256** → this becomes the **loop counter**
- **DL = 0** → starts at ASCII code 0

### The loop body (lines 9–13)

```asm
PRINT_LOOP:
    INT 21h         ; display the character currently in DL
    INC DL          ; move to next ASCII code
    DEC CX          ; one less iteration remaining
    JNZ PRINT_LOOP  ; if CX isn't 0 yet, go back to PRINT_LOOP
```

This is the heart of looping in assembly: **decrement a counter, then jump back if it's not zero yet.**

### What is JNZ?

**JNZ = Jump if Not Zero.** It looks at the **Zero Flag (ZF)** — which was just set by the `DEC CX` instruction right before it. If the result of that DEC was NOT zero, JNZ jumps back to `PRINT_LOOP`. If CX finally hit 0, JNZ does nothing, and execution falls through to the next line (the DOS exit code).

### What is a Label?

`PRINT_LOOP:` is called a **label** — it's just a name you give to a specific line in your code, so other instructions (like JNZ) can refer to it as a jump destination.

Rules for labels:
- End with a colon `:`
- Usually placed alone on their own line for readability
- When alone on a line, the label refers to the instruction that comes right after it

---

## 6.2 Conditional Jumps

`JNZ` is one example of a **conditional jump instruction**. General syntax:

```asm
Jxxx    destination_label
```

Where `Jxxx` is one of many jump mnemonics (JNZ, JE, JG, etc — you'll see a full table below).

**How it behaves:**
- If the jump's condition is **true** → the next instruction executed is the one at `destination_label`
- If the condition is **false** → execution just continues to the very next line as normal (the jump is skipped)

### Range Restriction — an important quirk

Because of how the machine code for conditional jumps is structured, the destination label **must be close by**:
- No more than **126 bytes before** the jump
- No more than **127 bytes after** the jump

This is a real limitation — if your loop body is too long, the jump target might be "out of range," and the assembler will give an error. (Section 6.3 shows the workaround.)

### How the CPU Actually Implements This

Recall from Chapter 5: the FLAGS register always reflects the result of whatever instruction ran last. A conditional jump simply **checks specific flag bits**:

- If the required flag combination matches the jump's condition → CPU sets **IP** to point at the destination label. Next fetch-execute cycle grabs the instruction there.
- If not → IP is left alone, so the very next instruction in memory sequence executes normally.

In our PRINT_LOOP example: `JNZ` checks **ZF**. If ZF = 0 (meaning DEC CX did NOT produce zero) → jump happens. If ZF = 1 (CX became exactly 0) → no jump, falls through to `MOV AH,4CH`.

### The Three Categories of Conditional Jumps

**Table 6.1 — full reference**

**1. Signed Jumps** — use when comparing numbers as *signed* (can be negative)

| Symbol | Meaning | Condition |
|---|---|---|
| JG / JNLE | jump if greater than | ZF=0 and SF=OF |
| JGE / JNL | jump if greater than or equal | SF=OF |
| JL / JNGE | jump if less than | SF≠OF |
| JLE / JNG | jump if less than or equal | ZF=1 or SF≠OF |

**2. Unsigned Jumps** — use when comparing numbers as *unsigned* (always ≥0)

| Symbol | Meaning | Condition |
|---|---|---|
| JA / JNBE | jump if above | CF=0 and ZF=0 |
| JAE / JNB | jump if above or equal | CF=0 |
| JB / JNAE | jump if below | CF=1 |
| JBE / JNA | jump if below or equal | CF=1 or ZF=1 |

**3. Single-Flag Jumps** — check just one specific flag directly

| Symbol | Meaning | Condition |
|---|---|---|
| JE / JZ | jump if equal / jump if zero | ZF=1 |
| JNE / JNZ | jump if not equal / not zero | ZF=0 |
| JC | jump if carry | CF=1 |
| JNC | jump if no carry | CF=0 |
| JO | jump if overflow | OF=1 |
| JNO | jump if no overflow | OF=0 |
| JS | jump if sign negative | SF=1 |
| JNS | jump if nonnegative sign | SF=0 |
| JP / JPE | jump if parity even | PF=1 |
| JNP / JPO | jump if parity odd | PF=0 |

**Note:** Many jumps have two names (like JG and JNLE) — they produce **identical machine code**. You just pick whichever name makes your code more readable in context ("jump if greater" reads better in some places, "jump if not less-or-equal" in others).

**Important:** Jump instructions themselves **never change the flags**. They only *read* them.

---

## The CMP Instruction

Where do the flags that a jump checks usually come from? Very often from **CMP (compare)**.

```asm
CMP    destination, source
```

CMP computes `destination − source` internally (exactly like SUB), and sets the flags based on that result — **but it does NOT store the result anywhere.** The destination is left completely unchanged. Think of CMP as "SUB, but throw away the answer, just keep the flags."

**Restrictions** (same as SUB): both operands can't both be memory locations; destination can't be a constant.

### Worked example

```asm
CMP AX, BX
JG  BELOW
```

Say AX = 7FFFh, BX = 0001h. CMP computes 7FFFh − 0001h = 7FFEh internally. Checking Table 6.1, JG requires `ZF=0 and SF=OF`. Since 7FFEh is a normal positive nonzero result with no overflow, all three conditions line up → JG's condition is satisfied → **jump to BELOW happens**.

You don't have to manually trace flags every time — just reason directly: "is AX greater than BX?" and pick the matching jump.

You can also put a jump right after other arithmetic instructions, not just CMP:
```asm
DEC AX
JL  THERE    ; if AX (signed) < 0, jump to THERE
```

---

## Signed Versus Unsigned Jumps — a critical trap

Each signed jump has an "unsigned twin" (e.g. JG ↔ JA). **They check different flags!**
- Signed jumps → ZF, SF, OF
- Unsigned jumps → ZF, CF

**Using the wrong one silently gives wrong answers.** Example:

```asm
; AX = 7FFFh, BX = 8000h
CMP AX, BX
JA  BELOW
```

If you're thinking of these as *signed* numbers: 7FFFh = +32767, 8000h = −32768. So 7FFFh **is** greater. But `JA` is the **unsigned** jump! Unsigned, 7FFFh = 32767 and 8000h = 32768 — so 7FFFh is actually **smaller** unsigned. JA's condition fails → **no jump happens**. This is the opposite of what you'd expect if you were sloppily thinking "signed."

**Rule of thumb:** decide up front whether your numbers should be treated as signed or unsigned, and stick to the matching jump family throughout.

### Working with characters

Standard ASCII character codes (0–127) always have their sign bit = 0, so signed and unsigned jumps behave identically for them — safe either way. But for **extended ASCII** (codes 80h–FFh, which have the high bit set), you **must use unsigned jumps**, since these codes would look negative under a signed interpretation.

---

### Example 6.1 — Put the bigger of two signed numbers in CX

```asm
MOV CX, AX     ; put AX in CX
CMP BX, CX     ; is BX bigger?
JLE NEXT       ; no, go on
MOV CX, BX     ; yes, put BX in CX
NEXT:
```

Logic: assume AX is the answer by default (copy it to CX). Then check if BX is actually bigger — if BX ≤ CX, skip the replacement; otherwise replace CX with BX.

---

## 6.3 The JMP Instruction

**JMP** = unconditional jump. Always jumps, no flag-checking involved.

```asm
JMP    destination
```

### Why does this matter? — Getting around the range restriction

Remember: conditional jumps (JNZ, JG, etc.) can only reach labels within ~126–127 bytes. But **JMP has no such limit** (within the same segment). So if your loop body is too big for a direct conditional jump, you use a **double-jump trick**:

Original (broken if loop body is too long):
```asm
TOP:
    ; body of the loop (too many instructions)
    DEC CX
    JNZ TOP      ; ERROR: TOP might be out of range
    MOV AX, BX
```

Fixed with JMP:
```asm
TOP:
    ; body of the loop
    DEC CX
    JNZ BOTTOM   ; BOTTOM is very close by — always in range
    JMP EXIT     ; if we reach here, CX hit 0, so skip the loop-back
BOTTOM:
    JMP TOP      ; unconditional jump, no range limit
EXIT:
    MOV AX, BX
```

The trick: `JNZ` only needs to jump a **short distance** to `BOTTOM`, which then does an unrestricted `JMP TOP` to go back to the start. This indirection sidesteps the range limit entirely.

---

## 6.4 High-Level Language Structures

Raw jumps are powerful but messy to reason about directly. This section shows how to **systematically translate familiar high-level constructs** (IF, CASE, loops) into clean, predictable assembly patterns. The book always writes **pseudocode first**, then translates it mechanically.

---

## 6.4.1 Branching Structures

### 1. IF-THEN

Pseudocode:
```
IF condition is true
 THEN
  execute true-branch statements
END_IF
```


**Pattern:** if condition is false, jump *around* the THEN block, straight to `END_IF`.

**Example 6.2** — Replace AX with its absolute value

Pseudocode:
```
IF AX < 0
 THEN
  replace AX by -AX
END_IF
```

Assembly:
```asm
;if AX < 0
    CMP AX, 0
    JNL END_IF     ; not less than 0 → skip
;then
    NEG AX          ; AX < 0 → negate it
END_IF:
```

Notice the pattern: we test the **opposite** condition and jump past the block if that opposite is true. "AX < 0" → test with `JNL` (jump if **not** less) to skip when the original condition is false.

---

### 2. IF-THEN-ELSE

Pseudocode:
```
IF condition is true
 THEN
  execute true-branch statements
 ELSE
  execute false-branch statements
END_IF
```

**Example 6.3** — Display whichever of AL, BL (extended ASCII chars) comes first alphabetically

Pseudocode:
```
IF AL <= BL
 THEN
  display the character in AL
 ELSE
  display the character in BL
END_IF
```

Assembly:
```asm
    MOV AH, 2          ; prepare to display
;if AL <= BL
    CMP AL, BL
    JNBE ELSE_          ; not (AL<=BL) → go to else
;then
    MOV DL, AL
    JMP DISPLAY         ; skip the else branch!
ELSE_:
    MOV DL, BL
DISPLAY:
    INT 21h              ; display it
END_IF:
```
**Key difference from IF-THEN:** after the THEN block runs, you need an explicit `JMP` to **skip over** the ELSE block — otherwise execution would fall straight into it! High-level languages do this skip automatically; in assembly, you have to code it yourself.

**Naming note:** the label can't literally be called `ELSE` because that's a reserved word in MASM — hence `ELSE_` with a trailing underscore.

We used the **unsigned** jump `JNBE` here because we're comparing extended ASCII characters.


MyVersion:
```asm
    MOV AH, 2          ; prepare to display
;if AL <= BL (signed)
    CMP AL, BL
    JLE THEN_
    JG  ELSE_
THEN_:                 ;then
    MOV DL, AL
    JMP END_IF         ; skip the else branch
ELSE_:                 ;else
    MOV DL, BL
END_IF:
    INT 21h            ; display it
```






#### But why don't people usually write it this way?



Because the second jump is **unnecessary**.



If `JLE` is **not** taken, then the condition **must** be `JG`.



So this:



```asm

CMP AL, BL

JLE THEN_

JG  ELSE_

```



is equivalent to simply:



```asm

CMP AL, BL

JG ELSE_



THEN_:

```



The second version uses one less instruction, so it's more efficient.



---



#### For learning, though...



I actually like your version because it makes the logic crystal clear:



```text

IF AL <= BL

    go to THEN

IF AL > BL

    go to ELSE

```



It's a perfectly reasonable way to think about the control flow while you're learning. Later, you'll naturally start removing the redundant jumps as you get comfortable with assembly.

---

### 3. CASE

A multi-way branch — like a chain of IF-ELSE-IF, testing one expression against several possible value sets.

Pseudocode:
```
CASE expression
   values_1: statements_1
   values_2: statements_2
   ...
   values_n: statements_n
END_CASE
```

**Example 6.4** — classify AX as negative/zero/positive, store result in BX

```asm
;case AX
    CMP AX, 0
    JL  NEGATIVE
    JE  ZERO
    JG  POSITIVE
NEGATIVE:
    MOV BX, -1
    JMP END_CASE
ZERO:
    MOV BX, 0
    JMP END_CASE
POSITIVE:
    MOV BX, 1
END_CASE:
```

Notice: only **one CMP** is needed at the top — since jump instructions never alter flags, all three subsequent jumps (JL, JE, JG) can safely reuse the same comparison result.

**Example 6.5** — if AL is 1 or 3 display 'o'; if AL is 2 or 4 display 'e'

```asm
;case AL
; 1,3:
    CMP AL, 1
    JE  ODD
    CMP AL, 3
    JE  ODD
; 2,4:
    CMP AL, 2
    JE  EVEN
    CMP AL, 4
    JE  EVEN
    JMP END_CASE      ; not 1..4, skip everything
ODD:
    MOV DL, 'o'
    JMP DISPLAY
EVEN:
    MOV DL, 'e'
DISPLAY:
    MOV AH, 2
    INT 21H
END_CASE:
```

Here multiple values map to the same branch — you just chain multiple CMP+JE pairs pointing at the same label.

---

### 4. Branches with Compound Conditions

Sometimes an IF condition isn't a single comparison but a combination:

```
condition_1 AND condition_2
condition_1 OR  condition_2
```

### AND Conditions

**Both** conditions must be true for the whole thing to be true. If **either** is false, the entire thing is false — so you can exit early the moment one fails.

**Example 6.6** — read a character, display it only if it's an uppercase letter

Pseudocode:
```
IF ('A' <= character) AND (character <= 'Z')
 THEN
  display character
END_IF
```

Assembly:
```asm
;read a character
    MOV AH, 1
    INT 21H
;if ('A' <= char) and (char <= 'Z')
    CMP AL, 'A'
    JNGE END_IF     ; char < 'A' → fails first condition, bail out
    CMP AL, 'Z'
    JNLE END_IF     ; char > 'Z' → fails second condition, bail out
;then display char
    MOV DL, AL
    MOV AH, 2
    INT 21H
END_IF:
```

**Pattern for AND:** test each sub-condition one at a time; the moment *any one* fails, jump straight to the end (skip the THEN block).


MyVersion:
```asm
;read a character
    MOV AH, 1
    INT 21H

;if ('A' <= char) and (char <= 'Z')

;-- condition 1: char >= 'A'
    CMP AL, 'A'
    JGE COND2       ; char >= 'A' → condition 1 true, check condition 2
    JL  END_IF      ; char < 'A'  → condition 1 false, whole AND fails, bail

COND2:
;-- condition 2: char <= 'Z'
    CMP AL, 'Z'
    JLE THEN_       ; char <= 'Z' → condition 2 true, both conditions hold
    JG  END_IF      ; char > 'Z'  → condition 2 false, bail

THEN_:
;then display char
    MOV DL, AL
    MOV AH, 2
    INT 21H

END_IF:
```


### OR Conditions

**At least one** condition needs to be true. Only false if **both** fail.

**Example 6.7** — read a char; if it's 'y' or 'Y', display it; otherwise exit the program

Pseudocode:
```
IF (character = 'y') OR (character = 'Y')
 THEN
  display it
 ELSE
  terminate the program
END_IF
```

Assembly:
```asm
;read a character
    MOV AH, 1
    INT 21H
;if (char = 'y') or (char = 'Y')
    CMP AL, 'y'
    JE  THEN         ; matches first option → go straight to THEN
    CMP AL, 'Y'
    JE  THEN         ; matches second option → go straight to THEN
    JMP ELSE_         ; neither matched → go to ELSE
THEN:
    MOV AH, 2
    MOV DL, AL
    INT 21H
    JMP END_IF
ELSE_:
    MOV AH, 4CH
    INT 21H          ; DOS exit
END_IF:
```

**Pattern for OR:** test each sub-condition; the moment *any one* succeeds, jump directly into the THEN block. Only fall through to ELSE if every single check fails.

---
---
# LOOP
---
## 6.4.2 Looping Structures

A **loop** repeats a block of instructions. Three flavors, differing in *when* and *how* the repeat count/condition is checked.

---

### FOR LOOP — repeat a known number of times

Pseudocode:
```
FOR loop_count times DO
  statements
END_FOR
```

### The LOOP instruction

```asm
LOOP    destination_label
```

This is a special all-in-one instruction: it automatically **decrements CX**, and if CX is still not 0, jumps to `destination_label`. If CX hits 0, execution just falls through to the next line. (Same 126-byte range restriction as other conditional jumps applies here too.)

Pattern:
```asm
    ; initialize CX to loop_count
TOP:
    ; body of the loop
    LOOP TOP
```

**Example 6.8** — display a row of 80 stars

```asm
    MOV CX, 80       ; number of stars
    MOV AH, 2         ; display character function
    MOV DL, '*'       ; character to display
TOP:
    INT 21h           ; display a star
    LOOP TOP          ; repeat 80 times
```

### ⚠️ Important gotcha: LOOP with CX = 0

If CX happens to be **0** when you enter the loop, `LOOP` will decrement it to **FFFFh** (since it wraps around, being unsigned) and treat that as "not zero" — so the loop runs **65,535 more times** instead of zero times! This is a classic bug.

**Fix — use JCXZ (Jump if CX is Zero) before entering the loop:**

```asm
JCXZ    destination_label
```

Guarded pattern:
```asm
    JCXZ SKIP      ; if CX is already 0, skip the whole loop
TOP:
    ; body of the loop
    LOOP TOP
SKIP:
```

---

### WHILE LOOP — condition checked at the TOP

Pseudocode:
```
WHILE condition DO
  statements
END_WHILE
```

Key property: the condition is checked **before** each iteration — including the very first one. If the condition is false right from the start, the loop body **never executes at all**.

**Example 6.9** — count characters in an input line (until carriage return)

Pseudocode:
```
initialize count to 0
read a character
WHILE character <> carriage_return DO
 count = count + 1
 read a character
END_WHILE
```

Assembly:
```asm
    MOV DX, 0         ; DX counts characters
    MOV AH, 1
    INT 21H           ; read first character into AL
WHILE_:
    CMP AL, 0DH       ; is it carriage return?
    JE  END_WHILE     ; yes → exit loop
    INC DX            ; not CR → count it
    INT 21H           ; read next character
    JMP WHILE_        ; loop back to check condition again
END_WHILE:
```

**Important detail:** because the condition is checked at the *top*, you must **prime** the loop — read the very first character *before* entering the loop, so there's something to test right away. Then read the *next* character at the *bottom* of the loop body, right before looping back.

(`WHILE_:` has a trailing underscore because `WHILE` is a reserved word.)

---

### REPEAT LOOP — condition checked at the BOTTOM

Pseudocode:
```
REPEAT
 statements
UNTIL condition
```

Key property: statements execute **first**, and the condition is only checked **after**. This guarantees the loop body runs **at least once**, no matter what.

**Example 6.10** — read characters until a blank is read

Pseudocode:
```
REPEAT
 read a character
UNTIL character is a blank
```

Assembly:
```asm
    MOV AH, 1
REPEAT:
    INT 21H            ; read a character into AL
;until
    CMP AL, ' '        ; is it a blank?
    JNE REPEAT         ; not blank → loop back
```

Notice how much shorter this is compared to WHILE — only **one** jump needed (at the bottom), versus WHILE's two jumps (conditional at top, unconditional JMP at bottom).

### WHILE vs REPEAT — when to use which

| | WHILE | REPEAT |
|---|---|---|
| Condition checked | Top (before) | Bottom (after) |
| Can skip body entirely? | Yes, if condition false initially | No — always runs at least once |
| Code length | Longer (2 jumps) | Shorter (1 jump) |

Choice is often just personal preference/context — use WHILE when the body might not need to run at all, REPEAT when you know it must run at least once (like reading input).

---

## 6.5 Programming with High-Level Structures — A Full Worked Example

This section demonstrates **top-down program design**: break a big problem into smaller subproblems, solve each subproblem, then assemble the pieces.

### The problem

Prompt for a line of text. Find the first and last capital letters typed (alphabetically). If no capitals were typed, say so.

Sample run:
```
Type a line of text:
THE QUICK BROWN FOX JUMPED.
First capital = B Last capital = X
```

### First refinement (the big picture, 3 steps)

1. Display the opening message
2. Read and process a line of text
3. Display the results

---

### Step 1 — Display the opening message

Trivial — direct translation to code:
```asm
MOV AH, 9
LEA DX, PROMPT
INT 21H
```

Data:
```asm
PROMPT DB 'Type a line of text:', 0DH, 0AH, '$'
```

---

### Step 2 — Read and process the line

This is the meaty part. Pseudocode:
```
Read a character
WHILE character is not a carriage return DO
  IF character is a capital letter (*)
   THEN
    IF character precedes first capital
     THEN
      first capital = character
    END_IF
    IF character follows last capital
     THEN
      last capital = character
    END_IF
  END_IF
  Read a character
END_WHILE
```

Line `(*)` is an **AND condition**: `('A' <= character) AND (character <= 'Z')`.

Notice this is a **WHILE loop** (checked at top) containing a **nested IF** (checking if it's a capital), which itself contains **two more nested IFs** (checking if it's a new first/last capital). This kind of nesting is completely normal — just build it piece by piece.

Assembly:
```asm
    MOV AH, 1
    INT 21H              ; char in AL
WHILE_:
;while character is not a carriage return do
    CMP AL, 0DH
    JE  END_WHILE        ; yes, exit
;if character is a capital letter
    CMP AL, 'A'
    JNGE END_IF          ; not a capital letter
    CMP AL, 'Z'
    JNLE END_IF          ; not a capital letter
;then
; if character precedes first capital
    CMP AL, FIRST
    JNL CHECK_LAST       ; no, >=
;then first capital = character
    MOV FIRST, AL
; end_if
CHECK_LAST:
; if character follows last capital
    CMP AL, LAST
    JNG END_IF           ; no, <=
; then last capital = character
    MOV LAST, AL
; end_if
END_IF:
;read a character
    INT 21H
    JMP WHILE_           ; repeat loop
END_WHILE:
```

### Clever initialization trick

`FIRST` and `LAST` need starting values *before* the loop runs (since comparisons happen against them). The book picks:

```asm
FIRST DB ']'
LAST  DB '@'
```

Why these specific characters? In ASCII order: `@` comes right *before* `A`, and `]` comes right *after* `Z`. So:
- **Any** actual capital letter will be "less than" `]` → the very first capital found will immediately replace FIRST
- **Any** actual capital letter will be "greater than" `@` → the very first capital found will immediately replace LAST

This guarantees correct behavior on the very first capital letter encountered, without needing a special "is this the first one?" check.

---

### Step 3 — Display the results

Pseudocode:
```
IF no capitals were typed
 THEN
  display "No capitals"
 ELSE
  display first capital and last capital
END_IF
```

Two possible messages are pre-declared:
```asm
NOCAP_MSG DB 'No capitals $'
CAP_MSG   DB 'First capital = '
FIRST     DB ']'
          DB ' Last capital = '
LAST      DB '@ $'
```

Clever trick again: `CAP_MSG`, `FIRST`, and `LAST` are declared **consecutively in memory**. So displaying `CAP_MSG` with function 9 automatically flows right into displaying the actual `FIRST` character and `LAST` character too — because there's no `$` terminator until the very end.

**How do we know if any capitals were found?** Just check: does `FIRST` still hold its initial value `]`? If yes → no capitals were ever typed.

```asm
    MOV AH, 9
;if no capitals were typed
    CMP FIRST, ']'
    JNE CAPS            ; not still ']' → capitals were found
;then
    LEA DX, NOCAP_MSG
    JMP DISPLAY
CAPS:
    LEA DX, CAP_MSG
DISPLAY:
    INT 21H
;end_if
```

### The complete program (PGM6_2.ASM)

```asm
TITLE PGM6_2: FIRST AND LAST CAPITALS
.MODEL SMALL
.STACK 100H
.DATA
PROMPT DB 'Type a line of text', 0DH, 0AH, '$'
NOCAP_MSG DB 0DH, 0AH, 'No capitals $'
CAP_MSG   DB 0DH, 0AH, 'First capital = '
FIRST DB ']'
      DB ' Last capital = '
LAST  DB '@ $'

.CODE
MAIN PROC
; initialize DS
    MOV AX, @DATA
    MOV DS, AX
; display opening message
    MOV AH, 9
    LEA DX, PROMPT
    INT 21H
; read and process a line of text
    MOV AH, 1
    INT 21H
WHILE_:
    CMP AL, 0DH
    JE  END_WHILE
    CMP AL, 'A'
    JNGE END_IF
    CMP AL, 'Z'
    JNLE END_IF
    CMP AL, FIRST
    JNL CHECK_LAST
    MOV FIRST, AL
CHECK_LAST:
    CMP AL, LAST
    JNG END_IF
    MOV LAST, AL
END_IF:
    INT 21H
    JMP WHILE_
END_WHILE:
; display results
    MOV AH, 9
    CMP FIRST, ']'
    JNE CAPS
    LEA DX, NOCAP_MSG
    JMP DISPLAY
CAPS:
    LEA DX, CAP_MSG
DISPLAY:
    INT 21H
; dos exit
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

**The takeaway method:** break the problem into refinements, translate each piece to pseudocode first, then mechanically convert pseudocode patterns (IF/CASE/WHILE/REPEAT) using the fixed templates from section 6.4. This is exactly how you should approach any assembly problem you're given.

---

## Summary (from the book)

- Jump instructions: **unconditional** (JMP) and **conditional** (JG, JNZ, etc.)
- Conditional jumps subdivide into: **signed**, **unsigned**, and **single-flag** jumps
- Conditional jumps read the status flags — **CMP** is the most common way to set those flags right before a jump
- Conditional jump range limit: 126 bytes before / 127 bytes after. **JMP has no such limit** and can work around it
- **IF-THEN**: skip the block if condition false
- **IF-THEN-ELSE**: need an extra JMP after the true-branch to skip the false-branch
- **CASE**: branching on an expression's value, using chained CMP+jump pairs
- **FOR loop** → `LOOP` instruction, CX as counter (watch out for CX=0 bug, use JCXZ)
- **WHILE loop** → condition checked at top; may run zero times
- **REPEAT loop** → condition checked at bottom; always runs at least once

---

That's the complete Chapter 6, Riyad. This is genuinely the chapter that turns you from "someone who can write instructions" into "someone who can actually program" in assembly — decisions and loops are the backbone of every real algorithm. Ready for the next chapter?