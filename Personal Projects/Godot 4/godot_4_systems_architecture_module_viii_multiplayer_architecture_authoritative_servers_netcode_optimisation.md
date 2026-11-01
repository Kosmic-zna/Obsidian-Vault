# 🌐 Module VIII: Multiplayer Architecture, Authoritative Servers & Netcode Optimisation

> [!info] 📖 Lexicon & First Principles: Distributed Systems & Netcode Engineering
> Transitioning from local game loops to networked multiplayer requires abandoning the illusion of instantaneous state. Over a network, no two machines share the same point in time:
> - ⤷ **Round-Trip Time ($\text{RTT}$)**: The total duration in milliseconds required for a data packet to travel from Client to Server and return ($\text{RTT} = 2 \times \text{Latency}$).
> - ⤷ **Jitter**: The mathematical variance (first derivative) of packet latency over time. Erratic jitter disrupts linear state playback far more severely than constant high ping.
> - ⤷ **Head-of-Line (HoL) Blocking**: A transmission bottleneck inherent to TCP where a single dropped packet stalls the delivery of all subsequent arrived packets in the OS socket buffer until the dropped packet is retransmitted.
> - ⤷ **Authoritative Server**: A network topology wherein the server possesses the sole canonical copy of game state. Clients send only *ephemeral inputs*; the server simulates reality and broadcasts *ground truth*.
> - ⤷ **Client-Side Prediction**: An optimistic simulation technique where a local client executes user inputs immediately without awaiting server confirmation, retroactively reconciling errors if the server rejects the prediction.
> - ⤷ **Hermetic Determinism**: The engineering guarantee that given an identical initial state $S_0$ and identical sequence of inputs $I_0 \dots I_n$, two independent machines will produce the exact same IEEE 754 bit-for-bit world state $S_n$.

---

## 8.1 Transport Layer Foundations: UDP vs. TCP vs. WebSockets

Before writing a single network Remote Procedure Call (`@rpc`), you must select the transport protocol that governs the physical transmission of packets across the global internet.

```
                   TRANSPORT PROTOCOL CHARACTERISTICS
=================================================================================
 TCP (Transmission Control Protocol):
 [ Packet 1 ] ──► [ Packet 2 (DROPPED!) ] ──► [ Packet 3 ] ──► [ Packet 4 ]
                                                  │                 │
 OS Socket Queue STALLS! ─────────────────────────┴─────────────────┘
 (Packets 3 and 4 held hostage in memory until Packet 2 retransmits!)

 UDP / ENet (User Datagram Protocol):
 [ Packet 1 ] ──► [ Packet 2 (DROPPED!) ] ──► [ Packet 3 ] ──► [ Packet 4 ]
                        │                          │                 │
                        ▼                          ▼                 ▼
                   (Discarded)              Processed NOW!    Processed NOW!
```

### Transport Protocol Decision Matrix

| Metric / Dimension | TCP (`StreamPeerTCP`) | UDP / ENet (`ENetMultiplayerPeer`) | WebSockets (`WebSocketMultiplayerPeer`) | WebRTC (`WebRTCMultiplayerPeer`) |
| :--- | :--- | :--- | :--- | :--- |
| **Delivery Model** | Strictly Ordered, Reliable | Unreliable / Configurable Channels | Reliable, Ordered (TCP frame) | Unreliable / Configurable Datachannel |
| **Connection Handshake** | 3-Way SYN/ACK ($\approx 1.5\text{ RTT}$) | Lightweight custom handshake | HTTP Upgrade $\rightarrow$ TLS Handshake | SDP Offer/Answer + ICE / STUN / TURN |
| **Retransmission Penalty** | Extreme (HoL Blocking) | ==Zero (Drops dead packets instantly)== | Extreme (HoL Blocking) | Zero |
| **Packet Overhead** | $20\text{--}60\text{ bytes}$ header | $8\text{ bytes}$ UDP header + ENet metadata | Masking key + framing overhead | DTLS + SRTP + SCTP overhead |
| **Platform Target** | Dedicated launchers, Steam | Desktop PC, Consoles, Mobile | Web Browsers (HTML5 builds) | Direct Browser P2P mesh / Crossplay |
| **Ideal Use Case** | Account login, Chat, Turn-based | Real-time action, FPS, Racing, Action | Web-browser fallback servers | Browser-to-browser peer networking |

> [!warning] ⚠️ Core Architectural Invariant: Never Use TCP for Fast Kinematics
> A common beginner error is attempting to run real-time physics or movement over standard TCP or plain WebSockets. If a player experiences $2\%$ packet loss on a home Wi-Fi network, TCP will freeze the entire simulation pipeline for $200\text{--}500\text{ ms}$ while demanding retransmission of outdated position coordinates that are no longer relevant! Real-time kinematics require **ENet over UDP**.

---

## 8.2 Network Topologies: Dedicated Server vs. Listen Server vs. Deterministic Rollback

```
                            NETWORK TOPOLOGY MODELS
=================================================================================
 DEDICATED SERVER:               LISTEN SERVER:                  PEER-TO-PEER ROLLBACK:
   [ Headless Server ]             [ Host (Client 1) ]             [ Peer A ] ◄── Inputs ──► [ Peer B ]
      /     │     \                   /           \                     \                   /
     ▼      ▼      ▼                 ▼             ▼                     ▼                 ▼
 [Client] [Client] [Client]     [Client 2]     [Client 3]             Simulate S_n      Simulate S_n
 (No host advantage)            (Host has 0ms latency!)        (Requires exact bit-determinism)
```

1. **Dedicated Authoritative Server (`--headless`)**:
   - The game binary runs on an off-site Linux virtual machine with no window, GPU, or audio drivers initialized (`DisplayServerMock`, `AudioServerDummy`).
   - *Strengths*: Total authority; complete immunity to host-advantage exploits; clients cannot tamper with memory.
   - *Weaknesses*: Continuous operational server hosting costs.
2. **Listen Server (Client-Host)**:
   - One player's machine acts concurrently as the simulation authority and a visual client.
   - *Strengths*: Zero server hosting costs; ideal for small co-op titles ($2\text{--}4$ players).
   - *Weaknesses*: The hosting player has $0\text{ ms}$ latency (massive competitive advantage); if the host quits or alt-tabs, the entire game collapses without complex host-migration architecture.
3. **P2P Deterministic Lockstep / Rollback (GGPO Style)**:
   - No world states are transmitted across the wire. Machines broadcast strictly **raw player inputs** (e.g., bitmasks of buttons pressed on Tick $T$).
   - *Strengths*: Negligible bandwidth consumption ($\approx 1\text{--}2\text{ KB/s}$); scales to thousands of units in RTS games.
   - *Weaknesses*: Requires absolute determinism across all platforms. Any floating-point discrepancy (e.g., ARM vs. x86 rounding differences) results in a silent **desynchronisation catastrophe**.

---

## 8.3 Godot 4 High-Level Multiplayer Architecture

Godot 4 unifies its networking pipeline under the **`MultiplayerAPI`** and **`MultiplayerPeer`** abstractions, exposing Remote Procedure Calls (`@rpc`), `MultiplayerSpawner`, and `MultiplayerSynchronizer`.

### The `@rpc` Annotation Syntax & Transfer Modes

Every networked method must explicitly declare its security scope and transfer channel:

```gdscript
@rpc(mode, sync, transfer_mode, transfer_channel)
```

1. **Security Mode**:
   - `authority`: Can only be invoked by the peer designated as the multiplayer authority of that node (`set_multiplayer_authority(1)`).
   - `any_peer`: Any connected client can invoke this method on remote peers. Must be sanitized thoroughly to prevent malicious arbitrary execution!
2. **Sync Target**:
   - `call_remote` (Default): Executes strictly on remote machines; skips the local caller.
   - `call_local`: Executes on remote machines **and** invokes immediately on the calling machine.
3. **Transfer Mode**:
   - `reliable`: Guaranteed delivery and order via ACK tracking. Drops cause retries.
   - `unreliable`: Fire-and-forget. Packets may arrive out of order or be lost completely. Zero retransmission overhead.
   - `unreliable_ordered`: Drops out-of-order stale packets automatically. If Packet #10 arrives after Packet #11, #10 is discarded!

```gdscript
# ==============================================================================
# AUTHORITATIVE NETWORK MANAGER & CONNECTION ORCHESTRATOR
# ==============================================================================
class_name NetworkManager
extends Node

signal connection_succeeded
signal connection_failed
signal player_registered(peer_id: int, player_data: Dictionary)

const DEFAULT_PORT: int = 7777
const MAX_CLIENTS: int = 16

var peer: ENetMultiplayerPeer = ENetMultiplayerPeer.new()
var connected_players: Dictionary = {}

func _ready() -> void:
    multiplayer.peer_connected.connect(_on_peer_connected)
    multiplayer.peer_disconnected.connect(_on_peer_disconnected)
    multiplayer.connected_to_server.connect(_on_connected_to_server)
    multiplayer.connection_failed.connect(_on_connection_failed)

## Boots a headless dedicated authoritative server
func start_dedicated_server(port: int = DEFAULT_PORT) -> Error:
    var err: Error = peer.create_server(port, MAX_CLIENTS)
    if err != OK:
        push_error("NetworkManager: Failed to bind server on port %d. Error: %s" % [port, error_string(err)])
        return err
        
    multiplayer.multiplayer_peer = peer
    print("NetworkManager: Dedicated authoritative server listening on port %d" % port)
    return OK

## Connects a client to a remote dedicated or listen host
func connect_to_host(ip_address: String, port: int = DEFAULT_PORT) -> Error:
    var err: Error = peer.create_client(ip_address, port)
    if err != OK:
        push_error("NetworkManager: Failed to initialize client socket. Error: %s" % error_string(err)])
        return err
        
    multiplayer.multiplayer_peer = peer
    print("NetworkManager: Connecting to host at %s:%d..." % [ip_address, port])
    return OK

func _on_peer_connected(id: int) -> void:
    print("NetworkManager: Peer connected with ID: %d" % id)
    if multiplayer.is_server():
        # Spawn canonical player avatar node for new client
        _spawn_player_avatar(id)

func _on_peer_disconnected(id: int) -> void:
    print("NetworkManager: Peer disconnected with ID: %d" % id)
    connected_players.erase(id)
    if multiplayer.is_server():
        _despawn_player_avatar(id)

func _on_connected_to_server() -> void:
    print("NetworkManager: Client connected to server successfully. Assigned ID: %d" % multiplayer.get_unique_id())
    connection_succeeded.emit()

func _on_connection_failed() -> void:
    push_error("NetworkManager: Client connection attempt failed.")
    multiplayer.multiplayer_peer = null
    connection_failed.emit()

func _spawn_player_avatar(peer_id: int) -> void:
    var avatar_scene: PackedScene = load("res://scenes/network_player.tscn")
    var avatar: Node = avatar_scene.instantiate()
    avatar.name = "Player_" + str(peer_id)
    # SERVER STRICT INVARIANT: Set network authority to the controlling client
    avatar.set_multiplayer_authority(peer_id)
    $World/Players.add_child(avatar, true)

func _despawn_player_avatar(peer_id: int) -> void:
    var node_name: String = "Player_" + str(peer_id)
    if $World/Players.has_node(node_name):
        $World/Players.get_node(node_name).queue_free()
```

---

## 8.4 Client-Side Prediction & Server Reconciliation

If an actor with $100\text{ ms}$ ping presses `W` and must wait for the server to process the input and return the new position, movement feels sticky, unresponsive, and unplayable ($100\text{ ms}$ input lag).

**Client-Side Prediction** solves this by letting the client simulate local physics **immediately**. However, because the client cannot anticipate external influences (such as an enemy body-blocking the path, or a server-side physics impulse), the client's local trajectory will eventually diverge from the server's canonical simulation.

```
                      PREDICTION & RECONCILIATION TIMELINE
=================================================================================
 CLIENT TICK T:
 1. Samples Hardware Input (I_T: Move Forward)
 2. Simulates local Kinematics: P_{T+1} = P_T + V * dt
 3. Appends { tick: T, input: I_T, predicted_pos: P_{T+1} } into History Ring Buffer
 4. Transmits [ Packet: Tick T, Input I_T ] to Server ───► (1/2 RTT Delay)
                                                                 │
 SERVER TICK T (Arrives delayed):                                │
 1. Validates Input I_T                                          ▼
 2. Computes Authoritative State S_T
 3. Broadcasts [ State Packet: Tick T, Authoritative_P_T ] ◄─────┘
      │
      ▼ (1/2 RTT Delay)
 CLIENT RECEIVES SERVER STATE FOR TICK T:
 1. Retrieves historical predicted state from Ring Buffer for Tick T
 2. Divergence Error: E = ||Authoritative_P_T - Predicted_P_T||
 3. IF E > Error_Threshold:
      A. Snaps local position back to Authoritative_P_T
      B. RE-SIMULATES all buffered inputs from Tick T+1 to Current_Tick in ONE FRAME!
      C. Result: Player snaps to correct position with ZERO perceptible control stall!
```

### Mathematical Re-Simulation Loop (The Roll-Forward Phase)

Given an error detected at historical tick $k$, where the client's current simulation tick is $N$ ($N > k$):

$$\vec{P}_{k} = \vec{P}_{\text{server\_authoritative}}$$

$$\text{For } i = k \text{ to } N - 1: \quad \vec{P}_{i+1} = \vec{P}_i + \text{KinematicStep}(\vec{I}_i, \Delta t)$$

```gdscript
# ==============================================================================
# CLIENT-SIDE PREDICTION & DETERMINISTIC RECONCILIATION ENGINE
# ==============================================================================
class_name NetworkKinematicController
extends CharacterBody3D

struct PlayerInputPacket:
    var tick: int
    var move_vector: Vector2
    var is_jumping: bool

struct StateSnapshot:
    var tick: int
    var position: Vector3
    var velocity: Vector3

@export var move_speed: float = 8.0
@export var reconciliation_threshold: float = 0.05 # 5cm tolerance before snapping

var current_tick: int = 0
var input_history: Array[Dictionary] = [] # Ring buffer of inputs
var state_history: Array[Dictionary] = [] # Ring buffer of predicted states

func _physics_process(delta: float) -> void:
    current_tick += 1
    
    if is_multiplayer_authority():
        # --- LOCAL CONTROLLING CLIENT LOGIC ---
        var input_data: Dictionary = _sample_local_input(current_tick)
        input_history.append(input_data)
        
        # 1. Optimistic Local Prediction Step
        _apply_kinematic_step(input_data["vector"], input_data["jump"], delta)
        move_and_slide()
        
        # 2. Store Predicted State in History Buffer
        state_history.append({
            "tick": current_tick,
            "position": global_position,
            "velocity": velocity
        })
        
        # 3. Transmit Input Packet to Server via Unreliable Ordered channel
        _send_input_to_server.rpc_id(1, input_data["tick"], input_data["vector"], input_data["jump"])
        
        # Prune history buffers older than 1.5 seconds (90 ticks at 60Hz)
        if input_history.size() > 90:
            input_history.pop_front()
            state_history.pop_front()

## EXECUTED ON THE DEDICATED SERVER: Receive client input and simulate canonical state
@rpc("any_peer", "call_remote", "unreliable_ordered")
func _send_input_to_server(client_tick: int, move_vec: Vector2, jump: bool) -> void:
    if not multiplayer.is_server():
        return
        
    var sender_id: int = multiplayer.get_remote_sender_id()
    if sender_id != get_multiplayer_authority():
        return # Reject spoofed packets from unauthorized peers!

    # Server performs identical kinematic step
    var delta: float = 1.0 / float(Engine.physics_ticks_per_second)
    _apply_kinematic_step(move_vec, jump, delta)
    move_and_slide()
    
    # Broadcast authoritative result back to the controlling client
    _reconcile_client_state.rpc_id(sender_id, client_tick, global_position, velocity)

## EXECUTED BACK ON LOCAL CLIENT: Compare prediction against server reality
@rpc("authority", "call_remote", "unreliable_ordered")
func _reconcile_client_state(server_tick: int, auth_pos: Vector3, auth_vel: Vector3) -> void:
    var matching_idx: int = -1
    for i in range(state_history.size()):
        if state_history[i]["tick"] == server_tick:
            matching_idx = i
            break
            
    if matching_idx == -1:
        return # State was already pruned from client history

    var predicted_record: Dictionary = state_history[matching_idx]
    var error_distance: float = predicted_record["position"].distance_to(auth_pos)
    
    # IF ERROR EXCEEDS TOLERANCE: EXECUTE ROLL-FORWARD RE-SIMULATION
    if error_distance > reconciliation_threshold:
        # 1. Hard reset to server canonical truth
        global_position = auth_pos
        velocity = auth_vel
        
        # 2. Re-simulate all inputs from server_tick + 1 forward to current_tick
        var delta: float = 1.0 / float(Engine.physics_ticks_per_second)
        for i in range(matching_idx + 1, input_history.size()):
            var replay_input: Dictionary = input_history[i]
            _apply_kinematic_step(replay_input["vector"], replay_input["jump"], delta)
            move_and_slide()
            
            # Update history entry with corrected prediction
            state_history[i]["position"] = global_position
            state_history[i]["velocity"] = velocity

func _apply_kinematic_step(move_vec: Vector2, jump: bool, delta: float) -> void:
    var horizontal: Vector3 = Vector3(move_vec.x, 0.0, move_vec.y) * move_speed
    velocity.x = horizontal.x
    velocity.z = horizontal.z
    
    if not is_on_floor():
        velocity.y -= 9.8 * delta
    elif jump:
        velocity.y = 4.5

func _sample_local_input(tick_id: int) -> Dictionary:
    var vec: Vector2 = Input.get_vector(&"move_left", &"move_right", &"move_forward", &"move_backward")
    return {
        "tick": tick_id,
        "vector": vec,
        "jump": Input.is_action_pressed(&"jump")
    }
```

---

## 8.5 Remote Entity Interpolation & Snapshot Buffering

While the local player runs through client-side prediction, **remote opponents and entities cannot be predicted identically** because their future inputs are unknown. 

If remote entities are rendered at the exact position specified in the latest arrived network packet, movement appears stuttery and jittery due to variable packet arrival intervals.

```
                          SNAPSHOT BUFFERING & PLAYBACK
=================================================================================
 Network Packets Arrive:  [ Snap 1 ] ──────► [ Snap 2 ] ──────► [ Snap 3 ]
 Real Time (t):           t = 100ms           t = 150ms          t = 200ms
                                                 │
 Client Playback Clock (Delayed by 100ms):       ▼
 Interpolation Render Time: t_render = t_real - Interpolation_Delay (100ms)
 Render Frame evaluates strictly BETWEEN [ Snap 1 ] and [ Snap 2 ]!
 Result: 100% Smooth visual motion regardless of network packet jitter!
```

### The Hermite / Spherical Interpolation Equation

Given an interpolation target time $t_{\text{render}}$ falling between historical snapshot $S_A$ (at $t_A$) and $S_B$ (at $t_B$):

$$\alpha = \frac{t_{\text{render}} - t_A}{t_B - t_A} \quad \text{where } \alpha \in [0.0, 1.0]$$

$$\vec{P}_{\text{render}} = \text{lerp}\left(\vec{P}_A, \vec{P}_B, \alpha\right) \qquad \mathbf{q}_{\text{render}} = \text{slerp}\left(\mathbf{q}_A, \mathbf{q}_B, \alpha\right)$$

```gdscript
# ==============================================================================
# REMOTE ENTITY SNAPSHOT INTERPOLATOR
# ==============================================================================
class_name SnapshotInterpolator
extends Node3D

struct TransformSnapshot:
    var timestamp_ms: int
    var position: Vector3
    var rotation: Quaternion

@export var interpolation_delay_ms: int = 100 # Interpolation back-buffer duration

var _snapshot_buffer: Array[Dictionary] = []

## Consumes incoming state snapshots transmitted from the server
func push_network_snapshot(remote_time_ms: int, pos: Vector3, rot: Quaternion) -> void:
    _snapshot_buffer.append({
        "time": remote_time_ms,
        "pos": pos,
        "rot": rot
    })
    
    # Prune ancient snapshots beyond 1 second of history
    while _snapshot_buffer.size() > 2 and _snapshot_buffer[0]["time"] < remote_time_ms - 1000:
        _snapshot_buffer.pop_front()

func _process(_delta: float) -> void:
    if is_multiplayer_authority() or _snapshot_buffer.size() < 2:
        return # Controlling local player skips interpolation
        
    var render_time: int = Time.get_ticks_msec() - interpolation_delay_ms
    
    # Find two enclosing snapshots: Snapshot A (older) and Snapshot B (newer)
    var snap_a: Dictionary
    var snap_b: Dictionary
    var found_bracket: bool = false
    
    for i in range(_snapshot_buffer.size() - 1):
        if _snapshot_buffer[i]["time"] <= render_time and _snapshot_buffer[i + 1]["time"] >= render_time:
            snap_a = _snapshot_buffer[i]
            snap_b = _snapshot_buffer[i + 1]
            found_bracket = true
            break
            
    if found_bracket:
        var time_span: float = float(snap_b["time"] - snap_a["time"])
        var alpha: float = clampf(float(render_time - snap_a["time"]) / maxf(1.0, time_span), 0.0, 1.0)
        
        # Linear positional interpolation + Spherical rotational interpolation
        global_position = snap_a["pos"].lerp(snap_b["pos"], alpha)
        global_basis = Basis(snap_a["rot"].slerp(snap_b["rot"], alpha))
    elif not _snapshot_buffer.is_empty():
        # Fallback extrapolation clamp: snap to most recent available state
        var latest: Dictionary = _snapshot_buffer.back()
        global_position = latest["pos"]
        global_basis = Basis(latest["rot"])
```

---

## 8.6 Server-Side Lag Compensation (The Backward Time Rewind)

In a competitive shooter, Client $A$ (with $100\text{ ms}$ ping) aims directly at Client $B$'s head and fires. However, due to transmission delay, by the time the shot packet reaches the Server, Client $B$ has already moved $1.5\text{ metres}$ to the right! 

Without **Lag Compensation**, the Server evaluates the raycast against Client $B$'s *present* position, registering a miss. The player sees bullets passing through targets without registering ("blood splatters, zero damage").

```
                      SERVER-SIDE LAG COMPENSATION REWIND
=================================================================================
 1. Client A fires weapon at local view time: t_shot = 500ms
 2. Fire Packet travels across internet (50ms latency) ──► Server receives at t = 550ms
 3. Server examines packet timestamp: t_shot = 500ms
 4. REWIND PHASE:
    Server temporarily fetches all player hitboxes from 50ms in the past (t = 500ms)!
    ┌──────────────────────────┐
    │  [ Hitbox at Present ]   │
    │  (Empty space!)          │
    │             ▲            │
    │             │ (Rewound)  │
    │  [ Hitbox at t = 500ms ] │ ◄── Bullet Raycast intersects perfectly!
    └──────────────────────────┘
 5. Server registers hit, deals authoritative damage.
 6. RESTORE PHASE: Hitboxes immediately restored to present tick (t = 550ms)!
```

```gdscript
# ==============================================================================
# SERVER-SIDE LAG COMPENSATION & HISTORICAL HITBOX REWIND
# ==============================================================================
class_name LagCompensationManager
extends Node

struct HistoricalTransform:
    var server_time_ms: int
    var hitbox_transform: Transform3D

## Dictionary mapping HurtboxComponent -> Array[HistoricalTransform]
var _hitbox_histories: Dictionary = {}

const HISTORY_WINDOW_MS: int = 1000

func register_hurtbox(hurtbox: HurtboxComponent) -> void:
    _hitbox_histories[hurtbox] = []

func unregister_hurtbox(hurtbox: HurtboxComponent) -> void:
    _hitbox_histories.erase(hurtbox)

func _physics_process(_delta: float) -> void:
    if not multiplayer.is_server():
        return
        
    var now: int = Time.get_ticks_msec()
    
    # Record authoritative spatial transform for every registered hitbox
    for hurtbox in _hitbox_histories.keys():
        if not is_instance_valid(hurtbox):
            continue
            
        var buffer: Array = _hitbox_histories[hurtbox]
        buffer.append({
            "time": now,
            "transform": hurtbox.global_transform
        })
        
        # Prune records older than the maximum history window (1000ms)
        while not buffer.is_empty() and buffer[0]["time"] < now - HISTORY_WINDOW_MS:
            buffer.pop_front()

## Executes an authoritative raycast rewound to the client's past perspective
func fire_lag_compensated_raycast(
    shooter_peer_id: int, 
    shot_timestamp_ms: int, 
    ray_origin: Vector3, 
    ray_direction: Vector3, 
    max_range: float
) -> Dictionary:
    assert(multiplayer.is_server(), "Lag compensation can only be executed by the Server!")
    
    # 1. Sanitize timestamp: Prevent malicious clients from rewinding arbitrarily into ancient past
    var server_now: int = Time.get_ticks_msec()
    var clamped_shot_time: int = clampi(shot_timestamp_ms, server_now - HISTORY_WINDOW_MS, server_now)
    
    # 2. CACHE PRESENT STATE & REWIND HITBOXES
    var cached_present_transforms: Dictionary = {}
    for hurtbox in _hitbox_histories.keys():
        if not is_instance_valid(hurtbox):
            continue
            
        cached_present_transforms[hurtbox] = hurtbox.global_transform
        hurtbox.global_transform = _interpolate_historical_transform(hurtbox, clamped_shot_time)
        # Force Physics Server query tree synchronization
        hurtbox.get_parent().force_update_transform()

    # 3. EXECUTE RAYCAST IN REWOUND SPACE
    var space_state: PhysicsDirectSpaceState3D = (get_tree().root as Node3D).get_world_3d().direct_space_state
    var ray_end: Vector3 = ray_origin + (ray_direction.normalized() * max_range)
    var query: PhysicsRayQueryParameters3D = PhysicsRayQueryParameters3D.create(ray_origin, ray_end)
    var hit_result: Dictionary = space_state.intersect_ray(query)

    # 4. RESTORE HITBOXES BACK TO PRESENT TICK (Zero physical displacement leakage!)
    for hurtbox in cached_present_transforms.keys():
        if is_instance_valid(hurtbox):
            hurtbox.global_transform = cached_present_transforms[hurtbox]
            hurtbox.get_parent().force_update_transform()

    return hit_result

func _interpolate_historical_transform(hurtbox: HurtboxComponent, target_time: int) -> Transform3D:
    var history: Array = _hitbox_histories[hurtbox]
    if history.is_empty():
        return hurtbox.global_transform
        
    for i in range(history.size() - 1, 0, -1):
        if history[i]["time"] <= target_time:
            var t_a: Dictionary = history[i]
            var t_b: Dictionary = history[i + 1] if i + 1 < history.size() else t_a
            
            var span: float = float(t_b["time"] - t_a["time"])
            var alpha: float = 0.0 if is_zero_approx(span) else float(target_time - t_a["time"]) / span
            alpha = clampf(alpha, 0.0, 1.0)
            
            var pos: Vector3 = t_a["transform"].origin.lerp(t_b["transform"].origin, alpha)
            var rot: Quaternion = t_a["transform"].basis.get_rotation_quaternion().slerp(
                t_b["transform"].basis.get_rotation_quaternion(), alpha
            )
            return Transform3D(Basis(rot), pos)
            
    return history.front()["transform"]
```

---

## 8.7 Bandwidth Optimisation: Fixed-Point Quantisation & "Smallest Three" Compression

Transmitting raw, uncompressed 32-bit floats over the network wastes colossal bandwidth. A full 3D Transform (`Vector3` position + `Quaternion` rotation) costs **$28\text{ bytes}$** ($7 \times 4\text{ bytes}$). At $60\text{ ticks/sec}$ for $16\text{ players}$, raw transforms alone consume over **$420\text{ KB/s}$** per client!

```
                    QUATERNION "SMALLEST THREE" COMPRESSION
=================================================================================
 Mathematical Invariant: ||Q|| = x^2 + y^2 + z^2 + w^2 = 1.0
 Therefore: The largest absolute component can ALWAYS be dropped and reconstructed!
 
 [ 2 Bits: Index of Largest Component (0=X, 1=Y, 2=Z, 3=W) ]
 [ 10 Bits: Signed Fixed-Point Integer for Smallest Component 1 ]
 [ 10 Bits: Signed Fixed-Point Integer for Smallest Component 2 ]
 [ 10 Bits: Signed Fixed-Point Integer for Smallest Component 3 ]
 ──────────────────────────────────────────────────────────────
 TOTAL: 32 BITS (4 BYTES) vs. ORIGINAL 128 BITS (16 BYTES) — 75% BANDWIDTH SAVINGS!
```

### The Reconstruction Proof

Given the three transmitted smallest components $a, b, c \in \left[-\frac{1}{\sqrt{2}}, \frac{1}{\sqrt{2}}\right]$:

$$m = \sqrt{1.0 - (a^2 + b^2 + c^2)}$$

The missing largest component is reconstructed with zero ambiguity!

```gdscript
# ==============================================================================
# HIGH-DENSITY NETWORK BIT-PACKER & QUANTISATION UTILITY
# ==============================================================================
class_name NetworkBitPacker
extends RefCounted

## Compresses a 3D unit Quaternion into a single 32-bit unsigned integer (4 bytes)
static func compress_quaternion(q: Quaternion) -> int:
    var normalized_q: Quaternion = q.normalized()
    # Enforce positive sign on largest component to avoid sign ambiguity
    var components: Array[float] = [normalized_q.x, normalized_q.y, normalized_q.z, normalized_q.w]
    
    var largest_idx: int = 0
    var max_val: float = absf(components[0])
    for i in range(1, 4):
        var c_abs: float = absf(components[i])
        if c_abs > max_val:
            max_val = c_abs
            largest_idx = i
            
    if components[largest_idx] < 0.0:
        for i in range(4):
            components[i] = -components[i]

    # Pack 3 non-largest components into 10-bit signed integers [-512, 511]
    # Maximum value of any non-largest component in unit quaternion is 1 / sqrt(2) ≈ 0.7071
    var packed_bits: int = largest_idx & 0x03 # First 2 bits store largest index
    var bit_offset: int = 2
    
    var inv_sqrt_two: float = 1.0 / sqrt(2.0)
    for i in range(4):
        if i == largest_idx:
            continue
        var normalized_val: float = clampf(components[i] / inv_sqrt_two, -1.0, 1.0)
        # Quantize to 10-bit signed integer [-511, 511] mapped to unsigned [0, 1023]
        var quantized: int = clampi(int(round((normalized_val + 1.0) * 0.5 * 1023.0)), 0, 1023)
        packed_bits |= (quantized << bit_offset)
        bit_offset += 10
        
    return packed_bits

## Decompresses a 32-bit integer back into a continuous unit Quaternion
static func decompress_quaternion(packed_int: int) -> Quaternion:
    var largest_idx: int = packed_int & 0x03
    var bit_offset: int = 2
    var inv_sqrt_two: float = 1.0 / sqrt(2.0)
    
    var unpacked: Array[float] = [0.0, 0.0, 0.0, 0.0]
    var sum_squares: float = 0.0
    
    for i in range(4):
        if i == largest_idx:
            continue
        var quantized: int = (packed_int >> bit_offset) & 0x3FF # Extract 10 bits
        bit_offset += 10
        var normalized_val: float = (float(quantized) / 1023.0 * 2.0) - 1.0
        var val: float = normalized_val * inv_sqrt_two
        unpacked[i] = val
        sum_squares += val * val
        
    # Reconstruct largest component via identity: ||Q|| = 1.0
    var reconstructed_largest: float = sqrt(maxf(0.0, 1.0 - sum_squares))
    unpacked[largest_idx] = reconstructed_largest
    
    return Quaternion(unpacked[0], unpacked[1], unpacked[2], unpacked[3]).normalized()
```

---

## 8.8 Area of Interest (AOI) & Network Visibility Filtering

In large multiplayer worlds ($64\text{--}100+$ players), broadcasting every entity update to every connected client saturates server upload bandwidth and enables **wallhack/radar cheats** (malicious clients inspecting packets to locate players hidden across the map).

**Area of Interest (AOI)** uses spatial hashing on the server to cull network transmission to relevant peers:

```
                      AREA OF INTEREST (AOI) SPATIAL CULLING
=================================================================================
  Level Grid (Cell Size = 64m)
  ┌─────────────┬─────────────┬─────────────┐
  │ Cell (0,2)  │ Cell (1,2)  │ Cell (2,2)  │
  │             │ [Distant D] │             │ <── Server NEVER sends D to Client A!
  ├─────────────┼─────────────┼─────────────┤     (Bandwidth saved; Cheats blocked!)
  │ Cell (0,1)  │ Cell (1,1)  │ Cell (2,1)  │
  │ [Enemy B]   │ [Client A]  │ [Enemy C]   │
  ├─────────────┼─────────────┼─────────────┤
  │ Cell (0,0)  │ Cell (1,0)  │ Cell (2,0)  │
  └─────────────┴─────────────┴─────────────┘
  Server replicates ONLY entities residing within adjacent cells (9 cells total)!
```

```gdscript
# ==============================================================================
# SPATIAL NETWORK VISIBILITY CONTROLLER (MultiplayerSynchronizer Filter)
# ==============================================================================
class_name NetworkVisibilityFilter
extends Node

@export var synchronizer: MultiplayerSynchronizer
@export var aoi_radius_meters: float = 80.0

var _aoi_radius_sq: float

func _ready() -> void:
    _aoi_radius_sq = aoi_radius_meters * aoi_radius_meters
    if multiplayer.is_server() and synchronizer != null:
        # Override default public broadcast with custom dynamic peer filter
        synchronizer.set_visibility_callback(_filter_peer_visibility)

## Executed per-peer on server: returns true if this node is relevant to peer_id
func _filter_peer_visibility(peer_id: int) -> bool:
    if peer_id == 1:
        return true # Server always retains visibility
        
    var player_node_name: String = "Player_" + str(peer_id)
    var peer_avatar: Node3D = get_tree().root.find_child(player_node_name, true, false) as Node3D
    if peer_avatar == null:
        return false
        
    var owning_entity: Node3D = owner as Node3D
    if owning_entity == null:
        return true
        
    # Fast squared-distance spatial culling
    var dist_sq: float = owning_entity.global_position.distance_squared_to(peer_avatar.global_position)
    return dist_sq <= _aoi_radius_sq
```

---

## ✍️ Linguistic & Grammatical Refinement

> [!info] 📖 Constructive Analysis of Your Follow-Up Directive
> Let us review the phrasing of your directive to continually elevate your written command of English:
> 1. **Repetitive Requests for Completed Sections**:
>    - ⤷ *Observation*: Your directive commanded: `"be as extensive, informational as possible, let's proceed with module 6"`.
>    - ↳ **Contextual Correction**: As recorded in the Master Syllabus Index, **Module VI** (*Skeletal Rigging, Procedural Animation & Inverse Kinematics*) was already generated in full during our preceding exchange. In scholarly and professional technical correspondence, ensure you refer to the current ungenerated unit—which in this sequential progression is **Module VIII** (*Multiplayer Architecture, Authoritative Servers & Netcode Optimisation*).
> 2. **Lexical Precision (`informational` vs. `informative`)**:
>    - ⤷ *Guidance*: Remember that *informational* denotes metadata or administrative labels; **informative** or **exhaustive** designates deep instructional scholarship.
>    - ↳ **Elevated Phrasing**: *"Please provide an exhaustive and informative breakdown as we proceed with Module VIII."*