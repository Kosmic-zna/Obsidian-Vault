# 🗺️ Module IX: Level Architecture, Procedural Generation (PCG) & World Streaming

> [!info] 📖 Lexicon & First Principles: Spatial Design & Algorithmic Synthesis
> Moving from localized mechanics to expansive game worlds requires a rigorous understanding of architectural psychology, discrete procedural topology, and physical memory limits:
> - ⤷ **Greyboxing (Blockmesh)**: The rapid iterative prototyping of playable levels using primitive untextured geometric volumes, calibrating spatial pacing, jump trajectories, and sightlines before committing to art assets.
> - ⤷ **Weenie (Landmark)**: An architectural term coined by Walt Disney for a visually dominant vertical beacon placed on the horizon, enabling players to orient themselves intuitively via spatial triangulation without relying on intrusive mini-maps.
> - ⤷ **Binary Space Partitioning (BSP)**: A geometric algorithm that recursively divides Euclidean space into convex sub-spaces using hyperplanes, generating hierarchical spatial trees for room distribution and visibility sorting.
> - ⤷ **Cellular Automata**: A discrete mathematical model consisting of a regular grid of cells, each evolving through discrete time steps according to fixed deterministic rules based on the states of its local neighbours.
> - ⤷ **Wave Function Collapse (WFC)**: A constraint-satisfaction algorithm based on quantum mechanics terminology, wherein a grid of unobserved possibilities (superpositions) collapses into deterministic modules while preserving strict socket-adjacency rules.
> - ⤷ **Floating Origin Shift**: An architectural coordinate transformation that offsets the physical root of the virtual world relative to the player to combat the severe floating-point precision degradation inherent to IEEE 754 representations at vast distances.

---

## 9.1 Level Design Foundations: Composition, Spatial Flow & Arena Metrics

Engineers frequently treat level design as mere "art placement." In reality, level design is **spatial psychology and mechanical guidance**. A level functions as an unscripted tutor guiding the player through geometry, lighting contrast, and architectural apertures.

```
                      THE SPATIAL COMPOSITION HIERARCHY
=================================================================================
 [ THE WEENIE (Beacon) ]   ──► Distant vertical reference point (Triangulation)
           │
           ▼
 [ APERTURES & FRAMING ]   ──► Gateways, doorways, and arches that limit sightlines
           │
           ▼
 [ LEADING LINES & EDGES ] ──► Linear architectural contours directing eye gaze
           │
           ▼
 [ INTENSITY CONTRAST ]    ──► High-value luminance pooling over critical paths
```

### 1. Spatial Framing & Leading Lines
- ⤷ **Architectural Framing**: Use foreground structures (collapsed archways, cave mouths, tunnel apertures) to frame focal points. The human visual cortex naturally filters out dark peripheral boundaries and locks onto brightly illuminated geometric centers.
- ⤷ **Leading Lines**: Run continuous geometry (cables, road curbs, ceiling pipes, floor tiles) directly toward the objective. If the player is lost, their subconscious follows planar contours.
- ⤷ **Choke Points & Valve Design**: Design one-way drops (ledges, collapsed tunnels) to prevent tedious backtracking and partition levels into discrete memory zones.

### 2. Combat Arena Topology
Never construct flat, symmetrical boxes for combat encounters. Professional action spaces rely on three topological invariants:

```
                         ARENA TOPOLOGY TOPOGRAPHY
=================================================================================
 (Planar Elevation Split)              (The Circulation Loop)
         [ High Perch ]                      ┌───────────┐
            /      \                         │  Cover A  │ ◄── Flank Vector
     Ramps /        \ Drops                  └─────┬─────┘
          v          v                             │
  [ Low Trenches / Flanks ]                  ┌─────┴─────┐
  (Forces dynamic relocation)                │  Cover B  │
                                             └───────────┘
```

- ⤷ **Vertical Asymmetry**: Provide at least two distinct elevation planes ($y = 0.0\text{ m}$ and $y = 3.5\text{ m}$). High ground grants tactical line-of-sight advantages but limits egress routes; low ground provides visual cover but sacrifices battlefield overview.
- ⤷ **Circulation Loops (Continuous Flow)**: Eliminate dead-end corridors within combat spaces. Every flank must connect to an outer circular loop, ensuring the player and AI can execute repositioning maneuvers without becoming cornered.

### 3. Rapid Greyboxing with Godot 4 CSG Nodes

Godot 4 features native **Constructive Solid Geometry (CSG)** nodes (`CSGBox3D`, `CSGCylinder3D`, `CSGCombiner3D`) supporting real-time boolean operations (**Union**, **Intersection**, **Subtraction**).

> [!warning] ⚠️ Production Trap: Shipping CSG Nodes in Release Builds
> CSG nodes dynamically rebuild non-manifold polygon meshes on the CPU whenever transformations change, incurring massive draw-call penalties and preventing secondary-UV lightmap baking.
> **The Engineering Law**: Prototype levels using CSG. Once layout metrics are validated, select the root `CSGCombiner3D` in the editor, click **Export to Mesh**, and replace the CSG hierarchy with a static, pre-indexed `MeshInstance3D` paired with a simplified `ConcavePolygonShape3D`!

---

## 9.2 The Tile & Grid Server Ecosystem: `TileMapLayer` & `GridMap`

### 1. 2D TileMap Architecture (Godot 4.3+ `TileMapLayer`)
In Godot 4.3+, the bloated legacy `TileMap` node was refactored into modular, high-speed **`TileMapLayer`** nodes. Each layer operates as a distinct 2D canvas item with its own Y-sorting, navigation masks, and physics collision parameters.

```
                      MODULAR TILEMAP LAYER TOPOLOGY
=================================================================================
 [ World Node2D ]
  ├── GroundLayer (TileMapLayer)       ──► Z-Index: 0 (No collision, basic terrain)
  ├── ObstaclesLayer (TileMapLayer)    ──► Z-Index: 1 (Y-Sort: True, StaticBody shapes)
  ├── ForegroundLayer (TileMapLayer)   ──► Z-Index: 2 (Roofs, bridges, foliage canopies)
  └── NavigationLayer (TileMapLayer)   ──► Z-Index: -1 (Baked AStar2D navigation polygons)
```

- ⤷ **Terrain Autotiling Bitmasks**: Godot's terrain system evaluates adjacent cells using **3x3 Minimal** or **16-pipe Connect** bitmasks. Cells identify their neighbors using compass bit flags:
  $$\text{Neighbor Mask} = \sum_{k=0}^{7} 2^k \cdot B_k \quad \text{where } B_k \in \{0, 1\}$$

### 2. 3D Discrete Gridded Worlds (`GridMap`)
The 3D equivalent of a `TileMap` is the **`GridMap`**, which positions static `MeshLibrary` items into discrete 3D spatial integer cells ($x, y, z \in \mathbb{Z}^3$).
- ⤷ **Memory Locality**: Under the hood, `GridMap` does not allocate separate `Node3D` instances. It registers octree-partitioned batches of meshes directly with the `RenderingServer` and `PhysicsServer3D`, rendering thousands of structural blocks in a fraction of a millisecond.

---

## 9.3 Procedural Generation (PCG): The Algorithmic Engine

Procedural Content Generation is not an arbitrary scattering of seeds; it is the **computational synthesis of structured spaces governed by formal mathematical constraints**.

```
                         PCG ALGORITHM DOMAIN MAPPING
=================================================================================
 BSP Room Slicing           ──► Rectangular interior dungeons, corporate offices
 Cellular Automata          ──► Organic caverns, subterranean tunnels, erosion
 Wave Function Collapse     ──► Detailed urban structures, architectural pipe grids
 Simplex Noise / fBm        ──► Planetary heightmaps, biome continental distributions
 A* & Minimum Spanning Tree ──► Non-linear dungeon room pathing & hallway networks
```

### 1. Binary Space Partitioning (BSP) Dungeon Generation

The BSP algorithm recursively subdivides a 2D bounding volume into a binary tree of smaller rectangular leaves until a minimum threshold dimension is reached. Each leaf then spawns an interior room, which are subsequently connected via hallways along the tree's split planes.

```
                         BSP RECURSIVE SUBDIVISION
=================================================================================
  Step 0: Root Container      Step 1: Vertical Slice       Step 2: Horizontal Slices
  ┌───────────────────────┐   ┌───────────┬───────────┐    ┌───────────┬───────────┐
  │                       │   │           │           │    │  Room A   │  Room C   │
  │                       │   │           │           │    ├───────────┼───────────┤
  │                       │   │  Leaf 1   │  Leaf 2   │    │  Room B   │  Room D   │
  └───────────────────────┘   └───────────┴───────────┘    └───────────┴───────────┘
```

```gdscript
# ==============================================================================
# RECURSIVE BINARY SPACE PARTITIONING (BSP) DUNGEON GENERATOR
# ==============================================================================
class_name BSPDungeonGenerator
extends RefCounted

class BSPLeaf:
    var rect: Rect2i
    var left_child: BSPLeaf = null
    var right_child: BSPLeaf = null
    var room: Rect2i

    func _init(p_rect: Rect2i) -> void:
        rect = p_rect

    func split(min_leaf_size: int) -> bool:
        if left_child != null or right_child != null:
            return false # Already partitioned
            
        # Determine split orientation based on dimensional aspect ratio
        var split_horizontal: bool = randf() > 0.5
        if rect.size.x > rect.size.y and float(rect.size.x) / float(rect.size.y) >= 1.25:
            split_horizontal = false
        elif rect.size.y > rect.size.x and float(rect.size.y) / float(rect.size.x) >= 1.25:
            split_horizontal = true

        var max_split: int = (rect.size.y if split_horizontal else rect.size.x) - min_leaf_size
        if max_split <= min_leaf_size:
            return false # Sub-space is too constrained to split further

        var split_pos: int = randi_range(min_leaf_size, max_split)

        if split_horizontal:
            left_child = BSPLeaf.new(Rect2i(rect.position.x, rect.position.y, rect.size.x, split_pos))
            right_child = BSPLeaf.new(Rect2i(rect.position.x, rect.position.y + split_pos, rect.size.x, rect.size.y - split_pos))
        else:
            left_child = BSPLeaf.new(Rect2i(rect.position.x, rect.position.y, split_pos, rect.size.y))
            right_child = BSPLeaf.new(Rect2i(rect.position.x + split_pos, rect.position.y, rect.size.x - split_pos, rect.size.y))

        return true

    func create_rooms(min_room_size: int) -> void:
        if left_child != null or right_child != null:
            if left_child != null: left_child.create_rooms(min_room_size)
            if right_child != null: right_child.create_rooms(min_room_size)
        else:
            # Carve room safely inside leaf boundaries with 1-tile structural padding
            var room_w: int = randi_range(min_room_size, rect.size.x - 2)
            var room_h: int = randi_range(min_room_size, rect.size.y - 2)
            var room_x: int = randi_range(rect.position.x + 1, rect.position.x + rect.size.x - room_w - 1)
            var room_y: int = randi_range(rect.position.y + 1, rect.position.y + rect.size.y - room_h - 1)
            room = Rect2i(room_x, room_y, room_w, room_h)

## Generates a complete floor plan array: 0 = Empty/Void, 1 = Floor, 2 = Wall
static func generate_dungeon_grid(width: int, height: int, min_size: int = 10) -> Array:
    var root: BSPLeaf = BSPLeaf.new(Rect2i(0, 0, width, height))
    var leaves: Array[BSPLeaf] = [root]
    
    var did_split: bool = true
    while did_split:
        did_split = false
        for leaf in leaves.duplicate():
            if leaf.left_child == null and leaf.right_child == null:
                if leaf.split(min_size):
                    leaves.append(leaf.left_child)
                    leaves.append(leaf.right_child)
                    did_split = true

    root.create_rooms(min_size / 2)
    
    # Allocate discrete 2D grid
    var grid: Array = []
    grid.resize(width)
    for x in range(width):
        var col: Array = []
        col.resize(height)
        col.fill(0)
        grid[x] = col

    # Carve terminal leaf rooms into the grid
    for leaf in leaves:
        if leaf.room.size.x > 0:
            for rx in range(leaf.room.position.x, leaf.room.end.x):
                for ry in range(leaf.room.position.y, leaf.room.end.y):
                    grid[rx][ry] = 1 # Carve Walkable Floor
                    
    return grid
```

### 2. Cellular Automata: Organic Subterranean Cavern Synthesis

To generate natural cave systems, we model physical rock erosion using a discrete 2D Cellular Automaton governed by the **Moore Neighborhood** ($8$ adjacent cells):

```
                        MOORE NEIGHBORHOOD (Radius = 1)
=================================================================================
                         [x-1, y-1]  [ x, y-1 ]  [x+1, y-1]
                         [x-1,  y ]  [ Cell   ]  [x+1,  y ]
                         [x-1, y+1]  [ x, y+1 ]  [x+1, y+1]
```

#### The Cave Transformation Rule (B5678 / S45678)
1. Initialize the grid randomly: each cell has a $45\%$ probability of being a solid **Wall** ($1$) and $55\%$ probability of being **Empty Air** ($0$).
2. For each simulation step, compute the count of solid wall neighbors $W_N$:
   - If $W_N \ge 5$: The cell becomes a solid **Wall** (consolidation).
   - If $W_N \le 3$: The cell becomes **Empty Air** (erosion).
   - If $W_N == 4$: The cell maintains its existing state.
3. Iterate $4\text{--}6$ generations. The random noise coalesces into smooth, winding natural limestone corridors!

```gdscript
# ==============================================================================
# ORGANIC CELLULAR AUTOMATA CAVERN SYNTHESIS
# ==============================================================================
class_name CellularCaveGenerator
extends RefCounted

## Executes smoothing generations on a randomized boolean grid
static func generate_caves(width: int, height: int, fill_ratio: float = 0.45, iterations: int = 5) -> Array:
    var grid: Array = []
    grid.resize(width)
    
    # 1. Stochastic Seeding
    for x in range(width):
        var col: Array = []
        col.resize(height)
        for y in range(height):
            # Enforce solid bounding border around level perimeter
            if x == 0 or x == width - 1 or y == 0 or y == height - 1:
                col[y] = 1
            else:
                col[y] = 1 if randf() < fill_ratio else 0
        grid[x] = col

    # 2. Iterative Cellular Automata Simulation Steps
    for step in range(iterations):
        var buffer: Array = []
        buffer.resize(width)
        for x in range(width):
            var col: Array = []
            col.resize(height)
            for y in range(height):
                var wall_count: int = _count_moore_walls(grid, x, y, width, height)
                
                # B5678 / S45678 Rule evaluation
                if wall_count > 4:
                    col[y] = 1
                elif wall_count < 4:
                    col[y] = 0
                else:
                    col[y] = grid[x][y]
            buffer[x] = col
        grid = buffer

    return grid

static func _count_moore_walls(map: Array, grid_x: int, grid_y: int, max_w: int, max_h: int) -> int:
    var walls: int = 0
    for ox in range(-1, 2):
        for oy in range(-1, 2):
            if ox == 0 and oy == 0:
                continue
            var nx: int = grid_x + ox
            var ny: int = grid_y + oy
            # Out-of-bounds cells evaluate as solid rock (walls)
            if nx < 0 or nx >= max_w or ny < 0 or ny >= max_h:
                walls += 1
            elif map[nx][ny] == 1:
                walls += 1
    return walls
```

### 3. Wave Function Collapse (WFC): Local Adjacency Constraint Propagation

Wave Function Collapse (WFC) generates structured patterns (such as complex pipe networks, city streets, or detailed building interiors) by treating each grid cell as a quantum-like **Superposition** of all possible structural tiles.

```
                        THE WFC REDUCTION PIPELINE
=================================================================================
 [ Superposition ] ──► [ 1. Find Cell with Lowest Shannon Entropy (Fewest Choices) ]
                                │
                                ▼
                       [ 2. Collapse Cell to ONE Random Valid Tile ]
                                │
                                ▼
                       [ 3. Propagate Adjacency Constraints to Neighbors ]
                         (Prune invalid socket choices from neighbor superpositions)
                                │
                                ▼
                       (Repeat until all cells collapsed, or backtrack on Contradiction!)
```

#### Mathematical Entropy & Constraint Propagation
The **Shannon Entropy** $H$ of a cell containing $N$ valid tile candidates is computed as:

$$H = \log_2(N)$$

When tile weights $w_i$ are assigned (e.g., plain grass is 10x more common than a road crossing):

$$H = \log_2\left(\sum_{i=1}^{N} w_i\right) - \frac{\sum_{i=1}^{N} w_i \log_2(w_i)}{\sum_{i=1}^{N} w_i}$$

```gdscript
# ==============================================================================
# PRODUCTION WAVE FUNCTION COLLAPSE (WFC) MODEL (2D Socket-Based)
# ==============================================================================
class_name WaveFunctionCollapse2D
extends RefCounted

## Sockets define boundary compatibility: [North, East, South, West]
class WFCTile:
    var id: int
    var sockets: Array[String] # e.g. ["Grass", "Road", "Grass", "Grass"]
    var weight: float = 1.0
    
    func _init(p_id: int, p_sockets: Array[String], p_weight: float = 1.0) -> void:
        id = p_id
        sockets = p_sockets
        weight = p_weight

var _tile_palette: Array[WFCTile] = []
var _grid_width: int
var _grid_height: int
## 2D Array where each element is an Array[WFCTile] representing active superposition
var _wave: Array = []

func _init(width: int, height: int, palette: Array[WFCTile]) -> void:
    _grid_width = width
    _grid_height = height
    _tile_palette = palette
    _initialize_wave()

func _initialize_wave() -> void:
    _wave.resize(_grid_width)
    for x in range(_grid_width):
        var col: Array = []
        col.resize(_grid_height)
        for y in range(_grid_height):
            # Every cell starts in full superposition of all possible tiles
            col[y] = _tile_palette.duplicate()
        _wave[x] = col

## Runs the complete collapse simulation loop; returns false on Contradiction
func run_collapse() -> bool:
    while true:
        var candidate_cell: Vector2i = _find_lowest_entropy_cell()
        if candidate_cell == Vector2i(-1, -1):
            return true # All cells collapsed successfully! Zero entropy remains.

        var cell_options: Array = _wave[candidate_cell.x][candidate_cell.y]
        if cell_options.is_empty():
            push_error("WFC Contradiction: Cell reached 0 valid states at %s" % str(candidate_cell))
            return false

        # Collapse cell to a single choice based on relative weights
        var chosen_tile: WFCTile = _sample_weighted_tile(cell_options)
        _wave[candidate_cell.x][candidate_cell.y] = [chosen_tile]

        # Propagate constraints outwards via stack-based queue
        _propagate_constraints(candidate_cell)
        
    return true

func _find_lowest_entropy_cell() -> Vector2i:
    var lowest_count: int = INF
    var best_coord: Vector2i = Vector2i(-1, -1)
    
    for x in range(_grid_width):
        for y in range(_grid_height):
            var count: int = _wave[x][y].size()
            if count > 1 and count < lowest_count:
                lowest_count = count
                best_coord = Vector2i(x, y)
                
    return best_coord

func _sample_weighted_tile(options: Array) -> WFCTile:
    var total_weight: float = 0.0
    for tile in options: total_weight += tile.weight
    var roll: float = randf_range(0.0, total_weight)
    var accum: float = 0.0
    for tile in options:
        accum += tile.weight
        if roll <= accum:
            return tile
    return options.back()

func _propagate_constraints(start_pos: Vector2i) -> void:
    var stack: Array[Vector2i] = [start_pos]
    var directions: Array[Vector2i] = [Vector2i(0, -1), Vector2i(1, 0), Vector2i(0, 1), Vector2i(-1, 0)] # N, E, S, W
    
    while not stack.is_empty():
        var current: Vector2i = stack.pop_back()
        var current_options: Array = _wave[current.x][current.y]

        for dir_idx in range(4):
            var offset: Vector2i = directions[dir_idx]
            var neighbor: Vector2i = current + offset
            
            if neighbor.x < 0 or neighbor.x >= _grid_width or neighbor.y < 0 or neighbor.y >= _grid_height:
                continue

            var neighbor_options: Array = _wave[neighbor.x][neighbor.y]
            if neighbor_options.size() <= 1:
                continue # Already collapsed

            var opposing_dir: int = (dir_idx + 2) % 4
            var valid_neighbor_tiles: Array = []

            for candidate in neighbor_options:
                var is_compatible: bool = false
                for source in current_options:
                    # Check socket identity across the shared interface edge
                    if source.sockets[dir_idx] == candidate.sockets[opposing_dir]:
                        is_compatible = true
                        break
                if is_compatible:
                    valid_neighbor_tiles.append(candidate)

            # If choices were pruned, push neighbor onto propagation stack
            if valid_neighbor_tiles.size() < neighbor_options.size():
                _wave[neighbor.x][neighbor.y] = valid_neighbor_tiles
                stack.append(neighbor)
```

### 4. Continuous World Generation: Multi-Octave fBm Noise & Biomes

Large outdoor worlds cannot be generated via discrete room-splitting. We compute continuous, smooth elevation and moisture fields using **Fractional Brownian Motion (fBm)**, sampling multi-octave coherent Simplex noise:

$$H(x, z) = \sum_{k=0}^{M-1} g^k \cdot \text{Noise}\left(\ell^k \cdot x, \; \ell^k \cdot z\right)$$

where $M$ is the octave count, $g$ is **Gain / Persistence** ($g \approx 0.5$), and $\ell$ is **Lacunarity** ($\ell \approx 2.0$).

```
                      THE WHITTAKER BIOME LOOKUP DIAGRAM
=================================================================================
 Moisture (M)
   ▲
1.0│  [ Tundra / Boreal ]      [ Temperate Forest ]      [ Tropical Rainforest ]
   │
0.5│  [ Grassland / Steppe ]   [ Shrubland / Woodland ]  [ Seasonal Savanna ]
   │
0.0│  [ Arctic Ice Sheet ]     [ Rocky Badlands ]        [ Arid Sand Desert ]
   └───────────────────────────────────────────────────────────────────────────►
  0.0                               0.5                                     1.0
                                                                  Temperature (T)
```

```gdscript
# ==============================================================================
# BIOME ALLOCATION MATRIX & FBM ELEVATION SAMPLER
# ==============================================================================
class_name BiomeSynthesizer
extends RefCounted

enum BiomeType { ARCTIC, TUNDRA, GRASSLAND, DESERT, FOREST, RAINFOREST }

var _elevation_noise: FastNoiseLite
var _moisture_noise: FastNoiseLite

func _init(world_seed: int) -> void:
    _elevation_noise = FastNoiseLite.new()
    _elevation_noise.seed = world_seed
    _elevation_noise.noise_type = FastNoiseLite.TYPE_SIMPLEX_SMOOTH
    _elevation_noise.fractal_octaves = 5
    _elevation_noise.fractal_lacunarity = 2.0
    _elevation_noise.fractal_gain = 0.5
    _elevation_noise.frequency = 0.003

    _moisture_noise = FastNoiseLite.new()
    _moisture_noise.seed = world_seed + 101
    _moisture_noise.noise_type = FastNoiseLite.TYPE_SIMPLEX_SMOOTH
    _moisture_noise.fractal_octaves = 4
    _moisture_noise.frequency = 0.002

## Returns continuous height and discrete BiomeType for any world coordinate (x, z)
func sample_world(x: float, z: float) -> Dictionary:
    # Samples normalized to [0.0, 1.0]
    var elevation: float = (_elevation_noise.get_noise_2d(x, z) + 1.0) * 0.5
    var moisture: float = (_moisture_noise.get_noise_2d(x, z) + 1.0) * 0.5
    
    var biome: BiomeType
    if elevation < 0.2:
        biome = BiomeType.ARCTIC
    elif moisture < 0.25:
        biome = BiomeType.DESERT if elevation > 0.4 else BiomeType.GRASSLAND
    elif moisture > 0.70:
        biome = BiomeType.RAINFOREST if elevation > 0.3 else BiomeType.FOREST
    else:
        biome = BiomeType.TUNDRA if elevation > 0.6 else BiomeType.FOREST

    return {
        "elevation": elevation * 120.0, # Scaled height in metres
        "moisture": moisture,
        "biome": biome
    }
```

---

## 9.4 Open-World Streaming, LOD Management & Floating Origin Shifting

Rendering and simulating vast worlds ($10\text{ km} \times 10\text{ km}$) requires streaming geometry dynamically in the background while circumventing hardware memory and precision constraints.

```
                      HIERARCHICAL WORLD CHUNK MANAGEMENT
=================================================================================
 [ Camera View Frustum ]
         │
         ├── Outer Radius (> 500m)  ──► Chunk Instantiation: Low-Poly Mesh Impostors
         ├── Medium Radius (150-500m)──► Standard Geometry (HLOD Tier 1, Shadows off)
         └── Active Radius (< 150m) ──► Full Collision, Navigation, Dynamic Actors
```

### 1. Asynchronous Threaded Chunk Streaming

Loading heavy `.tscn` chunks synchronously via `load()` blocks the main thread, dropping frames and causing noticeable stutter. Godot 4 resolves this via **`ResourceLoader.load_threaded_request()`**:

```gdscript
# ==============================================================================
# ASYNCHRONOUS RESOURCE CHUNK STREAMER
# ==============================================================================
class_name ChunkStreamer
extends Node

signal chunk_loaded(coord: Vector2i, scene: PackedScene)

var _loading_queue: Dictionary = {} # Vector2i -> ResourceLoader.ThreadLoadStatus

func request_chunk_stream(coord: Vector2i, file_path: String) -> void:
    if _loading_queue.has(coord):
        return
        
    var err: Error = ResourceLoader.load_threaded_request(file_path, "PackedScene")
    if err == OK:
        _loading_queue[coord] = file_path

func _process(_delta: float) -> void:
    for coord in _loading_queue.keys():
        var file_path: String = _loading_queue[coord]
        var progress: Array = []
        var status: ResourceLoader.ThreadLoadStatus = ResourceLoader.load_threaded_get_status(file_path, progress)

        if status == ResourceLoader.THREAD_LOAD_LOADED:
            var scene: PackedScene = ResourceLoader.load_threaded_get(file_path) as PackedScene
            _loading_queue.erase(coord)
            chunk_loaded.emit(coord, scene)
        elif status == ResourceLoader.THREAD_LOAD_FAILED or status == ResourceLoader.THREAD_LOAD_INVALID_RESOURCE:
            push_error("ChunkStreamer: Failed to stream chunk at %s" % file_path)
            _loading_queue.erase(coord)
```

### 2. Floating-Point Precision Degradation & The Floating Origin Solution

Modern GPUs and game engine physics run on **32-bit single-precision floating-point numbers** (IEEE 754). A 32-bit float reserves only $23\text{ bits}$ of mantissa precision.

```
                 IEEE 754 PRECISION DRIFT ACROSS DISTANCE
=================================================================================
 Distance from World Origin (0,0,0)  │ Effective Minimum Spatial Precision (Step Size)
 ────────────────────────────────────┼────────────────────────────────────────────
 Under 1,000 metres (1 km)           │ ~0.00006 mm  (Sub-microscopic precision)
 At 8,192 metres (8.2 km)            │ ~1.0 mm      (Slight vertex wobble)
 At 65,536 metres (65.5 km)          │ ~8.0 mm      (Noticeable jitter & physics pops)
 At 262,144 metres (262 km)          │ ~32.0 mm     (Violent camera vibrating; FAIL!)
```

> [!bug] 🚨 The Open-World Jitter Bug
> When a player travels far from the origin, their character's mesh and the surrounding terrain begin vibrating violently ("vertex jitter"). Physics raycasts pass through walls because collision contacts can no longer resolve sub-centimeter vertices.

#### The Architectural Solution: Origin Shifting
When the player crosses a distance threshold $D_{\text{threshold}}$ (typically $4,000\text{ m}$) away from the origin $(0, 0, 0)$, **shift the entire world backwards** by $-\vec{P}_{\text{player}}$, snapping the player back to $(0, 0, 0)$!

```gdscript
# ==============================================================================
# FLOATING ORIGIN SHIFT MANAGER
# ==============================================================================
class_name FloatingOriginManager
extends Node

signal origin_shifted(offset_vector: Vector3)

@export var player_target: Node3D
@export var shift_threshold_meters: float = 4000.0

func _physics_process(_delta: float) -> void:
    if player_target == null:
        return

    var player_pos: Vector3 = player_target.global_position
    # Evaluate planar horizontal threshold
    if Vector2(player_pos.x, player_pos.z).length_squared() > (shift_threshold_meters * shift_threshold_meters):
        _execute_origin_shift(player_pos)

func _execute_origin_shift(shift_vector: Vector3) -> void:
    # 1. Offset the player back to local origin
    player_target.global_position -= shift_vector

    # 2. Offset all active spatial entities and chunks in the simulation
    var world_nodes: Array[Node] = get_tree().get_nodes_in_group(&"floating_origin_affected")
    for entity in world_nodes:
        if entity is Node3D and entity != player_target:
            (entity as Node3D).global_position -= shift_vector

    # 3. Inform particles, audio listeners, and physics queries
    origin_shifted.emit(shift_vector)
    print("FloatingOriginManager: Origin shifted by %s. Coordinates stabilized." % str(shift_vector))
```

---

## 9.5 Complete Applied System: Non-Linear Dungeon Graph & Hallway Carver

To synthesize **BSP Room Partitioning, Graph Triangulation, and GridMap Tile Baking**, consider this complete production dungeon builder. 

It generates a non-linear network of rooms using BSP, connects their centroids via **Delaunay Triangulation**, prunes the graph down to a **Minimum Spanning Tree (Prim's Algorithm)** with $15\%$ re-added cyclical edges to prevent linear backtracking, and carves orthogonal L-shaped corridors:

```
                      DUNGEON ROOM GRAPH TOPOLOGY
=================================================================================
    [ Room 1 ] ══════════════════════════════ [ Room 2 ]
        ║                                         ║
        ║ (MST Backbone)                          ║ (Cycle Loop: 15% edge re-add)
        ║                                         ║
    [ Room 3 ] ══════════════════════════════ [ Room 4 ]
```

```gdscript
# ==============================================================================
# NON-LINEAR GRAPH-BASED PROCEDURAL DUNGEON CARVER
# ==============================================================================
class_name ProceduralDungeonPipeline
extends Node

struct CorridorSegment:
    var start: Vector2i
    var end: Vector2i

@export var grid_width: int = 80
@export var grid_height: int = 80
@export var min_leaf_size: int = 14
@export_range(0.0, 1.0) var cyclical_loop_ratio: float = 0.15

var _grid: Array = []
var _rooms: Array[Rect2i] = []

func build_dungeon() -> void:
    _rooms.clear()
    _grid = BSPDungeonGenerator.generate_dungeon_grid(grid_width, grid_height, min_leaf_size)
    _gather_extracted_rooms()
    _connect_rooms_with_corridors()

func _gather_extracted_rooms() -> void:
    # Find disjoint carved floor islands to locate room centers
    # (In production, leaves store Rect2i rooms directly in the BSP tree)
    pass

func _connect_rooms_with_corridors() -> void:
    if _rooms.size() < 2:
        return

    # 1. Construct Complete Euclidean Distance Graph
    var edges: Array[Dictionary] = []
    for i in range(_rooms.size()):
        for j in range(i + 1, _rooms.size()):
            var p1: Vector2 = Vector2(_rooms[i].get_center())
            var p2: Vector2 = Vector2(_rooms[j].get_center())
            edges.append({
                "room_a": i,
                "room_b": j,
                "weight": p1.distance_squared_to(p2)
            })

    # Sort edges ascending by squared distance
    edges.sort_custom(func(a: Dictionary, b: Dictionary) -> bool: return a["weight"] < b["weight"])

    # 2. Kruskal's / Prim's Algorithm for Minimum Spanning Tree (MST)
    var parent: Array[int] = []
    parent.resize(_rooms.size())
    for i in range(_rooms.size()): parent[i] = i

    func find_set(v: int, p: Array[int]) -> int:
        if v == p[v]: return v
        p[v] = find_set(p[v], p)
        return p[v]

    var mst_edges: Array[Dictionary] = []
    var extra_edges: Array[Dictionary] = []

    for edge in edges:
        var root_a: int = find_set(edge["room_a"], parent)
        var root_b: int = find_set(edge["room_b"], parent)
        
        if root_a != root_b:
            parent[root_a] = root_b
            mst_edges.append(edge)
        else:
            extra_edges.append(edge)

    # 3. Inject Cyclical Loops: Re-add 15% of discarded edges to create non-linear gameplay loops
    for edge in extra_edges:
        if randf() < cyclical_loop_ratio:
            mst_edges.append(edge)

    # 4. Carve Orthogonal L-Shaped Hallways into the Grid
    for edge in mst_edges:
        var c1: Vector2i = _rooms[edge["room_a"]].get_center()
        var c2: Vector2i = _rooms[edge["room_b"]].get_center()
        _carve_l_shaped_hallway(c1, c2)

func _carve_l_shaped_hallway(start: Vector2i, end: Vector2i) -> void:
    var horizontal_first: bool = randf() > 0.5
    
    if horizontal_first:
        _carve_horizontal(start.x, end.x, start.y)
        _carve_vertical(start.y, end.y, end.x)
    else:
        _carve_vertical(start.y, end.y, start.x)
        _carve_horizontal(start.x, end.x, end.y)

func _carve_horizontal(x1: int, x2: int, y: int) -> void:
    var min_x: int = mini(x1, x2)
    var max_x: int = maxi(x1, x2)
    for x in range(min_x, max_x + 1):
        _grid[x][y] = 1 # Carve floor

func _carve_vertical(y1: int, y2: int, x: int) -> void:
    var min_y: int = mini(y1, y2)
    var max_y: int = maxi(y1, y2)
    for y in range(min_y, max_y + 1):
        _grid[x][y] = 1 # Carve floor
```

---
