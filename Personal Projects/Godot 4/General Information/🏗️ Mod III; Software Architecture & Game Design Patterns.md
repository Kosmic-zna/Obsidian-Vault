# 🏗️ Module III: Software Architecture & Game Design Patterns

> [!info] 📖 Lexicon & First Principles: Architectural Engineering Terminology
> Before writing a single line of state or component code, we must define the formal systems constraints that prevent an enterprise codebase from collapsing under technical debt:
> - ⤷ **Coupling**: The degree of direct pointer awareness and functional co-dependence between two distinct modules. Tight coupling means refactoring class $A$ causes compilation or runtime crashes in class $B$. Loose coupling allows class $A$ to be extracted, modified, or deleted with zero side effects on class $B$.
> - ⤷ **Cohesion**: The measure of functional focus within an individual class boundary (the *Single Responsibility Principle*). High cohesion indicates that all member variables and methods cooperate strictly to solve one isolated computational responsibility.
> - ⤷ **Temporal Coupling**: An implicit, hidden dependency on the chronological order of execution. If invoking `calculate_velocity()` crashes unless `poll_input()` was invoked on the preceding line, the system suffers from temporal coupling.
> - ⤷ **Inversion of Control (IoC)**: An architectural rule where a computational framework or parent container dictates the lifecycle and invocation schedule of subordinate components, rather than components dictating when they run.
> - ⤷ **Dynamic Dispatch vs. Virtual Tables**: The low-level mechanics of resolving polymorphic methods at runtime. In GDScript, statically typed callables execute through deterministic method offsets rather than expensive hash table lookups.

---

## 3.1 The Golden Axiom: "Call Down, Signal Up"

The foundational architectural law of the Godot Engine is: **=="Call down, Signal up."==**

Violating this law is the primary cause of cyclic scene dependencies, brittle node path failures, and impossible unit testing. Godot structures active games as a directed, rooted tree of `Node` instances. Because parents construct, configure, and maintain the lifetime of their children, dependencies must flow strictly in one direction.

```
                  THE LAW OF DIRECTIONAL INTEGRITY
=====================================================================
                    [ Parent Node: Actor.gd ]
                          │          ▲
             Direct Call  │          │  Signal Emission
            (Downwards)   │          │  (Upwards)
       "$Gun.pull_trigger()" │          │  "ammo_depleted.emit()"
                          ▼          │
                    [ Child Node: Weapon.gd ]
```

### The Architectural Invariants

1. **Downwards Calling (Parent $\rightarrow$ Child)**:
   - ⤷ **The Mechanism**: Direct method invocations (`child.do_action()`), explicit property mutation (`child.max_speed = 20.0`), and parameter injection via constructor or setup routines.
   - ↳ **The Low-Level Logic**: The parent maintains a strong reference pointer to its child in memory. The parent knows the child's interface and is structurally responsible for orchestrating its lifecycle.
2. **Upwards Signaling (Child $\rightarrow$ Parent)**:
   - ⤷ **The Mechanism**: Native engine event notifications (**Signals**).
   - ↳ **The Low-Level Logic**: The child emits an interrupt-like broadcast into an array of function pointers (`Callable`). The child maintains **zero** pointers to its parent, does not know who is subscribed, and does not care what happens after the broadcast.

```
❌ THE FRAGILE ANTI-PATTERN: UPWARD CALLING
[Player (CharacterBody3D)]
   └── [Hurtbox (Area3D)] ──> Calls "get_parent().get_parent().take_damage(10)"
                              (Hardcoded tree path; crashes instantly if Hurtbox is 
                               re-parented or attached to an Enemy/Crate!)

✅ THE IDIOMATIC PATTERN: UPWARD SIGNALING
[Player (CharacterBody3D)] ──(Connects to signal)──┐
   └── [Hurtbox (Area3D)] ──(Emits event)──────────┴─► "damage_taken.emit(10)"
                              (Child knows nothing of Player; 100% reusable anywhere!)
```

---

## 3.2 Composition Over Inheritance: Component Architecture

Classical Object-Oriented Programming (OOP) attempts to model games through deep class inheritance trees:

$$\text{Object} \longrightarrow \text{Node} \longrightarrow \text{Entity} \longrightarrow \text{Actor} \longrightarrow \text{Monster} \longrightarrow \text{FlyingDragon}$$

This structure fails when features cross category boundaries. Consider a destructible wooden crate: it possesses health and takes damage like a monster, yet cannot move or execute AI. If `Crate` inherits from `Entity`, it duplicates `Actor`'s combat logic; if it inherits from `Actor`, it carries useless kinematic movement code.

### The Component Paradigm via Modular Nodes

Godot implements composition natively. **Nodes are components**. An "Entity" is not a massive monolithic class; it is an empty root container aggregating independent, single-responsibility child nodes:

```
                      MODULAR SCENE COMPOSITION
=====================================================================
 [ Humanoid Player ]                      [ Destructible Crate ]
  ├── CharacterBody3D (Root)               ├── StaticBody3D (Root)
  │    ├── HealthComponent                  │    ├── HealthComponent
  │    ├── HurtboxComponent                 │    ├── HurtboxComponent
  │    ├── HitboxComponent                  │    └── LootDropComponent
  │    └── VelocityComponent                └── CollisionShape3D
  └── CollisionShape3D
```

Notice that both the Player and the Crate share identical damage handling, armor mitigation, and death signals via `HealthComponent` and `HurtboxComponent`—with **zero shared class inheritance**.

---

### Step-by-Step Architectural Construction: The Core Combat Components

Let us construct this production combat framework piece by piece. We will analyze every single block of code as if performing a kernel review, explain its low-level memory mechanics, and then assemble the complete unified script.

---

### Component 1: `HealthComponent` (Pure Data & Mitigation Engine)

#### Chunk 1.1: Class Declaration & Interface Signals

```gdscript
class_name HealthComponent
extends Node

signal health_changed(new_health: float, max_health: float)
signal damage_mitigated(absorbed_amount: float)
signal entity_died
```

- **`class_name HealthComponent`**: Registers the script into Godot’s global class registry. This assigns a custom icon, exposes it in the "Create New Node" dialogue, and enables static compile-time type verification across the engine.
- **`extends Node`**: Inherits from raw `Node`. We deliberately avoid `Node2D` or `Node3D` because pure data components do not have spatial coordinates, transform matrices, or rendering overhead. This saves memory and keeps cache lines clean.
- **`signal health_changed(...)`**: Declares a signal that broadcasts two 64-bit IEEE 754 floats. Any listening system (e.g., UI health bars, floating combat text) receives the updated data without needing a direct reference to this node.
- **`signal damage_mitigated(...)`**: Fires whenever armor or shields absorb a portion of incoming damage, allowing sound and VFX systems to react.
- **`signal entity_died`**: A zero-parameter terminal signal emitted exactly once when health reaches zero.

---

#### Chunk 1.2: Encapsulated State & Constrained Setters

```gdscript
@export var max_health: float = 100.0:
	set(value):
		max_health = maxf(1.0, value)
		current_health = minf(current_health, max_health)

@export var armor_rating: float = 0.0

var current_health: float = 100.0:
	private_set
```

- **`@export var max_health: float = 100.0:`**: Exposes the float to the Godot Inspector for designers to balance. The colon initiates a **custom property setter**.
- **`set(value): max_health = maxf(1.0, value)`**: Defends internal class invariants. Maximum health cannot be zero or negative; `maxf(1.0, value)` guarantees the value is at least $1.0$, preventing division-by-zero errors in downstream percentage math ($\frac{\text{current}}{\text{max}}$).
- **`current_health = minf(current_health, max_health)`**: Automatically clamps current health if an external buff reduces the maximum pool.
- **`@export var armor_rating: float = 0.0`**: Defines the base armor mitigation value.
- **`var current_health: float = 100.0: private_set`**: Declares current health as publicly readable by any external observer, but restricts write operations strictly to methods internal to this class (`private_set`). External scripts cannot execute `health.current_health -= 50` directly; they are forced to call the public API.

---

#### Chunk 1.3: Initialization & The Mitigation Pipeline

```gdscript
func _ready() -> void:
	current_health = max_health

func apply_damage(raw_amount: float) -> void:
	if raw_amount <= 0.0 or is_zero_approx(current_health):
		return

	var mitigation_factor: float = 1.0 + (armor_rating / 100.0)
	var effective_damage: float = raw_amount / mitigation_factor
	var absorbed: float = raw_amount - effective_damage
	
	current_health = clampf(current_health - effective_damage, 0.0, max_health)
	
	if absorbed > 0.0:
		damage_mitigated.emit(absorbed)
		
	health_changed.emit(current_health, max_health)

	if is_zero_approx(current_health):
		entity_died.emit()
```

- **`func _ready() -> void: current_health = max_health`**: Synchronizes current health to the validated maximum when the node and its children enter the tree.
- **`if raw_amount <= 0.0 or is_zero_approx(current_health): return`**: Guard clause. Ignores negative damage (which should be routed through `heal()`) and discards hits on already-dead entities to prevent duplicate death events.
- **`var mitigation_factor: float = 1.0 + (armor_rating / 100.0)`**: Evaluates standard hyperbolic armor mitigation. If armor is $0$, the factor is $1.0$ ($100\%$ damage). If armor is $100$, the factor is $2.0$ ($50\%$ damage taken).
- **`var effective_damage: float = raw_amount / mitigation_factor`**: Resolves post-mitigation damage cleanly.
- **`current_health = clampf(...)`**: Mutates the internal state variable, safely bounded between $0.0$ and `max_health`.
- **`health_changed.emit(...)`**: Broadcasts the state change upwards to listening observers.
- **`if is_zero_approx(current_health): entity_died.emit()`**: Evaluates floating-point zero safely using `is_zero_approx()` rather than `== 0.0`, avoiding IEEE 754 precision bugs.

---

#### The Fully Assembled `HealthComponent.gd`

```gdscript
# ==============================================================================
# COMPONENT: HealthComponent.gd
# Architecture: Single-Responsibility Data Model & Damage Pipeline
# ==============================================================================
class_name HealthComponent
extends Node

## Dispatched whenever current_health is mutated via damage or healing
signal health_changed(new_health: float, max_health: float)
## Dispatched when armor reduces incoming raw damage
signal damage_mitigated(absorbed_amount: float)
## Dispatched strictly on the frame current_health reaches zero
signal entity_died

@export_group("Vital Statistics")
## Maximum permissible health; clamped to a minimum of 1.0
@export var max_health: float = 100.0:
	set(value):
		max_health = maxf(1.0, value)
		current_health = minf(current_health, max_health)

## Mitigates damage via the formula: Effective = Raw / (1.0 + Armor / 100.0)
@export var armor_rating: float = 0.0

## The active operational health pool. Read-only to external callers.
var current_health: float = 100.0:
	private_set

func _ready() -> void:
	current_health = max_health

## Public API: Submits damage through the armor mitigation pipeline
func apply_damage(raw_amount: float) -> void:
	# Guard against negative values and dead entities
	if raw_amount <= 0.0 or is_zero_approx(current_health):
		return

	# Hyperbolic armor mitigation formula
	var mitigation_factor: float = 1.0 + (armor_rating / 100.0)
	var effective_damage: float = raw_amount / mitigation_factor
	var absorbed: float = raw_amount - effective_damage
	
	current_health = clampf(current_health - effective_damage, 0.0, max_health)
	
	if absorbed > 0.0:
		damage_mitigated.emit(absorbed)
		
	health_changed.emit(current_health, max_health)

	if is_zero_approx(current_health):
		entity_died.emit()

## Public API: Restores health up to max_health
func heal(amount: float) -> void:
	if amount <= 0.0 or is_zero_approx(current_health):
		return
		
	current_health = clampf(current_health + amount, 0.0, max_health)
	health_changed.emit(current_health, max_health)
```

---

### Component 2: `HurtboxComponent` (The Spatial Damage Sensor)

Now let us build the receiver component. The Hurtbox is an `Area3D` that intercepts physical overlapping hits, handles invulnerability frames (i-frames), and forwards validated damage packets to an injected `HealthComponent`.

#### Chunk 2.1: Properties & Dependency Injection

```gdscript
class_name HurtboxComponent
extends Area3D

signal received_damage_packet(raw_amount: float, attacker_node: Node)

@export var health_component: HealthComponent
@export var invulnerability_duration: float = 0.25

var _is_invulnerable: bool = false
var _invulnerability_timer: Timer
```

- **`extends Area3D`**: Inherits spatial presence in the 3D physics server, allowing collision shapes to be registered for overlap detection without generating rigid-body collision forces.
- **`@export var health_component: HealthComponent`**: **Explicit Dependency Injection**. Rather than calling `get_node("../HealthComponent")` (which is brittle and fails if names change), the parent scene wires this pointer directly in the inspector.
- **`var _is_invulnerable: bool = false`**: An internal flag that prevents damage processing while invulnerability frames are active.

---

#### Chunk 2.2: Physics Configuration & Damage Ingestion

```gdscript
func _ready() -> void:
	monitoring = false
	monitorable = true
	_setup_timer()

func _setup_timer() -> void:
	_invulnerability_timer = Timer.new()
	_invulnerability_timer.one_shot = true
	_invulnerability_timer.timeout.connect(func() -> void: _is_invulnerable = false)
	add_child(_invulnerability_timer)

func accept_hit(raw_amount: float, source: Node) -> void:
	if _is_invulnerable or raw_amount <= 0.0:
		return

	if invulnerability_duration > 0.0:
		_is_invulnerable = true
		_invulnerability_timer.start(invulnerability_duration)

	received_damage_packet.emit(raw_amount, source)

	if health_component != null:
		health_component.apply_damage(raw_amount)
```

- **`monitoring = false; monitorable = true`**: **Performance optimization**. Hurtboxes are passive targets. They do not actively scan space for other objects; they simply present an area that active `Hitbox` volumes can detect. Setting `monitoring = false` removes this area from broadphase spatial tracking loops, reducing physics overhead.
- **`_setup_timer()`**: Programmatically creates and configures a `Timer` child, connecting its `timeout` signal to a lambda that resets the invulnerability state.
- **`accept_hit(...)`**: The public entry point invoked by an attacking `HitboxComponent`. If i-frames are inactive, it locks out further hits, emits an upward telemetry signal, and calls `apply_damage` on the injected `HealthComponent`.

---

#### The Fully Assembled `HurtboxComponent.gd`

```gdscript
# ==============================================================================
# COMPONENT: HurtboxComponent.gd
# Architecture: Spatial Damage Sensor & Invulnerability Gatekeeper
# ==============================================================================
class_name HurtboxComponent
extends Area3D

## Dispatched when a valid hit penetrates invulnerability frames
signal received_damage_packet(raw_amount: float, attacker_node: Node)

@export_group("Dependencies")
## Injected pointer to the companion HealthComponent
@export var health_component: HealthComponent

@export_group("Configuration")
## Temporal window of invulnerability granted after accepting damage (seconds)
@export var invulnerability_duration: float = 0.25

var _is_invulnerable: bool = false
var _invulnerability_timer: Timer

func _ready() -> void:
	# Passive detection: Do not monitor, but allow active hitboxes to monitor us
	monitoring = false
	monitorable = true
	_setup_timer()

func _setup_timer() -> void:
	_invulnerability_timer = Timer.new()
	_invulnerability_timer.one_shot = true
	_invulnerability_timer.timeout.connect(func() -> void: _is_invulnerable = false)
	add_child(_invulnerability_timer)

## Public API: Called by active HitboxComponents upon spatial intersection
func accept_hit(raw_amount: float, source: Node) -> void:
	if _is_invulnerable or raw_amount <= 0.0:
		return

	# Trigger invulnerability window
	if invulnerability_duration > 0.0:
		_is_invulnerable = true
		_invulnerability_timer.start(invulnerability_duration)

	received_damage_packet.emit(raw_amount, source)

	# Forward damage down to data model
	if health_component != null:
		health_component.apply_damage(raw_amount)
```

---

### Component 3: `HitboxComponent` (The Spatial Damage Striker)

#### The Fully Assembled `HitboxComponent.gd`

```gdscript
# ==============================================================================
# COMPONENT: HitboxComponent.gd
# Architecture: Active Spatial Striker (Detects Passive Hurtboxes)
# ==============================================================================
class_name HitboxComponent
extends Area3D

## Dispatched when this hitbox successfully delivers damage to a Hurtbox
signal hit_confirmed(target_hurtbox: HurtboxComponent)

@export_group("Combat Attributes")
## Base damage value passed into the target's damage pipeline
@export var base_damage: float = 25.0

func _ready() -> void:
	# Active detection: actively scan for passive hurtbox volumes
	monitoring = true
	monitorable = false
	area_entered.connect(_on_area_entered)

func _on_area_entered(area: Area3D) -> void:
	# Validate that the intersected area is a HurtboxComponent
	if area is HurtboxComponent:
		var hurtbox: HurtboxComponent = area as HurtboxComponent
		# Deliver damage payload, passing the root scene owner as the attacker
		hurtbox.accept_hit(base_damage, owner)
		hit_confirmed.emit(hurtbox)
```

- **`monitoring = true; monitorable = false`**: The inverse of the hurtbox. The hitbox actively checks for overlaps during the physics step, while remaining invisible to other hitboxes.
- **`area is HurtboxComponent`**: Evaluates the polymorphic type. If a weapon swings through environmental props, triggers, or coins, the engine discards the collision instantly without running damage logic.
- **`hurtbox.accept_hit(base_damage, owner)`**: Hands the damage payload to the hurtbox, passing `owner` (the root entity) so kill credits and status effects know who dealt the blow.

---

## 3.3 The State Pattern: Finite State Machines (FSM) & Pushdown Automata

As character mechanics expand, trying to manage behavior using boolean flags (`is_running`, `is_jumping`, `can_attack`) creates an unmanageable web of states:

```gdscript
# ❌ THE "BOOLEAN LABYRINTH" ANTI-PATTERN
func _physics_process(delta: float) -> void:
	if is_on_floor():
		if Input.is_action_just_pressed("jump") and not is_crouching and not is_attacking:
			is_jumping = true
		elif is_dashing and not is_stunned and not is_swinging:
			execute_dash()
```

With $N$ boolean variables, a character can exist in $2^N$ possible states—most of which represent broken edge cases (such as attacking and being stunned on the same frame).

A **Finite State Machine (FSM)** solves this by enforcing that an actor exists in **exactly one state at any given moment**.

```
                        FINITE STATE TRANSITION GRAPH
=====================================================================
                             [ IDLE ]
                            /   ▲    \
               move_input  /    │     \  jump_input
                          v     │      v
                      [ RUN ]   │    [ AIRBORNE ]
                          \     │      ▲
               jump_input  \    │     /  fall_off_ledge
                            v   │    /
                          [ AIRBORNE ]
```

---

### The Pushdown Automaton (State Stack Engine)

Standard FSMs discard their previous state during a transition. If an actor is interrupted by a temporary action—like a stun, weapon reload, or evasive roll—a simple FSM doesn't know which state to return to.

A **Pushdown Automaton** introduces a **Stack (LIFO)** data structure:
- **`push_state(new_state)`**: Pauses the current state and pushes it onto the stack beneath the new active state.
- **`pop_state()`**: Terminates the active state and resumes the interrupted state from the stack.

```
                   PUSHDOWN AUTOMATON (STATE STACK)
=====================================================================
1. Entity is running:            Stack: [ RUN ]
2. Player initiates an Attack:   Stack: [ RUN ][ ATTACK ] (Attack executes over Run)
3. Attack animation finishes:    Stack: [ RUN ] (Attack pops off; Run resumes!)
```

---

### Step-by-Step Architectural Construction: The State Stack Engine

Let us build the Pushdown Automaton line by line.

---

#### Class 1: `State.gd` (The Polymorphic Contract)

```gdscript
class_name State
extends Node

var actor: CharacterBody3D
var state_machine: StateMachine

func enter(msg: Dictionary = {}) -> void: pass
func exit() -> void: pass
func handle_input(event: InputEvent) -> void: pass
func update(delta: float) -> void: pass
func physics_update(delta: float) -> void: pass
```

- **`var actor: CharacterBody3D`**: A reference to the entity being controlled. This gives states access to velocity, mesh rotations, and raycasts.
- **`var state_machine: StateMachine`**: A reference back to the parent state machine so the state can request transitions (`state_machine.change_state("Run")`).
- **`func enter(msg: Dictionary = {})`**: Lifecycle hook executed the moment the state becomes active. Accepts an optional dictionary payload to pass context between states (e.g., initial knockback velocity).
- **`func exit()`**: Lifecycle hook executed the moment the state loses control, cleaning up tweens, animations, or temporary flags.
- **`physics_update(delta)`**: The state's local replacement for `_physics_process()`. The state machine calls this only on the currently active state.

---

#### Class 2: `StateMachine.gd` (The Stack Controller)

#### Chunk 2.1: Initialization & Reflection Registry

```gdscript
class_name StateMachine
extends Node

signal transitioned(old_state_name: StringName, new_state_name: StringName)

@export var initial_state: State

var _state_stack: Array[State] = []
var _states: Dictionary = {}

var current_state: State:
	get:
		return _state_stack.back() if not _state_stack.is_empty() else null

func _ready() -> void:
	var actor_node: CharacterBody3D = owner as CharacterBody3D
	assert(actor_node != null, "StateMachine: Owner root must be a CharacterBody3D.")

	for child in get_children():
		if child is State:
			var s: State = child as State
			_states[s.name] = s
			s.actor = actor_node
			s.state_machine = self

	if initial_state != null:
		change_state(initial_state.name)
```

- **`var _state_stack: Array[State] = []`**: The internal stack storing state history. The item at index `-1` (`_state_stack.back()`) is the currently executing state.
- **`var current_state: State: get:`**: A custom getter that returns the top of the stack, or `null` if the stack is uninitialized.
- **`owner as CharacterBody3D`**: In Godot, `owner` refers to the root node of the saved scene file. This links child state components to their controlling actor without needing hardcoded paths.
- **`for child in get_children(): if child is State:`**: Dynamic discovery. The state machine loops through its children, registers any `State` nodes into a lookup dictionary (`_states`), and injects references to the actor and itself.

---

#### Chunk 2.2: Execution Dispatchers

```gdscript
func _unhandled_input(event: InputEvent) -> void:
	if current_state != null:
		current_state.handle_input(event)

func _process(delta: float) -> void:
	if current_state != null:
		current_state.update(delta)

func _physics_process(delta: float) -> void:
	if current_state != null:
		current_state.physics_update(delta)
```

- These methods intercept Godot's engine lifecycle hooks and route them directly to the active state.
- Inactive states are simply skipped in memory—their code does not execute, consuming zero CPU cycles.

---

#### Chunk 2.3: Transition Mechanics (Swap, Push, Pop)

```gdscript
func change_state(target_state_name: StringName, msg: Dictionary = {}) -> void:
	assert(_states.has(target_state_name), "State does not exist: %s" % target_state_name)
	var target: State = _states[target_state_name]
	
	var old_name: StringName = &""
	if current_state != null:
		old_name = current_state.name
		current_state.exit()
	
	_state_stack.clear()
	_state_stack.push_back(target)
	target.enter(msg)
	transitioned.emit(old_name, target_state_name)

func push_state(target_state_name: StringName, msg: Dictionary = {}) -> void:
	assert(_states.has(target_state_name), "State does not exist: %s" % target_state_name)
	var target: State = _states[target_state_name]
	
	var old_name: StringName = &""
	if current_state != null:
		old_name = current_state.name
		current_state.exit()
		
	_state_stack.push_back(target)
	target.enter(msg)
	transitioned.emit(old_name, target_state_name)

func pop_state() -> void:
	if _state_stack.size() <= 1:
		push_warning("StateMachine: Cannot pop root baseline state.")
		return
		
	var abandoning: State = _state_stack.pop_back()
	abandoning.exit()
	
	var resumed: State = current_state
	resumed.enter({"resumed": true})
	transitioned.emit(abandoning.name, resumed.name)
```

- **`change_state(...)`**: Standard state transition. Clears the stack, calls `exit()` on the old state, sets the new state, and calls `enter()`.
- **`push_state(...)`**: Pushes an interruptive state onto the stack. It pauses the current state by calling `exit()`, but preserves it on the stack before pushing the new state on top.
- **`pop_state()`**: Restores the interrupted state. It pops the active state off the stack, calls `exit()`, and resumes the previous state by calling `enter({"resumed": true})`.

---

#### The Fully Assembled `StateMachine.gd`

```gdscript
# ==============================================================================
# CONTROLLER: StateMachine.gd
# Architecture: Pushdown Automaton Stack Controller
# ==============================================================================
class_name StateMachine
extends Node

## Dispatched whenever the active executing state changes
signal transitioned(old_state_name: StringName, new_state_name: StringName)

## The baseline state initialized when the tree runs
@export var initial_state: State

var _state_stack: Array[State] = []
var _states: Dictionary = {}

## Public property resolving the actively ticked state at the top of the stack
var current_state: State:
	get:
		return _state_stack.back() if not _state_stack.is_empty() else null

func _ready() -> void:
	var actor_node: CharacterBody3D = owner as CharacterBody3D
	assert(actor_node != null, "StateMachine: Owner root must be a CharacterBody3D.")

	# Register child states dynamically
	for child in get_children():
		if child is State:
			var s: State = child as State
			_states[s.name] = s
			s.actor = actor_node
			s.state_machine = self

	# Boot baseline state
	if initial_state != null:
		change_state(initial_state.name)

func _unhandled_input(event: InputEvent) -> void:
	if current_state != null:
		current_state.handle_input(event)

func _process(delta: float) -> void:
	if current_state != null:
		current_state.update(delta)

func _physics_process(delta: float) -> void:
	if current_state != null:
		current_state.physics_update(delta)

## Standard transition: Clears history and transitions absolutely
func change_state(target_state_name: StringName, msg: Dictionary = {}) -> void:
	assert(_states.has(target_state_name), "StateMachine: State %s not found." % target_state_name)
	var target: State = _states[target_state_name]
	
	var old_name: StringName = &""
	if current_state != null:
		old_name = current_state.name
		current_state.exit()
	
	_state_stack.clear()
	_state_stack.push_back(target)
	target.enter(msg)
	transitioned.emit(old_name, target_state_name)

## Pushes an interruptive state, pausing the existing state on the stack
func push_state(target_state_name: StringName, msg: Dictionary = {}) -> void:
	assert(_states.has(target_state_name), "StateMachine: State %s not found." % target_state_name)
	var target: State = _states[target_state_name]
	
	var old_name: StringName = &""
	if current_state != null:
		old_name = current_state.name
		current_state.exit()
		
	_state_stack.push_back(target)
	target.enter(msg)
	transitioned.emit(old_name, target_state_name)

## Pops the active state and resumes the previous state from the stack
func pop_state() -> void:
	if _state_stack.size() <= 1:
		push_warning("StateMachine: Cannot pop root baseline state.")
		return
		
	var abandoning: State = _state_stack.pop_back()
	abandoning.exit()
	
	var resumed: State = current_state
	resumed.enter({"resumed": true})
	transitioned.emit(abandoning.name, resumed.name)
```

---

#### Concrete State Implementations

Here is what concrete state scripts look like in practice:

```gdscript
# ==============================================================================
# STATE: IdleState.gd
# ==============================================================================
class_name IdleState
extends State

func enter(msg: Dictionary = {}) -> void:
	actor.velocity.x = 0.0
	actor.velocity.z = 0.0

func physics_update(_delta: float) -> void:
	if not actor.is_on_floor():
		state_machine.change_state(&"Airborne")
		return

	var input_vector: Vector2 = Input.get_vector(&"move_left", &"move_right", &"move_forward", &"move_backward")
	if not input_vector.is_zero_approx():
		state_machine.change_state(&"Run")
		return

	if Input.is_action_just_pressed(&"attack"):
		# Push the attack interrupt onto the stack
		state_machine.push_state(&"Attack")
		return
```

```gdscript
# ==============================================================================
# STATE: AttackState.gd (Transient Action Interrupt)
# ==============================================================================
class_name AttackState
extends State

@export var attack_duration: float = 0.35
var _elapsed: float = 0.0

func enter(msg: Dictionary = {}) -> void:
	_elapsed = 0.0
	# Retain a fraction of momentum during the attack swing
	actor.velocity.x *= 0.25
	actor.velocity.z *= 0.25

func physics_update(delta: float) -> void:
	_elapsed += delta
	if _elapsed >= attack_duration:
		# Attack complete: pop the stack to resume whatever state was interrupted
		state_machine.pop_state()
```

---

## 3.4 The Command Pattern: Input Independence & Undo/Redo

The **Command Pattern** encapsulates an action inside a standalone object (`Command`). Instead of input handlers directly modifying the world:

```gdscript
# ❌ DIRECT MUTATION ANTI-PATTERN
func _unhandled_input(event: InputEvent) -> void:
	if event.is_action_pressed("move_north"):
		player.position.y += 1 # Cannot be undone, replayed, or networked!
```

We wrap the operation into an object that knows both how to **execute** and how to **invert (undo)** itself:

```
                           THE COMMAND PIPELINE
=====================================================================
  [ Hardware Input ] ──► Creates: [ MoveCommand(Vector2i.UP) ]
                                          │
                                          ▼
                             [ CommandInvoker Node ]
                             - command.execute(actor)
                             - _undo_stack.push_back(command)
                                    /         \
                   Undo Triggered  /           \  Redo Triggered
                                  v             v
                              cmd.undo()    cmd.execute()
```

---

### Step-by-Step Architectural Construction: Reversible Command Subsystem

---

#### Chunk 1: The Command Interface

```gdscript
class_name Command
extends RefCounted

func execute(actor: Node) -> void:
	assert(false, "Command: execute() must be overridden.")

func undo(actor: Node) -> void:
	assert(false, "Command: undo() must be overridden.")
```

- **`extends RefCounted`**: Commands are lightweight data payloads, not nodes. Inheriting from `RefCounted` means their memory is freed automatically when dropped from the undo stack.
- **`assert(false, ...)`**: Enforces an abstract interface. If a subclass fails to implement `execute()` or `undo()`, the engine stops immediately in debug builds.

---

#### Chunk 2: Concrete Command (`MoveCommand`)

```gdscript
class_name MoveCommand
extends Command

var _direction: Vector2i
var _previous_position: Vector2i

func _init(direction: Vector2i) -> void:
	_direction = direction

func execute(actor: Node) -> void:
	var target: GridPawn = actor as GridPawn
	assert(target != null, "Target must be a GridPawn.")
	
	# Cache state before mutation for precise rollback
	_previous_position = target.grid_position
	target.grid_position += _direction
	target.global_position = target.grid_to_world(target.grid_position)

func undo(actor: Node) -> void:
	var target: GridPawn = actor as GridPawn
	assert(target != null, "Target must be a GridPawn.")
	
	# Restore prior state
	target.grid_position = _previous_position
	target.global_position = target.grid_to_world(_previous_position)
```

- **`_previous_position`**: Caches the actor's position *before* the command executes, making rollback trivial and exact.
- **`undo()`**: Reverses the transformation by restoring the cached coordinate, returning the entity to its previous state.

---

#### Chunk 3: The `CommandInvoker` (History Manager)

```gdscript
# ==============================================================================
# CONTROLLER: CommandInvoker.gd
# Architecture: Dual-Stack Undo/Redo Engine
# ==============================================================================
class_name CommandInvoker
extends Node

@export var controlled_actor: Node
@export var max_history: int = 50

var _undo_stack: Array[Command] = []
var _redo_stack: Array[Command] = []

func execute_command(cmd: Command) -> void:
	cmd.execute(controlled_actor)
	_undo_stack.push_back(cmd)
	_redo_stack.clear() # Executing a new action invalidates the redo branch
	
	if _undo_stack.size() > max_history:
		_undo_stack.pop_front() # Discard oldest command to bound memory

func undo() -> void:
	if _undo_stack.is_empty():
		return
		
	var cmd: Command = _undo_stack.pop_back()
	cmd.undo(controlled_actor)
	_redo_stack.push_back(cmd)

func redo() -> void:
	if _redo_stack.is_empty():
		return
		
	var cmd: Command = _redo_stack.pop_back()
	cmd.execute(controlled_actor)
	_undo_stack.push_back(cmd)
```

- **`_redo_stack.clear()`**: An important invariant in undo systems: executing a new command after an undo invalidates any future redo history.
- **`max_history`**: Limits the size of the undo stack, preventing unbounded memory growth in long gameplay sessions.

---

## 3.5 The Strategy Pattern: Algorithm Swapping via Custom Resources

The **Strategy Pattern** encapsulates a family of algorithms into interchangeable objects. In Godot, the idiomatic way to implement this is using **Custom Resources (`.tres`)**.

Because resources are data-driven assets that can contain executable scripts, you can swap out algorithms—such as weapon projectile physics—in the inspector without changing a single line of caller code.

```
                      THE RESOURCE STRATEGY PATTERN
=====================================================================
                             [ Weapon Node ]
                     @export strategy: FiringStrategy
                                    │
            ┌───────────────────────┼───────────────────────┐
            ▼                       ▼                       ▼
    [ HitscanStrategy ]    [ BallisticStrategy ]    [ HomingStrategy ]
    Direct Raycast         Physics Projectile       Steering Kinematics
```

---

### Step-by-Step Architectural Construction: Swappable Weapon Strategies

---

#### Step 1: The Strategy Interface (`FiringStrategy.gd`)

```gdscript
# ==============================================================================
# STRATEGY INTERFACE: FiringStrategy.gd
# ==============================================================================
class_name FiringStrategy
extends Resource

## Virtual interface contract: Executes an offensive projectile/hitscan algorithm
func execute_fire(origin: Node3D, target_vector: Vector3, collision_mask: int) -> void:
	assert(false, "FiringStrategy: execute_fire() must be overridden.")
```

- Extends `Resource`. This allows concrete strategies to be saved to disk as `.tres` files, inspected, and dragged into export slots.

---

#### Step 2: Concrete Strategy 1 (`HitscanStrategy.gd`)

```gdscript
# ==============================================================================
# STRATEGY: HitscanStrategy.gd (Instant Raycast Query)
# ==============================================================================
class_name HitscanStrategy
extends FiringStrategy

@export var range_distance: float = 1000.0
@export var base_damage: float = 40.0

func execute_fire(origin: Node3D, target_vector: Vector3, collision_mask: int) -> void:
	var space_state: PhysicsDirectSpaceState3D = origin.get_world_3d().direct_space_state
	var start_point: Vector3 = origin.global_position
	var end_point: Vector3 = start_point + (target_vector.normalized() * range_distance)
	
	var query: PhysicsRayQueryParameters3D = PhysicsRayQueryParameters3D.create(
		start_point, 
		end_point, 
        collision_mask
	)
	
	var result: Dictionary = space_state.intersect_ray(query)
	if not result.is_empty():
		var collider: Object = result["collider"]
		if collider is HurtboxComponent:
			(collider as HurtboxComponent).accept_hit(base_damage, origin)
```

- **`PhysicsDirectSpaceState3D`**: Bypasses node-based `RayCast3D` nodes entirely, querying the physics server directly for instant hitscan evaluation.
- Queries can be called hundreds of times in a single frame (e.g., shotgun pellets) with minimal performance overhead.

---

#### Step 3: Concrete Strategy 2 (`BallisticStrategy.gd`)

```gdscript
# ==============================================================================
# STRATEGY: BallisticStrategy.gd (Physical Projectile Spawner)
# ==============================================================================
class_name BallisticStrategy
extends FiringStrategy

@export var projectile_scene: PackedScene
@export var muzzle_velocity: float = 35.0

func execute_fire(origin: Node3D, target_vector: Vector3, collision_mask: int) -> void:
	assert(projectile_scene != null, "BallisticStrategy: projectile_scene is unassigned.")
	
	var projectile: Node3D = projectile_scene.instantiate() as Node3D
	origin.get_tree().root.add_child(projectile)
	projectile.global_position = origin.global_position
	
	if projectile is RigidBody3D:
		var rb: RigidBody3D = projectile as RigidBody3D
		rb.collision_mask = collision_mask
		rb.linear_velocity = target_vector.normalized() * muzzle_velocity
```

- Instantiates a physical projectile scene into the world and applies an impulse velocity, completely swapping gameplay behavior from instant raycasts to simulated ballistics.

---

#### Step 4: The Context Container (`ModularWeapon.gd`)

```gdscript
# ==============================================================================
# CONTEXT: ModularWeapon.gd
# Architecture: Context Container with Injected Strategy
# ==============================================================================
class_name ModularWeapon
extends Node3D

## Swap this resource in the Inspector to completely change weapon behavior!
@export var firing_strategy: FiringStrategy
@export_flags_3d_physics var hit_mask: int = 1

func pull_trigger(aim_direction: Vector3) -> void:
	if firing_strategy == null:
		push_error("ModularWeapon: No FiringStrategy resource assigned.")
		return
		
	# Polymorphic call: The weapon does not know or care which algorithm executes!
	firing_strategy.execute_fire(self, aim_direction, hit_mask)
```

- The weapon script contains zero weapon-type-specific logic—no `if is_rocket_launcher:` checks. Behavior is driven entirely by the injected resource.

---

## 3.6 Applied Synthesis: Complete Decoupled Entity Architecture

To see how all these pieces fit together, examine this production entity controller. It combines our **Component Architecture**, **Pushdown State Machine**, **Local Signals**, and **Strategy-driven Weapon**:

```
                       UNIFIED SCENE TOPOLOGY
=====================================================================
 [ VanguardActor (CharacterBody3D) ]
  ├── HealthComponent (Node)
  ├── HurtboxComponent (Area3D)
  │    └── CollisionShape3D
  ├── ModularWeapon (Node3D)
  │    └── [ Injected Resource: BallisticStrategy.tres ]
  └── StateMachine (Node)
       ├── IdleState (Node)
       ├── RunState (Node)
       └── StunState (Node)
```

```gdscript
# ==============================================================================
# PRODUCTION CONTROLLER: VanguardActor.gd
# Architecture: Complete System Synthesis
# ==============================================================================
class_name VanguardActor
extends CharacterBody3D

@onready var health_component: HealthComponent = $HealthComponent
@onready var hurtbox_component: HurtboxComponent = $HurtboxComponent
@onready var weapon: ModularWeapon = $ModularWeapon
@onready var state_machine: StateMachine = $StateMachine

func _ready() -> void:
	# 1. Wire internal component dependencies (Inversion of Control)
	hurtbox_component.health_component = health_component
	
	# 2. Subscribe to reactive component events
	health_component.health_changed.connect(_on_health_changed)
	health_component.entity_died.connect(_on_entity_died)
	
	# 3. Intercept damage signals to trigger dynamic state interrupts
	hurtbox_component.received_damage_packet.connect(_on_damaged)

func _on_damaged(raw_amount: float, attacker: Node) -> void:
	# Heavy hits push a temporary stun state onto the stack
	if raw_amount >= 30.0:
		state_machine.push_state(&"Stun", {"duration": 0.5})

func _on_health_changed(current: float, maximum: float) -> void:
	# Update local visual feedback (e.g., hit flash, health bars)
	pass

func _on_entity_died() -> void:
	# Safely disable collision shapes using deferred calls
	hurtbox_component.set_deferred(&"monitoring", false)
	hurtbox_component.set_deferred(&"monitorable", false)
	
	# Deallocate the entity cleanly from the scene tree
	queue_free()
```

---

## ✍️ Linguistic & Grammatical Refinement

> [!info] 📖 Constructive Analysis of Your Input
> Let us review the phrasing of your directive:
> 1. **Capitalisation & Sentence Boundaries**:
>    - ⤷ *Original*: `"i'm not the biggest fan of this, can you make it so it goes line by line..."`
>    - ↳ **Correction**: Capitalize the personal pronoun **"I"** and separate distinct thoughts with full stops or semicolons rather than joining multiple clauses with commas (a *comma splice*):
>      *"I am not particularly fond of this layout. Could you restructure it to proceed line by line..."*
> 2. **Idiomatic Precision in Technical Discourse**:
>    - ⤷ *Original*: `"like you're kernel coding"`
>    - ↳ **Correction**: The idiomatic technical phrasing is **"systems-level analysis"** or **"kernel-style code review"**—evaluating execution order, pointer safety, and register mechanics directly.