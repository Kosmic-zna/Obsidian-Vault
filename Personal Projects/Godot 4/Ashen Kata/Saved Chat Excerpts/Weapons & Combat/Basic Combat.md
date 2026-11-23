### 📋 Executive Summary & Architectural Verdict

- ✅ **State Management:** We use a **Node-Based Pushdown Automaton (Hierarchical State Machine)**. Locomotion (`Idle`, `Move`, `Attack`) operates on the base state layer, whilst instantaneous interrupts (`Deflect`, `Ghoststep`, `HitStun`) **push** onto a memory stack, pausing or cancelling the active swing and popping cleanly back when finished.
    
- ✅ **Weapon & Skill Data:** Do **not** use a monolithic global dictionary/spreadsheet, and do **not** hardcode stats inside individual weapon scripts. Use **Self-Contained Custom Resources (`.tres` Data-Driven Flyweight Pattern)**. One universal `WeaponController.gd` reads swappable `WeaponData.tres` files, which hold arrays of `AttackProfile.tres` resources.
    
- ✅ **Hit Detection:** Do **not** spawn and destroy physics nodes mid-combat (`instantiate()` causes memory allocation spikes). Attach a persistent **`Hitbox3D` (`Area3D`)** directly to the weapon bone (`BoneAttachment3D`), keep its collision shape disabled (`disabled = true`) by default, and toggle it on/off on exact animation frames via an `AnimationPlayer` method/property track.
    

### 📖 Plain-English Architectural Glossary

- **Decoupling (Separation of Concerns):** Designing components so they have zero awareness of who or what they are interacting with (an analogy: a wall socket provides electricity whether you plug in a toaster or a television; the socket does not check `if device == Toaster`).
    
- **Custom `Resource` (`.tres`):** Godot's native data-container object that lives on your hard drive as a lightweight configuration file rather than a scene node. It stores pure numbers, strings, and references (like a digital trading card containing stats) that any script can read without duplicating memory.
    
- **Pushdown Automaton (State Stack):** A state machine upgraded with a "Last-In, First-Out" memory stack. Instead of overwriting the previous state permanently, high-priority states (like a _Sekiro_ deflect) sit on top of the stack and pop off when finished, returning control automatically.
    
- **Hitbox (`Area3D`) vs. Hurtbox (`Area3D`):** Two distinct sensor volumes separated from physical movement collision (`CharacterBody3D`). A **Hitbox** represents the lethal edge of a blade ("I deal damage"); a **Hurtbox** represents the vulnerable flesh of an actor ("I receive damage").
    
- **Frame Data (Startup, Active, Recovery):** The three temporal phases of any martial arts action. **Startup** is the wind-up before the blade is lethal; **Active** is the brief window where the hitbox sensor is live; **Recovery** is the follow-through where the character pulls their weapon back into stance.
    

### 🏛️ 1. Why AAA Studios Reject Monolithic Scripts (The Scalability Trap)

When transitioning from smaller prototypes to a multi-weapon combat game, novices invariably write what software engineers call a **"God Object"**—stuffing movement, combo counters, parry timers, and weapon damage directly into `playerMovement.gd` using giant `if/else` chains:

```
=============================================================================
               THE NOVICE "SPAGHETTI" TRAP vs. AAA MODULARITY
=============================================================================

 ❌ MONOLITHIC "GOD SCRIPT" (Brittle & Unscalable):
 [ playerMovement.gd (3,000+ Lines) ]
   ├── if is_attacking and not is_staggered and not is_parrying:
   ├── if equipped_weapon == "Katana": damage = 25
   └── Result: Adding 1 new skill breaks 4 existing movement states!

 ✅ AAA COMPONENT & DATA-DRIVEN ARCHITECTURE (Modular & Scalable):
 [ Player (CharacterBody3D) ]  ◄── Purely handles velocity & physics!
   ├── [ StateMachine ]        ◄── Arbitrates which single State owns the frame
   ├── [ ShaChiComponent ]     ◄── Purely manages posture/resource math
   ├── [ Hurtbox3D ]           ◄── Listens for incoming strikes & checks parry window
   └── [ WeaponSocket ]        ◄── Reads equipped WeaponData.tres & toggles Hitbox3D
```

#### Why This Modularity Matters for Your Game:

1. **Player/Enemy Symmetry:** Because `ShaChiComponent`, `Hurtbox3D`, and `Hitbox3D` are standalone Lego bricks that do not depend on keyboard input, you can drag and drop the **exact same scenes** onto a Boss or Minion. Your boss uses the exact same parry, posture, and hitbox scripts as your player!
    
2. **Zero Regression Bugs:** Modifying how a Greatsword calculates stagger inside its `.tres` file will never accidentally break your `SpringArm3D` camera or air acceleration math.
    

### 🗂️ 2. Weapon & Skill Data: Global Table vs. Self-Contained Data?

You asked a critical architectural question: _Should we use a central weapon/skill table, or have every weapon contain its own data?_

#### The Verdict: Self-Contained Custom Resources (`.tres`), Driven by ONE Universal Script

In Roblox (Luau), developers frequently build a gigantic `ModuleScript` containing a massive nested table (`WeaponTable["TwinBlades"].Damage = 15`). In Godot 4, a monolithic dictionary is an anti-pattern because it loads every weapon's meshes, icons, and animations into RAM simultaneously and cannot be edited cleanly in the visual Inspector.

Conversely, writing a separate `.gd` script for every weapon (`Katana.gd`, `Greatsword.gd`, `Halberd.gd`) leads to massive code duplication.

Instead, AAA engines use the **Flyweight Resource Pattern**:

```
=============================================================================
                    THE FLYWEIGHT WEAPON DATA HIERARCHY
=============================================================================
  [ WeaponData.gd (Blueprint) ]           [ AttackProfile.gd (Blueprint) ]
               │                                         │
       (Saved as .tres files)                    (Saved as .tres files)
               ▼                                         ▼
  📄 TwinBlades.tres  ─────────────────►  ├── 📄 Twin_Light_01.tres (0.0 Sha-Chi)
  📄 Greatsword.tres                      ├── 📄 Twin_Light_02.tres (0.0 Sha-Chi)
  📄 DragonHead_Skill.tres                └── 📄 Twin_Heavy_01.tres (25.0 Sha-Chi)
```

#### Code Implementation: `AttackProfile.gd` & `WeaponData.gd`

Create a folder `res://core/combat/resources/` and save these two small scripts. Every weapon and active skill in your entire game will be built from these two files without writing a single line of extra weapon code:

GDScript

```
# res://core/combat/resources/AttackProfile.gd
class_name AttackProfile
extends Resource

enum ThreatGlint { NONE, BLUE_PARRY, RED_DODGE, BLACK_CRUSH }

@export_group("Animation & Timing")
@export var anim_name: StringName = &"slash_1"
@export var can_cancel_into_parry: bool = true  ## True for Light attacks, False for Heavy!

@export_group("Damage & Economy")
@export var hp_damage: float = 15.0
@export var sha_chi_damage: float = 20.0        ## Posture damage dealt to target's block/parry
@export var sha_chi_cost: float = 0.0           ## 0 for Light strings, >0 for Heavy combos
@export var threat_glint: ThreatGlint = ThreatGlint.NONE

@export_group("Physics & Juice")
@export var forward_lunge_impulse: float = 4.0  ## Forward step during the swing
@export var hit_stop_duration: float = 0.07     ## Micro-freeze on blade impact
```

GDScript

```
# res://core/combat/resources/WeaponData.gd
class_name WeaponData
extends Resource

@export var weapon_name: StringName = &"Unnamed Weapon"
@export var weapon_mesh: PackedScene                ## The 3D model spawned in the player's hand
@export var light_combo_chain: Array[AttackProfile] ## Ordered list of light swings (1 -> 2 -> 3)
@export var heavy_combo_chain: Array[AttackProfile] ## Ordered list of Sha-Chi heavy swings
@export var flux_swap_attack: AttackProfile         ## The attack fired when swapping mid-combo
```

**Why this is superior:** To create 30 weapons in your game, you right-click in the FileSystem dock $\rightarrow$ **New Resource** $\rightarrow$ `WeaponData`, drag in a 3D mesh, and fill out the damage numbers in the Inspector. **Zero new scripts required.**

### ⚙️ 3. The State Machine & Cancellation Architecture

In a _Phantom Blade_ / _Sekiro_ combat engine, standard state machines fail because they treat every state equally. If you are in `AttackState`, a rigid state machine locks you in until the animation ends.

To support our **Cancellation Priority Pyramid**, each state declares a `priority_tier` integer. A new state can only interrupt the current state if its priority is higher, OR if the current attack's `can_cancel_into_parry` flag is `true`:

```
=============================================================================
                  STATE MACHINE INTERRUPT ARBITRATION
=============================================================================
  [ Active State: LightAttack (Tier 1) ]  ──► AttackProfile.can_cancel_into_parry == true
                     │
                     ▼  (Player taps RMB / Guard!)
  [ Requested State: DeflectState (Tier 5) ]
                     │
                     ▼  (Tier 5 > Tier 1 AND Cancel Allowed!)
  ✅ INTERRUPT GRANTED: Immediately cut attack animation & raise blade to Parry!
```

#### Code Implementation: `CombatState.gd` & `StateMachine.gd`

GDScript

```
# res://core/fsm/CombatState.gd
class_name CombatState
extends Node

## Reference injected automatically by the parent StateMachine
var actor: CharacterBody3D
var state_machine: Node

## Cancellation Tier: 0=Locomotion, 1=LightAttack, 2=HeavyAttack, 5=Deflect/Dodge
@export var priority_tier: int = 0
@export var is_interruptible_by_deflect: bool = true

func enter(_msg: Dictionary = {}) -> void:
	pass

func exit() -> void:
	pass

func physics_update(_delta: float) -> void:
	pass

func handle_input(_event: InputEvent) -> void:
	pass
```

GDScript

```
# res://core/fsm/StateMachine.gd
class_name StateMachine
extends Node

@export var initial_state: CombatState
var current_state: CombatState
var state_stack: Array[CombatState] = []

func _ready() -> void:
	await owner.ready
	for child in get_children():
		if child is CombatState:
			child.actor = owner as CharacterBody3D
			child.state_machine = self
	
	if initial_state:
		current_state = initial_state
		current_state.enter()

func _unhandled_input(event: InputEvent) -> void:
	if current_state:
		current_state.handle_input(event)

func _physics_process(delta: float) -> void:
	if current_state:
		current_state.physics_update(delta)

## Standard transition (Replaces active state if priority allows)
func transition_to(target_state_name: StringName, msg: Dictionary = {}) -> bool:
	var target_state: CombatState = get_node_or_null(NodePath(target_state_name))
	if not target_state or target_state == current_state:
		return false
		
	# Enforce the Cancellation Priority Pyramid
	if target_state.priority_tier < current_state.priority_tier:
		return false
	if target_state.priority_tier == 5 and not current_state.is_interruptible_by_deflect:
		return false
		
	current_state.exit()
	current_state = target_state
	current_state.enter(msg)
	return true
```

### 🗡️ 4. Hit Detection: How Hitboxes & Parrying Actually Work

Never use `instantiate()` to spawn a hitbox every time you swing a sword, and never use continuous raycasts unless simulating high-velocity bullets. Instead, AAA melee games use **Persistent Bone-Attached Volume Sensors (`Area3D`)** combined with an **Already-Hit Registry**.

```
=============================================================================
            THE ANATOMY OF A MELEE SWING & DEFLECTION RESOLUTION
=============================================================================
  [ Animation Timeline (0.50 Seconds Total) ]
  0.00s ──────► 0.12s ═════════════════════════► 0.28s ─────────────► 0.50s
  [  STARTUP  ]       [ ACTIVE HITBOX WINDOW ]         [  RECOVERY  ]
  (Hitbox OFF)        (Hitbox Collision = ON!)         (Hitbox OFF)
                                 │
                                 ▼ (Overlaps Enemy Hurtbox3D)
                      ┌──────────────────────┐
                      │ Is Target Parrying?  │
                      └──┬────────────────┬──┘
                    YES  │                │  NO
                         ▼                ▼
             [ DEFLECT TRIGGERED! ]   [ DEAL HP & SHA-CHI DMG ]
             - 0 HP Damage taken      - Subtract HP
             - Spark VFX + Clack SFX  - Trigger Hit-Stop (0.07s)
             - Attacker loses Sha-Chi - Add Victim to `hit_targets` array!
```

#### Why the `hit_targets` Array is Mandatory:

An attack's active window lasts roughly `0.15 seconds` (about 9 physics frames at 60 FPS). Without a tracking list, your sword would damage the enemy **9 separate times in a single swing**, instantly killing them. By appending each struck target into an array (`_hit_targets.append(hurtbox)`) and clearing that array when the swing ends, a single swing guarantees **exactly one hit per enemy**.

#### Code Implementation: `Hitbox3D.gd` & `Hurtbox3D.gd`

GDScript

```
# res://core/combat/Hitbox3D.gd
class_name Hitbox3D
extends Area3D

@export var collision_shape: CollisionShape3D

var active_profile: AttackProfile
var _hit_targets: Array[Area3D] = []

func _ready() -> void:
	# Hitboxes ONLY detect Hurtboxes; they never collide with walls or floors
	monitoring = true
	monitorable = false
	collision_shape.disabled = true
	area_entered.connect(_on_area_entered)

## Called via AnimationPlayer Method Track or State script at the start of the Active window
func activate_hitbox(profile: AttackProfile) -> void:
	active_profile = profile
	_hit_targets.clear()
	collision_shape.set_deferred(&"disabled", false)

## Called when the Active window closes or if the attacker is interrupted!
func deactivate_hitbox() -> void:
	collision_shape.set_deferred(&"disabled", true)
	_hit_targets.clear()

func _on_area_entered(area: Area3D) -> void:
	if area.owner == owner or _hit_targets.has(area):
		return # Never hit yourself, and never hit the same enemy twice in one swing!
		
	if area is Hurtbox3D:
		_hit_targets.append(area)
		area.receive_strike(active_profile, owner)
```

GDScript

```
# res://core/combat/Hurtbox3D.gd
class_name Hurtbox3D
extends Area3D

signal strike_deflected(attacker: Node3D, profile: AttackProfile)
signal strike_blocked(attacker: Node3D, profile: AttackProfile)
signal strike_taken(attacker: Node3D, profile: AttackProfile)

## Temporal window (in seconds) for a Sekiro/Phantom Blade Perfect Deflect
const PARRY_WINDOW_DURATION: float = 0.16

var is_guarding: bool = false
var is_invulnerable: bool = false
var _parry_timer: float = 0.0

func _physics_process(delta: float) -> void:
	if _parry_timer > 0.0:
		_parry_timer -= delta

## Called the exact frame the player taps the Guard/Parry button (RMB / L1)
func start_guard() -> void:
	is_guarding = true
	_parry_timer = PARRY_WINDOW_DURATION

func release_guard() -> void:
	is_guarding = false
	_parry_timer = 0.0

func receive_strike(profile: AttackProfile, attacker: Node3D) -> void:
	if is_invulnerable:
		return # Player is mid-Ghoststep!

	# 1. Check for Red Glint (Unblockable / Unparryable Killer Move)
	if profile.threat_glint == AttackProfile.ThreatGlint.RED_DODGE:
		strike_taken.emit(attacker, profile)
		return

	# 2. Check for Frame-Perfect Deflection (Tap Guard within 160ms)
	if is_guarding and _parry_timer > 0.0:
		_parry_timer = 0.0 # Consume the parry window
		strike_deflected.emit(attacker, profile)
		return

	# 3. Check for Standard Block (Holding Guard after the 160ms window expired)
	if is_guarding:
		strike_blocked.emit(attacker, profile)
		return

	# 4. Unprotected Direct Hit
	strike_taken.emit(attacker, profile)
```

### 💡 Key Takeaways & How Your Player Scene Tree Looks Now

Look at how clean and modular your `MainPlayer.tscn` hierarchy becomes with this pattern:

```
Player (CharacterBody3D - playerMovement.gd)
├── CollisionShape3D (Physical World Capsule)
├── SpringArm3D
│   └── Camera3D
├── Hurtbox3D (Area3D - Listens for incoming attacks & checks _parry_timer)
│   └── CollisionShape3D
├── WeaponPivot (Or BoneAttachment3D on character hand)
│   └── Hitbox3D (Area3D - Toggled ON/OFF during sword swings)
│       └── CollisionShape3D
└── StateMachine (Arbitrates active state)
    ├── IdleMoveState
    ├── AttackState
    └── DeflectState
```

- **Why use `set_deferred(&"disabled", false)`?** Godot's physics engine locks collision trees while calculating overlaps. Changing `.disabled` directly inside a physics callback throws a red engine error; `set_deferred` safely applies the toggle at the end of the physics frame.
    
- **Why use `_parry_timer` instead of animation tracks for parrying?** Checking `if _parry_timer > 0.0` in `_physics_process` guarantees millisecond-accurate _Sekiro_ deflection responsiveness at 60 FPS, completely independent of whether your character's guard animation has finished blending!


The industry standard for high-speed martial arts games (_Sekiro_, _Monster Hunter_, _Devil May Cry_) is **Multi-Point Raycast Trail Sweeping (Weapon Tracers)**, augmented by **Persistent Bone-Attached Area3D Volumes** for wide blunt impacts. By placing two or three marker points along the blade (Base, Mid, Tip) and casting continuous physics rays from their _previous frame positions_ to their _current frame positions_, you create an unbroken virtual cutting sheet that **can never miss, can never tunnel, and allocates zero memory at runtime.**

### 📖 Plain-English Glossary of Hit Registration

- **Tunnelling (Bullet Through Paper):** A physics glitch where a high-speed object jumps from Position A in Frame 1 to Position B in Frame 2 without touching an obstacle situated between them, because the collision was never sampled in the intervening empty space.
    
- **Raycast Sweeping (Tracers):** Sampling points on a blade each frame and drawing continuous mathematical line segments between where those points were 16.6 ms16.6\text{ ms}16.6 ms ago and where they are now.
    
- **Allocation Churn:** Continuously calling `instantiate()` and `queue_free()` every time a sword swings, forcing the engine's garbage collector and memory allocator to constantly create and destroy heap memory, causing micro-stutters.
    
- **Direct Physics State (`PhysicsDirectSpaceState3D`):** Godot’s low-level, direct access gateway to the physics engine server, executing raycasts instantly in code without needing physical scene-tree nodes.
    

### 🏛️ The Four Melee Hit Detection Paradigms

```
=============================================================================
                  THE MELEE HIT REGISTRATION TAXONOMY
=============================================================================

 ❌ 1. SPATIAL NODE SPAWNING (The Primitive Trail):
 Frame 1: [Spawn Box] ──► Frame 2: [Spawn Box] ──► Frame 3: [Spawn Box]
 (Massive allocation overhead; gaps between boxes at high speed!)

 ⚠️ 2. STATIC BONE-ATTACHED VOLUME (Persistent Area3D):
 Hand ──[BoneAttachment3D]──► [Live Capsule Area3D]
 (Good for slow clubs; misses narrow, high-speed katana slashes!)

 ✅ 3. MULTI-POINT SWEPT TRACERS (AAA Action Standard):
 Frame N-1 (Past):    [Tip]●-------------------------●[Base]
                       \ \ \  Swept Raycast Sheet   \ \ \
 Frame N (Current):     ▼ ▼ ▼                       ▼ ▼ ▼
                      [Tip]●-------------------------●[Base]
 (100% continuous coverage; zero allocation; frame-rate independent!)
=============================================================================
```

|Method|How It Works|Strengths|Fatal Flaws|Where AAA Studios Use It|
|---|---|---|---|---|
|**1. Discrete Volume Spawning**|Spawns multiple `Area3D` nodes along the arc during the swing.|Visually intuitive to conceptualise.|Terrible performance; high allocation churn; leaves gaps between boxes during fast swings.|Primitive prototypes only; rejected in production.|
|**2. Persistent Bone Volume**|One static `Area3D` capsule attached to the blade, toggled on/off.|Zero heap allocations; trivial to set up via `AnimationPlayer`.|Thin blades swing so fast they teleport past hurtboxes between frames (tunnelling).|Broad hitboxes: great hammers, shields, kicks, monster bites.|
|**3. Swept Raycast Tracers**|Traces continuous vectors between blade markers from frame to frame.|**Zero tunnelling**; mathematically perfect precision; millisecond hit-point accuracy for spark VFX.|Requires custom raycast looping in `_physics_process()`.|**Fast edged weapons**: _Sekiro_ katanas, _Monster Hunter_ longswords.|
|**4. Shape-Casting (`ShapeCast3D`)**|Sweeps a 3D sphere or capsule along the blade's velocity vector.|Volumetric thickness combined with continuous sweep interpolation.|Computationally heavier than rays; can clip static terrain if poorly configured.|Heavy sweeping weapons, boss tail whips.|

### 🔍 Why Spawning Trailing Hitboxes Fails Mechanically

If you spawn trailing hitboxes as a character swings a sword:

1. **Framerate Dependency:** At 60 FPS60\text{ FPS}60 FPS, an animation frame lasts 0.0166 seconds0.0166\text{ seconds}0.0166 seconds. A high-speed Wuxia sword swing travels 180∘180^\circ180∘ in roughly 0.10 seconds0.10\text{ seconds}0.10 seconds (only 6 frames!). Spawning a box on each frame leaves massive physical vacuums between each box where an enemy hurtbox can sit completely undetected.
    
2. **Memory Fratricide:** Spawning 6 `Area3D` nodes per swing ×\times× 4 swings per combo ×\times× 5 active enemies on screen generates hundreds of transient physics objects entering and leaving Godot's `SceneTree`, which severely throttles lower-spec hardware and integrated graphics.
    

### ⚔️ The AAA Standard: Multi-Point Raycast Sweepers (Weapon Tracers)

Rather than moving physical shapes, we place 3 lightweight `Marker3D` nodes along the sword mesh:

- `Marker_Base` (near the hilt / guard)
    
- `Marker_Mid` (the centre of the blade)
    
- `Marker_Tip` (the point of the sword)
    

During the **Active Window** of an attack:

1. On every physics frame, we query the `global_position` of each marker.
    
2. We cast a line from each marker's **current position** back to where it was in the **previous physics frame**.
    
3. We also cross-cast rays diagonally between `Base(t-1)` and `Tip(t)` to ensure no enemy slips through the interior webbing of the swing.
    
4. If a ray intersects an enemy's `Hurtbox3D`, we extract the exact `Vector3` collision point to spawn the metallic deflection spark and sound effect, register the hit, and append the enemy to our `hit_targets` array.
    

```
                    THE WEAPON TRACER CUTTING PLANE
=============================================================================
  Tip (Frame t-1)  ●======================================► ● Tip (Frame t)
                   \           Ray 1 (Tip-to-Tip)         /
                    \                                    /
                     \         Ray 2 (Mid-to-Mid)       /
  Mid (Frame t-1)     ●================================► ● Mid (Frame t)
                       \                                /
                        \      Ray 3 (Base-to-Base)    /
                         \                            /
  Base (Frame t-1)        ●==========================► ● Base (Frame t)
=============================================================================
```

### 🛠️ Production GDScript Implementation: `WeaponTracer3D.gd`

Attach this lightweight node directly as a child of your equipped weapon mesh. It uses Godot 4’s low-level `PhysicsDirectSpaceState3D` for speed.

GDScript

```
# res://core/combat/WeaponTracer3D.gd
class_name WeaponTracer3D
extends Node3D

signal strike_landed(hurtbox: Area3D, hit_point: Vector3, hit_normal: Vector3)

@export_group("Tracer Anchors")
@export var marker_base: Marker3D
@export var marker_mid: Marker3D
@export var marker_tip: Marker3D

@export_group("Collision Configuration")
@export_flags_3d_physics var hurtbox_collision_mask: int = 4 ## Collision Layer dedicated to Hurtboxes

var is_active: bool = false
var active_profile: AttackProfile

var _prev_base_pos: Vector3 = Vector3.ZERO
var _prev_mid_pos: Vector3 = Vector3.ZERO
var _prev_tip_pos: Vector3 = Vector3.ZERO

var _hit_targets: Array[Area3D] = []

func _ready() -> void:
	# Ensure processing is disabled until an attack explicitly activates it
	set_physics_process(false)

## Invoked via AnimationPlayer Method Track during the active swing frames
func start_tracing(profile: AttackProfile) -> void:
	active_profile = profile
	_hit_targets.clear()
	
	# Snapshot the initial baseline coordinates
	_prev_base_pos = marker_base.global_position
	_prev_mid_pos = marker_mid.global_position
	_prev_tip_pos = marker_tip.global_position
	
	is_active = true
	set_physics_process(true)

## Invoked via AnimationPlayer Method Track when the swing enters recovery
func stop_tracing() -> void:
	is_active = false
	set_physics_process(false)
	_hit_targets.clear()

func _physics_process(_delta: float) -> void:
	if not is_active:
		return
		
	var space_state: PhysicsDirectSpaceState3D = get_world_3d().direct_space_state
	
	var cur_base: Vector3 = marker_base.global_position
	var cur_mid: Vector3 = marker_mid.global_position
	var cur_tip: Vector3 = marker_tip.global_position
	
	# Execute parallel sweeping rays (Blade Edge & Core)
	_cast_tracer_ray(space_state, _prev_tip_pos, cur_tip)
	_cast_tracer_ray(space_state, _prev_mid_pos, cur_mid)
	_cast_tracer_ray(space_state, _prev_base_pos, cur_base)
	
	# Execute diagonal cross-webbing ray (Prevents slip-through during massive rotational velocity)
	_cast_tracer_ray(space_state, _prev_base_pos, cur_tip)
	
	# Store positions for the subsequent frame's vector calculations
	_prev_base_pos = cur_base
	_prev_mid_pos = cur_mid
	_prev_tip_pos = cur_tip

func _cast_tracer_ray(space_state: PhysicsDirectSpaceState3D, from: Vector3, to: Vector3) -> void:
	if from.is_equal_approx(to):
		return
		
	var query: PhysicsRayQueryParameters3D = PhysicsRayQueryParameters3D.create(from, to, hurtbox_collision_mask)
	query.collide_with_areas = true       ## Must be true to detect Area3D Hurtboxes!
	query.collide_with_bodies = false     ## Ignores physical ground/walls
	
	var result: Dictionary = space_state.intersect_ray(query)
	
	if not result.is_empty():
		var collider: Object = result[&"collider"]
		if collider is Hurtbox3D and not _hit_targets.has(collider):
			# Prevent self-harm if hitting own hurtbox
			if collider.owner == owner:
				return
				
			_hit_targets.append(collider)
			var hit_pos: Vector3 = result[&"position"]
			var hit_norm: Vector3 = result[&"normal"]
			
			# Deliver hit data directly to the target
			collider.receive_strike(active_profile, owner)
			strike_landed.emit(collider, hit_pos, hit_norm)
```

### 🎬 How It Integrates Seamlessly with `AnimationPlayer`

You never write timers in code to guess when the sword is swinging. In Godot’s **AnimationPlayer**, you add a **Call Method Track** targeting your `WeaponTracer3D`:

```
=============================================================================
                ANIMATION TIMELINE HIT SYNCHRONISATION
=============================================================================
 Track: [SwordSlash_01]
 0.00s ────────────────► 0.14s ═══════════════════════► 0.32s ──────────────► 0.50s
 [ Startup Frames ]      │                            │     [ Recovery Frames ]
                         ▼                            ▼
                 start_tracing()                stop_tracing()
```

1. At `0.14s` (the instant the character begins cutting forward), the animation track calls:`WeaponTracer3D.start_tracing(current_attack_profile)` → Tracers go live!
    
2. At `0.32s` (the blade reaches the end of its arc), the animation track calls:`WeaponTracer3D.stop_tracing()` → Tracers deactivate instantly!
    

If an enemy strikes you at `0.10s` and staggers your character, the animation cancels, `stop_tracing()` is automatically executed by the state exit logic, and no phantom hitboxes are left floating in the world.

### 🛡️ Defensive Engineering Rules for Hit Registration

- **Rule 1: Never Let Hitboxes Collide With Static World Terrain:** Ensure your tracer raycast mask **only** scans Layer 3 or 4 (Hurtboxes). If your tracers detect `StaticBody3D` walls, your sword will clank and break against the ground during natural downward swings unless you specifically write a weapon-deflection system.
    
- **Rule 2: Accurate Spark Vectors:** Notice how `result[&"position"]` gives you the exact millimeter coordinate of where the ray struck the enemy's hurtbox mesh. Pass that vector directly into your GPU particle system (`spark_vfx.global_position = hit_pos`) so the metallic sparks erupt precisely from the contact point of the two blades.