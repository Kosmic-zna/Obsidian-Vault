# 🏛️ Module I: Bridging the Paradigm (Python & Roblox Luau $\rightarrow$ Godot 4)

> [!info] 📖 Lexicon & First Principles: Foundational Mechanical Terms
> Before analysing the inner mechanics of Godot 4, we must establish unambiguous definitions for the core runtime concepts. Misunderstanding these foundational terms leads directly to architectural collapse:
> - ⤷ **Execution Model**: The exact sequence, schedule, and threading rules by which an engine invokes your code across visual and physical CPU ticks.
> - ⤷ **Virtual Method (Engine Callback)**: A pre-defined hook in an engine base class (e.g., `_ready()`, `_process()`) that remains inert until an engineer overrides it in a derived script to inject custom behaviour.
> - ⤷ **Scene Tree**: A directed, rooted tree data structure maintained in continuous engine memory, dictating the processing order, spatial inheritance, and lifetime of every active entity in the simulation.
> - ⤷ **Static Typing**: A programming language discipline where variable types are declared explicitly and verified at parse/compile time, enabling the compiler to emit direct, unboxed bytecode instructions.
> - ⤷ **Dynamic Dispatch**: The runtime mechanism of determining which polymorphic method or memory address to invoke. Dynamic languages incur a lookup penalty on every call; statically typed GDScript bypasses this via deterministic offsets.
> - ⤷ **Deferred Invocation**: Storing a method call or state alteration inside a message queue to be executed strictly at an engine sync point (typically the boundary between physical computation frames), preventing memory corruption during active queries.

---

## 1.1 Architectural Translation Matrix: Roblox Luau vs. Godot 4

Coming from Roblox (`Luau`), your mental model is anchored around a fixed service hierarchy (`Workspace`, `ReplicatedStorage`, `ServerScriptService`, `Players`) populated by primitive instances whose physical properties, rendering, and networking behaviour are largely pre-determined by the engine.

Godot 4 rejects the concept of a fixed service hierarchy entirely. In Godot, **there is only the tree**. 

```
ROBLOX HIERARCHY (Hard-Coded Services)      GODOT 4 ARCHITECTURE (Pure Homogeneous Tree)
               DataModel                                      SceneTree
              /    |    \                                         |
     Workspace  Players  ReplicatedStorage                     Root (Window)
        |                                                         |
     Model (Character)                                       CurrentScene
     ├── Humanoid                                                 |
     ├── Part (Torso)                                  +----------+----------+
     └── Script                                        |                     |
                                                 Player (CharacterBody3D)  WorldEnvironment
                                                 ├── CollisionShape3D
                                                 ├── Camera3D
                                                 └── WeaponController (Node)
```

### Deep Comparative Analysis

| Dimension | Roblox (`Luau`) | Standard Python 3 | Godot 4 (`GDScript`) | Architectural Implication |
| :--- | :--- | :--- | :--- | :--- |
| **Primary Primitive** | `Instance` | `object` | `Object` $\rightarrow$ `RefCounted` or `Node` | Godot separates data objects (ref-counted) from tree entities (manually freed). |
| **Spatial Unit** | `Part` / `Model` | N/A | `Node2D` / `Node3D` | Positioning is not automatic. Pure `Node` objects have zero spatial or rendering overhead. |
| **Modular Prefabs** | `Model` saved to disk/toolbox | Python module (`.py`) | **PackedScene (`.tscn`)** | A scene is simultaneously an editable compound object and an instantiable class template. |
| **Logic Injection** | `Script` / `LocalScript` childed to Part | Functions within module | **Script Inheritance** (`extends Node`) | A script *is* an extension of the node it attaches to, not an external worker inspecting it. |
| **Type Discipline** | Optional Gradual (`--!strict`) | Type Annotations (Mypy) | **Optional Static Typing** | Types in Godot yield concrete bytecode optimisations and catch errors before runtime. |
| **Coroutines** | `task.spawn`, `task.wait(n)` | `asyncio.create_task`, `await` | `await signal` / `await timer` | Yielding pauses execution of that specific call stack until resumed by the engine's main loop. |
| **Indexing** | 1-indexed (`tbl[1]`) | 0-indexed (`lst[0]`) | ==0-indexed (`arr[0]`)== | Failing to adjust for zero-indexing will cause off-by-one errors (`IndexError`). |

### The Unification of "Script" and "Instance"
In Roblox Luau, you routinely construct a `Part` in `Workspace`, create a `Script` as a child node, and write code that references the parent:
```lua
-- ROBLOX LUAU ANTI-PATTERN (External Worker Mentality)
local part = script.Parent
local function onTouched(hit)
    print("Collided with: " .. hit.Name)
end
part.Touched:Connect(onTouched)
```
In Godot 4, you do not child a script to an object as an external observer. **A script *is* the node**. When you attach `player.gd` to a `CharacterBody3D`, that script creates an anonymous sub-class extending `CharacterBody3D`. The methods and properties of `CharacterBody3D` belong directly to `self`:

```gdscript
# GODOT 4 IDIOMATIC INHERITANCE
class_name Player
extends CharacterBody3D

func _ready() -> void:
    # 'velocity', 'move_and_slide()', and 'global_position' are native members of self
    print("Initialised entity at world coordinate: ", global_position)
```

---

## 1.2 The Tripartite Core: `Object`, `Node`, and `Resource`

To write decoupled, high-performance systems, you must categorise every piece of data and behaviour into one of three distinct engine hierarchies:

```
                                  +-------------------+
                                  |      Object       |
                                  +-------------------+
                                    /               \
                                   /                 \
                  +---------------------+      +---------------------+
                  |        Node         |      |     RefCounted      |
                  +---------------------+      +---------------------+
                     /        |        \                  |
                    /         |         \                 |
              Node2D       Node3D      Control    +------------------+
             (Canvas)     (Spatial)     (UI)      |     Resource     |
                                                  +------------------+
                                                    /       |        \
                                                   /        |         \
                                            Texture2D   PackedScene  ItemData
```

### 1. `Object`
The root C++ ancestor of all classes in Godot.
- ⤷ **Memory Discipline**: Fully manual memory management. If you instantiate a raw `Object.new()`, you must explicitly release it from memory via `free()`.
- ↳ **Capabilities**: Supports metadata, properties, first-class signals, and script binding.

### 2. `Node`
The building block of runtime gameplay structure, spatial hierarchy, and tree processing.
- ⤷ **Memory Discipline**: Tree-owned ownership lifecycle.
  - Adding a node to the tree (`add_child()`) binds its lifetime to its parent.
  - Calling `queue_free()` safely deallocates the node and all of its recursive children at the end of the current frame.
  - **Edge Case Warning**: Calling `remove_child(node)` does **not** free the node from memory. It becomes an **Orphan Node**, remaining in RAM until re-added or freed manually.
- ↳ **Execution**: Receives virtual lifecycle notifications (`_ready`, `_process`, `_physics_process`).

### 3. `Resource`
Data containers engineered for serialisation, caching, and cross-node sharing.
- ⤷ **Memory Discipline**: **Reference Counting (`RefCounted`)**.
  - A resource increments an internal counter for every variable holding a reference to it. The instant that counter hits zero, Godot's memory manager reclaims the allocation.
  - Cyclic references ($A \rightarrow B \rightarrow A$) prevent the reference counter from hitting zero, triggering a memory leak.
- ↳ **Shared by Default**: If $50$ enemy nodes load `"res://data/orc_stats.tres"`, all $50$ enemies point to the **exact same memory address**. Modifying an array inside that resource modifies it for every orc in the simulation simultaneously.

> [!tip] 💡 Intuition & Mental Model: The Architect's Analogy
> Visualise your game architecture as a theatrical stage production:
> - **Nodes are the Actors**: They stand on the stage, move through spatial coordinates, speak lines, and interact physically.
> - **Scenes are the Rehearsed Cast Ensembles**: Pre-configured groups of actors (e.g., an Actor holding a prop, equipped with a spotlight) ready to be dropped onto any stage at will.
> - **Resources are the Script and Costume Sheets**: Pure inanimate data written down on paper. Multiple actors can consult the same sheet simultaneously without duplicating the paper.

---

## 1.3 The Engine Lifecycle Pipeline & Frame Execution Order

A common failure mode for developers transitioning from Python or Lua is placing logic into whichever function seems to work first. This causes race conditions, frame-pacing stutter, and input drops. 

Godot's main loop executes in a strictly deterministic sequence every frame:

```
                                [ START OF FRAME ]
                                        |
                   +--------------------+--------------------+
                   | 1. OS & Window Event Dispatch          |
                   |    - Captures Raw Hardware Input        |
                   +--------------------+--------------------+
                                        |
                   +--------------------+--------------------+
                   | 2. Input Processing Pipeline            |
                   |    - _input(event)                      |
                   |    - Control UI Elements (Consume Event)|
                   |    - _unhandled_input(event)            |
                   +--------------------+--------------------+
                                        |
       +--------------------------------+--------------------------------+
       |                                                                 |
       v (Fixed Time Accumulator >= delta)                               v (Every Rendered Frame)
+-------------------------------+                                 +-------------------------------+
| 3. Physics Tick Cycle         |                                 | 4. Idle Frame Cycle           |
|    - _physics_process(delta)  |                                 |    - _process(delta)          |
|    - Kinematic movement       |                                 |    - Dynamic visual updates   |
|    - Physics Server step      |                                 |    - Camera positional sync   |
|    - Area/Body Signals Fire   |                                 |    - UI rendering preparation |
+-------------------------------+                                 +-------------------------------+
       |                                                                 |
       +--------------------------------+--------------------------------+
                                        |
                   +--------------------+--------------------+
                   | 5. Deferred Execution Queue             |
                   |    - call_deferred() invocations        |
                   |    - set_deferred() property updates    |
                   |    - queue_free() memory reclamation    |
                   +--------------------+--------------------+
                                        |
                   +--------------------+--------------------+
                   | 6. RenderingServer Flush                |
                   |    - GPU Rasterisation & Draw Calls     |
                   +--------------------+--------------------+
                                        |
                                 [ END OF FRAME ]
```

### Deterministic Lifecycle Callbacks

#### 1. `_init()`
- ⤷ **Trigger**: Memory allocation time via `MyClass.new()`.
- ↳ **Context**: The node does **not** exist in the Scene Tree. Global coordinates, physics spaces, and child nodes are unavailable. Do not attempt `$ChildNode` lookups here.

#### 2. `_enter_tree()`
- ⤷ **Trigger**: The node enters the active `SceneTree`.
- ↳ **Direction**: **Top-Down**. Parents enter the tree before their children.

#### 3. `_ready()`
- ⤷ **Trigger**: The node and all its descending children have fully entered the tree and initialised.
- ↳ **Direction**: **Bottom-Up**. Children run their `_ready()` callbacks before their parents.
- ↳ **Use Case**: Safe node referencing (`@onready var sprite = $Sprite2D`), hooking signal connections, allocating dynamic pools.

#### 4. `_physics_process(delta: float)`
- ⤷ **Trigger**: Fixed-rate physics tick (default: exactly $\Delta t = \frac{1}{60}\text{ s} \approx 0.01667\text{ s}$).
- ↳ **Context**: Completely decoupled from display refresh rates. Essential for deterministic gameplay mechanics: character movement, gravity integration, collision queries, and raycasting.
- ↳ **The Parameter `delta`**: The exact elapsed physical time. In physics processing, this value is a static constant unless the engine's time scale is adjusted.

#### 5. `_process(delta: float)`
- ⤷ **Trigger**: Visual render frame. Variable frequency governed by the display monitor's refresh rate ($60\text{ Hz}$, $144\text{ Hz}$, $240\text{ Hz}$) or unconstrained GPU limits.
- ↳ **Context**: For non-physics visual polish: UI updates, smooth camera interpolation, procedural mesh deformation, and audio parameter modulation.
- ↳ **The Parameter `delta`**: Fluctuates every single frame. You must multiply any continuous scalar change by `delta` to ensure temporal consistency across diverse hardware:
  $$\Delta x = v \cdot \Delta t$$

#### 6. `_exit_tree()`
- ⤷ **Trigger**: The node is extracted from the tree via `remove_child()` or destruction.
- ↳ **Use Case**: Disconnecting global autoload signals, releasing non-engine resources, flushing pending file writes to disk.

---

## 1.4 The Complete Input Pipeline: Preventing UI Contention

A frequent design flaw in amateur code is the inability to isolate gameplay actions from user interface interactions. If a player clicks an item inside their inventory UI, the underlying character weapon should not discharge.

Godot resolves this via an explicit **Input Propagation Hierarchy**:

```
[ Hardware Device: Keyboard / Mouse / Controller ]
                       |
                       v
             Viewport::push_input()
                       |
                       v
          +-------------------------+
          |   1. _input(event)      |  <-- Global interceptor; runs first for all nodes
          +-------------------------+
                       | (If not marked as handled)
                       v
          +-------------------------+
          | 2. Control Node GUI     |  <-- Consumes clicks on buttons, windows, sliders
          +-------------------------+
                       | (If event passes through unconsumed)
                       v
          +-------------------------+
          | 3. _gui_input(event)    |  <-- Custom controls handling internal events
          +-------------------------+
                       | (If not marked as handled)
                       v
          +-------------------------+
          | 4. _unhandled_input()   |  <-- GAMEPLAY LAYER: Put movement, shooting, jump here!
          +-------------------------+
```

### Input Processing in Practice

```gdscript
# Idiomatic Character Gameplay Input Controller
class_name PlayerInputHandler
extends Node

signal fire_command_issued(target_position: Vector2)
signal jump_command_issued

func _unhandled_input(event: InputEvent) -> void:
    # This block executes ONLY if the click did NOT hit a UI element
    if event.is_action_pressed("fire"):
        var click_position: Vector2 = get_viewport().get_mouse_position()
        fire_command_issued.emit(click_position)
        # Mark as handled to prevent sibling listeners from receiving it
        get_viewport().set_input_as_handled()
        
    elif event.is_action_pressed("jump"):
        jump_command_issued.emit()
        get_viewport().set_input_as_handled()
```

> [!check] 🔢 High-Yield Enumeration: UI Mouse Filter Settings
> When building UI overlays, configure the `mouse_filter` property on every `Control` node:
> 1. `MOUSE_FILTER_STOP` (Default): The control absorbs the mouse click entirely. The event **terminates**; `_unhandled_input` will never receive it.
> 2. `MOUSE_FILTER_PASS`: The control handles the click, but clones the event down to underlying nodes in the spatial tree.
> 3. `MOUSE_FILTER_IGNORE`: The control is completely transparent to mouse events, allowing them to pass through unhindered.

---

## 1.5 GDScript 2.0: Deep Static Typing & Performance Optimisation

In standard Python or untyped GDScript, every variable access involves **dynamic typing overhead**. The engine must perform dictionary-like hash lookups to confirm whether a field exists, determine its memory layout, and unbox its contents before computing a result:

```
DYNAMIC (Slow):   variable  ---> [ Type Inspection ] ---> [ Unbox ] ---> Compute
STATIC (Fast):    variable: float -------------------------------------> Direct SSE Register Compute
```

### Static Typing Syntax & Best Practices

```gdscript
# ==============================================================================
# PROFESSIONAL STATIC TYPING IMPLEMENTATION (Godot 4)
# ==============================================================================
class_name KineticEntity
extends CharacterBody3D

# Explicit type declarations for memory layout efficiency
const BASE_GRAVITY: float = 9.80665
const TERMINAL_VELOCITY: float = 54.0

# Exported variables with hard editor constraints
@export_group("Kinematic Parameters")
@export_range(1.0, 50.0, 0.5) var movement_speed: float = 12.5
@export var acceleration_curve: Curve

# Node referencing with explicit static node casting
@onready var visual_mesh: MeshInstance3D = $VisualMesh
@onready var collision_shape: CollisionShape3D = $CollisionShape3D

# Strongly typed internal state
var current_speed: float = 0.0
var move_direction: Vector3 = Vector3.ZERO
var is_grounded: bool = false

# Return types must be fully declared (void, bool, int, etc.)
func compute_jump_impulse(peak_height: float, time_to_peak: float) -> float:
    # Explicit type inference with operator ':='
    var calculated_gravity: float = (2.0 * peak_height) / (time_to_peak * time_to_peak)
    var initial_velocity: float = calculated_gravity * time_to_peak
    return initial_velocity

func apply_planar_friction(current_vel: Vector3, friction_coefficient: float, delta: float) -> Vector3:
    var horizontal_plane: Vector3 = Vector3(current_vel.x, 0.0, current_vel.z)
    var decayed_plane: Vector3 = horizontal_plane.move_toward(Vector3.ZERO, friction_coefficient * delta)
    return Vector3(decayed_plane.x, current_vel.y, decayed_plane.z)
```

### Static Typing Optimization Benchmarks

When you fully type a GDScript file (`var x: int = 0` and functions returning concrete types):
1. **Opcode Optimisation**: The GDScript compiler replaces slow generic opcodes (`OPCODE_CALL`) with typed variant opcodes (`OPCODE_CALL_TYPED`), bypassing type checks at runtime.
2. **Direct Memory Offsets**: Member variable lookups change from dynamic hash-table queries to fixed memory offset lookups (identical to C++ struct dereferencing).
3. **Engine Call Optimisations**: Interfacing with the core engine C++ APIs no longer requires boxing values into `Variant` wrappers.

> [!warning] ⚠️ Edge Case Trap: Type Incomplete Annotations
> Writing `var inventory = []` creates an untyped `Array` holding dynamic `Variant` elements. Always supply the nested type signature:
> `var inventory: Array[ItemData] = []`
> This enforces type safety inside the array and eliminates boxing allocations whenever elements are accessed or sorted.

---

## 1.6 Architectural Anti-Patterns: Bridging Roblox Habits

### Anti-Pattern 1: The Infinite Polling Loop (`while true do wait()`)

#### The Flawed Roblox Tradition
```lua
-- ROBLOX LUAU CODE: High CPU Waste Polling
while true do
    task.wait(0.1)
    if player.Character.Humanoid.Health < 20 then
        showLowHealthVFX()
    end
end
```

#### The Godot 4 Idiomatic Equivalent: Event-Driven Reactive Architecture
Never poll in a background loop or check states repeatedly inside `_process()` if the state changes sporadically. **Use Signals**.

```gdscript
# GODOT 4: Reactive Observer Execution
class_name HealthComponent
extends Node

signal health_depleted
signal health_dropped_critical(current_health: float)
signal health_updated(new_health: float, maximum_health: float)

@export var maximum_health: float = 100.0
var current_health: float = 100.0:
    set = set_current_health

func set_current_health(value: float) -> void:
    var clamped_value: float = clampf(value, 0.0, maximum_health)
    if is_equal_approx(current_health, clamped_value):
        return
        
    current_health = clamped_value
    health_updated.emit(current_health, maximum_health)
    
    if current_health <= 20.0 and current_health > 0.0:
        health_dropped_critical.emit(current_health)
    elif is_zero_approx(current_health):
        health_depleted.emit()
```

### Anti-Pattern 2: Path Fragility via Hardcoded Trees

#### The Flawed Tradition
```gdscript
# FRAGILE ANTI-PATTERN
func _on_hitbox_body_entered(body: Node) -> void:
    # If ANY node in this chain is renamed or moved, the whole game crashes:
    get_parent().get_parent().get_node("UIManager/HUD/HealthBar").update_bar()
```

#### The Architectural Solution: Inverted Control via Signals
Children must remain completely ignorant of their environmental context.

```gdscript
# DECOUPLED ARCHITECTURAL PATTERN
class_name HurtboxComponent
extends Area3D

# Child simply reports physical events outward
signal damage_taken(amount: float, source_node: Node)

func register_incoming_hit(damage: float, attacker: Node) -> void:
    damage_taken.emit(damage, attacker)
```

The parent entity, which constructed this tree, binds the signal to whatever system needs to respond:

```gdscript
# Root Entity Node assembling the components
extends CharacterBody3D

@onready var hurtbox: HurtboxComponent = $HurtboxComponent
@onready var health: HealthComponent = $HealthComponent

func _ready() -> void:
    # Parent orchestrates its own children
    hurtbox.damage_taken.connect(func(amount: float, attacker: Node) -> void:
        health.set_current_health(health.current_health - amount)
    )
```

---

## 1.7 Memory Management, Garbage Collection & Lifecycle Mechanics

Coming from Roblox (which employs a Lua-based Garbage Collector) or Python (which uses a cyclic GC alongside reference counting), Godot's memory model requires deliberate handling.

```
                              ALL ENGINE ALLOCATIONS
                                        |
                 +----------------------+----------------------+
                 |                                             |
         [ RefCounted Hierarchy ]                       [ Node / Object ]
                 |                                             |
        - Automated reclamation                       - Explicit destruction required
        - Triggered when ref_count == 0               - queue_free() cleans node + children
        - Resources, RefCounted instances             - Orphan nodes leak memory permanently!
```

### The Reference Cycle Catastrophe

Because `Resource` and `RefCounted` rely on standard reference counting, cyclic references **cannot be collected automatically**:

```gdscript
# MEMORY LEAK EXAMPLE: Uncollectable Cycle
class_name ClassA
extends RefCounted

var reference_to_b: RefCounted

class_name ClassB
extends RefCounted

var reference_to_a: RefCounted

func create_leak() -> void:
    var a = ClassA.new() # Ref count of a = 1
    var b = ClassB.new() # Ref count of b = 1
    
    a.reference_to_b = b # Ref count of b = 2
    b.reference_to_a = a # Ref count of a = 2
    
    # End of scope:
    # Local variable 'a' dropped: Ref count of a becomes 1 (held by b)
    # Local variable 'b' dropped: Ref count of b becomes 1 (held by a)
    # RESULT: Both instances remain stuck in RAM forever.
```

#### The Architectural Remedy: `WeakRef`
When two long-lived data objects must retain bidirectional awareness, one reference must be marked weak:

```gdscript
# SECURE DECOUPLED REFERENCE
class_name SecureClassB
extends RefCounted

var weak_reference_to_a: WeakRef

func bind_parent(a: ClassA) -> void:
    # weakref does NOT increment the internal reference counter
    weak_reference_to_a = weakref(a)

func query_parent() -> ClassA:
    if weak_reference_to_a != null:
        var resolved_obj: Object = weak_reference_to_a.get_ref()
        if resolved_obj != null:
            return resolved_obj as ClassA
    return null
```

---

## 1.8 Concrete Applied Implementation: Generic Object Pooler

To unify the lessons of **Scene Trees, Lifecycle Hooks, Dynamic Instantiation, and Memory Integrity**, examine this production-grade, genre-agnostic **Node Object Pooler**.

Object pooling completely eliminates runtime garbage generation, avoids frame-time drops caused by continuous `instantiate()` and `queue_free()` calls, and showcases safe node lifecycle control:

```gdscript
# ==============================================================================
# PRODUCTION-GRADE GENERIC NODE POOL
# ==============================================================================
class_name NodePool
extends Node

## The PackedScene blueprint instantiated to form the pool
@export var scene_template: PackedScene
## The initial allocation quantity pre-warmed into RAM
@export var prewarm_quantity: int = 50
## When active objects exceed capacity, can the pool allocate new instances?
@export var allow_dynamic_growth: bool = true

## Internal tracking queues
var _inactive_pool: Array[Node] = []
var _active_pool: Array[Node] = []

func _ready() -> void:
    assert(scene_template != null, "NodePool Configuration Error: scene_template is unassigned.")
    _prewarm_pool()

func _prewarm_pool() -> void:
    for i in range(prewarm_quantity):
        var instance: Node = _instantiate_element()
        _deactivate_and_store(instance)

func _instantiate_element() -> Node:
    var node: Node = scene_template.instantiate()
    # Add to the pool scene tree so lifecycle hooks fire once
    add_child(node)
    return node

## Retrieves an inactive node from the pool, or allocates a new one if growth is permitted
func acquire() -> Node:
    var candidate: Node = null
    
    if not _inactive_pool.is_empty():
        candidate = _inactive_pool.pop_back()
    elif allow_dynamic_growth:
        candidate = _instantiate_element()
    else:
        push_warning("NodePool Exhaustion: No available elements and allow_dynamic_growth is false.")
        return null

    _active_pool.append(candidate)
    _activate_element(candidate)
    return candidate

## Returns a currently active element back to the dormant storage queue
func release(node: Node) -> void:
    var active_index: int = _active_pool.find(node)
    if active_index == -1:
        push_error("NodePool Error: Attempted to release a node not managed by this pool.")
        return

    _active_pool.remove_at(active_index)
    _deactivate_and_store(node)

func _activate_element(node: Node) -> void:
    # 1. Enable visual presence
    if node is CanvasItem:
        node.visible = true
    elif node is Node3D:
        node.visible = true
        
    # 2. Re-enable processing ticks
    node.set_process(true)
    node.set_physics_process(true)
    node.set_process_unhandled_input(true)
    
    # 3. Inform the node if it implements custom reset hooks
    if node.has_method("on_pool_acquired"):
        node.call("on_pool_acquired")

func _deactivate_and_store(node: Node) -> void:
    # 1. Disable visual presence
    if node is CanvasItem:
        node.visible = false
    elif node is Node3D:
        node.visible = false
        
    # 2. Cease processing ticks completely to consume zero CPU time
    node.set_process(false)
    node.set_physics_process(false)
    node.set_process_unhandled_input(false)
    
    # 3. Inform the node if it implements teardown logic
    if node.has_method("on_pool_released"):
        node.call("on_pool_released")
        
    _inactive_pool.append(node)

func _exit_tree() -> void:
    # Explicit cleanup of internally held references
    _inactive_pool.clear()
    _active_pool.clear()
```

---