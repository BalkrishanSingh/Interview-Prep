# Domain 06: Graph Algorithms — Cycle Detection & Topological Sorting

---

## 1. Cycle Detection Paradigms

Detecting cycles is a prerequisite for many graph algorithms. The cycle detection algorithm depends fundamentally on whether the graph is **undirected** or **directed**.

| Graph Type | Algorithm | Core Invariant | Time Complexity | Space Complexity |
| :--- | :--- | :--- | :--- | :--- |
| **Undirected** | DFS with Parent Pointer / BFS / DSU | If an adjacent vertex $v$ is already visited and $v \neq \text{parent}$, a cycle exists. | $O(V + E)$ | $O(V)$ |
| **Directed** | 3-State DFS Coloring / Kahn's Algorithm | A cycle exists if and only if a **back-edge** points to an ancestor currently in the active recursion call stack. | $O(V + E)$ | $O(V)$ |

---

## 2. Directed Cycle Detection: The 3-State Coloring Model

In a directed graph, simply checking `if neighbor in visited:` produces false positives (e.g., cross-edges in a diamond DAG $A \to B \to D$ and $A \to C \to D$). We must distinguish between:
- **State 0 (Unvisited)**: Node has not been touched.
- **State 1 (Visiting / In Stack)**: Node is currently being processed in the active DFS path.
- **State 2 (Visited / Done)**: Node and all its descendants have been completely processed.

```
State 0 (White)  ──>  State 1 (Gray)  ──>  State 2 (Black)
[Unvisited]           [In Call Stack]       [Fully Explored]
                           |
       Back-edge to Gray Node = CYCLE DETECTED!
```

---

## 3. Topological Sort (Kahn's Algorithm vs. DFS Post-Order)

A **Topological Sort** of a Directed Acyclic Graph (DAG) is a linear ordering of vertices such that for every directed edge $u \to v$, vertex $u$ comes before $v$ in the ordering.
If the graph contains a directed cycle, no topological ordering is possible.

### 3.1 Kahn's Algorithm (BFS In-Degree Model)
1. Compute the **in-degree** (number of incoming edges) for every vertex.
2. Enqueue all vertices with $\text{in-degree} = 0$ (nodes with no dependencies).
3. While the queue is non-empty:
   - Dequeue vertex $u$, append $u$ to the topological ordering.
   - For each neighbor $v$ of $u$, decrement $\text{in-degree}[v]$ by 1.
   - If $\text{in-degree}[v]$ becomes 0, enqueue $v$.
4. **Cycle Invariant**: If the number of processed nodes in the ordering is less than $V$, the graph contains at least one cycle.

---

## 4. Benchmark Problem Deep Dive: LeetCode 207 / 210 — Course Schedule I & II

### 4.1 Problem Statement Breakdown & Line-by-Line Annotations
> *"There are a total of `numCourses` courses you have to take, labeled from `0` to `numCourses - 1`."*
- $V = \text{numCourses}$. Nodes are integers from $0$ to $V - 1$.

> *"You are given an array `prerequisites` where `prerequisites[i] = [a, b]` indicates that you must take course `b` first if you want to take course `a`."*
- Directed dependency edge: $b \to a$ ($b$ must precede $a$).

> *"Return the ordering of courses you should take to finish all courses. If it is impossible, return an empty array."*
- Construct a Topological Ordering. If a directed cycle exists, return `[]`.

---

### 4.2 Constraints & Complexity Analysis
- $1 \le \text{numCourses} \le 2,000$
- $0 \le \text{prerequisites.length} \le 5,000$
- Target Time: $O(V + E)$ where $V = \text{numCourses}$ and $E = \text{prerequisites.length}$.
- Target Space: $O(V + E)$ for the adjacency list and in-degree array.

---

### 4.3 Invariant & Trace Table

Consider `numCourses = 4`, `prerequisites = [[1,0],[2,0],[3,1],[3,2]]`:
- Graph edges: $0 \to 1$, $0 \to 2$, $1 \to 3$, $2 \to 3$.
- Initial in-degrees:
  - Node 0: 0
  - Node 1: 1
  - Node 2: 1
  - Node 3: 2

| Step | Queue State | Dequeued Node | Updated Neighbors (In-Degree) | Topo Order |
| :--- | :--- | :--- | :--- | :--- |
| Initial | `[0]` | — | — | `[]` |
| 1 | `[]` | `0` | $1 \to 0$, $2 \to 0 \implies$ Queue `[1, 2]` | `[0]` |
| 2 | `[2]` | `1` | $3 \to 1$ | `[0, 1]` |
| 3 | `[]` | `2` | $3 \to 0 \implies$ Queue `[3]` | `[0, 1, 2]` |
| 4 | `[]` | `3` | None | `[0, 1, 2, 3]` |

Result length equals $V = 4 \implies$ Valid DAG. Result is `[0, 1, 2, 3]`.

---

### 4.4 Complete Python Implementation (Kahn's Algorithm & DFS 3-State)

#### Implementation A: Kahn's Algorithm (BFS)
```python
from collections import deque, defaultdict

class SolutionKahn:
    def findOrder(self, numCourses: int, prerequisites: list[list[int]]) -> list[int]:
        adj = defaultdict(list)
        indegree = [0] * numCourses
        
        # Build adjacency list: b -> a
        for dest, src in prerequisites:
            adj[src].append(dest)
            indegree[dest] += 1
            
        # Queue all nodes with zero dependencies
        queue = deque([course for course in range(numCourses) if indegree[course] == 0])
        topo_order = []
        
        while queue:
            node = queue.popleft()
            topo_order.append(node)
            
            for neighbor in adj[node]:
                indegree[neighbor] -= 1
                if indegree[neighbor] == 0:
                    queue.append(neighbor)
                    
        # If order contains all courses, no cycle exists
        return topo_order if len(topo_order) == numCourses else []
```

#### Implementation B: 3-State DFS Cycle Detection & Reverse Post-Order
```python
class SolutionDFS:
    def findOrder(self, numCourses: int, prerequisites: list[list[int]]) -> list[int]:
        adj = defaultdict(list)
        for dest, src in prerequisites:
            adj[src].append(dest)
            
        # 0: Unvisited, 1: Visiting (in stack), 2: Visited (done)
        state = [0] * numCourses
        topo_order = []
        has_cycle = False
        
        def dfs(node: int) -> None:
            nonlocal has_cycle
            if has_cycle:
                return
                
            state[node] = 1  # Mark active in call stack
            
            for neighbor in adj[node]:
                if state[neighbor] == 1:
                    has_cycle = True  # Back-edge detected!
                    return
                elif state[neighbor] == 0:
                    dfs(neighbor)
                    
            state[node] = 2  # Mark completely explored
            topo_order.append(node)  # Post-order insertion
            
        for course in range(numCourses):
            if state[course] == 0:
                dfs(course)
                if has_cycle:
                    return []
                    
        # Reverse post-order produces topological ordering
        return topo_order[::-1]
```

- **Time Complexity**: $O(V + E)$ — Every vertex is processed once, every edge traversed once.
- **Space Complexity**: $O(V + E)$ — Adjacency list and recursion stack / queue.

---

### 4.5 Live Verbalization Script

> *"This problem reduces to finding a Topological Ordering of a Directed Acyclic Graph (DAG) formed by course dependencies.
> 
> I model each course as a vertex and each prerequisite `[a, b]` as a directed edge $b \to a$. To resolve dependencies iteratively, I apply Kahn's algorithm.
> 
> First, I calculate the in-degree of every course and seed a FIFO queue with all courses having an in-degree of 0, meaning they have no prerequisites. 
> 
> In each step, I pop a course from the queue, append it to my result list, and decrement the in-degrees of all its dependent downstream neighbors. Whenever a neighbor's in-degree reaches 0, it becomes eligible and is added to the queue.
> 
> At termination, if the length of the processed sequence matches the total number of courses, a valid curriculum exists and is returned. Otherwise, if the sequence has fewer nodes, a cyclic dependency exists and we return an empty array.
> 
> This runs in $O(V + E)$ time and consumes $O(V + E)$ space."*
