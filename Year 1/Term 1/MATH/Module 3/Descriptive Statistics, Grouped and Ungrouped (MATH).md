# 🏛️ Module 01: Foundations of Descriptive Statistics (Ungrouped Data)

---

## 📖 Prerequisite Lexicon: From First Principles
- **Data Array**: An ordered collection of raw numerical observations arranged in ascending (or descending) order of magnitude.
- **Parameter**: A measurable numerical characteristic describing an entire **population** (conventionally denoted by Greek letters: $\mu, \sigma, \sigma^2$).
- **Statistic**: A numerical measure computed from a **sample** subset to estimate the true underlying population parameter (conventionally denoted by Latin letters: $\bar{X}, s, s^2$).
- **Central Tendency**: A solitary representative value pinpointing the mathematical centre or typical value of a probability distribution or dataset.
- **Dispersion (Variation)**: The quantitative degree of spread, scatter, or variability of observations around their central anchor.

---

## ⚙️ Measures of Central Tendency

```
              ┌──────────────────────────────────────────────┐
              │         Measures of Central Tendency         │
              └──────────────────────┬───────────────────────┘
         ┌───────────────────┬───────┴───────────┬───────────────────┐
         ▼                   ▼                   ▼                   ▼
   ┌───────────┐       ┌───────────┐       ┌───────────┐       ┌───────────┐
   │ Arithmetic│       │  Median   │       │   Mode    │       │ Midrange  │
   │   Mean    │       │   (MD)    │       │   (Mo)    │       │   (MR)    │
   └───────────┘       └───────────┘       └───────────┘       └───────────┘
```

### 1. Arithmetic Mean
- ⤷ **Definition / Core Rule**: The arithmetic balance point obtained by taking the sum of all observed values divided by the total count of observations.
  - ↳ **Population Mean ($\mu$)**:
    $$\mu = \frac{\sum X}{N} = \frac{X_1 + X_2 + \cdots + X_N}{N}$$
    Where $N$ represents the finite population size.
  - ↳ **Sample Mean ($\bar{X}$)**:
    $$\bar{X} = \frac{\sum X}{n} = \frac{X_1 + X_2 + \cdots + X_n}{n}$$
    Where $n$ represents the sample size.
  - ↳ **Mathematical Property**: The algebraic sum of deviations about the mean is identically zero: $\sum (X - \bar{X}) = 0$.
  - ↳ **Vulnerability**: ==Highly sensitive to extreme outliers==. A single anomalous extreme value distorts the mean, pulling it toward the heavy tail.

> [!bug] 🚨 Lecture Material Discrepancy: Salary Example
> In the source slide entitled *"Population Mean - Example"*, the prompt lists salaries of $\$50\,000, \$20\,000, \$12\,000, \$9\,000,$ and $\$9\,000$, yet the solution line erroneously replicates the kitten age calculation: `(3+8+5+12+14+12)/6 = 9`.
> - **The Correct Calculation**:
>   $$\mu = \frac{50\,000 + 20\,000 + 12\,000 + 9\,000 + 9\,000}{5} = \frac{100\,000}{5} = \$20\,000$$

---

### 2. Weighted Mean
- ⤷ **Definition / Core Rule**: An adjusted mean employed when individual data entries contribute unequally to the final metric, each modulated by an assigned weight factor $w_i$.
  $$\bar{X}_w = \frac{\sum wX}{\sum w} = \frac{w_1 X_1 + w_2 X_2 + \cdots + w_k X_k}{w_1 + w_2 + \cdots + w_k}$$

> [!tip] 💡 Intuition: Centre of Mass
> Think of each score as a point along a uniform plank. The weights are physical loads stacked at those positions. The weighted mean is the exact fulcrum point where the plank balances horizontally without tilting.

> [!example] 🧪 Step-by-Step Derivation: Academic Grade Calculation
> An academic evaluation assigns: Tests ($40\%$), Final Exam ($25\%$), Quizzes ($25\%$), and Homework ($10\%$). The student obtains: Tests = $83$, Final Exam = $75$, Quizzes = $90$, Homework = $100$.
> 1. Formulate weights and observations:
>    $$\sum wX = (0.40)(83) + (0.25)(75) + (0.25)(90) + (0.10)(100)$$
> 2. Compute components:
>    $$\sum wX = 33.20 + 18.75 + 22.50 + 10.00 = 84.45$$
> 3. Normalise by total weight:
>    $$\sum w = 0.40 + 0.25 + 0.25 + 0.10 = 1.00 \implies \bar{X}_w = \frac{84.45}{1.00} = 84.45\%$$

---

### 3. Median ($MD$)
- ⤷ **Definition / Core Rule**: The positional centre dividing an ordered data array into two equal halves ($50\%$ below, $50\%$ above).
  - ↳ **Odd Sample Size ($n$)**: The median is the single central element located at index position $\frac{n + 1}{2}$.
  - ↳ **Even Sample Size ($n$)**: The median is the arithmetic mean of the two central elements at positions $\frac{n}{2}$ and $\frac{n}{2} + 1$:
    $$MD = \frac{X_{(n/2)} + X_{(n/2 + 1)}}{2}$$
  - ↳ **Robustness**: ==Extremely robust against outliers==. Altering the magnitude of the largest extreme value leaves the median completely invariant.

> [!bug] 🚨 Lecture Material Discrepancy: Typographical Glitches
> In the source slide for even-numbered medians, the slide prints:
> $$MD = (20 \div 23)/2 = 21.5$$
> This is a typographical blunder. The operator must be addition, not division: $\frac{20 + 23}{2} = 21.5$. Be wary of this transcription error if encountering slide-derived exam questions.

---

### 4. Mode ($M_o$) and Midrange ($MR$)
- ⤷ **Mode ($M_o$)**: The value occurring with maximal frequency.
  - ↳ **Unimodal**: One unique dominant frequency peak.
  - ↳ **Bimodal**: Exactly two distinct non-adjacent values tied for maximal frequency.
  - ↳ **Multimodal**: Three or more peaks sharing peak frequency.
  - ↳ **No Mode**: Occurs when ==every observed value exhibits equal frequency==.
- ⤷ **Midrange ($MR$)**: The unweighted mean of the absolute extremes in a distribution:
  $$MR = \frac{L + H}{2} = \frac{\text{Lowest Value} + \text{Highest Value}}{2}$$
  - ↳ **Vulnerability**: Extreme sensitivity to outliers; changing either the minimum or maximum completely destabilises the metric.

> [!bug] 🚨 Lecture Material Discrepancy: Midrange Slide Operator
> Slide 24 renders: `MR = (1 ÷ 8)/2 = 4.5`. The division symbol `÷` is another slide misprint. The correct formula is addition: $\frac{1 + 8}{2} = 4.5$.

---

## 📐 Distribution Shapes & Selection Criteria

![[Skewness.png]]

- ⤷ **Positively Skewed (Skewed to the Right)**:
  - The long tail stretches infinitely toward positive values.
  - Extreme values pull the mean rightward:
    $$\mathbf{\text{Mode} < \text{Median} < \text{Mean}}$$
- ⤷ **Symmetrical (Normal / Unimodal Distribution)**:
  - Both halves are mirror reflections across the central ordinate:
    $$\mathbf{\text{Mean} = \text{Median} = \text{Mode}}$$
- ⤷ **Negatively Skewed (Skewed to the Left)**:
  - The long tail stretches toward zero or negative infinity.
  - Extreme small values pull the mean down:
    $$\mathbf{\text{Mean} < \text{Median} < \text{Mode}}$$

### Optimal Measure Selection Matrix
| Variable Level of Measurement | Distributional Profile | Optimal Central Measure | Analytical Rationale |
| :--- | :--- | :--- | :--- |
| **Nominal** | Any | **Mode** | Mathematical operations are impermissible; only counts/frequencies exist. |
| **Ordinal** | Any | **Median** | Quantities indicate rank ordering; intervals between ranks are non-standard. |
| **Interval / Ratio** | Symmetric (Non-skewed) | **Mean** | Exploits all numerical values; optimal mathematical tractability. |
| **Interval / Ratio** | Skewed / Heavy Outliers | **Median** | The mean is distorted by extremes; the median reliably reflects typicality. |

---

## 🍞 Measures of Dispersion (Variation)

- ⤷ **Range ($R$)**:
  $$R = \text{Highest Value} - \text{Lowest Value} = H - L$$
- ⤷ **Population Variance ($\sigma^2$) & Population Standard Deviation ($\sigma$)**:
  $$\sigma^2 = \frac{\sum (X - \mu)^2}{N}, \qquad \sigma = \sqrt{\frac{\sum (X - \mu)^2}{N}}$$
- ⤷ **Sample Variance ($s^2$) & Sample Standard Deviation ($s$)**:
  $$s^2 = \frac{\sum (X - \bar{X})^2}{n - 1}, \qquad s = \sqrt{s^2} = \sqrt{\frac{\sum (X - \bar{X})^2}{n - 1}}$$

> [!tip] 💡 Intuition: Why Divide by $(n - 1)$? (Bessel's Correction)
> Sample points naturally cluster closer to their own sample mean $\bar{X}$ than to the broader population mean $\mu$. Calculating variance around $\bar{X}$ systematically underestimates total population variability. Dividing by $n - 1$ instead of $n$ inflates the fraction, providing an **unbiased estimator** that matches the expected population variance $\mathbb{E}[s^2] = \sigma^2$.

---

## 💍 Measures of Position (Ungrouped Data)

### 1. Standardised Scores ($z$-Scores)
- ⤷ **Definition**: The distance an individual score $X$ lies from the mean, expressed in units of standard deviation.
  - ↳ **Population $z$-Score**: $z = \frac{X - \mu}{\sigma}$
  - ↳ **Sample $z$-Score**: $z = \frac{X - \bar{X}}{s}$
  - ↳ **Sign Interpretation**: $z > 0 \implies$ falls above the mean; $z < 0 \implies$ falls below the mean; $z = 0 \implies$ coincides with the mean.

> [!check] 🔢 Comparative Position Determination
> To ascertain relative standing across distinct tests with differing means and deviations:
> 1. Calculate $z_A = \frac{X_A - \bar{X}_A}{s_A}$ and $z_B = \frac{X_B - \bar{X}_B}{s_B}$.
> 2. The distribution with the larger algebraic $z$-score occupies the superior relative percentile position within its group.

### 2. Percentile Rank for Raw / Ungrouped Data
- ⤷ **Definition**: The percentage of observations in a distribution falling strictly below a specified value $X$.
  $$\text{Percentile} = \left( \frac{\text{Number of values below } X + 0.5}{n} \right) \times 100\%$$
  - ↳ The $+0.5$ correction represents the midpoint of the interval occupied by the observation $X$ itself.

---
---

# ⚙️ Module 02: Analysis of Grouped Frequency Distributions

---

## 📖 Prerequisite Lexicon for Grouped Data
- **Class Limits**: The stated raw integer boundaries of an interval (e.g., $40\text{--}49$).
- **Class Boundaries (True Limits)**: Unbroken continuous dividing lines that eradicate gaps between adjacent discrete classes. Calculated by subtracting $0.5$ from the lower class limit and adding $0.5$ to the upper limit (e.g., $39.5\text{--}49.5$).
- **Classmark / Midpoint ($x$)**: The central balance point of an interval:
  $$x = \frac{\text{Lower Limit} + \text{Upper Limit}}{2} = \frac{\text{Lower Boundary} + \text{Upper Boundary}}{2}$$
- **Class Width ($h$)**: The physical span of a single continuous boundary interval:
  $$h = \text{Upper Class Boundary} - \text{Lower Class Boundary}$$
- **Cumulative Frequency ($<cf$)**: The running sum of frequencies up to and including the current class interval.

---

## 📐 Formulas for Grouped Data Analysis

### 1. Grouped Mean ($\bar{x}$)
$$\bar{x} = \frac{\sum f \cdot x}{n}$$
Where:
- $f = \text{class frequency}$
- $x = \text{class midpoint (classmark)}$
- $n = \sum f = \text{total sample frequency}$

---

### 2. Grouped Median ($M_m$)
$$M_m = l + \left( \frac{\frac{n}{2} - cf}{f_m} \right)h$$
Where:
- $l = \text{lower class boundary of the median class}$
- $n = \text{total sample size}$
- $cf = \text{cumulative frequency of the class immediately preceding the median class}$
- $f_m = \text{frequency of the median class}$
- $h = \text{class width}$

> [!check] 🔢 Locating the Median Class
> Compute the position indicator $L = \frac{n}{2}$. Inspect the ascending cumulative frequency column ($<cf$). The first class where $<cf \ge \frac{n}{2}$ is defined as the **median class**.

---

### 3. Grouped Mode ($M_o$)
$$M_o = l + \left( \frac{f_1 - f_0}{2f_1 - f_0 - f_2} \right)h$$
Where:
- **Modal Class**: The class containing the single greatest raw frequency ($f_1 = \max(f)$).
- $l = \text{lower class boundary of the modal class}$
- $f_1 = \text{frequency of the modal class}$
- $f_0 = \text{frequency of the class directly preceding the modal class}$
- $f_2 = \text{frequency of the class directly succeeding the modal class}$
- $h = \text{class width}$

---

### 4. Grouped Sample Variance ($s^2$) & Standard Deviation ($s$)
- ⤷ **Computational Shortcut Formula**:
  $$s^2 = \frac{\sum f \cdot x^2 - \frac{(\sum f \cdot x)^2}{n}}{n - 1}, \qquad s = \sqrt{s^2}$$
  - $\sum f \cdot x$: Sum of the products of each frequency and its midpoint.
  - $\sum f \cdot x^2$: Sum of the products of each frequency and its *squared* midpoint.

---

### 5. Grouped Quantiles (Percentiles, Deciles, Quartiles)
- ⤷ **Generalised Grouped Percentile Formula ($P_k$)**:
  $$P_k = l + \left( \frac{\frac{k \cdot n}{100} - cf}{f_k} \right)h$$
  Where:
  - $\frac{k \cdot n}{100}$ establishes the target positional rank.
  - $l = \text{lower boundary of the class containing the } k\text{-th percentile}$.
  - $cf = \text{cumulative frequency of the preceding class}$.
  - $f_k = \text{raw frequency of the percentile class}$.
  - $h = \text{class width}$.
- ⤷ **Decile & Quartile Equivalences**:
  - Quartiles: $Q_1 = P_{25}$, $Q_2 = P_{50} = M_m$, $Q_3 = P_{75}$.
  - Deciles: $D_1 = P_{10}$, $D_5 = P_{50}$, $D_7 = P_{70}$.

---

## 🧪 Comprehensive Master Walkthrough: The 38-Student Grade Cohort

Consider the primary dataset provided across the lecture slides:

| Class Interval (Grade) | Class Boundary | Frequency ($f$) | Midpoint ($x$) | $f \cdot x$ | $x^2$ | $f \cdot x^2$ | $<cf$ |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| $40\text{--}49$ | $39.5\text{--}49.5$ | $3$ | $44.5$ | $133.5$ | $1980.25$ | $5940.75$ | $3$ |
| $50\text{--}59$ | $49.5\text{--}59.5$ | $5$ | $54.5$ | $272.5$ | $2970.25$ | $14851.25$ | $8$ |
| $60\text{--}69$ | $59.5\text{--}69.5$ | $6$ | $64.5$ | $387.0$ | $4160.25$ | $24961.50$ | $14$ |
| $70\text{--}79$ | $69.5\text{--}79.5$ | $9$ | $74.5$ | $670.5$ | $5550.25$ | $49952.25$ | $23$ |
| $80\text{--}89$ | $79.5\text{--}89.5$ | $8$ | $84.5$ | $676.0$ | $7140.25$ | $57122.00$ | $31$ |
| $90\text{--}99$ | $89.5\text{--}99.5$ | $7$ | $94.5$ | $661.5$ | $8930.25$ | $62511.75$ | $38$ |
| **Totals** | — | $\mathbf{n = 38}$ | — | $\mathbf{2801.0}$ | — | $\mathbf{215339.5}$ | — |

### Step 1: Compute the Mean
$$\bar{x} = \frac{\sum f x}{n} = \frac{2801}{38} \approx \mathbf{73.71}$$

### Step 2: Compute the Median ($M_m$)
1. **Median Location**: $\frac{n}{2} = \frac{38}{2} = 19$.
2. **Locate Median Class**: $<cf$ reaches $23$ at interval $70\text{--}79$. Hence, the median class is $70\text{--}79$.
3. **Parameters**: $l = 69.5$, $cf = 14$, $f_m = 9$, $h = 10$.
4. **Execution**:
   $$M_m = 69.5 + \left( \frac{19 - 14}{9} \right)(10) = 69.5 + \left( \frac{5}{9} \right)(10) = 69.5 + 5.56 = \mathbf{75.06}$$

### Step 3: Compute the Mode ($M_o$)
1. **Locate Modal Class**: Highest frequency is $f = 9$ in class $70\text{--}79$.
2. **Parameters**: $l = 69.5$, $f_1 = 9$, $f_0 = 6$, $f_2 = 8$, $h = 10$.
3. **Execution**:
   $$M_o = 69.5 + \left( \frac{9 - 6}{2(9) - 6 - 8} \right)(10) = 69.5 + \left( \frac{3}{18 - 14} \right)(10) = 69.5 + \left( \frac{3}{4} \right)(10) = 69.5 + 7.5 = \mathbf{77.00}$$

### Step 4: Compute Variance ($s^2$) and Standard Deviation ($s$)
$$\sum f x = 2801, \qquad \sum f x^2 = 215339.5, \qquad n = 38$$
1. **Numerator**:
   $$\sum f x^2 - \frac{(\sum f x)^2}{n} = 215339.5 - \frac{(2801)^2}{38} = 215339.5 - \frac{7845601}{38} = 215339.5 - 206463.1842 = 8876.3158$$
2. **Denominator**: $n - 1 = 37$.
3. **Variance**:
   $$s^2 = \frac{8876.3158}{37} \approx \mathbf{239.9004}$$
4. **Standard Deviation**:
   $$s = \sqrt{239.9004} \approx \mathbf{15.49}$$

### Step 5: Compute Position Quantiles ($P_{35}$ and $D_7$)
- **For $P_{35}$**:
  - Target locator: $\frac{35 \times 38}{100} = 13.3$.
  - First class where $<cf \ge 13.3$ is $60\text{--}69$ ($<cf = 14$).
  - Extraction: $l = 59.5, cf = 8, f = 6, h = 10$.
  $$P_{35} = 59.5 + \left( \frac{13.3 - 8}{6} \right)(10) = 59.5 + \left( \frac{5.3}{6} \right)(10) = 59.5 + 8.83 = \mathbf{68.33}$$
- **For $D_7$ ($P_{70}$)**:
  - Target locator: $\frac{70 \times 38}{100} = 26.6$.
  - First class where $<cf \ge 26.6$ is $80\text{--}89$ ($<cf = 31$).
  - Extraction: $l = 79.5, cf = 23, f = 8, h = 10$.
  $$D_7 = 79.5 + \left( \frac{26.6 - 23}{8} \right)(10) = 79.5 + \left( \frac{3.6}{8} \right)(10) = 79.5 + 4.5 = \mathbf{84.00}$$

---

# 🛡️ Module 03: Inferential Hypothesis Testing for a Single Variance

---

## 📖 Theoretical Machinery & Principles
Inferential tests regarding a single population variance ($\sigma^2$) or standard deviation ($\sigma$) utilise the continuous **Chi-Square ($\chi^2$) Distribution**.

### Fundamental Assumptions
1. **Normality**: The underlying population from which the random sample is drawn ==MUST be normally distributed==. Unlike $t$-tests or $z$-tests on means, the $\chi^2$ test for variance is **NOT robust** to violations of normality.
2. **Independent Random Sampling**: Observations must be gathered independently without replacement from an infinite population or with replacement.

### Mathematical Definition of the Test Statistic
The sample variance $s^2$ is transformed into the test statistic $\chi^2$ via:
$$\chi^2 = \frac{(n - 1)s^2}{\sigma_0^2}$$
Where:
- $n = \text{sample size}$
- $n - 1 = \text{degrees of freedom } (df)$
- $s^2 = \text{calculated sample variance}$
- $\sigma_0^2 = \text{hypothesised population variance stated in null hypothesis } H_0$

> [!warning] ⚠️ Exam Trap & Edge Case: Variances vs. Standard Deviations
> Problem stems frequently state **standard deviations** ($s$ and $\sigma_0$) rather than variances ($s^2$ and $\sigma_0^2$). If the prompt states: *"A sample of size 12 has a standard deviation of 4, test if the population standard deviation exceeds 3"*, you **MUST square both values** before substituting into the equation:
> $$\chi^2 = \frac{(12 - 1)(4^2)}{3^2} = \frac{11 \times 16}{9} = \frac{176}{9} \approx 19.56$$
> Forgetting to square $s$ and $\sigma_0$ is the most common student error on this topic.

---

## 📡 The Geometry of Chi-Square Distributions
Unlike the symmetrical, bell-shaped Gaussian ($z$) or Student's ($t$) distributions:
1. **Non-negativity**: $\chi^2$ values cannot be negative ($\chi^2 \in [0, \infty)$) because they represent sums of squared standard normal variates: $\sum z_i^2$.
2. **Asymmetry**: Distributions are distinctly positively skewed (skewed to the right) for low degrees of freedom.
3. **Asymptotic Convergence**: As $df \to \infty$, the distribution progressively loses its skewness and converges toward a symmetrical Gaussian profile.
4. **Total Area**: The entire integral under the probability density curve equals $1.00$.

```
     f(χ²)
       ▲
       │     *
       │   *   *
       │  *      *
       │ *         *
       │*            *   df = 9 (Right-Skewed)
       │*              *
       │*                 *
       │*                     *
       └─────────────────────────*─────────────────► χ²
      0                           Critical Value (16.919)
```

---

## ⚙️ Reading Critical Values from the Right-Tail $\chi^2$ Table
Standard statistical tables tabulate the critical value $\chi^2_{\alpha}$ corresponding to the area remaining in the **right-hand tail**.

```
    Significance Level (α) = Area under the curve to the right of the critical value
    Degrees of Freedom (df) = n - 1
```

### Critical Value Selection Matrix
| Test Tail Direction | Alternative Hypothesis ($H_1$) | Rejection Region | Required Table Column |
| :--- | :--- | :--- | :--- |
| **Right-Tailed** | $H_1: \sigma^2 > \sigma_0^2$ | Reject $H_0$ if $\chi^2 > \chi^2_{\alpha}$ | Column $\mathbf{\alpha}$ |
| **Left-Tailed** | $H_1: \sigma^2 < \sigma_0^2$ | Reject $H_0$ if $\chi^2 < \chi^2_{1 - \alpha}$ | Column $\mathbf{1 - \alpha}$ |
| **Two-Tailed** | $H_1: \sigma^2 \neq \sigma_0^2$ | Reject $H_0$ if $\chi^2 > \chi^2_{\alpha/2}$ OR $\chi^2 < \chi^2_{1 - \alpha/2}$ | Columns $\mathbf{\alpha/2}$ (upper) and $\mathbf{1 - \alpha/2}$ (lower) |

> [!tip] 💡 Intuition: Why Left Tails Use $(1 - \alpha)$
> Because standard tables define area from the **right tail inward**, an area of $\alpha$ in the far left tail leaves an area of exactly $1 - \alpha$ to its right. Therefore, to capture the lower critical threshold, search under the $1 - \alpha$ header at your given $df$.

---

## 🧪 Comprehensive Walkthrough: The Industrial Bag-Filling Machine

> [!example] 🧪 Step-by-Step Derivation: The Packaging Variance Problem
> **Problem Statement**: A manufacturing plant asserts that the variance of bag weights is $\sigma^2 = 2.5$. A random inspection sample of $n = 10$ bags produces a sample variance of $s^2 = 4.0$. Test whether the true variance is significantly higher than claimed at the $\alpha = 0.05$ significance level.

### Step 1: Formulate Hypotheses
- **Null Hypothesis ($H_0$)**: $\sigma^2 \le 2.5$ (or $\sigma^2 = 2.5$) — The variance adheres to the factory specification.
- **Alternative Hypothesis ($H_1$)**: $\sigma^2 > 2.5$ — The variance exceeds specification (**Right-tailed test**).

### Step 2: Establish the Test Statistic
- Given parameters: $n = 10$, $s^2 = 4.0$, $\sigma_0^2 = 2.5$.
- Degrees of Freedom:
  $$df = n - 1 = 10 - 1 = 9$$
- Calculation:
  $$\chi^2 = \frac{(n - 1)s^2}{\sigma_0^2} = \frac{(10 - 1)(4.0)}{2.5} = \frac{9 \times 4.0}{2.5} = \frac{36}{2.5} = \mathbf{14.40}$$

### Step 3: Determine the Critical Threshold
1. Tail orientation: Right-tailed ($\alpha = 0.05$).
2. Locate row $df = 9$ in the right-tail Chi-Square table.
3. Locate column $\alpha = 0.05$.
4. Intersection value:
   $$\chi^2_{\text{crit}} = \chi^2_{0.05, \, 9} = \mathbf{16.919}$$

### Step 4: Formal Statistical Decision
- **Decision Rule**: Reject $H_0$ if $\chi^2_{\text{calc}} > \chi^2_{\text{crit}}$ ($16.919$).
- Comparison:
  $$\chi^2_{\text{calc}} = 14.40 < 16.919$$
- The test statistic falls into the non-rejection (retention) region.
- **Conclusion**: ==Fail to reject $H_0$==. There is insufficient statistical evidence at the $\alpha = 0.05$ level to substantiate the claim that the bag-weight variance is higher than $2.5$.

---

## 🔬 Comparative Table: $t$-Distribution vs. $\chi^2$-Distribution

| Dimension of Comparison | Student's $t$-Distribution | Chi-Square ($\chi^2$) Distribution |
| :--- | :--- | :--- |
| **Primary Parameter Tested** | Population Mean ($\mu$) with $\sigma$ unknown | Population Variance ($\sigma^2$) or Standard Deviation ($\sigma$) |
| **Symmetry Profile** | Symmetrical, bell-shaped about $t = 0$ | Asymmetric, positively skewed (long right tail) |
| **Domain (Support)** | $(-\infty, +\infty)$ | $[0, +\infty)$ |
| **Degrees of Freedom** | $df = n - 1$ | $df = n - 1$ |
| **Robustness to Non-Normality**| Relatively robust for moderate/large $n$ via CLT | ==Extremely sensitive (fragile)==; non-normality severely invalidates results |
| **Tail Split in Two-Tailed Tests** | Critical values are symmetric: $\pm t_{\alpha/2}$ | Critical values are asymmetric: $\chi^2_{1 - \alpha/2}$ and $\chi^2_{\alpha/2}$ |

