---
tags:
  - ashen-kata
  - combat-design
  - boss-arts
  - weapon-agnostic-skills
  - gamedev
aliases:
  - Weapon-Agnostic Boss Arts & Skills Reference
---
---
tags:
  - ashen-kata
  - combat-design
  - boss-arts
  - weapon-agnostic-skills
  - gamedev
aliases:
  - Weapon-Agnostic Boss Arts & Skills Reference
---

# 🔮 Weapon-Agnostic Skills & Boss Arts (`F` Slot Reference Library)

> [!abstract] Architectural Integration (`Ashen Kata`)
> These abilities inhabit the dedicated **`F` Boss Art / Relic slot**, remaining usable regardless of which melee weapon (`Katana`, `Odachi`, `Twin Blades`) is currently held. Each category maps directly to a distinct tactical role within *Ashen Kata*'s **Spirit Seal** and bidirectional **`Sha-Chi` posture** economy.

---

## ⚡ 1. Fast-Windup & Reaction Counters (`0.25s – 0.55s` Startup)
- ⤷ **Tactical Role**: Rapid mid-duel interrupts, deflection punishers, and snappy close-quarters bursts that ==fit inside narrow boss recovery windows without sacrificing tempo==.

| Skill Name | Source Game | Type | Mechanical Breakdown & *Ashen Kata* Translation |
| :--- | :--- | :--- | :--- |
| **Ashina Cross** | *Sekiro* | Melee | Quick-draw double-slash after a brief sheathe crouch; deals heavy `Sha-Chi` posture damage through guard if timed cleanly. |
| **Bestial Sling** | *Elden Ring* | Ranged | Instant shotgun-like fan of sharp stone shards; deals minimal HP damage but interrupts low-poise caster wind-ups and keeps `Sha-Chi` from decaying. |
| **Catch Flame** | *Elden Ring* | Melee | Near-instantaneous burst of fire from the palm; acts as a zero-commitment light-combo ender or roll-catch punish. |
| **Carian Slicer** | *Elden Ring* | Melee | Conjures an ethereal spectral blade for rapid, sweeping horizontal cuts; ignores physical shield deflection bounce. |
| **Nightjar Slash** | *Sekiro* | Melee / Gap-Closer | Leaping somersault cleave that closes mid-range distance rapidly while stepping over low ground sweeps. |
| **Shadowrush** | *Sekiro* | Melee / Pierce | High-velocity thrust that pierces enemy guard and launches the user upward off the target's shoulders on hit. |
| **O'Rin's Phantom Flurry** | *Sekiro* | Melee | Brief spectral phase-step followed by three rapid, near-weightless diagonal cuts that build steady posture pressure. |
| **Swift Glintstone Shard** | *Elden Ring* | Ranged | Mobile projectile castable while sprinting or mid-air; punishes bosses attempting to disengage and recover posture. |
| **Ichimonji** | *Sekiro* | Melee | Crisp downward overhead strike with tight startup; recovers a chunk of the user's own `Sha-Chi` posture upon impact. |
| **Lansseax's Glaive (Quick Sweep)** | *Elden Ring* | Melee / AoE | Conjures a sweeping arc of crimson lightning in front of the caster; clears surrounding mob crowds cleanly. |

> → *Design Note*: Fast-windup arts should cost **1 Spirit Seal** and feature near-zero recovery lag so players can deflect immediately afterward.

---

## 🔗 2. Combo Extenders & Flow Continuations (`Cancel-Window` Bridges)
- ⤷ **Tactical Role**: Designed to be woven directly into the end of a light-attack string (`LMB × 3 → F`), ==extending offensive pressure whilst depleting enemy poise==.

| Skill Name | Source Game | Type | Mechanical Breakdown & *Ashen Kata* Translation |
| :--- | :--- | :--- | :--- |
| **Floating Passage** | *Sekiro* | Melee | Flowing 5-hit dance of rapid, low-recovery slashes that overwhelms humanoid guards and forces continuous deflection checks. |
| **Spiral Cloud Passage** | *Sekiro* | Melee | Upgraded vacuum-wave flurry; emits short-range wind blades on every swing that chip through blocks. |
| **Black Flame Blade** | *Elden Ring* | Buff / Melee | Temporarily coats the active weapon in dark flame; adds percentage-based burn damage and extra `Stance Break` to the next 3-hit chain. |
| **Bloodflame Talons** | *Elden Ring* | Melee / Delayed AoE | Rakes the air with spectral claws that detonate after `0.6s`, letting the player continue swinging while the explosion interrupts the boss. |
| **Phantom Edge Chain-Whip** | *Phantom Blade Zero* | Melee / Mid-Range | Extends a segmented, shadow-forged blade whip mid-combo to yank staggered enemies back into melee range. |
| **Dragonclaw** | *Elden Ring* | Melee / Slam | Manifests a colossal spectral dragon claw for a heavy downward smash that chains directly into a second follow-up slam. |
| **Double Slash (Corpse Piler Arc)** | *Elden Ring* | Melee | Rapid twin diagonal slashes projecting crimson/ash crescent waves; bridges close melee into mid-range space. |
| **Praying Strikes — Exorcism** | *Sekiro* | Melee | Palm-strike and shoulder-check combo that deals pure `Sha-Chi` posture damage with zero blade lethality. |
| **Carian Piercer** | *Elden Ring* | Melee / Thrust | Conjures a long spectral greatsword thrust at the end of a fast katana string to catch back-stepping enemies. |
| **Dragonfire Sweep** | *Elden Ring* | Ranged / Cone | Channels a sweeping arc of dragon flame across the arena floor, denying space while resetting neutral positioning. |

---

## 🌪️ 3. Movement, Evasion & Repositioning Abilities (`Mobility & Counter-Checks`)
- ⤷ **Tactical Role**: Grants defensive utility, verticality, or instant flanking angles—ideal for ==evading unblockable perilous grabs and sweeping colossus attacks==.

| Skill Name | Source Game | Type | Mechanical Breakdown & *Ashen Kata* Translation |
| :--- | :--- | :--- | :--- |
| **Mist Raven** | *Sekiro* | Evasion / Teleport | Upon taking a hit in stance, dissolves into feathers/ash and teleports behind or above the attacker for a guaranteed counter-strike. |
| **Bloodhound's Step** | *Elden Ring* | Movement / Dash | Brief near-invisible dash with extended invulnerability frames (`i-frames`) that phases cleanly through wide boss cleaves. |
| **High Monk / Senpou Leaping Kicks** | *Sekiro* | Movement / Melee | Vaults into the air over low sweeping attacks, crashing down with a flying kick that inflicts massive posture damage. |
| **Shadowfall** | *Sekiro* | Movement / Melee | Piercing dash that launches the user high above the enemy on hit, opening an aerial follow-up or safe disengage. |
| **Mirage Blade Phase-Step** | *Phantom Blade Zero* | Movement / Blink | Leaves an afterimage decoy that draws enemy lock-on for `0.8s` while the player blinks laterally to a flank. |
| **Fortissax's Lightning Spear (Leap)** | *Elden Ring* | Movement / AoE | Launches the user airborne with brief hyper-armour before plunging dual lightning spears downward, dodging ground shockwaves. |
| **Raptor of the Mists** | *Elden Ring* | Evasion / Vertical | Low crouch stance that automatically vaults the user skyward when struck, setting up an immediate plunging execution. |
| **Nightjar Slash Reversal** | *Sekiro* | Movement / Retreat | Strikes the target and immediately backflips `5 m` out of danger, resetting spacing against heavy brutes. |
| **Waterfowl Ascent** | *Elden Ring* | Movement / Aerial | Rises smoothly into the air before dashing forward in a spherical whirlwind of slashes; bypasses ground hazards. |
| **Crucible Wings (Sky-Lance)** | *Elden Ring* | Movement / Thrust | Manifests spectral wings to hover briefly and rocket diagonally across the arena in a piercing aerial lunge. |

---

## 🔨 4. Slow-Windup Heavy Crushers (`1.0s – 1.6s` Commitment Tax)
- ⤷ **Tactical Role**: High-risk, high-poise-break haymakers requiring an earned opening—built to ==shatter armoured knight guards and topple colossal ankles==.

| Skill Name | Source Game | Type | Mechanical Breakdown & *Ashen Kata* Translation |
| :--- | :--- | :--- | :--- |
| **Mortal Draw** | *Sekiro* | Melee / Heavy | Wide, crimson-trailed unsheathing cleave that slices straight through enemy guards and deals massive HP and `Sha-Chi` damage. |
| **Empowered Mortal Draw** | *Sekiro* | Melee / Charged | Charged two-stage variant of Mortal Draw with extended crimson shockwave reach and colossal stagger force. |
| **Dragonmaw** | *Elden Ring* | Grab / Bite | Manifests a giant spectral dragon head that lunges forward and chomps down, flattening humanoid foes and dealing extreme `Stance Break`. |
| **Crucible Horns** | *Elden Ring* | Melee / Charge | Lowers shoulder with spectral horns and charges forward, launching humanoid enemies airborne on impact. |
| **Giantsflame Take Thee** | *Elden Ring* | Ranged / Artillery | Hurls a massive fireball with a lobbed arc that detonates in a heavy knockdown explosion. |
| **Wrath of Gold** | *Elden Ring* | Melee / Radial AoE | Releases an omnidirectional golden shockwave around the caster that blasts back surrounding mobs and interrupts medium foes. |
| **Ancient Dragons' Lightning Strike** | *Elden Ring* | Ranged / Multi-Hit | Kneels to summon jagged red lightning pillars radiating outward; shreds massive multi-hitbox colossi (`Tower Knight` / `Giant Lord`). |
| **Inescapable Frenzy** | *Elden Ring* | Grab | Lunges forward to seize a humanoid opponent by the helm, searing their mind with chaos flame for lethal posture/HP damage. |
| **Gavel of Haima** | *Elden Ring* | Melee / Slam | Conjures a colossal ethereal greathammer for a pancake-flattening overhead slam with high hyper-armour. |
| **One Mind** | *Sekiro* | Melee / Stance | Brief stationary sheathing stance followed by a blinding storm of invisible vacuum slashes inside a tight frontal sphere. |

> [!warning] Balancing Rule for Heavy Crushers
> Keep startup hyper-armour minimal so throwing a `1.4s` wind-up into a neutral duellist (`Onze` / `Crucible Commander`) gets punished—forcing the player to earn the window via a deflection stagger or *Ghoststep* flank.

---

## 🌕 5. High-Level Boss Arts & Spectacle Ultimates (`2–3 Spirit Seals` Cost)
- ⤷ **Tactical Role**: Capstone relics earned from major bosses; deliver ==cataclysmic *Rellana* / *Miquella*-scale spectacle under `AgX` HDR lighting== while remaining gated by aggressive deflection charging.

| Skill Name                       | Source Game        | Type               | Mechanical Breakdown & *Ashen Kata* Translation                                                                                               |
| :------------------------------- | :----------------- | :----------------- | :-------------------------------------------------------------------------------------------------------------------------------------------- |
| **Rellana's Twin Moons**         | *Elden Ring*       | Melee / Aerial AoE | Floats skyward, conjuring two luminous moons that crash sequentially into the earth—emitting two wide, jump-evadable shockwave rings.         |
| **Light of Miquella**            | *Elden Ring*       | Ranged / Orbital   | Calls down a blinding pillar of golden-white light followed by cascading secondary rays across a wide circular zone; devastating under `AgX`. |
| **Bayle's Flame-Lightning**      | *Elden Ring*       | Melee / Slam AoE   | Sprouts a spectral magma-lightning wing-arm, leaping into a devastating slam that erupts in jagged crimson lightning bolts.                   |
| **Bayle's Tyranny**              | *Elden Ring*       | Radial AoE / Roar  | Unleashes a cataclysmic draconic roar that detonates magma-lightning pools across the immediate arena.                                        |
| **Malenia's Scarlet Aeonia**     | *Elden Ring*       | Aerial / Dive AoE  | Rises airborne before plunging down to bloom into a massive radiant flower that bursts outward and leaves a lingering hazard zone.            |
| **Dragonlord's Placidusax Ruin** | *Elden Ring*       | Aerial / Beam      | Briefly transforms into a storm cloud before firing sweeping golden-red laser beams across the arena floor.                                   |
| **Messmer's Orb**                | *Elden Ring*       | Aerial / Dive Slam | Soars upward encased in a dark-flame sphere before crashing into the target and erupting into a ring of jagged fire spears.                   |
| **Midra's Flame of Frenzy**      | *Elden Ring*       | Aerial / Radial    | Levitates with a blazing chaos-sun head, raining continuous yellow-orange fire streams downward across a wide perimeter.                      |
| **One Mind — Ash Ascendant**     | *Sekiro* (Adapted) | Melee / Domain     | Freezes the air in a monochromatic domain flash, executing a rapid 9-hit judgment slash across every enemy inside a `6 m` radius.             |
| **Wolnir's Abyssal Surge**       | *Dark Souls III*   | Ranged / Wave      | Slams a cursed relic into the earth, sending a creeping wave of dark-indigo fog and skeletal shockwaves forward that drains enemy posture.    |