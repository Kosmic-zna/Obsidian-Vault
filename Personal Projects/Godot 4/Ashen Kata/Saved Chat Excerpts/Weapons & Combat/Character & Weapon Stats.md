> ✍️ **Phrasing Refinement:** "I do not wish to overcomplicate my game with millions of attributes requiring meticulous stat allocation or gear min-maxing; how can I design a progression system that is nearly level-only alongside streamlined weapon statistics—such as stance break and guard strength—offering meaningful variety without descending into _Elden Ring_'s labyrinth of stats and talismans?"
> 
>   

# ✅ Summary:

**Adopt a _Sekiro_-inspired "Unified Character Level + 4-Stat Weapon Identity" architecture.** Instead of scattering points across Strength, Dexterity, Arcane, and Faith, give the player **one single Character Level** (which linearly scales core Vitality, `Sha-Chi` Posture, and Spirit Seal capacity) while letting **weapons carry all build variety** through just **four clean, readable numbers** (`Slash Power`, `Stance Break`, `Guard Stability`, and `Skill Affinity`) plus **one passive weapon trait**—giving you crisp tactical variety without a single spreadsheet or talisman screen.

  

### 📖 Plain-English Glossary of Streamlined RPG Progression

- **Unified Level Progression (`Single-Track Scaling`):** A single level counter (e.g., Level `1` to `50`) where every level-up automatically increases your maximum HP, maximum `Sha-Chi` posture pool, and base attack multiplier—eliminating trap builds and stat-respec menus.
    
- **Weapon-Driven Build Identity (`Moveset + 4 Stats`):** Rather than changing your character's stats to fit a sword, every weapon (`Katana`, `Odachi`, `Twin Blades`, `Spear`) defines your playstyle immediately through its **attack speed, `X`/`C` weapon skills, and four clean combat ratings**.
    
- **Innate Weapon Passive (`1-Line Trait`):** Replacing _Elden Ring_'s four talisman slots with **one fixed passive perk baked directly into each weapon** (e.g., _"Clean deflects restore +15% Spirit"_ on a Katana, or _"Heavy finishers deal +30% Stance Break"_ on an Odachi), so switching weapons mid-combo (`Flux`) feels genuinely transformative.
    

### ⚖️ 1. The Minimalist Progression Blueprint (Zero Spreadsheet Bloat)

|**Progression Layer**|**What the Player Sees**|**How It Works Under the Hood (Godot 4 Resource)**|
|---|---|---|
|**1. Character Level** (`Lv. 1 → 50`)|**3 Core Bars:**<br><br>  <br>  <br><br>• **Vitality** (HP)<br><br>  <br>  <br><br>• **`Sha-Chi`** (Posture Capacity)<br><br>  <br>  <br><br>• **Spirit** (`X`/`C`/`F` Skill Charges)|Spending gathered Ash/XP at a shrine increments `character_level` by `1`, applying a clean curve (`base + level * growth`) across all three bars automatically.|
|**2. Weapon Upgrade** (`+0 → +10`)|**4 Clean Weapon Stats:**<br><br>  <br>  <br><br>• **Attack Power** (HP Damage)<br><br>  <br>  <br><br>• **Stance Break** (`Sha-Chi` Damage)<br><br>  <br>  <br><br>• **Guard Stability** (Posture lost on block)<br><br>  <br>  <br><br>• **Tempo / Weight** (`Fast`, `Balanced`, `Heavy`)|Upgrading a weapon with region-tier smithing stones scales its three numerical stats linearly while preserving its distinct tactical role.|
|**3. Weapon Trait & Skills**|**1 Innate Passive + `X` / `C` Skills**|Stored directly on `WeaponData.gd`—swapping weapons gives you new skills and a new passive perk without opening an accessory menu.|
|**4. Boss Relics** (`F` Slot)|**1 Equipped Boss Art**|Replaces all rings, talismans, and runes—you equip **one** boss power on `F` (e.g., _Ash-Lightning Plunge_ or _Twin Moon Slam_).|

### 🗡️ 2. How Just 4 Weapon Stats Create Real Tactical Variety

Even with only **four stats per weapon**, players make meaningful strategic choices before and during boss fights:

  

- **The Duelist Katana (`Fast`):** Moderate **Attack Power**, moderate **Stance Break**, high **Guard Stability** on timed deflects, and passive _"Deflects generate bonus Spirit"_ → _Best against hyper-aggressive humanoid duelists (`Onze` / `Crucible Commander`)._
    
- **The Executioner Odachi (`Heavy`):** High **Attack Power**, massive **Stance Break**, slower recovery, and passive _"Charged/Skill hits stagger through enemy light attacks"_ → _Best for cracking high-poise Black Knights and ankle-breaking giants (`Tower Knight`)._
    
- **The Twin Ash-Blades (`Rapid`):** Low **Guard Stability** (punishes sloppy blocking), rapid **Stance Break** accumulation through multi-hit `X`/`C` flurries, and passive _"Faster Ghoststep recovery"_ → _Best for aggressive hit-and-run spacing fights._
    

### 💻 3. Clean `WeaponData.gd` Resource Structure in Godot 4

You can store this entire streamlined weapon system inside a single lightweight Godot `Resource` script:

  

GDScript

```
class_name WeaponData
extends Resource

enum TempoClass { FAST, BALANCED, HEAVY }

@export_group("Identity")
@export var weapon_name: String = "Ash-Forged Katana"
@export var tempo: TempoClass = TempoClass.FAST
@export_multiline var innate_trait_description: String = "Clean deflections restore +15% bonus Spirit."

@export_group("Core Streamlined Stats (4 Numbers Only)")
@export var attack_power: float = 100.0       # Base HP damage dealt on hit
@export var stance_break: float = 25.0        # Base Sha-Chi posture damage dealt
@export var guard_stability: float = 65.0     # % of incoming posture damage negated on block
@export var deflect_spirit_gain: float = 1.15 # Multiplier for Spirit gained on clean parry

@export_group("Bound Weapon Skills (X & C)")
@export var skill_x: SkillData # Gap-closer / opener skill
@export var skill_c: SkillData # Heavy posture-breaker / chain finisher
```

### 🎯 Next Step Question

Would you like to **pair this `WeaponData` resource with a minimal `PlayerStats` autoload** (handling your single `character_level` and shrine upgrades), or jump straight into **building the shared `Combatant` / `EnemyBase` state machine** so your weapons can deal real `attack_power` and `stance_break` damage to a test dummy?

---

> ✍️ **Phrasing Refinement:** "How can I be certain these statistics are sufficient? Can we create dynamic boss variety using them, and what about character and enemy statistics?"
> 
>   

# ✅ Summary:

- **Yes—those four weapon numbers (`Slash Power`, `Stance Break`, `Guard Stability`, `Skill Affinity`) plus one passive trait are mathematically complete**, because every action in a _Sekiro_ / _Phantom Blade_-style swordfighter reduces to four fundamental variables: **health damage, posture damage, defensive mitigation, and skill-resource generation**.
    
- **How They Create Dynamic Boss Variety:** Instead of altering player spreadsheets, bosses test **different axes of those four stats**—such as high-poise knights who shrug off low `Stance Break`, relentless flurry duellists who shatter low `Guard Stability` if you block instead of deflecting, or agile monsters that starve low `Skill Affinity` builds of Spirit Seals.
    
- **Character & Enemy Stats (`Mirror Symmetry`):** Keep **both** the player and enemies on the exact same **four core combat stats** (`Vitality`, `Sha-Chi Posture`, `Poise`, and `Posture Recovery Rate`) so your shared `Combatant` state machine runs on one unified math formula.
    

### 📖 Plain-English Glossary of Minimalist Combat Math

- **Orthogonal Stats (`Zero Overlap`):** Four statistics are "orthogonal" when each controls a completely separate lever of combat (`HP`, `Posture`, `Defence`, `Resource`)—meaning no two stats do the same job, and removing any one of them leaves a noticeable mechanical gap.
    
- **`Poise` (`Hyper-Armour Threshold`):** A hidden or visible threshold on enemies and bosses determining how much `Stance Break` is required to flinch them mid-swing—differentiating a nimble _Onze_-style duellist (low poise, interrupts easily) from a _Black Knight_ or _Crucible Commander_ (high poise, powers through light slashes).
    
- **`Posture Recovery Rate` (`Sha-Chi Regeneration`):** How quickly an enemy's yellow/crimson posture bar drains when left unpressured—forcing aggressive, relentless swordplay against fast bosses whose `Sha-Chi` recovers rapidly at high health.
    

### ⚖️ 1. Why These 4 Weapon Stats Are Mathematically Complete

Every frame of melee combat in _Ashen Kata_ asks only four questions. If your weapon covers all four cleanly, adding more stats only creates UI clutter without adding real tactical depth:

  

|**Weapon Stat**|**Combat Question It Answers**|**What Happens When It Is High vs. Low**|
|---|---|---|
|**1. `Slash Power`**|_"How fast do I kill an enemy once I find an opening?"_|**High (`Odachi`):** Punishes brief boss recovery windows heavily.<br><br>  <br>  <br><br>**Low (`Twin Blades`):** Requires sustained combo pressure to chip HP down.|
|**2. `Stance Break`**|_"How fast do I fill the enemy's `Sha-Chi` bar and flinch them?"_|**High:** Interrupts mid-tier enemies (`Black Knights`) and topples giants (`Tower Knight`).<br><br>  <br>  <br><br>**Low:** Relies on clean deflections rather than raw strikes to break posture.|
|**3. `Guard Stability`**|_"How forgiving is an imperfect block when I mistime a deflect?"_|**High (`Katana` / `Greatsword`):** Protects your own posture during frantic 5-hit boss flurries.<br><br>  <br>  <br><br>**Low (`Twin Blades`):** Punishes passive guarding; forces _Ghoststep_ evasion or frame-perfect parries.|
|**4. `Skill Affinity`**|_"How fast do I charge Spirit Seals for `X`, `C`, and `F` Boss Arts?"_|**High:** Rewards aggressive deflection chains with frequent _Twin Moon_ or _Ash-Lightning_ casts.<br><br>  <br>  <br><br>**Low:** Trades frequent spectacle skills for raw physical `Slash Power`.|

### ⚔️ 2. How Bosses Dynamically Test Those Exact 4 Stats

You do not need elemental resistance tables (`Fire`, `Lightning`, `Bleed`, `Frost`) to force players to adapt their loadout or tactics across bosses:

  

1. **The Hyper-Aggressive Duelist (`Onze` / `Genichiro` Archetype):**
    
    - **Boss Profile:** Low `Poise` (flinches when interrupted), rapid `Posture Recovery Rate` at full HP, and 5-hit flurry combos.
        
    - **Stat Test:** Tests **`Guard Stability`** (surviving rapid parry strings) and **`Skill Affinity`** (converting multi-hit deflections into rapid `X`/`C` counter-skills before their posture recovers).
        
2. **The Heavy Armoured Commander (`Black Knight` / `Winged Crucible` Archetype):**
    
    - **Boss Profile:** High `Poise` (ignores light slashes during wind-ups), slower `Posture Recovery Rate`, and massive overhead cleaves.
        
    - **Stat Test:** Tests **`Stance Break`** (cracking their poise during delayed wind-ups) and **`Slash Power`** (capitalising on narrow punish windows after _Ghoststepping_ through unblockable wing/shoulder checks).
        
3. **The Colossal Juggernaut (`Tower Knight` / `Giant Lord` Archetype):**
    
    - **Boss Profile:** Massive `Vitality` pool, immune to standard flinching until ankle/wrist `Sha-Chi` threshold breaks.
        
    - **Stat Test:** Rewards high **`Stance Break`** weapons or heavy `F` Boss Arts (_Ash-Lightning Plunge_) to trigger a full-body posture collapse, followed by high **`Slash Power`** executions.
        

### 🧬 3. Streamlined Character & Enemy Stats (`CombatantStats` Mirror)

Keep **Player**, **Small Mobs**, **Medium Knights**, and **Bosses** on the **exact same 4-stat `CombatantStats` structure** so your damage and posture formulas never require special-case spaghetti code:

  

GDScript

```
class_name CombatantStats
extends Resource

@export_group("Unified Combatant Core (Shared by Player & Enemies)")
@export var max_vitality: float = 500.0          # Total HP pool
@export var max_sha_chi: float = 100.0           # Posture capacity before lethal stun / execution
@export var poise: float = 30.0                  # Stance Break needed to interrupt an active attack
@export var posture_recovery_rate: float = 12.0  # Sha-Chi drained per second when not under pressure
```

- **For the Player:** `max_vitality` and `max_sha_chi` scale cleanly with your single **Character Level (`1 → 50`)**, while `poise` and `posture_recovery_rate` stay crisp and predictable (with `posture_recovery_rate` speeding up when holding guard cleanly, just like _Sekiro_).
    
- **For Enemies & Bosses:** You simply author one `CombatantStats` `.tres` file per enemy archetype (e.g., `stats_mountain_duelist.tres` with `poise = 15, posture_recovery_rate = 22` vs. `stats_crucible_commander.tres` with `poise = 65, posture_recovery_rate = 8`).
    

### 🎯 Next Step Question

Would you like to **build the shared `Combatant` base node in Godot 4** using `CombatantStats` and `WeaponData` next—wiring up `take_hit(slash_power, stance_break)` and `deflect()` so you can test breaking a dummy's `Sha-Chi` posture in your `Region1` scene?

---
> ✍️ **Phrasing Refinement:** "How can I be certain these statistics are sufficient? Can we create dynamic boss variety using them, and what about character and enemy statistics?"
> 
>   

# ✅ Summary:

- **Yes—those four weapon numbers (`Slash Power`, `Stance Break`, `Guard Stability`, `Skill Affinity`) plus one passive trait are mathematically complete**, because every action in a _Sekiro_ / _Phantom Blade_-style swordfighter reduces to four fundamental variables: **health damage, posture damage, defensive mitigation, and skill-resource generation**.
    
- **How They Create Dynamic Boss Variety:** Instead of altering player spreadsheets, bosses test **different axes of those four stats**—such as high-poise knights who shrug off low `Stance Break`, relentless flurry duellists who shatter low `Guard Stability` if you block instead of deflecting, or agile monsters that starve low `Skill Affinity` builds of Spirit Seals.
    
- **Character & Enemy Stats (`Mirror Symmetry`):** Keep **both** the player and enemies on the exact same **four core combat stats** (`Vitality`, `Sha-Chi Posture`, `Poise`, and `Posture Recovery Rate`) so your shared `Combatant` state machine runs on one unified math formula.
    

### 📖 Plain-English Glossary of Minimalist Combat Math

- **Orthogonal Stats (`Zero Overlap`):** Four statistics are "orthogonal" when each controls a completely separate lever of combat (`HP`, `Posture`, `Defence`, `Resource`)—meaning no two stats do the same job, and removing any one of them leaves a noticeable mechanical gap.
    
- **`Poise` (`Hyper-Armour Threshold`):** A hidden or visible threshold on enemies and bosses determining how much `Stance Break` is required to flinch them mid-swing—differentiating a nimble _Onze_-style duellist (low poise, interrupts easily) from a _Black Knight_ or _Crucible Commander_ (high poise, powers through light slashes).
    
- **`Posture Recovery Rate` (`Sha-Chi Regeneration`):** How quickly an enemy's yellow/crimson posture bar drains when left unpressured—forcing aggressive, relentless swordplay against fast bosses whose `Sha-Chi` recovers rapidly at high health.
    

### ⚖️ 1. Why These 4 Weapon Stats Are Mathematically Complete

Every frame of melee combat in _Ashen Kata_ asks only four questions. If your weapon covers all four cleanly, adding more stats only creates UI clutter without adding real tactical depth:

  

|**Weapon Stat**|**Combat Question It Answers**|**What Happens When It Is High vs. Low**|
|---|---|---|
|**1. `Slash Power`**|_"How fast do I kill an enemy once I find an opening?"_|**High (`Odachi`):** Punishes brief boss recovery windows heavily.<br><br>  <br>  <br><br>**Low (`Twin Blades`):** Requires sustained combo pressure to chip HP down.|
|**2. `Stance Break`**|_"How fast do I fill the enemy's `Sha-Chi` bar and flinch them?"_|**High:** Interrupts mid-tier enemies (`Black Knights`) and topples giants (`Tower Knight`).<br><br>  <br>  <br><br>**Low:** Relies on clean deflections rather than raw strikes to break posture.|
|**3. `Guard Stability`**|_"How forgiving is an imperfect block when I mistime a deflect?"_|**High (`Katana` / `Greatsword`):** Protects your own posture during frantic 5-hit boss flurries.<br><br>  <br>  <br><br>**Low (`Twin Blades`):** Punishes passive guarding; forces _Ghoststep_ evasion or frame-perfect parries.|
|**4. `Skill Affinity`**|_"How fast do I charge Spirit Seals for `X`, `C`, and `F` Boss Arts?"_|**High:** Rewards aggressive deflection chains with frequent _Twin Moon_ or _Ash-Lightning_ casts.<br><br>  <br>  <br><br>**Low:** Trades frequent spectacle skills for raw physical `Slash Power`.|

### ⚔️ 2. How Bosses Dynamically Test Those Exact 4 Stats

You do not need elemental resistance tables (`Fire`, `Lightning`, `Bleed`, `Frost`) to force players to adapt their loadout or tactics across bosses:

  

1. **The Hyper-Aggressive Duelist (`Onze` / `Genichiro` Archetype):**
    
    - **Boss Profile:** Low `Poise` (flinches when interrupted), rapid `Posture Recovery Rate` at full HP, and 5-hit flurry combos.
        
    - **Stat Test:** Tests **`Guard Stability`** (surviving rapid parry strings) and **`Skill Affinity`** (converting multi-hit deflections into rapid `X`/`C` counter-skills before their posture recovers).
        
2. **The Heavy Armoured Commander (`Black Knight` / `Winged Crucible` Archetype):**
    
    - **Boss Profile:** High `Poise` (ignores light slashes during wind-ups), slower `Posture Recovery Rate`, and massive overhead cleaves.
        
    - **Stat Test:** Tests **`Stance Break`** (cracking their poise during delayed wind-ups) and **`Slash Power`** (capitalising on narrow punish windows after _Ghoststepping_ through unblockable wing/shoulder checks).
        
3. **The Colossal Juggernaut (`Tower Knight` / `Giant Lord` Archetype):**
    
    - **Boss Profile:** Massive `Vitality` pool, immune to standard flinching until ankle/wrist `Sha-Chi` threshold breaks.
        
    - **Stat Test:** Rewards high **`Stance Break`** weapons or heavy `F` Boss Arts (_Ash-Lightning Plunge_) to trigger a full-body posture collapse, followed by high **`Slash Power`** executions.
        

### 🧬 3. Streamlined Character & Enemy Stats (`CombatantStats` Mirror)

Keep **Player**, **Small Mobs**, **Medium Knights**, and **Bosses** on the **exact same 4-stat `CombatantStats` structure** so your damage and posture formulas never require special-case spaghetti code:

  

GDScript

```
class_name CombatantStats
extends Resource

@export_group("Unified Combatant Core (Shared by Player & Enemies)")
@export var max_vitality: float = 500.0          # Total HP pool
@export var max_sha_chi: float = 100.0           # Posture capacity before lethal stun / execution
@export var poise: float = 30.0                  # Stance Break needed to interrupt an active attack
@export var posture_recovery_rate: float = 12.0  # Sha-Chi drained per second when not under pressure
```

- **For the Player:** `max_vitality` and `max_sha_chi` scale cleanly with your single **Character Level (`1 → 50`)**, while `poise` and `posture_recovery_rate` stay crisp and predictable (with `posture_recovery_rate` speeding up when holding guard cleanly, just like _Sekiro_).
    
- **For Enemies & Bosses:** You simply author one `CombatantStats` `.tres` file per enemy archetype (e.g., `stats_mountain_duelist.tres` with `poise = 15, posture_recovery_rate = 22` vs. `stats_crucible_commander.tres` with `poise = 65, posture_recovery_rate = 8`).
    

### 🎯 Next Step Question

Would you like to **build the shared `Combatant` base node in Godot 4** using `CombatantStats` and `WeaponData` next—wiring up `take_hit(slash_power, stance_break)` and `deflect()` so you can test breaking a dummy's `Sha-Chi` posture in your `Region1` scene?

---

> ✍️ **Phrasing Refinement:** "I assume you can see my current work and started architecture on statistics; are you truly certain we can achieve _Elden Ring_-level depth using only those stats? What other statistics could I potentially introduce to diversify attacks, weapons, bosses, or player attributes beyond movesets alone?"

  

# ✅ Summary
Yes—with your current `AttackProfile.gd` (`hp_damage`, `poise_damage`, `poise_gain_on_hit`, `baked_lunge_distance`, `hitstop_duration`, `can_cancel_intoDeflect`) and a 4-stat weapon core (`Slash Power`, `Stance Break`, `Guard Stability`, `Skill Affinity`), you already possess 80% of _Sekiro_'s and _Elden Ring_'s real mechanical depth—because in _Elden Ring_, most "depth" is actually an illusion of twenty redundant scaling letters feeding into the exact same two outputs (`HP Damage` and `Poise/Stance Damage`).

  

However, if you want genuine build variety and boss counter-play _without_ bloating your UI into an _Elden Ring_ spreadsheet, you can introduce **4 high-leverage, non-spreadsheet "Multiplier & Interaction Stats"** that diversify weapons, attacks, and bosses dramatically whilst keeping your character level strictly single-track.

  

### 🔍 1. The Brutal Truth About _Elden Ring_'s Stats

In _Elden Ring_, having Strength, Dexterity, Quality, Heavy, Keen, and 80 soft-caps feels deep on a wiki, yet during an actual boss fight, two completely different melee builds reduce to the exact same four questions:

  

1. **How hard does a clean hit punish an opening?** (`hp_damage` / `Slash Power`)
    
2. **How fast does it break the boss's stance?** (`poise_damage` / `Stance Break`)
    
3. **How safe is my defence or recovery?** (`Guard Stability` + `can_cancel_intoDeflect`)
    
4. **How fast do I fuel special arts?** (`poise_gain_on_hit` / `Skill Affinity`)
    

Where _Elden Ring_ actually creates real tactical diversity isn't the level-up screen—it is **Damage Types vs. Armour Archetypes**, **Status Build-up Pressure**, **Hyper-Armour (Active Poise)**, and **Critical Execution Multipliers**.

  

### ⚔️ 2. Four High-Impact Stats to Add (Without Spreadsheet Bloat)

Rather than adding player attributes you have to dump points into, attach these **four crisp mechanical stats** directly onto your `AttackProfile`, `WeaponData`, and `CombatantStats` resources:

  

#### 1. `active_hyper_armour` (Attack-Level Trade Potential)

- **What it is:** A flat `Poise` bonus granted _only_ during the active swing frames of a specific attack (stored right on `AttackProfile.gd`).
    
- **Why it diversifies combat:** Fast katana slashes have `0.0` hyper-armour—if a _Black Knight_ clips you mid-swing, you flinch. Heavy _Odachi_ finishers or slow `F` Boss Arts carry high `active_hyper_armour`, letting you **intentionally trade through light enemy pokes** to land a stance-shattering cleave.
    

#### 2. `execution_multiplier` (Critical Riposte & Counter-Hit Lethality)

- **What it is:** A weapon or attack's bonus multiplier (`1.5×` to `3.0×`) when striking a **Poise-broken** enemy or landing a **Counter-Hit** (hitting an enemy during their own attack startup).
    
- **Why it diversifies combat:** Suddenly, a nimble dagger or twin-blade with modest base `hp_damage` becomes lethal in the hands of a skilled player who fishes for counter-hits and Poise breaks (`2.8×` execution multiplier), whereas a heavy _Odachi_ relies on consistent raw `hp_damage` (`1.6×` execution multiplier).
    

#### 3. `damage_trait` (Physical vs. Pierce vs. Crush — 3 Clean Types Only)

- **What it is:** A simple 3-way enum (`SLASH`, `PIERCE`, `CRUSH`) on `AttackProfile.gd` paired with simple enemy surface affinities (`Armoured Plate`, `Flesh/Beast`, `Stone/Colossus`).
    
- **Why it diversifies bosses & weapons:**
    
    - **`SLASH`** shreds unarmoured beasts and nimble duellists (_Onze_ / _Seeker_), but glances with reduced `poise_damage` off heavy plate.
        
    - **`CRUSH`** (pommel strikes, heavy hammers, _Mortal Draw_-style slams) cracks stone colossi (_Tower Knight_) and heavily armoured _Crucible / Black Knights_.
        
    - **`PIERCE`** (spears, katana thrusts, _Shadowrush_) deals bonus **Counter-Hit** damage when interrupting an enemy's lunge.
        

#### 4. `affliction_buildup` (1 Signature Status Slot per Weapon/Boss)

- **What it is:** Instead of 8 overlapping status bars, allow a weapon or boss attack to carry **at most one** lingering buildup (`Ashen Bleed`, `Scorch`, or `Abyssal Frost`) that triggers a single dramatic burst when filled:
    
    - **Ashen Bleed:** Instant `%` chunk of `HP` + `Poise` burst—rewards relentless, unbroken _Phantom Blade_ aggression before the gauge decays.
        
    - **Scorch / Cinder:** Temporarily halts the enemy's `posture_recovery_rate` for `6 seconds`—a massive tactical tool against fast bosses whose `Poise` normally regenerates rapidly.
        
    - **Abyssal Frost:** Slows enemy animation playback speed slightly (`0.88×`) for `5 seconds`, softening hyper-fast flurry chains.
        

### 📊 3. How This Keeps Your UI Minimalist While Tripling Build Variety

|**Entity**|**Core Stats (Always Visible / Simple)**|**Tactical Differentiators (Baked Into .tres Cards)**|
|---|---|---|
|**Player (`Lv. 1–50`)**|`Vitality` (HP), `Poise` (Stance), `Spirit Seals`|Zero stat points to allocate—build variety comes entirely from your **2 equipped weapons (`Flux`) + 1 `F` Boss Art**.|
|**Weapon (`WeaponData`)**|`Slash Power`, `Stance Break`, `Guard Stability`, `Skill Affinity`|`execution_multiplier` (`1.5×–3.0×`), `affliction_type` (optional), + **1 Innate Trait**.|
|**Attack (`AttackProfile`)**|`hp_damage`, `poise_damage`, `poise_gain_on_hit`|`damage_trait` (`SLASH / PIERCE / CRUSH`), `active_hyper_armour`, `can_cancel_intoDeflect`.|
|**Boss / Enemy (`CombatantStats`)**|`max_vitality`, `max_poise`, `poise`, `posture_recovery_rate`|`surface_type` (`Flesh / Plate / Stone`) + signature `affliction_buildup` on special attacks.|

### 🎯 Next Step Question

Would you like to **add `damage_trait` (`SLASH / PIERCE / CRUSH`), `active_hyper_armour`, and `execution_multiplier` directly into your `AttackProfile.gd`** right now, or move straight onto **Step 3 of our combat architecture (`LocomotionState` + `AttackState` with Root Motion and Frame Windows)**?