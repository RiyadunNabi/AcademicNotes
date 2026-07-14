# Uninformed Search — From Absolute Zero

This is the direct follow-up to Chapter 3 (Problem Solving by Searching), Riyad — now you have a formal problem definition, so this chapter asks: **given that formal definition, how do we actually search through the state-space to find a solution?**

---

## 1. What Does "Uninformed" Mean?

**Definition:** An **uninformed** (or "blind") search strategy has **no extra information** about how far a given state is from the goal — beyond what's given in the problem definition itself (initial state, actions, goal test, path cost).

**Analogy:** Imagine you're lost in an unfamiliar city with no map, no GPS, no signs pointing toward your destination. All you can do is systematically try streets — you have no sense of "getting warmer" or "getting colder." That's uninformed search. (Contrast this with **informed search**, covered next chapter, where you'd have something like a compass telling you roughly which direction the goal is.)

**Four uninformed strategies covered in this chapter:**
1. Breadth-first search (BFS)
2. Uniform-cost search (UCS)
3. Depth-first search (DFS)
4. Iterative deepening search (IDS)

> **Takeaway:** Uninformed search explores blindly — it can't tell a "promising" state from a "useless" one, it just follows a fixed exploration order.

---

## 2. Core Vocabulary You Need First

Before the algorithms make sense, you need four terms nailed down:

| Term | Definition |
|---|---|
| **Node** | A data structure representing a state *plus* bookkeeping info (which state generated it, what action led to it, path cost so far, depth) — not the same thing as a "state"! Two different nodes can represent the same state (e.g., reached via different paths). |
| **Expand** | Take a node, apply the successor function to it, and generate all its child nodes (successors). |
| **Fringe** (also called the **frontier** or **open list**) | The set of all generated-but-not-yet-expanded nodes. This is literally the "edge" of how far your search has explored. |
| **Branching factor (b)** | The maximum number of successors (children) any single node can have. |

**Why the node/state distinction matters:** In the 8-puzzle, the *state* is just "where are the tiles right now." But the *node* also remembers "I got here by moving the blank left, from this parent node, at depth 5, with cost 5." You need that bookkeeping to reconstruct the final solution path once you hit a goal.

> **Takeaway:** Every search algorithm in this chapter is really just "repeatedly pick a node from the fringe, expand it, add children back to the fringe" — the *only* thing that changes between algorithms is which node gets picked next.

---

## 3. The Four Performance Metrics (memorize these — heavily exam-tested)

Every search strategy is judged on exactly these four criteria (page 2 of your slides):

| Metric | Question it answers |
|---|---|
| **Completeness** | Is the algorithm *guaranteed* to find a solution, if one exists? |
| **Optimality** | Does it find the *best* (lowest path-cost) solution, or just *any* solution? |
| **Time complexity** | How many nodes get generated/expanded before finding a solution? |
| **Space complexity** | How much memory (max nodes held at once) does it need? |

**Why these four specifically?** Because a search algorithm is only useful in practice if it (a) actually terminates with an answer, (b) gives you a *good enough* answer, and doesn't (c) take forever or (d) run out of RAM. These four axes capture all the practical trade-offs you'll see across BFS, DFS, UCS, and IDS.

**Standard variables you'll see in complexity formulas:**
- `b` = branching factor
- `d` = depth of the *shallowest* goal node
- `m` = maximum depth of the state-space (could be infinite!)
- `l` = the depth limit (used in depth-limited search)

> **Takeaway:** Whenever you're asked to "evaluate" a search algorithm, you must address all four: complete? optimal? time? space? — not just one of them.

---

## 4. Breadth-First Search (BFS)

**Definition:** Expand the **shallowest** unexpanded node first — i.e., explore level by level, like ripples spreading out from a stone dropped in water.

**Implementation detail (this is the key mechanical fact):** the fringe is a **FIFO queue** (First-In-First-Out) — newly generated successor nodes go to the **back** of the queue, and you always pull from the **front**.

**Why FIFO produces level-by-level exploration:** Think of it like a queue at a shop counter — the first person to join is the first one served. If you add all of node A's children to the back of the queue, they won't be processed until every node already in the queue (i.e., everything at shallower or equal depth) has been processed first. This naturally enforces "finish this depth level before moving to the next."

**Walkthrough (from your slides, tree with root A, children B/C, grandchildren D/E/F/G):**

| Step | Fringe (queue) | Node just checked |
|---|---|---|
| Start | [A] | Is A goal? |
| Expand A | [B, C] | Is B goal? |
| Expand B | [C, D, E] | Is C goal? |
| Expand C | [D, E, F, G] | Is D goal? |

Notice: B and C (depth 1) are *both* fully processed before D, E, F, G (depth 2) even get touched. That's the level-by-level signature of BFS.

### Properties of BFS

| Metric | Result | Why |
|---|---|---|
| **Complete?** | Yes (if `b` is finite) | Since it explores every node at depth `k` before depth `k+1`, it *cannot* miss a goal that exists at some finite depth. |
| **Time** | `O(b^(d+1))` | You generate roughly `1 + b + b² + ... + b^d` nodes to reach depth `d`. |
| **Space** | `O(b^(d+1))` | This is the killer: BFS must hold *every* generated node in memory (either in the fringe or as an ancestor of a fringe node), because it processes breadth-wise, not depth-wise. |
| **Optimal?** | Yes — *but only if* step-cost is uniform (e.g., constant, like "1 per move") | If some edges cost more than others, a *shallower* goal isn't guaranteed to be *cheaper* — BFS only guarantees shortest in terms of *number of steps*, not total cost. |

**Failure case / limitation:** Space is the real killer, not time. For `b=10, d=5`, BFS needs to store ~1,111,100 nodes (see the IDS comparison table later) — that's often too much RAM well before it becomes too slow.

> **Takeaway:** BFS finds the shallowest solution and is complete + optimal (for uniform costs), but its memory appetite grows exponentially with depth — this is almost always the bottleneck in practice.

---

## 5. Uniform-Cost Search (UCS)

**The problem UCS solves:** BFS is only optimal when step costs are uniform. What if some actions cost more than others (e.g., driving 140 km vs. 75 km between two cities)? BFS could return a "shallow" path that's actually *more expensive* than a deeper one.

**Definition:** Expand the node with the **smallest cumulative path cost** `g(n)` — where `g(n)` = sum of step costs from the start state to node `n`.

**Implementation:** fringe = a **priority queue** ordered by `g(n)` (not FIFO like BFS — always pull out whichever fringe node currently has the lowest total path cost).

**Concrete example (from your slides, Figure 3.13):** Start state S, with edges to A (cost 1), B (cost 5), C (cost 15). But A itself leads to a goal G with an *additional* cost of 10 (total: 1+10=11). Meanwhile B leads to goal G with additional cost 5 (total: 5+5=10). Even though A is reached "cheaper" initially (cost 1 vs 5), the path through B ends up being the cheaper *complete* solution (cost 10 vs 11). UCS correctly picks the B-route as optimal because it always compares *total accumulated cost*, not just how "shallow" a node is.

**Relationship to BFS:** If all step costs are equal, UCS behaves *identically* to BFS (since minimizing "cost so far" becomes equivalent to minimizing "depth so far").

### Properties of UCS

| Metric | Result | Why |
|---|---|---|
| **Complete?** | Yes, *if* step cost ≥ some small positive number ε (epsilon) | If step costs could be zero (or worse, negative), the algorithm could loop through zero-cost edges forever without ever being "forced" to increase `g(n)` enough to explore other branches — it could get stuck in an infinite loop, never terminating. |
| **Time** | Number of nodes with path cost ≤ cost of optimal solution | Roughly `O(b^⌈C*/ε⌉)` where `C*` = cost of the optimal solution. |
| **Space** | Same as time — must store all those nodes | UCS has the same memory problem as BFS, just measured in cost-radius rather than depth-radius. |
| **Optimal?** | Yes, for *any* step cost (as long as ≥ 0) | This is the whole reason UCS exists — it fixes BFS's optimality problem when costs vary. |

> **Takeaway:** UCS is BFS's generalization to handle varying step costs — always expand the cheapest total-cost node, guaranteeing optimality regardless of cost structure (as long as costs are non-negative and bounded away from zero).

---

## 6. Depth-First Search (DFS)

**Definition:** Expand the **deepest** unexpanded node first — dive as far down one branch as possible before backtracking.

**Implementation:** fringe = a **LIFO queue** (Last-In-First-Out — essentially a **stack**). New successors go to the **front**, and you always pull from the front too — so you always process whatever was *most recently* added.

**Why LIFO produces depth-first behavior:** Imagine a stack of plates — you can only take from the top. If node A generates children B and C, and you push them onto the stack, B (say, pushed last, sits on top) gets expanded next — generating *its* children D and E, which get pushed on top of C. Now D gets expanded next, not C — because D is now on top. This creates a "dive deep down one path, only come back up when you hit a dead end" pattern.

**Walkthrough (from your slides):**

| Step | Queue (stack) | Node checked |
|---|---|---|
| Start | [A] | Is A goal? |
| Expand A | [B, C] | Is B goal? |
| Expand B | [D, E, C] | Is D goal? |
| Expand D | [H, I, E, C] | Is H goal? |

Notice: after expanding A → B, we go straight to expanding B → D (depth 2) rather than touching C (which is still at depth 1, sitting untouched at the back of the stack). That's the signature "dive first" behavior — completely different from BFS's "finish this level first."

### Properties of DFS

| Metric | Result | Why |
|---|---|---|
| **Complete?** | **No** — fails in infinite-depth (or very deep) spaces | If DFS picks a branch that goes infinitely deep (or loops) without ever hitting a goal, it will never backtrack to try other branches — it just keeps diving forever. (Can be partially fixed by not allowing repeated states along the current path.) |
| **Time** | `O(b^m)` where `m` = maximum depth of the space | **Terrible if `m` is much larger than `d`** (the actual goal depth) — you could dive down a very deep, goal-free branch for a long time before backtracking. *However*, if solutions are densely distributed throughout the tree, DFS can be much faster than BFS in practice, since it might stumble onto *a* solution quickly without exploring the whole breadth. |
| **Space** | `O(bm)` — **linear**, not exponential! | This is DFS's big advantage: you only need to remember the *single current path* from root to current node, plus the unexplored siblings along that path — not the entire fringe like BFS. |
| **Optimal?** | **No** | DFS will happily return the *first* goal it stumbles into, even if a shallower/cheaper goal exists elsewhere in the tree that it hasn't looked at yet. |

**Failure case example (VERY exam-relevant):** If the state space has cycles or infinite paths (e.g., an 8-puzzle-like problem where you could move a tile back and forth forever), a naive DFS can get stuck exploring one infinite branch and never terminate, even though a solution exists just one level over on a sibling branch.

> **Takeaway:** DFS trades away completeness and optimality guarantees in exchange for linear (not exponential) memory usage — the opposite trade-off from BFS.

---

## 7. Comparing BFS and DFS Directly

| Property | BFS | DFS |
|---|---|---|
| Fringe structure | FIFO queue | LIFO queue (stack) |
| Exploration pattern | Level by level | Dive deep, backtrack on dead ends |
| Complete? | Yes (finite b) | No |
| Optimal? | Yes (uniform cost) | No |
| Time | O(b^(d+1)) | O(b^m) |
| Space | O(b^(d+1)) — exponential | O(bm) — linear |

**The core tension:** BFS guarantees correctness (completeness + optimality) but costs you exponential memory. DFS is memory-cheap but gives up both guarantees. This tension is *exactly* what motivates the next algorithm.

---

## 8. Iterative Deepening Search (IDS)

**Motivating question (directly from your slides, page 26):** Can we get DFS's linear memory *and* BFS's completeness/optimality guarantees?

**Step 1 — Depth-Limited Search (DLS):** First, fix the infinite-depth problem of DFS by imposing a hard depth limit `L` — never expand any node beyond depth `L`. This guarantees termination (you can't dive forever), but now you have a new problem: what if the actual solution is *deeper* than `L`? You'd miss it entirely.

**Step 2 — Iterative Deepening:** Solve *that* problem by running depth-limited search repeatedly, increasing `L` one level at a time: first with `L=0`, then `L=1`, then `L=2`, and so on — until a goal is found.

**Definition (Iterative Deepening Search):** Run DLS with `L=0,1,2,3,...` in sequence, stopping as soon as a depth-limited search successfully finds a goal.

**Walkthrough intuition (from your slides' visual sequence, L=0,1,2,3):**
- `L=0`: only check the root node A. Not a goal? Give up, increase L.
- `L=1`: re-run DFS but only go 1 level deep (check A, B, C). Not found? Increase L.
- `L=2`: re-run DFS down to depth 2 (A, B, C, D, E, F, G). Not found? Increase L.
- `L=3`: now go to depth 3 (down to H, I, J, K, L, M, N, O)...

**Isn't this wasteful — re-exploring the same shallow nodes over and over?** Surprisingly, *not much*, and this is the key mathematical insight of the chapter (page 31 in your slides):

$$N_{DLS} = b^0 + b^1 + b^2 + ... + b^d$$
$$N_{IDS} = (d+1)b^0 + d \cdot b^1 + (d-1)b^2 + ... + 1 \cdot b^d$$

For `b=10, d=5`:
- `N_DLS = 111,111` (nodes generated in a single depth-limited search straight to depth 5)
- `N_IDS = 123,450` (nodes generated across *all* the repeated iterative-deepening passes, L=0 through L=5)
- `N_BFS = 1,111,100` (nodes generated by plain BFS to the same depth)

**Why is IDS so close to a single DLS pass, and nowhere near as bad as BFS?** Because the *deepest* level dominates the node count in an exponential tree — the bulk of all nodes live at depth `d`, and that layer is only generated *once* under IDS (during the final, `L=d` pass). The re-generation of shallower levels (0 through d-1) during earlier passes adds only a small overhead by comparison, since exponentially fewer nodes live at those shallower depths. This is why `O(b^d) ≠ O(b^(d+1))` — that one extra factor of `b` between BFS's asymptotic time and IDS's/DFS's asymptotic time is actually a *huge* practical gap when `b` is large.

### Properties of Iterative Deepening Search

| Metric | Result | Why |
|---|---|---|
| **Complete?** | Yes | Since `L` keeps increasing, it will *eventually* reach the true solution depth `d`, and — crucially — depth-limited search itself is complete for depth ≤ L. |
| **Time** | `O(b^d)` | As shown above — barely worse than a single depth-limited pass, and much better than BFS's `O(b^(d+1))`. |
| **Space** | `O(bd)` — **linear** | Because at any given moment, IDS is really just running a depth-limited DFS — and DFS only needs linear space (Section 6). |
| **Optimal?** | Yes, if step cost = 1 (or a non-decreasing function of depth) | Same optimality condition as BFS: since it explores shallowest solutions first (across successive `L` passes), and cost only increases (or stays flat) with depth, the first goal found at the current `L` is guaranteed cheapest. |

> **Takeaway:** Iterative Deepening Search combines DFS's linear space with BFS's completeness and optimality — making it the generally *preferred* uninformed strategy when the search space is large or the depth of the solution is unknown, despite the seemingly wasteful re-exploration.

---

## 9. Bidirectional Search

**Definition:** Simultaneously run search **forward** from the start state `S` *and* **backward** from the goal state `G`, stopping the moment the two search frontiers "meet in the middle."

**Why this helps (the math is the interesting part):** If a solution exists at depth `d`, running BFS forward alone costs `O(b^d)`. But if you search *forward* to depth `d/2` and *backward* to depth `d/2` simultaneously, each half costs only `O(b^(d/2))`, and — crucially — `2 × b^(d/2)` is vastly smaller than `b^d` for large `d`. This is why the complexity collapses to:

$$O(b^{d/2})$$

**Concrete illustration:** If `b=10, d=6`, forward-only BFS costs on the order of `10^6 = 1,000,000` nodes. Bidirectional search costs roughly `2 × 10^3 = 2,000` nodes — a massive improvement.

**Limitation/failure case (the reason this isn't used everywhere — this is exam-relevant):**
- Searching "backward from the goal" requires you to be able to compute the **predecessors** of the goal state — i.e., you need an *inverse* successor function. This is easy in some domains (e.g., reversible actions) but genuinely hard or ill-defined in others.
- Example from your slides: what are the predecessors of "checkmate" in chess? There's no simple, small, enumerable list — chess doesn't have a nicely invertible action structure.
- What if there are **multiple goal states**, or worse, only a **goal test** function (no explicit list of goal states at all)? Then you can't even start a backward search, since you don't know a concrete state to search backward *from*.

> **Takeaway:** Bidirectional search offers a dramatic `O(b^(d/2))` speedup over one-directional search, but only when the goal state(s) are explicitly known *and* actions can be meaningfully reversed — conditions that often don't hold in real problems.

---

## 10. Summary Table of All Algorithms (the single most exam-critical slide)

| Criterion | Breadth-First | Uniform-Cost | Depth-First | Depth-Limited | Iterative Deepening |
|---|---|---|---|---|---|
| **Complete?** | Yes | Yes | No | No | Yes |
| **Time** | O(b^(d+1)) | O(b^⌈C*/ε⌉) | O(b^m) | O(b^l) | O(b^d) |
| **Space** | O(b^(d+1)) | O(b^⌈C*/ε⌉) | O(bm) | O(bl) | O(bd) |
| **Optimal?** | Yes | Yes | No | No | Yes |

**Reading this table correctly:** Notice the two clusters — {BFS, UCS, IDS} are all *complete and optimal* but differ in exactly what they optimize for (depth vs. cost vs. memory-efficient depth), while {DFS, DLS} sacrifice both guarantees in exchange for far better (linear) space complexity.

> **Takeaway:** There's no free lunch among uninformed strategies — you're always trading off between guaranteed correctness (complete/optimal) and resource usage (time/space); IDS is usually the best default because it gets both guarantees at only linear space cost.

---

## 11. The Repeated States Problem

**The problem:** In many state-spaces, the *same* state can be reached via multiple different paths (e.g., in a grid, you can reach a cell by going right-then-up or up-then-right). If your search algorithm doesn't recognize "I've already been to this state," it will re-explore the *entire subtree* under that state again — potentially many times over.

**Why this is dangerous — concrete illustration from your slides:** Consider a state-space graph where state A connects to B, B connects to C, and so on, in a simple *linear* chain (a genuinely simple, "linear" problem). But if the underlying *search tree* representation doesn't collapse repeated states, each state might spawn duplicate branches — turning what should be a search over `n` states into a search over `2^n` tree nodes. **Failure to detect repeated states can turn a linear problem into an exponential one** — this is stated explicitly in your slides because it's a classic exam trap.

### Two Solutions

| Method | How it works | Trade-off |
|---|---|---|
| **Method 1: Avoid cycles only** | Don't create paths that loop back on themselves (check the current path only) | **Suboptimal but practical** — cheap to check (just look back along your current path), but doesn't stop you from re-exploring a state reached via a *different, non-cyclic* path elsewhere in the tree. |
| **Method 2: Full duplicate detection** | Keep a full "closed list" / "visited set" of every state ever generated; never re-generate a previously-seen state | **Optimal but memory-inefficient** — for the 8-puzzle, this means tracking up to `9! = 362,880` possible states, which costs real memory, but guarantees you never waste time re-exploring identical states. |

**Why this matters concretely:** the 8-puzzle has 9!=362,880 total possible arrangements, but a search *tree* (which doesn't collapse repeated states) can be vastly larger than that, since the same board arrangement can be reached by many different move sequences.

> **Takeaway:** Always weigh "avoid only cycles" (cheap, but leaves duplicate work on the table) against "track everything visited" (more memory, but eliminates wasted re-exploration) — the right choice depends on whether memory or time is your bottleneck.

---

## 12. Chapter Summary

1. Problem formulation requires **abstracting away real-world detail** to build a feasible-to-search state space (this connects back to Chapter 3's abstraction concept).
2. There's a family of uninformed strategies (BFS, UCS, DFS, IDS, Bidirectional) — each makes a different trade-off among completeness, optimality, time, and space.
3. **Iterative deepening is the generally recommended default** among these — it achieves the completeness and optimality of BFS while using only the linear memory of DFS, at only a modest time overhead.

---

Since your slides reference two external demo sites (rmit.edu.au and aima.cs.berkeley.edu) for visualizing these searches — want me to pull up a quick interactive visual here (e.g., an animated BFS vs. DFS vs. IDS trace on a small tree) so you can see the fringe evolve step by step, rather than just reading the static walkthroughs?