# 🏛️ Computer Programming 1: Arrays & Data Structures

---

## 📖 Prerequisite Lexicon: From First Principles

Before manipulating composite data structures, one must establish an uncompromising grasp of the foundational terminology:

- **Variable**: A named storage location in primary computer memory ($\text{RAM}$) configured to hold a single value of a specified data type at any given time.
- **Data Type**: A formal classification specifying the domain of values an identifier can assume and the valid operations performable upon it (e.g., `num`, `string`).
- **Contiguous Memory**: Memory blocks situated directly adjacent to one another in physical and logical address space without interstitial gaps or interruptions.
- **Accumulator**: A variable utilised to collect incrementing counts or cumulative totals across iterative program cycles.
- **Hard-Coded Value**: An immutable literal value directly embedded into source code rather than derived dynamically via computation or runtime user input.
- **Magic Number**: An unnamed numeric literal directly embedded into source statements whose semantic purpose and origin remain opaque; an egregious antipattern in software architecture.
- **Iteration**: The repeated execution of a deterministic block of programmatic logic through control-flow loops.

---

## ⚙️ Foundational Architecture of Arrays

### 1. Definition and Physical Memory Topology
- ⤷ **Array Definition**: A composite data structure comprising a finite, ordered series or collection of variables existing in computer memory, all of which ==share an identical identifier and an identical data type==.
- ⤷ **Element**: An individual storage variable within the array structure.
- ⤷ **Size (Dimension)**: The total quantity of individual elements an array is designated to accommodate.
- ⤷ **Physical Address Contiguity**:
  - ↳ Array elements are guaranteed to reside in strictly **contiguous memory**.
  - ↳ If an array contains $N$ elements and each element occupies $k$ bytes, the total memory footprint evaluates precisely to:
    $$\text{Total Memory (bytes)} = N \times k$$
  - ↳ Consequently, the memory allocation is always an exact integer multiple of the element count.

> [!tip] 💡 Intuition: The PO Box Grid Analogy
> Visualise an array as a unitary block of postal deposit boxes mounted on a post office wall. The wall represents a singular named structure (`mailBox`). Rather than forging distinct, separately named boxes across the town (`mailBoxAlice`, `mailBoxBob`, `mailBoxCharlie`), one unified structure holds all slots side by side. Each citizen opens their specific compartment simply by referencing a displacement number (`mailBox[0]`, `mailBox[1]`, `mailBox[2]`).

---

### 2. Subscripts, Indices, and Pointer Offsets
- ⤷ **Subscript (Index)**: An integer expression specifying the exact relative position of an element within the array.
  - ↳ **Mathematical Property**: Subscripts are universally a continuous sequence of integers starting at zero ($0, 1, 2, \dots, N-1$).
  - ↳ **Displacement Mechanism**: The subscript does not denote an arbitrary label; it quantifies the ==exact distance (offset) from the first element== (base address):
    $$\text{Address}(\text{Element}_i) = \text{Base Address} + (i \times \text{Size of Element})$$
  - ↳ Because the initial element is at offset distance zero from the base address, its subscript must be $0$.
- ⤷ **Syntactical Representation**: Subscripts are enclosed within brackets (`[]`) or parentheses (`()`), dictated by language syntax (e.g., `someVals[2]`).

```
Array Declaration: num someVals[3] = 25, 36, 47

Memory Offset:       +0              +1              +2
                +-----------+   +-----------+   +-----------+
Element Index:  |  [ 0 ]    |   |  [ 1 ]    |   |  [ 2 ]    |
Value Stored:   |    25     |   |    36     |   |    47     |
                +-----------+   +-----------+   +-----------+
```

---

## 📐 Algorithmic Optimisation: Replacing Nested Decisions

### 1. The Inefficiency of Cascading Conditionals
When aggregating categorical tallies (e.g., tracking employee dependent counts from $0$ to $5$), beginners routinely author deeply nested selection structures:

```
// Antipattern: Cumbersome Cascading Selection
if dep == 0 then
    count0 = count0 + 1
else
    if dep == 1 then
        count1 = count1 + 1
    else
        if dep == 2 then
            count2 = count2 + 1
        ...
```

- ❌ *Architectural Flaws*:
  - **Zero Scalability**: Introducing dependent category $6$ demands manual modification of declarations, conditional tests, and branching logic.
  - **Cognitive Debt**: Flowcharts develop unmanageable decision tails that degrade maintenance and auditing.

### 2. Direct-Index Substitution
- ⤷ An array of accumulators collapses $O(N)$ decision branches into a single $O(1)$ constant-time computational indexing statement:
  ```
  count[dep] = count[dep] + 1
  ```
- ⤷ **Operational Prerequisites for the Subscript Variable**:
  - Must be strictly **numeric with zero fractional component** (integer).
  - Must fall strictly within the inclusive boundary $[0, \text{Size} - 1]$.

```
+-------------------------------------------------------+
|  User Input (dep = 2)                                 |
+-------------------------------------------------------+
                           |
                           v
+-------------------------------------------------------+
|  Array Operation: count[2] = count[2] + 1             |
|  (Direct Memory Lookup — Bypasses 6 Nested Decisions) |
+-------------------------------------------------------+
```

---

## 🛡️ Constants in Array Implementation

Named constants enforce structural hygiene, eliminate "magic numbers," and prevent catastrophic out-of-bounds boundary mutations.

> [!check] 🔢 High-Yield Enumeration: Three Modalities of Constants in Arrays
> 1. **To Hold Array Dimension (Size)**:
>    - Declaration: `num SIZE = 6`
>    - Instantiation: `num VALID_ITEM[SIZE]`
>    - Purpose: Establishes a singular site of change. Altering `SIZE` automatically scales all iterative loops and bounds checks.
> 2. **As Array Element Values**:
>    - To store immutable reference tables that persist unmodified throughout execution:
>    - `string MONTH[12] = "January", "February", ..., "December"`
> 3. **As Named Array Subscripts**:
>    - To endow positional numbers with explicit domain semantics:
>    - `num INDIANA = 5`
>    - Operation: `output salesArray[INDIANA]` (replaces the opaque `salesArray[5]`).

---

## 🔍 Array Search Operations

### 1. Linear (Sequential) Search Mechanics
- ⤷ **Sequential Traversal**: Iterating over an array element by element from index $0$ upwards to locate a target value.
- ⤷ **Flag Variable**: A boolean or character indicator (`foundIt = "N"`) tracking whether the target condition has been fulfilled.

> [!tip] 💡 Intuition: The Guest List Clip-Board
> Linear search mimics a bouncer verifying a patron against a VIP guest manifest. You start with the first name on the clipboard. If a match occurs, you flip your mental checklist switch to "Admitted" (Flag = True) and proceed; if not, your eyes slide to the very next line down until you exhaust the list.

### 2. Search Efficiency & Loop Short-Circuiting
- ⤷ **Uncontrolled Traversal**: A simple loop testing each index until `sub == SIZE` will stubbornly execute iterations even after discovering the item at index $0$.
- ⤷ **Optimised Early Exit**: Introduce compound boolean logic within the loop header:
  $$\text{Condition: } (\text{sub} < \text{SIZE}) \land (\text{foundIt} == \text{"N"})$$
  - ↳ Once the target matches: `foundIt` flips to `"Y"`.
  - ↳ The compound condition instantly yields $\text{False}$, terminating iteration immediately and conserving CPU cycles.

```
+-------------------------------------------------------------+
| Start Traversal: sub = 0, foundIt = "N"                     |
+-------------------------------------------------------------+
                              |
                              v
                /-----------------------------\
               < sub < SIZE AND foundIt == "N" >
                \-----------------------------/
                              |
                     Yes      |      No
           +------------------+------------------+
           |                                     |
           v                                     v
+------------------------+            +-----------------------+
| Test: item ==          |            | Evaluate foundIt:     |
|       VALID_ITEM[sub]? |            | If "Y" -> Success     |
+------------------------+            | If "N" -> Error Count |
    | Yes          | No               +-----------------------+
    |              |
    v              |
+---------------+  |
| foundIt = "Y" |  |
| Extract Data  |  |
+---------------+  |
    |              |
    +-------+------+
            |
            v
+------------------------+
| sub = sub + 1          |
+------------------------+
            |
            +------> (Loop Back to Compound Header)
```

---

## 💍 Parallel Arrays

- ⤷ **Definition**: Two or more distinct arrays that maintain a strict 1-to-1 logical correspondence through shared indices.
- ⤷ **Operational Mechanism**: Elements positioned at the identical index across disparate arrays describe attributes of the **same logical entity**.

```
Index:               [ 0 ]       [ 1 ]       [ 2 ]       [ 3 ]       [ 4 ]       [ 5 ]
VALID_ITEM:       |  106   |   |  108   |   |  307   |   |  405   |   |  457   |   |  688   |
                     |           |           |           |           |           |
                     | (links)   | (links)   | (links)   | (links)   | (links)   | (links)
                     v           v           v           v           v           v
VALID_PRICE:      | $0.59  |   | $0.99  |   | $4.50  |   |$15.99  |   |$17.50  |   |$39.00  |
```

- ⤷ **Access Idiom**:
  1. Search the primary identification array: `VALID_ITEM[sub]`.
  2. Upon matching, fetch the payload from the corresponding index of the secondary attribute array: `price = VALID_PRICE[sub]`.

---

## 📡 Range-Match Operations

Direct single-value indexing fails when an input belongs to an infinite continuous range (e.g., volume discount tiers where quantities $\ge 26$ receive a $20\%$ discount).

### 1. Inefficient Brute-Force Allocation
- Declaring an array covering all conceivable discrete quantities (e.g., `DISCOUNT[76]`):
  - ❌ Consumes memory profligately with duplicate redundant coefficients.
  - ❌ Arbitrary upper threshold fails completely when a patron orders $77$ units.

### 2. Ordered Breakpoint Matching
- ⤷ Store solely the categorical limit boundaries within a compressed parallel structure:
  - `num QUAN_LIMIT[4] = 0, 9, 13, 26`
  - `num DISCOUNT[4]   = 0, 0.10, 0.15, 0.20`
- ⤷ **Top-Down Search Algorithm**:
  1. Initialize pointer to the high-water index: $x = \text{SIZE} - 1$.
  2. Decrement pointer while checking: `quantity < QUAN_LIMIT[x]`.
  3. Once `quantity >= QUAN_LIMIT[x]`, halt. The index $x$ maps directly to the correct bracket `DISCOUNT[x]`.

```
Ordered Limit:       [0] = 0        [1] = 9        [2] = 13       [3] = 26
Rate Assigned:       [0] = 0.00     [1] = 0.10     [2] = 0.15     [3] = 0.20
                        ^              ^              ^              ^
                        |              |              |              |
Input Quantity = 15:    |              |       (15 >= 13) ---> MATCH FOUND: x = 2
                        |              |                       Discount: 15%
```

---

## 🛡️ Array Bounds & Safety Enforcement

- ⤷ **Array Bounds**: The legal integer index interval defined for an array of size $N$, expressed mathematically as:
  $$[0, N - 1]$$
- ⤷ **Highest Usable Subscript**: Universally evaluated as $\text{Size} - 1$.
- ⤷ **Out-of-Bounds Error**: An exceptional runtime state triggered when an index variable falls outside the legal range ($x < 0$ or $x \ge N$).
  - ↳ **Type of Error**: ==Strictly a Logical Error (semantic defect)== during program compilation, which becomes catastrophic at runtime.

> [!warning] ⚠️ Exam Trap & Edge Case: The Off-By-One Memory Hazard
> Different language environments execute drastically disparate behaviours when bounds violations occur:
> 1. **Managed Runtime Safety (e.g., Java, C#)**: Halts execution immediately via a runtime exception (e.g., `ArrayIndexOutOfBoundsException`).
> 2. **Low-Level Unchecked Access (e.g., C, C++)**: Accesses whatever raw contiguous bytes reside past the array boundary, precipitating insidious data corruption, segmentation faults, or severe security vulnerabilities (buffer overflow exploits).

```
Legal Bounds for num array[5]:

Memory Slot:    |  array[0]  |  array[1]  |  array[2]  |  array[3]  |  array[4]  |    ???
Index:                0            1            2            3            4          5
Status:          [ VALID ]    [ VALID ]    [ VALID ]    [ VALID ]    [ VALID ]   [ OUT OF BOUNDS ]
```

---

## 🍞 Deterministic Processing: The `for` Loop

### 1. Loop Syntax & Invariants
- ⤷ The `for` loop serves as the optimal programmatic structure for array processing because it encapsulates three mechanical steps in a single atomic declaration:
  1. **Initialization**: Instantiates the loop control variable (e.g., `dep = 0`).
  2. **Evaluation**: Compares the variable against a terminal boundary (e.g., `to SIZE - 1`).
  3. **Alteration**: Deterministically increments the variable (e.g., `step 1`).

### 2. Production Pseudocode Implementation

```
// Dynamic upper-bound configuration
num SIZE = 5
num ARRAY_LIMIT = SIZE - 1
string DEPTS[SIZE] = "Accounting", "Personnel", "Technical", "Customer Service", "Marketing"

for dep = 0 to ARRAY_LIMIT step 1
    output DEPTS[dep]
endfor
```

> [!bug] 🚨 Lecture Material Discrepancy
> In the source lecture slides, Figure 6-26 is captioned:
> *"Figure 6-26 Pseudocode that uses a more efficient for loop to output month names"*
> Observe that the internal body of the code on that exact slide manipulates and displays `DEPTS[dep]` (department names: Accounting, Personnel, etc.), **not** month names. The slide title mislabels the business domain of the listing. Be vigilant should this figure prompt appear verbatim on an examination.

---

## 📊 Comprehensive Synthesis Matrix

| Dimension / Topic | Linear Match Search | Parallel Array Retrieval | Range-Match Search |
| :--- | :--- | :--- | :--- |
| **Data Requirements** | Single monolithic array | Two or more coordinated arrays | Two parallel arrays (bounds + rates) |
| **Index Coupling** | $1$-to-$1$ internal lookup | Shared index across arrays | Value falls within interval $[k_i, k_{i+1})$ |
| **Loop Condition** | `sub < SIZE AND foundIt == "N"` | `sub < SIZE AND foundIt == "N"` | Top-down scan: `quantity < QUAN_LIMIT[x]` |
| **Boundary Anchor** | Starts at $0$ | Starts at $0$ | Starts at $\text{SIZE} - 1$ |
| **Typical Use-Case** | Item existence validation | Item price/metadata cross-referencing | Graduated tax/discount slabs |