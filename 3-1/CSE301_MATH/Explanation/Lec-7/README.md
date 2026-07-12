Alright, let's dive into Lecture 7 without getting bogged down by the raw handwriting. This lecture covers one of the most famous and practical concepts in probability: **The Gambler's Ruin**.

This is the mathematical proof of why the phrase *"the house always wins"* is completely true.

### 1. The Game Setup (The Random Walk)

Imagine you (Player A) and a Casino (Player B) are flipping a coin for $1 bets.

* **$p$** = The probability that you win a round.
* **$q$** = The probability that the Casino wins a round ($1 - p$).
* **$i$** = The amount of money you start with.
* **$N$** = The total amount of money on the table (Your money + the Casino's money).

You are playing until someone goes completely bankrupt. The notes describe this as a **"Random Walk"** on a number line from $0$ to $N$. If you hit $0$, you are ruined. If you hit $N$, you bankrupt the casino (you win). These are called **"absorbing states"** because once you hit them, the game is over.

---

### 2. The Big Difference Equation

We want to find $P_i$, which is the probability that you win the *entire game* given that you currently have $\$i$.

Think about what happens on your very next flip:

* You win the flip (probability $p$): Your money goes up to $i+1$. Your new chance of winning the game becomes $P_{i+1}$.
* You lose the flip (probability $q$): Your money goes down to $i-1$. Your new chance of winning becomes $P_{i-1}$.

This logic creates the core equation of the page:


$$P_i = pP_{i+1} + qP_{i-1}$$

**The Boundaries:**

* If you have $\$0$, you are already bankrupt. So, your chance of winning is zero: $P_0 = 0$.
* If you have $\$N$, you've already won everything. Your chance of winning is 100%: $P_N = 1$.

---

### 3. Solving the Math (The Two Formulas)

The middle chunk of your notes uses algebra (specifically, characteristic equations where $P_i = x^i$) to solve that difference equation. 
My bad bro, you caught me! I definitely glossed over the heavy algebra in the middle of that page. You are absolutely right to call me out on it.

Let's crack open that exact calculation from your notes. This is how we get from the basic game rules to that crazy formula.

### Step 1: Setting up the "Guess"

We established the core rule of the game (the difference equation):


$$P_i = pP_{i+1} + qP_{i-1}$$

In math, when you have an equation like this, a standard trick is to guess that the solution looks like an exponential function. Your notes guess that **$P_i = x^i$**.

If we plug $x^i$ into our equation, we get:


$$x^i = px^{i+1} + qx^{i-1}$$

To simplify this, we divide the entire equation by $x^{i-1}$:


$$x = px^2 + q$$

Now, rearrange it to look like a standard quadratic equation ($ax^2 + bx + c = 0$):


$$px^2 - x + q = 0$$

### Step 2: The Quadratic Formula

Now we need to solve for $x$ using the quadratic formula: $x = \frac{-b \pm \sqrt{b^2 - 4ac}}{2a}$.
Plugging in our values ($a=p$, $b=-1$, $c=q$):


$$x = \frac{1 \pm \sqrt{1 - 4pq}}{2p}$$

This looks messy, but your notes do a beautiful simplification of the part inside the square root ($1 - 4pq$).
Since $q = 1 - p$, we can rewrite it:


$$1 - 4pq = 1 - 4p(1 - p)$$

$$= 1 - 4p + 4p^2$$

$$= (2p - 1)^2$$

Because it's a perfect square, taking the square root cancels it out! So our equation for $x$ becomes:


$$x = \frac{1 \pm (2p - 1)}{2p}$$

This gives us two possible roots (solutions) for $x$:

1. **Plus version:** $x_1 = \frac{1 + 2p - 1}{2p} = \frac{2p}{2p} = \mathbf{1}$
2. **Minus version:** $x_2 = \frac{1 - (2p - 1)}{2p} = \frac{2 - 2p}{2p} = \frac{2(1 - p)}{2p}$. Since $1-p = q$, this equals **$\frac{q}{p}$**

### Step 3: The General Solution

Because difference equations are linear, the total solution is a combination of both roots we just found, multiplied by some unknown constants ($A$ and $B$).


$$P_i = A \cdot (x_1)^i + B \cdot (x_2)^i$$

$$P_i = A \cdot (1)^i + B \cdot \left(\frac{q}{p}\right)^i$$

Since $1^i$ is always just $1$, it simplifies to:


$$P_i = A + B\left(\frac{q}{p}\right)^i$$

### Step 4: Finding A and B (The Boundaries)

To figure out what $A$ and $B$ are, we use the two things we know for absolute certain (the absorbing states):

1. If you start with $\$0$, you have a $0\%$ chance of winning: **$P_0 = 0$**
2. If you start with $\$N$, you have a $100\%$ chance of winning: **$P_N = 1$**

**Let's plug in $P_0 = 0$:**


$$0 = A + B\left(\frac{q}{p}\right)^0$$


Since anything to the power of 0 is 1, this means $0 = A + B$, which means **$B = -A$**.

**Let's plug in $P_N = 1$ (and substitute $B$ with $-A$):**


$$1 = A - A\left(\frac{q}{p}\right)^N$$


Factor out the $A$:


$$1 = A \left[1 - \left(\frac{q}{p}\right)^N\right]$$


Solve for $A$:


$$A = \frac{1}{1 - (q/p)^N}$$

### Step 5: The Final Boss Formula

Now we take our general solution ($P_i = A - A(q/p)^i$), factor out the $A$ to get $P_i = A(1 - (q/p)^i)$, and finally substitute that massive fraction we just found for $A$:

$$P_i = \left( \frac{1}{1 - (q/p)^N} \right) \cdot \left( 1 - \left(\frac{q}{p}\right)^i \right)$$

Which perfectly matches the master equation on your page:


$$P_i = \frac{1 - (q/p)^i}{1 - (q/p)^N}$$
-------------


#### Scenario A: The Game is Unfair ($p \neq 0.5$)

This is the reality of a casino. Let's say you are playing a game slightly rigged against you. The mathematical solution for your chance of winning the whole game ($P_i$) is:


$$P_i = \frac{1 - (q/p)^i}{1 - (q/p)^N}$$

#### Scenario B: The Game is perfectly Fair ($p = 0.5$)

If $p = 0.5$ and $q = 0.5$, then $q/p = 1$. If you plug $1$ into the unfair formula, you divide by zero, which breaks the math.

To fix this, your notes use **L'Hôpital's Rule** from calculus to find the limit as the game approaches fairness ($y \to 1$). Taking the derivative of the top and bottom gives us a beautifully simple formula:

$$\begin{align*} P_i &= \frac{1 - (q/p)^i}{1 - (q/p)^N} \\ \text{Let } y &= \frac{q}{p} \\ P_i &= \lim_{y \to 1} \frac{1 - y^i}{1 - y^N} \\ &= \frac{1 - 1^i}{1 - 1^N} = \frac{0}{0} \quad \text{(Apply L'Hôpital's Rule)} \\ &= \lim_{y \to 1} \frac{\frac{d}{dy}(1 - y^i)}{\frac{d}{dy}(1 - y^N)} \\ &= \lim_{y \to 1} \frac{-i y^{i-1}}{-N y^{N-1}} \\ &= \frac{-i (1)^{i-1}}{-N (1)^{N-1}} \\ &= \frac{-i}{-N} \\ &= \frac{i}{N} \end{align*}$$


$$P_i = \frac{i}{N}$$


*Translation:* If the game is fair, your chance of bankrupting the other player is simply the ratio of your starting money to the total money.

---

### 4. The "Casino Edge" Lesson (The most important part)

Look at the numbers jotted down at the end of the Gambler's Ruin section. This is where the math gets terrifying for gamblers.

Assume you and the casino start with the same amount of money ($i = N/2$).

**Case 1: A Short Game**

* Total money $N = 20$. You start with $10$.
* The game is *slightly* unfair. Your win probability $p = 0.49$ (a 49% chance, like playing Roulette).
* Using our formula, your chance of bankrupting the casino is roughly **$40\%$**. A 1% disadvantage dropped your overall chances by 10%.

**Case 2: A Long Game**

* Total money $N = 100$. You start with $50$.
* The game is still $p = 0.49$.
* Using our formula: $P_{50} = \frac{1 - (0.51/0.49)^{50}}{1 - (0.51/0.49)^{100}} \approx \mathbf{0.02}$

**The Lesson:** Even with a tiny 1% disadvantage, if you play long enough (or with high enough stakes), your probability of winning plummets to **$2\%$**. The longer you play an unfair game, the more certain your ruin becomes.

To help you visualize just how fast the odds turn against you, I've built an interactive calculator using the exact equations from your notes. Try sliding the win probability from a fair 50% to a casino-style 49% or 48% to see how drastically your survival chances drop.

*(Note: The very bottom of your page briefly introduces a **Random Variable** as a function that assigns a number to an outcome—like saying "Heads = 1, Tails = 0"—and defines a **Bernoulli distribution** as an event with only two outcomes, 1 or 0, success or failure. This is just vocabulary setting you up for the next lecture!)*