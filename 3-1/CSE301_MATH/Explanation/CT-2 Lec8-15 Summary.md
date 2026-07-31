# High-Yield Exam Cheatsheet — Lectures 8 to 15

All distributions categorized by type, followed by essential probability laws, variance shortcuts, and MGF rules.

---

## 1. Discrete Distributions Cheatsheet

Discrete distributions count specific outcomes over trials or events.

| Distribution | Story / Meaning | PMF | Mean $E(X)$ | Variance $\text{Var}(X)$ | MGF $M(t)$ |
| --- | --- | --- | --- | --- | --- |
| **Bernoulli** $\text{Bern}(p)$ | Single trial with success probability $p$. | $P(X=1)=p$, $P(X=0)=q$ (where $q=1-p$) | $p$ | $pq$ | $pe^t + q$ |
| **Binomial** $\text{Bin}(n,p)$ | Total successes in $n$ independent Bernoulli trials. | $P(X=k)=\binom{n}{k}p^k q^{n-k}$, $k\in\{0,\dots,n\}$ | $np$ | $npq$ | $(pe^t+q)^n$ |
| **Hypergeometric** | Successes in sample of $n$ drawn **without replacement** from $w$ white and $b$ black marbles. | $P(X=k)=\dfrac{\binom{w}{k}\binom{b}{n-k}}{\binom{b+w}{n}}$ | — | — | — |
| **Geometric** $\text{Geom}(p)$ | Number of independent **failures before** the 1st success. | $P(X=k)=q^k p$, $k\in\{0,1,2,\dots\}$ | $\frac{q}{p}$ | — | — |
| **First Success** $FS(p)$ | Total flips **until (and including)** the 1st success ($X=Y+1$). | $P(X=k)=q^{k-1}p$, $k\in\{1,2,3,\dots\}$ | $\frac{1}{p}$ | — | — |
| **Negative Binomial** | Number of independent **failures before** the $r$-th success. | $P(X=n)=\binom{n+r-1}{r-1}p^r q^n$, $n\in\{0,1,2,\dots\}$ | $\frac{rq}{p}$ | — | — |
| **Poisson** $\text{Pois}(\lambda)$ | Count of **rare events** occurring at rate $\lambda$. | $P(X=k)=\dfrac{e^{-\lambda}\lambda^k}{k!}$, $k\in\{0,1,2,\dots\}$ | $\lambda$ | $\lambda$ | $e^{\lambda(e^t-1)}$ |

### Key Discrete Properties & Additivity Rules

- **Sum of Independent Binomials:** If $X\sim\text{Bin}(n,p)$ and $Y\sim\text{Bin}(m,p)$ are independent, then $X+Y\sim\text{Bin}(n+m,p)$.
- **Sum of Independent Poissons:** If $X\sim\text{Pois}(\lambda)$ and $Y\sim\text{Pois}(\mu)$ are independent, then $X+Y\sim\text{Pois}(\lambda+\mu)$.
- **Poisson Paradigm (Law of Rare Events):** When $n$ is very large, each event probability $p_j$ is very small, and events are weakly dependent or independent, the count of events is approximately $\text{Pois}(\lambda)$ where $\lambda\approx\sum_{j=1}^{n}p_j$.
- **Binomial to Poisson Limit:** As $n\to\infty$ and $p\to 0$ while keeping $\lambda=np$ constant, $\text{Bin}(n,p)\to\text{Pois}(\lambda)$.

---

## 2. Continuous Distributions Cheatsheet

Continuous distributions measure intervals over continuous scales, where single-point probabilities are always zero ($P(X=a)=0$).

| Distribution | Story / Meaning | PDF | Mean $E(X)$ | Variance $\text{Var}(X)$ | MGF $M(t)$ |
| --- | --- | --- | --- | --- | --- |
| **Uniform** $\text{Unif}(a,b)$ | Random point in $[a,b]$ where equal intervals have equal probability. | $f(x)=\frac{1}{b-a}$, $a\le x\le b$ | $\frac{a+b}{2}$ | $\frac{(b-a)^2}{12}$ | — |
| **Standard Normal** $Z\sim\mathcal{N}(0,1)$ | Bell curve with mean 0 and variance 1. | $f(z)=\frac{1}{\sqrt{2\pi}}e^{-z^2/2}$ | **0** | **1** | $e^{t^2/2}$ |
| **General Normal** $X\sim\mathcal{N}(\mu,\sigma^2)$ | Scaled and shifted bell curve ($X=\mu+\sigma Z$). | $f_X(x)=\frac{1}{\sigma\sqrt{2\pi}}e^{-\frac12\left(\frac{x-\mu}{\sigma}\right)^2}$ | $\mu$ | $\sigma^2$ | — |
| **Exponential** $\text{Expo}(\lambda)$ | Continuous waiting time until the **1st event** of a Poisson process. | $f(t)=\lambda e^{-\lambda t}$, $t>0$ | $\frac{1}{\lambda}$ | $\frac{1}{\lambda^2}$ | $\frac{1}{1-t}$ (for $\lambda=1$) |

### Key Continuous Properties

- **Standard Normal CDF:** $\Phi(z)=\frac{1}{\sqrt{2\pi}}\int_{-\infty}^{z}e^{-t^2/2}\,dt$.
- **Standardization (Z-score):** $Z=\frac{X-\mu}{\sigma}$.
- **Normal Moments:** For $Z\sim\mathcal{N}(0,1)$, all odd moments equal **0**. Even moments: $E(Z^{2n})=\frac{(2n)!}{2^n n!}$.
- **Exponential CDF & Survival:** $F(t)=1-e^{-\lambda t}$. Survival: $P(T>t)=e^{-\lambda t}$.
- **Exponential Moments:** For $X\sim\text{Expo}(1)$, $E(X^n)=n!$. For $Y\sim\text{Expo}(\lambda)$, $E(Y^n)=\frac{n!}{\lambda^n}$.
- **Memoryless Property (Exponential):** For any $s,t\ge0$, $P(X>s+t\mid X>s)=P(X>t)$.

---

## 3. Essential Probability Rules & Theorems

### Expectation & The Fundamental Bridge

- **Linearity of Expectation:** $E(X+Y)=E(X)+E(Y)$ holds **even if $X$ and $Y$ are dependent**.
- **Scaling Expectation:** $E(cX)=cE(X)$.
- **Fundamental Bridge:** For an indicator variable $X$ of event $A$ ($X=1$ if $A$ occurs, 0 otherwise), $P(A)=E(X)$.
- **LOTUS (Law of the Unconscious Statistician):** $E(g(X))=\int_{-\infty}^{+\infty}g(x)f_X(x)\,dx$.

### Variance & Standard Deviation Shortcuts

- **Variance Shortcut:** $\text{Var}(X)=E(X^2)-(E(X))^2$.
- **Standard Deviation:** $\text{SD}(X)=\sqrt{\text{Var}(X)}$.
- **Shifting & Scaling Variance:** $\text{Var}(X+c)=\text{Var}(X)$. $\text{Var}(cX)=c^2\text{Var}(X)$.
- **Variance of a Sum:** $\text{Var}(X+Y)=\text{Var}(X)+\text{Var}(Y)$ **only if $X$ and $Y$ are independent**.

### Continuous Probability Calculus

- **PDF from CDF:** $f(x)=F'(x)=\frac{d}{dx}F(x)$.
- **CDF from PDF:** $F(x)=\int_{-\infty}^{x}f(t)\,dt$.
- **Interval Probabilities:** $P(a\le X\le b)=F(b)-F(a)=\int_{a}^{b}f(x)\,dx$.
- **Total Area Rule:** $\int_{-\infty}^{+\infty}f(x)\,dx=1$.

---

## 4. Moment Generating Functions (MGFs)

- **Definition:** $M(t)=E(e^{tX})=\sum_{n=0}^{\infty}E(X^n)\frac{t^n}{n!}$.
- **Generating Moments:** The $n$-th moment is the coefficient of $\frac{t^n}{n!}$ in the Taylor expansion of $M(t)$, equivalently $M^{(n)}(0)=E(X^n)$.
- **Uniqueness:** Same MGF $\implies$ same distribution.
- **Independent Sums:** If $X,Y$ independent, $M_{X+Y}(t)=M_X(t)M_Y(t)$.

---

## 5. Must-Know Famous Identities & Paradoxes

- **Vandermonde's Identity:** $\sum_{j=0}^{k}\binom{m}{k-j}\binom{n}{j}=\binom{m+n}{k}$.
- **Euler Limit Formula:** $\lim_{n\to\infty}\left(1+\frac{x}{n}\right)^n=e^x$.
- **Central Limit Theorem (CLT):** The sum of a large number of i.i.d. random variables from any distribution approaches a normal distribution.
- **St. Petersburg Paradox:** A coin game paying $\$2^X$ on flip $X$ has an infinite expected payout. Capping the max payout at $\$2^{30}$ reduces the expected payout to **30**.