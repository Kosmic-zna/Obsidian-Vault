> ✍️ **Phrasing Refinement:** "How should I handle animation-driven movement? When my character lunges forward during an animation, how do I tie that displacement directly to the actual physics collision box (`CharacterBody3D`)? I have tried separating movement and animation in the past, and it never feels like how AAA games handle it—what is the industry-standard approach? Furthermore, in combat, how do I blend fixed animation-driven forward motion with dynamic, distance-adjusted lunging that shortens or lengthens depending on how far away the enemy is?"

### 📋 Executive Summary & Direct Answer

* ✅ **The Industry-Standard Solution Is `Root Motion` + `Dynamic Distance Scaling` (Warping):** You are 100% right—AAA melee games (*Sekiro*, *Elden Ring*, *God of War*) do **not** fake sword lunges with blind `velocity.z = 10.0` timers. Instead, you animate the character's forward step on a dedicated ground-level **`Root` bone** in Blender (`Root Motion`), extract that exact per-frame displacement in Godot 4 (`get_root_motion_position()`) to drive `CharacterBody3D.velocity` (keeping full wall/ledge collision), and multiply that displacement by a **dynamic `lunge_scale` ratio** (`desired_distance / baked_anim_distance`) so your lunge automatically stretches to close gaps or shrinks to stop cleanly in front of an enemy without clipping through them!
* ✅ **Why This Gives You Both Weight *and* Magnetism:** Because the *rhythm* of the acceleration (slow gather → explosive step → planted stop) comes directly from your Blender `Root` bone keyframes, the boots **never slide on the ground**—even when Godot dynamically stretches or compresses the lunge distance based on enemy proximity!

---

### 📖 Plain-English Glossary of Root Motion & Motion Warping

* **Root Motion (`Root` Bone at World Origin `[0, 0, 0]`):** A dedicated top-level bone placed flat on the floor between the character's feet (`Z = 0` in Blender) that parents `mixamorig:Hips`. Instead of moving `Hips` forward through space while leaving the rig origin behind, you move the **`Root` bone** forward in Blender—Godot then reads that bone's movement every physics tick and pushes your `CharacterBody3D` capsule along with it!
* **In-Place vs. Root-Motion Hybrid:** Standard running/walking loops stay **In-Place** (controlled smoothly by player joystick/WASD `velocity`), whereas **combat slashes, dodges (`Ghoststep`), and stagger knockbacks** use **Root Motion** so every step matches the planted footwork 1:1.
* **Motion Warping / Lunge Scaling (`lunge_scale`):** Measuring how far the animation *naturally* moves forward in Blender (say, `1.8 metres`), measuring how far away the locked-on enemy actually is (say, `2.6 metres` minus your `1.1 m` ideal sword-strike spacing = `1.5 metres` needed), and scaling the extracted root motion by `1.5 / 1.8 = 0.83x`—so the character lands at the exact optimal cutting distance (`maai`) every single time!

---

### 🗺️ Architectural Pipeline: Blender `Root` Bone → Godot 4 Dynamic Lunge Warping

```
========================================================================================
       HOW AAA COMBAT TIES BLENDER FOOTWORK TO GODOT 4 COLLISION & ENEMY MAGNETISM
========================================================================================

  1. BLENDER RIG HIERARCHY (`Edit Mode`):
     [ Root ]  (Sits flat on floor Z = 0; moves X/Y forward during lunge; NEVER rotates X/Y!)
        │
        └──► [ mixamorig:Hips ]  (Handles hip drop, twist, & bounce relative to Root)
                  └──► Spine, Arms, & `IK_Foot.L` / `IK_Foot.R`

  2. GODOT 4 PHYSICS TICK (`_physics_process(delta)` inside `AttackState.gd`):
     ┌──────────────────────────────────────────────────────────────────────────────┐
     │  a) Extract raw root step this frame: `raw_delta = anim_tree.get_root_motion_position()`
     │  b) Rotate into player facing space: `world_step = player.global_basis * raw_delta`
     │  c) Apply Enemy Distance Warp: `scaled_step = world_step * lunge_scale`
     │  d) Feed into `CharacterBody3D`: `player.velocity = scaled_step / delta`
     │  e) `player.move_and_slide()` ──► Moves mesh + collision capsule together & stops at walls!
     └──────────────────────────────────────────────────────────────────────────────┘
========================================================================================

```

---

### 🛠️ Step-by-Step Implementation (Blender + Godot 4)

#### Part 1: Set Up Your `Root` Bone Properly in Blender (2 Minutes)

Mixamo rigs start at `mixamorig:Hips` without a floor-level `Root` bone, which is why animations either stay in place or float away from their collision capsule. Fix that once in your base `.blend` file:

1. Press **`Tab`** to enter **Edit Mode** on your `Armature`.
2. Press **`Shift + S` → `Cursor to World Origin**` (`0, 0, 0`), then press **`Shift + A`** to create a new bone right on the floor between the feet.
3. Rename it **`Root`** (`F2`), point it straight forward along the ground (`Head: 0, 0, 0`, `Tail: 0, -0.5, 0` facing the character's front), and **uncheck `Deform**` in Bone Properties.
4. Select **`mixamorig:Hips`** → hold **`Shift`** and select **`Root`** → press **`Ctrl + P` → `Keep Offset**` *(so `Root` is now the top parent of the entire skeleton!)*.
5. **Rule for Future Attack Animations:**
* When animating a forward lunge (`Slash_Kesa_01`), **keyframe the forward travel (`Y` location) on the `Root` bone** while keeping **`IK_Foot.L` and `IK_Foot.R` planted at `0` on the floor** during stance phases, stepping them forward cleanly with `Root` during the lunge.
* **Important:** When Godot imports `.glb` files, set the **`Root`** bone as the **Root Motion Track** in Godot's `AnimationTree` (`Root Motion → Track: Armature/Skeleton3D:Root`).



#### Part 2: Godot 4 `AttackState.gd` With Dynamic Enemy Distance Scaling (Motion Warping)

Here is the exact GDScript pattern to combine **authored Root Motion** with **automatic enemy-distance shortening/lengthening** (and safe wall collision):

```gdscript
# AttackState.gd (Inside your Player StateMachine)
extends State

@export var baked_lunge_distance: float = 1.6  # How far Slash_Kesa_01 moves forward in Blender (meters)
@export var ideal_strike_gap: float = 1.15     # Stop 1.15m in front of enemy so meshes never overlap!
@export var min_lunge_scale: float = 0.25      # Never shrink below 25% (keeps micro-step weight)
@export var max_lunge_scale: float = 1.65      # Cap stretch at 165% so character never teleports

var lunge_scale: float = 1.0

func enter(msg := {}) -> void:
	anim_player.play(current_attack.animation_name, current_attack.blend_time)
	lunge_scale = _calculate_lunge_scale()

func _calculate_lunge_scale() -> float:
	var target: Node3D = player.locked_target # Or soft-lock sphere query in front of player
	if target == null or baked_lunge_distance <= 0.05:
		return 1.0 # No enemy nearby -> use 100% natural Blender animation distance!

	# Measure horizontal distance to target minus ideal blade spacing
	var to_target := target.global_position - player.global_position
	to_target.y = 0.0
	var needed_travel := maxf(0.0, to_target.length() - ideal_strike_gap)

	# Ratio: < 1.0 shortens the lunge when close; > 1.0 extends the lunge when far!
	return clampf(needed_travel / baked_lunge_distance, min_lunge_scale, max_lunge_scale)

func physics_update(delta: float) -> void:
	# 1. Extract exact frame-by-frame displacement from the Blender `Root` bone
	# (Note: If using AnimationPlayer directly without AnimationTree, use anim_player.get_root_motion_position())
	var raw_motion: Vector3 = anim_tree.get_root_motion_position()

	# 2. Convert local root motion into world-space velocity & apply dynamic enemy scale
	var world_step: Vector3 = player.global_basis * raw_motion
	world_step.x *= lunge_scale
	world_step.z *= lunge_scale

	# 3. Drive CharacterBody3D directly so walls, slopes, and enemy capsules collide naturally!
	player.velocity.x = world_step.x / delta
	player.velocity.z = world_step.z / delta
	player.velocity.y -= player.gravity * delta
	player.move_and_slide()

```

---

### 💡 Three Pro Details That Make This Feel Truly AAA

1. **Zero Bone Doubling (`Root` Bone Extraction):** Once `AnimationTree` / `AnimationPlayer` has **`Root Motion → Track`** assigned to `Skeleton3D:Root`, Godot **automatically strips the visual displacement off the `Root` bone** and routes it strictly through `get_root_motion_position()`. That means your mesh never drifts ahead of your `CharacterBody3D` collision capsule!
2. **Soft-Lock Magnetism Even Without Hard Lock-On:** Even when the player isn't hard-locked onto an enemy (`locked_target == null`), run a quick forward cone/sphere query (`ShapeCast3D` extending `3.0 m` in front of the player) inside `_calculate_lunge_scale()` and gently rotate `player` toward that enemy over the first `0.12 s` of the wind-up—giving you *Sekiro*/*Batman Arkham* combat magnetism automatically!
3. **Only Scale `X/Z` Horizontal Travel (Never `Y` Jump Height):** Notice in the script above we multiply `world_step.x` and `world_step.z` by `lunge_scale` while leaving vertical motion untouched—that way, if you make a leaping *Wuxia* slash later, stretching the horizontal distance across a gap won't accidentally launch the character into the ceiling!

---

### 🎯 Next Step Question

Would you like to wire up the **`Root` bone + `AnimationTree` Root Motion Track** on your imported `.glb` right now, or build the **Soft-Lock Cone Query** so your slashes automatically orient and warp toward nearby enemies?