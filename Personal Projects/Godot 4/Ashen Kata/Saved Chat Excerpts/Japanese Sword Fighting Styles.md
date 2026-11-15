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

Which of these sword styles—**Iaijutsu Quick-Draw**, **Flowing Orbit**, or **Ōdachi Great-Katana Sweeps**—feels closest to the combat identity you envision for *Ashen Kata*?