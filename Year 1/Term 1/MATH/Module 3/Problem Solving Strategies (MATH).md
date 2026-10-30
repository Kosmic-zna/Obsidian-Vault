# 🏛️ Module 01: Foundations & The Pólya Heuristic Framework

## 📖 Prerequisite Lexicon: From First Principles

- **Heuristic**: An experience-based approach, mental shortcut, or pragmatic technique designed to facilitate problem solving, exploratory learning, and discovery when an algorithmic formula is not immediately evident.
- **Algorithm**: A rigid, finite, step-by-step mathematical or computational procedure that unfailingly yields a correct result for any valid input (e.g., long division).
- **Conjecture (Supposition)**: An assertion or proposition that is suspected to be true based on preliminary empirical evidence or intuition, but has not yet been formally verified or disproven.
- **Displacement**: The net geometric change in position of an entity relative to its initial point of origin, independent of the total distance travelled along the path.
- **Arithmetic Sequence**: An ordered sequence of numbers wherein the difference between any two consecutive terms remains strictly invariant (the constant common difference, $d$).
- **Summation ($\Sigma$)**: The total aggregate obtained by adding together a specified sequence of numbers or algebraic terms.

---

## ⚙️ Pólya’s Four Phases of Problem Solving

George Pólya (1887–1985), widely regarded as the **Father of Modern Problem Solving**, published his seminal work *How to Solve It* to systematise mathematical reasoning. Pólya defined mathematical problem solving as:
> *"Finding a way around a difficulty, around an obstacle, and finding a solution to a problem that is unknown."*

```
                     ┌─────────────────────────────────────────┐
                     │  Pólya's 4-Phase Heuristic Architecture │
                     └────────────────────┬────────────────────┘
                                          │
                                          ▼
                     ┌─────────────────────────────────────────┐
                     │     1. Understand the Problem           │
                     │  • Visualise • Organise • Connect       │
                     └────────────────────┬────────────────────┘
                                          │
                                          ▼
                     ┌─────────────────────────────────────────┐
                     │     2. Devise a Plan                    │
                     │  • Represent • Guess • Act • Transform  │
                     └────────────────────┬────────────────────┘
                                          │
                                          ▼
                     ┌─────────────────────────────────────────┐
                     │     3. Carry Out the Plan               │
                     │  • Execute knowledge • Apply logic      │
                     └────────────────────┬────────────────────┘
                                          │
                                          ▼
                     ┌─────────────────────────────────────────┐
                     │     4. Look Back (Verification)         │
                     │  • Assess reasonableness • Generalise   │
                     └─────────────────────────────────────────┘
```

> [!bug] 🚨 Lecture Material Discrepancy: Citation Dates
> Slide 4 attributes Pólya's work to **1973** (`George Polya (1973) - father of problem solving`), whilst Slide 24 cites **1957** (`Polya, G. (1957). How to Solve it: A New Aspect of Mathematical Method`).
> - **Factual Correction**: Pólya's foundational treatise *How to Solve It* was first published by Princeton University Press in **1945**. The 1957 and 1973 prints are later Anchor Books/Princeton paperback reprints. Do not be confused if an assessment references either date.

---

### Detailed Operational Breakdown of the 4 Phases

#### Phase 1: Understand the Problem
- ⤷ **Core Directive**: One cannot solve what one does not comprehend. You must identify the unknowns, the given data, and the operational constraints.
  - ↳ **Look for Information Given**: Delineate raw quantities, initial conditions, and parameters.
  - ↳ **Visualise the Information**: Translate semantic verbiage into spatial sketches, line diagrams, or mental imagery.
  - ↳ **Organise the Information**: Tabulate values or group related variables.
  - ↳ **Connect the Information**: Identify existing relationships between the known constraints and the unknown target.

#### Phase 2: Devise a Plan
- ⤷ **Core Directive**: Formulate a viable heuristic trajectory bridging the knowns to the unknowns.
  - ↳ **Category A: Make a Representation**
    - Draw a diagram (spatial, directional, or temporal).
    - Make a systematic (orderly) list or combinatorial table.
    - Formulate equations and assign algebraic variables.
  - ↳ **Category B: Make a Calculated Guess**
    - Guess and check (trial, error, and iterative refinement).
    - Look for an underlying numerical or structural pattern.
    - Make suppositions (counterfactual hypotheses).
  - ↳ **Category C: Go Through the Process**
    - Act it out (physical simulation).
    - Work backwards (reverse-engineer starting from the end state).
  - ↳ **Category D: Change the Problem**
    - Restate the problem in simpler, clearer language.
    - Simplify the problem (examine degenerate or small-number cases).
    - Solve a sub-problem (decompose complex tasks into modular steps).

#### Phase 3: Carry Out the Plan
- ⤷ **Core Directive**: Execute the heuristic strategy with precision and persistent logic.
  - ↳ **Use Mathematical Knowledge**: Deploy pertinent formulas, theorems, and arithmetic operations.
  - ↳ **Use Mathematical Skills**: Maintain algebraic hygiene, accurate arithmetic, and correct notation.
  - ↳ **Use Logical Thinking**: Check every intermediate deductive step. If a chosen plan fails, do not panic—discard it and pivot to an alternative plan.

#### Phase 4: Look Back
- ⤷ **Core Directive**: Retrospective analysis to solidify cognitive schemas and verify veracity.
  - ↳ **Check the Solution**: Is the magnitude plausible? Does it violate any physical or boundary constraints?
  - ↳ **Improve the Method**: Can the argument be streamlined or made more elegant?
  - ↳ **Seek Alternative Solutions**: Can the result be proven via a different heuristic (e.g., algebra vs. geometric diagram)?
  - ↳ **Extend the Method**: Can this logic be generalised to solve a broader family of problems?

---

# ⚙️ Module 02: Concrete Problem-Solving Heuristics & Worked Paradigms

```
                      ┌───────────────────────────────────────┐
                      │    The Core Heuristics Taxonomy       │
                      └──────────────────┬────────────────────┘
         ┌───────────────────┬───────────┴───────────┬───────────────────┐
         ▼                   ▼                       ▼                   ▼
   ┌───────────┐       ┌───────────┐           ┌───────────┐       ┌───────────┐
   │ Guess &   │       │ Systematic│           │ Visual    │       │ Pattern   │
   │   Check   │       │   Lists   │           │ Diagrams  │       │Recognition│
   └───────────┘       └───────────┘           └───────────┘       └───────────┘
         │                   │                       │                   │
         ▼                   ▼                       ▼                   ▼
   ┌───────────┐       ┌───────────┐           ┌───────────┐       ┌───────────┐
   │ Algebraic │       │Eliminating│           │ Working   │       │ Structural│
   │ Variables │       │Options    │           │ Backwards │       │  Analogy  │
   └───────────┘       └───────────┘           └───────────┘       └───────────┘
```

---

## 1. 📐 Strategy: Guess and Check (Iterative Convergence)

* ⤷ **Definition / Core Rule**: An exploratory heuristic wherein an educated trial estimate is substituted into the system's boundary conditions, tested for consistency, and iteratively adjusted based on the observed directional error.

> [!tip] 💡 Intuition: Artillery Calibration
> Guess-and-check is not blind guessing. It resembles artillery targeting: you fire an initial shot, observe whether it falls short or long, adjust the elevation, and rapidly bracket the true coordinate.

> [!example] 🧪 Step-by-Step Derivation: The Magic Triangle Additive Puzzle
> **Problem**: A planar triangular graph has vertices (red circles) and edge-midpoints (blue circles). The value of each blue circle equals the sum of its two adjacent red vertex circles. 
> Given the secondary triangle with target edge sums:
> - Top-to-Bottom-Left edge = $16$
> - Top-to-Bottom-Right edge = $11$
> - Base edge = $15$
> 
> Find the values of the three red vertex circles: Top ($T$), Bottom-Left ($L$), and Bottom-Right ($R$).

```
                      ( T )
                     /     \
                  [16]     [11]
                   /         \
                ( L )---[15]---( R )
```

1. **System Formulation**:
   $$\begin{cases} T + L = 16 \\ T + R = 11 \\ L + R = 15 \end{cases}$$
2. **Algebraic Summation Verification**:
   Sum all three equations simultaneously:
   $$(T + L) + (T + R) + (L + R) = 16 + 11 + 15$$
   $$2(T + L + R) = 42 \implies T + L + R = 21$$
3. **Exact Red Vertex Deduction**:
   - Since $L + R = 15$:
     $$T = 21 - (L + R) = 21 - 15 = \mathbf{6}$$
   - Since $T + R = 11$:
     $$L = 21 - (T + R) = 21 - 11 = \mathbf{10}$$
   - Since $T + L = 16$:
     $$R = 21 - (T + L) = 21 - 16 = \mathbf{5}$$
4. **Verification**:
   - $T + L = 6 + 10 = 16$ ✅
   - $T + R = 6 + 5 = 11$ ✅
   - $L + R = 10 + 5 = 15$ ✅

---

## 2. 📋 Strategy: Make an Orderly (Systematic) List

* ⤷ **Definition / Core Rule**: Tabulating all conceivable permutations, combinations, or states without omissions or duplicate counts by enforcing a rigorous lexicographical or numerical hierarchy.

> [!example] 🧪 Step-by-Step Derivation: The 5-Person Handshake Problem
> **Problem**: Five individuals $\{A, B, C, D, E\}$ attend a meeting. If every person shakes hands with every other person exactly once, how many discrete handshakes occur?

1. **Combinatorial Lexicographical Listing**:
   - Handshakes initiated by $A$: $(A, B), (A, C), (A, D), (A, E) \longrightarrow \mathbf{4}$
   - Handshakes initiated by $B$ (excluding $A$ to avoid duplicates): $(B, C), (B, D), (B, E) \longrightarrow \mathbf{3}$
   - Handshakes initiated by $C$ (excluding $A, B$): $(C, D), (C, E) \longrightarrow \mathbf{2}$
   - Handshakes initiated by $D$ (excluding $A, B, C$): $(D, E) \longrightarrow \mathbf{1}$
   - Handshakes initiated by $E$: Fully saturated $\longrightarrow \mathbf{0}$
2. **Total Aggregate**:
   $$\text{Total} = 4 + 3 + 2 + 1 = \mathbf{10\text{ handshakes}}$$
3. **Generalised Formula**: For any cohort of $n$ individuals, the number of distinct pairwise interactions is given by the binomial coefficient:
   $${n \choose 2} = \frac{n(n - 1)}{2} \implies {5 \choose 2} = \frac{5 \times 4}{2} = \mathbf{10}$$

---

## 3. 🗺️ Strategy: Draw a Diagram

* ⤷ **Definition / Core Rule**: Converting linguistic, relational, or topological descriptions into spatial, vector, or timeline models to make hidden geometric or temporal constraints immediately visible.

### Paradigm A: Girl Scout Directional Vector Displacement
> **Problem**: A troop departs from origin $(0,0)$. They traverse:
> 1. $1\text{ mile East}$
> 2. $2\text{ miles South}$
> 3. $3\text{ miles West}$
> 4. $1\text{ mile North}$
> 5. $2\text{ miles East}$
> 
> Determine the final net distance from the starting position.

```
       Start (0,0)  ──1 mi E──► (1,0)
            ▲                     │
            │                  2 mi S
            │                     ▼
          (0,-1) ◄──2 mi E─── (-2,-1)
            ▲                     ▲
            │                  1 mi N
            │                     │
            └────────3 mi W───────┘
```

1. **Cartesian Vector Decomposition**:
   Let the origin be $(0, 0)$, where East/North denote positive axes ($+\hat{i}, +\hat{j}$) and West/South denote negative axes ($-\hat{i}, -\hat{j}$).
   $$\Delta x = +1 - 3 + 2 = \mathbf{0\text{ miles}}$$
   $$\Delta y = -2 + 1 = \mathbf{-1\text{ mile}}$$
2. **Final Coordinates**:
   $$\vec{r}_{\text{final}} = (0, -1)$$
3. **Euclidean Distance**:
   $$d = \sqrt{(\Delta x)^2 + (\Delta y)^2} = \sqrt{0^2 + (-1)^2} = \mathbf{1\text{ mile}}$$

---

### Paradigm B: Relative Temporal Offsets (Stock Car Race Finish)
> **Problem**: Five vehicles $\{F, P, C, B, D\}$ cross the finish line subject to:
> - (a) $F$ finished $7\text{ s}$ before $C$.
> - (b) $P$ finished $6\text{ s}$ after $B$.
> - (c) $D$ finished $8\text{ s}$ after $B$.
> - (d) $C$ finished $2\text{ s}$ before $P$.
> 
> Establish the exact sequential arrival order of the five vehicles.

1. **Relative Timeline Parameterisation**:
   Assign time $t = 0$ to an arbitrary reference car, say Buick ($B$):
   $$t_B = 0$$
2. **Sequential Substitution**:
   - From (b): $P$ finishes $6\text{ s}$ after $B \implies t_P = t_B + 6 = \mathbf{+6}$
   - From (c): $D$ finishes $8\text{ s}$ after $B \implies t_D = t_B + 8 = \mathbf{+8}$
   - From (d): $C$ finishes $2\text{ s}$ before $P \implies t_C = t_P - 2 = 6 - 2 = \mathbf{+4}$
   - From (a): $F$ finishes $7\text{ s}$ before $C \implies t_F = t_C - 7 = 4 - 7 = \mathbf{-3}$
3. **Temporal Ordering (Ascending Time = Earlier Finish)**:
   $$t_F (-3) < t_B (0) < t_C (+4) < t_P (+6) < t_D (+8)$$
4. **Final Arrival Hierarchy**:
   $$\mathbf{\text{Ford } \longrightarrow \text{ Buick } \longrightarrow \text{ Chevrolet } \longrightarrow \text{ Pontiac } \longrightarrow \text{ Dodge}}$$

---

## 4. 🔍 Strategy: Look for a Pattern

* ⤷ **Definition / Core Rule**: Analysing discrete starting cases to identify repetitive recursive structures, arithmetic steps, or algebraic sequences, and using that rule to extrapolate further states.

---

### Paradigm A: The Boundary-Condition Fallacy (The Classic Frog in the Well)
> **Problem**: A frog rests at the bottom of a well $12\text{ feet}$ deep. Each day it climbs $+3\text{ feet}$; each night it slips back $-2\text{ feet}$. How many days does it take to escape?

```
Day  1 Start:  0 ft ──[Climbs to 3 ft]──► Slips to  1 ft
Day  2 Start:  1 ft ──[Climbs to 4 ft]──► Slips to  2 ft
...
Day  9 Start:  8 ft ──[Climbs to 11 ft]─► Slips to  9 ft
Day 10 Start:  9 ft ──[Climbs to 12 ft]─► REACHES TOP (Escapes before slipping!)
```

> [!warning] ⚠️ Exam Trap & Edge Case: The Net Advance Trap
> A common mistake is to calculate the net daily progress as $3 - 2 = 1\text{ foot/day}$ and conclude that the frog requires $\frac{12}{1} = 12\text{ days}$. 
> **Why this fails**: The moment the frog touches the rim ($12\text{ ft}$) during daylight, **it escapes immediately**. It does not slip back that night.

1. **Effective Well Depth Prior to Final Leap**:
   $$\text{Threshold} = 12\text{ ft} - 3\text{ ft (final leap)} = 9\text{ ft}$$
2. **Days to Reach Threshold**:
   $$\text{Days} = \frac{9\text{ ft}}{1\text{ ft/day}} = 9\text{ days}$$
3. **Day of Final Escape**:
   On Day 10, beginning at height $9\text{ ft}$, the frog climbs $+3\text{ ft}$, reaching $9 + 3 = \mathbf{12\text{ ft}}$. 
   **Total time required**: $\mathbf{10\text{ days}}$.

---

### Paradigm B: Numerical Sequence Extrapolation

| Given Sequence | First Differences ($\Delta_1$) | Underlying Nature / Rule | Next 2 Terms |
| :--- | :--- | :--- | :--- |
| $1, 4, 7, 10, 13, 16, 19, \dots$ | $+3, +3, +3, +3, +3, +3$ | Linear Arithmetic: $a_n = 3n - 2$ | $\mathbf{22, 25}$ |
| $19, 20, 22, 25, 29, 34, 40, \dots$ | $+1, +2, +3, +4, +5, +6$ | Quadratic: $\Delta_1$ increases by $+1$ | $\mathbf{47, 55}$ ($40+7, 47+8$) |
| $1, 4, 9, 16, 25, 36, 49, \dots$ | $+3, +5, +7, +9, +11, +13$ | Perfect Squares: $a_n = n^2$ | $\mathbf{64, 81}$ ($8^2, 9^2$) |

---

### Paradigm C: Arithmetic Series Summation (Gauss’s Pairing Technique)
> **Problem**: Compute the exact summation of the first $n$ natural numbers:
> - Case 1: First 10 counting numbers ($n = 10$).
> - Case 2: First 100 counting numbers ($n = 100$).

```
        1   +   2   +   3   + ... +  98  +  99  + 100
      100   +  99   +  98   + ... +   3  +   2  +   1
     ──────────────────────────────────────────────────
      101   + 101   + 101   + ... + 101  + 101  + 101  (100 pairs)
```

1. **Derivation of the Closed-Form Expression**:
   Let $S_n = 1 + 2 + 3 + \dots + n$.
   Writing the series forward and backward:
   $$S_n = 1 + 2 + \dots + n$$
   $$S_n = n + (n - 1) + \dots + 1$$
   Adding column-wise produces $n$ identical terms of magnitude $(n + 1)$:
   $$2S_n = n(n + 1) \implies \mathbf{S_n = \frac{n(n + 1)}{2}}$$

2. **Evaluations**:
   - For $n = 10$:
     $$S_{10} = \frac{10(11)}{2} = 5 \times 11 = \mathbf{55}$$
   - For $n = 100$:
     $$S_{100} = \frac{100(101)}{2} = 50 \times 101 = \mathbf{5050}$$

---

## 5. 🔣 Strategy: Use a Variable (Algebraic Modelling)

* ⤷ **Definition / Core Rule**: Translating verbal constraints into one or more formal algebraic equations, then using deductive manipulation to isolate the unknown quantity.

> [!example] 🧪 Step-by-Step Derivation: The Father-Son Algebra Problem
> **Problem**: To encourage his son to study algebra, a father agrees to pay the son $\text{Php } 8$ for every problem solved correctly, and fines him $\text{Php } 5$ for every incorrect solution. After completing $26$ problems, neither party owes anything to the other (net balance = $\text{Php } 0$). How many problems did the son solve correctly?

1. **Define the Variables**:
   Let $x = \text{number of problems solved correctly}$.
   Since $26$ problems were attempted in total:
   $$26 - x = \text{number of problems solved incorrectly}$$
2. **Formulate the Financial Equation**:
   $$\text{Earnings} - \text{Penalties} = \text{Net Balance}$$
   $$8(x) - 5(26 - x) = 0$$
3. **Algebraic Isolation**:
   $$8x - 130 + 5x = 0$$
   $$13x = 130 \implies x = \frac{130}{13} = \mathbf{10}$$
4. **Verification**:
   - Correct questions: $10 \times \text{Php } 8 = +\text{Php } 80$
   - Incorrect questions: $(26 - 10) = 16 \times (-\text{Php } 5) = -\text{Php } 80$
   - Net: $+80 - 80 = 0$ ✅

---

## 6. 🚫 Strategy: Eliminate Possibilities (Deductive Elimination)

* ⤷ **Definition / Core Rule**: Using negative or exclusionary constraints to systematically prune invalid candidate arrangements until only one logically consistent state remains.

> [!example] 🧪 Step-by-Step Derivation: Alien Age Ordering Puzzle
> **Problem**: Five aliens $\{ \text{Elyra, Ejah, Wino, Angel, Wenet} \}$ inhabit a spacecraft. Order them from **youngest to oldest** using the following clues:
> - Clue (a): Ejah is younger than Angel ($Ejah < Angel$).
> - Clue (b): Ejah is not the youngest in the group ($Ejah \neq \text{Rank } 1$).
> - Clue (c): Only one alien is older than Wenet ($\text{Wenet} = \text{Rank } 4$).
> - Clue (d): Wenet is younger than Wino ($\text{Wenet} < \text{Wino}$).

1. **Establish the Ranking Slots**:
   Let the positions be indexed $1$ (Youngest) to $5$ (Oldest):
   $$\underline{\hspace{1cm}}_1 \quad \underline{\hspace{1cm}}_2 \quad \underline{\hspace{1cm}}_3 \quad \underline{\hspace{1cm}}_4 \quad \underline{\hspace{1cm}}_5$$
2. **Direct Placement via Clues (c) and (d)**:
   - Clue (c) dictates that exactly one individual is older than Wenet. Thus, **Wenet must occupy Rank 4**.
   - Clue (d) states Wenet is younger than Wino. Therefore, **Wino must occupy Rank 5**.
   $$\underline{\hspace{1cm}}_1 \quad \underline{\hspace{1cm}}_2 \quad \underline{\hspace{1cm}}_3 \quad \underline{\text{Wenet}}_4 \quad \underline{\text{Wino}}_5$$
3. **Elimination via Clues (a) and (b)**:
   - Remaining candidates for Ranks 1, 2, and 3: $\{ \text{Elyra, Ejah, Angel} \}$.
   - Clue (a) dictates $Ejah < Angel$.
   - Clue (b) dictates Ejah cannot be Rank 1. 
   - Since Ejah is older than Rank 1, but younger than Angel, **Ejah must occupy Rank 2**, and **Angel must occupy Rank 3**.
   - By elimination, the remaining unassigned candidate, **Elyra**, must occupy Rank 1.
4. **Final Ordered Array (Youngest to Oldest)**:
   $$\mathbf{Elyra < Ejah < Angel < Wenet < Wino}$$

---

## 7. 🔄 Strategy: Work Backwards (Inverse Operations)

* ⤷ **Definition / Core Rule**: When a problem specifies the terminal condition and a sequence of state transformations, begin at the final value and systematically apply the inverse mathematical operations in reverse chronological sequence.

```
Forward Flow:   Initial State ──[Transformation 1]──► State 2 ──[Transformation 2]──► Final State
                                                                                             │
Reverse Flow:   Initial State ◄──[Inverse Op 1]───── State 2 ◄──[Inverse Op 2]───────┘
```

---

### Paradigm A: The Apple Thief and the Three Watchmen
> **Problem**: A thief steals apples from an orchard. Escaping, he encounters three successive watchmen. To each watchman, he gives **half of the apples he currently possesses, plus two more**. He exits the orchard with exactly **$1$ apple**. How many apples were stolen originally?

1. **State Equation Formulation (Forward)**:
   Let $A_{\text{in}}$ be apples before meeting a watchman, and $A_{\text{out}}$ be apples remaining after:
   $$A_{\text{out}} = \frac{A_{\text{in}}}{2} - 2$$
2. **Inverse Transformation (Backward)**:
   Solve for $A_{\text{in}}$:
   $$A_{\text{out}} + 2 = \frac{A_{\text{in}}}{2} \implies \mathbf{A_{\text{in}} = 2(A_{\text{out}} + 2)}$$
3. **Iterative Step-by-Step Backtracking**:
   - **Exiting Watchman 3**: Terminal count $A_3 = 1$.
     $$\text{Before Watchman 3}: A_2 = 2(1 + 2) = 2(3) = \mathbf{6\text{ apples}}$$
   - **Exiting Watchman 2**: Count is $6$.
     $$\text{Before Watchman 2}: A_1 = 2(6 + 2) = 2(8) = \mathbf{16\text{ apples}}$$
   - **Exiting Watchman 1**: Count is $16$.
     $$\text{Before Watchman 1 (Original Total)}: A_0 = 2(16 + 2) = 2(18) = \mathbf{36\text{ apples}}$$

---

### Paradigm B: The Departing Party Guests
> **Problem**: At $7\text{ pm}$, all invited guests have arrived. Each hour, a fraction of the attendees departs:
> - By $8\text{ pm}$: $\frac{1}{3}$ of the guests depart.
> - By $9\text{ pm}$: $\frac{1}{3}$ of the remainder depart.
> - By $10\text{ pm}$: $\frac{1}{3}$ of the remainder depart.
> - By $11\text{ pm}$: $\frac{1}{3}$ of the remainder depart, leaving exactly **$16$ guests**.
> 
> How many guests were present at $7\text{ pm}$?

1. **Forward Fraction Analysis**:
   If $\frac{1}{3}$ of the guests depart, the fraction of guests remaining is:
   $$1 - \frac{1}{3} = \frac{2}{3}$$
   Thus, for each hour:
   $$G_{t+1} = \frac{2}{3} G_t$$
2. **Inverse Transformation Formulation**:
   $$G_t = G_{t+1} \times \frac{3}{2}$$
3. **Step-by-Step Backtracking**:
   - **At 11 pm**: $G_{11} = 16$
   - **At 10 pm**: $G_{10} = \frac{16 \times 3}{2} = 8 \times 3 = \mathbf{24\text{ guests}}$
   - **At 9 pm**: $G_9 = \frac{24 \times 3}{2} = 12 \times 3 = \mathbf{36\text{ guests}}$
   - **At 8 pm**: $G_8 = \frac{36 \times 3}{2} = 18 \times 3 = \mathbf{54\text{ guests}}$
   - **At 7 pm (Initial Attendance)**: $G_7 = \frac{54 \times 3}{2} = 27 \times 3 = \mathbf{81\text{ guests}}$

---

# 📊 Comparative Heuristics Matrix

| Heuristic Strategy | When to Apply | Cognitive Mechanism | Common Pitfalls to Avoid |
| :--- | :--- | :--- | :--- |
| **Guess & Check** | Closed systems with discrete integers and interdependent variables | Iterative trial-and-error that adjusts guesses based on the error direction | Unsystematic guessing; failing to record and learn from prior attempts |
| **Orderly List** | Counting problems, combinations, or discrete allocations | Lexicographical ordering ($A \to B \to C$) to guarantee complete coverage | Forgetting duplicates; omitting boundary pairs (e.g., $(A, B)$ vs. $(B, A)$) |
| **Draw a Diagram** | Spatial displacement, vector movement, and relative timelines | Mapping semantic descriptions into clear spatial or graphical relationships | Inaccurate axis signs (e.g., confusing West with positive displacement) |
| **Look for a Pattern** | Sequences, growing geometric shapes, and iterative processes | Identifying differences between terms ($\Delta_1, \Delta_2$) or using algebraic series formulas | Overlooking boundary thresholds (e.g., the daytime top-reach in well problems) |
| **Use a Variable** | Word problems with balances, financial exchanges, or unknown counts | Setting up an equality $\sum \text{Gains} = \sum \text{Losses}$ | Assigning too many variables instead of expressing everything in terms of $x$ |
| **Eliminate Possibilities** | Ranking problems and logical deduction puzzles with multiple conditions | Placing anchored items first, then using negatives to prune invalid configurations | Making assumptions not explicitly stated in the given clues |
| **Work Backwards** | Sequential operations where only the final outcome is known | Inverting both the mathematical operations and the chronological order | Inverting the sequence order but forgetting to invert the arithmetic operations |

---
