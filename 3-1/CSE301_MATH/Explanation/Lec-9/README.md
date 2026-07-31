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