# 📜 ARCHITECTURAL SPECIFICATION: _Phantom Blade Zero_ ("Kung-Fu Punk") Combat Engine

> **System Context Instruction for AI Models:**
> 
> When generating GDScript 2.0 or designing combat architectures for this project, you must strictly adhere to the mechanical laws, state-cancellation rules, and resource economies defined below. We are **not** building a slow, high-commitment Soulslike; we are building a high-speed, rhythmic Wuxia action system that merges **Sekiro’s** deflection economy, **Monster Hunter / Elden Ring’s** multi-target spatial positioning, and **Character Action** weapon-swap combo extensions.

## 📖 Lexicon & Core Terminology (The Foundational Primitives)

- **Sha-Chi (殺氣 / Killing Intent):** A unified, bidirectional combat resource gauge shared by both the player and enemies. Unlike a standard stamina bar, Sha-Chi does _not_ drain from sprinting, basic light attacks, or standard dodging. It functions simultaneously as **offensive mana** (spent to execute Heavy/Sha-Chi Combos) and **defensive posture** (drained when blocking attacks).
    
- **Ghoststep (鬼步):** An automated, zero-recovery spatial blink/vault triggered upon executing a frame-perfect Parry against a **Blue Flash** attack or a frame-perfect Dodge against a **Red Flash** attack. It warps the player along an orbital arc directly behind the aggressor's back-plane, granting a guaranteed punish window.
    
- **Brutal Move (Blue Glint):** A telegraphed multi-hit or high-momentum enemy strike that can be blocked (at heavy Sha-Chi cost) or parried at the exact frame of impact to trigger a **Ghoststep** and inflict massive posture damage.
    
- **Killer Move (Red Glint):** An unblockable, unparryable lethal grab or sweeping crush. Attempting to block or parry results in immediate guard failure; it must be evaded via a timed **Perfect Dodge** to trigger a **Ghoststep**.
    
- **Guard-Crush Strike (Black Glint):** A heavy kinetic blow designed to shatter guards. Blocking it inflicts catastrophic Sha-Chi depletion, forcing an immediate **Weapon Swap** recovery or proactive spatial evasion.
    
- **Flux Swap (Mid-Combo Weapon Switch):** The act of swapping between Primary Weapon 1 and Primary Weapon 2 mid-string. Acting as an offensive/defensive _Ki-Pulse_, swapping weapons immediately **restores a chunk of the player's Sha-Chi** while extending the active combo chain across weapon archetypes (governed by a brief internal cooldown).
    
- **Phantom Edge (The Prosthetic / Sign Layer):** Heavy secondary kinetic tools (e.g., Fire-Breathing Dragon Head, Tiger Cannon, Bashpole, Bow) mapped to a dedicated input that consume **Spark Charges** rather than Sha-Chi. Used to interrupt boss hyper-armour, control crowds, or bridge ranged gaps.
    

## 🎬 Video Reference Telemetry Breakdown (`[https://youtu.be/B-XMWyUq6QA](https://youtu.be/B-XMWyUq6QA)`)

The reference footage demonstrates a 1v2 boss encounter against **Mistress's Guardian** (agile polearm/halberd striker) and **Iron Fiend, Young Master** (heavy ballistic/artillery bruiser), illustrating how the three pillars—Parry, Spacing, and Weapon Variety—interlock in real time:

- **[[00:00](https://www.youtube.com/watch?v=B-XMWyUq6QA&t=0)] – Dual-Boss Spatial Triangulation:** The player stands in open terrain tracking two asynchronous threat vectors. Rather than hard-locking camera rotation onto a single point, locomotion remains fluid to keep both bosses within the forward camera frustum.
    
- **[[00:02](https://www.youtube.com/watch?v=B-XMWyUq6QA&t=2)] – Phantom Edge Crowd Control & Hyper-Armour:** As _Mistress's Guardian_ closes the gap, the player summons the **Fire-Breathing Dragon Head** Phantom Edge, projecting a continuous cone of flame that damages the rusher while creating visual and spatial separation.
    
- **[[00:05](https://www.youtube.com/watch?v=B-XMWyUq6QA&t=5)] – Aerial Vault & Multi-Axis Evasion:** When _Mistress's Guardian_ initiates a sweeping rush, the player executes a vertical vault-flip directly over the boss's hitbox [[00:06](https://www.youtube.com/watch?v=B-XMWyUq6QA&t=6)], converting a defensive evade into an immediate rear-flank air-to-ground counter.
    
- **[[00:08](https://www.youtube.com/watch?v=B-XMWyUq6QA&t=8)] – Multi-Hit Deflection Cadence:** The player catches a rapid flurry of halberd strikes via consecutive timed parries (`L1` / `RMB` taps), generating bright metallic spark telemetry and zero health chip while rapidly eroding the boss's white posture/Sha-Chi bar.
    
- **[[00:11](https://www.youtube.com/watch?v=B-XMWyUq6QA&t=11)] – Red Glint Unblockable & Ghoststep:** The boss telegraphs a lethal **Red Flash (Killer Move)** [[00:12](https://www.youtube.com/watch?v=B-XMWyUq6QA&t=12)]. Instead of parrying, the player executes a timed **Perfect Dodge**, triggering **Ghoststep** to blur past the strike and immediately counter-slash with a dual-blade combo.
    
- **[[00:16](https://www.youtube.com/watch?v=B-XMWyUq6QA&t=16)] – Environmental Kiting & Ballistic Spacing:** While _Iron Fiend_ fires explosive artillery shells from range, the player uses dust occlusion and lateral sprinting to force the melee boss into an isolated 1v1 pocket, parrying a **Blue Flash** lunge at [[00:22](https://www.youtube.com/watch?v=B-XMWyUq6QA&t=22)].
    
- **[[00:25](https://www.youtube.com/watch?v=B-XMWyUq6QA&t=25)] – Relentless Dual-Weapon Pressure:** The player chains rapid dual-blade slashes into heavy staggering swings [[00:27](https://www.youtube.com/watch?v=B-XMWyUq6QA&t=27)], seamlessly dodging a second **Red Flash** assault at [[00:29](https://www.youtube.com/watch?v=B-XMWyUq6QA&t=29)] and answering with a spinning aerial kick/slash transition at [[00:35](https://www.youtube.com/watch?v=B-XMWyUq6QA&t=35)].
    
- **[[00:44](https://www.youtube.com/watch?v=B-XMWyUq6QA&t=44)] – Vertical Plunge & AoE Evasion:** Dealing with simultaneous aerial leaps from the bosses, the player weaves between ground impact shockwaves and counters during the bosses' heavy recovery frames [[00:48](https://www.youtube.com/watch?v=B-XMWyUq6QA&t=48)].
    

## ⚙️ Core System 1: The "Sha-Chi" Bidirectional Resource Economy

To prevent the passive "run away and wait for stamina to regenerate" loop of traditional Soulslikes, implement a single unified resource component (`ShaChiComponent.gd`):

```
                      THE SHA-CHI BIDIRECTIONAL LOOP
===================================================================================
                         [ SHA-CHI GAUGE (0 - 100) ]
                        /                           \
        OFFENSIVE SPEND                              DEFENSIVE DRAIN
   - Heavy / Sha-Chi Combos                     - Holding Guard (Continuous drain)
   - Power Surge / Weapon Arts                  - Blocking Standard Strikes
   - Canceling Recovery Frames                  - Blocking Black-Glint Attacks
                        \                           /
                         ▼                         ▼
                  [ INSTANT RECOVERY MECHANISMS (GAIN) ]
                  1. Perfect Parry (0 cost + damages enemy Sha-Chi!)
                  2. Flux Weapon Swap (Restores +35% Sha-Chi instantly!)
                  3. Landing Light Combo Finishers (+5% per hit)
```

### Architectural Rules for Models:

1. **Zero Locomotion Tax:** Sprinting, jumping, wall-running, and basic evasive dashes cost **$0.0$ Sha-Chi**. Movement is entirely unrestricted.
    
2. **Guard Penalty vs. Deflection Reward:**
    
    - _Holding Guard (`Stance`):_ Continuously drains Sha-Chi at rate $\lambda_{\text{hold}}$ and absorbs incoming hit damage as Sha-Chi reduction. If Sha-Chi hits $0.0$, the actor enters a **Guard-Broken Stagger State**.
        
    - _Timed Deflection (Tap `Stance` within $150\text{ ms}$ of impact):_ Costs **$0.0$ Sha-Chi**, inflicts $D_{\text{posture}}$ directly back onto the attacker's Sha-Chi gauge, and can cancel out of any light-attack startup or recovery frame.
        
3. **The Weapon-Swap Recovery ("Flux"):** Swapping between Primary Weapon A and Primary Weapon B triggers an active combat reload that restores a flat percentage of Sha-Chi (e.g., $+30\text{--}40\%$), governed by a $6.0\text{--}8.0\text{ s}$ cooldown. This forces players who accidentally block a heavy strike to immediately swap weapons and go on the offensive rather than retreating.
    

## ⚡ Core System 2: The Color-Coded "Simon Says" Threat Matrix & Ghoststep

Enemy attacks are categorized by a bitmask or enum inside their `AttackProfile` resource (`Module 2.2`). Each telegraph demands a distinct defensive response:

|**Telegraph Type**|**Visual / Audio Cue**|**Mechanical Properties**|**Required Player Input**|**Successful Resolution Outcome**|**Failure Penalty**|
|---|---|---|---|---|---|
|**Standard Strike**|No coloured glint (Raw animation telegraph)|Normal physical/elemental damage|**Tap Block** (Parry) or **Hold Block** or **Dodge**|**Parry:** Damages enemy Sha-Chi.<br><br>  <br><br>**Block:** Absorbs hit at Sha-Chi cost.|Takes standard HP damage & hit-stun.|
|**Brutal Move**|🔵 **Blue Glint** + High-pitched metallic chime|High multi-hit momentum; heavy guard chip|**Tap Block (Perfect Parry)** at exact impact frame|Triggers **Ghoststep** (auto-blinks behind enemy) + massive enemy Sha-Chi break.|Blocking drains $60\%+$ Sha-Chi; getting hit deals heavy HP damage.|
|**Killer Move**|🔴 **Red Glint** + Deep resonant bass warning|**Unblockable & Unparryable** (Grabs, sweeps, crushes)|**Tap Dodge (Perfect Evade)** right before contact|Triggers **Ghoststep** (vaults/blinks behind enemy) + slows local time briefly.|Ignores guard completely; triggers high-damage grab or launch.|
|**Crushing Blow**|⚫ **Black Glint** + Heavy shockwave distortion|Guard-shattering kinetic force|**Proactive Positional Spacing** or **Weapon-Swap Cancel**|Avoids impact zone entirely, or blocks and immediately **Weapon Swaps** to refill broken Sha-Chi.|Instant **Guard Break** (`Sha-Chi = 0`) + long vulnerable stun state.|

### Mathematical Implementation of `Ghoststep`:

When a `Ghoststep` is triggered against `target_actor: Node3D`:

1. Grant $0.35\text{ s}$ of absolute invulnerability (`HurtboxComponent._is_invulnerable = true`).
    
2. Compute the target's rear-flank world coordinate using the Basis forward invariant (`Module 4.4`):
    
    $$\vec{P}_{\text{behind}} = \vec{P}_{\text{enemy}} + \left(\mathbf{B}_{\text{enemy}}.\vec{z}_{\text{back}} \cdot d_{\text{offset}}\right)$$
    
3. Interpolate the player along a curved **Cubic Bézier** arc (`Module 4.6`) or vault spline over $0.18\text{ s}$ so they do not clip through the boss's capsule collider, rotating the player's `transform.basis` upon arrival to face $\vec{P}_{\text{enemy}}$.
    

## 🗡️ Core System 3: Dual-Weapon Loadouts & Animation Cancellation Hierarchy

To achieve the fluid, cinematic presentation seen in the video without requiring complex fighting-game motion inputs, combat relies on a strict **Cancellation Priority Hierarchy**:

```
                     ANIMATION CANCELLATION PRIORITY PYRAMID
===================================================================================
 [ Tier 4: Deflect / Dodge / Ghoststep ]  ◄── Can cancel Light Attacks & Recovery AT ANY FRAME!
                   ▲
 [ Tier 3: Phantom Edge / Power Surge ]   ◄── Cancels Heavy Recovery; grants Hyper-Armour
                   ▲
 [ Tier 2: Flux Weapon Swap Attack ]      ◄── Cancels Final Hit of Light/Heavy String + Restores Sha-Chi
                   ▲
 [ Tier 1: Heavy (Sha-Chi) Combo ]        ◄── High commitment! CANNOT be dodge/parry canceled mid-swing!
                   ▲
 [ Tier 0: Light Standard Combo ]         ◄── Low commitment; fast startup; zero Sha-Chi cost
```

### 1. The Dual-Primary Archetype Synergy

The player equips two melee weapons simultaneously (`Primary_Slot_A` and `Primary_Slot_B`), chosen from distinct weight/speed classes:

- **Fast / Agile Class (Dual Blades, Twin Daggers, Soft Whip-Sword):** Low damage per hit, rapid multi-hit strings, zero-commitment frames (can be parry-canceled or dodge-canceled at any point). Ideal for shaving down enemy Sha-Chi while staying safe against erratic bosses.
    
- **Heavy / Stagger Class (Greatsword, Heavy Broadsword, Cleaver):** High animation commitment (cannot be canceled mid-swing), massive health and stagger multiplier. Swapped into via **Flux** the exact moment an enemy's Sha-Chi gauge shatters or after a **Ghoststep** rear-flank.
    
- **Balanced / Tactical Class (Standard Longsword, Spear/Halberd):** Extended reach (`2.5m - 3.5m` hitbox volumes) and specialized counter-parry frames.
    

### 2. Context-Sensitive Crowd & Execution Routing

- **Multi-Target Directional Flicking:** When surrounded by multiple enemies (like the 1v2 boss fight in the clip), flicking the movement stick/WASD toward a secondary attacker during a light combo seamlessly redirects the next strike's root-motion lunge toward that secondary target without dropping the combo counter.
    
- **Synchronised Executions:** Depleting an enemy's Sha-Chi to $0.0$ opens a context-sensitive **Execution State** (`Triangle` / `F` key). If two staggered enemies are within $2.5\text{ metres}$ of each other, the execution state queries a dual-target animation that finishes both simultaneously.
    

## 📐 Module Mapping: How to Build This in Your Godot 4 Codebase

When asking any AI model to write code for these mechanics, point it to these exact architectural modules from your Masterlist:

|**Phantom Blade Zero Mechanic**|**Target Masterlist Module**|**Concrete GDScript Implementation Strategy**|
|---|---|---|
|**Light / Heavy / Deflect / Ghoststep States**|**Module 3.3:** _Pushdown Automaton (`StateMachine.gd`)_|Make `Idle`, `Move`, `LightCombo`, and `HeavyCombo` base states. Make `Deflect`, `Ghoststep`, and `Stun` **Push States** (`push_state()`) that interrupt active swings and pop back neatly.|
|**Weapon Variety & Attack Glints (Blue/Red/Black)**|**Module 2.2 & 2.5:** _Flyweight `AttackProfile` & Bitmasks_|Add a `ThreatGlint { NONE, BLUE_PARRY, RED_DODGE, BLACK_CRUSH }` enum and `sha_chi_damage: float` property to `AttackProfile.gd`.|
|**Ghoststep Rear-Flank Teleport / Vault**|**Module 4.4 & 4.6:** _Basis Transforms & Cubic Bézier Curves_|Calculate target rear position via `enemy.global_transform.basis.z * 1.8` and sweep the player around the enemy using `evaluate_cubic_bezier()`.|
|**1v2 Boss Aggression Pacing (No Unfair Overlapping)**|**Module 7.5:** _`CombatDirector.gd` (Attack Token System)_|Limit `max_melee_tokens = 1` and `max_ranged_tokens = 1`. While _Mistress's Guardian_ holds the melee token to rush you, _Iron Fiend_ is restricted to ranged cannon fire or circling!|
|**Root-Motion Kung-Fu Lunges**|**Module 6.2:** _`LocomotionAnimController.gd` (Root Motion)_|Drive sword combo forward momentum using `animation_tree.get_root_motion_position()` so feet never ice-skate during rapid martial arts flurries.|
