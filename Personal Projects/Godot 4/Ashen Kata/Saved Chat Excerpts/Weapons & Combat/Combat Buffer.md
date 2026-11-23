> ✍️ **Phrasing Refinement:** "What about the overall attack-chaining architecture? In an earlier combat prototype I left unfinished, my input listener recorded inputs immediately without waiting for the animation to finish; however, during a long animation, clicking twice would queue the second attack far too early, making the input buffer feel sluggish and involuntary rather than seamless. My instinct is to attach expiry timers to buffered inputs so stale presses are discarded—what is the AAA combat-design standard for resolving this disconnect between player intent and engine execution?"
> 
>   

### 📋 Executive Summary & Direct Answer

- ✅ **Your Instinct Is 100% Spot-On—That Is Literally How _Sekiro_, _Elden Ring_, and _God of War_ Solve It:** What you experienced in your old prototype is called **Infinite / Unbounded Input Queueing** (the classic "panicked double-tap traps you in Attack 2 half a second later" bug). AAA action games fix this exact disconnect using **Four Combined Rules**:
    
      
    1. **Expiring Single-Slot Buffer (`0.15 s–0.20 s` TTL — NOT a Multi-Input Queue):** Instead of an array/queue that hoards 3 clicks, the buffer holds **only the single most recent action** (`buffered_action: StringName`) with a strict **`0.18 s` expiration timestamp**. If you double-click at the very start of a slow wind-up, that second click expires long before `open_combo_window()` arrives—so it never traps you in an unwanted swing!
        
          
        
    2. **"Last Input Wins" Overwrite (Defensive Priority Override):** If you press `Attack` during a wind-up and then panic-press `Deflect` or `Ghoststep` `0.05 s` later because an enemy swings back, the newer `Deflect` input **immediately overwrites** `buffered_action = &"deflect"` instead of forcing you to swing into a blade.
        
          
        
    3. **Input-Lockout / Buffer-Open Gate (`open_buffer_window()`):** Many AAA titles (_Sekiro_, _Devil May Cry_, _Monster Hunter_) actually **ignore** combo buffering during the very first ~30% of a heavy wind-up (`Frames 0 → 10`), only opening the **Buffer Listening Window** right around the strike apex (`Frames 12 → 26`), and then executing it at `open_combo_window()` (`Frame 27`).
        
          
        
    4. **Hit-Stop Time Freeze Exclusion:** The `0.18 s` buffer timer runs in **real wall-clock / unscaled delta time** (or pauses during hit-stop) so a 4-frame blade-clash freeze doesn't accidentally eat the player's buffered follow-up!
        
          
        

### 📖 Plain-English Glossary of AAA Input Buffering

- **Unbounded FIFO Queue (Why Your Old Prototype Felt Bad):** Storing every click in a list (`[Attack, Attack, Dodge]`) and executing them one by one. Even a micro-twitch double-click locks the player out of control for 1.5 seconds.
    
      
    
- **Single-Slot Overwriting Buffer With TTL (Time-To-Live):** Storing **one** variable (`buffered_action`) alongside `buffer_expire_time`. Any new button press replaces the old one and resets the `0.18 s` clock. If `open_combo_window()` isn't reached within `0.18 s`, `buffered_action` silently vanishes.
    
      
    
- **Buffer Gate (`can_buffer`) vs. Execute Gate (`can_combo`):** Splitting an attack animation into three distinct psychological zones:
    
      
    - **Zone 1 — Early Wind-Up (`Frames 0 → 12`):** Buffer is **closed** (accidental double-taps at the start of Attack 1 are ignored).
        
          
        
    - **Zone 2 — Commit & Slash (`Frames 12 → 26`):** Buffer is **open** (`0.18 s` TTL)—a deliberate second press right as the blade swings feels crisp and responsive.
        
          
        
    - **Zone 3 — Cancel Window (`Frame 27+`):** `can_combo = true`—any valid non-expired buffered action fires immediately!
        
          
        

### 🗺️ Architectural Diagram: Unbounded Queue vs. AAA Single-Slot Expiring Buffer

```
========================================================================================
       WHY DOUBLE-CLICKING TRAPPED YOU BEFORE VS. AAA SINGLE-SLOT EXPIRING BUFFER
========================================================================================

 ❌ YOUR OLD PROTOTYPE (Unbounded Queue — No Expiration):
    Frame 0: Double-click (`Attack`, `Attack`) ──► Queue = [Attack_2] (Sits forever!)
    Frame 27 (`open_combo_window`): Pops `Attack_2` ──► Player swings involuntarily 0.45s later!

 --------------------------------------------------------------------------------------
 ✅ AAA COMBAT BUFFER (`Single-Slot` + `0.18s TTL` + `Last-Input-Wins` + `Buffer Gate`):

    [0 ─── Wind-Up ─── 12] [12 ── Slash & Recoil ── 26] [27 ─── Cancel Window ─── 58]
     │                      │                            │
     ▼                      ▼                            ▼
    `can_buffer = false`   `can_buffer = true`          `can_combo = true`
    (Early double-tap       • Press `Attack` -> stores   • Checks: Is `buffered_action`
     is safely ignored!)      `&"attack"` (TTL: 0.18s)     still valid (< 0.18s old)?
                            • Press `Deflect` 0.05s       ├─ YES: Execute immediately!
                              later -> OVERWRITES with    └─ NO (Expired): Stay in
                              `&"deflect"` (TTL: 0.18s)!       recovery frames!
========================================================================================
```

### 🛠️ Production-Ready Godot 4 `InputBuffer.gd` Component

Drop this standalone child node onto your `Player` (right alongside your `StateMachine`), and every combat state (`Idle`, `Attack`, `Deflect`, `Ghoststep`) can query it cleanly:

  

GDScript

```
# InputBuffer.gd
class_name InputBuffer
extends Node

@export var buffer_duration: float = 0.18 # 180ms AAA sweet spot (11 frames at 60 FPS)

const TRACKED_ACTIONS: Array[StringName] = [
	&"attack",
	&"heavy_attack",
	&"deflect",
	&"dodge",
]

var buffered_action: StringName = &""
var _time_remaining: float = 0.0
var is_listening: bool = true # Toggled by animation tracks during early wind-up!

func _process(delta: float) -> void:
	if _time_remaining > 0.0:
		_time_remaining -= delta
		if _time_remaining <= 0.0:
			clear()

func _unhandled_input(event: InputEvent) -> void:
	if not is_listening:
		return
	for action in TRACKED_ACTIONS:
		if event.is_action_pressed(action):
			# Single-slot overwrite: newer input (e.g. Deflect) replaces older Attack!
			buffered_action = action
			_time_remaining = buffer_duration
			break

# Called by AttackState when `open_combo_window()` or `open_deflect_window()` fires:
func consume(action: StringName) -> bool:
	if buffered_action == action and _time_remaining > 0.0:
		clear()
		return true
	return false

func consume_any() -> StringName:
	if buffered_action != &"" and _time_remaining > 0.0:
		var action := buffered_action
		clear()
		return action
	return &""

func clear() -> void:
	buffered_action = &""
	_time_remaining = 0.0
```

#### How Your `AttackState.gd` Uses It (3 Lines of Logic)

1. **At Frame `0` of `Slash_Kesa_01`:** `input_buffer.is_listening = false` and `input_buffer.clear()` _(prevents leftover double-clicks from Attack 1 spilling into Attack 2)_.
    
      
    
2. **At Frame `12` (Mid-Windup / Blade Lift):** Animation Method Call Track calls `input_buffer.is_listening = true` _(now the player can cleanly pre-buffer Attack 2 or an emergency Deflect)_.
    
      
    
3. **At Frame `27` (`open_combo_window()`):**
    
      
    
    GDScript
    
    ```
    func open_combo_window() -> void:
    	can_combo = true
    	var next_input := input_buffer.consume_any()
    	if next_input == &"deflect":
    		state_machine.transition_to("Deflect")
    	elif next_input == &"dodge":
    		state_machine.transition_to("Ghoststep")
    	elif next_input == &"attack" and current_attack.next_attack != null:
    		state_machine.transition_to("Attack", { "attack_data": current_attack.next_attack })
    ```
    

- **Why This Eliminates the "Disconnect" Completely:** Because `is_listening` ignores the first 12 frames of wind-up, `buffer_duration` expires stale presses after `180 ms`, and `consume_any()` lets `Deflect` or `Ghoststep` overwrite `Attack` if the player changes their mind mid-swing, the character **never** executes an action the player stopped wanting a quarter-second ago!

> ✍️ **Phrasing Refinement:** "What about the overall attack-chaining architecture? In an earlier combat prototype I left unfinished, my input listener recorded inputs immediately without waiting for the animation to finish; however, during a long animation, clicking twice would queue the second attack far too early, making the input buffer feel sluggish and involuntary rather than seamless. My instinct is to attach expiry timers to buffered inputs so stale presses are discarded—what is the AAA combat-design standard for resolving this disconnect between player intent and engine execution?"
> 
>   

### 📋 Executive Summary & Direct Answer

- ✅ **Your Instinct Is 100% Spot-On—That Is Literally How _Sekiro_, _Elden Ring_, and _God of War_ Solve It:** What you experienced in your old prototype is called **Infinite / Unbounded Input Queueing** (the classic "panicked double-tap traps you in Attack 2 half a second later" bug). AAA action games fix this exact disconnect using **Four Combined Rules**:
    
      
    1. **Expiring Single-Slot Buffer (`0.15 s–0.20 s` TTL — NOT a Multi-Input Queue):** Instead of an array/queue that hoards 3 clicks, the buffer holds **only the single most recent action** (`buffered_action: StringName`) with a strict **`0.18 s` expiration timestamp**. If you double-click at the very start of a slow wind-up, that second click expires long before `open_combo_window()` arrives—so it never traps you in an unwanted swing!
        
          
        
    2. **"Last Input Wins" Overwrite (Defensive Priority Override):** If you press `Attack` during a wind-up and then panic-press `Deflect` or `Ghoststep` `0.05 s` later because an enemy swings back, the newer `Deflect` input **immediately overwrites** `buffered_action = &"deflect"` instead of forcing you to swing into a blade.
        
          
        
    3. **Input-Lockout / Buffer-Open Gate (`open_buffer_window()`):** Many AAA titles (_Sekiro_, _Devil May Cry_, _Monster Hunter_) actually **ignore** combo buffering during the very first ~30% of a heavy wind-up (`Frames 0 → 10`), only opening the **Buffer Listening Window** right around the strike apex (`Frames 12 → 26`), and then executing it at `open_combo_window()` (`Frame 27`).
        
          
        
    4. **Hit-Stop Time Freeze Exclusion:** The `0.18 s` buffer timer runs in **real wall-clock / unscaled delta time** (or pauses during hit-stop) so a 4-frame blade-clash freeze doesn't accidentally eat the player's buffered follow-up!
        
          
        

### 📖 Plain-English Glossary of AAA Input Buffering

- **Unbounded FIFO Queue (Why Your Old Prototype Felt Bad):** Storing every click in a list (`[Attack, Attack, Dodge]`) and executing them one by one. Even a micro-twitch double-click locks the player out of control for 1.5 seconds.
    
      
    
- **Single-Slot Overwriting Buffer With TTL (Time-To-Live):** Storing **one** variable (`buffered_action`) alongside `buffer_expire_time`. Any new button press replaces the old one and resets the `0.18 s` clock. If `open_combo_window()` isn't reached within `0.18 s`, `buffered_action` silently vanishes.
    
      
    
- **Buffer Gate (`can_buffer`) vs. Execute Gate (`can_combo`):** Splitting an attack animation into three distinct psychological zones:
    
      
    - **Zone 1 — Early Wind-Up (`Frames 0 → 12`):** Buffer is **closed** (accidental double-taps at the start of Attack 1 are ignored).
        
          
        
    - **Zone 2 — Commit & Slash (`Frames 12 → 26`):** Buffer is **open** (`0.18 s` TTL)—a deliberate second press right as the blade swings feels crisp and responsive.
        
          
        
    - **Zone 3 — Cancel Window (`Frame 27+`):** `can_combo = true`—any valid non-expired buffered action fires immediately!
        
          
        

### 🗺️ Architectural Diagram: Unbounded Queue vs. AAA Single-Slot Expiring Buffer

```
========================================================================================
       WHY DOUBLE-CLICKING TRAPPED YOU BEFORE VS. AAA SINGLE-SLOT EXPIRING BUFFER
========================================================================================

 ❌ YOUR OLD PROTOTYPE (Unbounded Queue — No Expiration):
    Frame 0: Double-click (`Attack`, `Attack`) ──► Queue = [Attack_2] (Sits forever!)
    Frame 27 (`open_combo_window`): Pops `Attack_2` ──► Player swings involuntarily 0.45s later!

 --------------------------------------------------------------------------------------
 ✅ AAA COMBAT BUFFER (`Single-Slot` + `0.18s TTL` + `Last-Input-Wins` + `Buffer Gate`):

    [0 ─── Wind-Up ─── 12] [12 ── Slash & Recoil ── 26] [27 ─── Cancel Window ─── 58]
     │                      │                            │
     ▼                      ▼                            ▼
    `can_buffer = false`   `can_buffer = true`          `can_combo = true`
    (Early double-tap       • Press `Attack` -> stores   • Checks: Is `buffered_action`
     is safely ignored!)      `&"attack"` (TTL: 0.18s)     still valid (< 0.18s old)?
                            • Press `Deflect` 0.05s       ├─ YES: Execute immediately!
                              later -> OVERWRITES with    └─ NO (Expired): Stay in
                              `&"deflect"` (TTL: 0.18s)!       recovery frames!
========================================================================================
```

### 🛠️ Production-Ready Godot 4 `InputBuffer.gd` Component

Drop this standalone child node onto your `Player` (right alongside your `StateMachine`), and every combat state (`Idle`, `Attack`, `Deflect`, `Ghoststep`) can query it cleanly:

  

GDScript

```
# InputBuffer.gd
class_name InputBuffer
extends Node

@export var buffer_duration: float = 0.18 # 180ms AAA sweet spot (11 frames at 60 FPS)

const TRACKED_ACTIONS: Array[StringName] = [
	&"attack",
	&"heavy_attack",
	&"deflect",
	&"dodge",
]

var buffered_action: StringName = &""
var _time_remaining: float = 0.0
var is_listening: bool = true # Toggled by animation tracks during early wind-up!

func _process(delta: float) -> void:
	if _time_remaining > 0.0:
		_time_remaining -= delta
		if _time_remaining <= 0.0:
			clear()

func _unhandled_input(event: InputEvent) -> void:
	if not is_listening:
		return
	for action in TRACKED_ACTIONS:
		if event.is_action_pressed(action):
			# Single-slot overwrite: newer input (e.g. Deflect) replaces older Attack!
			buffered_action = action
			_time_remaining = buffer_duration
			break

# Called by AttackState when `open_combo_window()` or `open_deflect_window()` fires:
func consume(action: StringName) -> bool:
	if buffered_action == action and _time_remaining > 0.0:
		clear()
		return true
	return false

func consume_any() -> StringName:
	if buffered_action != &"" and _time_remaining > 0.0:
		var action := buffered_action
		clear()
		return action
	return &""

func clear() -> void:
	buffered_action = &""
	_time_remaining = 0.0
```

#### How Your `AttackState.gd` Uses It (3 Lines of Logic)

1. **At Frame `0` of `Slash_Kesa_01`:** `input_buffer.is_listening = false` and `input_buffer.clear()` _(prevents leftover double-clicks from Attack 1 spilling into Attack 2)_.
    
      
    
2. **At Frame `12` (Mid-Windup / Blade Lift):** Animation Method Call Track calls `input_buffer.is_listening = true` _(now the player can cleanly pre-buffer Attack 2 or an emergency Deflect)_.
    
      
    
3. **At Frame `27` (`open_combo_window()`):**
    
      
    
    GDScript
    
    ```
    func open_combo_window() -> void:
    	can_combo = true
    	var next_input := input_buffer.consume_any()
    	if next_input == &"deflect":
    		state_machine.transition_to("Deflect")
    	elif next_input == &"dodge":
    		state_machine.transition_to("Ghoststep")
    	elif next_input == &"attack" and current_attack.next_attack != null:
    		state_machine.transition_to("Attack", { "attack_data": current_attack.next_attack })
    ```
    

- **Why This Eliminates the "Disconnect" Completely:** Because `is_listening` ignores the first 12 frames of wind-up, `buffer_duration` expires stale presses after `180 ms`, and `consume_any()` lets `Deflect` or `Ghoststep` overwrite `Attack` if the player changes their mind mid-swing, the character **never** executes an action the player stopped wanting a quarter-second ago!

