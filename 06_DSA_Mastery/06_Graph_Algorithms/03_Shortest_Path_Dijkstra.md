# Domain 06: Graph Algorithms — Single-Source Shortest Paths (Dijkstra's Algorithm)

---

## 1. Shortest Path Paradigms & Algorithm Invariants

Selecting the correct shortest path algorithm depends strictly on edge weights and graph properties:

| Algorithm | Edge Weights | Graph Type | Time Complexity | Core Invariant |
| :--- | :--- | :--- | :--- | :--- |
| **BFS** | Unweighted / Uniform ($w = 1$) | Directed / Undirected | $O(V + E)$ | First arrival at a node guarantees the shortest path. |
| **Dijkstra** | Non-negative weights ($w \ge 0$) | Directed / Undirected | $O((V + E) \log V)$ | Greedily settles the node with the smallest tentative distance; once popped from min-heap, its shortest distance is finalized. |
| **Bellman-Ford** | Arbitrary weights (can be negative) | Directed / Undirected | $O(V \times E)$ | Relaxes all $E$ edges $V - 1$ times; detects negative cycles if a relaxation occurs on the $V$-th iteration. |
| **Floyd-Warshall** | Arbitrary weights, All-pairs | Directed / Undirected | $O(V^3)$ | Dynamic programming over intermediate vertices $k \in [1, V]$. |

---

## 2. Dijkstra's Algorithm Invariant & The Non-Negative Weight Requirement

Dijkstra maintains a set of finalized distances `dist` and a Min-Priority Queue of tuples `(current_distance, node)`.

```
                    ┌─────────────────────────┐
                    │ Min-Heap Priority Queue │
                    │ (tentative_dist, u)     │
                    └────────────┬────────────┘
                                 │ Pop minimum distance
                                 ▼
                     Has u been settled with
                     dist[u] < current_dist?
                                ╱ ╲
                          YES  ╱   ╲  NO
                              ▼     ▼
                            Skip   dist[u] = current_dist
                                   Relax all edges u -> v:
                                   new_d = dist[u] + weight
                                   If new_d < dist[v]:
                                       dist[v] = new_d
                                       push (new_d, v) to heap
```

### Why Negative Weights Break Dijkstra
Dijkstra greedily marks a vertex as settled as soon as it is popped from the min-heap. If negative edge weights exist, a longer positive path could later be reduced by a negative weight, violating the greedy choice property.

---

## 3. Benchmark Problem Deep Dive: LeetCode 743 — Network Delay Time

### 3.1 Problem Statement Breakdown & Line-by-Line Annotations
> *"You are given a network of `n` nodes, labeled from `1` to `n`."*
- 1-indexed graph with $V = n$.

> *"You are also given `times`, a list of travel times as directed edges `times[i] = (u_i, v_i, w_i)` where $u_i$ is the source node, $v_i$ is the target node, and $w_i$ is the time it takes for a signal to travel from source to target."*
- Directed weighted graph with non-negative edge weights $w_i \ge 0$.

> *"We will send a signal from a given node `k`. Return the minimum time it takes for all the `n` nodes to receive the signal. If it is impossible for all the `n` nodes to receive the signal, return `-1`."*
- The signal travels concurrently along all branches.
- Total time to reach all nodes = $\max_{v \in [1, n]} \text{dist}(k, v)$.
- If any node $v$ is unreachable ($\text{dist}(k, v) = \infty$), return $-1$.

---

### 3.2 Constraints & Complexity Analysis
- $1 \le k \le n \le 100$
- $1 \le \text{times.length} \le 6,000$
- $0 \le w_i \le 100$
- Target Time: $O(E \log V)$ with binary heap $\implies 6,000 \log(100) \approx 4 \times 10^4$ operations (well within $10^8$ threshold).
- Target Space: $O(V + E)$ for adjacency list, distance array, and priority queue.

---

### 3.3 Dry Run Trace Table

Consider $n = 4, k = 2$, `times = [[2,1,1],[2,3,1],[3,4,1]]`.

| Step | Heap State `[(d, node)]` | Popped `(d, u)` | Action / Relaxation | `dist` Array `[1, 2, 3, 4]` |
| :--- | :--- | :--- | :--- | :--- |
| Init | `[(0, 2)]` | — | Initialize `dist[2]=0`, all others $\infty$ | `[inf, 0, inf, inf]` |
| 1 | `[]` | `(0, 2)` | Relax edges $2 \to 1 (w=1)$, $2 \to 3 (w=1)$ | `[1, 0, 1, inf]` |
| 2 | `[(1, 1), (1, 3)]` | `(1, 1)` | Node 1 has no outgoing edges | `[1, 0, 1, inf]` |
| 3 | `[(1, 3)]` | `(1, 3)` | Relax edge $3 \to 4 (w=1) \implies \text{dist}[4]=1+1=2$ | `[1, 0, 1, 2]` |
| 4 | `[(2, 4)]` | `(2, 4)` | Node 4 has no outgoing edges | `[1, 0, 1, 2]` |

Final settled distances:
- $\text{dist}[1] = 1, \text{dist}[2] = 0, \text{dist}[3] = 1, \text{dist}[4] = 2$.
- Max settled distance = $\max(1, 0, 1, 2) = 2$.
- All nodes reached $\implies$ Return $2$.

---

### 3.4 Complete Python Implementation (Min-Heap Priority Queue)

```python
import heapq
from collections import defaultdict

class Solution:
    def networkDelayTime(self, times: list[list[int]], n: int, k: int) -> int:
        # Build weighted adjacency list: u -> (v, weight)
        adj = defaultdict(list)
        for u, v, w in times:
            adj[u].append((v, w))
            
        # Distance table initialized to infinity (1-indexed)
        distances = {node: float('inf') for node in range(1, n + 1)}
        distances[k] = 0
        
        # Priority Queue stores: (accumulated_dist, node)
        pq = [(0, k)]
        
        while pq:
            curr_dist, u = heapq.heappop(pq)
            
            # Optimization: stale heap entry check
            if curr_dist > distances[u]:
                continue
                
            # Relax all outgoing directed edges
            for v, weight in adj[u]:
                new_dist = curr_dist + weight
                if new_dist < distances[v]:
                    distances[v] = new_dist
                    heapq.heappush(pq, (new_dist, v))
                    
        # Find maximum time among all reachable nodes
        max_time = max(distances.values())
        return max_time if max_time < float('inf') else -1
```

- **Time Complexity**: $O(E \log V)$ — Every edge is relaxed at most once, and inserting into the priority queue takes $O(\log V)$.
- **Space Complexity**: $O(V + E)$ — To store the adjacency graph, heap, and distance lookup map.

---

### 3.5 Live Verbalization Script

> *"This problem asks for the minimum time required for a signal originating at source node $k$ to reach all $n$ nodes across a network with non-negative transmission weights.
> 
> Because all edge weights are non-negative, this is an exact match for Dijkstra's Single-Source Shortest Path algorithm. The signal propagates along the shortest path to each node, and the time for all nodes to receive the signal is the maximum of the shortest path distances from $k$ to every vertex.
> 
> I initialize a distance lookup table with infinity for all nodes and distance 0 for the source $k$. I push `(0, k)` into a min-heap.
> 
> In each iteration, I pop the vertex $u$ with the minimum tentative distance. If the popped distance exceeds our already recorded shortest distance for $u$, we skip it as a stale entry. Otherwise, we relax all outgoing edges $(u, v, w)$: if reaching $v$ through $u$ yields a smaller distance than `distances[v]`, we update `distances[v]` and push `(new_dist, v)` onto the heap.
> 
> After the heap empties, I check the maximum distance in the table. If any node remains at infinity, the network is disconnected and I return $-1$; otherwise, I return the maximum distance.
> 
> This runs in $O(E \log V)$ time and uses $O(V + E)$ space."*
