# 🎮 Module V: Universal Gameplay Systems Engineering

> [!info] 📖 Lexicon & First Principles: Gameplay Systems Engineering
> Transitioning from abstract mathematics to concrete gameplay architecture requires rigorous formal definitions of the runtime systems that govern player experience:
> - ⤷ **Kinematics**: The branch of classical mechanics describing the motion of points, bodies, and systems without consideration of the physical forces (mass, inertia) that cause them. Kinematic character bodies move by direct displacement vectors ($\Delta \vec{p} = \vec{v} \cdot \Delta t$).
> - ⤷ **Euler Integration**: A first-order numerical procedure for solving ordinary differential equations with a given initial value. Discrete physics steps approximate velocity and position over finite intervals ($\Delta t$).
> - ⤷ **Model-View-Controller (MVC)**: An architectural pattern separating internal data representations and business logic (Model) from graphical visualisations (View) and user input pipelines (Controller).
> - ⤷ **Autonomous Agent**: An entity that perceives its environment through sensors, processes decisions via internal logic, and acts upon that environment using steering forces or kinematic velocity updates without explicit player intervention.
> - ⤷ **Screen Trauma**: A non-linear scalar variable representing internal camera stress ($\tau \in [0, 1]$), whose mathematical shaping generates organic screen shake and translation offsets.

---

## 5.1 Kinematics, Physics & Responsive Character Controllers

Amateur platformer and movement controllers feel floaty, imprecise, or unresponsive because developers couple motion directly to engine defaults rather than calibrating kinematic equations to human psychological reaction times.

```
                  CONTINUOUS VS. DISCRETE EULER INTEGRATION
=============================================================================
Continuous:   s(t) = s_0 + v_0*t + 0.5*a*t^2   (Analytic curve; expensive)
Discrete:     v_{n+1} = v_n + a * Δt            (Step 1: Velocity accumulation)
              p_{n+1} = p_n + v_{n+1} * Δt      (Step 2: Positional translation)
```

### The Physics Body Taxonomy in Godot 4

| Body Type | Scripting Class | Transform Control | Collision Resolution | Target Use Case |
| :--- | :--- | :--- | :--- | :--- |
| **Static Body** | `StaticBody2D` / `3D` | Immovable (or constant velocity) | Unyielding obstacle | Terrain, architectural walls, conveyor belts |
| **Rigid Body** | `RigidBody2D` / `3D` | Physics Server (Forces/Torque) | Iterative impulse solver | Dynamic props, tumbling debris, ragdolls |
| **Character Body** | `CharacterBody2D` / `3D` | Pure Code (`move_and_slide()`) | Kinematic slide & snap | Players, humanoid NPCs, precise robotic agents |
| **Sensor Area** | `Area2D` / `3D` | Transform-bound | Ghost volume (overlap queries) | Triggers, hitboxes, water zones, audio portals |

### The Kinematic Jump Calculus: Eliminating Arbitrary Guesswork

Never configure a character's jump by guessing gravitational values and jump impulses in the Inspector. A professional engineer derives gravity ($g$) and jump impulse velocity ($v_0$) directly from design parameters: **Peak Jump Height** ($h$) and **Time-to-Peak** ($t_{\text{peak}}$).

Using standard kinematic equations under constant gravitational acceleration:
$$v(t) = v_0 - g t \qquad y(t) = v_0 t - \frac{1}{2} g t^2$$

At the apex of the jump, velocity reaches zero ($v(t_{\text{peak}}) = 0$):
$$0 = v_0 - g t_{\text{peak}} \implies v_0 = g t_{\text{peak}}$$

Substitute $v_0$ into the height equation at $t_{\text{peak}}$:
$$h = (g t_{\text{peak}}) t_{\text{peak}} - \frac{1}{2} g t_{\text{peak}}^2 = \frac{1}{2} g t_{\text{peak}}^2$$

Solving for Gravity ($g$) and Initial Velocity ($v_0$):
$$g = \frac{2h}{t_{\text{peak}}^2} \qquad v_0 = \frac{2h}{t_{\text{peak}}}$$

```
                   ASYMMETRIC GRAVITATIONAL TRAJECTORY
=============================================================================
Height (y)
   ▲                       Apex (v = 0)
   │                           ╭───╮
   │                         ╭─╯   ╰─╮  <-- Falling Gravity: g_fall = 2h / (t_fall)^2
   │    Rising Gravity:     ╭╯       │      (Snappier descent prevents "floatiness")
   │    g_rise = 2h/(t_peak)^2       │
   │                      ╭─╯         ╰╮
   │                     ╭╯            │
   └─────────────────────┴─────────────┴────────────────────────► Time (t)
                       t_peak        t_fall
```

### Tactical Forgiveness Mechanics: Coyote Time & Jump Buffering

Human motor responses cannot reliably coordinate single-frame inputs on $60\text{ Hz}$ or $144\text{ Hz}$ displays. Responsive character controllers incorporate temporal leeway buffers:

1. **Coyote Time**: Allows an entity to trigger a full-height jump for a short temporal window ($\Delta t \approx 100\text{--}150\text{ ms}$) *after* their collision shape has walked off a ledge.
2. **Jump Buffering**: Caches a jump command issued within $100\text{--}120\text{ ms}$ *prior* to contacting the floor, discharging the jump on the exact frame floor collision is registered.

```gdscript
# ==============================================================================
# PRODUCTION KINEMATIC CONTROLLER (Asymmetric Gravity & Forgiveness Buffers)
# ==============================================================================
class_name KinematicPlatformerController
extends CharacterBody2D

@export_group("Jump Calculus Parameters")
## Desired jump apex height in pixels
@export var jump_height: float = 120.0
## Duration in seconds to achieve jump apex
@export var time_to_apex: float = 0.35
## Duration in seconds to descend from apex to ground level
@export var time_to_descent: float = 0.28

@export_group("Forgiveness Windows")
@export var coyote_duration: float = 0.12
@export var jump_buffer_duration: float = 0.10

@export_group("Planar Locomotion")
@export var max_run_speed: float = 320.0
@export var ground_acceleration: float = 1800.0
@export var ground_friction: float = 2400.0
@export var air_acceleration: float = 900.0

# Derived mathematical constants
var _gravity_rise: float
var _gravity_fall: float
var _jump_impulse: float

# Temporal tracking timers
var _coyote_timer: float = 0.0
var _buffer_timer: float = 0.0

func _ready() -> void:
    _recalculate_kinematics()

func _recalculate_kinematics() -> void:
    # g = 2h / t^2
    _gravity_rise = (2.0 * jump_height) / (time_to_apex * time_to_apex)
    _gravity_fall = (2.0 * jump_height) / (time_to_descent * time_to_descent)
    # v0 = 2h / t
    _jump_impulse = -((2.0 * jump_height) / time_to_apex)

func _physics_process(delta: float) -> void:
    _update_timers(delta)
    _handle_jump_input()
    _apply_vertical_kinematics(delta)
    _apply_horizontal_kinematics(delta)
    
    move_and_slide()
    
    # Ground state reconciliation
    if is_on_floor():
        _coyote_timer = coyote_duration

func _update_timers(delta: float) -> void:
    _coyote_timer = maxf(0.0, _coyote_timer - delta)
    _buffer_timer = maxf(0.0, _buffer_timer - delta)

func _handle_jump_input() -> void:
    if Input.is_action_just_pressed(&"jump"):
        _buffer_timer = jump_buffer_duration

    # Execute jump if both buffer and coyote conditions are satisfied
    if _buffer_timer > 0.0 and _coyote_timer > 0.0:
        velocity.y = _jump_impulse
        _buffer_timer = 0.0
        _coyote_timer = 0.0
        
    # Variable jump height: release early to truncate jump arc
    if Input.is_action_just_released(&"jump") and velocity.y < 0.0:
        velocity.y *= 0.5

func _apply_vertical_kinematics(delta: float) -> void:
    if not is_on_floor():
        # Select asymmetric gravity: use heavier pull when falling or descending
        var active_gravity: float = _gravity_rise if velocity.y < 0.0 else _gravity_fall
        velocity.y += active_gravity * delta
    elif velocity.y > 0.0:
        # Ground clamping to maintain floor stickiness
        velocity.y = 0.0

func _apply_horizontal_kinematics(delta: float) -> void:
    var input_x: float = Input.get_axis(&"move_left", &"move_right")
    var target_speed: float = input_x * max_run_speed
    
    if is_on_floor():
        if not is_zero_approx(input_x):
            velocity.x = move_toward(velocity.x, target_speed, ground_acceleration * delta)
        else:
            velocity.x = move_toward(velocity.x, 0.0, ground_friction * delta)
    else:
        # Air control integration
        if not is_zero_approx(input_x):
            velocity.x = move_toward(velocity.x, target_speed, air_acceleration * delta)
```

---

## 5.2 Dynamic Cinematic Camera Systems

A naive camera pinned directly as a child of a player mesh produces jarring, mechanical movement that induces visual fatigue. A robust camera acts as an analytical tracking agent with spatial filtering, non-linear trauma damping, and physical occlusion resolution.

```
                           THE DYNAMIC CAMERA PIPELINE
=================================================================================
 [ Target Position ] ──► [ Deadzone Bounding Box ] ──► [ Velocity Look-Ahead ]
                                                              │
                                                              ▼
 [ Output Position ] ◄── [ Raycast Occlusion ] ◄── [ 6DOF Trauma Noise ]
```

### 1. The Screen Trauma Paradigm
Random translational jitter (`randf() * intensity`) creates high-frequency strobe shake. High-quality camera shake relies on a non-linear **Trauma Model**:
- Maintain a scalar variable $\text{Trauma} \in [0.0, 1.0]$.
- When explosions or weapon discharges occur, add trauma: $\text{Trauma} = \min(1.0, \text{Trauma} + \Delta \tau)$.
- Square or cube the trauma value to calculate effective power: $\text{Power} = \text{Trauma}^p$ where $p \in [2, 3]$.
- Modulate smooth, continuous **Gradient Simplex Noise** across time, multiplying by $\text{Power}$ to displace rotation and offset.

### 2. Physical Occlusion and `SpringArm3D`
In 3D third-person systems, exterior architecture between the camera focal point and the lens must not clip into the view frustum. `SpringArm3D` executes an automated raycast or sphere-cast backwards from its origin, snapping the camera forward along the spring vector if an obstruction is detected:

```gdscript
# ==============================================================================
# PRODUCTION 3D CAMERA RIG (Occlusion, Damped Tracking & Trauma Shake)
# ==============================================================================
class_name CinematicCameraRig
extends Node3D

@export_group("Tracking Dynamics")
@export var target_node: Node3D
@export var tracking_decay: float = 8.0
@export var look_ahead_factor: float = 0.25

@export_group("Trauma Parameters")
@export var trauma_decay_rate: float = 1.2
@export var max_shake_offset: Vector3 = Vector3(0.5, 0.5, 0.0)
@export var max_shake_roll: float = 0.1 # Radians

@onready var spring_arm: SpringArm3D = $SpringArm3D
@onready var camera: Camera3D = $SpringArm3D/Camera3D

var _trauma: float = 0.0
var _noise: FastNoiseLite = FastNoiseLite.new()
var _noise_time: float = 0.0

func _ready() -> void:
    _noise.seed = 1337
    _noise.frequency = 4.0
    _noise.noise_type = FastNoiseLite.TYPE_SIMPLEX

func add_trauma(amount: float) -> void:
    _trauma = clampf(_trauma + amount, 0.0, 1.0)

func _process(delta: float) -> void:
    if target_node == null:
        return
        
    _process_tracking(delta)
    _process_trauma(delta)

func _process_tracking(delta: float) -> void:
    # 1. Compute velocity projection
    var target_velocity: Vector3 = Vector3.ZERO
    if target_node is CharacterBody3D:
        target_velocity = (target_node as CharacterBody3D).velocity
    elif target_node is RigidBody3D:
        target_velocity = (target_node as RigidBody3D).linear_velocity
        
    var projected_target: Vector3 = target_node.global_position + (target_velocity * look_ahead_factor)
    
    # 2. Frame-rate independent exponential smoothing (Module IV formulation)
    var decay_weight: float = 1.0 - exp(-tracking_decay * delta)
    global_position = global_position.lerp(projected_target, decay_weight)

func _process_trauma(delta: float) -> void:
    if is_zero_approx(_trauma):
        camera.transform.origin = Vector3.ZERO
        camera.rotation.z = 0.0
        return
        
    # Linear decay of trauma scalar
    _trauma = maxf(0.0, _trauma - (trauma_decay_rate * delta))
    var shake_power: float = _trauma * _trauma # Quadratic curve
    
    _noise_time += delta * 60.0
    
    # Sample independent noise coordinates
    var shake_x: float = _noise.get_noise_2d(_noise_time, 0.0) * max_shake_offset.x * shake_power
    var shake_y: float = _noise.get_noise_2d(0.0, _noise_time) * max_shake_offset.y * shake_power
    var shake_roll: float = _noise.get_noise_2d(_noise_time, _noise_time) * max_shake_roll * shake_power
    
    camera.transform.origin = Vector3(shake_x, shake_y, 0.0)
    camera.rotation.z = shake_roll
```

---

## 5.3 Data-Driven Inventory, Equipment & Modifier Pipelines

The fatal mistake in inventory design is attaching data directly to visual UI controls or physical actor scripts. Inventory architecture requires a strict **Model-View-Controller (MVC)** separation:

```
                            INVENTORY MVC ARCHITECTURE
=================================================================================
       [ View / UI Control ]                     [ Model / Pure Data ]
        - GridContainer (Slots)                   - InventoryData (Resource)
        - SlotControl (Icons, Text)               - SlotData (Item + Count)
               ▲                                         │
               │ Emits Signals                           │ Emits Signals
               │ on_slot_clicked                         │ contents_changed
               │                                         ▼
               └──────────────── [ Controller ] ─────────┘
                                 - InventoryManager (Node)
                                 - Validates transactions
                                 - Mutates Model allocations
```

### The Additive & Multiplicative Stat Modifier Pipeline

Equipment, curses, and status buffs must alter entity statistics deterministically. Calculating stat updates through ad-hoc arithmetic (`base += 10; base *= 1.2`) causes rounding drift, order-of-operation bugs, and unmaintainable debuff logic.

Implement a formal calculation pipeline using the standard RPG stat resolution formula:
$$\text{Final Stat} = \left( \text{Base} + \sum_{i} \text{Flat}_i \right) \times \left(1.0 + \sum_{j} \text{PercentAdd}_j \right) \times \prod_{k} (1.0 + \text{PercentMult}_k)$$

```
STAT CALCULATION STAGES:
1. Base Value:        100.0 HP
2. Flat Bonuses:      +20.0 (Ring) + 30.0 (Belt) = +50.0  ---> Subtotal: 150.0
3. Additive Percents: +10% (Buff) + 15% (Talent) = +25%   ---> Subtotal: 150 * 1.25 = 187.5
4. Multipliers:       x 1.50 (Berserk Aura)               ---> Final:    187.5 * 1.5 = 281.25
```

```gdscript
# ==============================================================================
# DATA-DRIVEN INVENTORY & STAT MUTATION FRAMEWORK
# ==============================================================================

# --- ITEM DEFINITION MODEL (Pure Resource) ---
class_name ItemData
extends Resource

@export var identifier: StringName
@export var display_name: String
@export var max_stack_size: int = 99
@export var icon: Texture2D

# --- INVENTORY SLOT DATA MODEL ---
class_name SlotData
extends Resource

@export var item_data: ItemData
@export var quantity: int = 1:
    set(value):
        quantity = value
        if quantity <= 0:
            item_data = null
            quantity = 0

# --- THE AGNOSTIC INVENTORY DATA MODEL ---
class_name InventoryData
extends Resource

signal inventory_modified(slot_index: int)

@export var slots: Array[SlotData] = []

func initialize_empty(capacity: int) -> void:
    slots.clear()
    for i in range(capacity):
        slots.append(SlotData.new())

func add_item(new_item: ItemData, amount: int = 1) -> int:
    var remaining: int = amount
    
    # Pass 1: Aggregate into existing incomplete stacks
    for i in range(slots.size()):
        var slot: SlotData = slots[i]
        if slot.item_data != null and slot.item_data.identifier == new_item.identifier:
            var space_available: int = slot.item_data.max_stack_size - slot.quantity
            if space_available > 0:
                var insertion: int = mini(space_available, remaining)
                slot.quantity += insertion
                remaining -= insertion
                inventory_modified.emit(i)
                if remaining == 0:
                    return 0
                    
    # Pass 2: Occupy empty slots
    for i in range(slots.size()):
        var slot: SlotData = slots[i]
        if slot.item_data == null:
            var insertion: int = mini(new_item.max_stack_size, remaining)
            slot.item_data = new_item
            slot.quantity = insertion
            remaining -= insertion
            inventory_modified.emit(i)
            if remaining == 0:
                return 0
                
    return remaining # Returns unplaced residue if inventory is completely full

# --- DETERMINISTIC STAT ATTRIBUTE CONTROLLER ---
class_name ModifiableAttribute
extends RefCounted

signal value_recalculated(final_value: float)

enum ModifierType { FLAT, PERCENT_ADD, PERCENT_MULT }

class StatModifier:
    var source_id: StringName
    var type: ModifierType
    var magnitude: float
    
    func _init(p_source: StringName, p_type: ModifierType, p_magnitude: float) -> void:
        source_id = p_source
        type = p_type
        magnitude = p_magnitude

var base_value: float = 0.0:
    set(v):
        base_value = v
        recalculate()

var cached_final_value: float = 0.0
var _modifiers: Array[StatModifier] = []

func add_modifier(source: StringName, type: ModifierType, magnitude: float) -> void:
    _modifiers.append(StatModifier.new(source, type, magnitude))
    recalculate()

func remove_modifiers_from_source(source: StringName) -> void:
    _modifiers = _modifiers.filter(func(mod: StatModifier) -> bool: return mod.source_id != source)
    recalculate()

func recalculate() -> void:
    var flat_sum: float = 0.0
    var percent_add_sum: float = 0.0
    var percent_mult_product: float = 1.0
    
    for mod in _modifiers:
        match mod.type:
            ModifierType.FLAT:
                flat_sum += mod.magnitude
            ModifierType.PERCENT_ADD:
                percent_add_sum += mod.magnitude
            ModifierType.PERCENT_MULT:
                percent_mult_product *= (1.0 + mod.magnitude)
                
    # Deterministic Formula Evaluation:
    # Final = (Base + Sum(Flat)) * (1 + Sum(PercentAdd)) * Prod(1 + PercentMult)
    var step_one: float = base_value + flat_sum
    var step_two: float = step_one * (1.0 + percent_add_sum)
    cached_final_value = step_two * percent_mult_product
    
    value_recalculated.emit(cached_final_value)
```

---

## 5.4 Universal Interaction & Quest Automation Engines

Gameplay worlds require entities to evaluate what the player is focusing on without scattering duplicate trigger scripts across hundreds of interactive scenes.

```
                  THE OPTIMAL INTERACTION DISCOVERY CONE
=============================================================================
                    Target B (Dist: 2.5m, Dot: 0.98) ◄── SELECTED (Best Candidate)
                          o
                         /
     Observer           /
        ▲             /
        │   θ        /
        ┼───────────► ──────────────────────── Target A (Dist: 1.0m, Dot: 0.65)
      Player Origin
```

To determine which object the player intends to interact with, evaluate potential targets using a weighted scoring function that balances **Euclidean Distance** ($d$) and **Gaze Alignment** ($\hat{f} \cdot \hat{d}$):

$$S = \alpha \cdot (\hat{f} \cdot \hat{d}) + (1.0 - \alpha) \cdot \left(1.0 - \frac{d}{d_{\text{max}}}\right)$$

where $\alpha \in [0, 1]$ represents the directional alignment weight (typically $0.65\text{--}0.75$).

```gdscript
# ==============================================================================
# UNIVERSAL INTERACTION HARNESS
# ==============================================================================

# --- INTERFACE COMPONENT: Attach to any interactive world entity ---
class_name InteractableComponent
extends Area3D

signal interacted_by(initiator: Node)

@export var prompt_message: String = "Interact"
@export var is_active: bool = true

func execute_interaction(initiator: Node) -> void:
    if is_active:
        interacted_by.emit(initiator)

# --- DETECTOR CONTROLLER: Attach to the Player Rig ---
class_name InteractionDetector
extends Node3D

@export var observer_camera: Camera3D
@export var detection_radius: float = 4.0
@export_range(0.0, 1.0) var angle_weight: float = 0.7

var _active_candidates: Array[InteractableComponent] = []
var current_best_target: InteractableComponent = null

func register_candidate(interactable: InteractableComponent) -> void:
    if not _active_candidates.has(interactable):
        _active_candidates.append(interactable)

func unregister_candidate(interactable: InteractableComponent) -> void:
    _active_candidates.erase(interactable)
    if current_best_target == interactable:
        current_best_target = null

func _process(_delta: float) -> void:
    _evaluate_best_candidate()

func _evaluate_best_candidate() -> void:
    if _active_candidates.is_empty() or observer_camera == null:
        current_best_target = null
        return
        
    var forward: Vector3 = -observer_camera.global_transform.basis.z.normalized()
    var camera_pos: Vector3 = observer_camera.global_position
    
    var highest_score: float = -INF
    var chosen_node: InteractableComponent = null
    
    for candidate in _active_candidates:
        if not candidate.is_active:
            continue
            
        var to_target: Vector3 = candidate.global_position - camera_pos
        var dist: float = to_target.length()
        
        if dist > detection_radius:
            continue
            
        var dir: Vector3 = to_target / dist
        var dot: float = forward.dot(dir)
        
        # Immediate cull of objects situated behind the view plane
        if dot <= 0.2:
            continue
            
        # Normalised score calculation
        var distance_factor: float = 1.0 - (dist / detection_radius)
        var total_score: float = (angle_weight * dot) + ((1.0 - angle_weight) * distance_factor)
        
        if total_score > highest_score:
            highest_score = total_score
            chosen_node = candidate
            
    current_best_target = chosen_node

func execute_active_interaction() -> void:
    if current_best_target != null:
        current_best_target.execute_interaction(owner)
```

---

## 5.5 Artificial Intelligence: Steering, Pathfinding & Utility Systems

Amateur AI scripts rely on rigid paths and nested conditional statements (`if sees_player: chase()`). Scalable game AI separates spatial movement from decision selection.

```
                      MODULAR AI ARCHITECTURE
=============================================================================
 [ Spatial Perception ]  ──►  [ Decision Architecture ]  ──►  [ Spatial Actuator ]
 - NavigationServer3D         - Utility Curves / BT           - Reynolds Steering
 - Ray/Shape Casting          - Behavior States               - move_and_slide()
```

### 1. Craig Reynolds' Autonomous Steering Behaviors
Autonomous movement emerges from combining weighted steering force vectors:
$$\vec{F}_{\text{steering}} = \vec{v}_{\text{desired}} - \vec{v}_{\text{current}}$$
$$\vec{v}_{\text{desired}} = \text{clamp}_{\text{max\_speed}}\left(\frac{\vec{p}_{\text{target}} - \vec{p}_{\text{current}}}{\|\vec{p}_{\text{target}} - \vec{p}_{\text{current}}\|}\right)$$

```
                         SEEK VS. ARRIVE BEHAVIORS
=============================================================================
SEEK:   Full speed toward destination, overshoots target upon arrival.
ARRIVE: Linearly or quadratically dampens desired velocity inside slowing radius R:
        If distance < R:
            desired_velocity = (diff / distance) * max_speed * (distance / R)
```

### 2. Utility AI: Mathematical Response Curves
Rather than hardcoding priority checks, **Utility AI** scores every potential action $\in [0.0, 1.0]$ using mathematical response curves based on dynamic world parameters:

```gdscript
# ==============================================================================
# UTILITY RESPONSE ENGINE & CURVE MODELING
# ==============================================================================
class_name UtilityAIController
extends RefCounted

## Computes a logistic curve utility response (Soft threshold activation)
## Generates smooth S-curve behavior: low response until threshold midpoint, then rapid rise
static func response_logistic(x: float, midpoint: float, steepness: float) -> float:
    return 1.0 / (1.0 + exp(-steepness * (x - midpoint)))

## Computes an exponential decay response
## As danger x increases, utility of reckless actions drops off sharply
static func response_decay(x: float, decay_rate: float) -> float:
    return exp(-decay_rate * x)

## Evaluates action scores to select the highest-ranking candidate
static func select_best_action(actions: Array[Dictionary]) -> StringName:
    var best_action: StringName = &"idle"
    var highest_utility: float = -1.0
    
    for act in actions:
        var score: float = act["utility_score"]
        if score > highest_utility:
            highest_utility = score
            best_action = act["name"]
            
    return best_action
```

---

## 5.6 Reactive User Interface & Container Architecture

Building game UI that runs reliably on different display resolutions requires understanding Godot's **Anchor & Container Pipeline**.

```
                   THE ANCHOR RATIO SYSTEM (0.0 to 1.0)
=============================================================================
(0, 0) Left-Top                                         (1, 0) Right-Top
  ┌────────────────────────────────────────────────────────────┐
  │                                                            │
  │            [ Anchor Minimum: (0.5, 0.5) ]                  │
  │            [ Anchor Maximum: (0.5, 0.5) ]                  │
  │            Center Pinned: Offsets define fixed pixel size  │
  │                                                            │
  └────────────────────────────────────────────────────────────┘
(0, 1) Left-Bottom                                      (1, 1) Right-Bottom
```

### Critical Layout Rules:
1. **Never Manually Set Size/Position on Children of Containers**: When a `Control` is placed inside an `HBoxContainer`, `VBoxContainer`, or `GridContainer`, its position and dimensions are calculated by the parent container. Setting `position` or `size` manually will be overwritten on the next layout pass.
2. **Container Sizing Flags**: Use `size_flags_horizontal` and `size_flags_vertical` (`FILL`, `EXPAND`, `SHRINK_CENTER`) to tell parent containers how to distribute extra space.
3. **Procedural UI Animations via `Tween`**: Never use `_process()` to interpolate UI positions or alpha levels. Use the engine's `Tween` pipeline to animate values smoothly across hardware refresh rates:

```gdscript
## Tactile, bouncy UI presentation using custom easing curves
func present_modal_dialogue(panel: Control) -> void:
    panel.pivot_offset = panel.size * 0.5
    panel.scale = Vector2.ZERO
    panel.modulate.a = 0.0
    
    var tween: Tween = panel.create_tween().set_parallel(true)
    tween.set_trans(Tween.TRANS_BACK).set_ease(Tween.EASE_OUT)
    
    # Scale from 0 to 1 with elastic overshoot
    tween.tween_property(panel, "scale", Vector2.ONE, 0.35)
    # Fade alpha concurrently
    tween.tween_property(panel, "modulate:a", 1.0, 0.20)
```

---

## 5.7 Concrete Applied System: Autonomous Flocking Agent

To integrate the kinematics, spatial queries, and steering behaviors covered throughout this module, examine this production implementation of **Craig Reynolds' Flocking Boids** (`Separation`, `Alignment`, `Cohesion`):

$$\vec{F}_{\text{total}} = w_s \vec{F}_{\text{separation}} + w_a \vec{F}_{\text{alignment}} + w_c \vec{F}_{\text{cohesion}}$$

```
                       REYNOLDS BOID STEERING FORCES
=============================================================================
SEPARATION: Pushes away from neighbors within inner radius (prevents crowding)
ALIGNMENT:  Steers toward average heading/velocity of neighbors
COHESION:   Steers toward average center of mass of neighbors
```

```gdscript
# ==============================================================================
# AUTONOMOUS STEERING FLOCKING AGENT (2D Kinematic Implementation)
# ==============================================================================
class_name FlockingAgent2D
extends CharacterBody2D

@export_group("Kinematic Constraints")
@export var max_velocity: float = 240.0
@export var max_steering_force: float = 400.0

@export_group("Flocking Weights")
@export var weight_separation: float = 1.5
@export var weight_alignment: float = 1.0
@export var weight_cohesion: float = 1.0

@export_group("Sensor Radii")
@export var separation_radius: float = 40.0
@export var neighborhood_radius: float = 120.0

@onready var sensor_area: Area2D = $SensorArea

func _physics_process(delta: float) -> void:
    var neighbors: Array[FlockingAgent2D] = _gather_flock_neighbors()
    
    var force_sep: Vector2 = _compute_separation(neighbors)
    var force_ali: Vector2 = _compute_alignment(neighbors)
    var force_coh: Vector2 = _compute_cohesion(neighbors)
    
    var composite_steering: Vector2 = (force_sep * weight_separation) + \
                                      (force_ali * weight_alignment) + \
                                      (force_coh * weight_cohesion)
                                      
    # Clamp steering force to prevent instantaneous acceleration spikes
    composite_steering = composite_steering.limit_length(max_steering_force)
    
    velocity = (velocity + composite_steering * delta).limit_length(max_velocity)
    
    # Orient agent visual representation along heading
    if velocity.length_squared() > 10.0:
        rotation = velocity.angle()
        
    move_and_slide()

func _gather_flock_neighbors() -> Array[FlockingAgent2D]:
    var agents: Array[FlockingAgent2D] = []
    for body in sensor_area.get_overlapping_bodies():
        if body != self and body is FlockingAgent2D:
            agents.append(body as FlockingAgent2D)
    return agents

func _compute_separation(neighbors: Array[FlockingAgent2D]) -> Vector2:
    var steer: Vector2 = Vector2.ZERO
    var count: int = 0
    
    for other in neighbors:
        var dist: float = global_position.distance_to(other.global_position)
        if dist > 0.0 and dist < separation_radius:
            # Force varies inversely with distance: closer neighbors repel more strongly
            var diff: Vector2 = (global_position - other.global_position).normalized() / dist
            steer += diff
            count += 1
            
    if count > 0:
        steer /= float(count)
        if steer.length_squared() > 0.0:
            steer = (steer.normalized() * max_velocity) - velocity
            
    return steer

func _compute_alignment(neighbors: Array[FlockingAgent2D]) -> Vector2:
    var average_velocity: Vector2 = Vector2.ZERO
    var count: int = 0
    
    for other in neighbors:
        var dist: float = global_position.distance_to(other.global_position)
        if dist < neighborhood_radius:
            average_velocity += other.velocity
            count += 1
            
    if count > 0:
        average_velocity /= float(count)
        var steer: Vector2 = (average_velocity.normalized() * max_velocity) - velocity
        return steer
        
    return Vector2.ZERO

func _compute_cohesion(neighbors: Array[FlockingAgent2D]) -> Vector2:
    var center_of_mass: Vector2 = Vector2.ZERO
    var count: int = 0
    
    for other in neighbors:
        var dist: float = global_position.distance_to(other.global_position)
        if dist < neighborhood_radius:
            center_of_mass += other.global_position
            count += 1
            
    if count > 0:
        center_of_mass /= float(count)
        # Steer toward the center of mass
        var desired_heading: Vector2 = (center_of_mass - global_position).normalized() * max_velocity
        return desired_heading - velocity
        
    return Vector2.ZERO
```

---

