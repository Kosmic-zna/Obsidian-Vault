# 💻 Lesson 10/11: Systems Analysis, Software Development Life Cycle (SDLC), and Programming Foundations

---

## 🏛️ Foundational Concepts & The Role of a Developer

> [!info] 📖 Lexicon & First Principles
> - **Development**: The systematic end-to-end engineering lifecycle of conceiving, analysing, designing, implementing, securing, and deploying an information system, software application, or mobile utility from initial inception to end-user dissemination.
> - **Developer**: A software practitioner or engineer responsible for translating functional specifications and user requirements into secure, maintainable executable code and computing systems.
> - **Deliverable**: Any tangible, verifiable work product produced across the development trajectory (e.g., feasibility reports, entity-relationship diagrams, architectural schematics, interface mock-ups, source code files, test matrices).
> - **Documentation**: The consolidated repository of technical records, data dictionaries, operational summaries, and formal deliverables produced during a system’s development.

> [!bug] 🚨 Lecture Material Discrepancy: File Label vs. Slide Title
> The distributed file is marked `Lesson 10_Program Development.pdf`, yet internal slide headers are numbered `Lesson 11`. Verify with your instructor whether this material will be assessed under **Unit 10** or **Unit 11** in formal examinations.

- ⤷ **Three Cardinal Directives of Software Engineering**:
  - ↳ **Phase Partitioning**: Group discrete development efforts into structured, orderly phases (Planning, Analysis, Design, Implementation, Support/Security).
  - ↳ **Active User Engagement**: Directly involve the end-users for whom the software is engineered at every critical stage to prevent usability divergence.
  - ↳ **Standardisation**: Define and enforce clear engineering standards—systematic rules, linting configurations, and coding protocols—to ensure structural consistency and architectural maintainability.

- ⤷ **Catalysts for System Inception**:
  - ↳ **Problem Remediation**: Mitigating catastrophic operational defects (e.g., computational algorithmic flaws, data corruption bugs, system security breaches).
  - ↳ **System Enhancement**: Modernising legacy hardware, refactoring obsolete software dependencies, or optimising processing performance.
  - ↳ **Project Request**: The formalised document (also termed a *Request for System Services*) submitted by a user or department initiating the development cycle; constitutes the ==initial item of documentation== for the prospective project.

---

## ⚖️ Project Governance & Scope Management

> [!info] 📖 Lexicon & First Principles
> - **Project Scope**: The clearly bounded operational envelope defining the specific goals, functional capabilities, structural deadlines, and resource boundaries allocated to a project.
> - **Scope Creep (Feature Creep)**: The uncontrolled, unapproved expansion of a project's functional boundaries beyond its original charter, typically caused by gradual, unplanned feature additions that trigger schedule slippages and cost overruns.
> - **Change Management**: The formal, systematic discipline of identifying operational shifts, evaluating their impact, executing structured mitigations, and capitalising on strategic opportunities without derailing project constraints.

- ⤷ **Project Management Core Responsibilities**:
  - The process of planning, scheduling, and controlling development milestones to deliver an acceptable system on schedule and within budget.
  - ↳ **Five Governing Dimensions**:
    1. Defining strict ==Scope boundaries==.
    2. Enumerating ==Required activities== and functional work breakdown tasks.
    3. Formulating reliable ==Time and cost estimates== per subtask.
    4. Mapping the sequential ==Order of execution== (critical path dependencies).
    5. Pinpointing ==Concurrent activities== suitable for parallel execution.

---

### 📊 Scheduling Visualisation Tools: Gantt vs. PERT

| Visualisation Metric | 📅 Gantt Chart | 🕸️ PERT Chart (`Program Evaluation and Review Technique`) |
| :--- | :--- | :--- |
| **Structural Format** | Horizontal bar chart | Network diagram (nodes and directional vectors) |
| **Primary Axes** | Vertical: task activities; Horizontal: elapsed time | Graph topology: milestones (nodes) and dependencies (arrows) |
| **Core Analytic Strength** | Tracking concurrent progress and milestone timelines at a glance | ==Identifies the minimum completion time== and critical path bottlenecks |
| **Task Relationships** | Implied visually via bar overlaps | Explicitly mapped via directional mathematical vectors |

```
[ GANTT CHART REPRESENTATION ]
Task A: [========]
Task B:          [==============]
Task C:                  [==============]
Time   ----------------------------------->

[ PERT NETWORK REPRESENTATION ]
        +--> [ Task B ] --+
        |                 |
[ Start ]                 +--> [ Task D ] ---> [ End ]
        |                 |
        +--> [ Task C ] --+
```

---

## ⚙️ The Software Development Life Cycle (SDLC)

```
        ┌───────────────────────────────────────────────────┐
        │                                                   ▼
┌───────────────┐     ┌───────────────┐     ┌───────────────┐
│     PLAN      │ ──> │    ANALYSE    │ ──> │    DESIGN     │
└───────────────┘     └───────────────┘     └───────────────┘
        ▲                                           │
        │             ┌───────────────┐             │
        │             │   IMPLEMENT   │ <───────────┘
        │             └───────┬───────┘
        │                     │
        │                     ▼
        │             ┌───────────────┐
        └──────────── │SUPPORT/SECURE │
                      └───────────────┘
```

### 1. 📝 Phase 1: Planning
- ⤷ **Operational Trigger**: Initiated upon receipt of a formal project request; governed by an administrative steering committee.
- ⤷ **Four Foundational Mandates**:
  1. Review and formally approve/reject project proposals.
  2. Prioritise incoming institutional requests against enterprise strategy.
  3. Allocate corporate resources (budgetary capital, engineering personnel, infrastructural hardware).
  4. Form the core cross-functional project development team.

---

### 2. 🔍 Phase 2: Analysis

- ⤷ **Two Primary Operations**:
  - ↳ **Preliminary Investigation (Feasibility Study)**: A rapid evaluation undertaken to determine whether the project proposal warrants active resource expenditure.
  - ↳ **Detailed Analysis**: Comprehensive technical decomposition culminating in the publication of a formal *System Proposal*.

> [!check] 🔢 The Four Pillars of Feasibility Analysis
> - **Operational Feasibility**: Measures whether the completed product will function effectively within the organisation and ==satisfy specific end-user demands==.
> - **Schedule Feasibility**: Verifies whether projected deadline milestones are mathematically and operationally realistic.
> - **Technical Feasibility**: Evaluates whether internal developers possess the technical competencies, software toolchains, and hardware components to execute the system.
> - **Economic Feasibility**: Evaluates whether aggregate projected business benefits ==outweigh total development and lifecycle support costs== (Cost-Benefit Analysis).

- ⤷ **Information-Gathering Modalities**:
  - ↳ **Review Documentation**: Scrutinising existing procedural manuals, architectural blueprints, data logs, and defect reports.
  - ↳ **Observation**: Visually evaluating operators within their native work environment to identify unstated operational bottlenecks.
  - ↳ **Surveys / Questionnaires**: Aggregating broad, quantifiable feedback across extensive user demographics.
  - ↳ **Interviews**: Structured, deep-dive discussions with selected domain stakeholders.
  - ↳ **JAD Sessions (Joint Application Development)**: Intensive, collaborative cross-functional workshops uniting systems analysts, developers, management, and end-users to fast-track requirements gathering.

---

### 3. 📐 Phase 3: Design
- ⤷ **Core Objective**: Transform vetted functional requirements into precise technical execution blueprints; acquire supporting hardware and software suites.
- ⤷ **Hardware/Software Procurement Workflow** *(Executed if new tooling is required)*:
  1. Identify exact technical and architectural specifications.
  2. Solicit vendor proposals via requests for proposals (RFPs).
  3. Rigorously evaluate and stress-test vendor candidate systems.
  4. Make an objective procurement determination.
- ⤷ **Interface & Architectural Prototyping**: Generating mock-up user interfaces (UI), technical wireframes, database schemas, and early software prototypes.

---

### 4. 🚀 Phase 4: Implementation
- ⤷ **Core Objective**: Construct, validate, and transition the working system into operational production.
- ⤷ **Three Cardinal Activities**:
  - ↳ **Development**: Authoring production source code using select programming toolchains and target languages.
  - ↳ **Installation & Verification Testing**: Verifying discrete components, integration pipelines, and complete system performance.
  - ↳ **User Training**: Educating end-users via group seminars, individual coaching, interactive web tutorials, and comprehensive manuals.

---

### 5. 🛡️ Phase 5: Support and Security
- ⤷ **Core Objective**: Continuous post-deployment optimization, bug mitigation, vulnerability remediation, and service uptime maintenance.
- ⤷ **Continuous Quality Assurance (QA)**: QA engineers continuously audit functionality, report regressions, and validate defensive integrity against emergent vulnerabilities.

---

## 🔄 SDLC Methodologies: Predictive vs. Adaptive

```
[ WATERFALL: Linear / Sequential ]
Plan ──> Analyse ──> Design ──> Implement ──> Support & Secure

[ AGILE: Iterative / Incremental ]
Sprint 1: [ Plan -> Analyse -> Design -> Implement -> Secure ] ──> Working Subsystem
Sprint 2: [ Plan -> Analyse -> Design -> Implement -> Secure ] ──> Added Features
Sprint 3: [ Plan -> Analyse -> Design -> Implement -> Secure ] ──> Polished Release
```

### 💧 Predictive Development (Waterfall Model)
- ⤷ **Operational Mechanism**: Strict, sequential, linear progression. Each SDLC phase must achieve complete finalisation and sign-off before downstream stages commence.
- ⤷ **Strategic Drawback**: Highly inflexible; poorly suited to dynamic business environments where requirements evolve midway through the lifecycle.

### 🏃 Adaptive Development (Agile Methodology)
- ⤷ **Operational Mechanism**: Highly flexible, iterative framework that decomposes the macro-project into rapid development sprints.
- ⤷ **Strategic Advantage**: Delivers working functional increments continuously, allowing teams to ==absorb user feedback and adjust features dynamically== throughout development.

---

## 👥 Engineering Roles & Responsibilities

| Technical Role | Dedicated Operational Mandates |
| :--- | :--- |
| **Project Manager** | Orchestrates project components; arbitrates task allocation; balances the budget, timeline, and scope; liaises with senior corporate leadership. |
| **Systems Analyst** | Serves as the central bridge between enterprise users and technical engineers; analyses workflows; designs information system specifications. |
| **Designer (UI/UX)** | Architects visual styling, user interface interactions, visual hierarchy, typography palettes, and layout ergonomic standards. |
| **Programmer / Developer** | Translates technical blueprints and architectural specifications into clean, executable source code using development frameworks. |
| **Quality Assurance (QA) Tester** | Systematically probes applications to identify functional failures, defects, edge-case breakdowns, and security vulnerabilities. |
| **IT Support Specialist** | Serves as the customer-facing technical tier; triages post-deployment software malfunctions, operational defects, and end-user operational inquiries. |

---

## 💻 Programming Languages & Translation Engines

### ⏳ Historical Generations of Languages

- ⤷ **First-Generation Language ($1\text{GL}$): Machine Language**:
  - The native, low-level binary dialect understood directly by a computer's Central Processing Unit (CPU).
  - Consists entirely of primitive streams of ==binary digits ($0\text{s}$ and $1\text{s}$)==. Highly tedious, machine-dependent, and error-prone for humans to write.
- ⤷ **Second-Generation Language ($2\text{GL}$): Assembly Language**:
  - Low-level programming abstraction replacing raw binary opcodes with human-readable mnemonics (e.g., `MOV`, `ADD`, `JMP`).
  - Requires an *assembler* to translate symbolic instructions into target machine code.

---

### ⚙️ The Toolchain: Editors, Translators, and Environments

> [!info] 📖 Lexicon & First Principles
> - **Source Code Editor**: A specialized text editor designed specifically for writing computer programs, equipped with syntax highlighting, auto-completion, bracket-matching, and linting.
> - **Debugger**: A utility enabling developers to execute code instruction-by-instruction, inspect stack frames and variables, capture runtime exceptions, and diagnose logic errors.
> - **SDK (Software Development Kit)**: An expansive software package consolidating compilers, debuggers, libraries, APIs, and documentation required to build applications for a specific platform or OS.
> - **Code Repository**: A centralized, version-controlled web platform (e.g., GitHub, GitLab) engineered to host, track, and collaborate on source code trees.
> - **IDE (Integrated Development Environment)**: A unified software suite integrating a source editor, compiler/interpreter, and interactive debugger into a single graphical user interface.

```
+-----------------------------------------------------------+
|         INTEGRATED DEVELOPMENT ENVIRONMENT (IDE)          |
|                                                           |
|  [ Source Code Editor ]       [ Interactive Debugger ]     |
|   - Syntax Highlighting        - Breakpoints              |
|   - Auto-Completion            - Memory / Variable Stack  |
|                                                           |
|  [ Translation Engine (Compiler or Interpreter) ]         |
|  [ Built-in Software Development Kit (SDK) Toolchains ]   |
+-----------------------------------------------------------+
```

---

### 🔄 Program Translation: Compilers vs. Interpreters

> [!warning] ⚠️ Critical Exam Distinction: Compilation vs. Interpretation

```
[ COMPILATION ]
Source Code (Entire Program) ───[ Compiler ]───> Machine Code (Executable) ───> Instant CPU Execution

[ INTERPRETATION ]
Line 1 ───[ Interpreter ]───> Execute Instantly
Line 2 ───[ Interpreter ]───> Execute Instantly
Line 3 ───[ Interpreter ]───> Halts on Error!
```

- ⤷ **Compiler**:
  - Translates the entire high-level source program into machine language ==in a single comprehensive pass before execution==, yielding a standalone executable binary file (`.exe`).
  - Execution yields rapid runtime performance; initial compilation pass requires dedicated build time.

- ⤷ **Interpreter**:
  - Translates and immediately executes source code instructions ==sequentially, one line at a time==.
  - Translates on the fly; execution halts immediately upon encountering a syntax or runtime error. Facilitates rapid, ad-hoc debugging but produces slower runtime performance.

---

## 🧱 Object-Oriented Programming (OOP) Paradigm

> [!info] 📖 Lexicon & First Principles
> - **Object**: A self-contained programmatic entity encapsulating both internal operational data (attributes/properties) and the procedural routines that manipulate that data (methods).
> - **Class**: The abstract blueprint or template defining the structure, attributes, and available actions characterizing an object category.
> - **Method**: An actionable function or algorithmic procedure embedded within a class that governs the dynamic behavior of an instantiated object.

> [!tip] 💡 Intuition & Mental Model: The Architectural Analogy
> Consider a factory architectural blueprint for a performance sports car.
> - The **Class** is the *blueprint* itself: it specifies that any car built must possess properties (paint colour, engine capacity) and operational actions (accelerate, brake).
> - The **Object** is the *physical vehicle* driven off the assembly line: an instantiated instance possessing concrete values (e.g., `colour = "Red"`, `fuel = 85%`).
> - The **Method** is the *mechanical action* invoked: executing `accelerate()` reads current fuel reserves, executes internal combustion logic, and updates vehicular speed.

```
+--------------------------------------------------------+
|                      CLASS: Vehicle                    |
|--------------------------------------------------------|
| - [Data / Attributes]: colour, speed, fuel_capacity    |
|--------------------------------------------------------|
| + [Methods / Procedures]: accelerate(), brake(), turn()|
+--------------------------------------------------------+
                           │
             Instantiates concrete instances
                           │
                           ▼
+--------------------------------------------------------+
|                     OBJECT: car1                       |
|--------------------------------------------------------|
|   colour = "Red"                                       |
|   speed  = 60 mph                                      |
|   fuel_capacity = 95%                                  |
+--------------------------------------------------------+
```

---

## 🧪 Comprehensive Software Testing Matrices

### 🔬 Functional Testing Taxonomy

```
Unit Testing (Discrete Modules)
      │
      ▼
System Testing (All Modules Assembled)
      │
      ▼
Integration Testing (Bridging to External Applications)
      │
      ▼
Acceptance Testing (End-User Validation with Live Data)
```

- ⤷ **Unit Testing**: Isolated verification assessing that an individual program unit, method, or class functions correctly independently.
- ⤷ **System Testing**: Validates that all interconnected modules across the broader software program communicate and function seamlessly together.
- ⤷ **Integration Testing**: Verifies that the completed application interfaces correctly with distinct external systems, databases, and third-party APIs.
- ⤷ **Acceptance Testing**: The final pre-release validation conducted by real-world end-users to ensure the system operates predictably when handling live production data.

---

### 🛡️ Security Vulnerability Testing

- ⤷ **Security Audit**: A systematic inspection and assessment of an organization’s software codebase and structural architecture against established regulatory compliance frameworks and baseline security policies.
- ⤷ **Penetration Testing ("Ethical Hacking")**: An authorized, deliberate simulation of hostile cyberattacks designed to identify exploitable perimeter vulnerabilities, weak points, and security misconfigurations.
- ⤷ **Fuzz Testing (Fuzzing)**: An automated testing technique that inputs massive volumes of ==malformed, pseudo-random, or unexpected payload data== into application entry points to trigger buffer overflows, unhandled exceptions, and memory crashes.

---

## 🏪 App Store Deployment, IP, & Monetisation

### 📜 Intellectual Property (IP) Considerations
- ⤷ **Legal Safeguards**: Applications constitute proprietary intellectual assets. Developers must formalise software licensing models (e.g., Proprietary Commercial vs. Open Source), register trademarks, secure source copyright protections, and purchase domain names for distribution and customer support.

### 📱 App Store Distribution Protocol
1. **Account Registration**: Complete commercial onboarding with target app distribution platforms (e.g., Apple App Store, Google Play), which typically requires a nominal account configuration or annual developer fee.
2. **Metadata & Asset Assembly**: Construct the marketing listing containing the application title, visual icon, promotional screenshots, feature write-ups, and demonstration videos.
3. **Regulatory Classification Questionnaire**: Complete the statutory platform intake questionnaire regarding content maturity, target demographic disclosures, and privacy metrics to establish official app store ratings and user review collection.
4. **Post-Launch Maintenance**: Monitor user ratings and customer feedback continuously; patch reported functional errors through updates, and execute digital advertising campaigns across targeted social channels.

---

## ⚖️ Ethical Design & User Privacy

> [!info] 📖 Lexicon & First Principles
> - **Ethical Design**: The deliberate philosophy of architecting software interfaces and data pipelines that prioritise transparent data governance, elevate user experience (UX), and safeguard personal privacy rights by default.

- ⤷ **Core Tenets of Ethical Software Architecture**:
  - ↳ **Zero Obfuscation**: Never hide, camouflage, or obscure privacy controls, terms of service, cookie permissions, or location-tracking toggles.
  - ↳ **Accessible Privacy Declarations**: Provide prominent, understandable security charters and implement active mechanisms to notify users whenever privacy policies are amended.
  - ↳ **Data Minimisation**: Prohibit unnecessary telemetry; ==never harvest more user information than the absolute bare minimum== required to provide service functionality.
  - ↳ **Explicit Notification Authorisation**: Require affirmative opt-in consent before issuing outbound notifications.
  - ↳ **Ad Segmentation**: Establish crisp, unmistakable visual distinctions separating commercial advertisements from native application content.

> [!check] 🔢 Inbound vs. Outbound Notifications
> - **Push Notification**: An unprompted, proactive alert pushed to a user's device screen, SMS, or email by a remote server (e.g., alerts advertising an available game booster or feature update).
> - **Pull Notification**: An information transfer initiated in direct response to a conscious query or deliberate action executed by the local user.

---

## 🧮 Computational Problem Solving & Algorithmic Logic

> [!info] 📖 Lexicon & First Principles
> - **Algorithm**: A finite, unambiguous sequence of computational instructions and rules designed to execute a specific calculation, process data inputs, and generate predictable outputs.
> - **Binary System**: The base-2 positional numeral system using exclusively digits $0$ and $1$ to represent all computational memory registers and internal data payloads.
> - **Variable**: A designated, named memory location configured to store variable values during program execution.

- ⤷ **Three Fundamental Control Structures of Imperative Programming**:

### 1. 🔀 Sequencing
- ⤷ **Definition**: The straightforward, linear execution of computational commands completed consecutively in the exact order written.

### 2. 🔀 Selection (Conditional Logic)
- ⤷ **Definition**: Branching logic that evaluates boolean propositions to divert execution paths.
- ⤷ **Mechanism**: Direct programmatic evaluation where a condition yields strictly `True` or `False`.
  - `If [Condition] Then [Action A] Else [Action B] End`

```
           [ Condition Evaluated ]
                /          \
        (True) /            \ (False)
              ▼              ▼
       [ Execute Branch A ] [ Execute Branch B ]
              \              /
               \            /
                ▼          ▼
            [ Continue Program ]
```

### 3. 🔁 Iteration (Repetition / Loops)
- ⤷ **Definition**: The systematic repetition of a designated code block until a boundary condition is satisfied.
- ⤷ **Operational Triggers**:
  - Loops executed continually until a specific conditional state is triggered.
  - Loops configured to repeat for a fixed numeric count.
  - Enumeration loops iterating sequentially through all elements within an indexed data structure or array.

---

## 💡 High-Yield Review & Rapid Exam Recall

> [!warning] ⚠️ Critical Distinctions for the Exam Hall
> 1. **Compiler vs. Interpreter**: Compilers translate an *entire* program into standalone machine code before runtime; Interpreters process and execute code *line-by-line* on the fly, immediately halting upon errors.
> 2. **Gantt vs. PERT**: A Gantt chart maps schedule progress using horizontal bars over a calendar timeline; a PERT chart models tasks as a network graph to pinpoint dependencies and determine the *minimum time needed for project completion*.
> 3. **Push vs. Pull Notifications**: Push notifications are delivered proactively from the server without explicit user intervention; Pull notifications return data exclusively when requested by user actions.
> 4. **Unit vs. Integration vs. Acceptance Testing**: Unit tests check *individual methods/classes*; Integration tests ensure *multiple systems interact cleanly*; Acceptance tests are conducted by *real users handling production data*.
> 5. **Scope Creep vs. Change Management**: Scope creep is the *unplanned, informal swelling* of project requirements; Change Management is the *formal, controlled process* of reviewing, adopting, and balancing project changes.