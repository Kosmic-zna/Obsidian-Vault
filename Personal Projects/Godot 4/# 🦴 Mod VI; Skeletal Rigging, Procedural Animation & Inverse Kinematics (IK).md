# 🦴 Module VI: Skeletal Rigging, Procedural Animation & Inverse Kinematics (IK)

> [!info] 📖 Lexicon & First Principles: Skeletal & Kinematic Terminology
> Before synthesizing procedural motion or deriving inverse kinematic solvers, we must establish formal definitions for the anatomical and mathematical constructs of skeletal animation:
> - ⤷ **Armature (`Skeleton2D` / `Skeleton3D`)**: A hierarchical tree of virtual bones where each bone's global transform is the cumulative matrix product of its local pose and all ancestor bone poses up to the root.
> - ⤷ **Rest Pose vs. Pose Override**: The **Rest Pose** is the default, un-animated bind configuration of a skeleton (typically a T-Pose or A-Pose) used to weight mesh vertices. A **Pose Override** is the runtime matrix offset applied to a bone on a given frame.
> - ⤷ **Forward Kinematics (FK)**: The standard animation paradigm where parent joint angles are rotated explicitly ($\theta_1, \theta_2, \dots, \theta_n$), and the resulting world coordinate of the extremity is computed forward down the chain.
> - ⤷ **Inverse Kinematics (IK)**: The mathematical inverse of FK—defining the desired spatial coordinate of the terminal extremity first and ==solving backward for the interior joint rotations== required to place the extremity at that coordinate.
> - ⤷ **End-Effector**: The terminal bone or tip of a kinematic chain (e.g., hand, foot ankle, tail tip, arachnid tarsus) whose spatial target drives the IK solver.
> - ⤷ **Pole Target (Swivel Constraint)**: A spatial reference coordinate in 3D space that defines the plane of articulation for a hinge joint (such as an elbow or knee), preventing anatomical hyperextension or erratic roll flipping.

---

## 6.1 Skeletal Hierarchy & Coordinate Spaces in `Skeleton3D`

A common stumbling block for programmers attempting procedural animation is confusing the three distinct coordinate spaces operating simultaneously inside a `Skeleton3D` node. Unlike standard `Node3D` hierarchies, bones are not individual nodes in the Scene Tree; they are indexed elements inside a single contiguous C++ array managed by the `Skeleton3D` instance.

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

```gdscript
# ==============================================================================
# SKELETAL COORDINATE CONVERSION UTILITIES
# ==============================================================================
class_name SkeletalMath
extends RefCounted

## Converts a 3D world-space Transform3D into Skeleton-space Global Pose
static func world_to_skeleton_pose(skeleton: Skeleton3D, world_transform: Transform3D) -> Transform3D:
    return skeleton.global_transform.affine_inverse() * world_transform

## Converts a Skeleton-space Global Pose into a 3D world-space Transform3D
static func skeleton_pose_to_world(skeleton: Skeleton3D, skel_pose: Transform3D) -> Transform3D:
    return skeleton.global_transform * skel_pose

## Converts a Skeleton-space Global Pose into a bone's parent-relative Local Pose
static func global_pose_to_local_pose(skeleton: Skeleton3D, bone_idx: int, skel_global_pose: Transform3D) -> Transform3D:
    var parent_idx: int = skeleton.get_bone_parent(bone_idx)
    if parent_idx == -1:
        return skel_global_pose
    var parent_global_pose: Transform3D = skeleton.get_bone_global_pose(parent_idx)
    return parent_global_pose.affine_inverse() * skel_global_pose
```

> [!warning] ⚠️ Production Trap: `SkeletonModificationStack3D` Deprecation in Godot 4.3+
> Early Godot 4.0–4.2 tutorials reference `SkeletonModificationStack3D` and `SkeletonIK3D`. Both were officially **deprecated and replaced in Godot 4.3+** by the unified **`SkeletonModifier3D`** architecture. Never build new 3D procedural animation pipelines on deprecated modification stacks; always extend `SkeletonModifier3D` and override `_process_modification()`.

---

## 6.2 `AnimationTree`, Blend Spaces & Root Motion Extraction

Keyframed animations stored inside an `AnimationPlayer` are static tracks. To blend smoothly between directional locomotion states, layer upper-body combat actions over lower-body running, and synchronize physical velocity with footwork, we construct an **`AnimationTree`** powered by an `AnimationNodeBlendTree`.

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
If a player swings a sword or reloads a weapon while sprinting, playing the full-body attack animation overwrites the leg bones, causing the character's feet to slide rigidly across the floor ("ice-skating").
- ⤷ **The Solution**: Enable **Filter** on the `AnimationNodeOneShot` or `AnimationNodeBlend2` node and check only the `Spine`, `Chest`, `Shoulders`, `Arms`, and `Head` bones. The lower-body bones (`Hips`, `Thighs`, `Calves`, `Feet`) continue executing the `BlendSpace2D` sprint cycle uninterrupted.

### 3. Root Motion Extraction (`get_root_motion_position()`)
In precision melee combat games (e.g., *Monster Hunter*, *Elden Ring*), driving character velocity via a flat mathematical curve during a sword lunge causes the feet to slide out of sync with the floor texture.
- ⤷ **Root Motion Mechanism**: The animator animates the actual `Root` bone moving forward in 3D space during the lunge. At runtime, Godot strips that translation off the visual skeleton and hands the frame-delta displacement $\Delta \vec{p}_{\text{root}}$ and rotation $\Delta \mathbf{q}_{\text{root}}$ to your `CharacterBody3D` script:

$$\vec{v}_{\text{kinematic}} = \frac{\mathbf{B}_{\text{actor}} \cdot \Delta \vec{p}_{\text{root}}}{\Delta t}$$

```gdscript
# ==============================================================================
# PRODUCTION ANIMATION TREE & ROOT MOTION CONTROLLER
# ==============================================================================
class_name LocomotionAnimController
extends Node

@export var actor: CharacterBody3D
@export var animation_tree: AnimationTree
@export var blend_smoothing_speed: float = 12.0

# Pre-cached StringName parameter paths to avoid per-frame string allocation
const PARAM_LOCOMOTION_BLEND: StringName = &"parameters/Locomotion/blend_position"
const PARAM_ATTACK_REQUEST: StringName = &"parameters/AttackOneShot/request"
const PARAM_TIMESCALE: StringName = &"parameters/TimeScale/scale"

var _current_blend_pos: Vector2 = Vector2.ZERO

func _ready() -> void:
    assert(animation_tree != null, "LocomotionAnimController: AnimationTree unassigned.")
    animation_tree.active = true

func update_locomotion_blend(world_velocity: Vector3, max_speed: float, delta: float) -> void:
    # Convert world velocity into actor's local coordinate space (Right = +X, Forward = -Z)
    var local_vel: Vector3 = actor.global_transform.basis.orthonormalized().transposed() * world_velocity
    var target_blend: Vector2 = Vector2(local_vel.x, -local_vel.z) / maxf(0.001, max_speed)
    target_blend = target_blend.limit_length(1.0)
    
    # Frame-rate independent exponential smoothing
    var weight: float = 1.0 - exp(-blend_smoothing_speed * delta)
    _current_blend_pos = _current_blend_pos.lerp(target_blend, weight)
    
    animation_tree.set(PARAM_LOCOMOTION_BLEND, _current_blend_pos)

func trigger_upper_body_action() -> void:
    animation_tree.set(PARAM_ATTACK_REQUEST, AnimationNodeOneShot.ONE_SHOT_REQUEST_FIRE)

## Extracts baked root motion velocity to drive CharacterBody3D.velocity
func extract_root_motion_velocity(delta: float) -> Vector3:
    if delta <= 0.00001:
        return Vector3.ZERO
        
    # Retrieve rotation and position deltas for the current physics tick
    var delta_rot: Quaternion = animation_tree.get_root_motion_rotation()
    var delta_pos: Vector3 = animation_tree.get_root_motion_position()
    
    # Apply root rotation directly to the actor's basis
    actor.quaternion = (actor.quaternion * delta_rot).normalized()
    
    # Transform local root displacement into world-space velocity
    var world_displacement: Vector3 = actor.global_transform.basis * delta_pos
    return world_displacement / delta
```

---

## 6.3 Inverse Kinematics (IK): Mathematical Proofs & Solvers

When a character walks up a staircase, reaches for a door handle, or aims a rifle at an elevated target, pre-baked animations cannot anticipate the exact world coordinates of the environment. We must solve the **Inverse Kinematics** problem.

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

For any two-segment limb—such as an upper arm ($L_1$) and forearm ($L_2$), or thigh ($L_1$) and calf ($L_2$)—iterative solvers are unnecessary. Because three points (Root $\vec{R}$, Mid-Joint $\vec{J}$, and Target $\vec{T}$) always form a triangle in a 2D plane defined by the **Pole Target** $\vec{P}$, we can compute the exact joint angles in $\mathcal{O}(1)$ time using the **Law of Cosines**.

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

> [!example] 🧪 Step-by-Step Mathematical Proof: Two-Bone Analytical IK
> Let:
> - $L_1 = \|\vec{J} - \vec{R}\|$ be the length of the upper bone (thigh/upper arm).
> - $L_2 = \|\vec{E} - \vec{J}\|$ be the length of the lower bone (calf/forearm).
> - $D = \|\vec{T} - \vec{R}\|$ be the Euclidean distance from the root socket to the target.
> 
> **Step 1 — Reachability Clamping (Triangle Inequality)**:
> A triangle can only exist if $D$ lies between the fully folded length and fully extended length. To prevent floating-point domain errors ($\arccos(x)$ where $|x| > 1.0$, which returns `NaN`), we clamp $D$ with a tiny epsilon $\epsilon = 10^{-4}$:
> $$D_{\text{clamped}} = \text{clamp}\left(\|\vec{T} - \vec{R}\|, \, |L_1 - L_2| + \epsilon, \, (L_1 + L_2) - \epsilon\right)$$
> 
> **Step 2 — Solving Shoulder/Hip Pitch Angle ($\alpha$)**:
> By the Law of Cosines ($c^2 = a^2 + b^2 - 2ab\cos C$), the side opposite to angle $\alpha$ is $L_2$:
> $$L_2^2 = L_1^2 + D^2 - 2 L_1 D \cos(\alpha) \implies \cos(\alpha) = \frac{L_1^2 + D^2 - L_2^2}{2 L_1 D}$$
> $$\alpha = \arccos\left(\text{clamp}\left(\frac{L_1^2 + D^2 - L_2^2}{2 L_1 D}, \, -1.0, \, 1.0\right)\right)$$
> 
> **Step 3 — Solving Elbow/Knee Interior Flexion Angle ($\beta$)**:
> The side opposite to interior angle $\beta$ is $D$:
> $$D^2 = L_1^2 + L_2^2 - 2 L_1 L_2 \cos(\beta) \implies \cos(\beta) = \frac{L_1^2 + L_2^2 - D^2}{2 L_1 L_2}$$
> $$\beta = \arccos\left(\text{clamp}\left(\frac{L_1^2 + L_2^2 - D^2}{2 L_1 L_2}, \, -1.0, \, 1.0\right)\right)$$
> 
> **Step 4 — Constructing the Articulation Plane via the Pole Target ($\vec{P}$)**:
> In 3D space, a triangle of side lengths $(L_1, L_2, D)$ can swivel $360^\circ$ around the axis $\hat{u}_{\text{forward}} = \frac{\vec{T} - \vec{R}}{\|\vec{T} - \vec{R}\|}$. To lock the knee or elbow toward the Pole Target $\vec{P}$, we construct an orthonormal basis using the Gram-Schmidt process:
> $$\hat{u}_{\text{axis}} = \frac{\vec{T} - \vec{R}}{\|\vec{T} - \vec{R}\|} \qquad \hat{u}_{\text{hinge}} = \frac{\hat{u}_{\text{axis}} \times (\vec{P} - \vec{R})}{\|\hat{u}_{\text{axis}} \times (\vec{P} - \vec{R})\|}$$
> Rotating $\vec{R} \rightarrow \vec{T}$ around $\hat{u}_{\text{hinge}}$ by angle $\alpha$ yields the exact 3D coordinate of the mid-joint $\vec{J}$!

```gdscript
# ==============================================================================
# CUSTOM SKELETONMODIFIER3D: ANALYTICAL TWO-BONE IK SOLVER
# ==============================================================================
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

## Overridden virtual method called automatically by Skeleton3D during pose evaluation
func _process_modification() -> void:
    var skeleton: Skeleton3D = get_skeleton()
    if skeleton == null or target_node == null or pole_node == null:
        return
    if not _validate_bone_indices(skeleton):
        return

    # 1. Extract bone origins in Skeleton Global Pose space
    var root_pose: Transform3D = skeleton.get_bone_global_pose(_root_idx)
    var mid_pose: Transform3D = skeleton.get_bone_global_pose(_mid_idx)
    var tip_pose: Transform3D = skeleton.get_bone_global_pose(_tip_idx)

    var p_root: Vector3 = root_pose.origin
    var p_mid: Vector3 = mid_pose.origin
    var p_tip: Vector3 = tip_pose.origin

    var inv_skel: Transform3D = skeleton.global_transform.affine_inverse()
    var p_target: Vector3 = inv_skel * target_node.global_position
    var p_pole: Vector3 = inv_skel * pole_node.global_position

    # 2. Measure fixed segment lengths L1 and L2
    var l1: float = p_root.distance_to(p_mid)
    var l2: float = p_mid.distance_to(p_tip)
    var epsilon: float = 0.0001
    
    var to_target: Vector3 = p_target - p_root
    var raw_dist: float = to_target.length()
    var d: float = clampf(raw_dist, absf(l1 - l2) + epsilon, (l1 + l2) - epsilon)

    # 3. Law of Cosines: Solve angle alpha (root-to-target vs root-to-mid)
    var cos_alpha: float = clampf(((l1 * l1) + (d * d) - (l2 * l2)) / (2.0 * l1 * d), -1.0, 1.0)
    var alpha: float = acos(cos_alpha)

    # 4. Construct orthonormal articulation plane using Pole Target
    var dir_target: Vector3 = to_target / maxf(epsilon, raw_dist)
    var dir_pole: Vector3 = (p_pole - p_root).normalized()
    
    var hinge_normal: Vector3 = dir_target.cross(dir_pole)
    if hinge_normal.length_squared() < epsilon:
        hinge_normal = dir_target.cross(Vector3.UP)
    hinge_normal = hinge_normal.normalized()

    # 5. Compute exact Mid-Joint position and Target Tip position
    var mid_dir: Vector3 = dir_target.rotated(hinge_normal, -alpha).normalized()
    var solved_mid_pos: Vector3 = p_root + (mid_dir * l1)
    var solved_tip_pos: Vector3 = p_root + (dir_target * d)

    # 6. Construct solved Basis matrices (assuming +Y is bone forward axis along length)
    var root_y: Vector3 = (solved_mid_pos - p_root).normalized()
    var root_x: Vector3 = hinge_normal
    var root_z: Vector3 = root_x.cross(root_y).normalized()
    root_pose.basis = Basis(root_x, root_y, root_z).orthonormalized()

    var mid_y: Vector3 = (solved_tip_pos - solved_mid_pos).normalized()
    var mid_x: Vector3 = hinge_normal
    var mid_z: Vector3 = mid_x.cross(mid_y).normalized()
    mid_pose.origin = solved_mid_pos
    mid_pose.basis = Basis(mid_x, mid_y, mid_z).orthonormalized()

    # 7. Write back solved poses to Skeleton3D
    skeleton.set_bone_global_pose(_root_idx, root_pose)
    skeleton.set_bone_global_pose(_mid_idx, mid_pose)
```

---

### 2. Multi-Joint Iterative IK: FABRIK (Forward And Backward Reaching IK)

When a kinematic chain contains more than two segments ($N \ge 3$, such as a dragon's neck, a mechanical tentacle, a scorpion tail, or a spine), analytical trigonometry becomes under-determined (infinitely many valid solutions exist).

Rather than computing expensive Jacobian matrix pseudoinverses ($\mathbf{J}^+$) or slow Euler angle rotations (CCD), **FABRIK** solves the chain in pure Euclidean point space using iterative distance projections, converging to visual perfection in just $3\text{--}8$ iterations!

```
                      THE FABRIK TWO-PASS ITERATION LOOP
=================================================================================
 INITIAL STATE:                 PASS 1: FORWARD REACHING       PASS 2: BACKWARD REACHING
 Root p0 -> p1 -> p2 -> p3      Snap p3' to Target T;          Snap p0'' back to Root Origin;
                                pull p2', p1', p0' toward T    pull p1'', p2'', p3'' outward

   p0 ──► p1 ──► p2 ──► p3        p0'◄── p1'◄── p2'◄── p3'(T)   p0''──► p1''──► p2''──► p3''
                                 (Root pulled off socket!)      (Root locked; Tip at Target!)
```

#### Formal Mathematical Specification of FABRIK
Given $N+1$ joint positions $\vec{p}_0, \vec{p}_1, \dots, \vec{p}_N$ and $N$ fixed bone lengths $d_i = \|\vec{p}_{i+1} - \vec{p}_i\|$ for $i \in [0, N-1]$:
1. **Unreachable Fast-Path**: If $\|\vec{T} - \vec{p}_0\| \ge \sum_{i=0}^{N-1} d_i$, the target is out of reach. Simply point all bones straight toward $\vec{T}$ in a single $\mathcal{O}(N)$ pass and terminate:
   $$\vec{p}_{i+1} = \vec{p}_i + \frac{\vec{T} - \vec{p}_i}{\|\vec{T} - \vec{p}_i\|} \cdot d_i$$
2. **Stage A — Forward Reaching (Tip $\rightarrow$ Root)**:
   Set $\vec{p}_N' = \vec{T}$. For $i = N-1$ down to $0$:
   $$\vec{p}_i' = \vec{p}_{i+1}' + \frac{\vec{p}_i - \vec{p}_{i+1}'}{\|\vec{p}_i - \vec{p}_{i+1}'\|} \cdot d_i$$
3. **Stage B — Backward Reaching (Root $\rightarrow$ Tip)**:
   Set $\vec{p}_0'' = \vec{p}_{\text{root\_origin}}$. For $i = 0$ up to $N-1$:
   $$\vec{p}_{i+1}'' = \vec{p}_i'' + \frac{\vec{p}_{i+1}' - \vec{p}_i''}{\|\vec{p}_{i+1}' - \vec{p}_i''\|} \cdot d_i$$
4. **Termination**: Repeat Stages A and B until $\|\vec{p}_N'' - \vec{T}\|^2 < \epsilon^2$ or `max_iterations` is reached.

```gdscript
# ==============================================================================
# PRODUCTION N-BONE FABRIK SOLVER WITH CONICAL JOINT CONSTRAINTS
# ==============================================================================
class_name FABRIKSolver3D
extends RefCounted

## Solves an N-point chain in-place to reach target_pos while respecting max_cone_angle_rad
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

    # 1. Fast-Path: Target exceeds total chain reach
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

        # --- STAGE A: Forward Reaching (Tip to Root) ---
        points[n] = target_pos
        for i in range(n - 1, -1, -1):
            var dir_back: Vector3 = (points[i] - points[i + 1]).normalized()
            points[i] = points[i + 1] + (dir_back * lengths[i])

        # --- STAGE B: Backward Reaching (Root to Tip) with Conical Constraints ---
        points[0] = root_origin
        for i in range(n):
            var desired_dir: Vector3 = (points[i + 1] - points[i]).normalized()
            
            # Enforce conical angle constraint relative to previous bone vector
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

## 6.4 Procedural Foot Placement, Pelvis Adaptation & Arachnid Gaits

### 1. Bipedal Slope & Staircase Adaptation
When a bipedal character stands with one foot on a step of elevation $y = 0.4\text{ m}$ and the other foot on the floor at $y = 0.0\text{ m}$, simply moving the upper foot up via IK is only half the solution. Without lowering the character's **Pelvis / Hips bone**, the lower foot will hang $0.4\text{ m}$ in mid-air because its leg is already fully extended!

```
WITHOUT PELVIS ADAPTATION:              WITH PELVIS ADAPTATION:
Pelvis stays at flat-ground height;     Pelvis drops by Δy = min(foot_L_offset, foot_R_offset);
Lower leg hyperextends & floats!        Upper knee bends via IK, Lower foot plants firmly!

          [Pelvis]                                  │ (Pelvis lowered by Δy)
           /    \                                   ▼
     ╭────╯      \                               [Pelvis]
   [Foot L]       \                               /    \
  =========      [Foot R] (Floating!)       ╭────╯      \
  (High Step)                               [Foot L]     [Foot R]
                 =========                 =========    =========
```

- ⤷ **The Pelvis Offset Invariant**:
  1. Raycast downward from both foot bones ($L$ and $R$) to find ground hit elevations $y_{\text{hit}, L}$ and $y_{\text{hit}, R}$.
  2. Compute vertical displacements from the character's root base $y_{\text{root}}$:
     $$\Delta y_L = y_{\text{hit}, L} - y_{\text{root}}, \qquad \Delta y_R = y_{\text{hit}, R} - y_{\text{root}}$$
  3. Smoothly offset the `Pelvis` bone vertically by the **lowest** foot displacement:
     $$\Delta y_{\text{pelvis}} = \min(\Delta y_L, \Delta y_R)$$
  4. Align each ankle's rotation quaternion to the ground surface normal $\vec{n}_{\text{hit}}$ using cross-product basis reconstruction.

---

### 2. Procedural Multi-Legged Gait Synthesis (Arachnids, Mechs & Quadrupeds)

Keyframing walking animations for a 4-, 6-, or 8-legged creature on uneven terrain is practically impossible. Instead, we generate locomotion **100% procedurally** using raycast home targets and **Alternating Tetrapod / Tripod Gait Groups**.

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

#### The Parabolic Step Trajectory Equation
When a planted foot at world coordinate $\vec{P}_{\text{start}}$ lags behind its ideal home raycast target $\vec{P}_{\text{target}}$ by a distance exceeding stride threshold $R_{\text{stride}}$, it initiates a step over duration $T_{\text{step}}$. For normalized progress $t \in [0, 1]$:

$$t_{\text{smooth}} = 3t^2 - 2t^3 \quad (\text{Hermite Smoothstep})$$

$$\vec{P}_{\text{foot}}(t) = \text{lerp}\left(\vec{P}_{\text{start}}, \, \vec{P}_{\text{target}}, \, t_{\text{smooth}}\right) + \vec{n}_{\text{up}} \cdot \sin(\pi t) \cdot H_{\text{lift}}$$

Notice the elegance of $\sin(\pi t)$: at $t = 0$, $\sin(0) = 0$; at the apex $t = 0.5$, $\sin(\frac{\pi}{2}) = 1.0$ (maximum lift height $H_{\text{lift}}$); and upon landing at $t = 1.0$, $\sin(\pi) = 0$!

```gdscript
# ==============================================================================
# PRODUCTION PROCEDURAL GAIT & BODY NORMAL ALIGNMENT ENGINE
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
    var step_progress: float = 1.0 # 1.0 = firmly planted on ground
    
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
        leg.gait_group = i % 2 # Alternating Gait Groups (0 and 1)
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
            # Hermite smoothstep horizontal interpolation
            var smooth_t: float = t * t * (3.0 - 2.0 * t)
            var horizontal_pos: Vector3 = leg.step_start_pos.lerp(leg.step_goal_pos, smooth_t)
            # Parabolic sine arc for vertical foot clearance
            var vertical_arc: Vector3 = global_transform.basis.y * (sin(t * PI) * step_lift_height)
            
            leg.planted_world_pos = horizontal_pos + vertical_arc
            
        # Pin the IK Target node in world space so the foot does not slide!
        leg.ik_target.global_position = leg.planted_world_pos

func _evaluate_new_steps() -> void:
    # Check if either gait group is currently mid-air
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
            
        # Stability lock: Only step if the opposing gait group has all feet planted!
        if (leg.gait_group == 0 and group_1_busy) or (leg.gait_group == 1 and group_0_busy):
            continue
            
        var home_point: Vector3 = leg.raycast.get_collision_point()
        var dist_sq: float = leg.planted_world_pos.distance_squared_to(home_point)
        
        if dist_sq > (stride_threshold * stride_threshold):
            # Project step goal slightly ahead of current velocity so the leg steps INTO motion
            var overshoot: Vector3 = chassis_body.velocity * step_duration * velocity_overshoot_factor
            leg.step_start_pos = leg.planted_world_pos
            leg.step_goal_pos = home_point + overshoot
            leg.step_progress = 0.0
            
            if leg.gait_group == 0:
                group_0_busy = true
            else:
                group_1_busy = true

## Fits an average plane to all planted feet to tilt the spider/mech over slopes
func _align_chassis_to_legs(delta: float) -> void:
    if _legs.size() < 3:
        return
        
    var avg_foot_pos: Vector3 = Vector3.ZERO
    for leg in _legs:
        avg_foot_pos += leg.planted_world_pos
    avg_foot_pos /= float(_legs.size())

    # Accumulate cross-product normals across adjacent leg pairs around the centroid
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

    # Smoothly align torso height and surface pitch/roll
    var decay_weight: float = 1.0 - exp(-body_alignment_decay * delta)
    var desired_height: float = avg_foot_pos.y + body_height_offset
    global_position.y = lerpf(global_position.y, desired_height, decay_weight)

    var current_up: Vector3 = global_transform.basis.y
    var align_quat: Quaternion = Quaternion(current_up, target_up)
    var target_basis: Basis = Basis(align_quat) * global_transform.basis
    global_transform.basis = global_transform.basis.slerp(target_basis.orthonormalized(), decay_weight)
```

---

## 6.5 Active Ragdolls & Physics-Driven Animation (`PhysicalBoneSimulator3D`)

Standard ragdolls are completely limp; when an enemy dies, you call `physical_bones_start_simulation()` on `PhysicalBoneSimulator3D` and gravity collapses the skeleton.

An **Active Ragdoll** (seen in *Half-Life: Alyx*, *Gang Beasts*, *Euphoria* engine) combines keyframed animation with rigid-body physics simultaneously. Every `PhysicalBone3D` applies rotational **PD (Proportional-Derivative) Spring-Damper Torque** ($\vec{\tau}$) to steer its physical body toward the animated target bone orientation:

$$\vec{\tau} = k_p \cdot \vec{e}_{\theta} - k_d \cdot \vec{\omega}$$

where:
- $\vec{e}_{\theta}$ is the shortest-arc axis-angle error vector between the physical bone's current quaternion $\mathbf{q}_{\text{phys}}$ and the animated skeleton's target quaternion $\mathbf{q}_{\text{anim}}$ (derived from $\mathbf{q}_{\text{diff}} = \mathbf{q}_{\text{anim}} \cdot \mathbf{q}_{\text{phys}}^{-1}$).
- $k_p$ is the **Proportional Stiffness Gain** (muscular strength—high for torso/legs, lower for extremities).
- $k_d$ is the **Derivative Damping Gain** (prevents violent harmonic oscillation/vibrating).
- $\vec{\omega}$ is the physical bone's current `angular_velocity`.

```gdscript
# ==============================================================================
# ACTIVE RAGDOLL TORQUE MATCHING CONTROLLER (PD Quaternion Servo)
# ==============================================================================
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

    # 1. Compute target world orientation from the animated reference skeleton
    var anim_skel_pose: Transform3D = target_skeleton.get_bone_global_pose_no_override(_bone_id)
    var target_world_basis: Basis = (target_skeleton.global_transform * anim_skel_pose).basis
    
    var q_target: Quaternion = target_world_basis.get_rotation_quaternion()
    var q_current: Quaternion = state.transform.basis.get_rotation_quaternion()

    # 2. Compute difference quaternion: q_diff = q_target * inv(q_current)
    var q_diff: Quaternion = q_target * q_current.inverse()
    
    # Ensure shortest path across the 4D hypersphere (hemisphere check)
    if q_diff.w < 0.0:
        q_diff = -q_diff

    # 3. Extract rotation axis and angle error
    var axis: Vector3 = q_diff.get_axis()
    var angle: float = q_diff.get_angle()
    
    if not axis.is_normalized() or is_zero_approx(angle):
        return

    var angular_error: Vector3 = axis * angle

    # 4. Evaluate PD Controller: Torque = (Kp * error) - (Kd * angular_velocity)
    var raw_torque: Vector3 = (angular_error * stiffness_kp) - (state.angular_velocity * damping_kd)
    var final_torque: Vector3 = (raw_torque * muscular_strength_multiplier).limit_length(max_muscle_torque)

    state.apply_torque(final_torque)
```

---

