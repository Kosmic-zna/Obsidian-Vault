# 🏛️ Master Study Guide: Principles of Structured Pseudocode & Algorithmic Logic

---

## 📖 Module 1: Foundational Paradigm & Prerequisite Lexicon

### 🎓 Prerequisite Lexicon: From First Principles
- ⤷ **Algorithm**: A finite, unambiguous, step-by-step sequence of instructions designed to resolve a specific computational problem or execute a task.
- ⤷ **Pseudocode**: A high-level, human-readable artificial notation that emulates programming logic without adhering to the idiosyncratic syntactic constraints of specific target languages (e.g., C#, Java, Python).
- ⤷ **Control Structure**: A syntactic block that dictates the path of program execution based on sequential flow, Boolean decisions, or repeated evaluation.
- ⤷ **Identifier**: A symbolic name allocated to uniquely identify a memory location, variable, constant, or subprogram module.
- ⤷ **Sentinel Value**: A pre-designated terminal token (e.g., `QUIT`, `-1`, `eof`) used to signal the cessation of a data stream or iterative sequence.
- ⤷ **State**: The complete snapshot of all values held within the allocated memory variables of a program at any discrete point during execution.

### ⚙️ The Architectural Lifecycle of Structured Execution
- ⤷ **The Golden Rule of Structured Flow**: Every structured program or block possesses precisely ==one entry point and one exit point==. Uncontrolled branch jumps (such as `goto` statements) violate structural integrity and are strictly forbidden.
- ⤷ **The Canonical Von Neumann Sequence**:
  - ↳ **START**: Global initialisation and environment bootstrap.
  - ↳ **Declarations**: Explicit memory allocation and type assignment.
  - ↳ **Input**: Fetching extrinsic data into declared volatile memory.
  - ↳ **Processing**: Algorithmic arithmetic, state mutation, and selection logic.
  - ↳ **Output**: Rendering transformed states to external sinks (console, files).
  - ↳ **STOP**: Explicit resource deallocation and deterministic program termination.

```
+-------------------------------------------------------------+
|                          S T A R T                          |
+-------------------------------------------------------------+
                              |
                              v
+-------------------------------------------------------------+
|                D E C L A R A T I O N S                      |
|          (Memory Allocation & Type Assignment)              |
+-------------------------------------------------------------+
                              |
                              v
+-------------------------------------------------------------+
|                       I N P U T                             |
|          (Stream Acquisition: get / input)                  |
+-------------------------------------------------------------+
                              |
                              v
+-------------------------------------------------------------+
|                  P R O C E S S I N G                        |
|       [Sequence] ----> [Selection] ----> [Repetition]       |
+-------------------------------------------------------------+
                              |
                              v
+-------------------------------------------------------------+
|                      O U T P U T                            |
|          (Stream Projection: print / output)                |
+-------------------------------------------------------------+
                              |
                              v
+-------------------------------------------------------------+
|                           S T O P                           |
+-------------------------------------------------------------+
```

> [!tip] 💡 Intuition: The Culinary Recipe Blueprint
> Regard pseudocode as the master recipe drafted by an executive chef. It details the essential ingredients, the precise order of preparation, the thermal thresholds, and the resting intervals. It remains entirely unconcerned with whether the kitchen uses a gas hob, an induction plate, or a wood-fired hearth (analogous to Python, C#, or Java). The underlying culinary physics remains invariant.

---

## 🏛️ Module 2: Memory Architecture — Declarations, Data Types, & Scope

### ⚙️ Variable Allocation & Typing Paradigms
- ⤷ **Variable Declaration**: Reserves memory address space mapped to an identifier and enforces type safety constraints.
  - ↳ **CamelCase Convention**: Mandated for mutable variables (e.g., `hourlyWage`, `calculatedAnswer`, `customerBill`).
- ⤷ **Constant Declaration**: Maps an immutable literal to an uppercase identifier, preventing computational anomalies stemming from obscure hardcoded literals.
  - ↳ **UPPERCASE Convention**: Standard for immutable parameters (e.g., `RATE`, `PI`, `MAX_BUFFER`).

```markdown
Declarations
    num studentAge
    num grossSalary = 0.00
    string customerName = ""
    char terminalGrade = 'F'
    boolean isProcessed = false
    num TAX_RATE = 0.12
```

### 📐 Structural Comparison: Core Data Types
| Primitive Type | Domain / Admissible Values | Memory Storage Role | Canonical Example |
| :--- | :--- | :--- | :--- |
| `num` | Continuous real numbers: $\mathbb{R}$ (Integers & Floating-point) | Quantitative metrics, counts, currency, geometric bounds | `num price = 19.99` |
| `string` | Zero or more contiguous Unicode/ASCII characters | Textual entities, identification keys, descriptive prompts | `string userName = "Ada"` |
| `char` | Exactly one atomic character enclosed in single quotes | Modals, menu selections, flags | `char response = 'Y'` |
| `boolean` | Binary truth states: $\{\text{true}, \text{false}\}$ | Predicate flags, selection switches, loop guards | `boolean hasAccess = true` |

> [!warning] ⚠️ Exam Trap & Edge Case: The Magic Number Pitfall
> Writing an inline literal such as `overtimePay = hours * 1.5 * 25.00` is an egregious procedural error. If the statutory rate changes, an engineer must hunt down every isolated occurrence of `1.5`. Always encapsulate these within named constants:
> ```
> Declarations
>     num OVERTIME_FACTOR = 1.5
>     num STANDARD_RATE = 25.00
> ```

> [!warning] ⚠️ Exam Trap & Edge Case: Uninitialised State Hazard (Garbage Values)
> In strict logical compilation, declaring `num total` does **not** guarantee a value of $0$. Memory allocations hold non-deterministic remnant data ("garbage bits"). Operating on an uninitialised accumulator (`total = total + inputVal`) generates undefined behavior. ==Always explicitly initialise accumulators to 0 and string buffers to empty quotes `""`==.

---

## 📡 Module 3: Input/Output Streams & String Manipulation

### ⚙️ Stream Mechanics
- ⤷ **Input (`input` / `get`)**: A blocking operation halting program execution until an external device transmits valid data into target variable references.
  - ↳ Multiple target variables may be sequentially populated via comma-delimited lists.
- ⤷ **Output (`output` / `print`)**: Writes evaluated literals, expression results, or variable contents to standard output sinks.
  - ↳ Can chain multiple string literals and memory variables using the comma operator.

```markdown
output "Enter customer identifier and units consumed: "
input customerId, unitsConsumed
output "Account [", customerId, "] owes: £", (unitsConsumed * UNIT_PRICE)
```

> [!warning] ⚠️ Exam Trap & Edge Case: Input / Prompt Decoupling
> Never execute an `input` operation without an antecedent `output` prompt. An unprompted input instruction places the terminal into a silent waiting state, misleading the user into assuming the system has locked up or crashed.

---

## 📐 Module 4: Arithmetic Operations, Precedence, & Mathematical Edge Cases

### ⚙️ Operator Precedence Hierarchy
When evaluating complex algebraic assignments, operations resolve strictly according to mathematical precedence:
1. **Parentheses / Grouping**: `$()$` (Innermost to outermost)
2. **Multiplicative Group**: Multiplication `$*$`, Division `$/$`, Modulo `$\%$` (Left-to-right associativity)
3. **Additive Group**: Addition `$+$`, Subtraction `$-$` (Left-to-right associativity)
4. **Relational / Comparison**: `$=, <>, <, >, <=, >=$`
5. **Logical Conjunction**: `$\text{AND}$`
6. **Logical Disjunction**: `$\text{OR}$`
7. **Assignment**: `$= \text{ or } \leftarrow$` (Strictly evaluated right-to-left)

```markdown
// Algorithmic evaluation demonstration:
set finalScore = (baseScore + 10) * 2 - bonus / 4
```

> [!example] 🧪 Step-by-Step Derivation: Order of Operations
> Given variables: $A = 5$, $B = 3$, $C = 2$, $D = 4$:
> Evaluate: `result = A + B * C^2 / (D - 2)` *(assuming `^` denotes exponentiation)*:
> 1. Parentheses: $(D - 2) \rightarrow (4 - 2) = 2$
> 2. Exponentiation: $C^2 \rightarrow 2^2 = 4$
> 3. Sub-expression: $A + B * 4 / 2$
> 4. Multiplicative scan (left-to-right): $B * 4 \rightarrow 3 * 4 = 12$
> 5. Next multiplicative: $12 / 2 = 6$
> 6. Additive scan: $A + 6 \rightarrow 5 + 6 = 11$
> 7. Assignment: `result = 11`

> [!warning] ⚠️ Exam Trap & Edge Case: Division by Zero & Integer Truncation
> - ❌ **Division by Zero**: Any expression of the form $X / 0$ triggers an unrecoverable arithmetic exception or infinite state. You must construct a defensive guard prior to evaluation:
>   ```markdown
>   if divisor = 0 then
>       output "Error: Undefined quotient. Zero divisor detected."
>   else
>       quotient = dividend / divisor
>   endif
>   ```
> - ❌ **Integer Division vs. Floating Point**: In strict typed systems, dividing an integer by another integer (e.g., $5 / 2$) may truncate fractional components to return $2$, not $2.5$. If real-valued precision is required, declare the target variable and operands as floating-point `num`.

---

## ⚖️ Module 5: Selection Control Structures & Boolean Logic

### ⚙️ The Three Morphologies of Decision Blocks
1. **Single-Alternative (`if-then-endif`)**: Bypasses processing unless a condition holds true (the null branch executes zero mutations).
2. **Dual-Alternative (`if-then-else-endif`)**: Guarantees mutual exclusion; exactly one path is executed depending on the resolution of the predicate.
3. **Multiple-Alternative (`if-then-else if-else-endif`)**: Evaluates a chained ladder of mutually exclusive predicates, exiting the block immediately upon the first true condition.

```markdown
// Dual-Alternative Selection Pattern:
if employeeHours > STANDARD_WEEK then
    regularPay = STANDARD_WEEK * HOURLY_RATE
    overtimeHours = employeeHours - STANDARD_WEEK
    overtimePay = overtimeHours * (HOURLY_RATE * OVERTIME_MULTIPLIER)
    totalPay = regularPay + overtimePay
else
    totalPay = employeeHours * HOURLY_RATE
endif
```

### 🧠 Boolean Truth Matrix
| Condition $P$ | Condition $Q$ | $P \text{ AND } Q$ | $P \text{ OR } Q$ | $\text{NOT } P$ |
| :---: | :---: | :---: | :---: | :---: |
| $\text{True}$ | $\text{True}$ | $\mathbf{\text{True}}$ | $\mathbf{\text{True}}$ | $\text{False}$ |
| $\text{True}$ | $\text{False}$ | $\text{False}$ | $\mathbf{\text{True}}$ | $\text{False}$ |
| $\text{False}$ | $\text{True}$ | $\text{False}$ | $\mathbf{\text{True}}$ | $\mathbf{\text{True}}$ |
| $\text{False}$ | $\text{False}$ | $\text{False}$ | $\text{False}$ | $\mathbf{\text{True}}$ |

> [!tip] 💡 Intuition: Short-Circuit Logic Optimisation
> Modern language parsers evaluate compound Boolean expressions from left to right:
> - **In an `AND` expression**: If the left operand evaluates to $\text{False}$, the entire expression is doomed to fail; the parser ==aborts evaluation of the right operand immediately==.
> - **In an `OR` expression**: If the left operand evaluates to $\text{True}$, success is guaranteed; the parser skips evaluating the right operand.
> - **Architectural Tip**: Place high-probability failure conditions on the left of an `AND` chain to preserve compute cycles, and position potential null-pointer or boundary checks first:
>   ```markdown
>   if totalCandidates <> 0 AND (accumulatedScore / totalCandidates) > 75 then
>   // The zero-divisor check protects the division operation from ever executing!
>   ```

> [!warning] ⚠️ Exam Trap & Edge Case: The Dangling `else` Paradox
> In nested selections lacking clean indentation or matching `endif` markers, an `else` clause may be erroneously associated with the wrong `if` condition:
> ```markdown
> // AMBIGUOUS & FLAWED:
> if accountActive = true then
>     if accountBalance > 0 then
>         output "In Good Standing"
> else
>     output "Account Disabled" // To which 'if' does this belong?
>
> // RESOLVED THROUGH EXPLICIT SCOPING:
> if accountActive = true then
>     if accountBalance > 0 then
>         output "In Good Standing"
>     endif
> else
>     output "Account Disabled"
> endif
> ```

---

## 🔁 Module 6: Repetition Control Structures (Loops) & Boundary Pathologies

### ⚙️ Pre-Test vs. Post-Test Execution
- ⤷ **Pre-Test Loop (`while` / `for`)**: The condition is evaluated **before** entering the loop body. If the condition begins as $\text{False}$, the loop body executes **zero** times.
- ⤷ **Post-Test Loop (`do-while` / `repeat-until`)**: The loop body executes once unconditionally **before** the terminating condition is evaluated. Guarantees a minimum of one execution.

> [!bug] 🚨 Lecture Material Discrepancy
> The provided lecture notes contain an explicit typographical corruption in the `while` loop demonstration:
> `count = count +st Loop)endwhile`
> The line conflated an increment operation with a parenthetical header. In standard structured pseudocode, the statement must read:
> ```markdown
> count = count + 1
> endwhile
> ```

### 🔢 The Anatomy of a Loop Control Variable (LCV)
Every functionally valid loop requires three distinct, non-negotiable operations on its Loop Control Variable (LCV):
1. **Initialisation**: The LCV must be set to a predictable start value before condition evaluation.
2. **Testing**: The LCV must be checked within the controlling Boolean expression.
3. **Alteration**: The LCV must be explicitly mutated within the loop body toward the termination criteria.

```
                  +---------------------------+
                  |  1. INITIALISE THE LCV    |
                  |     (e.g., count = 0)     |
                  +---------------------------+
                                |
                                v
                /-------------------------------\
               /       2. TEST CONDITION         \
              <       (e.g., count < MAX)         >
               \                                 /
                \-------------------------------/
                             |          |
                     True    |          | False
                     +-------+          +-----------------------+
                     |                                          |
                     v                                          v
      +-----------------------------+               +-----------------------+
      |      EXECUTE LOOP BODY      |               |     EXIT THE LOOP     |
      |   (Payload Operations)      |               |  (Continue Mainline)  |
      +-----------------------------+               +-----------------------+
                     |
                     v
      +-----------------------------+
      |    3. ALTER THE LCV         |
      |   (e.g., count = count + 1) |
      +-----------------------------+
                     |
                     +------> [ Loop Back to Step 2 ]
```

> [!warning] ⚠️ Exam Trap & Edge Case: The Infinite Loop & Off-by-One (OBOE)
> - ❌ **Infinite Loop**: Occurs when the alteration step is omitted or alters the LCV away from the boundary condition:
>   ```markdown
>   num counter = 1
>   while counter > 0
>       output counter
>       counter = counter + 1 // Will iterate endlessly until memory overflow
>   endwhile
>   ```
> - ❌ **Off-By-One Error (Fencepost Error)**: Confusion between `<` and `<=` or starting at index `0` versus `1`.
>   - Iterating from `0 to 10` using `<=` executes **11** times.
>   - Iterating from `0 to 10` using `<` executes **10** times.
>   - Always verify the boundary bounds: $\text{Iterations} = \text{Upper Limit} - \text{Lower Limit} + 1$ (for inclusive bounds).

### ⚙️ The Priming Read Pattern (Processing Sentinel Streams)
- ⤷ **The Priming Read**: An initial input statement placed immediately **before** entering the loop to establish the starting state of the LCV against a sentinel value (e.g., `eof`, `-1`, `"QUIT"`).
- ⤷ **The Subsequent Read**: An identical input statement placed at the **absolute base** of the loop body. This ensures that sentinel values terminate the loop immediately without processing invalid terminal data through the business logic.

```markdown
// CANONICAL SENTINEL LOOP WITH PRIMING READ:
output "Enter customer transaction amount (-1 to terminate): "
input transactionAmount                  // <--- Priming Read

while transactionAmount <> -1
    totalRevenue = totalRevenue + transactionAmount
    transactionCount = transactionCount + 1
    
    output "Enter next amount (-1 to terminate): "
    input transactionAmount              // <--- Subsequent Read
endwhile

output "Aggregate Revenue: £", totalRevenue
```

> [!warning] ⚠️ Exam Trap & Edge Case: Processing the Sentinel Value
> Failure to structure the loop via a priming and subsequent read pattern invariably causes the sentinel value itself to be processed into the accumulator:
> ```markdown
> // FLAWED ARCHITECTURE:
> while transactionAmount <> -1
>     input transactionAmount
>     totalRevenue = totalRevenue + transactionAmount // BUG: Adds -1 to aggregate total!
> endwhile
> ```

---

## 🏛️ Module 7: Modularisation & Mainline Logic Architecture

### ⚙️ The Procedural Triad
To prevent monolithic "spaghetti code", robust systems partition their execution pipeline across three discrete modular phases:
- ⤷ **`housekeeping()`**: Executes global initialisation, allocates file handles, prints welcoming banners, and conducts the priming read.
- ⤷ **`detailLoop()`**: Contains the primary algorithmic transformation, processes a single logical record, and issues the subsequent read.
- ⤷ **`endOfJob()` / `finish()`**: Finalises computation, prints accumulators and statistical summaries, closes open streams, and executes clean teardown.

```markdown
start
    Declarations
        num customerId
        num callsMade
        num callMinutes
        num calculatedBill
        num BASE_FEE = 30.00
        num PER_MINUTE_SURCHARGE = 0.15
        boolean isEndOfFile = false

    housekeeping()
    while not isEndOfFile
        detailLoop()
    endwhile
    finish()
stop

housekeeping()
    output "=== TELECOMMUNICATIONS BILLING ENGINE ==="
    output "Enter Customer ID (or -1 to terminate): "
    input customerId
    if customerId = -1 then
        isEndOfFile = true
    else
        output "Enter total calls and aggregate minutes: "
        input callsMade, callMinutes
    endif
return

detailLoop()
    calculatedBill = BASE_FEE + (callMinutes * PER_MINUTE_SURCHARGE)
    output "Account Ref: ", customerId, " | Balance Due: £", calculatedBill
    
    output "Enter Customer ID (or -1 to terminate): "
    input customerId
    if customerId = -1 then
        isEndOfFile = true
    else
        output "Enter total calls and aggregate minutes: "
        input callsMade, callMinutes
    endif
return

finish()
    output "Batch processing terminated successfully."
    output "System state: Nominal. Deallocating memory..."
return
```

> [!tip] 💡 Intuition: Modular Scope Isolation
> Regard modules as autonomous workstations in a factory line. The `mainline` supervisor directs traffic, but does not micromanage the nuts and bolts. Local variables declared inside a module are born when the module is called, live within its scope, and are destroyed at `return`. If a variable must persist across module boundaries without parameter passing, it must be explicitly defined as a **global declaration** within the mainline block.

---

## 🛡️ Module 8: Defensive Programming, Input Validation, & Data Sanitisation

### ⚙️ The Validation Loop Guard
- ⤷ **Defensive Programming Mandate**: Never trust external input. If an end-user is prompted for an integer age, they might enter `"thirty"`, `0`, or `-42`. 
- ⤷ **Defensive Validation Loop**: A pre-test loop nested immediately downstream of an input statement that traps execution until the supplied value conforms to mathematical and structural constraints.

```markdown
// Comprehensive Validation Pattern:
output "Please provide candidate age (18 - 65): "
input candidateAge

while not isNumeric(candidateAge) OR candidateAge < 18 OR candidateAge > 65
    output "INVALID ENTRY DETECTED."
    output "Age must be a numeric integer bounded between 18 and 65."
    output "Please re-enter candidate age: "
    input candidateAge
endwhile
```

### 🔢 High-Yield Enumeration: Common Validation Predicates
> [!check] 🔢 Common Defensive Validation Utility Functions
> - `isNumeric(val)`: Returns $\text{True}$ if the operand can be safely parsed into a valid real or integer number; returns $\text{False}$ if non-numeric characters exist.
> - `isChar(val)`: Returns $\text{True}$ if the payload length is strictly equal to $1$ character.
> - `isWhitespace(val)`: Detects whether a string input consists entirely of empty spaces, carriage returns, or tabs.
> - `isRange(val, min, max)`: Evaluates whether $min \le val \le max$.
> - `isEOF()` / `eof`: Validates whether the end of a targeted file buffer or stream has been reached.

---

## 🧪 Comprehensive Self-Assessment Examination

```markdown
**Question 1**: Identify the primary structural hazard inherent in reading a data record directly at the top of a standard sentinel-controlled while loop without a priming read.
✅ **Correct Answer: Option B — The loop body executes its processing logic against the terminating sentinel value.**
⤷ Logical Justification: Without a priming read preceding the loop and a subsequent read at the base, the sentinel value itself is read immediately before business calculations occur, skewing accumulators and producing invalid outputs.
⤷ Elimination Matrix:
   - Option A (The loop causes an immediate stack overflow) is false; it loops cleanly, merely with invalid data.
   - Option C (The loop will never terminate) is false; it will terminate once the sentinel is supplied, albeit after processing it.
   - Option D (The condition will fail to compile) is false; the syntax is valid, but semantically flawed.

---

**Question 2**: If `flagA = false` and `flagB = true`, what does the compound expression `not flagA OR flagB AND false` resolve to under standard operator precedence?
✅ **Correct Answer: Option A — True**
⤷ Logical Justification: 
   1. Unary `not` executes first: `not flagA` = `not false` = `true`.
   2. Conjunction `AND` executes next: `flagB AND false` = `true AND false` = `false`.
   3. Disjunction `OR` executes last: `true OR false` = `true`.
⤷ Elimination Matrix:
   - Option B (False) is false; failure to prioritise `AND` over `OR` would invert the result.
   - Option C (Undefined) is false; Boolean algebra is strictly deterministic.
```