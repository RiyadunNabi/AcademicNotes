Full beginner-friendly breakdown of Chapter 7, Riyad — this chapter is where assembly language shows off a superpower that high-level languages (except C) mostly hide from you: **direct control over individual bits**.

---

# Chapter 7 — Logic, Shift, and Rotate Instructions

---

## Overview

Every piece of data in a computer is ultimately just a pattern of bits (0s and 1s). Most of the time, when you write `ADD`, `SUB`, `MOV`, etc., you're treating a byte or word as *a number* — you don't care about its individual bits, just its overall value.

But sometimes you *do* care about individual bits. Examples:
- Is this number even or odd? (look at just bit 0)
- Is this a lowercase or uppercase letter? (look at one specific bit)
- I want to pack several small flags into one byte to save memory — how do I set/clear/check just one of those flags without disturbing the others?

High-level languages other than C generally don't give you a clean way to do this. Assembly language does, through the **logic instructions**: `AND`, `OR`, `XOR`, `NOT`, and `TEST`.

This chapter covers four related topics:

- **Section 7.1 — Logic instructions**: `AND`, `OR`, `XOR`, `NOT` (and the flag-only cousin `TEST`). These let you **clear**, **set**, and **examine** individual bits in a register or memory variable. Used for tasks like converting lowercase to uppercase, and checking if a number is even or odd.
- **Section 7.2 — Shift instructions**: `SHL`/`SAL` (shift left) and `SHR`/`SAR` (shift right). These slide all the bits in an operand left or right by one or more positions. Because shifting left doubles a number and shifting right halves it, these instructions double as a **very fast way to multiply or divide by powers of 2** — much faster than the general `MUL`/`DIV` instructions you'll meet in Chapter 9.
- **Section 7.3 — Rotate instructions**: `ROL`, `ROR`, `RCL`, `RCR`. These work like shifts, except that the bit which gets pushed out one end doesn't just disappear — it wraps around and comes back in at the other end. Useful when you want to inspect or rearrange bits without permanently losing any of them.
- **Section 7.4 — Binary and Hex I/O**: A practical, real-world payoff — using everything from this chapter to read and write numbers in binary and hexadecimal format, character by character, through the keyboard and screen.

---

## 7.1 Logic Instructions

The core idea: the values 0 and 1, which we usually think of as numeric digits, can *also* be interpreted as the logical values **false** and **true**. Once you make that connection, ordinary Boolean logic (AND, OR, XOR, NOT) becomes something the CPU can perform directly on real data.

**Figure 7.1 — Truth tables for AND, OR, XOR, and NOT** (0 = false, 1 = true):

| a | b | a AND b | a OR b | a XOR b |
|---|---|---------|--------|---------|
| 0 | 0 | 0 | 0 | 0 |
| 0 | 1 | 0 | 1 | 1 |
| 1 | 0 | 0 | 1 | 1 |
| 1 | 1 | 1 | 1 | 0 |

| a | NOT a |
|---|-------|
| 0 | 1 |
| 1 | 0 |

In plain English:
- **AND** is true only if *both* inputs are true.
- **OR** is true if *at least one* input is true.
- **XOR** ("exclusive or") is true if the inputs are *different* from each other (one true, one false) — it's false when they're the same.
- **NOT** simply flips the value: 0 becomes 1, and 1 becomes 0.

**The key mechanical rule:** when you apply a logic operation to an 8-bit or 16-bit operand, the CPU doesn't do anything mysterious — it just applies that same truth table independently to *each corresponding pair of bits*, position by position. Bit 0 of the destination is combined with bit 0 of the source, bit 1 with bit 1, and so on. There's no "carrying" between bit positions like there is in addition.

### Example 7.1 — Worked logic operations

Perform the following:
1. `10101010 AND 11110000`
2. `10101010 OR 11110000`
3. `10101010 XOR 11110000`
4. `NOT 10101010`

**Solutions** (worked bit-by-bit, left to right):

1. **AND**: 
   ```
     10101010
   AND 11110000
   = 10100000
   ```
   Only positions where *both* bits are 1 survive as 1. Bits 7,6,5,4 of the first operand are 1,0,1,0 and the mask has 1,1,1,1 there → result keeps 1,0,1,0. Bits 3,2,1,0 of the first operand are 1,0,1,0 but the mask has 0,0,0,0 there → all forced to 0. Result: `10100000`.

2. **OR**:
   ```
     10101010
   OR  11110000
   = 11111010
   ```
   Any position where *either* bit is 1 becomes 1. Result: `11111010`.

3. **XOR**:
   ```
     10101010
   XOR 11110000
   = 01011010
   ```
   Any position where the two bits *differ* becomes 1; where they're the same, becomes 0. Result: `01011010`.

4. **NOT**: simply flip every bit of `10101010` → `01010101`.

---

### 7.1.1 AND, OR, and XOR Instructions

**Format:**
```asm
AND    destination, source
OR     destination, source
XOR    destination, source
```

The result of the operation is **stored into the destination**, overwriting whatever was there before. The destination must be a register or a memory location. The source can be a constant, a register, or a memory location — but (as with most two-operand instructions you've already met) **memory-to-memory is never allowed**; at least one of the two operands must be a register or an immediate constant.

**Effect on flags:**
- `SF`, `ZF`, `PF` reflect the result (same as after most arithmetic — sign flag, zero flag, parity flag are all set based on the actual output value)
- `AF` (auxiliary carry) is left undefined — it has no meaningful use for logic operations
- `CF` and `OF` are always cleared to 0 (a logic operation can never cause a "carry" or numeric overflow the way addition can)

#### The Masking Technique

The real power of AND/OR/XOR comes from **masks**. A mask is simply a specially chosen bit pattern you use as the *source* operand, designed so that when it's combined with the destination, only the bits you care about get changed — everything else is left alone.

To design masks correctly, memorize these six small facts (they fall directly out of the truth tables above), where `b` stands for any single bit (0 or 1):

```
b AND 1 = b        b OR 0 = b        b XOR 0 = b
b AND 0 = 0        b OR 1 = 1        b XOR 1 = ~b (complement of b)
```

Read the left column, middle column, and right column separately — each one tells you something extremely useful:

1. **AND can clear specific bits while preserving the rest.**
   A mask bit of **0** forces that destination bit to 0 (clears it).
   A mask bit of **1** leaves that destination bit exactly as it was (preserves it).

2. **OR can set specific bits while preserving the rest.**
   A mask bit of **1** forces that destination bit to 1 (sets it).
   A mask bit of **0** leaves that destination bit exactly as it was (preserves it).

3. **XOR can complement (flip) specific bits while preserving the rest.**
   A mask bit of **1** flips that destination bit (0→1 or 1→0).
   A mask bit of **0** leaves that destination bit exactly as it was (preserves it).

This is the single most important idea in the whole section — everything else in 7.1 is really just applications of these three rules.

### Example 7.2 — Clear the sign bit of AL

**Task:** Clear the sign bit (bit 7, the most significant bit) of AL, leaving every other bit unchanged.

**Solution:** Since we want to *clear* one specific bit and leave the rest alone, we reach for **AND**. Rule: put a 0 in the mask at the position(s) we want cleared, and 1 everywhere else we want preserved.

Bit 7 needs a 0; bits 6 down to 0 need 1s: `01111111b = 7Fh`.

```asm
AND AL, 7Fh
```

### Example 7.3 — Set the MSB and LSB of AL

**Task:** Set the most significant bit (bit 7) *and* the least significant bit (bit 0) of AL, while preserving all other bits.

**Solution:** Setting bits → use **OR**. Put 1s exactly where you want to force a 1, and 0s everywhere you want to preserve the existing value.

Bit 7 = 1, bit 0 = 1, everything in between = 0: `10000001b = 81h`.

```asm
OR AL, 81h
```

### Example 7.4 — Change (flip) the sign bit of DX

**Task:** Change (i.e., complement/flip) the sign bit of DX, a 16-bit register, without disturbing anything else.

**Solution:** Flipping a specific bit → use **XOR**. Put a 1 at the bit you want flipped, 0 everywhere else.

DX is 16 bits, so its sign bit is bit 15. Mask = `1000000000000000b = 8000h`.

```asm
XOR DX, 8000h
```

**Note from the book:** whenever a mask would be long — especially 16 bits — it's much safer to write it in **hex** rather than typing out 16 individual binary digits by hand. It's very easy to accidentally drop or duplicate a digit in a long binary literal, and hex is far less error-prone.

#### Frequently Occurring Masking Tasks

The book walks through several real, commonly needed jobs that the masking technique solves elegantly.

**Converting an ASCII digit character to its numeric value**

When your program reads a keystroke via DOS function `INT 21h`/`AH=1` (from Chapter 4), the register gets the *ASCII code* of whatever key was pressed — not the number itself. So if the user presses the "5" key, AL doesn't get `5`; it gets `35h` (the ASCII code for the character `'5'`).

One way to convert `35h` down to the actual value `5` is what you already know from earlier chapters:
```asm
SUB AL, 30h
```

But there's an alternative, using AND as a mask, that works because the ASCII codes for digit characters `'0'`–`'9'` run from `30h` to `39h` — in binary, that's always `0011` followed by the actual digit's 4-bit binary value in the low nibble. So if we just **clear the high nibble** (the top 4 bits) and keep the low nibble untouched, we're left with exactly the digit's numeric value:

```asm
AND AL, 0Fh   ; clears the high nibble (bits 4-7), keeps low nibble (bits 0-3)
```

Since `'0'` to `'9'` are all `30h` to `39h`, this mask converts *any* ASCII digit character straight to its decimal value.

**Why prefer AND over SUB here, if both work?** The book makes an important stylistic point: using the *logic* instruction `AND` instead of the *arithmetic* instruction `SUB` **signals to a reader of the code** that what's really going on here is a bit-pattern manipulation, not "real" arithmetic in the usual sense. It's a small thing, but it makes code more self-documenting — someone reading `AND AL, 0Fh` immediately thinks "ah, nibble-masking," whereas `SUB AL, 30h` looks like ordinary subtraction and hides the intent a bit.

*(The reverse task — converting a stored decimal digit back to its ASCII code — is left to Riyad as one of the book's exercises.)*

**Converting a lowercase letter to uppercase**

You already saw one way to do this back in Chapter 4:
```asm
SUB DL, 20h
```

The book now shows *why* that particular constant (`20h`) works, by comparing the actual binary ASCII codes of matching lowercase/uppercase letter pairs:

| Character | Code | Character | Code |
|-----------|----------|-----------|----------|
| a | 01100001 | A | 01000001 |
| b | 01100010 | B | 01000010 |
| … | … | … | … |
| z | 01111010 | Z | 01011010 |

Look closely: in every single pair, the *only* difference between the lowercase and uppercase version is **bit 5**. Lowercase always has bit 5 = 1; uppercase always has bit 5 = 0. Every other bit is identical between the pair.

That means converting lowercase → uppercase doesn't require subtraction at all — it just requires **clearing bit 5** and leaving every other bit exactly as it is. Mask: `11011111b = 0DFh` (a 0 at bit 5, 1s everywhere else).

```asm
AND DL, 0DFh
```

*(The reverse conversion, uppercase → lowercase, is again left as an exercise — hint: you'd need to *set* bit 5 instead, so think about which instruction that calls for.)*

**Clearing a register**

You already know two ways to clear (zero out) a register, say AX:
```asm
MOV AX, 0
```
or
```asm
SUB AX, AX
```

The book now introduces a **third** way, using the fact that `1 XOR 1 = 0` and `0 XOR 0 = 0` — in other words, *anything XORed with itself is always zero*, regardless of what the original bit pattern was:

```asm
XOR AX, AX
```

**Which is best?** It comes down to machine-code size: the `MOV AX, 0` form needs **3 bytes** of machine code (because it has to encode the actual constant 0 inside the instruction), whereas `SUB AX, AX` and `XOR AX, AX` only need **2 bytes** each (no constant needs to be encoded — the CPU just operates the register against itself). So the SUB/XOR forms are slightly more efficient and are commonly preferred for clearing registers.

**Important restriction:** this efficiency trick only works for *registers*. Because of the standing rule that memory-to-memory operations are never allowed, if you need to clear a *memory* location (not a register), you're forced to fall back on the `MOV memloc, 0` form — `SUB memloc, memloc` or `XOR memloc, memloc` would require both operands to be the same memory location twice, which the CPU can't do in a single instruction.

**Testing a register for zero**

At first glance, an instruction like
```asm
OR CX, CX
```
looks pointless — since `1 OR 1 = 1` and `0 OR 0 = 0`, ORing a register with itself never changes its value at all. So why would you ever write this?

The answer: **even though the value doesn't change, the flags still get updated** based on that (unchanged) result. In particular, if CX happens to contain 0, then `ZF` will be set to 1 after this instruction runs — exactly as if you had tested for zero. So `OR CX, CX` becomes a legitimate, commonly-used **alternative** to:
```asm
CMP CX, 0
```
for checking whether a register is zero, or (via the sign flag SF) for checking whether its contents are negative. Some assembly programmers prefer this style because it's a hair shorter/faster than a full CMP.

---

### 7.1.2 NOT Instruction

**Format:**
```asm
NOT destination
```

`NOT` performs the **one's complement** operation on the destination — meaning it simply flips every single bit: every 0 becomes 1, and every 1 becomes 0. There is exactly one operand (unlike AND/OR/XOR, which need a source and destination).

**Effect on flags:** none at all — `NOT` leaves every status flag exactly as it was before the instruction ran. This is a bit unusual compared to most instructions in this chapter, so it's worth remembering as a small exception.

### Example 7.5 — Complement the bits in AX

**Solution:**
```asm
NOT AX
```

That's it — every bit in AX simply flips.

---

### 7.1.3 TEST Instruction

**Format:**
```asm
TEST destination, source
```

`TEST` performs exactly the same computation as `AND destination, source` internally — but, crucially, it **throws away the result** and does *not* store anything back into the destination. The destination keeps its original value, completely untouched. The *entire point* of `TEST` is to update the flags so you can then act on them (usually with a conditional jump), without actually altering any data.

Think of the relationship between `TEST` and `AND` as exactly parallel to the relationship you already learned between `CMP` and `SUB` back in Chapter 6: `CMP` is "SUB, but throw away the answer, just keep the flags," and in the same way, `TEST` is "AND, but throw away the answer, just keep the flags."

**Effect on flags:**
- `SF`, `ZF`, `PF` reflect the result of the (discarded) AND operation
- `AF` is undefined
- `CF`, `OF` = 0

#### Examining Bits

`TEST` is the standard tool for checking whether specific individual bits in an operand are 1 or 0, *without modifying that operand*. The masking rule here mirrors the AND masking rule from 7.1.1: put a **1** in every bit position you want to examine, and **0** everywhere else.

Because `1 AND b = b` and `0 AND b = 0`, the (discarded) result of `TEST destination, mask` will have:
- a **1** in each tested position *if and only if* the destination actually has a 1 there
- a **0** everywhere else (both in untested positions, and in tested positions where the destination happens to have a 0)

So: **if the destination has 0s in *all* the tested positions, the overall (discarded) result is entirely 0 — and therefore `ZF` gets set to 1.** This single fact is what makes `TEST` so useful for bit-checking combined with a jump like `JZ`/`JNZ`.

### Example 7.6 — Jump to BELOW if AL contains an even number

**Solution:** An even number always has a **0 in bit 0** (the least significant bit) — that's literally what "even" means in binary. So we want to test just bit 0. Mask = `00000001b = 1`.

```asm
TEST AL, 1      ; is AL even?
JZ   BELOW      ; yes, go to BELOW
```

Walking through why this works: `TEST AL, 1` effectively computes `AL AND 1` (discarding the result but keeping the flags). If bit 0 of AL is 0 (i.e., AL is even), then the ANDed result is entirely 0, so `ZF` becomes 1 — and `JZ` (jump if zero flag is set) fires, sending execution to `BELOW`. If AL is odd, bit 0 is 1, the ANDed result is nonzero, `ZF` stays 0, and the jump is skipped.

---

## 7.2 Shift Instructions

The shift and rotate instructions both move the bits of an operand sideways — either left or right, by one or more positions. The key difference between the two families:

- **Shift**: any bit pushed out the end is simply **lost** (though as we'll see, it does get captured briefly in CF on its way out).
- **Rotate** (covered in 7.3): any bit pushed out one end gets **put back in at the other end** — nothing is lost; the bits just circulate.

Both families share the same two instruction formats:

For a **single** shift or rotate (moving by exactly 1 position):
```asm
Opcode destination, 1
```

For a shift or rotate of **N** positions:
```asm
Opcode destination, CL
```
where the count `N` is placed in the CL register beforehand. In both formats, the `destination` must be an 8-bit or 16-bit register or memory location.

*(Side note from the book: on Intel's more advanced processors than the 8086, shift/rotate instructions also allow an 8-bit immediate constant directly as the count, instead of requiring CL. But for the 8086 specifically — which is what this course targets — CL is the way to specify a count other than 1.)*

As the chapter will demonstrate, these instructions have a very practical superpower: they let you **multiply and divide by powers of 2 extremely fast**, and they're the foundation for doing binary/hex input and output later in section 7.4.

---

### 7.2.1 Left Shift Instructions

#### The SHL Instruction

**Format:**
```asm
SHL destination, 1
```

`SHL` (**sh**ift **l**eft) slides every bit in the destination one position to the left. Two things happen simultaneously at the two ends:
- A fresh **0** is shifted in at the rightmost (least significant) bit position.
- The **most significant bit** (the one bit that has nowhere left to go) gets pushed out into **CF**.

**Figure 7.2** illustrates this for both a 16-bit word (bits 15 down to 0) and an 8-bit byte (bits 7 down to 0) — in both cases, bits march one position to the left, a 0 enters from the right, and the bit that falls off the left edge lands in CF.

If you need to shift by more than 1 position at once, use:
```asm
SHL destination, CL
```
where CL holds the count `N`. This performs `N` individual single-left-shifts in a row, one after another. Importantly, **CL itself is left unchanged** after the instruction — only the destination is modified.

**Effect on flags:**
- `SF`, `PF`, `ZF` reflect the final result
- `AF` is undefined
- `CF` = the value of the *last* bit that got shifted out (i.e., if you shift by N positions, CF only reflects what happened on the *final* one of those N shifts, not any of the earlier ones)
- `OF` = 1 if the sign of the result changed on that last shift (again, only meaningful for the final shift when N > 1 — more on this shortly under "Overflow")

### Example 7.7 — SHL DH,CL where DH=8Ah, CL=3

**Task:** DH contains `8Ah`, CL contains `3`. What are the new values of DH and CF after `SHL DH,CL` executes?

**Solution:** `8Ah` in binary is `10001010`. We need to shift left 3 times. The easiest way to do this by hand: erase the leftmost 3 bits (they get discarded, one at a time, into CF, with only the very last one actually staying in CF) and append 3 fresh zero bits on the right.

`10001010` → drop the leftmost 3 bits (`100`) → left with `01010` → append three 0s on the right → `01010000b = 50h`.

Since the third (last) bit shifted out happened to be 0, **CF = 0** at the end.

New DH = `50h`, CF = `0`.

#### Multiplication by Left Shift

Here's a nice intuition-building analogy the book uses: think about the *decimal* number 235. If you shift every digit one position to the left and stick a 0 onto the new empty spot on the right, you get 2350 — which is exactly `235 × 10`. Shifting digits left, in whatever number base you're using, is the same operation as multiplying by that base.

The exact same logic applies to *binary* numbers, except the base is 2 instead of 10 — so a **left shift on a binary number doubles its value** (multiplies by 2).

**Worked mini-example:** suppose AL contains `5 = 00000101b`.
- One left shift gives `00001010b = 10d` — doubled, as expected.
- Another left shift on that result gives `00010100b = 20d` — doubled again.

#### The SAL Instruction

`SHL` can be used to multiply an operand by any power of 2 you like (shift by 1 to multiply by 2, by 2 to multiply by 4, by 3 to multiply by 8, and so on). However, when your *intent* is specifically arithmetic multiplication (rather than generic bit manipulation), the book introduces an alternate mnemonic: **SAL** (**s**hift **a**rithmetic **l**eft).

The important detail: **SHL and SAL are not two different mechanisms — they generate the exact same machine code.** They're simply two different names/spellings for the identical instruction, offered so that you can choose whichever name better communicates your *intent* to a human reader. If you're doing pure bit-twiddling, write `SHL`; if you're doing "real" multiplication, write `SAL`. The CPU doesn't care either way.

Left shifts also correctly multiply **negative** numbers by powers of 2. For example, if AX holds `FFFFh` (which represents −1 in two's complement), shifting left 3 times yields `FFF8h`, which represents −8 — exactly `−1 × 8`, as expected.

#### Overflow

Here's an important caveat when you're relying on shifts to do multiplication: **overflow can occur**, exactly as it can with regular multiplication, if the result no longer fits in the register.

- For a **single** left shift, both `CF` and `OF` are trustworthy indicators: `CF` tells you about unsigned overflow, and `OF` tells you about signed overflow, for that one shift.
- For a **multiple** shift (`SHL dest, CL` with CL > 1), the flags become **unreliable**. Why? Because internally, a multi-position shift is really just several single shifts performed back-to-back — and `CF`/`OF` only ever reflect what happened on the *very last* one of those single shifts, not the shift as a whole.

**Concrete example of the trap:** suppose `BL = 80h` and `CL = 2`, and you execute `SHL BL, CL`. Even though both signed and unsigned overflow genuinely occur during this operation, the final flag values end up `CF = OF = 0` — giving you a completely false "no overflow" signal. **Moral:** don't trust CF/OF to detect overflow after a multi-position shift; you'd need to check by other means (or simply reason mathematically ahead of time about whether overflow is possible).

### Example 7.8 — Multiply AX by 8 (assume no overflow)

**Task:** Write code to multiply the value in AX by 8, assuming overflow won't happen.

**Solution:** Since `8 = 2³`, multiplying by 8 requires exactly **3** left shifts:

```asm
MOV CL, 3      ; number of shifts to do
SAL AX, CL     ; multiply by 8
```

(Notice `SAL` is chosen here rather than `SHL`, precisely because the intent is genuine arithmetic multiplication — even though either mnemonic would produce identical machine code.)

---

### 7.2.2 Right Shift Instructions

#### The SHR Instruction

**Format:**
```asm
SHR destination, 1
```

`SHR` (**sh**ift **r**ight) is the mirror image of SHL: every bit slides one position to the *right*.
- A fresh **0** is shifted into the **most significant bit** position (the top, left end).
- The **rightmost bit** (which falls off the bottom edge) gets pushed into **CF**.

**Figure 7.3** shows this graphically for both word and byte operands.

For multi-position shifts:
```asm
SHR destination, CL
```
with CL holding the count `N`, performing `N` single right-shifts in sequence.

The effect on the flags is **the same as for SHL** (same rules: SF/PF/ZF reflect the result, AF undefined, CF = last bit shifted out, OF = 1 if sign changes on the last shift).

### Example 7.9 — SHR DH,CL where DH=8Ah, CL=2

**Task:** DH contains `8Ah`, CL contains `2`. Find DH and CF after `SHR DH,CL`.

**Solution:** `8Ah` in binary is `10001010`. Shift right twice: erase the rightmost 2 bits (they fall into CF, with only the very last one actually being retained there), and prepend two fresh 0 bits on the left.

`10001010` → drop rightmost 2 bits (`10`) → left with `100010` → prepend two 0s on the left → `00100010b = 22h`.

The last bit shifted out was 1, so **CF = 1**.

New DH = `22h`, CF = `1`.

#### The SAR Instruction

**Format:**
```asm
SAR destination, 1
SAR destination, CL
```

`SAR` (**s**hift **a**rithmetic **r**ight) behaves *almost* identically to SHR, with **one crucial difference**: instead of always shifting a fresh 0 into the MSB position, **SAR preserves the original value of the MSB** — if the MSB was 1 (a negative number in two's complement), it stays 1 after every shift; if it was 0, it stays 0. See **Figure 7.4**.

Why does this matter? Because the MSB is the **sign bit** in two's complement representation. If SAR blindly inserted 0s the way SHR does, a negative number would turn positive after just one right shift — which would completely break the "divide by 2" interpretation for negative numbers. By preserving the sign bit, SAR makes sure a negative number *stays* negative as you shift it right.

The effect on flags is the same as for SHR.

#### Division by Right Shift

Just as a left shift doubles a value, it's natural to guess that a right shift **halves** it — and that guess is correct, at least for even numbers.

**For odd numbers**, a right shift halves the value and then **rounds down** ("truncates" or "floors") to the nearest whole number, discarding the remainder. Example: if BL contains `00000101b = 5`, after one right shift, BL contains `00000010b = 2` (which is `5 ÷ 2 = 2.5`, rounded down to 2).

#### Signed and Unsigned Division

This is a critical distinction the book emphasizes: **when using right shifts to divide, you must pick the correct instruction depending on whether your number is signed or unsigned:**

- If the number should be interpreted as **unsigned**, use **SHR**.
- If the number should be interpreted as **signed**, use **SAR** — because only SAR preserves the sign bit correctly.

Using the wrong one will silently produce a wrong answer for negative numbers (it would corrupt the sign), even though it "looks fine" for positive numbers.

### Example 7.10 — Divide unsigned 65143 by 4 using right shifts

**Task:** Use right shifts to divide the *unsigned* number 65143 by 4, putting the quotient in AX.

**Solution:** Dividing by 4 = 2² requires **2** right shifts. Since the dividend is explicitly unsigned, we use **SHR** (not SAR):

```asm
MOV AX, 65143     ; AX has the number
MOV CL, 2         ; CL has number of right shifts
SHR AX, CL        ; divide by 4
```

### Example 7.11 — SAR AL,1 where AL = −15

**Task:** If AL contains −15, what is the decimal value of AL after `SAR AL,1`?

**Solution:** `SAR AL,1` divides the value by 2 and rounds down (toward negative infinity, since it's a signed operation).

Mathematically: −15 ÷ 2 = −7.5, and rounding *down* (not toward zero, but toward negative infinity) gives **−8**.

Checking with the actual bit pattern: −15 in 8-bit two's complement is `11110001b`. Applying SAR (which preserves the sign bit as it shifts) gives `11111000b`, which is indeed **−8** in two's complement.

This example is a good one to sit with for a moment, Riyad — it's a common trip-up point. "Rounding down" for a *negative* number means moving further into negative territory (toward −∞), not toward zero. −7.5 rounds down to −8, not up to −7. SAR always implements this "round toward negative infinity" behavior for negative operands, because of how the bit shifting works mechanically — it's a side effect of the representation, not something SAR is "deciding" to do; it just naturally comes out that way.

---

## 7.3 Rotate Instructions

Shifts permanently discard whichever bit falls off the end. **Rotates never lose any bits** — the bit that would otherwise fall off gets wrapped around and reinserted at the opposite end, like a bit going around a circular track.

### Rotate Left

**Format:**
```asm
ROL destination, 1
ROL destination, CL
```

`ROL` (**ro**tate **l**eft) shifts every bit one position to the left, same as SHL — but instead of a 0 being shifted in at the right, the **MSB itself wraps around** and becomes the new LSB. `CF` also receives a copy of that same bit that got rotated out of the MSB position. You can visualize the bits of the destination as forming a circle, with the least significant bit sitting right after the most significant bit in that circle — see **Figure 7.5**.

### Rotate Right

**Format:**
```asm
ROR destination, 1
ROR destination, CL
```

`ROR` (**ro**tate **r**ight) is the mirror image: bits move right, and the bit that falls off the rightmost position wraps around into the MSB position (and also into CF). See **Figure 7.6**.

#### Using CF to Inspect Bits Without Modifying the Data

Because `ROL` and `ROR` always copy the bit that gets rotated out into `CF`, they give you a clean way to **peek at each bit of an operand one at a time**, in order — all *without permanently changing the operand's contents* (as long as you rotate it the full 8 or 16 times, it ends up back exactly where it started).

### Example 7.12 — Count the number of 1 bits in BX using ROL

**Task:** Use `ROL` to count how many bits in BX are set to 1, without changing BX's final value. Store the count in AX.

**Solution:**
```asm
        XOR AX, AX      ; AX counts bits (starts at 0)
        MOV CX, 16      ; loop counter — 16 bits to check
TOP:
        ROL BX, 1       ; CF = bit rotated out (was BX's MSB)
        JNC NEXT        ; if that bit was 0, skip the increment
        INC AX          ; it was a 1 bit — increment the total
NEXT:
        LOOP TOP        ; loop until done (CX-- , jump if CX≠0)
```

**Walking through the logic:** Each pass through the loop, `ROL BX, 1` shifts BX's bits one position left and moves whatever fell out of the MSB into CF (and also wraps it around into BX's own new LSB — remember, ROL never loses data). We then check CF: if it's 0 (`JNC` = "**J**ump if **N**o **C**arry", i.e., jump if CF = 0), that particular bit wasn't a 1, so we skip straight to `NEXT` without incrementing. If CF is 1, execution falls through to `INC AX`, tallying one more 1-bit.

After exactly 16 rotations, BX has cycled all the way around and is back to its **original value** — we've inspected every bit without permanently disturbing the register — and AX holds the total count of 1 bits.

This exact rotate-and-check-CF technique is reused in Section 7.4 to implement binary output.

### Rotate Carry Left

**Format:**
```asm
RCL destination, 1
RCL destination, CL
```

`RCL` (**r**otate through **c**arry **l**eft) works almost like ROL, with one key difference: **CF is folded into the circle of bits being rotated.** The MSB shifts out into CF (exactly like ROL), but what shifts *in* at the LSB position is not a fresh copy of that same bit — it's whatever value **CF held from before this instruction ran**. See **Figure 7.7**. In other words, the "circle" being rotated is now (N+1) bits long for an N-bit operand: it includes CF as an extra bit in the loop.

### Rotate Carry Right

**Format:**
```asm
RCR destination, 1
RCR destination, CL
```

`RCR` (**r**otate through **c**arry **r**ight) is the mirror image of RCL: bits move right, the LSB shifts into CF, and the *previous* value of CF shifts into the MSB position. See **Figure 7.8**.

### Example 7.13 — RCR DH,CL where DH=8Ah, CF=1, CL=3

**Task:** DH contains `8Ah`, CF starts at 1, and CL contains 3. Find DH and CF after `RCR DH,CL` executes.

**Solution:** Since RCR folds CF into the rotation circle, we need to trace it one single-rotation-step at a time — you can't just "erase and reattach" bits the way you could for plain SHR, because CF is now part of the circulating pattern. The book walks through it step by step:

| Step | CF | DH |
|---|---|---|
| initial values | 1 | `10001010` |
| after 1st right rotation | 0 | `11000101` |
| after 2nd right rotation | 1 | `01100010` |
| after 3rd right rotation | 0 | `10110001 = B1h` |

**How each step works:** on every single right rotation, the current rightmost bit of DH moves into CF, and simultaneously the *old* value of CF (from before that step) moves into DH's new leftmost (MSB) position.

- **Step 1:** DH's rightmost bit is 0 → new CF = 0. Old CF was 1 → that 1 becomes DH's new MSB. Remaining bits of `10001010` shift right by one: drop the trailing 0, shift the rest right → `1000101` becomes the lower 7 bits, with the incoming 1 as the new MSB → `11000101`.
- **Step 2:** DH's rightmost bit is 1 → new CF = 1. Old CF (0, from step 1) becomes the new MSB → `01100010`.
- **Step 3:** DH's rightmost bit is 0 → new CF = 0. Old CF (1, from step 2) becomes the new MSB → `10110001b = B1h`.

Final answer: **DH = B1h, CF = 0**.

#### Effect of the Rotate Instructions on the Flags

For all four rotate instructions (ROL, ROR, RCL, RCR):
- `SF`, `PF`, `ZF` reflect the result
- `AF` is undefined
- `CF` = last bit shifted out (during the final single-rotation step, if N > 1)
- `OF` = 1 if the result changes sign on the *last* rotation

*(This mirrors exactly the same "only the last step counts for multi-position operations" caveat you saw for shifts back in 7.2.1 — the same overflow-flag unreliability applies here for multi-position rotates.)*

#### An Application: Reversing a Bit Pattern

Here's a nice combined use of shift *and* rotate together. Suppose AL contains `11011100`, and we want to produce its exact mirror image: `00111011` (first bit becomes last, last becomes first, and so on).

**The trick:** use `SHL` to push AL's bits out its left end one at a time (into CF), and immediately use `RCR` to rotate that same bit into the left end of *another* register (say, BL). Doing this exactly 8 times (for a byte) walks every one of AL's original bits, in order from MSB to LSB, straight into BL's positions from MSB to LSB too — but because BL is filling up from the left while emptying AL from the left, the overall effect ends up mirrored. Once all 8 bits have moved across, BL holds the reversed pattern, and we copy it back into AL.

```asm
        MOV CX, 8       ; number of operations to do
REVERSE:
        SHL AL, 1       ; get a bit into CF
        RCR BL, 1       ; rotate it into BL
        LOOP REVERSE    ; loop until done
        MOV AL, BL      ; AL gets the reversed pattern
```

**Why RCR specifically, and not ROR?** Because plain `ROR BL,1` would just rotate BL's *own* existing bits around in a circle — it has no way to bring in a *new* bit from outside (like the one sitting in CF from AL). `RCR`, by contrast, explicitly folds CF into the rotation — so the bit that SHL just pushed out of AL and into CF gets pulled into BL's own bit-circle as the new MSB. That's exactly the mechanism needed to transplant bits from AL into BL one at a time.

---

## 7.4 Binary and Hex I/O

This section is the practical payoff of the whole chapter: using shifts and rotates (plus logic instructions) to actually **read numbers typed in binary or hex format from the keyboard**, and **print numbers back out in binary or hex format** to the screen. Until now, all of our keyboard/screen I/O has worked one *character* at a time — this section shows how to bridge the gap between "a string of typed characters like `1`, `1`, `0`" and "an actual numeric value stored in a register."

### Binary Input

**Setup:** assume the user types a string of `0` and `1` characters at the keyboard, followed by pressing Enter (which sends a carriage return, `0Dh`). We need to convert that character string into an actual binary *value* stored in a register (BX, in the book's example), one character at a time as it's typed.

**Algorithm for Binary Input:**
```
Clear BX               /* BX will hold binary value */
Input a character      /* '0' or '1' */
WHILE character <> CR DO
  Convert character to binary value
  Left shift BX
  Insert value into lsb of BX
  Input a character
END_WHILE
```

**Why left-shift BX before inserting each new digit?** Think about how you build up a number digit-by-digit when *you* read it: each new digit you read is the *least significant* one so far, and everything you've already accumulated needs to shift one place to make room for it. This is exactly the same principle as the "shift left = multiply by the base" idea from section 7.2.1 — shifting BX left by one position (multiplying the accumulated value by 2, since we're in binary) makes room in the LSB position for the newly-read bit.

**Demonstration, for typed input `110`:**

```
Clear BX
  BX = 0000 0000 0000 0000
Input character '1', convert to 1
Left shift BX
  BX = 0000 0000 0000 0000
Insert value into lsb
  BX = 0000 0000 0000 0001
Input character '1', convert to 1
Left shift BX
  BX = 0000 0000 0000 0010
Insert value into lsb
  BX = 0000 0000 0000 0011
Input character '0', convert to 0
Left shift BX
  BX = 0000 0000 0000 0110
Insert value into lsb
  BX = 0000 0000 0000 0110
BX contains 110b.
```

Notice how each newly typed digit gets tucked into the rightmost position, after everything previously accumulated has been bumped one place to the left to make room — exactly like how you'd write digits on paper from left to right, except the algorithm processes them one at a time as they arrive.

**Assumptions this algorithm makes:** (1) every input character is either `'0'`, `'1'`, or the carriage return; (2) at most 16 binary digits get typed (since BX is a 16-bit register and can't hold more).

**Turning the algorithm into real 8086 assembly:**

As each new digit is typed, the previously accumulated bits in BX need to be shifted left to make room, and then an `OR` operation inserts the new bit into the now-empty LSB slot (recall from 7.1.1: OR with a mask of 1 *sets* a bit, and with a mask of 0 *preserves* it — here, the "mask" is just whatever single bit value the new digit converted to).

```asm
        XOR BX, BX          ; clear BX
        MOV AH, 1           ; input char function
        INT 21H             ; read a character
WHILE_:
        CMP AL, 0DH         ; CR?
        JE  END_WHILE       ; yes, done
        AND AL, 0FH         ; no, convert to binary value
        SHL BX, 1           ; make room for new value
        OR  BL, AL          ; put value into BX
        INT 21H             ; read a character
        JMP WHILE_          ; loop back
END_WHILE:
```

**Walking through each line:**
- `XOR BX, BX` — clears BX to start with a clean slate (the "clear a register" trick from 7.1.1).
- `MOV AH, 1` / `INT 21H` — reads one character from the keyboard into AL (the standard DOS "read character" function you know from Chapter 4).
- `CMP AL, 0DH` / `JE END_WHILE` — checks whether the typed character is a carriage return (ASCII `0Dh`); if so, the loop is over.
- `AND AL, 0FH` — since typed digit characters are `'0' = 30h` or `'1' = 31h`, masking off the high nibble (exactly the same trick as "converting an ASCII digit to a number" from 7.1.1) converts the character straight to its numeric value (0 or 1).
- `SHL BX, 1` — shifts the whole accumulated value in BX one position left, opening up the LSB slot for the new bit.
- `OR BL, AL` — since AL now holds either 0 or 1, ORing it into BL's freshly-vacated LSB slot inserts the new bit there (0 preserves the just-shifted-in 0, or 1 sets it to 1) without disturbing any of the other bits.
- `INT 21H` — reads the next character, and the loop (`JMP WHILE_`) repeats.

### Binary Output

The reverse problem: given a value sitting in BX, print it out as a string of `0`/`1` characters on the screen. This section gives only the **algorithm** — actually writing the full assembly code is left to Riyad as an exercise, following the same pattern as the input code above.

**Algorithm for Binary Output:**
```
FOR 16 times DO
  Rotate left BX     /* BX holds output value,
                         put msb into CF */
  IF CF = 1
   THEN
    output '1'
   ELSE
    output '0'
  END_IF
END_FOR
```

**The key idea:** this reuses exactly the "peek at bits via ROL and CF" technique from Example 7.12. Rotating BX left 16 times, one bit at a time, walks through every bit from the MSB down to the LSB, exposing each one briefly in CF so it can be checked and printed — and because it's a *rotate* (not a shift), BX ends up back at its **original value** once all 16 rotations are done, exactly the same trick as before.

### Hex Input

Hex input is a bit more involved than binary input, because hex digits can be either numeric characters (`'0'`–`'9'`) or letters (`'A'`–`'F'`), which have *different* relationships between their ASCII codes and their numeric values. The book simplifies things with two assumptions: (1) only **uppercase** letters are accepted, and (2) the user types **at most 4** hex characters (since 4 hex digits = 16 bits = exactly fills BX).

**Algorithm for Hex Input:**
```
Clear BX    /* BX will hold input value */
input hex character
WHILE character <> CR DO
  convert character to binary value
  left shift BX 4 times
  insert value into lower 4 bits of BX
  input a character
END_WHILE
```

**Why shift by 4 this time, instead of 1?** Each hex digit represents exactly **4 bits** (a "nibble"), not just 1 bit the way each binary digit did. So to make room for a whole new hex digit, we need to shift the accumulated value left by 4 bit-positions (equivalent to multiplying by 16, since hex is base 16) — the same underlying "shift left = multiply by the base" idea as before, just with base 16 instead of base 2.

**Demonstration, for typed input `6AB`:**

```
Clear BX
  BX = 0000 0000 0000 0000
Input '6', convert to 0110
Left shift BX 4 times
  BX = 0000 0000 0000 0000
Insert value into lower 4 bits of BX
  BX = 0000 0000 0000 0110
Input 'A', convert to Ah = 1010
Left shift BX 4 times
  BX = 0000 0000 0110 0000
Insert value into lower 4 bits of BX
  BX = 0000 0000 0110 1010
Input 'B', convert to 1011
Left shift BX 4 times
  BX = 0000 0110 1010 0000
Insert value into lower 4 bits of BX
  BX = 0000 0110 1010 1011
BX contains 06ABh.
```

**Converting a character to its hex digit value — the tricky part.** For digit characters `'0'`–`'9'` (ASCII `30h`–`39h`), masking off the high nibble (`AND AL, 0Fh`) works exactly as before. But for letter characters `'A'`–`'F'` (ASCII `41h`–`46h`), a plain nibble-mask *doesn't* give the right numeric value (`'A' AND 0Fh` = `41h AND 0Fh` = `1`, but we need `A` to become `10`) — so letters need a different conversion. The relationship is: `'A' = 41h` should map to the value `10 = 0Ah`, so subtracting `37h` from the ASCII code works (`41h − 37h = 0Ah`, correct!). The code below therefore branches: it checks whether the typed character is a digit or a letter, and applies the appropriate conversion to each.

**Full assembly code:**
```asm
        XOR BX, BX          ; clear BX
        MOV CL, 4           ; counter for 4 shifts
        MOV AH, 1           ; input character function
        INT 21H             ; input a character
WHILE_:
        CMP AL, 0DH         ; CR?
        JE  END_WHILE       ; yes, exit
; convert character to binary value
        CMP AL, 39H         ; a digit?
        JG  LETTER          ; no, a letter
; input is a digit
        AND AL, 0FH         ; convert digit to binary value
        JMP SHIFT           ; go to insert in BX
LETTER:
        SUB AL, 37H         ; convert letter to binary value
SHIFT:
        SHL BX, CL          ; make room for new value
; insert value into BX
        OR  BL, AL          ; put value into low 4 bits of BX
        INT 21H             ; input a character
        JMP WHILE_          ; loop until CR
END_WHILE:
```

**Walking through the branching logic:**
- `CMP AL, 39H` / `JG LETTER` — since `'9' = 39h` is the largest digit character and every letter character (`'A'`–`'F'` = `41h`–`46h`) has a larger ASCII code, comparing against `39h` cleanly distinguishes "digit" from "letter": if AL is greater than `39h` (`JG`, jump if greater — a *signed* comparison, but safe here since all these codes are well within the positive/ASCII range), it must be a letter, so we branch to `LETTER`.
- **Digit path:** `AND AL, 0FH` converts the digit character directly to its value (same nibble-masking trick as before), then jumps straight to `SHIFT`, skipping the letter-conversion code.
- **Letter path:** `SUB AL, 37H` converts the letter's ASCII code to its correct hex value (as derived above: `41h − 37h = 0Ah` for `'A'`, and so on up through `'F' = 46h − 37h = 0Fh`).
- `SHIFT:` — both paths converge here. `SHL BX, CL` shifts the accumulated value left by 4 (CL was preloaded with 4), and `OR BL, AL` inserts the newly-converted 4-bit value into the now-open low nibble.
- The loop then reads the next character and repeats, exactly as in the binary-input code, until a carriage return ends it.

**Note from the book:** this program does **no validation** — if the user types something that isn't a valid hex digit or a carriage return, the code will silently produce garbage rather than catching the error. (One of the chapter's programming exercises asks you to add exactly this kind of validation.)

### Hex Output

The final piece: given a value in BX, print its hex representation (4 characters) to the screen. BX holds 16 bits, which is exactly four 4-bit nibbles — i.e., four hex digit values. The approach: start from the **leftmost** nibble (the most significant), peel it off, convert it to a printable hex character, print it, then rotate the whole thing so the *next* nibble becomes accessible, and repeat 4 times total.

**Algorithm for Hex Output:**
```
FOR 4 times DO
  Move BH to DL      /* BX holds output value */
  shift DL 4 times to the right
  IF DL < 10
   THEN
    convert to character in '0'..'9'
   ELSE
    convert to character in 'A'..'F'
  END_IF
  output character
  Rotate BX left 4 times
END_FOR
```

**Why move BH to DL, then shift DL right by 4, instead of working with BX directly?** BH holds the *upper byte* of BX, which contains the two nibbles we currently care about extracting (specifically, its own upper nibble is the digit we want *right now*, since we've arranged via rotation for the digit-of-interest to always sit at the very top of BX). Shifting **DL** (not BX) right by 4 isolates that upper nibble of BH down into DL's low nibble, giving us a clean 0–15 value to convert and print — all without disturbing BX itself, which still needs its remaining nibbles for the next iterations.

**Why compare against 10, and why two different conversions?** A nibble value of 0–9 should print as the *digit character* `'0'`–`'9'`, while a nibble value of 10–15 should print as the *letter character* `'A'`–`'F'`. These require different arithmetic (adding `30h` for digits vs. `37h` for letters, mirroring the same asymmetry we saw in hex *input*, just in reverse), so the algorithm branches on whether the nibble is below 10.

**Why rotate BX left 4 (not shift) at the end of each pass?** Because we still need *all four* original nibbles by the time we're done — rotating (rather than shifting) guarantees that after all 4 iterations, BX has cycled all the way around back to its **original value**, with no data lost along the way, exactly the same non-destructive-inspection principle used throughout this chapter.

**Demonstration, for BX = 4CA9h:**

```
BX = 4CA9h = 0100 1100 1010 1001
Move BH to DL
  DL = 0100 1100
Shift DL 4 times to the right
  DL = 0000 0100
Convert to character and output
  DL = 0011 0100 = 34h = '4'
Rotate BX left 4 times
  BX = 1100 1010 1001 0100
Move BH to DL
  DL = 1100 1010
Shift DL 4 times to the right
  DL = 0000 1100
Convert to character and output
  DL = 0100 0011 = 43h = 'C'
Rotate BX left 4 times
  BX = 1010 1001 0100 1100
Move BH to DL
  DL = 1010 1001
Shift DL 4 times to the right
  DL = 0000 1010
Convert to character and output
  DL = 0100 0010 = 42h = 'B'
Rotate BX left 4 times
  BX = 1001 0100 1100 1010
Move BH to DL
  DL = 1001 0100
Shift DL 4 times to the right
  DL = 0000 1001
Convert to character and output
  DL = 0011 1001 = 39h = '9'
Rotate BX 4 times to the left
  BX = 0100 1100 1010 1001 = original contents
```

**Tracing through what's printed, in order:** `'4'`, then `'C'`, then `'B'`, then `'9'` — spelling out `4CB9`... 

Wait — let's double check against the original value `4CA9h`: the printed sequence is `'4'`, `'C'`, `'B'`, `'9'`. But the original was `4CA9h`, and the third printed digit came out as `'B'` where we'd expect `'A'`. Looking at the trace itself: after the second rotation, DL's upper nibble was `1010` = `Ah` = 10 decimal, and the conversion step should give `10 + 30h = 3Ah`... but 10 is ≥ 10, so it should take the **letter** branch instead: `10 + 37h = 41h = 'A'`, not `'B'`. This looks like a small inconsistency in the book's own worked demonstration (likely an OCR/scanning artifact or an original typo in the printed textbook) — mechanically, a nibble value of `1010b = 10 decimal` should convert to the letter `'A'` (since `0Ah` corresponds exactly to hex digit `A`), giving the fully correct output `4CA9` (matching the original BX value, as it should, since hex output should always round-trip back to the original hex digits). I'm flagging this discrepancy for you, Riyad, so you don't get confused if you're cross-checking against the scanned PDF by hand — trust the *algorithm and reasoning*, not that one specific character in the printed demo.

The final line confirms the rotation mechanism worked correctly regardless: after 4 full left-rotations of 4 bits each (16 bits total), BX has cycled completely around and **equals its original contents**, `0100 1100 1010 1001` = `4CA9h` — proving no data was lost in the process.

*(Coding the full hex-output algorithm into real assembly is left to Riyad as an exercise, following the same translation pattern used for hex input above.)*

---

## Summary (from the book)

- The five logic instructions are **AND, OR, NOT, XOR, and TEST**.
- **AND** can be used to **clear** individual bits in the destination (0 in the mask clears, 1 preserves).
- **OR** is useful for **setting** individual bits in the destination (1 in the mask sets, 0 preserves). It can also test the destination for zero (`OR reg, reg`).
- **XOR** can be used to **complement** individual bits in the destination (1 in the mask flips, 0 preserves). It can also zero out the destination (`XOR reg, reg`).
- **NOT** performs the one's complement operation on the destination — flips every bit, no flags affected.
- **TEST** can be used to examine individual bits of the destination without modifying it — e.g., to determine if the destination contains an even or odd number.
- **SAL** and **SHL** shift each destination bit left one place. The MSB goes into CF, and a 0 is shifted into the LSB.
- **SHR** shifts each destination bit right one place. The LSB goes into CF, and a 0 is shifted into the MSB.
- **SAR** operates like SHR, except the value of the MSB (the sign bit) is preserved.
- The shift instructions can do multiplication and division by 2. **SHL/SAL double** the destination's value (unless overflow occurs). **SHR/SAR halve** the destination's value if it's even; if odd, they halve it and round down to the nearest integer. **SHR** is for unsigned arithmetic; **SAR** is for signed arithmetic.
- **ROL** shifts each bit left one position; the MSB rotates around into the LSB. **ROR** does the mirror image — each bit goes right one position, and the LSB rotates into the MSB. For both, CF gets a copy of the last bit rotated out.
- **RCL** and **RCR** operate like ROL and ROR, except that CF is folded into the circle of rotation: a bit rotated out goes into CF, and the *previous* value of CF rotates into the destination.
- Multiple shifts and rotates (by more than 1 position) can be performed by loading the count into **CL** first.
- The shift and rotate instructions are the foundation for doing binary and hex I/O.

---

## Glossary

| Term | Meaning |
|---|---|
| **clear** | Change a bit's value to 0 |
| **complement** | Change a bit from 0 to 1, or from 1 to 0 (flip it) |
| **mask** | A bit pattern used in logical operations to clear, set, or test specific bits in an operand |
| **set** | Change a bit's value to 1 |

## New Instructions Introduced This Chapter

`AND`, `NOT`, `OR`, `RCL`, `RCR`, `ROL`, `ROR`, `SAL`/`SHL`, `SAR`, `SHR`, `TEST`, `XOR`

---

That's the complete Chapter 7, Riyad. This chapter gives you the tools to reach past "numbers" and manipulate the raw *bits* directly — clearing, setting, flipping, shifting, and rotating them at will. It's exactly this kind of bit-level control that makes assembly (and C) uniquely powerful for things like packing flags, fast multiply/divide by powers of 2, and building your own I/O routines from scratch (as you just saw in 7.4). These same shift/rotate/mask techniques will keep reappearing throughout the rest of the course — 7.4's binary/hex I/O patterns in particular are a template you'll likely reuse. Ready for Chapter 8?