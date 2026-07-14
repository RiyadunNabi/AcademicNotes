Perfect. This chapter is the **foundation of AI search algorithms**. If you understand this chapter, later algorithms like **BFS, DFS, UCS, Greedy Search, and A*** become much easier.

I'll explain **everything from zero**, assuming you've never studied AI before.

---

# Chapter 3: Problem Solving by Searching

## First Question...

### What is "Problem Solving" in AI?

Imagine you have a robot.

You tell it:

> "Go from your bedroom to the kitchen."

The robot has never gone there before.

So what should it do?

It has to **think**.

It has to ask itself:

* Where am I now?
* Where do I want to go?
* What paths are available?
* Which path should I choose?

This process is called

> **Problem Solving**

And the most common way AI solves problems is

> **Searching**

---

# Real-Life Example

Imagine you're in BUET.

You want to go to

**Bashundhara City.**

You don't magically teleport there.

Instead you think

```
Current Location
        ↓
BUET

Can go

→ Shahbag
→ Nilkhet
→ New Market

From Shahbag

→ Banglamotor

From Banglamotor

→ Kawran Bazar

...

Eventually

→ Bashundhara City
```

You're mentally searching through possible roads.

That is exactly what AI does.

---

# Problem-Solving Agent

What is an **agent**?

An agent is simply

> Something that observes the environment and takes actions.

Examples

* Robot
* Self-driving car
* Chess AI
* Google Maps
* Vacuum cleaner robot

A problem-solving agent does one extra thing:

Instead of acting randomly,

it **searches** for a solution first.

---

# State Space

This is the most important concept.

## What is a State?

A **state** means

> A complete description of the current situation.

Example

Suppose you're playing Chess.

One arrangement of all pieces

```
♔ ♕ ♖ ...
```

is one state.

Move one pawn.

Now the board changes.

That's another state.

Every different board arrangement

=

One state.

---

Another example

You're traveling.

```
Arad
```

This is one state.

Drive to

```
Sibiu
```

Now this is another state.

Drive to

```
Fagaras
```

Another state.

So

```
State = Current city
```

---

## What is State Space?

State space means

> All possible states.

Imagine all cities in Romania.

```
Arad

Sibiu

Timisoara

Zerind

Fagaras

Bucharest

...
```

All these cities together form the

**State Space**

Think of it like

```
Entire World

contains

every possible situation.
```

---

# Goal State

A goal state is simply

> The state you want to reach.

Example

Current

```
Arad
```

Goal

```
Bucharest
```

Done.

---

Another example

Current

```
Your room
```

Goal

```
Kitchen
```

---

Another example

Current

```
Chess game
```

Goal

```
Checkmate opponent
```

---

Some problems have

**one goal**

Example

Reach

```
Mammoth
```

Only Mammoth works.

---

Some problems have

**many goals**

Example

Go to

"Any city with a ski resort."

Maybe

```
City A

City B

City C
```

All satisfy the goal.

---

# Operators (Actions)

Operator means

> An action that changes one state into another.

Example

Current

```
Arad
```

Operator

```
Drive to Sibiu
```

New State

```
Sibiu
```

---

Another example

Current

```
Blank tile here
```

Operator

```
Move blank left
```

New puzzle arrangement.

---

Another example

Current

```
Robot standing
```

Operator

```
Walk forward
```

New state.

---

So

```
State
   |
Action
   |
New State
```

---

# Initial Simplifying Assumptions

Real life is complicated.

So AI books simplify the world first.

---

## 1. Static Environment

Meaning

Nothing changes while we're thinking.

Example

Google Maps is calculating a route.

Nobody suddenly builds a new road.

Nothing changes.

Static.

---

If roads suddenly disappear,

that's dynamic.

---

## 2. Observable Environment

Means

The agent can see everything it needs.

Example

Chess.

You see every piece.

Nothing hidden.

---

Poker

Not observable.

Opponent's cards are hidden.

---

## 3. Discrete Environment

Discrete means

Things happen in separate steps.

Example

Chess

Move pawn.

Stop.

Move bishop.

Stop.

Each move is separate.

---

Continuous example

Driving.

You can turn steering wheel

```
0.1°

0.2°

0.25°

0.251°

```

Infinite possibilities.

That's continuous.

---

## 4. Deterministic

Means

Same action

→ Always same result.

Example

Chess

Move rook.

It always goes where expected.

---

Not deterministic

Throwing dice.

Roll 6?

Unknown.

---

# Romania Example

Suppose you're on vacation.

You're currently at

```
Arad
```

Tomorrow your flight leaves from

```
Bucharest
```

Question

How do you reach Bucharest?

AI first defines the problem.

---

## States

Cities

```
Arad

Sibiu

Fagaras

...

Bucharest
```

---

## Operators

Driving between connected cities.

Example

```
Arad → Sibiu

Arad → Zerind

Sibiu → Fagaras
```

---

## Goal

```
Reach Bucharest
```

---

Then AI searches

```
Arad

↓

Sibiu

↓

Fagaras

↓

Bucharest
```

Finished.

---

# State-Space Problem Formulation

Every search problem has **4 parts**.

---

## 1. Initial State

Where do we start?

Example

```
Arad
```

---

## 2. Successor Function (Actions)

Big scary name.

Very simple idea.

It answers:

> "If I'm here, where can I go next?"

Example

Current

```
Arad
```

Possible moves

```
Arad

↓

Sibiu

↓

Timisoara

↓

Zerind
```

So

```
S(Arad)

=

{

Sibiu,

Timisoara,

Zerind

}
```

This is called the **successor function**.

It returns all possible next states.

---

## 3. Goal Test

Question

"Am I done?"

Example

Current city

```
Bucharest
```

Goal?

Yes.

Stop.

---

Current city

```
Arad
```

Goal?

No.

Continue searching.

---

## 4. Path Cost

Sometimes multiple paths exist.

Example

Path A

```
100 km
```

Path B

```
500 km
```

Obviously choose

100 km.

The total travel distance is called

**Path Cost**.

Path cost can also mean

* Time
* Fuel
* Money
* Number of moves

depending on the problem.

---

# Abstraction

Very important.

Real life contains millions of details.

Example

Driving.

Should AI care about

* Road color?
* Trees?
* Lamp posts?
* Weather?
* Every pebble?

No.

It ignores unnecessary details.

It keeps only

```
City

Road

Distance
```

This simplification is called

> **Abstraction**

---

Example

Instead of modeling

```
Every traffic light

Every lane

Every building

Every person
```

AI simply models

```
City A

↓

City B

↓

City C
```

Much easier.

---

# State-Space Graph

Everything we've discussed can be drawn as a graph.

Example

```
Arad

 /     \

Sibiu  Zerind

 |

Fagaras

 |

Bucharest
```

Here

Each circle

=

State

Each line

=

Operator (action)

Finding a solution means finding a path from the start node to the goal node.

---

# Traveling Salesperson Problem (TSP)

One of the most famous AI problems.

Imagine a delivery person.

Cities:

```
A

B

C

D

E
```

Rules

* Visit every city exactly once.
* Return to the starting city.
* Minimize total distance.

Example

```
A

↓

C

↓

E

↓

B

↓

D

↓

A
```

Among all possible tours,

find the shortest one.

---

# 8-Queens Problem

You have an 8×8 chessboard.

Place 8 queens so that **no queen attacks another**.

A queen attacks horizontally, vertically, and diagonally.

The search problem is:

* **State:** current placement of queens.
* **Initial state:** empty board.
* **Action:** place another queen in a valid position.
* **Goal:** 8 queens placed with no attacks.
* **Path cost:** usually 1 per placement.

---

# Robot Assembly

Imagine a robot assembling a chair.

* **State:** positions of the robot and all chair parts.
* **Initial state:** parts are separate.
* **Action:** move robot arm, pick up a part, screw parts together.
* **Goal:** chair fully assembled.
* **Path cost:** time taken or number of actions.

---

# Learning a Spam Email Classifier

Even machine learning can be viewed as a search problem.

* **State:** current values of the model's parameters (weights).
* **Initial state:** random parameter values.
* **Action:** adjust the parameters during training.
* **Goal:** achieve the best accuracy on the training data.
* **Path cost:** training time or number of updates.

Instead of searching cities, we're searching for the **best parameter values**.

---

# 8-Puzzle

The puzzle has 8 numbered tiles and one blank space.

Example goal:

```
1 2 3
4 5 6
7 8 _
```

* **State:** arrangement of all tiles.
* **Initial state:** the given starting arrangement.
* **Action:** move the blank up, down, left, or right.
* **Goal:** reach the target arrangement.
* **Path cost:** usually 1 per move.

---

# Water Jug Problem

You have:

* a 4-gallon jug,
* a 3-gallon jug,
* unlimited water.

Goal:

Get **exactly 2 gallons** in the 4-gallon jug.

A state is written as **(x, y)**:

* `x` = water in the 4-gallon jug.
* `y` = water in the 3-gallon jug.

Examples:

* `(0,0)` → both empty.
* `(4,0)` → 4-gallon full, 3-gallon empty.
* `(2,3)` → 2 gallons in the big jug, 3 gallons in the small jug.

Possible actions include:

* Fill a jug.
* Empty a jug.
* Pour from one jug into the other.

One valid solution is:

```
(0,0)
↓

Fill 3-gallon

(0,3)

↓

Pour into 4-gallon

(3,0)

↓

Fill 3-gallon

(3,3)

↓

Pour into 4-gallon until full

(4,2)

↓

Empty 4-gallon

(0,2)

↓

Pour 3-gallon into 4-gallon

(2,0)
```

Now the 4-gallon jug contains exactly **2 gallons**, so we've reached the goal.

---

# VLSI Layout Problem

VLSI stands for **Very Large Scale Integration**, where millions of electronic components are placed on a computer chip.

The AI must decide:

* where to place each component,
* how to connect them,
* avoid overlaps,
* use as little chip area as possible,
* keep the wires short.

This is another search problem because there are an enormous number of possible layouts, and the AI searches for a very good one.

---

# Summary (Most Important Concepts)

Every AI search problem can be described using **five key elements**:

| Concept                    | Meaning                                                                  |
| -------------------------- | ------------------------------------------------------------------------ |
| **State**                  | The current situation.                                                   |
| **State Space**            | All possible situations.                                                 |
| **Initial State**          | Where we start.                                                          |
| **Operator (Action)**      | A legal move from one state to another.                                  |
| **Goal State / Goal Test** | The desired final situation or the test that checks if we've reached it. |
| **Path Cost**              | The total cost of reaching the goal (distance, time, moves, etc.).       |

The overall idea of this chapter is simple:

> **AI solves many problems by representing the world as states and actions, then searching through those states to find a path from the initial state to a goal state.** Once you understand this model, algorithms like **Breadth-First Search (BFS)**, **Depth-First Search (DFS)**, **Uniform Cost Search (UCS)**, and **A*** are simply different strategies for performing that search.
