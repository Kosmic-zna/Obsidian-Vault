# 🏗️ Module III: Software Architecture & Game Design Patterns

> [!info] 📖 Lexicon & First Principles: Architectural Terminology
> Before engineering production gameplay loops, we must define the structural disciplines that prevent large codebases from degenerating into unmaintainable webs:
> - ⤷ **Coupling**: The degree of direct interdependence between software modules. High coupling means changing module $A$ breaks module $B$; loose coupling allows modules to be refactored or deleted in complete isolation.
> - ⤷ **Cohesion**: The degree to which elements inside a single module belong together logically. A class with high cohesion executes one well-defined responsibility (Single Responsibility Principle).
> - ⤷ **Encapsulation**: Enclosing internal state variables and requiring external callers to interact through a strict public interface, preventing external code from corrupting internal invariants.
> - ⤷ **Polymorphism**: The capacity of different underlying data types to respond to the same uniform method signature through shared inheritance or interface contracts.
> - ⤷ **Temporal Coupling**: A subtle bug where method calls or state modifications are implicitly dependent on an undocumented, fragile execution order (e.g., method $B$ crashing unless method $A$ was executed on the previous line).

---

## 3.1 The Golden Axiom: "Call Down, Signal Up"

The single most consequential failure mode among intermediate Godot developers is violating the natural hierarchy of the **Scene Tree**. Godot organizes scenes into directed acyclic trees. If child nodes make assumptions about their ancestors, the scene cannot be instantiated independently, isolated for unit testing, or rearranged in the editor without triggering runtime crashes.

```
THE LAW OF DIRECTIONAL INTEGRITY
=================================
             [ Parent Node ]
                    |
      Direct Call   |   ▲   Signal Emission
      (Downwards)   |   |   (Upwards)
      "$Gun.fire()" |   |   "reloaded.emit()"
                    v   |
              [ Child Node ]
```

### The Directional Invariants

1. **Downwards Communication (Parent $\rightarrow$ Child)**:
   - ⤷ **Mechanism**: Direct method invocation, property assignment, and explicit dependency injection.
   - ↳ **The Justification**: A parent node *owns* its direct children. It knows their spatial composition, configures their parameters, and directs their routines directly (e.g., `health_component.apply_damage(25.0)`).
2. **Upwards Communication (Child $\rightarrow$ Parent)**:
   - ⤷ **Mechanism**: First-class **Signals** (`signal`).
   - ↳ **The Justification**: A child node must remain completely ignorant of its environment. It must never reach into its ancestor hierarchy via `get_parent()`, `get_node("../..")`, or global singletons to deliver status reports. The child reports what happened to itself; it leaves the decision of *how to respond* entirely to whatever parent is listening.

```
ANTI-PATTERN: UPWARD CALLING (Rigid, Fragile, Unreusable)
[Player]
   └── [HurtboxArea] ──> Calls "get_parent().take_damage(10)" 
                         ❌ Crashes instantly if Hurtbox is placed on a Crate!

IDIOMATIC: UPWARD SIGNALING (Decoupled, Universal)
[Player] (Connects signal to its own HealthComponent)
   └── [HurtboxArea] ──> Emits "damage_received(10)"
                         ✅ Works seamlessly on Players, Enemies, or Crates!
```

> [!warning] ⚠️ Production Trap: `get_node()` Path Fragility
> Using hardcoded node paths such as `get_node("../UI/MarginContainer/Label")` creates an instant point of failure. The moment an artist wraps that `Label` in an extra `HBoxContainer`, your code throws a `Nil Object Reference` runtime crash. Always inject references downwards using `@export` or `@onready var target = $Path`, and report upwards exclusively via signals.

---

## 3.2 Composition Over Inheritance: Component Architecture

Object-Oriented Programming (OOP) teaches developers to model systems via deep inheritance hierarchies (`Entity` $\rightarrow$ `Actor` $\rightarrow$ `Pawn` $\rightarrow$ `Monster` $\rightarrow$ `FlyingDemon`). In game development, this classical taxonomy inevitably triggers the **Diamond Problem** and structural paralysis:

```
THE INHERITANCE TRAP: CLASS EXPLOSION
======================================
                 [ Entity ]
                /          \
         [ Actor ]        [ Prop ]
          /     \             |
     [ Enemy ]  [ Player ] [ Chest ]
        |
  [ FlyingDemon ]
        |
  How do we build a "Flying Chest" or a "Destructible Barricade"?
  - If Barricade inherits from Actor, it inherits movement and inventory bloat.
  - If Barricade inherits from Prop, it duplicates Actor's health and damage code!
```

### The Architectural Solution: Entity-Component Model via Godot Nodes

Godot implements composition natively without needing external Entity-Component-System (ECS) frameworks. **Nodes are components**. An "Entity" is merely an empty root node (`Node2D` or `Node3D`) that acts as an aggregation point for modular, single-responsibility child nodes:

```
COMPOSITION VIA NODES (Plug-and-Play Assembly)
==============================================
[ Goblin (CharacterBody3D) ]              [ WoodenCrate (StaticBody3D) ]
├── HealthComponent (Node)                 ├── HealthComponent (Node)
├── HurtboxComponent (Area3D)             ├── HurtboxComponent (Area3D)
├── HitboxComponent (Area3D)              └── DropsLootComponent (Node)
├── VelocityComponent (Node)
└── AIController (Node)
```

Both entities share the exact same damage mechanics, invulnerability logic, and health clamping without sharing a single line of class inheritance.

### Production Implementation: Decoupled Combat Components

#### 1. The Model: `HealthComponent`

```gdscript
# ==============================================================================
# COMPONENT: Pure Health Data & Clamping Logic
# ==============================================================================
class_name HealthComponent
extends Node

signal health_changed(new_health: float, max_health: float)
signal damage_mitigated(absorbed_amount: float)
signal entity_died

@export var max_health: float = 100.0:
    set(value):
        max_health = maxf(1.0, value)
        current_health = minf(current_health, max_health)

@export var armor_rating: float = 0.0

var current_health: float = 100.0:
    private_set

func _ready() -> void:
    current_health = max_health

## Applies incoming damage through the deterministic mitigation pipeline
func apply_damage(raw_amount: float) -> void:
    if raw_amount <= 0.0 or is_zero_approx(current_health):
        return

    # Armor formula: Effective Damage = Raw / (1 + Armor / 100)
    var mitigation_factor: float = 1.0 + (armor_rating / 100.0)
    var effective_damage: float = raw_amount / mitigation_factor
    var absorbed: float = raw_amount - effective_damage
    
    current_health = clampf(current_health - effective_damage, 0.0, max_health)
    
    if absorbed > 0.0:
        damage_mitigated.emit(absorbed)
        
    health_changed.emit(current_health, max_health)

    if is_zero_approx(current_health):
        entity_died.emit()

func heal(amount: float) -> void:
    if amount <= 0.0 or is_zero_approx(current_health):
        return
        
    current_health = clampf(current_health + amount, 0.0, max_health)
    health_changed.emit(current_health, max_health)
```

#### 2. The Sensor: `HurtboxComponent`

```gdscript
# ==============================================================================
# COMPONENT: Spatial Damage Receiver (Area3D)
# ==============================================================================
class_name HurtboxComponent
extends Area3D

signal received_damage_packet(raw_amount: float, attacker_node: Node)

@export var health_component: HealthComponent
@export var invulnerability_duration: float = 0.25

var _is_invulnerable: bool = false
var _invulnerability_timer: Timer

func _ready() -> void:
    # Ensure monitoring flags are strictly configured for hurtbox duties
    monitoring = false
    monitorable = true
    _setup_timer()

func _setup_timer() -> void:
    _invulnerability_timer = Timer.new()
    _invulnerability_timer.one_shot = true
    _invulnerability_timer.timeout.connect(func() -> void: _is_invulnerable = false)
    add_child(_invulnerability_timer)

## Called by incoming HitboxComponents upon physics overlap
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

#### 3. The Striker: `HitboxComponent`

```gdscript
# ==============================================================================
# COMPONENT: Spatial Damage Delivery (Area3D)
# ==============================================================================
class_name HitboxComponent
extends Area3D

signal hit_confirmed(target_hurtbox: HurtboxComponent)

@export var base_damage: float = 25.0

func _ready() -> void:
    # Hitboxes must actively search for passive Hurtboxes
    monitoring = true
    monitorable = false
    area_entered.connect(_on_area_entered)

func _on_area_entered(area: Area3D) -> void:
    if area is HurtboxComponent:
        var hurtbox: HurtboxComponent = area as HurtboxComponent
        hurtbox.accept_hit(base_damage, owner)
        hit_confirmed.emit(hurtbox)
```

---

## 3.3 The State Pattern: Finite State Machines (FSM) & Pushdown Automata

Amateur movement logic often begins as a few simple checks:

```gdscript
# THE "BOOLEAN LABYRINTH" ANTI-PATTERN
func _physics_process(delta: float) -> void:
    if is_on_floor():
        if Input.is_action_just_pressed("jump") and not is_crouching and not is_attacking:
            is_jumping = true
        elif is_dashing and not is_stunned:
            dash()
    elif is_in_air:
        if can_double_jump and not has_wall_jumped:
            double_jump()
```

As features grow, the number of combinations explodes quadratically. A system with $N$ independent Booleans possesses $2^N$ possible states, the vast majority of which represent invalid edge cases (e.g., `is_attacking = true` and `is_stunned = true` simultaneously).

### Formalizing State Transitions

A **Finite State Machine (FSM)** constraints an actor to **exactly one active state at any discrete moment**:

```
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

### Advanced Architectural Variant: The Pushdown Automaton (State Stack)

Standard FSMs discard historical state upon transition. If a player opens an inventory menu, gets stunned, or executes an interruptive sword slash during a patrol route, a standard FSM does not know which state to return to.

A **Pushdown Automaton** utilizes a **Stack (LIFO)** data structure to store state history:
- ⤷ **`push_state(new_state)`**: Pauses the current state and places it onto the stack beneath the new active state.
- ⤷ **`pop_state()`**: Terminates the current state and restores the interrupted state from the stack.

```
PUSHDOWN AUTOMATON (STACK TRANSITIONS)
======================================
1. Entity is running:          Stack: [ RUN ]
2. Player initiates an Attack: Stack: [ RUN ][ ATTACK ] (Attack executes over Run)
3. Attack animation ends:      Stack: [ RUN ] (Attack pops off, Run immediately resumes!)
```

### Complete, Production-Grade Pushdown State Machine

```gdscript
# ==============================================================================
# BASE STATE CONTRACT: All entity states must extend this class
# ==============================================================================
class_name State
extends Node

## Weak reference or direct pointer to the owning context
var actor: CharacterBody3D
var state_machine: StateMachine

## Lifecycle hooks: Executed strictly by the state machine
func enter(msg: Dictionary = {}) -> void:
    pass

func exit() -> void:
    pass

func handle_input(event: InputEvent) -> void:
    pass

func update(delta: float) -> void:
    pass

func physics_update(delta: float) -> void:
    pass
```

```gdscript
# ==============================================================================
# STATE MACHINE CONTROLLER (Pushdown Automaton Support)
# ==============================================================================
class_name StateMachine
extends Node

signal transitioned(old_state_name: StringName, new_state_name: StringName)

@export var initial_state: State

## Stack maintaining paused state history for pop operations
var _state_stack: Array[State] = []
var _states: Dictionary = {}

var current_state: State:
    get:
        return _state_stack.back() if not _state_stack.is_empty() else null

func _ready() -> void:
    var actor_node: CharacterBody3D = owner as CharacterBody3D
    assert(actor_node != null, "StateMachine: Owner must be a CharacterBody3D.")

    # Register child nodes as states
    for child in get_children():
        if child is State:
            var s: State = child as State
            _states[s.name] = s
            s.actor = actor_node
            s.state_machine = self

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

## Standard absolute transition: Clears the stack and starts fresh
func change_state(target_state_name: StringName, msg: Dictionary = {}) -> void:
    assert(_states.has(target_state_name), "State %s does not exist." % target_state_name)
    var target: State = _states[target_state_name]
    
    var old_name: StringName = &""
    if current_state != null:
        old_name = current_state.name
        current_state.exit()
    
    _state_stack.clear()
    _state_stack.push_back(target)
    target.enter(msg)
    transitioned.emit(old_name, target_state_name)

## Pushes an interruptive state onto the stack (pausing the current state)
func push_state(target_state_name: StringName, msg: Dictionary = {}) -> void:
    assert(_states.has(target_state_name), "State %s does not exist." % target_state_name)
    var target: State = _states[target_state_name]
    
    var old_name: StringName = &""
    if current_state != null:
        old_name = current_state.name
        current_state.exit() # Pauses execution of interrupted state
        
    _state_stack.push_back(target)
    target.enter(msg)
    transitioned.emit(old_name, target_state_name)

## Restores the previously interrupted state
func pop_state() -> void:
    if _state_stack.size() <= 1:
        push_warning("StateMachine: Cannot pop the root baseline state.")
        return
        
    var abandoning: State = _state_stack.pop_back()
    abandoning.exit()
    
    var resumed: State = current_state
    resumed.enter({"resumed": true})
    transitioned.emit(abandoning.name, resumed.name)
```

#### Concrete State Implementations

```gdscript
# ==============================================================================
# CONCRETE STATE: Player Idle
# ==============================================================================
class_name IdleState
extends State

func enter(msg: Dictionary = {}) -> void:
    actor.velocity.x = 0.0
    actor.velocity.z = 0.0

func physics_update(delta: float) -> void:
    # Check transitions
    if not actor.is_on_floor():
        state_machine.change_state(&"Airborne")
        return

    var input_vector: Vector2 = Input.get_vector("move_left", "move_right", "move_forward", "move_backward")
    if not input_vector.is_zero_approx():
        state_machine.change_state(&"Run")
        return

    if Input.is_action_just_pressed("attack"):
        # Push attack: returns to Idle when complete!
        state_machine.push_state(&"Attack")
        return
```

```gdscript
# ==============================================================================
# CONCRETE STATE: Player Attack (Transient State)
# ==============================================================================
class_name AttackState
extends State

@export var attack_duration: float = 0.4
var _elapsed: float = 0.0

func enter(msg: Dictionary = {}) -> void:
    _elapsed = 0.0
    # Decelerate actor during attack commitment
    actor.velocity.x *= 0.2
    actor.velocity.z *= 0.2

func physics_update(delta: float) -> void:
    _elapsed += delta
    if _elapsed >= attack_duration:
        # State goal completed: pop back to whatever state was interrupted
        state_machine.pop_state()
```

---

## 3.4 The Observer Pattern: Local Signals vs. Global Event Buses

Godot's `signal` system is an implementation of the **Observer Pattern**. It provides high performance and type safety. However, engineers frequently misuse signals by choosing the wrong scope:

```
                      SIGNAL COMMUNICATION SCOPES
┌──────────────────────────────┬──────────────────────────────┐
│ Local Direct Signals         │ Global Autoload Event Bus    │
├──────────────────────────────┼──────────────────────────────┤
│ Parent-Child boundaries      │ Cross-domain systems         │
│ Tight architectural cohesion  │ Zero structural hierarchy    │
│ High encapsulation           │ Complete spatial independence│
│ Example: Hurtbox -> Health   │ Example: BossDied -> HUD     │
└──────────────────────────────┴──────────────────────────────┘
```

```
LOCAL SIGNALS (Bounded & Explicit)
[ Enemy Scene ]
   ├── [ Hurtbox ] ──(damaged)──> [ Health ] ──(died)──> [ Root Enemy ] (Deallocates self)

GLOBAL EVENT BUS (Dispersed & Asynchronous)
[ Boss Enemy ] ────(Events.boss_slain.emit)────┐
                                               ▼
                         ┌────────────────────────────────────────┐
                         │       GLOBAL EVENT BUS (Autoload)      │
                         └────────────────────────────────────────┘
                                               │
                 ┌─────────────────────────────┼────────────────────────────┐
                 ▼                             ▼                            ▼
         [ HUD Controller ]          [ Quest Tracker ]            [ Audio Engine ]
         "Show Victory Text"         "Mark Objective Done"        "Crossfade BGM"
```

### The Pitfall of the "God Event Bus"
Connecting every signal in your game to a single global Autoload turns your architecture into an untraceable mesh. Finding where an event originated or which subscribers are responding to it becomes nearly impossible in large projects.

> [!tip] 💡 Intuition & Mental Model: The Regional Switchboard
> Treat a Global Event Bus like emergency dispatch services. You do not route calls to your household microwave through national dispatch; you only dispatch matters of global interest:
> - `game_paused(is_paused: bool)`
> - `player_currency_changed(new_balance: int)`
> - `scene_change_requested(scene_path: String)`

### Strongly-Typed Event Bus Implementation

```gdscript
# ==============================================================================
# GLOBAL EVENT BUS: Register in Project Settings -> Autoloads as "Events"
# ==============================================================================
extends Node

# --- SYSTEM & LIFECYCLE CHANNELS ---
signal game_state_changed(is_paused: bool)
signal scene_transition_demanded(target_scene_path: String)

# --- ECONOMY & PROGRESSION CHANNELS ---
signal currency_transacted(currency_id: StringName, delta: int, balance: int)
signal quest_objective_updated(quest_id: StringName, step: int, is_complete: bool)

# --- COMBAT & TELEMETRY CHANNELS ---
signal boss_encounter_initiated(boss_name: String, max_health: float)
signal boss_health_updated(current: float, max: float)
signal player_died
```

---

## 3.5 The Command Pattern: Decoupled Action Execution & Undo Buffers

The **Command Pattern** turns a request into a standalone object. Instead of an input handler mutating game state directly:

```gdscript
# DIRECT MUTATION (Amateur: Impossible to rebind, replay, or undo)
func _unhandled_input(event: InputEvent) -> void:
    if event.is_action_pressed("move_up"):
        player.grid_position.y -= 1
```

The action is encapsulated within an object containing its execution parameters, execution logic, and **inversion logic**:

```
                       COMMAND EXECUTION LIFECYCLE
                       ===========================
                               [ Hardware Input ]
                                       |
                                       v
                             [ Action Factory ]
                                       |
                                       v
                           Creates [ MoveCommand ]
                                       |
                                       v
                    +─────────────────────────────────────+
                    |           Command Invoker           |
                    |  - command.execute(target_actor)    |
                    |  - _undo_stack.push_back(command)   |
                    +─────────────────────────────────────+
                                  /         \
                 Undo Triggered  /           \  Redo Triggered
                                v             v
                    command.undo()           command.execute()
```

### Production Implementation: Reversible Command Subsystem

```gdscript
# ==============================================================================
# ABSTRACT COMMAND INTERFACE
# ==============================================================================
class_name Command
extends RefCounted

## Executes the command payload against the targeted entity
func execute(actor: Node) -> void:
    assert(false, "Command: execute() must be overridden.")

## Inverts the operations performed in execute()
func undo(actor: Node) -> void:
    assert(false, "Command: undo() must be overridden.")
```

```gdscript
# ==============================================================================
# CONCRETE COMMAND: Reversible Discrete Grid Movement
# ==============================================================================
class_name MoveCommand
extends Command

var _direction: Vector2i
var _previous_position: Vector2i

func _init(direction: Vector2i) -> void:
    _direction = direction

func execute(actor: Node) -> void:
    var target: GridPawn = actor as GridPawn
    assert(target != null, "Target must be a GridPawn.")
    
    # Cache state for precise rollback
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

```gdscript
# ==============================================================================
# COMMAND INVOKER: Manages Action History, Undo, and Redo
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
        _undo_stack.pop_front()

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

---

## 3.6 The Strategy Pattern: Runtime Algorithm Swapping via Resources

The **Strategy Pattern** defines a family of interchangeable algorithms, encapsulates each one inside an object, and makes them swappable at runtime. 

In Godot, the idiomatic way to implement the Strategy Pattern is using custom **`Resource`** classes. Because resources are data-driven assets that can contain executable scripts, you can swap behaviors inside the Godot Inspector simply by dragging and dropping `.tres` files.

```
                    THE RESOURCE STRATEGY PATTERN
                    =============================
                          [ Weapon Node ]
                          - @export strategy: FiringStrategy
                          - func fire(): strategy.execute_fire(...)
                                        │
                 ┌──────────────────────┼──────────────────────┐
                 ▼                      ▼                      ▼
         [ HitscanStrategy ]    [ BallisticStrategy ]   [ HomingStrategy ]
         Raycast evaluation     Parabolic physics       Vector steering
```

### Production Strategy Implementation

#### 1. The Strategy Interface

```gdscript
# ==============================================================================
# ABSTRACT STRATEGY RESOURCE: Base Firing Logic
# ==============================================================================
class_name FiringStrategy
extends Resource

## Swappable firing algorithm
func execute_fire(origin: Node3D, target_vector: Vector3, collision_mask: int) -> void:
    assert(false, "FiringStrategy: execute_fire() must be overridden.")
```

#### 2. Concrete Strategy: Instant Hitscan (Raycast)

```gdscript
# ==============================================================================
# CONCRETE STRATEGY: Instant Hitscan Firing (RayCast via Direct Space State)
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

#### 3. Concrete Strategy: Ballistic Projectile

```gdscript
# ==============================================================================
# CONCRETE STRATEGY: Ballistic Physical Projectile Spawn
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

#### 4. The Context Class: Weapon Controller

```gdscript
# ==============================================================================
# CONTEXT: Weapon Controller using Swappable Strategy
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
        
    firing_strategy.execute_fire(self, aim_direction, hit_mask)
```

---

## 3.7 Structural Pattern Comparison Matrix

| Pattern Name | Core Motivation | Primary Godot Mechanism | Fragility Vector |
| :--- | :--- | :--- | :--- |
| **Component Architecture** | Decouples shared behaviors from deep inheritance trees | Child `Node` compositions (`Area3D`, `Node`) | Over-fragmentation; proliferation of tiny nodes |
| **Finite State Machine** | Eliminates nested Boolean flags; guarantees mutually exclusive execution | Isolated `Node` state handlers | Can balloon in size if handling complex parallel states |
| **Pushdown Automata** | Preserves state history during transient interrupts | LIFO `Array` Stack inside FSM controller | Stack overflow if interrupted states fail to pop |
| **Observer (Signal Bus)** | Decouples spatial entities across disparate systems | First-class `signal` or Autoload Singletons | Degenerates into hard-to-trace "spaghetti signals" |
| **Command Pattern** | Encapsulates actions into standalone objects for replays and rollbacks | Polymorphic `RefCounted` objects with `execute()` / `undo()` | Memory overhead if storing deep history stacks |
| **Strategy Pattern** | Swaps execution algorithms at runtime without changing caller code | Custom `Resource` assets (`.tres`) | Incurring memory indirection on simple algorithms |

---

## 3.8 Concrete Applied System: Decoupled Entity Controller

To synthesize **Component Architecture, Pushdown State Machines, Local Signaling, and Dynamic Strategy Injection**, consider this production-grade **Kinematic Actor Architecture**.

```
                           SCENE COMPOSITION HIERARCHY
                           ===========================
                        [ VanguardActor (CharacterBody3D) ]
                         ├── HealthComponent (Node)
                         ├── HurtboxComponent (Area3D)
                         ├── ModularWeapon (Node3D)
                         │    └── [ Injected Strategy: BallisticStrategy.tres ]
                         └── StateMachine (Node)
                              ├── IdleState (Node)
                              ├── RunState (Node)
                              └── StunState (Node)
```

```gdscript
# ==============================================================================
# INTEGRATED ENTITY CONTROLLER
# ==============================================================================
class_name VanguardActor
extends CharacterBody3D

@onready var health_component: HealthComponent = $HealthComponent
@onready var hurtbox_component: HurtboxComponent = $HurtboxComponent
@onready var weapon: ModularWeapon = $ModularWeapon
@onready var state_machine: StateMachine = $StateMachine

func _ready() -> void:
    # 1. Wire internal component pipelines
    hurtbox_component.health_component = health_component
    
    # 2. Hook reactive lifecycle events
    health_component.health_changed.connect(_on_health_changed)
    health_component.entity_died.connect(_on_entity_died)
    
    # 3. Intercept damage interrupts to push stun state
    hurtbox_component.received_damage_packet.connect(_on_damaged)

func _on_damaged(raw_amount: float, attacker: Node) -> void:
    # High damage interrupts current action and pushes temporary Stun state
    if raw_amount >= 30.0:
        state_machine.push_state(&"Stun", {"duration": 0.5})

func _on_health_changed(current: float, maximum: float) -> void:
    # Local reaction: notify local display mesh or trigger flash shader
    pass

func _on_entity_died() -> void:
    # Disable physics collisions safely
    hurtbox_component.set_deferred("monitoring", false)
    hurtbox_component.set_deferred("monitorable", false)
    
    # Transition to terminal state or clean up
    queue_free()
```

---

## ✍️ Linguistic & Grammatical Refinement

> [!info] 📖 Constructive Analysis of Your Follow-Up Directive
> Let us examine the syntax and phrasing of your prompt to maintain linguistic rigor:
> 1. **Punctuation & Clause Boundaries (Comma Splices)**:
>    - ⤷ *Original*: `"use this for reference, do the same thing expanding upon the masterlist's information, proceed with the module 3"`
>    - ↳ **Correction**: The sentence chains three imperative clauses together using only commas. Separate them into clean, structured statements:
>      *"Use this as a reference. Expand upon the master list's information in the same comprehensive manner, and proceed with Module 3."*
> 2. **Definite Article Elision**:
>    - ⤷ *Original*: `"proceed with the module 3"`
>    - ↳ **Correction**: Omit the definite article when a noun is followed by its designating number or label:
>      *"proceed with Module 3"* (or alternatively, *"proceed with the third module"*).