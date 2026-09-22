# Domain 07: Dynamic Programming — Unbounded Knapsack & Coin Change II

---

## 1. Unbounded Knapsack Formulation

In the **Unbounded Knapsack** problem, an item can be chosen an unlimited number of times.

### 0/1 vs. Unbounded Recurrence Comparison
- **0/1 Knapsack**: When item $i$ is taken, the next decision is made on the remaining items $i - 1$:
  $$\text{dp}[i][c] = \max(\text{dp}[i-1][c], \text{dp}[i-1][c - w_i] + v_i)$$
- **Unbounded Knapsack**: When item $i$ is taken, item $i$ remains available for subsequent picks:
  $$\text{dp}[i][c] = \max(\text{dp}[i-1][c], \text{dp}[i][c - w_i] + v_i)$$

Notice the transition index in the second term: $\text{dp}[i][\dots]$ instead of $\text{dp}[i-1][\dots]$.

---

## 2. The 1D Forward Iteration Invariant

Because the subproblem relies on the **current row's** updated values, compressing the 2D table to 1D requires iterating capacity $c$ **forward** from $w \to W$.

```
Forward Loop:
for c in range(coin, amount + 1):
    dp[c] += dp[c - coin]  # Uses the newly updated value dp[c - coin] of the SAME item!
```

---

## 3. Combinations vs. Permutations: Loop Order Matters

A frequent pitfall in counting DP is distinguishing between **unordered combinations** and **ordered permutations**:

```
Problem: Count ways to form amount 4 using coins [1, 2].
Combinations: [1, 1, 1, 1], [1, 1, 2], [2, 2] -> 3 ways (Coin Change II)
Permutations: [1, 1, 2], [1, 2, 1], [2, 1, 1], ... -> 5 ways (Combination Sum IV)
```

| Desired Result | Outer Loop | Inner Loop | Why |
| :--- | :--- | :--- | :--- |
| **Combinations** (Order does not matter) | `for coin in coins:` | `for a in range(coin, amount + 1):` | Enforces non-decreasing coin indices, eliminating duplicates like `[1, 2]` vs `[2, 1]`. |
| **Permutations** (Order matters) | `for a in range(1, amount + 1):` | `for coin in coins:` | Every coin denomination is tried at every sequence step, generating all orderings. |

---

## 4. Benchmark Problem Deep Dive: LeetCode 518 — Coin Change II

### 4.1 Problem Statement Breakdown & Line-by-Line Annotations
> *"You are given an integer array `coins` representing coins of different denominations and an integer `amount` representing a total amount of money."*
- Denominations list `coins`, target `amount`.

> *"Return the number of combinations that make up that amount. If that amount of money cannot be made up by any combination of the coins, return 0."*
- Counting problem: Combinations only (different orderings of the same coins are not counted as distinct).
- Unlimited supply of each coin $\implies$ Unbounded Knapsack.

---

### 4.2 Constraints & Complexity Analysis
- $1 \le \text{coins.length} \le 300$
- $1 \le \text{coins}[i] \le 5,000$
- $0 \le \text{amount} \le 5,000$
- Target Time: $O(N \times A)$ where $N = \text{len(coins)}$ and $A = \text{amount} \implies 300 \times 5,000 = 1.5 \times 10^6$ ops (runs in ~20 ms).
- Target Space: $O(A) = 5,000$ integers (1D space-optimized).

---

### 4.3 Dry Run Trace Table

Consider `amount = 5, coins = [1, 2, 5]`:
- Initial: `dp = [1, 0, 0, 0, 0, 0]`

| Coin | Loop $a \in [\text{coin}, 5]$ | Updated `dp` Array `[0, 1, 2, 3, 4, 5]` |
| :--- | :--- | :--- |
| Init | — | `[1, 0, 0, 0, 0, 0]` |
| 1 | $a \in [1, 5]$: `dp[a] += dp[a - 1]` | `[1, 1, 1, 1, 1, 1]` |
| 2 | `a = 2: dp[2] += dp[0]` $\implies 2$<br>`a = 3: dp[3] += dp[1]` $\implies 2$<br>`a = 4: dp[4] += dp[2]` $\implies 3$<br>`a = 5: dp[5] += dp[3]` $\implies 3$ | `[1, 1, 2, 2, 3, 3]` |
| 5 | `a = 5: dp[5] += dp[0]` $\implies 3 + 1 = 4$ | `[1, 1, 2, 2, 3, 4]` |

Result: `dp[5] = 4`. The 4 combinations are:
1. `5`
2. `2 + 2 + 1`
3. `2 + 1 + 1 + 1`
4. `1 + 1 + 1 + 1 + 1`

---

### 4.4 Complete Python Implementations

#### Top-Down (Memoization)
```python
from functools import lru_cache

class SolutionTopDown:
    def change(self, amount: int, coins: list[int]) -> int:
        n = len(coins)
        
        @lru_cache(maxsize=None)
        def dp(i: int, rem: int) -> int:
            if rem == 0:
                return 1
            if rem < 0 or i >= n:
                return 0
                
            # Choice 1: Take current coin and stay at index i (unbounded reuse)
            take = dp(i, rem - coins[i])
            # Choice 2: Skip current coin and move to next index i + 1
            skip = dp(i + 1, rem)
            
            return take + skip
            
        return dp(0, amount)
```

#### Bottom-Up (1D Space-Optimized)
```python
class Solution:
    def change(self, amount: int, coins: list[int]) -> int:
        # dp[a] represents the number of combinations to make amount a
        dp = [0] * (amount + 1)
        dp[0] = 1  # Base case: 1 combination to make amount 0 (the empty set)
        
        # Outer loop over coins ensures combinations, not permutations
        for coin in coins:
            # Inner loop forwards from coin to amount allows unbounded reuse
            for a in range(coin, amount + 1):
                dp[a] += dp[a - coin]
                
        return dp[amount]
```

- **Time Complexity**: $O(\text{len(coins)} \times \text{amount})$.
- **Space Complexity**: $O(\text{amount})$.

---

## 5. Classic Unbounded Knapsack: Rod Cutting Problem

Given a rod of length $N$ and an array of prices for pieces of lengths $1$ to $N$, find the maximum revenue obtainable by cutting up the rod.

```python
def rod_cutting(price: list[int], n: int) -> int:
    # dp[l] stores max profit for a rod of length l
    dp = [0] * (n + 1)
    
    for l in range(1, n + 1):
        max_val = 0
        for i in range(1, l + 1):
            max_val = max(max_val, price[i - 1] + dp[l - i])
        dp[l] = max_val
        
    return dp[n]
```

---

### 5.1 Live Verbalization Script

> *"For Coin Change II, we need to find the number of unique combinations of coins that sum to `amount`, where each coin denomination can be reused without limit.
> 
> Because we are counting combinations rather than permutations, the order in which coins are considered matters. To avoid duplicate sequences like `[1, 2]` and `[2, 1]`, I place the loop over coins on the outside and the loop over amounts on the inside.
> 
> I initialize a 1D DP array of size `amount + 1` with zeros, setting `dp[0] = 1` because there is exactly one way to make sum 0 (using no coins).
> 
> For each coin denomination, I iterate forward through amounts from `coin` up to `amount`. The forward iteration allows the same coin to be used multiple times, directly modeling the Unbounded Knapsack property. For each amount `a`, I add `dp[a - coin]` to `dp[a]`.
> 
> Finally, `dp[amount]` contains the total number of combinations.
> 
> The time complexity is $O(N \times \text{amount})$ where $N$ is the number of coins, and the space complexity is $O(\text{amount})$."*

---

## 6. Benchmark Problem Deep Dive: LeetCode 377 — Combination Sum IV (Ordered Sequences)

### 6.1 Problem Statement Breakdown & Line-by-Line Annotations
> *"Given an array of distinct integers `nums` and a target integer `target`, return the number of possible combinations that add up to `target`."*
- *Crucial note on problem nomenclature*: Despite the name "Combination Sum IV", the problem explicitly considers different orderings as **distinct sequences** (e.g., `(1, 2)` and `(2, 1)` both count). Thus, mathematically, this is counting **Permutations with Replacement**.

---

### 6.2 The Loop Order Invariant: Coin Change II vs. Combination Sum IV

| Problem | Goal | Outer Loop | Inner Loop | Core Recurrence |
| :--- | :--- | :--- | :--- | :--- |
| **LC 518 (Coin Change II)** | Unordered Combinations | `for coin in coins:` | `for a in range(coin, target + 1):` | `dp[a] += dp[a - coin]` |
| **LC 377 (Comb Sum IV)** | Ordered Permutations | `for a in range(1, target + 1):` | `for num in nums:` | `dp[a] += dp[a - num]` |

**Why reversing loop order switches combinations to permutations:**
- In LC 518, processing coin 1 completely before coin 2 ensures that 2 can never precede 1, eliminating `[2, 1]`.
- In LC 377, for any given sum `a`, *every* candidate `num` is tested as the possible final step to reach `a`. This considers both ending in 1 after 2 (`2 + 1 = 3`) and ending in 2 after 1 (`1 + 2 = 3`), capturing all orderings.

---

### 6.3 Complete Python Implementation

```python
class SolutionCombSumIV:
    def combinationSum4(self, nums: list[int], target: int) -> int:
        # dp[a] stores count of ordered sequences that sum to a
        dp = [0] * (target + 1)
        dp[0] = 1  # Base case: 1 way to form sum 0 (empty sequence)
        
        # Outer loop over amounts: every coin can be the last step to form amount a
        for a in range(1, target + 1):
            for num in nums:
                if a - num >= 0:
                    dp[a] += dp[a - num]
                    
        return dp[target]
```

- **Time Complexity**: $O(\text{target} \times |\text{nums}|)$.
- **Space Complexity**: $O(\text{target})$.

---

### 6.4 Live Verbalization Script

> *"Although LeetCode 377 is titled 'Combination Sum IV', the problem states that different orderings are counted as distinct, making it an ordered permutation counting problem.
> 
> In contrast to Coin Change II (where coins are iterated in the outer loop to enforce a canonical non-decreasing order), here I iterate over the target sum `a` in the outer loop, and iterate over all numbers in the inner loop.
> 
> For each sum `a` from 1 to `target`, any number `num <= a` can serve as the final element in a sequence summing to `a`. Therefore, `dp[a]` equals the sum of `dp[a - num]` across all valid numbers.
> 
> With `dp[0] = 1`, this computes all permutations in $O(\text{target} \times |\text{nums}|)$ time with $O(\text{target})$ space."*

