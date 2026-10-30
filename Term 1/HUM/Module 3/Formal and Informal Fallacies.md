# 🏛️ Module 3: Formal & Informal Logical Fallacies

## 📐 Part I: The Symbolic Engine (Term Distribution Quick-Reference)

- **Extension (Distribution)**
  - ⤷ **Definition**: Determines whether a syllogistic term refers to every single member of its class (**Universal** = Capital letter: $M, P, S$) or merely a partial subset of its class (**Particular** = lowercase letter: $m, p, s$)[cite: 1, 7, 46].
  - ↳ **The Two Golden Laws of Extension**:
    1. **Quantity Controls the Subject ($S$)**: Universal quantifiers (*All, Every, No*) make the subject **Universal** ($S, M, P$)[cite: 1, 47]. Particular quantifiers (*Some, Not all, Most*) make the subject **Particular** ($s, m, p$)[cite: 1, 47].
    2. **Quality Controls the Predicate ($P$)**: ==All Affirmative propositions ($+$) have Particular predicates ($p, m, s$)==[cite: 1, 47]. ==All Negative propositions ($-$) have Universal predicates ($P, M, S$)==[cite: 1, 47].

> [!tip] 💡 Intuition: Why Do Negative Statements Have Capital Predicates?
> Imagine saying, *"All dogs are animals"* ($M+p$)[cite: 1, 60]. Do dogs make up **100%** of all animals on Earth? No—only *some* animals are dogs, so the predicate is lowercase ($p$)[cite: 1]. Now imagine saying, *"No dogs are cats"* ($S-M$)[cite: 60]. To guarantee that not a single dog is a cat, you must mentally inspect **100% of all cats in existence** and exiled dogs from the entire cat kingdom[cite: 1]. Because the entire class is excluded, negative predicates are always **Universal (Capital)**[cite: 1, 47].

| Prop | Standard Template | Quality | Subject Extension | Predicate Extension | Symbolic Pattern |
| :---: | :--- | :---: | :---: | :---: | :---: |
| **$A$** | All $S$ are $P$[cite: 1] | Affirmative ($+$)[cite: 1, 47] | Universal ($S$)[cite: 1, 47] | Particular ($p$)[cite: 1, 47] | $S + p$[cite: 1, 49] |
| **$E$** | No $S$ are $P$ / All $S$ are not $P$[cite: 1] | Negative ($-$)[cite: 1, 47] | Universal ($S$)[cite: 1, 47] | Universal ($P$)[cite: 1, 47] | $S - P$[cite: 1, 53] |
| **$I$** | Some $S$ are $P$[cite: 1] | Affirmative ($+$)[cite: 1, 47] | Particular ($s$)[cite: 1, 47] | Particular ($p$)[cite: 1, 47] | $s + p$[cite: 1, 55] |
| **$O$** | Some $S$ are not $P$ / Not all $S$ are $P$[cite: 1] | Negative ($-$)[cite: 1, 47] | Particular ($s$)[cite: 1, 47] | Universal ($P$)[cite: 1, 47] | $s - P$[cite: 1, 51] |

---

## ⚖️ Part II: The Eight Canonical Rules & Formal Fallacies

> [!check] 🔢 High-Yield Enumeration: The 8 Rules vs. 10 Formal Fallacies
> The eight classical rules of the categorical syllogism are divided into two operational halves[cite: 1, 4, 42]:
> - **Rules 1–4 (Governing the Terms)** yield **5 Fallacies**: (1) Equivocation, (2) Illicit Major, (3) Illicit Minor, (4) Misplaced Middle Term, and (5) Undistributed Middle Term[cite: 4, 71].
> - **Rules 5–8 (Governing the Propositions)** yield **5 Fallacies**: (6) Negative Conclusion Drawn from Affirmative Premises, (7) Exclusive / Negative Premises, (8) Two Particular Premises, (9) Universal Conclusion Drawn from a Particular Premise, and (10) Affirmative Conclusion Drawn from a Negative Premise[cite: 4, 72].

### 🧩 A. Rules Governing the Terms (Rules 1 to 4)

#### 🛡️ Rule 1: The Middle Term Must Always Be Taken in the Same Sense[cite: 1, 5, 8, 43]
- **Formal Requirement**: The middle term ($M$) must maintain an identical semantic definition, nuance, and objective reference across both the major and minor premises[cite: 1].
  - ⤷ **Associated Fallacy**: **Fallacy of Equivocation** (classically termed *Quaternio Terminorum* or the **Fallacy of Four Terms**)[cite: 1, 6, 8, 43].
  - ↳ **Mechanism**: Using a single word or phrase to signify two entirely different concepts covertly splits the middle term into two disconnected terms ($M_1$ and $M_2$), destroying the logical bridge between $S$ and $P$[cite: 1, 8].
  - ❌ *Invalid Example 1 (Hardware vs. Biology)*:
    - **Major Premise**: Some handheld devices are mice.[cite: 1, 9]
    - **Minor Premise**: A rodent is a mouse.[cite: 1, 9]
    - **Conclusion**: $\therefore$ A handheld device is a rodent.[cite: 1, 9]
    - **Defect**: "Mice/mouse" shifts meaning from a computer peripheral to a biological mammal[cite: 1, 8, 9].
  - ❌ *Invalid Example 2 (Theological vs. Textual)*:
    - **Major Premise**: Jesus is the Word of God.[cite: 44]
    - **Minor Premise**: The Bible is the Word of God.[cite: 44]
    - **Conclusion**: $\therefore$ Jesus is the Bible.[cite: 44]
    - **Defect**: "Word of God" shifts from the incarnate *Logos* (divine person) to Sacred Scripture (written text).

> [!tip] 💡 Intuition: The Shifting Bridge
> Imagine building a bridge across a river where the north half is built in London and the south half is built in Manila, simply because both streets are named "King's Road." If you try to walk across, you fall into the abyss. Equivocation is pretending two different locations are the same bridge just because they share a name[cite: 1, 8].

---

#### 🛡️ Rule 2: The Major Term ($P$) and Minor Term ($S$) Cannot Have a Greater Extension in the Conclusion Than in the Premises[cite: 1, 5, 10, 45]
- **Formal Requirement**: ==If a term ($S$ or $P$) is particular/undistributed ($s, p$) in its premise, it is strictly forbidden from becoming universal/distributed ($S, P$) in the conclusion==[cite: 1, 10, 45, 59, 61]. (Note: Shrinking from Universal $\rightarrow$ Particular [$S \rightarrow s$ or $P \rightarrow p$] is completely legal; only expanding from Particular $\rightarrow$ Universal [$s \rightarrow S$ or $p \rightarrow P$] is a crime!)[cite: 59, 61]
  - ⤷ **Associated Fallacy 2A**: **Fallacy of Illicit Major Term**[cite: 1, 6, 10, 58]
    - ↳ **Mechanism**: The Major Term is particular ($p$) in the Major Premise, but illegally expands into a universal ($P$) in the Conclusion ($p \rightarrow P$)[cite: 1, 10, 11, 58]. ==Almost always triggered when an Affirmative Major Premise ($+$) is paired with a Negative Minor Premise ($-$) in Figure 1==[cite: 11, 60].
    - ❌ *Invalid Example 1*:
      - **Major Premise ($A$)**: All hammers are tools. $\rightarrow M + p$[cite: 1, 11, 58]
      - **Minor Premise ($E$)**: But, no chisels are hammers. $\rightarrow S - M$[cite: 1, 11, 58]
      - **Conclusion ($E$)**: $\therefore$ No chisels are tools. $\rightarrow S - P$[cite: 1, 11, 58]
      - **Symbolic Diagnosis**: "Tools" is the predicate of an affirmative premise ($p$), covering only *some* tools. In the negative conclusion, "tools" becomes universal ($P$), excluding chisels from *100% of all tools* ($p \rightarrow P$)[cite: 1, 11, 58].
    - ❌ *Invalid Example 2*:
      - **Major Premise ($A$)**: All dogs are animals. $\rightarrow M + p$[cite: 60]
      - **Minor Premise ($E$)**: No cats are dogs. $\rightarrow S - M$[cite: 60]
      - **Conclusion ($E$)**: $\therefore$ Therefore, no cats are animals. $\rightarrow S - P$ (Illicit expansion: $p \rightarrow P$)[cite: 60].
  - ⤷ **Associated Fallacy 2B**: **Fallacy of Illicit Minor Term**[cite: 1, 6, 10, 61]
    - ↳ **Mechanism**: The Minor Term is particular ($s$) in the Minor Premise, but illegally expands into a universal ($S$) in the Conclusion ($s \rightarrow S$)[cite: 1, 10, 61, 62].
    - ❌ *Invalid Example*:
      - **Major Premise ($A$)**: All whales are mammals. $\rightarrow P + m$[cite: 62]
      - **Minor Premise ($A$)**: All mammals are animals. $\rightarrow M + s$[cite: 62]
      - **Conclusion ($A$)**: $\therefore$ Therefore, all animals are whales. $\rightarrow S + p$[cite: 62]
      - **Symbolic Diagnosis**: "Animals" is the predicate of an affirmative minor premise ($s$), referring only to *some* animals. Yet the conclusion makes a claim about *all* animals ($S$). Since $s \rightarrow S$, it commits **Illicit Minor**[cite: 62].

> [!tip] 💡 Intuition: The Bank Overdraft Rule
> The conclusion is the **effect**, and the premises are the **cause**[cite: 1, 11, 58]. In logic as in banking, you cannot withdraw $£1,000,000$ (a Universal claim about *all* members) if you only deposited $£10$ (a Particular claim about *some* members)[cite: 1]. The effect can never be bigger than its cause[cite: 1, 11, 58].

> [!warning] ⚠️ Exam Trap & Slide Erratum (Slide 35 vs. Slide 36)
> On Slide 35 of your second lecture deck, the slide titled *Fallacy of Illicit Major Term* accidentally shows: *"All persons have dignity ($P+m$); All Filipinos have dignity ($S+m$); $\therefore$ All Filipinos are persons ($S+p$)"*[cite: 59]. Look closely at the symbols: $p$ did **not** become $P$ ($p$ is lowercase in both the premise and conclusion!), and Slide 37 actually marks this exact syllogism with a green checkmark for Minor Term quantity[cite: 59, 61]! That specific syllogism actually violates **Rule 4 (Undistributed Middle: $m, m$)**, as Slide 45 later confirms[cite: 69]. Always trust the **symbolic trace** ($p \rightarrow P$ for Illicit Major; $s \rightarrow S$ for Illicit Minor) over a mislabelled slide header!

---

#### 🛡️ Rule 3: The Middle Term ($M$) Should Not Occur in the Conclusion[cite: 1, 5, 12, 64]
- **Formal Requirement**: The middle term exists exclusively to mediate between $S$ and $P$ in the premises; once the connection is forged, $M$ must vanish from the conclusion[cite: 1, 12, 64, 65].
  - ⤷ **Associated Fallacy**: **Fallacy of Misplaced Middle Term**[cite: 1, 6, 12, 64]
  - ↳ **Mechanism**: Dragging the middle term ($M$) down into the third line instead of uniting $S$ and $P$ directly[cite: 1, 12, 64, 66].
  - ❌ *Invalid Example 1*:
    - **Major Premise**: A steward is a flight attendant. $\rightarrow M + p$[cite: 1, 12, 64]
    - **Minor Premise**: But, a steward is a male. $\rightarrow M + s$[cite: 1, 12, 64]
    - **Conclusion**: $\therefore$ A steward is a male flight attendant. $\rightarrow M + sp$[cite: 1, 12, 64]
    - **Defect**: "Steward" ($M$) intrudes into the conclusion[cite: 1, 12, 64].
  - ❌ *Invalid Example 2*:
    - **Major Premise**: All persons have dignity.[cite: 66]
    - **Minor Premise**: All Filipinos have dignity.[cite: 66]
    - **Conclusion**: $\therefore$ Therefore, all Filipinos are people with dignity. (Defect: "dignity" [$M$] was carried into the conclusion instead of concluding *"All Filipinos are persons"*)[cite: 65, 66].

> [!tip] 💡 Intuition: The Third-Wheel Chaperone
> Think of $M$ as a matchmaker who introduces $S$ and $P$ on a blind date[cite: 1]. Once $S$ and $P$ get married in the conclusion, the matchmaker is supposed to go home[cite: 1]. If the matchmaker moves into their honeymoon suite ($M$ in the conclusion), the marriage is invalid[cite: 1, 12, 65].

---

#### 🛡️ Rule 4: The Middle Term ($M$) Must Be Distributed Universally at Least Once in the Premises[cite: 1, 5, 13, 67]
- **Formal Requirement**: ==At least one premise must feature a capital **$M$**== (either one $M$ and one $m$, or two capital $M$s)[cite: 1, 13, 67, 68]. If both premises contain a lowercase $m$, the syllogism collapses[cite: 1, 14, 69, 70].
  - ⤷ **Associated Fallacy**: **Fallacy of Undistributed Middle Term**[cite: 1, 6, 13, 67]
  - ↳ **Mechanism**: *What is true of the whole is true of its parts*, but if neither premise speaks of the **whole** middle class ($M$), then $S$ and $P$ may occupy two completely separate corners of the middle class without ever touching each other[cite: 1, 13, 67].
  - ❌ *Invalid Example 1 (Figure 2 — Two Affirmative Premises)*:
    - **Major Premise ($A$)**: All stewardesses are females. $\rightarrow P + m$[cite: 1, 14, 70]
    - **Minor Premise ($A$)**: But, all mothers are females. $\rightarrow S + m$[cite: 1, 14, 70]
    - **Conclusion ($A$)**: $\therefore$ All mothers are stewardesses. $\rightarrow S + p$[cite: 1, 14, 70]
    - **Symbolic Diagnosis**: Because both premises are affirmative and "females" sits in the predicate spot both times, both occurrences are particular ($m$ and $m$)[cite: 1, 14, 70].
  - ❌ *Invalid Example 2*:
    - **Major Premise ($A$)**: All apples are fruits. $\rightarrow P + m$[cite: 14, 70]
    - **Minor Premise ($I$)**: But, some fruits are mangoes. $\rightarrow m + s$[cite: 14, 70]
    - **Conclusion ($I$)**: $\therefore$ Some mangoes are apples. $\rightarrow s + p$ (Defect: $m$ and $m$—never distributed!)[cite: 14, 70].
  - ❌ *Invalid Example 3*:
    - **Major Premise ($A$)**: All persons have dignity. $\rightarrow P + m$[cite: 68, 69]
    - **Minor Premise ($A$)**: All Filipinos have dignity. $\rightarrow S + m$[cite: 68, 69]
    - **Conclusion ($A$)**: $\therefore$ Therefore, all Filipinos are persons. $\rightarrow S + p$ (Defect: $m$ and $m$)[cite: 68, 69].

> [!tip] 💡 Intuition: Two Strangers in a Massive Country
> Suppose I tell you: *"All Tokyo residents live in Asia ($P+m$)"* and *"All Filipinos live in Asia ($S+m$)."* Does that make all Filipinos residents of Tokyo? Of course not! Because "Asia" ($m$) is so massive and neither sentence covered **100% of Asia ($M$)**, Tokyo residents and Filipinos can sit in two totally separate islands of Asia without overlapping at all[cite: 1].

---

### 📜 B. Rules Governing the Propositions (Rules 5 to 8)

#### 🛡️ Rule 5: Two Affirmative Premises Cannot Give a Negative Conclusion[cite: 1, 5, 15, 73]
- **Formal Requirement**: If both premises are affirmative ($+$ and $+$), the conclusion must likewise be affirmative ($+$)[cite: 1, 15, 73, 76].
  - ⤷ **Associated Fallacy**: **Fallacy of a Negative Conclusion Drawn from Affirmative Premises**[cite: 1, 6, 15, 72, 73]
  - ↳ **Mechanism**: A negative conclusion asserts separation/exclusion between $S$ and $P$, which expresses something completely absent from premises that only assert inclusion/union[cite: 1, 15, 73].
  - ❌ *Invalid Example 1*:
    - **Major Premise ($A$)**: All stones are hard. ($+$)[cite: 1, 16, 74, 75]
    - **Minor Premise ($I$)**: Some diamonds are stones. ($+$)[cite: 1, 16, 74, 75]
    - **Conclusion ($O$)**: $\therefore$ Some diamonds are not hard. ($-$)[cite: 1, 16, 74, 75]
  - ❌ *Invalid Example 2*:
    - **Major Premise ($A$)**: All fish have fins. $\rightarrow M + p$[cite: 77]
    - **Minor Premise ($A$)**: All goldfish are fish. $\rightarrow S + m$[cite: 77]
    - **Conclusion ($E$)**: $\therefore$ Therefore, all goldfish do not have fins. $\rightarrow S - P$ (Valid conclusion is $+$: *"Therefore, all goldfish have fins"* [$S+p$])[cite: 76, 77].

---

#### 🛡️ Rule 6: From Two Negative Premises, Nothing Follows[cite: 1, 5, 17, 78]
- **Formal Requirement**: At least one premise must be affirmative ($+$)[cite: 1]. Two negative premises ($-$ and $-$) cannot yield any valid deductive conclusion whatsoever[cite: 1, 17, 78].
  - ⤷ **Associated Fallacy**: **Fallacy of Exclusive / Negative Premises**[cite: 1, 6, 17, 72, 78]
  - ↳ **Mechanism**: If $P$ is excluded from $M$, and $S$ is also excluded from $M$, no relationship can be established between $S$ and $P$—we have no idea whether $S$ and $P$ overlap outside of $M$ or sit miles apart[cite: 1, 17, 78].
  - ❌ *Invalid Example 1*:
    - **Major Premise ($E$)**: A chair is not a table. ($-$)[cite: 1, 18, 79, 80]
    - **Minor Premise ($E$)**: But, a table is not a pen. ($-$)[cite: 1, 18, 79, 80]
    - **Conclusion ($E$)**: $\therefore$ A pen is not a chair. ($-$)[cite: 1, 18, 79, 80]
  - ❌ *Invalid Example 2*:
    - **Major Premise ($E$)**: No reptiles are mammals. $\rightarrow M - P$[cite: 81]
    - **Minor Premise ($E$)**: No dogs are reptiles. $\rightarrow S - M$[cite: 81]
    - **Conclusion ($E$)**: $\therefore$ Therefore, no dogs are mammals. $\rightarrow S - P$ (Notice how factually absurd the conclusion is: dogs *are* mammals! Because both premises were negative, the logic broke down completely)[cite: 81].

---

#### 🛡️ Rule 7: From Two Particular Premises, Nothing Follows[cite: 1, 5, 19, 82]
- **Formal Requirement**: ==At least one premise must be Universal ($A$ or $E$)==[cite: 1, 88, 89]. Two particular premises ($I$ and $I$, $O$ and $O$, or $I$ and $O$) can never produce a valid conclusion[cite: 1, 19, 20, 82, 83].
  - ⤷ **Associated Fallacy**: **Fallacy of Two Particular Premises**[cite: 1, 6, 19, 72, 82]
  - ↳ **The 3 Proof Cases (Why It Always Fails)**:
    - **Case $\alpha$ (Both Affirmative: $I + I$)**: Every single subject and predicate is particular ($m, p, s$), leaving zero capital letters and automatically violating **Rule 4 (Undistributed Middle)**[cite: 1, 19, 82, 89].
    - **Case $b$ (Both Negative: $O + O$)**: Automatically violates **Rule 6 (Two Negative Premises)**[cite: 1, 20, 83].
    - **Case $c$ (One Affirmative, One Negative: $I + O$ or $O + I$)**: Only one term is universal in the premises (the predicate of the negative $O$ premise)[cite: 1]. If that capital letter is given to $M$ (to obey Rule 4), then $P$ remains lowercase ($p$) in the premise—yet the negative conclusion forces a capital $P$, triggering **Illicit Major**[cite: 1, 20, 21, 83, 87]!
  - ❌ *Invalid Example 1 ($I$ and $O$)*:
    - **Major Premise ($I$)**: Some cats are pets. $\rightarrow m + p$[cite: 1, 21, 84, 87]
    - **Minor Premise ($O$)**: But, some pythons are not cats. $\rightarrow s - M$[cite: 1, 21, 84, 87]
    - **Conclusion ($O$)**: $\therefore$ Some pythons are not pets. $\rightarrow s - P$ (Commits **Two Particular Premises** AND **Illicit Major** [$p \rightarrow P$])[cite: 1, 21, 84, 87].
  - ❌ *Invalid Example 2 ($O$ and $I$)*:
    - **Major Premise ($O$)**: Some horseback riders are not males. $\rightarrow m - P$[cite: 21, 86, 87]
    - **Minor Premise ($I$)**: But, some drivers are horseback riders. $\rightarrow s + m$[cite: 21, 86, 87]
    - **Conclusion ($O$)**: $\therefore$ Some drivers are not males. $\rightarrow s - P$ (Commits **Two Particular Premises** AND **Undistributed Middle** [$m, m$])[cite: 21, 86, 87].
  - ❌ *Invalid Example 3 ($I$ and $I$)*:
    - **Major Premise ($I$)**: Some reptiles are mammals. $\rightarrow m + p$[cite: 88, 89]
    - **Minor Premise ($I$)**: Some dogs are reptiles. $\rightarrow s + m$[cite: 88, 89]
    - **Conclusion ($I$)**: $\therefore$ Therefore, some dogs are mammals. $\rightarrow s + p$ (Commits **Two Particular Premises** AND **Undistributed Middle** [$m, m$])[cite: 88, 89].

---

#### 🛡️ Rule 8: The Conclusion Always Follows the Weaker Premise[cite: 1, 5, 22, 90]
- **Formal Requirement**: An argument is only as strong as its weakest link[cite: 1]:
  1. **Weakness in Quantity**: ==Particular ($I, O$) is weaker than Universal ($A, E$)==[cite: 1]. If one premise is Universal and the other is Particular, the conclusion **must be Particular**[cite: 1, 22, 90].
  2. **Weakness in Quality**: ==Negative ($-$) is weaker than Affirmative ($+$)==[cite: 1]. If one premise is Affirmative and the other is Negative, the conclusion **must be Negative**[cite: 1, 96, 98].
  - ⤷ **Associated Fallacy 8A (Quantity Defect)**: **Fallacy of the Universal Conclusion Drawn from a Particular Premise**[cite: 1, 6, 22, 72, 90, 93]
    - ❌ *Invalid Example 1*:
      - **Major Premise ($A$)**: All mongoloids are mentally retarded. $\rightarrow M + p$[cite: 1, 23, 91, 92] *(Note: Historical terminology from source slides)*
      - **Minor Premise ($I$)**: But, some mongoloids are special children. $\rightarrow m + s$[cite: 1, 23, 91, 92]
      - **Conclusion ($A$)**: $\therefore$ All special children are mentally retarded. $\rightarrow S + p$ (Violates **Rule 8** by drawing a Universal $A$ conclusion from a Particular $I$ premise, AND violates **Rule 2: Illicit Minor** [$s \rightarrow S$])[cite: 1, 23, 91, 92].
    - ❌ *Invalid Example 2*:
      - **Major Premise ($A$)**: All cats are marsupials. $\rightarrow M + p$[cite: 93, 94]
      - **Minor Premise ($I$)**: Some mammals are cats. $\rightarrow s + m$[cite: 93, 94]
      - **Conclusion ($A$)**: $\therefore$ Therefore, all mammals are marsupials. $\rightarrow S + p$ (Invalid: Universal conclusion from a particular premise! Valid conclusion is $I$: *"Therefore, some mammals are marsupials"* [$s+p$])[cite: 93, 94, 95].
  - ⤷ **Associated Fallacy 8B (Quality Defect)**: **Fallacy of an Affirmative Conclusion Drawn from a Negative Premise**[cite: 72, 96, 97]
    - ❌ *Invalid Example*:
      - **Major Premise ($E$)**: All cats are NOT marsupials. $\rightarrow M - P$[cite: 96, 97]
      - **Minor Premise ($I$)**: Some mammals are cats. $\rightarrow s + m$[cite: 96, 97]
      - **Conclusion ($I$)**: $\therefore$ Therefore, some mammals are marsupials. $\rightarrow s + p$ (Invalid: Affirmative $+$ conclusion drawn from a negative $-$ premise! Valid conclusion is $O$: *"Therefore, some mammals are not marsupials"* [$s-P$])[cite: 96, 97, 98].

---

### 🧪 C. Solved Lecture Drills & Exercises (100% Coverage)

> [!example] 🧪 Step-by-Step Derivation: Slide 32 Symbolic Translation Exercise
> Below are the exact symbolic translations for the 7 practice propositions on Slide 32 of `M3_L8-9_Rules... 2.pdf`[cite: 56]:
> 1. **"Not all foods are expensive."** $\rightarrow$ Proposition $O$ ("Not all" = Particular negative) $\rightarrow$ **$s - P$**[cite: 1, 47, 56]
> 2. **"No logicians are irrational."** $\rightarrow$ Proposition $E$ (Universal negative) $\rightarrow$ **$S - P$**[cite: 1, 47, 56]
> 3. **"All artists are creative."** $\rightarrow$ Proposition $A$ (Universal affirmative) $\rightarrow$ **$S + p$**[cite: 1, 47, 56]
> 4. **"Some butterflies are big."** $\rightarrow$ Proposition $I$ (Particular affirmative) $\rightarrow$ **$s + p$**[cite: 1, 47, 56]
> 5. **"Some works are not easy."** $\rightarrow$ Proposition $O$ (Particular negative) $\rightarrow$ **$s - P$**[cite: 1, 47, 56]
> 6. **"All tables are not yet clean."** $\rightarrow$ Proposition $E$ ("All... not" in your curriculum's module template = Universal negative) $\rightarrow$ **$S - P$**[cite: 1, 47, 56]
> 7. **"Some actors are good."** $\rightarrow$ Proposition $I$ (Particular affirmative) $\rightarrow$ **$s + p$**[cite: 1, 47, 56]

> [!example] 🧪 Step-by-Step Derivation: End-of-Deck Formal Fallacy Drills
> These four drills appear at the end of both PDF decks (`Slides 26–29` and `Slides 71–74`)[cite: 24, 25, 99, 100]. Here are their complete symbolic traces and fallacy verdicts:
> 
> - **Drill 1**:
>   - **Syllogism**: *"Rape is immoral ($P + m$); But, cheating is immoral ($S + m$); $\therefore$ Cheating is rape ($S + p$)."*[cite: 26, 101]
>   - **Symbolic Trace**: Both premises are Affirmative ($+$) in Figure 2 ($PRE\text{--}PRE$), making the middle term ("immoral") particular in both premises ($m$ and $m$).
>   - ✅ **Verdict**: **Fallacy of Undistributed Middle Term** (Violates Rule 4)[cite: 6, 13, 67].
> 
> - **Drill 2**:
>   - **Syllogism**: *"A martyr dies for a noble cause ($M + p$); But, some wives are martyrs ($s + m$); $\therefore$ Some wives do not die for a noble cause ($s - P$)."*[cite: 27, 102]
>   - **Symbolic Trace**: Both premises are Affirmative ($+$ and $+$), yet the conclusion is Negative ($-$). Furthermore, "dies for a noble cause" is particular ($p$) in the major premise, but becomes universal ($P$) in the negative conclusion ($p \rightarrow P$).
>   - ✅ **Verdict**: Primarily **Fallacy of a Negative Conclusion Drawn from Affirmative Premises** (Violates Rule 5)[cite: 6, 15, 73], and secondarily **Fallacy of Illicit Major Term** (Violates Rule 2: $p \rightarrow P$)[cite: 6, 10, 45]. *(Note: Also plays on conversational equivocation of "martyr" as a literal martyr vs. a long-suffering spouse, though structurally it is the classic Rule 5 drill!)*
> 
> - **Drill 3**:
>   - **Syllogism**: *"Waiters wait; But, true love waits; $\therefore$ True love is a waiter."*[cite: 28, 103]
>   - **Symbolic Trace**: "Wait" in the major premise means *serving tables at a restaurant*, whereas "waits" in the minor premise means *enduring patiently over time*. (Structurally, if read literally as $P+m, S+m \therefore S+p$, it also has an undistributed middle).
>   - ✅ **Verdict**: **Fallacy of Equivocation** (Violates Rule 1)[cite: 6, 8, 43] (and structurally **Fallacy of Undistributed Middle Term** under Rule 4)[cite: 6, 13, 67].
> 
> - **Drill 4**:
>   - **Syllogism**: *"Horror stories send chilling effects ($M + p$); But, murder stories are horror stories ($S + m$); $\therefore$ Murder stories don't send chilling effects ($S - P$)."*[cite: 29, 104]
>   - **Symbolic Trace**: Two Affirmative premises ($+$ and $+$) illegally yield a Negative conclusion ($-$), which also forces $p \rightarrow P$.
>   - ✅ **Verdict**: **Fallacy of a Negative Conclusion Drawn from Affirmative Premises** (Violates Rule 5)[cite: 6, 15, 73] (plus **Fallacy of Illicit Major Term** under Rule 2)[cite: 6, 10, 45].

---

## 🗣️ Part III: Informal Fallacies

Unlike Formal Fallacies—which can be diagnosed purely by inspecting symbolic letters ($M, P, S$)—**Informal Fallacies** are errors of **content, relevance, presumption, or ambiguity** embedded in natural human language. Even if the grammar looks valid, the premises fail to provide rational support for the conclusion.

### 🎯 Category A: Fallacies of Relevance
*Premises are logically irrelevant to the truth of the conclusion; they rely on emotional, psychological, or personal diversion to manufacture agreement.*

- **1. *Argumentum ad Baculum* (Appeal to Force / Stick)**
  - ⤷ **Definition**: Attempting to win an argument or force the acceptance of a conclusion by threatening physical force, coercion, intimidation, or social/financial retaliation instead of providing evidence.
  - ↳ **Latin Literal**: *"Appeal to the stick/club."*
  - → *Example*: *"You must agree that the new company tax policy is fair; otherwise, I will terminate your contract tomorrow morning."*

- **2. *Argumentum ad Misericordiam* (Appeal to Pity / Mercy)**
  - ⤷ **Definition**: Manipulating the listener's compassion, guilt, or sympathy to get a conclusion accepted, while ignoring the objective facts or merits of the case.
  - ↳ **Latin Literal**: *"Appeal to pity/mercy."*
  - → *Example*: *"Professor, you must pass my final logic exam—if I fail this course, my parents will take away my allowance and I won't be able to sleep for weeks!"*

- **3. *Argumentum ad Hominem* (Attack Against the Person)**
  - ⤷ **Definition**: Rejecting a claim by attacking the character, circumstances, or hypocrisy of the person making the argument, rather than refuting the logic of the argument itself.
  - ↳ **Sub-Type 3A: *Ad Hominem* (Abusive / Direct)**: Directly insulting the opponent's intelligence, morals, or appearance.
    - → *Example*: *"You cannot trust Dr. Smith's mathematical proof because he is an unpleasant, poorly dressed man."*
  - ↳ **Sub-Type 3B: *Ad Hominem* (Circumstantial)**: Dismissing someone's argument solely because of their personal situation, job, nationality, or self-interest (*"Of course you'd say that because you are X"*).
    - → *Example*: *"Father Rodriguez argues that charity is a moral duty, but we can ignore him—he's a priest, so he's paid to say that!"*
  - ↳ **Sub-Type 3C: *Ad Hominem Tu Quoque* ("You Too!" / Appeal to Hypocrisy)**: Deflecting criticism or rejecting a moral/factual claim by pointing out that the speaker fails to practice what they preach.
    - → *Example*: *"My doctor told me that smoking two packs a day will ruin my lungs, but he smokes cigarettes himself—so smoking clearly isn't dangerous!"*

> [!warning] ⚠️ Exam Trap: Why *Tu Quoque* Is a Fallacy
> Even if a doctor is a massive hypocrite who smokes three packs a day, **hypocrisy does not change biological reality**. A person's actions do not make their factual statement false!

- **4. Fallacy of Accident (*Dicto Simpliciter*)**
  - ⤷ **Definition**: Applying a general rule rigidly to a specific, exceptional case where "accidental" (special) circumstances make the general rule inapplicable.
  - ↳ **Core Mechanism**: **General Rule $\rightarrow$ Forced onto an Exceptional Case** (ignoring valid exceptions).
  - → *Example*: *"Cutting people with a sharp knife is a crime. Surgeons cut people with sharp knives. Therefore, surgeons are criminals."*

- **5. *Ignoratio Elenchi* (Irrelevant Conclusion / Missing the Point)**
  - ⤷ **Definition**: Presenting premises that actually support *one* conclusion, but mistakenly claiming that they prove a completely *different* (often much more specific or unrelated) conclusion.
  - ↳ **Latin Literal**: *"Ignorance of the refutation."*
  - → *Example*: *"Murder is a horrific crime that destroys families and tears society apart. Therefore, the defendant standing in this courtroom is guilty of murder."* (Proving that *murder is bad* does not prove *who committed it*!)

- **6. Red Herring**
  - ⤷ **Definition**: Deliberately introducing a tangential, emotionally charged, or flashy topic mid-argument to distract the audience from the original issue under debate.
  - ↳ **Origin Story**: Named after the practice of dragging a pungent smoked fish across a trail to throw hunting hounds off the scent.
  - → *Example*: *"You ask why my department lost ₱500,000 in the budget audit, but have you thought about how hard our employees work and how much they love their families?"*

> [!tip] 💡 Intuition: *Ignoratio Elenchi* vs. Red Herring
> Students constantly confuse these two! Here is the razor-sharp difference:
> - **Red Herring** changes the **path** mid-conversation to distract you (*changing the subject to avoid answering*).
> - **Ignoratio Elenchi** stays on the topic during the premises, but jumps onto the **wrong train platform at the very end** (*the premises prove Point A, but the speaker concludes Point B*).

- **7. Genetic Fallacy**
  - ⤷ **Definition**: Judging a claim, idea, or institution as automatically true or false based solely on its **historical origin, source, or how it began**, rather than evaluating its current evidence or merits.
  - → *Example*: *"The Volkswagen Beetle was originally commissioned in 1930s Germany under a dictator; therefore, driving a Volkswagen today makes you an evil person."* (Or: *"You only believe the Earth is round because your primary school teacher told you so!"*)

---

### 🧩 Category B: Other Informal Fallacies (Classroom Slide Taxonomy)
*Exact breakdown of the 9 Informal Fallacies and Filipino classroom mnemonics from your lecture slide[cite: 106].*

- **1. *Argumentum ad Verecundiam* (Appeal to Misplaced Authority)**[cite: 106]
  - ⤷ **Definition**: Insisting that a claim is true simply because a famous person, celebrity, or authority figure said so—especially when that person has **zero expertise** in the relevant field[cite: 106]. *(Note: Spelled `Vericundiam` on your lecture slide; keep both spellings in mind for exams!)*[cite: 106]
  - → *Example*: *"This specific cryptocurrency will definitely triple in value next month because my favourite basketball player endorsed it in a commercial."*

- **2. *Argumentum ad Ignorantiam* (Appeal to Ignorance)**[cite: 106]
  - ⤷ **Definition**: Claiming that a statement is **True** simply because nobody has proven it **False**, or claiming it is **False** simply because nobody has proven it **True**[cite: 106].
  - ↳ **Core Rule**: ==Absence of evidence is NOT evidence of absence==. Shifting the burden of proof onto the sceptic is fallacious.
  - → *Example*: *"No scientist has ever proven that telepathic aliens are NOT hiding inside Mount Makiling; therefore, telepathic aliens definitely live inside Mount Makiling!"*

- **3. Hasty Generalization (*"The Part Constitutes the Whole" / Converse Accident*)**[cite: 106]
  - ⤷ **Definition**: Drawing a sweeping universal conclusion about an entire population based on a sample size that is far too small, unrepresentative, or biased[cite: 106].
  - ↳ **Core Mechanism**: **Tiny/Exceptional Part $\rightarrow$ Applied to the Entire Whole** (The exact mirror opposite of the *Fallacy of Accident*)[cite: 106].
  - → *Example*: *"My first boyfriend from Laguna cheated on me, and my sister's boyfriend from Laguna was rude. Therefore, all men from Laguna are terrible partners!"*

- **4. *Post Hoc, Ergo Propter Hoc* (*"After This, Therefore Because of This"* / False Cause)**[cite: 106]
  - ⤷ **Definition**: Assuming that just because Event $A$ happened *before* Event $B$ in time, Event $A$ must have *caused* Event $B$[cite: 106]. Confusing chronological sequence with causal connection (superstition).
  - → *Example*: *"I wore my lucky mismatched socks to the university entrance exam and scored in the top tier; therefore, my mismatched socks caused my high score!"*

- **5. Slippery Slope (*"Si O.A. Fallacy"*)**[cite: 106]
  - ⤷ **Definition**: Asserting without sufficient evidence that allowing one small, harmless action today will inevitably trigger a catastrophic, unstoppable domino effect ending in an extreme disaster[cite: 106].
  - ↳ **Why it's called the *"Si O.A. (Over-Acting) Fallacy"***: Because the speaker dramatically overreacts, jumping from a minor step to an apocalyptic conclusion without proving why the middle steps must happen[cite: 106].
  - → *Example*: *"If you let me miss just one gym session today, I'll lose all my discipline, fail my computer science degree, become unemployable, and end up living in a cardboard box!"*

- **6. Weak Analogy (*"Parang ganito lang 'yan Fallacy"* / False Analogy)**[cite: 106]
  - ⤷ **Definition**: Comparing two things that share a few superficial similarities, and concluding that they must also be identical in a deeper function—even though they have massive, fatal differences[cite: 106].
  - ↳ **Why it's called the *"Parang ganito lang 'yan Fallacy"***: In casual debate, speakers love saying *"Parang ganito lang 'yan..."* ("It's just like this...") to equate two wildly incomparable things[cite: 106].
  - → *Example*: *"Running a country's economy is just like running a sari-sari store ('Parang ganito lang 'yan'): if a store never borrows money, a government should never issue bonds."*

- **7. *Petitio Principii* (Begging the Question / Circular Reasoning)**[cite: 106]
  - ⤷ **Definition**: Assuming the truth of the very conclusion you are trying to prove right inside your premise (often by disguising it in synonyms or circular logic)[cite: 106].
  - ↳ **Intuition**: If your conclusion is a heavy suitcase, your premises are supposed to carry it. In *Petitio Principii*, you put the suitcase inside itself and claim it is carrying its own weight!
  - → *Example*: *"Free trade is always good for a nation's economy because unrestricted commerce brings economic benefits to the country."* (Notice how the premise and conclusion say the exact same thing in different words!)

- **8. Complex Question (*"Wala kang lusot Fallacy"* / Loaded Question)**[cite: 106]
  - ⤷ **Definition**: Asking a single question that secretly smuggles in an unproven, incriminating assumption—so that whether the victim answers *"Yes"* or *"No"*, they are trapped into admitting guilt[cite: 106].
  - ↳ **Why it's called the *"Wala kang lusot (No way out) Fallacy"***: Because it is a rhetorical trap with no innocent answer unless you break the question apart[cite: 106].
  - → *Example*: *"Have you stopped cheating on your logic quizzes—yes or no?"* (If you say *Yes*, you admit you used to cheat; if you say *No*, you admit you are still cheating! *Wala kang lusot!*)

- **9. False Dilemma (*"No Choice Fallacy"* / Black-and-White Fallacy / Either-Or)**[cite: 106]
  - ⤷ **Definition**: Artificially restricting a complex situation to only **two extreme options** (*"Either $A$ or $B$"*) when in reality there are plenty of valid middle-ground alternatives ($C, D, E$)[cite: 106].
  - ↳ **Why it's called the *"No Choice Fallacy"***: Because the speaker bullies you into picking their preferred option by pretending the only other choice is total ruin[cite: 106].
  - → *Example*: *"Either you spend 16 hours a day coding in Python, or you will fail as a programmer and never get a job."* (Ignores balanced, consistent study schedules!)

---

## 📊 Part IV: Master Comparison Matrix (Exam Quick-Skim)

| Fallacy Name | Category | Core Trigger / Diagnostic Clue | Classroom Alias / Mnemonic |
| :--- | :---: | :--- | :--- |
| **Equivocation**[cite: 6, 8] | Formal (Rule 1)[cite: 5, 8] | Middle term ($M$) shifts definition between premises[cite: 1, 8] | Four Terms (*Quaternio Terminorum*)[cite: 1] |
| **Illicit Major**[cite: 6, 10] | Formal (Rule 2)[cite: 5, 10] | $p$ in Major Premise $\rightarrow P$ in Conclusion[cite: 1, 10, 11] | Overdraft on the Predicate ($p \rightarrow P$)[cite: 1, 11] |
| **Illicit Minor**[cite: 6, 10] | Formal (Rule 2)[cite: 5, 10] | $s$ in Minor Premise $\rightarrow S$ in Conclusion[cite: 1, 10, 62] | Overdraft on the Subject ($s \rightarrow S$)[cite: 1, 62] |
| **Misplaced Middle**[cite: 6, 12] | Formal (Rule 3)[cite: 5, 12] | Middle term ($M$) appears in the Conclusion[cite: 1, 12, 65] | Matchmaker in the Honeymoon Suite[cite: 1] |
| **Undistributed Middle**[cite: 6, 13] | Formal (Rule 4)[cite: 5, 13] | Both premises have lowercase $m$ ($m$ and $m$)[cite: 1, 14, 69] | Never covers the whole Middle class[cite: 1, 13] |
| **Neg. Conclusion from Aff.**[cite: 6, 15] | Formal (Rule 5)[cite: 5, 15] | Premise $1 (+) + \text{Premise } 2 (+) \rightarrow \text{Conclusion } (-)$[cite: 1, 16, 77] | Separation drawn from pure inclusion[cite: 1] |
| **Exclusive Premises**[cite: 6, 17] | Formal (Rule 6)[cite: 5, 17] | Premise $1 (-) + \text{Premise } 2 (-)$[cite: 1, 18, 81] | Two Negatives = Zero connection[cite: 1, 17] |
| **Two Particular Premises**[cite: 6, 19] | Formal (Rule 7)[cite: 5, 19] | Both premises are $I$ or $O$ ($I/I, O/O, I/O$)[cite: 1, 19, 20, 88] | Missing at least one $A$ or $E$ premise[cite: 88, 89] |
| **Univ. from Particular**[cite: 6, 22] | Formal (Rule 8A)[cite: 5, 22] | One $I/O$ premise $\rightarrow A/E$ Conclusion[cite: 1, 22, 93] | Violates Quantity weakness[cite: 1, 22] |
| **Aff. from Negative**[cite: 72, 96] | Formal (Rule 8B)[cite: 72, 96] | One $E/O$ premise $\rightarrow A/I$ Conclusion[cite: 96, 97] | Violates Quality weakness[cite: 1, 96] |
| ***Ad Baculum*** | Informal (Relevance) | Threats, force, intimidation, or coercion | Appeal to the Stick / Force |
| ***Ad Misericordiam*** | Informal (Relevance) | Guilt-tripping, sob stories, emotional pity | Appeal to Pity / Mercy |
| ***Ad Hominem*** | Informal (Relevance) | Attacking character, bias (*Circumstantial*), or hypocrisy (*Tu Quoque*) | Attack on the Person / *"You too!"* |
| **Fallacy of Accident** | Informal (Relevance) | General Rule forced onto an exceptional case | Ignoring valid exceptions |
| ***Ignoratio Elenchi*** | Informal (Relevance) | Premises prove Point $A$, speaker concludes Point $B$ | Irrelevant Conclusion / Missing the Point |
| **Red Herring** | Informal (Relevance) | Changing the subject mid-debate to distract | Smoked fish distraction |
| **Genetic Fallacy** | Informal (Relevance) | Judging truth solely by historical origin/source | Origin = Current Truth/Falsity |
| ***Ad Verecundiam***[cite: 106] | Informal (Other)[cite: 106] | Citing a celebrity or non-expert authority[cite: 106] | Appeal to Authority (`Vericundiam`)[cite: 106] |
| ***Ad Ignorantiam***[cite: 106] | Informal (Other)[cite: 106] | *"Not proven false = True"* (or vice versa)[cite: 106] | Appeal to Ignorance[cite: 106] |
| **Hasty Generalization**[cite: 106] | Informal (Other)[cite: 106] | Tiny sample size applied to the entire group[cite: 106] | *"The Part Constitutes the Whole"*[cite: 106] |
| ***Post Hoc, Ergo Propter Hoc***[cite: 106] | Informal (Other)[cite: 106] | $A$ happened before $B$, so $A$ caused $B$[cite: 106] | *"After This, Therefore Because of This"*[cite: 106] |
| **Slippery Slope**[cite: 106] | Informal (Other)[cite: 106] | Small step $\rightarrow$ Unproven catastrophic chain reaction[cite: 106] | ***Si O.A. Fallacy***[cite: 106] |
| **Weak Analogy**[cite: 106] | Informal (Other)[cite: 106] | Comparing two things with fatal differences[cite: 106] | ***Parang ganito lang 'yan Fallacy***[cite: 106] |
| ***Petitio Principii***[cite: 106] | Informal (Other)[cite: 106] | Conclusion is secretly assumed inside the premise[cite: 106] | Begging the Question / Circular[cite: 106] |
| **Complex Question**[cite: 106] | Informal (Other)[cite: 106] | Trap question smuggling in a guilty assumption[cite: 106] | ***Wala kang lusot Fallacy***[cite: 106] |
| **False Dilemma**[cite: 106] | Informal (Other)[cite: 106] | Forcing 2 extreme choices when middle ground exists[cite: 106] | ***No Choice Fallacy***[cite: 106] |