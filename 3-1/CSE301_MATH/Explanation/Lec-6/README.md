Ah, the legendary **Monty Hall Problem**! This is one of the most famous counter-intuitive brain teasers in history.

Let's break down this entire lecture note step-by-step so you can see exactly why switching doors doubles your chances of winning the car.

---

### The Game Setup

You are on a game show with 3 closed doors:

* **1 door** hides a brand-new **Car** 🚗
* **2 doors** hide **Goats** 🐐🐐

**The Rules:**

1. You pick a door (let's assume you pick **Door 1**).
2. The host, Monty, *knows* what is behind every door. He will always open a *different* door that reveals a goat.
3. If he has a choice between two goat doors (because you picked the car), he flips a coin ($1/2$ chance each).
4. Monty then asks the ultimate question: **Should you switch doors?**

---

### Part 1: Mapping out the Scenarios (The Tree Diagram)

Let's trace out every path that can happen when you choose **Door 1**.

* **Path 1: The Car is behind Door 1** (Probability = $\frac{1}{3}$)
* Since you picked the car, both remaining doors (2 and 3) have goats.
* Monty flips a coin to choose which goat to show you.
* Chance Monty opens Door 2: $\frac{1}{3} \times \frac{1}{2} = \mathbf{\frac{1}{6}}$
* Chance Monty opens Door 3: $\frac{1}{3} \times \frac{1}{2} = \mathbf{\frac{1}{6}}$


* **Path 2: The Car is behind Door 2** (Probability = $\frac{1}{3}$)
* You picked Door 1 (goat). Door 2 has the car. Door 3 has a goat.
* Monty *must* open a goat door that isn't yours. He is forced to open Door 3.
* Chance Monty opens Door 3: $\frac{1}{3} \times 1 = \mathbf{\frac{1}{3}}$


* **Path 3: The Car is behind Door 3** (Probability = $\frac{1}{3}$)
* You picked Door 1 (goat). Door 3 has the car. Door 2 has a goat.
* Monty is forced to open Door 2.
* Chance Monty opens Door 2: $\frac{1}{3} \times 1 = \mathbf{\frac{1}{3}}$



---

### Part 2: The Calculation (Conditional Probability)

Let's say Monty opens **Door 2** to show you a goat. Should you switch to Door 3?
We use conditional probability to find the chance of winning if we switch:

$$\text{P(Success if we switch} \mid \text{Monty opens Door 2)}$$

If Monty opens Door 2, we are in one of two worlds from our tree diagram above:

1. **World A:** You picked the car (Door 1), and Monty opened Door 2 by choice (Chance = $\frac{1}{6}$). If you switch now, you **lose**.
2. **World B:** The car is actually behind Door 3, and Monty was forced to open Door 2 (Chance = $\frac{1}{3}$). If you switch now, you **win**.

Let's plug these into our conditional probability formula:


$$\text{P(Car is at 3} \mid \text{Opened 2)} = \frac{\text{Chance of World B}}{\text{Chance of World A} + \text{Chance of World B}}$$

$$\text{P(Car is at 3} \mid \text{Opened 2)} = \frac{\frac{1}{3}}{\frac{1}{6} + \frac{1}{3}} = \frac{\frac{2}{6}}{\frac{3}{6}} = \mathbf{\frac{2}{3}}$$

By switching, your chance of winning jumps from $\frac{1}{3}$ to **$\frac{2}{3}$ (66.7%)**!

---

### Part 3: The Overall Success Rate (Law of Total Probability)

The bottom part of your note calculates the overall probability of winning if your strategy is *always* to switch.

Let:

* $S = \text{Success (winning the car by switching)}$
* $D_1, D_2, D_3 = \text{The car is behind Door 1, 2, or 3 respectively (Each has a } \frac{1}{3} \text{ chance)}$

Using the **Law of Total Probability**:


$$P(S) = P(D_1)P(S\vert{}D_1) + P(D_2)P(S\vert{}D_2) + P(D_3)P(S\vert{}D_3)$$

* If the car is at $D_1$ (you initially picked it), switching makes you lose: $P(S\vert{}D_1) = 0$
* If the car is at $D_2$ (you picked a goat), switching makes you win: $P(S\vert{}D_2) = 1$
* If the car is at $D_3$ (you picked a goat), switching makes you win: $P(S\vert{}D_3) = 1$

Now, just plug in the numbers and calculate:


$$P(S) = \left(\frac{1}{3} \times 0\right) + \left(\frac{1}{3} \times 1\right) + \left(\frac{1}{3} \times 1\right)$$

$$P(S) = 0 + \frac{1}{3} + \frac{1}{3} = \mathbf{\frac{2}{3}}$$

### The Big Intuition Takeaway

Why does this happen? When you first pick a door, you have a **$\frac{2}{3}$** chance of picking a **goat**. Because Monty *must* reveal the other goat, every time you initially pick a goat, switching wins you the car. Since you pick a goat $\frac{2}{3}$ of the time at the start, switching wins $\frac{2}{3}$ of the time!



---


This is one of the most mind-bending concepts in statistics: **Simpson's Paradox**. It happens when a trend appears in different groups of data but completely disappears or reverses when these groups are combined.

Your notes use a classic example featuring characters from *The Simpsons*—Dr. Hibbert and Dr. Nick—to illustrate this perfectly. Let's break down the tables and the math so it makes complete sense.

### 1. Decoding the Tables (The Raw Numbers)

The tables compare the success ($S$) and failure ($F$) rates of two doctors performing two types of procedures: Heart surgery (hard) and applying a Bandage (easy).

**Dr. Hibbert's Record:**

* **Heart Surgery:** 70 Successes, 20 Failures. Total = 90 patients.
* Success Rate = $70 / 90 \approx \mathbf{77\%}$


* **Bandage:** 10 Successes, 0 Failures. Total = 10 patients.
* Success Rate = $10 / 10 = \mathbf{100\%}$


* **OVERALL:** 80 total Successes, 20 total Failures out of 100 patients.
* Overall Success Rate = $80 / 100 = \mathbf{80\%}$



**Dr. Nick's Record:**

* **Heart Surgery:** 0 Successes, 10 Failures. Total = 10 patients.
* Success Rate = $0 / 10 = \mathbf{0\%}$


* **Bandage:** 81 Successes, 9 Failures. Total = 90 patients.
* Success Rate = $81 / 90 = \mathbf{90\%}$


* **OVERALL:** 81 total Successes, 19 total Failures out of 100 patients.
* Overall Success Rate = $81 / 100 = \mathbf{81\%}$



---

### 2. The Paradox Revealed

Look closely at the percentages we just calculated:

1. If you need **heart surgery**, who do you want? Dr. Hibbert (77%) is vastly better than Dr. Nick (0%).
2. If you need a **bandage**, who do you want? Dr. Hibbert (100%) is better than Dr. Nick (90%).
3. But if you look at their **overall** success rates without knowing the procedure, Dr. Nick (81%) looks better than Dr. Hibbert (80%)!

**The Paradox:** Dr. Hibbert is better at *every individual procedure*, but Dr. Nick looks like the better doctor *overall*.

### 3. Why does this happen? (The Hidden Variable)

The paradox is an illusion caused by a "confounding variable"—in this case, the **difficulty of the procedure** mixed with **unequal group sizes**.

Dr. Nick's overall score looks great because he almost exclusively takes on the easy cases (90 bandage patients) and avoids the hard ones (only 10 heart patients). Dr. Hibbert is taking on all the difficult, high-risk heart surgeries (90 patients), which naturally have a lower success rate, pulling his overall average down.

When you combine the data, the easy cases overwhelm the hard cases, skewing the total average.

---

### 4. Translating the Math Notation

The bottom half of your notes puts this exact scenario into formal probability notation.

* $A$ = Successful surgery
* $B$ = Treated by Dr. Nick ($B^c$ means "Not Nick," which is Dr. Hibbert)
* $C$ = Heart surgery ($C^c$ means "Not Heart," which is Bandage)

Here is how to read those formulas:

* $P(A\vert{}B, C) < P(A\vert{}B^c, C)$: The probability of success given Nick AND Heart is **less than** success given Hibbert AND Heart. (0% < 77%).
* $P(A\vert{}B, C^c) < P(A\vert{}B^c, C^c)$: The probability of success given Nick AND Bandage is **less than** success given Hibbert AND Bandage. (90% < 100%).
* **But,** $P(A\vert{}B) > P(A\vert{}B^c)$: The overall probability of success with Nick is **greater than** with Hibbert. (81% > 80%).

The final equation uses the **Law of Total Probability** to show exactly how Dr. Nick's overall success rate $P(A\vert{}B)$ is calculated by weighing his success in each category by how often he performs that category.
Let’s unpack that exact equation from the bottom of your notes and do the math for Dr. Nick.

### The Equation Setup

We want to calculate $P(A\vert{}B)$ — which is the overall probability of a successful surgery ($A$) given that the doctor is Dr. Nick ($B$).

Your notes use the **Law of Total Probability** to split this into two parts: his heart surgeries ($C$) and his bandage procedures ($C^c$).

Here is the formula from your notes:


$$P(A\vert{}B) = P(C\vert{}B)P(A\vert{}B,C) + P(C^c\vert{}B)P(A\vert{}B,C^c)$$

### Translating the Math into Numbers

Let's pull the numbers directly from Dr. Nick's table for each part of this equation:

**1. The Heart Surgery Part: $P(C\vert{}B)P(A\vert{}B,C)$**

* $P(C\vert{}B)$: Out of all of Dr. Nick's patients, what fraction got heart surgery? He saw $100$ patients total, and $10$ had heart surgery.
* $P(C\vert{}B) = 10/100 = \mathbf{0.10}$


* $P(A\vert{}B,C)$: When Dr. Nick does heart surgery, what is his success rate? He had $0$ successes out of $10$.
* $P(A\vert{}B,C) = 0/10 = \mathbf{0}$


* Multiply them: $0.10 \times 0 = \mathbf{0}$

**2. The Bandage Part: $P(C^c\vert{}B)P(A\vert{}B,C^c)$**

* $P(C^c\vert{}B)$: Out of all of Dr. Nick's patients, what fraction got a bandage? He saw $100$ patients total, and $90$ got a bandage.
* $P(C^c\vert{}B) = 90/100 = \mathbf{0.90}$


* $P(A\vert{}B,C^c)$: When Dr. Nick applies a bandage, what is his success rate? He had $81$ successes out of $90$.
* $P(A\vert{}B,C^c) = 81/90 = \mathbf{0.90}$


* Multiply them: $0.90 \times 0.90 = \mathbf{0.81}$

### The Final Calculation

Now we just add the two parts together according to the formula:


$$P(A\vert{}B) = 0 + 0.81 = \mathbf{0.81}$$

**0.81 is exactly the 81%** overall success rate shown in Dr. Nick's table!

This equation proves mathematically *why* Dr. Nick's overall score is so high: the formula heavily **weights** his 90% bandage success rate (because he does it $90\%$ of the time) and practically ignores his $0\%$ heart surgery success rate (because he only does it $10\%$ of the time).