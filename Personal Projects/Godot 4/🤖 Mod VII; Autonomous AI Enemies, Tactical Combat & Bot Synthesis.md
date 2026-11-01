# 🤖 Module VII: Autonomous AI Enemies, Tactical Combat & Bot Synthesis

> [!info] 📖 Lexicon & First Principles: Artificial Intelligence Architecture
> Transitioning from deterministic script branches (`if sees_player: attack()`) to authentic game AI requires establishing formal computational models for decision-making, spatial reasoning, and simulated motor control:
> - ⤷ **Agent**: An autonomous entity that perceives its environment through simulated sensors, processes that perceptual data against internal goals, and issues motor commands through physical or kinematic actuators.
> - ⤷ **Blackboard**: A centralised, decoupled memory dictionary shared across AI decision nodes and sensory systems. Sensors write world knowledge to the Blackboard; decision nodes read from it without knowing which sensor gathered the data.
> - ⤷ **Goal-Directed Behaviour**: An AI paradigm where an agent does not execute rigid hardcoded sequences, but instead selects actions dynamically to transition the current world state into a desired goal state.
> - ⤷ **Context Steering**: A vector-field movement paradigm that discretises continuous radial space into directional slots of **Interest** and **Danger**, allowing an agent to balance pathfinding, obstacle avoidance, and tactical positioning simultaneously without path-thrashing.
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
| **Modularity & Reuse** | Low (State transitions tightly coupled) | ==Extreme (Tasks are drag-and-drop nodes)== | High (Actions evaluated in isolation) | Extreme (Actions declare pre/post conditions) |
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

- ⤷ **Sequence (`->`)**: Executes children in sequence. If a child returns `FAILURE`, the Sequence halts and returns `FAILURE`. It returns `SUCCESS` ==only if every single child succeeds== (logical AND).
- ⤷ **Selector (`?`)**: Executes children in sequence. If a child returns `SUCCESS`, the Selector halts and returns `SUCCESS`. It returns `FAILURE` ==only if every single child fails== (logical OR / Fallback).
- ⤷ **Decorator**: Wraps a single child node to alter its result:
  - `Inverter`: Flips `SUCCESS` $\leftrightarrow$ `FAILURE` (`RUNNING` is untouched).
  - `Repeater`: Re-executes the child $N$ times or infinitely.
  - `TimeLimit`: Forces child `FAILURE` if it remains `RUNNING` past $T$ seconds.

```gdscript
# ==============================================================================
# PRODUCTION BEHAVIOUR TREE & BLACKBOARD ENGINE (Godot 4)
# ==============================================================================
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

# --- COMPOSITE: SEQUENCE (Logical AND) ---
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

# --- COMPOSITE: SELECTOR (Logical OR / Fallback) ---
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

```gdscript
# ==============================================================================
# MULTI-MODAL SENSORY PERCEPTION & SIGHT SENSOR (Godot 4)
# ==============================================================================
class_name PerceptionSensor3D
extends Node3D

signal target_acquired(target: Node3D)
signal target_lost(last_known_pos: Vector3)

@export_group("Vision Parameters")
@export var sight_radius: float = 24.0
@export var fov_degrees: float = 110.0
@export_flags_3d_physics var occlusion_mask: int = 1 # Static world geometry
@export var awareness_fill_time: float = 0.65 # Seconds to full alert at point-blank

var current_target: Node3D = null
var awareness_level: float = 0.0 # 0.0 to 1.0
var last_known_position: Vector3 = Vector3.ZERO
var is_alert: bool = false

func evaluate_vision(candidate: Node3D, delta: float) -> void:
    if candidate == null or not is_instance_valid(candidate):
        _decay_awareness(delta)
        return

    var eye_pos: Vector3 = global_position
    var to_target: Vector3 = candidate.global_position - eye_pos
    var dist_sq: float = to_target.length_squared()
    
    # 1. Distance check
    if dist_sq > (sight_radius * sight_radius):
        _decay_awareness(delta)
        return

    var dist: float = sqrt(dist_sq)
    var dir_to_target: Vector3 = to_target / dist
    var eye_forward: Vector3 = -global_transform.basis.z.normalized()

    # 2. Field of View Cone check
    var half_angle_rad: float = deg_to_rad(fov_degrees * 0.5)
    if eye_forward.dot(dir_to_target) < cos(half_angle_rad):
        _decay_awareness(delta)
        return

    # 3. Multi-Ray Occlusion Check
    if not _has_line_of_sight(candidate):
        _decay_awareness(delta)
        return

    # 4. Awareness accumulation
    var proximity_mult: float = 1.0 - (dist / sight_radius)
    var fill_speed: float = (1.0 / maxf(0.01, awareness_fill_time)) * (1.0 + proximity_mult)
    awareness_level = minf(1.0, awareness_level + (fill_speed * delta))

    if awareness_level >= 1.0:
        last_known_position = candidate.global_position
        if not is_alert:
            is_alert = true
            current_target = candidate
            target_acquired.emit(candidate)

func _has_line_of_sight(target: Node3D) -> bool:
    var space_state: PhysicsDirectSpaceState3D = get_world_3d().direct_space_state
    var eye_pos: Vector3 = global_position
    
    # Check Head, Center, and Base coordinates
    var sample_points: Array[Vector3] = [
        target.global_position + Vector3(0, 1.6, 0), # Head
        target.global_position + Vector3(0, 0.9, 0), # Chest
        target.global_position + Vector3(0, 0.2, 0)  # Feet
    ]
    
    for point in sample_points:
        var query: PhysicsRayQueryParameters3D = PhysicsRayQueryParameters3D.create(
            eye_pos,
            point,
            occlusion_mask
        )
        var hit: Dictionary = space_state.intersect_ray(query)
        if hit.is_empty():
            return true # Direct unblocked ray detected!
            
    return false

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

**Context Steering** eliminates this entirely. We discretise continuous planar motion into $N$ fixed radial slots (typically $N = 8$ or $N = 16$). Every frame, the agent computes two independent radial arrays:
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

```gdscript
# ==============================================================================
# PRODUCTION 8-SLOT CONTEXT STEERING CONTROLLER (3D Planar)
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
    
    # Initialize 8 radial directions in horizontal XZ plane
    for i in range(NUM_SLOTS):
        var angle: float = (float(i) / float(NUM_SLOTS)) * TAU
        _directions[i] = Vector3(sin(angle), 0.0, cos(angle)).normalized()

func clear() -> void:
    for i in range(NUM_SLOTS):
        _interest[i] = 0.0
        _danger[i] = 0.0

## Adds interest towards a target direction (e.g. seeking player)
func add_interest(target_dir: Vector3, weight: float = 1.0) -> void:
    var dir_flat: Vector3 = Vector3(target_dir.x, 0.0, target_dir.z).normalized()
    for i in range(NUM_SLOTS):
        var dot: float = _directions[i].dot(dir_flat)
        if dot > 0.0:
            _interest[i] = maxf(_interest[i], dot * weight)

## Adds danger in directions blocked by obstacles via raycasts
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
            # Danger scales inversely with proximity: closer obstacle = higher danger
            var factor: float = 1.0 - (dist / ray_dist)
            _danger[i] = maxf(_danger[i], factor)

## Computes the composite unblocked steering vector
func compute_resolved_direction() -> Vector3:
    var final_vector: Vector3 = Vector3.ZERO
    
    for i in range(NUM_SLOTS):
        # Subtract danger from interest; clamp to zero
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

```gdscript
# ==============================================================================
# GLOBAL COMBAT DIRECTOR & TOKEN THROTTLER
# ==============================================================================
class_name CombatDirector
extends Node

@export var max_melee_tokens: int = 2
@export var max_ranged_tokens: int = 1

var _active_melee_tokens: int = 0
var _active_ranged_tokens: int = 0

## Requests permission to execute an offensive attack
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

## Releases the token back to the pool once attack wind-down finishes
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

```gdscript
# ==============================================================================
# HUMAN-EMULATING BOT BRAIN (InputProvider Implementation)
# ==============================================================================
class_name BotBrainInputProvider
extends Node

@export_group("Humanisation Constraints")
@export var reaction_delay_ms: int = 220 # Human visual latency window
@export var aim_stiffness: float = 14.0 # Angular tracking speed
@export var aim_damping: float = 1.8 # Critically damped tracking

var _virtual_move_vector: Vector2 = Vector2.ZERO
var _virtual_fire_pressed: bool = false
var _virtual_aim_quaternion: Quaternion = Quaternion.IDENTITY

# Spatial ring buffer storing delayed perceptual target transforms
var _history_buffer: Array[Dictionary] = []

func record_target_snapshot(target_pos: Vector3) -> void:
    var now: int = Time.get_ticks_msec()
    _history_buffer.append({"time": now, "pos": target_pos})
    
    # Prune elements older than 1 second
    while not _history_buffer.is_empty() and (_history_buffer.front()["time"] < now - 1000):
        _history_buffer.pop_front()

## Retrieves target position from the delayed past (simulating human visual lag)
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
    
    # Frame-rate independent exponential interpolation matching human tracking
    var decay: float = 1.0 - exp(-aim_stiffness * delta)
    var smoothed_quat: Quaternion = current_quat.slerp(target_quat, decay)
    
    return Basis(smoothed_quat)

# --- UNIFIED INPUT INTERFACE CONTRACT ---
func get_movement_vector() -> Vector2:
    return _virtual_move_vector

func is_action_pressed(action_name: StringName) -> bool:
    if action_name == &"fire":
        return _virtual_fire_pressed
    return false
```

---

## ✍️ Linguistic & Grammatical Refinement

> [!info] 📖 Constructive Analysis of Your Follow-Up Directive
> Let us review the syntax of your command so you may continually refine your command of English:
> 1. **Redundant Adverbial Compounds & Run-On Structure**:
>    - ⤷ *Original*: `"be as extensive, informational as possible, let's proceed with  module 7"`
>    - ↳ **Correction**: 
>      - The adjective *informational* designates "providing metadata or descriptive context" (e.g., an *informational brochure*); the precise scholarly adjective for content conveying deep instructional depth is **informative** or **exhaustive**.
>      - Joining the initial descriptive directive with the subsequent imperative via a bare comma creates a *comma splice*. Separate them with a full stop.
>      - Note the accidental double space between `"with"` and `"module 7"`, as well as the uncapitalised proper designator *Module 7*.
>    - ↳ **Elevated Revision**: *"Please be as exhaustive and informative as possible. Let us proceed with Module 7."*