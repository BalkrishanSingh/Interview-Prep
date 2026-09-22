# Domain 07: Dynamic Programming — Interval DP & Matrix Chain Multiplication

---

## 1. Interval DP Mental Model

In **Interval Dynamic Programming**, the state parameters represent the boundaries of a contiguous range or subarray:
- Subproblem: $\text{dp}[i][j]$ represents the optimal solution for the interval from index $i$ to index $j$.
- Invariant: A solution for interval $[i, j]$ is formed by evaluating all possible split points $k \in [i \dots j-1]$ that divide the range into two independent sub-intervals $[i, k]$ and $[k+1, j]$:
  $$\text{dp}[i][j] = \min_{i \le k < j} / \max_{i \le k < j} \left( \text{dp}[i][k] + \text{dp}[k+1][j] + \text{transition\_cost}(i, k, j) \right)$$

### Evaluation Order: Increasing Interval Length
Because $\text{dp}[i][j]$ depends on strictly shorter sub-intervals, loops must iterate by increasing interval length $L \in [2 \dots N]$:
```python
for length in range(2, n + 1):
    for i in range(n - length + 1):
        j = i + length - 1
        for k in range(i, j):
            dp[i][j] = min(dp[i][j], dp[i][k] + dp[k + 1][j] + cost)
```

---

## 2. Benchmark Problem 1: Matrix Chain Multiplication (MCM)

Given a sequence of matrices $A_1, A_2, \dots, A_n$ where matrix $A_i$ has dimension $p_{i-1} \times p_i$, find the parenthesization that minimizes the total scalar multiplications.

### Recurrence
$$\text{dp}[i][j] = \min_{i \le k < j} \left( \text{dp}[i][k] + \text{dp}[k+1][j] + p_{i-1} \cdot p_k \cdot p_j \right)$$
- Base Case: $\text{dp}[i][i] = 0$ (multiplying a single matrix costs 0 operations).

```python
def matrix_chain_multiplication(p: list[int]) -> int:
    n = len(p) - 1  # Number of matrices
    dp = [[0] * (n + 1) for _ in range(n + 1)]
    
    for length in range(2, n + 1):
        for i in range(1, n - length + 2):
            j = i + length - 1
            dp[i][j] = float('inf')
            for k in range(i, j):
                cost = dp[i][k] + dp[k + 1][j] + p[i - 1] * p[k] * p[j]
                dp[i][j] = min(dp[i][j], cost)
                
    return dp[1][n]
```
- **Time Complexity**: $O(N^3)$ — 3 nested loops (length, start $i$, split $k$).
- **Space Complexity**: $O(N^2)$.

---

## 3. Benchmark Problem 2: LeetCode 312 — Burst Balloons

### 3.1 Problem Statement Breakdown & Line-by-Line Annotations
> *"You are given `n` balloons, indexed from `0` to `n - 1`. Each balloon is painted with a number on it represented by an array `nums`."*
- Array of positive integers.

> *"You are asked to burst all the balloons. If you burst balloon `i`, you will get `nums[i - 1] * nums[i] * nums[i + 1]` coins."*
- When balloon $i$ is burst, its left and right neighbors become immediately adjacent!
- If $i - 1$ or $i + 1$ goes out of bounds, treat the value as $1$.

> *"Return the maximum coins you can collect by bursting the balloons wisely."*

---

### 3.2 The Core Breakthrough: Reverse Thinking ("Which Balloon Bursts LAST?")

- **Why Forward Thinking Fails**:
  If we decide which balloon $k$ to burst *first*, balloons $k-1$ and $k+1$ become adjacent. The left subproblem $[i \dots k-1]$ and the right subproblem $[k+1 \dots j]$ now depend on each other's boundaries, destroying optimal substructure.

- **The Reverse Invariant (Bursting $k$ LAST)**:
  Pad `nums` with $1$ on both ends: `padded = [1] + nums + [1]`.
  Consider the open interval $(i, j)$ where boundary balloons $i$ and $j$ remain **unburst**.
  Suppose balloon $k \in (i, j)$ is the **very last balloon burst** within this range:
  - Because all other balloons in $(i, j)$ were burst prior to $k$, when $k$ finally bursts, its immediate adjacent neighbors are guaranteed to be the boundary balloons $i$ and $j$!
  - Coins earned from bursting $k$:
    $$\text{coins} = \text{padded}[i] \cdot \text{padded}[k] \cdot \text{padded}[j]$$
  - Crucially, the subproblem on $(i, k)$ and the subproblem on $(k, j)$ are now **completely independent**!

---

### 3.3 Recurrence Relation
Let $\text{dp}[i][j]$ be the maximum coins obtained by bursting all balloons strictly between index $i$ and index $j$:
$$\text{dp}[i][j] = \max_{i < k < j} \left( \text{dp}[i][k] + \text{dp}[k][j] + \text{padded}[i] \cdot \text{padded}[k] \cdot \text{padded}[j] \right)$$

---

### 3.4 Complete Python Implementation

```python
class SolutionBurstBalloons:
    def maxCoins(self, nums: list[int]) -> int:
        # Pad array with virtual 1s at both boundaries
        padded = [1] + nums + [1]
        n = len(padded)
        
        # dp[i][j] stores max coins from bursting balloons in open interval (i, j)
        dp = [[0] * n for _ in range(n)]
        
        # length is the distance between i and j (minimum distance 2: e.g., i=0, j=2, k=1)
        for length in range(2, n):
            for i in range(n - length):
                j = i + length
                # Test which balloon k in range (i, j) bursts LAST
                max_coins = 0
                for k in range(i + 1, j):
                    coins = dp[i][k] + dp[k][j] + padded[i] * padded[k] * padded[j]
                    if coins > max_coins:
                        max_coins = coins
                dp[i][j] = max_coins
                
        return dp[0][n - 1]
```

- **Time Complexity**: $O(N^3)$ — Outer loops iterate over all intervals $O(N^2)$, inner loop tests $O(N)$ split points.
- **Space Complexity**: $O(N^2)$ — 2D table of size $(N+2) \times (N+2)$.

---

### 3.5 Live Verbalization Script

> *"For Burst Balloons, choosing which balloon to burst first creates dependency coupling between subproblems because adjacent neighbors change dynamically.
> 
> The breakthrough is to invert the problem: instead of asking which balloon bursts first, I ask **which balloon $k$ bursts LAST** in the open interval $(i, j)$.
> 
> I pad the array with 1 at both boundaries.
> If balloon $k$ is the last balloon to burst in $(i, j)$, all other balloons in $(i, k)$ and $(k, j)$ have already vanished. Therefore, when $k$ bursts, its immediate neighbors are guaranteed to be the boundary balloons $i$ and $j$, earning `padded[i] * padded[k] * padded[j]` coins.
> 
> This decouples the problem into two completely independent subproblems: `dp[i][k]` and `dp[k][j]`.
> 
> I evaluate intervals in increasing length from 2 to $N + 1$. For each interval, I find the split point $k$ that maximizes the sum of both subproblems plus the coins from bursting $k$ last.
> 
> The final answer is `dp[0][n - 1]`, running in $O(N^3)$ time and $O(N^2)$ space."*
