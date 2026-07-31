# CSE 301 — Worked Solutions for Tomorrow's CT
Organized by topic (Discrete → Continuous → Rules → MGF). Each question is stated exactly as given, followed by a formula-first solution.

---

# 1. Discrete Distributions

## Binomial

### Q3(a) [09/09/2025] (10)
> Find the mean and variance of a binomial random variable.

**Setup:** $X\sim \text{Bin}(n,p)$, $P(X=k)=\binom{n}{k}p^kq^{n-k}$, $q=1-p$.

**Mean:**
$$E(X)=\sum_{k=0}^{n}k\binom{n}{k}p^kq^{n-k}=\sum_{k=1}^{n}n\binom{n-1}{k-1}p^kq^{n-k}$$
$$=np\sum_{j=0}^{n-1}\binom{n-1}{j}p^jq^{n-1-j}=np(p+q)^{n-1}=np$$

**Variance** (via $E[X(X-1)]$):
$$E[X(X-1)]=\sum_{k=0}^n k(k-1)\binom{n}{k}p^kq^{n-k}=n(n-1)p^2\sum_{j=0}^{n-2}\binom{n-2}{j}p^jq^{n-2-j}=n(n-1)p^2$$
$$E(X^2)=E[X(X-1)]+E(X)=n(n-1)p^2+np$$
$$\text{Var}(X)=E(X^2)-[E(X)]^2=n(n-1)p^2+np-n^2p^2=np-np^2$$
$$\boxed{E(X)=np,\qquad \text{Var}(X)=np(1-p)=npq}$$

---

### Q2 [30/04/2023] (15)
> Prove that when the number of trials n in a binomial distribution is large and the probability of success p is very small, the binomial distribution can be approximated by a Poisson distribution with mean λ = np.

**Setup:** Let $\lambda=np$ fixed, $n\to\infty$, $p=\lambda/n\to0$.
$$P(X=k)=\binom{n}{k}p^k(1-p)^{n-k}=\frac{n!}{k!(n-k)!}\left(\frac{\lambda}{n}\right)^k\left(1-\frac{\lambda}{n}\right)^{n-k}$$

**Rearrange:**
$$P(X=k)=\frac{\lambda^k}{k!}\cdot\underbrace{\frac{n(n-1)\cdots(n-k+1)}{n^k}}_{A}\cdot\underbrace{\left(1-\frac{\lambda}{n}\right)^{n}}_{B}\cdot\underbrace{\left(1-\frac{\lambda}{n}\right)^{-k}}_{C}$$

**Take limits as $n\to\infty$:**
$$A\to 1,\qquad B\to e^{-\lambda}\ \text{(Euler limit)},\qquad C\to 1$$

$$\boxed{P(X=k)\to \frac{e^{-\lambda}\lambda^k}{k!}}\quad\Rightarrow\quad X\to \text{Pois}(\lambda)$$

---

### Q1(c) [25/03/2019], Q5(a) [05/08/2017], Q5(a) [18/02/2018] — DVD problem (identical, 3+ appearances)
> It is known that DVDs produced by a certain company will be defective with probability 0.01, independent of each other. The company sells the DVDs in packages of size 10 and offers a money-back guarantee that if at least 1 of the 10 DVDs in a package is defective, money will be returned. If someone buys 3 packages, what is the probability that he or she will return exactly 1 of them?

**Step 1 — probability a single package is returned:**
$$P(\text{return}) = 1-P(\text{no defects in 10}) = 1-(1-0.01)^{10}$$
$$=1-(0.99)^{10}=1-0.904382=0.095618$$

**Step 2 — 3 packages, Binomial(3, p=0.095618), want exactly 1 returned:**
$$P(Y=1)=\binom{3}{1}p(1-p)^{2}$$
$$=3(0.095618)(0.904382)^2=3(0.095618)(0.817908)$$
$$\boxed{P(Y=1)\approx 0.2346}$$

---

### Q5(b) [05/08/2017] — Airline overbooking
> An airline knows that 5 percent of the people making reservations on a certain flight will not show up. Consequently, their policy is to sell 52 tickets for a flight that can hold only 50 passengers. What is the probability that there will be a seat available for every passenger who shows up?

**Setup:** Let $Y$ = number of no-shows, $Y\sim\text{Bin}(52,0.05)$. Seats suffice iff $Y\ge 2$ (since at most 50 can show up). Approximate with Poisson, $\lambda=np=52(0.05)=2.6$.

**Formula:**
$$P(Y\ge2)=1-P(Y=0)-P(Y=1)=1-e^{-\lambda}-\lambda e^{-\lambda}$$

**Plug in $\lambda=2.6$:**
$$e^{-2.6}=0.074274$$
$$P(Y=0)=0.074274,\qquad P(Y=1)=2.6(0.074274)=0.193112$$
$$P(Y\ge2)=1-0.074274-0.193112$$
$$\boxed{P(Y\ge2)\approx 0.7326}$$

---

### Q8(b) [18/02/2018], [05/08/2017] — MGF of Binomial + sum of independent binomials (13)
> Define moments of a Random Variable. How can we obtain them from moment generating function? Derive the moment generating function for the binomial distribution with parameters n and p. If X and Y are independent binomial random variables with parameters (n,p) and (m,p), respectively, then what is the distribution of X+Y.

**Moments:** the $n$-th moment of $X$ is $E(X^n)$. From the MGF $M(t)=E(e^{tX})$:
$$M^{(n)}(0)=E(X^n)$$
(the $n$-th derivative of $M(t)$ at $t=0$ gives the $n$-th moment.)

**MGF derivation** for $X\sim\text{Bin}(n,p)$:
$$M_X(t)=E(e^{tX})=\sum_{k=0}^{n}e^{tk}\binom{n}{k}p^kq^{n-k}=\sum_{k=0}^n\binom{n}{k}(pe^t)^kq^{n-k}$$
$$\boxed{M_X(t)=(pe^t+q)^n}\quad\text{(Binomial theorem)}$$

**Sum of independent binomials** $X\sim\text{Bin}(n,p)$, $Y\sim\text{Bin}(m,p)$:
$$M_{X+Y}(t)=M_X(t)M_Y(t)=(pe^t+q)^n(pe^t+q)^m=(pe^t+q)^{n+m}$$
This is exactly the MGF of $\text{Bin}(n+m,p)$. By uniqueness of MGFs:
$$\boxed{X+Y\sim \text{Bin}(n+m,\,p)}$$

---

## Negative Binomial & Poisson (expectations)

### Q3(b) [09/09/2025] (10)
> Find the expected values of negative binomial and Poisson random variables.

**Negative Binomial** ($X$ = number of failures before the $r$-th success, param $p$): write $X=X_1+\cdots+X_r$ where each $X_i\sim\text{Geom}(p)$ (failures between successive successes), $E(X_i)=q/p$.
$$E(X)=\sum_{i=1}^{r}E(X_i)=r\cdot\frac{q}{p}$$
$$\boxed{E(X)=\frac{rq}{p}}$$

**Poisson** $X\sim\text{Pois}(\lambda)$:
$$E(X)=\sum_{k=0}^{\infty}k\frac{e^{-\lambda}\lambda^k}{k!}=\lambda e^{-\lambda}\sum_{k=1}^{\infty}\frac{\lambda^{k-1}}{(k-1)!}=\lambda e^{-\lambda}e^{\lambda}$$
$$\boxed{E(X)=\lambda}$$

---

## Hypergeometric

### Q5(a) [10/01/2021] (15)
> Twenty tickets are sold in a lottery, numbered 1 to 20, inclusive. Five tickets are drawn for prizes. Find out the probability that two of the five winning tickets are in the range between 1 to 5, two are in the range between 6 to 10, and one is in the range between 11 to 20.

**Formula (multivariate hypergeometric):**
$$P=\frac{\binom{5}{2}\binom{5}{2}\binom{10}{1}}{\binom{20}{5}}$$

**Plug in:**
$$\binom{5}{2}=10,\quad \binom{5}{2}=10,\quad\binom{10}{1}=10,\quad \binom{20}{5}=15504$$
$$P=\frac{10\times10\times10}{15504}=\frac{1000}{15504}$$
$$\boxed{P=\frac{125}{1938}\approx0.0645}$$

---

## Poisson

### Q2(a) [09/09/2025] (10)
> It is observed that 0.2% bolts produced in a machine shop are faulty. The bolts are supplied in packets of 100. If a batch has 2000 packets supplied, what is the approximate number of packets with faulty bolts?

**Step 1:** Per packet, $\lambda=np=100(0.002)=0.2$.

**Step 2 — probability a packet has $\ge1$ faulty bolt:**
$$P(\text{faulty packet})=1-P(X=0)=1-e^{-\lambda}=1-e^{-0.2}$$
$$=1-0.818731=0.181269$$

**Step 3 — expected number of such packets out of 2000:**
$$E(\#\text{packets})=2000\times0.181269$$
$$\boxed{\approx 362.5\ (\text{about }363\text{ packets})}$$

---

### Q4(b) [09/09/2025] (10)
> Use Moment Generating Functions (MGFs) to find the distribution of the sum of two independent Poisson random variables.

**MGF of Poisson($\lambda$):**
$$M_X(t)=E(e^{tX})=\sum_{k=0}^\infty e^{tk}\frac{e^{-\lambda}\lambda^k}{k!}=e^{-\lambda}\sum_{k=0}^\infty\frac{(\lambda e^t)^k}{k!}=e^{-\lambda}e^{\lambda e^t}$$
$$\boxed{M_X(t)=e^{\lambda(e^t-1)}}$$

**Sum:** $X\sim\text{Pois}(\lambda_1)$, $Y\sim\text{Pois}(\lambda_2)$ independent:
$$M_{X+Y}(t)=M_X(t)M_Y(t)=e^{\lambda_1(e^t-1)}e^{\lambda_2(e^t-1)}=e^{(\lambda_1+\lambda_2)(e^t-1)}$$
This is the MGF of $\text{Pois}(\lambda_1+\lambda_2)$, so by uniqueness:
$$\boxed{X+Y\sim\text{Pois}(\lambda_1+\lambda_2)}$$

---

### Q1(b) [12/01/2025] (10) — Typist problem
> A manuscript is sent to a typing firm consisting of typists A, B, and C. If it is typed by A, then the number of errors made is a Poisson random variable with mean 2.6; if typed by B, then the number of errors is a Poisson random variable with mean 3; and if typed by C, then it is a Poisson random variable with mean 3.4. Let X denote the number of errors in the typed manuscript. Assume that each typist is equally likely to do the work.
> i) Find E[X]  ii) Find Var(X)

**Setup:** $T\in\{A,B,C\}$ each w.p. $1/3$; $X\mid T\sim\text{Pois}(\lambda_T)$, $\lambda_A=2.6,\lambda_B=3,\lambda_C=3.4$.

**(i) Law of total expectation:**
$$E(X)=E[E(X\mid T)]=\frac13(2.6+3+3.4)=\frac{9}{3}$$
$$\boxed{E(X)=3}$$

**(ii) Law of total variance:**
$$\text{Var}(X)=E[\text{Var}(X\mid T)]+\text{Var}[E(X\mid T)]$$

*First term* (Poisson: Var = mean):
$$E[\text{Var}(X\mid T)]=\frac13(2.6+3+3.4)=3$$

*Second term:* $E(X\mid T)$ takes values $2.6,3,3.4$ each w.p. $1/3$, mean $3$:
$$\text{Var}[E(X\mid T)]=\frac13\left[(2.6-3)^2+(3-3)^2+(3.4-3)^2\right]=\frac13[0.16+0+0.16]=0.1067$$

$$\boxed{\text{Var}(X)=3+0.1067=3.1067}$$

---

### Q2(a) [22/03/2022] (8) — Traffic accidents
> The number of traffic accidents on any given day is a Poisson random variable with mean 2, and these random variables for different days are independent.
> (i) What is the probability that there is a total of six accidents over two days?
> (ii) Take some 5 days, for instance, SAT-WED this week. What is the probability that at least three of these five days each have exactly two accidents?

**(i)** Sum over 2 independent days: $\text{Pois}(2)+\text{Pois}(2)=\text{Pois}(4)$.
$$P(N=6)=\frac{e^{-4}4^6}{6!}=\frac{0.0183156\times4096}{720}$$
$$\boxed{P(N=6)\approx0.1042}$$

**(ii)** Probability a single day has exactly 2 accidents:
$$p=P(\text{Pois}(2)=2)=\frac{e^{-2}2^2}{2!}=e^{-2}\times2=0.2707$$

Let $Y=$ number of days (out of 5) with exactly 2 accidents, $Y\sim\text{Bin}(5,p)$, want $P(Y\ge3)$:
$$P(Y\ge3)=\binom{5}{3}p^3q^2+\binom{5}{4}p^4q+\binom{5}{5}p^5,\qquad q=0.7293$$

**Plug in** ($p=0.2707$): $p^3=0.01983,\ p^4=0.005368,\ p^5=0.001453,\ q^2=0.53192$
$$10(0.01983)(0.53192)+5(0.005368)(0.7293)+0.001453$$
$$=0.10549+0.01958+0.00145$$
$$\boxed{P(Y\ge3)\approx0.1265}$$

---

### Q2(c) [22/03/2022] (12) — Poisson thinning
> Let N be a Poisson random variable with mean λ. Given N, let X be another binomial random variable with parameters N and p. Let Y = N – X be a random variable and given that Y is non-negative. Prove that Y is a Poisson random variable with mean λ(1 – p).

**Setup:** $N\sim\text{Pois}(\lambda)$; $X\mid N=n\sim\text{Bin}(n,p)$; $Y=N-X$.

**Compute $P(Y=y)$ by conditioning on $N$:**
$$P(Y=y)=\sum_{n=y}^{\infty}P(N=n)\,P(X=n-y\mid N=n)$$
$$=\sum_{n=y}^{\infty}\frac{e^{-\lambda}\lambda^n}{n!}\binom{n}{y}(1-p)^yp^{n-y}$$

**Simplify** ($\binom{n}{y}=\frac{n!}{y!(n-y)!}$):
$$=\frac{e^{-\lambda}(1-p)^y}{y!}\sum_{n=y}^{\infty}\frac{\lambda^n p^{n-y}}{(n-y)!}$$

Let $m=n-y$:
$$=\frac{e^{-\lambda}(1-p)^y\lambda^y}{y!}\sum_{m=0}^\infty\frac{(\lambda p)^m}{m!}=\frac{e^{-\lambda}[\lambda(1-p)]^y}{y!}e^{\lambda p}$$

$$=\frac{e^{-\lambda(1-p)}[\lambda(1-p)]^y}{y!}$$

$$\boxed{Y\sim\text{Pois}(\lambda(1-p))}\qquad\blacksquare$$

---

### Q4(a) [22/03/2022] (8) — Bank customers
> Customers arrive at a bank at a Poisson rate λ. Suppose three customers arrived during the first hour. What is the probability that exactly two customers arrived during the first 20 minutes?

**Key fact:** given $N(1\text{ hr})=3$, arrival times are distributed as 3 independent points uniform on the hour. So the number landing in the first $20/60=1/3$ of the hour is:
$$X\mid N(1)=3 \ \sim\ \text{Bin}\!\left(3,\ \tfrac13\right)$$

**Plug in:**
$$P(X=2)=\binom{3}{2}\left(\frac13\right)^2\left(\frac23\right)^1=3\cdot\frac19\cdot\frac23=\frac{6}{27}$$
$$\boxed{P(X=2)=\frac29\approx0.2222}$$

---

### Q6(c) [18/02/2018] (10) — Store, compound Poisson
> The number of customers entering a store on a given day is Poisson distributed with mean λ = 10. The amount of money spent by a customer is uniformly distributed over (0, 100). Find the mean and variance of the amount of money that the store takes in on a given day.

**Setup:** $N\sim\text{Pois}(10)$, spend $Y_i\sim\text{Unif}(0,100)$ iid, $S=\sum_{i=1}^N Y_i$.
$$E(Y)=\frac{100}{2}=50,\qquad \text{Var}(Y)=\frac{100^2}{12}=833.33,\qquad E(Y^2)=\text{Var}(Y)+E(Y)^2=833.33+2500=3333.33$$

**Mean (Wald's identity):**
$$E(S)=E(N)E(Y)=10\times50$$
$$\boxed{E(S)=500}$$

**Variance (compound Poisson formula, since $\text{Var}(N)=E(N)=\lambda$):**
$$\text{Var}(S)=E(N)\text{Var}(Y)+\text{Var}(N)[E(Y)]^2=\lambda\,\text{Var}(Y)+\lambda[E(Y)]^2=\lambda E(Y^2)$$
$$\text{Var}(S)=10\times3333.33$$
$$\boxed{\text{Var}(S)=33333.33}$$

---

### Q8(a) [18/02/2018], [05/08/2017] (10) — Conditional expectation
> If X and Y are independent Poisson random variables with respective means λ₁ and λ₂, calculate the conditional expected value of X given that X + Y = n.

**Step 1 — distribution of $X\mid X+Y=n$:**
$$P(X=k\mid X+Y=n)=\frac{P(X=k)P(Y=n-k)}{P(X+Y=n)}=\frac{\dfrac{e^{-\lambda_1}\lambda_1^k}{k!}\cdot\dfrac{e^{-\lambda_2}\lambda_2^{n-k}}{(n-k)!}}{\dfrac{e^{-(\lambda_1+\lambda_2)}(\lambda_1+\lambda_2)^n}{n!}}$$

$$=\binom{n}{k}\left(\frac{\lambda_1}{\lambda_1+\lambda_2}\right)^k\left(\frac{\lambda_2}{\lambda_1+\lambda_2}\right)^{n-k}$$

This is $\text{Bin}\!\left(n,\ \dfrac{\lambda_1}{\lambda_1+\lambda_2}\right)$.

**Step 2 — take expectation of a Binomial:**
$$\boxed{E(X\mid X+Y=n)=\frac{n\lambda_1}{\lambda_1+\lambda_2}}$$

---

### Q5(c) [23/07/2016] (15) — MGF of Poisson + sum
> Define moments of a Random Variable. How can we obtain them from moment generating function? Derive the moment generating function for the Poisson distribution with parameter λ? If X and Y are independent Poisson random variables with parameters λ₁ and λ₂, respectively, then what is the distribution of X+Y?

*(Same derivation as Q4(b) [09/09/2025] above.)*
$$M_X(t)=e^{\lambda(e^t-1)}\ \Rightarrow\ M_{X+Y}(t)=e^{(\lambda_1+\lambda_2)(e^t-1)}\ \Rightarrow\ \boxed{X+Y\sim\text{Pois}(\lambda_1+\lambda_2)}$$

---

## Memoryless Property

### Q1(b) [09/09/2025] (10)
> Explain the memoryless property of a probability distribution. Analyze which continuous distribution is memoryless.

**Definition:**
$$P(X>s+t\mid X>s)=P(X>t)\qquad\forall s,t\ge0$$
Meaning: given the process has "survived" past time $s$, the remaining waiting time has the same distribution as starting fresh — no "aging."

**Which continuous distribution:** Exponential is the *only* continuous memoryless distribution.

**Proof for $X\sim\text{Expo}(\lambda)$:**
$$P(X>s+t\mid X>s)=\frac{P(X>s+t)}{P(X>s)}=\frac{e^{-\lambda(s+t)}}{e^{-\lambda s}}=e^{-\lambda t}=P(X>t)\qquad\blacksquare$$

---

# 2. Continuous Distributions

## Uniform

### Q4(c) [09/09/2025] (15)
> Find the mean and variance of a uniform random variable over a 3-dimensional cube having sides (0, a), (0, b), (0, c).

**Setup:** Joint density constant on the box:
$$f(x,y,z)=\frac{1}{abc},\qquad 0<x<a,\ 0<y<b,\ 0<z<c$$
Since this factors as $f(x,y,z)=\frac1a\cdot\frac1b\cdot\frac1c$, the coordinates are **independent**, each marginally uniform: $X\sim\text{Unif}(0,a)$, $Y\sim\text{Unif}(0,b)$, $Z\sim\text{Unif}(0,c)$.

**Formulas** (Uniform mean/variance): $E=\frac{\text{upper}}{2}$, $\text{Var}=\frac{\text{upper}^2}{12}$.

$$\boxed{E(X)=\frac a2,\quad E(Y)=\frac b2,\quad E(Z)=\frac c2}$$
$$\boxed{\text{Var}(X)=\frac{a^2}{12},\quad \text{Var}(Y)=\frac{b^2}{12},\quad \text{Var}(Z)=\frac{c^2}{12}}$$

---

### Q1(b) [22/03/2022] (10)
> Let X be a random (uniform) number x; 0 < x < 1 and let Y = X². Find the covariance of X and Y. Also, specify if X and Y are positively or negatively correlated, or uncorrelated?

**Formula:**
$$\text{Cov}(X,Y)=E(XY)-E(X)E(Y)=E(X^3)-E(X)E(X^2)$$

**Moments of $X\sim\text{Unif}(0,1)$:** $E(X^n)=\int_0^1 x^n\,dx=\dfrac{1}{n+1}$
$$E(X)=\frac12,\qquad E(X^2)=\frac13,\qquad E(X^3)=\frac14$$

**Plug in:**
$$\text{Cov}(X,Y)=\frac14-\frac12\cdot\frac13=\frac14-\frac16=\frac{3}{12}-\frac{2}{12}$$
$$\boxed{\text{Cov}(X,Y)=\frac{1}{12}>0}$$

Since $\text{Cov}(X,Y)>0$: **$X$ and $Y$ are positively correlated.**

---

## Normal

### Q2(c) [09/09/2025] (15)
> If z₁, z₂ are standard normal random variables then what is the distribution of |z₁ – z₂|?

**Step 1 — distribution of $D=Z_1-Z_2$:**
$$E(D)=0,\qquad \text{Var}(D)=\text{Var}(Z_1)+\text{Var}(Z_2)=1+1=2\quad(\text{independence assumed})$$
$$D\sim N(0,2)$$

**Step 2 — distribution of $|D|$ (half-normal with $\sigma^2=2$):**
$$f_D(x)=\frac{1}{\sqrt{2\pi\cdot2}}e^{-x^2/4}$$
For $x\ge0$, folding both tails onto the positive axis:
$$f_{|D|}(x)=2f_D(x)=\frac{2}{\sqrt{4\pi}}e^{-x^2/4}$$
$$\boxed{f_{|Z_1-Z_2|}(x)=\frac{1}{\sqrt{\pi}}e^{-x^2/4},\quad x\ge0}\qquad(\text{Half-Normal},\ \sigma^2=2)$$

---

### Q3(a) [30/04/2023] (15)
> Derive the moment generating function of the normal distribution with parameter μ and σ², and calculate its first two moments.

**Step 1 — MGF of standard normal $Z\sim N(0,1)$:**
$$M_Z(t)=\int_{-\infty}^{\infty}e^{tz}\frac{1}{\sqrt{2\pi}}e^{-z^2/2}dz$$
Complete the square: $-\frac{z^2}{2}+tz=-\frac{(z-t)^2}{2}+\frac{t^2}{2}$
$$M_Z(t)=e^{t^2/2}\int_{-\infty}^{\infty}\frac{1}{\sqrt{2\pi}}e^{-(z-t)^2/2}dz=e^{t^2/2}\cdot1$$
$$\boxed{M_Z(t)=e^{t^2/2}}$$

**Step 2 — MGF of $X=\mu+\sigma Z\sim N(\mu,\sigma^2)$:**
$$M_X(t)=E(e^{t(\mu+\sigma Z)})=e^{\mu t}E(e^{t\sigma Z})=e^{\mu t}M_Z(\sigma t)=e^{\mu t}e^{\sigma^2t^2/2}$$
$$\boxed{M_X(t)=e^{\mu t+\sigma^2t^2/2}}$$

**First two moments:**
$$M_X'(t)=(\mu+\sigma^2t)e^{\mu t+\sigma^2t^2/2}\ \Rightarrow\ M_X'(0)=\mu$$
$$\boxed{E(X)=\mu}$$
$$M_X''(t)=\sigma^2e^{\mu t+\sigma^2t^2/2}+(\mu+\sigma^2t)^2e^{\mu t+\sigma^2t^2/2}\ \Rightarrow\ M_X''(0)=\sigma^2+\mu^2$$
$$\boxed{E(X^2)=\sigma^2+\mu^2}$$

---

### Q7(b) [09/09/2025] (10) — CLT application
> The weights of a population of workers have mean 167 and standard deviation 27. If a sample of 36 workers is chosen, approximate the probability that the sample mean of their weights lies between 163 and 170.

**Formula (CLT):** $\bar X\approx N\!\left(\mu,\ \dfrac{\sigma^2}{n}\right)$, standardize with $Z=\dfrac{\bar X-\mu}{\sigma/\sqrt n}$.

**Plug in** $\mu=167,\ \sigma=27,\ n=36$: $\sigma_{\bar X}=27/\sqrt{36}=27/6=4.5$
$$P(163<\bar X<170)=P\!\left(\frac{163-167}{4.5}<Z<\frac{170-167}{4.5}\right)=P(-0.889<Z<0.667)$$

**Look up standard normal table:**
$$\Phi(0.667)\approx0.7486,\qquad \Phi(-0.889)=1-\Phi(0.889)\approx1-0.8133=0.1867$$
$$P=0.7486-0.1867$$
$$\boxed{P(163<\bar X<170)\approx0.562}$$

---

## Exponential

### Q6 [30/04/2023] (15) — Data center storage devices
> Suppose a data centre has 100 storage devices. The designer assumed the time to failure is exponentially distributed with mean two years (λ = 1/2).
> (a) P(first failure occurs after 1 year)?
> (b) P(exactly 10 failures in the 13th month)?
> (c) Is exponential modeling reasonable?

**Setup:** Each device: $\text{Expo}(\lambda=0.5/\text{yr})$. The pooled failure process (min of 100 independent exponentials / superposition) is $\text{Expo/Poisson}$ with combined rate:
$$\mu=100\times\lambda=100\times0.5=50\ \text{per year}$$

**(a)** Time of first failure $T_{\min}\sim\text{Expo}(\mu=50)$:
$$P(T_{\min}>1)=e^{-\mu\cdot1}=e^{-50}$$
$$\boxed{P(T_{\min}>1)=e^{-50}\approx1.9\times10^{-22}}\quad(\text{essentially }0)$$

**(b)** 13th month = interval of length $1/12$ year, using the pooled Poisson process rate $\mu=50/\text{yr}$:
$$\lambda_{\text{month}}=\mu\times\frac{1}{12}=\frac{50}{12}=4.1\overline{6}$$
$$P(N=10)=\frac{e^{-\lambda_{\text{month}}}\lambda_{\text{month}}^{10}}{10!}=\frac{e^{-4.1667}(4.1667)^{10}}{3{,}628{,}800}$$

**Plug in:** $(4.1667)^{10}\approx1{,}577{,}712$, $e^{-4.1667}\approx0.015505$
$$P(N=10)=0.015505\times\frac{1{,}577{,}712}{3{,}628{,}800}=0.015505\times0.43475$$
$$\boxed{P(N=10)\approx0.00674}$$

**(c)** No — not reasonable. The constant-hazard (memoryless) exponential model predicts the first failure almost surely within days ($E[T_{\min}]=1/50$ yr $\approx7.3$ days), and it cannot capture wear-out (increasing failure rate over time) that real physical storage devices exhibit.

---

### Q8(b) [10/01/2021] (10) — Lifetime test, expected end time
> 100 items are simultaneously put on a lifetime test with iid Exp(mean 200 hrs) lifetimes. Test ends at the 5th failure. Find the expected end time.

**Formula (order statistics of iid exponentials — memoryless gap property):** the gap between the $(k-1)$-th and $k$-th failure among $n$ surviving items is $\text{Expo}(\lambda(n-k+1))$, so
$$E[T_{(r)}]=\frac{1}{\lambda}\sum_{k=1}^{r}\frac{1}{n-k+1}$$

**Plug in** $\lambda=1/200$, $n=100$, $r=5$:
$$E[T_{(5)}]=200\left(\frac{1}{100}+\frac{1}{99}+\frac{1}{98}+\frac{1}{97}+\frac{1}{96}\right)$$
$$=200(0.01+0.010101+0.010204+0.010309+0.010417)$$
$$=200(0.051031)$$
$$\boxed{E[T_{(5)}]\approx10.21\text{ hours}}$$

---

### Q3(c) [12/01/2025], Q3(d) [22/03/2022] — Exponential memoryless proof
> Prove that Exponential distribution is a memoryless distribution.

$$P(X>s+t\mid X>s)=\frac{P(X>s+t)}{P(X>s)}=\frac{e^{-\lambda(s+t)}}{e^{-\lambda s}}=e^{-\lambda t}=P(X>t)\qquad\blacksquare$$

---

### Q3(d)/Q3(c) [multiple dates: 12/01/2025, 29/10/2023, 25/03/2019, 18/02/2018, 23/07/2016, 22/03/2022] — Poisson inter-arrival times are iid Exponential (6+ appearances)
> Consider a Poisson process with rate λ. Let T₁ be the time of the first event, and for n>1, Tₙ the elapsed time between the (n−1)st and nth event. Show that Tₙ are iid Exponential with mean 1/λ.

**Step 1 — $T_1$:**
$$P(T_1>t)=P(N(t)=0)=\frac{e^{-\lambda t}(\lambda t)^0}{0!}=e^{-\lambda t}\ \Rightarrow\ T_1\sim\text{Expo}(\lambda)$$

**Step 2 — $T_n$ for $n\ge2$, conditioning on the past:**
$$P(T_n>t\mid T_1=t_1,\dots,T_{n-1}=t_{n-1})=P(\text{no events in }(t_1+\cdots+t_{n-1},\ t_1+\cdots+t_{n-1}+t])$$
By **stationary and independent increments** of the Poisson process, this equals
$$P(N(t)=0)=e^{-\lambda t}$$
independent of $t_1,\dots,t_{n-1}$.

**Conclusion:** each $T_n$ has CDF $1-e^{-\lambda t}$, independent of the others:
$$\boxed{T_n\overset{iid}{\sim}\text{Expo}(\lambda),\quad E(T_n)=\frac1\lambda}\qquad\blacksquare$$

---

### Q3(d) [25/03/2019], Q7(d) [05/08/2017, 23/07/2016] — $P(X_1<X_2)$ for independent exponentials (3+ appearances)
> Suppose X₁, X₂ are independent exponential random variables with respective means 1/λ₁ and 1/λ₂; what is P{X₁ < X₂}?

**Formula (condition on $X_1$):**
$$P(X_1<X_2)=\int_0^\infty P(X_2>x)f_{X_1}(x)\,dx=\int_0^\infty e^{-\lambda_2 x}\cdot\lambda_1e^{-\lambda_1x}\,dx$$

**Plug in and integrate:**
$$=\lambda_1\int_0^\infty e^{-(\lambda_1+\lambda_2)x}dx=\lambda_1\cdot\frac{1}{\lambda_1+\lambda_2}$$
$$\boxed{P(X_1<X_2)=\frac{\lambda_1}{\lambda_1+\lambda_2}}$$

---

### Q3(c) [22/03/2022] (6) — Counting process / Poisson process definition
> Define counting process and illustrate its properties. When do we consider a counting process as a Poisson process?

**Counting process $\{N(t), t\ge0\}$:** counts the number of events by time $t$, satisfying:
1. $N(t)\ge0$, integer-valued.
2. Non-decreasing in $t$.
3. For $s<t$, $N(t)-N(s)$ = number of events in $(s,t]$.

**Poisson process** with rate $\lambda$: a counting process additionally satisfying
1. $N(0)=0$
2. **Independent increments** — numbers of events in disjoint intervals are independent
3. **Stationary increments** — distribution of $N(t+s)-N(s)$ depends only on $t$
4. $P(N(h)=1)=\lambda h+o(h)$, $P(N(h)\ge2)=o(h)$ as $h\to0$

Equivalently: $N(t)\sim\text{Pois}(\lambda t)$ for every $t$.

---

# 3. Essential Probability Rules & Theorems

### Q2(b) [09/09/2025] (10) — Expected empty boxes
> Randomly, k distinguishable balls are placed into n distinguishable boxes, with all possibilities equally likely. Find the expected number of empty boxes.

**Formula (indicator + linearity):** let $I_j=1$ if box $j$ is empty.
$$P(I_j=1)=\left(\frac{n-1}{n}\right)^k\quad(\text{each of the }k\text{ balls avoids box }j)$$
$$E(\#\text{empty})=\sum_{j=1}^n E(I_j)=n\left(\frac{n-1}{n}\right)^k$$
$$\boxed{E(\#\text{empty boxes})=n\left(1-\frac1n\right)^k}$$

---

### Q4(a) [30/04/2023] (5) — Law of total expectation
> Prove the law of total expectation for discrete random variables.

**Claim:** $E(X)=\sum_y E(X\mid Y=y)P(Y=y)=E[E(X\mid Y)]$

**Proof:**
$$\sum_y E(X\mid Y=y)P(Y=y)=\sum_y\left[\sum_x xP(X=x\mid Y=y)\right]P(Y=y)$$
$$=\sum_y\sum_x x\,P(X=x,Y=y)=\sum_x x\sum_y P(X=x,Y=y)=\sum_x xP(X=x)$$
$$\boxed{=E(X)}\qquad\blacksquare$$

---

### Q1(b) [22/03/2022] — Covariance of uniform & its square
*(Same as under Continuous Distributions → Uniform above.)*
$$\boxed{\text{Cov}(X,X^2)=\frac{1}{12}>0\ \Rightarrow\ \text{positively correlated}}$$

---

### Q3(b) [30/04/2023] (5) — CLT statement
> State and explain the central limit theorem.

**Statement:** if $X_1,\dots,X_n$ are iid with mean $\mu$ and variance $\sigma^2<\infty$, then as $n\to\infty$:
$$\frac{\bar X_n-\mu}{\sigma/\sqrt n}\ \xrightarrow{d}\ N(0,1)$$
Equivalently, $S_n=X_1+\cdots+X_n$ is approximately $N(n\mu,\ n\sigma^2)$ for large $n$ — **regardless of the shape of the original distribution.**

---

### Q7(b) [09/09/2025] — CLT application (sample mean weights)
*(Solved above under Continuous Distributions → Normal.)*
$$\boxed{P(163<\bar X<170)\approx0.562}$$

---

# 4. Moment Generating Functions (MGFs)

### Two-chips MGF problem — appears 7+ times: Q2(a)[12/01/2025], Q1(c)[29/10/2023], Q1(b)[25/03/2019], Q5(b)[18/02/2018], Q8(c)[18/02/2018], Q5(b)[05/08/2017], Q8(c)[05/08/2017]
> Two chips are drawn at random without replacement from a box that contains five chips numbered 1 through 5. If the sum of chips drawn is even, X = 5; if the sum is odd, X = −3.
> (i) Find the MGF for X. (ii) Use the MGF to find the first and second moments. (iii) Find E(X) and Var(X).

**Step 0 — find $P(\text{sum even})$ and $P(\text{sum odd})$:**
Chips: $\{1,2,3,4,5\}$ — 3 odd $\{1,3,5\}$, 2 even $\{2,4\}$. Total pairs $=\binom52=10$.
- Sum even $\Leftrightarrow$ both odd or both even: $\binom32+\binom22=3+1=4$ pairs
- Sum odd $\Leftrightarrow$ one odd, one even: $3\times2=6$ pairs

$$P(\text{even})=\frac{4}{10}=\frac25,\qquad P(\text{odd})=\frac{6}{10}=\frac35$$
$$X=5\text{ w.p. }\tfrac25,\qquad X=-3\text{ w.p. }\tfrac35$$

**(i) MGF:**
$$M(t)=E(e^{tX})=\frac25e^{5t}+\frac35e^{-3t}$$

**(ii) Moments from derivatives, $M^{(n)}(0)=E(X^n)$:**
$$M'(t)=2e^{5t}-\frac95e^{-3t}\ \Rightarrow\ M'(0)=2-\frac95=\frac{10-9}{5}$$
$$\boxed{E(X)=\frac15=0.2}$$

$$M''(t)=10e^{5t}+\frac{27}{5}e^{-3t}\ \Rightarrow\ M''(0)=10+\frac{27}{5}=\frac{50+27}{5}$$
$$\boxed{E(X^2)=\frac{77}{5}=15.4}$$

**(iii) Variance:**
$$\text{Var}(X)=E(X^2)-[E(X)]^2=\frac{77}{5}-\left(\frac15\right)^2=\frac{385}{25}-\frac{1}{25}$$
$$\boxed{\text{Var}(X)=\frac{384}{25}=15.36}$$

---

### Q8(b) [18/02/2018], [05/08/2017] — MGF of Binomial + sum
*(Solved above under Discrete → Binomial.)*
$$\boxed{M_X(t)=(pe^t+q)^n,\qquad X+Y\sim\text{Bin}(n+m,p)}$$

### Q5(c) [23/07/2016] — MGF of Poisson + sum
*(Solved above under Discrete → Poisson.)*
$$\boxed{M_X(t)=e^{\lambda(e^t-1)},\qquad X+Y\sim\text{Pois}(\lambda_1+\lambda_2)}$$

### Q3(a) [30/04/2023] — MGF of Normal
*(Solved above under Continuous → Normal.)*
$$\boxed{M_X(t)=e^{\mu t+\sigma^2t^2/2},\quad E(X)=\mu,\quad E(X^2)=\mu^2+\sigma^2}$$

---

# 5. Priority Recap for Last-Minute Review

| # | Topic | Answer |
|---|---|---|
| 1 | Two-chips MGF | $M(t)=\tfrac25e^{5t}+\tfrac35e^{-3t}$, $E(X)=\tfrac15$, Var$=\tfrac{384}{25}$ |
| 2 | DVD Binomial | $P\approx0.2346$ |
| 3 | Poisson inter-arrivals iid Expo | Stationary + independent increments $\Rightarrow e^{-\lambda t}$ tail |
| 4 | $P(X_1<X_2)$ exponentials | $\lambda_1/(\lambda_1+\lambda_2)$ |
| 5 | Exponential memoryless | $e^{-\lambda(s+t)}/e^{-\lambda s}=e^{-\lambda t}$ |
| 6 | Binomial→Poisson | $\lambda=np$, limit of $(pe^t+q)$-type expansion |
| 7 | Poisson thinning $E[X\mid X+Y{=}n]$ | $n\lambda_1/(\lambda_1+\lambda_2)$ |
| 8 | Sum of Poissons/Binomials via MGF | Multiply MGFs, match form |
| 9 | CLT application | Standardize, $\sigma_{\bar X}=\sigma/\sqrt n$ |
| 10 | Normal MGF & moments | $e^{\mu t+\sigma^2t^2/2}$, $E(X)=\mu$, $E(X^2)=\mu^2+\sigma^2$ |

**Note on Section 5 of the bank (Vandermonde's Identity, Euler limit, St. Petersburg Paradox):** no exact past-paper question matched these — they're pure cheatsheet formulas this term. Just memorize the identities as given; if asked, state + one-line proof/example (e.g., Euler limit is exactly what's used in the Binomial→Poisson proof above).

Good luck, bhai — ভালো করো কাল! 🔥