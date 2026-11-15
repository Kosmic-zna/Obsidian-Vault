> ✍️ **Phrasing Refinement:** "By 'stance,' I actually meant sword-fighting styles—specifically distinct methods of swinging and combat flow."

### 📋 Executive Summary & Direct Verdict

* ✅ **Clarified Focus:** Rather than static guard postures (*kamae*), we are looking at **Kenjutsu Combat Styles (Ryūha & Archetypes)**—the actual **rhythm, swing geometry, footwork, and kinetic personality** of how a warrior swings a katana or ōdachi across a combo.
* ✅ **Four High-Contrast Sword Styles That Avoid Repetitive Slashing & Are Beginner-Friendly to Animate:**
1. **Iaijutsu / Battōjutsu (Explosive Quick-Draw & Sheathe-Return):** Coiled stillness → single-frame hyper-fast horizontal/diagonal snap slash → rhythmic blood-flick (`chiburi`) and scabbard re-sheathing (`nōtō`).
2. **Jigen-ryū / Heavy Descending Cleave (Brutal Overhead Momentum):** High, aggressive vertical & steep diagonal strikes (`kesa-giri`) powered by full torso drop and forward lunges—no delicate wrist twirls required.
3. **Suiō-ryū / Flowing Water & Rebound Style (Continuous Circular Momentum):** Using the momentum of one miss or deflection to whip the blade in a continuous figure-eight orbit (`nagashi` / flowing parry-riposte) rather than resetting to neutral after every swing.
4. **Ōdachi / Nodachi Centrifugal Sweep Style (Great-Katana Pendulum Physics):** Letting a massive $160\text{ cm}$ blade's weight pull the torso around in wide $270^\circ$ horizontal arcs, dragging the blade along the ash between swings.



---

### 📖 Plain-English Glossary of Sword-Style Kinematics

* **Anticipation → Snap → Follow-Through (The 3-Beat Anime/Sekiro Timing Rule):** Instead of moving a sword at a boring, constant speed, you hold the wind-up pose for $8\text{–}12\text{ frames}$ (**Anticipation**), whip the blade across the screen in $2\text{–}3\text{ frames}$ (**Snap**), and hold the extended finish pose for $10\text{ frames}$ (**Follow-Through**). This makes beginner animations look razor-sharp and effortless to read.
* **Kesa-giri (Diagonal Monk’s-Sash Cut):** A $45^\circ$ downward diagonal slash from shoulder to opposite hip—the cleanest, most natural arc for a human shoulder joint to animate in Blender.
* **Chiburi & Nōtō (Blood-Shake & Re-Sheathing):** The iconic ritual flourish after a combo where the swordsman flicks the blade to clear ash/blood and slides it cleanly back into the scabbard (`saya`) at the hip.

---

### 🏛️ Kinematic Flow Comparison: Four Distinct Sword-Fighting Styles

```
========================================================================================
             SWING GEOMETRY & TEMPO CONTRAST ACROSS FOUR SWORD STYLES
========================================================================================

 1. IAIJUTSU (Quick-Draw Burst)              2. JIGEN-RYŪ (Heavy Overhead Crusher)
    [Coiled Crouch] ═══► ⚡ [Flash Slash]       [High Skyward Lift]
                           └─► [Slow Nōtō]             │ (Full body drop!)
                                                       ▼
                                                [Earth-Splitting Cleave]

 2. FLOWING WATER (Figure-8 Orbit)           4. ŌDACHI PENDULUM (Great-Katana Sweep)
       ╭──► Slash 1 (Diagonal Down)             [Low Ash Drag] ──► (270° Spin Sweep)
       │         │                                                        │
       ╰── Slash 2 (Rising Rebound) ◄──┘                     [Shoulder Catch] ◄──┘
========================================================================================

```

---

### ⚔️ 1. Standard Katana Fighting Styles (Distinct Flow & Easy Keyframing)

| Sword Style | How It Swings & Flows (Combat Feel) | Why It Is Easy to Animate in Blender | Games & Media Inspiration |
| --- | --- | --- | --- |
| **1. Iaijutsu / Battōjutsu** *(Quick-Draw & Flash-Step Style)* | **Zero wasted motion.** You fight from the scabbard: explosive forward dash + single lightning-fast draw-slash, followed by a calm, deliberate re-sheathe (`nōtō`). Combo follow-ups use the scabbard itself to bash/deflect before a second draw. | **80% of the animation is crisp pose-to-pose stillness.** You only animate one 3-frame slash arc and a clean 15-frame scabbard return—no complex 6-hit flailing chains. | ***Sekiro: Shadows Die Twice*** (*Ashina Cross* & *One Mind*), ***Devil May Cry 5*** (Vergil’s Yamato combos), ***Ghost of Tsushima*** (*Heavenly Strike* / Standoff). |
| **2. Jigen-ryū / Ashina Brute Style** *(Relentless Overhead Crusher)* | **Violent, committed verticality.** Every swing starts high above the head or shoulder and crashes straight down into the opponent's guard, driving forward with heavy stomps. No fancy spins—pure lethal intent. | Vertical and $45^\circ$ diagonal (`kesa-giri`) swings follow the natural hinge of the shoulders and spine, meaning **zero awkward wrist-twisting** in Blender. | ***Sekiro*** (Isshin Ashina & Owl’s heavy downward slams), ***Vagabond*** (Sasaki Kojirō’s *Tsubame Gaeshi* swallow-reversal), ***Dororo*** (Hyakkimaru’s aggressive lunges). |
| **3. Suiō-ryū / Flowing Orbit Style** *(Continuous Rebound & Deflect Flow)* | **Never stops moving.** Instead of stopping the blade after a slash, the sword’s tip loops in a smooth figure-eight (`∞`), turning a downward slash directly into a rising backhand uppercut and a spinning deflect. | You literally copy-paste and mirror your slash keyframes in reverse (`Left-to-Right` → `Right-to-Left` rising), creating a hypnotic 3-hit dance out of just **two base poses**. | ***Ghost of Tsushima*** (*Water Stance* flowing surge combo), ***Rurouni Kenshin*** (*Hiten Mitsurugi-ryū* multi-angle vacuum slashes), ***Sifu*** (fluid parry-into-strike chains). |
| **4. Shinobi One-Handed / Reverse-Tempo Style** *(Agile Acrobat & Off-Hand Mix)* | **Unpredictable angles.** Wields the katana mostly one-handed while using the free left hand for balance, palm strikes, or throwing ash/kunai mid-combo, mixing normal slashes with inverted backhand hooks. | **Single-hand grip eliminates two-handed IK locking headaches**, and short, snappy wrist/elbow flicks look ultra-fast with minimal torso travel. | ***Sekiro*** (Wolf’s primary *Kusabimaru* + Shinobi Prosthetic weaving), ***Bleach*** (Ichigo / Byakuya one-handed blitzes), ***Metal Gear Rising*** (Raiden’s blade-mode angles). |

---

### 🗡️ 2. Differing Weapon Styles (Ōdachi, Nagamaki & Dual-Tempo)

If you give the player weapon switching—or stance switching—each weapon archetype naturally enforces a completely different swing rhythm:

* **Ōdachi / Nodachi (Great Katana — Heavy Centrifugal Style):**
* **Swing Mechanics:** Because a $160\text{ cm}$ blade is too long to flick with the wrists, **the sword leads the body**. The warrior plants their front foot, whips their hips first, and lets the massive blade lag slightly behind before sweeping across a huge $270^\circ$ horizontal or rising arc—ending with the blade's momentum carrying it onto the opposite shoulder.
* **Why It Looks Stunning & Animates Easily:** Adding a **2-frame drag/lag** on the sword hand behind the chest rotation immediately sells the illusion of a 15-pound steel blade without needing extra keyframes.
* **Inspiration:** ***Elden Ring: Shadow of the Erdtree*** (*Great Katana* & *Rakshasa’s Great Katana* sweeping arcs), ***Monster Hunter Rise*** (*Long Sword* Spirit Blade / Crimson Slash), ***Gintama*** / ***Berserk*** (heavy momentum stops).


* **Nagamaki (Long-Hilt Katana — Pole-Lever Whirlwind Style):**
* **Swing Mechanics:** With a handle almost as long as the blade, the warrior slides their hands along the wrap to choke up for fast close-quarters thrusts or slide back to the pommel for massive, long-reach cleaves and overhead helicopter spins.
* **Inspiration:** ***Nioh 2*** (*Odachi* / *Switchglaive* stance-switching mid-combo), ***Dynasty Warriors / Samurai Warriors*** (wide crowd-clearing sweeps).


* **Katana + Wakizashi (Daishō Dual-Blade Scissor Style):**
* **Swing Mechanics:** Long katana controls mid-range space with wide horizontal sweeps while the short companion blade (`wakizashi`) parries inward or delivers rapid double-cross scissor cuts (`jūji-giri`) at close range.
* **Inspiration:** ***Ghost of Tsushima*** (Dual-wield *Ghost* / *Way of the Flame* cross-slashes), ***Nioh 2*** (*Dual Swords* *Sign of the Cross* & *Water Sword* flurry).



---

### 🛠️ How to Prototype Your First 3-Hit Combo in Godot & Blender

When you are ready to build your first sword combo for *Ashen Kata*, keep the scope surgical:

1. **Pick One Core Identity First:** Pair **Iaijutsu Quick-Draw** (for sprint-ambush / heavy opener) with **Flowing Orbit 3-Hit Combo** (Light Attack 1: Diagonal Down → Light Attack 2: Rising Rebound → Light Attack 3: Heavy Overhead Cleave).
2. **Use Godot’s `AnimationLibrary` + Root Motion (or Code-Driven Impulse):** Keep your animations **in-place** (just like `Jog_Fwd` and `Sprint`) and apply a tiny forward velocity burst (`velocity += -visual_model.global_basis.z * lunge_speed`) in `playerMovement.gd` on the exact frame the slash snaps—giving every swing crisp *Sekiro*-style forward magnetism without fighting root-motion drift!

> ✍️ **Phrasing Refinement:** "I will begin manually animating a basic combat swing grounded in kenjutsu fighting styles alongside activated iaijutsu skills; could you provide a structured table of distinctive, non-repetitive attack patterns—categorised by normal, heavy, miscellaneous, and skill applications with specific game inspirations—and identify the relevant modules in my Masterlist covering the engineering architecture behind combat development?"
> 
>   

### 📋 Executive Summary & Direct Verdict

- ✅ **The Combat Animation Blueprint:** To avoid repetitive "left-right-left" slashing while keeping manual Blender keyframing achievable for a beginner, build your moveset around **alternating planes of motion and kinetic tempo**—pairing flowing diagonal/rising kenjutsu strings for normal attacks with committed overhead/centrifugal strikes for heavy attacks, and saving explosive **scabbard-draw bursts (_Iaijutsu_)** for activated skills.
    
      
    
- ✅ **Essential `Masterlist.md` Reading Order for Combat Engineering:** Before animating or coding your sword swings, read **Module III (Sections 3.2 & 3.3)** for `HitboxComponent`/`HurtboxComponent` spatial collision and Pushdown State Machine interrupts (`AttackState`), **Module VI (Section 6.2)** for `AnimationTree` upper-body bone filtering and Root Motion lunges, **Module II (Section 2.2)** for data-driven `AttackProfile` resources (hitstop, knockback, trauma), and **Module V (Section 5.2)** for screen-shake impact feedback.
    
      
    

### 📖 Plain-English Glossary of Combat & Animation Terms

- **Plane of Motion (Cut Trajectory):** The geometric angle along which a blade travels—such as downward diagonal (_kesa-giri_), rising uppercut (_kiri-age_), horizontal sweep (_yoko-giri_), or linear thrust (_tsuki_). Alternating planes between consecutive hits is what makes a combo feel dynamic rather than repetitive.
    
      
    
- **Hitbox vs. Hurtbox (`Area3D` Volumes):** A **Hitbox** is an active collision volume attached to your katana blade that turns on _only_ during the lethal swing frames (`active frames`); a **Hurtbox** is a passive collision capsule wrapped around an enemy's body that receives the hit.
    
      
    
- **Active Frames vs. Recovery Frames:** Every melee animation splits into three phases—**Startup** (wind-up anticipation), **Active** (the 3–5 frames where the blade deals damage), and **Recovery** (follow-through where the character can cancel into a dodge or next combo hit).
    
      
    
- **Hitstop (Freeze-Frame Juice):** Pausing or slowing `anim_player.speed_scale` for $40\text{–}90\text{ milliseconds}$ on the exact frame a katana connects with an enemy, creating the visceral sensation of steel biting through resistance.
    
      
    
- **Nōtō & Chiburi (Sheathe-Return & Blood-Flick):** Traditional kenjutsu finishing flourishes—flicking the blade to clear ash/blood (_chiburi_) and sliding it back into the scabbard (_nōtō_)—which serve as natural idle transitions or stamina/posture recovery animations.
    
      
    

### ⚔️ Non-Repetitive Kenjutsu & Iaijutsu Combat Moveset Table

Each entry below is designed around **clear silhouette poses** (using IK hand controllers in Blender) so you can achieve flashy, high-impact motion without complex finger or wrist micro-animation:

  

|**Attack Name & Style**|**Role / Slot**|**Swing Pattern & Animation Keyframes**|**Why It Feels Distinct & Easy to Animate**|**Specific Game / Media Inspiration**|
|---|---|---|---|---|
|**1. Kesa-Nagashi** _(Flowing Sash-Cut — Suiō-ryū)_|**Normal 1** _(Opener)_|**45° Downward Diagonal:** Steps forward with left foot, slashing cleanly from right shoulder to left hip; blade ends low at the left knee.|Follows the natural hinge of the shoulder; ending low sets up the rising second hit with zero awkward reset.|**_Ghost of Tsushima_** (_Water Stance_ 1st strike) & **_Sekiro_** (_Kusabimaru_ 1st light slash).|
|**2. Kiri-Age Gaeshi** _(Rebound Rising Cut — Suiō-ryū)_|**Normal 2** _(Combo Link)_|**45° Rising Uppercut:** Uses the low finish of Normal 1 to whip the blade upward from left knee across to upper-right skyward guard (_hassō_).|You literally mirror Normal 1's arc in reverse—creating fluid figure-eight momentum out of two poses.|**_Ghost of Tsushima_** (_Water Stance_ surge) & **_Rurouni Kenshin_** (_Hiten Mitsurugi_ rising reversal).|
|**3. Kaiten-Zuki** _(Spinning Corkscrew Thrust — Hokushin Ittō-ryū)_|**Normal 3** _(Combo Finisher)_|**360° Pivot → Linear Lunge:** Spins leftward on the front heel while tucking the hilt to the ribs, exploding forward into a long two-handed piercing thrust.|Replaces a third repetitive slash with a sharp **linear piercing silhouette** that pushes enemies back.|**_Sekiro_** (_Whirlwind Slash_ → _Shadowrush_ thrust) & **_Nioh 2_** (Mid-stance Katana finisher).|
|**4. Jigen-Rakka** _(Sky-Splitting Vertical Drop — Jigen-ryū)_|**Heavy Attack** _(Neutral / Guard Break)_|**High Vertical Cleave:** Raises hilt high above the right ear (`Hassō-no-kamae`), leaps/stomps heavily forward, and slams the blade straight down to the ground.|Maximum **anticipation-to-snap contrast** (12-frame hold at peak → 3-frame slam); shatters enemy guard posture.|**_Sekiro_** (_Ichimonji_ / _Ichimonji: Double_) & **_For Honor_** (Kensei’s top unblockable finisher).|
|**5. Ōdachi Chi-Barai** _(Ash-Drag Sweeping Arc — Nodachi)_|**Heavy Attack** _(Charged / Crowd Cleave)_|**Low Ground-Drag → 270° Horizontal Sweep:** Trails the blade tip behind the ankle during charge, then whips hips in a wide horizontal cyclone.|Dragging the tip along the ground builds dramatic tension before releasing a huge circular hitbox.|**_Elden Ring: Shadow of the Erdtree_** (_Great Katana_ / _Rakshasa_ sweeps) & **_Monster Hunter Rise_** (_Long Sword_ Spirit Roundslash).|
|**6. Tsuka-Ate & Chiburi** _(Pommel Bash / Blood-Flick)_|**Misc** _(Sprint Attack / Posture Stagger / Flourish)_|**Close-Range Hilt Strike / Blade Flick:** Drives the sword pommel (_tsuka_) into the enemy’s chin mid-sprint, or flicks blade sideways to snap back into scabbard.|Gives your sprint a snappy opener without long recovery frames; _chiburi_ doubles as a stylish post-fight taunt.|**_Sekiro_** (Sprint slash / _Senpou Leaping Kicks_ stagger) & **_Ghost of Tsushima_** (Swipe-right blade wipe/sheathe).|
|**7. Ashina Jūji-Giri** _(Cross-Draw Flash — Iaijutsu)_|**Activated Skill 1** _(Burst Gap-Closer)_|**Coiled Crouch → Double Cross-Slash:** Left thumb clicks guard (_tsuba_) out of scabbard; character dashes forward in a blur, carving an **X** across the target before re-sheathing.|80% of the clip is a tense coiled crouch pose; the actual slash happens across just 4 rapid frames with trail VFX.|**_Sekiro_** (_Ashina Cross_ & _One Mind_) & **_Devil May Cry 5_** (Vergil’s _Rapid Slash_ / _Judgement Cut_).|
|**8. Tsubame-Gaeshi** _(Swallow Reversal — Ganryū Iaijutsu)_|**Activated Skill 2** _(Anti-Air / Counter Skill)_|**Vertical Slam → Instant Mid-Air Upward Reversal:** Delivers a deceptive downward draw-slash, immediately flipping wrists mid-swing to slice upward along the exact same line.|Catches dodging or airborne enemies off-guard; looks ultra-technical while reusing the exact same vertical arc twice.|**_Fate/stay night_** (Sasaki Kojirō’s _Tsubame Gaeshi_) & **_Nioh 2_** (_Iai Quickdraw_ → _Night Rain_).|
|**9. Kaze-no-Nagi** _(Vacuum Crescent Draw — Battōjutsu)_|**Activated Skill 3** _(Mid-Range Wave / Ōdachi Skill)_|**Wide Horizontal Draw-Sweep:** Plants wide horse stance (_kiba-dachi_), draws the blade in a sweeping $180^\circ$ horizontal arc that launches a razor-thin wind/ash crescent projectile.|Keeps the player stationary in an iconic samurai silhouette while spawning a projectile hitbox (`BallisticStrategy`).|**_Elden Ring_** (_Moonveil_ _Transient Moonlight_ / _Waterfowl_) & **_Ghost of Tsushima_** (_Dance of Wrath_).|

### 📚 Masterlist Architecture Roadmap: What This Phase Entails

```
========================================================================================
             HOW YOUR COMBAT SYSTEM MAPS TO `Masterlist.md` MODULES
========================================================================================

  [ 1. INPUT & STATE ]       Module III (3.3): Pushdown State Machine (`AttackState`)
         │                   ──► Interrupts `Run`/`Idle` cleanly; pops back on finish
         ▼
  [ 2. ANIMATION & MOTION ]  Module VI (6.2): `AnimationTree` (`OneShot` Upper-Body Filter
         │                   + `extract_root_motion_velocity()` for forward sword lunges)
         ▼
  [ 3. HIT DETECTION ]       Module III (3.1 & 3.2): `HitboxComponent` ──► `HurtboxComponent`
         │                   ──► `HealthComponent` ("Call Down, Signal Up")
         ▼
  [ 4. DATA & GAME FEEL ]    Module II (2.2) & Module V (5.2): `AttackProfile` Resource
                             (Damage, Knockback, Hitstop) + `CinematicCameraRig` Trauma
========================================================================================
```

Consult these exact sections in your **`Masterlist.md`** as you build each layer of your combat pipeline:

  

1. **Module III — Section 3.3: The State Pattern (`StateMachine.gd` & `AttackState.gd`)**
    
      
    - **What You Will Learn:** Why adding `is_attacking` booleans to `playerMovement.gd` eventually creates unmaintainable bugs, and how a **Pushdown Automaton (State Stack)** pushes an `AttackState` over your locomotion state during a swing and automatically `pop_state()`s back to `Idle` or `Run` when the animation finishes.
        
          
        
2. **Module III — Sections 3.1 & 3.2: Component Architecture (`HitboxComponent`, `HurtboxComponent`, `HealthComponent`)**
    
      
    - **What You Will Learn:** How to attach an active `HitboxComponent` (`Area3D` with `monitoring = true`) to your katana mesh and toggle it on/off strictly during your swing's **active frames** via `AnimationPlayer` Call Method tracks—delivering damage cleanly to enemy `HurtboxComponent` volumes with built-in invulnerability frames (i-frames).
        
          
        
3. **Module VI — Section 6.2: `AnimationTree`, Per-Bone Filtering & Root Motion Extraction**
    
      
    - **What You Will Learn:** Two vital combat animation techniques:
        
          
        - **Per-Bone Filtering (`AnimationNodeOneShot`):** Lets your upper body swing the katana while your lower body keeps running smoothly without "ice-skating".
            
              
            
        - **Root Motion Extraction (`get_root_motion_position()`):** Lets you animate forward steps and _Iaijutsu_ lunges directly inside Blender so your character's feet stay locked to the terrain during attacks.
            
              
            
4. **Module II — Section 2.2: Data-Driven Combat Payloads (`AttackProfile` Resource)**
    
      
    - **What You Will Learn:** How to package each attack's damage, knockback impulse, launch angle, hitstop freeze duration (`hitstop_duration_sec`), and screen trauma (`camera_trauma_injection`) into reusable `.tres` files so Normal 1, Heavy Cleave, and _Ashina Jūji-Giri_ feel completely distinct without writing separate combat scripts.
        
          
        
5. **Module III — Section 3.5: The Strategy Pattern via Custom Resources**
    
      
    - **What You Will Learn:** How to swap weapon behaviours (e.g., standard katana melee hitboxes vs. _Kaze-no-Nagi_ crescent wave projectiles vs. _Ōdachi_ wide arcs) dynamically at runtime using swappable `Resource` strategies.
        
          
        
6. **Module IV (Sections 4.2 & 4.5) & Module V (Section 5.2): Combat Math & Camera Juice**
    
      
    - **What You Will Learn:** Using **Dot Products** (Section 4.2) to check whether an attack lands within a frontal cleave cone or triggers a critical backstab[cite: 16], **Quaternion SLERP** (Section 4.5) to smoothly snap your character toward a locked-on target during wind-up[cite: 16], and **`CinematicCameraRig.add_trauma()`** (Section 5.2) to inject visceral screen shake on blade impact[cite: 16].
        
          
        

### 💡 Three Practical Rules Before Keyframing Your First Swing in Blender

- **Rule 1 — Animate In-Place First (Code-Driven Lunge):** Until you set up `AnimationTree` Root Motion (Module VI, Section 6.2)[cite: 16], keep Y Bot's hips roughly centred over the origin (`0, 0, 0`) in Blender and trigger a small forward velocity burst in GDScript on the active swing frame—it integrates immediately with your current `AnimationPlayer` setup.
    
      
    
- **Rule 2 — Exaggerate the Silhouette & Timing:** Use **30 FPS** in Blender (`1 second = 30 frames`). Hold your wind-up pose on frames `1–8`, whip the sword across to the follow-through pose by frame `12`, and ease out from frames `13–24`.
    
      
    
- **Rule 3 — Use `AnimationPlayer` Call Method Tracks for Hitboxes:** In Godot's `AnimationPlayer`, add a **Call Method Track** targeting your katana's `HitboxComponent` to call `enable_hitbox()` on frame `9` and `disable_hitbox()` on frame `13`—guaranteeing frame-accurate hit detection synced to your visual blade arc.