---
How many bytes can be addressed?
Depends on how many bits the processor uses for addresses.
The 8086 uses 20-bit addresses → 2²⁰ = 1,048,576 = 1 megabyte (1 MB) of memory. explain
---

---
Sure, Riyad. This is actually a very important concept in computer architecture.

### Step 1: What is an address?

Think of memory like a huge apartment building.

* Each **room = 1 byte of memory**
* Each room has a **room number = memory address**

For example:

```
Address      Stores
00000        A
00001        B
00002        C
...
```

The processor uses these addresses to know **where to read or write data**.

---

### Step 2: What does "20-bit address" mean?

A bit can be either **0** or **1**.

If an address has **20 bits**, it looks like this:

```
00000000000000000000
```

or

```
11111111111111111111
```

Since each bit has 2 possibilities (0 or 1),

the total number of different addresses is

[
2^{20}
]

because there are 20 positions, each with 2 choices.

---

### Step 3: Calculate (2^{20})

```
2^10 = 1024
2^20 = 1024 × 1024
     = 1,048,576
```

So there are

```
1,048,576 different addresses.
```

---

### Step 4: Why does this equal 1 MB?

In the 8086,

* **Each address points to exactly 1 byte.**

So

```
1 address = 1 byte
```

Therefore,

```
1,048,576 addresses
=
1,048,576 bytes
```

Since

```
1 MB = 1024 × 1024 bytes
     = 1,048,576 bytes
```

the maximum memory is

```
1 MB
```

---

### Visualization

```
20-bit Address Bus

00000000000000000000   --> Byte 0
00000000000000000001   --> Byte 1
00000000000000000010   --> Byte 2
...
11111111111111111111   --> Byte 1,048,575
```

Total bytes:

```
0
1
2
...
1,048,575
```

Number of bytes:

```
1,048,576 bytes
= 1 MB
```

---

### General Rule

If a processor has **n address bits**, then it can address

[
2^n \text{ bytes}
]

(assuming each address refers to one byte).

Examples:

| Address Bits   |  Number of Addresses |                                   Memory Size |
| -------------- | -------------------: | --------------------------------------------: |
| 8 bits         |          (2^8 = 256) |                                     256 bytes |
| 16 bits        |    (2^{16} = 65,536) |                                         64 KB |
| 20 bits (8086) | (2^{20} = 1,048,576) |                                          1 MB |
| 32 bits        |             (2^{32}) |                                          4 GB |
| 64 bits        |             (2^{64}) | About 16 exabytes (theoretical address space) |

### One-line summary

> **The 8086 has a 20-bit address bus, so it can generate (2^{20}) unique addresses. Since each address points to 1 byte, it can access (2^{20} = 1,048,576) bytes = 1 MB of memory.**
