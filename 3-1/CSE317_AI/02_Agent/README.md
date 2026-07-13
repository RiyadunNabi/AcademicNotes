
---

## Chapter 2: Intelligent Agents \u2014 Deep Dive

---

## Concept 1: What is an Agent? *(Slides 2\u20134)*

### The Core Idea

Before building intelligent systems, we need a precise mental model of *what kind of thing* we're building. The answer is: an **agent**.

> **An agent is anything that perceives its environment through sensors and acts upon that environment through actuators.**

That's the formal definition, but let's make it concrete.

Think about **you**. Right now:
- Your **eyes and ears** are picking up information from the world around you \u2014 that's perception
- Your **hands are scrolling**, your **brain is processing** \u2014 that's acting on the environment

Now think about a **thermostat**:
- Its **temperature sensor** reads the room \u2014 that's perception
- Its **heating element** turns on or off \u2014 that's acting

Both you and a thermostat are agents by this definition. The difference is *how intelligently* they act.

---

### The Three Types of Agents

The slides give us three concrete examples:

**Human agent:**
- Sensors: eyes, ears, skin, nose, tongue
- Actuators: hands, legs, mouth, facial muscles

**Robot agent:**
- Sensors: cameras, infrared range finders, GPS, microphones
- Actuators: motors (to move wheels, arms, joints)

**Software agent** (e.g., a spam filter):
- Sensors: incoming email text
- Actuators: "move to spam" or "keep in inbox" action

The key insight: **agents come in all forms**. What unifies them is the perceive \u2192 decide \u2192 act loop.

---

### Percepts vs. Actions \u2014 The Technical Vocabulary

The slides introduce precise terminology:

- **Percept** = a single input the agent receives at one moment (like one frame from a camera)
- **Percept sequence** = the *history* of all percepts so far
- **Action** = what the agent does in response

This distinction matters: a *smart* agent doesn't just react to the current percept \u2014 it uses the *entire history* of what it's seen to make decisions. A chess engine doesn't just look at the board right now \u2014 it remembers every move made so far.

---

### Properties of an Intelligent Agent *(Slide 3)*

Not every agent is *intelligent*. An intelligent agent should be:

**1. Autonomous** \u2014 it makes its own decisions, not just following a script. If you have to tell it what to do every step, it's not autonomous.

**2. Reactive** \u2014 it responds to changes in the environment. If the environment changes, the agent adapts.

**3. Pro-active (goal-directed)** \u2014 it doesn't just react passively. It *pursues goals*, taking initiative to make things happen.

**4. Social** \u2014 it can interact with other agents (humans, other AIs) through the environment.

A simple thermostat fails on almost all of these. It's not goal-directed (it just follows a threshold), not social, and only weakly reactive. A truly intelligent agent combines all four.

---

### The Agent Function *(Slide 5)*

Here's the mathematical way to think about an agent:

> **f: P* \u2192 A**

Where:
- P* = the set of all possible percept sequences (history of everything the agent has ever seen)
- A = the set of all possible actions
- f = the **agent function** that maps percept history to an action

In plain English: *given everything you've ever seen, what do you do next?*

But this function lives in the abstract. In practice, you implement it as an **agent program** running on **hardware (architecture)**:

> **agent = architecture + program**

The architecture is the physical or computational substrate (a robot body, a server). The program is the logic that decides what to do. Together they produce the agent's behavior.

---

## Concept 2: The Vacuum Cleaner World *(Slide 6)*

This is the classic textbook example used to make agents concrete. Let's walk through it carefully.

**The world:**
- Two rooms: A and B
- Each room can be Clean or Dirty
- The vacuum can be in either room

**Percepts:** What the vacuum "sees" at each moment \u2014 its location and the room's state. Examples:
- [A, Dirty] \u2014 the vacuum is in room A and it's dirty
- [B, Clean] \u2014 the vacuum is in room B and it's clean

**Actions:** What the vacuum can do:
- **Left** \u2014 move to room A
- **Right** \u2014 move to room B
- **Suck** \u2014 clean the current room
- **NoOp** \u2014 do nothing

**An agent function** for the vacuum would look like:

| Percept | Action |
|---|---|
| [A, Dirty] | Suck |
| [A, Clean] | Right |
| [B, Dirty] | Suck |
| [B, Clean] | Left |

This is simple enough to write as a table. But real-world environments have *millions* of possible states \u2014 you can't enumerate them all. That's why we need smarter approaches.

---

## Concept 3: Rational Agents *(Slides 7\u20138)*

### What Makes an Agent Rational?

We've defined agents. Now: what makes one *rational*?

First, we need a **performance measure** \u2014 an objective criterion for how well the agent is doing.

Examples from the slides:
- Robot driver \u2192 performance measure could be: *reach the destination safely and quickly*
- Chess program \u2192 performance measure: *win the game*
- Spam classifier \u2192 performance measure: *correctly classify emails, minimize false positives*

> **A rational agent selects actions that are expected to maximize its performance measure, given its percept sequence and built-in knowledge.**

Notice the word **expected**. The world is uncertain \u2014 the agent can't always know what will happen. Rationality means making the *best bet*, not guaranteeing perfect outcomes.

---

### Rationality \u2260 Omniscience

This is a crucial distinction the slides make:

**Omniscience** = knowing everything, always making the perfect choice  
**Rationality** = making the *best possible choice given what you know*

No real agent is omniscient. A doctor doesn't know with 100% certainty what disease a patient has \u2014 but a rational doctor orders the right tests, considers the evidence, and prescribes the most likely-to-help treatment. That's rationality under uncertainty.

---

### Exploration and Learning

A rational agent also actively gathers information. If you don't know something relevant to your goal, you should *find out*.

Example: a robot navigating an unknown building doesn't just sit still \u2014 it explores, maps the environment, and updates its knowledge. This is **information gathering behavior**, and it's a hallmark of intelligence.

The slides also distinguish **autonomy**:

> An agent is autonomous if its behavior is determined by its own percepts and experience, not solely by what its designer pre-programmed.

A fully autonomous agent can learn and adapt. A non-autonomous agent just blindly follows its initial programming regardless of what it experiences.

---

## Concept 4: PEAS Framework *(Slides 9, 12\u201313)*

Before you can build an intelligent agent, you need to carefully specify what kind of agent you're building. The **PEAS** framework gives you a structured way to do this.

**PEAS stands for:**
- **P**erformance measure \u2014 what does "doing well" mean?
- **E**nvironment \u2014 what world does the agent operate in?
- **A**ctuators \u2014 what can the agent *do*?
- **S**ensors \u2014 what can the agent *perceive*?

---

### PEAS Example 1: DARPA Robot Driver *(Slide 12)*

| PEAS | Details |
|---|---|
| **Performance** | Time to complete course; safety (no crashes) |
| **Environment** | Roads, other traffic, obstacles, weather |
| **Actuators** | Steering wheel, accelerator, brake, horn, signals |
| **Sensors** | Optical cameras, lasers, sonar, GPS, speedometer, odometer |

Notice how carefully you have to think about each component. What counts as "success"? What is the agent actually sensing? What can it actually *do*?

---

### PEAS Example 2: Medical Diagnosis System *(Slide 13)*

| PEAS | Details |
|---|---|
| **Performance** | Healthy patient, minimize cost, avoid lawsuits |
| **Environment** | Patient, hospital, staff |
| **Actuators** | Screen display (questions, diagnoses, treatment recommendations) |
| **Sensors** | Keyboard (doctor enters symptoms and patient answers) |

This one is interesting because the "actuators" are just a screen \u2014 the agent can only *display* information, it can't physically treat anyone. And its "sensors" are just keyboard input \u2014 it can only know what's typed in.

PEAS forces you to be precise about what the agent can and cannot do. Many AI projects fail because designers were vague about this upfront.

---

## Concept 5: Types of Environments *(Slides 14\u201321)*

This is one of the most important conceptual sections. The difficulty of building an intelligent agent depends enormously on what *kind of environment* it operates in. The slides define six key dimensions.

---

### Dimension 1: Fully Observable vs. Partially Observable

**Fully observable:** The agent's sensors give it access to the *complete state* of the environment at all times.

Example: Chess \u2014 you can see the entire board at all times.

**Partially observable:** The agent can only see *part* of the environment.

Example: Driving \u2014 you can't see around corners. You don't know what's behind that truck. You're making decisions with incomplete information.

*Most real-world environments are partially observable.* This is what makes them hard.

---

### Dimension 2: Deterministic vs. Stochastic

**Deterministic:** Given the current state and the agent's action, the *next state is perfectly predictable*.

Example: A simple math calculator \u2014 given input, output is always the same.

**Stochastic:** There's *randomness* \u2014 the same action in the same state can lead to different outcomes.

Example: Driving in the rain \u2014 even if you brake the same way, you might skid or not depending on a thousand micro-factors.

There's also a special case: **strategic** \u2014 the environment is deterministic *except* for the actions of *other agents* (like other players in a game). You can predict physics, but not other people's choices.

---

### Dimension 3: Episodic vs. Sequential

**Episodic:** Each decision is *independent*. What you did in the past doesn't affect what you should do now.

Example: A spam filter \u2014 classifying *this* email doesn't depend on what emails came before it.

**Sequential:** Current decisions *affect future decisions*. You need to think ahead.

Example: Chess \u2014 every move affects the entire rest of the game. Moving a pawn now could cost you the game in 20 moves.

Sequential environments require planning. Episodic ones don't.

---

### Dimension 4: Static vs. Dynamic

**Static:** The environment *doesn't change while the agent is thinking*. Take all the time you want.

Example: Crossword puzzle \u2014 the letters don't move while you're thinking.

**Dynamic:** The environment *keeps changing* even while the agent deliberates.

Example: Driving \u2014 other cars are moving while you're deciding whether to change lanes.

There's also **semidynamic:** the environment itself is static, but the *agent's performance score* changes with time. Example: Chess with a clock \u2014 the board doesn't move on its own, but your score worsens the longer you take.

---

### Dimension 5: Discrete vs. Continuous

**Discrete:** A finite, countable set of distinct states, percepts, and actions.

Example: Chess \u2014 there's a finite (though huge) number of board positions.

**Continuous:** States and actions exist on a continuous spectrum.

Example: Driving \u2014 the car's position is a real number (not just "left" or "right"), speed is continuous, steering angle is continuous.

Continuous environments are mathematically harder to reason about.

---

### Dimension 6: Single Agent vs. Multi-Agent

**Single agent:** One agent operating alone.

Example: A crossword solver \u2014 there's no opponent, just you and the puzzle.

**Multi-agent:** Multiple agents interacting, potentially with competing goals.

Example: Driving \u2014 every other car is also an agent with its own goals. Poker \u2014 other players are adversarial agents actively trying to defeat you.

Multi-agent environments require reasoning about *other agents' behavior* \u2014 much harder.

---

### The Environment Table *(Slides 16\u201321)*

The slides build up a comparison table. Let me give you the full version:

| Environment | Observable | Deterministic | Episodic | Static | Discrete | Single agent |
|---|---|---|---|---|---|---|
| **Solitaire** | No | Yes | Yes | Yes | Yes | Yes |
| **Driving** | No | No | No | No | No | No |
| **Internet shopping** | No | No | No | No | Yes | No |
| **Medical diagnosis** | No | No | No | No | No | Yes |
| **Chess (with clock)** | Yes | Strategic | Sequential | Semi | Discrete | Multi |
| **Crossword puzzle** | Yes | Deterministic | Sequential | Static | Discrete | Single |

The bottom line from the slides: **most real-world domains are in the hardest category** \u2014 partially observable, stochastic, sequential, dynamic, continuous, multi-agent. This is why AI is so difficult.

---

## Concept 6: Types of Agent Programs *(Slides 22\u201330)*

Now the slides get to the heart of the chapter: how do we *implement* an agent's decision-making? There are five types, from simplest to most powerful.

---

### Type 1: Table-Driven Agent *(Slides 23\u201325)*

**Idea:** Build a giant lookup table. For every possible percept sequence, pre-specify the correct action.

```
If you saw [A, Dirty] \u2192 Suck
If you saw [A, Clean] \u2192 Right
...
```

**Analogy:** It's like having a recipe book with every possible dish situation covered.

**Why it fails:**

The table is *astronomically large*. Chess alone has ~10��\u2070 possible states \u2014 that's more than atoms in the observable universe. You cannot build or store this table.

Also:
- It has no memory of *why* it's doing things \u2014 no understanding
- It can't adapt if the environment changes (you'd have to rewrite the entire table)
- It can't handle anything not explicitly in the table

This is essentially a **hardcoded database**, not real intelligence.

---

### Type 2: Simple Reflex Agent *(Slide 26)*

**Idea:** Instead of a table, use **condition-action rules**:

```
IF dirty THEN suck
IF in_room_A AND clean THEN move_right
IF in_room_B AND clean THEN move_left
```

**Analogy:** Like a set of reflexes. Touch hot stove \u2192 pull hand back. No thinking required.

**Improvement over table:** Rules generalize better. One rule covers many possible states.

**Why it still fails:**

- **Stateless** \u2014 it has no memory of the past. Each decision is made purely from the current percept.
- Can't handle situations where *history matters*.

Example: if you're navigating a building and you enter a room, a reflex agent can't remember "I already checked this room" \u2014 it might loop forever.

- Still can't adapt to environment changes (rules are hardcoded)

---

### Type 3: Model-Based Reflex Agent (Agents with Memory) *(Slide 23)*

**Idea:** Add **internal state** \u2014 a memory of what the world looked like in the past.

The agent maintains an **internal model** of the world and updates it with each new percept. Decisions are based on the current percept *plus* the internal model.

**Analogy:** Now you have a map you're drawing as you explore. You know "I've already cleaned room A" because you remember it.

**How it works:**
1. Receive current percept
2. Update internal model using: "what did I know before?" + "what action did I just take?" + "what am I perceiving now?"
3. Use rules to pick the best action based on updated model

**Improvement:** Can handle partially observable environments better, because the internal state compensates for what you can't currently see.

**Limitation:** Still rule-based. Doesn't think about *goals* or *future outcomes*. Just reacts according to rules, even if the rules are informed by memory.

---

### Type 4: Goal-Based Agent *(Slides 27\u201328)*

**Idea:** Give the agent explicit **goals** \u2014 descriptions of desirable situations \u2014 and let it *plan* how to achieve them.

**Analogy:** Instead of reflex rules ("if this, do that"), you have a destination ("I want to reach Chittagong") and you figure out the best path to get there.

**Key difference:** Goal-based agents are **deliberative**, not reactive. They reason about the future:

> "If I take action A, I'll be in state X. From state X, can I reach my goal? Or should I take action B instead?"

This is the first agent type that truly *thinks ahead*. It may need to consider long sequences of actions before finding one that achieves the goal.

**Limitation:** Goals are binary \u2014 achieved or not achieved. There's no concept of *how well* you achieved the goal, or the *cost* of achieving it.

Example: A robot told to "reach the destination" with a goal-based agent will find *some* path. But maybe it takes a terribly inefficient route. It doesn't care \u2014 the goal is reached.

---

### Type 5: Utility-Based Agent *(Slides 29\u201330)*

**Idea:** Replace binary goals with a **utility function** \u2014 a numerical measure of how "happy" or "good" each possible state is.

> **U: State \u2192 Real Number**

The agent chooses actions that maximize expected utility.

**Analogy:** Instead of just "reach Chittagong" as your goal, you have preferences:
- Reach Chittagong quickly \u2192 higher utility
- Reach Chittagong safely \u2192 higher utility
- Reach Chittagong cheaply \u2192 higher utility

A utility function lets you trade these off. Maybe arriving 2 hours faster isn't worth the safety risk. The agent reasons about all these factors simultaneously.

**Key improvement over goal-based:**
- Can distinguish between *multiple goals* and weigh them against each other
- Can handle situations where no perfect solution exists \u2014 find the *best available* option
- Handles uncertainty properly: chooses the action with the highest *expected* utility (probability-weighted)

**Example:**
- Path A reaches destination in 2 hours, 90% chance of success
- Path B reaches destination in 3 hours, 99% chance of success

A goal-based agent might pick either. A utility-based agent calculates expected utility for each and picks the better trade-off.

This is the foundation of modern AI decision-making \u2014 it's literally the mathematical framework behind AlphaGo, self-driving cars, recommendation algorithms, everything.

---

### Bonus: Learning Agent

The slides also mention **learning agents** \u2014 agents that don't just act on pre-programmed knowledge but *improve from experience*.

A learning agent has:
- A **learning element** that modifies the agent's behavior based on feedback
- A **performance element** that decides current actions
- A **critic** that evaluates how well the agent is doing
- A **problem generator** that suggests new experiences to try

This is essentially what machine learning is: agents that get better over time by interacting with their environment.

---

## Final Summary *(Slides 31\u201332)*

The chapter establishes a complete framework:

**What is an agent?** Something that perceives its environment and acts on it.

**What makes it rational?** It maximizes expected performance given its percept history.

**What is its environment like?** Characterized by 6 dimensions \u2014 observability, determinism, episodicity, dynamism, discreteness, number of agents. Real environments are usually hard on all 6.

**How do we build agent programs?** Five types \u2014 table-driven, simple reflex, model-based reflex, goal-based, utility-based \u2014 each more powerful (and complex) than the last.

**The core formula:**
> agent = architecture + program  
> rational agent = architecture + program that maximizes expected utility

---

What do you want to dig deeper into, Riyad \u2014 the environment types, the agent types, or the PEAS framework?