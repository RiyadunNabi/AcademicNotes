Perfect, Riyad. This section introduces one of the most important ideas in DOS assembly. Let's relate everything to **C functions**.

---

# What is `INT`?

Think of `INT` as **calling a built-in operating system function**.

In C, you write:

```c
printf("Hello");
```

You don't know how `printf()` prints to the screen.

You simply **call** it.

Assembly is similar.

```asm
INT 21h
```

means

> **"DOS, please do something for me."**

---

# But how does DOS know WHAT I want?

Imagine DOS has hundreds of functions.

Like this:

```text
DOS

1 -> Read keyboard
2 -> Print one character
9 -> Print string
4C -> Exit program
...
```

Before calling DOS, you tell it **which function** you want.

Where?

Inside the **AH register**.

For example,

```asm
MOV AH, 1
INT 21h
```

means

> DOS, perform **Function 1**.

---

Another example:

```asm
MOV AH, 2
INT 21h
```

means

> DOS, perform **Function 2**.

---

## C Analogy

Imagine DOS were written in C:

```c
DOS(function_number);
```

Then

```c
DOS(1);
```

means

> Read keyboard.

and

```c
DOS(2);
```

means

> Display one character.

The only difference is that assembly stores the function number in `AH`.

---

# Function 1 — Read one key

```asm
MOV AH,1
INT 21h
```

DOS waits until you press a key.

Suppose you press

```text
A
```

ASCII of A is

```text
41h
```

DOS stores

```text
AL = 41h
```

Now your program knows which key you pressed.

---

### C equivalent

```c
char c = getchar();
```

Exactly the same idea.

---

# What does "echoed" mean?

The book says

> The character is echoed.

Suppose you type

```text
A
```

You immediately see

```text
A
```

appear on the screen.

DOS automatically prints it.

You didn't write any print instruction.

That's called **echo**.

---

# Function 2 — Display one character

Now suppose you want to print a character.

You do

```asm
MOV AH,2
MOV DL,'?'
INT 21h
```

DOS looks inside `DL`.

It finds

```text
?
```

and prints it.

---

### C equivalent

```c
putchar('?');
```

or

```c
printf("?");
```

---

# Why DL?

Because DOS says

> "For Function 2, I'll read the character from DL."

It's simply the agreed convention.

---

# Why does Function 2 change AL?

Suppose you do

```asm
MOV AH,1
INT 21h
```

User presses

```text
A
```

Now

```text
AL = 'A'
```

Then you do

```asm
MOV AH,2
MOV DL,'?'
INT 21h
```

DOS prints

```text
?
```

But while doing that, DOS is allowed to modify some registers—including `AL`.

So after printing,

```text
AL
```

may no longer contain `'A'`.

If you still need `'A'`, save it first:

```asm
MOV BL, AL    ; save the character
```

Later you can use

```asm
MOV AL, BL
```

to get it back.

---

# Control characters

Normally

```asm
MOV DL,'A'
INT 21h
```

prints

```text
A
```

But what if

```asm
MOV DL,0Ah
INT 21h
```

?

DOS sees

```text
0Ah
```

which is the ASCII control code for **Line Feed**.

Instead of drawing a weird symbol, DOS interprets it as a command:

> "Move the cursor down one line."

Similarly,

| Value | Meaning                          |
| ----- | -------------------------------- |
| 07h   | Beep 🔔                          |
| 08h   | Backspace                        |
| 09h   | Tab                              |
| 0Ah   | Move down one line               |
| 0Dh   | Move cursor to beginning of line |

These are called **control characters** because they **control the terminal** instead of displaying a printable symbol.

---

## Think of `INT 21h` like a C library

Imagine DOS had these C functions:

```c
char readKey();
void printChar(char c);
void printString(char *s);
void exitProgram();
```

Assembly doesn't call them by name.

Instead, it does this:

```text
AH = 1   -> readKey()

AH = 2   -> printChar()

AH = 9   -> printString()

AH = 4Ch -> exitProgram()
```

Then one instruction:

```asm
INT 21h
```

acts like a dispatcher:

> "Look at `AH`, figure out which DOS function I want, and execute it."

So you can think of `INT 21h` as a single entry point to many DOS services, with the value in `AH` selecting which service to run.
