# 📊 The Nature of Statistics

## 📖 Prerequisite Lexicon: Foundational Building Blocks

- ⤷ **Data**: Raw, unprocessed factual observations, measurements, figures, or counts collected from an environment. In plain English, data constitutes the bare facts before any meaning, narrative, or synthesis is extracted from them.
    
- ⤷ **Variable**: Any measurable characteristic, trait, or property of an entity that can vary across different subjects or change over time.
    
- ⤷ **Observation (Datum)**: The specific realized numerical or categorical value recorded for a single variable on a particular subject.
    
- ⤷ **Census**: An exhaustive accounting wherein every single member or constituent entity of an entire target group is directly inspected or measured.
    
- ⤷ **Inference**: The intellectual leap of deriving general conclusions, estimates, or probabilistic predictions regarding an unseen whole based exclusively on a visible part.
    

## 🏛️ Foundations of Statistical Science

- ⤷ **Definition of Statistics (Comprehensive Academic View)**: The mathematical science concerned with the systematic collection, organisation, presentation, analysis, and interpretation of numerical data to extract actionable information and deduce sound conclusions.
    
- ⤷ **Definition of Statistics (McCarthy's Operational View)**: A set of mathematical rules, algorithms, and formal procedures engineered to condense vast, unmanageable masses of data into intelligible proportions, thereby enabling empirical deduction.
    
- ⤷ **Statistician**: A professional formally equipped to orchestrate data collection protocols, evaluate empirical evidence, quantify uncertainty, and draw grounded conclusions.
    
- ⤷ **Measurement**: The operational assignment of numbers, symbols, or labels to physical phenomena or attributes according to pre-determined mathematical rules.
    
    - → _Example_: Recording the historical height of Dr. José Rizal as $4\text{ ft } 11\text{ in}$.
        

> [!tip] 💡 Intuition: The Data Compression Mental Model Imagine attempting to read a raw server log containing two million HTTP requests line by line; your brain would stall. Statistics acts as a lossy yet optimal compression algorithm: it condenses the torrential deluge of raw events into an intelligible summary—such as an average latency or an error rate—allowing you to perceive the systemic state at a glance.

## 👥 Populations Versus Samples: Parameters Versus Statistics

```
=============================================================
 POPULATION (Complete Universe: Size N) ──► Parameter (Greek: μ, σ, N)
         │
         │  [ Sampling: n = N / (1 + Ne²) ]
         ▼
   SAMPLE (Observed Subset: Size n)     ──► Statistic (Latin: x̄, s, n)
=============================================================
```

- ⤷ **Population ($N$)**: The exhaustive aggregate of all individual elements, subjects, units, or items that share a defined set of characteristics under empirical investigation.
    
    - ↳ **Parameter**: A numerical index or summary metric that describes a characteristic of the entire population. Parameters are constant, fixed values, yet they remain largely unknown in practice because measuring an entire population is rarely feasible.
        
- ⤷ **Sample ($n$)**: A representative portion or sub-collection of units systematically extracted from the target population for direct analysis.
    
    - ↳ **Statistic**: A numerical measure computed directly from sample data. Statistics are observable, variable, and serve as empirical estimators for their corresponding population parameters.
        

|**Analytical Dimension**|**Population (NNN)**|**Sample (nnn)**|
|---|---|---|
|**Scope**|Complete collection of all units under study|Concrete subset selected for empirical analysis|
|**Descriptive Metric**|**Parameter** (e.g., Population Mean $\mu$)|**Statistic** (e.g., Sample Mean $\bar{X}$)|
|**Feasibility & Cost**|Extremely costly, labour-intensive, and often logistically impossible|Highly cost-effective, rapid, and logistically manageable|
|**Mnemonic Anchor**|**P**opulation $\leftrightarrow$ **P**arameter|**S**ample $\leftrightarrow$ **S**tatistic|

### 🧮 Sample Size Determination: Slovin's Formula

- ⤷ When the target population size $N$ is finite and known, the minimal acceptable sample size $n$ required to achieve an empirical tolerance margin of error $e$ is determined algebraically:
    

$$n = \frac{N}{1 + N e^2}$$

- ↳ Where:
    
    - $n$ = Required sample size
        
    - $N$ = Total population size
        
    - $e$ = Tolerable margin of error (expressed as a decimal fraction, e.g., $5\% = 0.05$)
        

> [!bug] 🚨 Lecture Material Discrepancy: Slovin's Formula Attribution Slide 45 displays the formula $n = \frac{N}{1 + N e^2}$ without providing its formal title. In Philippine academic curricula (including Mapúa/MCL), this construct is routinely referred to on examinations as **Slovin's Formula** (historically identical to Yamane's 1967 simplified formulation). Commit the structural equation to memory under this name.

## 🔬 The Two Grand Branches of Statistics

```
                           STATISTICS
                               │
         ┌─────────────────────┴─────────────────────┐
         ▼                                           ▼
   DESCRIPTIVE                                  INFERENTIAL
(Summarise the Known)                     (Generalise to the Unknown)
• Mean, Median, Mode           • Hypothesis Testing
• Tables, Charts, Graphs       • Interval Estimation
• Observed Percentages[cite: 55, 56]     • Future Predictions
```

### 1. Descriptive Statistics

- ⤷ **Core Role**: The collection, organisation, summarisation, condensation, and graphical presentation of empirical data.
    
- ↳ **Operational Boundary**: Confined strictly to describing the specific group of subjects actually observed; ==it never makes claims about unobserved individuals or future outcomes==.
    
- ↳ **Standard Methodology**:
    
    - **Data Collection**: Standardised surveys and observational records.
        
    - **Data Presentation**: Frequency distribution tables, histograms, bar charts, and line plots.
        
    - **Data Characterisation**: Calculation of sample metrics, such as the arithmetic mean[cite: 54]:
        

$$\bar{X} = \frac{\sum X_i}{n}$$

[cite: 54]

- → _Applied Examples_:
    
    - Calculating the departmental allocation of a university student's weekly allowance.
        
    - Reporting that Malayan Colleges Laguna achieved a $100\%$ passing rate in the 9th Industrial Engineering Certification Examination, with one graduate placing 5th nationally.
        
    - Tabulating Stephen Curry's official 2015–16 NBA metrics ($30.1\text{ PPG}$, $6.7\text{ APG}$, $5.4\text{ RPG}$).
        

### 2. Inferential Statistics

- ⤷ **Core Role**: Generalising findings from a measured sample to make assertions, estimations, tests of significance, or forecasts concerning an unmeasured population.
    
- ↳ **Operational Boundary**: Extends beyond immediate observations; it calculates probabilities to evaluate real-world claims under conditions of uncertainty[cite: 57, 58].
    
- ↳ **Primary Workflows**:
    
    - **Estimation**: Using an observed sample mean ($\bar{X}$) to approximate the true population mean ($\mu$).
        
    - **Hypothesis Testing**: Formulating an empirical claim and assessing whether sample evidence refutes the null state[cite: 57, 58, 60].
        
    - **Forecasting & Modelling**: Projecting future behavioural or environmental trajectories from sample trends.
        
- → _Applied Examples_:
    
    - Inferring from clinical trial samples that patients on blood-pressure-lowering medication suffer an increased risk of heart disease.
        
    - Testing whether there is a statistically significant difference between the average heights of male and female students at an institution.
        
    - Projecting that by the year 2040, at least 3.5 billion people worldwide will face severe water shortages.
        

## 🗂️ Taxonomy of Data: Types & Classifications

```
                                  DATA
                                    │
         ┌──────────────────────────┴──────────────────────────┐
         ▼                                                     ▼
    QUALITATIVE                                           QUANTITATIVE
   (Categorical)                                           (Numerical)
• Words, Labels, Codes                       • Counts, Scales, Weights[cite: 65, 69]
• Non-arithmetic                         • Arithmetic is meaningful[cite: 65, 79]
                                                               │
                                            ┌──────────────────┴──────────────────┐
                                            ▼                                     ▼
                                        DISCRETE                              CONTINUOUS
                                   (Finite / Counted)                    (Infinite / Measured)
                                   • Integers ($0, 1, 2$)     • Real continuum[cite: 67]
                                   • Absolute gaps[cite: 67]            • No gaps / jumps[cite: 67]
```

### 1. Qualitative (Categorical) Data

- ⤷ **Definition**: Variables comprising labels, names, or non-numerical category identifiers that classify subjects based on qualitative attributes[cite: 64].
    
- ↳ **Operational Rule**: ==Mathematical or arithmetic operations (e.g., addition, division) on these values are impossible or meaningless==, even if categories are coded using numerical digits[cite: 64, 72].
    
- → _Examples_: Academic major, political party affiliation, eye colour, biological sex, stance on capital punishment (Pro vs. Anti).
    

### 2. Quantitative (Numerical) Data

- ⤷ **Definition**: Variables consisting of numerical counts or objective physical measurements for which arithmetic ordering and calculation are naturally meaningful.
    
- ↳ **Sub-Class A: Discrete Quantitative Data**
    
    - ↳ **Core Rule**: Numerical values obtained strictly via **counting**; mathematically integer-valued ($x \in \mathbb{Z}_{\ge 0}$) with discrete gaps between consecutive points.
        
    - → _Examples_: Number of attendees at a conference, number of defects produced per hour in a manufacturing plant, total children in a household.
        
- ↳ **Sub-Class B: Continuous Quantitative Data**
    
    - ↳ **Core Rule**: Numerical values obtained strictly via **measurement**; mathematically real-valued ($x \in \mathbb{R}$) along an unbroken continuous spectrum.
        
    - ↳ **Defining Quality**: ==Between any two conceivable points, there exists an infinite number of intermediate fractional or decimal values==[cite: 67].
        
    - → _Examples_: Ambient temperature readings, running time/chronological age, electrical voltage, waiting duration (e.g., hours elapsed waiting for a partner).
        

> [!warning] ⚠️ Exam Trap & Edge Case: The "Age" Classification On introductory mathematics examinations, students routinely mislabel **Age** as discrete because individuals report it as an integer ("18 years old"). Scientifically and statistically, age is continuous time elapsed since birth ($18.4239\dots\text{ years}$); it is continuous data grouped into discrete intervals for conversational convenience.

## 📐 The Four Levels of Measurement: The NOIR Hierarchy

```
[RATIO]    ◄── Absolute True Zero Present (Ratios & All Math Valid)
   ▲
[INTERVAL] ◄── Meaningful Differences; Arbitrary Zero (No Absolute Absence)
   ▲
[ORDINAL]  ◄── Rankable Hierarchy; Unequal or Meaningless Intervals
   ▲
[NOMINAL]  ◄── Raw Labels & Qualitative Categories Only (No Order)
```

```
==========================================================================
 Level     Categorise?   Rank Order?   Equal Differences?   Absolute True Zero?
==========================================================================
 Nominal       YES           NO                NO                   NO       
 Ordinal       YES           YES               NO                   NO      
 Interval      YES           YES               YES                  NO      
 Ratio         YES           YES               YES                  YES     
==========================================================================
```

### 1. Nominal Level (The Naming Scale)

- ⤷ **Characteristics**: The lowest and most basic level of measurement. Data consist purely of labels, titles, identifiers, or mutually exclusive categories[cite: 72].
    
- ↳ **Mathematical Limit**: ==No rank order exists, and no arithmetic operations can be performed==[cite: 72].
    
- → _Examples_: Blood types ($A, B, AB, O$), marital status (single, married, separated), university names, provincial residence, motor vehicle colour codes, national identification numbers.
    

### 2. Ordinal Level (The Ranking Scale)

- ⤷ **Characteristics**: Data can be arranged in a distinct logical ranking or directional order.
    
- ↳ **Mathematical Limit**: While relative order is established ($A > B > C$), ==the differences between adjacent ranks are undefined, variable, or mathematically meaningless==[cite: 75, 76]. You cannot state _how much more_ one rank possesses relative to another[cite: 75].
    
- → _Examples_:
    
    - Qualitative ratings: Product ratings (_Poor, Good, Excellent_).
        
    - Socioeconomic tiers: _Lower, Middle, Upper Class_.
        
    - Subjective clinical pain scales: _None, Low, Moderate, Severe_.
        
    - Academic survey matrices: Standard 5-point Likert Scales (_Strongly Agree, Agree, Neutral, Disagree, Strongly Disagree_).
        

### 3. Interval Level (The Meaningful Difference Scale)

- ⤷ **Characteristics**: Data are numerical, possess intrinsic rank, and equal numerical distances between points represent equal quantities of the underlying attribute.
    
- ↳ **Mathematical Limit**: ==Possesses an arbitrary zero point==[cite: 78]. A reading of $0$ does _not_ indicate the complete absence of the characteristic, meaning ratios are mathematically invalid.
    
    - → _Why True Zero Matters_: $80^\circ\text{F}$ is not "twice as hot" as $40^\circ\text{F}$, because $0^\circ\text{F}$ does not represent the absolute extinction of thermal energy[cite: 78].
        
- → _Examples_: Ambient temperature scales (Celsius and Fahrenheit), standardised intelligence quotients (IQ scores).
    

### 4. Ratio Level (The Absolute Scale)

- ⤷ **Characteristics**: The highest and most mathematically complete level of measurement. Possesses ordered categories, precise equal intervals, and a genuine, non-arbitrary **absolute true zero**[cite: 79, 80].
    
- ↳ **Mathematical Freedom**: Because zero signifies the total absence of the variable, ==all arithmetic operations—including division and ratio calculations—are mathematically valid==[cite: 79, 80].
    
    - → _Why Ratios Work_: A worker earning $P40,000$ earns precisely double a worker earning $P20,000$, and zero pesos denotes an absolute absence of money.
        
- → _Examples_: Monthly wages, physical height, total body weight, unit production volumes, stock price variations, travel distance between branch sites, grams of dietary fats consumed.
    

## 🎯 Sampling Methodologies: Statistical Versus Non-Statistical

```
                          SAMPLING TECHNIQUES
                                   │
         ┌─────────────────────────┴─────────────────────────┐
         ▼                                                   ▼
NON-STATISTICAL                                         STATISTICAL
(Non-Probability)                                       (Probability)
• Unknown probabilities                  • Known, calculable selection chances
• Susceptible to selection bias[cite: 85, 88]          • Required for formal inference[cite: 57, 84, 89]
  ├─ Convenience                           ├─ Simple Random[cite: 87, 90]
  ├─ Judgmental / Purposive                ├─ Stratified Random[cite: 87, 91]
  └─ Snowball                                  ├─ Systematic[cite: 87, 92]
                                                          └─ Cluster[cite: 87, 93]
```

### 1. Non-Statistical (Non-Probability) Sampling

- ⤷ **Core Characteristic**: Elements are selected through non-random mechanisms where individual selection probabilities are unknown[cite: 87, 89]. These techniques are economical and rapid, but do not provide a mathematically rigorous basis for generalising to a population.
    
- ↳ **Methods**:
    
    - **Convenience Sampling**: Subjects are selected purely based on ease of physical or temporal access for the researcher (e.g., surveying peers walking through a university cafeteria)[cite: 87, 88].
        
    - **Judgmental / Purposive Sampling**: The researcher hand-picks participants who, in their subjective assessment, possess the specific profile or expertise required[cite: 87, 88].
        
    - **Snowball Sampling**: Initial participants refer and recruit further acquaintances into the study; useful when studying hard-to-reach or vulnerable populations[cite: 88].
        

### 2. Statistical (Probability) Sampling

- ⤷ **Core Characteristic**: Every individual unit in the population has a known, non-zero probability of being selected[cite: 89]. This eliminates systemic researcher bias and enables valid statistical inference.
    

```
┌────────────────────────────────────────────────────────────────────────┐
│                      PROBABILITY SAMPLING MECHANICS                    │
├──────────────────────────┬─────────────────────────────────────────────┤
│ Method                   │ Operational Execution                       │
├──────────────────────────┼─────────────────────────────────────────────┤
│ 1. Simple Random         │ Lottery selection / Random Number Generator │
│ 2. Stratified            │ Strata (Homogeneous) ──► Random from ALL    │
│ 3. Systematic            │ Interval k = N / n   ──► Select every k-th  │
│ 4. Cluster               │ Clusters (Heterogeneous) ──► ALL from SOME  │
└──────────────────────────┴─────────────────────────────────────────────┘
```

- ↳ **Method 1: Simple Random Sampling**
    
    - ↳ Every individual and every possible sample of size $n$ has an equal probability of selection.
        
    - ↳ Executed via certified random number tables, automated computer generators, or a physical lottery (names in a hat).
        
- ↳ **Method 2: Stratified Random Sampling**
    
    - ↳ The population is divided into mutually exclusive, internally homogeneous subgroups termed **strata**, stratified by a common attribute (e.g., gender, college major, income bracket).
        
    - ↳ An independent simple random sample is extracted from **every single stratum**, and these sub-samples are pooled to form the final sample.
        
- ↳ **Method 3: Systematic Random Sampling**
    
    - ↳ The population frame $N$ is placed into an ordered sequence (e.g., alphabetical, temporal, or spatial).
        
    - ↳ The sampling interval $k$ is calculated as:
        

$$k = \frac{N}{n}$$

- ↳ A random seed between $1$ and $k$ is selected as the initial unit; subsequently, **every $k$-th individual** is drawn until the target size $n$ is attained.
    
- ↳ **Method 4: Cluster Sampling**
    
    - ↳ The population is divided into diverse geographic or organizational units called **clusters**, which ideally reflect the variety of the overall population.
        
    - ↳ A simple random sample of **entire clusters** is selected; every element within the chosen clusters is measured.
        

## ⚖️ Strategic Comparison: Stratified Versus Cluster Sampling

> [!tip] 💡 Intuition: Cake Layers vs. Slices
> 
> - **Stratified Sampling** is like a layered cake: every horizontal layer (chocolate, vanilla, strawberry) is uniform. To sample the cake fairly, you take a bite from _every single layer_.
>     
> - **Cluster Sampling** is like cutting the entire cake into vertical slices: each slice contains _all_ the different layers inside it. To sample the cake efficiently, you pick a few entire slices and evaluate everything in them.
>     

|**Comparative Feature**|**Stratified Random Sampling**|**Cluster Sampling**|
|---|---|---|
|**Subgroup Composition**|**Homogeneous within** (members share identical traits)|**Heterogeneous within** (members reflect broad diversity)|
|**Between-Group Trait**|**Heterogeneous between** (strata differ distinctly from each other)|**Homogeneous between** (clusters resemble one another in structure)|
|**Selection Principle**|==Random individuals selected from **ALL** subgroups==|==**ALL** individuals measured from **SOME** randomly chosen subgroups==|
|**Boundary Generation**|Imposed deliberately by the researcher|Naturally occurring (e.g., geographic boundaries, buildings, schools)|
|**Primary Objective**|Maximise sample representativeness and statistical precision|Minimise administrative cost and travel/logistical friction|

## 📝 Practice Problem Solving & Classification Review

### Module 1: Descriptive Versus Inferential Practice

**Question 1**: "On average, 100 people choke to death on ballpoint pens every year." ✅ **Correct Answer: Option [A] — Descriptive Statistics**

- **Logical Justification**: The statement summarises historical, retrospective mortality records. It reports a recorded descriptive average without drawing inferences or projecting future trends.
    
- **Elimination Matrix**:
    
    - _Inferential Statistics_: Incorrect. The statement does not predict a future year or test a hypothesis; it summarises existing archival data.
        

**Question 2**: "The average person's left hand does 56% of the typing." ✅ **Correct Answer: Option [A] — Descriptive Statistics**

- **Logical Justification**: This claim presents a summary calculation derived from observed typing activity. It describes a recorded metric without testing a wider hypothesis.
    
- **Elimination Matrix**:
    
    - _Inferential Statistics_: Incorrect. It is an empirical summary percentage of recorded keystrokes, not a generalising projection.
        

**Question 3**: "By 2040 at least 3.5 billion people will run short of water." ✅ **Correct Answer: Option [B] — Inferential Statistics**

- **Logical Justification**: This statement looks forward to an unobserved future point in time. Projecting future demographic patterns from sample models requires inferential forecasting.
    
- **Elimination Matrix**:
    
    - _Descriptive Statistics_: Incorrect. One cannot describe events in 2040 before they happen; the data must be inferred.
        

**Question 4**: "Allergy therapy makes bees go away." ✅ **Correct Answer: Option [B] — Inferential Statistics**

- **Logical Justification**: The assertion claims a general causal relationship between an intervention and an outcome[cite: 57, 61]. Determining relationships and drawing broad conclusions from clinical trials is inferential.
    
- **Elimination Matrix**:
    
    - _Descriptive Statistics_: Incorrect. This is a general claim about treatment efficacy, not a summary of an observed sample.
        

**Question 5**: "Nerve impulses to and from the brain travel as fast as 170 miles per hour." ✅ **Correct Answer: Option [A] — Descriptive Statistics**

- **Logical Justification**: Reports an observed physiological measurement of transmission speed without generating forecasts or inferring broader relationships.
    
- **Elimination Matrix**:
    
    - _Inferential Statistics_: Incorrect. It reports a direct scientific measurement.
        

### Module 2: Sampling Methodology Identification

**Question 1**: "In a large school district, all teachers from two buildings are interviewed to determine whether they believe the students have less homework to do now than previous years." ✅ **Correct Answer: Option [D] — Cluster Sampling**

- **Logical Justification**: The school buildings represent naturally occurring, intact administrative clusters. The researcher selects two entire buildings at random and measures **all** teachers within them.
    
- **Elimination Matrix**:
    
    - _Simple Random_: Incorrect. Individual teachers were not chosen independently at random[cite: 90].
        
    - _Stratified_: Incorrect. Teachers were not sampled proportionally across every school building.
        
    - _Systematic_: Incorrect. There is no calculated fixed sampling interval $k$ applied across an ordered list.
        

**Question 2**: "The team needs to get a sample of 4000 students from the population and select 480 English, 1120 Science, 960 Computer Science, 840 Engineering, and 600 Math students which provides a better representation of students' college majors in U.S." ✅ **Correct Answer: Option [B] — Stratified Random Sampling**

- **Logical Justification**: The population is deliberately partitioned into mutually exclusive subgroups (academic majors), and independent samples are collected from **every single subgroup** to ensure representation.
    
- **Elimination Matrix**:
    
    - _Cluster_: Incorrect. The study does not sample a few majors and survey everyone in them; it samples across every major.
        
    - _Systematic_: Incorrect. Selection is based on categorical strata, not an interval step.
        
    - _Simple Random_: Incorrect. Purely random selection would not enforce exact numerical quotas by major.
        

**Question 3**: "Every 100th hamburger manufactured is checked to determine its fat content." ✅ **Correct Answer: Option [C] — Systematic Sampling**

- **Logical Justification**: The production line provides an ordered temporal sequence, and quality assurance inspects units using a fixed interval ($k = 100$).
    
- **Elimination Matrix**:
    
    - _Simple Random_: Incorrect. Units between the intervals have a $0\%$ chance of selection once the interval is established.
        
    - _Stratified_: Incorrect. Hamburgers are not grouped into pre-defined categories before sampling[cite: 91].
        
    - _Cluster_: Incorrect. Whole batches are not being selected in lumps[cite: 93].
        

**Question 4**: "Mail carriers of a large city are divided into four groups according to gender (male or female) and according to whether they walk or ride on their routes. Then 10 are selected from each group and interviewed to determine whether they have been bitten by a dog last year." ✅ **Correct Answer: Option [B] — Stratified Random Sampling**

- **Logical Justification**: The population is partitioned into four distinct strata based on two variables (gender and transit mode). A fixed sample size ($n=10$) is then drawn from **every single group**.
    
- **Elimination Matrix**:
    
    - _Cluster_: Incorrect. If it were cluster sampling, the researcher would pick one or two groups and interview every single carrier in them.
        
    - _Systematic_: Incorrect. There is no $k$-th interval stepping mechanism[cite: 92].
        
    - _Simple Random_: Incorrect. Unconstrained random selection could not guarantee equal representation across all four groups.