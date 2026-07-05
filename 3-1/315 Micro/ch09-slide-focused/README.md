
## Part 4 — Chapter 9 (New Material): Multiplication Instructions

This is where the slides move into genuinely new territory: the *real* multiplication instructions, `MUL` and `IMUL` — as opposed to the shift-and-add technique from Chapter 8, which only worked for multiplying by powers of 2 (or required a full loop for general multiplication).

### IMUL and MUL

- **`imul source`** — **signed** multiplication.
- **`mul source`** — **unsigned** multiplication.

The `source` can be a **register or memory location** — but **not a constant** (an immediate value). If you need to multiply by a literal constant, you'd first load that constant into a register, then use that register as the source.

### Byte and Word Multiplication (A × B)

Multiplication instructions on the 8086 always work with an implicit operand convention — there's no way to specify *two* arbitrary registers as both factors; one of the two factors (call it "B") is *always* a fixed, implied register, and the other factor ("A") is whatever you name as the `source` operand.

**If two bytes are multiplied**, the result is a full **16-bit word** (since multiplying two 8-bit numbers can produce a result needing up to 16 bits):
- **A** (the named operand): `source`
- **B** (the implied operand): **AL**
- **Product**: **AX** (the full 16-bit result)

**If two words are multiplied**, the result is a full **32-bit doubleword** (since multiplying two 16-bit numbers can produce a result needing up to 32 bits):
- **A** (the named operand): `source`
- **B** (the implied operand): **AX**
- **Product, most-significant 16 bits**: **DX**
- **Product, least-significant 16 bits**: **AX**

**In short:**
- **Byte form:** `AX = AL × source`
- **Word form:** `DX:AX = AX × source`

(The notation `DX:AX` means "treat DX and AX together as one 32-bit value, with DX holding the upper half and AX the lower half" — exactly analogous to how `CS:IP` or `SS:SP` combine a segment and offset.)

### Worked Example

Suppose AX contains `0001h` and BX contains `FFFFh`:

```asm
mul bx    ; dx = 0000h    ax = FFFFh
imul bx   ; dx = FFFFh    ax = FFFFh   (represents -1)
```

**Walking through `mul bx` (unsigned):** treating both operands as unsigned, AX = 1 and BX = `FFFFh` = 65535. The true product is `1 × 65535 = 65535 = 0000FFFFh` as a 32-bit value — so DX (upper half) = `0000h` and AX (lower half) = `FFFFh`. Matches.

**Walking through `imul bx` (signed):** treating both operands as *signed*, AX = 1 (positive, unaffected by sign), but BX = `FFFFh` is now interpreted as **−1** (since it's all 1-bits in two's complement). The true product is `1 × (−1) = −1`, and −1 as a 32-bit two's complement value is `FFFFFFFFh` — so DX = `FFFFh` and AX = `FFFFh`. Matches — and note how the *exact same bit pattern* in BX (`FFFFh`) produces a completely different mathematical answer depending on whether you use MUL or IMUL, purely because of how the *interpretation* of that bit pattern changes.

### Signed vs. Unsigned Multiplication — C Analogy

The slides show a nice side-by-side comparison of how the *exact same* assembly structure corresponds to either signed or unsigned C code, purely by choosing `IMUL` vs `MUL`:

**Signed multiplication (C: `short int`):**
```c
short int x = 0x8000;
short int y = 0xFFFF;
x = x * y;
```
```asm
MOV x, 8000H
MOV y, FFFFH
MOV AX, x
MOV BX, y
IMUL BX
```

**Unsigned multiplication (C: `unsigned short int`):**
```c
unsigned short int x = 0x8000;
unsigned short int y = 0xFFFF;
x = x * y;
```
```asm
MOV x, 8000H
MOV y, FFFFH
MOV AX, x
MOV BX, y
MUL BX
```

**The key insight here:** the *data* and even the *assembly instructions* look nearly identical — the only thing that changes is swapping `MUL` for `IMUL`. This mirrors exactly how, in C, changing a variable's declared type from `short int` to `unsigned short int` doesn't change the *bits* stored in memory at all — it only changes how the compiler *interprets* those same bits when performing arithmetic on them. Assembly makes this normally-invisible distinction completely explicit: *you* are the one choosing the interpretation, instruction by instruction.

### Effect on Flags for MUL/IMUL

- **SF, ZF, AF, and PF are all undefined** after a multiplication — don't rely on any of them.
- **CF/OF** work as follows:
  - **For MUL:** CF/OF = **0** if the upper half of the result is entirely 0 (meaning the product actually fit within the *lower* half alone, and the full double-width result was unnecessary). CF/OF = **1** otherwise (meaning you genuinely needed the extra width to hold the answer).
  - **For IMUL:** CF/OF = **0** if the upper half of the result is exactly the **sign extension** of the lower half (i.e., all 0s if the lower half is a positive/zero number, or all 1s if the lower half is negative — meaning again that the "extra" width wasn't actually needed to represent the answer). CF/OF = **1** otherwise.

**Why does this make sense?** The upper half (DX, or AH for byte multiplication) is really only needed when the product is "big" — bigger than could fit in the lower half alone. CF/OF = 0 signals "you didn't actually need that extra room, the answer would have fit in just AX (or AL) alone" — a convenient hint that you can safely ignore DX/AH if you only care about a same-size product. CF/OF = 1 warns you: "the answer genuinely needed the full double-width result — don't throw away the upper half, or you'll lose data."

### More Worked Examples

**Case 1 — AX = `FFFFh`, BX = `FFFFh`:**

| Instruction | Hex Product | DX | AX | CF/OF |
|---|---|---|---|---|
| `MUL BX` | `FFFE0001` (4294836225 decimal) | `FFFE` (nonzero) | `0001` | 1 |
| `IMUL BX` | `00000001` | `0000` | `0001` | 0 |

**Checking `MUL BX`:** unsigned, `FFFFh × FFFFh = 65535 × 65535 = 4294836225 = FFFE0001h`. Splitting into halves: DX = `FFFEh` (nonzero, so CF/OF = 1 — correctly signals "you need DX too, don't discard it").

**Checking `IMUL BX`:** signed, both operands are `FFFFh = −1`. `(−1) × (−1) = 1`, giving the 32-bit result `00000001h`. DX = `0000h`, and since AX (`0001h`) is a small positive number, its correct sign extension would indeed be all 0s — which is exactly what DX shows. Since the upper half matches the expected sign extension, CF/OF = 0 — correctly signals "AX alone would have been enough."

**Case 2 — AL = `80h`, BL = `FFh`** *(byte-sized multiplication — see the correction note below)*:

| Instruction | Hex Product | AH | AL | CF/OF |
|---|---|---|---|---|
| `MUL BL` | `7F80` | `7F` (nonzero) | `80` | 1 |
| `IMUL BL` | `0080` | `00` (no sign extension) | `80` | 1 |

**A correction to flag here, Riyad:** the slide labels these two instructions as `MUL BX` and `IMUL BX`, but given that the result table breaks the product down into **AH** and **AL** (rather than DX and AX), and the input values `80h`/`FFh` are only 2 hex digits (i.e., single-byte values), this must actually be **byte-form multiplication** — meaning the source register named should be an **8-bit register** (like **BL**), not the 16-bit **BX**. I've corrected the instruction names in the table above to `MUL BL` / `IMUL BL` to match what's actually being computed; just be aware the original slide text says "BX" in both spots, which is a minor mislabeling.

**Checking `MUL BL`:** unsigned, AL = `80h` = 128, BL = `FFh` = 255. Product = `128 × 255 = 32640 = 7F80h`. AH = `7Fh` (nonzero) → CF/OF = 1.

**Checking `IMUL BL`:** signed, AL = `80h` = **−128** (since MSB is set), BL = `FFh` = **−1**. Product = `(−128) × (−1) = 128 = 0080h`. AH = `00h`. Is this the correct sign extension of AL (`80h`)? AL = `80h` has its MSB set — meaning it represents a *negative* number — so its correct sign extension would be **all 1s** (`FFh`), not `00h`. Since AH doesn't match the expected sign extension, this signals CF/OF = 1 ("you needed the full width — the byte-only interpretation of AL alone would have been wrong / lossy").

---

## Part 5 — Chapter 9 (New Material): Division Instructions

### The Four Division-Related Instructions

- **`cbw`** — convert byte to word (sign-extends AL into all of AX)
- **`cwd`** — convert word to doubleword (sign-extends AX into all of DX:AX)
- **`div source`** — unsigned divide
- **`idiv source`** — signed divide

### Byte and Word Division (A ÷ B)

Just like multiplication had an implicit-operand convention, so does division. **When a division is performed, you always get two results back**: the **quotient** and the **remainder** — and both of these end up being the **same size as the divisor**.

**For the byte form:**
- Divisor (B): the `source` operand you name
- Dividend (A): **AX** (a full 16-bit value gets divided)
- Quotient: **AL**
- Remainder: **AH**

**For the word form:**
- Divisor (B): the `source` operand you name
- Dividend (A): **DX:AX** (a full 32-bit value gets divided)
- Quotient: **AX**
- Remainder: **DX**

**Why does the dividend need to be "one size up" from the divisor?** Think about ordinary long division: dividing, say, a 4-digit number by a 2-digit number is a completely normal, everyday operation — you'd never expect to divide a 2-digit number by another 2-digit number and reliably get a useful multi-digit quotient out. The 8086's division instructions mirror this: to get a *meaningful*, full-width quotient and remainder out of a *byte*-sized divisor, you need to feed in a *word*-sized dividend (twice the width) — and similarly, a *doubleword*-sized dividend for a *word*-sized divisor.

### Worked Example

Suppose DX = `0000h`, AX = `0005h`, and BX = `FFFEh` (which represents **−2** in two's complement):

```asm
div bx    ; ax = 0000h    dx = 0005h
idiv bx   ; ax = FFFEh    dx = 0001h
```

**Walking through `div bx` (unsigned):** the dividend is `DX:AX = 00000005h = 5` (unsigned), and the divisor BX is treated as unsigned `FFFEh = 65534`. Dividing `5 ÷ 65534` gives a quotient of **0** (5 is far smaller than 65534, so it goes in zero whole times) with a remainder of **5** (the whole original dividend is left over). So AX (quotient) = `0000h`, DX (remainder) = `0005h`. Matches.

**Walking through `idiv bx` (signed):** the dividend is still `5` (positive, unaffected by sign), but now BX = `FFFEh` is interpreted as **−2**. Dividing `5 ÷ (−2) = −2.5`. **The 8086 truncates toward zero** for signed division (not "round down" the way SAR does for shifting — division truncation specifically chops off the fractional part, moving *toward* zero rather than *away* from it). Truncating `−2.5` toward zero gives a quotient of **−2** (represented as `FFFEh`). The remainder is then whatever's left to make the arithmetic balance: `5 − ((−2) × (−2)) = 5 − 4 = 1`. So AX (quotient) = `FFFEh`, DX (remainder) = `0001h`. Matches.

*(This "truncate toward zero" detail is worth remembering, Riyad — it's a genuinely different rounding rule than the "round toward negative infinity" behavior you learned for `SAR` back in Chapter 7. Division and right-shifting are related concepts, but they don't handle negative numbers' fractional remainders in the same way.)*

### Divide Overflow

It's possible for the **quotient** to come out **too big to fit** in its designated destination (`AL` for byte division, `AX` for word division). This typically happens **if the divisor is much smaller than the dividend** — e.g., trying to divide a huge number by a tiny one produces a huge quotient that simply can't be represented in the (comparatively small) destination register.

When this happens, **the program terminates immediately**, and the system displays the message **"Divide Overflow"**. Unlike the flag-based overflow detection you saw for addition/subtraction/shifting (where the program keeps running and it's up to *you* to check the flag), a divide overflow is treated as a fatal error that halts execution outright — there's no flag to check *before* the fact; the CPU itself stops the program.

### Sign Extension of the Dividend

This is a subtlety that trips people up: **because the dividend for division always has to be "one size up" from the divisor** (word-sized dividend for a byte divisor, doubleword-sized dividend for a word divisor), you need to correctly fill in that extra width *before* dividing — even in cases where your "real" number would have fit in the smaller size all along.

**Word division** (dividing something into AX, using a word-sized divisor):
- The dividend must be placed in **DX:AX**, even if the actual number you care about would fit entirely within AX alone.
- **For `div` (unsigned):** DX should simply be **cleared** to 0 — since an unsigned number has no sign to extend, the "extra" upper half is just filled with zeros.
- **For `idiv` (signed):** DX should be made the **sign extension** of AX, using the **`cwd`** instruction — meaning DX gets filled entirely with copies of AX's sign bit (all 0s if AX is positive, all 1s if AX is negative), so that the combined 32-bit `DX:AX` value correctly represents the same signed number as AX did on its own.

**Worked example — computing `−1250 ÷ 7`:**
```asm
MOV AX, -1250
CWD          ; sign extend
MOV BX, 7
IDIV BX
```
Since AX starts out negative (`−1250`), `CWD` copies AX's sign bit (1, since it's negative) into every bit of DX — making DX = `FFFFh`. This correctly extends the 16-bit value `−1250` into an equivalent 32-bit signed value `DX:AX`, ready to be divided by the word-sized divisor in BX.

**Byte division** (dividing something into AL, using a byte-sized divisor):
- The dividend must be placed in **AX**, even if the actual number you care about would fit entirely within AL alone.
- **For `div` (unsigned):** AH should simply be **cleared** to 0.
- **For `idiv` (signed):** AH should be made the **sign extension** of AL, using the **`cbw`** instruction (the byte-sized equivalent of `cwd` — copies AL's sign bit into all of AH).

**The general pattern to remember:** whenever you're about to divide, ask yourself "is my dividend currently sitting in a register that's the *correct*, one-size-up width for my divisor?" If it's currently sitting in the smaller register alone, you must explicitly widen it first — with a plain `MOV ..., 0` (or `XOR`) if unsigned, or with `CBW`/`CWD` if signed — **before** issuing `DIV`/`IDIV`. Forgetting this step is a very common source of bugs: leaving garbage in the upper half (DX or AH) will silently corrupt your division, since the CPU will faithfully divide whatever full-width value is actually sitting there, garbage and all.

---

## Quick-Reference Summary

**Flags (review):**
- CF = unsigned overflow indicator (carry out of MSB on addition); not affected by INC/DEC.
- OF = signed overflow indicator (result has wrong sign after adding two same-signed numbers).
- ZF, SF, PF reflect zero-ness, sign, and parity of the low byte respectively.

**Logic instructions (review):** AND clears bits (0 = clear, 1 = preserve), OR sets bits (1 = set, 0 = preserve), XOR flips bits (1 = flip, 0 = preserve), NOT flips everything, TEST is AND-without-writing (for flow control).

**Shift/Rotate (review):** SHL/SAL and SHR/SAR double/halve a value; CF and OF are only trustworthy for single-position shifts, never for multi-position ones. SAR preserves the sign bit (use for signed values); SHR does not (use for unsigned values).

**New — Multiplication:**
- `MUL source` = unsigned; `IMUL source` = signed.
- Byte form: `AX = AL × source`. Word form: `DX:AX = AX × source`.
- CF/OF = 0 means "the upper half wasn't actually needed" (safe to ignore DX/AH); CF/OF = 1 means "you need the full double-width result."

**New — Division:**
- `DIV source` = unsigned; `IDIV source` = signed.
- Byte form: dividend in AX, quotient in AL, remainder in AH. Word form: dividend in DX:AX, quotient in AX, remainder in DX.
- Signed division truncates toward zero (not toward −∞, unlike SAR).
- Must correctly widen the dividend first: clear the upper half for unsigned division, or sign-extend it with CBW/CWD for signed division.
- If the quotient doesn't fit in its destination, the program halts with a "Divide Overflow" error.

---

## References (as cited in the slide deck)
Chapters 7, 9 — *Assembly Language Programming* by Yu and Marut.

---

That covers the whole slide deck, Riyad — flags/logic/shift review plus the new MUL/IMUL/DIV/IDIV material from Chapter 9. Two things I corrected along the way for accuracy (both flagged inline above): the SAR example's OF value, and the register width (BX→BL) in the byte-multiplication example table. Everything else checked out cleanly against the actual arithmetic when I worked through it by hand. Good luck with the exam prep — shout if you want me to also do a proper full Chapter 9 textbook-style breakdown later once you have time to go through the actual book PDF.