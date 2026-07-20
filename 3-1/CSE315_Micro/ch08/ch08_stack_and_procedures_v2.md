# Chapter 8 — Stack & Procedures (learn from the comments)

Every file below is standalone. Everything is taught INSIDE the code as comments. Assemble each one separately.

---

## 1. PUSH / POP

```asm
.MODEL SMALL
.STACK 100H              ; reserve 100h (256) bytes for stack. SP starts at 0100h = "empty"
.CODE
MAIN PROC
    MOV AX, 1234H
    MOV BX, 5678H

    PUSH AX             ; step1: SP = SP-2  (0100h -> 00FEh)
                         ; step2: mem[SS:00FE] = 1234H (copy of AX, AX itself unchanged)

    PUSH BX              ; SP = 00FEh -> 00FCh
                          ; mem[SS:00FC] = 5678H
                          ; stack now (top->bottom): 5678h, 1234h  <- LIFO, BX pushed last = on top

    POP CX                ; step1: CX = mem[SS:SP] = 5678h  (grabs the TOP, which was pushed LAST)
                           ; step2: SP = SP+2 (00FCh -> 00FEh)

    POP DX                 ; DX = mem[SS:00FEh] = 1234h
                            ; SP = 00FEh -> 0100h  (empty again, back to start)

    ; RULES (can't be broken, memorize):
    ; - PUSH/POP only take 16-bit reg/mem. "PUSH DL" is ILLEGAL (DL is 8-bit)
    ; - "PUSH 5" is ILLEGAL on 8086 (no pushing raw immediate values)
    ; - stack GROWS DOWNWARD: each push makes SP smaller, not bigger

    MOV AH, 4CH
    INT 21H
MAIN ENDP
END MAIN
```

---

## 2. PUSHF / POPF

```asm
.MODEL SMALL
.STACK 100H
.CODE
MAIN PROC
    MOV AX, 5
    CMP AX, 5          ; AX-5 = 0 -> ZF gets set to 1 right now

    PUSHF              ; push the WHOLE flags register onto stack in one shot
                        ; (this is how you "save" a flag state you'll need later)

    MOV AX, 9
    CMP AX, 3          ; AX-3 != 0 -> ZF is now 0 (flags got trashed/changed)

    POPF                ; pop stack back into FLAGS register -> ZF is 1 again
                         ; (exactly restores whatever flag-state you pushed earlier)

    ; use-case: you're about to run code that will mess with flags,
    ; but you still need the OLD flag result afterward -> PUSHF before, POPF after

    MOV AH, 4CH
    INT 21H
MAIN ENDP
END MAIN
```

---

## 3. Stack-based string reversal (LIFO used as a trick)

```asm
TITLE REVERSE_INPUT
.MODEL SMALL
.STACK 100H
.CODE
MAIN PROC
    MOV AH, 2
    MOV DL, '?'
    INT 21H              ; just prints a '?' prompt

    XOR CX, CX           ; CX = 0. this will COUNT how many chars we push
                          ; (need this because we don't know in advance how many chars typed)

    MOV AH, 1
    INT 21H              ; read 1 char from keyboard into AL

WHILE_:
    CMP AL, 0DH          ; 0Dh = Enter key (carriage return). did user press Enter?
    JE  END_WHILE        ; yes -> stop reading, go pop everything back out

    PUSH AX               ; push the char. NOTE: char is only in AL, but PUSH
                           ; is word-only, so entire AX goes on stack (a bit wasteful, but only option)
    INC CX                 ; count++ (so we know how many times to POP later)
    INT 21H                 ; read next char (AH is still 1 from before)
    JMP WHILE_               ; loop back and check again

END_WHILE:
    MOV AH, 2
    MOV DL, 0DH
    INT 21H               ; print CR
    MOV DL, 0AH
    INT 21H               ; print LF -> together these move cursor to a genuinely new line

    JCXZ EXIT             ; if count==0 (user typed nothing, just hit Enter) -> skip popping
                           ; IMPORTANT: popping an empty stack would be a bug. this guards it.

TOP:
    POP DX                 ; grab the MOST RECENTLY pushed char first (LIFO)
                            ; -> this is literally why the output comes out reversed, for free
    INT 21H                 ; AH is still 2 -> this displays DL
    LOOP TOP                 ; auto: CX--, jump to TOP if CX != 0. runs exactly `count` times

EXIT:
    MOV AH, 4CH
    INT 21H
MAIN ENDP
END MAIN
```
**Type `HELLO` -> get `OLLEH`.** No reversal logic written anywhere — it's a free side-effect of stack order.

---

## 4. Minimal procedure — CALL / RET mechanics

```asm
.MODEL SMALL
.STACK 100H
.CODE
MAIN PROC
    CALL SAY_HI          ; CALL does TWO things automatically:
                          ;   1) pushes the offset of the NEXT instruction (the MOV AH,4CH below)
                          ;      onto the stack -> this is the "return address"
                          ;   2) sets IP = SAY_HI's offset -> control jumps there NOW

    ; <-- execution resumes exactly HERE after SAY_HI's RET runs
    MOV AH, 4CH
    INT 21H
MAIN ENDP

SAY_HI PROC
    MOV AH, 2
    MOV DL, 'X'
    INT 21H

    RET                   ; RET does ONE thing: pops the top of stack straight into IP
                           ; that top-of-stack value IS the return address CALL saved earlier
                           ; -> so execution jumps back to right after the original CALL
SAY_HI ENDP
END MAIN
```
**Trace in DEBUG:** before `CALL`, `SP=0100`. Right after `CALL` executes, `SP=00FE` and `IP` = SAY_HI's offset. Right after `RET`, `SP=0100` again and `IP` = the instruction right after CALL.
`CALL` = push-return-address + jump. `RET` = pop-into-IP. That's the entire trick — no compiler magic.

---

## 5. Procedure with register-convention I/O (shift-and-add multiply)

```asm
.MODEL SMALL
.STACK 100H
.CODE
MAIN PROC
    MOV AX, 7             ; A goes into AX (this is our "input" convention)
    MOV BX, 13             ; B goes into BX
    CALL MULTIPLY           ; jump into procedure; product will come back in DX

    ; DX now holds 91 (7*13) here
    MOV AH, 4CH
    INT 21H
MAIN ENDP

; input:  AX = A, BX = B  (0-FFh range so DX never overflows)
; output: DX = product
; uses:   nothing else
; ^ THIS comment block IS the "function signature" -- assembly has none built-in,
;   so you invent + document the convention yourself.
MULTIPLY PROC
    PUSH AX               ; save caller's AX before we trash it -- caller may still need it
    PUSH BX               ; same for BX. (good habit even if THIS caller doesn't need them back)

    XOR DX, DX            ; product = 0  (XOR reg,reg is the classic "clear to zero" trick)

REPEAT:
    TEST BX, 1            ; checks bit0 of BX WITHOUT changing BX. sets ZF=1 if that bit is 0
    JZ  END_IF             ; bit was 0 (B is even) -> skip the add
    ADD DX, AX              ; bit was 1 (B is odd) -> product += A
END_IF:
    SHL AX, 1               ; A = A*2   (doubling A each pass, like shifting a partial product left)
    SHR BX, 1                ; B = B/2, and the bit we just tested gets discarded

    JNZ REPEAT                ; SHR just set flags -- if BX isn't 0 yet, loop again
                               ; (this IS the "UNTIL B=0" loop-exit, just written as its opposite)

    POP BX                     ; MUST pop in REVERSE order of push (LIFO!)
    POP AX                      ; pushed AX,BX -> must pop BX,AX or you'll SWAP their values

    RET                          ; back to MAIN, right after CALL MULTIPLY
MULTIPLY ENDP
END MAIN
```
Trace this in DEBUG (`RAX`→7, `RBX`→D, then alternate `G17`/`T`) and literally watch DX climb: 0 → 7 → 7 → 35(23h) → 91(5Bh).

---

## 6. Nested calls — a procedure calling another procedure

```asm
.MODEL SMALL
.STACK 100H
.CODE
MAIN PROC
    CALL OUTER             ; push return-addr-1 (back into MAIN), jump to OUTER
    MOV AH, 4CH
    INT 21H
MAIN ENDP

OUTER PROC
    MOV AH, 2
    MOV DL, 'A'
    INT 21H

    CALL INNER              ; push return-addr-2 (back into OUTER, at 'C' line below), jump to INNER
                             ; stack NOW holds TWO return addresses stacked up:
                             ;   top    = return-addr-2 (back to OUTER)
                             ;   bottom = return-addr-1 (back to MAIN)
    MOV DL, 'C'
    INT 21H

    RET                       ; pops return-addr-1 -> back to MAIN
OUTER ENDP

INNER PROC
    MOV DL, 'B'
    INT 21H
    RET                        ; pops return-addr-2 (the TOP one) -> back to OUTER, at 'C' line
INNER ENDP
END MAIN
```
Output: `ABC`. Each `CALL` stacks its own return address; each `RET` pops the most recent one first — this exact LIFO chaining is what makes recursion possible later (a procedure calling itself just keeps stacking more return addresses).

---

## 7. DEBUG commands (comment-style reference)

```
U              ; unassemble -- show raw memory as instructions
R              ; show all registers + flags
RAX            ; -> prompts you to type a new value into AX (same for RBX, etc.)
T              ; trace -- execute ONE instruction, then show updated registers
G 17           ; run continuously until JUST BEFORE offset 17h
G              ; run to completion (no stopping point)
D SS:F0 FF     ; dump raw bytes of stack segment, offset F0 to FF
Q              ; quit DEBUG

; gotcha: memory dumps are little-endian.
; return address 0003h in memory shows as bytes: 03 00 (low byte FIRST)
```

---

## One-paragraph mental model

`SS:SP` always points at the current top of the stack. `PUSH`/`POP` are the raw primitives: copy a word in/out, move `SP` by 2. `CALL` is nothing but `PUSH(return_address)` + `IP = target`. `RET` is nothing but `POP -> IP`. Procedures are 100% built out of the same stack ops you already know — there's no separate "function call" hardware feature, just disciplined use of a LIFO memory region.
