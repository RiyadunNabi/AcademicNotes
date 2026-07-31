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