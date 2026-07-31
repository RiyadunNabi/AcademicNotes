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