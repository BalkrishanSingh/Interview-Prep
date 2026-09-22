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
- If `sum < target`, moving `left` forward (`left + 1`) can only *increase* the sum.
- If `sum > target`, moving `right` backward (`right - 1`) can only *decrease* the sum.
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

---

## 4. Benchmark Problem 2: LeetCode 15 — 3Sum

### 4.1 Problem Statement Breakdown & Line-by-Line Annotations
> *"Given an integer array `nums`, return all the triplets `[nums[i], nums[j], nums[k]]` such that `i != j`, `i != k`, and `j != k`, and `nums[i] + nums[j] + nums[k] == 0`."*
- Triplet sum to 0.
- Output cannot contain duplicate triplets.

---

### 4.2 The Sorting & Dual Pointer Invariant ($O(N^2)$ Time, $O(1)$ Extra Space)

1. **Sort `nums` in non-decreasing order**: Enables directional two-pointer adjustments and duplicate skipping.
2. **Fix outer pointer $i$**: For each unique value `nums[i]`, find two numbers in `nums[i+1 ... N-1]` that sum to `-nums[i]`.
3. **Duplicate Pruning Invariants**:
   - **Outer loop skip**: If $i > 0$ and `nums[i] == nums[i - 1]`, skip to avoid repeating the same first element.
   - **Inner two-pointer skips**: When a valid triplet is found (`nums[i] + nums[l] + nums[r] == 0`), increment $l$ and decrement $r$, then skip all identical adjacent values:
     `while l < r and nums[l] == nums[l - 1]: l += 1`
     `while l < r and nums[r] == nums[r + 1]: r -= 1`

---

### 4.3 Complete Python Implementation

```python
class Solution3Sum:
    def threeSum(self, nums: list[int]) -> list[list[int]]:
        nums.sort()
        n = len(nums)
        triplets = []
        
        for i in range(n - 2):
            # Early exit: if smallest element is positive, sum cannot be 0
            if nums[i] > 0:
                break
                
            # Skip duplicate values for outer pointer i
            if i > 0 and nums[i] == nums[i - 1]:
                continue
                
            l, r = i + 1, n - 1
            target = -nums[i]
            
            while l < r:
                curr_sum = nums[l] + nums[r]
                if curr_sum == target:
                    triplets.append([nums[i], nums[l], nums[r]])
                    l += 1
                    r -= 1
                    # Skip duplicate inner pointers
                    while l < r and nums[l] == nums[l - 1]:
                        l += 1
                    while l < r and nums[r] == nums[r + 1]:
                        r -= 1
                elif curr_sum < target:
                    l += 1
                else:
                    r -= 1
                    
        return triplets
```

- **Time Complexity**: $O(N^2)$ — Sorting takes $O(N \log N)$; outer loop runs $N$ times with an $O(N)$ two-pointer scan.
- **Space Complexity**: $O(1)$ auxiliary space (ignoring sorting memory).

---

### 4.4 Live Verbalization Script

> *"For 3Sum, I sort the array first. Sorting transforms the problem into $N$ two-pointer Two-Sum subproblems and allows clean duplicate elimination without sets.
> 
> I iterate the first element index $i$ from 0 to $N-3$. If `nums[i] > 0`, we can terminate immediately because all subsequent numbers are non-negative and cannot sum to 0. If $i > 0$ and `nums[i] == nums[i-1]`, I skip to prevent duplicate triplet roots.
> 
> For each $i$, I set two pointers: $l = i + 1$ and $r = N - 1$.
> If `nums[l] + nums[r] == -nums[i]`, we record the triplet and move both pointers inward, skipping all duplicate adjacent values of $l$ and $r$.
> If the sum is too small, we increment $l$; if too large, we decrement $r$.
> 
> This runs in $O(N^2)$ time and $O(1)$ auxiliary space."*

---

## 5. Benchmark Problem 3: LeetCode 42 — Trapping Rain Water

### 5.1 Problem Statement Breakdown & Line-by-Line Annotations
> *"Given `n` non-negative integers representing an elevation map where the width of each bar is 1, compute how much water it can trap after raining."*

---

### 5.2 The Two-Pointer Bottleneck Invariant ($O(N)$ Time, $O(1)$ Space)

The volume of water trapped above bar $i$ is governed by the shorter of its maximum walls to the left and right:
$$\text{water}[i] = \max(0, \min(L_{\max}, R_{\max}) - \text{height}[i])$$

Where $L_{\max} = \max_{0 \le k \le i}(\text{height}[k])$ and $R_{\max} = \max_{i \le k < N}(\text{height}[k])$.

Instead of precomputing prefix and suffix max arrays in $O(N)$ space, we maintain two pointers $l = 0$ and $r = N - 1$ with running values `left_max` and `right_max`:
- **The Deciding Invariant**:
  If `left_max < right_max`: The bottleneck wall for bar $l$ is **strictly determined by `left_max`**, regardless of whether future bars between $l$ and $r$ are even higher than `right_max`.
  Therefore, we can unconditionally add `left_max - height[l]` to total water and advance `l += 1`.
- Otherwise: The bottleneck wall for bar $r$ is strictly bounded by `right_max`. We add `right_max - height[r]` and advance `r -= 1`.

---

### 5.3 Complete Python Implementation ($O(1)$ Space)

```python
class SolutionTrappingWater:
    def trap(self, height: list[int]) -> int:
        if not height:
            return 0
            
        l, r = 0, len(height) - 1
        left_max = height[l]
        right_max = height[r]
        total_water = 0
        
        while l < r:
            if left_max < right_max:
                l += 1
                left_max = max(left_max, height[l])
                total_water += left_max - height[l]
            else:
                r -= 1
                right_max = max(right_max, height[r])
                total_water += right_max - height[r]
                
        return total_water
```

- **Time Complexity**: $O(N)$ — Single pass where each step increments $l$ or decrements $r$.
- **Space Complexity**: $O(1)$ — Only scalar pointers and max trackers used.

---

### 5.4 Live Verbalization Script

> *"For Trapping Rain Water, the water retained above any bar is limited by the shorter of its surrounding walls: `min(left_max, right_max) - height[i]`.
> 
> Rather than using $O(N)$ auxiliary arrays to store prefix and suffix maximums, I use two pointers at the ends: `l = 0` and `r = n - 1`, tracking running `left_max` and `right_max`.
> 
> The core insight is: if `left_max < right_max`, the height of water at `l` is strictly bound by `left_max`. It does not matter what heights exist between `l` and `r` because we already know a wall at least as tall as `right_max` exists to the right. Therefore, we can safely compute trapped water at `l` as `left_max - height[l]` and increment `l`.
> 
> Symmetrically, if `right_max <= left_max`, water at `r` is bound by `right_max`, so we compute water at `r` and decrement `r`.
> 
> This processes the entire array in a single $O(N)$ pass using $O(1)$ space."*

