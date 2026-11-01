# 🦴 Module VI: Skeletal Rigging, Procedural Animation & Inverse Kinematics (IK)

> [!info] 📖 Lexicon & First Principles: Skeletal & Kinematic Terminology
> Transitioning from keyframed animation playback to dynamic procedural motion requires formalizing the anatomical and mathematical constructs of real-time skeletal systems:
> - ⤷ **Armature (`Skeleton2D` / `Skeleton3D`)**: A hierarchical tree of virtual bones where each bone's global transform is the cumulative matrix product of its local pose and all ancestor bone poses up to the root.
> - ⤷ **Rest Pose vs. Pose Override**: The **Rest Pose** is the bind configuration of a skeleton (e.g., T-Pose or A-Pose) used to weight mesh vertices in model space. A **Pose Override** is a runtime matrix offset applied to a bone on a discrete frame by animations or procedural scripts.
> - ⤷ **Forward Kinematics (FK)**: The standard animation paradigm where parent joint angles are rotated explicitly ($\theta_1, \theta_2, \dots, \theta_n$), and the resulting world coordinate of the extremity is computed forward down the chain.
> - ⤷ **Inverse Kinematics (IK)**: The mathematical inverse of FK—defining the desired spatial coordinate of the terminal extremity first and solving backward for the interior joint rotations required to place the extremity at that coordinate.
> - ⤷ **End-Effector**: The terminal bone or tip of a kinematic chain (e.g., hand, foot ankle, tail tip, arachnid tarsus) whose spatial target drives the IK solver.
> - ⤷ **Pole Target (Swivel Constraint)**: A spatial reference coordinate in 3D space that defines the plane of articulation for a hinge joint (such as an elbow or knee), preventing unnatural anatomical hyperextension or erratic axis flipping.

---

## 6.1 Skeletal Hierarchy & Coordinate Spaces in `Skeleton3D`

A primary stumbling block in procedural animation is confusing the three distinct coordinate spaces operating simultaneously inside a `Skeleton3D` node. 

Unlike standard `Node3D` hierarchies, bones are **not** individual nodes in the Scene Tree. They are indexed entries within a contiguous C++ internal buffer managed by the `Skeleton3D` server instance.

```
                      THE THREE SKELETAL COORDINATE SPACES
=================================================================================
 [ 1. Bone Local / Rest Space ]  Transform relative to the immediate parent bone
               │
               ▼  * parent_global_pose
 [ 2. Skeleton Global Pose ]     Transform relative to the Skeleton3D origin (0,0,0)
               │
               ▼  * skeleton_node.global_transform
 [ 3. World Global Space ]       Absolute coordinate inside the 3D physics level
```

### The Matrix Concatenation Law of Skeletal Chains

Given a kinematic chain of bones indexed $0$ (root) through $n$ (end-effector), let $\mathbf{L}_i$ denote the local pose matrix of bone $i$ relative to its parent $i-1$. The **Skeleton Global Pose** $\mathbf{G}_i$ of bone $i$ is the ordered product:

$$\mathbf{G}_i = \mathbf{G}_{i-1} \cdot \mathbf{L}_i = \prod_{k=0}^{i} \mathbf{L}_k$$

Consequently, to convert a world-space raycast hit point $\vec{P}_{\text{world}}$ into a `Skeleton3D` global pose coordinate $\vec{P}_{\text{skel}}$:

$$\vec{P}_{\text{skel}} = \mathbf{T}_{\text{skeleton}}^{-1} \cdot \vec{P}_{\text{world}}$$

---

### Step-by-Step Architectural Construction: Skeletal Coordinate Utilities

Let us construct our skeletal coordinate transformation library chunk by chunk, examining the low-level mechanics of matrix inversion, parent indexing, and global-to-local space conversions.

#### Chunk 1.1: World Space to Skeleton Pose Space Conversion

```gdscript
class_name SkeletalMath
extends RefCounted

static func world_to_skeleton_pose(skeleton: Skeleton3D, world_transform: Transform3D) -> Transform3D:
	return skeleton.global_transform.affine_inverse() * world_transform

static func skeleton_pose_to_world(skeleton: Skeleton3D, skel_pose: Transform3D) -> Transform3D:
	return skeleton.global_transform * skel_pose
```

- **`skeleton.global_transform.affine_inverse()`**: Inverts the affine transformation matrix of the `Skeleton3D` node (undoing its world translation, rotation, and scale). Multiplying this inverted matrix by a world-space transform maps coordinates into the skeleton's root coordinate system ($[0,0,0]$ at the skeleton origin).
- **`skeleton.global_transform * skel_pose`**: The forward projection. Multiplies the skeleton's global transform matrix by a skeleton-space pose, placing it into absolute 3D world physics space.

#### Chunk 1.2: Skeleton Global Pose to Parent-Relative Local Space

```gdscript
static func global_pose_to_local_pose(skeleton: Skeleton3D, bone_idx: int, skel_global_pose: Transform3D) -> Transform3D:
	var parent_idx: int = skeleton.get_bone_parent(bone_idx)
	if parent_idx == -1:
		return skel_global_pose
	var parent_global_pose: Transform3D = skeleton.get_bone_global_pose(parent_idx)
	return parent_global_pose.affine_inverse() * skel_global_pose
```

- **`skeleton.get_bone_parent(bone_idx)`**: Queries the internal C++ bone hierarchy buffer for the parent index. If it returns `-1`, the bone is an absolute root bone with no parent, meaning its local pose is already identical to its skeleton global pose.
- **`parent_global_pose.affine_inverse() * skel_global_pose`**: Solves the equation $\mathbf{G}_{\text{child}} = \mathbf{G}_{\text{parent}} \cdot \mathbf{L}_{\text{child}}$ for $\mathbf{L}_{\text{child}}$. Multiplying the child's global pose by the parent's inverted global pose yields the exact parent-relative local pose matrix required by `skeleton.set_bone_pose()`.

---

#### The Fully Assembled `SkeletalMath.gd`

```gdscript
# ==============================================================================
# SCRIPT: SkeletalMath.gd
# Architecture: Skeletal Space Transformations & Matrix Conversions
# ==============================================================================
class_name SkeletalMath
extends RefCounted

## Converts an absolute 3D world-space Transform3D into Skeleton-relative Global Pose space
static func world_to_skeleton_pose(skeleton: Skeleton3D, world_transform: Transform3D) -> Transform3D:
	return skeleton.global_transform.affine_inverse() * world_transform

## Converts a Skeleton-relative Global Pose Transform3D into absolute 3D world coordinates
static func skeleton_pose_to_world(skeleton: Skeleton3D, skel_pose: Transform3D) -> Transform3D:
	return skeleton.global_transform * skel_pose

## Converts a Skeleton-space Global Pose into a bone's parent-relative Local Pose matrix
static func global_pose_to_local_pose(skeleton: Skeleton3D, bone_idx: int, skel_global_pose: Transform3D) -> Transform3D:
	var parent_idx: int = skeleton.get_bone_parent(bone_idx)
	if parent_idx == -1:
		return skel_global_pose
	var parent_global_pose: Transform3D = skeleton.get_bone_global_pose(parent_idx)
	return parent_global_pose.affine_inverse() * skel_global_pose
```

---

## 6.2 `AnimationTree`, Blend Spaces & Root Motion Extraction

Keyframed animations stored inside an `AnimationPlayer` are static, pre-baked tracks. To blend smoothly between directional movement states, layer upper-body actions over running legs, and extract physical velocity, we construct an **`AnimationTree`** powered by an `AnimationNodeBlendTree`.

```
                        HYBRID LOCOMOTION & COMBAT BLEND TREE
=================================================================================
 [BlendSpace2D: Locomotion] ──(Base Pose)──┐
  (Idle / Walk / Strafe)                   ▼
                                    [ OneShot: Attack ] ──► [ TimeScale ] ──► OUTPUT
 [Animation: SwordSlash]    ──(Filtered)───┘
                              (Upper Body Bones ONLY)
```

### 1. 2D Delaunay Blend Spaces (`AnimationNodeBlendSpace2D`)
A `BlendSpace2D` maps a normalized 2D local velocity vector $(v_x, v_z) \in [-1, 1]^2$ onto a Cartesian plane populated with animation clips:
- $(0, 0) \rightarrow \text{Idle}$
- $(0, 1) \rightarrow \text{Run\_Forward}$, $(0, -1) \rightarrow \text{Run\_Backward}$
- $(-1, 0) \rightarrow \text{Strafe\_Left}$, $(1, 0) \rightarrow \text{Strafe\_Right}$

Using Delaunay triangulation, the engine computes **Barycentric Coordinates** $(\lambda_1, \lambda_2, \lambda_3)$ where $\lambda_1 + \lambda_2 + \lambda_3 = 1.0$ for the triangle enclosing the input point, blending the three nearest animations proportionally without sudden snapping.

### 2. Per-Bone Hierarchical Filtering (`OneShot` & `Blend2`)
If a player swings a weapon while sprinting, playing the full-body attack animation overwrites leg bone positions, causing the character's feet to slide rigidly across the floor ("ice-skating").
- ⤷ **The Solution**: Enable **Filter** on the `AnimationNodeOneShot` or `AnimationNodeBlend2` node and check only the `Spine`, `Chest`, `Shoulders`, `Arms`, and `Head` bones. The lower-body bones (`Hips`, `Thighs`, `Calves`, `Feet`) continue executing the `BlendSpace2D` sprint cycle uninterrupted.

### 3. Root Motion Extraction (`get_root_motion_position()`)
In precision melee combat, driving character velocity via flat mathematical formulas during an attack lunge causes feet to slide out of sync with the ground.
- ⤷ **Root Motion Mechanism**: The animator animates the actual `Root` bone moving forward in 3D space during the lunge. At runtime, Godot strips that translation off the visual skeleton and hands the frame-delta displacement $\Delta \vec{p}_{\text{root}}$ and rotation $\Delta \mathbf{q}_{\text{root}}$ to your `CharacterBody3D` script:

$$\vec{v}_{\text{kinematic}} = \frac{\mathbf{B}_{\text{actor}} \cdot \Delta \vec{p}_{\text{root}}}{\Delta t}$$

---

### Step-by-Step Architectural Construction: Animation Tree & Root Motion Controller

Let us deconstruct our locomotion and root motion pipeline chunk by chunk.

#### Chunk 2.1: Node Bindings & Parameter Path Interning

```gdscript
class_name LocomotionAnimController
extends Node

@export var actor: CharacterBody3D
@export var animation_tree: AnimationTree
@export var blend_smoothing_speed: float = 12.0

const PARAM_LOCOMOTION_BLEND: StringName = &"parameters/Locomotion/blend_position"
const PARAM_ATTACK_REQUEST: StringName = &"parameters/AttackOneShot/request"
const PARAM_TIMESCALE: StringName = &"parameters/TimeScale/scale"

var _current_blend_pos: Vector2 = Vector2.ZERO

func _ready() -> void:
	assert(animation_tree != null, "LocomotionAnimController: AnimationTree unassigned.")
	animation_tree.active = true
```

- **`StringName (&"...")`**: Interns the node path strings within Godot's StringName table. Setting properties inside an `AnimationTree` every single frame via dynamic strings causes string hash lookups; interning them converts parameter lookups into fast pointer comparisons.
- **`animation_tree.active = true`**: Forces the animation processor to begin ticking within the engine frame loop.

#### Chunk 2.2: Local Velocity Transformation & Exponential Blend Smoothing

```gdscript
func update_locomotion_blend(world_velocity: Vector3, max_speed: float, delta: float) -> void:
	var local_vel: Vector3 = actor.global_transform.basis.orthonormalized().transposed() * world_velocity
	var target_blend: Vector2 = Vector2(local_vel.x, -local_vel.z) / maxf(0.001, max_speed)
	target_blend = target_blend.limit_length(1.0)
	
	var weight: float = 1.0 - exp(-blend_smoothing_speed * delta)
	_current_blend_pos = _current_blend_pos.lerp(target_blend, weight)
	
	animation_tree.set(PARAM_LOCOMOTION_BLEND, _current_blend_pos)
```

- **`actor.global_transform.basis.orthonormalized().transposed() * world_velocity`**: Transforms the absolute world velocity vector into the actor's local coordinate frame using transpose inversion (Module IV). In this local frame, $+X$ is local right, and $-Z$ is local forward.
- **`Vector2(local_vel.x, -local_vel.z) / max_speed`**: Normalizes local speeds into the range $[-1.0, 1.0]$ required by the 2D blend space. Note that local forward ($-Z$) maps to positive $+Y$ on the blend plane.
- **`1.0 - exp(-blend_smoothing_speed * delta)`**: Evaluates our exact frame-rate independent exponential decay formula. The blend coordinate glides toward target inputs smoothly without frame-rate variance.

#### Chunk 2.3: Root Motion Extraction & Kinematic Velocity Synthesis

```gdscript
func extract_root_motion_velocity(delta: float) -> Vector3:
	if delta <= 0.00001:
		return Vector3.ZERO
		
	var delta_rot: Quaternion = animation_tree.get_root_motion_rotation()
	var delta_pos: Vector3 = animation_tree.get_root_motion_position()
	
	actor.quaternion = (actor.quaternion * delta_rot).normalized()
	
	var world_displacement: Vector3 = actor.global_transform.basis * delta_pos
	return world_displacement / delta
```

- **`animation_tree.get_root_motion_rotation()` & `get_root_motion_position()`**: Reads the discrete displacement baked into the skeleton's root bone over the elapsed frame.
- **`actor.quaternion = (actor.quaternion * delta_rot).normalized()`**: Applies the root bone's rotational delta directly to the actor's global orientation, allowing animations to turn characters organically.
- **`actor.global_transform.basis * delta_pos`**: Converts root translation from local space into world displacement. Dividing by `delta` yields the exact kinematic velocity vector to pass into `CharacterBody3D.velocity`.

---

#### The Fully Assembled `LocomotionAnimController.gd`

```gdscript
# ==============================================================================
# SCRIPT: LocomotionAnimController.gd
# Architecture: AnimationTree Blending & Root Motion Extraction
# ==============================================================================
class_name LocomotionAnimController
extends Node

@export_group("Dependencies")
## Controlling kinematic character body
@export var actor: CharacterBody3D
## Underlying animation blend tree
@export var animation_tree: AnimationTree

@export_group("Tuning")
## Frame-rate independent blend smoothing factor
@export var blend_smoothing_speed: float = 12.0

# Interned StringNames for zero-allocation property access
const PARAM_LOCOMOTION_BLEND: StringName = &"parameters/Locomotion/blend_position"
const PARAM_ATTACK_REQUEST: StringName = &"parameters/AttackOneShot/request"
const PARAM_TIMESCALE: StringName = &"parameters/TimeScale/scale"

var _current_blend_pos: Vector2 = Vector2.ZERO

func _ready() -> void:
	assert(animation_tree != null, "LocomotionAnimController: AnimationTree unassigned.")
	animation_tree.active = true

## Updates the 2D locomotion blend space coordinates from entity velocity
func update_locomotion_blend(world_velocity: Vector3, max_speed: float, delta: float) -> void:
	# Convert world velocity into actor's local coordinate frame
	var local_vel: Vector3 = actor.global_transform.basis.orthonormalized().transposed() * world_velocity
	var target_blend: Vector2 = Vector2(local_vel.x, -local_vel.z) / maxf(0.001, max_speed)
	target_blend = target_blend.limit_length(1.0)
	
	# Frame-rate independent exponential smoothing
	var weight: float = 1.0 - exp(-blend_smoothing_speed * delta)
	_current_blend_pos = _current_blend_pos.lerp(target_blend, weight)
	
	animation_tree.set(PARAM_LOCOMOTION_BLEND, _current_blend_pos)

## Fires a transient upper-body filtered action animation
func trigger_upper_body_action() -> void:
	animation_tree.set(PARAM_ATTACK_REQUEST, AnimationNodeOneShot.ONE_SHOT_REQUEST_FIRE)

## Extracts baked root motion deltas to drive kinematic velocity
func extract_root_motion_velocity(delta: float) -> Vector3:
	if delta <= 0.00001:
		return Vector3.ZERO
		
	var delta_rot: Quaternion = animation_tree.get_root_motion_rotation()
	var delta_pos: Vector3 = animation_tree.get_root_motion_position()
	
	# Apply root angular delta
	actor.quaternion = (actor.quaternion * delta_rot).normalized()
	
	# Convert local linear displacement into world-space velocity
	var world_displacement: Vector3 = actor.global_transform.basis * delta_pos
	return world_displacement / delta
```

---

## 6.3 Inverse Kinematics (IK): Mathematical Proofs & Solvers

When characters walk over uneven terrain, climb staircases, or aim weapons at dynamic targets, pre-baked animations cannot anticipate world contact points. We solve this using **Inverse Kinematics**.

```
FORWARD KINEMATICS (FK):              INVERSE KINEMATICS (IK):
Given: Joint angles (θ1, θ2)          Given: Target coordinate T(x, y, z)
Find:  End-effector position E        Find:  Joint angles (θ1, θ2) so E == T

      Root (O)                              Root (O)
         \ θ1                                  \ θ1 = ?
          o Joint (J)                           o Joint (J)
           \ θ2                                  \ θ2 = ?
            ▼                                     ▼
        End-Effector (E)                      [Target T]
```

| IK Solver Class | Algorithmic Mechanism | Complexity per Solve | Joint Constraints | Ideal Production Use Case |
| :--- | :--- | :--- | :--- | :--- |
| **Analytical Two-Bone IK** | Closed-form trigonometry (**Law of Cosines**) | $\mathcal{O}(1)$ — Exact in 1 pass | Swivel Pole Target | Humanoid arms (elbows) & legs (knees) |
| **CCD (Cyclic Coordinate Descent)** | Iterative angular minimization from tip to root | $\mathcal{O}(k \cdot N)$ | Per-axis Euler clamps | Mechanical cranes, robotic turrets, tails |
| **FABRIK (Forward & Backward Reaching)** | Iterative geometric line projection in Euclidean space | $\mathcal{O}(k \cdot N)$ — Ultra-fast | Conical angle limits | Multi-jointed spines, tentacles, ropes, chains |

---

### 1. Analytical Two-Bone IK (The Law of Cosines Derivation)

For two-segment limbs (e.g., upper arm $L_1$ and forearm $L_2$, or thigh $L_1$ and calf $L_2$), iterative solvers are unnecessary. The three joints (Root $\vec{R}$, Mid-Joint $\vec{J}$, and Target $\vec{T}$) form a triangle on an articulation plane defined by the **Pole Target** $\vec{P}$, which we solve in $\mathcal{O}(1)$ time via the **Law of Cosines**.

```
                   THE TWO-BONE IK TRIANGLE GEOMETRY
=================================================================================
                             Mid-Joint (J)
                              ╱ ╲  Interior Knee/Elbow Angle (β)
                             ╱ β ╲
        Upper Limb (L1)     ╱     ╲     Lower Limb (L2)
                           ╱       ╲
                          ╱ α       ╲
               Root (R)  o───────────o  Target (T)
                              D = ||T - R||
```

#### Step-by-Step Mathematical Derivation:
Let $L_1 = \|\vec{J} - \vec{R}\|$, $L_2 = \|\vec{E} - \vec{J}\|$, and $D = \|\vec{T} - \vec{R}\|$.

1. **Triangle Inequality Clamping**: To prevent $\arccos(x)$ domain errors ($|x| > 1.0$), clamp target distance $D$ using epsilon $\epsilon = 10^{-4}$:
   $$D_{\text{clamped}} = \text{clamp}\left(\|\vec{T} - \vec{R}\|, \, |L_1 - L_2| + \epsilon, \, (L_1 + L_2) - \epsilon\right)$$
2. **Solving Root-to-Mid Angle ($\alpha$)**: By the Law of Cosines ($L_2^2 = L_1^2 + D^2 - 2 L_1 D \cos\alpha$):
   $$\alpha = \arccos\left(\text{clamp}\left(\frac{L_1^2 + D^2 - L_2^2}{2 L_1 D}, \, -1.0, \, 1.0\right)\right)$$
3. **Constructing the Articulation Plane via Pole Target ($\vec{P}$)**:
   $$\hat{u}_{\text{target}} = \frac{\vec{T} - \vec{R}}{\|\vec{T} - \vec{R}\|} \qquad \hat{u}_{\text{pole}} = \frac{\vec{P} - \vec{R}}{\|\vec{P} - \vec{R}\|}$$
   $$\hat{u}_{\text{hinge}} = \frac{\hat{u}_{\text{target}} \times \hat{u}_{\text{pole}}}{\|\hat{u}_{\text{target}} \times \hat{u}_{\text{pole}}\|}$$
   Rotating $\hat{u}_{\text{target}}$ around $\hat{u}_{\text{hinge}}$ by angle $-\alpha$ and scaling by $L_1$ produces the exact 3D position of the mid-joint $\vec{J}$!

---

### Step-by-Step Architectural Construction: Two-Bone IK via `SkeletonModifier3D`

Let us construct a custom `SkeletonModifier3D` implementing analytical two-bone IK chunk by chunk.

#### Chunk 3.1: Bone Index Resolution & Guard Checks

```gdscript
@tool
class_name AnalyticalTwoBoneIK3D
extends SkeletonModifier3D

@export var root_bone_name: StringName
@export var mid_bone_name: StringName
@export var tip_bone_name: StringName

@export var target_node: Node3D
@export var pole_node: Node3D

var _root_idx: int = -1
var _mid_idx: int = -1
var _tip_idx: int = -1

func _validate_bone_indices(skeleton: Skeleton3D) -> bool:
	_root_idx = skeleton.find_bone(root_bone_name)
	_mid_idx = skeleton.find_bone(mid_bone_name)
	_tip_idx = skeleton.find_bone(tip_bone_name)
	return _root_idx != -1 and _mid_idx != -1 and _tip_idx != -1
```

- **`@tool`**: Allows the modifier to execute inside the Godot Editor viewport, providing real-time visual feedback when gizmos are moved.
- **`extends SkeletonModifier3D`**: Uses Godot 4.3+'s replacement for deprecated IK stacks. `SkeletonModifier3D` integrates directly into the skeleton evaluation pipeline before mesh skinning occurs.
- **`skeleton.find_bone(...)`**: Caches internal bone indices once, eliminating string comparisons during frame evaluation.

#### Chunk 3.2: Coordinate Mapping & Law of Cosines Evaluation

```gdscript
func _process_modification() -> void:
	var skeleton: Skeleton3D = get_skeleton()
	if skeleton == null or target_node == null or pole_node == null:
		return
	if not _validate_bone_indices(skeleton):
		return

	var root_pose: Transform3D = skeleton.get_bone_global_pose(_root_idx)
	var mid_pose: Transform3D = skeleton.get_bone_global_pose(_mid_idx)
	var tip_pose: Transform3D = skeleton.get_bone_global_pose(_tip_idx)

	var p_root: Vector3 = root_pose.origin
	var p_mid: Vector3 = mid_pose.origin
	var p_tip: Vector3 = tip_pose.origin

	var inv_skel: Transform3D = skeleton.global_transform.affine_inverse()
	var p_target: Vector3 = inv_skel * target_node.global_position
	var p_pole: Vector3 = inv_skel * pole_node.global_position

	var l1: float = p_root.distance_to(p_mid)
	var l2: float = p_mid.distance_to(p_tip)
	var epsilon: float = 0.0001
	
	var to_target: Vector3 = p_target - p_root
	var raw_dist: float = to_target.length()
	var d: float = clampf(raw_dist, absf(l1 - l2) + epsilon, (l1 + l2) - epsilon)

	var cos_alpha: float = clampf(((l1 * l1) + (d * d) - (l2 * l2)) / (2.0 * l1 * d), -1.0, 1.0)
	var alpha: float = acos(cos_alpha)
```

- **`inv_skel * target_node.global_position`**: Converts world coordinates of the target and pole nodes into the skeleton's global pose space.
- **`var d = clampf(raw_dist, ...)`**: Clamps target distance to prevent triangle collapse or unreachable targets, protecting the division from producing `NaN` in `acos()`.
- **`acos(cos_alpha)`**: Computes the interior shoulder/hip pitch angle $\alpha$ using the Law of Cosines.

#### Chunk 3.3: Articulation Basis Construction & Pose Assignment

```gdscript
	var dir_target: Vector3 = to_target / maxf(epsilon, raw_dist)
	var dir_pole: Vector3 = (p_pole - p_root).normalized()
	
	var hinge_normal: Vector3 = dir_target.cross(dir_pole)
	if hinge_normal.length_squared() < epsilon:
		hinge_normal = dir_target.cross(Vector3.UP)
	hinge_normal = hinge_normal.normalized()

	var mid_dir: Vector3 = dir_target.rotated(hinge_normal, -alpha).normalized()
	var solved_mid_pos: Vector3 = p_root + (mid_dir * l1)
	var solved_tip_pos: Vector3 = p_root + (dir_target * d)

	var root_y: Vector3 = (solved_mid_pos - p_root).normalized()
	var root_x: Vector3 = hinge_normal
	var root_z: Vector3 = root_x.cross(root_y).normalized()
	root_pose.basis = Basis(root_x, root_y, root_z).orthonormalized()

	var mid_y: Vector3 = (solved_tip_pos - solved_mid_pos).normalized()
	var mid_x: Vector3 = hinge_normal
	var mid_z: Vector3 = mid_x.cross(mid_y).normalized()
	mid_pose.origin = solved_mid_pos
	mid_pose.basis = Basis(mid_x, mid_y, mid_z).orthonormalized()

	skeleton.set_bone_global_pose(_root_idx, root_pose)
	skeleton.set_bone_global_pose(_mid_idx, mid_pose)
```

- **`dir_target.cross(dir_pole)`**: Generates the hinge normal perpendicular to the target-pole triangle, establishing the plane of articulation.
- **`dir_target.rotated(hinge_normal, -alpha)`**: Rotates the upper limb vector on the articulation plane by angle $-\alpha$, finding the exact 3D position of the mid joint (`solved_mid_pos`).
- **`Basis(root_x, root_y, root_z).orthonormalized()`**: Builds orthonormal orientation matrices aligning the upper and lower bone segments along their displacement vectors, then writes the updated transforms directly into `Skeleton3D`.

---

#### The Fully Assembled `AnalyticalTwoBoneIK3D.gd`

```gdscript
# ==============================================================================
# SCRIPT: AnalyticalTwoBoneIK3D.gd
# Architecture: Closed-Form Law of Cosines SkeletonModifier3D
# ==============================================================================
@tool
class_name AnalyticalTwoBoneIK3D
extends SkeletonModifier3D

@export_group("Bones")
## Root socket bone name (e.g. UpperArm, Thigh)
@export var root_bone_name: StringName
## Articulating middle joint name (e.g. Forearm, Calf)
@export var mid_bone_name: StringName
## Terminal extremity bone name (e.g. Hand, Foot)
@export var tip_bone_name: StringName

@export_group("Spatial Targets")
## 3D target node defining the desired end-effector position
@export var target_node: Node3D
## 3D guide node defining the elbow/knee bending direction
@export var pole_node: Node3D

var _root_idx: int = -1
var _mid_idx: int = -1
var _tip_idx: int = -1

func _validate_bone_indices(skeleton: Skeleton3D) -> bool:
	_root_idx = skeleton.find_bone(root_bone_name)
	_mid_idx = skeleton.find_bone(mid_bone_name)
	_tip_idx = skeleton.find_bone(tip_bone_name)
	return _root_idx != -1 and _mid_idx != -1 and _tip_idx != -1

## Virtual callback executed by Skeleton3D during pose calculation
func _process_modification() -> void:
	var skeleton: Skeleton3D = get_skeleton()
	if skeleton == null or target_node == null or pole_node == null:
		return
	if not _validate_bone_indices(skeleton):
		return

	# 1. Read existing bone global poses in skeleton space
	var root_pose: Transform3D = skeleton.get_bone_global_pose(_root_idx)
	var mid_pose: Transform3D = skeleton.get_bone_global_pose(_mid_idx)
	var tip_pose: Transform3D = skeleton.get_bone_global_pose(_tip_idx)

	var p_root: Vector3 = root_pose.origin
	var p_mid: Vector3 = mid_pose.origin
	var p_tip: Vector3 = tip_pose.origin

	# 2. Map external 3D target coordinates into skeleton-local space
	var inv_skel: Transform3D = skeleton.global_transform.affine_inverse()
	var p_target: Vector3 = inv_skel * target_node.global_position
	var p_pole: Vector3 = inv_skel * pole_node.global_position

	# 3. Calculate limb segment lengths
	var l1: float = p_root.distance_to(p_mid)
	var l2: float = p_mid.distance_to(p_tip)
	var epsilon: float = 0.0001
	
	var to_target: Vector3 = p_target - p_root
	var raw_dist: float = to_target.length()
	var d: float = clampf(raw_dist, absf(l1 - l2) + epsilon, (l1 + l2) - epsilon)

	# 4. Law of Cosines: Solve interior angle alpha
	var cos_alpha: float = clampf(((l1 * l1) + (d * d) - (l2 * l2)) / (2.0 * l1 * d), -1.0, 1.0)
	var alpha: float = acos(cos_alpha)

	# 5. Construct articulation plane from Pole Target
	var dir_target: Vector3 = to_target / maxf(epsilon, raw_dist)
	var dir_pole: Vector3 = (p_pole - p_root).normalized()
	
	var hinge_normal: Vector3 = dir_target.cross(dir_pole)
	if hinge_normal.length_squared() < epsilon:
		hinge_normal = dir_target.cross(Vector3.UP)
	hinge_normal = hinge_normal.normalized()

	# 6. Compute solved joint positions
	var mid_dir: Vector3 = dir_target.rotated(hinge_normal, -alpha).normalized()
	var solved_mid_pos: Vector3 = p_root + (mid_dir * l1)
	var solved_tip_pos: Vector3 = p_root + (dir_target * d)

	# 7. Construct solved Basis matrices (bone length aligned to +Y axis)
	var root_y: Vector3 = (solved_mid_pos - p_root).normalized()
	var root_x: Vector3 = hinge_normal
	var root_z: Vector3 = root_x.cross(root_y).normalized()
	root_pose.basis = Basis(root_x, root_y, root_z).orthonormalized()

	var mid_y: Vector3 = (solved_tip_pos - solved_mid_pos).normalized()
	var mid_x: Vector3 = hinge_normal
	var mid_z: Vector3 = mid_x.cross(mid_y).normalized()
	mid_pose.origin = solved_mid_pos
	mid_pose.basis = Basis(mid_x, mid_y, mid_z).orthonormalized()

	# 8. Write updated poses into Skeleton3D
	skeleton.set_bone_global_pose(_root_idx, root_pose)
	skeleton.set_bone_global_pose(_mid_idx, mid_pose)
```

---

### 2. Multi-Joint Iterative IK: FABRIK (Forward And Backward Reaching IK)

When a kinematic chain contains three or more segments ($N \ge 3$, such as tentacles, spines, ropes, or robotic tails), analytical trigonometry becomes under-determined. 

**FABRIK** solves the chain using iterative Euclidean distance projections, converging in $3\text{--}8$ iterations without evaluating costly trigonometric operations or matrix inversions.

```
                      THE FABRIK TWO-PASS ITERATION LOOP
=================================================================================
 INITIAL STATE:                 PASS 1: FORWARD REACHING       PASS 2: BACKWARD REACHING
 Root p0 -> p1 -> p2 -> p3      Snap p3' to Target T;          Snap p0'' back to Root Origin;
                                pull p2', p1', p0' toward T    pull p1'', p2'', p3'' outward

   p0 ──► p1 ──► p2 ──► p3        p0'◄── p1'◄── p2'◄── p3'(T)   p0''──► p1''──► p2''──► p3''
                                 (Root pulled off socket!)      (Root locked; Tip at Target!)
```

---

### Step-by-Step Architectural Construction: FABRIK with Conical Constraints

Let us deconstruct the forward and backward reaching passes and conical constraint checks.

#### Chunk 4.1: Reachability Fast-Path

```gdscript
class_name FABRIKSolver3D
extends RefCounted

static func solve_chain(
	points: PackedVector3Array,
	lengths: PackedFloat32Array,
	target_pos: Vector3,
	max_cone_angle_rad: float = PI,
	tolerance: float = 0.005,
	max_iterations: int = 8
) -> void:
	var n: int = points.size() - 1
	if n < 1:
		return

	var root_origin: Vector3 = points[0]
	var total_length: float = 0.0
	for length in lengths:
		total_length += length

	var root_to_target: Vector3 = target_pos - root_origin
	if root_to_target.length() >= total_length:
		var dir: Vector3 = root_to_target.normalized()
		for i in range(n):
			points[i + 1] = points[i] + (dir * lengths[i])
		return
```

- **`PackedVector3Array` & `PackedFloat32Array`**: Passes contiguous memory buffers. Solves points in-place without heap allocations.
- **Unreachable Fast-Path**: If the target distance exceeds the total length of the chain ($\|\vec{T} - \vec{p}_0\| \ge \sum L_i$), the chain cannot reach it. We skip iterative convergence entirely, pointing all bones straight toward the target in a single $\mathcal{O}(N)$ pass.

#### Chunk 4.2: Forward Reaching (Tip $\rightarrow$ Root)

```gdscript
	var tolerance_sq: float = tolerance * tolerance

	for _iter in range(max_iterations):
		if points[n].distance_squared_to(target_pos) <= tolerance_sq:
			break

		# Pass 1: Forward Reaching
		points[n] = target_pos
		for i in range(n - 1, -1, -1):
			var dir_back: Vector3 = (points[i] - points[i + 1]).normalized()
			points[i] = points[i + 1] + (dir_back * lengths[i])
```

- **`points[n].distance_squared_to(target_pos) <= tolerance_sq`**: Early exit check using squared distance. Terminates the solve loop as soon as the tip reaches the target.
- **Forward Pass**: Snaps the end-effector ($\vec{p}_n$) directly to the target. It iterates backward down the chain, pulling each parent bone along the line connecting them to preserve fixed segment lengths ($L_i$). The root bone will temporarily be pulled away from its fixed socket.

#### Chunk 4.3: Backward Reaching with Conical Angle Constraints

```gdscript
		# Pass 2: Backward Reaching
		points[0] = root_origin
		for i in range(n):
			var desired_dir: Vector3 = (points[i + 1] - points[i]).normalized()
			
			# Enforce conical angle constraint relative to the preceding bone
			if i > 0 and max_cone_angle_rad < PI:
				var prev_bone_dir: Vector3 = (points[i] - points[i - 1]).normalized()
				var angle: float = prev_bone_dir.angle_to(desired_dir)
				if angle > max_cone_angle_rad:
					var rot_axis: Vector3 = prev_bone_dir.cross(desired_dir)
					if rot_axis.length_squared() > 0.00001:
						desired_dir = prev_bone_dir.rotated(rot_axis.normalized(), max_cone_angle_rad)

			points[i + 1] = points[i] + (desired_dir * lengths[i])
```

- **Backward Pass**: Snaps the root bone ($\vec{p}_0$) back to its fixed origin socket. It iterates forward, projecting child joints along their segment vectors to restore bone lengths.
- **Conical Constraint**: Measures the deflection angle between the current bone and its predecessor. If the joint bends beyond `max_cone_angle_rad`, it clamps the vector along the cone perimeter via cross-product axis rotation, preventing unnatural joint breaks.

---

#### The Fully Assembled `FABRIKSolver3D.gd`

```gdscript
# ==============================================================================
# SCRIPT: FABRIKSolver3D.gd
# Architecture: Multi-Joint Forward and Backward Reaching IK Solver
# ==============================================================================
class_name FABRIKSolver3D
extends RefCounted

## Solves an N-joint chain in-place toward target_pos while respecting conical joint bounds
static func solve_chain(
	points: PackedVector3Array,
	lengths: PackedFloat32Array,
	target_pos: Vector3,
	max_cone_angle_rad: float = PI,
	tolerance: float = 0.005,
	max_iterations: int = 8
) -> void:
	var n: int = points.size() - 1
	if n < 1:
		return

	var root_origin: Vector3 = points[0]
	var total_length: float = 0.0
	for length in lengths:
		total_length += length

	# 1. Fast-Path: Target exceeds maximum chain reach
	var root_to_target: Vector3 = target_pos - root_origin
	if root_to_target.length() >= total_length:
		var dir: Vector3 = root_to_target.normalized()
		for i in range(n):
			points[i + 1] = points[i] + (dir * lengths[i])
		return

	var tolerance_sq: float = tolerance * tolerance

	# 2. Iterative Forward & Backward Reaching Loop
	for _iter in range(max_iterations):
		if points[n].distance_squared_to(target_pos) <= tolerance_sq:
			break

		# --- STAGE A: Forward Reaching (Tip -> Root) ---
		points[n] = target_pos
		for i in range(n - 1, -1, -1):
			var dir_back: Vector3 = (points[i] - points[i + 1]).normalized()
			points[i] = points[i + 1] + (dir_back * lengths[i])

		# --- STAGE B: Backward Reaching (Root -> Tip) with Conical Constraints ---
		points[0] = root_origin
		for i in range(n):
			var desired_dir: Vector3 = (points[i + 1] - points[i]).normalized()
			
			# Conical constraint check against predecessor direction
			if i > 0 and max_cone_angle_rad < PI:
				var prev_bone_dir: Vector3 = (points[i] - points[i - 1]).normalized()
				var angle: float = prev_bone_dir.angle_to(desired_dir)
				if angle > max_cone_angle_rad:
					var rot_axis: Vector3 = prev_bone_dir.cross(desired_dir)
					if rot_axis.length_squared() > 0.00001:
						desired_dir = prev_bone_dir.rotated(rot_axis.normalized(), max_cone_angle_rad)

			points[i + 1] = points[i] + (desired_dir * lengths[i])
```

---

## 6.4 Procedural Multi-Legged Gait Synthesis & Terrain Alignment

Keyframing movement for a 4-, 6-, or 8-legged creature across uneven terrain is practically impossible. We generate locomotion **100% procedurally** using raycast targets, parabolic step arcs, and **Alternating Gait Groups**.

```
                   ALTERNATING TETRAPOD GAIT (8-LEGGED ARACHNID)
=================================================================================
         Group A (●) vs. Group B (○) — Opposite diagonal legs step together!

                   Left Side       ╔═══════╗       Right Side
                    ● Leg L1 ──────╢ HEAD  ╟────── ○ Leg R1
                    ○ Leg L2 ──────╢       ╟────── ● Leg R2
                    ● Leg L3 ──────╢ TORSO ╟────── ○ Leg R3
                    ○ Leg L4 ──────╢       ╟────── ● Leg R4
                                   ╚═══════╝
  Invariant: Leg L1 (Group A) is FORBIDDEN from lifting off the ground while
  any leg in Group B is mid-air! This guarantees static physical stability.
```

### The Parabolic Step Trajectory Equation
When a planted foot at $\vec{P}_{\text{start}}$ lags behind its home raycast point $\vec{P}_{\text{target}}$ by distance $R_{\text{stride}}$, it initiates a step over duration $T_{\text{step}}$. For normalized progress $t \in [0, 1]$:

$$t_{\text{smooth}} = 3t^2 - 2t^3 \quad (\text{Hermite Smoothstep})$$

$$\vec{P}_{\text{foot}}(t) = \text{lerp}\left(\vec{P}_{\text{start}}, \, \vec{P}_{\text{target}}, \, t_{\text{smooth}}\right) + \hat{u}_{\text{up}} \cdot \sin(\pi t) \cdot H_{\text{lift}}$$

At $t = 0$, $\sin(0) = 0$. At the step apex ($t = 0.5$), $\sin(\frac{\pi}{2}) = 1.0$ (maximum vertical clearance). At landing ($t = 1.0$), $\sin(\pi) = 0$.

---

### Step-by-Step Architectural Construction: Procedural Walker Controller

Let us deconstruct our gait management, step evaluation, and chassis orientation logic.

#### Chunk 5.1: Leg Tracking Data Structures & Setup

```gdscript
class_name ProceduralWalkerController
extends Node3D

class LegStepper:
	var leg_id: int
	var gait_group: int # 0 = Group A, 1 = Group B
	var raycast: RayCast3D
	var ik_target: Marker3D
	
	var planted_world_pos: Vector3
	var step_start_pos: Vector3
	var step_goal_pos: Vector3
	var step_progress: float = 1.0
	
	func is_stepping() -> bool:
		return step_progress < 1.0

@export_group("Stride & Gait Parameters")
@export var stride_threshold: float = 0.65
@export var step_duration: float = 0.18
@export var step_lift_height: float = 0.35
@export var velocity_overshoot_factor: float = 0.45
@export var body_height_offset: float = 1.1
@export var body_alignment_decay: float = 10.0

@export_group("Scene References")
@export var chassis_body: CharacterBody3D
@export var raycast_container: Node3D
@export var ik_target_container: Node3D

var _legs: Array[LegStepper] = []
```

- **`class LegStepper`**: Holds the runtime step state for each limb, including start points, target goals, and the active progress accumulator ($t \in [0, 1]$).
- **`gait_group: int = i % 2`**: Partitions alternating legs into Group 0 and Group 1.
- **`step_progress: float = 1.0`**: Initialized to $1.0$ (grounded). Values $< 1.0$ indicate the foot is mid-air.

#### Chunk 5.2: Step Trajectory Updates

```gdscript
func _update_stepping_cycles(delta: float) -> void:
	for leg in _legs:
		if leg.is_stepping():
			leg.step_progress = minf(1.0, leg.step_progress + (delta / step_duration))
			var t: float = leg.step_progress
			var smooth_t: float = t * t * (3.0 - 2.0 * t)
			var horizontal_pos: Vector3 = leg.step_start_pos.lerp(leg.step_goal_pos, smooth_t)
			var vertical_arc: Vector3 = global_transform.basis.y * (sin(t * PI) * step_lift_height)
			
			leg.planted_world_pos = horizontal_pos + vertical_arc
			
		leg.ik_target.global_position = leg.planted_world_pos
```

- **`t * t * (3.0 - 2.0 * t)`**: Evaluates cubic Hermite smoothstep, providing gentle acceleration and deceleration on foot liftoff and touchdown.
- **`global_transform.basis.y * (sin(t * PI) * step_lift_height)`**: Elevates the foot along the creature's local up vector using a sine arc.
- **`leg.ik_target.global_position = leg.planted_world_pos`**: Pins the IK target in world space so the planted foot does not slide as the main body moves forward.

#### Chunk 5.3: Stride Evaluation & Gait Group Locking

```gdscript
func _evaluate_new_steps() -> void:
	var group_0_busy: bool = false
	var group_1_busy: bool = false
	for leg in _legs:
		if leg.is_stepping():
			if leg.gait_group == 0: group_0_busy = true
			else: group_1_busy = true

	for leg in _legs:
		if leg.is_stepping() or not leg.raycast.is_colliding():
			continue
			
		# Lock: Do not step if the opposing group is mid-air
		if (leg.gait_group == 0 and group_1_busy) or (leg.gait_group == 1 and group_0_busy):
			continue
			
		var home_point: Vector3 = leg.raycast.get_collision_point()
		var dist_sq: float = leg.planted_world_pos.distance_squared_to(home_point)
		
		if dist_sq > (stride_threshold * stride_threshold):
			var overshoot: Vector3 = chassis_body.velocity * step_duration * velocity_overshoot_factor
			leg.step_start_pos = leg.planted_world_pos
			leg.step_goal_pos = home_point + overshoot
			leg.step_progress = 0.0
			
			if leg.gait_group == 0: group_0_busy = true
			else: group_1_busy = true
```

- **`group_busy` Gatekeeper**: Enforces alternating gaits. A leg in Group A cannot lift off while any leg in Group B is airborne, ensuring the creature always maintains grounded contact points for balance.
- **`chassis_body.velocity * step_duration * overshoot`**: Velocity Prediction. Rather than stepping toward where the home point is *now*, it predicts where the home point will be when the step finishes, allowing the creature to step forward into motion.

#### Chunk 5.4: Surface Normal Alignment via Geometric Cross Products

```gdscript
func _align_chassis_to_legs(delta: float) -> void:
	if _legs.size() < 3:
		return
		
	var avg_foot_pos: Vector3 = Vector3.ZERO
	for leg in _legs:
		avg_foot_pos += leg.planted_world_pos
	avg_foot_pos /= float(_legs.size())

	var composite_normal: Vector3 = Vector3.ZERO
	var count: int = _legs.size()
	for i in range(count):
		var v1: Vector3 = _legs[i].planted_world_pos - avg_foot_pos
		var v2: Vector3 = _legs[(i + 1) % count].planted_world_pos - avg_foot_pos
		var n: Vector3 = v1.cross(v2)
		if n.dot(global_transform.basis.y) < 0.0:
			n = -n
		composite_normal += n

	var target_up: Vector3 = composite_normal.normalized()
	if target_up.length_squared() < 0.01:
		target_up = Vector3.UP

	var decay_weight: float = 1.0 - exp(-body_alignment_decay * delta)
	var desired_height: float = avg_foot_pos.y + body_height_offset
	global_position.y = lerpf(global_position.y, desired_height, decay_weight)

	var current_up: Vector3 = global_transform.basis.y
	var align_quat: Quaternion = Quaternion(current_up, target_up)
	var target_basis: Basis = Basis(align_quat) * global_transform.basis
	global_transform.basis = global_transform.basis.slerp(target_basis.orthonormalized(), decay_weight)
```

- **`avg_foot_pos`**: Finds the center of mass (centroid) of all planted contact points.
- **`v1.cross(v2)`**: Evaluates cross products between adjacent leg vectors radiating from the centroid, constructing a composite terrain normal vector.
- **`Quaternion(current_up, target_up)`**: Calculates the shortest rotational arc to align the body's up vector with the terrain slope, then interpolates smoothly using exponential decay.

---

#### The Fully Assembled `ProceduralWalkerController.gd`

```gdscript
# ==============================================================================
# SCRIPT: ProceduralWalkerController.gd
# Architecture: Alternating Gait Engine & Surface Normal Alignment
# ==============================================================================
class_name ProceduralWalkerController
extends Node3D

class LegStepper:
	var leg_id: int
	var gait_group: int # 0 = Group A, 1 = Group B
	var raycast: RayCast3D
	var ik_target: Marker3D
	
	var planted_world_pos: Vector3
	var step_start_pos: Vector3
	var step_goal_pos: Vector3
	var step_progress: float = 1.0
	
	func is_stepping() -> bool:
		return step_progress < 1.0

@export_group("Stride & Gait Parameters")
## Maximum distance between planted foot and home target before a step triggers
@export var stride_threshold: float = 0.65
## Temporal duration of a single leg swing (seconds)
@export var step_duration: float = 0.18
## Peak apex height of the parabolic foot arc (metres)
@export var step_lift_height: float = 0.35
## Multiplier projecting step targets ahead of body velocity
@export var velocity_overshoot_factor: float = 0.45
## Vertical height offset of the torso above the average foot plane
@export var body_height_offset: float = 1.1
## Exponential decay rate for torso smoothing
@export var body_alignment_decay: float = 10.0

@export_group("Scene References")
## Controlling kinematic body
@export var chassis_body: CharacterBody3D
## Container node holding RayCast3D nodes pointing at ground targets
@export var raycast_container: Node3D
## Container node holding Marker3D targets feeding the limb IK solvers
@export var ik_target_container: Node3D

var _legs: Array[LegStepper] = []

func _ready() -> void:
	_initialize_leg_arrays()

func _initialize_leg_arrays() -> void:
	var ray_count: int = raycast_container.get_child_count()
	for i in range(ray_count):
		var ray: RayCast3D = raycast_container.get_child(i) as RayCast3D
		var target: Marker3D = ik_target_container.get_child(i) as Marker3D
		if ray == null or target == null:
			continue
			
		var leg: LegStepper = LegStepper.new()
		leg.leg_id = i
		leg.gait_group = i % 2
		leg.raycast = ray
		leg.ik_target = target
		
		ray.force_raycast_update()
		leg.planted_world_pos = ray.get_collision_point() if ray.is_colliding() else target.global_position
		leg.step_goal_pos = leg.planted_world_pos
		_legs.append(leg)

func _physics_process(delta: float) -> void:
	_update_stepping_cycles(delta)
	_evaluate_new_steps()
	_align_chassis_to_legs(delta)

func _update_stepping_cycles(delta: float) -> void:
	for leg in _legs:
		if leg.is_stepping():
			leg.step_progress = minf(1.0, leg.step_progress + (delta / step_duration))
			var t: float = leg.step_progress
			# Smoothstep horizontal progress
			var smooth_t: float = t * t * (3.0 - 2.0 * t)
			var horizontal_pos: Vector3 = leg.step_start_pos.lerp(leg.step_goal_pos, smooth_t)
			# Parabolic vertical arc
			var vertical_arc: Vector3 = global_transform.basis.y * (sin(t * PI) * step_lift_height)
			
			leg.planted_world_pos = horizontal_pos + vertical_arc
			
		# Pin IK target in world space
		leg.ik_target.global_position = leg.planted_world_pos

func _evaluate_new_steps() -> void:
	var group_0_busy: bool = false
	var group_1_busy: bool = false
	for leg in _legs:
		if leg.is_stepping():
			if leg.gait_group == 0:
				group_0_busy = true
			else:
				group_1_busy = true

	for leg in _legs:
		if leg.is_stepping() or not leg.raycast.is_colliding():
			continue
			
		# Lock gait groups: only step if opposite group is grounded
		if (leg.gait_group == 0 and group_1_busy) or (leg.gait_group == 1 and group_0_busy):
			continue
			
		var home_point: Vector3 = leg.raycast.get_collision_point()
		var dist_sq: float = leg.planted_world_pos.distance_squared_to(home_point)
		
		if dist_sq > (stride_threshold * stride_threshold):
			var overshoot: Vector3 = chassis_body.velocity * step_duration * velocity_overshoot_factor
			leg.step_start_pos = leg.planted_world_pos
			leg.step_goal_pos = home_point + overshoot
			leg.step_progress = 0.0
			
			if leg.gait_group == 0:
				group_0_busy = true
			else:
				group_1_busy = true

func _align_chassis_to_legs(delta: float) -> void:
	if _legs.size() < 3:
		return
		
	var avg_foot_pos: Vector3 = Vector3.ZERO
	for leg in _legs:
		avg_foot_pos += leg.planted_world_pos
	avg_foot_pos /= float(_legs.size())

	# Compute composite surface normal from cross products around the centroid
	var composite_normal: Vector3 = Vector3.ZERO
	var count: int = _legs.size()
	for i in range(count):
		var v1: Vector3 = _legs[i].planted_world_pos - avg_foot_pos
		var v2: Vector3 = _legs[(i + 1) % count].planted_world_pos - avg_foot_pos
		var n: Vector3 = v1.cross(v2)
		if n.dot(global_transform.basis.y) < 0.0:
			n = -n
		composite_normal += n

	var target_up: Vector3 = composite_normal.normalized()
	if target_up.length_squared() < 0.01:
		target_up = Vector3.UP

	var decay_weight: float = 1.0 - exp(-body_alignment_decay * delta)
	var desired_height: float = avg_foot_pos.y + body_height_offset
	global_position.y = lerpf(global_position.y, desired_height, decay_weight)

	# Align body up vector to the terrain normal
	var current_up: Vector3 = global_transform.basis.y
	var align_quat: Quaternion = Quaternion(current_up, target_up)
	var target_basis: Basis = Basis(align_quat) * global_transform.basis
	global_transform.basis = global_transform.basis.slerp(target_basis.orthonormalized(), decay_weight)
```

---

## 6.5 Active Ragdolls & Physics-Driven Animation (`PhysicalBoneSimulator3D`)

Standard ragdolls are passive: when an entity dies, simulation starts and gravity collapses the bones into a limp pile.

An **Active Ragdoll** blends keyframed animation with rigid-body dynamics simultaneously. Each `PhysicalBone3D` uses **Proportional-Derivative (PD) Spring-Damper Torque** ($\vec{\tau}$) to drive its physical body toward animated target bone orientations:

$$\vec{\tau} = k_p \cdot \vec{e}_{\theta} - k_d \cdot \vec{\omega}$$

where:
- $\vec{e}_{\theta}$ is the shortest-arc axis-angle error vector between the physical bone's current quaternion $\mathbf{q}_{\text{phys}}$ and the target animation quaternion $\mathbf{q}_{\text{anim}}$ (derived from $\mathbf{q}_{\text{diff}} = \mathbf{q}_{\text{anim}} \cdot \mathbf{q}_{\text{phys}}^{-1}$).
- $k_p$ is the **Proportional Stiffness Gain** (muscular strength).
- $k_d$ is the **Derivative Damping Gain** (prevents violent oscillation).
- $\vec{\omega}$ is the bone's current `angular_velocity`.

---

### Step-by-Step Architectural Construction: Active Ragdoll PD Servo

Let us deconstruct the quaternion error evaluation, derivative damping, and torque application line by line.

#### Chunk 6.1: State Binding & Difference Quaternion Extraction

```gdscript
class_name ActiveRagdollBoneController
extends PhysicalBone3D

@export var target_skeleton: Skeleton3D
@export var stiffness_kp: float = 650.0
@export var damping_kd: float = 45.0
@export var max_muscle_torque: float = 1200.0
@export_range(0.0, 1.0) var muscular_strength_multiplier: float = 1.0

var _bone_id: int = -1

func _ready() -> void:
	if target_skeleton != null:
		_bone_id = target_skeleton.find_bone(bone_name)

func _integrate_forces(state: PhysicsDirectBodyState3D) -> void:
	if target_skeleton == null or _bone_id == -1 or is_zero_approx(muscular_strength_multiplier):
		return

	var anim_skel_pose: Transform3D = target_skeleton.get_bone_global_pose_no_override(_bone_id)
	var target_world_basis: Basis = (target_skeleton.global_transform * anim_skel_pose).basis
	
	var q_target: Quaternion = target_world_basis.get_rotation_quaternion()
	var q_current: Quaternion = state.transform.basis.get_rotation_quaternion()

	var q_diff: Quaternion = q_target * q_current.inverse()
	if q_diff.w < 0.0:
		q_diff = -q_diff
```

- **`_integrate_forces(state)`**: Overrides the physics server hook. This executes during the physics integration step, giving us access to angular velocity and body transforms.
- **`q_target * q_current.inverse()`**: Solves for the relative difference quaternion $\mathbf{q}_{\text{diff}}$ that rotates the physical bone from its current orientation into the animated target orientation.
- **`if q_diff.w < 0.0: q_diff = -q_diff`**: **The 4D Hemisphere Invariant**. Because quaternions double-cover 3D rotations ($\mathbf{q}$ and $-\mathbf{q}$ represent the same physical rotation), checking if $w < 0$ and inverting signs ensures the solver always traverses the shortest rotational arc on the 4D hypersphere.

#### Chunk 6.2: Axis-Angle Extraction & PD Torque Synthesis

```gdscript
	var axis: Vector3 = q_diff.get_axis()
	var angle: float = q_diff.get_angle()
	
	if not axis.is_normalized() or is_zero_approx(angle):
		return

	var angular_error: Vector3 = axis * angle

	var raw_torque: Vector3 = (angular_error * stiffness_kp) - (state.angular_velocity * damping_kd)
	var final_torque: Vector3 = (raw_torque * muscular_strength_multiplier).limit_length(max_muscle_torque)

	state.apply_torque(final_torque)
```

- **`axis * angle`**: Extracts the error vector $\vec{e}_{\theta}$ where the direction vector is the axis of rotation and the magnitude is the angular discrepancy in radians.
- **`(angular_error * stiffness_kp) - (state.angular_velocity * damping_kd)`**: The core PD control loop:
  - The proportional term ($\vec{e}_{\theta} \cdot k_p$) pushes the bone toward the animated pose like a physical spring.
  - The derivative term ($\vec{\omega} \cdot k_d$) resists high angular speeds, damping oscillations and preventing the ragdoll from vibrating.
- **`state.apply_torque(final_torque)`**: Injects physical rotational torque into the rigid body simulation.

---

#### The Fully Assembled `ActiveRagdollBoneController.gd`

```gdscript
# ==============================================================================
# SCRIPT: ActiveRagdollBoneController.gd
# Architecture: Proportional-Derivative (PD) Active Ragdoll Torque Servo
# ==============================================================================
class_name ActiveRagdollBoneController
extends PhysicalBone3D

@export_group("Animation Source")
## Animated reference skeleton whose poses serve as physical targets
@export var target_skeleton: Skeleton3D

@export_group("PD Muscle Parameters")
## Proportional spring stiffness (muscular power)
@export var stiffness_kp: float = 650.0
## Derivative damper gain (prevents violent oscillation)
@export var damping_kd: float = 45.0
## Maximum torque magnitude applied to the physical bone
@export var max_muscle_torque: float = 1200.0
## Overall muscular tension scalar [0.0 = Limp ragdoll, 1.0 = Full muscle stiffness]
@export_range(0.0, 1.0) var muscular_strength_multiplier: float = 1.0

var _bone_id: int = -1

func _ready() -> void:
	if target_skeleton != null:
		_bone_id = target_skeleton.find_bone(bone_name)

## Physics Server force integration callback
func _integrate_forces(state: PhysicsDirectBodyState3D) -> void:
	if target_skeleton == null or _bone_id == -1 or is_zero_approx(muscular_strength_multiplier):
		return

	# 1. Sample un-overridden animated pose in world space
	var anim_skel_pose: Transform3D = target_skeleton.get_bone_global_pose_no_override(_bone_id)
	var target_world_basis: Basis = (target_skeleton.global_transform * anim_skel_pose).basis
	
	var q_target: Quaternion = target_world_basis.get_rotation_quaternion()
	var q_current: Quaternion = state.transform.basis.get_rotation_quaternion()

	# 2. Compute difference quaternion: q_diff = q_target * inv(q_current)
	var q_diff: Quaternion = q_target * q_current.inverse()
	
	# Shortest-path hemisphere check
	if q_diff.w < 0.0:
		q_diff = -q_diff

	# 3. Extract rotation axis and angular error
	var axis: Vector3 = q_diff.get_axis()
	var angle: float = q_diff.get_angle()
	
	if not axis.is_normalized() or is_zero_approx(angle):
		return

	var angular_error: Vector3 = axis * angle

	# 4. PD Controller: Torque = (Kp * error) - (Kd * angular_velocity)
	var raw_torque: Vector3 = (angular_error * stiffness_kp) - (state.angular_velocity * damping_kd)
	var final_torque: Vector3 = (raw_torque * muscular_strength_multiplier).limit_length(max_muscle_torque)

	state.apply_torque(final_torque)
```

---
