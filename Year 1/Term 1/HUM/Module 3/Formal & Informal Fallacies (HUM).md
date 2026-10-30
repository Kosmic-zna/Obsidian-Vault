---
state: "[[Final]]"
---
# 🛡️ Module 3: Formal & Informal Fallacies

## 📖 Prerequisite Lexicon (From First Principles)
- **Fallacy** (from Latin *fallacia*, meaning "deceit" or "trick")
  - ⤷ **Definition**: A structural or psychological defect in an argument that causes it to be logically invalid or unsound, whilst often appearing superficially persuasive.
- **Formal Fallacy**
  - ⤷ **Definition**: An error exclusively in the **logical structure (form)** of a deductive argument. You can spot a formal fallacy purely by looking at the symbolic blueprint ($S$, $P$, $M$, $+$, $-$), without even knowing what the words mean in the real world.
- **Informal Fallacy**
  - ⤷ **Definition**: An error in the **content, context, or psychological relevance** of an argument. It cannot be detected by symbolic notation alone; you must examine the actual meaning of the sentences to see how the speaker is manipulating emotion, changing the subject, or making unwarranted assumptions.
- **Extension (Distribution)**
  - ⤷ **Definition**: The scope of a term—whether it refers to **every member** of its class (**Universal** = Capital letter: $M, P, S$) or **only a portion** of its class (**Particular** = lowercase letter: $m, p, s$).
  - ↳ **Subject Rule (Controlled by Quantity)**: Universal propositions ($A, E$) have **Universal** subjects ($S, P, M$). Particular propositions ($I, O$) have **Particular** subjects ($s, p, m$).
  - ↳ **Predicate Rule (Controlled by Quality)**: ==Affirmative propositions ($+$) ALWAYS have **Particular** predicates ($p, s, m$)==. ==Negative propositions ($-$) ALWAYS have **Universal** predicates ($P, S, M$)==.

---

## 🏛️ Part I: The Eight Canonical Rules & Formal Fallacies

> [!check] 🔢 High-Yield Enumeration: The Two Halves of Categorical Rules
> - **Rules 1 to 4 (Rules Governing Terms)**: Regulate the meaning, scope (extension), and placement of $M$, $S$, and $P$.
>   - ↳ *Fallacies*: (1) Equivocation, (2) Illicit Major, (3) Illicit Minor, (4) Misplaced Middle Term, (5) Undistributed Middle Term.
> - **Rules 5 to 8 (Rules Governing Propositions)**: Regulate the quality ($+$ vs. $-$) and quantity (Universal vs. Particular) of the statements.
>   - ↳ *Fallacies*: (1) Negative Conclusion Drawn from Affirmative Premises, (2) Exclusive/Negative Premises, (3) Two Particular Premises, (4) Universal Conclusion Drawn from a Particular Premise, (5) Affirmative Conclusion Drawn from a Negative Premise.

---

### 🧩 Section A: Rules Governing Terms (Rules 1–4)

#### 📐 Rule 1: The Middle Term Must Always Be Taken in the Same Sense
- **Formal Requirement**: The middle term ($M$) must maintain an identical definition and semantic reference across both the major and minor premises.
- **Associated Error**: **Fallacy of Equivocation** (also known as the *Fallacy of Four Terms* / *Quaternio Terminorum*).
  - ⤷ **Definition**: An argument that uses a single word or phrase to mean two different things across the premises, covertly splitting the middle term into two unrelated concepts.

> [!tip] 💡 Intuition: The Shifting Bridge
> A logical bridge ($M$) cannot connect two riverbanks ($S$ and $P$) if one half of the bridge is built in London and the other half in Manila. Even though the word looks identical on paper, shifting its meaning midway through leaves four distinct concepts in a three-term structure.

- **Worked Lecture Examples**:
  - ❌ *Invalid Example 1 (Mouse)*:
    - **Major Premise**: "Some handheld devices are mice." (Computer hardware peripheral)
    - **Minor Premise**: "A rodent is a mouse." (Small mammal with large ears and a long tail)
    - **Conclusion**: $\therefore$ "A handheld device is a rodent."
    - ↳ **Defect**: "Mouse/mice" shifts from electronic hardware to a biological animal.
  - ❌ *Invalid Example 2 (Word of God)*:
    - **Major Premise**: "Jesus is the Word of God." (Divine Logos / Second Person of the Trinity)
    - **Minor Premise**: "The Bible is the Word of God." (Sacred written scripture)
    - **Conclusion**: $\therefore$ "Jesus is the Bible."
    - ↳ **Defect**: "Word of God" shifts from a theological personhood sense to a textual sense.

---

#### 📐 Rule 2: The Major Term ($P$) and Minor Term ($S$) Cannot Have a Greater Extension in the Conclusion Than in the Premises
- **Formal Requirement**: A term that is **particular** (lowercase $p$ or $s$) in the premise ==cannot become **universal** (Capital $P$ or $S$) in the conclusion==. However, moving from Universal in the premise to Particular in the conclusion ($P \rightarrow p$ or $S \rightarrow s$) is completely valid.
- **Associated Errors**:
  - ⤷ **Fallacy of Illicit Major Term**: The major term ($P$, predicate of the conclusion) is particular ($p$) in the major premise, but becomes universal ($P$) in the conclusion ($p \rightarrow P$).
  - ⤷ **Fallacy of Illicit Minor Term**: The minor term ($S$, subject of the conclusion) is particular ($s$) in the minor premise, but becomes universal ($S$) in the conclusion ($s \rightarrow S$).

> [!tip] 💡 Intuition: Cause and Effect Scope
> The conclusion is the "effect" and the premises are the "cause." The effect can never be greater than its cause. If your evidence only examined *some* members of a group ($p$ or $s$), you cannot suddenly make a sweeping claim about *100%* of that group ($P$ or $S$) in your conclusion.

> [!warning] ⚠️ Exam Trap & Edge Case: Why Negative Conclusions Trigger Illicit Major
> Because **every negative conclusion ($-$) automatically makes its predicate Universal ($P$)**, any syllogism with a negative conclusion *must* have a Universal $P$ in the major premise. If the major premise is an $A$ proposition where $P$ is on the right side ($M + p$), drawing a negative conclusion ($S - P$ or $s - P$) will ==ALWAYS trigger the Fallacy of Illicit Major==!

- **Worked Lecture Examples**:
  - ❌ *Invalid Example 1 (Illicit Major — Hammers)*:
    - **Major Premise ($A$)**: "All hammers are tools." $\rightarrow M + p$
    - **Minor Premise ($E$)**: "But, no chisels are hammers." $\rightarrow S - M$
    - **Conclusion ($E$)**: $\therefore$ "No chisels are tools." $\rightarrow S - P$
    - ↳ **Symbolic Trace**: "Tools" is particular ($p$) in the major premise (predicate of $+$) but becomes universal ($P$) in the conclusion (predicate of $-$). Since $p \rightarrow P$, it commits **Illicit Major**.
  - ❌ *Invalid Example 2 (Illicit Major — Dogs & Cats)*:
    - **Major Premise ($A$)**: "All dogs are animals." $\rightarrow M + p$
    - **Minor Premise ($E$)**: "No cats are dogs." $\rightarrow S - M$
    - **Conclusion ($E$)**: $\therefore$ "Therefore, no cats are animals." $\rightarrow S - P$
    - ↳ **Symbolic Trace**: "Animals" expands illegally from $p$ to $P$ (**Illicit Major**).
  - ❌ *Invalid Example 3 (Illicit Minor — Whales & Mammals)*:
    - **Major Premise ($A$)**: "All whales are mammals." $\rightarrow P + m$
    - **Minor Premise ($A$)**: "All mammals are animals." $\rightarrow M + s$
    - **Conclusion ($A$)**: $\therefore$ "Therefore, all animals are whales." $\rightarrow S + p$
    - ↳ **Symbolic Trace**: "Animals" ($S$) is the predicate of an affirmative minor premise, making it particular ($s$). In the conclusion, "All animals" makes it universal ($S$). Since $s \rightarrow S$, it commits **Illicit Minor**.

---

#### 📐 Rule 3: The Middle Term ($M$) Should Not Occur in the Conclusion
- **Formal Requirement**: The middle term ($M$) must appear in both premises to link $S$ and $P$, and must completely vanish in the conclusion.
- **Associated Error**: **Fallacy of Misplaced Middle Term**.

> [!tip] 💡 Intuition: The Matchmaker's Exit
> Think of $M$ as a matchmaker who introduces $S$ and $P$ in the premises. Once $S$ and $P$ are united in the conclusion, the matchmaker must step out of the room. If $M$ intrudes into the conclusion, no independent relationship between $S$ and $P$ has actually been established.

- **Worked Lecture Examples**:
  - ❌ *Invalid Example 1 (Steward)*:
    - **Major Premise**: "A steward is a flight attendant." $\rightarrow M + p$
    - **Minor Premise**: "But, a steward is a male." $\rightarrow M + s$
    - **Conclusion**: $\therefore$ "A steward is a male flight attendant." $\rightarrow M + sp$
    - ↳ **Defect**: The middle term "steward" ($M$) is repeated as the subject of the conclusion.
  - ❌ *Invalid Example 2 (Filipinos & Dignity)*:
    - **Major Premise**: "All persons have dignity."
    - **Minor Premise**: "All Filipinos have dignity."
    - **Conclusion**: $\therefore$ "Therefore, all Filipinos are people with dignity."
    - ↳ **Defect**: "Dignity" ($M$) is dragged into the conclusion instead of cleanly concluding "All Filipinos are persons."

---

#### 📐 Rule 4: The Middle Term ($M$) Must Be Distributed Universally at Least Once in the Premises
- **Formal Requirement**: Across the two premises, the middle term must appear as a capital **$M$** at least once (either $M$ in the major premise, $M$ in the minor premise, or both). If both premises contain lowercase **$m$**, the argument is invalid.
- **Associated Error**: **Fallacy of Undistributed Middle Term**.

> [!tip] 💡 Intuition: Two Strangers in a Massive Country
> Principle: *"What is true of the whole is true of its parts, but what is true of a part is not necessarily true of another part."* If I tell you "Stewardesses are females ($P + m$)" and "Mothers are females ($S + m$)", both groups only occupy a small corner of the massive circle of "females" ($m$). Their circles never have to touch each other! At least one premise must cover 100% of the middle term ($M$) to guarantee an overlap.

- **Worked Lecture Examples**:
  - ❌ *Invalid Example 1 (Stewardesses & Mothers)*:
    - **Major Premise ($A$)**: "All stewardesses are females." $\rightarrow P + m$
    - **Minor Premise ($A$)**: "But, all mothers are females." $\rightarrow S + m$
    - **Conclusion ($A$)**: $\therefore$ "All mothers are stewardesses." $\rightarrow S + p$
    - ↳ **Symbolic Trace**: Both premises are affirmative ($+$) with "females" in the predicate position, rendering both $m$ and $m$.
  - ❌ *Invalid Example 2 (Filipinos & Dignity)*:
    - **Major Premise ($A$)**: "All persons have dignity." $\rightarrow P + m$
    - **Minor Premise ($A$)**: "All Filipinos have dignity." $\rightarrow S + m$
    - **Conclusion ($A$)**: $\therefore$ "Therefore, all Filipinos are persons." $\rightarrow S + p$
    - ↳ **Symbolic Trace**: Even though all three statements are factually true in the real world, the **form** is invalid ($m$ and $m$ in Figure 2 AAA), committing **Undistributed Middle Term**.
  - ❌ *Invalid Example 3 (Apples & Mangoes)*:
    - **Major Premise ($A$)**: "All apples are fruits." $\rightarrow P + m$
    - **Minor Premise ($I$)**: "But, some fruits are mangoes." $\rightarrow m + s$
    - **Conclusion ($I$)**: $\therefore$ "Some mangoes are apples." $\rightarrow s + p$
    - ↳ **Symbolic Trace**: In Premise 1, "fruits" is the predicate of an affirmative statement ($m$). In Premise 2, "some fruits" is a particular subject ($m$). Neither is capital $M$.

> [!bug] 🚨 Lecture Material Discrepancy (Slide Typo Alert)
> In your first PDF (`M3_L8-9... 1.pdf`, Slide 13), the symbolic translation for *"All stewardesses are females; But, all mothers are females"* is typed with capital letters as `$P + M$` and `$S + M$`, and the conclusion of the mango example *"Some mangoes are apples"* is typed with a capital `$S + p$`. Your second PDF (`M3_L8-9... 2.pdf`, Slide 40) corrects the first two to lowercase `$P + m$` and `$S + m$`, as affirmative predicates are always particular ($m$), and "Some mangoes" is a particular subject ($s$). Always use lowercase **$m$** for affirmative predicates on your exam!

---

### 📜 Section B: Rules Governing Propositions (Rules 5–8)

#### 📐 Rule 5: Two Affirmative Premises Cannot Give a Negative Conclusion
- **Formal Requirement**: If both premises are affirmative ($+$ and $+$), the conclusion must also be affirmative ($+$).
- **Associated Error**: **Fallacy of a Negative Conclusion Drawn from Affirmative Premises**.

> [!tip] 💡 Intuition: Inclusion Cannot Breed Exclusion
> A negative conclusion expresses separation. If Premise 1 unites $M$ and $P$ ($+$), and Premise 2 unites $S$ and $M$ ($+$), there is zero exclusionary force in the premises to tear $S$ and $P$ apart ($-$) in the conclusion.

- **Worked Lecture Examples**:
  - ❌ *Invalid Example 1 (Diamonds & Stones)*:
    - **Major Premise ($A$)**: "All stones are hard." ($+$)
    - **Minor Premise ($I$)**: "Some diamonds are stones." ($+$)
    - **Conclusion ($O$)**: $\therefore$ "Some diamonds are not hard." ($-$)
    - ↳ **Symbolic Trace**: $+ \text{ and } + \rightarrow -$ violates Rule 5.
  - ❌ *Invalid Example 2 (Goldfish)*:
    - **Major Premise ($A$)**: "All fish have fins." $\rightarrow M + p$
    - **Minor Premise ($A$)**: "All goldfish are fish." $\rightarrow S + m$
    - **Conclusion ($E$)**: $\therefore$ "Therefore, all goldfish do not have fins." $\rightarrow S - P$
    - ↳ **Symbolic Trace**: Drawing a negative conclusion ($S - P$) from two affirmative premises ($M + p$ and $S + m$) violates Rule 5 (and simultaneously triggers Illicit Major: $p \rightarrow P$).
    - → *Valid Correction*: "Therefore, all goldfish have fins." ($S + p$).

---

#### 📐 Rule 6: From Two Negative Premises, Nothing Follows
- **Formal Requirement**: At least one premise must be affirmative ($+$). Two negative premises ($-$ and $-$) can never yield a valid conclusion.
- **Associated Error**: **Fallacy of Exclusive / Negative Premises**.

> [!tip] 💡 Intuition: Total Isolation
> If you know that $S$ is completely separate from $M$, and $P$ is also completely separate from $M$, you have established zero relationship between $S$ and $P$. $S$ and $P$ might be identical to each other, or miles apart—the middle term cannot mediate if it rejects both sides.

- **Worked Lecture Examples**:
  - ❌ *Invalid Example 1 (Chair, Table, Pen)*:
    - **Major Premise ($E$)**: "A chair is not a table." ($-$)
    - **Minor Premise ($E$)**: "But, a table is not a pen." ($-$)
    - **Conclusion ($E$)**: $\therefore$ "A pen is not a chair." ($-$)
  - ❌ *Invalid Example 2 (Reptiles, Mammals, Dogs)*:
    - **Major Premise ($E$)**: "No reptiles are mammals." $\rightarrow M - P$
    - **Minor Premise ($E$)**: "No dogs are reptiles." $\rightarrow S - M$
    - **Conclusion ($E$)**: $\therefore$ "Therefore, no dogs are mammals." $\rightarrow S - P$
    - ↳ **Defect**: Even though the premises are true, two negative premises ($-$ and $-$) sever the logical bridge, producing a false conclusion ("no dogs are mammals") and committing the **Fallacy of Exclusive/Negative Premises**.

---

#### 📐 Rule 7: From Two Particular Premises, Nothing Follows
- **Formal Requirement**: At least one premise must be **Universal** ($A$ or $E$). If both premises are **Particular** ($I$ and $I$, $O$ and $O$, or $I$ and $O$), no valid conclusion can follow.
- **Associated Error**: **Fallacy of Two Particular Premises**.

> [!tip] 💡 Intuition: Why All 3 Combinations of Particular Premises Collapse
> 1. **Case $\alpha$ (Both Particular Affirmative: $I + I$)**: Every subject and predicate in an $I$ statement is particular ($m + p$ and $s + m$). Zero terms are universal, which automatically violates **Rule 4 (Undistributed Middle)**.
> 2. **Case $\beta$ (Both Particular Negative: $O + O$)**: Both premises are negative ($-$ and $-$), which automatically violates **Rule 6 (Exclusive Premises)**.
> 3. **Case $\gamma$ (One Affirmative, One Negative: $I + O$ or $O + I$)**: An $I$ statement distributes zero terms, and an $O$ statement distributes only its predicate (1 universal term total). However, having one negative premise forces a negative conclusion (by Rule 8), which requires a universal predicate $P$ (requiring $P$ to be universal in the premise by Rule 2). Since your only universal slot must be used on $M$ (Rule 4), $P$ is left particular ($p \rightarrow P$), inevitably violating **Rule 2 (Illicit Major)** or **Rule 4 (Undistributed Middle)**!

- **Worked Lecture Examples**:
  - ❌ *Invalid Example 1 (Cats, Pets, Pythons — $I$ and $O$)*:
    - **Major Premise ($I$)**: "Some cats are pets." $\rightarrow m + p$
    - **Minor Premise ($O$)**: "But, some pythons are not cats." $\rightarrow s - M$
    - **Conclusion ($O$)**: $\therefore$ "Some pythons are not pets." $\rightarrow s - P$
    - ↳ **Symbolic Trace**: Two particular premises ($I$ and $O$). Notice how it also commits **Illicit Major** ($p \rightarrow P$).
  - ❌ *Invalid Example 2 (Horseback Riders & Drivers — $O$ and $I$)*:
    - **Major Premise ($O$)**: "Some horseback riders are not males." $\rightarrow m - P$
    - **Minor Premise ($I$)**: "But, some drivers are horseback riders." $\rightarrow s + m$
    - **Conclusion ($O$)**: $\therefore$ "Some drivers are not males." $\rightarrow s - P$
    - ↳ **Symbolic Trace**: Two particular premises ($O$ and $I$). Notice how it also commits **Undistributed Middle** ($m$ and $m$).
  - ❌ *Invalid Example 3 (Reptiles, Mammals, Dogs — $I$ and $I$)*:
    - **Major Premise ($I$)**: "Some reptiles are mammals." $\rightarrow m + p$
    - **Minor Premise ($I$)**: "Some dogs are reptiles." $\rightarrow s + m$
    - **Conclusion ($I$)**: $\therefore$ "Therefore, some dogs are mammals." $\rightarrow s + p$
    - ↳ **Symbolic Trace**: Both premises are $I$ propositions (**Two Particular Premises** and **Undistributed Middle**).

---

#### 📐 Rule 8: The Conclusion Always Follows the Weaker Premise
- **Formal Requirement**:
  - ↳ **Hierarchy of Weakness in Quantity**: **Particular** ($I, O$) is weaker than **Universal** ($A, E$). If one premise is Universal and the other is Particular, the conclusion ==MUST be Particular==.
  - ↳ **Hierarchy of Weakness in Quality**: **Negative** ($E, O$) is weaker than **Affirmative** ($A, I$). If one premise is Affirmative and the other is Negative, the conclusion ==MUST be Negative==.
- **Associated Errors**:
  - ⤷ **Fallacy of a Universal Conclusion Drawn from a Particular Premise**: One premise is particular, yet the conclusion is universal.
  - ⤷ **Fallacy of an Affirmative Conclusion Drawn from a Negative Premise**: One premise is negative, yet the conclusion is affirmative.

> [!tip] 💡 Intuition: A Chain Is Only as Strong as Its Weakest Link
> If half of your evidence is partial ("Some") or exclusionary ("Not"), your final deduction cannot magically upgrade itself into a 100% universal or positive guarantee.

- **Worked Lecture Examples**:
  - ❌ *Invalid Example 1 (Universal from Particular — Special Children)*:
    - **Major Premise ($A$)**: "All mongoloids are mentally retarded." $\rightarrow M + p$
    - **Minor Premise ($I$)**: "But, some mongoloids are special children." $\rightarrow m + s$
    - **Conclusion ($A$)**: $\therefore$ "All special children are mentally retarded." $\rightarrow S + p$
    - ↳ **Symbolic Trace**: Premise 2 is Particular ($I$), but the conclusion is Universal ($A$). Commits **Universal Conclusion Drawn from a Particular Premise** (and **Illicit Minor**: $s \rightarrow S$).
  - ❌ *Invalid Example 2 (Universal from Particular — Cats & Marsupials)*:
    - **Major Premise ($A$)**: "All cats are marsupials." $\rightarrow M + p$
    - **Minor Premise ($I$)**: "Some mammals are cats." $\rightarrow s + m$
    - **Conclusion ($A$)**: $\therefore$ "Therefore, all mammals are marsupials." $\rightarrow S + p$
    - ↳ **Symbolic Trace**: Mood $AAI$ is required, not $AAA$. Commits **Universal Conclusion Drawn from a Particular Premise** (and **Illicit Minor**: $s \rightarrow S$).
    - → *Valid Correction*: "Therefore, **some** mammals are marsupials." ($s + p$).
  - ❌ *Invalid Example 3 (Affirmative from Negative — Cats & Marsupials)*:
    - **Major Premise ($E$)**: "All cats are NOT marsupials." $\rightarrow M - P$
    - **Minor Premise ($I$)**: "Some mammals are cats." $\rightarrow s + m$
    - **Conclusion ($I$)**: $\therefore$ "Therefore, some mammals are marsupials." $\rightarrow s + p$
    - ↳ **Symbolic Trace**: Major premise is Negative ($E$), Minor is Affirmative ($I$), but Conclusion is Affirmative ($I$). Commits **Affirmative Conclusion Drawn from a Negative Premise**.
    - → *Valid Correction*: "Therefore, some mammals are **not** marsupials." ($s - P$, Valid Figure 1 *Ferio*).

---

### 🧪 Section C: Solved Lecture Drills (From Slides)

> [!example] 🧪 Step-by-Step Derivation of the 4 Lecture Drills
> **Drill 1**:
> - *Major Premise*: "Rape is immoral;" $\rightarrow P + m$
> - *Minor Premise*: "But, cheating is immoral;" $\rightarrow S + m$
> - *Conclusion*: $\therefore$ "Cheating is rape." $\rightarrow S + p$
> - ✅ **Fallacy Committed**: **Fallacy of Undistributed Middle Term** (Violates Rule 4). "Immoral" is the predicate of two affirmative propositions, making it particular ($m$) in both premises.
>
> **Drill 2**:
> - *Major Premise*: "A martyr dies for a noble cause;" $\rightarrow M + p$
> - *Minor Premise*: "But, some wives are martyrs;" $\rightarrow s + m$
> - *Conclusion*: $\therefore$ "Some wives do not die for a noble cause." $\rightarrow s - P$
> - ✅ **Fallacies Committed**:
>   1. **Fallacy of a Negative Conclusion Drawn from Affirmative Premises** (Violates Rule 5: $+ \text{ and } + \rightarrow -$).
>   2. **Fallacy of Illicit Major Term** (Violates Rule 2: "dies for a noble cause" is particular $p$ in the major premise, but becomes universal $P$ in the negative conclusion).
>
> **Drill 3**:
> - *Major Premise*: "Waiters wait;"
> - *Minor Premise*: "But, true love waits;"
> - *Conclusion*: $\therefore$ "True love is a waiter."
> - ✅ **Fallacy Committed**: **Fallacy of Equivocation** (Violates Rule 1) *or* **Fallacy of Undistributed Middle Term** (Violates Rule 4).
>   - ↳ *Why Equivocation*: "Wait" in the first premise means serving tables professionally as restaurant staff, whereas "waits" in the second premise means enduring patiently over time.
>   - ↳ *Why Undistributed Middle*: Even if taken in the same literal sense ("things that wait"), both premises are affirmative with "wait(s)" in the predicate ($P + m$ and $S + m$), leaving $m$ undistributed.
>
> **Drill 4**:
> - *Major Premise*: "Horror stories send chilling effects;" $\rightarrow M + p$
> - *Minor Premise*: "But, murder stories are horror stories;" $\rightarrow S + m$
> - *Conclusion*: $\therefore$ "Murder stories don't send chilling effects." $\rightarrow S - P$
> - ✅ **Fallacies Committed**:
>   1. **Fallacy of a Negative Conclusion Drawn from Affirmative Premises** (Violates Rule 5: $+ \text{ and } + \rightarrow -$).
>   2. **Fallacy of Illicit Major Term** (Violates Rule 2: "send chilling effects" expands from $p \rightarrow P$).

---

### 📊 Master Reference Table: The 8 Rules & Formal Fallacies

| Rule # | Category | Core Requirement | Formal Fallacy Committed | Symbolic Red Flag |
| :--- | :--- | :--- | :--- | :--- |
| **Rule 1** | Terms | $M$ must have the exact same meaning in both premises. | **Fallacy of Equivocation** | 1 word = 2 different definitions |
| **Rule 2** | Terms | $P$ and $S$ cannot expand from particular in premise to universal in conclusion. | **Fallacy of Illicit Major**<br>**Fallacy of Illicit Minor** | Premise $p \rightarrow$ Conclusion $P$<br>Premise $s \rightarrow$ Conclusion $S$ |
| **Rule 3** | Terms | $M$ must never appear in the conclusion. | **Fallacy of Misplaced Middle Term** | $M$ or $m$ is written in Line 3 |
| **Rule 4** | Terms | $M$ must be Universal (Capital $M$) at least once in the premises. | **Fallacy of Undistributed Middle Term** | Premise 1 has $m$ AND Premise 2 has $m$ |
| **Rule 5** | Props | Two affirmative premises ($+, +$) must yield an affirmative conclusion ($+$). | **Fallacy of a Negative Conclusion Drawn from Affirmative Premises** | Premise $1(+)$ + Premise $2(+)$ $\rightarrow$ Conclusion $(-)$ |
| **Rule 6** | Props | Two negative premises ($-, -$) yield no valid conclusion. | **Fallacy of Exclusive / Negative Premises** | Premise $1(-)$ + Premise $2(-)$ |
| **Rule 7** | Props | Two particular premises ($I, O$) yield no valid conclusion. | **Fallacy of Two Particular Premises** | $I+I$, $O+O$, $I+O$, or $O+I$ |
| **Rule 8** | Props | Conclusion must follow the weaker premise (Particular $<$ Universal; $-$ $<$ $+$). | **Universal Conclusion from a Particular Premise**<br>**Affirmative Conclusion from a Negative Premise** | $A/E + I/O \rightarrow A/E$ conclusion<br>$+ \text{ and } - \rightarrow +$ conclusion |

---

## 🗣️ Part II: Informal Fallacies

### 🎯 Section A: Fallacies of Relevance (From Your Lecture Notes)
*Fallacies of relevance occur when the premises are logically irrelevant to the conclusion; they appear persuasive only because they trigger emotional, psychological, or personal distractions.*

#### 1. **Argumentum ad Baculum (Appeal to Force / Stick)**
- ⤷ **Definition**: Attempting to win an argument or force the acceptance of a conclusion by threatening physical force, coercion, intimidation, or negative consequences rather than providing logical evidence (*baculum* = Latin for "club" or "stick").
  - ↳ **Mechanism**: Replaces rational justification with fear ("Agree with me, or else you will suffer harm").
  - → *Example*: "Chairman, you should vote in favour of my budget proposal, because if you don't, I will make sure the board fires you next week."

#### 2. **Argumentum ad Misericordiam (Appeal to Pity)**
- ⤷ **Definition**: Attempting to prove a factual conclusion or evade a rule by manipulating the listener's sympathy, guilt, or compassion rather than addressing the objective merits of the case (*misericordia* = Latin for "pity" or "mercy").
  - ↳ **Mechanism**: Substitutes emotional heartbreak for logical proof.
  - → *Example*: "Professor, I deserve an A on this logic exam because my dog passed away last night, I haven't slept in three days, and my parents will disown me if I fail."

#### 3. **Argumentum ad Hominem (Attack on the Person)**
- ⤷ **Definition**: Rejecting someone's argument by attacking the **person** making the claim (their character, circumstances, or past actions) rather than refuting their actual reasoning or evidence (*ad hominem* = Latin for "to the man/person").
  - ↳ **Sub-Type 1: Abusive Ad Hominem** (Direct personal insult): Attacking the speaker's intelligence, morals, or appearance.
  - ↳ **Sub-Type 2: Circumstantial Ad Hominem**: Dismissing a person's argument by claiming they are only making it because of their personal situation, job, nationality, or self-interest.
    - → *Example (Circumstantial)*: "Of course Dr Reyes argues that hospital funding should be increased—she's a surgeon! Naturally she would say that to line her own pockets, so we can ignore her data."
  - ↳ **Sub-Type 3: Tu Quoque ("You Too" / Appeal to Hypocrisy)**: Deflecting a valid criticism or argument by pointing out that the accuser fails to practise what they preach.
    - → *Example (Tu Quoque)*: "Dad, you say smoking is fatal and that I should quit, but you've been smoking a pack a day since you were twenty! Therefore, smoking isn't bad for me." (Pointing out the father's hypocrisy does not change the medical fact that smoking causes cancer).

#### 4. **Fallacy of Accident (Destroying the Exception / Dicto Simpliciter)**
- ⤷ **Definition**: Applying a general rule rigidly to a specific, atypical case where accidental (special) circumstances make the rule inapplicable.
  - ↳ **Mechanism**: Treats a broad rule of thumb as an absolute, exceptionless law, ignoring context.
  - → *Example*: "Cutting people with a sharp knife is a crime (General Rule). Surgeons cut people with sharp knives during operations (Specific Exceptional Case). Therefore, surgeons are criminals."

#### 5. **Ignoratio Elenchi (Irrelevant Conclusion / Missing the Point)**
- ⤷ **Definition**: Presenting premises that legitimately support *one* conclusion, but then claiming that they prove a *completely different* conclusion (*ignoratio elenchi* = Latin for "ignorance of the refutation").
  - ↳ **Mechanism**: The speaker argues passionately and coherently, but misses the specific target of the debate.
  - → *Example*: In a courtroom trial determining whether a defendant is *guilty of murder*, the prosecutor spends an hour proving that *murder is a horrific, gruesome crime*. (Proving that murder is horrible does not prove that *this specific defendant* committed it).

#### 6. **Red Herring**
- ⤷ **Definition**: Deliberately introducing an irrelevant, tangential, or emotionally charged topic in the middle of an argument to divert attention away from the original issue.
  - ↳ **Origin / Intuition**: Named after the practice of dragging a pungent smoked kipper (red herring) across a trail to throw hunting hounds off the scent.
  - ↳ **Distinction from *Ignoratio Elenchi***: In *Ignoratio Elenchi*, the speaker genuinely thinks their premises prove the conclusion (they miss the point). In a *Red Herring*, the speaker **changes the subject** mid-conversation to escape a losing topic.
  - → *Example*: "You ask why our company polluted the local river? Well, think about how many thousands of jobs our factory provides to the community! Without us, families would starve!"

#### 7. **Genetic Fallacy**
- ⤷ **Definition**: Judging the truth or falsity of an idea, claim, or institution solely based on its **historical origin, source, or how it began**, rather than evaluating its current evidence or merits.
  - ↳ **Mechanism**: Assumes that if the birthplace or original motive of an idea was flawed (or noble), the idea itself must permanently remain flawed (or true).
  - → *Example*: "You shouldn't believe the chemical structure of the benzene ring—the scientist Kekulé only discovered it because he had a bizarre daydream about a snake biting its own tail!" (How the idea originated has no bearing on whether the chemical formula is experimentally true).

---

### 🖼️ Section B: Other Informal Fallacies (From Your Handout Image)

#### 1. **Argumentum ad Verecundiam (Appeal to Inappropriate Authority)**
> [!bug] 🚨 Lecture Material Discrepancy (Spelling Alert)
> Your handout image spells this **`Argumentum ad Vericundiam`** (with an *i*). The classical Latin spelling is **`Argumentum ad Verecundiam`** (with an *e*, meaning "appeal to modesty/reverence"). Recognise both spellings for your test!

- ⤷ **Definition**: Insisting that a claim is true simply because an authority figure, celebrity, or famous person said so—especially when that person **lacks relevant expertise** in the specific field being discussed.
  - → *Example*: "This cryptocurrency is guaranteed to triple in value by December because my favourite basketball player endorsed it in a commercial."

#### 2. **Argumentum ad Ignorantiam (Appeal to Ignorance)**
- ⤷ **Definition**: Arguing that a statement must be **TRUE** simply because it has never been proven **FALSE**—or, conversely, arguing that a statement must be **FALSE** simply because it has never been proven **TRUE**.
  - ↳ **Intuition**: *"Absence of evidence is not evidence of absence."* A lack of proof only proves that we do not know yet; it cannot be weaponised as positive proof for either side.
  - → *Example*: "No scientist has ever been able to prove that telepathy does not exist; therefore, telepathy is real."

#### 3. **Hasty Generalisation (*The Part Constitutes the Whole*)**
- ⤷ **Definition**: Drawing a sweeping, universal conclusion about an entire population or group based on a sample size that is **too small, unrepresentative, or atypical**.
  - ↳ **Mechanism**: Jumping to conclusions from isolated anecdotes ("What is true of one or two instances must be true of all").
  - → *Example*: "My first two boyfriends from Manila were unfaithful. Therefore, all men from Manila are cheaters."

#### 4. **Post Hoc, Ergo Propter Hoc (*After This, Therefore Because of This*)**
- ⤷ **Definition**: Assuming that just because Event $A$ happened *before* Event $B$ in time, Event $A$ must have **caused** Event $B$ (also known as the *False Cause* fallacy).
  - ↳ **Mechanism**: Confuses chronological sequence (correlation/coincidence) with causal connection (causation).
  - → *Example*: "I wore my lucky unwashed socks to the exam this morning and got the highest score in the class. Therefore, my socks caused me to ace the test."

#### 5. **Slippery Slope (*"Si O.A." Fallacy*)**
- ⤷ **Definition**: Arguing without sufficient evidence that permitting a small, harmless first step will inevitably trigger an unstoppable domino effect of extreme, catastrophic consequences.
  - ↳ **Local Mnemonic (*"Si O.A." / Over-Acting Fallacy*)**: Immediately jumping from a minor action to an exaggerated doomsday scenario.
  - → *Example*: "If you let your daughter go to a study group until 7:00 PM today, tomorrow she'll stay out past midnight, next month she'll drop out of university, and within a year she'll be living on the streets!"

#### 6. **Weak Analogy (*"Parang ganito lang 'yan" Fallacy*)**
- ⤷ **Definition**: Comparing two things that share a few superficial similarities to conclude that they must also be alike in another respect, whilst ignoring **massive, fundamental differences** between them.
  - ↳ **Local Mnemonic (*"Parang ganito lang 'yan"*)**: Forcing a flawed comparison ("It's just like this...") where the two items are completely incomparable on the point that matters.
  - → *Example*: "Employees are just like nails. Just as you have to hit nails on the head to make them work, you have to bully and strike fear into employees to make them productive."

#### 7. **Petitio Principii (Begging the Question / Circular Reasoning)**
- ⤷ **Definition**: An argument in which the conclusion to be proven is **already assumed (explicitly or in disguised wording) inside the premises**.
  - ↳ **Intuition**: Trying to lift yourself off the ground by pulling on your own shoelaces. Instead of offering independent evidence, the speaker just repeats the claim in different words ($A \text{ is true because } A \text{ is true}$).
  - → *Example*: "Paranormal ghosts are real because I have experienced supernatural spirits with my own eyes." (Assuming that what you experienced was a "supernatural spirit" already presupposes that ghosts are real).

#### 8. **Complex Question (*"Wala kang lusot" Fallacy* / Loaded Question)**
- ⤷ **Definition**: Asking a single question that secretly smuggles in an unproven, incriminating presupposition, such that **any direct answer ("Yes" or "No") traps the respondent into admitting guilt**.
  - ↳ **Local Mnemonic (*"Wala kang lusot" — No Way Out*)**: A rhetorical trap where both "Yes" and "No" condemn you.
  - → *Example*: "Have you stopped cheating on your logic exams?"
    - ↳ If you answer **"Yes"**: You admit you used to cheat in the past.
    - ↳ If you answer **"No"**: You admit you are still cheating right now.

#### 9. **False Dilemma (*"No Choice" Fallacy* / Black-and-White Fallacy)**
- ⤷ **Definition**: Artificially restricting a complex situation to **only two extreme choices** ("Either $A$ or $B$"), whilst ignoring viable middle ground or alternative options.
  - ↳ **Local Mnemonic (*"No Choice" Fallacy*)**: Forcing the listener into a corner by pretending no third option exists.
  - → *Example*: "Either you support every single policy of the administration, or you hate your country and want it to collapse." (Ignores the obvious third option: loving your country whilst criticising specific policies).

---

### 📊 Master Cheat-Sheet: Informal Fallacies Quick-Spotter

| Fallacy Name | Latin / Local Tag | 3-Word Trigger / Core Trick |
| :--- | :--- | :--- |
| **Appeal to Force** | *Ad Baculum* | Threatening harm / coercion |
| **Appeal to Pity** | *Ad Misericordiam* | Manipulating sympathy / guilt |
| **Personal Attack** | *Ad Hominem* (Abusive, Circumstantial, *Tu Quoque*) | Attacking the speaker |
| **Fallacy of Accident** | *Dicto Simpliciter* | Ignoring valid exceptions |
| **Irrelevant Conclusion** | *Ignoratio Elenchi* | Missing the point |
| **Red Herring** | Deliberate Diversion | Changing the subject |
| **Genetic Fallacy** | Origin Bias | Judging by origin |
| **Appeal to Authority** | *Ad Verecundiam* (*Vericundiam*) | Citing non-experts / celebrities |
| **Appeal to Ignorance** | *Ad Ignorantiam* | Unproven $\neq$ True/False |
| **Hasty Generalisation** | *The Part Constitutes the Whole* | Sample too small |
| **False Cause** | *Post Hoc, Ergo Propter Hoc* | Sequence $\neq$ Causation |
| **Slippery Slope** | *Si O.A. Fallacy* | Exaggerated domino effect |
| **Weak Analogy** | *Parang ganito lang 'yan Fallacy* | Flawed, superficial comparison |
| **Begging the Question** | *Petitio Principii* | Circular reasoning ($A \rightarrow A$) |
| **Complex Question** | *Wala kang lusot Fallacy* | Trapped "Yes/No" presupposition |
| **False Dilemma** | *No Choice Fallacy* | Fake "Either-Or" extremes |