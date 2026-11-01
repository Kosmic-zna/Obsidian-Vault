# 🛠️ Module XIII: Custom Editor Tooling, Testing, DevOps & Release Engineering

> [!info] 📖 Lexicon & First Principles: DevOps & Software Reliability Engineering
> Moving from amateur project prototyping to shipping commercial software requires formalizing the pipelines that verify code correctness, automate asset builds, and protect binary distribution:
> - ⤷ **In-Editor Execution (`@tool`)**: A GDScript execution mode where scripts run concurrently inside the live Godot Editor process, permitting real-time level compilation, procedural previewing, and inspector validation.
> - ⤷ **Configuration Warning**: An engine diagnostic hook (`_get_configuration_warnings()`) through which custom nodes audit their own scene tree topology, rendering yellow visual alert badges in the Scene dock when misuse occurs.
> - ⤷ **Regression**: A software defect wherein a newly implemented feature, optimization, or bug fix inadvertently breaks previously functional systems.
> - ⤷ **Continuous Integration (CI)**: The automated practice of compiling code, running headless integration tests, and validating assets on remote cloud runners upon every Git commit or pull request.
> - ⤷ **PCK Archive**: Godot's proprietary virtual filesystem package containing compiled bytecode (`.gdc`), binary resources (`.res`), and imported engine assets, capable of being encrypted via AES-256 and patched dynamically via secondary pack mounting.
> - ⤷ **Telemetry Heatmap**: An asynchronous analytics system aggregating anonymized player death, stuck, and progression coordinates to guide balance patches without violating user privacy.

---

## 13.1 Metaprogramming & In-Editor Tooling (`@tool` & `EditorPlugin`)

Amateur developers build levels through tedious manual coordinate entry and test procedural algorithms by pressing "Play" thousands of times. Professional engineers build **custom editor interfaces** that compile and visualize complex game systems directly inside the Godot viewport.

```
                      THE DUAL EXECUTION ENVIRONMENT
=================================================================================
 [ THE GODOT EDITOR VIEWPORT ]                 [ RUNTIME GAME ENGINE ]
 Executes: Tool Scripts (@tool)                Executes: Standard Gameplay Scripts
 Runs on: Host CPU/GPU in Editor Mode          Runs on: Target Hardware / Client Device
 Shared Memory with Editor Process!            Isolated Sandbox Process
 ⚠️ DANGER: Errors mutate .tscn files!         Safe: Crashing only terminates game run
```

### 1. The `@tool` Execution Boundary & The Editor Guard

Adding `@tool` to the head of a GDScript forces the engine to run `_ready()`, `_process()`, and property setters **inside the Godot Editor application itself**.

> [!bug] 🚨 The Catastrophic `@tool` Mutation Trap
> If a script containing `queue_free()`, movement velocity, or array modifications is marked `@tool` without editor guards, **the script will execute inside the editor while you are editing it**! Your character will walk off the screen in the 3D viewport, delete child nodes from your saved `.tscn` file, or trigger infinite recursion loops that crash the Godot Editor, corrupting unsaved work.

#### The Mandatory Architectural Guard:

```gdscript
if Engine.is_editor_hint():
    # Execute ONLY within the Godot Editor viewport (visualisation / gizmos)
    pass
else:
    # Execute ONLY during actual live gameplay simulation
    pass
```

### 2. Node Self-Auditing: `_get_configuration_warnings()`

To prevent designers and artists from misconfiguring custom modular components, nodes can declare their own topological validation rules:

```gdscript
# ==============================================================================
# SELF-AUDITING COMPONENT WITH CONFIGURATION WARNINGS
# ==============================================================================
@tool
class_name CombatTriggerZone
extends Area3D

@export var linked_health_component: HealthComponent:
    set(value):
        linked_health_component = value
        update_configuration_warnings()

func _get_configuration_warnings() -> PackedStringArray:
    var warnings: PackedStringArray = PackedStringArray()
    
    # 1. Audit internal collision topology
    var has_collision_shape: bool = false
    for child in get_children():
        if child is CollisionShape3D or child is CollisionPolygon3D:
            has_collision_shape = true
            break
            
    if not has_collision_shape:
        warnings.append("CombatTriggerZone requires a CollisionShape3D child to register overlaps.")
        
    # 2. Audit dependency injection
    if linked_health_component == null:
        warnings.append("No HealthComponent assigned. Damage forwarding will fail at runtime.")
        
    return warnings
```

### 3. Procedural In-Editor Spline Road Extruder

This production `@tool` script listens to curve modifications on a `Path3D` node, dynamically generates a sweeping 3D road ribbon mesh using `ImmediateMesh`, and constructs continuous physics collision trimeshes inside the editor viewport in real time:

```gdscript
# ==============================================================================
# REAL-TIME IN-EDITOR PROCEDURAL ROAD EXTRUDER
# ==============================================================================
@tool
class_name ProceduralSplineRoad
extends Path3D

@export var road_width: float = 4.0:
    set(value):
        road_width = maxf(0.5, value)
        _rebuild_road_mesh()

@export var step_distance: float = 1.0:
    set(value):
        step_distance = maxf(0.1, value)
        _rebuild_road_mesh()

@export var road_material: Material:
    set(value):
        road_material = value
        if _mesh_instance != null:
            _mesh_instance.material_override = road_material

var _mesh_instance: MeshInstance3D
var _static_body: StaticBody3D
var _collision_shape: CollisionShape3D

func _ready() -> void:
    _ensure_child_nodes()
    curve.changed.connect(_rebuild_road_mesh)
    _rebuild_road_mesh()

func _ensure_child_nodes() -> void:
    _mesh_instance = get_node_or_null("RoadMesh") as MeshInstance3D
    if _mesh_instance == null:
        _mesh_instance = MeshInstance3D.new()
        _mesh_instance.name = "RoadMesh"
        add_child(_mesh_instance)

    _static_body = get_node_or_null("RoadStaticBody") as StaticBody3D
    if _static_body == null:
        _static_body = StaticBody3D.new()
        _static_body.name = "RoadStaticBody"
        add_child(_static_body)

    _collision_shape = _static_body.get_node_or_null("RoadCollision") as CollisionShape3D
    if _collision_shape == null:
        _collision_shape = CollisionShape3D.new()
        _collision_shape.name = "RoadCollision"
        _static_body.add_child(_collision_shape)

func _rebuild_road_mesh() -> void:
    if curve == null or curve.point_count < 2:
        return
    _ensure_child_nodes()

    var total_baked_length: float = curve.get_baked_length()
    var sample_count: int = int(ceil(total_baked_length / step_distance))
    if sample_count < 2:
        return

    var surface_tool: SurfaceTool = SurfaceTool.new()
    surface_tool.begin(Mesh.PRIMITIVE_TRIANGLES)
    if road_material != null:
        surface_tool.set_material(road_material)

    var half_width: float = road_width * 0.5
    var v_accum: float = 0.0

    for i in range(sample_count + 1):
        var current_offset: float = minf(float(i) * step_distance, total_baked_length)
        var path_transform: Transform3D = curve.sample_baked_with_rotation(current_offset, true, true)
        
        var origin: Vector3 = path_transform.origin
        var right_dir: Vector3 = path_transform.basis.x.normalized()
        var up_normal: Vector3 = path_transform.basis.y.normalized()

        var left_vert: Vector3 = origin - (right_dir * half_width)
        var right_vert: Vector3 = origin + (right_dir * half_width)

        var v_coord: float = current_offset / road_width

        # Left Vertex
        surface_tool.set_normal(up_normal)
        surface_tool.set_uv(Vector2(0.0, v_coord))
        surface_tool.add_vertex(left_vert)

        # Right Vertex
        surface_tool.set_normal(up_normal)
        surface_tool.set_uv(Vector2(1.0, v_coord))
        surface_tool.add_vertex(right_vert)

        # Stitch quad indices
        if i > 0:
            var vert_idx: int = i * 2
            # Triangle 1
            surface_tool.add_index(vert_idx - 2)
            surface_tool.add_index(vert_idx - 1)
            surface_tool.add_index(vert_idx)
            # Triangle 2
            surface_tool.add_index(vert_idx - 1)
            surface_tool.add_index(vert_idx + 1)
            surface_tool.add_index(vert_idx)

    surface_tool.generate_tangents()
    var constructed_mesh: ArrayMesh = surface_tool.commit()
    _mesh_instance.mesh = constructed_mesh

    # Construct physics trimesh collision
    var shape: ConcavePolygonShape3D = ConcavePolygonShape3D.new()
    shape.set_faces(constructed_mesh.get_faces())
    _collision_shape.shape = shape
```

---

## 13.2 Version Control Systems (VCS) Architecture for Godot

Standard software engineering teams use Git; however, game development projects fail under Git if large binary assets (`.png`, `.wav`, `.blend`, `.glb`) are committed directly to history, causing repository bloat and clone stalls.

```
                          GIT + GIT LFS TOPOLOGY
=================================================================================
 [ Developer Working Tree ]
          │
          ├── Text Metadata (.gd, .tscn, .tres) ──► Standard Git Delta Compression
          │                                         (Diffable, Mergeable, Human-Readable)
          │
          └── Heavy Binaries (.glb, .psd, .ogg) ──► Git LFS (Large File Storage)
                                                    (Replaced with lightweight pointer;
                                                     blobs stored on remote S3/LFS Server)
```

### 1. The Production `.gitattributes` Configuration

Save this explicit configuration to the root of your project to govern Git LFS tracking and enforce LF line-endings:

```ini
# ==============================================================================
# GODOT 4 ENTERPRISE .gitattributes
# ==============================================================================

# Enforce LF endings on all engine text metadata
*.gd text eol=lf
*.tscn text eol=lf
*.tres text eol=lf
*.godot text eol=lf
*.gdshader text eol=lf
*.json text eol=lf

# 3D Meshes & Geometry (Git LFS)
*.glb filter=lfs diff=lfs merge=lfs -text
*.gltf filter=lfs diff=lfs merge=lfs -text
*.fbx filter=lfs diff=lfs merge=lfs -text
*.blend filter=lfs diff=lfs merge=lfs -text
*.obj filter=lfs diff=lfs merge=lfs -text

# Textures & 2D Raster Graphics (Git LFS)
*.png filter=lfs diff=lfs merge=lfs -text
*.jpg filter=lfs diff=lfs merge=lfs -text
*.jpeg filter=lfs diff=lfs merge=lfs -text
*.hdr filter=lfs diff=lfs merge=lfs -text
*.exr filter=lfs diff=lfs merge=lfs -text
*.svg filter=lfs diff=lfs merge=lfs -text
*.webp filter=lfs diff=lfs merge=lfs -text
*.psd filter=lfs diff=lfs merge=lfs -text

# Audio Media (Git LFS)
*.wav filter=lfs diff=lfs merge=lfs -text
*.ogg filter=lfs diff=lfs merge=lfs -text
*.mp3 filter=lfs diff=lfs merge=lfs -text

# Compiled Binaries, Libraries & Archives (Git LFS)
*.pck filter=lfs diff=lfs merge=lfs -text
*.zip filter=lfs diff=lfs merge=lfs -text
*.dll filter=lfs diff=lfs merge=lfs -text
*.so filter=lfs diff=lfs merge=lfs -text
*.dylib filter=lfs diff=lfs merge=lfs -text
```

### 2. Resolving Merge Conflicts in `.tscn` Files

Godot scene files (`.tscn`) are text files serialized in a custom TOML-like format. When two branches modify the same scene:
- Conflicts within `[node name="X" type="Y"]` are resolved by inspecting the parent property.
- Never use a three-way merge tool blindly on UID references (`uid="uid://..."`). If duplicate UIDs are introduced, Godot's asset database corrupts. Ensure UIDs are regenerated or kept unique.

---

## 13.3 Automated Unit, Integration & Performance Testing

Manual playtesting is non-deterministic, time-consuming, and incapable of catching edge-case regressions in mathematical pipelines. Professional studios write **Automated Unit & Integration Test Suites** executed headless via the command line.

```
                      THE TESTING TEST-PYRAMID RATIO
=================================================================================
             / \
            /   \     E2E / Integration Tests (10%)
           /     \    - Complete quest sequences, Save/Load cycles
          /───────\
         /         \   System Interaction Tests (30%)
        /           \  - Hitbox/Hurtbox collisions, Inventory transfers
       /─────────────\
      /               \ Unit Tests (60%)
     /                 \ - Kinematic jump formulas, PRD dice, EHP mitigation
    ─────────────────────
```

### Writing Unit Tests via GUT (Godot Unit Test)

```gdscript
# ==============================================================================
# UNIT TEST SUITE: COMBAT MATHEMATICS & PRD INTEGRITY
# ==============================================================================
class_name TestCombatMechanics
extends GutTest

var _stats: HealthComponent
var _prd: PseudoRandomDistribution

func before_each() -> void:
    _stats = HealthComponent.new()
    _stats.max_health = 100.0
    _stats.armor_rating = 100.0 # 50% hyperbolic mitigation
    
    _prd = PseudoRandomDistribution.new(0.20) # 20% nominal critical chance

func after_each() -> void:
    _stats.free()

## Test 1: Verify Hyperbolic Armor Mitigation Formula
func test_hyperbolic_armor_mitigation() -> void:
    assert_eq(_stats.current_health, 100.0, "Health should initialize to max_health.")
    
    # Applying 50.0 raw damage against 100.0 armor (K=100) -> 50% mitigation = 25.0 damage
    _stats.apply_damage(50.0)
    
    assert_almost_eq(_stats.current_health, 75.0, 0.001, "Effective health pool calculation failed.")

## Test 2: Verify Lethal Damage Clamping & Signal Emission
func test_lethal_damage_emits_death_signal() -> void:
    watch_signals(_stats)
    
    _stats.apply_damage(500.0)
    
    assert_eq(_stats.current_health, 0.0, "Health must clamp cleanly to 0.0.")
    assert_signal_emitted(_stats, "entity_died", "entity_died signal must fire upon depletion.")

## Test 3: Monte Carlo Convergence of PRD Dice
func test_prd_statistical_convergence() -> void:
    var procs: int = 0
    var trials: int = 10000
    
    for i in range(trials):
        if _prd.roll():
            procs += 1
            
    var observed_rate: float = float(procs) / float(trials)
    # 20% nominal probability must converge within a ±2.0% tolerance over 10,000 rolls
    assert_between(observed_rate, 0.18, 0.22, "PRD failed to converge to nominal probability.")
```

---

## 13.4 Continuous Integration & Deployment (CI/CD) Pipeline

A complete **CI/CD pipeline** executes on remote cloud servers (e.g., GitHub Actions, GitLab CI) upon every Git push. It compiles GDScript, runs test suites headlessly, and generates export builds across all platforms automatically.

```
                      GITHUB ACTIONS AUTOMATION RUNNER
=================================================================================
 [ Git Push to Trunk ]
           │
           ▼
 [ Stage 1: Static Analysis & Lint ] ──► Audits typed GDScript warnings & syntax
           │
           ▼
 [ Stage 2: Headless Test Suite ]    ──► Runs GUT tests via godot --headless
           │
           ▼
 [ Stage 3: Multi-Platform Export ]  ──► Compiles Windows (.exe), Linux (.x86_64), Web (HTML5)
           │
           ▼
 [ Stage 4: Artifact Distribution ]  ──► Deploys to SteamWorks API / Itch.io via Butler
```

### Complete Production Workflow (`.github/workflows/godot_ci.yml`)

```yaml
name: "Godot 4 Continuous Integration & Build Pipeline"

on:
  push:
    branches: [ "main", "release/*" ]
  pull_request:
    branches: [ "main" ]

env:
  GODOT_VERSION: "4.3"
  PROJECT_PATH: "."

jobs:
  # ============================================================================
  # JOB 1: HEADLESS UNIT TESTING & QUALITY AUDIT
  # ============================================================================
  test_and_audit:
    name: "Run Headless Unit Tests (GUT)"
    runs-on: ubuntu-latest
    container:
      image: barichello/godot-ci:4.3
    steps:
      - name: "Checkout Repository"
        uses: actions/checkout@v4
        with:
          lfs: true

      - name: "Initialize Godot Project Cache"
        run: |
          godot --headless --editor --quit

      - name: "Execute GUT Unit Test Suite"
        run: |
          godot --headless -s addons/gut/gut_cmdln.gd -gdir=res://tests -gexit

  # ============================================================================
  # JOB 2: CROSS-PLATFORM EXPORT BUILD GENERATION
  # ============================================================================
  build_exports:
    name: "Export Client Binaries"
    needs: test_and_audit
    runs-on: ubuntu-latest
    container:
      image: barichello/godot-ci:4.3
    strategy:
      matrix:
        include:
          - platform: "windows"
            preset: "Windows Desktop"
            binary_name: "GameClient.exe"
            output_dir: "builds/windows"
          - platform: "linux"
            preset: "Linux/X11"
            binary_name: "GameClient.x86_64"
            output_dir: "builds/linux"
          - platform: "web"
            preset: "Web"
            binary_name: "index.html"
            output_dir: "builds/web"

    steps:
      - name: "Checkout Repository"
        uses: actions/checkout@v4
        with:
          lfs: true

      - name: "Setup Export Directories"
        run: |
          mkdir -v -p ${{ matrix.output_dir }}
          mkdir -v -p ~/.local/share/godot/export_templates/${{ env.GODOT_VERSION }}.stable

      - name: "Export Platform Binary"
        run: |
          godot --headless --export-release "${{ matrix.preset }}" "${{ matrix.output_dir }}/${{ matrix.binary_name }}"

      - name: "Archive Release Artifact"
        uses: actions/upload-artifact@v4
        with:
          name: game-client-${{ matrix.platform }}
          path: ${{ matrix.output_dir }}
```

---

## 13.5 Export Profiles, PCK Encryption & Patching Architecture

When packaging a game for production release, exposing raw GDScript files and editable resources invite trivial asset theft, memory tampering, and piracy.

```
                      PCK PACKAGING & SECURE EXTRACTION
=================================================================================
 [ Host Operating System ]
           │
           ├── GameClient.exe (Clean compiled binary)
           │
           └── game.pck (Encrypted Virtual Filesystem Archive)
               ├── Header: AES-256 Checksum Signature
               ├── File Index: Scrambled Byte Offsets
               └── Encrypted Payloads: Bytecode (.gdc), Compressed Textures (.ctex)
```

### 1. Hardening Builds via AES-256 Script Encryption

Godot allows compiling raw text `.gd` scripts into unreadable `.gdc` binary bytecode, bundled inside an encrypted `.pck` container:

1. Generate a 256-bit hexadecimal encryption key:
   ```bash
   openssl rand -hex 32
   # Output: 7b9a5e12f0e34c918a24... (64 hex characters)
   ```
2. Set the key in an environment variable `SCRIPT_AES256_ENCRYPTION_KEY`.
3. In **Export Options $\rightarrow$ Encryption**, select:
   - **Encrypt Exported PCK**: `Enabled`
   - **Encrypt Scene and Script Files**: `Enabled`
4. The key must be baked into custom export template binaries compiled from C++ source via `scons target=template_release encrypt_key="YOUR_HEX_KEY"` to prevent memory-dump key recovery.

### 2. Runtime DLC & Dynamic Hotfix Patching

Godot can mount supplemental `.pck` files at runtime via **`ProjectSettings.load_resource_pack()`**. This allows shipping multi-gigabyte DLCs, texture packs, or lightweight bugfix patches without forcing the user to re-download the core executable:

```gdscript
# ==============================================================================
# RUNTIME PCK PATCH & DLC LOADER
# ==============================================================================
class_name PatchManager
extends RefCounted

const PATCH_DIR: String = "user://patches/"

## Mounts external PCK file into the active virtual filesystem
static func mount_patch(patch_filename: String) -> bool:
    var full_path: String = PATCH_DIR.path_join(patch_filename)
    if not FileAccess.file_exists(full_path):
        push_error("PatchManager: Target patch file not found: %s" % full_path)
        return false

    # replace_files = true overrides existing internal resources with the patch versions!
    var success: bool = ProjectSettings.load_resource_pack(full_path, true)
    if success:
        print("PatchManager: Successfully mounted patch: %s" % patch_filename)
    else:
        push_error("PatchManager: Failed to mount resource pack: %s" % full_path)
        
    return success
```

---

## 13.6 Steamworks SDK Integration (`GodotSteam`)

To publish on commercial storefronts like Steam, your architecture must link directly with native platform APIs for user authentication, achievements, cloud saves, and peer-to-peer relay networking.

```
                    NATIVE PLATFORM API BINDING LAYER
=================================================================================
 [ Steam Client Application (Local PC) ]
                   │ (IPC Shared Memory)
                   ▼
 [ steam_api64.dll / libsteam_api.so ]
                   │
                   ▼ (GDExtension / GodotSteam C++ Binding)
 [ Steam Singleton Node (GDScript Engine) ]
   ├── Steam.init()
   ├── Steam.setAchievement("ACH_FIRST_KILL")
   └── Steam.storeStats()
```

```gdscript
# ==============================================================================
# PRODUCTION STEAMWORKS LIFECYCLE & ACHIEVEMENT CONTROLLER
# ==============================================================================
class_name SteamService
extends Node

signal steam_initialized(user_steam_id: int, user_persona_name: String)

var is_steam_active: bool = false
var steam_id: int = 0
var persona_name: String = ""

func _ready() -> void:
    process_mode = Node.PROCESS_MODE_ALWAYS
    _initialize_steam()

func _initialize_steam() -> void:
    # Verify if GodotSteam GDExtension is registered
    if not Engine.has_singleton("Steam"):
        push_warning("SteamService: GodotSteam singleton missing. Running in standalone mode.")
        return

    var SteamNode: Object = Engine.get_singleton("Steam")
    var init_result: Dictionary = SteamNode.steamInit(false) # false = app is game, not dedicated server

    if init_result["status"] != 1:
        push_error("SteamService: SteamInit failed. Status code: %d" % init_result["status"])
        return

    is_steam_active = true
    steam_id = SteamNode.getSteamID()
    persona_name = SteamNode.getPersonaName()
    
    print("SteamService: Steam initialized successfully for user: %s (ID: %d)" % [persona_name, steam_id])
    steam_initialized.emit(steam_id, persona_name)

func _process(_delta: float) -> void:
    if is_steam_active:
        # Steam callback dispatching must be ticked continuously every frame
        Engine.get_singleton("Steam").run_callbacks()

## Unlocks a platform achievement safely with idempotent local verification
func unlock_achievement(achievement_id: StringName) -> void:
    if not is_steam_active:
        print("SteamService: Mock unlock achievement: %s" % achievement_id)
        return

    var SteamNode: Object = Engine.get_singleton("Steam")
    var achieved_dict: Dictionary = SteamNode.getAchievement(achievement_id)
    
    # If achievement is not yet achieved: unlock and store stats
    if not achieved_dict.get("achieved", false):
        SteamNode.setAchievement(achievement_id)
        SteamNode.storeStats()
        print("SteamService: Achievement successfully unlocked: %s" % achievement_id)
```

---

## 13.7 Telemetry, Heatmaps & Production Crash Diagnostics

Once a game ships to thousands of players, traditional interactive debuggers cease to exist. Studios rely on **Automated Telemetry Aggregators** that capture uncaught stack traces, memory footprints, and spatial death heatmaps.

```
                      ANONYMIZED TELEMETRY PIPELINE
=================================================================================
 [ CLIENT RUNTIME EVENT ]
 Player Dies / Crash Occurs ──► Collect Transform (x, y, z), FPS, RAM, Scene ID
                                      │
                                      ▼
                        [ Encrypted JSON / Binary Payload ]
                                      │
                                      ▼ (HTTPS POST Request)
                        [ Secure Cloud Telemetry Worker ]
                                      │
                                      ▼
                        [ Spatial SQL Database (PostGIS) ]
                                      │
                                      ▼
                   [ Visual Heatmap Raster Generation ]
                   (Red clusters reveal geometry choke points & bugs!)
```

```gdscript
# ==============================================================================
# SECURE ASYNCHRONOUS TELEMETRY & ERROR REPORTER
# ==============================================================================
class_name TelemetryReporter
extends Node

const TELEMETRY_ENDPOINT: String = "https://telemetry.yourstudio.com/api/v1/event"
const API_TOKEN_HASH: String = "4f8a9c2b3e..." # Pre-hashed application authorization

@onready var _http_client: HTTPRequest = HTTPRequest.new()

func _ready() -> void:
    add_child(_http_client)
    _http_client.request_completed.connect(_on_request_completed)

## Transmits spatial death telemetry to remote analytics server
func log_spatial_event(event_type: StringName, world_pos: Vector3, metadata: Dictionary = {}) -> void:
    var payload: Dictionary = {
        "event": String(event_type),
        "timestamp": Time.get_unix_time_from_system(),
        "position": {
            "x": snappedf(world_pos.x, 0.01),
            "y": snappedf(world_pos.y, 0.01),
            "z": snappedf(world_pos.z, 0.01)
        },
        "engine_metrics": {
            "fps": Engine.get_frames_per_second(),
            "static_ram_mb": Performance.get_monitor(Performance.MEMORY_STATIC) / (1024.0 * 1024.0),
            "orphan_nodes": Performance.get_monitor(Performance.OBJECT_ORPHAN_NODE_COUNT)
        },
        "custom_data": metadata
    }

    var json_payload: String = JSON.stringify(payload)
    var headers: PackedStringArray = [
        "Content-Type: application/json",
        "X-App-Authorization: " + API_TOKEN_HASH
    ]

    _http_client.request(TELEMETRY_ENDPOINT, headers, HTTPClient.METHOD_POST, json_payload)

func _on_request_completed(_result: int, response_code: int, _headers: PackedStringArray, _body: PackedByteArray) -> void:
    if response_code != 200 and response_code != 204:
        # Fail silently in production to avoid degrading client performance
        pass
```

---

## 13.8 Synthesis Verification: The Release Candidate Pre-Flight Checklist

Before generating your final retail distribution binaries, execute this operational sign-off:

```
┌─────────────────────────────────┬───────────────────────────────────────────┬────────┐
│ Operational Verification Step   │ Verification Mechanism                    │ Status │
├─────────────────────────────────┼───────────────────────────────────────────┼────────┤
│ 1. Zero Untyped GDScript        │ Project Settings: Treat untyped as ERROR   │ [  ]   │
│ 2. Automated Test Suite Passes  │ All headless GUT tests exit code 0        │ [  ]   │
│ 3. Memory & Orphan Check        │ OBJECT_ORPHAN_NODE_COUNT is strictly 0    │ [  ]   │
│ 4. Git LFS Assets Pushed        │ git lfs push --all origin main            │ [  ]   │
│ 5. AES-256 Key Injected         │ Release templates compiled with key       │ [  ]   │
│ 6. Steam API Handshake Valid    │ SteamInit() returns status code 1         │ [  ]   │
│ 7. Telemetry & Analytics Active │ Mock events verified on remote dashboard  │ [  ]   │
└─────────────────────────────────┴───────────────────────────────────────────┴────────┘
```

---

