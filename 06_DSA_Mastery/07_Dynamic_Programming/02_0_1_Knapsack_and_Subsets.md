# Domain 07: Dynamic Programming — 0/1 Knapsack & Subset Sum

---

## 1. 0/1 Knapsack Mental Model & Recurrence

In the classical **0/1 Knapsack** problem, we are given $N$ items, each with a weight $w_i$ and value $v_i$, and a knapsack of capacity $W$. Each item can either be **chosen once (1)** or **left behind (0)**.

### State Definition
Let $\text{dp}[i][c]$ be the maximum value achievable considering items from index $0$ to $i-1$ with remaining capacity $c$.

### Decision Recurrence
$$\text{dp}[i][c] = \begin{cases} \text{dp}[i-1][c] & \text{if } w_{i-1} > c \\ \max(\text{dp}[i-1][c], \text{dp}[i-1][c - w_{i-1}] + v_{i-1}) & \text{if } w_{i-1} \le c \end{cases}$$

---

## 2. Space Optimization: Why Reverse Iteration Prevents Double Counting

When compressing the 2D table $\text{dp}[N+1][W+1]$ to a single 1D array $\text{dp}[W+1]$:

```
Row i-1: [ ... , dp[c - w], ... , dp[c], ... ]
                     │                 │
                     └────┬────────────┘
                          ▼
Row i:   [ ... , dp[c - w], ... , new_dp[c], ... ]
```

- If we iterate $c$ **forward** ($0 \to W$): $\text{dp}[c - w]$ is updated in row $i$ *before* we compute $\text{dp}[c]$, effectively allowing the same item to be used multiple times (Unbounded Knapsack).
- If we iterate $c$ **backward** ($W \to w$): $\text{dp}[c - w]$ still holds the value from the **previous row $i-1$**, preserving the 0/1 constraint.

---

## 3. Benchmark Problem Deep Dive: LeetCode 416 — Partition Equal Subset Sum

### 3.1 Problem Statement Breakdown & Line-by-Line Annotations
> *"Given an integer array `nums`, return `true` if you can partition the array into two subsets such that the sum of the elements in both subsets is equal or `false` otherwise."*
- Let $S = \sum \text{nums}$.
- If $S$ is odd: $S \% 2 \neq 0 \implies$ impossible to split into two equal integers $\implies$ return `false`.
- If $S$ is even: Target subset sum is $T = S / 2$.
- The problem reduces to: *Does there exist a subset of `nums` that sums exactly to $T$?* (0/1 Subset Sum feasibility).

---

### 3.2 Constraints & Complexity Analysis
- $1 \le \text{nums.length} \le 200$
- $1 \le \text{nums}[i] \le 100$
- Total sum $S \le 200 \times 100 = 20,000 \implies T = S / 2 \le 10,000$.
- Target Time: $O(N \times T) = 200 \times 10,000 = 2 \times 10^6$ operations (runs in ~15 ms).
- Target Space: $O(T) = 10,000$ booleans (1D space-optimized).

---

### 3.3 Recurrence Relation & Base Cases
Let $\text{dp}[c]$ be a boolean indicating whether a subset sum of $c$ can be formed using a subset of elements seen so far:
$$\text{dp}[c] = \text{dp}[c] \lor \text{dp}[c - \text{num}]$$

**Base Case**:
- $\text{dp}[0] = \text{True}$ (the empty subset sums to 0).
- $\text{dp}[c] = \text{False}$ for $c > 0$.

---

### 3.4 Dry Run Trace Table

Consider `nums = [1, 5, 11, 5]`:
- Sum $S = 22 \implies \text{Target } T = 11$.

| Element `num` | Reverse Inner Loop $c \in [11, \text{num}]$ | Updated True Indices in `dp` |
| :--- | :--- | :--- |
| Init | — | `{0}` |
| 1 | $c = 1 \implies \text{dp}[1] = \text{dp}[0] = \text{True}$ | `{0, 1}` |
| 5 | $c = 6 \implies \text{dp}[6] = \text{dp}[1] = \text{True}$; $c = 5 \implies \text{dp}[5] = \text{dp}[0] = \text{True}$ | `{0, 1, 5, 6}` |
| 11 | $c = 11 \implies \text{dp}[11] = \text{dp}[0] = \text{True}$ | `{0, 1, 5, 6, 11}` |

Since $\text{dp}[11] = \text{True}$, return `True`.

---

### 3.5 Complete Python Implementations

#### Top-Down (Memoization)
```python
from functools import lru_cache

class SolutionTopDown:
    def canPartition(self, nums: list[int]) -> bool:
        total_sum = sum(nums)
        if total_sum % 2 != 0:
            return False
            
        target = total_sum // 2
        n = len(nums)
        
        @lru_cache(maxsize=None)
        def dp(index: int, curr_sum: int) -> bool:
            if curr_sum == target:
                return True
            if curr_sum > target or index >= n:
                return False
                
            # Choice 1: Include nums[index]
            # Choice 2: Exclude nums[index]
            return dp(index + 1, curr_sum + nums[index]) or dp(index + 1, curr_sum)
            
        return dp(0, 0)
```

#### Bottom-Up (1D Space-Optimized)
```python
class Solution:
    def canPartition(self, nums: list[int]) -> bool:
        total_sum = sum(nums)
        
        # If the total sum is odd, two equal integer partitions cannot exist
        if total_sum % 2 != 0:
            return False
            
        target = total_sum // 2
        
        # dp[c] is True if a subset with sum c can be formed
        dp = [False] * (target + 1)
        dp[0] = True  # Base case: 0 sum is always possible with empty subset
        
        for num in nums:
            # Iterate backwards to prevent using the current num multiple times
            for c in range(target, num - 1, -1):
                dp[c] = dp[c] or dp[c - num]
                
            # Early exit optimization
            if dp[target]:
                return True
                
        return dp[target]
```

- **Time Complexity**: $O(N \times \text{target})$ where $\text{target} = \sum \text{nums} / 2$.
- **Space Complexity**: $O(\text{target})$ — Reduced from $O(N \times \text{target})$ using 1D reverse iteration.

---

## 4. Benchmark Problem 2: LeetCode 494 — Target Sum

### 4.1 Mathematical Reduction to Subset Sum
Given `nums` and `target`, assign `+` or `-` to each element so the total equals `target`.
Let $P$ be the subset of numbers with positive signs, and $N$ be the subset with negative signs:
$$\sum P - \sum N = \text{target}$$
$$\sum P + \sum N = \text{total\_sum}$$
Adding the two equations:
$$2 \sum P = \text{target} + \text{total\_sum} \implies \sum P = \frac{\text{target} + \text{total\_sum}}{2}$$

**Validity Check**:
1. $(\text{target} + \text{total\_sum})$ must be even and non-negative.
2. If $|\text{target}| > \text{total\_sum}$, return $0$.

The problem now identically matches finding the number of subsets with sum equal to $\frac{\text{target} + \text{total\_sum}}{2}$.

```python
class SolutionTargetSum:
    def findTargetSumWays(self, nums: list[int], target: int) -> int:
        total_sum = sum(nums)
        
        if (total_sum + target) % 2 != 0 or total_sum < abs(target):
            return 0
            
        subset_target = (total_sum + target) // 2
        
        # dp[c] stores the count of subsets that sum to c
        dp = [0] * (subset_target + 1)
        dp[0] = 1
        
        for num in nums:
            for c in range(subset_target, num - 1, -1):
                dp[c] += dp[c - num]
                
        return dp[subset_target]
```

---

### 4.2 Live Verbalization Script

> *"For Partition Equal Subset Sum, the goal is to partition `nums` into two subsets with equal sums.
> 
> First, I sum all elements. If the total sum is odd, an equal integer split is mathematically impossible, so I return `false`. Otherwise, our target is half the total sum, and the problem becomes a 0/1 Knapsack feasibility question: can we select a subset that sums exactly to `target`?
> 
> I use a 1D boolean DP array of size `target + 1`, where `dp[c]` denotes whether sum `c` is achievable. `dp[0]` is initialized to `true`.
> 
> For each number in the array, I traverse the DP table backwards from `target` down to `num`. Traversing backwards is essential because it guarantees that `dp[c - num]` represents the state from the previous iteration, ensuring each number is included at most once.
> 
> If `dp[target]` becomes `true`, we can immediately terminate and return `true`.
> 
> This runs in $O(N \times \text{target})$ time and $O(\text{target})$ space."*
