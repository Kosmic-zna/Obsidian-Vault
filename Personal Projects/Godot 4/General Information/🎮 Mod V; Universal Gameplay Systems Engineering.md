# 🎮 Module V: Universal Gameplay Systems Engineering

> [!info] 📖 Lexicon & First Principles: Gameplay Systems Engineering
> Transitioning from abstract spatial mathematics to concrete gameplay architecture requires formalizing the low-level systems that govern player experience:
> - ⤷ **Kinematics**: The branch of classical mechanics describing the motion of points, bodies, and systems without consideration of the physical forces (mass, inertia) that cause them. Kinematic character bodies move by direct displacement vectors ($\Delta \vec{p} = \vec{v} \cdot \Delta t$).
> - ⤷ **Discrete Euler Integration**: A first-order numerical procedure for solving ordinary differential equations over finite time intervals ($\Delta t$). Step 1 accumulates velocity from acceleration; Step 2 accumulates position from velocity.
> - ⤷ **Model-View-Controller (MVC)**: An architectural separation of concerns where raw data and mutation rules (Model) exist independently from graphical control nodes (View) and input polling loops (Controller).
> - ⤷ **Screen Trauma**: A non-linear scalar variable ($\tau \in [0, 1]$) representing camera stress, whose mathematical power curves generate organic screen shake without high-frequency stutter.
> - ⤷ **Autonomous Steering**: A decentralized movement paradigm where an agent calculates localized steering force vectors based on environmental targets, neighbors, and obstacles rather than hardcoded spline paths.

---

## 5.1 Kinematics, Physics & Responsive Character Controllers

Platformer and character controllers frequently suffer from "floatiness" or input unresponsiveness when developers rely on arbitrary constants (`gravity = 9.8`, `jump_speed = -400`) and bare input polling. 

A production character controller derives its physical constants through **kinematic calculus** and decouples human perceptual reaction times from physical ticks using **temporal leeway buffers**.

```
                  CONTINUOUS VS. DISCRETE EULER INTEGRATION
=============================================================================
Continuous:   s(t) = s_0 + v_0*t + 0.5*a*t^2   (Analytic curve; expensive)
Discrete:     v_{n+1} = v_n + a * Δt            (Step 1: Velocity accumulation)
              p_{n+1} = p_n + v_{n+1} * Δt      (Step 2: Positional translation)
```

---

### 1. The Physics Body Taxonomy in Godot 4

| Body Type | Scripting Class | Transform Control | Collision Resolution | Target Use Case |
| :--- | :--- | :--- | :--- | :--- |
| **Static Body** | `StaticBody2D` / `3D` | Immovable (or constant velocity) | Unyielding obstacle | Terrain, architectural walls, conveyor belts |
| **Rigid Body** | `RigidBody2D` / `3D` | Physics Server (Forces/Torque) | Iterative impulse solver | Dynamic props, tumbling debris, ragdolls |
| **Character Body** | `CharacterBody2D` / `3D` | Pure Code (`move_and_slide()`) | Kinematic slide & snap | Players, humanoid NPCs, precise robotic agents |
| **Sensor Area** | `Area2D` / `3D` | Transform-bound | Ghost volume (overlap queries) | Triggers, hitboxes, water zones, audio portals |

---

### 2. The Kinematic Jump Calculus: Eliminating Arbitrary Guesswork

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

---

### Step-by-Step Architectural Construction: Kinematic Platformer Controller

Let us construct a production 2D kinematic controller chunk by chunk, examining the low-level mechanics of time accumulation, asymmetric gravity, and forgiveness timers.

#### Chunk 1.1: Jump Calculus & State Declarations

```gdscript
class_name KinematicPlatformerController
extends CharacterBody2D

@export_group("Jump Calculus Parameters")
@export var jump_height: float = 120.0
@export var time_to_apex: float = 0.35
@export var time_to_descent: float = 0.28

@export_group("Forgiveness Windows")
@export var coyote_duration: float = 0.12
@export var jump_buffer_duration: float = 0.10

@export_group("Planar Locomotion")
@export var max_run_speed: float = 320.0
@export var ground_acceleration: float = 1800.0
@export var ground_friction: float = 2400.0
@export var air_acceleration: float = 900.0

var _gravity_rise: float
var _gravity_fall: float
var _jump_impulse: float

var _coyote_timer: float = 0.0
var _buffer_timer: float = 0.0
```

- **`@export var jump_height / time_to_apex / time_to_descent`**: Designers adjust these three metrics exclusively. Instead of tweaking uncalibrated engine numbers, they define exact pixel height and temporal duration.
- **`time_to_descent < time_to_apex`**: Notice the difference in duration ($0.35\text{ s}$ rise vs. $0.28\text{ s}$ fall). This implements **Asymmetric Gravity**—the character rises gracefully, but drops swiftly back to earth, eliminating floatiness.
- **`_coyote_timer` & `_buffer_timer`**: Dedicated floating-point accumulators. Rather than instantiating node-based `Timer` objects (which incur tree overhead, signal dispatching, and dynamic memory allocations), raw float timers run directly in CPU registers.

#### Chunk 1.2: Kinematic Derivation & Temporal Updates

```gdscript
func _ready() -> void:
	_recalculate_kinematics()

func _recalculate_kinematics() -> void:
	_gravity_rise = (2.0 * jump_height) / (time_to_apex * time_to_apex)
	_gravity_fall = (2.0 * jump_height) / (time_to_descent * time_to_descent)
	_jump_impulse = -((2.0 * jump_height) / time_to_apex)

func _update_timers(delta: float) -> void:
	_coyote_timer = maxf(0.0, _coyote_timer - delta)
	_buffer_timer = maxf(0.0, _buffer_timer - delta)
```

- **`_recalculate_kinematics()`**: Derives $g_{\text{rise}}$, $g_{\text{fall}}$, and $v_0$ via our kinematic formulas. Note that `_jump_impulse` is explicitly negative because Godot 2D operates in a screen-space coordinate system where $-Y$ points upwards.
- **`maxf(0.0, _timer - delta)`**: Decrements timers every physics tick without allowing underflow. This simple floating-point subtraction costs a fraction of a nanosecond and leaves zero memory footprint.

#### Chunk 1.3: Jump Logic with Forgiveness Windows

```gdscript
func _handle_jump_input() -> void:
	if Input.is_action_just_pressed(&"jump"):
		_buffer_timer = jump_buffer_duration

	if _buffer_timer > 0.0 and _coyote_timer > 0.0:
		velocity.y = _jump_impulse
		_buffer_timer = 0.0
		_coyote_timer = 0.0
		
	if Input.is_action_just_released(&"jump") and velocity.y < 0.0:
		velocity.y *= 0.5
```

- **`if Input.is_action_just_pressed(&"jump"): _buffer_timer = ...`**: Jump Buffering. If the player presses Jump within $100\text{ ms}$ before landing, the intent is stored.
- **`if _buffer_timer > 0.0 and _coyote_timer > 0.0:`**: The jump triggers if *both* conditions are met: the player recently intended to jump (`_buffer_timer > 0.0`), and they were grounded recently enough to jump (`_coyote_timer > 0.0`). Once executed, both timers are cleared to prevent double-activation.
- **`velocity.y *= 0.5`**: **Variable Jump Height**. If the player releases the jump button while rising ($v_y < 0$), cutting upward velocity in half drops the character early, offering precise tactile control over jump arcs.

#### Chunk 1.4: Directional Movement & Integration

```gdscript
func _apply_vertical_kinematics(delta: float) -> void:
	if not is_on_floor():
		var active_gravity: float = _gravity_rise if velocity.y < 0.0 else _gravity_fall
		velocity.y += active_gravity * delta
	elif velocity.y > 0.0:
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
		if not is_zero_approx(input_x):
			velocity.x = move_toward(velocity.x, target_speed, air_acceleration * delta)

func _physics_process(delta: float) -> void:
	_update_timers(delta)
	_handle_jump_input()
	_apply_vertical_kinematics(delta)
	_apply_horizontal_kinematics(delta)
	
	move_and_slide()
	
	if is_on_floor():
		_coyote_timer = coyote_duration
```

- **`active_gravity = _gravity_rise if velocity.y < 0.0 else _gravity_fall`**: Dynamically switches gravitational constants based on the sign of $v_y$. The moment the jump apex is reached ($v_y \ge 0$), the heavier falling gravity takes over.
- **`move_toward(velocity.x, target_speed, ground_acceleration * delta)`**: A native C++ helper that accelerates velocity toward the target without overshoot.
- **`if is_on_floor(): _coyote_timer = coyote_duration`**: Resets Coyote Time while standing on the floor. When the player walks off a ledge into open air, `_coyote_timer` begins ticking down, preserving their jump window for $120\text{ ms}$.

---

#### The Fully Assembled `KinematicPlatformerController.gd`

```gdscript
# ==============================================================================
# SCRIPT: KinematicPlatformerController.gd
# Architecture: Calculus-Driven Movement & Tactical Forgiveness
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
## Permissible airborne jump window after walking off a ledge (seconds)
@export var coyote_duration: float = 0.12
## Permissible pre-landing input buffering window (seconds)
@export var jump_buffer_duration: float = 0.10

@export_group("Planar Locomotion")
## Maximum horizontal running velocity (pixels/second)
@export var max_run_speed: float = 320.0
## Acceleration applied when initiating horizontal movement on ground
@export var ground_acceleration: float = 1800.0
## Deceleration applied when stopping on ground
@export var ground_friction: float = 2400.0
## Acceleration applied when steering mid-air
@export var air_acceleration: float = 900.0

# Derived kinematic parameters
var _gravity_rise: float
var _gravity_fall: float
var _jump_impulse: float

# Register-level timer accumulators
var _coyote_timer: float = 0.0
var _buffer_timer: float = 0.0

func _ready() -> void:
	_recalculate_kinematics()

## Computes exact gravitational constants and jump velocity from design variables
func _recalculate_kinematics() -> void:
	# g = 2h / t^2
	_gravity_rise = (2.0 * jump_height) / (time_to_apex * time_to_apex)
	_gravity_fall = (2.0 * jump_height) / (time_to_descent * time_to_descent)
	# v0 = 2h / t (Negative in Godot 2D coordinate space)
	_jump_impulse = -((2.0 * jump_height) / time_to_apex)

func _physics_process(delta: float) -> void:
	_update_timers(delta)
	_handle_jump_input()
	_apply_vertical_kinematics(delta)
	_apply_horizontal_kinematics(delta)
	
	move_and_slide()
	
	# Reset coyote window if grounded on current frame
	if is_on_floor():
		_coyote_timer = coyote_duration

func _update_timers(delta: float) -> void:
	_coyote_timer = maxf(0.0, _coyote_timer - delta)
	_buffer_timer = maxf(0.0, _buffer_timer - delta)

func _handle_jump_input() -> void:
	# Buffer jump input upon press
	if Input.is_action_just_pressed(&"jump"):
		_buffer_timer = jump_buffer_duration

	# Discharge jump if both buffer and ground/coyote checks are satisfied
	if _buffer_timer > 0.0 and _coyote_timer > 0.0:
		velocity.y = _jump_impulse
		_buffer_timer = 0.0
		_coyote_timer = 0.0
		
	# Variable jump height: truncate ascent if button is released early
	if Input.is_action_just_released(&"jump") and velocity.y < 0.0:
		velocity.y *= 0.5

func _apply_vertical_kinematics(delta: float) -> void:
	if not is_on_floor():
		# Asymmetric gravity: rising gravity vs. falling gravity
		var active_gravity: float = _gravity_rise if velocity.y < 0.0 else _gravity_fall
		velocity.y += active_gravity * delta
	elif velocity.y > 0.0:
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
		if not is_zero_approx(input_x):
			velocity.x = move_toward(velocity.x, target_speed, air_acceleration * delta)
```

---

## 5.2 Dynamic Cinematic Camera Systems

Attaching a `Camera3D` directly as a static child of a player mesh produces harsh, mechanical movement that can trigger motion sickness. A production camera rig acts as a dynamic tracker with spatial filtering, non-linear trauma damping, and physical occlusion handling.

```
                           THE DYNAMIC CAMERA PIPELINE
=================================================================================
 [ Target Position ] ──► [ Deadzone Bounding Box ] ──► [ Velocity Look-Ahead ]
                                                              │
                                                              ▼
 [ Output Position ] ◄── [ Raycast Occlusion ] ◄── [ 6DOF Trauma Noise ]
```

---

### Step-by-Step Architectural Construction: Cinematic Camera Rig

Let us build a full third-person camera rig featuring velocity look-ahead, exponential tracking, and Simplex-driven trauma shake across 6 Degrees of Freedom (6DOF).

#### Chunk 2.1: Rig Hierarchy & Trauma Model Setup

```gdscript
class_name CinematicCameraRig
extends Node3D

@export_group("Tracking Dynamics")
@export var target_node: Node3D
@export var tracking_decay: float = 8.0
@export var look_ahead_factor: float = 0.25

@export_group("Trauma Parameters")
@export var trauma_decay_rate: float = 1.2
@export var max_shake_offset: Vector3 = Vector3(0.5, 0.5, 0.0)
@export var max_shake_roll: float = 0.1

@onready var spring_arm: SpringArm3D = $SpringArm3D
@onready var camera: Camera3D = $SpringArm3D/Camera3D

var _trauma: float = 0.0
var _noise: FastNoiseLite = FastNoiseLite.new()
var _noise_time: float = 0.0
```

- **`tracking_decay` & `look_ahead_factor`**: Control positional tracking and how far the camera leads in the direction of the target's current velocity vector.
- **`_trauma: float = 0.0`**: Internal stress scalar ($\tau \in [0, 1]$). When an impact occurs, we add to trauma; every frame, trauma decays linearly back to zero.
- **`FastNoiseLite`**: High-performance coherent gradient noise. Unlike white noise (`randf()`), coherent noise guarantees continuous, organic motion between adjacent frames.

#### Chunk 2.2: Velocity-Projected Exponential Tracking

```gdscript
func _process_tracking(delta: float) -> void:
	var target_velocity: Vector3 = Vector3.ZERO
	if target_node is CharacterBody3D:
		target_velocity = (target_node as CharacterBody3D).velocity
	elif target_node is RigidBody3D:
		target_velocity = (target_node as RigidBody3D).linear_velocity
		
	var projected_target: Vector3 = target_node.global_position + (target_velocity * look_ahead_factor)
	
	var decay_weight: float = 1.0 - exp(-tracking_decay * delta)
	global_position = global_position.lerp(projected_target, decay_weight)
```

- **`target_node.global_position + (target_velocity * look_ahead_factor)`**: Velocity Look-Ahead. Leads the camera slightly ahead of the character's movement direction so incoming hazards remain visible in the forward view.
- **`1.0 - exp(-tracking_decay * delta)`**: Frame-rate independent exponential decay (derived in Module IV). The camera glides toward the target at an identical rate whether rendered at $30\text{ FPS}$ or $144\text{ FPS}$.

#### Chunk 2.3: Non-Linear Trauma Shake

```gdscript
func add_trauma(amount: float) -> void:
	_trauma = clampf(_trauma + amount, 0.0, 1.0)

func _process_trauma(delta: float) -> void:
	if is_zero_approx(_trauma):
		camera.transform.origin = Vector3.ZERO
		camera.rotation.z = 0.0
		return
		
	_trauma = maxf(0.0, _trauma - (trauma_decay_rate * delta))
	var shake_power: float = _trauma * _trauma
	
	_noise_time += delta * 60.0
	
	var shake_x: float = _noise.get_noise_2d(_noise_time, 0.0) * max_shake_offset.x * shake_power
	var shake_y: float = _noise.get_noise_2d(0.0, _noise_time) * max_shake_offset.y * shake_power
	var shake_roll: float = _noise.get_noise_2d(_noise_time, _noise_time) * max_shake_roll * shake_power
	
	camera.transform.origin = Vector3(shake_x, shake_y, 0.0)
	camera.rotation.z = shake_roll
```

- **`var shake_power = _trauma * _trauma`**: **The Non-Linear Power Curve**. Human perception of impact is logarithmic. Squaring or cubing trauma ensures light damage creates subtle vibration, while heavy hits deliver visceral impact.
- **`_noise.get_noise_2d(...)`**: Samples orthogonal 2D noise coordinates across time, generating continuous shake offsets across translational ($X, Y$) and rotational Roll ($Z$) axes.

---

#### The Fully Assembled `CinematicCameraRig.gd`

```gdscript
# ==============================================================================
# SCRIPT: CinematicCameraRig.gd
# Architecture: Occlusion-Safe Tracking & Non-Linear Trauma Shaker
# ==============================================================================
class_name CinematicCameraRig
extends Node3D

@export_group("Tracking Dynamics")
## Target entity tracked by the camera rig
@export var target_node: Node3D
## Frame-rate independent exponential tracking speed
@export var tracking_decay: float = 8.0
## Multiplier projecting camera target ahead of current entity velocity
@export var look_ahead_factor: float = 0.25

@export_group("Trauma Parameters")
## Linear reduction rate of trauma per second
@export var trauma_decay_rate: float = 1.2
## Maximum translational displacement applied during maximum trauma (metres)
@export var max_shake_offset: Vector3 = Vector3(0.5, 0.5, 0.0)
## Maximum rotational roll applied during maximum trauma (radians)
@export var max_shake_roll: float = 0.1

@onready var spring_arm: SpringArm3D = $SpringArm3D
@onready var camera: Camera3D = $SpringArm3D/Camera3D

var _trauma: float = 0.0
var _noise: FastNoiseLite = FastNoiseLite.new()
var _noise_time: float = 0.0

func _ready() -> void:
	_noise.seed = 1337
	_noise.frequency = 4.0
	_noise.noise_type = FastNoiseLite.TYPE_SIMPLEX

## Public API: Injects non-linear stress trauma [0.0, 1.0] onto the camera
func add_trauma(amount: float) -> void:
	_trauma = clampf(_trauma + amount, 0.0, 1.0)

func _process(delta: float) -> void:
	if target_node == null:
		return
		
	_process_tracking(delta)
	_process_trauma(delta)

func _process_tracking(delta: float) -> void:
	var target_velocity: Vector3 = Vector3.ZERO
	if target_node is CharacterBody3D:
		target_velocity = (target_node as CharacterBody3D).velocity
	elif target_node is RigidBody3D:
		target_velocity = (target_node as RigidBody3D).linear_velocity
		
	var projected_target: Vector3 = target_node.global_position + (target_velocity * look_ahead_factor)
	
	# Frame-rate independent exponential smoothing
	var decay_weight: float = 1.0 - exp(-tracking_decay * delta)
	global_position = global_position.lerp(projected_target, decay_weight)

func _process_trauma(delta: float) -> void:
	if is_zero_approx(_trauma):
		camera.transform.origin = Vector3.ZERO
		camera.rotation.z = 0.0
		return
		
	# Linear decay of trauma scalar
	_trauma = maxf(0.0, _trauma - (trauma_decay_rate * delta))
	# Quadratic power shaping
	var shake_power: float = _trauma * _trauma
	
	_noise_time += delta * 60.0
	
	# Sample continuous Simplex gradients
	var shake_x: float = _noise.get_noise_2d(_noise_time, 0.0) * max_shake_offset.x * shake_power
	var shake_y: float = _noise.get_noise_2d(0.0, _noise_time) * max_shake_offset.y * shake_power
	var shake_roll: float = _noise.get_noise_2d(_noise_time, _noise_time) * max_shake_roll * shake_power
	
	camera.transform.origin = Vector3(shake_x, shake_y, 0.0)
	camera.rotation.z = shake_roll
```

---

## 5.3 Data-Driven Inventory, Equipment & Modifier Pipelines

Managing item counts inside UI labels (`$Label.text = "x5"`) or calculating stats through ad-hoc mutation (`strength += 10; strength *= 1.2`) inevitably leads to rounding errors, calculation drift, and broken inventory state.

A scalable system separates concerns using **Model-View-Controller (MVC)** patterns and calculates character attributes using an **additive/multiplicative modifier pipeline**.

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

---

### Step-by-Step Architectural Construction: Data-Driven Inventory & Stat Pipelines

Let us construct our inventory models, stack aggregation routines, and deterministic stat calculation pipelines.

#### Chunk 3.1: Item & Slot Resource Models

```gdscript
class_name ItemData
extends Resource

@export var identifier: StringName
@export var display_name: String
@export var max_stack_size: int = 99
@export var icon: Texture2D

class_name SlotData
extends Resource

@export var item_data: ItemData
@export var quantity: int = 1:
	set(value):
		quantity = value
		if quantity <= 0:
			item_data = null
			quantity = 0
```

- **`extends Resource`**: Items are data models, not nodes. Saving them as custom `.tres` resources allows items to be authored in the editor, referenced across multiple slots, and saved to disk with zero node overhead.
- **`set(value): if quantity <= 0: item_data = null`**: Model-level state cleanup. If a stack drops to zero, the slot automatically clears its item reference, preventing empty slots from holding phantom references.

#### Chunk 3.2: Multi-Pass Stack Aggregation

```gdscript
class_name InventoryData
extends Resource

signal inventory_modified(slot_index: int)

@export var slots: Array[SlotData] = []

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
				
	return remaining
```

- **Two-Pass Placement**:
  - **Pass 1** fills existing stacks up to `max_stack_size`.
  - **Pass 2** claims empty slots for any leftover items.
- **`inventory_modified.emit(i)`**: Emits the specific slot index that changed. The UI view listens to this signal and redraws *only* that slot, avoiding expensive full-inventory re-renders.
- **`return remaining`**: Clean transaction interface. Returns `0` if all items were stored, or returns the leftover quantity if the inventory is full so the caller can drop the excess into the world.

#### Chunk 3.3: Deterministic Stat Modifier Pipeline

To calculate dynamic stats (buffs, gear, status effects), we use a structured calculation pipeline:
$$\text{Final Stat} = \left( \text{Base} + \sum \text{Flat} \right) \times \left(1.0 + \sum \text{PercentAdd} \right) \times \prod (1.0 + \text{PercentMult})$$

```gdscript
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
				
	var step_one: float = base_value + flat_sum
	var step_two: float = step_one * (1.0 + percent_add_sum)
	cached_final_value = step_two * percent_mult_product
	
	value_recalculated.emit(cached_final_value)
```

- **`enum ModifierType`**: Separates modifiers into three distinct evaluation stages:
  - `FLAT`: Adds flat values ($+20\text{ HP}$).
  - `PERCENT_ADD`: Additive percentage bonuses ($+10\% + 15\% = +25\%$).
  - `PERCENT_MULT`: Multiplicative bonuses ($\times 1.50$ Berserk buff).
- **`cached_final_value`**: Caches the resolved stat. Systems can read the value instantly without recalculating across modifiers on every query.
- **`recalculate()`**: Rebuilds the final stat deterministically whenever modifiers are added or removed, preventing numerical drift.

---

#### The Fully Assembled `InventoryAndStats.gd`

```gdscript
# ==============================================================================
# SCRIPT: InventoryAndStats.gd
# Architecture: MVC Inventory Models & Deterministic Stat Pipelines
# ==============================================================================

# --- ITEM DATA MODEL ---
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

# --- INVENTORY CONTAINER DATA MODEL ---
class_name InventoryData
extends Resource

signal inventory_modified(slot_index: int)

@export var slots: Array[SlotData] = []

## Initializes empty inventory slots up to capacity
func initialize_empty(capacity: int) -> void:
	slots.clear()
	for i in range(capacity):
		slots.append(SlotData.new())

## Places items across inventory slots; returns unplaced residue
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
					
	# Pass 2: Claim empty slots
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
				
	return remaining

# --- DETERMINISTIC STAT ATTRIBUTE MODIFIER ---
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

## Injects a new modifier and updates the cached stat
func add_modifier(source: StringName, type: ModifierType, magnitude: float) -> void:
	_modifiers.append(StatModifier.new(source, type, magnitude))
	recalculate()

## Removes all modifiers associated with a source identifier (e.g., unequipping an item)
func remove_modifiers_from_source(source: StringName) -> void:
	_modifiers = _modifiers.filter(func(mod: StatModifier) -> bool: return mod.source_id != source)
	recalculate()

## Resolves the final value using the deterministic pipeline
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
				
	# Final = (Base + Sum(Flat)) * (1 + Sum(PercentAdd)) * Prod(1 + PercentMult)
	var step_one: float = base_value + flat_sum
	var step_two: float = step_one * (1.0 + percent_add_sum)
	cached_final_value = step_two * percent_mult_product
	
	value_recalculated.emit(cached_final_value)
```

---

## 5.4 Universal Interaction & Quest Automation Engines

Rather than scattering duplicate trigger code across every chest, door, and NPC, we implement an **Interactable Component** paired with an analytical **Gaze-Angle Scorer**.

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

To find which object the player intends to interact with, candidate targets are evaluated using a weighted scoring formula balancing **Distance** ($d$) and **Gaze Alignment** ($\hat{f} \cdot \hat{d}$):
$$S = \alpha \cdot (\hat{f} \cdot \hat{d}) + (1.0 - \alpha) \cdot \left(1.0 - \frac{d}{d_{\text{max}}}\right)$$

---

### Step-by-Step Architectural Construction: Interaction Detector

Let us break down the candidate selection and weighted evaluation logic.

#### Chunk 4.1: Candidate Registration & Spatial Cull

```gdscript
class_name InteractionDetector
extends Node3D

@export var observer_camera: Camera3D
@export var detection_radius: float = 4.0
@export_range(0.0, 1.0) var angle_weight: float = 0.7

var _active_candidates: Array[InteractableComponent] = []
var current_best_target: InteractableComponent = null

func _evaluate_best_candidate() -> void:
	if _active_candidates.is_empty() or observer_camera == null:
		current_best_target = null
		return
		
	var forward: Vector3 = -observer_camera.global_transform.basis.z.normalized()
	var camera_pos: Vector3 = observer_camera.global_position
	
	var highest_score: float = -INF
	var chosen_node: InteractableComponent = null
```

- **`angle_weight: float = 0.7`**: Weights gaze alignment ($70\%$) higher than physical distance ($30\%$). If a player stands next to an item on their left but looks directly at a door ahead, the system selects the door.
- **`-observer_camera.global_transform.basis.z.normalized()`**: Extracts the camera's true forward vector in world space.

#### Chunk 4.2: Directional Dot Product & Composite Scoring

```gdscript
	for candidate in _active_candidates:
		if not candidate.is_active:
			continue
			
		var to_target: Vector3 = candidate.global_position - camera_pos
		var dist: float = to_target.length()
		
		if dist > detection_radius:
			continue
			
		var dir: Vector3 = to_target / dist
		var dot: float = forward.dot(dir)
		
		# Discard candidates behind the camera plane
		if dot <= 0.2:
			continue
			
		var distance_factor: float = 1.0 - (dist / detection_radius)
		var total_score: float = (angle_weight * dot) + ((1.0 - angle_weight) * distance_factor)
		
		if total_score > highest_score:
			highest_score = total_score
			chosen_node = candidate
			
	current_best_target = chosen_node
```

- **`if dot <= 0.2: continue`**: Immediate cull. Discards any object outside a generous forward field of view (including anything behind the camera).
- **`total_score = (angle_weight * dot) + (...)`**: Evaluates the normalized composite score. The candidate with the highest combined alignment and proximity is selected as the active target.

---

#### The Fully Assembled `InteractionFramework.gd`

```gdscript
# ==============================================================================
# SCRIPT: InteractionFramework.gd
# Architecture: Component-Based World Interaction & Gaze-Angle Scoring
# ==============================================================================

# --- COMPONENT: Attach to any interactive entity ---
class_name InteractableComponent
extends Area3D

signal interacted_by(initiator: Node)

@export var prompt_message: String = "Interact"
@export var is_active: bool = true

## Executes the interaction payload
func execute_interaction(initiator: Node) -> void:
	if is_active:
		interacted_by.emit(initiator)

# --- DETECTOR: Attach to the Player Rig ---
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

## Finds the best interaction candidate using distance and view-angle dot products
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
		
		# Discard objects situated behind the view plane
		if dot <= 0.2:
			continue
			
		var distance_factor: float = 1.0 - (dist / detection_radius)
		var total_score: float = (angle_weight * dot) + ((1.0 - angle_weight) * distance_factor)
		
		if total_score > highest_score:
			highest_score = total_score
			chosen_node = candidate
			
	current_best_target = chosen_node

## Executes interaction on the current best candidate
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

---

### Step-by-Step Architectural Construction: Utility Response Curves

Rather than hardcoding priority checks, **Utility AI** scores every potential action $\in [0.0, 1.0]$ using mathematical response curves based on dynamic world parameters.

#### Chunk 5.1: Mathematical Response Functions

```gdscript
class_name UtilityAIController
extends RefCounted

static func response_logistic(x: float, midpoint: float, steepness: float) -> float:
	return 1.0 / (1.0 + exp(-steepness * (x - midpoint)))

static func response_decay(x: float, decay_rate: float) -> float:
	return exp(-decay_rate * x)
```

- **`response_logistic(...)`**: Generates a smooth, S-shaped activation curve. Useful for behaviors like bravery or retreat—utility stays low until danger crosses `midpoint`, then ramps up quickly.
- **`response_decay(...)`**: Exponential decay curve ($e^{-\lambda x}$). Useful for behaviors where utility drops off sharply as danger or distance increases.

#### Chunk 5.2: Highest-Score Action Selection

```gdscript
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

- Iterates through scored actions and returns the highest-scoring candidate. This cleanly resolves competing priorities without long, fragile chains of nested `if/elif` statements.

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

---

### Step-by-Step Architectural Construction: Flocking Boid Agent

Let us deconstruct each of the three steering calculations line by line.

#### Chunk 7.1: Separation Force Calculation

```gdscript
func _compute_separation(neighbors: Array[FlockingAgent2D]) -> Vector2:
	var steer: Vector2 = Vector2.ZERO
	var count: int = 0
	
	for other in neighbors:
		var dist: float = global_position.distance_to(other.global_position)
		if dist > 0.0 and dist < separation_radius:
			var diff: Vector2 = (global_position - other.global_position).normalized() / dist
			steer += diff
			count += 1
			
	if count > 0:
		steer /= float(count)
		if steer.length_squared() > 0.0:
			steer = (steer.normalized() * max_velocity) - velocity
			
	return steer
```

- **`diff = (global_position - other.global_position).normalized() / dist`**: The inverse-distance repulsion formula. The closer a neighbor is, the stronger the repulsion force pushing away from them.
- **`(steer.normalized() * max_velocity) - velocity`**: Standard Reynolds steering force: $\vec{F}_{\text{steer}} = \vec{v}_{\text{desired}} - \vec{v}_{\text{current}}$.

#### Chunk 7.2: Alignment Force Calculation

```gdscript
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
```

- Computes the average velocity vector of all local flockmates. Steers the agent to match the group's heading and travel speed.

#### Chunk 7.3: Cohesion Force Calculation

```gdscript
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
		var desired_heading: Vector2 = (center_of_mass - global_position).normalized() * max_velocity
		return desired_heading - velocity
		
	return Vector2.ZERO
```

- Sums neighbor positions to locate the local group's center of mass, steering the agent toward the center to keep the flock together.

#### Chunk 7.4: Composite Force Blending & Kinematic Step

```gdscript
func _physics_process(delta: float) -> void:
	var neighbors: Array[FlockingAgent2D] = _gather_flock_neighbors()
	
	var force_sep: Vector2 = _compute_separation(neighbors)
	var force_ali: Vector2 = _compute_alignment(neighbors)
	var force_coh: Vector2 = _compute_cohesion(neighbors)
	
	var composite_steering: Vector2 = (force_sep * weight_separation) + \
	                                  (force_ali * weight_alignment) + \
	                                  (force_coh * weight_cohesion)
	                                  
	composite_steering = composite_steering.limit_length(max_steering_force)
	
	velocity = (velocity + composite_steering * delta).limit_length(max_velocity)
	
	if velocity.length_squared() > 10.0:
		rotation = velocity.angle()
		
	move_and_slide()
```

- **`composite_steering.limit_length(max_steering_force)`**: Prevents sudden acceleration spikes by clamping the composite steering force.
- **`velocity = (velocity + ...).limit_length(max_velocity)`**: Integrates steering acceleration into velocity, bounded by maximum travel speed.
- **`rotation = velocity.angle()`**: Aligns the agent's visual rotation with its current heading.

---

#### The Fully Assembled `FlockingAgent2D.gd`

```gdscript
# ==============================================================================
# SCRIPT: FlockingAgent2D.gd
# Architecture: Craig Reynolds' Autonomous Flocking Agent (2D Kinematics)
# ==============================================================================
class_name FlockingAgent2D
extends CharacterBody2D

@export_group("Kinematic Constraints")
## Maximum permissible movement speed (pixels/second)
@export var max_velocity: float = 240.0
## Maximum force magnitude applied per physics tick
@export var max_steering_force: float = 400.0

@export_group("Flocking Weights")
## Multiplier applied to separation repulsion
@export var weight_separation: float = 1.5
## Multiplier applied to heading alignment
@export var weight_alignment: float = 1.0
## Multiplier applied to center-of-mass attraction
@export var weight_cohesion: float = 1.0

@export_group("Sensor Radii")
## Radius within which agents push apart to prevent crowding
@export var separation_radius: float = 40.0
## Radius within which flockmates contribute to alignment and cohesion
@export var neighborhood_radius: float = 120.0

@onready var sensor_area: Area2D = $SensorArea

func _physics_process(delta: float) -> void:
	var neighbors: Array[FlockingAgent2D] = _gather_flock_neighbors()
	
	var force_sep: Vector2 = _compute_separation(neighbors)
	var force_ali: Vector2 = _compute_alignment(neighbors)
	var force_coh: Vector2 = _compute_cohesion(neighbors)
	
	# Weighted vector sum of all steering forces
	var composite_steering: Vector2 = (force_sep * weight_separation) + \
	                                  (force_ali * weight_alignment) + \
	                                  (force_coh * weight_cohesion)
	                                  
	# Clamp steering force to prevent acceleration spikes
	composite_steering = composite_steering.limit_length(max_steering_force)
	
	# Euler velocity integration
	velocity = (velocity + composite_steering * delta).limit_length(max_velocity)
	
	# Align visual rotation with current velocity heading
	if velocity.length_squared() > 10.0:
		rotation = velocity.angle()
		
	move_and_slide()

## Gathers overlapping flock agents from the sensor volume
func _gather_flock_neighbors() -> Array[FlockingAgent2D]:
	var agents: Array[FlockingAgent2D] = []
	for body in sensor_area.get_overlapping_bodies():
		if body != self and body is FlockingAgent2D:
			agents.append(body as FlockingAgent2D)
	return agents

## Separation: Repels from nearby neighbors to avoid crowding
func _compute_separation(neighbors: Array[FlockingAgent2D]) -> Vector2:
	var steer: Vector2 = Vector2.ZERO
	var count: int = 0
	
	for other in neighbors:
		var dist: float = global_position.distance_to(other.global_position)
		if dist > 0.0 and dist < separation_radius:
			# Inverse-distance repulsion
			var diff: Vector2 = (global_position - other.global_position).normalized() / dist
			steer += diff
			count += 1
			
	if count > 0:
		steer /= float(count)
		if steer.length_squared() > 0.0:
			steer = (steer.normalized() * max_velocity) - velocity
			
	return steer

## Alignment: Steers toward the average heading of nearby flockmates
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

## Cohesion: Steers toward the center of mass of nearby flockmates
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
		var desired_heading: Vector2 = (center_of_mass - global_position).normalized() * max_velocity
		return desired_heading - velocity
		
	return Vector2.ZERO
```

---
