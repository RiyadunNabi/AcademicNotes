# Arrays and Addressing Modes (CSE 315)

---

## Part 1 — One Dimensional Arrays

An array is just a list of elements, stored one after another in memory, each one right next to the last.

### Declaring arrays

```asm
MSG DB 'abcde'              ; 5 bytes: a,b,c,d,e
W   DW 10, 20, 30, 40, 50, 60   ; 6 words (2 bytes each)
```

- `DB` = define byte(s). `DW` = define word(s).
- `MSG` is a **5-byte** array (1 byte per character).
- `W` is a **6-word** array = **12 bytes** total (2 bytes per integer).

### The base address

The address of the array's *name* (its first element) is called the **base address**. Every other element sits at some fixed offset from that base address.

**Example — array W in memory** (assume W starts at offset `0200h`):

| Offset address | Symbolic address | Decimal content |
|---|---|---|
| 0200h | W | 10 |
| 0202h | W+2h | 20 |
| 0204h | W+4h | 30 |
| 0206h | W+6h | 40 |
| 0208h | W+8h | 50 |
| 020Ah | W+Ah | 60 |

**Point to remember:** each word = 2 bytes, so consecutive elements are 2 bytes apart in offset (`0200h → 0202h → 0204h...`). If this were a byte array (`DB`), elements would be 1 byte apart instead.

### DUP operator — repeating values without typing them all out

```asm
repeat_count DUP (value)
```

**Example 1 — 100 words, all initialized to 0:**
```asm
GAMMA DW 100 DUP (0)
```

**Example 2 — 212 bytes, left uninitialized:**
```asm
DELTA DB 212 DUP (?)
```
(`?` means "don't care / leave garbage" — no specific value is forced.)

**Example 3 — nested DUP:**
```asm
LINE DB 5, 4, 3 DUP (2, 3 DUP (0), 1)
```
is exactly equivalent to typing out:
```asm
LINE DB 5, 4, 2, 0, 0, 0, 1, 2, 0, 0, 0, 1, 2, 0, 0, 0, 1
```

**How to read a nested DUP, step by step:** work from the *inside* out.
- Inner: `3 DUP (0)` → `0, 0, 0`
- So `(2, 3 DUP (0), 1)` → `2, 0, 0, 0, 1`
- Outer: `3 DUP (2, 0, 0, 0, 1)` → that whole group repeated 3 times → `2,0,0,0,1, 2,0,0,0,1, 2,0,0,0,1`
- Full line: `5, 4,` then that repeated group → `5, 4, 2,0,0,0,1, 2,0,0,0,1, 2,0,0,0,1`

**Point to remember:** `N DUP (...)` just means "paste whatever's in the parentheses, N times in a row." Nested DUP is the same trick applied recursively.

---

## Part 2 — Finding the Location of Any Element

**Formula:**
```
Location of A[position] = A + (position − 1) × S
```
where:
- `A` = base address of the array
- `S` = size in bytes of one element (**S=1** for a byte array, **S=2** for a word array)
- `position` = 1-based position you're looking for (1st element, 2nd element, ...)

| Position | Location |
|---|---|
| 1 | A |
| 2 | A + 1×S |
| 3 | A + 2×S |
| ... | ... |
| N | A + (N−1)×S |

**Worked example:** Find the 3rd element of word array `W`.
- S = 2 (word array)
- Location = `W + (3−1)×2 = W + 4`
- Matches the table above: `W+4h` holds `30`, the 3rd value. ✓

**Point to remember:** it's `(position − 1) × S`, **not** `position × S` — because the 1st element sits at offset 0 from the base, not offset S. This "off by one" is the most common mistake here.

---

## Part 3 — Addressing Modes Overview

**Modes you already know:**

1. **Register mode** — operand is a register. `MOV AX, 0` → destination AX is register mode.
2. **Immediate mode** — operand is a constant. Same instruction, `0` is immediate mode.
3. **Direct mode** — operand is a variable (a fixed, named memory address). `ADD X, AX` → X is direct mode, AX is register mode.

**Four new modes covered in this chapter** — all of these let the CPU calculate an address *dynamically* rather than using a fixed name, which is exactly what you need to walk through an array:

1. Register indirect mode
2. Based mode
3. Indexed mode
4. Based indexed mode

---

## Part 4 — Register Indirect Mode

**Format:** `[register]`

The register holds the **offset address** of the operand — not the operand's value itself, but *where to find it*. Think of the register as a pointer.

- Register can be **BX, SI, DI, or BP**.
- If the register is **BX, SI, or DI** → segment comes from **DS**.
- If the register is **BP** → segment comes from **SS**.

**Point to remember:** `[BX]` means "go to the memory address stored in BX, and use *that*." It's the difference between a variable and a pointer to a variable.

### Worked example — sum all 10 elements of an array

```asm
W DW 10, 20, 30, 40, 50, 60, 70, 80, 90, 100
```
```asm
        XOR  AX, AX      ; AX holds sum, start at 0
        LEA  SI, W        ; SI = offset address of W (SI now points at W)
        MOV  CX, 10       ; CX = number of elements to add
ADDNOS:
        ADD  AX, [SI]     ; sum = sum + (value SI is currently pointing at)
        ADD  SI, 2        ; move pointer forward by 2 bytes (1 word) to next element
        LOOP ADDNOS       ; repeat until CX = 0
```

**Walking through it, iteration by iteration:**
- `LEA SI, W` — SI = address of W's first element (10).
- Iteration 1: `ADD AX, [SI]` adds the value *pointed to* by SI (which is 10) → AX = 10. Then `ADD SI, 2` moves SI forward to point at the next word (20).
- Iteration 2: adds 20 → AX = 30. SI moves to point at 30.
- ...continues for all 10 elements...
- Final: AX = 10+20+...+100 = 550.

**Point to remember:** `ADD SI, 2` is what makes this walk through the *whole array* instead of reading the same element 10 times. Forgetting this line is the #1 bug — you'd add the first element 10 times instead of summing all 10 elements. The `2` is there because each element is a **word** (2 bytes); for a byte array you'd use `ADD SI, 1` instead.

---

## Part 5 — Based Mode

**Format (all five of these mean the exact same thing):**
```
[register + displacement]
[displacement + register]
[register] + displacement
displacement + [register]
displacement[register]
```

- Register can be **BX** (base register) or **BP** (base pointer).
- Displacement can be: a variable's offset address (e.g. `A`), a plain constant (e.g. `-2`), or a variable plus/minus a constant (e.g. `A+2`).
- If **BX** is used → segment from **DS**. If **BP** is used → segment from **SS**.

**Point to remember:** Based mode = register indirect mode **plus** a fixed offset added on top. It's how you access an array element when you know *both* a base address (the array name) *and* an offset from a register (e.g., "index number × element size" sitting in a register).

### Worked example — access the 3rd element of array W using BX as an index

Setup: W is a word array, and BX contains **4** (this is the byte-offset for the 3rd element: `(3−1)×2 = 4`, matching the formula from Part 2).

All five of these lines do exactly the same thing — move the 3rd element of W into AX:

```asm
MOV AX, [BX + W]     ; [register + displacement]
MOV AX, [W + BX]     ; [displacement + register]
MOV AX, [BX] + W     ; [register] + displacement
MOV AX, W + [BX]     ; displacement + [register]
MOV AX, W[BX]        ; displacement[register]
```

**Why this gets the 3rd element:** the actual address used is `W + BX = W + 4`, which (per the Part 2 formula) is exactly the location of the 3rd element. If BX had instead contained `0`, this would access the 1st element; if BX contained `2`, the 2nd element; and so on.

**Point to remember:** the *displacement* (W) is the array's fixed base address (known at assembly time), and the *register* (BX) is the variable part — usually holding `index × element_size`, so you can change BX at runtime to walk through different elements without rewriting the instruction.

---

## Part 6 — Indexed Mode

**Format** — identical five variations as Based Mode:
```
[register + displacement]
[displacement + register]
[register] + displacement
displacement + [register]
displacement[register]
```

- Register can be **SI** (source index) or **DI** (destination index).
- Displacement: same options as Based Mode (variable address, constant, or variable ± constant).
- If **SI or DI** is used → segment always comes from **DS** (no SS option here, unlike Based Mode with BP).

**Point to remember:** Indexed mode is mechanically identical to Based mode — same formats, same "base address + running offset" idea — the *only* difference is which registers are allowed (SI/DI here, vs BX/BP for Based mode). This is exactly why SI and DI are literally named "source index" and "destination index" — they're designed for this exact array-walking job (and you already saw SI doing exactly this job back in the register-indirect sum example in Part 4).

**Example, following the same pattern as Based mode:** if SI contains `4` and W is the same word array as before:
```asm
MOV AX, [SI + W]     ; gets the 3rd element of W, same logic as the BX example
```

---

## Part 7 — PTR Operator

**The problem it solves:** an instruction like
```asm
MOV [BX], 1
```
is **not allowed**, because the assembler has no way to know whether you mean to store `1` as a single **byte** or as a full **word** at that memory address — `[BX]` alone doesn't say how big the destination is.

**The fix — PTR lets you explicitly state the size (type-cast the memory access):**
```asm
MOV BYTE PTR [BX], 1    ; store 1 as a single byte
MOV WORD PTR [BX], 1    ; store 1 as a full word (2 bytes)
```

**Point to remember:** `PTR` doesn't change *where* you're writing (still the address in BX) — it only tells the assembler *how much space* to treat that write as covering. Use `BYTE PTR` for byte arrays/data, `WORD PTR` for word arrays/data. Without it, any instruction where the size is ambiguous (bare register-indirect/based/indexed addressing with an immediate source, no other clue about size) will fail to assemble.

---

## Quick-Reference Summary

- Array element location: `A + (position − 1) × S` — S=1 for byte arrays, S=2 for word arrays. Watch the "−1".
- `N DUP (value)` repeats `value` N times; nesting works inside-out.
- **Register indirect** `[reg]`: reg holds the address, walk arrays by incrementing the register (by 1 for bytes, by 2 for words).
- **Based** `[reg + disp]`: reg = BX or BP; disp = array base address (or constant). BX→DS, BP→SS.
- **Indexed** `[reg + disp]`: reg = SI or DI only; same idea as Based, always DS.
- All five written forms of Based/Indexed mode (`[reg+disp]`, `[disp+reg]`, `[reg]+disp`, `disp+[reg]`, `disp[reg]`) are 100% equivalent — pick whichever reads clearest.
- `BYTE PTR` / `WORD PTR`: use whenever the assembler can't tell the size of a memory operand on its own (typically: register-indirect/based/indexed destination + immediate source).

---

**Reference:** Lecture slides, *Arrays and Addressing Modes* — CSE 315.