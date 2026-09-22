# Domain 09: Monotonic Structures — Monotonic Queue & Sliding Window Maximum

---

## 1. Monotonic Queue Mental Model

A standard queue supports FIFO operations in $O(1)$. A **Monotonic Queue** (implemented via a doubly-ended queue / `deque`) maintains elements in strictly decreasing order of value while preserving their temporal arrival order:
- The element at the **front of the deque** is guaranteed to be the **maximum element** in the active sliding window.
- Operations take $O(1)$ amortized time.

---

## 2. Benchmark Problem: LeetCode 239 — Sliding Window Maximum

### 2.1 Problem Statement Breakdown & Line-by-Line Annotations
> *"You are given an array of integers `nums`, there is a sliding window of size `k` which is moving from the very left of the array to the very right."*
- Window bounds: $[i - k + 1, i]$.

> *"You can only see the `k` numbers in the window. Each time the sliding window moves right by one position."*
- Total windows: $N - k + 1$.

> *"Return the max sliding window."*
- A naive heap approach runs in $O(N \log k)$ or $O(N \log N)$.
- We must achieve the optimal **$O(N)$ linear time**.

---

### 2.2 The Two Deque Invariants

We store **indices** in the deque:

1. **Window Expiration Invariant (Front of Deque)**:
   Any index in the deque that falls outside the current sliding window boundary ($idx \le right - k$) must be popped from the **left** (`popleft()`).

2. **Dominance / Monotonicity Invariant (Back of Deque)**:
   Before inserting incoming element $nums[right]$, pop all indices from the **right** (`pop()`) whose values are $\le nums[right]$.
   - *Dominance Proof*: An existing element $x \le nums[right]$ is both smaller and entered the window earlier. It will expire before $nums[right]$ and can never serve as the maximum for any current or future window. Therefore, it is permanently obsolete.

3. **Query Invariant**:
   After maintaining the two invariants, `nums[deque[0]]` is the guaranteed maximum for window ending at `right`.

---

### 2.3 Dry Run Trace Table

Consider `nums = [1, 3, -1, -3, 5, 3, 6, 7]`, $k = 3$:

| Step $r$ | `nums[r]` | Evict Stale (< $r - k + 1$) | Evict Dominated ($\le nums[r]$) | Deque (Indices) | Deque (Values) | Result Added ($r \ge k-1$) |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 0 | 1 | — | None | `[0]` | `[1]` | — |
| 1 | 3 | — | Pop 0 ($1 \le 3$) | `[1]` | `[3]` | — |
| 2 | -1 | — | None | `[1, 2]` | `[3, -1]` | **3** |
| 3 | -3 | None | None | `[1, 2, 3]` | `[3, -1, -3]` | **3** |
| 4 | 5 | Pop 1 ($1 \le 1$) | Pop 3 (-3 $\le$ 5), Pop 2 (-1 $\le$ 5) | `[4]` | `[5]` | **5** |
| 5 | 3 | None | None | `[4, 5]` | `[5, 3]` | **5** |
| 6 | 6 | None | Pop 5 (3 $\le$ 6), Pop 4 (5 $\le$ 6) | `[6]` | `[6]` | **6** |
| 7 | 7 | None | Pop 6 (6 $\le$ 7) | `[7]` | `[7]` | **7** |

Output: `[3, 3, 5, 5, 6, 7]`.

---

### 2.4 Complete Python Implementation ($O(N)$ Time)

```python
from collections import deque

class SolutionSlidingWindowMax:
    def maxSlidingWindow(self, nums: list[int], k: int) -> list[int]:
        q = deque()  # Stores indices in strictly decreasing order of values
        result = []
        
        for r, num in enumerate(nums):
            # 1. Remove indices that are out of current window bounds
            if q and q[0] <= r - k:
                q.popleft()
                
            # 2. Maintain decreasing order: remove smaller elements from back
            while q and nums[q[-1]] <= num:
                q.pop()
                
            # 3. Add current index
            q.append(r)
            
            # 4. Record window maximum once the first full window is reached
            if r >= k - 1:
                result.append(nums[q[0]])
                
        return result
```

- **Time Complexity**: $O(N)$ — Every element is appended to the deque once and popped at most once.
- **Space Complexity**: $O(k)$ — Deque stores at most $k$ indices at any time.

---

### 2.5 Live Verbalization Script

> *"To find the maximum in each sliding window in linear $O(N)$ time, I use a monotonically decreasing double-ended queue (`deque`) storing array indices.
> 
> The front of the deque `deque[0]` will always represent the maximum element for the current window.
> 
> As I scan through each index `r`:
> First, I evict indices from the left that have fallen outside the window: `if q[0] <= r - k: q.popleft()`.
> 
> Second, before appending the incoming number `num`, I pop all indices from the right whose values are less than or equal to `num`. This is because any smaller element located to the left of `num` is permanently dominated: it will expire sooner and is smaller than `num`, so it can never be the maximum in this or any future window.
> 
> Finally, once `r >= k - 1`, I append `nums[q[0]]` to the output.
> 
> This processes each element in $O(1)$ amortized time, yielding an overall $O(N)$ time complexity and $O(k)$ auxiliary space."*
