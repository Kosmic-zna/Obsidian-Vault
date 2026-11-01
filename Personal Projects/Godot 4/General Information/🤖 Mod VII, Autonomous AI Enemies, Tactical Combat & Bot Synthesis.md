# 🤖 Module VII: Autonomous AI Enemies, Tactical Combat & Bot Synthesis

> [!info] 📖 Lexicon & First Principles: Artificial Intelligence Architecture
> Transitioning from hardcoded decision trees (`if sees_player: attack()`) to authentic game AI requires establishing formal computational models for decision-making, spatial reasoning, and simulated motor control:
> - ⤷ **Autonomous Agent**: An entity that perceives its environment through simulated sensors, evaluates perceptual data against internal priorities, and acts upon that environment via physical or kinematic actuators.
> - ⤷ **Blackboard**: A centralized, decoupled memory store shared across decision nodes and sensory systems. Sensors write world states to the Blackboard; decision nodes read from it without knowing which sensor gathered the data.
> - ⤷ **Goal-Directed Behavior**: An AI paradigm where an agent does not execute rigid hardcoded sequences, but dynamically selects tasks to transition the current world state into a desired target state.
> - ⤷ **Context Steering**: A vector-field movement paradigm that discretizes continuous radial space into directional slots of **Interest** and **Danger**, allowing an agent to balance pathfinding, obstacle avoidance, and tactical positioning simultaneously without path-thrashing.
> - ⤷ **Combat Token (Ticket System)**: A concurrency throttle that limits how many hostile entities can execute offensive melee lunges or high-damage attacks against the player simultaneously, preventing unfair visual chaos and unavoidable damage spikes.

---

## 7.1 Decision Architectures: HFSM vs. Behaviour Trees vs. Utility AI vs. GOAP

Choosing the wrong AI architecture for an enemy type is catastrophic: simple Finite State Machines collapse under combat complexity, while full Goal-Oriented Action Planners incur unnecessary CPU overhead for simple cannon-fodder minions.

```
                         THE AI ARCHITECTURE CONTINUUM
=================================================================================
 Deterministic & Scripted                                   Emergent & Dynamic
 ◄─────────────────────────────────────────────────────────────────────────────►
  [ HFSM ]               [ Behaviour Trees ]     [ Utility AI ]        [ GOAP ]
  Hardcoded Transitions   Hierarchical Ticks      Mathematical Curves   A* Search on States
  Boss Phase Scripts      Shooters / Stealth RPG  Life-Sims / Dynamic   Emergent Sandbox
```

### Deep Comparative Matrix

| Architecture Dimension | Hierarchical FSM (HFSM) | Behaviour Tree (BT) | Utility AI | GOAP (Goal-Oriented) |
| :--- | :--- | :--- | :--- | :--- |
| **Primary Evaluator** | State transition triggers | Root-to-leaf tick traversal | Continuous scoring functions | Backward-chaining $A^*$ on state graph |
| **Modularity & Reuse** | Low (State transitions tightly coupled) | **Extreme** (Tasks are drag-and-drop nodes) | High (Actions evaluated in isolation) | Extreme (Actions declare pre/post conditions) |
| **Handling Interruption** | Requires explicit transition arrows | Handled natively by Selectors / Aborts | Next tick picks new highest utility score | Plan invalidated; regenerates new plan |
| **Complexity Scaling** | $\mathcal{O}(S^2)$ transition links | $\mathcal{O}(N)$ nodes in tree | $\mathcal{O}(A)$ actions scored | $\mathcal{O}(b^d)$ state-space search |
| **Ideal Game Genre** | Multi-phase boss encounters | Tactical FPS, Stealth games, Action RPGs | Emergent companions, RPG party members | Sandbox survival, immersive sims (*F.E.A.R.*) |

---

## 7.2 The Behaviour Tree (BT) Engine: Execution Rules & Blackboard Mechanics

A **Behaviour Tree** is a directed hierarchical tree evaluated from root to leaf on tick updates. Every node in the tree implements a simple execution contract: when ticked, it must return one of three canonical statuses:
1. `SUCCESS`: The node accomplished its goal.
2. `FAILURE`: The node failed to accomplish its goal.
3. `RUNNING`: The node requires more physical frames to complete its task (e.g., walking to a destination or playing an attack animation).

```
                      THE BEHAVIOUR TREE CONTROL FLOW
=================================================================================
                              [ ROOT ]
                                 │
                                 ▼
                     [ Selector (Fallback: ?) ]
                      Evaluates left to right;
                      Stops on FIRST SUCCESS
                     /                        \
                    ▼                          ▼
       [ Sequence (All-or-Nothing: ->) ]   [ Action: Flee ]
        Evaluates left to right;
        Stops on FIRST FAILURE
       /               \
      ▼                 ▼
 [ Condition: Has Ammo? ] [ Action: Fire Weapon ]
```

### Composite Nodes & Decorators
- ⤷ **Sequence (`->`)**: Executes children sequentially. If any child returns `FAILURE`, the Sequence halts and returns `FAILURE`. It returns `SUCCESS` only if every single child succeeds (logical AND).
- ⤷ **Selector (`?`)**: Executes children sequentially. If any child returns `SUCCESS`, the Selector halts and returns `SUCCESS`. It returns `FAILURE` only if every single child fails (logical OR / Fallback).
- ⤷ **Decorator**: Wraps a single child node to alter its result:
  - `Inverter`: Flips `SUCCESS` $\leftrightarrow$ `FAILURE` (`RUNNING` is untouched).
  - `Repeater`: Re-executes the child $N$ times or infinitely.
  - `TimeLimit`: Forces child `FAILURE` if it remains `RUNNING` past $T$ seconds.

---

### Step-by-Step Architectural Construction: Behaviour Tree Engine

Let us construct our modular Behaviour Tree system chunk by chunk, examining the low-level virtual machine, status state management, composite looping, and blackboard reference binding.

#### Chunk 1.1: Base BTNode Polymorphic Contract & Lifecycle

```gdscript
class_name BTNode
extends RefCounted

enum Status { FRESH, RUNNING, SUCCESS, FAILURE }

var status: Status = Status.FRESH
var blackboard: Dictionary = {}
var actor: CharacterBody3D

func execute(delta: float) -> Status:
	if status != Status.RUNNING:
		_enter()
	status = _tick(delta)
	if status != Status.RUNNING:
		_exit()
	return status

func _enter() -> void: pass
func _tick(_delta: float) -> Status: return Status.SUCCESS
func _exit() -> void: pass
```

- **`extends RefCounted`**: BT nodes are lightweight task representations, not spatial game world objects. Using `RefCounted` ensures task nodes carry zero `Node3D` transform overhead, and their memory is freed automatically without SceneTree registration.
- **`execute(delta)`**: The virtual machine wrapper. If the node was not already `RUNNING` on the previous frame, it invokes `_enter()` to initialize timers or target data. Once `_tick()` produces a definitive terminal state (`SUCCESS` or `FAILURE`), it automatically invokes `_exit()` for cleanup.
- **`var blackboard: Dictionary`**: A reference to a shared memory dictionary passed down the tree. All nodes read from and write to this dictionary rather than maintaining hardcoded references to other nodes.

#### Chunk 1.2: The Sequence Composite (Logical AND)

```gdscript
class_name BTSequence
extends BTNode

var children: Array[BTNode] = []
var _current_child_idx: int = 0

func _enter() -> void:
	_current_child_idx = 0

func _tick(delta: float) -> Status:
	while _current_child_idx < children.size():
		var child: BTNode = children[_current_child_idx]
		child.actor = actor
		child.blackboard = blackboard
		
		var child_status: Status = child.execute(delta)
		
		if child_status == Status.RUNNING:
			return Status.RUNNING
		elif child_status == Status.FAILURE:
			return Status.FAILURE
		elif child_status == Status.SUCCESS:
			_current_child_idx += 1
			
	return Status.SUCCESS
```

- **`_current_child_idx: int = 0`**: State persistence across frames. When a child returns `RUNNING`, the Sequence suspends itself without resetting `_current_child_idx`. On the subsequent frame, it resumes execution *at that exact child* rather than starting from index 0.
- **`if child_status == Status.FAILURE: return Status.FAILURE`**: Short-circuit failure evaluation. If an initial condition check fails (e.g., `HasLineOfSight` returns `FAILURE`), the Sequence terminates immediately, skipping expensive downstream action nodes (such as pathfinding or animation playback).

#### Chunk 1.3: The Selector Composite (Logical OR / Fallback)

```gdscript
class_name BTSelector
extends BTNode

var children: Array[BTNode] = []
var _current_child_idx: int = 0

func _enter() -> void:
	_current_child_idx = 0

func _tick(delta: float) -> Status:
	while _current_child_idx < children.size():
		var child: BTNode = children[_current_child_idx]
		child.actor = actor
		child.blackboard = blackboard
		
		var child_status: Status = child.execute(delta)
		
		if child_status == Status.RUNNING:
			return Status.RUNNING
		elif child_status == Status.SUCCESS:
			return Status.SUCCESS
		elif child_status == Status.FAILURE:
			_current_child_idx += 1
			
	return Status.FAILURE
```

- **Fallback Priority Pipeline**: Children are evaluated strictly from left to right (index 0 upward). This creates natural behavioral priority: index 0 can be `EmergencyHeal`, index 1 `MeleeAttack`, index 2 `RangedAttack`, and index 3 `Patrol`.
- **`if child_status == Status.SUCCESS: return Status.SUCCESS`**: Short-circuit success evaluation. The moment any branch succeeds in handling the current situation, the Selector terminates with `SUCCESS`, avoiding execution of lower-priority fallbacks.

---

#### The Fully Assembled `BehaviourTreeEngine.gd`

```gdscript
# ==============================================================================
# SCRIPT: BehaviourTreeEngine.gd
# Architecture: Core Behaviour Tree Task Virtual Machine & Composites
# ==============================================================================
class_name BTNode
extends RefCounted

## Canonical execution state returned by all Behaviour Tree nodes
enum Status { FRESH, RUNNING, SUCCESS, FAILURE }

var status: Status = Status.FRESH
var blackboard: Dictionary = {}
var actor: CharacterBody3D

## Core execution pump: wraps enter, tick, and exit lifecycles
func execute(delta: float) -> Status:
	if status != Status.RUNNING:
		_enter()
	status = _tick(delta)
	if status != Status.RUNNING:
		_exit()
	return status

## Executed on the first frame this task becomes active
func _enter() -> void: 
	pass

## Executed every frame while the node remains active
func _tick(_delta: float) -> Status: 
	return Status.SUCCESS

## Executed when the node terminates with SUCCESS or FAILURE
func _exit() -> void: 
	pass

# ==============================================================================
# COMPOSITE: BTSequence (Logical AND)
# ==============================================================================
class_name BTSequence
extends BTNode

var children: Array[BTNode] = []
var _current_child_idx: int = 0

func _enter() -> void:
	_current_child_idx = 0

func _tick(delta: float) -> Status:
	while _current_child_idx < children.size():
		var child: BTNode = children[_current_child_idx]
		child.actor = actor
		child.blackboard = blackboard
		
		var child_status: Status = child.execute(delta)
		
		# Propagate child running or failure states up the tree
		if child_status == Status.RUNNING:
			return Status.RUNNING
		elif child_status == Status.FAILURE:
			return Status.FAILURE
		elif child_status == Status.SUCCESS:
			_current_child_idx += 1
			
	return Status.SUCCESS

# ==============================================================================
# COMPOSITE: BTSelector (Logical OR / Fallback)
# ==============================================================================
class_name BTSelector
extends BTNode

var children: Array[BTNode] = []
var _current_child_idx: int = 0

func _enter() -> void:
	_current_child_idx = 0

func _tick(delta: float) -> Status:
	while _current_child_idx < children.size():
		var child: BTNode = children[_current_child_idx]
		child.actor = actor
		child.blackboard = blackboard
		
		var child_status: Status = child.execute(delta)
		
		# Return immediately on running or first successful branch
		if child_status == Status.RUNNING:
			return Status.RUNNING
		elif child_status == Status.SUCCESS:
			return Status.SUCCESS
		elif child_status == Status.FAILURE:
			_current_child_idx += 1
			
	return Status.FAILURE
```

---

## 7.3 Multi-Modal Sensory Perception & Target Memory

Realistic enemies must never "cheat" by reading player variables directly from RAM. An immersive adversary perceives the world through simulated physical senses: **Vision**, **Audition**, and **Tactile Damage**, governed by reaction latencies and memory decay.

```
                      THE MULTI-MODAL SENSORY PIPELINE
=================================================================================
 [ 3D Space State ] ──► [ 1. Distance Cull: ||P_target - P_eye|| < R_max ]
                                │
                                ▼
                        [ 2. Field of View Cone: Dot Product >= cos(FOV / 2) ]
                                │
                                ▼
                        [ 3. Multi-Ray Bone Occlusion Query ]
                          (Traces to Head, Center-Chest, Pelvis)
                                │
                                ▼
                        [ 4. Light Level & Camouflage Attenuation ]
                                │
                                ▼
                        [ Continuous Alertness Meter: S += rate * dt ]
                                │
                         (S >= 1.0) ──► Target Confirmed on Blackboard!
```

### The Sight Perception Mathematical Pipeline

1. **Euclidean Radial Check**: Fast squared distance reject ($\|\vec{P}_{\text{target}} - \vec{P}_{\text{eye}}\|^2 \le R_{\text{view}}^2$).
2. **Horizontal Field of View (FOV)**:
   Given normalised eye forward unit vector $\hat{f}$ and normalised vector to target $\hat{d} = \frac{\vec{P}_{\text{target}} - \vec{P}_{\text{eye}}}{\|\vec{P}_{\text{target}} - \vec{P}_{\text{eye}}\|}$:
   $$\hat{f} \cdot \hat{d} \ge \cos\left(\frac{\theta_{\text{FOV}}}{2}\right)$$
3. **Multi-Ray Structural Occlusion**: A single raycast to the player's origin frequently fails when the player peeks from behind cover. Tracing three distinct rays to skeletal points (**Head**, **Thorax**, **Pelvis**) guarantees detection if any body segment is exposed.
4. **Awareness Accumulation**: Rather than flipping from `Idle` to `Combat` in $1\text{ frame}$ (which feels mechanical), scale awareness accumulation rate $r_{\text{detect}}$ inversely with distance:
   $$r_{\text{detect}} = \left(1.0 - \frac{d}{R_{\text{view}}}\right) \cdot M_{\text{light}}$$
   $$\text{Awareness}_{t+\Delta t} = \min\left(1.0, \, \text{Awareness}_t + r_{\text{detect}} \cdot \Delta t\right)$$

---

### Step-by-Step Architectural Construction: 3D Perception Sensor

Let us construct our multi-modal perception sensor chunk by chunk, examining distance threshold culling, cosine-space FOV checks, multi-ray skeletal intersection, and progressive alertness decay.

#### Chunk 2.1: Properties & Signal Declarations

```gdscript
class_name PerceptionSensor3D
extends Node3D

signal target_acquired(target: Node3D)
signal target_lost(last_known_pos: Vector3)

@export_group("Vision Parameters")
@export var sight_radius: float = 24.0
@export var fov_degrees: float = 110.0
@export_flags_3d_physics var occlusion_mask: int = 1
@export var awareness_fill_time: float = 0.65

var current_target: Node3D = null
var awareness_level: float = 0.0
var last_known_position: Vector3 = Vector3.ZERO
var is_alert: bool = false
```

- **`@export_flags_3d_physics var occlusion_mask: int = 1`**: Instructs the sensor which physical collision layers block line-of-sight (typically static world geometry like walls, floors, and crates).
- **`awareness_level: float = 0.0`**: A continuous awareness accumulator $\in [0.0, 1.0]$. The player can briefly dart between cover without immediately triggering full combat alert.
- **`last_known_position: Vector3`**: Target memory. When visual contact breaks, the AI retains this position coordinate to initiate search behaviors.

#### Chunk 2.2: Distance, FOV, and Line-of-Sight Evaluation

```gdscript
func evaluate_vision(candidate: Node3D, delta: float) -> void:
	if candidate == null or not is_instance_valid(candidate):
		_decay_awareness(delta)
		return

	var eye_pos: Vector3 = global_position
	var to_target: Vector3 = candidate.global_position - eye_pos
	var dist_sq: float = to_target.length_squared()
	
	if dist_sq > (sight_radius * sight_radius):
		_decay_awareness(delta)
		return

	var dist: float = sqrt(dist_sq)
	var dir_to_target: Vector3 = to_target / dist
	var eye_forward: Vector3 = -global_transform.basis.z.normalized()

	var half_angle_rad: float = deg_to_rad(fov_degrees * 0.5)
	if eye_forward.dot(dir_to_target) < cos(half_angle_rad):
		_decay_awareness(delta)
		return

	if not _has_line_of_sight(candidate):
		_decay_awareness(delta)
		return

	var proximity_mult: float = 1.0 - (dist / sight_radius)
	var fill_speed: float = (1.0 / maxf(0.01, awareness_fill_time)) * (1.0 + proximity_mult)
	awareness_level = minf(1.0, awareness_level + (fill_speed * delta))

	if awareness_level >= 1.0:
		last_known_position = candidate.global_position
		if not is_alert:
			is_alert = true
			current_target = candidate
			target_acquired.emit(candidate)
```

- **`dist_sq > (sight_radius * sight_radius)`**: Fast-reject squared distance check (Module IV). Discards distant candidates before evaluating trigonometry or raycasts.
- **`eye_forward.dot(dir_to_target) < cos(half_angle_rad)`**: Cosine-space FOV comparison. Bypasses expensive `acos()` operations.
- **`proximity_mult`**: Scales awareness accumulation speed based on proximity. A player standing $2\text{ metres}$ away fills the awareness meter substantially faster than one lurking at the edge of the visual radius.

#### Chunk 2.3: Multi-Ray Skeletal Occlusion & Awareness Decay

```gdscript
func _has_line_of_sight(target: Node3D) -> bool:
	var space_state: PhysicsDirectSpaceState3D = get_world_3d().direct_space_state
	var eye_pos: Vector3 = global_position
	
	var sample_points: Array[Vector3] = [
		target.global_position + Vector3(0, 1.6, 0),
		target.global_position + Vector3(0, 0.9, 0),
		target.global_position + Vector3(0, 0.2, 0)
	]
	
	for point in sample_points:
		var query: PhysicsRayQueryParameters3D = PhysicsRayQueryParameters3D.create(
			eye_pos,
			point,
			occlusion_mask
		)
		var hit: Dictionary = space_state.intersect_ray(query)
		if hit.is_empty():
			return true
			
	return false

func _decay_awareness(delta: float) -> void:
	if awareness_level > 0.0:
		awareness_level = maxf(0.0, awareness_level - (0.5 * delta))
		if awareness_level <= 0.0 and is_alert:
			is_alert = false
			current_target = null
			target_lost.emit(last_known_position)
```

- **`sample_points`**: Traces rays toward the Head ($+1.6\text{ m}$), Thorax ($+0.9\text{ m}$), and Feet ($+0.2\text{ m}$). If *any single ray* reaches the candidate unoccluded, line-of-sight is confirmed.
- **`_decay_awareness(delta)`**: When the player breaks line of sight or leaves the FOV cone, awareness drains gradually. When it reaches 0.0, the target is officially declared lost, emitting `target_lost` with the last recorded coordinates.

---

#### The Fully Assembled `PerceptionSensor3D.gd`

```gdscript
# ==============================================================================
# SCRIPT: PerceptionSensor3D.gd
# Architecture: Multi-Modal Sensory Perception & Target Memory Pipeline
# ==============================================================================
class_name PerceptionSensor3D
extends Node3D

## Dispatched when awareness reaches 1.0 and a target is officially acquired
signal target_acquired(target: Node3D)
## Dispatched when awareness decays back to 0.0 after breaking line of sight
signal target_lost(last_known_pos: Vector3)

@export_group("Vision Parameters")
## Maximum detection distance in metres
@export var sight_radius: float = 24.0
## Total horizontal viewing angle in degrees
@export var fov_degrees: float = 110.0
## Collision mask for physical obstacles that obstruct line-of-sight
@export_flags_3d_physics var occlusion_mask: int = 1
## Time in seconds required to reach full alert at minimum distance
@export var awareness_fill_time: float = 0.65

var current_target: Node3D = null
var awareness_level: float = 0.0 # Bounded in [0.0, 1.0]
var last_known_position: Vector3 = Vector3.ZERO
var is_alert: bool = false

## Evaluates visibility and awareness accumulation against a target entity
func evaluate_vision(candidate: Node3D, delta: float) -> void:
	if candidate == null or not is_instance_valid(candidate):
		_decay_awareness(delta)
		return

	var eye_pos: Vector3 = global_position
	var to_target: Vector3 = candidate.global_position - eye_pos
	var dist_sq: float = to_target.length_squared()
	
	# 1. Fast Euclidean distance cull
	if dist_sq > (sight_radius * sight_radius):
		_decay_awareness(delta)
		return

	var dist: float = sqrt(dist_sq)
	var dir_to_target: Vector3 = to_target / dist
	var eye_forward: Vector3 = -global_transform.basis.z.normalized()

	# 2. Horizontal Field of View (FOV) check
	var half_angle_rad: float = deg_to_rad(fov_degrees * 0.5)
	if eye_forward.dot(dir_to_target) < cos(half_angle_rad):
		_decay_awareness(delta)
		return

	# 3. Multi-Ray Skeletal Occlusion Check
	if not _has_line_of_sight(candidate):
		_decay_awareness(delta)
		return

	# 4. Awareness Accumulation based on proximity
	var proximity_mult: float = 1.0 - (dist / sight_radius)
	var fill_speed: float = (1.0 / maxf(0.01, awareness_fill_time)) * (1.0 + proximity_mult)
	awareness_level = minf(1.0, awareness_level + (fill_speed * delta))

	if awareness_level >= 1.0:
		last_known_position = candidate.global_position
		if not is_alert:
			is_alert = true
			current_target = candidate
			target_acquired.emit(candidate)

## Traces multiple rays to verify skeletal exposure
func _has_line_of_sight(target: Node3D) -> bool:
	var space_state: PhysicsDirectSpaceState3D = get_world_3d().direct_space_state
	var eye_pos: Vector3 = global_position
	
	# Sample Head, Thorax, and Base coordinates
	var sample_points: Array[Vector3] = [
		target.global_position + Vector3(0, 1.6, 0),
		target.global_position + Vector3(0, 0.9, 0),
		target.global_position + Vector3(0, 0.2, 0)
	]
	
	for point in sample_points:
		var query: PhysicsRayQueryParameters3D = PhysicsRayQueryParameters3D.create(
			eye_pos,
			point,
			occlusion_mask
		)
		var hit: Dictionary = space_state.intersect_ray(query)
		if hit.is_empty():
			return true
			
	return false

## Decays awareness linearly when target is unobserved
func _decay_awareness(delta: float) -> void:
	if awareness_level > 0.0:
		awareness_level = maxf(0.0, awareness_level - (0.5 * delta))
		if awareness_level <= 0.0 and is_alert:
			is_alert = false
			current_target = null
			target_lost.emit(last_known_position)
```

---

## 7.4 Spatial Intelligence: Context Steering (Interest vs. Danger Maps)

Traditional obstacle avoidance (e.g., whiskers or simple raycasts that add an avoidance force) suffers from **Local Minima Traps**—an enemy running straight into a corner or doorway oscillates wildly as avoidance vectors fight seeking vectors.

**Context Steering** eliminates this entirely. We discretize continuous planar motion into $N$ fixed radial slots (typically $N = 8$ or $N = 16$). Every frame, the agent computes two independent radial arrays:
1. **Interest Map ($\mathbf{I}$)**: How strongly the agent wants to move in direction $d_i \in [0.0, 1.0]$ (based on target pursuit, patrol paths, flanking angles).
2. **Danger Map ($\mathbf{D}$)**: How dangerous direction $d_i \in [0.0, 1.0]$ is (based on raycasts to walls, incoming projectiles, allied entity separation).

```
                      CONTEXT STEERING DISCRETISATION (N = 8)
=================================================================================
        INTEREST MAP (Move Right/Up)                DANGER MAP (Wall to Right)
                   [0] N                                       [0] N
              [7] ╲  ▲  ╱ [1]                             [7] ╲  ▲  ╱ [1]
                   \ │ /                                       \ │ /
     [6] W ◄───────  ┼  ───────► [2] E (1.0)     [6] W ◄───────  ┼  ───────► [2] E (1.0 DANGER)
                   / │ \                                       / │ \
              [5] ╱  ▼  ╲ [3]                             [5] ╱  ▼  ╲ [3]
                   [4] S                                       [4] S
                                         │
                                         ▼
                      FINAL RESOLVED VECTOR: V = I - D
      Slot [2] (East) is completely eliminated by Danger!
      Agent smoothly selects Slot [1] (North-East) around the corner!
```

### The Algebraic Resolution
For each direction vector $\hat{v}_i$ where $i \in [0, N-1]$:

$$R_i = \max\left(0.0, \, I_i - D_i\right)$$

$$\vec{V}_{\text{final}} = \sum_{i=0}^{N-1} R_i \cdot \hat{v}_i$$

---

### Step-by-Step Architectural Construction: 8-Slot Context Steering Engine

Let us deconstruct the radial slot initialization, directional interest weighting, danger raycasting, and composite vector resolution line by line.

#### Chunk 3.1: Slot Allocation & Angular Discretization

```gdscript
class_name ContextSteeringController3D
extends RefCounted

const NUM_SLOTS: int = 8
var _directions: Array[Vector3] = []
var _interest: Array[float] = []
var _danger: Array[float] = []

func _init() -> void:
	_directions.resize(NUM_SLOTS)
	_interest.resize(NUM_SLOTS)
	_danger.resize(NUM_SLOTS)
	
	for i in range(NUM_SLOTS):
		var angle: float = (float(i) / float(NUM_SLOTS)) * TAU
		_directions[i] = Vector3(sin(angle), 0.0, cos(angle)).normalized()

func clear() -> void:
	for i in range(NUM_SLOTS):
		_interest[i] = 0.0
		_danger[i] = 0.0
```

- **`const NUM_SLOTS: int = 8`**: Discretizes planar space into 8 compass vectors ($45^\circ$ increments). 8 slots offer sufficient resolution to maneuver around obstacles while minimizing CPU raycast overhead.
- **`_directions[i] = Vector3(sin(angle), 0.0, cos(angle))`**: Pre-computes normalized radial unit vectors along the horizontal $XZ$ ground plane.
- **`clear()`**: Flushes interest and danger weights to zero at the start of each frame before running new queries.

#### Chunk 3.2: Interest Projection & Danger Raycasting

```gdscript
func add_interest(target_dir: Vector3, weight: float = 1.0) -> void:
	var dir_flat: Vector3 = Vector3(target_dir.x, 0.0, target_dir.z).normalized()
	for i in range(NUM_SLOTS):
		var dot: float = _directions[i].dot(dir_flat)
		if dot > 0.0:
			_interest[i] = maxf(_interest[i], dot * weight)

func evaluate_danger(actor_pos: Vector3, space_state: PhysicsDirectSpaceState3D, ray_dist: float, mask: int) -> void:
	for i in range(NUM_SLOTS):
		var query: PhysicsRayQueryParameters3D = PhysicsRayQueryParameters3D.create(
			actor_pos,
			actor_pos + (_directions[i] * ray_dist),
			mask
		)
		var hit: Dictionary = space_state.intersect_ray(query)
		if not hit.is_empty():
			var dist: float = actor_pos.distance_to(hit["position"])
			var factor: float = 1.0 - (dist / ray_dist)
			_danger[i] = maxf(_danger[i], factor)
```

- **`_directions[i].dot(dir_flat)`**: Projects the desired movement heading onto each of the 8 radial slots. Slots aligned with the target receive high interest scores (up to $1.0$); perpendicular or opposing slots receive $0.0$.
- **`evaluate_danger(...)`**: Casts physical probe rays along each slot direction. If a ray strikes a wall, danger scales inversely with distance: an obstacle right next to the agent generates near $1.0$ danger, while a distant obstacle produces minimal danger.

#### Chunk 3.3: Vector Subtraction & Final Steering Synthesis

```gdscript
func compute_resolved_direction() -> Vector3:
	var final_vector: Vector3 = Vector3.ZERO
	
	for i in range(NUM_SLOTS):
		var effective_weight: float = maxf(0.0, _interest[i] - _danger[i])
		final_vector += _directions[i] * effective_weight
		
	return final_vector.normalized()
```

- **`maxf(0.0, _interest[i] - _danger[i])`**: The core operation. Danger cancels out interest in blocked directions. If an enemy wants to move East ($I_2 = 1.0$), but a wall is directly to its East ($D_2 = 1.0$), the net weight becomes $0.0$.
- **`final_vector += _directions[i] * effective_weight`**: Sums the remaining unblocked interest components. If East is blocked, the adjacent North-East and South-East vectors pull the agent around the obstruction smoothly, resolving the movement heading without path oscillation.

---

#### The Fully Assembled `ContextSteeringController3D.gd`

```gdscript
# ==============================================================================
# SCRIPT: ContextSteeringController3D.gd
# Architecture: 8-Slot Radial Context Steering Vector Engine
# ==============================================================================
class_name ContextSteeringController3D
extends RefCounted

const NUM_SLOTS: int = 8
var _directions: Array[Vector3] = []
var _interest: Array[float] = []
var _danger: Array[float] = []

func _init() -> void:
	_directions.resize(NUM_SLOTS)
	_interest.resize(NUM_SLOTS)
	_danger.resize(NUM_SLOTS)
	
	# Pre-compute 8 horizontal unit vectors at 45-degree intervals
	for i in range(NUM_SLOTS):
		var angle: float = (float(i) / float(NUM_SLOTS)) * TAU
		_directions[i] = Vector3(sin(angle), 0.0, cos(angle)).normalized()

## Clears interest and danger maps for the current simulation tick
func clear() -> void:
	for i in range(NUM_SLOTS):
		_interest[i] = 0.0
		_danger[i] = 0.0

## Injects interest towards a target direction (e.g. seeking player or waypoint)
func add_interest(target_dir: Vector3, weight: float = 1.0) -> void:
	var dir_flat: Vector3 = Vector3(target_dir.x, 0.0, target_dir.z).normalized()
	for i in range(NUM_SLOTS):
		var dot: float = _directions[i].dot(dir_flat)
		if dot > 0.0:
			_interest[i] = maxf(_interest[i], dot * weight)

## Traces rays to calculate obstacle proximity danger per radial slot
func evaluate_danger(actor_pos: Vector3, space_state: PhysicsDirectSpaceState3D, ray_dist: float, mask: int) -> void:
	for i in range(NUM_SLOTS):
		var query: PhysicsRayQueryParameters3D = PhysicsRayQueryParameters3D.create(
			actor_pos,
			actor_pos + (_directions[i] * ray_dist),
			mask
		)
		var hit: Dictionary = space_state.intersect_ray(query)
		if not hit.is_empty():
			var dist: float = actor_pos.distance_to(hit["position"])
			# Danger scales inversely with proximity to obstacle
			var factor: float = 1.0 - (dist / ray_dist)
			_danger[i] = maxf(_danger[i], factor)

## Resolves the final composite steering direction by subtracting danger from interest
func compute_resolved_direction() -> Vector3:
	var final_vector: Vector3 = Vector3.ZERO
	
	for i in range(NUM_SLOTS):
		var effective_weight: float = maxf(0.0, _interest[i] - _danger[i])
		final_vector += _directions[i] * effective_weight
		
	return final_vector.normalized()
```

---

## 7.5 Combat Coordination: Attack Tokens & Squad Tactics

When $10$ aggressive enemies surround the player, having all $10$ execute an attack simultaneously destroys gameplay feel: the player cannot react, cannot parry, and takes unavoidable lethal burst damage.

Professional combat systems (*DOOM Eternal*, *Batman: Arkham*, *Assassin's Creed*) utilize an **Attack Token System** managed by a high-level **Combat Director**.

```
                   THE SQUAD COMBAT DIRECTOR TOKEN TOPOLOGY
=================================================================================
                            [ Combat Director ]
                      Maintains Pool: Max 2 Attack Tokens
                                     │
                   ┌─────────────────┴─────────────────┐
                   ▼                                   ▼
        [ Token Granted: Slot 1 ]           [ Token Granted: Slot 2 ]
        Enemy Orc A: Lunges forward!        Enemy Orc B: Fires crossbow!
                   │                                   │
                   └─────────────────┬─────────────────┘
                                     │
                   [ Enemies with NO Token: Denied! ]
                   - Enforce circling / taunting
                   - Flank to maintain tactical spacing
                   - Wait for active token return upon attack completion!
```

---

### Step-by-Step Architectural Construction: Combat Director

Let us deconstruct our centralized attack token broker line by line.

#### Chunk 4.1: Token State & Type Separation

```gdscript
class_name CombatDirector
extends Node

@export var max_melee_tokens: int = 2
@export var max_ranged_tokens: int = 1

var _active_melee_tokens: int = 0
var _active_ranged_tokens: int = 0
```

- **Separated Token Pools**: Distinct pools for `melee` and `ranged` attacks allow an archer to shoot from the perimeter while a melee swordsman engages in close combat, preventing enemies from bottlenecking each other across different ranges.

#### Chunk 4.2: Token Requests & Return Validation

```gdscript
func request_attack_token(is_ranged: bool) -> bool:
	if is_ranged:
		if _active_ranged_tokens < max_ranged_tokens:
			_active_ranged_tokens += 1
			return true
	else:
		if _active_melee_tokens < max_melee_tokens:
			_active_melee_tokens += 1
			return true
			
	return false

func return_attack_token(is_ranged: bool) -> void:
	if is_ranged:
		_active_ranged_tokens = maxi(0, _active_ranged_tokens - 1)
	else:
		_active_melee_tokens = maxi(0, _active_melee_tokens - 1)
```

- **`request_attack_token(...)`**: Non-blocking atomic query. If a token is available, the director increments the active count and returns `true`. If full, it returns `false`, instructing the AI state machine to remain in a circling, taunting, or repositioning state.
- **`maxi(0, ... - 1)`**: Prevents integer underflow if tokens are returned erroneously or after an enemy dies mid-swing.

---

#### The Fully Assembled `CombatDirector.gd`

```gdscript
# ==============================================================================
# SCRIPT: CombatDirector.gd
# Architecture: Centralized Squad Combat Token Broker
# ==============================================================================
class_name CombatDirector
extends Node

@export_group("Concurrency Limits")
## Maximum simultaneous melee attacks permitted against the player
@export var max_melee_tokens: int = 2
## Maximum simultaneous ranged/projectile attacks permitted against the player
@export var max_ranged_tokens: int = 1

var _active_melee_tokens: int = 0
var _active_ranged_tokens: int = 0

## Requests an attack authorization token; returns true if granted
func request_attack_token(is_ranged: bool) -> bool:
	if is_ranged:
		if _active_ranged_tokens < max_ranged_tokens:
			_active_ranged_tokens += 1
			return true
	else:
		if _active_melee_tokens < max_melee_tokens:
			_active_melee_tokens += 1
			return true
			
	return false

## Releases an attack token back to the shared pool
func return_attack_token(is_ranged: bool) -> void:
	if is_ranged:
		_active_ranged_tokens = maxi(0, _active_ranged_tokens - 1)
	else:
		_active_melee_tokens = maxi(0, _active_melee_tokens - 1)
```

---

## 7.6 Bot Engineering: Input Abstraction & Humanised Emulation

The amateur method of bot creation is building a script that bypasses character controls and directly overrides `velocity` or teleports orientation via `look_at()`. This creates superhuman, robotic bots with instantaneous reaction times, infallible aim, and disjointed movement that feels completely foreign to human players.

### 1. The Unified Input-Provider Abstraction
Never check `Input.is_action_pressed()` directly in your gameplay or weapon code! Implement an abstract **`InputProvider`** interface:

```
                       UNIFIED CONTROLLER ABSTRACTION
=================================================================================
                      [ Kinematic Entity Controller ]
                                     │
                     Reads: input_provider.get_move_vector()
                            input_provider.is_fire_pressed()
                                     │
                     ┌───────────────┴───────────────┐
                     ▼                               ▼
       [ HardwareInputProvider ]           [ BotBrainInputProvider ]
       Reads: Gamepad, Keyboard, Mouse     Generates: Synthetic Joy Vectors,
                                           Damped Aim Curves, Reaction Delays
```

### 2. Humanising the Bot (Reaction Latency & Spring-Damped Aim)
To prevent "aimbot" behaviour:
1. **Perception Ring Buffer (Latency Injection)**: Route observed player positions through a $200\text{--}300\text{ ms}$ Ring Buffer (from Module II). The bot aims at where the player *was* a quarter of a second ago!
2. **Second-Order Spring-Damper Aiming**: Drive the bot's aim reticle toward the target using a damped harmonic oscillator with subtle Simplex noise injected, matching human arm movement.

---

### Step-by-Step Architectural Construction: Humanized Bot Brain

Let us deconstruct our delayed observation ring buffer, damped harmonic aim tracking, and synthetic input generation line by line.

#### Chunk 5.1: Latency Ingestion & Historical Buffering

```gdscript
class_name BotBrainInputProvider
extends Node

@export_group("Humanisation Constraints")
@export var reaction_delay_ms: int = 220
@export var aim_stiffness: float = 14.0
@export var aim_damping: float = 1.8

var _virtual_move_vector: Vector2 = Vector2.ZERO
var _virtual_fire_pressed: bool = false
var _history_buffer: Array[Dictionary] = []

func record_target_snapshot(target_pos: Vector3) -> void:
	var now: int = Time.get_ticks_msec()
	_history_buffer.append({"time": now, "pos": target_pos})
	
	while not _history_buffer.is_empty() and (_history_buffer.front()["time"] < now - 1000):
		_history_buffer.pop_front()
```

- **`reaction_delay_ms = 220`**: Average human visual reaction time ($200\text{--}250\text{ ms}$). The bot cannot react instantaneously to sudden player direction changes.
- **`record_target_snapshot(...)`**: Samples the target's current position and stores it in a timestamped history buffer, pruning records older than 1 second.

#### Chunk 5.2: Temporal Query & Damped Aim Integration

```gdscript
func get_delayed_target_position() -> Vector3:
	var query_time: int = Time.get_ticks_msec() - reaction_delay_ms
	
	for i in range(_history_buffer.size() - 1, -1, -1):
		if _history_buffer[i]["time"] <= query_time:
			return _history_buffer[i]["pos"]
			
	return _history_buffer.back()["pos"] if not _history_buffer.is_empty() else Vector3.ZERO

func update_synthetic_aim(current_basis: Basis, target_pos: Vector3, delta: float) -> Basis:
	var to_target: Vector3 = (target_pos - current_basis.origin).normalized()
	if to_target.is_zero_approx():
		return current_basis

	var target_quat: Quaternion = Basis.looking_at(to_target, Vector3.UP).get_rotation_quaternion()
	var current_quat: Quaternion = current_basis.get_rotation_quaternion()
	
	var decay: float = 1.0 - exp(-aim_stiffness * delta)
	var smoothed_quat: Quaternion = current_quat.slerp(target_quat, decay)
	
	return Basis(smoothed_quat)
```

- **`get_delayed_target_position()`**: Searches backward through the history buffer to retrieve the target's spatial coordinates from $220\text{ ms}$ in the past.
- **`Basis.looking_at(...).get_rotation_quaternion()`**: Converts the delayed aim direction into a unit Quaternion.
- **`current_quat.slerp(target_quat, decay)`**: Smoothly interpolates the bot's aim toward the delayed target via frame-rate independent exponential SLERP (Module IV), eliminating snap-turning.

#### Chunk 5.3: Abstracted Input Provider Implementation

```gdscript
func set_synthetic_inputs(move_vec: Vector2, fire: bool) -> void:
	_virtual_move_vector = move_vec
	_virtual_fire_pressed = fire

func get_movement_vector() -> Vector2:
	return _virtual_move_vector

func is_action_pressed(action_name: StringName) -> bool:
	if action_name == &"fire":
		return _virtual_fire_pressed
	return false
```

- **Unified Controller Decoupling**: Implements the identical interface contract used by `HardwareInputProvider`. The underlying character controller does not know whether inputs originate from a physical Xbox controller, keyboard, or AI script!

---

#### The Fully Assembled `BotBrainInputProvider.gd`

```gdscript
# ==============================================================================
# SCRIPT: BotBrainInputProvider.gd
# Architecture: Humanized Bot Brain & Input Abstraction Provider
# ==============================================================================
class_name BotBrainInputProvider
extends Node

@export_group("Humanisation Constraints")
## Simulated human visual perception latency in milliseconds
@export var reaction_delay_ms: int = 220
## Angular tracking speed constant
@export var aim_stiffness: float = 14.0
## Tracking damping coefficient
@export var aim_damping: float = 1.8

var _virtual_move_vector: Vector2 = Vector2.ZERO
var _virtual_fire_pressed: bool = false

# Timestamped perception buffer: [{ time: int, pos: Vector3 }]
var _history_buffer: Array[Dictionary] = []

## Records target position for historical reaction delay lookups
func record_target_snapshot(target_pos: Vector3) -> void:
	var now: int = Time.get_ticks_msec()
	_history_buffer.append({"time": now, "pos": target_pos})
	
	# Prune historical snapshots older than 1 second
	while not _history_buffer.is_empty() and (_history_buffer.front()["time"] < now - 1000):
		_history_buffer.pop_front()

## Queries target position from the delayed past to emulate human latency
func get_delayed_target_position() -> Vector3:
	var query_time: int = Time.get_ticks_msec() - reaction_delay_ms
	
	for i in range(_history_buffer.size() - 1, -1, -1):
		if _history_buffer[i]["time"] <= query_time:
			return _history_buffer[i]["pos"]
			
	return _history_buffer.back()["pos"] if not _history_buffer.is_empty() else Vector3.ZERO

## Smoothly updates aim orientation toward a target using exponential SLERP
func update_synthetic_aim(current_basis: Basis, target_pos: Vector3, delta: float) -> Basis:
	var to_target: Vector3 = (target_pos - current_basis.origin).normalized()
	if to_target.is_zero_approx():
		return current_basis

	var target_quat: Quaternion = Basis.looking_at(to_target, Vector3.UP).get_rotation_quaternion()
	var current_quat: Quaternion = current_basis.get_rotation_quaternion()
	
	# Frame-rate independent exponential interpolation
	var decay: float = 1.0 - exp(-aim_stiffness * delta)
	var smoothed_quat: Quaternion = current_quat.slerp(target_quat, decay)
	
	return Basis(smoothed_quat)

## Sets synthetic inputs from high-level AI decision routines
func set_synthetic_inputs(move_vec: Vector2, fire: bool) -> void:
	_virtual_move_vector = move_vec
	_virtual_fire_pressed = fire

# --- UNIFIED INPUT PROVIDER CONTRACT ---

## Returns virtual analog movement vector
func get_movement_vector() -> Vector2:
	return _virtual_move_vector

## Returns true if virtual action button is pressed
func is_action_pressed(action_name: StringName) -> bool:
	if action_name == &"fire":
		return _virtual_fire_pressed
	return false
```

---

