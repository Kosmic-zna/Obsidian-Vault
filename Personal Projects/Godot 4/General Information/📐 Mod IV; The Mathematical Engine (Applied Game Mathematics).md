# 📐 Module IV: The Mathematical Engine (Applied Game Mathematics)

> [!info] 📖 Lexicon & First Principles: Linear Algebra & Spatial Calculus
> Mathematics in game development is not abstract academic symbolism; it is the concrete geometric machinery governing space, orientation, perspective, and motion:
> - ⤷ **Scalar**: A single numeric magnitude lacking directional properties ($\alpha \in \mathbb{R}$, e.g., mass, temporal duration, speed).
> - ⤷ **Vector**: An ordered tuple representing both ==magnitude and spatial direction== in an $n$-dimensional Euclidean space ($\vec{v} \in \mathbb{R}^n$).
> - ⤷ **Unit Vector (Versor)**: A vector whose Euclidean length is normalised to exactly one ($\|\hat{v}\| = 1.0$). Represents pure directional intent.
> - ⤷ **Basis**: A minimal set of linearly independent vectors defining the coordinate axes of a reference frame. In Godot 3D, a $3 \times 3$ matrix representing local $X$, $Y$, and $Z$ orientations.
> - ⤷ **Orthogonal**: Meeting at a strictly perpendicular angle ($\theta = \frac{\pi}{2}\text{ rad} = 90^\circ$). The dot product between orthogonal vectors is zero ($\vec{u} \cdot \vec{v} = 0$).
> - ⤷ **Orthonormal**: A collection of vectors that are simultaneously **orthogonal** to each other and individually **normalised** to unit length ($\|\hat{u}\| = 1$).
> - ⤷ **Affine Transform**: A geometric mapping that preserves straight lines and parallelism while combining linear transformations (rotation, scaling, shear) with spatial translation.

---

## 4.1 Vector Algebra in $\mathbb{R}^2$ and $\mathbb{R}^3$

Vectors form the baseline language of computational physics and spatial queries. Treating vectors as mere bags of coordinates ($x, y, z$) obscures their geometric essence.

```
                           THE DISPLACEMENT VECTOR
                           =======================
                                     B (Target: [8, 7])
                                    /
                                   /  ▲  Displacement: d = B - A
                                  /  /                 d = [6, 4]
                                 /  /
                                /  /
                               /  /
    A (Origin: [2, 3]) ───────┘
```

### The Universal Rule: "Target Minus Origin"
To construct a vector directing an actor at position $\vec{A}$ toward a destination $\vec{B}$, the arithmetic order is invariant:
$$\vec{d} = \vec{B} - \vec{A}$$

Reversing the terms ($\vec{A} - \vec{B}$) produces the exact opposite vector pointing away from the target (the retreat or flee vector).

### Euclidean Norm & The Squared Distance Optimization
The magnitude (length) of a vector in 3D Euclidean space is derived via the Pythagorean theorem:
$$\|\vec{v}\| = \sqrt{v_x^2 + v_y^2 + v_z^2}$$

Calculating the square root ($\sqrt{\cdot}$) requires iterative CPU arithmetic (such as the binary digit-by-digit or Newton-Raphson approximation methods), which introduces notable latency when executed thousands of times per frame inside tight loops.

```
OPTIMISATION: AVOIDING THE SQUARE ROOT IN RADIUS QUERIES
=========================================================
Testing if an entity is within interaction radius R = 5.0 metres:

AMATEUR (Slow):   if origin.distance_to(target) < 5.0:       # Performs sqrt()
ARCHITECT (Fast): if origin.distance_squared_to(target) < 25.0: # Pure multiplication (5.0^2)
```

$$\text{If } \|\vec{A} - \vec{B}\| < R \iff \|\vec{A} - \vec{B}\|^2 < R^2$$

### Normalisation & The Diagonal Speed Exploit
Normalisation converts a non-zero vector into a unit vector ($\hat{v}$) pointing in the identical direction:
$$\hat{v} = \frac{\vec{v}}{\|\vec{v}\|} = \left[ \frac{v_x}{\sqrt{v_x^2 + v_y^2 + v_z^2}}, \frac{v_y}{\sqrt{v_x^2 + v_y^2 + v_z^2}}, \frac{v_z}{\sqrt{v_x^2 + v_y^2 + v_z^2}} \right]$$

> [!bug] 🚨 The Diagonal Movement Exploit
> When a player inputs concurrent digital signals for Forward ($+Y$ or $-Z$) and Right ($+X$), the raw vector is $\vec{v} = [1, 1]$.
> $$\|\vec{v}\| = \sqrt{1^2 + 1^2} = \sqrt{2} \approx 1.4142$$
> Without explicit normalisation, an actor moving at base speed $S = 10\text{ m/s}$ will traverse space at $14.14\text{ m/s}$ diagonally—a ==41.4% unearned velocity boost== that completely destabilises gameplay balance and physics pacing.

```gdscript
# PRODUCTION IMPLEMENTATION: Clamped Vector Normalisation
func compute_movement_input() -> Vector2:
    # get_vector automatically deadzones and clamps magnitude to a unit circle (1.0)
    var input: Vector2 = Input.get_vector(
        &"move_left", 
        &"move_right", 
        &"move_forward", 
        &"move_backward"
    )
    return input # Guaranteed: ||input|| <= 1.0
```

---

## 4.2 The Dot Product ($\vec{a} \cdot \vec{b}$): The Similarity Detector

The algebraic dot product (scalar product) of two vectors collapses multi-dimensional coordinates into a single scalar value:
$$\vec{a} \cdot \vec{b} = \sum_{i=1}^{n} a_i b_i = a_x b_x + a_y b_y + a_z b_z$$

The geometric definition links the dot product directly to the cosine of the angle $\theta$ between the vectors:
$$\vec{a} \cdot \vec{b} = \|\vec{a}\| \|\vec{b}\| \cos(\theta)$$

When evaluating normalised unit vectors ($\|\hat{a}\| = \|\hat{b}\| = 1.0$), the magnitude terms vanish:
$$\hat{a} \cdot \hat{b} = \cos(\theta) \quad \implies \quad \theta = \arccos(\hat{a} \cdot \hat{b})$$

```
                   THE DOT PRODUCT ALIGNMENT GAUGE
=====================================================================
    a · b = +1.0             a · b = 0.0             a · b = -1.0
  (Exact Parallel)       (Strictly Orthogonal)      (Direct Opposition)
   θ = 0 rad (0°)          θ = π/2 rad (90°)          θ = π rad (180°)

         ▲                       ▲                        ▲
         │                       │                        │
       a │ b                   a │                      a │
         │                       │                        │
         ▼                       └──────►                 ▼
                                    b                     b (Opposite)
```

```
NUMERICAL VALUE SPECTRUM (for unit vectors):
[ +1.0 ] ──> Pointing in identical direction (θ = 0°)
[ > 0.0] ──> Forming an acute angle (θ < 90°, target is in front)
[   0.0] ──> Strictly perpendicular / orthogonal (θ = 90°)
[ < 0.0] ──> Forming an obtuse angle (θ > 90°, target is behind)
[ -1.0 ] ──> Pointing in exact opposite direction (θ = 180°)
```

### Applied Gameplay Case Studies

#### 1. Field of View (Vision Cone) Sensor

```gdscript
## Returns true if the target is within the observer's viewing cone
func is_target_in_fov(
    observer_origin: Vector3, 
    observer_forward: Vector3, 
    target_position: Vector3, 
    fov_degrees: float
) -> bool:
    var to_target: Vector3 = (target_position - observer_origin).normalized()
    var forward: Vector3 = observer_forward.normalized()
    
    var dot: float = forward.dot(to_target)
    # Convert half-angle threshold to cosine space
    var threshold_cos: float = cos(deg_to_rad(fov_degrees * 0.5))
    
    # In cosine space, a smaller angle yields a HIGHER value
    return dot >= threshold_cos
```

#### 2. Critical Backstab Detection
Backstab logic requires two distinct criteria:
1. The attacker is positioned **behind** the victim's back plane.
2. The attacker is facing the **same general direction** as the victim.

```gdscript
func evaluate_backstab(attacker: Node3D, victim: Node3D) -> bool:
    var victim_forward: Vector3 = -victim.global_transform.basis.z.normalized()
    var attacker_forward: Vector3 = -attacker.global_transform.basis.z.normalized()
    var to_attacker: Vector3 = (attacker.global_position - victim.global_position).normalized()
    
    # 1. Is the attacker physically behind the victim?
    # Behind means the vector from victim to attacker points backwards relative to victim_forward
    var is_behind: bool = victim_forward.dot(to_attacker) < -0.3 # ~107° to 180°
    
    # 2. Are both actors looking in roughly the same direction?
    var is_facing_same_way: bool = victim_forward.dot(attacker_forward) > 0.7 # θ < ~45°
    
    return is_behind and is_facing_same_way
```

#### 3. Vector Projection and Surface Sliding
When a kinematic body impacts a surface normal $\hat{n}$, its forward velocity $\vec{v}$ must be projected onto the plane to slide smoothly without sticking:
$$\vec{v}_{\parallel} = \text{proj}_{\hat{n}}(\vec{v}) = (\vec{v} \cdot \hat{n}) \hat{n}$$
$$\vec{v}_{\text{slide}} = \vec{v} - \vec{v}_{\parallel} = \vec{v} - (\vec{v} \cdot \hat{n}) \hat{n}$$

```gdscript
## Computes slide velocity along a collision surface without using engine helpers
func calculate_slide(current_velocity: Vector3, surface_normal: Vector3) -> Vector3:
    var normal: Vector3 = surface_normal.normalized()
    var normal_component: float = current_velocity.dot(normal)
    # Strip away the velocity component pushing into or pulling off the surface
    return current_velocity - (normal * normal_component)
```

---

## 4.3 The Cross Product ($\vec{a} \times \vec{b}$): The Perpendicular Generator

The vector cross product operates strictly in $\mathbb{R}^3$ (or returns an orthogonal pseudoscalar in $\mathbb{R}^2$). Given two non-collinear vectors $\vec{a}$ and $\vec{b}$, the cross product produces a third vector $\vec{c}$ that is **strictly perpendicular to both**:
$$\vec{c} = \vec{a} \times \vec{b} = \begin{bmatrix} a_y b_z - a_z b_y \\ a_z b_x - a_x b_z \\ a_x b_y - a_y b_x \end{bmatrix}$$

The magnitude of the cross product matches the area of the parallelogram spanned by the two vectors:
$$\|\vec{a} \times \vec{b}\| = \|\vec{a}\| \|\vec{b}\| \sin(\theta)$$

```
                     THE RIGHT-HAND RULE IN 3D SPACE
                     ===============================
                                 ▲ c = a x b (Upward Normal)
                                 │
                                 │
                                 │
                                 ├──────► b (Index Finger)
                                /
                               /
                              ▼ a (Thumb)
```

### Critical Cross Product Invariants:
1. **Anti-Commutative**: $\vec{a} \times \vec{b} = -(\vec{b} \times \vec{a})$. Inverting the input operand order completely reverses the resultant vector's direction!
2. **Parallel Vectors**: If $\vec{a}$ and $\vec{b}$ are collinear ($\theta = 0$ or $\pi$), $\sin(\theta) = 0$, resulting in the zero vector: $\vec{a} \times \vec{b} = \vec{0}$.
3. **Right-Handed Coordinate System**: Point your index finger along $\vec{a}$, curl your middle finger toward $\vec{b}$, and your thumb points toward $\vec{a} \times \vec{b}$.

### The 2D "Perp-Dot" Product (Signed Curvature & Left/Right Steering)
In 2D, the cross product collapses to a scalar determinant representing the signed area:
$$\text{cross}_{2D}(\vec{a}, \vec{b}) = a_x b_y - a_y b_x$$

- If $\text{cross}_{2D}(\vec{a}, \vec{b}) > 0$: $\vec{b}$ is rotated **counter-clockwise (to the left)** of $\vec{a}$.
- If $\text{cross}_{2D}(\vec{a}, \vec{b}) < 0$: $\vec{b}$ is rotated **clockwise (to the right)** of $\vec{a}$.
- If $\text{cross}_{2D}(\vec{a}, \vec{b}) = 0$: $\vec{a}$ and $\vec{b}$ are **strictly collinear**.

```gdscript
## Computes steering torque to steer towards a target point in 2D
func get_steering_direction(ship_transform: Transform2D, target_pos: Vector2) -> float:
    # In Godot 2D, transform.x is the local forward vector
    var forward: Vector2 = ship_transform.x.normalized()
    var to_target: Vector2 = (target_pos - ship_transform.origin).normalized()
    
    # 2D cross product returns the signed sine of the relative angle
    var signed_turn: float = forward.cross(to_target)
    return signf(signed_turn) # +1.0 = Turn Left, -1.0 = Turn Right, 0.0 = Straight
```

### Constructing an Orthonormal Basis from a Single Normal
When procedural surface geometry or raycast collisions return an arbitrary ground normal $\hat{n}$, you must construct a complete 3D coordinate system (Basis) aligned to that slope:

```gdscript
## Derives an orthonormal 3x3 Basis aligned to an arbitrary surface normal
func construct_basis_from_normal(surface_normal: Vector3) -> Basis:
    var up: Vector3 = surface_normal.normalized()
    
    # Select an arbitrary reference vector not parallel to 'up'
    var arbitrary: Vector3 = Vector3.UP
    if absf(up.dot(arbitrary)) > 0.99:
        # Near collinearity detected: switch reference to avoid zero cross product
        arbitrary = Vector3.RIGHT
        
    var right: Vector3 = arbitrary.cross(up).normalized()
    var forward: Vector3 = up.cross(right).normalized()
    
    # Columns define local X (Right), Y (Up), and Z (Back)
    return Basis(right, up, forward)
```

---

## 4.4 Matrices, Coordinate Spaces & Transforms

Every spatial node in Godot exists within a nested chain of coordinate systems. Understanding matrix transformations is necessary to translate positions, velocities, and raycasts across local and global spaces.

```
                           COORDINATE SPACE CONTINUUM
=================================================================================
 [ Local Model Space ]   Vertex coordinates relative to the object's pivot center
          │
          ▼  * Node.transform (Translation, Rotation, Scale)
 [ Global World Space ]  Absolute coordinates inside the level simulation
          │
          ▼  * Camera.transform.affine_inverse()
 [ View / Camera Space ] Space centered at the lens; +Z points toward the player
          │
          ▼  * Projection Matrix (Perspective FoV / Orthographic)
 [ Clip / Screen Space ] Normalised Device Coordinates (NDC) mapped to monitor pixels
```

### The Anatomy of `Transform3D` and `Basis`

In Godot 4, a `Transform3D` is represented by a $4 \times 3$ affine transformation matrix, internally split into a $3 \times 3$ `Basis` matrix and a `Vector3` `origin`:

$$\mathbf{T} = \left[ \begin{array}{ccc|c} 
B_{xx} & B_{yx} & B_{zx} & O_x \\
B_{xy} & B_{yy} & B_{zy} & O_y \\
B_{xz} & B_{yz} & B_{zz} & O_z
\end{array} \right] = \left[ \begin{array}{ccc|c} 
\vec{X}_{\text{right}} & \vec{Y}_{\text{up}} & \vec{Z}_{\text{back}} & \vec{O}_{\text{origin}}
\end{array} \right]$$

```
IMPORTANT: GODOT'S FORWARD DIRECTION
====================================
Godot uses a Right-Handed Cartesian Coordinate System where:
- Basis.x = Local +X axis (Points to the character's RIGHT)
- Basis.y = Local +Y axis (Points to the character's UP)
- Basis.z = Local +Z axis (Points BEHIND the character!)
==LOCAL FORWARD is strictly Vector3.FORWARD = Vector3(0, 0, -1) (-Basis.z)==
```

### Coordinate Conversion: Local vs. Global

Converting a point from local coordinates $\vec{p}_{\text{local}}$ to global coordinates $\vec{p}_{\text{global}}$ requires matrix-vector multiplication followed by origin translation:
$$\vec{p}_{\text{global}} = \mathbf{B} \cdot \vec{p}_{\text{local}} + \vec{o}$$

In reverse, moving from global to local space requires subtracting the origin and multiplying by the **inverse** of the Basis matrix:
$$\vec{p}_{\text{local}} = \mathbf{B}^{-1} \cdot (\vec{p}_{\text{global}} - \vec{o})$$

If the Basis is purely orthonormal (containing only rotation with uniform scale of $1.0$), the matrix inverse equals its **transpose** ($\mathbf{B}^{-1} = \mathbf{B}^T$), which costs almost zero CPU cycles to compute.

```gdscript
# NATIVE TRANSFORMATION UTILITIES
var local_muzzle_pos: Vector3 = Vector3(0.5, 0.2, -1.2) # Relative to weapon root

# Convert local offset to exact global world coordinate:
var world_muzzle_pos: Vector3 = weapon.global_transform * local_muzzle_pos
# Equivalent to: weapon.to_global(local_muzzle_pos)

# Convert world target point into local weapon coordinates:
var target_in_local_space: Vector3 = weapon.global_transform.affine_inverse() * world_target_pos
# Equivalent to: weapon.to_local(world_target_pos)
```

> [!warning] ⚠️ Edge Case Trap: Transform Scaling & Orthogonality Drift
> Scaling a node non-uniformly (`scale = Vector3(1.0, 2.0, 1.0)`) scales the basis vectors, making them non-unit length ($\|\text{basis.y}\| = 2.0$). Repeatedly rotating and scaling transforms at runtime introduces floating-point error accumulation, causing the axes to lose orthogonality (shear drift).
> To cleanse drift, call: `transform.basis = transform.basis.orthonormalized()`

---

## 4.5 Rotations: Trigonometry, Radians & Quaternions

Handling rotations using naive spherical angles creates mathematical singularities.

### Radians vs. Degrees
CPUs compute trigonometric Taylor series natively using **radians**:
$$\text{rad} = \text{deg} \cdot \left(\frac{\pi}{180^\circ}\right) \qquad \text{deg} = \text{rad} \cdot \left(\frac{180^\circ}{\pi}\right)$$
- $360^\circ = 2\pi\text{ rad} \approx 6.28318$
- $180^\circ = \pi\text{ rad} \approx 3.14159$
- $90^\circ = \frac{\pi}{2}\text{ rad} \approx 1.57079$

### The Arc-Tangent Singularity: Use `atan2` Exclusively
Given a 2D direction vector $(x, y)$, computing its planar angle via standard $\tan^{-1}\left(\frac{y}{x}\right)$ fails catastrophically whenever $x = 0$ (division by zero) and cannot distinguish Quadrant I ($+x, +y$) from Quadrant III ($-x, -y$).

Always use `atan2(y, x)`:
$$\theta = \text{atan2}(y, x) \quad \text{where } \theta \in (-\pi, \pi]$$

```gdscript
# Correctly obtaining rotation angle from an analogue joystick:
var angle: float = atan2(input_vector.y, input_vector.x)
```

### Euler Angles and Gimbal Lock

Euler rotations define orientation by rotating sequentially around three fixed axes: **Yaw** ($Y$), **Pitch** ($X$), and **Roll** ($Z$).

```
                    THE MECHANICS OF GIMBAL LOCK
=====================================================================
Pitch rotated to strictly ±90° (Looking straight UP or DOWN):
- The Yaw axis and Roll axis become collinear!
- Rotation around Yaw produces the identical mathematical result as Roll.
- ==A degree of freedom is completely destroyed.==
Result: Wild, uncontrollable camera spinning and mathematical singularities.
```

### Quaternions (`Quaternion`): 4D Rotation Representation

To bypass Gimbal Lock and interpolate rotations cleanly, game engines employ **Quaternions**—hypercomplex four-dimensional numbers:
$$\mathbf{q} = \begin{bmatrix} q_x \\ q_y \\ q_z \\ q_w \end{bmatrix} = \left[ \vec{u} \cdot \sin\left(\frac{\theta}{2}\right), \; \cos\left(\frac{\theta}{2}\right) \right]$$

where $\vec{u}$ is a 3D unit axis of rotation, and $\theta$ is the angle of rotation around that axis.

#### Core Properties of Quaternions:
1. **Unit Length Constraint**: All rotation quaternions must satisfy $\|\mathbf{q}\| = \sqrt{x^2 + y^2 + z^2 + w^2} = 1.0$.
2. **Double Covering**: $\mathbf{q}$ and $-\mathbf{q}$ represent the **exact same physical 3D orientation**, but traverse opposite geodesic paths around the 4D hypersphere.
3. **Spherical Linear Interpolation (`slerp`)**: Interpolates between two orientations along the shortest spherical arc with strictly constant angular velocity:
$$\text{slerp}(\mathbf{q}_1, \mathbf{q}_2, t) = \frac{\sin((1 - t)\Omega)}{\sin(\Omega)}\mathbf{q}_1 + \frac{\sin(t\Omega)}{\sin(\Omega)}\mathbf{q}_2 \quad \text{where } \cos(\Omega) = \mathbf{q}_1 \cdot \mathbf{q}_2$$

```gdscript
## Smoothly interpolates an actor's orientation toward a target rotation
func rotate_towards_target(actor: Node3D, target_transform: Transform3D, delta: float) -> void:
    var current_quat: Quaternion = actor.global_transform.basis.get_rotation_quaternion()
    var target_quat: Quaternion = target_transform.basis.get_rotation_quaternion()
    
    # Slerp smoothing with frame-rate independent decay constant (lambda = 10.0)
    var decay_weight: float = 1.0 - exp(-10.0 * delta)
    var smoothed_quat: Quaternion = current_quat.slerp(target_quat, decay_weight)
    
    # Re-project quaternion into basis matrix
    actor.global_transform.basis = Basis(smoothed_quat)
```

---

## 4.6 Interpolation, Curves & Calculus of Motion

Interpolation smooths values between discrete points over time. Using the incorrect equation creates severe bugs when display refresh rates change.

### Linear Interpolation (`lerp`)
$$\text{lerp}(a, b, t) = a + (b - a) \cdot t \quad \text{where } t \in [0, 1]$$

### The Frame-Rate Independence Proof: The Broken `lerp` Fallacy

> [!bug] 🚨 The Common "Broken Lerp" Anti-Pattern
> Thousands of published games contain this flawed line inside their frame loops:
> `current_val = lerp(current_val, target_val, 0.1 * delta)`
> This code is **mathematically flawed**. Because `current_val` updates iteratively every frame, `delta` scales linearly inside an exponential process, meaning the smoothing speed changes across different refresh rates ($30\text{ FPS}$, $60\text{ FPS}$, and $144\text{ FPS}$).

```
PROVING THE ERROR OF NAIVE LERP OVER VARIABLE DELTA
===================================================
Target = 100.0, Initial = 0.0, Smooth factor = 0.5 per second.

Simulation across 1.0 Second of real time:
- System A (Running at 1 FPS, dt = 1.0s):
  x_1 = lerp(0, 100, 0.5 * 1.0) = 50.0

- System B (Running at 2 FPS, dt = 0.5s):
  Step 1: x_0.5 = lerp(0, 100, 0.5 * 0.5) = 25.0
  Step 2: x_1.0 = lerp(25, 100, 0.5 * 0.5) = 25 + 75 * 0.25 = 43.75

50.0 != 43.75!
The two machines arrive at different states over the identical duration!
On a 144Hz monitor, the camera or object moves at an entirely different rate!
```

#### The Mathematical Derivation of Frame-Rate Independent Exponential Decay

We model iterative approach toward a target as a continuous first-order differential equation:
$$\frac{dx}{dt} = -\lambda(x(t) - x_{\text{target}})$$

Separating variables and integrating across elapsed timestep $\Delta t$:
$$\int_{x(t)}^{x(t + \Delta t)} \frac{1}{x - x_{\text{target}}} \, dx = -\lambda \int_{0}^{\Delta t} dt$$
$$\ln\left( \frac{x(t + \Delta t) - x_{\text{target}}}{x(t) - x_{\text{target}}} \right) = -\lambda \Delta t$$
$$x(t + \Delta t) - x_{\text{target}} = (x(t) - x_{\text{target}}) \cdot e^{-\lambda \Delta t}$$
$$x(t + \Delta t) = x_{\text{target}} + (x(t) - x_{\text{target}}) \cdot e^{-\lambda \Delta t}$$

Rearranging into linear interpolation structure ($a + (b - a) \cdot w$):
$$x(t + \Delta t) = \text{lerp}\left(x(t), \; x_{\text{target}}, \; 1.0 - e^{-\lambda \Delta t}\right)$$

where $\lambda > 0$ is the **decay rate constant** (half-life velocity).

```gdscript
## Mathematically exact frame-rate independent exponential smoothing
## Higher decay_rate = snappier convergence (e.g., 5.0 to 25.0)
static func exp_decay_float(current: float, target: float, decay_rate: float, delta: float) -> float:
    return lerp(current, target, 1.0 - exp(-decay_rate * delta))

static func exp_decay_vec3(current: Vector3, target: Vector3, decay_rate: float, delta: float) -> Vector3:
    return current.lerp(target, 1.0 - exp(-decay_rate * delta))
```

### Bézier Curves & Splines

Bézier curves interpolate smoothly across multiple control points. They are commonly used for dynamic camera rails, ballistic mortar arcs, and homing paths.

```
                  CUBIC BÉZIER CURVE GEOMETRY
================================================================
              P1 (Control 1)            P2 (Control 2)
                 o                         o
                /                           \
               /                             \
              /                               \
    P0 o─────┘                                 └─────o P3
  (Start)                                             (End)
```

1. **Quadratic Bézier** ($3$ points: $P_0, P_1, P_2$):
   $$B(t) = (1 - t)^2 P_0 + 2(1 - t)t P_1 + t^2 P_2 \quad \text{where } t \in [0, 1]$$

2. **Cubic Bézier** ($4$ points: $P_0, P_1, P_2, P_3$):
   $$B(t) = (1 - t)^3 P_0 + 3(1 - t)^2 t P_1 + 3(1 - t)t^2 P_2 + t^3 P_3$$

```gdscript
## Computes a cubic Bézier curve position at progress parameter t
static func evaluate_cubic_bezier(p0: Vector3, p1: Vector3, p2: Vector3, p3: Vector3, t: float) -> Vector3:
    var u: float = 1.0 - t
    var tt: float = t * t
    var uu: float = u * u
    var uuu: float = uu * u
    var ttt: float = tt * t
    
    var point: Vector3 = (uuu * p0) + (3.0 * uu * t * p1) + (3.0 * u * tt * p2) + (ttt * p3)
    return point
```

---

## 4.7 Stochastic Systems & Procedural Coherent Noise

True randomness does not exist in deterministic computing; engines rely on **Pseudo-Random Number Generators (PRNG)**.

```
                    WHITE NOISE vs. COHERENT NOISE
=====================================================================
White Noise (randf):                   Coherent Noise (FastNoiseLite):
Value                                  Value
  1.0 |   |  |   |   |   |               1.0 |       .---.
      | | | || | | | | | |                   |      /     \     .-.
  0.5 | | | || | | | | | |               0.5 | .--'       '---'   \
      | ||| ||||||||||||||                   |/                    \
  0.0 └───┴──┴─┴──┴──┴──┴──► Time        0.0 └──────────────────────► Time
  Completely decorrelated adjacent ticks     Continuous, organic gradients
```

### Gaussian (Normal) Distribution via Box-Muller Transform
Standard `randf_range()` produces a uniform flat distribution: every number in the range has an equal probability of being chosen. Real-world physical traits (weapon inaccuracy spread, entity heights, drop chances) cluster around a central mean ($\mu$) with variance governed by standard deviation ($\sigma$).

The **Box-Muller transform** turns two uniform random numbers $U_1, U_2 \in (0, 1)$ into a standard normal distribution:
$$Z = \sqrt{-2 \ln(U_1)} \cdot \cos(2\pi U_2)$$
$$\text{Sample} = \mu + Z \cdot \sigma$$

```gdscript
## Generates a random float conforming to a Gaussian (bell curve) distribution
static func randf_gaussian(mean: float, std_dev: float) -> float:
    # Ensure uniform samples avoid zero to prevent ln(0) errors
    var u1: float = maxf(0.00001, randf())
    var u2: float = randf()
    
    var z0: float = sqrt(-2.0 * log(u1)) * cos(TAU * u2)
    return mean + (z0 * std_dev)
```

---

## 4.8 Applied Mathematical System: Kinematic Ballistic Predictor

To synthesize **Vector Norms, Projectile Integration, Spline Tracing, and Spatial Raycasting**, consider this complete **Ballistic Trajectory Predictor**. 

Given a muzzle origin, fire impulse velocity, gravitational acceleration, and aerodynamic drag, it calculates and renders an explicit trajectory arc, checking for physics impacts along the path:

```
                          BALLISTIC TRAJECTORY PREDICTION
===================================================================================
     Muzzle Origin
          o
           \
            \  v0
             '.
               \
                '.
                  \
                   '.
                     \     Discrete Ray Intersect Query
                      '.                 |
                        \                v
                         ' . _           [ Obstacle ]
                               ' ─ ─ ─ ─ ─ ─ x Collision Impact Point
```

```gdscript
# ==============================================================================
# BALLISTIC TRAJECTORY PREDICTOR (Analytic & Numerical Integration)
# ==============================================================================
class_name BallisticPredictor
extends RefCounted

## Computes an array of sampled points along a ballistic flight path
## Stops early if an intervening obstacle is detected
static func calculate_trajectory(
    world_3d: World3D,
    start_pos: Vector3, 
    initial_velocity: Vector3, 
    gravity_vector: Vector3, 
    time_step: float = 0.05, 
    max_steps: int = 60,
    collision_mask: int = 1
) -> PackedVector3Array:
    
    var points: PackedVector3Array = PackedVector3Array()
    points.append(start_pos)
    
    var space_state: PhysicsDirectSpaceState3D = world_3d.direct_space_state
    var current_pos: Vector3 = start_pos
    var current_vel: Vector3 = initial_velocity
    
    for step in range(max_steps):
        # Explicit Euler numerical integration:
        # v_{n+1} = v_n + g * dt
        # p_{n+1} = p_n + v_{n+1} * dt
        var next_vel: Vector3 = current_vel + (gravity_vector * time_step)
        var next_pos: Vector3 = current_pos + (next_vel * time_step)
        
        # Spatial raycast query between segment intervals
        var query: PhysicsRayQueryParameters3D = PhysicsRayQueryParameters3D.create(
            current_pos, 
            next_pos, 
            collision_mask
        )
        
        var hit: Dictionary = space_state.intersect_ray(query)
        if not hit.is_empty():
            # Collision registered: terminate path at the precise hit position
            points.append(hit["position"])
            break
            
        points.append(next_pos)
        current_pos = next_pos
        current_vel = next_vel
        
    return points

## Analytically calculates the required launch pitch angle to strike a target
## Returns -1.0 if the target is out of ballistic range
static func solve_launch_angle(
    muzzle_pos: Vector3, 
    target_pos: Vector3, 
    muzzle_speed: float, 
    gravity: float, 
    prefer_high_arc: bool = false
) -> float:
    var diff: Vector3 = target_pos - muzzle_pos
    var planar_diff: Vector2 = Vector2(diff.x, diff.z)
    var x: float = planar_diff.length()
    var y: float = diff.y
    var v: float = muzzle_speed
    var g: float = gravity
    
    var v2: float = v * v
    var v4: float = v2 * v2
    
    # Radical equation of ballistic projectile trajectory:
    # v^4 - g * (g * x^2 + 2 * y * v^2)
    var radical: float = v4 - g * (g * (x * x) + (2.0 * y * v2))
    
    if radical < 0.0:
        # Target out of physical range given launch speed and gravity
        return -1.0
        
    var root: float = sqrt(radical)
    var angle_low: float = atan2(v2 - root, g * x)
    var angle_high: float = atan2(v2 + root, g * x)
    
    return angle_high if prefer_high_arc else angle_low
```

---

