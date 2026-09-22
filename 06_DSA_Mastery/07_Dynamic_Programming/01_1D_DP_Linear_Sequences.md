# Domain 07: Dynamic Programming — 1D DP & Linear Sequences

---

## 1. 1D DP Mental Model & Transition Taxonomy

1D Dynamic Programming addresses optimization or counting problems where decisions at step $i$ depend strictly on a fixed window of preceding states (e.g., $i-1, i-2$) or an iteration over all previous states $j \in [0, i-1]$.

```
State Dependency Window:
dp[i] = f(dp[i-1], dp[i-2])           ──> O(1) space optimization possible
dp[i] = min_{0 <= j < i} (dp[j] + cost) ──> O(N^2) or O(N log N) transitions
```

### The 3 Archetypal Patterns
1. **Fibonacci / Step Transitions**: `dp[i] = dp[i-1] + dp[i-2]` (e.g., Climbing Stairs, Min Cost Climbing Stairs).
2. **Mutual Exclusion / Non-Adjacent Transitions**: `dp[i] = max(dp[i-1], dp[i-2] + val[i])` (e.g., House Robber).
3. **Change-Making / Unbounded Transitions**: `dp[i] = min_{c in coins} (dp[i - c] + 1)` (e.g., Coin Change).

---

## 2. Benchmark Problem 1: LeetCode 198 — House Robber

### 2.1 Problem Statement Breakdown & Line-by-Line Annotations
> *"You are a professional robber planning to rob houses along a street. Each house has a certain amount of money stashed."*
- Given an array `nums` of length $N$ where `nums[i]` is money in house $i$.

> *"The only constraint stopping you is that adjacent houses have security systems connected and it will automatically contact the police if two adjacent houses were broken into on the same night."*
- **Constraint**: Cannot rob both house $i$ and house $i+1$.

> *"Return the maximum amount of money you can rob tonight without alerting the police."*
- Objective: Maximize total robbed money under the non-adjacent constraint.

---

### 2.2 Constraints & Complexity Analysis
- $1 \le \text{nums.length} \le 100$
- $0 \le \text{nums}[i] \le 400$
- Target Time: $O(N)$ linear time.
- Target Space: $O(1)$ space using two rolling variables.

---

### 2.3 Decision Exploration & Recurrence Relation
At each house $i$, we make a binary choice:
1. **Rob house $i$**: Earn $\text{nums}[i]$, but cannot rob house $i-1$. Max money is $\text{nums}[i] + \text{dp}[i-2]$.
2. **Skip house $i$**: Max money is whatever we earned up to house $i-1$, i.e., $\text{dp}[i-1]$.

**Recurrence Relation**:
$$\text{dp}[i] = \max(\text{dp}[i-1], \text{dp}[i-2] + \text{nums}[i])$$

**Base Cases**:
- $\text{dp}[0] = \text{nums}[0]$
- $\text{dp}[1] = \max(\text{nums}[0], \text{nums}[1])$

---

### 2.4 Dry Run Trace Table

Consider `nums = [2, 7, 9, 3, 1]`:

| House $i$ | `nums[i]` | Option 1: Skip (`dp[i-1]`) | Option 2: Rob (`dp[i-2] + nums[i]`) | `dp[i]` (Max) |
| :--- | :--- | :--- | :--- | :--- |
| 0 | 2 | — | — | 2 |
| 1 | 7 | 2 | 7 | 7 |
| 2 | 9 | 7 | $2 + 9 = 11$ | 11 |
| 3 | 3 | 11 | $7 + 3 = 10$ | 11 |
| 4 | 1 | 11 | $11 + 1 = 12$ | **12** |

Result: `12`.

---

### 2.5 Complete Python Implementations

#### Top-Down (Memoization)
```python
from functools import lru_cache

class SolutionTopDown:
    def rob(self, nums: list[int]) -> int:
        n = len(nums)
        
        @lru_cache(maxsize=None)
        def dp(i: int) -> int:
            if i >= n:
                return 0
            # Choice 1: Rob current and skip next; Choice 2: Skip current
            rob_current = nums[i] + dp(i + 2)
            skip_current = dp(i + 1)
            return max(rob_current, skip_current)
            
        return dp(0)
```

#### Bottom-Up Space-Optimized ($O(1)$ Space)
```python
class Solution:
    def rob(self, nums: list[int]) -> int:
        if not nums:
            return 0
        if len(nums) == 1:
            return nums[0]
            
        # rob1 represents dp[i-2], rob2 represents dp[i-1]
        rob1, rob2 = 0, 0
        
        for num in nums:
            new_rob = max(rob2, rob1 + num)
            rob1 = rob2
            rob2 = new_rob
            
        return rob2
```

---

## 3. Benchmark Problem 2: LeetCode 322 — Coin Change

### 3.1 Problem Statement Breakdown & Line-by-Line Annotations
> *"You are given an integer array `coins` representing coins of different denominations and an integer `amount` representing a total amount of money."*
- Denominations array `coins`, target `amount`.

> *"Return the fewest number of coins that you need to make up that amount. If that amount of money cannot be made up by any combination of the coins, return `-1`."*
- Optimization: Minimize total coins.
- Feasibility check: Return $-1$ if impossible.
- Each denomination is available in **infinite quantity** (Unbounded / 1D state on `amount`).

---

### 3.2 Constraints & Complexity Analysis
- $1 \le \text{coins.length} \le 12$
- $1 \le \text{coins}[i] \le 2^{31} - 1$
- $0 \le \text{amount} \le 10,000$
- Target Time: $O(A \times C)$ where $A = \text{amount}$ and $C = \text{len(coins)} \implies 10,000 \times 12 = 1.2 \times 10^5$ operations (instantaneous).
- Target Space: $O(A)$ for 1D DP table.

---

### 3.3 Recurrence Relation & Base Cases
Let $\text{dp}[a]$ be the minimum number of coins needed to make sum $a$:
$$\text{dp}[a] = \min_{c \in \text{coins}, c \le a} (\text{dp}[a - c] + 1)$$

**Base Case**:
- $\text{dp}[0] = 0$ (0 coins needed to make sum 0).
- $\text{dp}[a] = \infty$ for $a > 0$ initially.

---

### 3.4 Complete Python Implementation (Bottom-Up Tabulation)

```python
class Solution:
    def coinChange(self, coins: list[int], amount: int) -> int:
        # Initialize dp array with infinity
        dp = [float('inf')] * (amount + 1)
        dp[0] = 0  # Base case: 0 coins for amount 0
        
        for a in range(1, amount + 1):
            for c in coins:
                if a - c >= 0:
                    dp[a] = min(dp[a], dp[a - c] + 1)
                    
        return dp[amount] if dp[amount] != float('inf') else -1
```

- **Time Complexity**: $O(\text{amount} \times |\text{coins}|)$.
- **Space Complexity**: $O(\text{amount})$ — 1D array of size $\text{amount} + 1$.

---

### 3.5 Live Verbalization Script

> *"For Coin Change, we want to find the minimum number of coins that sum to `amount`. Because each coin can be used infinitely many times, the subproblem state depends strictly on the remaining amount.
> 
> I define `dp[a]` as the minimum coins needed to make amount `a`. The base case is `dp[0] = 0`, and all other amounts are initialized to infinity.
> 
> For every amount from $1$ to `amount`, I iterate over each coin denomination `c`. If `a - c >= 0`, using coin `c` leaves a subproblem of `a - c`. Thus, `dp[a] = min(dp[a], dp[a - c] + 1)`.
> 
> After filling the array, if `dp[amount]` remains infinity, it is impossible to form the target and I return $-1$; otherwise, I return `dp[amount]`.
> 
> This runs in $O(\text{amount} \times |\text{coins}|)$ time and consumes $O(\text{amount})$ space."*
