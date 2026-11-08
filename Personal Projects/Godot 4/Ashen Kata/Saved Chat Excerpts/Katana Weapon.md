### 📋 Executive Summary & Verdict

- ✅ **Weapon Choice:** Start with a **Two-Handed Curved Single-Edge Blade (Katana / Dao / Long Saber)**. It offers the ideal balance: intuitive visual orientation for directional slashes, a distinct cutting edge, and symmetrical two-handed grip physics that keep the torso grounded.
    
- ✅ **Fighting Style Verdict:** **Adopt Sekiro’s Ashina Kenjutsu or Ghost of Tsushima’s Stone Stance; completely avoid The Witcher 3’s pirouette style.** Circular spins and pirouettes require complex hip pivots, weight distribution, and foot crossing that invariably cause severe foot-sliding ("ice-skating") when animated by beginners. Linear, grounded planar cuts allow you to build punchy, readable combat using just three key poses per swing.
    

### 🚫 Why _The Witcher 3's_ Pirouette Style is a Trap for Beginners

Geralt’s fencing style in _The Witcher 3_ looks cinematic because it is a theatrical fusion of historical German longsword (_Kunst des Fechtens_) and classical stage ballet:

```
=============================================================================
             THE NOVICE ANIMATION PITFALL: PIROUETTES vs. PLANAR CUTS
=============================================================================

 ❌ THE PIRROUETTE / BALLET TRAP (The Witcher 3):
 [ Windup ] ──► [ 360° Hip Spin ] ──► [ Cross Legs ] ──► [ Off-Axis Slash ]
   - Root motion constantly rotates on the Y-axis.
   - Feet slide across the ground unless bone-pinned with Inverse Kinematics (IK).
   - Hitbox tracers sweep in erratic spheres, making hit-registration messy.

 ✅ THE GROUNDED PLANAR CADENCE (Sekiro / Ghost of Tsushima):
 [ Squared Hips ] ──► [ Straight Step Forward ] ──► [ Flat 45° Cutting Plane ]
   - Zero body rotation on the floor plane; only linear forward translation.
   - Both feet remain planted or step along a straight axis.
   - Clean, predictable 2D slicing planes that never miss enemy hurtboxes.
=============================================================================
```

If you attempt to keyframe full-body pirouettes without years of animation experience, the hips will wobble, the root motion will drift off-centre, and your weapon tracers will cut unpredictably.

### 🥋 The Ideal Reference Style: Grounded Classical Kenjutsu

Instead, copy the crisp, deliberate discipline of **Ashina Kenjutsu** (_Sekiro_) and **Stone Stance** (_Ghost of Tsushima_). The torso remains squared to the opponent, every strike cuts through a flat geometric plane, and the weapon returns immediately to a defensive centre-line guard.

#### 🎥 High-Fidelity Reference Material

1. **Ghost of Tsushima Stone Stance Martial Arts Breakdown (BladeMan B):**
    
    - **Reference Video:** [http://www.youtube.com/watch?v=sDvVUdv_hyg](http://www.youtube.com/watch?v=sDvVUdv_hyg)
        
    - **Why It Works:** This demonstration shows real-world martial arts forms executing the Stone Stance in clean profile. Observe how the swordsman's centre of gravity stays low and stable. The cuts are economical, fast, and recover immediately without unnecessary twirls.
        
2. **Sekiro Deflection & Kusabimaru Pacing (ONGBAL):**
    
    - **Reference Video:** [http://www.youtube.com/watch?v=yHC_S7NW9co](http://www.youtube.com/watch?v=yHC_S7NW9co)
        
    - **Why It Works:** Notice the instantaneous transition from posture to parry. Wolf does not wind up; his blade snaps straight from neutral _Seigan-no-kamae_ (middle guard) to an angled $45^\circ$ roof guard to catch incoming steel.
        

### 📦 The "Core 4" Animation Starter Pack

To build your first functional combat loop, you only need to create **four simple animations**:

```
 [ 0. Neutral Stance ] (Loop)
         │
         ├── Tap Attack ──► [ 1. Downward Diagonal Cut ] (Right Shoulder -> Left Hip)
         │                           │
         │                     Tap Attack
         │                           │
         │                           ▼
         │                  [ 2. Horizontal Under-Cut ] (Left Hip -> Right Ribs)
         │                           │
         │                     Tap Attack
         │                           │
         │                           ▼
         │                  [ 3. Overhead Vertical Cleave ] (Heavy Finisher)
         │
         └── Tap Guard  ──► [ 4. Snap Deflect / Parry ] (Instant Centerline Block)
```

#### 1. Attack 1: The Diagonal Downward Cut (_Kesa-Giri_)

- **The Motion:** The blade starts above the right ear and cuts down across the body at a $45^\circ$ angle, finishing outside the left hip.
    
- **Footwork:** A short, assertive half-step forward with the right foot.
    
- **Tracer Plane:** Generates a flat diagonal sheet that easily catches tall and wide enemy hurtboxes alike.
    

#### 2. Attack 2: The Rising Horizontal Cut (_Kiriage / Do-Giri_)

- **The Motion:** Taking advantage of the momentum from Attack 1 (which ended at the left hip), the blade reverses trajectory and slices horizontally or slightly upward across the enemy's torso from left to right.
    
- **Footwork:** Hips stay low; the left foot slides forward to match the right.
    
- **Tracer Plane:** A flat horizontal line parallel to the ground ($Y = 1.0\text{ m}$).
    

#### 3. Attack 3: The Overhead Executioner (_Shomen-Uchi_)

- **The Motion:** Both arms raise the hilt directly above the crown of the head, and the blade cleaves straight down along the vertical centreline into the dirt.
    
- **Footwork:** A full commitment lunge forward.
    
- **Gameplay Role:** This is your heavy finisher. Set `can_cancel_into_parry = false` on its `AttackProfile.tres`, inject a generous forward impulse, and give it `0.10s` of hit-stop freeze when it connects.
    

#### 4. The Deflect / Guard Snap (_Seigan Snap_)

- **The Motion:** From neutral, the arms immediately push forward and angle the blade at roughly $30^\circ\text{--}45^\circ$ across the chest, edge facing outward.
    
- **Key Timing:** **Startup must be 1 to 2 frames maximum!** Parries cannot have wind-ups; the blade must snap into defensive alignment instantly the moment the player taps RMB.
    

### ✏️ How to Keyframe Each Attack: The "3-Pose Rule"

Do not try to animate frame-by-frame (frame 1, frame 2, frame 3...). Professional animators use **Pose-to-Pose Blockout**:

```
 Frame 0: Neutral       Frame 4: Anticipation      Frame 7: Strike / Apex       Frame 18: Recovery
┌────────────────┐     ┌─────────────────────┐    ┌─────────────────────┐     ┌──────────────────┐
│ Blade centred, │ ──► │ Sword pulled back;  │ ──►│ Blade fully extended│ ──► │ Weapon returns   │
│ knees relaxed  │     │ torso coils tightly │    │ past target (HIT!)  │     │ toward centreline│
└────────────────┘     └─────────────────────┘    └─────────────────────┘     └──────────────────┘
```

1. **Pose 1: The Anticipation / Wind-Up (Frame 3–4):**
    
    - Pull the sword back slightly. Rotate the shoulders $15^\circ$ away from the strike direction to store visual tension.
        
2. **Pose 2: The Contact Point / Apex (Frame 6–8):**
    
    - The moment of maximum extension where the blade reaches the target. This is where your `WeaponTracer3D` collides and where you apply hit-stop.
        
3. **Pose 3: The Recovery / Follow-Through (Frame 16–20):**
    
    - The sword settles at the end of its arc before easing back into your neutral guard stance.
        

By focusing strictly on getting these **three key poses** looking sharp and balanced, your combat will feel crisp, readable, and responsive without requiring complex secondary motion or cloth physics.