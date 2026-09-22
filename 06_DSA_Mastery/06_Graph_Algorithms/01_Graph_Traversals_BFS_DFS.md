# Domain 06: Graph Algorithms — Representations & BFS/DFS Traversals

---

## 1. Graph Representation: Adjacency List

A graph $G = (V, E)$ consists of vertices $V$ and edges $E$. In interview settings, **Adjacency Lists** are preferred over Adjacency Matrices because they consume $O(V + E)$ space rather than $O(V^2)$ and allow iterating only over existing neighbors.

```python
from collections import defaultdict

# 1. Undirected Graph
adj = defaultdict(list)
for u, v in edges:
    adj[u].append(v)
    adj[v].append(u)

# 2. Directed Weighted Graph
adj_weighted = defaultdict(list)
for u, v, weight in edges:
    adj_weighted[u].append((v, weight))
```

---

## 2. BFS vs. DFS Invariant Selection

| Algorithm | Data Structure | Invariant | Best Used For |
| :--- | :--- | :--- | :--- |
| **BFS (Breadth-First Search)** | Queue (FIFO) | Explores nodes in strictly non-decreasing order of edge distance from source. | **Shortest path on unweighted graphs**, level-order expansion. |
| **DFS (Depth-First Search)** | Recursion / Stack (LIFO) | Explores as deep as possible along each branch before backtracking. | **Cycle detection**, connected components, path finding, maze solving. |

---

## 3. Benchmark Problem Deep Dive: LeetCode 200 — Number of Islands

### 3.1 Problem Statement Breakdown & Line-by-Line Annotations
> *"Given an `m x n` 2D binary grid `grid` which represents a map of `'1'`s (land) and `'0'`s (water),"*
- 2D matrix where cells are `'1'` or `'0'`.

> *"return the number of islands."*
- An island is surrounded by water and is formed by connecting adjacent lands horizontally or vertically (4-directional connectivity).
- Mathematical definition: Find the number of **Connected Components** in an undirected grid graph.

---

### 3.2 Constraints & Complexity Analysis
- $m = \text{grid.length}, n = \text{grid}[i]\text{.length}$
- $1 \le m, n \le 300 \implies$ Total cells $V = m \times n \le 90,000$.
- Each cell has at most 4 edges $\implies E \le 4V$.
- Target Time: $O(m \times n)$ (each cell visited a constant number of times).
- Space Complexity: $O(m \times n)$ worst-case call stack or queue (when entire grid is land).

---

### 3.3 Direction Exploration & Invariant Proof
- **Connected Component Exploration**:
  - Scan the grid cell by cell with nested loops $(r, c)$.
  - When a cell with `'1'` is found, increment our island counter by 1.
  - Launch a BFS or DFS from $(r, c)$ to "sink" the entire island (mutating `'1'` to `'0'` or marking `visited`).
  - By the time the traversal completes, all contiguous land cells connected to $(r, c)$ are marked as visited, preventing them from being counted again.

---

### 3.4 Complete Python Implementation (DFS In-Place)

```python
class Solution:
    def numIslands(self, grid: list[list[str]]) -> int:
        if not grid:
            return 0
            
        rows, cols = len(grid), len(grid[0])
        island_count = 0
        
        def dfs(r: int, c: int) -> None:
            # Boundary check and water check
            if r < 0 or r >= rows or c < 0 or c >= cols or grid[r][c] != '1':
                return
                
            # Sink the land in-place to prevent revisit (O(1) memory overhead)
            grid[r][c] = '0'
            
            # Explore 4 orthogonal directions
            dfs(r + 1, c)
            dfs(r - 1, c)
            dfs(r, c + 1)
            dfs(r, c - 1)
            
        for r in range(rows):
            for c in range(cols):
                if grid[r][c] == '1':
                    island_count += 1
                    dfs(r, c)  # Sinks the entire connected component
                    
        return island_count
```

- **Time Complexity**: $O(m \times n)$ — Every cell is visited at most 5 times (once in the main loop, 4 times from neighbors).
- **Space Complexity**: $O(m \times n)$ — Recursive call stack depth in the worst case (grid filled with `'1'`).

---

### 3.5 Live Verbalization Script

> *"This problem is equivalent to counting the number of connected components in an undirected grid graph where vertices are land cells and edges connect horizontally and vertically adjacent cells.
> 
> I iterate through every cell `(r, c)`. When I encounter an unvisited land cell `'1'`, I increment my island counter and immediately initiate a Depth-First Search.
> 
> The DFS visits all 4-directionally reachable land cells, setting their value in-place to `'0'`. This 'sinks' the island, ensuring that subsequent grid iterations do not recount cells belonging to this component.
> 
> Once the DFS completes, we resume scanning for the next unvisited land cell. This visits each cell in $O(m \times n)$ time with $O(m \times n)$ stack space."*
