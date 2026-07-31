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