😂 Bro, that's actually a very good approach **while learning**.

-----------------

## Even easier mental rule

Don't memorize flags.

Just remember the **opposite pairs**.

| Pseudocode    | THEN jump | ELSE jump |
| ------------- | --------- | --------- |
| `==`          | `JE`      | `JNE`     |
| `!=`          | `JNE`     | `JE`      |
| `<` (signed)  | `JL`      | `JGE`     |
| `<=` (signed) | `JLE`     | `JG`      |
| `>` (signed)  | `JG`      | `JLE`     |
| `>=` (signed) | `JGE`     | `JL`      |

Unsigned:

| Pseudocode | THEN jump | ELSE jump |
| ---------- | --------- | --------- |
| `<`        | `JB`      | `JAE`     |
| `<=`       | `JBE`     | `JA`      |
| `>`        | `JA`      | `JBE`     |
| `>=`       | `JAE`     | `JB`      |

-----------
So you can use this mental template.

### Pseudocode

```text
IF condition
THEN
    ...
ELSE
    ...
END_IF
```

Then, during an exam, you can mechanically write:
### Your assembly template

```asm
CMP ...
J<condition> THEN_
J<opposite_condition> ELSE_

THEN_:
    ; then statements
    JMP END_IF

ELSE_:
    ; else statements

END_IF:
```
-----------------




You're not writing production code. You're trying to **translate pseudocode into assembly quickly during an exam**.





For example:

```text
IF AL <= BL
THEN
    display AL
ELSE
    display BL
END_IF
```

becomes

```asm
MOV AH, 2

CMP AL, BL
JLE THEN_
JG  ELSE_

THEN_:
    MOV DL, AL
    JMP END_IF

ELSE_:
    MOV DL, BL

END_IF:
    INT 21h
```

This is **completely valid**.

---


It's one extra jump, but it's **very easy to derive under time pressure**. Once you're comfortable with assembly, you can optimize it by removing the redundant jump if you want. For now, prioritizing a pattern that's easy to remember is a sensible strategy.
