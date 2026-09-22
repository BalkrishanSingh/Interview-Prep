# Domain 06: Graph Algorithms — Disjoint Set Union (Union-Find / DSU)

---

## 1. Disjoint Set Union (DSU) Mental Model

Disjoint Set Union (Union-Find) maintains a collection of disjoint (non-overlapping) dynamic sets. Each set is identified by a representative element (its **root**).

It supports two fundamental operations in nearly $O(1)$ time:
1. `find(x)`: Determine which set element $x$ belongs to by returning the representative root.
2. `union(x, y)`: Merge the set containing $x$ with the set containing $y$.

```
Initial Sets: {1}, {2}, {3}, {4}
union(1, 2)  ──>  [1] <── 2
union(3, 4)  ──>  [3] <── 4
union(2, 4)  ──>  [1] <── 2
                   │
                  [3] <── 4
```

---

## 2. Optimizations: Path Compression & Union by Rank / Size

Without optimizations, union operations can degenerate a tree into a linear linked list of depth $N$, yielding $O(N)$ per operation. With two standard optimizations, the amortized time complexity drops to $O(\alpha(N))$, where $\alpha$ is the extremely slow-growing **Inverse Ackermann function** ($\alpha(N) \le 4$ for all practical universe sizes).

### 2.1 Path Compression (in `find`)
Flattens the structure of the tree whenever `find` is executed by making every traversed node point directly to the canonical root.

```python
def find(self, x: int) -> int:
    if self.parent[x] != x:
        self.parent[x] = self.find(self.parent[x])  # Path compression
    return self.parent[x]
```

### 2.2 Union by Rank / Size (in `union`)
Always attaches the smaller tree (lower rank or fewer elements) under the root of the larger tree. This keeps the tree height strictly logarithmic: $O(\log N)$.

```python
def union(self, x: int, y: int) -> bool:
    root_x, root_y = self.find(x), self.find(y)
    if root_x == root_y:
        return False  # Already in the same set (Cycle detected!)
        
    if self.rank[root_x] < self.rank[root_y]:
        self.parent[root_x] = root_y
    elif self.rank[root_x] > self.rank[root_y]:
        self.parent[root_y] = root_x
    else:
        self.parent[root_y] = root_x
        self.rank[root_x] += 1
    return True
```

---

## 3. Benchmark Problem Deep Dive: LeetCode 684 — Redundant Connection

### 3.1 Problem Statement Breakdown & Line-by-Line Annotations
> *"In this problem, a tree is an undirected graph that is connected and has no cycles."*
- A tree of $N$ vertices contains exactly $N - 1$ edges and is acyclic.

> *"You are given a graph that started as a tree with `n` nodes labeled from `1` to `n`, with one additional edge added."*
- The given graph has $n$ vertices and exactly $n$ edges $\implies$ exactly one cycle exists.

> *"The added edge has two different vertices chosen from `1` to `n`, and was not an edge that already existed."*
- Exactly one redundant edge creates the cycle.

> *"Return an edge that can be removed so that the resulting graph is a tree of `n` nodes. If there are multiple answers, return the answer that occurs last in the input."*
- Process edges in order. The first edge $(u, v)$ whose endpoints already share the same root in the DSU is the redundant edge that creates the cycle.

---

### 3.2 Constraints & Complexity Analysis
- $n = \text{edges.length}$
- $3 \le n \le 1,000$
- Target Time: $O(N \cdot \alpha(N)) \approx O(N)$ linear time.
- Target Space: $O(N)$ for `parent` and `rank` arrays.

---

### 3.3 Dry Run Trace Table

Consider `edges = [[1, 2], [1, 3], [2, 3]]`:
- Initially: `parent = [0, 1, 2, 3]`, `rank = [0, 0, 0, 0]`.

| Edge `[u, v]` | `find(u)` | `find(v)` | `find(u) == find(v)`? | Action / DSU State |
| :--- | :--- | :--- | :--- | :--- |
| `[1, 2]` | `1` | `2` | No | Merge: `parent[2] = 1`, `rank[1] = 1` |
| `[1, 3]` | `1` | `3` | No | Merge: `parent[3] = 1`, `rank[1] = 1` |
| `[2, 3]` | `find(2) = 1` | `find(3) = 1` | **YES** | **Cycle detected! Return `[2, 3]`** |

---

### 3.4 Complete Python Implementation

```python
class UnionFind:
    def __init__(self, size: int):
        # 1-indexed initialization
        self.parent = list(range(size + 1))
        self.rank = [0] * (size + 1)
        
    def find(self, x: int) -> int:
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])  # Path compression
        return self.parent[x]
        
    def union(self, x: int, y: int) -> bool:
        root_x = self.find(x)
        root_y = self.find(y)
        
        # If roots are already identical, an edge between x and y creates a cycle
        if root_x == root_y:
            return False
            
        # Union by rank
        if self.rank[root_x] < self.rank[root_y]:
            self.parent[root_x] = root_y
        elif self.rank[root_x] > self.rank[root_y]:
            self.parent[root_y] = root_x
        else:
            self.parent[root_y] = root_x
            self.rank[root_x] += 1
            
        return True


class Solution:
    def findRedundantConnection(self, edges: list[list[int]]) -> list[int]:
        n = len(edges)
        dsu = UnionFind(n)
        
        for u, v in edges:
            if not dsu.union(u, v):
                return [u, v]
                
        return []
```

- **Time Complexity**: $O(N \cdot \alpha(N)) \approx O(N)$ — Each of the $N$ edges undergoes at most two `find` and one `union` operations, running in nearly constant amortized time.
- **Space Complexity**: $O(N)$ — For the parent and rank arrays storing $N + 1$ elements.

---

### 3.5 Live Verbalization Script

> *"A tree with $N$ nodes contains exactly $N - 1$ edges. The problem provides $N$ edges, meaning exactly one extra edge creates a cycle.
> 
> To find the redundant edge that closes the cycle, I use a Disjoint Set Union (Union-Find) data structure with Path Compression and Union by Rank.
> 
> I initialize each node from $1$ to $N$ as its own parent. Then, I iterate through the input edges one by one. For each edge `[u, v]`, I find the canonical roots of $u$ and $v$.
> 
> If `find(u) == find(v)`, both nodes already belong to the same connected component. Adding an edge between them creates a cycle, meaning this edge is redundant. Because the question specifies returning the edge that appears last in the input if there are multiple, the first edge during sequential traversal that connects two already-connected vertices is guaranteed to be our answer.
> 
> If the roots are distinct, I merge the two sets using `union`.
> 
> This approach runs in $O(N \cdot \alpha(N))$ time, which is effectively linear $O(N)$, and requires $O(N)$ auxiliary space."*
