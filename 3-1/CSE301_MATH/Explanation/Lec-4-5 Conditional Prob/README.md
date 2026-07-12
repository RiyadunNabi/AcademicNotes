Welcome back to class! These next three pages move us from basic combinations into one of the most powerful concepts in all of statistics: **Conditional Probability** and **Bayes' Rule**.

This is all about how we should update our beliefs or expectations when we get *new information*. Let's break down the core ideas in your notes.

### 1. The Newton-Pepys Problem (A Historical Warm-up)

On the first page, your notes mention a famous problem from 1693. Samuel Pepys (a famous diarist) wrote to Isaac Newton asking which of these three scenarios is most likely when rolling fair dice:

* (A) Rolling 6 dice and getting at least one 6.
* (B) Rolling 12 dice and getting at least two 6s.
* (C) Rolling 18 dice and getting at least three 6s.

A lot of people guess these are all equally likely because the ratio is the same (1 in 6). But Newton correctly calculated that **(A) is the most likely**. Your notes show the math: the probability drops from about 66.5% for (A), to 61.9% for (B), to 59.7% for (C). It’s a great example of why we can't always rely on gut intuition in probability!

---

### 2. The Core Concept: Conditional Probability

The big question these notes ask is: *"How should we update probability/uncertainty when new information is revealed to us?"*

We write this mathematically as **$P(A\vert{}B)$**, which is read as "The probability of event A happening, *given* that we know event B has already happened."

The foundational formula (from the second page) is:


$$P(A\vert{}B) = \frac{P(A \cap B)}{P(B)}$$


*Think of it this way:* Since we know $B$ happened, our entire "universe" shrinks down to only the times $B$ happens. Out of those times, how often does $A$ *also* happen (the intersection, $A \cap B$)?

### 3. Bayes' Rule & The Law of Total Probability

Page 2 lists three theorems, but **Theorem 3 (Bayes' Rule)** is the crown jewel:


$$P(A\vert{}B) = \frac{P(A)P(B\vert{}A)}{P(B)}$$

Bayes' Rule is a mathematical machine for "reversing" conditionals. It lets you figure out $P(A\vert{}B)$ if you already know $P(B\vert{}A)$.

To find the bottom part, $P(B)$, your notes show the **Law of Total Probability**. This just means that to find the total probability of event $B$, you add up all the different ways $B$ could possibly happen across all different scenarios ($A_1, A_2, \dots, A_n$).

---

### 4. The Mind-Blowing Example: Disease Testing

The third page has a classic example that trips up almost everyone—including doctors! Let's walk through it because it perfectly demonstrates why Bayes' Rule is so important.

**The Setup:**

* A rare disease affects 1% of the population. So, the probability you have the disease before taking a test is $P(D) = 0.01$.
* You take a test that is 95% accurate. This means if you are sick, it correctly tests positive 95% of the time: $P(T\vert{}D) = 0.95$.
* It also means if you are healthy, it incorrectly tests positive 5% of the time (a false positive): $P(T\vert{}D^c) = 0.05$.

**The Question:** You test positive. What is the actual probability that you have the disease ($P(D\vert{}T)$)?

Most people intuitively guess 95%. Let's look at the calculation in your notes using Bayes' Rule:


$$P(D\vert{}T) = \frac{P(D) \cdot P(T\vert{}D)}{P(D) \cdot P(T\vert{}D) + P(D^c) \cdot P(T\vert{}D^c)}$$

$$P(D\vert{}T) = \frac{0.01 \cdot 0.95}{(0.01 \cdot 0.95) + (0.99 \cdot 0.05)} \approx 0.16$$

**The Result:** Even with a positive result on a 95% accurate test, you only have a **16% chance** of actually having the disease!

**Why does this happen?** Because the disease is so rare (1%), the massive number of healthy people (99%) taking the test generates a huge amount of *false positives*. Those false positives drown out the tiny number of true positives.

To help this click, I've set up an interactive sandbox below where you can adjust how rare a disease is and how accurate a test is to see exactly how the math plays out visually.

How do you feel about the transition from basic counting formulas to this kind of logical updating? Does the medical test example make sense, or would you like to break down the "playing card" example right above it in the notes?