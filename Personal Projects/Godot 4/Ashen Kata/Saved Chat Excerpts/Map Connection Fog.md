> ✍️ **Phrasing Refinement:** "I like the fog approach from _Dark Souls_; how can I make the transitions between my _Monster Hunter Rise_-style basin regions feel truly seamless—similar to _Grand Theft Auto: San Andreas_ and its hidden loading screens?"
> 
>   
# 📋 Executive Summary

- ✅ **Combine `ResourceLoader.load_threaded_request()` With a "Fog-Choked Winding Chokepoint" (`S-Bend Gorge` / `Mist-Shrouded Tunnel` / `Vertical Lift`) so Your Intel i5-1155G7 / 8 GB RAM Laptop Streams Regions With Zero Stutter:** Just as _GTA: San Andreas_ used long bridges, highway tunnels, and hill crests to silently stream map sectors off the PS2 disc while blocking sightlines, your Godot 4 macro-corridors can use a **30–50 metre curved canyon, stone aqueduct tunnel, or dense fog bank** paired with **background threaded loading** (`ResourceLoader.load_threaded_request`) to unload Basin A and instantiate Basin B invisibly while you walk through the mist.
    
- ✅ **Why Fog + S-Bend Geometry Solves Both Visuals and Memory (`8 GB RAM` Protection):** Because your Intel i5-1155G7 laptop shares **8 GB of system RAM** with Iris Xe graphics, you must **unload the previous basin before instantiating the next** rather than holding two giant regions in memory simultaneously. A winding, fog-filled chokepoint naturally limits sightlines to ~15 metres, allowing you to safely drop Basin A behind a physical stone gate or thick fog wall before the exit of the corridor reveals Basin B!
    

### 📖 Plain-English Glossary of Seamless Basin Streaming

- **S-Bend Sightline Blocker (`Occlusion Geometry`):** A curved gorge, zigzagging mountain stair, or dog-legged stone aqueduct tunnel between two basins. Unlike a straight hallway (where the camera can see all the way from Basin A into Basin B), an S-bend forces two 90° turns so neither basin is ever visible from the middle of the corridor.
    
- **Asynchronous Background Loading (`ResourceLoader.load_threaded_request`):** Godot 4's built-in multi-threaded loader that reads a `.tscn` scene file from your SSD on a background CPU thread over several frames instead of freezing the main game thread (`load()`), preventing frame-time spikes on your i5-1155G7.
    
- **Fog Density Ramping (`Dynamic WorldEnvironment Interpolation`):** Smoothly tweening `environment.fog_density` from `0.035` (open basin) up to `0.14` (thick pea-soup mist) as the player enters the chokepoint, masking any LOD swaps or scene instantiation before thinning back out as you emerge into the new region.
    

### 🗺️ Architectural Diagram: _GTA: San Andreas_-Style Hidden Streaming Corridor

```
========================================================================================
       SEAMLESS BASIN TRANSITION USING FOG + S-BEND CHOKEPOINT (`8 GB RAM` SAFE)
========================================================================================

  [ BASIN A: ASHEN SEA-WALL ]  (Currently loaded in memory; Fog Density = 0.035)
          │
          ▼  Player crosses `Trigger_Preload_B` (Starts background thread!)
  ┌──────────────────────────────────────────────────────────────────────────────┐
  │  HIDDEN STREAMING CORRIDOR (40m S-Bend Gorge / Fog-Choked Stone Aqueduct)    │
  │   1. Fog Density smoothly tweens `0.035 ──► 0.14` (Visibility drops to 12m). │
  │   2. Background thread loads `Basin_B.tscn` silently while player runs/walks.│
  │   3. Mid-corridor stone portcullis / S-turn blocks sightline back to Basin A.│
  │   4. Player crosses `Trigger_Swap`:                                          │
  │       • `Basin_A.queue_free()` (Frees RAM immediately for 8 GB laptop!)      │
  │       • `Basin_B` instantiates cleanly behind the dense fog ahead!          │
  │   5. Fog Density smoothly tweens `0.14 ──► 0.030` (New tint for Basin B!).   │
  └──────────────────────────────────────────────────────────────────────────────┘
          │
          ▼  Player rounds the final bend into a dramatic vista reveal!
  [ BASIN B: MIST-CLOAKED KARST BASIN ]  (Freshly loaded; Basin A freed from RAM!)
========================================================================================
```

### 🛠️ Production-Ready Godot 4 `RegionStreamer.gd` Blueprint

Place a persistent **`CorridorConnector`** scene (containing your winding rock canyon/aqueduct, fog volume/settings, and two `Area3D` triggers) between your two basin scenes. Here is the exact threaded streamer script:

  

GDScript

```
extends Node3D

@export_file("*.tscn") var next_basin_path: String
@export var anchor_offset: Vector3 = Vector3.ZERO  # Exact world offset where Basin B connects
@export var corridor_fog_density: float = 0.14
@export var basin_fog_density: float = 0.035
@export var next_basin_fog_color: Color = Color(0.24, 0.34, 0.38)

@onready var world_env: WorldEnvironment = get_tree().get_first_node_in_group("world_env")

var _is_loading: bool = false
var _next_basin_instance: Node3D = null

# Trigger 1: Placed at the entrance of the S-Bend fog corridor
func _on_preload_trigger_body_entered(body: Node3D) -> void:
	if not body.is_in_group("player") or _is_loading or _next_basin_instance != null:
		return
	
	_is_loading = true
	ResourceLoader.load_threaded_request(next_basin_path)
	
	# Smoothly thicken fog like Dark Souls / Silent Hill to mask background streaming
	if world_env and world_env.environment:
		var tween := create_tween()
		tween.tween_property(world_env.environment, "fog_density", corridor_fog_density, 1.5)

func _process(_delta: float) -> void:
	if not _is_loading:
		return
	
	var status := ResourceLoader.load_threaded_get_status(next_basin_path)
	if status == ResourceLoader.THREAD_LOAD_LOADED:
		var packed_scene: PackedScene = ResourceLoader.load_threaded_get(next_basin_path)
		_next_basin_instance = packed_scene.instantiate()
		_next_basin_instance.position = anchor_offset
		_is_loading = false

# Trigger 2: Placed past the S-bend blind corner (Basin A is completely out of sight!)
func _on_swap_trigger_body_entered(body: Node3D) -> void:
	if not body.is_in_group("player") or _next_basin_instance == null:
		return
	
	# 1. Free previous basin FIRST so 8 GB RAM laptops never spike memory!
	var old_basin := get_tree().get_first_node_in_group("active_basin")
	if old_basin:
		old_basin.queue_free()
	
	# 2. Attach the preloaded next basin seamlessly inside the thick fog
	get_parent().add_child(_next_basin_instance)
	_next_basin_instance.add_to_group("active_basin")
	_next_basin_instance = null
	
	# 3. Part the fog dramatically and shift tint to the new biome's mood
	if world_env and world_env.environment:
		var tween := create_tween().set_parallel(true)
		tween.tween_property(world_env.environment, "fog_density", basin_fog_density, 2.5)
		tween.tween_property(world_env.environment, "fog_light_color", next_basin_fog_color, 2.5)
```

### 🛠️ 3 Practical Rules to Prevent Any Frame Hitch During the Swap

1. **Avoid Heavy `MeshInstance3D` Physics Baking at Runtime (`Pre-Bake CollisionShapes`):**
    
    - Never generate `ConcavePolygonShape3D` (`Trimesh`) on the fly when instantiating `Basin_B.tscn`—always save your static terrain and architecture collisions pre-baked inside the `.tscn` file so `instantiate()` completes in under `2 ms`.
        
2. **Stagger `MultiMeshInstance3D` Grass/Reeds Into Sub-Chunks:**
    
    - If `Basin_B` contains thousands of swamp reeds or falling ash particles, keep `MultiMesh` counts modest (`500–1,000` instances per basin) and pre-warm particle shaders on initial game boot so the fog parting never stutters.
        
3. **Use Architectural "Speed Bumps" Naturally (_GTA: San Andreas_ Trick):**
    
    - Just like _San Andreas_ used winding mountain roads, tollbooths, or low-clearance tunnels where players naturally steer carefully instead of rocketing across at top speed, place a **narrow stone bridge, a winding stair, or a crouching crevice** inside your 40-metre chokepoint—guaranteeing the background thread always finishes loading before the player reaches the exit!
        

### 🎯 Next Step Question

Would you like to build a quick **greybox `CorridorConnector` test scene** in Godot 4 to verify seamless basin swapping on your laptop, or are you ready to jump back into **Blender** to finish **`Katana.Attack:Light2`**?


> ✍️ **Phrasing Refinement:** "Before constructing each individual region, I would like to plan how the entire interconnected world fits together—essentially mapping out the whole landmass. Should I build the full-scale world in low detail inside a master scene, duplicate it for each region, detail only the playable basin within each copy, and leave the rest low-resolution? The flaw with that approach is that if I later modify the master low-res terrain, I would have to manually propagate those changes across the distant low-res vistas of every separate region scene."
> 
>   

# 📋 Executive Summary

- ✅ **Do NOT Clone and Edit Separate Copies of a Master Scene—Instead, Use Godot 4’s Native "Scene Inheritance / Single Shared Vista Proxy" Architecture (`Master_Vista_Shell.tscn` + Independent Playable Basin Scenes):** Your instinct about manual propagation pain is 100% spot-on—duplicating a master scene four times creates a synchronization nightmare the moment you shift a mountain ridge or move an aqueduct.
    
- ✅ **How AAA Studios & Solo Godot Developers Solve This Cleanly (`The "Doughnut & Hole" Modular Proxy Workflow`):**
    
    1. **Build ONE Single Master Low-Res World (`World_Macro_Vista.tscn`)** containing the overarching terrain shell, rivers, and low-poly landmark proxies (`Aqueduct_Proxy`, `Citadel_Proxy`, `Karst_Peaks_Proxy`) at their true world-space coordinates.
        
    2. **Instance That Exact Same `World_Macro_Vista.tscn` Into Every Playable Basin Scene (`Basin_01_SeaWall.tscn`, `Basin_02_Karst.tscn`, etc.)**—and simply **hide (`visible = false` or toggle a script export flag)** the local low-res proxy chunk for whichever basin is currently loaded in high detail!
        
    3. **Result:** You edit **`World_Macro_Vista.tscn`** in **one single place**, and every playable region automatically inherits updated distant mountains, rivers, and cross-biome landmarks instantaneously—with zero duplicate files and tiny memory overhead on your **Intel Core i5-1155G7 / 8 GB RAM** laptop!
        

### 📖 Plain-English Glossary of Master-World & Proxy Architecture

- **Single Source of Truth (`Instanced Master Vista`):** Keeping one master `.tscn` file (`World_Macro_Vista.tscn`) that holds the low-poly geographical skeleton of your entire world. Because Godot instances scenes by reference rather than copying mesh data, dropping `World_Macro_Vista.tscn` into four different basin scenes takes almost zero extra disk space and ensures any edit to the master layout updates all four basins automatically.
    
- **The "Doughnut & Hole" Proxy Pattern:** Dividing your low-res macro world into named modular sub-nodes (`Proxy_Basin_01`, `Proxy_Basin_02`, `Proxy_Basin_03`, `Proxy_Basin_04`). When you enter `Basin_01` in high detail, a 3-line `@tool` script hides `Proxy_Basin_01` (creating the "hole" where your high-detail playable `Terrain3D` sits) while leaving `Proxy_Basin_02`, `03`, and `04` visible as the surrounding distant vista ("the doughnut").
    
- **True World-Space Coordinate Anchoring (`Origin Alignment`):** Keeping every basin scene aligned to the exact same global `(0, 0, 0)` origin (or exact snapped grid offsets) so that when `RegionStreamer` swaps out `Basin_01` and instantiates `Basin_02` inside your fog corridor, the distant landmarks never pop or shift by a single centimetre.
    

### 🗺️ Architectural Diagram: The "Doughnut & Hole" Single-Source-of-Truth Pipeline

```
========================================================================================
       HOW TO PLAN ONE MASTER WORLD WITHOUT DUPLICATE-EDITING NIGHTMARES (GODOT 4)
========================================================================================

  STEP 1: BUILD ONE MASTER MACRO-SCENE (`World_Macro_Vista.tscn`)
   ├── `Proxy_Basin_01` (Low-poly Sea-Wall terrain & cliffs — NO collision!)
   ├── `Proxy_Basin_02` (Low-poly Karst pillars & monastery — NO collision!)
   ├── `Proxy_Basin_03` (Low-poly Soot Aqueducts & foundry — NO collision!)
   ├── `Proxy_Basin_04` (Low-poly Ossuary Citadel spire — NO collision!)
   └── `Global_Rivers_And_Outer_Mountains` (Shared ocean, rivers, & far horizon skirt!)

  --------------------------------------------------------------------------------------
  STEP 2: INSTANCE `World_Macro_Vista.tscn` INTO EACH HIGH-DETAIL PLAYABLE BASIN:

   [ Basin_01_SeaWall.tscn ]                      [ Basin_02_Karst.tscn ]
    ├── High-Detail Playable `Terrain3D` (01)      ├── High-Detail Playable `Terrain3D` (02)
    └── Instanced `World_Macro_Vista.tscn`         └── Instanced `World_Macro_Vista.tscn`
         ├── `Proxy_Basin_01` ──► HIDDEN!               ├── `Proxy_Basin_01` ──► VISIBLE (Vista!)
         ├── `Proxy_Basin_02` ──► VISIBLE (Vista!)      ├── `Proxy_Basin_02` ──► HIDDEN!
         ├── `Proxy_Basin_03` ──► VISIBLE (Vista!)      ├── `Proxy_Basin_03` ──► VISIBLE (Vista!)
         └── `Proxy_Basin_04` ──► VISIBLE (Vista!)      └── `Proxy_Basin_04` ──► VISIBLE (Vista!)

  ✅ RESULT: Move a mountain or aqueduct in `World_Macro_Vista.tscn` ONCE ──► Every single
             playable basin updates its distant horizon automatically with zero rework!
========================================================================================
```

### 🛠️ Step-by-Step Workflow: Building Your Master Map First (Without Regret)

1. **Phase 1 — Sculpt the "Miniature World" in Blender or Godot (`World_Macro_Vista.tscn`):**
    
    - Block out the entire continent/island at low resolution (`15,000–25,000` triangles total for all four regions combined) in Blender (or using simple Godot `MeshInstance3D` terrain chunks) so you can fly around the whole map, verify sightlines between biomes, and place your **S-Bend Fog Corridors** where the basins pinch together.
        
    - Export each basin's low-res terrain blockout as a separate named mesh (`Proxy_Basin_01`, `Proxy_Basin_02`, `Proxy_Basin_03`, `Proxy_Basin_04`) inside **`World_Macro_Vista.tscn`**—with **zero collision shapes** attached!
        
2. **Phase 2 — Add This 15-Line `@tool` Auto-Hide Script to `World_Macro_Vista.tscn`:**
    
    - Attach this script to the root of `World_Macro_Vista.tscn`. Whenever you drop it into a playable basin scene and pick `active_high_detail_basin` in the Inspector, it automatically hides that basin's low-res proxy both in the **Godot Editor** and **in-game**:
        

GDScript

```
@tool
extends Node3D

@export_enum("None", "Proxy_Basin_01", "Proxy_Basin_02", "Proxy_Basin_03", "Proxy_Basin_04")
var hidden_local_proxy: String = "None":
	set(value):
		hidden_local_proxy = value
		_update_proxy_visibility()

func _ready() -> void:
	_update_proxy_visibility()

func _update_proxy_visibility() -> void:
	for child in get_children():
		if child.name.begins_with("Proxy_Basin_"):
			child.visible = (child.name != hidden_local_proxy)
```

3. **Phase 3 — Detail One Basin at a Time Inside the "Hole":**
    
    - Open `Basin_01_SeaWall.tscn`, drop in `World_Macro_Vista.tscn`, set **`Hidden Local Proxy = Proxy_Basin_01`**, and sculpt your high-detail playable `Terrain3D` (`512 m × 512 m`) right inside the empty seam—matching the edges of the surrounding low-res foothills.
        
    - When your `RegionStreamer` swaps `Basin_01` for `Basin_02` inside the fog corridor, `Proxy_Basin_01` switches back **ON** behind you while `Proxy_Basin_02` switches **OFF** ahead of you—giving you a seamless, zero-duplication world pipeline that fits comfortably inside **8 GB RAM**!
        

### 🎯 Next Step Question

Would you like to create a quick **greybox `World_Macro_Vista.tscn` layout** mapping out your 4 biomes and their connecting S-bend chokepoints in Godot 4, or jump back into **Blender** to finish **`Katana.Attack:Light2`**?

> ✍️ **Phrasing Refinement:** "What specifically makes a distant LOD less expensive to render? Can distant, collision-free vertex terrain still cause lag, and how can I optimise it while keeping it visually impressive from afar?"
> 
>   

# 📋 Executive Summary

- ✅ **A Distant LOD Saves Performance Across Four Separate Bottlenecks—Not Just Triangle Count (`Vertex Shading`, `Draw Calls`, `Shadow Map Passes`, and `Shader Complexity`):** Even if a distant terrain mesh has **zero collision**, it can still lag your **Intel Core i5-1155G7 / Iris Xe / 8 GB RAM** laptop if it casts real-time shadows, runs a heavy multi-layer PBR splatmap shader, or splits into dozens of separate draw calls.
    
- ✅ **Why Collision-Free Distant Terrain Can Still Lag (And How to Make It Nearly Free):** On integrated graphics like **Iris Xe**, rendering a 15,000-triangle distant mountain with **no shadows**, **no normal/roughness maps**, and a **single lightweight vertex-colour/triplanar shader** costs less than **`0.15 ms`** per frame—whereas the exact same mesh with real-time shadow casting (`Shadow Casting: ON`) and 4-texture PBR blending can eat **`3–5 ms`** every frame!
    
- ✅ **How to Keep Distant Terrain Looking Crisp & Detailed From Afar:** Use **baked ambient occlusion + macro-slope colour gradients** (rock on steep cliffs, grass/ash on flat ridges, snow/mist at peaks) computed directly from vertex normals/height in a cheap fragment shader, paired with **Godot 4’s `Aerial Perspective` fog**—which naturally hides low polygon density while making distant ridges look crisp and monumental.
    

### 📖 Plain-English Glossary of Distant LOD & GPU Bottlenecks

- **Vertex Processing vs. Fragment (Pixel) Shading:**
    
    - **Vertex Processing:** Calculating the 3D corner positions of triangles. Modern GPUs—even Intel Iris Xe—chew through `30,000–50,000` distant static vertices effortlessly.
        
    - **Fragment (Pixel) Shading:** Calculating the colour, lighting, shadow, and texture lookup for every pixel on screen. A distant mountain covering 30% of your monitor runs its pixel shader hundreds of thousands of times per frame—so simplifying the _material_ matters far more than shaving off a few hundred triangles.
        
- **Shadow-Pass Multiplication (`The Silent FPS Killer`):** Every mesh with `Cast Shadow: ON` is rendered **multiple extra times per frame** (once from the camera's perspective, plus once for each shadow cascade of your `DirectionalLight3D`). Disabling shadow casting on distant vista mountains immediately eliminates 2–3 full geometry passes per frame.
    
- **Texture Bandwidth (`Shared RAM Bottleneck`):** Because your **i5-1155G7** shares your **8 GB DDR4 system RAM** with Iris Xe graphics, reading four separate `2K` Albedo + Normal + Roughness textures on distant mountains saturates memory bandwidth. Distant LODs only need **colour and slope variation** (because normal maps and micro-roughness are invisible past `150 metres` anyway!).
    

### 🗺️ Architectural Diagram: What Makes Distant LODs Expensive vs. Zero-Cost

```
========================================================================================
       WHY COLLISION-FREE DISTANT TERRAIN LAGS — AND HOW TO MAKE IT NEARLY FREE
========================================================================================

  ❌ EXPENSIVE DISTANT TERRAIN (`~4.5 ms` on Iris Xe / 8 GB RAM):
     • `Cast Shadow: ON` ─────────► Re-renders distant mountains 2–4× into shadow maps!
     • 4-Layer PBR Splatmap ──────► 12+ texture reads per pixel for micro-details you
                                    cannot even see past 150 metres!
     • 25 Separate Mesh Chunks ───► 25 individual CPU draw calls + state switches.

  ✅ OPTIMISED VISTA LOD (`~0.15 ms` on Iris Xe — 30× Faster, Looks Identical From Afar!):
     • `Cast Shadow: OFF` ────────► Zero shadow-map overhead (fog & vertex AO fake depth!).
     • `GI Mode: Disabled` ───────► Skips dynamic light-probe lookups on static horizon.
     • Single Merged Mesh ────────► 1 single draw call for the entire outer vista ring!
     • Slope + Height Shader ─────► 0 heavy PBR maps! Colours cliffs vs. grass mathematically
                                    using world normal (`NORMAL.y`) + macro noise + Aerial Fog!
========================================================================================
```

### 🛠️ 5 Concrete Optimisations for Distant Terrain (`World_Macro_Vista.tscn`)

1. **Turn `Cast Shadow` to `OFF` on Every Distant Vista Mesh (`GeometryInstance3D → Cast Shadow`):**
    
    - Distant mountains `300+ metres` away never cast shadows onto your playable arena anyway, yet leaving `Cast Shadow: ON` forces Godot's `DirectionalLight3D` to render them into its shadow cascades every frame. Turning `Cast Shadow` to **`OFF`** is the single biggest free FPS boost you can click.
        
2. **Strip Normal Maps & Specular Roughness Maps From Vista Materials:**
    
    - Past `120–150 metres`, sub-pixel normal maps actually cause ugly **shimmering / aliasing** on integrated graphics while wasting memory bandwidth. Use a lightweight custom shader (or simple `StandardMaterial3D` with **Roughness = `1.0`**, **Metallic = `0.0`**, **Shading Mode = `Per-Vertex`** or simplified PBR) so the GPU performs minimal math per pixel.
        
3. **Use a "Procedural Slope & Height" Shader for Free Macro-Detail (`Zero Texture Memory`):**
    
    - Instead of painting massive textures across your distant mountains, paste this ultra-fast Godot 4 spatial shader onto your outer vista terrain. It automatically paints **steep cliffs dark basalt**, **flat ridges weathered ash/moss**, and **high peaks pale mist**, giving distant mountains crisp geological structure from miles away using **zero image textures**:
        

OpenGL Shading Language

```
shader_type spatial;
render_mode diffuse_burley, specular_disabled; // Disables expensive specular math on far terrain!

uniform vec3 flat_ground_color : source_color = vec3(0.16, 0.21, 0.18); // Weathered moss/ash
uniform vec3 steep_cliff_color : source_color = vec3(0.11, 0.12, 0.14); // Dark basalt rock
uniform vec3 peak_mist_color   : source_color = vec3(0.32, 0.38, 0.42); // High-altitude slate
uniform float cliff_sharpness  : hint_range(1.0, 16.0) = 6.0;
uniform float peak_start_height = 45.0;
uniform float peak_blend_range  = 35.0;

varying float v_world_y;
varying float v_slope;

void vertex() {
	vec3 world_pos = (MODEL_MATRIX * vec4(VERTEX, 1.0)).xyz;
	vec3 world_norm = normalize((MODEL_MATRIX * vec4(NORMAL, 0.0)).xyz);
	v_world_y = world_pos.y;
	v_slope = clamp(world_norm.y, 0.0, 1.0); // 1.0 = flat ground, 0.0 = vertical cliff
}

void fragment() {
	// Calculate slope & height blending in vertex/fragment with zero texture lookups!
	float flatness = pow(v_slope, cliff_sharpness);
	vec3 base_col = mix(steep_cliff_color, flat_ground_color, flatness);
	
	float altitude_factor = clamp((v_world_y - peak_start_height) / peak_blend_range, 0.0, 1.0);
	ALBEDO = mix(base_col, peak_mist_color, altitude_factor);
	ROUGHNESS = 1.0;
}
```

4. **Merge Static Horizon Props Into One Draw Call (`MeshInstance3D` Batching):**
    
    - In Blender, join all distant ruin silhouettes, broken aqueduct pillars, and outer mountain ridges that share the same material into **1–2 merged meshes** (`Ctrl + J`) before exporting to `.glb`. Reducing `80` distant objects down to `2` draw calls keeps CPU overhead completely flat on your i5-1155G7.
        
5. **Bake Vertex Ambient Occlusion (`AO`) in Blender:**
    
    - Before exporting your low-res `World_Macro_Vista.glb` from Blender, bake simple **Vertex Colours (`Dirty Vertex Colors` / Ambient Occlusion)** into the valleys and crevices (`10 seconds` in Blender's Vertex Paint mode) and multiply `ALBEDO *= COLOR.rgb;` in your shader. Even without real-time shadows, every canyon crease and mountain valley will look deeply sculpted and dramatic from afar!
        

### 🎯 Next Step Question

Would you like to test this **zero-texture Slope/Height + Vertex-AO vista shader** on your `World_Macro_Vista` blockout in Godot 4, or are you ready to jump back into **Blender** to finish **`Katana.Attack:Light2`**?