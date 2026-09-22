# Domain 02: Binary Search — Binary Search on Answer Space

---

## 1. The "Binary Search on Answer" Mental Model

When a problem asks for the **minimum** or **maximum** value of a variable $k$ satisfying a condition, and direct computation of $k$ is complex, we invert the problem:
Instead of computing the answer directly, we **guess an answer $k$ and verify its feasibility via a boolean predicate function $P(k)$**.

```
MONOTONIC FEASIBILITY PREDICATE P(k):
Speed k:    1       2       3       4       5       6       7       8
Feasible: [False,  False,  False,  True,   True,   True,   True,   True]
                                     ▲
                              Optimal Minimum k
```

### The Invariant:
If $P(k)$ is monotonic:
- If speed $k$ is feasible ($P(k) = \text{True}$), any speed greater than $k$ is also feasible. The minimum possible speed must lie in $[ \text{low}, k ]$.
- If speed $k$ is infeasible ($P(k) = \text{False}$), no speed less than or equal to $k$ can possibly be feasible. The answer must lie in $[ k + 1, \text{high} ]$.

---

## 2. Reusable Python Boilerplate Template

```python
def binary_search_on_answer_template(low: int, high: int) -> int:
    """
    Template for finding minimum k where condition(k) == True.
    """
    def is_feasible(k: int) -> bool:
        # User-defined feasibility check: O(N)
        return condition_satisfied(k)
        
    ans = high
    while low <= high:
        mid = low + (high - low) // 2
        if is_feasible(mid):
            ans = mid         # Candidate answer found: try smaller
            high = mid - 1
        else:
            low = mid + 1     # Infeasible: must increase k
            
    return ans
```

---

## 3. Benchmark Problem Deep Dive: LeetCode 875 — Koko Eating Bananas

### 3.1 Problem Statement Breakdown & Line-by-Line Annotations
> *"Koko loves to eat bananas. There are `n` piles of bananas, the $i$-th pile has `piles[i]` bananas."*
- `piles = [p_0, p_1, \dots, p_{n-1}]`.
- In 1 hour, Koko eats at most $k$ bananas from a single chosen pile.

> *"The guards have gone and will come back in `h` hours."*
- Total time constraint: $\text{Total Hours} \le h$.
- Since $h \ge \text{piles.length}$, Koko spends at least 1 hour on every pile ($n$ piles require at least $n$ hours).

> *"If the pile has less than `k` bananas, she eats all of them instead and will not eat any more bananas during this hour."*
- Crucial rule: Bananas cannot be combined across piles in the same hour!
- Hours required to finish pile $p$: $\lceil p / k \rceil = \lfloor (p + k - 1) / k \rfloor$.

> *"Return the minimum integer `k` such that she can eat all the bananas within `h` hours."*
- Goal: $\min k$ such that $\sum_{i=0}^{n-1} \lceil \text{piles}[i] / k \rceil \le h$.

---

### 3.2 Constraints & Complexity Analysis
- $1 \le \text{piles.length} \le 10^4$
- $\text{piles.length} \le h \le 10^9$
- $1 \le \text{piles}[i] \le 10^9$
- **Bounds on $k$**:
  - Minimum possible speed: $\text{low} = 1$ (eating 1 banana/hour).
  - Maximum possible speed: $\text{high} = \max(\text{piles}) = 10^9$ (finishes any pile in exactly 1 hour).
  - Search Space Size: $10^9$.
  - Linear scan: $10^9 \times 10^4 = 10^{13}$ operations $\implies$ **Definite TLE**.
  - Binary Search: $\log_2(10^9) \approx 30$ iterations. Total operations: $30 \times 10^4 = 3 \times 10^5 \implies$ **Under 5 milliseconds**!

---

### 3.3 Direction Exploration & Invariant Proof
- **Feasibility Function**:
  ```python
  def hours_needed(k: int) -> int:
      return sum((p + k - 1) // k for p in piles)
  ```
- **Monotonicity**: As speed $k$ increases, $\lceil p / k \rceil$ is monotonically non-increasing. Thus, the total hours required is monotonically decreasing.
- This strict monotonicity enables binary search on answer space $[1, \max(\text{piles})]$.

---

### 3.4 Step-by-Step Dry Run Trace Table
Input: `piles = [3, 6, 7, 11]`, `h = 8`
Search Space: $\text{low} = 1, \text{high} = 11$

| Iteration | `low` | `high` | `mid` ($k$) | Hours per pile $\lceil p / k \rceil$ | Total Hours | Feasible? ($\le 8$) | Action |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | 1 | 11 | 6 | $\lceil 3/6 \rceil=1, \lceil 6/6 \rceil=1, \lceil 7/6 \rceil=2, \lceil 11/6 \rceil=2$ | $1+1+2+2 = \mathbf{6}$ | **Yes** ($6 \le 8$) | Record $ans=6$, try smaller: `high = 5` |
| 2 | 1 | 5 | 3 | $\lceil 3/3 \rceil=1, \lceil 6/3 \rceil=2, \lceil 7/3 \rceil=3, \lceil 11/3 \rceil=4$ | $1+2+3+4 = \mathbf{10}$ | **No** ($10 > 8$) | Too slow: `low = mid + 1 = 4` |
| 3 | 4 | 5 | 4 | $\lceil 3/4 \rceil=1, \lceil 6/4 \rceil=2, \lceil 7/4 \rceil=2, \lceil 11/4 \rceil=3$ | $1+2+2+3 = \mathbf{8}$ | **Yes** ($8 \le 8$) | Record $ans=4$, try smaller: `high = 3` |
| 4 | 4 | 3 | - | Loop terminates (`low > high`) | - | - | **Return ans = 4** |

---

### 3.5 Complete Python Implementation

```python
class Solution:
    def minEatingSpeed(self, piles: list[int], h: int) -> int:
        low = 1
        high = max(piles)
        ans = high
        
        while low <= high:
            mid = low + (high - low) // 2
            
            # Compute total hours at eating speed 'mid'
            total_hours = 0
            for p in piles:
                # Integer ceiling division: ceil(p / mid) == (p + mid - 1) // mid
                total_hours += (p + mid - 1) // mid
                
            if total_hours <= h:
                ans = mid         # Feasible speed, search for smaller
                high = mid - 1
            else:
                low = mid + 1     # Infeasible (too slow), increase speed
                
        return ans
```

- **Time Complexity**: $O(N \log(\max(\text{piles})))$ — Binary search takes $O(\log(\max P))$ steps, each calculating hours in $O(N)$.
- **Space Complexity**: $O(1)$ — In-place computation.

---

### 3.6 Live Verbalization Script

> *"We are asked to find the minimum integer speed $k$ to consume all banana piles within $h$ hours.
> 
> Directly calculating $k$ is non-trivial, but checking if a given speed $k$ is feasible takes only $O(N)$ time by summing $\lceil \text{pile} / k \rceil$.
> 
> Furthermore, feasibility is monotonic: if Koko can finish at speed $k$, she can also finish at any speed greater than $k$. If she cannot finish at speed $k$, any slower speed will also fail.
> 
> Therefore, we can binary search over the answer space of possible speeds, where the lower bound is 1 and the upper bound is $\max(\text{piles})$.
> 
> For each candidate speed `mid`, we compute the total hours. If total hours $\le h$, we record `mid` as a valid candidate and search the lower half for a smaller speed. Otherwise, we search the upper half.
> 
> This reduces an intractable $O(N \cdot M)$ search down to $O(N \log M)$ time, executing in under 30 iterations for values up to $10^9$."*
