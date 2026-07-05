# 8086 Assembly — Full-Code Practice Cheatsheet
*Topics ordered the way they should be learned: Foundations (syntax/data/DOS I/O) → FLAGS → Jumps → IF/CASE → Loops → Full Program Design → Logic Ops → Shift/Rotate → MUL/DIV → Arrays & Addressing Modes.*

Copy each program by hand. Every block below is a **complete, standalone program** — practice writing the whole skeleton (`TITLE`, `.MODEL`, `.STACK`, `.DATA`, `.CODE`, `MAIN PROC/ENDP`, `END MAIN`) every single time, since that's exam muscle memory too.

---

## 0. Foundations — Syntax, Data, Basic Instructions, DOS I/O (Ch. 4)

### 0.1 The Complete Program Template (memorize this cold)

```asm
TITLE TEMPLATE: BASIC PROGRAM SKELETON
.MODEL SMALL
.STACK 100H
.DATA
    ; variable definitions go here
.CODE
MAIN    PROC
    MOV AX, @DATA    ; only needed if you have a .DATA segment
    MOV DS, AX       ; can't move a constant directly into a segment register
    ; instructions go here
    MOV AH, 4CH      ; DOS exit function
    INT 21H          ; return control to DOS
MAIN    ENDP
    END MAIN
```

### 0.2 Byte, Word, and Named Constants (DB, DW, EQU)

```asm
TITLE EX4_1_DATA_DEFS
.MODEL SMALL
.STACK 100H
.DATA
LF        EQU   0AH             ; named constant, no memory allocated
CR        EQU   0DH
ALPHA     DB    4                ; byte, initialized
BYT       DB    ?                ; byte, uninitialized
WRD       DW    -2               ; word, initialized
SUM       DW    ?                ; word, uninitialized
B_ARRAY   DB    10H, 20H, 30H    ; byte array
W_ARRAY   DW    1000, 40, 29887, 329  ; word array
LETTERS   DB    'ABC'            ; same as DB 41H, 42H, 43H
MASK      EQU   10010010B        ; binary constant
.CODE
MAIN PROC
    MOV AX, @DATA
    MOV DS, AX
    MOV DL, LF        ; identical machine code to MOV DL, 0AH
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

### 0.3 MOV, XCHG — Rules and Restrictions

```asm
TITLE EX4_2_MOV_XCHG
.MODEL SMALL
.STACK 100H
.DATA
WORD1 DW 100
WORD2 DW 200
.CODE
MAIN PROC
    MOV AX, @DATA
    MOV DS, AX
    MOV AX, WORD1      ; copy WORD1 into AX (WORD1 unchanged)
    XCHG AX, WORD2     ; swap AX and WORD2
    ; MOV WORD1, WORD2  <-- ILLEGAL: no direct memory-to-memory
    MOV AX, WORD2      ; must go through a register instead
    MOV WORD1, AX
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

### 0.4 ADD, SUB, INC, DEC, NEG

```asm
TITLE EX4_3_ARITHMETIC_BASICS
.MODEL SMALL
.STACK 100H
.DATA
WORD1 DW 10
BYTE1 DB 5
BYTE2 DB 3
.CODE
MAIN PROC
    MOV AX, @DATA
    MOV DS, AX
    ADD WORD1, AX      ; WORD1 = WORD1 + AX
    SUB AX, 4          ; AX = AX - 4
    INC WORD1          ; WORD1 = WORD1 + 1
    DEC AX             ; AX = AX - 1
    ; ADD BYTE1, BYTE2  <-- ILLEGAL: no direct memory-to-memory
    MOV AL, BYTE2      ; fix: go through a register
    ADD BYTE1, AL
    MOV BX, 2
    NEG BX             ; BX = FFFEh (two's complement of 2, i.e. -2)
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

### 0.5 High-Level Assignment Translations

```asm
TITLE EX4_4_HLL_TRANSLATIONS
.MODEL SMALL
.STACK 100H
.DATA
A DW 5
B DW 12
.CODE
MAIN PROC
    MOV AX, @DATA
    MOV DS, AX
    ; B = A
    MOV AX, A
    MOV B, AX
    ; A = 5 - A   (shorter version using NEG)
    NEG A
    ADD A, 5
    ; A = B - 2*A
    MOV AX, B
    SUB AX, A
    SUB AX, A
    MOV A, AX
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

### 0.6 PGM4_1 — Echo Program (Function 1 read, Function 2 display)

```asm
TITLE PGM4_1: ECHO PROGRAM
.MODEL SMALL
.STACK 100H
.CODE
MAIN    PROC
    ; display prompt
    MOV AH, 2
    MOV DL, '?'
    INT 21H
    ; input a character
    MOV AH, 1
    INT 21H
    MOV BL, AL          ; save it (function 2 will overwrite AL)
    ; go to new line
    MOV AH, 2
    MOV DL, 0DH
    INT 21H
    MOV DL, 0AH
    INT 21H
    ; display character
    MOV DL, BL
    INT 21H
    ; return to DOS
    MOV AH, 4CH
    INT 21H
MAIN    ENDP
    END MAIN
```

### 0.7 PGM4_2 — Print String Program (Function 9, LEA, @DATA fix)

```asm
TITLE PGM4_2: PRINT STRING PROGRAM
.MODEL SMALL
.STACK 100H
.DATA
MSG    DB    'HELLO!$'
.CODE
MAIN    PROC
    ; initialize DS (DOS points DS at the PSP, not your data, by default)
    MOV AX, @DATA
    MOV DS, AX
    ; display message
    LEA DX, MSG
    MOV AH, 9
    INT 21H
    ; return to DOS
    MOV AH, 4CH
    INT 21H
MAIN    ENDP
    END MAIN
```

### 0.8 PGM4_3 — Case Conversion Program (full combined example)

```asm
TITLE PGM4_3: CASE CONVERSION PROGRAM
.MODEL SMALL
.STACK 100H
.DATA
MSG1    DB    'ENTER A LOWER CASE LETTER: $'
MSG2    DB    0DH, 0AH, 'IN UPPER CASE IT IS: '
CHAR    DB    ?, '$'
.CODE
MAIN    PROC
    ; initialize DS
    MOV AX, @DATA
    MOV DS, AX
    ; print user prompt
    LEA DX, MSG1
    MOV AH, 9
    INT 21H
    ; input character and convert to upper case
    MOV AH, 1
    INT 21H             ; read lowercase letter into AL
    SUB AL, 20H         ; 'a'-'A' = 20h, so subtracting converts case
    MOV CHAR, AL        ; store it in CHAR
    ; display on next line (MSG2 + CHAR are consecutive in memory)
    LEA DX, MSG2
    MOV AH, 9
    INT 21H
    ; DOS exit
    MOV AH, 4CH
    INT 21H
MAIN    ENDP
    END MAIN
```

---

## 1. FLAGS Register — Core Demo (Ch. 5)

```asm
TITLE PGM5_1: CHECK FLAGS
;used in DEBUG to check flag settings
.MODEL SMALL
.STACK 100H
.CODE
MAIN PROC
    MOV AX, 4000H    ;AX = 4000h
    ADD AX, AX       ;AX = 8000h
    SUB AX, 0FFFFH   ;AX = 8001h
    NEG AX           ;AX = 7FFFh
    INC AX           ;AX = 8000h
    MOV AH, 4CH
    INT 21H          ;DOS exit
MAIN ENDP
    END MAIN
```

### 1.1 Example 5.1 — ADD AX, BX (both FFFFh)

```asm
TITLE EX5_1: ADD FLAGS TEST
.MODEL SMALL
.STACK 100H
.CODE
MAIN PROC
    MOV AX, 0FFFFH
    MOV BX, 0FFFFH
    ADD AX, BX        ; AX = FFFEh, CF=1, OF=0, SF=1, ZF=0, PF=0
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

### 1.2 Example 5.2 — ADD AL, BL (both 80h)

```asm
TITLE EX5_2: ADD BYTE FLAGS TEST
.MODEL SMALL
.STACK 100H
.CODE
MAIN PROC
    MOV AL, 80H
    MOV BL, 80H
    ADD AL, BL        ; AL = 00h, CF=1, OF=1, SF=0, ZF=1, PF=1
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

### 1.3 Example 5.3 — SUB AX, BX (8000h − 0001h)

```asm
TITLE EX5_3: SUB FLAGS TEST
.MODEL SMALL
.STACK 100H
.CODE
MAIN PROC
    MOV AX, 8000H
    MOV BX, 0001H
    SUB AX, BX        ; AX = 7FFFh, CF=0, OF=1, SF=0, ZF=0, PF=1
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

### 1.4 Example 5.4 — INC AL (FFh, shows CF is untouched)

```asm
TITLE EX5_4: INC FLAGS TEST
.MODEL SMALL
.STACK 100H
.CODE
MAIN PROC
    MOV AL, 0FFH
    INC AL            ; AL = 00h, ZF=1, SF=0, PF=1, OF=0
                       ; CF is UNAFFECTED by INC (keeps whatever it was before)
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

### 1.5 Example 5.5 — MOV AX, −5 (no flags change)

```asm
TITLE EX5_5: MOV NO FLAGS
.MODEL SMALL
.STACK 100H
.CODE
MAIN PROC
    MOV AX, -5         ; AX = FFFBh, NO flags touched at all
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

### 1.6 Example 5.6 — NEG AX (8000h wraps to itself)

```asm
TITLE EX5_6: NEG FLAGS TEST
.MODEL SMALL
.STACK 100H
.CODE
MAIN PROC
    MOV AX, 8000H
    NEG AX             ; AX = 8000h (wraps!), CF=1, OF=1, SF=1, ZF=0, PF=1
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

---

## 2. Unconditional & Conditional Jumps (Ch. 6)

### 2.1 PGM6_1 — Display All 256 IBM Characters (JNZ loop)

```asm
TITLE PGM6_1: IBM CHARACTER DISPLAY
.MODEL SMALL
.STACK 100H
.CODE
MAIN PROC
    MOV AH, 2         ; display char function
    MOV CX, 256       ; no. of chars to display
    MOV DL, 0         ; DL has ASCII code of null char
PRINT_LOOP:
    INT 21h           ; display a char
    INC DL            ; increment ASCII code
    DEC CX            ; decrement counter
    JNZ PRINT_LOOP    ; keep going if CX not 0
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

### 2.2 CMP + JG (basic conditional jump)

```asm
TITLE EX_CMP_JG
.MODEL SMALL
.STACK 100H
.CODE
MAIN PROC
    MOV AX, 7FFFH
    MOV BX, 0001H
    CMP AX, BX
    JG  BELOW         ; AX > BX (signed) -> jump taken
    MOV CX, 0          ; skipped
    JMP DONE
BELOW:
    MOV CX, 1          ; executed
DONE:
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

### 2.3 Signed vs Unsigned Jump Trap

```asm
TITLE EX_SIGNED_UNSIGNED_TRAP
.MODEL SMALL
.STACK 100H
.CODE
MAIN PROC
    MOV AX, 7FFFH      ; signed: +32767, unsigned: 32767
    MOV BX, 8000H      ; signed: -32768, unsigned: 32768
    CMP AX, BX
    JA  BELOW          ; UNSIGNED "above" check -> AX(32767) < BX(32768) -> NOT taken
    MOV CX, 0          ; this executes (no jump happened)
    JMP DONE
BELOW:
    MOV CX, 1
DONE:
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

### 2.4 Example 6.1 — Bigger of Two Signed Numbers into CX (book style)

```asm
TITLE EX6_1_MAX_BOOK
.MODEL SMALL
.STACK 100H
.CODE
MAIN PROC
    MOV AX, 25
    MOV BX, 40
    MOV CX, AX     ; assume AX is the maximum for now
    CMP BX, CX
    JLE NEXT       ; BX <= CX, keep CX as is
    MOV CX, BX     ; BX is bigger, update CX
NEXT:
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

### 2.4b Example 6.1 — Explicit Double-Jump Style

```asm
TITLE EX6_1_MAX_EXPLICIT
.MODEL SMALL
.STACK 100H
.CODE
MAIN PROC
    MOV AX, 25
    MOV BX, 40
    MOV CX, AX       ; assume AX is the maximum for now
    CMP BX, CX
    JG  THEN_        ; BX > CX -> go to THEN
    JLE END_IF       ; BX <= CX -> skip, CX already holds max
THEN_:
    MOV CX, BX       ; BX is bigger -> update CX
END_IF:
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

### 2.5 Double-Jump Trick (getting around 126-byte range limit)

```asm
TITLE EX_LONG_LOOP_DOUBLEJUMP
.MODEL SMALL
.STACK 100H
.CODE
MAIN PROC
    MOV CX, 5
TOP:
    ; imagine a very long loop body sits here (too far for direct JNZ)
    NOP
    DEC CX
    JNZ BOTTOM       ; BOTTOM is close by -> always in range
    JMP EXIT         ; CX hit 0 -> skip the loop-back
BOTTOM:
    JMP TOP          ; unconditional jump, no range limit
EXIT:
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

---

## 3. IF / CASE Structures (Ch. 6.4.1)

### 3.1 Example 6.2 — IF-THEN: Absolute Value of AX

```asm
TITLE EX6_2_ABS_VALUE
.MODEL SMALL
.STACK 100H
.CODE
MAIN PROC
    MOV AX, -7
    CMP AX, 0
    JNL END_IF       ; not less than 0 -> skip
    NEG AX           ; AX < 0 -> negate it
END_IF:
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

### 3.2 Example 6.3 — IF-THEN-ELSE: Display Alphabetically First of AL, BL

```asm
TITLE EX6_3_IF_THEN_ELSE
.MODEL SMALL
.STACK 100H
.CODE
MAIN PROC
    MOV AL, 'K'
    MOV BL, 'D'
    MOV AH, 2          ; prepare to display
    CMP AL, BL
    JNBE ELSE_         ; not (AL<=BL) -> go to else (unsigned, for ext. ASCII)
    MOV DL, AL
    JMP DISPLAY        ; skip the else branch
ELSE_:
    MOV DL, BL
DISPLAY:
    INT 21h            ; display it
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

### 3.3 Example 6.4 — CASE: Classify AX as Negative/Zero/Positive

```asm
TITLE EX6_4_CASE_SIGN
.MODEL SMALL
.STACK 100H
.CODE
MAIN PROC
    MOV AX, -3
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
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

### 3.4 Example 6.5 — CASE: Display 'o' for 1/3, 'e' for 2/4

```asm
TITLE EX6_5_CASE_MULTI
.MODEL SMALL
.STACK 100H
.CODE
MAIN PROC
    MOV AL, 3
    CMP AL, 1
    JE  ODD
    CMP AL, 3
    JE  ODD
    CMP AL, 2
    JE  EVEN
    CMP AL, 4
    JE  EVEN
    JMP END_CASE       ; not 1..4, skip everything
ODD:
    MOV DL, 'o'
    JMP DISPLAY
EVEN:
    MOV DL, 'e'
DISPLAY:
    MOV AH, 2
    INT 21H
END_CASE:
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

### 3.5 Example 6.6 — AND Condition: Display Only if Uppercase Letter

```asm
TITLE EX6_6_AND_CONDITION
.MODEL SMALL
.STACK 100H
.CODE
MAIN PROC
    MOV AH, 1
    INT 21H            ; read a character into AL
    CMP AL, 'A'
    JNGE END_IF        ; char < 'A' -> fails, bail out
    CMP AL, 'Z'
    JNLE END_IF        ; char > 'Z' -> fails, bail out
    MOV DL, AL
    MOV AH, 2
    INT 21H            ; display char
END_IF:
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

### 3.6 Example 6.7 — OR Condition: 'y'/'Y' Displays, Else Exits

```asm
TITLE EX6_7_OR_CONDITION
.MODEL SMALL
.STACK 100H
.CODE
MAIN PROC
    MOV AH, 1
    INT 21H            ; read a character into AL
    CMP AL, 'y'
    JE  THEN_          ; matches first option
    CMP AL, 'Y'
    JE  THEN_          ; matches second option
    JMP ELSE_          ; neither matched
THEN_:
    MOV AH, 2
    MOV DL, AL
    INT 21H
    JMP END_IF
ELSE_:
    MOV AH, 4CH
    INT 21H            ; DOS exit
END_IF:
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

---

## 4. FOR / WHILE / REPEAT Loops (Ch. 6.4.2)

### 4.1 Example 6.8 — LOOP Instruction: 80 Stars

```asm
TITLE EX6_8_LOOP_STARS
.MODEL SMALL
.STACK 100H
.CODE
MAIN PROC
    MOV CX, 80        ; number of stars
    MOV AH, 2          ; display character function
    MOV DL, '*'        ; character to display
TOP:
    INT 21h            ; display a star
    LOOP TOP           ; DEC CX, jump to TOP if CX != 0
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

### 4.2 JCXZ-Guarded LOOP (avoids the CX=0 wraparound bug)

```asm
TITLE EX_JCXZ_GUARD
.MODEL SMALL
.STACK 100H
.CODE
MAIN PROC
    MOV CX, 0          ; try changing this to 0 to see the bug WITHOUT the guard
    MOV AH, 2
    MOV DL, '*'
    JCXZ SKIP          ; if CX is already 0, skip the whole loop
TOP:
    INT 21h
    LOOP TOP
SKIP:
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

### 4.3 Example 6.9 — WHILE Loop: Count Characters Until Carriage Return

```asm
TITLE EX6_9_WHILE_COUNT
.MODEL SMALL
.STACK 100H
.CODE
MAIN PROC
    MOV DX, 0          ; DX counts characters
    MOV AH, 1
    INT 21H            ; read first character into AL
WHILE_:
    CMP AL, 0DH        ; is it carriage return?
    JE  END_WHILE      ; yes -> exit loop
    INC DX             ; not CR -> count it
    INT 21H            ; read next character
    JMP WHILE_         ; loop back to check condition again
END_WHILE:
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

### 4.4 Example 6.10 — REPEAT Loop: Read Until Blank

```asm
TITLE EX6_10_REPEAT_BLANK
.MODEL SMALL
.STACK 100H
.CODE
MAIN PROC
    MOV AH, 1
REPEAT_:
    INT 21H            ; read a character into AL
    CMP AL, ' '        ; is it a blank?
    JNE REPEAT_        ; not blank -> loop back
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

---

## 5. Full Program Design — Top-Down Worked Example (Ch. 6.5)

### 5.1 PGM6_2 — First and Last Capital Letters

```asm
TITLE PGM6_2: FIRST AND LAST CAPITALS
.MODEL SMALL
.STACK 100H
.DATA
PROMPT    DB 'Type a line of text', 0DH, 0AH, '$'
NOCAP_MSG DB 0DH, 0AH, 'No capitals $'
CAP_MSG   DB 0DH, 0AH, 'First capital = '
FIRST     DB ']'
          DB ' Last capital = '
LAST      DB '@ $'
.CODE
MAIN PROC
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
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

---

## 6. Logic Instructions — AND, OR, XOR, NOT, TEST (Ch. 7 review)

### 6.1 Clearing a Register / Setting / Clearing Bits

```asm
TITLE EX7_1_AND_OR_XOR
.MODEL SMALL
.STACK 100H
.CODE
MAIN PROC
    XOR AX, AX         ; AX = 0000h, clears register (also ZF=1)
    MOV AL, 00H
    OR  AL, 81H        ; sets MSB and LSB, AL = 81h
    MOV AL, 0FFH
    AND AL, 7FH        ; clears sign bit, AL = 7Fh
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

### 6.2 ASCII Digit Character to Numeric Value (AND 0Fh)

```asm
TITLE EX7_2_ASCII_TO_DIGIT
.MODEL SMALL
.STACK 100H
.CODE
MAIN PROC
    MOV AL, '5'        ; AL = 35h
    AND AL, 0FH        ; AL = 05h (numeric value 5)
    ; reverse trick: OR AL, 30H  -> converts digit back to its ASCII char
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

### 6.3 NOT Instruction

```asm
TITLE EX7_3_NOT
.MODEL SMALL
.STACK 100H
.CODE
MAIN PROC
    MOV AL, 0F0H
    NOT AL             ; AL = 0Fh (one's complement, flips every bit)
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

### 6.4 TEST Instruction — Even/Odd Check

```asm
TITLE EX7_4_TEST_ODD_EVEN
.MODEL SMALL
.STACK 100H
.CODE
MAIN PROC
    MOV AL, 7          ; try 6 for the even case
    TEST AL, 1         ; AL AND 1, result discarded, only flags set
    JZ  EVEN_NUM
    JNZ ODD_NUM
EVEN_NUM:
    MOV BX, 0
    JMP DONE
ODD_NUM:
    MOV BX, 1
DONE:
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

---

## 7. Shift & Rotate Instructions (Ch. 7 review)

### 7.1 SHL/SAL — DH = 8Ah, Shift Left by 3

```asm
TITLE EX7_5_SHL_BYTE
.MODEL SMALL
.STACK 100H
.CODE
MAIN PROC
    MOV DH, 8AH
    MOV CL, 3
    SHL DH, CL         ; DH = 50h, CF=0 (bits fall off the top, gone forever)
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

### 7.2 SHL — Same Bits but 16-bit Register (compare with 7.1)

```asm
TITLE EX7_6_SHL_WORD
.MODEL SMALL
.STACK 100H
.CODE
MAIN PROC
    MOV DX, 008AH
    MOV CX, 3
    SHL DX, CX         ; DX = 0450h (nowhere for bits to fall off, room in upper byte)
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

### 7.3 SHR vs SAR — AL = −15, Shift Right by 1

```asm
TITLE EX7_7_SHR_VS_SAR
.MODEL SMALL
.STACK 100H
.CODE
MAIN PROC
    MOV AL, -15        ; AL = F1h = 1111 0001
    MOV CL, 1
    SHR AL, CL         ; AL = 78h (=120), CF=1, OF=1 (sign flipped: unsigned-style shift)
    MOV AL, -15        ; reset AL
    SAR AL, CL         ; AL = F8h (=-8), CF=1, OF=0 (sign preserved: signed-style shift)
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

### 7.4 Multiplication via SHL — the Flag-Lying Trap (DH=80h, shift by 2)

```asm
TITLE EX7_8_SHL_OVERFLOW_TRAP
.MODEL SMALL
.STACK 100H
.CODE
MAIN PROC
    MOV DH, 80H        ; 128 decimal
    MOV CL, 2
    SHL DH, CL         ; true answer = 128*4 = 512 (needs 9 bits!)
                        ; but DH = 00h, CF=0, OF=0 -- flags LIE, real overflow occurred
                        ; (CF/OF only reflect the LAST of the internal single-bit shifts)
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

---

## 8. Multiplication — MUL and IMUL (Ch. 9)

### 8.1 Basic Word Multiplication

```asm
TITLE EX9_1_MUL_BASIC
.MODEL SMALL
.STACK 100H
.CODE
MAIN PROC
    MOV AX, 5
    MOV BX, 3
    MUL BX             ; DX:AX = AX * BX -> AX = 15, DX = 0
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

### 8.2 MUL vs IMUL on Same Bits (AX=0001h, BX=FFFFh)

```asm
TITLE EX9_2_MUL_VS_IMUL
.MODEL SMALL
.STACK 100H
.CODE
MAIN PROC
    MOV AX, 0001H
    MOV BX, 0FFFFH
    MUL BX             ; unsigned: 1*65535 = 65535 -> DX=0000h AX=FFFFh
    MOV AX, 0001H      ; reset AX (MUL overwrote it)
    IMUL BX            ; signed: 1*(-1) = -1        -> DX=FFFFh AX=FFFFh
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

### 8.3 Byte Multiplication (AL=80h, BL=FFh)

```asm
TITLE EX9_3_BYTE_MUL
.MODEL SMALL
.STACK 100H
.CODE
MAIN PROC
    MOV AL, 80H
    MOV BL, 0FFH
    MUL BL             ; unsigned: 128*255 = 32640 -> AH=7Fh AL=80h, CF/OF=1
    MOV AL, 80H        ; reset AL
    IMUL BL            ; signed: (-128)*(-1) = 128  -> AH=00h AL=80h, CF/OF=1 (mismatch: AL negative but AH=00)
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

### 8.4 C-Style Multiplication Translation

```asm
TITLE EX9_4_C_STYLE_MUL
.MODEL SMALL
.STACK 100H
.DATA
X DW 8000H
Y DW 0FFFFH
.CODE
MAIN PROC
    MOV AX, @DATA
    MOV DS, AX
    MOV AX, X          ; x goes into AX
    MOV BX, Y          ; y goes into BX
    IMUL BX            ; AX = AX * BX  -> this IS "x = x*y" (signed)
    MOV X, AX          ; store result back into x
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

---

## 9. Division — DIV and IDIV (Ch. 9)

### 9.1 Basic Word Division with Sign Extension

```asm
TITLE EX9_5_DIV_BASIC
.MODEL SMALL
.STACK 100H
.CODE
MAIN PROC
    MOV AX, 0005H
    XOR DX, DX         ; clear DX (unsigned dividend upper half)
    MOV BX, 0FFFEH     ; -2 signed
    DIV BX             ; unsigned: 5/65534 = 0 remainder 5 -> AX=0000h DX=0005h
    MOV AX, 0005H
    CWD                ; sign-extend AX into DX (DX=0000h since AX positive)
    IDIV BX            ; signed: 5/(-2) = -2 remainder 1  -> AX=FFFEh DX=0001h
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

### 9.2 Signed Word Division — Truncation Toward Zero

```asm
TITLE EX9_6_SIGNED_DIV_TRUNC
.MODEL SMALL
.STACK 100H
.CODE
MAIN PROC
    MOV AX, -1250
    CWD                ; DX = FFFFh, sign-extend AX (AX is negative)
    MOV BX, 7
    IDIV BX            ; -1250/7 = -178 remainder -4 (truncated toward 0, not -inf)
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

### 9.3 Byte Division with CBW

```asm
TITLE EX9_7_BYTE_DIV_CBW
.MODEL SMALL
.STACK 100H
.CODE
MAIN PROC
    MOV AL, -50
    CBW                ; sign-extend AL into AH (fills upper half correctly)
    MOV BL, 8
    IDIV BL            ; -50/8 = -6 remainder -2 -> AL=quotient, AH=remainder
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

### 9.4 Unsigned Byte Division (clear AH manually)

```asm
TITLE EX9_8_BYTE_DIV_UNSIGNED
.MODEL SMALL
.STACK 100H
.CODE
MAIN PROC
    MOV AL, 200
    MOV AH, 0          ; clear upper half (unsigned dividend)
    MOV BL, 9
    DIV BL             ; 200/9 = 22 remainder 2 -> AL=22, AH=2
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

---

## 10. Arrays & Addressing Modes

### 10.1 Array Declarations with DUP

```asm
TITLE EX10_1_ARRAY_DECL
.MODEL SMALL
.STACK 100H
.DATA
MSG   DB 'abcde'                    ; 5-byte array
W     DW 10, 20, 30, 40, 50, 60     ; 6-word array (12 bytes)
GAMMA DW 100 DUP (0)                ; 100 words, all initialized to 0
DELTA DB 212 DUP (?)                ; 212 bytes, uninitialized
LINE  DB 5, 4, 3 DUP (2, 3 DUP (0), 1)  ; nested DUP
.CODE
MAIN PROC
    MOV AX, @DATA
    MOV DS, AX
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

### 10.2 Register Indirect Mode — Sum All Elements of an Array

```asm
TITLE EX10_2_ARRAY_SUM
.MODEL SMALL
.STACK 100H
.DATA
W DW 10, 20, 30, 40, 50, 60, 70, 80, 90, 100
.CODE
MAIN PROC
    MOV AX, @DATA
    MOV DS, AX
    XOR  AX, AX        ; AX holds sum, start at 0
    LEA  SI, W         ; SI = offset address of W
    MOV  CX, 10        ; CX = number of elements to add
ADDNOS:
    ADD  AX, [SI]      ; sum = sum + value pointed to by SI
    ADD  SI, 2         ; move pointer forward by 2 bytes (1 word)
    LOOP ADDNOS        ; repeat until CX = 0
    ; final AX = 550
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

### 10.3 Based Mode — Access 3rd Element of Word Array via BX

```asm
TITLE EX10_3_BASED_MODE
.MODEL SMALL
.STACK 100H
.DATA
W DW 10, 20, 30, 40, 50, 60
.CODE
MAIN PROC
    MOV AX, @DATA
    MOV DS, AX
    MOV BX, 4          ; (3-1)*2 = 4, byte-offset for 3rd element
    MOV AX, [BX + W]   ; AX = 30 (3rd element)
    ; all of these are equivalent:
    ; MOV AX, [W + BX]
    ; MOV AX, [BX] + W
    ; MOV AX, W + [BX]
    ; MOV AX, W[BX]
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

### 10.4 Indexed Mode — Same Idea Using SI

```asm
TITLE EX10_4_INDEXED_MODE
.MODEL SMALL
.STACK 100H
.DATA
W DW 10, 20, 30, 40, 50, 60
.CODE
MAIN PROC
    MOV AX, @DATA
    MOV DS, AX
    MOV SI, 4          ; byte-offset for 3rd element
    MOV AX, [SI + W]   ; AX = 30 (3rd element), same logic as Based mode
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

### 10.5 PTR Operator — Disambiguating Memory Access Size

```asm
TITLE EX10_5_PTR_OPERATOR
.MODEL SMALL
.STACK 100H
.DATA
ARR DB 10 DUP (0)
.CODE
MAIN PROC
    MOV AX, @DATA
    MOV DS, AX
    LEA BX, ARR
    MOV BYTE PTR [BX], 1     ; store 1 as a single byte
    MOV WORD PTR [BX], 1     ; store 1 as a full word (2 bytes)
    ; MOV [BX], 1  <-- NOT ALLOWED without PTR (assembler can't tell the size)
    MOV AH, 4CH
    INT 21H
MAIN ENDP
    END MAIN
```

---

## Exam-Day Quick Recall Table

| Concept | Instruction(s) | One-line reminder |
|---|---|---|
| Unsigned overflow | CF | carry out of MSB |
| Signed overflow | OF | wrong sign on result |
| Zero result | ZF | result == 0 |
| Negative result | SF | MSB of result == 1 |
| Even parity | PF | even count of 1s in low byte |
| Unconditional jump | JMP | no range limit |
| Conditional jump range | — | -126 to +127 bytes only |
| Loop shorthand | LOOP | DEC CX then JNZ, in one instruction |
| CX=0 bug fix | JCXZ | check before entering loop |
| Mask bits off | AND | 0 in mask clears that bit |
| Force bits on | OR | 1 in mask sets that bit |
| Flip all bits | NOT | one's complement |
| Set flags only | TEST | like AND but discards result |
| Multiply by 2 | SHL/SAL | left shift, MSB -> CF |
| Divide by 2 (signed) | SAR | right shift, MSB preserved (sign) |
| Divide by 2 (unsigned) | SHR | right shift, 0 -> MSB |
| MUL/IMUL operand rule | — | 1 operand typed; other is AL/AX; answer in AX or DX:AX |
| DIV/IDIV operand rule | — | divisor typed; dividend always AX or DX:AX |
| Sign-extend before signed division | CBW / CWD | AL->AX / AX->DX:AX |
| Array element address | — | `A + (position-1)*S` |
| Pointer registers | BX, SI, DI, BP | BX/SI/DI -> DS; BP -> SS |
