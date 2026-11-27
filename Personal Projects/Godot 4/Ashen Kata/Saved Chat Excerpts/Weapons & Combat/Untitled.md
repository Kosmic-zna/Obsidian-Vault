✍️ **Phrasing Refinement:** "Everything is working as expected now. Could you lay out a plan for building our X, C, and F skills so they stay modular—supporting optional custom scripts, grabs, VFX, and custom AoE hitboxes rather than just basic blade slashes? Also, should skills have their own separate card resource, or should they be integrated directly into our existing `AttackProfile`?"

  

✅ **Summary:** Do **not** bloat `AttackProfile` with grab offsets and AoE radii, and do **not** build a disconnected skill card that forces you to duplicate your combat code. Instead, use **Inheritance + Composition**: create a **`SkillProfile` that extends `AttackProfile`** (so every skill automatically inherits hit-stop, Poise break, Impact Tiers, root motion, and DS3 combo redirection for free) and plug in **three modular slots**—an optional **`SkillBehaviour` script resource** (for grabs, teleports, or custom logic), a **`Hitbox` prefab slot** (for AoE spheres, cones, and shockwaves), and a **`VFX` socket array** triggered straight from your `AnimationPlayer` timeline.

  

### 🧩 1. The Big Question: Modify `AttackProfile` or Create a New Card?

Here is the trap most indie combat games fall into early on.

  

If you cram every skill feature—Spirit Seal costs, grab anchor offsets, projectile speeds, AoE radii—directly onto `AttackProfile.gd`, then every humble `Katana_Light_1.tres` becomes an ugly 40-field spreadsheet full of empty checkboxes.

  

Yet if you invent a completely separate `SkillData` card from scratch, your `WeaponTracer3D`, `Combatant`, and `AttackState` suddenly have to check _"Wait, am I holding an `AttackProfile` or a `SkillData`?"_ at every single turn. That way lies spaghetti code.

  

So how do we get the best of both worlds? **Inheritance.**

  

🧬 `inheritance` (`class_name SkillProfile extends AttackProfile`): _a child resource that automatically possesses every single property of `AttackProfile`, whilst adding extra skill-only fields on top._

  

Look at what happens the moment `SkillProfile` extends `AttackProfile`:

  

- `AttackState`, `Combatant`, and `WeaponTracer3D` never need to care whether you pressed LMB, X, C, or F—to them, it is still an `AttackProfile` carrying `hp_damage`, `poise_damage`, `attack_tier`, `armour_tier`, `affliction_type`, and `baked_lunge_distance`.
    
- Your basic light attacks (`Katana_Light_1/2/3.tres`) stay lean and uncluttered.
    
- Your **X & C Weapon Arts** (stored on `WeaponData`) and **F Ashen Rites / Boss Arts** (stored on `PlayerLoadout`) use `SkillProfile.tres`, unlocking extra fields like `seal_cost`, `cooldown`, `hitbox_prefab`, `vfx_scenes`, and `custom_behaviour`.
    

### 🗺️ 2. The Modular Architecture Blueprint

Before we touch a single script, picture how every piece plugs into the exact `AttackState` you already have working today:

  

Plaintext

```
==========================================================================================
             ASHEN KATA — MODULAR X / C / F SKILL & HITBOX ARCHITECTURE
==========================================================================================

  [ INPUT LAYER ]             [ DATA LAYER (.tres) ]            [ EXECUTION LAYER ]
  ───────────────             ──────────────────────            ───────────────────
  LMB ──────────────────────► AttackProfile.tres ────────────┐
  X / C (Weapon Arts) ──────► WeaponData ──► SkillProfile ───┼─► AttackState.gd
  F     (Ashen Rite)  ──────► Loadout    ──► SkillProfile ───┘   (One Unified State!)
                                                  │
                ┌─────────────────────────────────┼──────────────────────────────┐
                ▼                                 ▼                              ▼
     [ HITBOX DELIVERY ]               [ CHOREOGRAPHED VFX ]          [ OPTIONAL BEHAVIOUR ]
     • Default: WeaponTracer3D         • `vfx_scenes[]` array         • `SkillBehaviour`
     • Custom : Spawn `AoEHitbox3D`      triggered via timeline         (Strategy Resource)
       (Sphere / Cone / Ring / Beam)     at named Rig Sockets           • Grabs & Throws
       → Feeds same `Combatant`          (Feet, BladeTip, World)        • Blinks / Teleports
         hit resolution pipeline!                                       • Projectiles / Buffs
==========================================================================================
```

Notice the elegance here: **90% of your skills require zero custom GDScript.** A leaping overhead cleave, a spinning katana flourish, or a ground-shattering slam is simply a `.tres` card wired to an animation clip, an AoE hitbox prefab, and a VFX scene.

  

### ⚙️ 3. Modular Custom Scripts via `SkillBehaviour` (The 10% Wildcards)

What happens when you want to build _Mist Raven_ (teleport behind an attacker), _Inescapable Frenzy_ (grab an enemy by the helm), or _Rellana's Twin Moons_ (levitate and drop two exploding spheres)?

  

Do **not** create fifteen separate FSM states (`GrabState`, `TeleportState`, `MoonSlamState`) that duplicate root motion, gravity, and hit-stop logic. Instead, use the **Strategy Pattern**—give `SkillProfile` one optional `@export var behaviour: SkillBehaviour` slot.

  

🧠 `strategy pattern`: _swapping out a small, self-contained logic module (a `Resource` script) at runtime so one core state (`AttackState`) can perform wildly different special mechanics without bloating._

  

`SkillBehaviour` is a tiny base `Resource` class with five empty virtual hooks that `AttackState` calls automatically whenever `behaviour != null`:

  

- **`on_enter(ctx)`** → Runs on Frame 0 (e.g. deduct Spirit Seals, toggle brief levitation or i-frames).
    
- **`on_physics(delta, ctx)`** → Runs every physics tick (e.g. pull a grabbed victim smoothly to your hand socket, or glide upward during an aerial slam).
    
- **`on_timeline_event(event_name, ctx)`** → Fired directly from Godot's `AnimationPlayer` Call Method Track (`trigger_skill_event("drop_moon")`) so custom code stays frame-locked to your animation.
    
- **`on_hit_landed(victim, hit_pos, ctx)`** → Fires the exact frame a blade or grab box connects (e.g. lock the victim into a grab cinematically!).
    
- **`on_exit(ctx)`** → Guaranteed cleanup even if a boss smacks you out of the skill mid-windup (restores gravity, releases grabbed targets, clears temporary buffs).
    

_(Here `ctx` is just a lightweight dictionary or reference bundle containing `actor`, `visual_model`, `anim_player`, `combatant`, and `profile`—keeping every custom skill script completely decoupled from your player scene!)_

  

### 💥 4. Custom Hitboxes for AoE Attacks (Beyond Basic Slashes)

Your `WeaponTracer3D` is brilliant for razor-sharp katana arcs, because sweeping rays between `HiltMarker` and `TipMarker` matches steel geometry to the millimetre.

  

Here is the catch: **rays on a sword cannot represent a 5-metre shockwave ring, a cone of dragon flame, or an orbital pillar of light.**

  

Rather than shoving `if is_aoe:` radius math into `AttackState.gd`, we build **one reusable `AoEHitbox3D.gd` script** attached to simple `Area3D` `.tscn` prefabs:

  

- **How You Author One:** Create a mini scene—say `ShockwaveRing_Hitbox.tscn`—containing an `Area3D` + `CollisionShape3D` (Sphere, Cylinder, Box, or Cone) **plus its matching visual shader mesh** right inside the same scene. What you see is literally what hits!
    
- **How It Fires:** On your `AttackProfile` (or `SkillProfile`), add an `@export var use_weapon_tracer: bool = true` toggle and `@export var spawned_hitboxes: Array[PackedScene]`.
    
- **Timeline Control:** On the exact impact frame in `AnimationPlayer`, your Call Method Track calls `spawn_hitbox(0, &"Feet")` (or `&"BladeTip"` / `&"Forward"`). `AttackState` instantiates the prefab, hands it `attacker` + `current_profile`, and the `AoEHitbox3D` checks every `Hurtbox3D` inside its shape—routing straight into your existing `Combatant.take_hit()` pipeline so Deflects, Poise breaks, Afflictions, and Hit-Stop all work identically!
    
- **Built-In Multi-Hit Support:** Give `AoEHitbox3D` a `tick_interval` and `lifetime` export. Set `tick_interval = 0.0` for a single crisp slam explosion, or `0.15s` for a lingering _One Mind_ slash dome or _Scarlet Aeonia_ bloom.
    

### 🤼 5. How Grabs Work Cleanly (Without Soft-Locking Your Game)

Grabs are notorious for breaking indie action games because two characters suddenly have to move as one. With `SkillBehaviour` + your existing `custom_reaction_state` design, a grab stays remarkably tidy:

  

1. **Phase 1 — The Lunge & Grab Check:** Pressing `F` enters `AttackState` with `Grab_Profile.tres` (which carries `GrabBehaviour.gd`). Your timeline spawns a short-range `GrabBox` (`Area3D`) in front of your hands.
    
2. **Phase 2 — The Lock-In (`on_hit_landed`):** The moment `GrabBox` touches a valid enemy (`armour_tier < COLOSSAL` so you cannot suplex a 20-metre Tower Knight):
    
    - `GrabBehaviour` forces the enemy's `StateMachine` into `&"GrabbedState"` (disabling their AI, gravity, and collision capsule temporarily).
        
    - It jumps the player's `AnimationPlayer` to the synchronised grab-execution clip (`"Grab_Connect"`) and welds the enemy's transform to a `GrabAnchor` (`Marker3D`) on your player rig each tick inside `on_physics()`.
        
3. **Phase 3 — Climax & Guaranteed Release:**
    
    - A Call Method key on the slam/stab frame calls `trigger_skill_event(&"grab_impact")` → deals the massive HP/Poise burst, camera trauma, and impact VFX.
        
    - On `on_exit()` (whether the grab finishes naturally _or_ another enemy knocks you out of it), `GrabBehaviour` **always** re-enables the victim's collider and boots them into `KnockdownState` so nobody ever gets permanently glued to your hand.
        

### ✨ 6. Modular VFX Spawning (Timeline & Socket Anchoring)

Just like your AoE hitboxes, VFX should never be hardcoded inside `AttackState.gd`. Instead, wire up **socket anchoring** alongside your choreographed one-shot VFX prefabs:

  

📌 `socket anchoring`: _registering named `Marker3D` points on your character (`Feet`, `Chest`, `BladeTip`, `HandRight`) in a simple Dictionary so any skill can spawn a shader ring or particle burst at the exact anatomical point without hardcoding coordinates._

  

- **Timeline VFX (`spawn_vfx(index, socket_name, follow_socket)`):**
    
    - Add `@export var vfx_scenes: Array[PackedScene]` onto `AttackProfile` / `SkillProfile`.
        
    - Call `spawn_vfx(0, &"Feet", false)` from your animation timeline: `follow_socket = false` drops a ground crack or dust ring into world space so it stays planted on the arena floor while you keep moving; `follow_socket = true` parents an aura or flame trail directly to your blade or chest.
        
    - Every VFX `.tscn` runs its own `AnimationPlayer` and calls `queue_free()` on finish, keeping `AttackState` completely hands-off.
        
- **Per-Move Impact VFX (`@export var impact_vfx: PackedScene`):**
    
    - Spawned automatically inside `_on_hit_landed(_victim, _hit_pos, profile)` at `_hit_pos`—so light slashes spark clean steel whilst an _Ash-Lightning_ skill detonates in crimson lightning without writing a single `if` check.
        

### 🪜 7. Your Phased Implementation Plan (Zero Regressions)

Do not build all five systems in one chaotic evening—roll them out in **five tight, testable phases** so your working katana loop never breaks:

  

1. **Phase 1 — Data & Input Wiring (`SkillProfile`, `WeaponData`, `PlayerLoadout`, X/C/F):**
    
    - Create `SkillProfile.gd` (`extends AttackProfile`), `WeaponData.gd` (`skill_x`, `skill_c`), and `PlayerLoadout.gd` (`equipped_weapon`, `equipped_rite_f`).
        
    - Register `skill_x`, `skill_c`, and `skill_f` in `InputBuffer.gd` and wire `LocomotionState` + `AttackState` combo-cancel hooks so pressing X, C, or F launches the equipped `SkillProfile` cleanly.
        
2. **Phase 2 — Universal Hitbox & VFX Spawner (`AoEHitbox3D` + Socket Registry):**
    
    - Build `AoEHitbox3D.gd` (`Area3D`) that talks directly to `Hurtbox3D` / `Combatant`.
        
    - Add a lightweight `VFXSockets` dictionary (`Feet`, `BladeTip`, `Chest`) plus `spawn_hitbox()` and `spawn_vfx()` Call Method hooks inside `AttackState.gd`.
        
3. **Phase 3 — Pluggable `SkillBehaviour` Hooks:**
    
    - Create the 15-line `SkillBehaviour.gd` base `Resource` and wire its 5 lifecycle hooks (`on_enter`, `on_physics`, `on_timeline_event`, `on_hit_landed`, `on_exit`) into `AttackState.gd`.
        
4. **Phase 4 — Build Three Vertical-Slice Proof Skills (Proves Every Path):**
    
    - **Slot `X` (Pure Data Skill — Zero Custom Script):** Fast lunging blade art reusing `WeaponTracer3D` + custom slash VFX.
        
    - **Slot `C` (AoE Hitbox + VFX Skill):** Ground slam or spinning crescent spawning an `AoEHitbox3D` shockwave ring at `&"Feet"`.
        
    - **Slot `F` (Custom `SkillBehaviour` Skill):** Either a short **Grab** (testing `GrabBehaviour` + `GrabbedState`) or a **Blink / Phase-Step** counter.
        
5. **Phase 5 — Live Validation Against `TestDummy.tscn` & Spirit Seal Economy:**
    
    - Drop a reactive `TestDummy.tscn` (with `Combatant`, `Hurtbox3D`, and `GrabbedState`) into `CombatArena` to verify AoE multi-hits, Poise breaks, hit-stop, and Spirit Seal spending live.
        

### 🎯 Next Step

Does this **`SkillProfile extends AttackProfile` + `AoEHitbox3D` + `SkillBehaviour`** blueprint match the modularity you had in mind for _Ashen Kata_—and would you like to start immediately with **Phase 1 (creating `SkillProfile.gd`, `WeaponData.gd`, and wiring X/C/F into `InputBuffer` and `AttackState`)**, or drop a **`TestDummy.tscn`** into `CombatArena` first so we have a live punching bag to test every phase against?

✍️ **Phrasing Refinement:** "Everything is working as expected now. Could you lay out a plan for building our X, C, and F skills so they stay modular—supporting optional custom scripts, grabs, VFX, and custom AoE hitboxes rather than just basic blade slashes? Also, should skills have their own separate card resource, or should they be integrated directly into our existing `AttackProfile`?"

✅ **Summary:** Do **not** bloat `AttackProfile` with grab offsets and AoE radii, and do **not** build a disconnected skill card that forces you to duplicate your combat code. Instead, use **Inheritance + Composition**: create a **`SkillProfile` that extends `AttackProfile**` (so every skill automatically inherits hit-stop, Poise break, Impact Tiers, root motion, and DS3 combo redirection for free) and plug in **three modular slots**—an optional **`SkillBehaviour` script resource** (for grabs, teleports, or custom logic), a **`Hitbox` prefab slot** (for AoE spheres, cones, and shockwaves), and a **`VFX` socket array** triggered straight from your `AnimationPlayer` timeline.

---

### 🧩 1. The Big Question: Modify `AttackProfile` or Create a New Card?

Here is the trap most indie combat games fall into early on.

If you cram every skill feature—Spirit Seal costs, grab anchor offsets, projectile speeds, AoE radii—directly onto `AttackProfile.gd`, then every humble `Katana_Light_1.tres` becomes an ugly 40-field spreadsheet full of empty checkboxes.

Yet if you invent a completely separate `SkillData` card from scratch, your `WeaponTracer3D`, `Combatant`, and `AttackState` suddenly have to check *"Wait, am I holding an `AttackProfile` or a `SkillData`?"* at every single turn. That way lies spaghetti code.

So how do we get the best of both worlds? **Inheritance.**

🧬 `inheritance` (`class_name SkillProfile extends AttackProfile`): *a child resource that automatically possesses every single property of `AttackProfile`, whilst adding extra skill-only fields on top.*

Look at what happens the moment `SkillProfile` extends `AttackProfile`:

* `AttackState`, `Combatant`, and `WeaponTracer3D` never need to care whether you pressed LMB, X, C, or F—to them, it is still an `AttackProfile` carrying `hp_damage`, `poise_damage`, `attack_tier`, `armour_tier`, `affliction_type`, and `baked_lunge_distance`.


* Your basic light attacks (`Katana_Light_1/2/3.tres`) stay lean and uncluttered.
* Your **X & C Weapon Arts** (stored on `WeaponData`) and **F Ashen Rites / Boss Arts** (stored on `PlayerLoadout`) use `SkillProfile.tres`, unlocking extra fields like `seal_cost`, `cooldown`, `hitbox_prefab`, `vfx_scenes`, and `custom_behaviour`.



---

### 🗺️ 2. The Modular Architecture Blueprint

Before we touch a single script, picture how every piece plugs into the exact `AttackState` you already have working today:

```text
==========================================================================================
             ASHEN KATA — MODULAR X / C / F SKILL & HITBOX ARCHITECTURE
==========================================================================================

  [ INPUT LAYER ]             [ DATA LAYER (.tres) ]            [ EXECUTION LAYER ]
  ───────────────             ──────────────────────            ───────────────────
  LMB ──────────────────────► AttackProfile.tres ────────────┐
  X / C (Weapon Arts) ──────► WeaponData ──► SkillProfile ───┼─► AttackState.gd
  F     (Ashen Rite)  ──────► Loadout    ──► SkillProfile ───┘   (One Unified State!)
                                                  │
                ┌─────────────────────────────────┼──────────────────────────────┐
                ▼                                 ▼                              ▼
     [ HITBOX DELIVERY ]               [ CHOREOGRAPHED VFX ]          [ OPTIONAL BEHAVIOUR ]
     • Default: WeaponTracer3D         • `vfx_scenes[]` array         • `SkillBehaviour`
     • Custom : Spawn `AoEHitbox3D`      triggered via timeline         (Strategy Resource)
       (Sphere / Cone / Ring / Beam)     at named Rig Sockets           • Grabs & Throws
       → Feeds same `Combatant`          (Feet, BladeTip, World)        • Blinks / Teleports
         hit resolution pipeline!                                       • Projectiles / Buffs
==========================================================================================

```

Notice the elegance here: **90% of your skills require zero custom GDScript.** A leaping overhead cleave, a spinning katana flourish, or a ground-shattering slam is simply a `.tres` card wired to an animation clip, an AoE hitbox prefab, and a VFX scene.

---

### ⚙️ 3. Modular Custom Scripts via `SkillBehaviour` (The 10% Wildcards)

What happens when you want to build *Mist Raven* (teleport behind an attacker), *Inescapable Frenzy* (grab an enemy by the helm), or *Rellana's Twin Moons* (levitate and drop two exploding spheres)?

Do **not** create fifteen separate FSM states (`GrabState`, `TeleportState`, `MoonSlamState`) that duplicate root motion, gravity, and hit-stop logic. Instead, use the **Strategy Pattern**—give `SkillProfile` one optional `@export var behaviour: SkillBehaviour` slot.

🧠 `strategy pattern`: *swapping out a small, self-contained logic module (a `Resource` script) at runtime so one core state (`AttackState`) can perform wildly different special mechanics without bloating.*

`SkillBehaviour` is a tiny base `Resource` class with five empty virtual hooks that `AttackState` calls automatically whenever `behaviour != null`:

* **`on_enter(ctx)`** → Runs on Frame 0 (e.g. deduct Spirit Seals, toggle brief levitation or i-frames).
* **`on_physics(delta, ctx)`** → Runs every physics tick (e.g. pull a grabbed victim smoothly to your hand socket, or glide upward during an aerial slam).
* **`on_timeline_event(event_name, ctx)`** → Fired directly from Godot's `AnimationPlayer` Call Method Track (`trigger_skill_event("drop_moon")`) so custom code stays frame-locked to your animation.
* **`on_hit_landed(victim, hit_pos, ctx)`** → Fires the exact frame a blade or grab box connects (e.g. lock the victim into a grab cinematically!).
* **`on_exit(ctx)`** → Guaranteed cleanup even if a boss smacks you out of the skill mid-windup (restores gravity, releases grabbed targets, clears temporary buffs).

*(Here `ctx` is just a lightweight dictionary or reference bundle containing `actor`, `visual_model`, `anim_player`, `combatant`, and `profile`—keeping every custom skill script completely decoupled from your player scene!)*

---

### 💥 4. Custom Hitboxes for AoE Attacks (Beyond Basic Slashes)

Your `WeaponTracer3D` is brilliant for razor-sharp katana arcs, because sweeping rays between `HiltMarker` and `TipMarker` matches steel geometry to the millimetre.

Here is the catch: **rays on a sword cannot represent a 5-metre shockwave ring, a cone of dragon flame, or an orbital pillar of light.**

Rather than shoving `if is_aoe:` radius math into `AttackState.gd`, we build **one reusable `AoEHitbox3D.gd` script** attached to simple `Area3D` `.tscn` prefabs:

* **How You Author One:** Create a mini scene—say `ShockwaveRing_Hitbox.tscn`—containing an `Area3D` + `CollisionShape3D` (Sphere, Cylinder, Box, or Cone) **plus its matching visual shader mesh** right inside the same scene. What you see is literally what hits!
* **How It Fires:** On your `AttackProfile` (or `SkillProfile`), add an `@export var use_weapon_tracer: bool = true` toggle and `@export var spawned_hitboxes: Array[PackedScene]`.
* **Timeline Control:** On the exact impact frame in `AnimationPlayer`, your Call Method Track calls `spawn_hitbox(0, &"Feet")` (or `&"BladeTip"` / `&"Forward"`). `AttackState` instantiates the prefab, hands it `attacker` + `current_profile`, and the `AoEHitbox3D` checks every `Hurtbox3D` inside its shape—routing straight into your existing `Combatant.take_hit()` pipeline so Deflects, Poise breaks, Afflictions, and Hit-Stop all work identically!


* **Built-In Multi-Hit Support:** Give `AoEHitbox3D` a `tick_interval` and `lifetime` export. Set `tick_interval = 0.0` for a single crisp slam explosion, or `0.15s` for a lingering *One Mind* slash dome or *Scarlet Aeonia* bloom.



---

### 🤼 5. How Grabs Work Cleanly (Without Soft-Locking Your Game)

Grabs are notorious for breaking indie action games because two characters suddenly have to move as one. With `SkillBehaviour` + your existing `custom_reaction_state` design, a grab stays remarkably tidy:

1. **Phase 1 — The Lunge & Grab Check:** Pressing `F` enters `AttackState` with `Grab_Profile.tres` (which carries `GrabBehaviour.gd`). Your timeline spawns a short-range `GrabBox` (`Area3D`) in front of your hands.
2. **Phase 2 — The Lock-In (`on_hit_landed`):** The moment `GrabBox` touches a valid enemy (`armour_tier < COLOSSAL` so you cannot suplex a 20-metre Tower Knight):
* `GrabBehaviour` forces the enemy's `StateMachine` into `&"GrabbedState"` (disabling their AI, gravity, and collision capsule temporarily).


* It jumps the player's `AnimationPlayer` to the synchronised grab-execution clip (`"Grab_Connect"`) and welds the enemy's transform to a `GrabAnchor` (`Marker3D`) on your player rig each tick inside `on_physics()`.


3. **Phase 3 — Climax & Guaranteed Release:**
* A Call Method key on the slam/stab frame calls `trigger_skill_event(&"grab_impact")` → deals the massive HP/Poise burst, camera trauma, and impact VFX.
* On `on_exit()` (whether the grab finishes naturally *or* another enemy knocks you out of it), `GrabBehaviour` **always** re-enables the victim's collider and boots them into `KnockdownState` so nobody ever gets permanently glued to your hand.





---

### ✨ 6. Modular VFX Spawning (Timeline & Socket Anchoring)

Just like your AoE hitboxes, VFX should never be hardcoded inside `AttackState.gd`. Instead, wire up **socket anchoring** alongside your choreographed one-shot VFX prefabs:

📌 `socket anchoring`: *registering named `Marker3D` points on your character (`Feet`, `Chest`, `BladeTip`, `HandRight`) in a simple Dictionary so any skill can spawn a shader ring or particle burst at the exact anatomical point without hardcoding coordinates.*

* **Timeline VFX (`spawn_vfx(index, socket_name, follow_socket)`):**
* Add `@export var vfx_scenes: Array[PackedScene]` onto `AttackProfile` / `SkillProfile`.
* Call `spawn_vfx(0, &"Feet", false)` from your animation timeline: `follow_socket = false` drops a ground crack or dust ring into world space so it stays planted on the arena floor while you keep moving; `follow_socket = true` parents an aura or flame trail directly to your blade or chest.
* Every VFX `.tscn` runs its own `AnimationPlayer` and calls `queue_free()` on finish, keeping `AttackState` completely hands-off.




* **Per-Move Impact VFX (`@export var impact_vfx: PackedScene`):**
* Spawned automatically inside `_on_hit_landed(_victim, _hit_pos, profile)` at `_hit_pos`—so light slashes spark clean steel whilst an *Ash-Lightning* skill detonates in crimson lightning without writing a single `if` check.



---

### 🪜 7. Your Phased Implementation Plan (Zero Regressions)

Do not build all five systems in one chaotic evening—roll them out in **five tight, testable phases** so your working katana loop never breaks:

1. **Phase 1 — Data & Input Wiring (`SkillProfile`, `WeaponData`, `PlayerLoadout`, X/C/F):**
* Create `SkillProfile.gd` (`extends AttackProfile`), `WeaponData.gd` (`skill_x`, `skill_c`), and `PlayerLoadout.gd` (`equipped_weapon`, `equipped_rite_f`).


* Register `skill_x`, `skill_c`, and `skill_f` in `InputBuffer.gd` and wire `LocomotionState` + `AttackState` combo-cancel hooks so pressing X, C, or F launches the equipped `SkillProfile` cleanly.




2. **Phase 2 — Universal Hitbox & VFX Spawner (`AoEHitbox3D` + Socket Registry):**
* Build `AoEHitbox3D.gd` (`Area3D`) that talks directly to `Hurtbox3D` / `Combatant`.
* Add a lightweight `VFXSockets` dictionary (`Feet`, `BladeTip`, `Chest`) plus `spawn_hitbox()` and `spawn_vfx()` Call Method hooks inside `AttackState.gd`.


3. **Phase 3 — Pluggable `SkillBehaviour` Hooks:**
* Create the 15-line `SkillBehaviour.gd` base `Resource` and wire its 5 lifecycle hooks (`on_enter`, `on_physics`, `on_timeline_event`, `on_hit_landed`, `on_exit`) into `AttackState.gd`.


4. **Phase 4 — Build Three Vertical-Slice Proof Skills (Proves Every Path):**
* **Slot `X` (Pure Data Skill — Zero Custom Script):** Fast lunging blade art reusing `WeaponTracer3D` + custom slash VFX.
* **Slot `C` (AoE Hitbox + VFX Skill):** Ground slam or spinning crescent spawning an `AoEHitbox3D` shockwave ring at `&"Feet"`.
* **Slot `F` (Custom `SkillBehaviour` Skill):** Either a short **Grab** (testing `GrabBehaviour` + `GrabbedState`) or a **Blink / Phase-Step** counter.


5. **Phase 5 — Live Validation Against `TestDummy.tscn` & Spirit Seal Economy:**
* Drop a reactive `TestDummy.tscn` (with `Combatant`, `Hurtbox3D`, and `GrabbedState`) into `CombatArena` to verify AoE multi-hits, Poise breaks, hit-stop, and Spirit Seal spending live.





---

### 🎯 Next Step

Does this **`SkillProfile extends AttackProfile` + `AoEHitbox3D` + `SkillBehaviour**` blueprint match the modularity you had in mind for *Ashen Kata*—and would you like to start immediately with **Phase 1 (creating `SkillProfile.gd`, `WeaponData.gd`, and wiring X/C/F into `InputBuffer` and `AttackState`)**, or drop a **`TestDummy.tscn`** into `CombatArena` first so we have a live punching bag to test every phase against?

✍️ **Phrasing Refinement:** "Everything is working as expected now. Could you lay out a plan for building our X, C, and F skills so they stay modular—supporting optional custom scripts, grabs, VFX, and custom AoE hitboxes rather than just basic blade slashes? Also, should skills have their own separate card resource, or should they be integrated directly into our existing `AttackProfile`?"

✅ **Summary:** Do **not** bloat `AttackProfile` with grab offsets and AoE radii, and do **not** build a disconnected skill card that forces you to duplicate your combat code. Instead, use **Inheritance + Composition**: create a **`SkillProfile` that extends `AttackProfile**` (so every skill automatically inherits hit-stop, Poise break, Impact Tiers, root motion, and DS3 combo redirection for free) and plug in **three modular slots**—an optional **`SkillBehaviour` script resource** (for grabs, teleports, or custom logic), a **`Hitbox` prefab slot** (for AoE spheres, cones, and shockwaves), and a **`VFX` socket array** triggered straight from your `AnimationPlayer` timeline.

---

### 🧩 1. The Big Question: Modify `AttackProfile` or Create a New Card?

Here is the trap most indie combat games fall into early on.

If you cram every skill feature—Spirit Seal costs, grab anchor offsets, projectile speeds, AoE radii—directly onto `AttackProfile.gd`, then every humble `Katana_Light_1.tres` becomes an ugly 40-field spreadsheet full of empty checkboxes.

Yet if you invent a completely separate `SkillData` card from scratch, your `WeaponTracer3D`, `Combatant`, and `AttackState` suddenly have to check *"Wait, am I holding an `AttackProfile` or a `SkillData`?"* at every single turn. That way lies spaghetti code.

So how do we get the best of both worlds? **Inheritance.**

🧬 `inheritance` (`class_name SkillProfile extends AttackProfile`): *a child resource that automatically possesses every single property of `AttackProfile`, whilst adding extra skill-only fields on top.*

Look at what happens the moment `SkillProfile` extends `AttackProfile`:

* `AttackState`, `Combatant`, and `WeaponTracer3D` never need to care whether you pressed LMB, X, C, or F—to them, it is still an `AttackProfile` carrying `hp_damage`, `poise_damage`, `attack_tier`, `armour_tier`, `affliction_type`, and `baked_lunge_distance`.


* Your basic light attacks (`Katana_Light_1/2/3.tres`) stay lean and uncluttered.
* Your **X & C Weapon Arts** (stored on `WeaponData`) and **F Ashen Rites / Boss Arts** (stored on `PlayerLoadout`) use `SkillProfile.tres`, unlocking extra fields like `seal_cost`, `cooldown`, `hitbox_prefab`, `vfx_scenes`, and `custom_behaviour`.



---

### 🗺️ 2. The Modular Architecture Blueprint

Before we touch a single script, picture how every piece plugs into the exact `AttackState` you already have working today:

```text
==========================================================================================
             ASHEN KATA — MODULAR X / C / F SKILL & HITBOX ARCHITECTURE
==========================================================================================

  [ INPUT LAYER ]             [ DATA LAYER (.tres) ]            [ EXECUTION LAYER ]
  ───────────────             ──────────────────────            ───────────────────
  LMB ──────────────────────► AttackProfile.tres ────────────┐
  X / C (Weapon Arts) ──────► WeaponData ──► SkillProfile ───┼─► AttackState.gd
  F     (Ashen Rite)  ──────► Loadout    ──► SkillProfile ───┘   (One Unified State!)
                                                  │
                ┌─────────────────────────────────┼──────────────────────────────┐
                ▼                                 ▼                              ▼
     [ HITBOX DELIVERY ]               [ CHOREOGRAPHED VFX ]          [ OPTIONAL BEHAVIOUR ]
     • Default: WeaponTracer3D         • `vfx_scenes[]` array         • `SkillBehaviour`
     • Custom : Spawn `AoEHitbox3D`      triggered via timeline         (Strategy Resource)
       (Sphere / Cone / Ring / Beam)     at named Rig Sockets           • Grabs & Throws
       → Feeds same `Combatant`          (Feet, BladeTip, World)        • Blinks / Teleports
         hit resolution pipeline!                                       • Projectiles / Buffs
==========================================================================================

```

Notice the elegance here: **90% of your skills require zero custom GDScript.** A leaping overhead cleave, a spinning katana flourish, or a ground-shattering slam is simply a `.tres` card wired to an animation clip, an AoE hitbox prefab, and a VFX scene.

---

### ⚙️ 3. Modular Custom Scripts via `SkillBehaviour` (The 10% Wildcards)

What happens when you want to build *Mist Raven* (teleport behind an attacker), *Inescapable Frenzy* (grab an enemy by the helm), or *Rellana's Twin Moons* (levitate and drop two exploding spheres)?

Do **not** create fifteen separate FSM states (`GrabState`, `TeleportState`, `MoonSlamState`) that duplicate root motion, gravity, and hit-stop logic. Instead, use the **Strategy Pattern**—give `SkillProfile` one optional `@export var behaviour: SkillBehaviour` slot.

🧠 `strategy pattern`: *swapping out a small, self-contained logic module (a `Resource` script) at runtime so one core state (`AttackState`) can perform wildly different special mechanics without bloating.*

`SkillBehaviour` is a tiny base `Resource` class with five empty virtual hooks that `AttackState` calls automatically whenever `behaviour != null`:

* **`on_enter(ctx)`** → Runs on Frame 0 (e.g. deduct Spirit Seals, toggle brief levitation or i-frames).
* **`on_physics(delta, ctx)`** → Runs every physics tick (e.g. pull a grabbed victim smoothly to your hand socket, or glide upward during an aerial slam).
* **`on_timeline_event(event_name, ctx)`** → Fired directly from Godot's `AnimationPlayer` Call Method Track (`trigger_skill_event("drop_moon")`) so custom code stays frame-locked to your animation.
* **`on_hit_landed(victim, hit_pos, ctx)`** → Fires the exact frame a blade or grab box connects (e.g. lock the victim into a grab cinematically!).
* **`on_exit(ctx)`** → Guaranteed cleanup even if a boss smacks you out of the skill mid-windup (restores gravity, releases grabbed targets, clears temporary buffs).

*(Here `ctx` is just a lightweight dictionary or reference bundle containing `actor`, `visual_model`, `anim_player`, `combatant`, and `profile`—keeping every custom skill script completely decoupled from your player scene!)*

---

### 💥 4. Custom Hitboxes for AoE Attacks (Beyond Basic Slashes)

Your `WeaponTracer3D` is brilliant for razor-sharp katana arcs, because sweeping rays between `HiltMarker` and `TipMarker` matches steel geometry to the millimetre.

Here is the catch: **rays on a sword cannot represent a 5-metre shockwave ring, a cone of dragon flame, or an orbital pillar of light.**

Rather than shoving `if is_aoe:` radius math into `AttackState.gd`, we build **one reusable `AoEHitbox3D.gd` script** attached to simple `Area3D` `.tscn` prefabs:

* **How You Author One:** Create a mini scene—say `ShockwaveRing_Hitbox.tscn`—containing an `Area3D` + `CollisionShape3D` (Sphere, Cylinder, Box, or Cone) **plus its matching visual shader mesh** right inside the same scene. What you see is literally what hits!
* **How It Fires:** On your `AttackProfile` (or `SkillProfile`), add an `@export var use_weapon_tracer: bool = true` toggle and `@export var spawned_hitboxes: Array[PackedScene]`.
* **Timeline Control:** On the exact impact frame in `AnimationPlayer`, your Call Method Track calls `spawn_hitbox(0, &"Feet")` (or `&"BladeTip"` / `&"Forward"`). `AttackState` instantiates the prefab, hands it `attacker` + `current_profile`, and the `AoEHitbox3D` checks every `Hurtbox3D` inside its shape—routing straight into your existing `Combatant.take_hit()` pipeline so Deflects, Poise breaks, Afflictions, and Hit-Stop all work identically!


* **Built-In Multi-Hit Support:** Give `AoEHitbox3D` a `tick_interval` and `lifetime` export. Set `tick_interval = 0.0` for a single crisp slam explosion, or `0.15s` for a lingering *One Mind* slash dome or *Scarlet Aeonia* bloom.



---

### 🤼 5. How Grabs Work Cleanly (Without Soft-Locking Your Game)

Grabs are notorious for breaking indie action games because two characters suddenly have to move as one. With `SkillBehaviour` + your existing `custom_reaction_state` design, a grab stays remarkably tidy:

1. **Phase 1 — The Lunge & Grab Check:** Pressing `F` enters `AttackState` with `Grab_Profile.tres` (which carries `GrabBehaviour.gd`). Your timeline spawns a short-range `GrabBox` (`Area3D`) in front of your hands.
2. **Phase 2 — The Lock-In (`on_hit_landed`):** The moment `GrabBox` touches a valid enemy (`armour_tier < COLOSSAL` so you cannot suplex a 20-metre Tower Knight):
* `GrabBehaviour` forces the enemy's `StateMachine` into `&"GrabbedState"` (disabling their AI, gravity, and collision capsule temporarily).


* It jumps the player's `AnimationPlayer` to the synchronised grab-execution clip (`"Grab_Connect"`) and welds the enemy's transform to a `GrabAnchor` (`Marker3D`) on your player rig each tick inside `on_physics()`.


3. **Phase 3 — Climax & Guaranteed Release:**
* A Call Method key on the slam/stab frame calls `trigger_skill_event(&"grab_impact")` → deals the massive HP/Poise burst, camera trauma, and impact VFX.
* On `on_exit()` (whether the grab finishes naturally *or* another enemy knocks you out of it), `GrabBehaviour` **always** re-enables the victim's collider and boots them into `KnockdownState` so nobody ever gets permanently glued to your hand.





---

### ✨ 6. Modular VFX Spawning (Timeline & Socket Anchoring)

Just like your AoE hitboxes, VFX should never be hardcoded inside `AttackState.gd`. Instead, wire up **socket anchoring** alongside your choreographed one-shot VFX prefabs:

📌 `socket anchoring`: *registering named `Marker3D` points on your character (`Feet`, `Chest`, `BladeTip`, `HandRight`) in a simple Dictionary so any skill can spawn a shader ring or particle burst at the exact anatomical point without hardcoding coordinates.*

* **Timeline VFX (`spawn_vfx(index, socket_name, follow_socket)`):**
* Add `@export var vfx_scenes: Array[PackedScene]` onto `AttackProfile` / `SkillProfile`.
* Call `spawn_vfx(0, &"Feet", false)` from your animation timeline: `follow_socket = false` drops a ground crack or dust ring into world space so it stays planted on the arena floor while you keep moving; `follow_socket = true` parents an aura or flame trail directly to your blade or chest.
* Every VFX `.tscn` runs its own `AnimationPlayer` and calls `queue_free()` on finish, keeping `AttackState` completely hands-off.




* **Per-Move Impact VFX (`@export var impact_vfx: PackedScene`):**
* Spawned automatically inside `_on_hit_landed(_victim, _hit_pos, profile)` at `_hit_pos`—so light slashes spark clean steel whilst an *Ash-Lightning* skill detonates in crimson lightning without writing a single `if` check.



---

### 🪜 7. Your Phased Implementation Plan (Zero Regressions)

Do not build all five systems in one chaotic evening—roll them out in **five tight, testable phases** so your working katana loop never breaks:

1. **Phase 1 — Data & Input Wiring (`SkillProfile`, `WeaponData`, `PlayerLoadout`, X/C/F):**
* Create `SkillProfile.gd` (`extends AttackProfile`), `WeaponData.gd` (`skill_x`, `skill_c`), and `PlayerLoadout.gd` (`equipped_weapon`, `equipped_rite_f`).


* Register `skill_x`, `skill_c`, and `skill_f` in `InputBuffer.gd` and wire `LocomotionState` + `AttackState` combo-cancel hooks so pressing X, C, or F launches the equipped `SkillProfile` cleanly.




2. **Phase 2 — Universal Hitbox & VFX Spawner (`AoEHitbox3D` + Socket Registry):**
* Build `AoEHitbox3D.gd` (`Area3D`) that talks directly to `Hurtbox3D` / `Combatant`.
* Add a lightweight `VFXSockets` dictionary (`Feet`, `BladeTip`, `Chest`) plus `spawn_hitbox()` and `spawn_vfx()` Call Method hooks inside `AttackState.gd`.


3. **Phase 3 — Pluggable `SkillBehaviour` Hooks:**
* Create the 15-line `SkillBehaviour.gd` base `Resource` and wire its 5 lifecycle hooks (`on_enter`, `on_physics`, `on_timeline_event`, `on_hit_landed`, `on_exit`) into `AttackState.gd`.


4. **Phase 4 — Build Three Vertical-Slice Proof Skills (Proves Every Path):**
* **Slot `X` (Pure Data Skill — Zero Custom Script):** Fast lunging blade art reusing `WeaponTracer3D` + custom slash VFX.
* **Slot `C` (AoE Hitbox + VFX Skill):** Ground slam or spinning crescent spawning an `AoEHitbox3D` shockwave ring at `&"Feet"`.
* **Slot `F` (Custom `SkillBehaviour` Skill):** Either a short **Grab** (testing `GrabBehaviour` + `GrabbedState`) or a **Blink / Phase-Step** counter.


5. **Phase 5 — Live Validation Against `TestDummy.tscn` & Spirit Seal Economy:**
* Drop a reactive `TestDummy.tscn` (with `Combatant`, `Hurtbox3D`, and `GrabbedState`) into `CombatArena` to verify AoE multi-hits, Poise breaks, hit-stop, and Spirit Seal spending live.





---

### 🎯 Next Step

Does this **`SkillProfile extends AttackProfile` + `AoEHitbox3D` + `SkillBehaviour**` blueprint match the modularity you had in mind for *Ashen Kata*—and would you like to start immediately with **Phase 1 (creating `SkillProfile.gd`, `WeaponData.gd`, and wiring X/C/F into `InputBuffer` and `AttackState`)**, or drop a **`TestDummy.tscn`** into `CombatArena` first so we have a live punching bag to test every phase against?