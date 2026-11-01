# ⚡ Module VII: Diagnostics, Profiling, Optimisation & Edge-Case Engineering

> [!info] 📖 Lexicon & First Principles: Performance & Runtime Diagnostics
> Transitioning from amateur scripting to senior systems architecture requires diagnosing computational bottlenecks at the hardware, memory, and engine server layers:
> - ⤷ **Frame Budget**: The total window of real time ($\Delta t$) available to execute all simulation logic, physics integration, audio processing, driver synchronization, and GPU rasterisation before a frame drop occurs.
> - ⤷ **Frame-Time Pacing (Jitter)**: Irregular variance in the duration of individual frames. A game running at an average of $60\text{ FPS}$ with erratic frame times ($8\text{ ms} \rightarrow 32\text{ ms} \rightarrow 5\text{ ms}$) feels noticeably stuttery compared to a locked, uniform $16.66\text{ ms}$ delivery.
> - ⤷ **Spatial Hash Grid**: An associative data structure that maps continuous 2D/3D Euclidean coordinates into discrete, integer-keyed spatial buckets, reducing pair-wise proximity testing complexity from quadratic to near-linear scale.
> - ⤷ **Orphan Node**: A heap-allocated `Node` instance detached from the `SceneTree` via `remove_child()` but never explicitly freed via `queue_free()`. It lingers permanently in RAM, leaking memory and native resources.
> - ⤷ **WorkerThreadPool**: Godot 4's native multi-threaded task scheduler that distributes arbitrary, thread-safe computational jobs across all available CPU host cores.
> - ⤷ **IEEE 754 Representation**: The international standard for binary floating-point computation, which represents real numbers using fixed bit fields (sign, biased exponent, mantissa), inherently causing rounding discrepancies in base-10 fractional math.

---

## 7.1 The $16.67\text{ ms}$ Frame Budget & Profiler Diagnostics

The primary constraint of real-time software is that physical time never stops. If your game targets a fluid refresh rate, every subsystem must complete its work within a hard temporal budget:

$$\text{Budget} = \frac{1000\text{ ms}}{\text{Target FPS}} \qquad \implies \qquad 60\text{ FPS} \approx 16.67\text{ ms}, \quad 144\text{ FPS} \approx 6.94\text{ ms}$$

```
                       THE 16.67ms FRAME TIME BUDGET SLICE
=================================================================================
 [ Engine Overhead & OS Events: ~1.5ms ]
 [ Fixed Physics Step (_physics_process): ~3.0ms ]
 [ Gameplay Scripting & AI Logic (_process): ~4.0ms ]
 [ SceneTree Updates, Visibility & Culling: ~2.0ms ]
 [ Vulkan Draw-Call Assembly & State Dispatch: ~2.5ms ]
 [ GPU Fragment Shading, Lighting & Post-Processing: ~3.67ms ]
 ───────────────────────────────────────────────────────────────────────────────
 TOTAL TIME CONSUMED: 16.67ms (Target met. Exceeding this causes dropped frames)
```

### The Analytical Diagnostic Workflow

Never refactor code based on intuition or unmeasured speculation. Profile using Godot's built-in **Profiler**, **Visual Profiler**, and **Monitors**:

```
                       PROFILER TRIAGE DECISION TREE
=================================================================================
 Is Frame Time > 16.67ms?
    │
    ├── YES ──► Check "Process Time" vs. "Physics Time"
    │             │
    │             ├── Process Spike ──► Inspect GDScript Profiler:
    │             │                     Identify functions with high "Self Time"
    │             │                     or calls exceeding 1,000/frame.
    │             │
    │             └── Physics Spike ──► Check Physics 2D/3D Monitors:
    │                                   Are Active Objects or Collision Pairs
    │                                   scaling quadratically?
    │
    └── NO  ──► Inspect "Frame-Time Graph" for periodic spikes:
                  │
                  ├── Regular 1-second cadence ──► Heap allocations (Garbage collection)
                  └── Sporadic multi-frame stalls ─► Resource loading / Pipeline compilation
```

1. **Inclusive Time vs. Self Time**:
   - ⤷ **Inclusive Time**: The total time spent inside a function *plus* all child functions and engine methods it calls.
   - ⤷ **Self Time**: The time spent *strictly* inside the function's own local instructions. ==High Self Time highlights algorithmic inefficiencies== within that specific function body.
2. **Visual Profiler**:
   - Isolates CPU culling, shadow pass generation, transparent object passes, and post-processing passes. If the shadow pass consumes $> 4\text{ ms}$, adjust shadow cascade distances and directional shadow resolutions rather than rewriting GDScript.

---

## 7.2 Algorithmic Complexity (Big-$\mathcal{O}$) in Real-Time Loops

Amateur code often runs smoothly in testing with $20$ entities, only to slow down in production when handling $1,000$ entities. This slowdown stems directly from asymptotic complexity scaling.

```
                    BIG-O SCALABILITY COMPARISON MATRIX
┌───────────────┬─────────────────┬───────────────────┬─────────────────────────┐
│ Complexity    │ Class Name      │ $N = 50$ Ops      │ $N = 2,000$ Ops         │
├───────────────┼─────────────────┼───────────────────┼─────────────────────────┤
│ $\mathcal{O}(1)$       │ Constant        │ $1$ op            │ $1$ op (Hash map)       │
│ $\mathcal{O}(\log N)$   │ Logarithmic     │ $\approx 6$ ops   │ $\approx 11$ ops (Tree) │
│ $\mathcal{O}(N)$       │ Linear          │ $50$ ops          │ $2,000$ ops (Single loop)│
│ $\mathcal{O}(N \log N)$│ Linearithmic    │ $\approx 282$ ops │ $\approx 21,931$ ops    │
│ $\mathcal{O}(N^2)$     │ Quadratic       │ $2,500$ ops       │ $4,000,000$ ops 🚨 FAIL │
└───────────────┴─────────────────┴───────────────────┴─────────────────────────┘
```

### The $N^2$ Proximity Detection Anti-Pattern

Checking whether any entity in the game is within melee range of another entity using nested loops is a common source of quadratic slowdown:

```gdscript
# ANTI-PATTERN: O(N^2) Nested Spatial Testing
# 1,000 entities = 1,000,000 distance calculations EVERY frame!
func _process(delta: float) -> void:
    for i in range(entities.size()):
        for j in range(i + 1, entities.size()):
            var dist_sq: float = entities[i].global_position.distance_squared_to(entities[j].global_position)
            if dist_sq < 4.0:
                resolve_overlap(entities[i], entities[j])
```

### The Solution: 2D/3D Spatial Hash Partitioning

A **Spatial Hash Grid** subdivides continuous Euclidean space into a virtual grid of uniform cells. Instead of comparing an entity against every other entity in the level, it registers each entity into a cell coordinate based on its position:

$$\text{CellX} = \lfloor x / S \rfloor, \quad \text{CellY} = \lfloor y / S \rfloor, \quad \text{CellZ} = \lfloor z / S \rfloor$$

where $S$ is the cell size (typically chosen as twice the maximum query radius). Proximity searches then query only the entity's current cell and its immediate neighbours ($9$ cells in 2D, $27$ in 3D), bringing average complexity down to near-linear $\mathcal{O}(N)$ performance:

```
                      SPATIAL HASH GRID PARTITIONING
=================================================================================
    Grid Cell Size S = 10m
    ┌─────────────┬─────────────┬─────────────┐
    │ Cell (-1,1) │ Cell (0,1)  │ Cell (1,1)  │
    │             │   [Entity B]│             │
    ├─────────────┼─────────────┼─────────────┤
    │ Cell (-1,0) │ Cell (0,0)  │ Cell (1,0)  │
    │             │  [Player]   │   [Entity C]│
    ├─────────────┼─────────────┼─────────────┤
    │ Cell (-1,-1)│ Cell (0,-1) │ Cell (1,-1) │
    │ [Distant D] │             │             │
    └─────────────┴─────────────┴─────────────┘
    Player queries ONLY adjacent 9 cells. Distant Entity D is NEVER evaluated!
```

```gdscript
# ==============================================================================
# HIGH-SPEED 2D SPATIAL HASH GRID (Near O(1) Localised Queries)
# ==============================================================================
class_name SpatialHashGrid2D
extends RefCounted

var _cell_size: float
var _inv_cell_size: float
## Dictionary mapping Vector2i (cell coordinate) -> Array[Node2D]
var _grid: Dictionary = {}

func _init(cell_size: float = 64.0) -> void:
    assert(cell_size > 0.0, "Cell size must be strictly positive.")
    _cell_size = cell_size
    _inv_cell_size = 1.0 / cell_size

## Clears and rebuilds the grid allocations in a single frame pass
func clear() -> void:
    _grid.clear()

## Maps a continuous spatial coordinate to an integer cell coordinate
func world_to_grid(world_pos: Vector2) -> Vector2i:
    return Vector2i(
        int(floor(world_pos.x * _inv_cell_size)),
        int(floor(world_pos.y * _inv_cell_size))
    )

## Inserts an entity into its corresponding grid cell
func insert(entity: Node2D) -> void:
    var cell: Vector2i = world_to_grid(entity.global_position)
    if not _grid.has(cell):
        var bucket: Array[Node2D] = []
        _grid[cell] = bucket
    _grid[cell].append(entity)

## Queries entities residing within the cell and its 8 adjacent neighbours
func query_radius(world_pos: Vector2, radius: float) -> Array[Node2D]:
    var results: Array[Node2D] = []
    var radius_sq: float = radius * radius
    
    var min_cell: Vector2i = world_to_grid(world_pos - Vector2(radius, radius))
    var max_cell: Vector2i = world_to_grid(world_pos + Vector2(radius, radius))
    
    for cx in range(min_cell.x, max_cell.x + 1):
        for cy in range(min_cell.y, max_cell.y + 1):
            var cell_coord: Vector2i = Vector2i(cx, cy)
            if not _grid.has(cell_coord):
                continue
                
            var bucket: Array[Node2D] = _grid[cell_coord]
            for entity in bucket:
                if world_pos.distance_squared_to(entity.global_position) <= radius_sq:
                    results.append(entity)
                    
    return results
```

---

## 7.3 Memory Lifecycles, Orphan Leak Detection & Weak References

Memory leaks in garbage-collected environments differ from manual languages like C. In GDScript, leaks rarely manifest as crashes; instead, they appear as **retained references that prevent objects from being freed**, steadily increasing RAM usage until the operating system terminates the process.

```
                      GODOT MEMORY DUAL-TRACK ARCHITECTURE
=================================================================================
 TRACK 1: RefCounted Subsystem                TRACK 2: Node / Object Subsystem
 (Resources, RefCounted instances)            (Spatial nodes, UI Control nodes)
 ─────────────────────────────────            ─────────────────────────────────
  Memory managed via reference counts:        Manual lifetime management:
  - ref_count increments on assignment        - Instantiated via .new()
  - ref_count decrements when out of scope    - add_child() binds to Tree
  - Free occurs when ref_count == 0           - queue_free() safely deallocates
  ⚠️ LEAK RISK: Cyclic dependencies           ⚠️ LEAK RISK: Orphan nodes lingering
```

### 1. The Orphan Node Detector

An orphan node occurs when code calls `parent.remove_child(child)` or instantiates a node without mounting it to the tree via `add_child()`, and subsequently abandons the variable without calling `queue_free()`. The node remains in RAM indefinitely:

```gdscript
# ==============================================================================
# ORPHAN NODE AUDITOR & LEAK TELEMETRY
# ==============================================================================
class_name MemoryDiagnostics
extends RefCounted

## Prints a warning if orphan nodes are detected in the active engine instance
static func audit_orphan_nodes() -> void:
    # Query performance monitor for total unparented node count
    var orphan_count: int = Performance.get_monitor(Performance.OBJECT_ORPHAN_NODE_COUNT) as int
    
    if orphan_count > 0:
        push_warning("MemoryDiagnostics: [ORPHAN LEAK] %d unparented nodes lingering in RAM!" % orphan_count)
        # Optional: Print total Object allocations currently residing in host memory
        var total_objects: int = Performance.get_monitor(Performance.OBJECT_COUNT) as int
        var total_resources: int = Performance.get_monitor(Performance.OBJECT_RESOURCE_COUNT) as int
        print("Telemetry Summary -> Total Objects: %d | Total Resources: %d" % [total_objects, total_resources])
    else:
        print("MemoryDiagnostics: Memory audit clean. Zero orphan nodes detected.")
```

### 2. Breaking Cyclic Dependencies via WeakRef

If two `RefCounted` or `Resource` instances point to each other directly, their reference counts can never hit zero:

```
               CYCLIC DEPENDENCY: PERMANENT REFERENCE LOCK
=================================================================================
 [ CharacterStats (RefCounted) ] ──(var equipment)──► [ InventoryItem (RefCounted) ]
                                 ◄──(var owner_stats)─┘
 Both reference counts remain >= 1 forever. Memory leaks permanently upon scope exit!
```

To break the cycle, replace one strong reference with a **`WeakRef`**, which points to an object without incrementing its internal reference counter:

```gdscript
# ==============================================================================
# WEAK REFERENCE CIRCULAR RESOLUTION
# ==============================================================================
class_name InventoryItem
extends RefCounted

var item_name: String = "Mythic Blade"
## WEAK REFERENCE: Does NOT increment reference counter of CharacterStats
var _owner_stats_weak: WeakRef

func bind_owner(stats: RefCounted) -> void:
    _owner_stats_weak = weakref(stats)

func get_owner_stats() -> RefCounted:
    if _owner_stats_weak == null:
        return null
        
    var resolved: Object = _owner_stats_weak.get_ref()
    if resolved != null and is_instance_valid(resolved):
        return resolved as RefCounted
        
    return null # Target object was deallocated cleanly
```

---

## 7.4 Floating-Point Precision Drift (IEEE 754) & Physics Deferral

Writing game logic that assumes exact decimal equality inevitably produces elusive, hard-to-reproduce bugs:

```gdscript
# THE EQUALITY FAILURE FALLACY
var sum: float = 0.1 + 0.2
if sum == 0.3:
    print("Exact match!") # NEVER EXECUTES!
```

### The Mechanics of IEEE 754 Representation

CPUs represent standard 32-bit floats using binary scientific notation:

$$\text{Value} = (-1)^{\text{sign}} \times 2^{\text{exponent} - 127} \times \left(1 + \sum_{i=1}^{23} b_{23-i} 2^{-i}\right)$$

Because powers of $2$ cannot cleanly express fractions like $\frac{1}{10}$ or $\frac{2}{10}$, values like $0.1$ and $0.2$ are approximate recurring binary fractions. The sum evaluates to `0.3000000000000000444...`, meaning direct `==` comparisons fail:

```gdscript
# PRODUCTION DISCIPLINE: Approximate Equality Assertions
# Scalar equality:
if is_equal_approx(sum, 0.3):
    print("Resolved safely via epsilon threshold!")

# Zero comparison:
if is_zero_approx(velocity.length()):
    stop_locomotion()

# Vector equality:
if current_pos.is_equal_approx(target_pos):
    snap_to_waypoint()
```

### Physics Query Flushes & `set_deferred`

```
                      THE MID-TICK DEFERRAL CONFLICT
=================================================================================
 [ Physics Server Processing Iteration ] ──► Detects collision between Body & Area
          │
          ▼
 Emits signal: "_on_area_entered"
          │
          ▼
 User script calls: "collision_shape.disabled = true" 🚨 ENGINE ERROR!
 (Physics Server is currently mid-step; mutating collision states here causes memory corruption!)
```

To resolve this, defer physics-state changes to the end of the physics frame using **`set_deferred()`**:

```gdscript
func _on_hitbox_entered(body: Node2D) -> void:
    # ❌ WRONG: Triggers "Can't change this state while flushing queries" error
    # $CollisionShape2D.disabled = true
    
    # ✅ CORRECT: Defers assignment to the end of the current physics step
    $CollisionShape2D.set_deferred("disabled", true)
    
    # Deferred method invocation if complex scene tree modification is required:
    call_deferred(&"_safe_detach_sensor")

func _safe_detach_sensor() -> void:
    if is_instance_valid($Sensor):
        $Sensor.queue_free()
```

---

## 7.5 Concurrency: Safe Multithreading via `WorkerThreadPool`

Modern desktop and mobile CPUs use parallel multi-core architectures ($8\text{--}16$ hardware threads). Running heavy calculations (such as procedural map generation, path recalculations, or file decompression) entirely on the main thread causes hitching and stutter.

However, multithreading introduces severe race conditions if background threads interact with the **SceneTree**:

```
                       MULTITHREADING INTEGRITY RULES
=================================================================================
 [ WORKER THREAD ]                                  [ MAIN ENGINE THREAD ]
  - Pure mathematical computation                    - SceneTree structure modification
  - Noise generation / Pathfinding graphs             - Rendering and direct UI updates
  - File reading / Parsing binary streams            - Node creation & add_child() calls
                │                                                    ▲
                └─────── Communicates via Thread-Safe Queue ─────────┘
```

> [!warning] ⚠️ Core Architectural Invariant: The Main Thread Barrier
> Background worker threads must **never** call `add_child()`, `remove_child()`, modify UI elements, or interact directly with nodes mounted to the active SceneTree. Threads process raw data in isolation; they hand results back to the main thread for integration via thread-safe queues or deferred calls.

### Concrete Production Implementation: Background Noise Chunk Generator

```gdscript
# ==============================================================================
# ASYNCHRONOUS THREAD-POOL TASK DISPATCHER
# ==============================================================================
class_name AsyncChunkProcessor
extends Node

signal chunk_generation_completed(chunk_id: Vector2i, height_data: PackedFloat32Array)

@export var chunk_dimension: int = 64
@export var noise_profile: FastNoiseLite

## Thread-safe task payload container
class ChunkTask:
    var target_coord: Vector2i
    var dimension: int
    var noise: FastNoiseLite
    var output_buffer: PackedFloat32Array
    
    func _init(coord: Vector2i, dim: int, n: FastNoiseLite) -> void:
        target_coord = coord
        dimension = dim
        noise = n
        output_buffer = PackedFloat32Array()
        output_buffer.resize(dim * dim)

## Requests generation of terrain elevation data on a background worker thread
func request_chunk_async(chunk_coord: Vector2i) -> void:
    var task: ChunkTask = ChunkTask.new(chunk_coord, chunk_dimension, noise_profile)
    
    # Dispatch task to Godot's C++ WorkerThreadPool
    var task_id: int = WorkerThreadPool.add_task(
        _execute_background_task.bind(task),
        true, # High priority
        "GenerateChunk_" + str(chunk_coord)
    )

## EXECUTED EXCLUSIVELY ON WORKER THREAD (No SceneTree access permitted!)
func _execute_background_task(task: ChunkTask) -> void:
    var dim: int = task.dimension
    var offset_x: float = float(task.target_coord.x * dim)
    var offset_y: float = float(task.target_coord.y * dim)
    var buffer: PackedFloat32Array = task.output_buffer
    
    var index: int = 0
    for y in range(dim):
        var world_y: float = offset_y + float(y)
        for x in range(dim):
            var world_x: float = offset_x + float(x)
            # Intensive procedural Simplex noise sampling
            var elevation: float = task.noise.get_noise_2d(world_x, world_y)
            buffer[index] = elevation
            index += 1
            
    # Safely dispatch completion signal back to the main thread
    call_deferred(&"_notify_task_complete", task.target_coord, task.output_buffer)

## EXECUTED BACK ON MAIN THREAD
func _notify_task_complete(coord: Vector2i, heightmap: PackedFloat32Array) -> void:
    chunk_generation_completed.emit(coord, heightmap)
```

---

## 7.6 Diagnostics and Profiling Metrics Matrix

| Diagnostic Metric | Engine Monitor Identifier | Normal Threshold | Red-Flag Threshold | Remedial Action |
| :--- | :--- | :--- | :--- | :--- |
| **Physics Frame Time** | `TIME_PHYSICS_PROCESS` | $< 3.5\text{ ms}$ | $> 8.0\text{ ms}$ | Reduce `physics_ticks_per_second`, simplify collision shapes, check for broadphase overhead |
| **Orphan Node Count** | `OBJECT_ORPHAN_NODE_COUNT` | Strict $0$ | $\ge 1$ | Trace unparented `Node.new()` calls or missing `queue_free()` calls on detached nodes |
| **Active 3D Objects** | `PHYSICS_3D_ACTIVE_OBJECTS` | Contextual | Sharp spikes | Put stationary rigid bodies to sleep (`sleeping = true`) or convert to kinematic bodies |
| **Draw Calls** | `RENDER_TOTAL_DRAW_CALLS_IN_FRAME` | $< 500$ (Mobile)<br>$< 2,500$ (PC) | $> 4,000$ | Combine meshes, merge materials, or convert instances to `MultiMeshInstance3D` |
| **Video VRAM Usage** | `RENDER_VIDEO_MEM_USED` | Within GPU limits | Sustained growth | Compress textures (VRAM Compressed / Basis Universal), downscale uncompressed lightmaps |

---

## 7.7 Concrete Applied System: Frame-Pacing Telemetry & Performance Monitor

To synthesize **profiling diagnostics, floating-point analysis, orphan tracking, and telemetry display**, consider this production-grade, in-game performance monitor:

```gdscript
# ==============================================================================
# FRAME-PACING TELEMETRY HUD & DIAGNOSTIC PROBE
# ==============================================================================
class_name PerformanceTelemetryProbe
extends CanvasLayer

@export var sampling_window: float = 0.5
@export var warning_frame_time_ms: float = 16.67

@onready var readout_label: Label = $MarginContainer/ReadoutLabel

var _elapsed_window: float = 0.0
var _frame_count: int = 0
var _worst_frame_time_ms: float = 0.0

func _ready() -> void:
    # Ensure diagnostics render above all gameplay and UI layers
    layer = 128
    process_mode = Node.PROCESS_MODE_ALWAYS

func _process(delta: float) -> void:
    _frame_count += 1
    _elapsed_window += delta
    
    # Convert delta to milliseconds
    var current_frame_ms: float = delta * 1000.0
    if current_frame_ms > _worst_frame_time_ms:
        _worst_frame_time_ms = current_frame_ms
        
    # Aggregate data over sampling window
    if _elapsed_window >= sampling_window:
        _refresh_readout()
        _elapsed_window = 0.0
        _frame_count = 0
        _worst_frame_time_ms = 0.0

func _refresh_readout() -> void:
    var fps: float = Engine.get_frames_per_second()
    var process_ms: float = Performance.get_monitor(Performance.TIME_PROCESS) * 1000.0
    var physics_ms: float = Performance.get_monitor(Performance.TIME_PHYSICS_PROCESS) * 1000.0
    var memory_ram_mb: float = Performance.get_monitor(Performance.MEMORY_STATIC) / (1024.0 * 1024.0)
    var orphan_nodes: int = Performance.get_monitor(Performance.OBJECT_ORPHAN_NODE_COUNT) as int
    var draw_calls: int = Performance.get_monitor(Performance.RENDER_TOTAL_DRAW_CALLS_IN_FRAME) as int

    var format_template: String = (
        "PERFORMANCE TELEMETRY\n" +
        "───────────────────────────────\n" +
        "FPS (Frames/sec) : %3d\n" +
        "Frame Time Avg   : %6.2f ms\n" +
        "Worst Spike      : %6.2f ms %s\n" +
        "Process Time     : %6.2f ms\n" +
        "Physics Time     : %6.2f ms\n" +
        "Draw Calls       : %5d\n" +
        "Static Memory    : %6.2f MB\n" +
        "Orphan Nodes     : %3d %s"
    )

    var spike_alert: String = "🚨" if _worst_frame_time_ms > warning_frame_time_ms else "✅"
    var orphan_alert: String = "⚠️ LEAK!" if orphan_nodes > 0 else "✅"

    readout_label.text = format_template % [
        fps,
        (1000.0 / maxf(1.0, fps)),
        _worst_frame_time_ms,
        spike_alert,
        process_ms,
        physics_ms,
        draw_calls,
        memory_ram_mb,
        orphan_nodes,
        orphan_alert
    ]
```

---

## ✍️ Linguistic & Grammatical Refinement

> [!info] 📖 Constructive Analysis of Your Follow-Up Directive
> Let us review the syntax of your command:
> 1. **Capitalisation & Sentence Boundaries**:
>    - ⤷ *Original*: `"proceed with module 7"`
>    - ↳ **Correction**: The sentence opens with an uncapitalised imperative and lacks terminal punctuation. In technical documentation, formal sections function as proper nouns:
>      *"Proceed with Module 7."*
> 2. **Tone & Consistency**:
>    - ⤷ *Guidance*: Structuring prompts with consistent grammatical precision (e.g., *"Please proceed with Module 7"*) matches the academic tone maintained throughout this curriculum.