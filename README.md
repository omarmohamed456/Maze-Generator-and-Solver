# Maze Generator and Solver

A Python program that generates a maze using **Kruskal's algorithm** and solves it using three different search algorithms: **BFS**, **DFS**, and **A\***. The main purpose is to compare their performance, especially speed. The project includes both a **GUI version** (Pygame + Tkinter) and a **CLI version** (terminal with ANSI visualization).

---

## Table of Contents

1. [Overview](#overview)
2. [Features](#features)
3. [How It Works](#how-it-works)
   - [Maze Generation (Kruskal's Algorithm)](#1-maze-generation-kruskals-algorithm)
   - [Maze Solving Algorithms](#2-maze-solving-algorithms)
     - [BFS](#bfs-breadth-first-search)
     - [DFS](#dfs-depth-first-search)
     - [A\*](#a-a-star)
   - [Visualization](#3-visualization)
   - [GUI](#4-gui)
4. [Conclusion](#5-conclusion)

---

## Overview

This project generates a **perfect maze** — a maze with exactly one path between any two points (no cycles). It then solves it using three algorithms and lets you compare how each one performs in terms of:

- **Path length**
- **Total steps explored**
- **Time taken**

---

## Features

- Maze generation using Kruskal's algorithm
- Three solving algorithms: BFS, DFS, A*
- Step-by-step visual search animation
- Reproducible mazes via a seed input
- Terminal (CLI) and GUI versions
- Side-by-side results comparison

---

## How It Works

### 1. Maze Generation (Kruskal's Algorithm)

The maze is represented as a 2D grid where cells are connected by removing walls between them. Kruskal's algorithm ensures no cycles are created, producing a perfect maze.

**Core rule:** *Only remove a wall if it connects two different sets.*

#### Key Concepts

**Bit Flags** — Walls are represented using directional bit flags:

```
N = 1  # 0001 → North (top wall)
S = 2  # 0010 → South (bottom wall)
E = 4  # 0100 → East  (right wall)
W = 8  # 1000 → West  (left wall)
```

The bitwise OR (`|`) operator opens walls (e.g., `N | E = 0101` means top and right walls are open). The bitwise AND (`&`) operator checks if a wall is open (e.g., `cell & W` checks if the left wall is open).

**The `Tree` Class** — The core of Kruskal's algorithm, responsible for tracking connected sets and avoiding cycles:

| Method | Description |
|---|---|
| `__init__()` | Initializes the node with no parent |
| `root()` | Traces to the root to identify which set/tree a cell belongs to |
| `connected(tree)` | Checks if two cells are in the same set (used to detect cycles) |
| `connect(tree)` | Joins two cells into the same set (union operation) |

**Grid Initialization:**

```python
grid = [[0 for _ in range(width)] for _ in range(height)]   # Visual/logical grid
sets = [[Tree() for _ in range(width)] for _ in range(height)]  # Union-find structure
```

**Entrance and Exit** — A random cell on the top row has its North wall removed (entrance), and a random cell on the bottom row has its South wall removed (exit).

**Kruskal's Loop** — The algorithm shuffles all edges (walls between cells) and iterates through them. For each edge, if the two cells it connects belong to different sets, the wall is removed and the sets are merged:

```python
grid[y][x] |= direction           # Open wall on current cell
grid[ny][nx] |= OPPOSITE[direction]  # Open corresponding wall on neighbor
set1.connect(set2)                # Merge the two sets
```

**Seed Support** — Users can input a seed value that is passed to `random.seed(seed)`, making maze generation fully reproducible.

---

### 2. Maze Solving Algorithms

All three algorithms return: the **path**, a **found** boolean, **path length**, **total steps explored**, and **time taken**.

#### BFS (Breadth-First Search)

**How it works:** BFS explores the maze layer by layer, visiting all cells at the same distance from the start before moving further. It guarantees the **shortest path**.

**Implementation highlights:**
- Uses a `deque` (double-ended queue) for O(1) enqueue/dequeue
- A `visited` set prevents revisiting cells
- A `parent` dict tracks the path (key = neighbor, value = current cell)
- A timer measures search duration

#### DFS (Depth-First Search)

**How it works:** DFS dives deep into the maze along one path until it hits a dead end, then backtracks to the most recent unexplored decision point and tries a different direction.

**Implementation highlights:**
- Same structure as BFS, but uses a **stack** (`list` with `pop()`) instead of a deque
- Does **not** guarantee the shortest path

#### A* (A-Star)

**How it works:** A* selects which cell to explore next based on `f(n) = g(n) + h(n)`, where:
- `g(n)` = distance already traveled from the start
- `h(n)` = heuristic estimate of remaining distance (Manhattan distance)
- `f(n)` = total estimated cost

This makes A* smarter and generally faster than BFS and DFS, as it focuses the search toward the goal.

**Manhattan Distance Heuristic** — Used because movement is restricted to 4 directions (up, down, left, right). It calculates the straight-line grid distance between two cells.

**Implementation highlights:**
- **Open list**: A `heapq` (min-heap priority queue) sorted by `f(n)` — cells with lower costs are explored first
- **Closed list (explored)**: A `set` for O(1) lookups, contains already-explored cells
- `g_score` dict stores the best known cost to reach each cell

---

### 3. Visualization

Shared helper functions (used by all algorithms) live in a separate file:

| Function | Description |
|---|---|
| `get_neighbors()` | Returns all connected (wall-removed) neighboring cells |
| `display_path()` | Highlights the final solution path |
| `display_search()` | Animates the search step-by-step |

#### Terminal (CLI) Color Coding

| Color | Meaning |
|---|---|
| Blue | Visited cells |
| Red | Current cell being explored |
| Green | Frontier (discovered but not yet explored) |
| Yellow | Final solution path |

ANSI escape codes are used to render colors and to overwrite the terminal in-place (`\033[H` moves the cursor to the top so the maze updates without scrolling).

#### GUI

Each algorithm has a GUI-specific visualizer (e.g., `bfs_visualize_gui()`) that calls `draw_visualization()` to render on the main Pygame screen. Results are shown in a separate Tkinter window (`show_bfs/dfs/a_star_results_window()`). A `show_all_results_window()` function displays all three results at once for easy comparison.

---

### 4. GUI

The GUI is built with **Pygame** (main screen, maze rendering, search animation) and **Tkinter** (input form, results windows).

**On startup**, the user is prompted via a Tkinter input window (`get_maze_parameters()`) to enter:
- Maze **width** and **height** (number of cells)
- **Seed** (for reproducibility)
- **Delay time** (animation speed)

The main Pygame window displays the generated maze with algorithm controls at the bottom. The maze is drawn by `draw_maze()`. Users can run each algorithm individually or view all results at once.

---

## 5. Conclusion

| Algorithm | Path Quality | Speed | Notes |
|---|---|---|---|
| **A\*** | Optimal (shortest) | Fastest (generally) | Best overall — uses heuristics to guide the search |
| **BFS** | Optimal (shortest) | Moderate | Explores evenly in all directions |
| **DFS** | Not optimal | Variable | May take fewer steps on some mazes but not reliable |

**A\*** is the best-performing algorithm in most cases. DFS can occasionally produce fewer explored steps than A\* on specific mazes, but A\* consistently finds the optimal path in the least time.
