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