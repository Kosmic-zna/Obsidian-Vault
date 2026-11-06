# 🏛️ Module 01: Bivariate Data & Correlation Analysis

---

> [!info] 📖 Lexicon & First Principles
> - **Bivariate Data**: Data consisting of paired observations $(x, y)$ measured simultaneously on two distinct variables for each individual or experimental unit.
> - **Independent Variable ($x$)**: The explanatory or predictor variable that can be controlled, manipulated, or used as the basis for forecasting (conventionally plotted along the horizontal abscissa).
> - **Dependent Variable ($y$)**: The response or outcome variable whose behaviour is hypothesised to change in response to variations in $x$, and which cannot be directly controlled (conventionally plotted along the vertical ordinate).
> - **Correlation**: A statistical analytical method used to determine whether a ==quantifiable linear relationship== exists between two numerical variables, examining both its **direction** and **strength**.
> - **Sample Correlation Coefficient ($r$)**: Specifically the *Pearson Product-Moment Correlation Coefficient* (PPMC), a dimensionless statistic computed from sample data that quantifies the strength and direction of the linear association between $x$ and $y$.
> - **Population Correlation Coefficient ($\rho$)**: Pronounced *rho*; the theoretical parameter representing the linear correlation computed across **all possible pairs** of data values $(x, y)$ within the entire population.

---

## 🖼️ Graphical Analysis: Scatter Plots

- ⤷ **Definition / Core Rule**: A **scatter plot** (or scatter diagram) is a two-dimensional Cartesian graph of ordered pairs $(x, y)$ consisting of the independent variable $x$ and the dependent variable $y$.
  - ↳ **Primary Analytical Function**: To visually inspect the nature, direction, and linearity of the trend linking two variables before committing to algebraic computations.

> [!tip] 💡 Intuition & Mental Model: The Imaginary Line of Best Fit
> Imagine stretching a taut thread across the cloud of plotted data points so that it passes as close to all of them simultaneously as geometrically possible. The tighter the points hug this imaginary thread, the **stronger** the statistical relationship between the two variables; the more dispersed the cloud, the **weaker** the relationship.

### Taxonomy of Bivariate Relationships

| Relationship Classification | Visual Cluster Geometry | Co-movement Behaviour | Typical Real-World Illustration |
| :--- | :--- | :--- | :--- |
| **Positive Linear Relationship** | Points cluster in an ascending band running from **lower left to upper right**. | ==Variables move in tandem==: as $x$ increases, $y$ increases; as $x$ decreases, $y$ decreases. | Chronological Age ($x$) vs. Systolic Blood Pressure ($y$). |
| **Negative Linear Relationship** | Points cluster in a descending band running from **upper left to lower right**. | ==Variables move inversely==: as $x$ increases, $y$ decreases; as $x$ decreases, $y$ increases. | Number of Class Absences ($x$) vs. Final Course Grade ($y$). |
| **No Relationship (Null Association)** | Points form a random, amorphous cloud with **no discernible clustering** or slope. | Fluctuations in $x$ yield no predictable directional shift in $y$. | Random paired drawings ($x, y$) where no underlying mechanism links the two metrics. |

---

## 📐 Pearson's Sample Correlation Coefficient ($r$)

### 1. Mathematical Domain & Continuum of $r$
- ⤷ **Definition / Core Rule**: The sample correlation coefficient $r$ is mathematically bounded within the closed interval $[-1, +1]$:
  $$-1 \le r \le +1$$
  - ↳ **Sign ($+$ or $-$)**: Dictates the **direction** (slope orientation) of the linear relationship.
  - ↳ **Magnitude ($|r|$)**: Dictates the **strength** (tightness of clustering) around the linear trendline.

```
   Perfect Negative          No Linear            Perfect Positive
     Correlation            Relationship            Correlation
          │                      │                       │
          ▼                      ▼                       ▼
         -1.0       -0.5        0.0        +0.5        +1.0
          ◄──────────┼───────────┼───────────┼───────────►
        Strong     Moderate     Weak      Moderate     Strong
       Negative    Negative   or None     Positive    Positive
```

- ⤷ **Boundary Interpretations**:
  - ↳ **$r = +1$**: **Perfect positive linear correlation** — every single data point falls strictly on an upward-sloping straight line.
  - ↳ **$r$ close to $+1$**: **Strong positive association** — points cluster tightly along a rising band.
  - ↳ **$r$ close to $0$**: **Weak or no linear association** — points exhibit high scatter with no clear linear trajectory.
  - ↳ **$r$ close to $-1$**: **Strong negative association** — points cluster tightly along a falling band.
  - ↳ **$r = -1$**: **Perfect negative linear correlation** — every single data point falls strictly on a downward-sloping straight line.

---

### 2. Computational Formula for $r$

> [!bug] 🚨 Lecture Material Discrepancy: Missing Exponent & Bracket in Formula
> On the slide entitled *"Formula for the Correlation Coefficient ($r$)"*, the denominator is printed as:
> `\sqrt{n(\Sigma x^2) - (\Sigma x)][n(\Sigma y^2) - (\Sigma y)^2]}`
> This contains two critical typographical blunders: the opening square bracket `[` is omitted before the first $n$, and the exponent `$2$` is omitted on the $(\sum x)^2$ term. Using the un-squared $\sum x$ in an examination will yield an entirely erroneous radical (or an imaginary number). Always apply the corrected formula below.

- ⤷ **Definition / Core Rule**: The computational shortcut formula for the Pearson correlation coefficient across $n$ data pairs is:
  $$r = \frac{n\left(\sum xy\right) - \left(\sum x\right)\left(\sum y\right)}{\sqrt{\left[n\left(\sum x^2\right) - \left(\sum x\right)^2\right]\left[n\left(\sum y^2\right) - \left(\sum y\right)^2\right]}}$$
  Where:
  - $n = \text{total number of ordered data pairs } (x, y)$
  - $\sum x = \text{sum of all independent variable observations}$
  - $\sum y = \text{sum of all dependent variable observations}$
  - $\sum xy = \text{sum of the pairwise products of } x \text{ and } y$
  - $\sum x^2 = \text{sum of the squares of each individual } x \text{ value}$
  - $\sum y^2 = \text{sum of the squares of each individual } y \text{ value}$

> [!warning] ⚠️ Exam Trap & Edge Case: $\sum x^2$ vs. $(\sum x)^2$
> Never conflate $\sum x^2$ with $(\sum x)^2$:
> - **$\sum x^2$ (Sum of Squares)**: Square each individual $x$ observation **first**, then sum the column downwards.
> - **$(\sum x)^2$ (Square of the Sum)**: Sum the raw $x$ column **first**, then square that single total.
> - Furthermore, by the Cauchy-Schwarz inequality, the quantities $n(\sum x^2) - (\sum x)^2$ and $n(\sum y^2) - (\sum y)^2$ **must always be strictly non-negative** ($\ge 0$). If your calculator yields a negative number inside the square root, you have swapped $\sum x^2$ and $(\sum x)^2$.

---

## 🧪 Master Walkthrough: Student Absences vs. Final Grades

> [!example] 🧪 Step-by-Step Derivation: Computing $r$ for the 7-Student Cohort
> **Problem Statement**: A random sample of $n = 7$ students from a statistics class is evaluated to determine the relationship between the number of absences ($x$) and the final course grade in percent ($y$). Compute and interpret the sample correlation coefficient $r$.

### Step 1: Construct the Summation Matrix
| Student | Number of Absences ($x$) | Final Grade ($y\%$) | $xy$ | $x^2$ | $y^2$ |
| :---: | :---: | :---: | :---: | :---: | :---: |
| **A** | $6$ | $82$ | $492$ | $36$ | $6\,724$ |
| **B** | $2$ | $86$ | $172$ | $4$ | $7\,396$ |
| **C** | $15$ | $43$ | $645$ | $225$ | $1\,849$ |
| **D** | $9$ | $74$ | $666$ | $81$ | $5\,476$ |
| **E** | $12$ | $58$ | $696$ | $144$ | $3\,364$ |
| **F** | $5$ | $90$ | $450$ | $25$ | $8\,100$ |
| **G** | $8$ | $78$ | $624$ | $64$ | $6\,084$ |
| **Totals ($n = 7$)** | $\mathbf{\sum x = 57}$ | $\mathbf{\sum y = 511}$ | $\mathbf{\sum xy = 3\,745}$ | $\mathbf{\sum x^2 = 579}$ | $\mathbf{\sum y^2 = 38\,993}$ |

### Step 2: Substitute Totals into the Pearson Formula
1. **Compute the Numerator (Covariance Term)**:
   $$n\left(\sum xy\right) - \left(\sum x\right)\left(\sum y\right) = (7)(3\,745) - (57)(511) = 26\,215 - 29\,127 = \mathbf{-2\,912}$$
2. **Compute the Left Radical Bracket ($x$-Variation Term)**:
   $$n\left(\sum x^2\right) - \left(\sum x\right)^2 = (7)(579) - (57)^2 = 4\,053 - 3\,249 = \mathbf{804}$$
3. **Compute the Right Radical Bracket ($y$-Variation Term)**:
   $$n\left(\sum y^2\right) - \left(\sum y\right)^2 = (7)(38\,993) - (511)^2 = 272\,951 - 261\,121 = \mathbf{11\,830}$$
4. **Assemble and Evaluate $r$**:
   $$r = \frac{-2\,912}{\sqrt{(804)(11\,830)}} = \frac{-2\,912}{\sqrt{9\,511\,320}} = \frac{-2\,912}{3\,084.0428} \approx \mathbf{-0.9442} \approx \mathbf{-0.944}$$

### Step 3: Formal Analytical Interpretation
- Because $r = -0.944$ lies very close to $-1.00$, there exists a ==strong negative linear relationship== between a student's number of absences and their final grade. Specifically, as the number of absences increases, a student's final grade systematically decreases.

---
---

# 🛡️ Module 02: Inferential Hypothesis Testing for $\rho$

---

## ⚙️ Theoretical Machinery: Significance of the Correlation Coefficient

- ⤷ **Definition / Core Rule**: Computing a non-zero sample correlation $r$ from a small sample does not automatically prove that a genuine relationship exists in the wider population; the observed $r$ could merely be an artefact of **random sampling fluctuation (chance)**. We therefore test the population parameter $\rho$ via a formal Student's $t$-test.
  - ↳ **Null Hypothesis ($H_0$)**:
    $$H_0: \rho = 0$$
    Asserts that there is ==no linear correlation== between $x$ and $y$ in the population (any deviation of $r$ from $0$ is purely due to random chance).
  - ↳ **Alternative Hypothesis ($H_1$)**:
    $$H_1: \rho \neq 0$$
    Asserts that there is a ==statistically significant correlation== between the variables in the population (**Two-tailed test**).

> [!tip] 💡 Intuition & Mental Model: Signal-to-Noise Ratio
> Think of the $t$-test statistic for correlation as a signal-to-noise detector. A sample of size $n = 3$ can easily produce a high $r$ purely by coincidence. The formula scales your observed $r$ by the sample size $\sqrt{n - 2}$ (the sample's evidential weight) and divides by $\sqrt{1 - r^2}$ (the unexplained noise). Only when the signal overwhelms the noise does $t$ cross the critical threshold into the rejection region.

### Test Statistic & Degrees of Freedom
- ⤷ **Formula for the $t$-Test for the Correlation Coefficient**:
  $$t = r\sqrt{\frac{n - 2}{1 - r^2}}$$
  Where:
  - $r = \text{computed sample correlation coefficient}$
  - $n = \text{number of paired observations } (x, y)$
  - $df = n - 2 = \text{degrees of freedom}$

> [!warning] ⚠️ Exam Trap & Edge Case: Why $df = n - 2$?
> In a single-sample $t$-test for a mean (Module 03 of your previous reviewer), degrees of freedom equal $n - 1$ because only one parameter ($\bar{X}$) is estimated. In bivariate correlation and regression, we simultaneously estimate **two** parameters ($\bar{x}$ and $\bar{y}$, or equivalently the intercept $a$ and slope $b$), which consumes **two constraints**. Consequently, you ==MUST use $df = n - 2$== when consulting the $t$-distribution table (Table F). Using $n - 1$ will select the wrong row and invalidate your critical value.

> [!bug] 🚨 Lecture Material Discrepancy: Table F OCR Corruption
> On the slide displaying **Table F (The $t$ Distribution)**, the column for Two Tails $\alpha = 0.05$ contains corrupted numerical entries:
> - Row $df = 2$ prints `4603` instead of the true mathematical value **$4.303$**.
> - Row $df = 3$ prints `382` instead of the true mathematical value **$3.182$**.
> - Row $df = 4$ (One tail $\alpha = 0.10$) is left completely blank instead of **$1.533$**.
> Ensure your personal reference sheet records the true decimal values!

---

## 🧪 Master Walkthrough: US Car Rental Fleet vs. Annual Revenue

> [!example] 🧪 Step-by-Step Derivation: Testing $H_0: \rho = 0$
> **Problem Statement**: Data are collected for $n = 6$ United States car rental companies comparing fleet size in ten-thousands of cars ($x$) and annual revenue in billions of dollars ($y$). Given $\alpha = 0.05$ and a computed sample correlation of $r = 0.9819798097$ (or $r \approx 0.982$), test the significance of the correlation coefficient.

| Company | Fleet Size $x$ (in $10\,000\text{s}$) | Revenue $y$ (in $\$$ billions) |
| :---: | :---: | :---: |
| **A** | $63.0$ | $7.0$ |
| **B** | $29.0$ | $3.9$ |
| **C** | $20.8$ | $2.1$ |
| **D** | $19.1$ | $2.8$ |
| **E** | $13.4$ | $1.4$ |
| **F** | $8.5$ | $1.5$ |

> [!abstract] ⚙️ Process & Operational Flow: 5-Step Hypothesis Testing Pipeline
> **Step 1: State Hypotheses** $\rightarrow$ **Step 2: Locate Critical Value ($C.V.$)** $\rightarrow$ **Step 3: Compute Test Value ($t$)** $\rightarrow$ **Step 4: Formulate Decision** $\rightarrow$ **Step 5: Contextual Summary**

1. **Step 1: State the Hypotheses**:
   - $H_0: \rho = 0$ (No significant population correlation)
   - $H_1: \rho \neq 0$ (Significant population correlation — **Two-tailed test**)
2. **Step 2: Find the Critical Value ($C.V.$)**:
   - Given $\alpha = 0.05$ (two-tailed) and degrees of freedom $df = n - 2 = 6 - 2 = 4$.
   - From the $t$-distribution table at row $df = 4$ and column $\text{Two tails } \alpha = 0.05$:
     $$C.V. = \pm 2.776$$
3. **Step 3: Compute the Test Statistic ($t$)**:
   $$t = r\sqrt{\frac{n - 2}{1 - r^2}} = 0.9819798097\sqrt{\frac{6 - 2}{1 - (0.9819798097)^2}}$$
   $$t = 0.9819798097\sqrt{\frac{4}{1 - 0.9642843467}} = 0.9819798097\sqrt{\frac{4}{0.0357156533}} = 0.9819798097(10.5828) \approx \mathbf{+10.392} \approx \mathbf{+10.4}$$
4. **Step 4: Make the Statistical Decision**:
   ```
                    Gaussian / Student's t-Curve (df = 4)
                                     _.-""""-._
                                   .'          '.
                                  /              \
                                 ;                ;
                         _..---..|                |..---.._
               ░░░░░░░░░         |                |         ░░░░░░░░░
        ───────┼─────────────────┼────────────────┼─────────────────┼───────┼───► t
            -2.776               0             +2.776            +10.4
          [Reject H0]      [Do Not Reject H0]                [Reject H0]
   ```
   - Compare test statistic against critical threshold: $t_{\text{calc}} = +10.4 > +2.776$.
   - Because the test value falls deep inside the right-hand critical (rejection) region, we ==Reject $H_0$== (and do not reject $H_1$).
5. **Step 5: Summarise the Results**:
   - There is a statistically significant linear relationship between the number of cars a rental agency owns and its annual revenue at the $\alpha = 0.05$ significance level.

---
---

# 📐 Module 03: Simple Linear Regression & Determination

---

## ⚙️ Linear Regression & The Line of Best Fit

- ⤷ **Definition / Core Rule**: **Linear Regression** is an algebraic and statistical methodology for modelling the relationship between a dependent variable $y$ and an independent variable $x$ by fitting a linear equation to observed data.
  - ↳ **Regression Line (Line of Best Fit)**: The unique straight line passing through the scatter plot that minimises the sum of squared vertical distances (residuals) between the observed data points $y$ and the line's predicted values $y'$.
  - ↳ **Equation of the Regression Line**:
    $$y' = a + bx$$
    Where:
    - $y' = \text{predicted (estimated) value of the dependent variable } y \text{ for a given } x$
    - $a = \text{the } y'\text{-intercept}$ (the predicted value of $y'$ when $x = 0$)
    - $b = \text{the slope of the regression line}$ (the marginal rate of change in $y'$ per $1\text{-unit}$ increase in $x$)

> [!warning] ⚠️ Exam Trap & Edge Case: Algebraic Equation Order ($a + bx$ vs. $mx + b$)
> In elementary algebra, a line is written as $y = mx + b$, where $m$ is the slope and $b$ is the intercept. In Bluman's statistics convention (used throughout your lecture slides), the equation is written as **$y' = a + bx$**, where:
> - **$a$ is the $y'$-intercept** (constant term).
> - **$b$ is the slope** (coefficient attached to $x$).
> Never swap the roles of $a$ and $b$ when assembling your final regression equation!

### Formulas for the Regression Coefficients ($a$ and $b$)

- ⤷ **$y'$-Intercept ($a$)**:
  $$a = \frac{\left(\sum y\right)\left(\sum x^2\right) - \left(\sum x\right)\left(\sum xy\right)}{n\left(\sum x^2\right) - \left(\sum x\right)^2}$$
- ⤷ **Slope ($b$)**:
  $$b = \frac{n\left(\sum xy\right) - \left(\sum x\right)\left(\sum y\right)}{n\left(\sum x^2\right) - \left(\sum x\right)^2}$$

> [!check] 🔢 High-Yield Computational Shortcuts & Algebraic Identities
> 1. **Identical Denominators**: Notice that the denominator for both $a$ and $b$ is identical: $n(\sum x^2) - (\sum x)^2$. Furthermore, this is the exact same $x$-variation bracket you already computed inside the square root for $r$!
> 2. **Identical Numerators for $b$ and $r$**: The numerator of the slope $b$ is identical to the numerator of the correlation coefficient $r$: $n(\sum xy) - (\sum x)(\sum y)$. Thus, **$b$ and $r$ MUST always share the exact same algebraic sign ($+$ or $-$)**.
> 3. **Mean-Centroid Identity for $a$**: Once you have computed $b$, you can verify $a$ instantaneously because the regression line is mathematically guaranteed to pass through the centroid $(\bar{x}, \bar{y})$:
>    $$a = \bar{y} - b\bar{x} = \frac{\sum y - b\left(\sum x\right)}{n}$$
> 4. **When is Forecasting Valid?**: Regression prediction should only be relied upon when the correlation coefficient $r$ has been confirmed to be **statistically significant** (i.e., $H_0: \rho = 0$ is rejected). If $r$ is not significant, the regression line is meaningless for prediction.

---

## 🧪 Master Walkthrough: Deriving & Forecasting with the Regression Line

> [!example] 🧪 Step-by-Step Derivation: The 7-Student Absences Regression Model
> **Problem Statement**: Using the 7-student absences dataset from Module 01:
> - **(a)** Find the equation of the regression line $y' = a + bx$.
> - **(b)** Predict the final grade ($y'$) of a student who has accumulated $x = 10$ absences.

### Part (a): Compute Intercept $a$ and Slope $b$
Recall our summation parameters from the $n = 7$ table (note that $\sum y^2$ is **not required** to find the regression line):
$$\sum x = 57, \qquad \sum y = 511, \qquad \sum xy = 3\,745, \qquad \sum x^2 = 579$$

1. **Calculate the Common Denominator**:
   $$n\left(\sum x^2\right) - \left(\sum x\right)^2 = (7)(579) - (57)^2 = 4\,053 - 3\,249 = \mathbf{804}$$
2. **Calculate the $y'$-Intercept ($a$)**:
   $$a = \frac{\left(\sum y\right)\left(\sum x^2\right) - \left(\sum x\right)\left(\sum xy\right)}{n\left(\sum x^2\right) - \left(\sum x\right)^2} = \frac{(511)(579) - (57)(3\,745)}{804}$$
   $$a = \frac{295\,869 - 213\,465}{804} = \frac{82\,404}{804} \approx \mathbf{102.4925} \approx \mathbf{102.493}$$
3. **Calculate the Slope ($b$)**:
   $$b = \frac{n\left(\sum xy\right) - \left(\sum x\right)\left(\sum y\right)}{n\left(\sum x^2\right) - \left(\sum x\right)^2} = \frac{(7)(3\,745) - (57)(511)}{804}$$
   $$b = \frac{26\,215 - 29\,127}{804} = \frac{-2\,912}{804} \approx \mathbf{-3.6219} \approx \mathbf{-3.622}$$
4. **Formulate the Regression Line Equation**:
   $$y' = 102.493 - 3.622x$$
   - *Physical Meaning*: A student with $0$ absences is predicted to score $102.493\%$, and each additional absence reduces the predicted final grade by $3.622$ percentage points.

### Part (b): Point Prediction for $x = 10$ Absences
Substitute $x = 10$ directly into the derived regression equation:
$$y' = 102.493 - 3.622(10)$$
$$y' = 102.493 - 36.220 = \mathbf{66.273}$$
- **Conclusion**: The predicted final grade for a student with $10$ absences is **$66.273\%$**.

---

## 🍞 Coefficient of Determination ($r^2$) & Non-Determination ($1 - r^2$)

- ⤷ **Coefficient of Determination ($r^2$)**:
  - ↳ **Definition / Core Rule**: Denoted by $r^2$ (the exact mathematical square of the linear correlation coefficient $r$), it measures the ==proportion (or percentage) of total variation in the dependent variable $y$== that is **explained** by the regression line and the independent variable $x$.
  - ↳ **Range**: $0 \le r^2 \le 1$ (usually expressed as a percentage between $0\%$ and $100\%$).
  > → *Example*: If $r = 0.90$, then $r^2 = (0.90)^2 = \mathbf{0.81}$ (or **$81\%$**). This means that $81\%$ of the variation in the dependent variable $y$ is accounted for by variations in the independent variable $x$ via the linear model.
- ⤷ **Coefficient of Non-Determination ($1 - r^2$)**:
  - ↳ **Definition / Core Rule**: The complementary proportion of total variation in the dependent variable $y$ that remains ==unexplained by the regression line==, attributable instead to random error, chance, or unmeasured lurking variables.
  $$\text{Coefficient of Non-Determination} = 1 - r^2$$
  > → *Example*: If $r = 0.90$, then $r^2 = 0.81$, and the coefficient of non-determination is $1 - 0.81 = \mathbf{0.19}$ (or **$19\%$**). Thus, $19\%$ of the variation in $y$ is unexplained by $x$.

> [!tip] 💡 Intuition & Mental Model: The Pie of Total Variance
> Imagine the total variability in student grades ($y$) as a complete $100\%$ pie. Computing $r^2$ slices that pie into two distinct pieces:
> - **$r^2$ (Explained Variation)**: The slice of grade differences directly attributable to class attendance ($x$). For our absences dataset ($r = -0.944$), $r^2 = (-0.944)^2 \approx 0.891$ ($89.1\%$).
> - **$1 - r^2$ (Unexplained / Residual Variation)**: The remaining slice ($1 - 0.891 = 0.109$ or $10.9\%$) caused by external factors not in our model—such as prior mathematical aptitude, hours of sleep, or study habits.

---

## 🔬 Master Formula & Concept Comparison Matrix

| Metric / Construct | Symbolic Notation | Mathematical Formula | Analytical Purpose & Domain |
| :--- | :---: | :--- | :--- |
| **Pearson Correlation Coefficient** | $r$ | $\dfrac{n(\sum xy) - (\sum x)(\sum y)}{\sqrt{[n(\sum x^2) - (\sum x)^2][n(\sum y^2) - (\sum y)^2]}}$ | Measures linear **strength & direction**; $r \in [-1, +1]$. Dimensionless. |
| **$t$-Test for Correlation Significance** | $t$ | $r\sqrt{\dfrac{n - 2}{1 - r^2}} \quad (df = n - 2)$ | Tests $H_0: \rho = 0$ vs. $H_1: \rho \neq 0$; determines if $r$ is statistically distinct from zero. |
| **Regression Line ($y'$-Intercept)** | $a$ | $\dfrac{(\sum y)(\sum x^2) - (\sum x)(\sum xy)}{n(\sum x^2) - (\sum x)^2} = \bar{y} - b\bar{x}$ | Predicted baseline value of $y'$ when $x = 0$. |
| **Regression Line (Slope)** | $b$ | $\dfrac{n(\sum xy) - (\sum x)(\sum y)}{n(\sum x^2) - (\sum x)^2}$ | Marginal change in predicted $y'$ per unit increase in $x$. Shares sign with $r$. |
| **Coefficient of Determination** | $r^2$ | $(r)^2$ | Proportion of $y$-variation **explained** by $x$ and the regression line; $r^2 \in [0, 1]$. |
| **Coefficient of Non-Determination** | $1 - r^2$ | $1 - (r)^2$ | Proportion of $y$-variation **unexplained** (residual error/lurking factors). |