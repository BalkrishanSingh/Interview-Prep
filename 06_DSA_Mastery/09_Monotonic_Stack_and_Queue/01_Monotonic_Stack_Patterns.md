# Domain 09: Monotonic Structures — Monotonic Stack Patterns

---

## 1. Monotonic Stack Mental Model

A **Monotonic Stack** is a stack data structure whose elements are maintained in strictly monotonic order (either monotonically increasing or monotonically decreasing).

Whenever an incoming element violates the monotonicity invariant, elements are popped from the stack until the invariant is restored.

| Stack Type | Elements Inside Stack | Popped When | Solves Problem Archetype |
| :--- | :--- | :--- | :--- |
| **Monotonically Decreasing** | High $\to$ Low | Incoming element > `stack[-1]` | **Next Greater Element**, Warmer temperatures. |
| **Monotonically Increasing** | Low $\to$ High | Incoming element < `stack[-1]` | **Next Smaller Element**, Histogram rectangle width limits. |

### The $O(N)$ Amortized Complexity Invariant
Although a single iteration may pop up to $N$ elements, **each element is pushed onto the stack exactly once and popped at most once**. The total number of push and pop operations across the entire algorithm is at most $2N$, guaranteeing strict $O(N)$ total execution time.

---

## 2. Benchmark Problem 1: LeetCode 739 — Daily Temperatures

### 2.1 Problem Statement Breakdown & Line-by-Line Annotations
> *"Given an array of integers `temperatures` represents the daily temperatures, return an array `answer` such that `answer[i]` is the number of days you have to wait after the $i$-th day to get a warmer temperature."*
- For each index $i$, find the smallest distance $j - i$ such that $j > i$ and $\text{temperatures}[j] > \text{temperatures}[i]$.
- If no future day is warmer, `answer[i] = 0`.

---

### 2.2 Monotonic Decreasing Stack Invariant
- Store **indices** in the stack (storing indices allows computing distance $j - i$ and looking up temperatures `temperatures[idx]`).
- When encountering temperature $T$:
  - While stack is non-empty and $T > \text{temperatures}[\text{stack}[-1]]$:
    - The warmer day for $\text{stack}[-1]$ has arrived!
    - Pop `prev_day = stack.pop()`.
    - Record distance: `answer[prev_day] = i - prev_day`.
  - Push current index $i$ onto stack.

---

### 2.3 Dry Run Trace Table

Consider `temperatures = [73, 74, 75, 71, 69, 72, 76, 73]`:

| Day $i$ | Temp $T$ | Action / Pops | Distance Recorded | Stack After Step (Indices) |
| :--- | :--- | :--- | :--- | :--- |
| 0 | 73 | Push 0 | — | `[0]` |
| 1 | 74 | $74 > 73 \implies$ Pop 0 | `ans[0] = 1 - 0 = 1` | `[1]` |
| 2 | 75 | $75 > 74 \implies$ Pop 1 | `ans[1] = 2 - 1 = 1` | `[2]` |
| 3 | 71 | $71 \le 75 \implies$ Push 3 | — | `[2, 3]` |
| 4 | 69 | $69 \le 71 \implies$ Push 4 | — | `[2, 3, 4]` |
| 5 | 72 | $72 > 69 \implies$ Pop 4<br>$72 > 71 \implies$ Pop 3 | `ans[4] = 5 - 4 = 1`<br>`ans[3] = 5 - 3 = 2` | `[2, 5]` |
| 6 | 76 | $76 > 72 \implies$ Pop 5<br>$76 > 75 \implies$ Pop 2 | `ans[5] = 6 - 5 = 1`<br>`ans[2] = 6 - 2 = 4` | `[6]` |
| 7 | 73 | $73 \le 76 \implies$ Push 7 | — | `[6, 7]` |

Output: `[1, 1, 4, 2, 1, 1, 0, 0]`.

---

### 2.4 Complete Python Implementation

```python
class SolutionDailyTemperatures:
    def dailyTemperatures(self, temperatures: list[int]) -> list[int]:
        n = len(temperatures)
        answer = [0] * n
        stack = []  # Monotonic decreasing stack storing indices
        
        for curr_day, curr_temp in enumerate(temperatures):
            while stack and curr_temp > temperatures[stack[-1]]:
                prev_day = stack.pop()
                answer[prev_day] = curr_day - prev_day
            stack.append(curr_day)
            
        return answer
```

- **Time Complexity**: $O(N)$ — Each index pushed once, popped at most once.
- **Space Complexity**: $O(N)$ — Stack size in worst case (monotonically decreasing temperatures).

---

## 3. Benchmark Problem 2: LeetCode 84 — Largest Rectangle in Histogram

### 3.1 Problem Statement Breakdown & Line-by-Line Annotations
> *"Given an array of integers `heights` representing the histogram's bar height where the width of each bar is 1, return the area of the largest rectangle in the histogram."*

---

### 3.2 The Bottleneck Invariant
For any given bar $i$ with height $H = \text{heights}[i]$, what is the widest rectangle with height $H$?
- It extends to the left until encountering the first bar **strictly shorter** than $H$ (index $L$).
- It extends to the right until encountering the first bar **strictly shorter** than $H$ (index $R$).
- The maximum width with height $H$ is:
  $$\text{width} = R - L - 1$$
- Total area = $\text{heights}[i] \times (R - L - 1)$.

A **Monotonically Increasing Stack** computes both boundaries $L$ and $R$ for all bars in a single pass:
- When incoming bar $R$ has height `< heights[stack[-1]]`:
  - Bar $i = \text{stack.pop()}$ has found its **right boundary** ($R$).
  - Its **left boundary** is the new top of the stack: $L = \text{stack}[-1]$!
  - We immediately calculate and update max area for bar $i$.

### Dummy Sentinels Optimization
By appending a virtual bar of height `0` to the end of `heights` and pushing a sentinel index `-1` (height `0`), all bars remaining in the stack are automatically flushed cleanly at the end without extra post-processing code.

---

### 3.3 Complete Python Implementation

```python
class SolutionLargestRectangle:
    def largestRectangleArea(self, heights: list[int]) -> int:
        # Stack stores indices; initialize with -1 to serve as left sentinel
        stack = [-1]
        max_area = 0
        
        # Append 0 to flush all remaining elements at the end
        heights.append(0)
        
        for r, h in enumerate(heights):
            # Monotonically increasing stack: pop when current height is smaller
            while stack[-1] != -1 and heights[stack[-1]] > h:
                height = heights[stack.pop()]
                left_boundary = stack[-1]
                width = r - left_boundary - 1
                max_area = max(max_area, height * width)
                
            stack.append(r)
            
        heights.pop()  # Restore input array
        return max_area
```

- **Time Complexity**: $O(N)$ — Every bar is pushed and popped at most once.
- **Space Complexity**: $O(N)$ — Auxiliary stack memory.

---

### 3.4 Live Verbalization Script

> *"To find the largest rectangle in a histogram in $O(N)$ time, I consider each bar as the potential minimum height of a candidate rectangle.
> 
> A bar of height $H$ can expand horizontally until it hits a shorter bar on the left (boundary $L$) and a shorter bar on the right (boundary $R$). Its maximum possible area is $H \times (R - L - 1)$.
> 
> I use a monotonically increasing stack of indices.
> 
> As I iterate through the bars, if the incoming bar is shorter than the bar at the top of the stack, the incoming bar acts as the right boundary $R$ for the top bar.
> 
> I pop the bar $i$. The new top of the stack is strictly shorter than bar $i$, so it acts as the left boundary $L$. The width is simply $R - L - 1$, and we evaluate the area.
> 
> Appending a dummy 0 height at the end ensures all bars are popped and processed. This achieves linear $O(N)$ time with $O(N)$ stack space."*
