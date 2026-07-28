The uploaded exercise contains **4 questions/sub-questions**: one expectation proof and three parts on axis-aligned rectangles.  

---

# Problem 2

### Statement

Show that

[
\mathbb E_{S\sim D^m}!\left[L_S(h)\right]
=========================================

L_{(D,f)}(h),
]

where

[
L_S(h)=\frac1m\sum_{i=1}^{m}\mathbf1[h(x_i)\neq f(x_i)].
]

---

### Solution

[
\begin{aligned}
\mathbb E[L_S(h)]
&=
\mathbb E\left[\frac1m\sum_{i=1}^{m}
\mathbf1[h(x_i)\neq f(x_i)]
\right] [2mm]
&=
\frac1m\sum_{i=1}^{m}
\mathbb E!\left[
\mathbf1[h(x_i)\neq f(x_i)]
\right] [2mm]
&=
\frac1m\sum_{i=1}^{m}
P(h(x_i)\neq f(x_i)) [2mm]
&=
\frac1m\sum_{i=1}^{m}
L_{(D,f)}(h) [2mm]
&=
L_{(D,f)}(h).
\end{aligned}
]

Hence,

[
\boxed{\mathbb E_{S\sim D^m}[L_S(h)]
====================================

L_{(D,f)}(h)}.
]

---

# Problem 3.1

### Statement

Algorithm (A) returns the **smallest rectangle enclosing all positive training examples**.

Show that (A) is an ERM algorithm.

---

### Solution

Let

[
R(S)=\text{smallest rectangle containing all positive samples}.
]

Since

* every positive sample lies inside (R(S)),
* every negative sample lies outside the target rectangle (R^*),

we have

[
R(S)\subseteq R^*.
]

Therefore,

[
\begin{aligned}
\forall (x,y)\in S,
\qquad
h_{R(S)}(x,y)
=============

f(x,y).
\end{aligned}
]

Hence,

[
L_S(h_{R(S)})=0.
]

Since empirical loss cannot be negative,

[
0=L_S(h_{R(S)})
===============

\min_{h\in H}L_S(h).
]

Thus,

[
\boxed{A\text{ is an ERM algorithm}.}
]

---

# Problem 3.2

### Statement

Show that if

[
m\ge
\frac{4}{\epsilon}
\ln!\left(\frac4\delta\right),
]

then with probability at least

[
1-\delta,
]

algorithm (A) outputs a hypothesis whose error is at most (\epsilon).

---

### Solution

Since (R(S)) contains only positive examples,

[
\boxed{R(S)\subseteq R^*.}
]

If every strip

[
R_1,R_2,R_3,R_4
]

contains at least one positive example,

then

[
D(R^*\setminus R(S))
\le
4\cdot\frac{\epsilon}{4}
=\epsilon.
]

Hence,

[
L_D(h)\le\epsilon.
]

---

For each strip,

[
\begin{aligned}
P(R_i\text{ has no sample})
&=
\left(1-\frac{\epsilon}{4}\right)^m\
&\le
e^{-m\epsilon/4}.
\end{aligned}
]

Using the union bound,

[
\begin{aligned}
P(L_D(h)>\epsilon)
&\le
4e^{-m\epsilon/4}.
\end{aligned}
]

Require

[
4e^{-m\epsilon/4}\le\delta.
]

Taking logarithm,

[
\begin{aligned}
-\frac{m\epsilon}{4}
&\le
\ln\frac{\delta}{4}
[1mm]
m
&\ge
\frac4\epsilon
\ln\frac4\delta.
\end{aligned}
]

Therefore,

[
\boxed{
P(L_D(h)\le\epsilon)
\ge
1-\delta.
}
]

---

# Problem 3.3

### Statement

Repeat the previous result for axis-aligned rectangles in (\mathbb R^d).

---

### Solution

There are

[
2d
]

boundary regions.

Each has probability mass

[
\frac{\epsilon}{2d}.
]

For one region,

[
P(\text{no sample})
\le
e^{-m\epsilon/(2d)}.
]

Applying the union bound,

[
P(L_D(h)>\epsilon)
\le
2d,
e^{-m\epsilon/(2d)}.
]

Require

[
2d,e^{-m\epsilon/(2d)}
\le
\delta.
]

Thus,

[
\begin{aligned}
m
&\ge
\frac{2d}{\epsilon}
\ln\frac{2d}{\delta}.
\end{aligned}
]

Hence,

[
\boxed{
m\ge
\frac{2d}{\epsilon}
\ln!\left(\frac{2d}{\delta}\right).
}
]

---

# Problem 3.4

### Statement

Show that the runtime of algorithm (A) is polynomial in

[
d,\qquad
\frac1\epsilon,\qquad
\log\frac1\delta.
]

---

### Solution

For each positive example,

* compare all (d) coordinates,
* update minimum and maximum.

Thus,

[
T(m,d)=O(md).
]

Using

[
m=
O!\left(
\frac{d}{\epsilon}
\log\frac{d}{\delta}
\right),
]

[
\begin{aligned}
T
&=
O(md)\
&=
O!\left(
\frac{d^2}{\epsilon}
\log\frac{d}{\delta}
\right).
\end{aligned}
]

Hence the runtime is polynomial in

[
\boxed{
d,;
\frac1\epsilon,;
\log\frac1\delta.
}
]





==========================




**Problem 2 — Statement**

Let $\mathcal{H}$ be a class of binary classifiers over a domain $\mathcal{X}$. Let $\mathcal{D}$ be an unknown distribution over $\mathcal{X}$, and let $f$ be the target hypothesis in $\mathcal{H}$. Fix some $h \in \mathcal{H}$. Show that the expected value of $L_S(h)$ over the choice of $S|_x$ equals $L_{(\mathcal D,f)}(h)$, namely,

$$\mathbb{E}_{S|_x\sim \mathcal D^m}[L_S(h)] = L_{(\mathcal D,f)}(h).$$

**Solution**

$$
\begin{aligned}
L_S(h) &= \frac{1}{m}\sum_{i=1}^m \mathbf{1}[h(x_i)\neq f(x_i)] \\[4pt]
\mathbb{E}_{S|_x\sim \mathcal D^m}[L_S(h)] &= \frac{1}{m}\sum_{i=1}^m \mathbb{E}_{x_i\sim \mathcal D}\big[\mathbf{1}[h(x_i)\neq f(x_i)]\big] \\
&= \frac{1}{m}\sum_{i=1}^m \Pr_{x\sim \mathcal D}[h(x)\neq f(x)] \\
&= \frac{1}{m}\cdot m\cdot L_{(\mathcal D,f)}(h) = L_{(\mathcal D,f)}(h)
\end{aligned}
$$

---

**Problem 3 — Statement (Axis-aligned rectangles)**

An axis-aligned rectangle classifier in the plane assigns the value $1$ to a point iff it is inside a certain rectangle. Formally, given real numbers $a_1\le b_1,\ a_2\le b_2$, define $h_{(a_1,b_1,a_2,b_2)}$ by

$$h_{(a_1,b_1,a_2,b_2)}(x_1,x_2)=\begin{cases}1 & \text{if } a_1\le x_1\le b_1 \text{ and } a_2\le x_2\le b_2\\ 0 & \text{otherwise}\end{cases}$$

The class of all such rectangles is $\mathcal H^2_{\text{rec}} = \{h_{(a_1,b_1,a_2,b_2)} : a_1\le b_1,\ a_2\le b_2\}$ — an infinite hypothesis class. Throughout, assume realizability.

**3.1** Let $A$ be the algorithm that returns the smallest rectangle enclosing all positive examples in the training set. Show that $A$ is an ERM.

**Solution**

$$
\begin{aligned}
&\text{By realizability, } \exists\, R^*=R(a_1^*,b_1^*,a_2^*,b_2^*)\text{ s.t. } L_S(h_{R^*})=0.\\
&\text{So } R^* \text{ contains every positive point and excludes every negative point of } S.\\
&R(S) := \text{smallest axis-aligned rectangle containing all positives in } S.\\
&\text{Since } R^* \text{ is itself such a rectangle} \;\Rightarrow\; R(S)\subseteq R^*.\\
&\text{Positives: contained in } R(S) \text{ by construction} \Rightarrow \text{correctly labeled.}\\
&\text{Negatives: lie outside } R^*, \text{ and } R(S)\subseteq R^* \Rightarrow \text{lie outside } R(S) \text{ too} \Rightarrow \text{correctly labeled.}\\
&\therefore L_S(h_{R(S)})=0=\min_h L_S(h) \;\Rightarrow\; A \text{ is an ERM.}
\end{aligned}
$$

**3.2** Show that if $A$ receives a training set of size $\ge \dfrac{4\log(4/\delta)}{\epsilon}$, then with probability at least $1-\delta$ it returns a hypothesis with error at most $\epsilon$.

*Hint:* Fix distribution $\mathcal D$ over $\mathcal X$, let $R^*=R(a_1^*,b_1^*,a_2^*,b_2^*)$ generate the labels, with corresponding hypothesis $f$. Let $a_1\ge a_1^*$ be such that the probability mass of $R_1=R(a_1^*,a_1,a_2^*,b_2^*)$ is exactly $\epsilon/4$. Similarly let $b_1,a_2,b_2$ define $R_2=R(b_1,b_1^*,a_2^*,b_2^*)$, $R_3=R(a_1^*,b_1^*,a_2^*,a_2)$, $R_4=R(a_1^*,b_1^*,b_2,b_2^*)$, each of mass exactly $\epsilon/4$. Let $R(S)$ be the rectangle returned by $A$.

- Show that $R(S)\subseteq R^*$.
- Show that if $S$ contains positive examples in all of $R_1,R_2,R_3,R_4$, then the hypothesis returned by $A$ has error at most $\epsilon$.
- For each $i\in\{1,\dots,4\}$, upper bound the probability that $S$ does not contain an example from $R_i$.
- Use the union bound to conclude the argument.

**Solution**

$$
\begin{aligned}
\textbf{(a)}\quad & R(S)\subseteq R^*: \text{ same argument as above } (R(S)\text{ is the smallest rectangle}\\
&\text{enclosing points known to lie in } R^*,\text{ and } R^* \text{ is one such enclosing rectangle}).
\end{aligned}
$$

$$
\begin{aligned}
\textbf{(b)}\quad &\text{Since } R(S)\subseteq R^*,\text{ all classification errors are false negatives, i.e.}\\
&\ \text{error}(h_{R(S)}) = \Pr_{x\sim\mathcal D}[x\in R^*\setminus R(S)].\\
&\text{If } S \text{ has a positive point in each of } R_1,R_2,R_3,R_4,\text{ then each side of } R(S)\\
&\ \text{is pushed to within } \epsilon/4 \text{ mass of the corresponding side of } R^*, \text{ so}\\
&\ R^*\setminus R(S) \subseteq R_1\cup R_2\cup R_3\cup R_4.\\
&\Rightarrow \text{error}(h_{R(S)}) \le \sum_{i=1}^4 \Pr[R_i] = 4\cdot \frac{\epsilon}{4}=\epsilon.
\end{aligned}
$$

$$
\begin{aligned}
\textbf{(c)}\quad &\Pr[S \text{ has no point in } R_i] = (1-\epsilon/4)^m \le e^{-\epsilon m/4}, \quad i=1,\dots,4.
\end{aligned}
$$

$$
\begin{aligned}
\textbf{(d)}\quad &\Pr[\exists i: S \text{ misses } R_i] \le \sum_{i=1}^4 \Pr[S \text{ misses } R_i] \le 4e^{-\epsilon m/4}.\\
&\text{Set } 4e^{-\epsilon m/4}\le \delta \;\Rightarrow\; m \ge \frac{4}{\epsilon}\ln\!\frac{4}{\delta}.\\
&\text{Then w.p.} \ge 1-\delta,\ S \text{ meets all four strips} \Rightarrow \text{error}(A(S))\le \epsilon.
\end{aligned}
$$

**3.3** Repeat the previous question for the class of axis-aligned rectangles in $\mathbb R^d$.

**Solution**

$$
\begin{aligned}
&\text{An axis-aligned rectangle in } \mathbb{R}^d \text{ has } 2d \text{ boundary sides.}\\
&\text{Define } 2d \text{ "margin strips" } R_1,\dots,R_{2d}, \text{ each of mass } \epsilon/(2d),\\
&\text{analogous to the } d=2 \text{ construction (two strips per coordinate).}\\
&\Pr[\text{miss } R_i] \le \Big(1-\frac{\epsilon}{2d}\Big)^m \le e^{-\epsilon m/(2d)}.\\
&\text{Union bound: } \Pr[\exists i: \text{miss } R_i] \le 2d\, e^{-\epsilon m/(2d)} \le \delta\\
&\Rightarrow m \ge \frac{2d}{\epsilon}\ln\!\frac{2d}{\delta}.\\
&\text{As before, if all } 2d \text{ strips contain a sample point, } R^*\setminus R(S) \subseteq \bigcup_i R_i,\\
&\text{so } \text{error}(A(S)) \le 2d\cdot\frac{\epsilon}{2d} = \epsilon \text{ with probability} \ge 1-\delta.
\end{aligned}
$$

**3.4** Show that the runtime of applying algorithm $A$ is polynomial in $d$, $1/\epsilon$, and $\log(1/\delta)$.

**Solution**

$$
\begin{aligned}
&\text{For each of the } d \text{ coordinates, } A \text{ computes } \min \text{ and } \max \text{ over positive}\\
&\text{examples: cost } O(md) \text{ to scan } m \text{ points in } \mathbb{R}^d.\\
&\text{With } m = \Theta\!\left(\frac{d}{\epsilon}\log\frac{d}{\delta}\right)\text{ (from part 3), total runtime}\\
&O(md) = O\!\left(\frac{d^2}{\epsilon}\log\frac{d}{\delta}\right),\\
&\text{which is polynomial in } d,\ 1/\epsilon,\ \text{and } \log(1/\delta).
\end{aligned}
$$