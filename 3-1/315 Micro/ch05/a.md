Great catch, Riyad. The explanation in your notes is a bit misleading. Let's analyze it carefully.

---

## We have:

```text
11111111
+00000001
---------
1 00000000
```

The result stored in the register is

```text
00000000
```

Now let's think about **Overflow Flag (OF)**.

---

# What does OF mean?

**OF is only for signed arithmetic.**

It asks:

> **"Did the signed result become impossible to represent?"**

Not

> "Was there a carry?"

That's **CF's** job.

---

## Step 1: Interpret the numbers as signed

We're adding

```text
11111111 = -1
```

and

```text
00000001 = +1
```

So mathematically,

```text
-1 + 1 = 0
```

Can an 8-bit signed integer represent 0?

Yes.

```
00000000 = 0
```

So there is **no overflow**.

Therefore,

```text
OF = 0
```

---

# Another way (the rule)

Overflow occurs only when:

* positive + positive → negative ❌
* negative + negative → positive ❌

because those results don't make sense in signed arithmetic.

Here we have

```text
-1 + +1
```

One negative.

One positive.

Overflow **cannot happen**.

Therefore,

```text
OF = 0
```

---

# Compare with a case where OF = 1

Example:

```text
01111111
+00000001
----------
10000000
```

Signed interpretation:

```text
127 + 1 = -128
```

That's impossible mathematically.

The correct answer is 128, but an 8-bit signed integer cannot represent 128.

So

```text
OF = 1
```

---

Another example:

```text
10000000
+11111111
----------
01111111
```

Signed:

```text
-128 + (-1) = 127
```

Impossible.

Should be -129, but that doesn't fit.

Again,

```text
OF = 1
```

---

# Why doesn't the carry matter?

Your example produced a carry:

```text
11111111
+00000001
---------
1 00000000
^
carry
```

That carry is relevant for **unsigned** arithmetic, not signed arithmetic.

Since the signed result is

```text
-1 + 1 = 0
```

which is perfectly valid,

```text
OF = 0
```

even though there was a carry out.

---

### One clarification about your notes

The sentence:

> "we added FFh (−1 signed) and 1h (+1 signed), which are different signs → signed overflow is impossible"

is correct **for this example**, but it's not the *definition* of overflow. The deeper reason is:

> The true mathematical result (**0**) is representable in an 8-bit signed integer, so the CPU sets **OF = 0**.

The "different signs" rule is just a quick shortcut that tells you the same thing.
