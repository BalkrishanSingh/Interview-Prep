# Domain 01: Two Pointers Technique

---

## 1. Algorithmic Blueprint & Invariant

The **Two Pointers Technique** uses two indices that traverse a data structure either towards each other (converging), in the same direction at different speeds (fast/slow), or from independent starting boundaries.

```
CONVERGING TWO POINTERS:
[ Left Pointer ] ────────►                       ◄──────── [ Right Pointer ]
Index:  0     1     2     3     4     5     6     7     8     9
Array: [ 2,   7,   11,   15,   18,   22,   25,   30,   35,   40 ]
```

### The Invariant:
When an array is sorted, the sum $\text{arr}[\text{left}] + \text{arr}[\text{right}]$ is monotonic:
- If $\text{sum} < \text{target}$, moving `left` forward ($\text{left} + 1$) can only *increase* the sum.
- If $\text{sum} > \text{target}$, moving `right` backward ($\text{right} - 1$) can only *decrease* the sum.
- **Why this eliminates $O(N^2)$ brute force**: At each step, a single comparison eliminates an entire row or column of the search matrix, ensuring $O(N)$ linear time.

---

## 2. Reusable Python Boilerplate Template

```python
def two_pointers_template(arr: list[int], target: int) -> list[int]:
    """
    Standard converging two pointers template on sorted array.
    Time Complexity: O(N)
    Space Complexity: O(1)
    """
    left = 0
    right = len(arr) - 1
    
    while left < right:
        curr = arr[left] + arr[right]
        if curr == target:
            return [left, right]
        elif curr < target:
            left += 1   # Need larger value
        else:
            right -= 1  # Need smaller value
            
    return []
```

---

## 3. Benchmark Problem Deep Dive: LeetCode 11 — Container With Most Water

### 3.1 Problem Statement Breakdown & Line-by-Line Annotations
> *"You are given an integer array `height` of length $n$."*
- `height[i]` gives the vertical height of a line at coordinate $(i, 0)$ to $(i, \text{height}[i])$.
- $n$ vertical lines drawn such that the two endpoints of the $i$-th line are $(i, 0)$ and $(i, \text{height}[i])$.

> *"Find two lines that together with the x-axis form a container, such that the container contains the most water."*
- A container formed by index $i$ and index $j$ ($i < j$) has:
  - Width: $j - i$
  - Height limit: $\min(\text{height}[i], \text{height}[j])$ (water spills over the shorter bar).
  - Contained Area: $(j - i) \times \min(\text{height}[i], \text{height}[j])$.

> *"Return the maximum amount of water a container can store. Notice that you may not slant the container."*
- Optimization goal: $\max_{i < j} \left( (j - i) \times \min(\text{height}[i], \text{height}[j]) \right)$.

---

### 3.2 Constraints & Complexity Analysis
- $n = \text{height.length} \in [2, 10^5]$
- $\text{height}[i] \in [0, 10^4]$
- **Complexity Assessment**:
  - $n = 10^5 \implies n^2 = 10^{10}$ operations.
  - A nested loop checking every pair $(i, j)$ requires $O(n^2)$ time $\implies$ **Definite TLE**.
  - We must achieve $O(n)$ time with $O(1)$ auxiliary space.

---

### 3.3 Direction Exploration & Invariant Proof
- **Direction 1 (Brute Force)**:
  - Check every pair $(i, j)$ where $0 \le i < j < n$.
  - Time: $O(n^2)$, Space: $O(1)$.
- **Direction 2 (Two Pointers from Max Width)**:
  - Start with widest possible container: $\text{left} = 0, \text{right} = n - 1$.
  - Width is at its theoretical maximum: $n - 1$.
  - To find a larger area as width shrinks ($w = w - 1$), the height **must strictly increase**.
  - Which pointer do we move?
    - The area is strictly bottlenecked by $\min(\text{height}[\text{left}], \text{height}[\text{right}])$.
    - If we move the taller line inward, the width decreases ($j - i - 1$), but the height is still bounded by the shorter line or less. The area can **never increase**!
    - The *only* hope of discovering a larger area is to discard the shorter line and move its pointer inward:
      $$\text{If } \text{height}[\text{left}] < \text{height}[\text{right}] \implies \text{left} = \text{left} + 1$$
      $$\text{Else } \text{right} = \text{right} - 1$$

---

### 3.4 Step-by-Step Dry Run Trace Table
Input: `height = [1, 8, 6, 2, 5, 4, 8, 3, 7]`

| Step | `left` | `right` | `h[left]` | `h[right]` | `width` ($r - l$) | `min_height` | `current_area` | `max_area` | Action Taken |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | 0 | 8 | 1 | 7 | 8 | 1 | $8 \times 1 = 8$ | **8** | `h[0] < h[8]` $\implies$ `left++` |
| 2 | 1 | 8 | 8 | 7 | 7 | 7 | $7 \times 7 = 49$ | **49** | `h[1] > h[8]` $\implies$ `right--` |
| 3 | 1 | 7 | 8 | 3 | 6 | 3 | $6 \times 3 = 18$ | **49** | `h[1] > h[7]` $\implies$ `right--` |
| 4 | 1 | 6 | 8 | 8 | 5 | 8 | $5 \times 8 = 40$ | **49** | `h[1] == h[6]` $\implies$ `right--` |
| 5 | 1 | 5 | 8 | 4 | 4 | 4 | $4 \times 4 = 16$ | **49** | `h[1] > h[5]` $\implies$ `right--` |
| 6 | 1 | 4 | 8 | 5 | 3 | 5 | $3 \times 5 = 15$ | **49** | `h[1] > h[4]` $\implies$ `right--` |
| 7 | 1 | 3 | 8 | 2 | 2 | 2 | $2 \times 2 = 4$ | **49** | `h[1] > h[3]` $\implies$ `right--` |
| 8 | 1 | 2 | 8 | 6 | 1 | 6 | $1 \times 6 = 6$ | **49** | `h[1] > h[2]` $\implies$ `right--` |
| 9 | 1 | 1 | - | - | - | - | - | **49** | `left == right` $\implies$ Loop terminates |

**Final Result**: `49`.

---

### 3.5 Complete Python Implementation

```python
class Solution:
    def maxArea(self, height: list[int]) -> int:
        left = 0
        right = len(height) - 1
        max_water = 0
        
        while left < right:
            h_left = height[left]
            h_right = height[right]
            
            # Area = width * min(height)
            width = right - left
            current_water = width * (h_left if h_left < h_right else h_right)
            
            if current_water > max_water:
                max_water = current_water
                
            # Discard the shorter boundary
            if h_left < h_right:
                left += 1
            else:
                right -= 1
                
        return max_water
```

- **Time Complexity**: $O(n)$ — Each iteration shifts either `left` or `right` by 1. Exactly $n-1$ iterations.
- **Space Complexity**: $O(1)$ — Only scalar integer variables used.

---

### 3.6 Live Verbalization Script

> *"To find the maximum area between two lines, the area is governed by `width * min(h[left], h[right])`.
> 
> A brute-force evaluation of all pairs would take $O(n^2)$ time, which will fail for $n = 10^5$. 
> 
> Instead, I observe that the container width is maximized by choosing the endpoints `left = 0` and `right = n - 1`. From this starting position, any inward movement of a pointer will strictly decrease the width by 1. Therefore, to achieve a larger area, the bottleneck height must increase.
> 
> Because the current water height is limited by the shorter of the two lines, moving the taller line inward can never increase the area—the width decreases, but the height ceiling is still held down by the shorter line. The only chance of finding a greater area is to discard the shorter line by advancing its pointer.
> 
> This invariant guarantees that we never prematurely discard any potential optimal solution, processing the array in $O(n)$ time and $O(1)$ auxiliary space."*
