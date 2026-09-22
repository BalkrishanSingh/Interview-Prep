# Domain 07: Dynamic Programming — Grid & Matrix Path Problems

---

## 1. Grid DP Mental Model & The DAG Property

A 2D matrix of size $M \times N$ where movement is restricted (e.g., only moving **Right** and **Down**) forms a **Directed Acyclic Graph (DAG)**:
- Every cell $(r, c)$ can only be reached from $(r-1, c)$ (from above) and $(r, c-1)$ (from the left).
- Because there are no cycles, states can be populated sequentially in row-major order:
  $$\text{dp}[r][c] = f(\text{dp}[r-1][c], \text{dp}[r][c-1])$$

```
     (r-1, c)
         │
         ▼
(r, c-1) ──> (r, c)
```

### Space Optimization Invariant ($O(N)$ Space)
Because computing row $r$ requires only the cells immediately above in row $r-1$ and to the left in the current row, a 2D matrix of size $M \times N$ can always be compressed to a **1D array of size $N$**:
```python
# dp[c] acts as dp[r-1][c] before update, and dp[c-1] acts as dp[r][c-1]
dp[c] = dp[c] + dp[c - 1]
```

---

## 2. Benchmark Problem 1: LeetCode 62 & 63 — Unique Paths I & II

### 2.1 Problem Statement Breakdown & Line-by-Line Annotations
> *"There is a robot on an `m x n` grid. The robot is initially located at the top-left corner `(0, 0)`."*
- Source state: `(0, 0)`. Target destination: `(m-1, n-1)`.

> *"The robot can only move either down or right at any point in time."*
- Transitions allowed: $(r+1, c)$ and $(r, c+1)$.

> *"In Unique Paths II: An obstacle and space are marked as `1` and `0` respectively in `obstacleGrid`."*
- If cell `(r, c)` contains an obstacle (`obstacleGrid[r][c] == 1`), no path can pass through it $\implies \text{dp}[r][c] = 0$.

---

### 2.2 Constraints & Complexity Analysis
- $1 \le m, n \le 100$
- Target Time: $O(m \times n) \le 10,000$ operations (instantaneous).
- Target Space: $O(n)$ using 1D rolling array.

---

### 2.3 Recurrence Relation & Base Cases
Let $\text{dp}[c]$ be the number of paths reaching column $c$ in the current row:
- If `obstacleGrid[r][c] == 1`: $\text{dp}[c] = 0$.
- Otherwise: $\text{dp}[c] = \text{dp}[c] + \text{dp}[c-1]$.

**Base Case**:
- If `obstacleGrid[0][0] == 0`, `dp[0] = 1`, else `0`.

---

### 2.4 Complete Python Implementation ($O(N)$ Space)

```python
class SolutionUniquePathsWithObstacles:
    def uniquePathsWithObstacles(self, obstacleGrid: list[list[int]]) -> int:
        if not obstacleGrid or obstacleGrid[0][0] == 1:
            return 0
            
        rows, cols = len(obstacleGrid), len(obstacleGrid[0])
        dp = [0] * cols
        dp[0] = 1  # Starting position
        
        for r in range(rows):
            for c in range(cols):
                if obstacleGrid[r][c] == 1:
                    dp[c] = 0  # Obstacle blocks all paths through this cell
                elif c > 0:
                    dp[c] += dp[c - 1]  # Add paths coming from the left
                    
        return dp[cols - 1]
```

---

## 3. Benchmark Problem 2: LeetCode 64 — Minimum Path Sum

### 3.1 Problem Statement & Annotations
> *"Given a `m x n` `grid` filled with non-negative numbers, find a path from top left to bottom right, which minimizes the sum of all numbers along its path."*
- Optimization: Minimize accumulated cell values.

### 3.2 Recurrence Relation
$$\text{dp}[r][c] = \text{grid}[r][c] + \min(\text{dp}[r-1][c], \text{dp}[r][c-1])$$

```python
class SolutionMinPathSum:
    def minPathSum(self, grid: list[list[int]]) -> int:
        rows, cols = len(grid), len(grid[0])
        dp = [float('inf')] * cols
        dp[0] = 0  # Seed to allow starting cell to initialize
        
        for r in range(rows):
            # First column in row r can only come from above
            dp[0] += grid[r][0]
            for c in range(1, cols):
                dp[c] = grid[r][c] + min(dp[c], dp[c - 1])
                
        return dp[cols - 1]
```
- **Time Complexity**: $O(M \times N)$.
- **Space Complexity**: $O(N)$.

---

## 4. Benchmark Problem 3: LeetCode 174 — Dungeon Game (Reverse Grid DP)

### 4.1 Problem Statement Breakdown & Line-by-Line Annotations
> *"The demons had captured the princess and imprisoned her in the bottom-right corner of a `dungeon`."*
- Destination target: `(m-1, n-1)`. Starting cell: `(0, 0)`.

> *"The knight has an initial health points represented by a positive integer. If at any point his health points drops to 0 or below, he dies immediately."*
- Invariant: Knight health must be $\ge 1$ at **every** step of the journey.

> *"Determine the knight's minimum initial health so that he is able to rescue the princess."*

---

### 4.2 Why Forward DP Fails & The Reverse Invariant

- **Why Forward DP Fails**: If we compute forward from `(0, 0)`, a state with higher remaining health might have required a much higher initial investment. The subproblem exhibits no optimal substructure moving forward because future damage dictates past requirements.
- **The Reverse Invariant**: Start at the princess `(m-1, n-1)` and work **backwards** to `(0, 0)`.
  Let $\text{dp}[r][c]$ be the **minimum health required upon entering cell $(r, c)$** to survive to the end:
  - From $(r, c)$, the knight can step either Right $(r, c+1)$ or Down $(r+1, c)$.
  - The knight greedily chooses the branch needing less entry health:
    $$\text{min\_future\_health} = \min(\text{dp}[r+1][c], \text{dp}[r][c+1])$$
  - To enter $(r, c)$ and leave with `min_future_health` after absorbing `dungeon[r][c]`:
    $$\text{required} = \text{min\_future\_health} - \text{dungeon}[r][c]$$
  - Because health must never drop to 0 or below, the required health can never be less than 1:
    $$\text{dp}[r][c] = \max(1, \text{min\_future\_health} - \text{dungeon}[r][c])$$

---

### 4.3 Complete Python Implementation ($O(N)$ Space Reverse DP)

```python
class SolutionDungeonGame:
    def calculateMinimumHP(self, dungeon: list[list[int]]) -> int:
        rows, cols = len(dungeon), len(dungeon[0])
        
        # dp array of size cols + 1 initialized to infinity
        dp = [float('inf')] * (cols + 1)
        
        # Base case guards at destination (m-1, n-1):
        # Exiting the princess cell requires at least 1 HP
        dp[cols - 1] = 1
        dp[cols] = float('inf')
        
        for r in range(rows - 1, -1, -1):
            for c in range(cols - 1, -1, -1):
                # When at the princess cell, special boundary handling
                if r == rows - 1 and c == cols - 1:
                    min_exit_health = 1
                else:
                    # Minimum health required at next step (Down: dp[c], Right: dp[c+1])
                    min_exit_health = min(dp[c], dp[c + 1])
                    
                dp[c] = max(1, min_exit_health - dungeon[r][c])
                
        return dp[0]
```

- **Time Complexity**: $O(M \times N)$ — Every cell evaluated once.
- **Space Complexity**: $O(N)$ — 1D array of size $N + 1$.

---

### 4.4 Live Verbalization Script

> *"For Dungeon Game, a standard forward DP from top-left to bottom-right fails because a path with high remaining health might have required an unnecessarily high initial health earlier.
> 
> Instead, I formulate a reverse dynamic programming traversal from the princess at `(m-1, n-1)` back to the entrance at `(0, 0)`.
> 
> I define `dp[r][c]` as the minimum health points needed upon *entering* cell `(r, c)` to successfully complete the rescue.
> 
> From cell `(r, c)`, the knight will transition to whichever adjacent cell (Right or Down) demands lower entrance health: `min_exit = min(dp[r+1][c], dp[r][c+1])`.
> 
> Accounting for the current room's value, the entrance health must satisfy: `entry_health + dungeon[r][c] >= min_exit`, which simplifies to `entry_health = min_exit - dungeon[r][c]`. Since the knight must always remain alive, health can never fall below 1, giving:
> `dp[r][c] = max(1, min_exit - dungeon[r][c])`.
> 
> Working backwards to `(0, 0)` gives the exact initial health needed in $O(M \times N)$ time with $O(N)$ space."*
