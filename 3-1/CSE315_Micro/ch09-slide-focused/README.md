# Chapter 9 — Multiplication and Division Instructions

---

## Part 1 — Multiplication: MUL and IMUL

**`MUL reg`** / **`IMUL reg`** — only ONE register is typed in the instruction. The second number is never typed — it's always taken automatically from **AL** (if `reg` is 8-bit) or **AX** (if `reg` is 16-bit). The answer is also never typed — it always lands automatically in **AX** (byte case) or **DX:AX** (word case).

**MUL** = unsigned. **IMUL** = signed. Same rule, just different interpretation of the bits.

### Byte multiplication (8-bit reg named)

```
AX = AL × reg
```
- Typed operand (`reg`, e.g. BL) = one number.
- AL = the other number, automatically.
- AX = the answer, automatically. (Since 8-bit × 8-bit can need up to 16 bits, the full AX is used to hold it — nothing is lost.)

### Word multiplication (16-bit reg named)

```
DX:AX = AX × reg
```
- Typed operand (`reg`, e.g. BX) = one number.
- AX = the other number, automatically.
- Answer is 32 bits, so it's split: DX = upper 16 bits, AX = lower 16 bits.

### Worked example

```asm
MOV AX, 5
MOV BX, 3
MUL BX
```
- Before: AX = 5, BX = 3.
- `MUL BX` → multiplies AX × BX = 5 × 3 = 15.
- After: AX = 15, DX = 0.

**AX is overwritten** — it went in as an ingredient (5) and came out as the answer (15). This is the single most important thing to remember about MUL/IMUL.

### Signed example (why MUL ≠ IMUL on the same bits)

AX = `0001h`, BX = `FFFFh`.

```asm
MUL BX     ; unsigned: 1 × 65535 = 65535   → DX=0000h AX=FFFFh
IMUL BX    ; signed:   1 × (-1)  = -1      → DX=FFFFh AX=FFFFh
```
Same bits in BX (`FFFFh`), completely different answer, because MUL reads it as 65535 and IMUL reads it as −1.

### C analogy — this is literally what `x = x*y` compiles to

```c
short int x = 0x8000, y = 0xFFFF;
x = x * y;
```
```asm
MOV AX, x     ; x goes into AX
MOV BX, y     ; y goes into BX
IMUL BX       ; AX = AX * BX  → this IS "x = x*y"
```
x sits in AX → gets multiplied by y (BX) → answer overwrites AX → that's exactly where the C variable `x` lives. Same code with `MUL BX` instead of `IMUL BX` = the unsigned version. Nothing else changes.

### Flags: CF/OF tell you "did I actually need the upper half?"

- SF, ZF, AF, PF: undefined, ignore them.
- **CF/OF = 0** → the upper half (DX or AH) is "empty" (all zero for MUL, or correct sign-extension for IMUL) → you didn't need it, AX/AL alone was enough.
- **CF/OF = 1** → the upper half actually holds real data → you MUST keep it, don't discard it.

**Point to remember:** CF/OF for MUL/IMUL is not really an "error" flag — it's a "do you need DX/AH or not" flag.

### Example: AX=FFFFh, BX=FFFFh

| Instr | Product | DX | AX | CF/OF |
|---|---|---|---|---|
| `MUL BX` | 65535×65535 = FFFE0001h | FFFE | 0001 | 1 (DX has real data) |
| `IMUL BX` | (−1)×(−1) = 1 = 00000001h | 0000 | 0001 | 0 (DX is just sign-extension of a positive AX) |

### Example: AL=80h, BL=FFh (note: byte case, so BL not BX)

| Instr | Product | AH | AL | CF/OF |
|---|---|---|---|---|
| `MUL BL` | 128×255 = 7F80h | 7F | 80 | 1 |
| `IMUL BL` | (−128)×(−1) = 128 = 0080h | 00 | 80 | 1 (AL=80h is negative, so correct sign-ext would be FFh, not 00h — mismatch) |

---

## Part 2 — Division: DIV and IDIV

Same "only one operand typed" rule as multiplication, but reversed: you name the **divisor**, and the **dividend + quotient + remainder** are all automatic, in fixed registers.

**DIV** = unsigned. **IDIV** = signed.

### Byte division (8-bit reg named = divisor)

```
AL = quotient
AH = remainder
```
dividend = AX (automatic, always — even if your number would fit in AL alone)

### Word division (16-bit reg named = divisor)

```
AX = quotient
DX = remainder
```
dividend = DX:AX (automatic, always — even if your number would fit in AX alone)

**Point to remember:** the dividend is always one size bigger than the divisor. If your actual number is smaller, you must still put it into that bigger space correctly (see sign-extension below) — you can't just leave the upper half untouched.

### Worked example

DX=0000h, AX=0005h, BX=FFFEh (= −2 signed).

```asm
DIV BX     ; unsigned: 5 ÷ 65534 = 0 remainder 5   → AX=0000h DX=0005h
IDIV BX    ; signed:   5 ÷ (-2)  = -2 remainder 1  → AX=FFFEh DX=0001h
```

**Point to remember — signed division truncates toward ZERO, not toward −∞.** `5 ÷ (−2) = −2.5` → truncated to **−2** (not −3). This is different from SAR, which rounds toward −∞. Don't mix these two rules up.

### Divide Overflow

If the quotient is too big to fit in AL (byte) or AX (word) — usually because the divisor is much smaller than the dividend — the program **crashes immediately** with a "Divide Overflow" message. There's no flag to check beforehand; it just halts.

### Sign extension — filling the "extra" upper half correctly before dividing

Since the dividend is always one size bigger than the divisor, and you usually start with a number that only fills the smaller half, you must fill in the upper half yourself before calling DIV/IDIV:

| Division type | Fill upper half with | How |
|---|---|---|
| Word DIV (unsigned) | 0 | clear DX (e.g. `XOR DX,DX` or `MOV DX,0`) |
| Word IDIV (signed) | sign of AX | `CWD` [Convert Word to Doubleword] (copies AX's sign bit into all of DX) |
| Byte DIV (unsigned) | 0 | clear AH (`MOV AH,0`) |
| Byte IDIV (signed) | sign of AL | `CBW` [Convert Byte to Word] (copies AL's sign bit into all of AH) |

**Example — `−1250 ÷ 7`:**
```asm
MOV AX, -1250
CWD          ; DX = FFFFh (sign-extend AX, since AX is negative)
MOV BX, 7
IDIV BX
```

**Point to remember:** forgetting to clear/sign-extend the upper half before DIV/IDIV is the #1 bug here — whatever garbage is sitting in DX/AH gets divided along with your real number, silently giving a wrong answer instead of an error.

---

## Quick-Reference Summary

- `MUL reg` / `IMUL reg`: one operand typed, second one always AL/AX automatically, answer always lands in AX or DX:AX automatically.
- Byte: `AX = AL × reg`. Word: `DX:AX = AX × reg`.
- CF/OF = 1 means "the upper half has real data, don't discard it." CF/OF = 0 means "upper half was unnecessary."
- `DIV reg` / `IDIV reg`: divisor typed, dividend always AX or DX:AX automatically, quotient/remainder land in AL/AH or AX/DX automatically.
- Signed division truncates toward zero (not −∞ like SAR).
- Always clear or sign-extend the upper half of the dividend before dividing — `CBW`/`CWD` for signed, plain clear for unsigned.
- Quotient too big to fit → program halts with "Divide Overflow," no flag warning beforehand.

---

**Reference:** Chapter 9, *Assembly Language Programming* by Yu and Marut.