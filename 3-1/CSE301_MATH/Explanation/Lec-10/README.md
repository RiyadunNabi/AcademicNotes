Bhai, Lecture 10 is pure gold! This lecture moves from counting successes in a fixed number of trials (Binomial) to **waiting for successes** (Geometric and Negative Binomial), and then shows off the insane power of **Linearity of Expectation** on a famous permutation problem and a classic probability paradox.

Let’s break down every single section from scratch, lesson by lesson, so you master every line of these notes!

---

## Lesson 1: The Geometric Distribution (Waiting for the 1st Success)

When you flip a coin repeatedly until you get your first Head (success), how many **failures** do you get before that first success? That count is the **Geometric distribution**: $X \sim \text{Geom}(p)$.

### 1. The Story & PMF

* **The Story:** Number of independent $\text{Bern}(p)$ failures before the first success.
* **Examples of sequences:**
* `H` $\rightarrow 0$ failures ($k=0$)
* `T H` $\rightarrow 1$ failure ($k=1$)
* `T T H` $\rightarrow 2$ failures ($k=2$)
* `T T T H` $\rightarrow 3$ failures ($k=3$)


* **Probability Mass Function (PMF):**

$$P(X = k) = q^k p, \quad \text{for } k \in \{0, 1, 2, \dots\}$$



*(You must fail $k$ times independently, each with probability $q = 1-p$, and then succeed once with probability $p$.)*

### 2. Sanity Check: Do probabilities sum to 1?

Using the formula for the sum of an infinite geometric series ($\sum_{k=0}^{\infty} q^k = \frac{1}{1-q}$):


$$\sum_{k=0}^{\infty} q^k p = p \sum_{k=0}^{\infty} q^k = p \cdot \frac{1}{1-q} = p \cdot \frac{1}{p} = 1$$

---

### 3. Expected Value $E(X)$: Two Different Proofs!

The notes give **two different ways** to prove that the expected number of failures is $E(X) = \frac{q}{p}$.

#### Proof 1: The Calculus / Series Proof (Left Side)

1. **Write the definition of expectation:**

$$E(X) = \sum_{k=0}^{\infty} k q^k p = p \sum_{k=0}^{\infty} k q^k$$


2. **Use a calculus trick on geometric series (Middle Box):**
* We know: $\sum_{k=0}^{\infty} q^k = \frac{1}{1-q}$
* Take the derivative of both sides with respect to $q$:

$$\sum_{k=0}^{\infty} k q^{k-1} = \frac{1}{(1-q)^2}$$


* Multiply both sides by $q$ to get $k q^k$ inside the sum:

$$\sum_{k=0}^{\infty} k q^k = \frac{q}{(1-q)^2} = \frac{q}{p^2}$$




3. **Substitute back into $E(X)$:**

$$E(X) = p \cdot \frac{q}{p^2} = \frac{q}{p}$$



---

#### Proof 2: The Story Proof / First-Step Analysis (Right Side)

This is much faster and doesn't need calculus! Let $c = E(X)$ be the expected number of failures. Let's condition on the very first coin flip:

* **Case 1: First flip is Heads (probability $p$)**
* You succeeded immediately! You have **0** failures.


* **Case 2: First flip is Tails (probability $q$)**
* You just wasted **1** failure.
* But because coin flips have **no memory** (a fresh start), the expected number of *additional* failures you still need is $c$. So your total failures in this case is $(1 + c)$.



Set up the equation:


$$c = p \cdot 0 + q \cdot (1 + c)$$

$$c = q + qc \implies c - qc = q \implies c(1 - q) = q$$


Since $1 - q = p$:


$$cp = q \implies c = \frac{q}{p}$$

---

### 4. First Success Distribution: $FS(p)$

* What if instead of counting *failures before the first success*, we count the **total flips until (and including) the first success**?
* Let $X \sim FS(p)$. This means $X \in \{1, 2, 3, \dots\}$.
* **Relationship to Geometric:** If $Y$ is the number of failures, then $Y \sim \text{Geom}(p)$ and:

$$X = Y + 1$$


* **Expected Value:**

$$E(X) = E(Y + 1) = E(Y) + 1 = \frac{q}{p} + 1 = \frac{1-p}{p} + 1 = \frac{1}{p}$$



*(Example: If a coin has a $p = \frac{1}{6}$ chance of success, you expect to wait $\frac{1}{p} = 6$ flips on average to get your first success!)*

---

## Lesson 2: The Negative Binomial Distribution (Waiting for the $r$-th Success)

What if you don't stop at the 1st success, but keep flipping until you get **$r$ successes**?

* **The Story:** Number of independent $\text{Bern}(p)$ failures before the **$r$-th success**.
* **Example Sequence ($r=4$):** `1 0 0 0 1 0 0 1 0 0 0 0 1` (where `1` is success, `0` is failure; we stop the moment the 4th success lands).

### 1. The PMF Formula

$$P(X = n) = \binom{n + r - 1}{r - 1} p^r q^n, \quad \text{for } n \in \{0, 1, 2, \dots\}$$

* **Why this formula works:**
* To have exactly $n$ failures before the $r$-th success, you must flip the coin a total of $(n + r)$ times.
* The **very last flip** (trial $n + r$) **must** be the $r$-th success (probability $p$).
* That leaves $(n + r - 1)$ earlier trials. Exactly $(r - 1)$ of those must be successes, and $n$ must be failures.
* How many ways can we choose which spots get those first $(r - 1)$ successes? Exactly $\binom{n + r - 1}{r - 1}$ ways!



### 2. Expected Value via Linearity (No Algebra Needed!)

Instead of doing a nightmare summation, use **Linearity of Expectation**:

* Let $X_1$ be the failures before the 1st success.
* Let $X_2$ be the failures *between* the 1st and 2nd success.
* Let $X_j$ be the failures between the $(j-1)$-th and $j$-th success.

Because every trial is independent and memoryless, each individual gap $X_j$ is an independent Geometric random variable: $X_j \sim \text{Geom}(p)$. Therefore:


$$X = X_1 + X_2 + \dots + X_r$$

$$E(X) = E(X_1) + E(X_2) + \dots + E(X_r) = \frac{q}{p} + \frac{q}{p} + \dots + \frac{q}{p} = \frac{rq}{p}$$

---

## Lesson 3: Example – Local Maxima in a Random Permutation

This is one of the coolest applications of indicator variables and linearity in the course!

### 1. The Problem

* Take a random permutation of the numbers $\{1, 2, \dots, n\}$ where $n \ge 2$.
* A number is a **local maximum** if it is larger than its immediate neighbors.
* **Example:** In the sequence `3 2 1 4 7 6 5`:
* `3` is a local maximum (larger than `2` on its right; it has no left neighbor).
* `7` is a local maximum (larger than `4` on its left and `6` on its right).


* **Question:** What is the expected number of local maxima?

### 2. The Solution using Indicators

Let $I$ be the total number of local maxima. Break it into a sum of indicators for each position $j$:


$$I = I_1 + I_2 + \dots + I_n$$


where $I_j = 1$ if the number at position $j$ is a local maximum, and $0$ otherwise.

By **Linearity of Expectation**:


$$E(I) = E(I_1) + E(I_2) + \dots + E(I_n)$$

Now let's find the probability $E(I_j) = P(\text{position } j \text{ is a local max})$ for each spot:

* **The Endpoints ($j = 1$ and $j = n$):**
* An endpoint only has **1 neighbor**.
* Between any two adjacent numbers in a random permutation, by symmetry, either one is equally likely to be larger.
* Therefore:

$$E(I_1) = \frac{1}{2} \quad \text{and} \quad E(I_n) = \frac{1}{2}$$




* **The Interior Points ($j \in \{2, 3, \dots, n-1\}$):**
* There are $(n - 2)$ interior positions.
* An interior point has **2 neighbors** (one on the left, one on the right), forming a group of 3 adjacent numbers.
* In a random permutation, all $3! = 6$ orderings of those 3 numbers are equally likely. By symmetry, each of the 3 numbers has an equal $\frac{1}{3}$ chance of being the largest among them!
* Therefore, for any interior position:

$$E(I_j) = \frac{1}{3}$$





### 3. Adding It All Up

$$E(I) = \underbrace{\frac{1}{2}}_{\text{left end}} + \underbrace{(n - 2) \cdot \frac{1}{3}}_{\text{interior points}} + \underbrace{\frac{1}{2}}_{\text{right end}}$$

$$E(I) = 1 + \frac{n - 2}{3} = \frac{3 + n - 2}{3} = \frac{n + 1}{3}$$

For example, in a permutation of $n = 11$ numbers, you expect exactly $\frac{11 + 1}{3} = 4$ local maxima on average!

---

## Lesson 4: The St. Petersburg Paradox

At the very bottom, the notes introduce a famous mind-bending problem in economics and probability:

* **The Game:** You flip a fair coin ($p = \frac{1}{2}$) until heads appears. Let $X$ be the number of flips required (so $X \sim FS(1/2)$).
* **The Payout:** If heads appears on flip $X$, you win $\$2^X$.
* Heads on flip 1 $\rightarrow$ win $\$2^1 = \$2$
* Heads on flip 2 $\rightarrow$ win $\$2^2 = \$4$
* Heads on flip 3 $\rightarrow$ win $\$2^3 = \$8$


* **The Paradox:** How much money should you expect to win on average? Let $Y = 2^X$ be your earnings. Let's calculate $E(Y)$:

$$E(Y) = \sum_{k=1}^{\infty} (\text{Payout for } k \text{ flips}) \times P(X = k)$$

$$E(Y) = \sum_{k=1}^{\infty} 2^k \left(\frac{1}{2}\right)^k = \sum_{k=1}^{\infty} 1 = 1 + 1 + 1 + 1 + \dots = \infty$$

* **Why is it a "Paradox"?** The mathematical expected payout is **infinite ($\infty$)**, which suggests you should be willing to pay any amount of money (even $\$1,000,000$) just to play this game once. But in real life, almost nobody would pay even $\$20$ to play it, because the chance of getting a huge payout is astronomically tiny!