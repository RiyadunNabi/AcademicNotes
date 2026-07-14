# Problem Solving by Searching — From Absolute Zero

Riyad, this covers Chapter 3 of your CSE317/318 AI course (AIMA-based). Since this is exam material with strict terminology marking, I'll define every term precisely the way AIMA and your slides use it. Let's build this up piece by piece.

---

## 1. What is a "Problem-Solving Agent"?

**Definition:** An **intelligent agent** is just a program that perceives its environment and decides on actions. A **problem-solving agent** is a specific *type* of intelligent agent that achieves its goals by **searching** through a **state-space** — a structured representation of "all the situations the world could be in."

Think of it like this: imagine you're in Arad (a city in Romania) and you need to get to Bucharest. You don't teleport there — you mentally simulate different routes ("if I go to Sibiu, then Fagaras, then Bucharest...") before actually driving. That mental simulation over possible routes *is* search.

**Why search at all?** Because the agent doesn't have a direct action that solves the problem in one step. It has to find a **sequence of actions** that gets it from where it is now to where it wants to be.

> **Takeaway:** A problem-solving agent doesn't act blindly — it first searches through possible action-sequences in its head, then executes the one that works.

---

## 2. The State-Space Model

**Definition:** The **state-space** is the agent's internal model of "the world" — a (usually) discrete set of distinct situations called **states**.

- Example: In the Romania driving problem, each **state** = "being in a particular city" (Arad, Sibiu, Bucharest, etc.)
- The state-space is *not* the real world — it's an **abstraction** (more on this in Section 5) of the real world, simplified enough for a computer to reason about.

**Why discrete states?** Computers reason more easily over a finite, enumerable set of possibilities than over continuous, infinite reality. If we tried to model *every* physical detail of "being in Arad" (exact GPS coordinates, tire pressure, etc.), the problem would be intractable. So we throw away irrelevant detail and keep only what matters for solving the problem — that's abstraction.

> **Takeaway:** The state-space is a simplified map of possible situations, not the real world itself.

---

## 3. Goal States

**Definition:** A **goal** is a state (or set of states) that the agent wants to reach. The **goal test** is the function that checks whether a given state counts as a goal.

Two flavors, both from your slides (page 3):

| Type | Example | Meaning |
|---|---|---|
| **Single goal state** | "Drive to Mammoth" | Exactly one state satisfies the goal |
| **Set of goal states** | "Drive to a town with a ski resort" | Many different states could satisfy the goal — any one of them is acceptable |

**Why this distinction matters:** the goal test can be a simple equality check ("is current city == Bucharest?") or a more complex predicate ("does this city have a ski resort?"). AIMA's chess example: `Checkmate(x)` — you don't compare to *one* fixed checkmate position, you test *any* state where the property "checkmate" holds.

> **Takeaway:** A goal isn't always one specific state — it can be any state satisfying some condition.

---

## 4. Operators (Actions / Successor Function)

**Definition:** **Operators** (also called **actions** or the **successor function**) are the legal moves that transform one state into another.

Formally: `S(x)` = the set of `<action, resulting-state>` pairs available from state `x`.

Example from your slides:
```
S(Arad) = { <Arad→Zerind, Zerind>, <Arad→Sibiu, Sibiu>, <Arad→Timisoara, Timisoara> }
```
This reads as: "From Arad, you can take the action 'drive to Zerind' and end up in state Zerind; or take 'drive to Sibiu' and end up in Sibiu," etc.

**Why formalize it this way?** Because a search algorithm needs a *mechanical* way to generate all possible next moves from any given state — it can't "look at a map" the way a human does. The successor function is exactly that mechanical rule.

> **Takeaway:** Operators are the rules that generate new states from old ones — they define the "edges" of your search graph.

---

## 5. Abstraction (WHY we simplify)

**Definition (from your slides, page 8):** Abstraction is *"the process of removing irrelevant detail to create a high-level representation."*

**Why it's necessary:** The real world is too detailed to model exactly. If you tried to include every physical variable when planning a road trip (weather, exact fuel level, tire wear...), the search space would explode and become computationally useless. So you strip away detail and keep only what's *relevant to solving the problem*.

**Example:** For the Romania navigation problem:
- Abstracted model: nodes = cities, edges = roads. Ignore things like fuel stops, traffic lights, on-ramps.
- These details *can* be added later if needed (e.g., a more refined sub-problem), but they're not needed to find *which cities to pass through*.

**Failure case / limitation:** If you abstract away *too much*, you might lose information critical to correctness. For example, if two cities are connected by a road but it's seasonally closed (a real-world detail), and your abstraction ignores this, your "solution" could be physically infeasible. **Good abstractions retain all details relevant to the task, and only the relevant ones.**

> **Takeaway:** Abstraction is a deliberate trade-off — simplify enough to make search tractable, but not so much that you lose solution-critical details.

---

## 6. Formal Definition of a "Problem" (the core of this chapter)

This is the single most exam-relevant slide (page 6). A **problem** is formally defined by exactly **four components**:

| # | Component | Meaning | Romania Example |
|---|---|---|---|
| 1 | **Initial state** | Where the agent starts | "at Arad" |
| 2 | **Actions / Successor function** `S(x)` | Set of `<action, state>` pairs reachable from `x` | `S(Arad) = {<Arad→Zerind, Zerind>, ...}` |
| 3 | **Goal test** | Function that checks if a state is a goal | `x == "at Bucharest"` |
| 4 | **Path cost** | An **additive** function assigning a numeric cost to a sequence of actions | Sum of distances traveled |

**Path cost detail:** The path cost function is built from a **step cost** `c(x, a, y)` — the cost of going from state `x` to state `y` via action `a`. It's assumed `c(x,a,y) ≥ 0` (no negative costs — this assumption matters later for algorithms like Uniform Cost Search and A*, which break if costs can be negative).

**Definition — Solution:** A **solution** is a sequence of actions that leads from the initial state to a goal state. An **optimal solution** is the one with the lowest path cost among all solutions.

**Why exactly these four?** Because together, they contain *everything* a generic search algorithm needs: where to start (1), how to move (2), how to know when to stop (3), and how to judge solution quality (4). Any problem you can express this way becomes solvable by a general-purpose search algorithm, without the algorithm needing any special knowledge about *what* the problem is about (driving, puzzles, robotics — doesn't matter).

> **Takeaway:** Any search problem = (initial state, actions, goal test, path cost) — memorize this 4-tuple, it's the backbone of the entire chapter.

---

## 7. The State-Space Graph

Once you have a formal problem, you can visualize it as a **graph**:

- **Nodes** = states
- **Directed arcs (edges)** = operators/actions (directed because going from A→B isn't necessarily the same action as B→A)
- **A solution** = a **path** from the start node `S` to a goal node `G`

**Two distinct phases** (page 9, important terminology distinction your exam might test):
1. **Problem formulation** — describing states, operators, initial state, goal abstractly (the *definition* stage).
2. **Problem solving** — actually generating enough of the search space (via a search algorithm) to find a path to the goal (the *execution* stage).

**Why separate these two?** Because the graph is usually far too large to write out in full (e.g., 8-puzzle has 181,440 reachable states; chess has ~10^40+). You never build the *whole* graph — you generate only the portion of it a search algorithm actually needs to explore.

> **Takeaway:** The state-space graph is rarely built explicitly — it's generated node-by-node, on demand, by the search algorithm.

---

## 8. Worked Examples (applying the 4-tuple)

This is where AIMA drills the formalism into you — by forcing you to define states/actions/goal-test/path-cost for wildly different problems. Let's go through each from your slides.

### 8.1 The 8-Puzzle

| Component | Definition |
|---|---|
| **States** | Locations of the 8 numbered tiles + blank on the 3×3 grid |
| **Initial state** | Whatever starting arrangement is given |
| **Actions** | Move blank Left, Right, Up, Down |
| **Goal test** | Does current arrangement match the given goal arrangement? |
| **Path cost** | 1 per move |

**Note the subtlety:** Actions are described as moving the *blank*, not moving numbered tiles — this is a modeling choice that makes the action set uniform (always ≤4 possible moves) regardless of which tile numbers surround the blank.

### 8.2 8-Queens Problem

**Goal:** Place 8 queens on a chessboard so none attacks another (no two share a row, column, or diagonal).

Two competing formulations shown in your slides — this is a great example of how the *same* problem can be formulated differently, with real performance consequences:

| Formulation | States | Actions |
|---|---|---|
| **Incremental (naive)** | Any arrangement of ≤8 queens anywhere on board | Add a queen to *any* empty square |
| **Incremental (smarter)** | Arrangements of ≤8 queens confined to leftmost *n* columns, 1 per column, non-attacking | Add queen to leftmost empty column, only in a non-attacked square |

**Why does the smarter formulation matter?** It shrinks the search space enormously by building in constraints (never place an attacked queen) directly into the action definition, rather than generating invalid states and filtering them out later. This is a recurring theme in AI: **how you formulate the problem massively affects how hard it is to solve** — even before you pick a search algorithm.

- **Goal test:** 8 queens on board, none attacking.
- **Path cost:** 1 per move (irrelevant here really, since we only care about reaching *any* valid goal, not the cheapest path).

### 8.3 Robot Assembly

| Component | Definition |
|---|---|
| **States** | Configuration of the robot (joint angles, positions) *and* the object parts |
| **Initial state** | Any starting configuration |
| **Actions** | **Continuous** motion of robot joints |
| **Goal test** | Is the object fully assembled? |
| **Path cost** | Time taken, or number of actions |

**Important limitation flagged here:** unlike the 8-puzzle (discrete states, discrete actions), robot assembly has a **continuous** action space (joint angles are real numbers, not a finite set of moves). This breaks the "simplifying assumption" from page 3 (discrete environment/actions) — so classic discrete search algorithms don't directly apply; you'd need to discretize the space or use different techniques (e.g., motion planning algorithms).

### 8.4 Spam Email Classifier (as a search/optimization problem)

| Component | Definition |
|---|---|
| **States** | Settings of the parameters in your model |
| **Initial state** | Random parameter settings |
| **Actions** | Moving in parameter space |
| **Goal test** | Optimal accuracy on training data |
| **Path cost** | Time taken to find optimal parameters |

**Why this example matters (and connects to your ML interest):** This is explicitly flagged in your slides as an **optimization problem** — many machine learning problems (including the gradient descent you'll see in Andrew Ng's course) can be reframed as search/optimization: "search" over parameter space for the setting that minimizes error. This is the same abstract 4-tuple framework, just applied somewhere unexpected. It shows the *generality* of the search framework — that's the whole point of this chapter.

### 8.5 The Water Jug Problem

Classic AIMA/Russell-Norvig style puzzle: 4-gallon jug + 3-gallon jug, unlimited water supply, goal = exactly 2 gallons in the 4-gallon jug.

**State representation:** `(x, y)` where `x` = contents of 4-gallon jug, `y` = contents of 3-gallon jug.
- **Start state:** `(0, 0)`
- **Goal state:** `(2, n)` for any `n` (the 3-gallon jug's content doesn't matter)

**Operators** (this is the classic example of writing out **production rules** — condition → action):

| Rule | Effect | Condition |
|---|---|---|
| 1 | `(x,y) → (4,y)` | Fill 4-gal jug, if `x<4` |
| 2 | `(x,y) → (x,3)` | Fill 3-gal jug, if `y<3` |
| 3 | `(x,y) → (x−d,y)` | Pour water out of 4-gal, if `x>0` |
| ... | ... | ... |
| 7 | `(x,y) → (4, y−(4−x))` | Pour 3-gal into 4-gal until 4-gal full, if `x+y≥4, y>0` |
| 9 | `(x,y) → (x+y, 0)` | Pour *all* of 3-gal into 4-gal, if `x+y≤4, y>0` |

**Why write rules this explicitly?** This is the clearest illustration of what "operators" *actually* are in implementation terms: a **precondition** (when is this rule legal to apply?) and an **effect** (what does the state become?). This precondition→effect pattern is exactly what you'll re-encounter in classical planning (STRIPS-style) later in AI.

**One example solution path (from your slides):**

| 4-gal | 3-gal | Rule Applied |
|---|---|---|
| 0 | 0 | — |
| 0 | 3 | Rule 2 (fill 3-gal) |
| 3 | 0 | Rule 7 (pour 3→4) |
| 3 | 3 | Rule 2 (fill 3-gal) |
| 4 | 2 | Rule 7 (pour 3→4) |
| 0 | 2 | Rule 5 (empty 4-gal) |
| 2 | 0 | Rule 7 (pour 3→4) ✅ Goal! |

> **Takeaway:** The water jug problem shows operators can be written as precise condition→effect production rules — this pattern generalizes to any search problem.

### 8.6 Traveling Salesperson Problem (TSP)

- **State:** a sequence of cities visited so far
- **Start state:** `S0 = A` (just the starting city)
- **Goal:** a complete tour (visits every city exactly once, returns to start)
- **Path cost:** total tour distance (this is the interesting part — TSP asks for the *cheapest* complete tour, not just *any* one)

**Why TSP is famous/hard:** the number of possible tours grows factorially with the number of cities — this is a classic **NP-hard** problem, meaning no known algorithm solves it efficiently for large city counts. It's included here to show that even though the 4-tuple formalism applies cleanly, *formulating* a problem doesn't mean it's *easy to solve*.

### 8.7 VLSI Layout Problem

Real-world engineering problem: positioning millions of circuit components on a chip to minimize area/delay/capacitance and maximize manufacturing yield.

- Split into **cell layout** (grouping components into functional cells, placing them without overlap) and **channel routing** (finding wire paths between cells).

**Why it's mentioned:** to show search isn't just for puzzles/games — it scales (conceptually) to real engineering problems with **millions** of variables. In practice, the state-space is far too large for basic search algorithms; VLSI design uses specialized heuristics and optimization techniques.

---

## 9. Summary Table: All the Examples Side-by-Side

| Problem | States | Actions | Discrete or Continuous? |
|---|---|---|---|
| Romania navigation | Cities | Drive between cities | Discrete |
| 8-puzzle | Tile arrangement | Move blank U/D/L/R | Discrete |
| 8-queens | Queen placements | Add a queen | Discrete |
| Robot assembly | Joint angles + part positions | Continuous joint motion | **Continuous** |
| Spam classifier | Model parameters | Move in parameter space | Continuous (optimization) |
| Water jug | `(x,y)` jug contents | Fill/empty/pour rules | Discrete |
| TSP | Sequence of visited cities | Visit next city | Discrete, but combinatorially huge |
| VLSI layout | Component positions | Place/route components | Mixed, huge scale |

**Why this table matters for your exam:** it shows the *generality* of the problem-solving-agent framework — the same four-part definition (states, actions, goal test, path cost) applies to driving directions, puzzles, robotics, and machine learning alike. But it also shows the framework's **limits**: it works cleanly for discrete problems; continuous or astronomically large state-spaces need extra machinery (discretization, heuristics, or entirely different techniques) beyond basic uninformed search.

---

## 10. What Comes Next (per your slides)

This chapter sets up three families of algorithms you'll study next:
1. **Uninformed search** — BFS, DFS, Uniform Cost, Iterative Deepening (no domain knowledge, just explores blindly)
2. **Informed (heuristic) search** — Greedy best-first, A*, memory-bounded variants (uses an estimate of "how close to the goal" to guide search)
3. **Local search / optimization** — Hill-climbing, Simulated Annealing, Genetic Algorithms (doesn't track a path at all — just tries to find a good state directly, useful for problems like 8-queens or the spam classifier example)

> **Overall chapter takeaway:** Before you can search for a solution, you must first *formulate* the problem precisely as (initial state, actions, goal test, path cost) over an abstracted state-space — everything else in the AI search unit builds on getting this formulation right.

---

Since your slides mention this is the AIMA-based, exact-terminology-marked half of CSE317/318 — want me to make a quick self-test (fill-in-the-blank style, matching the "states? actions? goal test? path cost?" format from the slides) for a couple of *new* problems not in this deck, so you can practice the formulation pattern cold?