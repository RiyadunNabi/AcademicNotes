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