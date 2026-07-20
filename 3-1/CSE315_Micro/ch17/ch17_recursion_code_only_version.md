# Chapter 17 — Recursion (learn from the comments)

Builds directly on Ch8 (stack, CALL/RET). Same rule: every file is standalone, teaching lives in the comments.

---

## 1. Passing parameters ON THE STACK (not registers) — the prerequisite skill

```asm
TITLE ADD_WORDS
.MODEL SMALL
.STACK 100H
.DATA
WORD1   DW   2
WORD2   DW   5
.CODE
MAIN PROC
    MOV AX, @DATA
    MOV DS, AX

    PUSH WORD1          ; push param1 (2)  -> ends up DEEPER in stack
    PUSH WORD2           ; push param2 (5) -> ends up on TOP (pushed last)
    CALL ADD_WORDS         ; CALL then pushes the return address too

    ; at this exact moment stack (top->bottom) = [return_addr, 5, 2]

    MOV AH, 4CH
    INT 21H
MAIN ENDP

; input:  stack has [ret_addr(top), WORD2, WORD1]
; output: AX = sum
ADD_WORDS PROC NEAR
    PUSH BP              ; save CALLER's BP (defensive, same habit as Ch8 PUSH AX/BX)
    MOV  BP, SP           ; ANCHOR bp at current top. SP will move later (if we push/pop
                           ; more inside here) but BP stays fixed -> stable reference point

    ; WHY BP AND NOT SP DIRECTLY?
    ; -> 8086 hardware rule: SP CANNOT be used in [SP+n] addressing. Illegal.
    ; -> BP CAN be used in [BP+n] addressing, AND the CPU auto-assumes segment=SS
    ;    (exactly the stack segment we need) whenever BP appears in an address. Free bonus.

    ; stack layout right now, counting UP from BP in 2-byte (word) steps:
    ;   [BP+0] = saved BP value        (just pushed)
    ;   [BP+2] = return address         (pushed by CALL)
    ;   [BP+4] = WORD2 (=5)              (pushed LAST by caller -> closer to top -> smaller offset)
    ;   [BP+6] = WORD1 (=2)               (pushed FIRST by caller -> further from top)

    MOV AX, [BP+6]        ; AX = WORD1 = 2
    ADD AX, [BP+4]         ; AX = AX + WORD2 = 2+5 = 7

    POP BP                  ; undo our PUSH BP -> caller's BP is restored
    RET 4                    ; pop return_addr into IP, THEN add 4 to SP
                              ; -> this "throws away" the 2 pushed param words (4 bytes)
                              ;    without needing 2 separate POPs. caller doesn't need
                              ;    to clean up its own pushes -- procedure does it via RET n
ADD_WORDS ENDP
END MAIN
```
**Trace with DEBUG:** after both PUSH+CALL, `SP` has dropped 6 bytes total (2+2 params, +2 return addr). After `RET 4`, `SP` is back to `0100h` exactly — fully balanced.

---

## 2. The core recursion idea, translated straight to assembly comments (mental model only, no new asm here)

```
; A recursive procedure = a procedure whose body contains a CALL to ITSELF.
;
; 3 ingredients every recursive procedure MUST have:
;   1) breaks into a SMALLER version of the same problem   (e.g. N-1 instead of N)
;   2) an ESCAPE CASE that needs no further recursion       (e.g. N=1)
;   3) after the recursive call returns, WORK RESUMES        (e.g. multiply by N)
;
; Missing #2 = infinite recursion = stack keeps growing (PUSH BP over and over)
;              until .STACK segment runs out of room = "stack overflow", for real,
;              at the byte level -- not just an abstract error message.
```

---

## 3. FACTORIAL — one recursive call, fully commented

```asm
TITLE FACTORIAL
.MODEL SMALL
.STACK 100H
.CODE
MAIN PROC
    MOV AX, 3          ; want to compute 3!
    PUSH AX             ; N=3 goes on stack as the parameter
    CALL FACTORIAL       ; AX will hold the answer when this returns

    MOV AH, 4CH
    INT 21H
MAIN ENDP

; input:  stack = [ret_addr(top), N]
; output: AX = N!
FACTORIAL PROC NEAR
    PUSH BP
    MOV  BP, SP          ; [BP+4] = N (only ONE param this time, so BP+4 not BP+6)

    CMP  WORD PTR [BP+4], 1   ; is N == 1?  (WORD PTR needed: tells assembler "compare
                                ; a 16-bit value here", since bare "1" is ambiguous size)
    JG   ELSE_                 ; N > 1 -> go recurse. (N==1 falls through to escape case)

    ; ESCAPE CASE (N=1)
    MOV  AX, 1
    JMP  RETURN                ; skip the recursive branch entirely

ELSE_:
    ; RECURSIVE CASE (N>1) -- implements RESULT = N * FACTORIAL(N-1)
    MOV  CX, [BP+4]       ; CX = our own N
    DEC  CX                ; CX = N-1
    PUSH CX                  ; push N-1 as the parameter for the INNER call
    CALL FACTORIAL              ; *** calls itself ***
                                  ; when this returns: AX = (N-1)!
                                  ; IMPORTANT: our own BP is UNTOUCHED by all this --
                                  ; the inner call saved+restored ITS OWN copy of BP,
                                  ; so [BP+4] still correctly means "OUR N" right here

    MUL  WORD PTR [BP+4]           ; AX = AX * N = (N-1)! * N = N!   <- work resumes here

RETURN:
    POP  BP
    RET  2                          ; discard the 1 pushed param (N), 2 bytes
FACTORIAL ENDP
END MAIN
```

### The exact stack picture while tracing FACTORIAL(3) — as comments

```
; call 1 (N=3) pushes BP, stack: [BP0][ret->MAIN][3]              <- BP anchors here
;   N=3 != 1 -> push (3-1=2), CALL again
;
; call 2 (N=2) pushes BP, stack adds: [BP1][ret->call1's MUL][2]  <- BP re-anchors here
;   N=2 != 1 -> push (2-1=1), CALL again
;
; call 3 (N=1) pushes BP, stack adds: [BP2][ret->call2's MUL][1]  <- BP re-anchors here
;   N=1 == 1 -> ESCAPE: AX=1, jump to RETURN
;   POP BP (back to call2's BP), RET 2 (pop ret addr -> back into call2, discard the "1")
;
; call 2 resumes at MUL WORD PTR[BP+4]. [BP+4] here = call2's OWN N = 2
;   AX = 1(from call3) * 2 = 2
;   POP BP (back to call1's BP), RET 2 (back into call1, discard the "2")
;
; call 1 resumes at MUL WORD PTR[BP+4]. [BP+4] here = call1's OWN N = 3
;   AX = 2(from call2) * 3 = 6
;   POP BP (back to MAIN's BP), RET 2 -> back to MAIN
;
; FINAL: AX = 6 = 3!  correct.
; multiplication order was 1*2=2, then 2*3=6  == literally 1*2*3, unwound innermost-first
```

---

## 4. FIND_MAX — same recursion pattern, applied to an array

```asm
TITLE FIND_MAX
.MODEL SMALL
.STACK 100H
.DATA
A   DW   10, 50, 20, 4        ; array, 1-indexed conceptually: A[1]=10, A[2]=50, A[3]=20, A[4]=4
.CODE
MAIN PROC
    MOV AX, @DATA
    MOV DS, AX
    MOV AX, 4              ; N = number of elements
    PUSH AX
    CALL FIND_MAX            ; AX <- largest element
    MOV AH, 4CH
    INT 21H
MAIN ENDP

; input:  stack = [ret_addr(top), N]
; output: AX = max of A[1..N]
FIND_MAX PROC NEAR
    PUSH BP
    MOV  BP, SP

    CMP  WORD PTR [BP+4], 1     ; N==1?
    JG   ELSE_

    ; ESCAPE CASE: only one element -> it IS the max
    MOV  AX, A                    ; "A" alone means A[1] (first word of the array)
    JMP  END_IF1

ELSE_:
    MOV  CX, [BP+4]        ; our N
    DEC  CX                  ; N-1
    PUSH CX                    ; param for inner call
    CALL FIND_MAX                ; recursive call -> AX = max of A[1..N-1] when it returns

    ; now compare that max against A[N] (OUR N, still valid via our own BP)
    MOV  BX, [BP+4]          ; BX = N
    SHL  BX, 1                 ; BX = 2*N          (word array -> each elt is 2 bytes)
    SUB  BX, 2                   ; BX = 2*(N-1)       -> exactly the byte-offset of A[N]
                                   ; (1-indexed array formula: offset = 2*(index-1))
    CMP  A[BX], AX                  ; is A[N] > current MAX (in AX)?
    JLE  END_IF1                      ; no -> AX already correct, nothing to do
                                        ; (note: no code needed for the "else: MAX=MAX" branch --
                                        ;  AX is already right, so there's simply nothing to write)
    MOV  AX, A[BX]                       ; yes -> update MAX = A[N]

END_IF1:
    POP  BP
    RET  2
FIND_MAX ENDP
END MAIN
```
Trace it exactly like FACTORIAL — 4 nested calls stack up, escape fires at N=1 (AX=A[1]=10), then unwinding compares A[2]=50 (new max), A[3]=20 (no change), A[4]=4 (no change) → final AX=50.

---

## 5. BINOMIAL — the twist: TWO recursive calls in one invocation

```asm
TITLE BINOMIAL_COEFFICIENTS
.MODEL SMALL
.STACK 100H
.CODE
MAIN PROC
    MOV AX, 2          ; K=2
    PUSH AX
    MOV AX, 3           ; N=3
    PUSH AX
    CALL BINOMIAL         ; computes C(3,2), answer in AX
    MOV AH, 4CH
    INT 21H
MAIN ENDP

; NOTE the push order: K pushed FIRST, then N pushed LAST (closer to top)
; so inside: [BP+4]=N, [BP+6]=K   (mirror this order in EVERY recursive call below,
; or the offsets inside the nested call will read garbage)
BINOMIAL PROC NEAR
    PUSH BP
    MOV  BP, SP

    MOV  AX, [BP+6]        ; AX = K
    CMP  AX, [BP+4]          ; K == N?
    JE   THEN                  ; yes -> escape case #1 (edge of Pascal's triangle)
    CMP  AX, 0                   ; K == 0?
    JNE  ELSE_                     ; no (and K!=N too) -> must recurse
                                     ; (falls through to THEN if K WAS 0 -> escape case #2)
THEN:
    MOV  AX, 1               ; RESULT = 1  (both edge cases land here)
    JMP  RETURN

ELSE_:
    ; ---- recursive call #1: compute C(N-1, K) ----
    PUSH [BP+6]              ; push K unchanged     (SAME push order as MAIN: K then N)
    MOV  CX, [BP+4]            ; get our N
    DEC  CX                      ; N-1
    PUSH CX                        ; push N-1
    CALL BINOMIAL                    ; -> AX = RESULT1 = C(N-1,K)
    PUSH AX                            ; *** MUST stash RESULT1 on stack ***
                                         ; because the NEXT recursive call will overwrite AX
                                         ; with its own answer -- exact same defensive-save
                                         ; idea as Ch8's PUSH AX before scratch work

    ; ---- recursive call #2: compute C(N-1, K-1) ----
    MOV  CX, [BP+6]           ; get our K
    DEC  CX                     ; K-1
    PUSH CX                       ; push K-1
    MOV  CX, [BP+4]                 ; get our N
    DEC  CX                           ; N-1
    PUSH CX                             ; push N-1
    CALL BINOMIAL                         ; -> AX = RESULT2 = C(N-1,K-1)

    ; ---- combine ----
    POP  BX                    ; BX = RESULT1 (the value we stashed earlier)
    ADD  AX, BX                  ; AX = RESULT2 + RESULT1 = final answer

RETURN:
    POP  BP
    RET  4                      ; discard BOTH pushed params (K and N = 4 bytes)
BINOMIAL ENDP
END MAIN
```

### Hand-trace C(3,2) as comments

```
; C(3,2): K=2,N=3 -> neither K=N nor K=0 -> recurse
;
;   call#1 -> C(2,2): K=2,N=2 -> K=N -> ESCAPE -> returns 1        (this is RESULT1)
;   call#2 -> C(2,1): K=1,N=2 -> neither -> recurse:
;        call#2a -> C(1,1): K=N -> ESCAPE -> returns 1             (RESULT1 of call#2)
;        call#2b -> C(1,0): K=0 -> ESCAPE -> returns 1             (RESULT2 of call#2)
;        call#2 combines: 1+1 = 2                                   (this is RESULT2 of outer)
;
;   outer combines: RESULT1(1) + RESULT2(2) = 3
;
; FINAL: C(3,2) = 3   correct (matches Pascal's triangle row 1 3 3 1)
```

---

## Why this all matters (one comment block)

```
; What C++/Java hides, you just wrote by hand:
;   - every function call = CALL = push return address
;   - every local var/param = lives in an "activation record" =
;       [saved BP][return addr][params] -- one fresh copy PER call, stacked on top
;       of the previous call's copy, because .STACK is finite LIFO memory
;   - BP = compiler's hidden "frame pointer", made visible
;   - StackOverflowError / recursion-limit crashes = literally this: .STACK 100H
;       filling up with activation record after activation record because some
;       escape case (ingredient #2) never fired
```
