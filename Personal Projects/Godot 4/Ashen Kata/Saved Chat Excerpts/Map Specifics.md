### 🏛️ Executive Verdict & Architecture Overview

Adopting a **"Chokepoint Airlock" (or S-Bend Streaming Portal)** architecture is the definitive gold standard for action-adventure performance. By using winding caverns, deep ravines, or mountain tunnels as spatial buffers, you gain all the benefits of discrete scene loading without pulling the player out of their immersion. Mechanically, this functions identically to the classic airlocks of _Metroid Prime_, the elevator transitions of _Dark Souls_, and the interior door markers of _Grand Theft Auto V_: the physical environment is partitioned into independent cells, while the transitional hallway masks the asynchronous swapping of scenes and coordinate rebasing.

### 📖 Plain-English Architectural Glossary

- **Airlock Corridor (The S-Bend):** A winding, visually occluded hallway (such as an S-shaped cave) connecting two massive combat arenas, designed to completely block line-of-sight between them.
    
- **Coordinate Rebasing / Teleportation:** Moving an entity's `global_position` instantaneously to a corresponding doorway marker in a secondary scene without resetting local variables like velocity or camera yaw.
    
- **Asynchronous Scene Streaming:** Background loading of a `PackedScene` on a secondary CPU thread (`ResourceLoader.load_threaded_request`) so the active frame rate never drops or stutters whilst fetching assets from the storage drive.
    
- **A/B Double Buffering:** Keeping only two zones in memory simultaneously: Zone A (the current sector) and Zone B (the approaching sector), immediately freeing Zone A the moment the airlock closes behind the player.
    

### 🗺️ The Spatial Mechanics of a Chokepoint Portal

```
=============================================================================
               THE CHOKEPOINT AIRLOCK STREAMING PIPELINE
=============================================================================

 [ SECTOR A: Sunken Canyon ]
             │
             ▼  (Player runs toward mountain)
 ┌────────────────────────────────────────────────────────┐
 │ TRIGGER 1: Approach Volume (Area3D)                    │
 │ ↳ Fires: ResourceLoader.load_threaded_request("ZoneB") │
 └───────────────────────┬────────────────────────────────┘
                         │
                         ▼
 ┌────────────────────────────────────────────────────────┐
 │ THE AIRLOCK: Winding S-Bend Cavern                     │
 │ ↳ Line-of-sight is 100% occluded by cave geometry      │
 │ ↳ Audio transitions: Wind fades -> Echoes amplify      │
 └───────────────────────┬────────────────────────────────┘
                         │
                         ▼
 ┌────────────────────────────────────────────────────────┐
 │ TRIGGER 2: Teleport / Handover Volume (Area3D)         │
 │ ↳ Instantiates Sector B into Scene Tree                │
 │ ↳ Repositions Player to Sector B's Entry Marker3D      │
 │ ↳ Frees Sector A from VRAM (queue_free)                │
 └───────────────────────┬────────────────────────────────┘
                         │
                         ▼
 [ SECTOR B: Scorched Ridge Arena ]
```

### ⚙️ Why This Pattern Succeeds (The Two Paradigms)

Depending on your artistic and technical preference, this chokepoint pattern can be executed in two distinct flavours:

#### 1. The Seamless S-Bend (The _Metroid Prime_ / _Dark Souls_ Model)

- **How it works:** The cave has an entrance in Sector A and an identical exit in Sector B. The player never stops walking.
    
- **The trick:** As the player rounds the middle curve of the cave where neither exterior is visible, the engine loads Sector B, attaches it to the world root, translates Sector B's entrance to match the cave's exact world coordinates, and deletes Sector A behind them.
    
- **Player experience:** 100%100\%100% seamless locomotion with zero loading screens, zero fade-to-black, and zero immersion breaks.
    

#### 2. The Interior Warp Threshold (The _GTA V_ / _Skyrim_ Model)

- **How it works:** When entering a cave mouth or ruined archway, walking into the dark trigger volume initiates a brief 0.3–0.5-second0.3\text{--}0.5\text{-second}0.3–0.5-second visual cinematic (e.g. screen vignette dip, dust haze, or narrow passage squeeze animation).
    
- **The trick:** The player is decoupled, the current map is unloaded, the new map is loaded into memory, and the player is spawned at `Marker3D_Arrival` facing forward with matching velocity.
    
- **Player experience:** Clean, deliberate pacing that allows radical aesthetic shifts (e.g. stepping from a sunny desert canyon directly into a glowing subterranean crystal cavern) without needing to worry about matching horizon lines or lighting bleed.
    

### 🛠️ Production Architecture: The `ZonePortal3D` Node

To implement this modularly in Godot 4 without spaghetti code, build a dedicated, reusable portal component (`ZonePortal3D.tscn`) composed of an `Area3D` collision volume and a target destination property.

#### 1. The World Root Structure

```
res://scenes/
├── core/
│   └── WorldRoot.tscn       <-- Master container node
│       ├── Player           <-- Persistent Player node (never deleted)
│       └── ActiveZone       <-- Holds the currently loaded Sector scene
```

#### 2. The `ZonePortal3D.gd` Script

Attach this script to an `Area3D` situated inside your connecting tunnel or cave mouth:

GDScript

```
class_name ZonePortal3D
extends Area3D

## The scene path to the connected sector
@export_file("*.tscn") var target_zone_path: String = ""

## Unique name or NodePath of the arrival Marker3D in the destination zone
@export var arrival_marker_tag: String = "Entrance_From_Canyon"

## Optional: Display a brief atmospheric fade rather than an instant snap
@export var use_cinematic_fade: bool = true

func _ready() -> void:
	body_entered.connect(_on_body_entered)

func _on_body_entered(body: Node3D) -> void:
	if not body.is_in_group(&"player"):
		return
		
	# Inform the autoload WorldDirector to execute the zone handover
	WorldDirector.transition_to_zone(target_zone_path, arrival_marker_tag, use_cinematic_fade)
```

#### 3. The `WorldDirector.gd` (Autoload Singleton)

This script manages scene instancing, background loading, and coordinate rebasing:

GDScript

```
extends Node

var _target_zone_path: String = ""
var _arrival_marker_tag: String = ""
var _is_transitioning: bool = false

func transition_to_zone(zone_path: String, marker_tag: String, with_fade: bool) -> void:
	if _is_transitioning:
		return
	_is_transitioning = true
	_target_zone_path = zone_path
	_arrival_marker_tag = marker_tag

	if with_fade:
		# Trigger UI fade out
		await ScreenFader.fade_out(0.4)

	# 1. Background load the incoming sector
	ResourceLoader.load_threaded_request(_target_zone_path)
	
	# 2. Wait until loaded
	while ResourceLoader.load_threaded_get_status(_target_zone_path) != ResourceLoader.THREAD_LOAD_LOADED:
		await get_tree().process_frame

	var packed_zone: PackedScene = ResourceLoader.load_threaded_get_resource(_target_zone_path)
	var new_zone_instance: Node3D = packed_zone.instantiate()

	# 3. Swap the active zone under the WorldRoot
	var world_root: Node = get_tree().root.get_node("WorldRoot")
	var old_zone: Node = world_root.get_node("ActiveZone")
	old_zone.queue_free()
	
	new_zone_instance.name = "ActiveZone"
	world_root.add_child(new_zone_instance)

	# 4. Teleport the persistent player to the arrival marker
	var arrival_marker: Marker3D = new_zone_instance.find_child(_arrival_marker_tag, true, false)
	var player: CharacterBody3D = world_root.get_node("Player")
	
	if arrival_marker and player:
		player.global_transform = arrival_marker.global_transform

	if with_fade:
		# Restore sight
		await ScreenFader.fade_in(0.4)
		
	_is_transitioning = false
```

### 💡 Core Takeaways for Your Development Pipeline

- **Zero Asset Overlap:** Sector A and Sector B do not need to exist in the same coordinate space. Sector A can be positioned at `(0, 0, 0)` in its own scene file, and Sector B can also be built at `(0, 0, 0)` in its scene file. The arrival `Marker3D` handles all coordinate translation seamlessly.
    
- **Total VRAM Flushes:** The moment `old_zone.queue_free()` is invoked, Godot's rendering engine drops all terrain textures, shadow buffers, and static meshes belonging to the previous area, freeing up your Intel Iris Xe integrated graphics to deliver locked 60 FPS in the new zone.
    
- **Environmental Logic:** Design your cave chokepoints with an intentional **S-curve**. Straight hallways allow sunlight and distant geometry to bleed across the threshold; a double-corner curve guarantees that neither the player nor the camera can peek into the unloaded sector before the handover completes.


### 🏛️ The Architecture of the Master Prompt

To prevent AI models from generating generic, bloated Soulslike tutorials or breaking your existing codebase, the prompt is structured into five cascading layers of enforcement:

```
=============================================================================
                  MASTER PROMPT ENFORCEMENT HIERARCHY
=============================================================================
  [ 1. Identity & Core Vision ]   ──► Phantom Blade Zero + Sekiro + Monster Hunter
                 │
                 ▼
  [ 2. Locomotion Kinematics ]    ──► CharacterBody3D + Unified Vector2 Accel
                 │
                 ▼
  [ 3. Combat & Sha-Chi Engine ]  ──► Bidirectional Gauge + Glints + Ghoststep
                 │
                 ▼
  [ 4. World & Streaming Spec ]   ──► Wide-Linear Sectors + Chokepoint Airlocks
                 │
                 ▼
  [ 5. Hardware & Coding Rules ]  ──► Strict GDScript 2.0 + Intel Iris Xe Limits
=============================================================================
```

### 📜 Copy-Paste Master Reference Prompt

Copy the entire block below and save it as `MASTER_SYSTEM_PROMPT.md`:

Markdown

```
# SYSTEM CONTEXT: GODOT 4.x 3D WUXIA-SOULSLIKE COMBAT & WORLD ARCHITECTURE

## 1. PROJECT IDENTITY & DESIGN PHILOSOPHY
You are the Lead Systems Architect for a 3D third-person action RPG built in **Godot 4.x (GDScript 2.0)**. 
The game's mechanical identity is a high-speed, rhythmic Wuxia action hybrid ("Kung-Fu Punk") that synthesises four specific pillars:
1. **Phantom Blade Zero & Sekiro:** Intimate, face-to-face rhythmic deflection, low-commitment light attack strings, and high-speed orbital counter-vaults (*Ghoststep*).
2. **Elden Ring & Monster Hunter:** Multi-target spatial positioning, deliberate heavy-weapon commitment, and varied weapon archetype movesets.
3. **The Witcher 3 & Assassin's Creed Odyssey:** Snappy, mid-combo tactical secondary abilities (*Phantom Edges* / Signs) that integrate seamlessly without breaking parry flow.
4. **Dragon's Dogma: Dark Arisen & Monster Hunter Rise:** A "Wide-Linear" macro-corridor world structure consisting of self-contained combat clearings connected by winding chokepoints (caves, ravines, tunnels) that asynchronously stream adjacent sectors and flush VRAM.

---

## 2. LOCOMOTION & KINEMATIC CONTROLLER INVARIANTS
The player is driven by a custom `CharacterBody3D` script. Never use artificial, instant-stop arcade physics; always preserve momentum and inertia:
- **Coordinate Translation:** Always transform 2D input (`Input.get_vector`) into world space via the character's rotational basis:
  `var move_direction: Vector3 = (transform.basis * Vector3(input_2d.x, 0.0, input_2d.y)).normalized()`
  *(Remember: Local Forward in Godot is strictly `-transform.basis.z`).*
- **Unified Planar Acceleration (No Manhattan Dog-Legging):** Never accelerate `velocity.x` and `velocity.z` on separate scalar lines. Always combine horizontal velocity into a single `Vector2(velocity.x, velocity.z)` and interpolate toward `target_planar` using `Vector2.move_toward(target_planar, current_accel * delta)`.
- **Decoupled Ground vs. Air Authority:**
  - `ground_acceleration` (~24.0 - 30.0): Snappy, high-traction starts, stops, and pivots when `is_on_floor() == true`.
  - `air_acceleration` (~4.0 - 6.0): Preserves launch momentum when airborne; directional inputs mid-air act only as subtle trajectory nudges.
- **Vertical Grounding Clamp:** When `is_on_floor()` is `true` and not jumping, clamp `velocity.y = 0.0` (or a small negative snap bias) to eliminate phantom polygon-edge bouncing.
- **Camera Rig:** Driven by `_unhandled_input` using `InputEventMouseMotion` when `Input.MOUSE_MODE_CAPTURED`. Horizontal yaw rotates the `CharacterBody3D` (`rotate_y`); vertical pitch rotates the child `SpringArm3D` (`rotate_x`), clamped via `clampf(spring_arm.rotation.x, min_pitch, max_pitch)`.

---

## 3. COMBAT ENGINE & THE "SHA-CHI" RESOURCE LOOP
We do NOT use a passive Soulslike stamina bar that taxes running or rolling. We use a unified, bidirectional **Sha-Chi (Killing Intent)** gauge (`0.0` to `100.0`):
- **Zero Locomotion Tax:** Sprinting, jumping, and standard evasive steps cost `0.0` Sha-Chi.
- **Defensive Drain vs. Offensive Spend:**
  - Holding Guard continuously drains Sha-Chi and absorbs hit damage as Sha-Chi loss. Hitting `0.0` triggers a **Guard-Broken Stagger State**.
  - Executing Heavy Combos (Sha-Chi Combos) or Weapon Arts actively spends Sha-Chi.
- **Active Recovery Mechanics:**
  - **Perfect Deflection (Tap Guard within ~150ms of impact):** Costs `0.0` Sha-Chi, deals heavy posture/Sha-Chi damage to the attacker, and can cancel any Light Attack frame.
  - **Flux Weapon Swap (Mid-Combo Switch):** Swapping between `Primary_Weapon_A` (Fast/Agile, zero commitment) and `Primary_Weapon_B` (Heavy/Stagger, high commitment) mid-string acts as an aggressive Ki-Pulse, instantly restoring +30% to +40% Sha-Chi (governed by an internal cooldown).
- **Phantom Edges (Secondary Tools):** Mapped to a dedicated skill input, consuming **Spark Charges** rather than Sha-Chi to provide crowd control, ranged pressure, or hyper-armour breaks.

---

## 4. THE THREAT GLINT MATRIX & "GHOSTSTEP" MECHANICS
Enemy `AttackProfile` resources carry a telegraph enum (`NONE`, `BLUE_PARRY`, `RED_DODGE`, `BLACK_CRUSH`):
1. **Standard Strike (No Glint):** Can be blocked, parried, or dodged normally.
2. **Brutal Move (Blue Glint):** High-momentum flurry. Parrying at the exact impact frame triggers **Ghoststep** and shatters enemy Sha-Chi.
3. **Killer Move (Red Glint):** Unblockable and unparryable sweep/grab. Must be answered with a timed **Perfect Dodge** right before contact to trigger **Ghoststep**.
4. **Crushing Blow (Black Glint):** Guard-shattering blow. Blocking depletes almost all Sha-Chi; must be out-spaced or absorbed and immediately recovered via a **Flux Weapon Swap**.
- **Ghoststep Execution:** Grants `0.35s` invulnerability and interpolates the player along a **Cubic Bézier** spline behind the attacker (`enemy.global_transform.basis.z * offset`), auto-rotating the player's basis to face the enemy's back for an immediate counter-slash.

---

## 5. ANIMATION CANCELLATION HIERARCHY & SOLO PIPELINE
- **Cancellation Pyramid (Lowest to Highest Priority):**
  `[0: Idle/Move] -> [1: Light Combo] -> [2: Heavy Sha-Chi Combo] -> [3: Flux Swap Attack] -> [4: Phantom Edge] -> [5: Deflect / Dodge / Ghoststep]`.
  *Rule:* Tier 5 (Deflect/Dodge) can cancel out of Tier 1 (Light Combo) startup or recovery at any frame. Tier 2 (Heavy Combo) requires strict positional commitment and cannot be parry-canceled mid-swing.
- **Animation Pipeline:** Built for a solo developer using retargeted humanoid mocap (`Godot 4 BoneMap` + `AnimationTree` StateMachines) enhanced in code via **Playback Speed Warping** (fast startups), **Hit-Stop** (`speed_scale = 0.0` for `0.06s - 0.10s` on impact), directional camera impulses, and bright deflection spark VFX.

---

## 6. WORLD TOPOLOGY & "CHOKEPOINT AIRLOCK" STREAMING
- **Wide-Linear Macro-Corridors:** The world is divided into discrete, self-contained biome sectors (`PackedScene` files). Each sector consists of 12m-18m throttles (for skirmisher mobs) opening into 35m-50m pocket arenas (for mini-bosses and bosses), bordered by natural rock cliffs (`StaticBody3D` + `OccluderInstance3D`).
- **Chokepoint Airlock Transitions:** Sectors connect via winding S-bend caves, ravines, or interior thresholds (`ZonePortal3D`).
- **Memory Management:** Entering an airlock triggers `ResourceLoader.load_threaded_request()` via a persistent `WorldDirector` autoload. Upon crossing the handover threshold, the new sector instantiates, the persistent `Player` node teleports to the destination `Marker3D`, and the previous sector calls `queue_free()` to completely flush VRAM.

---

## 7. HARDWARE CONSTRAINTS & GDSCRIPT 2.0 CODING STANDARDS
- **Target Baseline Hardware:** Must maintain a locked **60 FPS on Intel Iris Xe Integrated Graphics** (8GB RAM laptop environment) while scaling cleanly to desktop GPUs.
  - Avoid full-screen alpha overdraw and excessive triplanar shader sampling.
  - Keep `DirectionalLight3D` shadow cascades to 2-3 max, shadow resolution at `1024` or `2048`, max distance ~40m-50m, and disable shadow casting on small foliage/clutter.
- **Strict GDScript 2.0 Standards:**
  - Always use **strict static typing** (`var speed: float = 6.0`, `func _physics_process(delta: float) -> void:`).
  - Always use interned **`StringName` literals** (`&"move_left"`, `&"jump"`) for input polling, groups, and signal dispatches.
  - Maintain clean lifecycle separation: one-off setup/wiring in `_ready()`, discrete input triggers in `_unhandled_input()`, and continuous kinematic integration in `_physics_process()`.
```