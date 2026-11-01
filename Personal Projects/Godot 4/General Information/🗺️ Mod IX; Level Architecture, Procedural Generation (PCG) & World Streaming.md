# 🗺️ Module IX: Level Architecture, Procedural Generation (PCG) & World Streaming

> [!info] 📖 Lexicon & First Principles: Spatial Design & Algorithmic Synthesis
> Moving from localized player mechanics to expansive, responsive game worlds requires a rigorous understanding of architectural psychology, discrete procedural topology, and hardware memory limits:
> - ⤷ **Greyboxing (Blockmesh Prototyping)**: The rapid iterative construction of playable spaces using untextured geometric primitives, calibrating spatial pacing, jump trajectories, cover metrics, and sightlines before committing to production art assets.
> - ⤷ **Weenie (Landmark Beacon)**: An architectural term coined by Walt Disney for a towering, visually dominant vertical structure placed on the horizon, enabling players to orient themselves intuitively via spatial triangulation without relying on intrusive mini-maps or navigational UI markers.
> - ⤷ **Binary Space Partitioning (BSP)**: A geometric algorithm that recursively divides Euclidean space into convex sub-spaces using hyperplanes, generating a hierarchical binary tree for non-overlapping room distribution and visibility sorting.
> - ⤷ **Cellular Automaton**: A discrete mathematical model consisting of a regular grid of cells, each evolving through discrete time steps according to fixed deterministic rules based on the states of its local Moore or von Neumann neighbours.
> - ⤷ **Wave Function Collapse (WFC)**: A constraint-satisfaction algorithm based on quantum mechanics terminology, wherein a grid of unobserved possibilities (superpositions) collapses into deterministic modules while preserving strict socket-adjacency rules.
> - ⤷ **Shannon Entropy**: A formal information-theoretic measure ($H = -\sum p \log_2 p$) of uncertainty within a probability distribution, utilized in WFC to select the cell with the fewest remaining valid choices.
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

---

### Step-by-Step Architectural Construction: Binary Space Partitioning (BSP)

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

#### Chunk 1.1: The BSP Leaf Node & Splitting Constraints

```gdscript
class_name BSPDungeonGenerator
extends RefCounted

class BSPLeaf:
	var rect: Rect2i
	var left_child: BSPLeaf = null
	var right_child: BSPLeaf = null
	var room: Rect2i = Rect2i()

	func _init(p_rect: Rect2i) -> void:
		rect = p_rect

	func split(min_leaf_size: int) -> bool:
		if left_child != null or right_child != null:
			return false
			
		var split_horizontal: bool = randf() > 0.5
		if rect.size.x > rect.size.y and float(rect.size.x) / float(rect.size.y) >= 1.25:
			split_horizontal = false
		elif rect.size.y > rect.size.x and float(rect.size.y) / float(rect.size.x) >= 1.25:
			split_horizontal = true

		var max_split: int = (rect.size.y if split_horizontal else rect.size.x) - min_leaf_size
		if max_split <= min_leaf_size:
			return false

		var split_pos: int = randi_range(min_leaf_size, max_split)

		if split_horizontal:
			left_child = BSPLeaf.new(Rect2i(rect.position.x, rect.position.y, rect.size.x, split_pos))
			right_child = BSPLeaf.new(Rect2i(rect.position.x, rect.position.y + split_pos, rect.size.x, rect.size.y - split_pos))
		else:
			left_child = BSPLeaf.new(Rect2i(rect.position.x, rect.position.y, split_pos, rect.size.y))
			right_child = BSPLeaf.new(Rect2i(rect.position.x + split_pos, rect.position.y, rect.size.x - split_pos, rect.size.y))

		return true
```

- **Aspect Ratio Balancing (`float(rect.size.x) / float(rect.size.y) >= 1.25`)**: Prevents long, degenerate sliver leaves. If a leaf is significantly wider than it is tall, the algorithm forces a vertical split; if taller, it forces a horizontal split.
- **`max_split <= min_leaf_size`**: Bounds checking. Ensures that both resulting child partitions have enough room to contain a viable, playable room without violating the `min_leaf_size` threshold.

#### Chunk 1.2: Interior Room Carving & Grid Baking

```gdscript
	func create_rooms(min_room_size: int) -> void:
		if left_child != null or right_child != null:
			if left_child != null: left_child.create_rooms(min_room_size)
			if right_child != null: right_child.create_rooms(min_room_size)
		else:
			var room_w: int = randi_range(min_room_size, rect.size.x - 2)
			var room_h: int = randi_range(min_room_size, rect.size.y - 2)
			var room_x: int = randi_range(rect.position.x + 1, rect.position.x + rect.size.x - room_w - 1)
			var room_y: int = randi_range(rect.position.y + 1, rect.position.y + rect.size.y - room_h - 1)
			room = Rect2i(room_x, room_y, room_w, room_h)

	func get_all_rooms() -> Array[Rect2i]:
		if left_child == null and right_child == null:
			return [room] if room.size.x > 0 else []
		var result: Array[Rect2i] = []
		if left_child != null: result.append_array(left_child.get_all_rooms())
		if right_child != null: result.append_array(right_child.get_all_rooms())
		return result
```

- **`rect.position.x + 1` & `- 2` Padding**: Enforces a mandatory structural buffer along the leaf perimeter, ensuring that rooms spawned in adjacent partitions do not accidentally fuse together into an amorphous blob before corridors are carved.
- **Recursive Leaf Harvesting (`get_all_rooms()`)**: Gathers all carved leaf rooms into a flat array for downstream corridor generation and graph triangulation.

---

#### The Fully Assembled `BSPDungeonGenerator.gd`

```gdscript
# ==============================================================================
# SCRIPT: BSPDungeonGenerator.gd
# Architecture: Recursive Binary Space Partitioning Dungeon Synthesizer
# ==============================================================================
class_name BSPDungeonGenerator
extends RefCounted

class BSPLeaf:
	var rect: Rect2i
	var left_child: BSPLeaf = null
	var right_child: BSPLeaf = null
	var room: Rect2i = Rect2i()

	func _init(p_rect: Rect2i) -> void:
		rect = p_rect

	## Attempts to partition the leaf into two sub-rectangles
	func split(min_leaf_size: int) -> bool:
		if left_child != null or right_child != null:
			return false
			
		# Enforce aspect ratio balancing to prevent narrow corridors
		var split_horizontal: bool = randf() > 0.5
		if rect.size.x > rect.size.y and float(rect.size.x) / float(rect.size.y) >= 1.25:
			split_horizontal = false
		elif rect.size.y > rect.size.x and float(rect.size.y) / float(rect.size.x) >= 1.25:
			split_horizontal = true

		var max_split: int = (rect.size.y if split_horizontal else rect.size.x) - min_leaf_size
		if max_split <= min_leaf_size:
			return false

		var split_pos: int = randi_range(min_leaf_size, max_split)

		if split_horizontal:
			left_child = BSPLeaf.new(Rect2i(rect.position.x, rect.position.y, rect.size.x, split_pos))
			right_child = BSPLeaf.new(Rect2i(rect.position.x, rect.position.y + split_pos, rect.size.x, rect.size.y - split_pos))
		else:
			left_child = BSPLeaf.new(Rect2i(rect.position.x, rect.position.y, split_pos, rect.size.y))
			right_child = BSPLeaf.new(Rect2i(rect.position.x + split_pos, rect.position.y, rect.size.x - split_pos, rect.size.y))

		return true

	## Recursively carves interior rooms into terminal leaf nodes
	func create_rooms(min_room_size: int) -> void:
		if left_child != null or right_child != null:
			if left_child != null: left_child.create_rooms(min_room_size)
			if right_child != null: right_child.create_rooms(min_room_size)
		else:
			# Carve interior room with 1-tile wall padding
			var room_w: int = randi_range(min_room_size, rect.size.x - 2)
			var room_h: int = randi_range(min_room_size, rect.size.y - 2)
			var room_x: int = randi_range(rect.position.x + 1, rect.position.x + rect.size.x - room_w - 1)
			var room_y: int = randi_range(rect.position.y + 1, rect.position.y + rect.size.y - room_h - 1)
			room = Rect2i(room_x, room_y, room_w, room_h)

	## Collects all carved rooms from terminal leaves
	func get_all_rooms() -> Array[Rect2i]:
		if left_child == null and right_child == null:
			return [room] if room.size.x > 0 else []
		var result: Array[Rect2i] = []
		if left_child != null: result.append_array(left_child.get_all_rooms())
		if right_child != null: result.append_array(right_child.get_all_rooms())
		return result

## Generates a 2D grid dictionary containing the carved layout and harvested rooms
static func generate_dungeon(width: int, height: int, min_leaf_size: int = 12) -> Dictionary:
	var root: BSPLeaf = BSPLeaf.new(Rect2i(0, 0, width, height))
	var leaves: Array[BSPLeaf] = [root]
	
	var did_split: bool = true
	while did_split:
		did_split = false
		for leaf in leaves.duplicate():
			if leaf.left_child == null and leaf.right_child == null:
				if leaf.split(min_leaf_size):
					leaves.append(leaf.left_child)
					leaves.append(leaf.right_child)
					did_split = true

	root.create_rooms(maxi(4, min_leaf_size / 2))
	var extracted_rooms: Array[Rect2i] = root.get_all_rooms()
	
	# Allocate 2D integer grid: 0 = Void, 1 = Floor
	var grid: Array = []
	grid.resize(width)
	for x in range(width):
		var col: Array = []
		col.resize(height)
		col.fill(0)
		grid[x] = col

	# Rasterize rooms into the grid
	for r in extracted_rooms:
		for rx in range(r.position.x, r.end.x):
			for ry in range(r.position.y, r.end.y):
				grid[rx][ry] = 1
				
	return {
		"grid": grid,
		"rooms": extracted_rooms
	}
```

---

### Step-by-Step Architectural Construction: Cellular Automata Caverns

To generate organic cavern systems, we model rock erosion using a discrete 2D Cellular Automaton governed by the **Moore Neighborhood** ($8$ adjacent surrounding cells):

```
                        MOORE NEIGHBORHOOD (Radius = 1)
=================================================================================
                         [x-1, y-1]  [ x, y-1 ]  [x+1, y-1]
                         [x-1,  y ]  [ Cell   ]  [x+1,  y ]
                         [x-1, y+1]  [ x, y+1 ]  [x+1, y+1]
```

#### Chunk 2.1: Stochastic Seeding & Border Clamping

```gdscript
class_name CellularCaveGenerator
extends RefCounted

static func generate_caves(width: int, height: int, fill_ratio: float = 0.45, iterations: int = 5) -> Array:
	var grid: Array = []
	grid.resize(width)
	
	for x in range(width):
		var col: Array = []
		col.resize(height)
		for y in range(height):
			if x == 0 or x == width - 1 or y == 0 or y == height - 1:
				col[y] = 1
			else:
				col[y] = 1 if randf() < fill_ratio else 0
		grid[x] = col
```

- **Perimeter Invariant (`x == 0 or x == width - 1 ...`)**: Forcing outer boundary cells to solid wall ($1$) guarantees that subsequent smoothing passes cannot erode holes in the edge of the world, preventing players and physics objects from falling into infinite void space.
- **`fill_ratio: float = 0.45`**: Initial thermodynamic density. At $45\%$ initial wall density, Moore-neighborhood erosion reliably converges into open, winding limestone caverns. Values $> 52\%$ collapse into isolated, impassable pockets.

#### Chunk 2.2: The Moore Neighborhood Filter & Double Buffering

```gdscript
	for _step in range(iterations):
		var buffer: Array = []
		buffer.resize(width)
		for x in range(width):
			var col: Array = []
			col.resize(height)
			for y in range(height):
				var wall_count: int = _count_moore_walls(grid, x, y, width, height)
				if wall_count > 4:
					col[y] = 1
				elif wall_count < 4:
					col[y] = 0
				else:
					col[y] = grid[x][y]
			buffer[x] = col
		grid = buffer

	return grid
```

- **Double-Buffering (`var buffer = ...; grid = buffer`)**: **Critical Simulation Invariant**. If you mutate `grid[x][y]` in-place while looping through cells, subsequent cell evaluations read the *already updated* states from earlier in the same frame, introducing severe diagonal directional bias. Writing updates to an isolated secondary buffer and swapping arrays at the end of the pass guarantees isotropic, uniform erosion.
- **The B5678/S45678 Rule**: A cell with $\ge 5$ wall neighbors solidifies into a wall; $\le 3$ erodes into empty air; $4$ preserves its current state.

---

#### The Fully Assembled `CellularCaveGenerator.gd`

```gdscript
# ==============================================================================
# SCRIPT: CellularCaveGenerator.gd
# Architecture: Double-Buffered Cellular Automata Cavern Synthesizer
# ==============================================================================
class_name CellularCaveGenerator
extends RefCounted

## Synthesizes organic caves via iterative Moore-neighborhood smoothing
static func generate_caves(width: int, height: int, fill_ratio: float = 0.45, iterations: int = 5) -> Array:
	var grid: Array = []
	grid.resize(width)
	
	# 1. Stochastic Seeding with solid perimeter enforcement
	for x in range(width):
		var col: Array = []
		col.resize(height)
		for y in range(height):
			if x == 0 or x == width - 1 or y == 0 or y == height - 1:
				col[y] = 1
			else:
				col[y] = 1 if randf() < fill_ratio else 0
		grid[x] = col

	# 2. Double-Buffered Smoothing Passes
	for _step in range(iterations):
		var buffer: Array = []
		buffer.resize(width)
		for x in range(width):
			var col: Array = []
			col.resize(height)
			for y in range(height):
				var wall_count: int = _count_moore_walls(grid, x, y, width, height)
				
				# B5678 / S45678 Transition Rule
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
			# Out-of-bounds coordinates evaluate as solid rock
			if nx < 0 or nx >= max_w or ny < 0 or ny >= max_h:
				walls += 1
			elif map[nx][ny] == 1:
				walls += 1
	return walls
```

---

### Step-by-Step Architectural Construction: Wave Function Collapse (WFC)

Wave Function Collapse (WFC) treats each grid coordinate as a quantum-like **Superposition** of all possible structural tiles. It collapses the cell with the lowest Shannon entropy, then propagates socket-matching constraints outward across adjacent cells.

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

#### Chunk 3.1: Tile Sockets & Superposition Wave Grid

```gdscript
class_name WaveFunctionCollapse2D
extends RefCounted

class WFCTile:
	var id: int
	var sockets: Array[String] # [North, East, South, West]
	var weight: float = 1.0
	
	func _init(p_id: int, p_sockets: Array[String], p_weight: float = 1.0) -> void:
		id = p_id
		sockets = p_sockets
		weight = p_weight

var _tile_palette: Array[WFCTile] = []
var _grid_width: int
var _grid_height: int
var _wave: Array = [] # 2D Array of Array[WFCTile]
```

- **`sockets: Array[String]`**: Sockets define boundary compatibility along four directional interfaces: `[North (0), East (1), South (2), West (3)]`. For example, a road tile might declare sockets `["Road", "Grass", "Road", "Grass"]`.
- **`_wave: Array`**: The superposition grid. At initialization, every coordinate contains an array containing *all possible tiles* in the palette.

#### Chunk 3.2: Entropy Minimization & Weighted Collapse

```gdscript
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
```

- **Lowest Entropy Selection**: Scans the grid for uncollapsed cells (`count > 1`) with the fewest remaining valid possibilities. Collapsing the most constrained cell first dramatically minimizes the probability of hitting an unresolvable contradiction later in generation.
- **Weighted Sampling**: Selects a tile using normalized weights, allowing common tiles (like grass) to appear frequently while preserving rare structures (like shrines or intersections).

#### Chunk 3.3: Constraint Propagation Stack

```gdscript
func _propagate_constraints(start_pos: Vector2i) -> void:
	var stack: Array[Vector2i] = [start_pos]
	var directions: Array[Vector2i] = [Vector2i(0, -1), Vector2i(1, 0), Vector2i(0, 1), Vector2i(-1, 0)]
	
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
				continue

			var opposing_dir: int = (dir_idx + 2) % 4
			var valid_neighbor_tiles: Array = []

			for candidate in neighbor_options:
				var is_compatible: bool = false
				for source in current_options:
					if source.sockets[dir_idx] == candidate.sockets[opposing_dir]:
						is_compatible = true
						break
				if is_compatible:
					valid_neighbor_tiles.append(candidate)

			if valid_neighbor_tiles.size() < neighbor_options.size():
				_wave[neighbor.x][neighbor.y] = valid_neighbor_tiles
				stack.append(neighbor)
```

- **`opposing_dir: int = (dir_idx + 2) % 4`**: Evaluates the shared directional interface. The North socket (index 0) of the source cell must match the South socket (index 2) of the neighbor above it; East (1) matches West (3).
- **Stack-Based Wave Front**: When a neighbor's superposition is pruned (`valid_neighbor_tiles.size() < neighbor_options.size()`), that neighbor is pushed onto the stack. Its own neighbors must now be evaluated, propagating the constraint outward across the level until equilibrium is reached.

---

#### The Fully Assembled `WaveFunctionCollapse2D.gd`

```gdscript
# ==============================================================================
# SCRIPT: WaveFunctionCollapse2D.gd
# Architecture: Constraint Propagation Wave Function Collapse Synthesizer
# ==============================================================================
class_name WaveFunctionCollapse2D
extends RefCounted

class WFCTile:
	var id: int
	var sockets: Array[String] # [North, East, South, West]
	var weight: float = 1.0
	
	func _init(p_id: int, p_sockets: Array[String], p_weight: float = 1.0) -> void:
		id = p_id
		sockets = p_sockets
		weight = p_weight

var _tile_palette: Array[WFCTile] = []
var _grid_width: int
var _grid_height: int
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
			col[y] = _tile_palette.duplicate()
		_wave[x] = col

## Executes the iterative collapse loop; returns false on contradiction
func run_collapse() -> bool:
	while true:
		var candidate_cell: Vector2i = _find_lowest_entropy_cell()
		if candidate_cell == Vector2i(-1, -1):
			return true # All cells successfully collapsed to single states

		var cell_options: Array = _wave[candidate_cell.x][candidate_cell.y]
		if cell_options.is_empty():
			push_error("WFC Contradiction: Zero valid choices remain at %s" % str(candidate_cell))
			return false

		# Collapse target coordinate
		var chosen_tile: WFCTile = _sample_weighted_tile(cell_options)
		_wave[candidate_cell.x][candidate_cell.y] = [chosen_tile]

		# Propagate adjacency rules
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
	var directions: Array[Vector2i] = [Vector2i(0, -1), Vector2i(1, 0), Vector2i(0, 1), Vector2i(-1, 0)]
	
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
				continue

			var opposing_dir: int = (dir_idx + 2) % 4
			var valid_neighbor_tiles: Array = []

			for candidate in neighbor_options:
				var is_compatible: bool = false
				for source in current_options:
					if source.sockets[dir_idx] == candidate.sockets[opposing_dir]:
						is_compatible = true
						break
				if is_compatible:
					valid_neighbor_tiles.append(candidate)

			# If choices were eliminated, push neighbor onto propagation stack
			if valid_neighbor_tiles.size() < neighbor_options.size():
				_wave[neighbor.x][neighbor.y] = valid_neighbor_tiles
				stack.append(neighbor)
```

---

### Step-by-Step Architectural Construction: Continuous Multi-Octave fBm & Whittaker Biomes

Continuous open worlds rely on **Fractional Brownian Motion (fBm)**, evaluating multi-octave coherent noise for elevation and moisture:

$$H(x, z) = \sum_{k=0}^{M-1} g^k \cdot \text{Noise}\left(\ell^k \cdot x, \; \ell^k \cdot z\right)$$

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

#### Chunk 4.1: Octave Configuration & Noise Sampling

```gdscript
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
```

- **`fractal_octaves = 5`**: Layers 5 progressive octaves of noise. Octave 0 generates continental mountain masses; Octave 4 generates small rocky surface bumps.
- **`fractal_lacunarity = 2.0` & `fractal_gain = 0.5`**: Each subsequent octave doubles in frequency ($2\times$) while halving in amplitude ($0.5\times$).
- **Seed De-correlation (`seed + 101`)**: Sampling elevation and moisture with identical seeds causes mountains to always align with rivers. Offsetting the seed ensures elevation and moisture vary independently.

#### Chunk 4.2: Biome Classification Matrix

```gdscript
func sample_world(x: float, z: float) -> Dictionary:
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
		"elevation": elevation * 120.0,
		"moisture": moisture,
		"biome": biome
	}
```

- **Normalized Mapping (`+ 1.0) * 0.5`**: Maps raw Simplex output $[-1.0, 1.0]$ into a clean $[0.0, 1.0]$ range.
- **Bi-Variate Whittaker Lookup**: Uses elevation and moisture coordinates to assign categorical gameplay rules, foliage sets, and ground shader materials.

---

#### The Fully Assembled `BiomeSynthesizer.gd`

```gdscript
# ==============================================================================
# SCRIPT: BiomeSynthesizer.gd
# Architecture: Multi-Octave fBm Elevation & Whittaker Biome Matrix
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

## Returns continuous height and categorical biome type at world coordinate (x, z)
func sample_world(x: float, z: float) -> Dictionary:
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
		"elevation": elevation * 120.0,
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

Loading heavy `.tscn` chunks synchronously via `load()` blocks the main thread, dropping frames and causing noticeable stutter. Godot 4 resolves this via **`ResourceLoader.load_threaded_request()`**.

---

### Step-by-Step Architectural Construction: Threaded Chunk Streamer

#### Chunk 5.1: Threaded Loading Request & Polling

```gdscript
class_name ChunkStreamer
extends Node

signal chunk_loaded(coord: Vector2i, scene: PackedScene)

var _loading_queue: Dictionary = {}

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

- **`ResourceLoader.load_threaded_request(...)`**: Offloads disk read and scene deserialization to Godot's internal worker thread pool, keeping the main thread running at full frame rate.
- **`THREAD_LOAD_LOADED`**: Polls non-blocking progress. Once loading completes, it retrieves the `PackedScene` and signals the scene spawner to mount it into the active tree.

---

### Step-by-Step Architectural Construction: Floating Origin Shifter

Modern GPUs and physics engines run on **32-bit single-precision floating-point numbers** (IEEE 754), reserving only 23 bits of mantissa precision. Beyond $8,192\text{ metres}$, precision drops below $1\text{ mm}$, causing visible mesh vibrating and collision pass-through bugs.

#### Chunk 6.1: Planar Distance Thresholding & Scene Shifting

```gdscript
class_name FloatingOriginManager
extends Node

signal origin_shifted(offset_vector: Vector3)

@export var player_target: Node3D
@export var shift_threshold_meters: float = 4000.0

func _physics_process(_delta: float) -> void:
	if player_target == null:
		return

	var player_pos: Vector3 = player_target.global_position
	if Vector2(player_pos.x, player_pos.z).length_squared() > (shift_threshold_meters * shift_threshold_meters):
		_execute_origin_shift(player_pos)

func _execute_origin_shift(shift_vector: Vector3) -> void:
	player_target.global_position -= shift_vector

	var world_nodes: Array[Node] = get_tree().get_nodes_in_group(&"floating_origin_affected")
	for entity in world_nodes:
		if entity is Node3D and entity != player_target:
			(entity as Node3D).global_position -= shift_vector

	origin_shifted.emit(shift_vector)
```

- **`shift_threshold_meters: float = 4000.0`**: Triggers origin shifts well before floating-point jitter becomes perceptible ($8,192\text{ m}$).
- **`global_position -= shift_vector`**: Offsets the player and all active world entities backward by $-\vec{P}_{\text{player}}$, snapping the player back to $[0, 0, 0]$ and preserving maximum floating-point precision indefinitely.

---

#### The Fully Assembled `FloatingOriginManager.gd`

```gdscript
# ==============================================================================
# SCRIPT: FloatingOriginManager.gd
# Architecture: Large-World IEEE 754 Floating Origin Coordinator
# ==============================================================================
class_name FloatingOriginManager
extends Node

## Dispatched when world coordinates are reset, notifying physics queries and VFX
signal origin_shifted(offset_vector: Vector3)

@export_group("Target Tracking")
## The focal player node whose distance from origin drives the shift
@export var player_target: Node3D
## Maximum distance from (0,0,0) permitted before triggering a shift (metres)
@export var shift_threshold_meters: float = 4000.0

func _physics_process(_delta: float) -> void:
	if player_target == null:
		return

	var player_pos: Vector3 = player_target.global_position
	# Check horizontal distance against threshold squared
	if Vector2(player_pos.x, player_pos.z).length_squared() > (shift_threshold_meters * shift_threshold_meters):
		_execute_origin_shift(player_pos)

## Shifts the player and registered entities back toward origin
func _execute_origin_shift(shift_vector: Vector3) -> void:
	# 1. Reset player position
	player_target.global_position -= shift_vector

	# 2. Shift all active world entities
	var world_nodes: Array[Node] = get_tree().get_nodes_in_group(&"floating_origin_affected")
	for entity in world_nodes:
		if entity is Node3D and entity != player_target:
			(entity as Node3D).global_position -= shift_vector

	# 3. Notify listeners
	origin_shifted.emit(shift_vector)
	print("FloatingOriginManager: World origin shifted by %s. Precision stabilized." % str(shift_vector))
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

---

### Step-by-Step Architectural Construction: Procedural Dungeon Pipeline

Let us deconstruct our graph compilation, MST pruning, cyclical edge injection, and orthogonal corridor rasterization line by line.

#### Chunk 7.1: Euclidean Distance Edge Matrix

```gdscript
class_name ProceduralDungeonPipeline
extends Node

@export var grid_width: int = 80
@export var grid_height: int = 80
@export var min_leaf_size: int = 14
@export_range(0.0, 1.0) var cyclical_loop_ratio: float = 0.15

var _grid: Array = []
var _rooms: Array[Rect2i] = []

func _build_distance_graph() -> Array[Dictionary]:
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

	edges.sort_custom(func(a: Dictionary, b: Dictionary) -> bool: return a["weight"] < b["weight"])
	return edges
```

- **All-Pairs Edge Construction**: Builds a complete graph connecting every room centroid to every other room centroid.
- **`edges.sort_custom(...)`**: Sorts edges ascending by squared distance, establishing the greedy selection order required by Kruskal's algorithm.

#### Chunk 7.2: Kruskal's Minimum Spanning Tree (MST) & Cycle Loop Injection

```gdscript
func _solve_mst_with_loops(edges: Array[Dictionary]) -> Array[Dictionary]:
	var parent: Array[int] = []
	parent.resize(_rooms.size())
	for i in range(_rooms.size()): parent[i] = i

	var find_set = func(v: int, p_arr: Array[int], self_ref: Callable) -> int:
		if v == p_arr[v]: return v
		p_arr[v] = self_ref.call(p_arr[v], p_arr, self_ref)
		return p_arr[v]

	var mst_edges: Array[Dictionary] = []
	var discarded_edges: Array[Dictionary] = []

	for edge in edges:
		var root_a: int = find_set.call(edge["room_a"], parent, find_set)
		var root_b: int = find_set.call(edge["room_b"], parent, find_set)
		
		if root_a != root_b:
			parent[root_a] = root_b
			mst_edges.append(edge)
		else:
			discarded_edges.append(edge)

	# Inject cyclical loops
	for edge in discarded_edges:
		if randf() < cyclical_loop_ratio:
			mst_edges.append(edge)

	return mst_edges
```

- **Union-Find / Disjoint-Set Forest (`find_set`)**: Detects cycles in near-$\mathcal{O}(1)$ time with path compression. If two rooms already share a common root parent, adding the edge would form a closed loop.
- **Cyclical Edge Injection (`randf() < cyclical_loop_ratio`)**: **The Non-Linear Pacing Invariant**. A pure Minimum Spanning Tree forms a single branching tree with zero loops. If the player reaches a dead end, they are forced to backtrack along the exact path they walked. Injecting $15\%$ of discarded edges creates circular loops, allowing dynamic flanking and natural dungeon navigation.

#### Chunk 7.3: Orthogonal L-Shaped Corridor Rasterization

```gdscript
func _carve_corridors(mst_edges: Array[Dictionary]) -> void:
	for edge in mst_edges:
		var c1: Vector2i = _rooms[edge["room_a"]].get_center()
		var c2: Vector2i = _rooms[edge["room_b"]].get_center()
		
		if randf() > 0.5:
			_carve_horizontal(c1.x, c2.x, c1.y)
			_carve_vertical(c1.y, c2.y, c2.x)
		else:
			_carve_vertical(c1.y, c2.y, c1.x)
			_carve_horizontal(c1.x, c2.x, c2.y)

func _carve_horizontal(x1: int, x2: int, y: int) -> void:
	for x in range(mini(x1, x2), maxi(x1, x2) + 1):
		_grid[x][y] = 1

func _carve_vertical(y1: int, y2: int, x: int) -> void:
	for y in range(mini(y1, y2), maxi(y1, y2) + 1):
		_grid[x][y] = 1
```

- **Randomized L-Bend Orientation (`randf() > 0.5`)**: Alternates between horizontal-first and vertical-first dog-leg corridors, producing varied architectural layouts rather than repetitive parallel hallways.

---

#### The Fully Assembled `ProceduralDungeonPipeline.gd`

```gdscript
# ==============================================================================
# SCRIPT: ProceduralDungeonPipeline.gd
# Architecture: Non-Linear Graph-Based Procedural Dungeon Carver (MST + Loops)
# ==============================================================================
class_name ProceduralDungeonPipeline
extends Node

@export_group("Grid Constraints")
## Width of the discrete generation grid in cells
@export var grid_width: int = 80
## Height of the discrete generation grid in cells
@export var grid_height: int = 80
## Minimum dimension threshold for recursive BSP leaf splitting
@export var min_leaf_size: int = 14

@export_group("Topology")
## Percentage of non-spanning edges re-injected to form cyclical gameplay loops
@export_range(0.0, 1.0) var cyclical_loop_ratio: float = 0.15

var _grid: Array = []
var _rooms: Array[Rect2i] = []

## Builds a complete non-linear procedural floor plan
func build_dungeon() -> Dictionary:
	# 1. Subdivide space and generate rooms via BSP
	var bsp_data: Dictionary = BSPDungeonGenerator.generate_dungeon(grid_width, grid_height, min_leaf_size)
	_grid = bsp_data["grid"]
	_rooms = bsp_data["rooms"]

	if _rooms.size() < 2:
		return {"grid": _grid, "rooms": _rooms}

	# 2. Build sorted Euclidean distance graph
	var edges: Array[Dictionary] = _build_distance_graph()

	# 3. Solve Minimum Spanning Tree with cyclical loop injection
	var final_edges: Array[Dictionary] = _solve_mst_with_loops(edges)

	# 4. Rasterize orthogonal corridors into the grid
	_carve_corridors(final_edges)

	return {
		"grid": _grid,
		"rooms": _rooms
	}

func _build_distance_graph() -> Array[Dictionary]:
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

	edges.sort_custom(func(a: Dictionary, b: Dictionary) -> bool: return a["weight"] < b["weight"])
	return edges

func _solve_mst_with_loops(edges: Array[Dictionary]) -> Array[Dictionary]:
	var parent: Array[int] = []
	parent.resize(_rooms.size())
	for i in range(_rooms.size()): parent[i] = i

	var find_set = func(v: int, p_arr: Array[int], self_ref: Callable) -> int:
		if v == p_arr[v]: return v
		p_arr[v] = self_ref.call(p_arr[v], p_arr, self_ref)
		return p_arr[v]

	var mst_edges: Array[Dictionary] = []
	var discarded_edges: Array[Dictionary] = []

	for edge in edges:
		var root_a: int = find_set.call(edge["room_a"], parent, find_set)
		var root_b: int = find_set.call(edge["room_b"], parent, find_set)
		
		if root_a != root_b:
			parent[root_a] = root_b
			mst_edges.append(edge)
		else:
			discarded_edges.append(edge)

	# Re-add discarded edges to create loops
	for edge in discarded_edges:
		if randf() < cyclical_loop_ratio:
			mst_edges.append(edge)

	return mst_edges

func _carve_corridors(mst_edges: Array[Dictionary]) -> void:
	for edge in mst_edges:
		var c1: Vector2i = _rooms[edge["room_a"]].get_center()
		var c2: Vector2i = _rooms[edge["room_b"]].get_center()
		
		if randf() > 0.5:
			_carve_horizontal(c1.x, c2.x, c1.y)
			_carve_vertical(c1.y, c2.y, c2.x)
		else:
			_carve_vertical(c1.y, c2.y, c1.x)
			_carve_horizontal(c1.x, c2.x, c2.y)

func _carve_horizontal(x1: int, x2: int, y: int) -> void:
	for x in range(mini(x1, x2), maxi(x1, x2) + 1):
		_grid[x][y] = 1

func _carve_vertical(y1: int, y2: int, x: int) -> void:
	for y in range(mini(y1, y2), maxi(y1, y2) + 1):
		_grid[x][y] = 1
```

---

