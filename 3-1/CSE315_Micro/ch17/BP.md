BP = **Base Pointer**, a 16-bit register on the 8086.

It's a general-purpose register like AX/BX/CX/DX, but it has one special hardware behavior: **any time you use `[BP+...]` in an address, the CPU automatically assumes that address is in the SS (stack) segment** — not DS like BX would default to.

That's the whole reason it exists in this context: you can't do `[SP+4]` (illegal addressing mode), so the trick is:

```asm
PUSH BP        ; save caller's BP first (you're about to overwrite it)
MOV  BP, SP    ; now BP = current SP, "anchored" at the stack top
```

After this, `SP` can keep moving (as your procedure pushes/pops other stuff internally), but `BP` stays frozen at that one spot — giving you a stable landmark to reach into the stack and grab parameters (`[BP+4]`, `[BP+6]`, etc.) reliably, no matter what else happens to SP later in the procedure.

That's it — it's not magic, it's just "a register that's allowed to address the stack segment, and that you manually park at a fixed spot."