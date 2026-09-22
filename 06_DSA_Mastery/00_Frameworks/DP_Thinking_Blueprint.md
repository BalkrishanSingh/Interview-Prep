# Framework: Dynamic Programming (DP) Thinking Blueprint

---

## 1. The Core DP Mental Model

Dynamic Programming applies strictly when a problem exhibits two mathematical properties:
1. **Optimal Substructure**: The optimal solution to the global problem incorporates optimal solutions to its constituent subproblems.
2. **Overlapping Subproblems**: The recursive search tree evaluates identical subproblems repeatedly with identical state parameters.

```
                    f(4)
               /            \
           f(3)              f(2)
          /    \            /    \
      f(2)      f(1)     f(1)    f(0)
     /    \
  f(1)    f(0)
  
  Notice: f(2), f(1), and f(0) are computed repeatedly!
  Pure Recursion: O(2^N) exponential time.
  With Memoization: O(N) linear time, evaluating each unique state once.
```

---

## 2. The 8-Step Problem Solving Blueprint (Line-by-Line Dissection)

Whenever encountering an optimization (min/max), counting (number of ways), or feasibility (can we reach target) problem:

### Step 1: Line-by-Line Problem Dissection
Read the problem statement and annotate every sentence:
- What does each input array or matrix element represent?
- What are the allowable actions / transitions at step $i$?
- What constitutes the starting state and the goal/termination state?
- Are we minimizing, maximizing, counting, or checking boolean feasibility?

### Step 2: Constraint & Complexity Envelope
Analyze the mathematical bounds:
- $N \le 20 \implies O(2^N)$ or $O(N!)$ backtracking is viable.
- $N \le 100 \implies O(N^3)$ or $O(N^4)$ 2D/3D DP viable.
- $N \le 1,000 \to 2,000 \implies O(N^2)$ 2D DP viable ($10^6$ operations).
- $N \le 100,000 \to 10^6 \implies O(N \log N)$ or $O(N)$ 1D DP required ($10^8$ operations threshold).

### Step 3: Decision & State Space Exploration
- What state parameters uniquely identify a subproblem? (e.g., index $i$, remaining capacity $w$).
- What decisions / choices are available at state $S$?
- Draw the recursive decision branch:
  - Choice 1: Take element $\to$ new state $(i+1, w - \text{weight}[i])$
  - Choice 2: Skip element $\to$ new state $(i+1, w)$

### Step 4: Recurrence Relation & Base Cases
Formulate the mathematical equation:
- **Base Case(s)**: Out-of-bounds index, zero capacity, target reached.
- **Recurrence**:
  $$\text{dp}(i, w) = \min / \max / \sum (\text{action choices})$$

### Step 5: Summary & Invariants
Summarize the subproblem invariant in one sentence:
*"Let $\text{dp}[i][w]$ represent the maximum value achievable considering items from index $i$ to $N-1$ with remaining capacity $w$."*

### Step 6: Top-Down Implementation (Recursion + Memoization)
Write the clean recursive solution using Python's `@functools.lru_cache(None)` or an explicit memo dictionary/table.

### Step 7: Bottom-Up Tabulation (Iterative Transition)
Convert top-down into an iterative loop:
- Identify table dimensions: 1D array of size $N+1$, or 2D array of size $(N+1) \times (W+1)$.
- Identify filling direction: Base cases $\to$ Target state (e.g., small index to large, or right-to-left).
- Replace recursive call $\text{solve}(i+1)$ with table lookup $\text{dp}[i+1]$.

### Step 8: Space Optimization
If $\text{dp}[i]$ depends only on $\text{dp}[i-1]$ and $\text{dp}[i-2]$:
- Reduce table from $O(N)$ to $O(1)$ space using two rolling variables (`prev1`, `prev2`).
If $\text{dp}[i][w]$ depends only on row $i-1$:
- Reduce table from $O(N \times W)$ to $O(W)$ using 1D array traversed backwards.

---

## 3. Top-Down & Bottom-Up Python Templates

### 3.1 Generic Top-Down (Memoization) Template
```python
from functools import lru_cache

def solve_dp_topdown(arr: list[int]) -> int:
    n = len(arr)
    
    @lru_cache(maxsize=None)
    def dp(index: int, state_var: int) -> int:
        # 1. Base Case: Terminal condition
        if index >= n:
            return 0  # Or infinity / -infinity depending on min/max
        
        # 2. Explore Actions / Transitions
        # Choice A: Take action 1
        res1 = arr[index] + dp(index + 1, state_var - arr[index])
        
        # Choice B: Take action 2
        res2 = dp(index + 1, state_var)
        
        # 3. Combine Choices (Optimize)
        return max(res1, res2)
    
    return dp(0, initial_state)
```

### 3.2 Generic Bottom-Up (Tabulation) Template
```python
def solve_dp_bottomup(arr: list[int], target: int) -> int:
    n = len(arr)
    # 1. Initialize DP table with identity values (0, float('inf'), or float('-inf'))
    dp = [0] * (target + 1)
    
    # 2. Set Base Case
    dp[0] = 0
    
    # 3. Iterative State Transition (Loop order ensures dependencies are precomputed)
    for num in arr:
        for w in range(target, num - 1, -1):  # Reverse for 0/1 knapsack
            dp[w] = max(dp[w], dp[w - num] + value)
            
    return dp[target]
```

---

## 4. Live Verbalization Framework

When solving a Dynamic Programming problem in front of an evaluator:

1. **Clarify Constraints & Verify Brute Force**:
   > *"Given that $N \le 1000$, a brute force recursive branching search generates $O(2^N)$ subproblems, which will exceed our compute budget and time out. However, notice that subproblems overlap—for instance, reaching index $i$ with remaining capacity $w$ can be arrived at from multiple different decision paths."*
2. **Define State Precisely**:
   > *"I will define my DP state $\text{dp}(i)$ as the minimum cost required to reach the top starting from index $i$."*
3. **State Recurrence Relation**:
   > *"At state $i$, I have two valid actions: climb 1 step or climb 2 steps. Therefore: $\text{dp}(i) = \text{cost}[i] + \min(\text{dp}(i+1), \text{dp}(i+2))$."*
4. **Identify Base Cases**:
   > *"The base case occurs when $i \ge N$. At this boundary, we have already reached or passed the top, requiring 0 additional cost."*
5. **Propose Space Optimization**:
   > *"Because calculating $\text{dp}[i]$ only requires values from the immediate two subsequent states ($i+1$ and $i+2$), we can optimize auxiliary space from $O(N)$ down to $O(1)$ using two variables."*
