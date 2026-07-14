Excellent. This chapter teaches the **basic search algorithms** used in AI.

Think of them as **different ways of exploring a maze**.

The goal is always the same:

> **Find a path from the Start State to the Goal State.**

The only difference is **how they search**.

---

# Uninformed Search

First, what does **uninformed** mean?

It literally means

> **The algorithm has no extra information about where the goal is.**

Imagine you're in a huge maze.

Someone tells you

> "The treasure is somewhere."

But they don't tell you

* whether it's left or right
* whether it's near or far
* whether you're getting closer

You just blindly explore.

That's why uninformed search is also called

> **Blind Search**

---

# Example

Imagine this tree.

```
                A
             /     \
            B       C
          /  \     /  \
         D    E   F    G
        / \
       H   I
```

Suppose the goal is **G**.

Different search algorithms will explore this tree differently.

---

# How do we judge a search algorithm?

There are **4 important questions**.

---

# 1. Completeness

Question:

> Will the algorithm definitely find a solution if one exists?

Example

Maze

```
Start

 |

 |

Goal
```

If the algorithm is complete,

it **will eventually reach the goal.**

If not,

it may wander forever.

---

Example

Suppose

```
Start

 |

 |

Infinite tunnel
```

If the algorithm keeps walking forever,

it never reaches the goal.

Not complete.

---

# 2. Optimality

Question

> Does it find the BEST solution?

Notice:

Finding **a solution**

≠

Finding the **best solution**

---

Example

```
Road A
Distance = 10 km

Road B
Distance = 100 km
```

Both reach the destination.

Only Road A is optimal.

---

# 3. Time Complexity

Question

How much work does the algorithm do?

Usually measured by

How many nodes it visits.

Less nodes

↓

Less time.

---

# 4. Space Complexity

Question

How much memory does it use?

Some algorithms remember thousands of nodes.

Others remember only a few.

Less memory

↓

Better.

---

# Uninformed Search Algorithms

There are four main ones.

```
1. Breadth-First Search (BFS)

2. Uniform Cost Search (UCS)

3. Depth-First Search (DFS)

4. Iterative Deepening Search (IDS)
```

We'll learn each carefully.

---

# Breadth-First Search (BFS)

This is the easiest one.

## Main idea

Explore

> **All nearby nodes first.**

Think of throwing a stone into water.

```
        *
      * * *
    * * * * *
```

The waves spread outward evenly.

BFS works exactly like that.

---

Example

```
                A
             /     \
            B       C
          /  \     /  \
         D    E   F    G
```

Goal = G

---

We start at

```
A
```

Queue

```
[A]
```

---

Remove A.

Visit it.

Add children.

Queue

```
[B C]
```

---

Remove B.

Add children.

Queue

```
[C D E]
```

---

Remove C.

Add children.

Queue

```
[D E F G]
```

---

Remove D

Queue

```
[E F G]
```

---

Remove E

Queue

```
[F G]
```

---

Remove F

Queue

```
[G]
```

---

Remove G

Goal found.

---

Visited order

```
A

B C

D E F G
```

Notice something?

It finishes one **entire level**

before going deeper.

---

# Why is it called Breadth First?

Because it searches

**across**

before

**down.**

---

# BFS uses a Queue

Queue means

> First In First Out (FIFO)

Like standing in a bank.

```
Ali arrives

↓

John arrives

↓

Sara arrives
```

Who leaves first?

Ali.

First in

↓

First out.

---

Queue

```
Front

A B C D

Back
```

Remove from front.

Add to back.

---

# BFS Properties

---

## Complete?

Yes.

As long as there are not infinitely many children,

BFS eventually reaches the goal.

---

## Optimal?

Yes

ONLY IF

Every step costs the same.

Example

Every road

```
1 km
```

Then

Shortest number of steps

=

Shortest path.

---

But suppose

```
Road 1

1 km

Road 2

100 km
```

Then BFS can fail.

We'll see why soon.

---

## Time Complexity

Notation

```
b
```

means

Branching factor.

---

What is Branching Factor?

Suppose every node has

3 children.

```
A

↓

3 children

↓

Each has 3 children

↓

Each has 3 children
```

Then

```
b = 3
```

---

Suppose goal depth

```
d = 4
```

BFS explores roughly

```
1

+

3

+

9

+

27

+

81
```

This becomes

```
O(b^(d+1))
```

Don't worry about the formula.

Just remember

> BFS may visit MANY nodes.

---

## Space Complexity

This is BFS's biggest weakness.

It stores almost everything.

Imagine

```
100000 nodes
```

All sitting in memory.

Very expensive.

---

# Uniform Cost Search (UCS)

Why do we need UCS?

Because BFS assumes

every move costs the same.

Real life isn't like that.

---

Example

```
A

|

B (cost 100)

|

Goal
```

Another path

```
A

|

C (cost 1)

|

D (cost 1)

|

Goal (cost 1)
```

BFS chooses

```
A→B→Goal

2 moves
```

Cost

```
100
```

But another path costs

```
3
```

Clearly better.

---

So instead of asking

> Which node is shallowest?

UCS asks

> Which path is cheapest so far?

---

It always expands

the node with

the **smallest total path cost**.

This cost is written as

```
g(n)
```

Meaning

> Cost from Start to node n.

---

Example

Queue

```
A

Cost = 0
```

Expand.

```
B cost 10

C cost 3
```

Which comes next?

Not B.

C.

Because

```
3 < 10
```

---

# UCS uses a Priority Queue

Not FIFO.

Not LIFO.

Instead

Smallest cost comes first.

---

Example

```
Node

Cost

A

2

B

10

C

5
```

Queue becomes

```
A

C

B
```

Sorted by cost.

---

# UCS Properties

Complete?

Yes.

Optimal?

Always yes

as long as costs are positive.

---

# Depth-First Search (DFS)

This behaves completely differently.

Instead of exploring nearby nodes,

it goes

> **As deep as possible first.**

---

Example

```
                A
             /     \
            B       C
          /  \     /  \
         D    E   F    G
        / \
       H   I
```

Start

```
A
```

Go to

```
B
```

Then

```
D
```

Then

```
H
```

Cannot go deeper.

Come back.

Visit

```
I
```

Back again.

Visit

```
E
```

Back.

Then

```
C
```

etc.

---

Visited order

```
A

B

D

H

I

E

C

F

G
```

Very different from BFS.

---

# DFS uses a Stack

Stack = Last In First Out (LIFO)

Think of plates.

```
Plate A

Plate B

Plate C
```

Which leaves first?

Plate C.

Last in

↓

First out.

---

# DFS Properties

---

## Complete?

No.

Imagine

```
A

|

B

|

C

|

D

|

E

|

...
```

Infinite depth.

DFS keeps going forever.

Never returns.

---

## Optimal?

No.

Suppose

```
Goal

↓

Near
```

But DFS first goes

```
1000 levels down
```

It might find a terrible solution first.

---

## Space

This is DFS's biggest advantage.

It remembers only

one path.

Example

```
A

↓

B

↓

D

↓

H
```

Only these nodes are stored.

Very small memory.

---

# BFS vs DFS

Imagine looking for your friend inside a building.

BFS

```
Check every room

on Floor 1

↓

Then Floor 2

↓

Then Floor 3
```

DFS

```
Go to first room

↓

Go upstairs

↓

Keep going

↓

Reach roof

↓

Come all the way back
```

---

# Iterative Deepening Search (IDS)

Suppose DFS gets stuck forever.

How can we fix it?

Simple.

Don't allow it to go infinitely deep.

---

Suppose

Maximum depth

```
L = 2
```

DFS cannot go deeper than level 2.

---

If goal not found

Increase limit.

```
L = 3
```

Still not found.

Increase.

```
L = 4
```

Continue.

---

Example

Goal at depth 3.

Searches become

```
Depth 0

A
```

Not found.

---

```
Depth 1

A

B

C
```

Not found.

---

```
Depth 2

A

B

D

E

C

F

G
```

Not found.

---

```
Depth 3

Goal found.
```

---

Notice something?

It repeats earlier nodes.

At first this seems wasteful.

But surprisingly,

most nodes are near the bottom of the tree, so repeating the upper levels doesn't add much extra work.

---

# Why IDS is so popular

It combines the best parts of BFS and DFS.

From BFS

✔ Complete

✔ Finds shortest solution (when each step costs the same)

From DFS

✔ Uses very little memory

That's why IDS is often preferred when memory is limited.

---

# Bidirectional Search

Suppose you want to travel

```
A → G
```

Normal search

```
A

↓

...

↓

G
```

Instead,

search from **both ends**.

```
A →

← G
```

Eventually they meet.

```
A

↓

C

↓

D

↑

F

↑

G
```

Meeting point

```
D
```

This can dramatically reduce the number of explored nodes because each search only has to go about half the distance.

The challenge is that searching backward is not always easy. For example, in chess, given a checkmate position, it's hard to list all possible previous positions.

---

# Repeated States

This is an important practical issue.

Imagine this graph:

```
A

↓

B

↓

C

↑

|
```

You can go in circles forever.

```
A

↓

B

↓

C

↓

B

↓

C

↓

B
```

Without checking repeated states,

the search wastes huge amounts of time or may never finish.

---

## Solution 1: Avoid cycles in the current path

Before moving to a new state, check:

> "Is this state already on my current path?"

If yes, don't go there.

This prevents loops like:

```
A → B → C → B → C → ...
```

---

## Solution 2: Never generate the same state twice

Keep a **visited set**.

Whenever you generate a state:

* If it's new, explore it.
* If you've already seen it, skip it.

Example:

```
Visited

A

B

C
```

If another path reaches **B**, ignore it.

This is more effective, but it uses more memory because you must remember every visited state.

---

# Final Comparison

| Algorithm | How it searches                  | Complete?            | Optimal?                 | Memory      |
| --------- | -------------------------------- | -------------------- | ------------------------ | ----------- |
| **BFS**   | Level by level                   | ✅ Yes                | ✅ Yes (equal step costs) | ❌ Very high |
| **UCS**   | Lowest path cost first           | ✅ Yes                | ✅ Yes (positive costs)   | ❌ High      |
| **DFS**   | Go as deep as possible first     | ❌ No (can get stuck) | ❌ No                     | ✅ Very low  |
| **IDS**   | DFS with increasing depth limits | ✅ Yes                | ✅ Yes (equal step costs) | ✅ Low       |

## The Big Picture

All four algorithms are trying to solve **the same search problem**. The only difference is **which node they choose to expand next**:

* **BFS:** "Expand the **shallowest** node first."
* **UCS:** "Expand the **cheapest** path so far."
* **DFS:** "Expand the **deepest** node first."
* **IDS:** "Run DFS repeatedly with gradually increasing depth limits."

This single choice—**which node to expand next**—is what makes their behavior, speed, memory usage, and solution quality different.
