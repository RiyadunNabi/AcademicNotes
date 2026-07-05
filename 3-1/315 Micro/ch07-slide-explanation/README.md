No worries, Riyad — this one is based directly on your teacher's lecture slide deck rather than the Yu & Marut textbook PDF, since that's what you've got time for right now. The slides actually cover two things: a **quick review of Chapter 7 material** (flags, logic instructions, shift/rotate instructions — which you already have detailed notes on) and then the **genuinely new material from Chapter 9: multiplication and division instructions** (`MUL`, `IMUL`, `DIV`, `IDIV`, plus the sign-extension helpers `CBW`/`CWD`). I've gone through every slide in order, expanded the terse bullet points into full explanations, double-checked all the worked numeric examples by hand, and flagged two small errors I found in the slide deck itself so you don't get tripped up by them.

---

# Lecture Notes: Arithmetic and Logic Instructions (CSE 315)

---

## Part 1 — Review: The FLAGS Register

This first chunk of slides is a recap of material you already have full notes on (originally Chapter 5), so I'll keep this part brief and focus on making sure the specific worked examples are crystal clear.

### The Six Flags You Need to Know

| Flag | Intel Mnemonic |
|---|---|
| Overflow | OF |
| Sign | SF |
| Zero | ZF |
| Auxiliary Carry | AF |
| Parity | PF |
| Carry | CF |

*(The full FLAGS register also has DF, IF, and TF — direction, interrupt-enable, and trap flags — but those control CPU *behavior* rather than reflecting the result of an arithmetic/logic operation, so they're not the focus here.)*

### Overflow Flag — Two Different Kinds of "Overflow"

This is a point worth being very clear on, because English uses one word ("overflow") for two genuinely different situations depending on whether you're thinking of your numbers as **unsigned** or **signed**.

**Unsigned overflow → CF (Carry Flag).** This happens when the *true* mathematical result needs one more bit than the register has room for — i.e., there's a "carry out" of the most significant bit.

Slide example:
```
  1111 1111 1111 1111
+ 0000 0000 0000 0001
-----------------------
1 0000 0000 0000 0000
```
Here, `FFFFh + 1` should mathematically equal `10000h` — a 17-bit result. Since a 16-bit register can only hold the lower 16 bits (`0000h`), that leftover 17th bit spills out as a carry, and **CF gets set to 1** to record that a carry actually happened. If you were treating these as *unsigned* numbers, this is your signal that the arithmetic overflowed the register's capacity.

**Signed overflow → OF (Overflow Flag).** This happens when you add two numbers that are both interpreted as *signed* (two's complement) values, and the mathematically correct answer simply can't be represented correctly as a signed number in that register width — the result comes out with the *wrong sign*.

Slide example:
```
  0111 1111 1111 1111
+ 0111 1111 1111 1111
-----------------------
  1111 1111 1111 1110 = FFFEh
```
Here, `7FFFh` represents the largest positive 16-bit signed number (+32767). Adding two of them together should give roughly +65534 — but that's far too big to represent as a signed 16-bit number (whose valid positive range tops out at +32767). The actual bit-level result, `FFFEh`, has its MSB set to 1 — meaning the CPU is telling us this is **−2**, a nonsensical negative answer for "positive + positive." Since the result flipped to the *wrong* sign, **OF gets set to 1**.

### The Rule for Recognizing OF and CF

- **CF = 1** if there's a carry out of the MSB during addition. *(Important exception: CF is **not** affected by `INC` or `DEC`, even though those are technically addition/subtraction by 1 — this is a quirk worth remembering.)*
- **OF = 1** if you add two numbers **of the same sign** and the result comes out with a **different sign**. If you add two numbers of *different* signs, overflow is mathematically impossible — there's no way to overflow in that case, since the true sum is guaranteed to fit between the two operands' magnitudes.
- A slightly more mechanical way some textbooks phrase the same rule: **if the carry going *into* the MSB position doesn't match the carry coming *out* of the MSB position, then OF = 1.**

### ZF, SF, and PF — the Simpler Three

- **ZF = 1** if the result is exactly zero.
- **SF = 1** if the MSB of the result is 1 (i.e., the result "looks negative" in two's complement).
- **PF = 1** if the **low byte** of the result has an **even** number of 1 bits (this is called "even parity"). Note it's *specifically the low byte* that gets checked, even for word-sized results.

### Worked Example — Effect of the Flags (Example 5.3 from the book)

**Task:** `SUB AX, BX`, where AX contains `8000h` and BX contains `0001h`.

**Solution:**
```
    8000h
  - 0001h
  ---------
    7FFFh = 0111 1111 1111 1111
```
The result stored in AX is `7FFFh`. Let's check each flag:

- **SF = 0**, because the MSB of `7FFFh` is 0.
- **PF = 1**, because there are 8 (an even number) of 1-bits in the low byte of the result (the low byte is `FFh = 1111 1111`, all eight bits set — 8 is even).
- **ZF = 0**, because the result is nonzero.
- **CF = 0**, because — thinking of this as *unsigned* subtraction — we're subtracting a smaller unsigned number (`0001h`) from a larger one (`8000h`), so there's no borrow needed.

**Now for OF, which needs a bit more care:** in a *signed* sense, `8000h` represents a large negative number, and we're subtracting a small positive number (`0001h`) from it. Subtracting a positive number is mathematically the same as *adding* its negative — so this is really "negative + negative," which should produce an even more negative result. But the actual result, `7FFFh`, has its MSB = 0, meaning it looks **positive** — the wrong sign entirely. Because the result came out with the wrong sign, **OF = 1**.

---

## Part 2 — Review: AND, OR, XOR, NOT, and TEST

Again, a recap of material you already have detailed Chapter 7 notes on — here's the condensed version matching the slide's presentation.

### AND, OR, XOR

**Syntax:**
```asm
AND destination, source
OR  destination, source
XOR destination, source
```
The restrictions on `destination` and `source` are exactly the same as for `ADD` or `SUB` (register or memory for destination; register, memory, or constant for source; no memory-to-memory).

**Effect on flags:**
- SF, ZF, PF reflect the result
- AF is undefined
- CF, OF = 0

**Examples from the slide, with what each accomplishes:**

| Instruction | Purpose |
|---|---|
| `XOR AX, AX` | Clearing a register (zeroing it out) |
| `OR AL, 81h` | Set the MSB and LSB while preserving other bits (`81h = 1000 0001` — 1s exactly where we want to force bits on) |
| `AND AL, 7Fh` | Clear the sign bit (`7Fh = 0111 1111` — a 0 at bit 7 clears it, 1s elsewhere preserve everything else) |

**Another very practical AND example — converting an ASCII digit character to its numeric value:**
```asm
AND AL, 0Fh
```
ASCII digit characters `'0'` through `'9'` all sit at hex values `30h` through `39h`. In binary, that's always the fixed pattern `0011` in the high nibble, followed by the digit's actual value in the low nibble. Masking with `0Fh = 0000 1111` clears that high nibble entirely and keeps the low nibble untouched — which is exactly the numeric digit value. A few concrete cases:

| Character | ASCII (Hex) | Binary | AND `0Fh` | Result |
|---|---|---|---|---|
| `'0'` | 30h | `0011 0000` | `0000 1111` | `00h` (0) |
| `'5'` | 35h | `0011 0101` | `0000 1111` | `05h` (5) |
| `'9'` | 39h | `0011 1001` | `0000 1111` | `09h` (9) |

*(Note this trick is specifically for single digit characters. If you ever need to go the other way — convert a numeric value back into its displayable ASCII character — you'd do the opposite: `OR AL, 30h`, which sets that same high nibble back to `0011` while leaving the digit value in the low nibble untouched.)*

### NOT Instruction

**Format:**
```asm
NOT destination
```
Works on a **single operand** and performs the **one's complement** operation — flips every bit (0↔1). Has no effect on the flags.

### TEST Instruction

**Format:**
```asm
TEST destination, source
```
Performs the exact same internal computation as `AND`, **except it doesn't write the result back into the destination** — the destination is left completely unchanged. TEST's only job is to set/reset the flags, so you can act on them afterward (usually via a conditional jump). This is why it's typically used for flow control.

**Example — testing whether a number is even or odd:**
```asm
TEST AL, 1
```

| Number type | Last binary bit | `TEST AL,1` result | Zero Flag (ZF) | Next jump instruction |
|---|---|---|---|---|
| Even (e.g., 6) | ends in 0 | `00000000` (zero) | ZF = 1 | `JZ` (jump if zero) |
| Odd (e.g., 7) | ends in 1 | `00000001` (not zero) | ZF = 0 | `JNZ` (jump if not zero) |

The logic: `TEST AL,1` computes `AL AND 1` internally (discarding the result). Since the mask `1` only has its LSB set, the (discarded) result is nonzero **only if** AL's own LSB is 1 — meaning AL is odd. If AL is even (LSB = 0), the AND result is entirely 0, so ZF gets set to 1.

---

## Part 3 — Review: Shift and Rotate Instructions (with an important warning about multiplication)

### General Format and Flag Effects

**Two possible formats:**
```asm
Opcode destination, 1
Opcode destination, CL
```

**Effect on flags:**
- SF, ZF, PF reflect the result
- AF is undefined
- **CF's value changes depending on which shift/rotate instruction you use** (i.e., which end the bit falls off, and whether it wraps around)
- **OF = 1 if the result changes sign on the *last* shift/rotation** (this "last one only" caveat matters a lot — see below)

### SHL and SAL Instructions

- Shift the bits in the destination **to the left**.
- The **MSB is shifted into CF**.
- A **0 is shifted into the LSB**.
- **SAL and SHL generate the exact same machine code** — they're just two spellings of the identical instruction, chosen depending on whether you want to signal "generic bit manipulation" (SHL) or "arithmetic multiplication" (SAL) to a human reader.

**Worked example — `SHL DH, CL` where DH = `8AH`, CL = `3`:**

| Step | Operation | Binary value of DH | Hex | CF |
|---|---|---|---|---|
| Initial | start value before shifting | `1000 1010` | `8AH` | — |
| Shift 1 | shift left, insert 0 at LSB | `0001 0100` | `14H` | 1 |
| Shift 2 | shift left, insert 0 at LSB | `0010 1000` | `28H` | 0 |
| Shift 3 | final shift left, insert 0 at LSB | `0101 0000` | `50H` | 0 |

Final answer: **DH = 50H, CF = 0.** *(This matches Example 7.7 from the textbook exactly, if you want to cross-check.)*

**Two more worked examples from the slides**, showing how the *same* starting bit pattern behaves differently depending on register width:

- **If DH = 8AH, CL = 3** → `SHL DH, CL` gives **DH = 50H, CF = 0, OF = 0**.
- **If DX = 8AH, CX = 3** → `SHL DX, CX` gives **DX = 450H, CF = 0, OF = 0**.

**Why the different final values, even though both started from the "same" `8AH`?** Because DH is only 8 bits wide, so `8AH` fills the *entire* register — any bits shifted left immediately fall off the top edge and are gone (only surviving briefly in CF). DX, on the other hand, is 16 bits wide, and `8AH` as a 16-bit value is really `008AH` — meaning there's a whole empty upper byte of "room" for the shifted bits to move into safely, without falling off the register at all. This is exactly why DX ends up holding `0450h` (the bits that got shifted "into" the upper byte are still fully present) while DH only manages to hold `50h` (having permanently lost its top 3 bits along the way, since there's nowhere left inside DH for them to go).

### Multiplication Using SHL/SAL — and Why You Can't Always Trust the Flags

**The core idea (from Chapter 7):** a left shift on a binary number multiplies it by 2.

**A valid case — no overflow:**
If DH = `2H`, CL = `1` → `SHL DH, CL` gives **DH = 4H, CF = 0, OF = 0**. This is just `2 × 2¹ = 4`, and since the true mathematical answer fits comfortably in 8 bits, the flags correctly show no overflow.

**An invalid case — genuine overflow, but the flags lie to you:**
If DH = `80H`, CL = `2` → `SHL DH, CL` gives **DH = 00H, CF = 0, OF = 0**.

Let's check the actual math: `80h = 128` decimal, and we're multiplying by `2² = 4`, so the *true* answer should be `128 × 4 = 512`. But 512 needs 9 bits to represent, and DH only has 8 — so this is a genuine, real overflow. And yet, **both CF and OF report 0, as if nothing went wrong at all.**

**Why does this happen?** Because a multi-position shift (`CL = 2` here) is really executed internally as a *series* of individual single-bit shifts, one after another — and **CF and OF only ever get updated to reflect the result of the very last one of those single shifts**, throwing away any information about what happened on the earlier ones. Walking through it: shift 1 takes `1000 0000` → `0000 0000`, with CF picking up the bit that fell out (which was 1) and OF becoming 1 (since the sign flipped from negative-looking to positive-looking on that shift). But then shift 2 takes that *already-zero* value and shifts it again: `0000 0000` → `0000 0000` — nothing interesting happens on this second shift, the bit that falls out this time is just a 0, so the *final* CF = 0 and OF = 0, completely erasing the evidence that something went wrong on the first shift.

**The takeaway, worth committing to memory: CF and OF are only reliable overflow indicators for a *single*-position shift. The moment you shift by more than one position at a time (`CL > 1`), you can't trust either flag to detect overflow — you'd need to reason about it mathematically ahead of time instead.**

### SHR and SAR Instructions

- Shift the bits in the destination **to the right**.
- The **LSB is shifted into CF**.
- In the case of **SAR** specifically, the **MSB retains its original value** (rather than having a fresh 0 shifted in, the way SHR does).

**Why does SAR preserve the MSB?** Because the MSB is the *sign bit* in two's complement. If SAR blindly inserted 0s the way SHR does, a negative number would flip to positive after just one shift — completely breaking the "divide by 2" interpretation for negative values. By keeping the sign bit fixed, SAR guarantees a negative number *stays* negative as you shift it.

**This gives you the rule for which one to use:**
- **SHR should be used for unsigned interpretation** — it doesn't preserve sign, which is exactly correct for a number that has no sign to begin with.
- **SAR should be used for signed interpretation** — it preserves sign, which is essential for negative numbers to divide correctly.

**Worked example — AL = −15, CL = 1:**

`SHR AL, CL`: −15 in 8-bit two's complement is `1111 0001` (since 256 − 15 = 241 = `F1h`). SHR shifts everything right and inserts a fresh **0** at the MSB: result = `0111 1000 = 78h = 120` decimal. CF gets the bit that fell off the bottom, which was the original LSB = 1, so **CF = 1**. And since the sign flipped from negative (MSB was 1) to positive (MSB is now 0), **OF = 1** as well. Final: **AL = 120H, CF = 1, OF = 1.**

`SAR AL, CL`: same starting bit pattern `1111 0001`, but SAR keeps the MSB fixed at its original value (1, since the number is negative) instead of inserting a fresh 0: result = `1111 1000 = F8h = −8` decimal (matching Example 7.11 from the textbook). CF again picks up the bit that fell off the bottom (the original LSB, which was 1), so **CF = 1**.

**A correction worth flagging here, Riyad:** the slide lists `OF = 1` for this SAR case too — but that doesn't actually check out. Since SAR is *specifically designed* to preserve the sign bit, the sign can never change as a result of a SAR operation — the MSB before and after are guaranteed identical by construction. That means **OF should always be 0 for SAR**, and specifically for this example, OF = 0 (not 1). I suspect this is simply a copy-paste slip in the slide deck (the line right above it, for SHR, correctly has OF = 1, and it looks like that value just got duplicated down onto the SAR line beneath it). So to be precise: **`SAR AL, CL` here gives AL = −8, CF = 1, OF = 0.**

---
