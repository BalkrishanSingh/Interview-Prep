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

---

## 4. Benchmark Problem 3: LeetCode 300 — Longest Increasing Subsequence (LIS)

### 4.1 Problem Statement Breakdown & Line-by-Line Annotations
> *"Given an integer array `nums`, return the length of the longest strictly increasing subsequence."*
- Subsequence elements do not need to be contiguous, but their relative order is preserved.
- **Strictly increasing**: $nums[i_1] < nums[i_2] < \dots < nums[i_k]$ where $i_1 < i_2 < \dots < i_k$.

---

### 4.2 The Patience Sorting / Binary Search Invariant ($O(N \log N)$ Time)

While standard 1D DP yields $O(N^2)$ via $\text{dp}[i] = 1 + \max_{j < i, \text{nums}[j] < \text{nums}[i]} \text{dp}[j]$, interviews expect the optimal $O(N \log N)$ algorithm using **Patience Sorting**:

Maintain an array `tails` where:
- `tails[len]` is the **smallest tail element** among all increasing subsequences of length `len + 1` found so far.

**Why is `tails` strictly sorted?**
- If an increasing subsequence of length $k + 1$ exists ending in value $X$, its prefix of length $k$ ends in some value $Y < X$. Therefore, $\text{tails}[k] < \text{tails}[k + 1]$ always holds.
- Because `tails` is monotonically increasing, we can use **Binary Search** (`bisect_left`) to find the first element $\ge \text{num}$ in $O(\log N)$ time.

**Update Rule for incoming number `x`**:
1. If `x` is greater than all elements in `tails`: append `x` to `tails` (extends the maximum LIS length by 1).
2. Otherwise: replace the first element in `tails` that is $\ge x$ with `x` (lowers the tail barrier for future elements, increasing chances of extending longer subsequences later).

---

### 4.3 Complete Python Implementation ($O(N \log N)$ Optimal)

```python
from bisect import bisect_left

class SolutionLIS:
    def lengthOfLIS(self, nums: list[int]) -> int:
        if not nums:
            return 0
            
        tails = []
        
        for x in nums:
            # Find insertion point: first element in tails >= x
            idx = bisect_left(tails, x)
            
            if idx == len(tails):
                tails.append(x)
            else:
                tails[idx] = x
                
        return len(tails)
```

- **Time Complexity**: $O(N \log N)$ — $N$ elements, each performing a binary search in $O(\log N)$.
- **Space Complexity**: $O(N)$ — The `tails` array stores at most $N$ values.

---

### 4.4 Live Verbalization Script

> *"To find the Longest Increasing Subsequence length in $O(N \log N)$ time, I use the Patience Sorting algorithm with binary search.
> 
> I maintain a dynamic array `tails`, where `tails[i]` stores the minimum ending value among all discovered increasing subsequences of length `i + 1`. 
> 
> Because an increasing subsequence of length $k+1$ must have a prefix of length $k$ ending with a strictly smaller value, the `tails` array is guaranteed to remain strictly sorted at all times.
> 
> For each number in `nums`, I use `bisect_left` to locate the first element in `tails` greater than or equal to `x`. If `x` is larger than all current tails, it extends our longest subsequence, so I append it. Otherwise, I replace `tails[idx]` with `x`. This greedy replacement does not alter the length of subsequences found so far, but minimizes the tail value, optimizing future potential extensions.
> 
> The final length of `tails` equals the maximum LIS length. This runs in $O(N \log N)$ time and uses $O(N)$ auxiliary memory."*

---

## 5. Benchmark Problem 4: LeetCode 91 — Decode Ways

### 5.1 Problem Statement Breakdown & Line-by-Line Annotations
> *"A message containing letters from A-Z can be encoded into numbers using the mapping 'A' -> '1', 'B' -> '2', ... 'Z' -> '26'."*
- 1-digit mapping: '1' through '9' are valid ('0' by itself is INVALID).
- 2-digit mapping: '10' through '26' are valid.

> *"Given a string `s` containing only digits, return the number of ways to decode it."*

---

### 5.2 Transition Logic & Space-Optimized DP

Let $\text{dp}[i]$ represent the number of ways to decode prefix $s[0 \dots i-1]$:
1. **Single Digit Check**:
   If $s[i-1] \neq '0'$, we can decode $s[i-1]$ as a single character:
   $$\text{dp}[i] = \text{dp}[i] + \text{dp}[i-1]$$
2. **Two Digit Check**:
   If $s[i-2 \dots i-1] \in [10, 26]$, we can decode the 2-digit pair as a single character:
   $$\text{dp}[i] = \text{dp}[i] + \text{dp}[i-2]$$

Because $\text{dp}[i]$ depends only on $\text{dp}[i-1]$ and $\text{dp}[i-2]$, we can compress the table to $O(1)$ space using two variables.

---

### 5.3 Complete Python Implementation ($O(1)$ Space)

```python
class SolutionDecodeWays:
    def numDecodings(self, s: str) -> int:
        if not s or s[0] == '0':
            return 0
            
        n = len(s)
        # prev2 represents dp[i-2], prev1 represents dp[i-1]
        prev2 = 1  # Empty string base case dp[0] = 1
        prev1 = 1  # First valid character dp[1] = 1
        
        for i in range(2, n + 1):
            curr = 0
            one_digit = int(s[i - 1:i])
            two_digits = int(s[i - 2:i])
            
            # Single digit decode
            if 1 <= one_digit <= 9:
                curr += prev1
                
            # Two digits decode
            if 10 <= two_digits <= 26:
                curr += prev2
                
            prev2 = prev1
            prev1 = curr
            
        return prev1
```

- **Time Complexity**: $O(N)$ — Single pass through string of length $N$.
- **Space Complexity**: $O(1)$ — Rolling variables only.

---

### 5.4 Live Verbalization Script

> *"For Decode Ways, this is a linear dynamic programming problem where each position has at most two branching transitions: decoding a single digit or decoding a two-digit number.
> 
> The base cases are: an empty string has 1 decoding, and a string starting with '0' has 0 decodings.
> 
> For each index from 2 to $N$:
> If the current single digit is between 1 and 9, it contributes all decodings from `prev1` (the state without this digit).
> If the two-digit substring formed with the previous digit falls within 10 to 26, it contributes all decodings from `prev2` (the state without both digits).
> 
> Because each step requires only the preceding two states, I maintain two rolling integer variables, achieving $O(N)$ time with $O(1)$ auxiliary space."*

