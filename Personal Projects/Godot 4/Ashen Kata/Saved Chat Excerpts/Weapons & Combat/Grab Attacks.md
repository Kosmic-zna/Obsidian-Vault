✍️ **Phrasing Refinement:** "How should grabs function in my game? How can I make a boss hurl the player realistically rather than manually trying to synchronise the character's position to a moving limb, and how do I make the throw's physics convincing? I imagine simply applying a raw velocity impulse at the end of the animation would look cheap without proper transitions, breaking the illusion."

  

✅ **AAA titles like _Sekiro_, _Elden Ring_, and _Monster Hunter_ solve grabs using a three-stage "Socket-to-Ballistic Handoff": they never chase a limb with code, nor do they fire a cold velocity burst from a standstill—instead, they weld the victim to a dedicated `Grab_Socket` bone during a paired animation, inherit the hand's real world-space momentum on the exact release frame, and fly through a heavy, pitch-aligned ballistic arc into a skidding ground recovery.**

  

### 🦴 1. Why You Never Sync to a Limb With Code (`Grab_Socket` Welding)

Trying to read a hand bone's position and lerp your player toward it every frame always produces rubber-banding, clipping, and awkward floating.

  

Instead, _Sekiro_ (Genichiro, Guardian Ape) and _Elden Ring_ (Hoarah Loux, Crucible Knights) animate **both characters together inside one Blender file** around a shared reference bone called a `Grab_Socket`.

  

🦴 `Grab_Socket`: _a helper bone parented to the boss's palm or claw in Blender whose transform drives the victim's root during a paired grab animation._

  

- **How the Lock Works in Godot 4:** The moment a perilous grab connects (`AttackProfile.custom_reaction_state = &"GrabbedState"`), the victim's `GrabbedState` disables player-to-boss capsule collision and attaches a `RemoteTransform3D` on the boss's `Grab_Socket` directly to the player's root.
    
- **Zero Drift Guarantee:** Because `RemoteTransform3D` copies the socket's exact position and rotation every physics tick, the victim stays locked inside the boss's fist through violent shakes, slams, and wind-ups with pixel-perfect contact.
    

### 🚀 2. Why Applying Velocity at the End Looks Cheap (And How to Fix the Transition)

Your intuition is dead right: if a boss swings its arm, freezes on the final frame, and your script suddenly injects `velocity = Vector3(0, 8, -18)`, the human eye instantly catches the seam.

  

In real physics—and in _God of War_ or _Monster Hunter_—a thrown object never accelerates out of thin air; **it simply keeps the velocity the hand already built up during the swing**.

  

🎯 `momentum inheritance`: _sampling the throwing socket's actual frame-by-frame displacement just before release so the victim leaves the hand at the exact speed and trajectory of the arm._

  

1. **Animate the Follow-Through in Blender:** Never stop the boss's throwing arm at the release point. Animate the boss swinging **past** the release frame (`Frames 40 → 55`), with the peak arm speed occurring right at the release window (`Frame 46`).
    
2. **Sample Socket Velocity (`3-Frame Rolling Average`):** Inside `GrabbedState`, track the `Grab_Socket`'s world position each physics frame (`socket_vel = (current_pos - prev_pos) / delta`). When the animation triggers `release_throw()` via a **Call Method Track**, you unparent `RemoteTransform3D` and hand that exact `socket_vel` (clamped and lightly assisted toward your desired throw vector) straight to `CharacterBody3D.velocity`.
    
3. **Blend the Pose Into an Airborne Tumble (`0.10s` Cross-Fade):** On that exact same release frame, cross-fade (`0.10s`) from `Player_Grabbed` into `Player_Thrown_Air`—where the limbs splay outward from centrifugal force rather than staying frozen in a chokehold pose.
    

### ☄️ 3. Making the Flight & Landing Feel Heavy (`ThrownState → KnockdownState`)

Standard platformer jump gravity (`9.8 m/s²`) makes a hurled warrior float like a beach ball. To give a _Hoarah Loux_ or _Tower Knight_ throw genuine weight, transition through **three crisp physical phases**:

  

|**Phase**|**State & Trigger**|**Physical & Visual Rules That Sell the Illusion**|
|---|---|---|
|**1. Launch & Apex**|`ThrownState` (`release_throw()`)|Inherit `socket_vel`, apply **`1.5×` heavier ballistic gravity**, and tilt the visual mesh's pitch smoothly along `velocity.normalized()` so the body arcs naturally head- or back-first.|
|**2. Ground Impact**|`on_floor` becomes `true`|Trigger an immediate **`0.06s` hit-stop micro-freeze**, spawn a directional dust-cloud decal, fire `camera_trauma`, and transition into `KnockdownState`.|
|**3. Skid & Recovery**|`KnockdownState`|Retain **`35%` of horizontal impact velocity** and bleed it off over `0.25s` using your existing `ground_traction` braking—so the player skids across the stone before rolling onto one knee with brief wake-up invulnerability.|

### 🎯 Next Step

Would you like to **scaffold `GrabbedState.gd` and `ThrownState.gd`** as plug-in states for your `StateMachine` once Step 2 (`CombatState`, `StateMachine`, `InputBuffer`) is wired up, or inspect how the **Blender paired-animation setup** for a boss grab (`Grab_Socket` + dual rig preview) is structured first?