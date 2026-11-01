# ⚖️ Module X: Formal Game Design Theory, Systems Balancing & Game Economies

> [!info] 📖 Lexicon & First Principles: Ludological & Systems Engineering
> Software architecture provides the functional skeleton; **Game Design** is the mathematical and psychological engineering of player agency, cognitive load, systemic feedback, and economic equilibrium:
> - ⤷ **Ludology**: The formal academic study of games, rule systems, and play activities, distinct from narrative or visual media analysis.
> - ⤷ **MDA Framework (Mechanics, Dynamics, Aesthetics)**: A formal post-mortem and design framework establishing that rules (**Mechanics**) produce runtime systemic interactions (**Dynamics**), which evoke subjective emotional responses (**Aesthetics**) in the human player.
> - ⤷ **Intransitive Dynamic (Cyclical Dominance)**: A relational loop where no single element is universally superior ($A > B > C > A$, typified by Rock-Paper-Scissors), precluding degenerative dominant tactics.
> - ⤷ **Mudflation**: The runaway economic devaluation of currency, equipment, or statistical power caused by continuous, unthrottled generation without commensurate, permanent sinks.
> - ⤷ **Pseudo-Random Distribution (PRD)**: A statistical algorithm that alters event probabilities dynamically on consecutive successes or failures to minimize variance, aligning true mathematical odds with human cognitive perception.
> - ⤷ **Hitstop (Freeze-Frame)**: The deliberate temporal freezing of character kinematics and animation updates for several frames ($50\text{--}100\text{ ms}$) upon verified weapon impact, imparting psychological weight and kinetic tactile feedback ("juice").

---

## 10.1 The MDA Framework & Pacing Architecture

A common failure mode in game development is designing backwards—beginning with a disconnected visual theme or narrative premise and grafting mechanical rules onto it. The **MDA Framework** formalizes the architectural divide between developer intent and player perception:

```
                         THE MDA PERSPECTIVE PARADOX
=================================================================================
 DESIGNER PERSPECTIVE (Creation Pipeline):
 [ MECHANICS ] ──────────► [ DYNAMICS ] ──────────► [ AESTHETICS ]
 Concrete Rules            Runtime System           Emotional & Cognitive
 & GDScript Code           Interactions             Player Experience
       ▲                                                    │
       │                                                    ▼
 PLAYER PERSPECTIVE (Experiential Consumption):
 (Players consume Aesthetics FIRST, deduce Dynamics, and rarely see Mechanics directly!)
```

### The Taxonomy of Formal Aesthetics (Hunicke, LeBlanc, Zubek)

When architecting a game, define its target aesthetic profile from the eight foundational ludic emotional drivers:

1. **Sensation**: Play as sensory pleasure (tactile kinematics, punchy soundscapes, screen trauma).
2. **Fantasy**: Play as make-believe (world immersion, atmospheric coherence).
3. **Narrative**: Play as drama (dynamic quest state machines, dramatic pacing curves).
4. **Challenge**: Play as obstacle mastery (high skill ceiling, tight hitboxes, rigorous combat mechanics).
5. **Fellowship**: Play as social framework (multiplayer coordination, squad roles, cooperative dependency).
6. **Discovery**: Play as uncharted territory (procedural generation, branching world streaming, hidden mechanics).
7. **Expression**: Play as self-discovery (custom loadouts, sandbox crafting, aesthetic customization).
8. **Submission**: Play as pastime (compulsion loops, idle accumulation, ritualistic progression).

---

## 10.2 Compulsion Loops & Flow Channel Calibration

A game's systemic retention relies on **Nested Compulsion Loops** operating across different temporal scales:

```
                         NESTED COMPULSION ARCHITECTURE
=================================================================================
 [ 1. MICRO LOOP (200ms - 5s) ]
 Tactical Kinematics: Aim ──► Discharge Weapon ──► Hitstop / Recoil ──► Dash
               │
               ▼
 [ 2. MESO LOOP (3m - 20m) ]
 Encounter Strategy: Clear Room ──► Collect Loot ──► Resolve Objectives ──► Re-equip
               │
               ▼
 [ 3. MACRO LOOP (2h - 50h+) ]
 Long-Term Mastery: Unlock Skill DAG ──► Optimize Meta-Build ──► Defeat Boss ──► NG+
```

### Csikszentmihalyi’s Flow Channel & Dynamic Tension

To keep a player engaged, the difficulty of the game must scale in lockstep with their developing mental model and mechanical dexterity. Deviations from this corridor result in psychological failure states:

```
                      THE MENTAL FLOW EQUILIBRIUM
=================================================================================
 Challenge (Difficulty)
    ▲                                    / ANXIETY & FRUSTRATION ZONE
    │                                  /  (Task exceeds player capacity)
    │                                / 
    │                     ╭─────────╯
    │                   ╭─╯  THE FLOW CHANNEL
    │                 ╭─╯    (Optimal Engagement Corridor)
    │               ╭─╯ 
    │             ╭─╯   
    │           ╭─╯    BOREDOM & APATHY ZONE
    │         ╭─╯      (Player mastery renders task trivial)
    └─────────┴──────────────────────────────────────────────────────►
   Low                                                             High
                             Player Skill / Competence
```

#### The Sawtooth Pacing Curve

Never subject a player to uninterrupted, escalating difficulty. The human sympathetic nervous system experiences fatigue under sustained cognitive stress. Professional pacing introduces an oscillatory **Sawtooth Wave**:

```
 Tension / Cognitive Load
    ▲             Peak Encounter (Boss)
    │                 ▲
    │        ▲       / │
    │       / \     /  │ (Cathartic Valley: Safe Room / Merchant / Dialogue)
    │  ▲   /   \   /   └───┐
    │ / \ /     \ /        │
    └─┴───┴───────┴────────┴─────────────────────────────────────────► Time (t)
```

---

## 10.3 Cybernetic Feedback Loops: Amplification vs. Homeostasis

Game systems are cybernetic machines governed by closed-loop information circuits:

```
                     CYBERNETIC FEEDBACK SYSTEM TOPOLOGY
=================================================================================
 POSITIVE FEEDBACK (Destabilizing / Snowballing):
 [ Win Combat ] ──► [ Extra Gold / XP ] ──► [ Superior Stats ] ──► [ Easier Wins ]
       ▲                                                                 │
       └─────────────────────────────────────────────────────────────────┘
 (Amplifies initial advantage; shortens match duration; risks early resignations)

 NEGATIVE FEEDBACK (Homeostatic / Rubber-Banding):
 [ Leader Pulls Ahead ] ──► [ Higher Maintenance / Bounties ] ──► [ Catch-up Windows ]
       ▲                                                                 │
       └─────────────────────────────────────────────────────────────────┘
 (Dampens divergence; preserves tension; risks frustrating high-skill players)
```

### 1. Positive Feedback (Reinforcing / Snowballing)
- ⤷ **Mathematical Behavior**: Divergent ($\frac{dy}{dt} = +k \cdot y$).
- ↳ **Function**: Rewards skillful execution with compounding advantages, accelerating resolution.
- ↳ **Pathology**: The **"Rich Get Richer"** trap. If an RTS player loses their first squad and falls behind economically, the winner's output grows exponentially, making the remainder of the match a drawn-out formality.

### 2. Negative Feedback (Self-Correcting / Homeostatic)
- ⤷ **Mathematical Behavior**: Convergent ($\frac{dy}{dt} = -k \cdot (y - y_{\text{target}})$).
- ↳ **Function**: Preserves contest parity by penalizing leaders (e.g., death-bounties in MOBAs, dynamic rubber-banding in arcade racing).
- ↳ **Pathology**: If over-tuned, skilled players feel their effort is invalidated because early leads are artificially erased.

---

## 10.4 Combat Mathematics: EHP, DPS, TTK & Diminishing Returns

Amateur combat design frequently balances characters by tweaking arbitrary numbers until "it feels okay." Systems architects balance combat through **Kinematic and Statistical Invariants**.

```
                   THE TIME-TO-KILL (TTK) RELATIONSHIP
=================================================================================
                 Target Effective Health Pool (EHP)
  TTK (seconds) = ──────────────────────────────────
                   Attacker Sustained Damage (DPS)
```

### 1. Effective Health Pool ($EHP$) & Non-Linear Armor Scaling

When calculating damage reduction from armor ($A$), a linear formula ($\text{Damage} = \text{Raw} - A$) is fundamentally flawed: high-rate-of-fire weapons become useless against moderate armor, while heavy-hitting sniper weapons ignore it entirely.

Instead, implement the classic **Diminishing Returns Hyperbolic Curve**:

$$\text{Damage Multiplier} = \frac{K}{K + A} \quad \implies \quad \text{Damage Taken} = \text{Raw Damage} \times \left(\frac{K}{K + A}\right)$$

where $K$ is the design constant (typically $K = 100$). Under this formulation, **every point of armor provides a constant, linear increase to Effective Health Pool ($EHP$)**:

$$EHP = \frac{\text{Base HP}}{\text{Damage Multiplier}} = \text{Base HP} \times \left(1.0 + \frac{A}{K}\right)$$

```
                         ARMOR MITIGATION BEHAVIOR (K = 100)
┌───────────────┬──────────────────────┬──────────────────────┬───────────────────┐
│ Armor Value A │ Damage Taken Multi   │ Damage Reduction %   │ Relative EHP      │
├───────────────┼──────────────────────┼──────────────────────┼───────────────────┤
│ 0             │ 100 / (100 + 0) = 1.0│ 0.0%                 │ 1.00x Base HP     │
│ 50            │ 100 / 150 = 0.6667   │ 33.3%                │ 1.50x Base HP     │
│ 100           │ 100 / 200 = 0.5000   │ 50.0%                │ 2.00x Base HP     │
│ 200           │ 100 / 300 = 0.3333   │ 66.7%                │ 3.00x Base HP     │
│ 300           │ 100 / 400 = 0.2500   │ 75.0%                │ 4.00x Base HP     │
└───────────────┴──────────────────────┴──────────────────────┴───────────────────┘
Notice the elegance: Each 100 points of Armor adds EXACTLY +100% of Base HP to EHP!
```

### 2. Sustained vs. Burst Damage per Second ($DPS$)

Sustained damage accounts for magazine capacity, fire rate, reload durations, and critical strike profiles:

$$\text{DPS}_{\text{sustained}} = \frac{D_{\text{bullet}} \times N_{\text{mag}} \times \left(1.0 + P_{\text{crit}} \cdot (M_{\text{crit}} - 1.0)\right)}{\left(\frac{N_{\text{mag}}}{R_{\text{fire}}}\right) + T_{\text{reload}}}$$

where:
- $D_{\text{bullet}}$: Base damage per projectile.
- $N_{\text{mag}}$: Magazine round capacity.
- $R_{\text{fire}}$: Fire rate in rounds per second ($\text{Hz}$).
- $T_{\text{reload}}$: Duration of reload sequence in seconds.
- $P_{\text{crit}}$: Critical hit probability $\in [0.0, 1.0]$.
- $M_{\text{crit}}$: Critical hit damage multiplier (e.g., $2.0\times$).

```gdscript
# ==============================================================================
# COMBAT METRIC SOLVER & TIME-TO-KILL (TTK) ANALYSER
# ==============================================================================
class_name CombatAnalytics
extends RefCounted

## Calculates the exact Effective Health Pool (EHP) given Base HP and Armor
static func compute_ehp(base_hp: float, armor: float, k_constant: float = 100.0) -> float:
    assert(armor >= 0.0, "Armor rating cannot be negative.")
    return base_hp * (1.0 + (armor / k_constant))

## Calculates sustained DPS factoring in reload cycles and critical hit expectations
static func compute_sustained_dps(
    bullet_dmg: float,
    mag_size: int,
    fire_rate_hz: float,
    reload_time: float,
    crit_chance: float,
    crit_mult: float
) -> float:
    assert(mag_size > 0 and fire_rate_hz > 0.0, "Invalid weapon parameters.")
    var cycle_firing_time: float = float(mag_size) / fire_rate_hz
    var total_cycle_time: float = cycle_firing_time + reload_time
    
    var expected_crit_factor: float = 1.0 + (clampf(crit_chance, 0.0, 1.0) * (crit_mult - 1.0))
    var total_damage_per_mag: float = float(mag_size) * bullet_dmg * expected_crit_factor
    
    return total_damage_per_mag / total_cycle_time

## Computes precise TTK in seconds between an attacker profile and target profile
static func compute_ttk(
    target_hp: float,
    target_armor: float,
    bullet_dmg: float,
    mag_size: int,
    fire_rate_hz: float,
    reload_time: float,
    crit_chance: float = 0.0,
    crit_mult: float = 1.0
) -> float:
    var ehp: float = compute_ehp(target_hp, target_armor)
    var dps: float = compute_sustained_dps(bullet_dmg, mag_size, fire_rate_hz, reload_time, crit_chance, crit_mult)
    return ehp / maxf(0.0001, dps)
```

---

## 10.5 Economic Engineering: Sources, Sinks, Faucets & Mudflation

Virtual economies in RPGs, MMOs, and survival games fail when treated like real-world national economies. In the real world, goods deteriorate and money circulates; in video games, **players extract currency from infinite voids and hoard it permanently**.

```
                       THE ECONOMIC EQUILIBRIUM TANK
=================================================================================
 FAUCETS / SOURCES (Infinite Spigots)
 [ Quest Rewards ]  [ Mob Loot Drops ]  [ Salvage Returns ]  [ Mining Nodes ]
        │                   │                   │                   │
        ▼                   ▼                   ▼                   ▼
 ═════════════════════════════════════════════════════════════════════════════
                       CIRCULATING PLAYER CURRENCY TANK
 ═════════════════════════════════════════════════════════════════════════════
        │                   │                   │                   │
        ▼                   ▼                   ▼                   ▼
 [ Repair Costs ]   [ Tax / Auction ]   [ Crafting Fees ]   [ Consumables ]
 SINKS / DRAINS (Permanent Destruction of Value)
```

### The Law of Economic Stability

Let $F(t)$ denote the total currency created across all faucets at time $t$, and $S(t)$ denote the currency permanently removed via sinks:

$$\Delta M = \int_{0}^{T} \left( \sum F_i(t) - \sum S_j(t) \right) dt$$

1. **If $\sum F > \sum S$ (Mudflation)**: The circulating supply of gold balloons. Common vendor items lose all psychological value. High-end player-to-player auction prices escalate to arbitrary caps, creating an insurmountable wealth barrier for novice players.
2. **If $\sum S > \sum F$ (Deflationary Starvation)**: Players cannot afford baseline repairs or core consumables, causing them to disengage from risk-bearing activities.
3. **The Target Invariant**: Design **Percentage-Based Dynamic Sinks** rather than flat numerical sinks:
   - Tax player-to-player trade via a flat percentage fee ($\text{Tax} = \text{Price} \times 0.08$).
   - Scale repair costs relative to character wealth or item rarity tiers ($C_{\text{repair}} \propto \text{Tier}^2$).

---

## 10.6 Probability Engineering: The Pseudo-Random Distribution (PRD)

Humans are notoriously poor at evaluating true mathematical independence. Under pure Independent Random Distribution (IRD) with $P = 20\%$:
- The chance of failing $6$ times in a row is $(0.8)^6 \approx 26.2\%$.
- The chance of failing $12$ times in a row is $(0.8)^{12} \approx 6.87\%$.

When a player misses a $20\%$ critical strike or dodge $12$ times in a row, they do not think *"Ah, a standard Poisson distribution variance!"* They perceive the game as broken or maliciously rigged.

```
                   PURE RANDOM (IRD) VS. PSEUDO-RANDOM (PRD)
=================================================================================
 Pure Random (Flat 20%):   [ O ][ O ][ O ][ O ][ O ][ O ][ O ][ O ][ O ][ O ][ X ] 
 (Extremely prone to dry spells and jarring multi-crit streaks!)

 Pseudo-Random (C = 0.055):[ 5.5% ][ 11% ][ 16.5% ][ 22% ][ 27.5% -> PROC! ]
 (Probability climbs with every failure; resets immediately after success!)
```

### The PRD Mathematical Formulation (Warcraft III / Dota 2 Standard)

Rather than checking against a flat probability $P$, PRD computes an escalating probability $P(N)$ where $N$ is the number of swings since the last successful event:

$$P(N) = N \times C$$

If the roll fails, $N$ increments by $1$, increasing the probability for the next attempt. When the roll succeeds, $N$ resets to $1$. 

The design constant $C$ is solved mathematically such that the **aggregate observed rate across infinite samples equals the nominal probability $P$**:

$$P_{\text{nominal}} \approx \sum_{N=1}^{N_{\text{max}}} P(\text{precisely on } N\text{th attempt}) = \sum_{N=1}^{N_{\text{max}}} N \cdot C \cdot \prod_{j=1}^{N-1} (1 - j \cdot C)$$

```gdscript
# ==============================================================================
# PSEUDO-RANDOM DISTRIBUTION (PRD) ENGINE
# ==============================================================================
class_name PseudoRandomDistribution
extends RefCounted

## Lookup table: Nominal Probability -> Optimal C Base Constant
## Derived from Monte Carlo simulation / Markov chain convergence
const C_LOOKUP: Dictionary = {
    0.05: 0.00380,
    0.10: 0.01475,
    0.15: 0.03222,
    0.20: 0.05564,
    0.25: 0.08474,
    0.30: 0.11939,
    0.35: 0.15962,
    0.40: 0.20541,
    0.45: 0.25686,
    0.50: 0.31382
}

var _nominal_probability: float
var _c_constant: float
var _fail_count: int = 1

func _init(nominal_prob: float = 0.20) -> void:
    set_probability(nominal_prob)

func set_probability(prob: float) -> void:
    _nominal_probability = clampf(prob, 0.01, 0.99)
    _c_constant = _approximate_c(_nominal_probability)
    _fail_count = 1

## Rolls for success using escalating PRD logic
func roll() -> bool:
    var current_chance: float = float(_fail_count) * _c_constant
    var sample: float = randf()
    
    if sample <= current_chance:
        _fail_count = 1 # Reset fail counter upon verified success
        return true
    else:
        _fail_count += 1 # Increment failure counter to escalate subsequent odds
        return false

func _approximate_c(p: float) -> float:
    # Snap to nearest discrete key in pre-computed table if available
    var rounded_key: float = snappedf(p, 0.05)
    if C_LOOKUP.has(rounded_key):
        return C_LOOKUP[rounded_key]
        
    # Heuristic approximation polynomial for arbitrary values: C ≈ 0.31 * P^1.8
    return 0.315 * pow(p, 1.82)
```

---

## 10.7 Weighted Loot Tables with Pity Bounds

Dropping randomized items requires balancing weighted item generation with deterministic bounds ("pity counters") to cap negative player streaks on legendary tier drops.

```
                      PREFIX-SUM ARRAY BINARY SEARCH
=================================================================================
 Items:      [ Common (60) ]  [ Uncommon (25) ]  [ Rare (12) ]  [ Mythic (3) ]
 Prefix Sum: 0 ──────────── 60 ──────────────── 85 ─────────── 97 ────────── 100
 Roll: 72.4  ──────────────────────────────────────► Falls in [60, 85) Interval!
 Result: UNCOMMON TIER! (Resolved in O(log N) via Binary Search!)
```

```gdscript
# ==============================================================================
# WEIGHTED DROP TABLE SYSTEM WITH HARD PITY ENGINE
# ==============================================================================
class_name LootTableSystem
extends RefCounted

class DropEntry:
    var item_resource: Resource
    var weight: float
    var is_legendary: bool
    
    func _init(res: Resource, w: float, legendary: bool = false) -> void:
        item_resource = res
        weight = w
        is_legendary = legendary

var _entries: Array[DropEntry] = []
var _prefix_sums: PackedFloat32Array = PackedFloat32Array()
var _total_weight: float = 0.0

# Pity Subsystem tracking
var rolls_since_last_legendary: int = 0
const HARD_PITY_THRESHOLD: int = 50

func add_entry(item: Resource, weight: float, is_legendary: bool = false) -> void:
    assert(weight > 0.0, "Drop weight must be positive.")
    var entry: DropEntry = DropEntry.new(item, weight, is_legendary)
    _entries.append(entry)
    _recalculate_prefix_sums()

func _recalculate_prefix_sums() -> void:
    _prefix_sums.clear()
    _total_weight = 0.0
    for entry in _entries:
        _total_weight += entry.weight
        _prefix_sums.append(_total_weight)

## Evaluates a weighted drop with O(log N) complexity and hard pity enforcement
func evaluate_drop() -> Resource:
    if _entries.is_empty():
        return null
        
    rolls_since_last_legendary += 1

    # 1. HARD PITY OVERRIDE: Force legendary drop if streak threshold is exceeded
    if rolls_since_last_legendary >= HARD_PITY_THRESHOLD:
        for entry in _entries:
            if entry.is_legendary:
                rolls_since_last_legendary = 0
                return entry.item_resource

    # 2. STANDARD WEIGHTED RANDOM DRAW
    var roll: float = randf_range(0.0, _total_weight)
    
    # Binary Search over Prefix-Sum array: O(log N)
    var low: int = 0
    var high: int = _prefix_sums.size() - 1
    var selected_idx: int = high
    
    while low <= high:
        var mid: int = (low + high) / 2
        if _prefix_sums[mid] >= roll:
            selected_idx = mid
            high = mid - 1 # Search left for lower boundary
        else:
            low = mid + 1

    var picked: DropEntry = _entries[selected_idx]
    if picked.is_legendary:
        rolls_since_last_legendary = 0
        
    return picked.item_resource
```

---

## 10.8 Tactile "Juice" & Kinesthetic Feedback

A mechanic can be mathematically balanced yet feel terrible to play. **"Juice"** (or Game Feel) is the non-functional audiovisual feedback that accentuates player actions.

```
                          THE KINESTHETIC HIT PIPELINE
=================================================================================
 Melee Sword Impacts Hurtbox!
           │
           ├── 1. HITSTOP FREEZE ──► Freeze actor frames for 60ms (Zero displacement)
           ├── 2. MESH JITTER   ──► Apply high-frequency noise offset to Victim
           ├── 3. AUDIO SYSTEM  ──► Play pitched impact SFX + Transient Sub-bass
           ├── 4. GPU SHADER    ──► Trigger 1-frame pure white Hit-Flash (EMISSION)
           └── 5. CAMERA RIG    ──► Inject +0.3 Trauma onto Camera (Rotational shake)
```

```gdscript
# ==============================================================================
# TACTILE GAME FEEL & HITSTOP ORCHESTRATOR
# ==============================================================================
class_name KinestheticFeedback
extends Node

## Executes multi-layered impact feedback across time-scale, mesh, and camera
static func trigger_impact(
    attacker: Node, 
    victim: Node, 
    camera_rig: CinematicCameraRig,
    hitstop_duration_sec: float = 0.06
) -> void:
    
    # 1. CAMERA TRAUMA INJECTION
    if camera_rig != null:
        camera_rig.add_trauma(0.35)

    # 2. TRANSIENT HIT-FLASH SHADER TRIGGER
    _apply_hit_flash(victim)

    # 3. KINEMATIC HITSTOP (Temporarily freezes processing without engine pausing)
    _execute_hitstop(attacker, victim, hitstop_duration_sec)

static func _apply_hit_flash(target: Node) -> void:
    var mesh: GeometryInstance3D = target.find_child("MeshInstance3D", true, false) as GeometryInstance3D
    if mesh != null and mesh.material_override != null:
        var mat: Material = mesh.material_override
        # Trigger hit-flash uniform parameter on shader
        mat.set("shader_parameter/flash_intensity", 1.0)
        
        # Smooth decay via SceneTree Tween
        var tween: Tween = target.get_tree().create_tween()
        tween.tween_property(mat, "shader_parameter/flash_intensity", 0.0, 0.15)

static func _execute_hitstop(attacker: Node, victim: Node, duration: float) -> void:
    # Disable process updates locally to preserve physical position without pausing the world
    attacker.set_physics_process(false)
    victim.set_physics_process(false)
    
    # Resume after asynchronous duration
    await attacker.get_tree().create_timer(duration, true, false, true).timeout
    
    if is_instance_valid(attacker):
        attacker.set_physics_process(true)
    if is_instance_valid(victim):
        victim.set_physics_process(true)
```

---

## 10.9 Applied Synthesis: The Complete Attribute Modifier & Combat Equation

To integrate combat invariants, hyperbolic mitigation, and deterministic modifiers, review the unified **Combat Resolution Pipeline**:

```
                       UNIFIED DAMAGE RESOLUTION PIPELINE
=================================================================================
 [ ATTACKER ]                                        [ VICTIM ]
 Base Weapon Damage: 40.0                            Base Armor Rating: 75.0
 Buff Pipeline (Additive + Mult)                     Equipment Armor Modifiers: +25.0
 ────────────────────────────────                    ────────────────────────────────
 Final Raw Damage: 58.0                              Total Armor: 100.0 (K = 100)
        │                                                   │
        ▼                                                   ▼
 Critical Hit Check (PRD)                            Hyperbolic Mitigation Factor
 (20% Nominal -> 5.5% Base C)                        Mitigation = 100 / (100 + 100) = 0.50
        │                                                   │
        └─────────────────────────┬─────────────────────────┘
                                  │
                                  ▼
                     [ RESOLVED EFFECTIVE DAMAGE ]
                     Damage = 58.0 * 0.50 = 29.0 HP
                                  │
                                  ▼
                     [ KINESTHETIC JUICE FLUSH ]
                     Hitstop (60ms) + Flash + Trauma (0.35)
```

---
