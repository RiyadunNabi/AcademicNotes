
# --- Lecture 8 ---

Bhai, I’ve got you covered! These are classic probability notes (very much in the style of Harvard’s **Stat 110** course) covering the **Binomial Distribution** and how to prove what happens when you add two independent Binomial random variables together.

Let's break down the entire slide from scratch, lesson by lesson, so every single symbol and line makes complete sense.

---

## Lesson 1: What is a Binomial Distribution? (The "Story")

At the top of the page, the notes introduce the Binomial distribution: $X \sim \text{Bin}(n, p)$.

### 1. The Building Block: Bernoulli Trial

Before getting to Binomial, you need to know what a **Bernoulli trial** is:

* A experiment with only **two possible outcomes**: **Success (1)** or **Failure (0)**.
* The probability of Success is $p$, and the probability of Failure is $q = 1 - p$.
* Example: Flipping a biased coin where heads (success) comes up with probability $p$.

### 2. The Binomial Story

* **The Definition:** If you repeat that Bernoulli trial $n$ times independently, the total number of successes you get is a random variable $X$.
* We write this as:

$$X \sim \text{Bin}(n, p)$$


* **The "Story":** In probability, a "story proof" or "story definition" means understanding *what the random variable actually represents in real life* rather than just memorizing a formula. Here, the story is simply: **$X$ is the count of successes in $n$ independent coin flips.**

---

## Lesson 2: Sum of Indicator Random Variables

Under **(2) Sum of indicator random variables**, the notes show a very powerful way to write a Binomial random variable algebraically:

$$X = X_1 + X_2 + \dots + X_n$$

### How It Works:

* Each $X_j$ is an **indicator random variable** for the $j\text{-th}$ trial:

$$X_j = \begin{cases} 1, & \text{if the } j\text{-th trial is a success} \\ 0, & \text{otherwise (failure)} \end{cases}$$


* Because every trial is identical and doesn't affect the others, we say $X_1, X_2, \dots, X_n$ are **i.i.d.** (**I**ndependent and **I**dentically **D**istributed) $\text{Bern}(p)$ random variables.
* **Why do we do this?** Adding up 1s (for successes) and 0s (for failures) automatically counts the total number of successes! This makes hard math proofs much easier later on.

---

## Lesson 3: Probability Mass Function (PMF) & CDF

Under **(3) PMF (probability mass function)**, we answer the question: *What is the exact probability of getting exactly $k$ successes out of $n$ trials?*

### 1. The PMF Formula

$$P(X = k) = \binom{n}{k} p^k q^{n-k}, \quad \text{where } q = 1-p \text{ and } k \in \{0, 1, \dots, n\}$$

* **Why this formula works (The side diagram):**
* Look at the top-right diagram: `T T T H H H H` (3 Tails, 4 Heads out of 7 flips).
* The probability of getting any *specific* sequence of $k$ successes and $n-k$ failures is $p^k q^{n-k}$.
* But those successes could happen in any order! How many ways can we choose $k$ spots out of $n$ trials for the successes? That’s "n choose k", written as $\binom{n}{k}$.



### 2. The Cumulative Distribution Function (CDF)

* The notes briefly mention $F(x) = P(X \le x)$.
* While the **PMF** gives the probability of getting *exactly* $k$ successes, the **CDF** $F(x)$ gives the probability of getting *at most* $x$ successes.
* For a discrete random variable with possible values $a_1, a_2, \dots, a_n$, every individual probability is non-negative ($P_j \ge 0$), and the sum of all possible probabilities must equal 1 ($\sum P_j = 1$).

### 3. Sanity Check: Why do the probabilities sum to 1?

The notes prove this using the **Binomial Theorem**:


$$\sum_{k=0}^{n} P(X = k) = \sum_{k=0}^{n} \binom{n}{k} p^k q^{n-k} = (p + q)^n$$


Since $q = 1 - p$, we know $p + q = 1$. Therefore:


$$(1)^n = 1$$


This confirms our PMF is mathematically valid!

---

## Lesson 4: Adding Two Independent Binomials

The bottom half of the slide answers a major question: **What happens if we add two independent Binomial random variables that share the same success probability $p$?**

### The Theorem:

If $X \sim \text{Bin}(n, p)$ and $Y \sim \text{Bin}(m, p)$ are independent, then:


$$X + Y \sim \text{Bin}(m+n, p)$$

The notes provide **three different proofs** for this one theorem to build deep intuition:

---

### Proof 1: The "Story" Proof (Fastest & Most Intuitive)

* Imagine you flip a coin $n$ times and count the successes ($X$).
* Then, your friend flips the **same type of coin** $m$ times and counts the successes ($Y$).
* If you combine your results, what did you just do? You simply flipped the coin a total of $n + m$ times!
* Therefore, the total successes $X + Y$ must follow a Binomial distribution with $n + m$ trials: $\text{Bin}(m+n, p)$.

---

### Proof 2: The Indicator Proof (Using Building Blocks)

* Express both variables as sums of their Bernoulli indicators:

$$X = X_1 + X_2 + \dots + X_n$$


$$Y = Y_1 + Y_2 + \dots + Y_m$$


* Add them together:

$$X + Y = \sum_{i=1}^{n} X_i + \sum_{j=1}^{m} Y_j$$


* Because $X$ and $Y$ are independent and share the same $p$, you now have $n + m$ independent Bernoulli indicator variables added together. By definition, a sum of $n + m$ i.i.d. Bernoulli variables is $\text{Bin}(m+n, p)$.

---

### Proof 3: The PMF / Algebraic Proof (The Heavy Math at the Bottom)

This is a **convolution proof** using the Law of Total Probability. We want to find $P(X + Y = k)$.

**Step 1: Condition on all possible values of $X$**


$$P(X + Y = k) = \sum_{j=0}^{k} P(X + Y = k \mid X = j) P(X = j)$$

**Step 2: Simplify using independence**
If $X = j$, then for $X + Y$ to equal $k$, $Y$ must equal $k - j$. Because $X$ and $Y$ are independent, we can drop the conditioning:


$$= \sum_{j=0}^{k} P(Y = k - j) P(X = j)$$

**Step 3: Plug in the PMF formulas for $X$ and $Y$**


$$= \sum_{j=0}^{k} \left[ \binom{m}{k-j} p^{k-j} q^{m-k+j} \right] \left[ \binom{n}{j} p^j q^{n-j} \right]$$

**Step 4: Combine powers of $p$ and $q$**

* Notice the exponents of $p$: $(k - j) + j = k$.
* Notice the exponents of $q$: $(m - k + j) + (n - j) = m + n - k$.
* Since $p^k q^{m+n-k}$ does not depend on the summation index $j$, we can pull it out of the sum:

$$= p^k q^{m+n-k} \sum_{j=0}^{k} \binom{m}{k-j} \binom{n}{j}$$



**Step 5: Apply Vandermonde's Identity**
In combinatorics, the identity $\sum_{j=0}^{k} \binom{m}{k-j} \binom{n}{j} = \binom{m+n}{k}$ means: *"Choosing $k$ people from a group of $m$ men and $n$ women is the same as summing over all ways to choose $k-j$ men and $j$ women."*

Substituting this identity into our equation gives:


$$P(X + Y = k) = \binom{m+n}{k} p^k q^{m+n-k}$$

Which is exactly the PMF of **$\text{Bin}(m+n, p)$**!
# --- Lecture 9 ---

Bhai, Lecture 9 is an absolute gem! This lecture introduces three of the most important concepts in probability: the **Hypergeometric Distribution**, the definition of **Independence**, and the superpower known as **Linearity of Expectation** (along with the "Fundamental Bridge").

Let's break down this entire slide from scratch, lesson by lesson, so you understand every single formula and proof!

---

## Lesson 1: The Hypergeometric Distribution (Sampling Without Replacement)

In Lecture 8, the **Binomial** distribution was about coin flips (independent trials *with* replacement). But what happens when you deal cards or pick marbles **without replacement**? That gives us the **Hypergeometric distribution**.

### 1. Example: Aces in a 5-Card Hand

* Suppose you are dealt 5 cards from a standard 52-card deck. Let $X$ be the number of Aces in your hand ($X = \text{\#aces}$).
* Since there are only 4 Aces in the entire deck, you can't get more than 4 Aces. Therefore:

$$P(X = k) = 0 \quad \text{if } k \notin \{0, 1, 2, 3, 4\}$$


* For valid values of $k \in \{0, 1, 2, 3, 4\}$, the Probability Mass Function (PMF) is:

$$P(X = k) = \frac{\binom{4}{k} \binom{48}{5-k}}{\binom{52}{5}}$$


* **Why this formula works:**
* **Denominator:** How many total ways can you choose 5 cards out of 52? $\binom{52}{5}$.
* **Numerator:** To get exactly $k$ Aces, you must choose $k$ Aces out of the 4 available ($\binom{4}{k}$), **and** choose the remaining $5 - k$ cards from the 48 non-Aces ($\binom{48}{5-k}$).



---

### 2. The General Story: Marbles in an Urn

Now let's generalize this! Suppose an urn contains $w$ **white** marbles and $b$ **black** marbles (so $w + b$ total marbles). You pick a random sample of $n$ marbles **without replacement**. Let $X$ be the number of white marbles drawn.

* **The General Hypergeometric PMF:**

$$P(X = k) = \frac{\binom{w}{k} \binom{b}{n-k}}{\binom{b+w}{n}}$$


* **Sanity Check (Why do these sum to 1?):**
* The notes check: $\sum_{k=0}^{n} P(X = k) = 1$.
* Using **Vandermonde's Identity** (from Lecture 8), we know that:

$$\sum_{k=0}^{n} \binom{w}{k} \binom{b}{n-k} = \binom{b+w}{n}$$


* Since the numerator sums up to equal the denominator, the total probability is exactly 1!



---

## Lesson 2: Independence of Random Variables

Next, the notes formally define what it means for two random variables $X$ and $Y$ to be **independent**:

* **General Case (using CDFs):**

$$P(X \le x, Y \le y) = P(X \le x) \cdot P(Y \le y)$$


* **Discrete Case (using PMFs):**

$$P(X = x, Y = y) = P(X = x) \cdot P(Y = y)$$


* **In plain English:** $X$ and $Y$ are independent if knowing the value of $X$ gives you zero information about the value of $Y$, meaning their joint probability is just the product of their individual probabilities.

---

## Lesson 3: Expected Value $E(X)$ (What is an Average?)

What does **Expected Value** mean? It’s simply the **long-run weighted average** of a random variable.

### 1. Intuition from a Simple Average

Look at the dataset in the notes: `1, 1, 1, 1, 1, 3, 3, 5` (eight numbers total).

* Instead of adding all eight numbers and dividing by 8, you can group them by how often they appear (their relative frequency/probability):

$$\text{Average} = \frac{5}{8}\cdot 1 + \frac{2}{8}\cdot 3 + \frac{1}{8}\cdot 5$$


* Notice the pattern: **(Probability of value) $\times$ (Value)**.

### 2. Formal Definition of Expected Value

For any discrete random variable $X$:


$$E(X) = \sum_{x} x P(X = x)$$


*(Sum over all possible values $x$ of the value multiplied by its probability).*

---

## Lesson 4: The Three Columns — Expectation & The Fundamental Bridge

The bottom-left and middle of the slide compares Bernoulli and Binomial expectations, introducing a legendary probability rule called the **Fundamental Bridge**.

### Column 1: Bernoulli Distribution & The Fundamental Bridge

Let $X \sim \text{Bern}(p)$. $X$ can only be 1 (with probability $p$) or 0 (with probability $q$).

* **Calculate $E(X)$ using the definition:**

$$E(X) = 1 \cdot P(X = 1) + 0 \cdot P(X = 0) = 1 \cdot p + 0 \cdot q = p$$


* **The Fundamental Bridge:**
* Let $X$ be an **indicator random variable** for an event $A$:

$$X = \begin{cases} 1, & \text{if } A \text{ occurs} \\ 0, & \text{otherwise} \end{cases}$$


* Since $P(X = 1) = P(A)$, we get a powerful identity:

$$P(A) = E(X)$$


* **Why is this called a "Bridge"?** It builds a bridge between **Probability** and **Expected Value**. Whenever you want to find the probability of an event, you can instead find the expected value of its indicator variable!



---

### Column 2: Binomial Expectation (The Hard Algebraic Proof)

What is $E(X)$ if $X \sim \text{Bin}(n, p)$? Let's prove it directly from the PMF using algebra:

1. **Write out the definition:**

$$E(X) = \sum_{k=0}^{n} k \binom{n}{k} p^k q^{n-k}$$



*(Note: When $k = 0$, the term is 0, so we can start the sum at $k = 1$)*.
2. **Use the Committee Identity $k \binom{n}{k} = n \binom{n-1}{k-1}$:**
*(Choosing a committee of size $k$ with 1 president from $n$ people is the same as choosing the president first ($n$ ways), then choosing the remaining $k-1$ members).*

$$E(X) = \sum_{k=1}^{n} n \binom{n-1}{k-1} p^k q^{n-k}$$


3. **Factor out $np$:**

$$E(X) = np \sum_{k=1}^{n} \binom{n-1}{k-1} p^{k-1} q^{n-k}$$


4. **Change of variables (Let $j = k - 1$, so when $k = 1 \implies j = 0$):**

$$E(X) = np \sum_{j=0}^{n-1} \binom{n-1}{j} p^j q^{(n-1)-j}$$


5. **Apply the Binomial Theorem:**
The entire summation is simply the expansion of $(p + q)^{n-1}$. Since $p + q = 1$:

$$E(X) = np(1)^{n-1} = np$$



---

### Column 3: Binomial Expectation (The Easy Linearity Way!)

That algebraic proof was a lot of work. Column 3 shows how to do it in **one line** using **Linearity of Expectation**.

* **The Golden Rule of Linearity:**

$$E(X + Y) = E(X) + E(Y) \quad \text{(Even if } X \text{ and } Y \text{ are DEPENDENT!)}$$


$$E(cX) = cE(X)$$


* **Applying it to Binomial:**
* Remember from Lecture 8 that a Binomial variable is just the sum of $n$ Bernoulli indicators:

$$X = X_1 + X_2 + \dots + X_n$$


* Take the expected value of both sides:

$$E(X) = E(X_1) + E(X_2) + \dots + E(X_n)$$


* Since each trial is $\text{Bern}(p)$, each $E(X_j) = p$:

$$E(X) = p + p + \dots + p = np$$


* Boom! No calculus, no committee identities—just pure intuition!



---

## Lesson 5: The Magic of Linearity (Expected Aces in a Hand)

At the very bottom of the slide, the notes apply **Linearity of Expectation** to solve a problem that would be horrible to solve algebraically: **What is the expected number of Aces in a 5-card hand?**

### The Setup:

* Let $X$ be the total number of Aces in a 5-card hand.
* Express $X$ as a sum of indicator variables for each of the 5 cards in your hand:

$$X = X_1 + X_2 + X_3 + X_4 + X_5$$



where $X_j = 1$ if the $j\text{-th}$ card in your hand is an Ace, and $0$ otherwise.

### The Calculation:

1. **Use Linearity of Expectation:**

$$E(X) = E(X_1 + X_2 + \dots + X_5) = E(X_1) + E(X_2) + \dots + E(X_5)$$



*(Even though the cards are drawn without replacement—meaning $X_1, X_2, \dots$ are **dependent**—linearity still works!)*
2. **Find $E(X_j)$ for a single card:**
* By the Fundamental Bridge, $E(X_j) = P(\text{the } j\text{-th card is an Ace})$.
* By **symmetry** (unconditional probability), before you look at the deck, *any* card has an equal $\frac{4}{52}$ chance of being an Ace!
* Therefore, $E(X_j) = \frac{4}{52}$ for every $j \in \{1, 2, 3, 4, 5\}$.


3. **Add them up:**

$$E(X) = \frac{4}{52} + \frac{4}{52} + \frac{4}{52} + \frac{4}{52} + \frac{4}{52} = 5 \times \frac{4}{52} = \frac{20}{52} = \frac{5}{13}$$



On average, you will get **$\frac{5}{13}$ of an Ace** (about 0.385 Aces) per 5-card hand!
# --- Lecture 10 ---

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
# --- Lecture 11 ---

Bhai, Lecture 11 is legendary! This lecture introduces the **Poisson Distribution**—one of the most famous distributions in all of statistics—and shows how it acts as the universal approximation for counting **rare events** (even when trials are slightly dependent!).

Let’s break down this entire slide from scratch, lesson by lesson, so every single step and proof is crystal clear!

---

## Warm-Up: Resolving the St. Petersburg Paradox (Top Line)

At the very top of the page, the notes wrap up the **St. Petersburg Paradox** from Lecture 10:

* Remember, mathematically, if the payout is unbounded ($2^1, 2^2, 2^3, \dots$), the expected value is infinite:

$$E(Y) = \sum_{k=1}^{\infty} 2^k \cdot \frac{1}{2^k} = \sum_{k=1}^{\infty} 1 = \infty$$


* **The Realistic Fix (Bounded Payout):** No casino or bank has infinite money! What if the maximum payout is capped at **$\$2^{30}$** (meaning the game stops after at most 30 flips)?
* Now, the expected value is simply a finite sum:

$$E(Y) = \sum_{k=1}^{30} 2^k \cdot \frac{1}{2^k} = \sum_{k=1}^{30} 1 = 30$$


* A realistic maximum cap drops the expected value from $\infty$ down to just **$\$30$**!

---

## Lesson 1: What is the Poisson Distribution? (PMF & Sanity Check)

We write that a random variable follows a Poisson distribution as:


$$X \sim \text{Pois}(\lambda)$$


where $\lambda > 0$ is the **rate parameter** (the average number of events that occur in a fixed interval).

### 1. Probability Mass Function (PMF)

For any non-negative integer $k \in \{0, 1, 2, \dots\}$, the probability of observing exactly $k$ events is:


$$P(X = k) = \frac{e^{-\lambda} \lambda^k}{k!}$$

### 2. Sanity Check: Why do these probabilities sum to 1?

To prove this is a valid probability distribution, we use the famous **Taylor series expansion of $e^x$**:


$$e^x = \sum_{k=0}^{\infty} \frac{x^k}{k!}$$

Let's sum our PMF over all possible values of $k$:


$$\sum_{k=0}^{\infty} P(X = k) = \sum_{k=0}^{\infty} \frac{e^{-\lambda} \lambda^k}{k!} = e^{-\lambda} \sum_{k=0}^{\infty} \frac{\lambda^k}{k!}$$


Since that infinite sum is simply $e^\lambda$:


$$= e^{-\lambda} \cdot e^{\lambda} = e^0 = 1$$


It checks out!

---

## Lesson 2: Expected Value $E(X) = \lambda$ (The Calculus Proof)

What is the expected value of a Poisson random variable? Let's prove that **$E(X) = \lambda$** directly from the PMF:

1. **Write out the definition of Expectation:**

$$E(X) = \sum_{k=0}^{\infty} k \cdot \frac{e^{-\lambda} \lambda^k}{k!}$$



*(When $k = 0$, the term is $0$, so we can start the summation at $k = 1$.)*
2. **Cancel $k$ with $k!$ in the denominator:**
Since $k! = k \cdot (k-1)!$, we get $\frac{k}{k!} = \frac{1}{(k-1)!}$:

$$E(X) = e^{-\lambda} \sum_{k=1}^{\infty} \frac{\lambda^k}{(k-1)!}$$


3. **Factor out one $\lambda$:**

$$E(X) = e^{-\lambda} \lambda \sum_{k=1}^{\infty} \frac{\lambda^{k-1}}{(k-1)!}$$


4. **Change of variables (Let $j = k - 1$):**
When $k = 1 \implies j = 0$:

$$E(X) = e^{-\lambda} \lambda \sum_{j=0}^{\infty} \frac{\lambda^j}{j!}$$


5. **Recognize the Taylor series again:**
That entire summation is just $e^\lambda$:

$$E(X) = e^{-\lambda} \lambda e^\lambda = \lambda$$



---

## Lesson 3: The Poisson Paradigm (The Law of Rare Events)

Why is the Poisson distribution so important? Because it models **counts of rare events**.

### 1. Real-World Applications

* Number of **emails** arriving in your inbox in an hour.
* Number of **earthquakes** in a region per year.
* Number of **typos** on a book page.

### 2. The Poisson Paradigm

Suppose you have $n$ events $A_1, A_2, \dots, A_n$, where each event has probability $P(A_j) = p_j$.
If:

1. **$n$ is very large** (many opportunities for the event to happen),
2. **Each $p_j$ is very small** (each individual event is rare), and
3. The events are **independent or weakly dependent**,

Then the total number of events that occur is **approximately Poisson** with rate parameter:


$$\lambda \approx \sum_{j=1}^{n} p_j$$

---

## Lesson 4: How Binomial Becomes Poisson (The Limit Proof)

The middle block of notes gives the rigorous mathematical proof of **why** the Binomial distribution converges to the Poisson distribution when $n \to \infty$ and $p \to 0$, while keeping the average $\lambda = np$ constant.

### The Setup:

Let $X \sim \text{Bin}(n, p)$. Since $\lambda = np$, we can write $p = \frac{\lambda}{n}$.
Let's substitute $p = \frac{\lambda}{n}$ into the Binomial PMF:


$$P(X = k) = \binom{n}{k} p^k (1 - p)^{n-k} = \frac{n(n-1)\cdots(n-k+1)}{k!} \cdot \left(\frac{\lambda}{n}\right)^k \cdot \left(1 - \frac{\lambda}{n}\right)^{n-k}$$

Let's rearrange the terms into four separate pieces and take the limit as **$n \to \infty$**:


$$P(X = k) = \frac{1}{k!} \lambda^k \cdot \left[ \frac{n(n-1)\cdots(n-k+1)}{n^k} \right] \cdot \left(1 - \frac{\lambda}{n}\right)^n \cdot \left(1 - \frac{\lambda}{n}\right)^{-k}$$

### Let's evaluate each bracket as $n \to \infty$:

1. **The fraction $\frac{n(n-1)\cdots(n-k+1)}{n^k}$:**
There are $k$ terms in the numerator and $k$ factors of $n$ in the denominator. As $n \to \infty$, this fraction approaches **$1$**.
2. **The exponential limit $\left(1 - \frac{\lambda}{n}\right)^n$:**
Using the famous Euler limit formula $\lim_{n \to \infty} \left(1 + \frac{x}{n}\right)^n = e^x$, this term approaches **$e^{-\lambda}$**.
3. **The last term $\left(1 - \frac{\lambda}{n}\right)^{-k}$:**
Since $k$ is a fixed number, as $n \to \infty$, $\frac{\lambda}{n} \to 0$, so $(1 - 0)^{-k}$ approaches **$1$**.

### Combining the limits:

$$\lim_{n \to \infty} P(X = k) = \frac{1}{k!} \lambda^k \cdot 1 \cdot e^{-\lambda} \cdot 1 = \frac{e^{-\lambda} \lambda^k}{k!}$$


That is exactly the **Poisson PMF**!

---

## Lesson 5: Classic Example — The Birthday Problem for Triplets

At the bottom of the slide, we see a brilliant real-world application of the Poisson Paradigm: **In a group of $n$ people, what is the approximate probability of finding at least 3 people who share the same birthday?**

### 1. Why is this hard to solve exactly?

* Finding matching pairs or triplets using exact probability is messy because triplet matches are **dependent** (e.g., if Person A & Person B share a birthday, and Person B & Person C share a birthday, then Person A & Person C *must* share a birthday).
* But because the dependence is **weak** and matching birthdays is **rare**, the **Poisson Paradigm** works like magic!

### 2. Step-by-Step Solution:

1. **Count the number of possible triplets:**
How many groups of 3 people can we form out of $n$ people? Exactly $\binom{n}{3}$.
2. **Define indicator variables:**
Let $I_{ijk} = 1$ if person $i$, person $j$, and person $k$ all share the same birthday ($i < j < k$), and $0$ otherwise.
3. **Find the probability of a single triplet matching:**
* Person $i$ can have any birthday ($\frac{365}{365} = 1$).
* Person $j$ must match Person $i$ ($\frac{1}{365}$).
* Person $k$ must also match Person $i$ ($\frac{1}{365}$).
* So, $P(I_{ijk} = 1) = \frac{1}{365^2}$.


4. **Find the expected number of matching triplets ($\lambda$):**
By Linearity of Expectation:

$$\lambda = E(\text{\# triplets match}) = \binom{n}{3} \cdot \frac{1}{365^2}$$


5. **Apply Poisson Approximation:**
Let $X$ be the number of matching triplets. By the Poisson Paradigm:

$$X \stackrel{\text{approx}}{\sim} \text{Pois}(\lambda), \quad \text{where } \lambda = \frac{\binom{n}{3}}{365^2}$$


6. **Calculate the probability of AT LEAST ONE matching triplet ($X \ge 1$):**

$$P(X \ge 1) = 1 - P(X = 0)$$



Using the Poisson PMF formula for $k = 0$:

$$P(X = 0) \approx \frac{e^{-\lambda} \lambda^0}{0!} = e^{-\lambda}$$



Therefore:

$$P(X \ge 1) \approx 1 - e^{-\lambda}$$



*(Example: If you have $n = 100$ people in a lecture hall, $\lambda = \frac{\binom{100}{3}}{365^2} \approx 1.214$. The probability of at least one birthday triplet is $1 - e^{-1.214} \approx \mathbf{70.3\%}$!)*
# --- Lecture 12 ---

Bhai, welcome to **Lecture 12**! This is a huge milestone in your probability journey—we are officially crossing over from **Discrete Random Variables** (where we use summations $\sum$) to **Continuous Random Variables** (where we use calculus and integrals $\int$).

Let’s break down this entire slide from scratch, lesson by lesson, so you understand every single formula, integral, and concept!

---

## Lesson 1: Discrete vs. Continuous & What is a PDF?

At the top of the page, the notes compare the discrete world we just left with the continuous world we are entering.

### 1. The Comparison Table

| Feature | Discrete Random Variable ($X$) | Continuous Random Variable ($X$) |
| --- | --- | --- |
| **Probability Function** | **PMF**: $P(X = x)$ | **PDF**: $f_X(x)$ *(Probability Density Function)* |
| **Single-Point Probability** | Can be greater than 0 | **Always zero:** $P(X = a) = 0$ |
| **CDF ($F_X(x)$)** | $F_X(x) = P(X \le x) = \sum_{t \le x} P(X = t)$ | $F_X(x) = P(X \le x) = \int_{-\infty}^{x} f_X(t) \, dt$ |
| **Expected Value ($E(X)$)** | $E(X) = \sum_{x} x P(X = x)$ | $E(X) = \int_{-\infty}^{+\infty} x f_X(x) \, dx$ |

---

### 2. Why is $P(X = a) = 0$ for Continuous Variables?

* In continuous probability (like measuring exact height, time, or a random number on a line), there are **infinitely many** possible points.
* The chance of hitting one *exact, infinitely precise* mathematical point is **zero**.
* Therefore, for continuous distributions, we only care about probabilities over **intervals**:

$$P(a \le X \le b) = \int_{a}^{b} f(x) \, dx$$



### 3. What does a PDF $f(x)$ actually mean? (The Small Curve Diagram)

* Since $f(x_0)$ is a **density** (not a probability), it can actually be greater than 1!
* To get a real probability, you must multiply the density by a tiny interval width $\epsilon$:

$$P\left(x_0 - \frac{\epsilon}{2} \le X \le x_0 + \frac{\epsilon}{2}\right) \approx \epsilon \cdot f(x_0), \quad \text{for } \epsilon > 0 \text{ very small}$$


* **Intuition:** Think of density like mass per unit length. A single point has no length, so it has no mass (no probability). But a tiny slice of length $\epsilon$ has mass $\text{Density} \times \text{Length}$!

---

## Lesson 2: Rules of a PDF & The Calculus Connection

Under the table, the notes list the fundamental mathematical rules that connect the PDF $f(x)$ and the CDF $F(x)$.

### 1. Two Requirements for a Valid PDF

Just like discrete PMFs must be non-negative and sum to 1, a continuous PDF must satisfy:

1. **Non-negativity:** $f(x) \ge 0$ for all $x$.
2. **Total area is 1:**

$$\int_{-\infty}^{+\infty} f(x) \, dx = 1$$



### 2. PDF $\leftrightarrow$ CDF (Fundamental Theorem of Calculus)

* **To get the CDF from the PDF:** Integrate from $-\infty$ up to $x$:

$$F_X(x) = P(X \le x) = \int_{-\infty}^{x} f(t) \, dt$$


* **To get the PDF from the CDF:** Take the derivative with respect to $x$:

$$f(x) = F'(x) = \frac{d}{dx} F(x)$$



### 3. Finding Interval Probabilities

To find the probability that $X$ lands between $a$ and $b$:


$$P(a \le X \le b) = \int_{a}^{b} f(x) \, dx = \int_{-\infty}^{b} f(x) \, dx - \int_{-\infty}^{a} f(x) \, dx = F(b) - F(a)$$


*(Note: Because single points have probability 0, it doesn't matter if you use $<$ or $\le$: $P(a < X < b) = P(a \le X \le b)$).*

---

## Lesson 3: Variance & Standard Deviation (The Shortcut Formula)

In the middle of the slide, we define how spread out a distribution is around its mean.

### 1. Definitions

* **Variance ($\text{Var}(X)$):** The expected squared distance from the mean:

$$\text{Var}(X) = E\left[(X - E(X))^2\right]$$


* **Standard Deviation ($\text{SD}(X)$):** The square root of variance (which puts units back into the original scale):

$$\text{SD}(X) = \sqrt{\text{Var}(X)}$$



### 2. Proof of the Shortcut Formula

Instead of calculating $E[(X - E(X))^2]$ directly, we almost always use the **shortcut formula**:


$$\text{Var}(X) = E(X^2) - (E(X))^2$$

**Let's prove it algebraically:**

1. Expand the quadratic inside the expectation:

$$\text{Var}(X) = E\left[X^2 - 2X E(X) + (E(X))^2\right]$$


2. Apply **Linearity of Expectation** *(remember, $E(X)$ is just a constant number!)*:

$$\text{Var}(X) = E(X^2) - 2E(X) \cdot E(X) + (E(X))^2$$


3. Simplify:

$$\text{Var}(X) = E(X^2) - 2(E(X))^2 + (E(X))^2 = E(X^2) - (E(X))^2$$



---

## Lesson 4: The Continuous Uniform Distribution — $\text{Unif}(a, b)$

Now we meet our very first continuous distribution: **The Uniform Distribution** on the interval $[a, b]$.

### 1. The Story & Intuition

* **The Story:** Picking a completely random point on the segment $[a, b]$, where every region of equal size has the same chance of being chosen ($\text{probability} \propto \text{length}$).
* **Example:** A spinner on a wheel, or choosing a random arrival time between 1:00 PM and 2:00 PM.

---

### 2. Deriving the PDF $f(x)$

Because the probability is uniform, the height of the PDF must be a constant $c$ between $a$ and $b$:


$$f(x) = \begin{cases} c, & \text{if } a \le x \le b \\ 0, & \text{otherwise} \end{cases}$$

**How do we find $c$?** The total area under the curve must equal 1:


$$\int_{-\infty}^{+\infty} f(x) \, dx = \int_{a}^{b} c \, dx = c(b - a) = 1 \implies c = \frac{1}{b - a}$$


Therefore, the valid PDF is:


$$f(x) = \frac{1}{b - a}, \quad \text{for } a \le x \le b$$

---

### 3. Deriving the CDF $F(x)$

Let's integrate the PDF from $a$ up to $x$:


$$F(x) = \int_{a}^{x} f(t) \, dt = \int_{a}^{x} \frac{1}{b - a} \, dt = \frac{x - a}{b - a}$$

Written as a complete piecewise function:


$$F(x) = \begin{cases} 0, & \text{if } x < a \\ \frac{x - a}{b - a}, & \text{if } a \le x \le b \\ 1, & \text{if } x > b \end{cases}$$

---

### 4. Deriving Expected Value $E(X)$

What is the average value of a uniform distribution? Let's integrate:


$$E(X) = \int_{a}^{b} x f(x) \, dx = \int_{a}^{b} x \left(\frac{1}{b - a}\right) dx = \frac{1}{b - a} \left[ \frac{x^2}{2} \right]_{a}^{b}$$

$$= \frac{1}{b - a} \left( \frac{b^2 - a^2}{2} \right)$$


Using the difference of squares difference ($b^2 - a^2 = (b - a)(b + a)$):


$$E(X) = \frac{(b - a)(a + b)}{2(b - a)} = \frac{a + b}{2}$$


*(Makes perfect intuitive sense: the average is exactly the midpoint between $a$ and $b$!)*

---

### 5. What is LOTUS? (Law of the Unconscious Statistician)

The notes write:


$$E(X^2) = \int_{a}^{b} x^2 f(x) \, dx \quad \text{[Law of unconscious statistician (LOTUS)]}$$

* **Why is it called LOTUS?** Many students unconsciously assume that to find $E(g(X))$, you just integrate $g(x) f(x) \, dx$ without realizing this is actually a deep theorem!
* In general, for any function $g(X)$:

$$E(g(X)) = \int_{-\infty}^{+\infty} g(x) f_X(x) \, dx$$



---

## Lesson 5: Standard Uniform $\text{Unif}(0, 1)$ & General Variance

At the bottom of the slide, we use **LOTUS** and our shortcut formula to find the variance of a uniform distribution.

### 1. The Standard Uniform Case: $U \sim \text{Unif}(0, 1)$

Here, $a = 0$ and $b = 1$, which means the PDF is simply $f(u) = 1$.

* **Mean $E(U)$:**

$$E(U) = \frac{0 + 1}{2} = \frac{1}{2}$$


* **Second Moment $E(U^2)$ using LOTUS:**

$$E(U^2) = \int_{0}^{1} u^2 f(u) \, du = \int_{0}^{1} u^2 (1) \, du = \left[ \frac{u^3}{3} \right]_{0}^{1} = \frac{1}{3}$$


* **Variance $\text{Var}(U)$:**

$$\text{Var}(U) = E(U^2) - (E(U))^2 = \frac{1}{3} - \left(\frac{1}{2}\right)^2 = \frac{1}{3} - \frac{1}{4} = \frac{1}{12}$$



---

### 2. General Variance Formula for $\text{Unif}(a, b)$

At the very bottom right, the notes point to the general case for any interval $[a, b]$:


$$\text{Var}(X) = \frac{(b - a)^2}{12}$$

*(Notice how the spread only depends on the **length** of the interval $(b - a)$, squared!)*
# --- Lecture 13 ---

Bhai, Lecture 13 is the crown jewel of probability! We have officially arrived at the **Normal (Gaussian) Distribution**—the famous bell curve that rules the universe of statistics.

This slide is packed with some of the most beautiful math in the entire course, including how to prove where that mysterious $\sqrt{2\pi}$ comes from using multivariable calculus, and how to build any Normal distribution from scratch. Let's break it down lesson by lesson!

---

## Lesson 1: Why the Normal Distribution? (The Central Limit Theorem)

At the very top, the notes state: **"Central Limit Thm: Sum of lots of iid r.v.s looks normal."**

* **What this means:** Why does the normal distribution appear everywhere in nature (heights, exam scores, measurement errors)?
* **The Central Limit Theorem (CLT):** If you take a large number of independent and identically distributed (i.i.d.) random variables from *any* distribution and add them together, their sum will always form a bell curve!

---

## Lesson 2: The Standard Normal PDF & The $\sqrt{2\pi}$ Mystery

Let $Z \sim \mathcal{N}(0, 1)$ be a **Standard Normal** random variable (mean 0, variance 1).

### 1. The Bell Curve Shape

The core shape of the bell curve is given by the exponential function:


$$f(z) = c e^{-z^2/2}$$


where $c$ is a **normalizing constant** that makes sure the total area under the curve equals 1:


$$\int_{-\infty}^{+\infty} f(z) \, dz = c \int_{-\infty}^{+\infty} e^{-z^2/2} \, dz = 1$$

But how do we solve $\int_{-\infty}^{+\infty} e^{-z^2/2} \, dz$? You cannot integrate $e^{-z^2/2}$ using standard calculus! Instead, we use a brilliant **multivariable calculus trick** shown on the right side of the page.

---

### 2. The Polar Coordinate / Jacobian Proof

**Step 1: Square the integral by using two independent variables $x$ and $y$**
Let $I = \int_{-\infty}^{+\infty} e^{-z^2/2} \, dz$. Multiplying $I \times I$ gives a double integral over the entire 2D plane:


$$I^2 = \left( \int_{-\infty}^{+\infty} e^{-x^2/2} \, dx \right) \left( \int_{-\infty}^{+\infty} e^{-y^2/2} \, dy \right) = \int_{-\infty}^{+\infty} \int_{-\infty}^{+\infty} e^{-(x^2 + y^2)/2} \, dx dy$$

**Step 2: Switch to Polar Coordinates $(r, \theta)$**

* Let $x = r \cos\theta$ and $y = r \sin\theta$.
* Since $x^2 + y^2 = r^2$, the exponent simply becomes $-r^2/2$.
* The radius $r$ goes from 0 to $\infty$, and the angle $\theta$ goes from 0 to $2\pi$.

**Step 3: Apply the Jacobian Determinant ($J = r$)**
When changing variables in a double integral, area elements change by the **Jacobian determinant** $J$:


$$J = \begin{vmatrix} \frac{\partial x}{\partial r} & \frac{\partial x}{\partial \theta} \\ \frac{\partial y}{\partial r} & \frac{\partial y}{\partial \theta} \end{vmatrix} = \begin{vmatrix} \cos\theta & -r\sin\theta \\ \sin\theta & r\cos\theta \end{vmatrix} = r\cos^2\theta - (-r\sin^2\theta) = r(\cos^2\theta + \sin^2\theta) = r$$


Therefore, $dx dy = r \, dr d\theta$.

**Step 4: Solve the polar integral using substitution**


$$I^2 = \int_0^{2\pi} \int_0^{\infty} e^{-r^2/2} r \, dr d\theta$$


Let $u = r^2/2$, which means $du = r \, dr$. As $r$ goes from 0 to $\infty$, $u$ also goes from 0 to $\infty$:


$$I^2 = \int_0^{2\pi} \left( \int_0^{\infty} e^{-u} \, du \right) d\theta$$


Since $\int_0^{\infty} e^{-u} \, du = 1$, we are left with:


$$I^2 = \int_0^{2\pi} 1 \, d\theta = 2\pi$$

**Step 5: Take the square root to find $c$**
Since $I^2 = 2\pi$, our original integral is $I = \sqrt{2\pi}$. Therefore:


$$c \sqrt{2\pi} = 1 \implies c = \frac{1}{\sqrt{2\pi}}$$


This gives us the official **Standard Normal PDF**:


$$f(z) = \frac{1}{\sqrt{2\pi}} e^{-z^2/2}$$

---

## Lesson 3: Mean and Variance of $Z \sim \mathcal{N}(0, 1)$

Now that we have the PDF, let's prove that its mean is 0 and its variance is 1.

### 1. Why $E(Z) = 0$ (Symmetry of Odd Functions)

$$E(Z) = \frac{1}{\sqrt{2\pi}} \int_{-\infty}^{+\infty} z e^{-z^2/2} \, dz = 0$$

* **Why?** An **odd function** satisfies $g(-z) = -g(z)$. Here, $z$ is odd and $e^{-z^2/2}$ is even, so their product $z e^{-z^2/2}$ is an odd function.
* Integrating an odd function from $-\infty$ to $+\infty$ always cancels out to **0** because the negative left half mirrors the positive right half exactly!

---

### 2. Why $\text{Var}(Z) = 1$ (Integration by Parts)

Using our variance shortcut formula:


$$\text{Var}(Z) = E(Z^2) - (E(Z))^2 = E(Z^2) - 0 = E(Z^2)$$

Let's calculate $E(Z^2)$:


$$E(Z^2) = \frac{1}{\sqrt{2\pi}} \int_{-\infty}^{+\infty} z^2 e^{-z^2/2} \, dz$$


Because $z^2 e^{-z^2/2}$ is an **even function** (symmetric around 0), we can integrate from 0 to $\infty$ and multiply by 2:


$$E(Z^2) = \frac{2}{\sqrt{2\pi}} \int_0^{\infty} z^2 e^{-z^2/2} \, dz$$

**We solve $\int_0^{\infty} z^2 e^{-z^2/2} \, dz$ using Integration by Parts ($\int u \, dv = uv - \int v \, du$):**

* Let $u = z \implies du = dz$.
* Let $dv = z e^{-z^2/2} \, dz \implies v = -e^{-z^2/2}$.

Applying the formula:


$$\int_0^{\infty} z \left(z e^{-z^2/2}\right) dz = \left[ -z e^{-z^2/2} \right]_0^{\infty} - \int_0^{\infty} \left(-e^{-z^2/2}\right) dz$$

* The boundary term $\left[ -z e^{-z^2/2} \right]_0^{\infty}$ evaluates to 0 (because $e^{-z^2/2}$ shrinks to 0 much faster than $z$ grows).
* The remaining integral $\int_0^{\infty} e^{-z^2/2} \, dz$ is simply half of the Gaussian integral we proved earlier: $\frac{\sqrt{2\pi}}{2}$.

Plugging this back into $E(Z^2)$:


$$E(Z^2) = \frac{2}{\sqrt{2\pi}} \times \frac{\sqrt{2\pi}}{2} = 1 \implies \text{Var}(Z) = 1$$

---

## Lesson 4: The Standard Normal CDF ($\Phi$) & Variance Rules

### 1. Notation for the Standard Normal CDF

In statistics, the cumulative distribution function (CDF) of the standard normal is so important that it gets its own Greek letter, **$\Phi$ (Phi)**:


$$\Phi(z) = P(Z \le z) = \frac{1}{\sqrt{2\pi}} \int_{-\infty}^{z} e^{-t^2/2} \, dt$$

---

### 2. Location and Scale Transformation ($X = \mu + \sigma Z$)

What if we want a normal distribution with any mean $\mu$ (location) and any standard deviation $\sigma > 0$ (scale)?

* We simply stretch/scale $Z$ by $\sigma$, and shift/locate it by adding $\mu$:

$$X = \mu + \sigma Z \implies X \sim \mathcal{N}(\mu, \sigma^2)$$


* **Check the Mean using Linearity:**

$$E(X) = E(\mu + \sigma Z) = E(\mu) + \sigma E(Z) = \mu + \sigma(0) = \mu$$



---

### 3. Properties of Variance (Middle Block)

The notes remind us how variance behaves under algebra:

* **Adding a constant does not change spread:**

$$\text{Var}(X + c) = \text{Var}(X)$$


* **Multiplying by a constant squares the spread:**

$$\text{Var}(cX) = c^2 \text{Var}(X)$$


* **Variance of a sum:**

$$\text{Var}(X + Y) \neq \text{Var}(X) + \text{Var}(Y) \quad \text{[Only equal if } X \text{ and } Y \text{ are INDEPENDENT!]}$$



Applying this to our transformed normal variable $X = \mu + \sigma Z$:


$$\text{Var}(X) = \text{Var}(\mu + \sigma Z) = \sigma^2 \text{Var}(Z) = \sigma^2(1) = \sigma^2$$

---

## Lesson 5: Deriving the General Normal PDF via Standardization

At the very bottom of the slide, we derive the famous PDF formula for any $X \sim \mathcal{N}(\mu, \sigma^2)$.

**Step 1: Standardize $X$ back into $Z$**
If $X = \mu + \sigma Z$, then rearranging for $Z$ gives the **Z-score (standardization)**:


$$Z = \frac{X - \mu}{\sigma}$$

**Step 2: Express the CDF of $X$ using $\Phi$**


$$F_X(x) = P(X \le x) = P\left( \frac{X - \mu}{\sigma} \le \frac{x - \mu}{\sigma} \right) = P\left( Z \le \frac{x - \mu}{\sigma} \right) = \Phi\left( \frac{x - \mu}{\sigma} \right)$$

**Step 3: Differentiate the CDF to get the PDF (using the Chain Rule)**
To find $f_X(x)$, we take the derivative of $\Phi\left(\frac{x-\mu}{\sigma}\right)$ with respect to $x$:

* The derivative of the outside function $\Phi'(z)$ is the standard normal PDF $\frac{1}{\sqrt{2\pi}} e^{-z^2/2}$.
* The derivative of the inside function $\frac{d}{dx}\left(\frac{x-\mu}{\sigma}\right)$ is $\frac{1}{\sigma}$.

Multiplying them together by the **Chain Rule** gives the universal **Normal PDF**:


$$f_X(x) = \frac{1}{\sigma \sqrt{2\pi}} e^{-\frac{1}{2} \left( \frac{x - \mu}{\sigma} \right)^2}$$
# --- Lecture 14 ---

Bhai, welcome to **Lecture 14**! This is a super important lecture that ties everything together: we first prove the **Variances of Poisson and Binomial** distributions, and then we introduce the **Exponential Distribution**—the continuous cousin of the Geometric distribution—along with its famous **Memoryless Property**.

Let’s break down this entire slide from scratch, lesson by lesson, so you understand every single formula and derivation!

---

## Lesson 1: Variance of the Poisson Distribution ($\text{Var}(X) = \lambda$)

We already know from Lecture 11 that if $X \sim \text{Pois}(\lambda)$, its mean is $E(X) = \lambda$. Here, we prove a remarkable fact: **Its variance is also $\lambda$!**

### 1. Using the Variance Shortcut

$$\text{Var}(X) = E(X^2) - (E(X))^2 = E(X^2) - \lambda^2$$


To find $\text{Var}(X)$, we first need to calculate the second moment, $E(X^2)$:


$$E(X^2) = \sum_{k=0}^{\infty} k^2 P(X = k) = e^{-\lambda} \sum_{k=0}^{\infty} k^2 \frac{\lambda^k}{k!}$$

---

### 2. The Calculus Series Trick (Right-Side Box)

How do we sum $\sum_{k=0}^{\infty} k^2 \frac{\lambda^k}{k!}$? We use derivatives on the Taylor series of $e^\lambda$:

1. **Start with the standard Taylor series:**

$$\sum_{k=0}^{\infty} \frac{\lambda^k}{k!} = e^\lambda$$


2. **Take the 1st derivative with respect to $\lambda$:**

$$\sum_{k=0}^{\infty} k \frac{\lambda^{k-1}}{k!} = e^\lambda \implies \sum_{k=0}^{\infty} k \frac{\lambda^k}{k!} = \lambda e^\lambda \quad \text{(after multiplying both sides by }\lambda\text{)}$$


3. **Take the 2nd derivative with respect to $\lambda$ (using the Product Rule on $\lambda e^\lambda$):**

$$\sum_{k=0}^{\infty} k^2 \frac{\lambda^{k-1}}{k!} = (1 \cdot e^\lambda + \lambda e^\lambda) = e^\lambda(1 + \lambda)$$


4. **Multiply both sides by $\lambda$ to get $k^2 \lambda^k$ inside:**

$$\sum_{k=0}^{\infty} k^2 \frac{\lambda^k}{k!} = \lambda e^\lambda(1 + \lambda)$$



---

### 3. Plugging Back to Find $\text{Var}(X)$

Substitute that series result back into $E(X^2)$:


$$E(X^2) = e^{-\lambda} \left[ \lambda e^\lambda (1 + \lambda) \right] = \lambda(1 + \lambda) = \lambda + \lambda^2$$

Now use the shortcut formula:


$$\text{Var}(X) = E(X^2) - (E(X))^2 = (\lambda + \lambda^2) - \lambda^2 = \lambda$$


*(Incredible result: For any Poisson distribution, **Mean = Variance = $\lambda$**!)*

---

## Lesson 2: Variance of the Binomial Distribution ($\text{Var}(X) = npq$)

Next, we prove that if $X \sim \text{Bin}(n, p)$, its variance is **$npq$** (where $q = 1-p$) using **indicator variables**.

### 1. Setting up $X^2$ using Indicators

Let $X = I_1 + I_2 + \dots + I_n$, where each $I_j \sim \text{Bern}(p)$ is an i.i.d. indicator for trial $j$.
When we square the sum $(I_1 + I_2 + \dots + I_n)^2$, we get two types of terms:


$$X^2 = \underbrace{\sum_{i=1}^{n} I_i^2}_{\text{n squared terms}} + \underbrace{2 \sum_{i < j} I_i I_j}_{\binom{n}{2} \text{ cross-product pairs}}$$

---

### 2. Taking Expected Value $E(X^2)$ by Linearity

$$E(X^2) = n E(I_1^2) + 2 \binom{n}{2} E(I_1 I_2)$$

* **For a squared indicator $I_1^2$:** Because an indicator only takes values $0$ or $1$ (and $0^2=0$, $1^2=1$), we have $I_1^2 = I_1$. Therefore:

$$E(I_1^2) = E(I_1) = p$$


* **For a cross-product $I_1 I_2$:** $I_1 I_2 = 1$ only when **both** trial 1 and trial 2 are successes. Since independent trials multiply:

$$E(I_1 I_2) = P(I_1 = 1 \text{ and } I_2 = 1) = p \cdot p = p^2$$


* **Now substitute:**

$$E(X^2) = n(p) + 2 \left[ \frac{n(n-1)}{2} \right] p^2 = np + n(n-1)p^2 = np + n^2 p^2 - np^2$$



---

### 3. Finding $\text{Var}(X)$

$$\text{Var}(X) = E(X^2) - (E(X))^2 = (np + n^2p^2 - np^2) - (np)^2$$


Since $(np)^2 = n^2 p^2$, those cancel out:


$$\text{Var}(X) = np - np^2 = np(1 - p) = npq$$

---

## Lesson 3: The Exponential Distribution (Waiting for the First Event)

How do we transition from counting events (Poisson) to measuring **time until the first event**?

### 1. The Story: Emails Arriving in Time $t$

* Suppose emails arrive according to a Poisson process with rate $\lambda$ emails per unit time.
* In a time window of length $t$, the number of emails $N_t$ follows:

$$N_t \sim \text{Pois}(\lambda t)$$


* Let $T$ be the **continuous time you wait until the very first email arrives**. What is the CDF and PDF of $T$?

---

### 2. Deriving the Exponential CDF & PDF

Instead of finding $P(T \le t)$ directly, it is much easier to look at the **complement**:

* What does $T > t$ mean in plain English? It means **you waited more than time $t$, so ZERO emails arrived between time $0$ and $t$!**

$$P(T > t) = P(N_t = 0)$$


* Using the Poisson PMF formula for $k = 0$:

$$P(N_t = 0) = \frac{e^{-\lambda t} (\lambda t)^0}{0!} = e^{-\lambda t}$$


* Therefore, the **Cumulative Distribution Function (CDF)** of waiting time $T$ is:

$$F(t) = P(T \le t) = 1 - P(T > t) = 1 - e^{-\lambda t}, \quad \text{for } t > 0$$


* To get the **Probability Density Function (PDF)**, take the derivative of the CDF with respect to $t$:

$$f(t) = F'(t) = \frac{d}{dt}\left(1 - e^{-\lambda t}\right) = \lambda e^{-\lambda t}, \quad \text{for } t > 0$$



This defines the **Exponential Distribution**: $X \sim \text{Expo}(\lambda)$.

---

## Lesson 4: Mean & Variance of $\text{Expo}(\lambda)$ via Scaling

Instead of doing difficult calculus with $\lambda$ inside the integral, the notes use a brilliant scaling trick!

### 1. Standard Exponential $Y \sim \text{Expo}(1)$

Let $Y = \lambda X$. Let's check the CDF of $Y$:


$$P(Y \le y) = P(\lambda X \le y) = P\left(X \le \frac{y}{\lambda}\right) = 1 - e^{-\lambda (y/\lambda)} = 1 - e^{-y}$$


This proves that $Y \sim \text{Expo}(1)$ (an exponential distribution with rate $\lambda = 1$, where PDF $f(y) = e^{-y}$).

---

### 2. Mean and Variance of $Y \sim \text{Expo}(1)$ (Integration by Parts)

* **Mean $E(Y)$:**

$$E(Y) = \int_0^{\infty} y e^{-y} \, dy = 1$$


* **Second Moment $E(Y^2)$ using Integration by Parts (Right-Side Scratchwork):**

$$\int y e^{-y} dy = -y e^{-y} + \int e^{-y} dy = -y e^{-y} - e^{-y}$$



Evaluating $\int_0^{\infty} y^2 e^{-y} \, dy$ gives $E(Y^2) = 2$.
* **Variance $\text{Var}(Y)$:**

$$\text{Var}(Y) = E(Y^2) - (E(Y))^2 = 2 - 1^2 = 1$$



---

### 3. Scaling Back to find $E(X)$ and $\text{Var}(X)$ for $\text{Expo}(\lambda)$

Since $Y = \lambda X$, we can invert it: $X = \frac{Y}{\lambda}$.

* **Mean of $X \sim \text{Expo}(\lambda)$:**

$$E(X) = E\left(\frac{Y}{\lambda}\right) = \frac{1}{\lambda} E(Y) = \frac{1}{\lambda}(1) = \frac{1}{\lambda}$$



*(Makes total sense: If emails arrive at a rate of $\lambda = 5$ per hour, you expect to wait $\frac{1}{5}$ of an hour for the first one!)*
* **Variance of $X \sim \text{Expo}(\lambda)$:**

$$\text{Var}(X) = \text{Var}\left(\frac{Y}{\lambda}\right) = \frac{1}{\lambda^2} \text{Var}(Y) = \frac{1}{\lambda^2}(1) = \frac{1}{\lambda^2}$$



---

## Lesson 5: The Memoryless Property

At the very bottom of the slide, we prove the most famous property of the Exponential distribution: **It has no memory!**

### 1. What is the Memoryless Property?

In math terms, for any times $s, t \ge 0$:


$$P(X > s + t \mid X > s) = P(X > t)$$

* **In plain English:** Suppose you have already waited $s = 10$ minutes for a bus ($X > 10$). The probability that you will have to wait *an additional* $t = 5$ minutes is **exactly the same** as if you had just arrived at the bus stop fresh ($P(X > 5)$)! The bus doesn't "remember" how long you've been standing there.

---

### 2. The Algebraic Proof

Using the definition of conditional probability $P(A \mid B) = \frac{P(A \cap B)}{P(B)}$:

1. **Set up the conditional probability:**

$$P(X > s + t \mid X > s) = \frac{P(X > s + t \text{ and } X > s)}{P(X > s)}$$


2. **Simplify the intersection:**
If $X > s + t$, then it is *automatically* greater than $s$. So the intersection is simply $P(X > s + t)$:

$$= \frac{P(X > s + t)}{P(X > s)}$$


3. **Plug in the survival probability $P(X > x) = e^{-\lambda x}$:**

$$= \frac{e^{-\lambda(s + t)}}{e^{-\lambda s}} = \frac{e^{-\lambda s} \cdot e^{-\lambda t}}{e^{-\lambda s}} = e^{-\lambda t} = P(X > t)$$



Boom! The $e^{-\lambda s}$ terms cancel out completely, proving the Exponential distribution is **100% memoryless**!
# --- Lecture 15 ---

Bhai, welcome to **Lecture 15**! This lecture introduces one of the most powerful analytical tools in all of probability and statistics: **Moment Generating Functions (MGFs)**.

An MGF is like a mathematical DNA sequence for a random variable—it encodes *everything* about the distribution in a single function, allowing us to find moments easily and prove what happens when we add independent random variables together.

Let’s break down this entire slide from scratch, lesson by lesson, so you master every single proof and derivation!

---

## Lesson 1: What is an MGF and Why Do We Care?

### 1. What is a "Moment"?

In probability, the **$n\text{-th}$ moment** of a random variable $X$ is simply the expected value of $X$ raised to the power of $n$:


$$E(X^n)$$

* **1st Moment ($n=1$):** The Mean $E(X)$
* **2nd Moment ($n=2$):** Used to find Variance $\text{Var}(X) = E(X^2) - (E(X))^2$

---

### 2. Formal Definition of the MGF

The **Moment Generating Function** of a random variable $X$ is defined as the expected value of $e^{tX}$ as a function of an auxiliary variable $t$:


$$M(t) = E(e^{tX})$$


*(This is valid as long as the expectation is finite on some interval $(-a, a)$ around $0$ where $a > 0$.)*

---

### 3. Why Study MGFs? (The Three Superpowers)

Why do we define it using the exponential function $e^{tX}$? Look at the Taylor series expansion of $e^{tX}$:


$$E(e^{tX}) = E\left(\sum_{n=0}^{\infty} \frac{t^n X^n}{n!}\right) = \sum_{n=0}^{\infty} E(X^n) \frac{t^n}{n!}$$

This expansion gives MGFs **three superpowers** listed on the slide:

* **Superpower 1: It Generates Moments!**
The $n\text{-th}$ moment $E(X^n)$ is simply the **coefficient of $\frac{t^n}{n!}$** in the Taylor series expansion of $M(t)$.
Equivalently, if you take the $n\text{-th}$ derivative of $M(t)$ and plug in $t = 0$, you get the $n\text{-th}$ moment:

$$M^{(n)}(0) = E(X^n)$$


* **Superpower 2: Uniqueness (It Determines the Distribution)**
If two random variables $X$ and $Y$ have the **same MGF** ($M_X(t) = M_Y(t)$), then they must have the **exact same distribution** (the same PDF/CDF). An MGF uniquely identifies a distribution!
* **Superpower 3: Sums of Independent Variables Are Easy!**
If $X$ and $Y$ are **independent** random variables, the MGF of their sum is simply the **product** of their individual MGFs:

$$M_{X+Y}(t) = M_X(t) \cdot M_Y(t)$$


* **Proof:** Since $X$ and $Y$ are independent, $e^{tX}$ and $e^{tY}$ are also independent. Therefore:

$$M_{X+Y}(t) = E(e^{t(X+Y)}) = E(e^{tX} \cdot e^{tY}) = E(e^{tX}) \cdot E(e^{tY}) = M_X(t) M_Y(t)$$





---

## Lesson 2: Discrete Examples — Bernoulli & Binomial

Let’s see how fast we can find the MGFs for our favorite discrete distributions.

### 1. Bernoulli Distribution: $X \sim \text{Bern}(p)$

A Bernoulli random variable equals $1$ with probability $p$, and $0$ with probability $q = 1-p$.
Using the definition of expected value:


$$M(t) = E(e^{tX}) = e^{t \cdot 1} P(X=1) + e^{t \cdot 0} P(X=0) = p e^t + q \cdot 1 = p e^t + q$$

---

### 2. Binomial Distribution: $X \sim \text{Bin}(n, p)$

We know from Lecture 8 that a Binomial random variable is just the sum of $n$ independent $\text{Bern}(p)$ trials:


$$X = X_1 + X_2 + \dots + X_n$$


Using **Superpower 3** (the product rule for independent sums):


$$M_X(t) = M_{X_1}(t) \cdot M_{X_2}(t) \dots M_{X_n}(t) = (p e^t + q)^n$$


No messy binomial combinatorics needed—just one line of algebra!

---

## Lesson 3: Standard Normal $Z \sim \mathcal{N}(0,1)$ & All Its Moments

In the middle of the slide, we derive the MGF of the **Standard Normal** distribution and use it to unlock a famous formula for all normal moments.

### 1. Deriving $M(t) = e^{t^2/2}$ (Completing the Square)

$$M(t) = E(e^{tZ}) = \frac{1}{\sqrt{2\pi}} \int_{-\infty}^{+\infty} e^{tz} e^{-z^2/2} \, dz = \frac{1}{\sqrt{2\pi}} \int_{-\infty}^{+\infty} e^{-\frac{1}{2}(z^2 - 2tz)} \, dz$$

Let's **complete the square** in the exponent:


$$z^2 - 2tz = (z^2 - 2tz + t^2) - t^2 = (z - t)^2 - t^2$$


Substitute this back into the integral:


$$M(t) = \frac{1}{\sqrt{2\pi}} \int_{-\infty}^{+\infty} e^{-\frac{1}{2}\left[(z - t)^2 - t^2\right]} \, dz = e^{t^2/2} \cdot \left[ \frac{1}{\sqrt{2\pi}} \int_{-\infty}^{+\infty} e^{-\frac{1}{2}(z - t)^2} \, dz \right]$$

Look at the term inside the brackets: that is simply the integral over the entire real line of a Normal PDF with mean $t$ and variance $1$, which must equal **$1$**! Therefore:


$$M(t) = e^{t^2/2}$$

---

### 2. Finding All Moments of $Z \sim \mathcal{N}(0, 1)$

Now let's use **Superpower 1** to find $E(Z^n)$ for any integer $n$:

* **Odd Moments ($n = 1, 3, 5, \dots$):**
By symmetry of the bell curve around $0$, every odd moment is exactly zero:

$$E(Z^n) = 0 \quad \text{for all odd } n$$


* **Even Moments ($n = 2, 4, 6, \dots$):**
Let's expand $M(t) = e^{t^2/2}$ using the Taylor series for $e^x$:

$$M(t) = \sum_{n=0}^{\infty} \frac{(t^2/2)^n}{n!} = \sum_{n=0}^{\infty} \frac{t^{2n}}{2^n n!}$$



We want to match this with the standard MGF series form $\sum \text{Coefficient} \times \frac{t^{2n}}{(2n)!}$. Let's multiply and divide by $(2n)!$:

$$M(t) = \sum_{n=0}^{\infty} \left[ \frac{(2n)!}{2^n n!} \right] \frac{t^{2n}}{(2n)!}$$



Because the coefficient of $\frac{t^{2n}}{(2n)!}$ is the $(2n)\text{-th}$ moment, we get the legendary formula for even moments of a standard normal:

$$E(Z^{2n}) = \frac{(2n)!}{2^n n!}$$



*(For example, $E(Z^2) = \frac{2!}{2^1 \cdot 1!} = 1$, and $E(Z^4) = \frac{4!}{2^2 \cdot 2!} = \frac{24}{8} = 3$.)*

---

## Lesson 4: Exponential Distribution & Its Moments ($n!$)

Next, we look at the continuous waiting time distribution from Lecture 14: the Exponential distribution.

### 1. Standard Exponential: $X \sim \text{Expo}(1)$

Here, the PDF is $f(x) = e^{-x}$ for $x > 0$. Let's integrate to find $M(t)$:


$$M(t) = E(e^{tX}) = \int_0^{\infty} e^{tx} e^{-x} \, dx = \int_0^{\infty} e^{-x(1 - t)} \, dx$$


For $t < 1$, this integral converges to:


$$M(t) = \left[ -\frac{1}{1 - t} e^{-x(1 - t)} \right]_0^{\infty} = 0 - \left(-\frac{1}{1 - t}\right) = \frac{1}{1 - t}, \quad \text{for } t < 1$$

---

### 2. Finding $E(X^n)$ via Geometric Series

How do we expand $\frac{1}{1 - t}$ into a Taylor series? It is simply the **Geometric Series**:


$$\frac{1}{1 - t} = \sum_{n=0}^{\infty} t^n = 1 + t + t^2 + t^3 + \dots$$


Let's rewrite each term $t^n$ so it has $\frac{t^n}{n!}$ at the end:


$$t^n = n! \cdot \frac{t^n}{n!} \implies M(t) = \sum_{n=0}^{\infty} n! \frac{t^n}{n!}$$


Matching the coefficient of $\frac{t^n}{n!}$ reveals that for $X \sim \text{Expo}(1)$:


$$E(X^n) = n! \quad \text{for any integer } n \ge 0$$

---

### 3. Scaling to General Exponential: $Y \sim \text{Expo}(\lambda)$

We know from Lecture 14 that if $Y \sim \text{Expo}(\lambda)$ and $X \sim \text{Expo}(1)$, then $Y = \frac{X}{\lambda}$.
Therefore, raising both sides to the $n\text{-th}$ power:


$$Y^n = \frac{X^n}{\lambda^n} \implies E(Y^n) = \frac{E(X^n)}{\lambda^n} = \frac{n!}{\lambda^n}$$

---

## Lesson 5: Poisson MGF & Adding Two Independent Poissons

At the very bottom of the slide, we see a beautiful proof of why **the sum of two independent Poisson random variables is still Poisson**.

### 1. Deriving the Poisson MGF: $X \sim \text{Pois}(\lambda)$

Using the Poisson PMF $P(X=k) = \frac{e^{-\lambda} \lambda^k}{k!}$:


$$M_X(t) = E(e^{tX}) = \sum_{k=0}^{\infty} e^{tk} \frac{e^{-\lambda} \lambda^k}{k!} = e^{-\lambda} \sum_{k=0}^{\infty} \frac{(\lambda e^t)^k}{k!}$$


Recognize the series? $\sum_{k=0}^{\infty} \frac{(\lambda e^t)^k}{k!}$ is the Taylor series for $e^{\lambda e^t}$! Therefore:


$$M_X(t) = e^{-\lambda} e^{\lambda e^t} = e^{\lambda(e^t - 1)}$$

---

### 2. Proving $X + Y \sim \text{Pois}(\lambda + \mu)$

Suppose $X \sim \text{Pois}(\lambda)$ and $Y \sim \text{Pois}(\mu)$ are **independent**. What is the distribution of their sum $X + Y$?

Let's multiply their MGFs using **Superpower 3**:


$$M_{X+Y}(t) = M_X(t) \cdot M_Y(t) = e^{\lambda(e^t - 1)} \cdot e^{\mu(e^t - 1)}$$


Combine the exponents:


$$M_{X+Y}(t) = e^{(\lambda + \mu)(e^t - 1)}$$

Look at the result: this is exactly the MGF of a Poisson distribution with rate parameter **$(\lambda + \mu)$**!

By **Superpower 2 (Uniqueness of MGFs)**, since $X + Y$ has a Poisson MGF, it **must** follow a Poisson distribution:


$$X + Y \sim \text{Pois}(\lambda + \mu)$$