# Domain 02: Binary Search — Core Patterns & Rotated Arrays

---

## 1. Algorithmic Blueprint & Invariant

Binary search operates by iteratively halving a monotonic search interval. At each step, a single comparison eliminates half of the remaining candidate space, yielding $O(\log N)$ logarithmic time complexity.

```
Search Space: [ low ───────────── mid ───────────── high ]
Target comparison:
• If target == arr[mid]  ──► Found
• If target > arr[mid]   ──► Eliminate left half:  low = mid + 1
• If target < arr[mid]   ──► Eliminate right half: high = mid - 1
```

### The 3 Core Template Paradigms
1. **Exact Match Search**: Search interval `low <= high`. Returns index of target or `-1`.
2. **Lower Bound (First Element $\ge \text{target}$)**: Finds first occurrence or insertion point (`bisect_left`).
3. **Upper Bound (First Element $> \text{target}$)**: Finds insertion point after duplicates (`bisect_right`).

---

## 2. Reusable Python Boilerplate Templates

### 2.1 Standard Exact Match
```python
def binary_search_exact(nums: list[int], target: int) -> int:
    low, high = 0, len(nums) - 1
    while low <= high:
        mid = low + (high - low) // 2
        if nums[mid] == target:
            return mid
        elif nums[mid] < target:
            low = mid + 1
        else:
            high = mid - 1
    return -1
```

### 2.2 Lower Bound (Bisect Left)
```python
def binary_search_lower_bound(nums: list[int], target: int) -> int:
    low, high = 0, len(nums)
    while low < high:
        mid = low + (high - low) // 2
        if nums[mid] < target:
            low = mid + 1
        else:
            high = mid
    return low  # First index where nums[index] >= target
```

---

## 3. Benchmark Problem Deep Dive: LeetCode 33 — Search in Rotated Sorted Array

### 3.1 Problem Statement Breakdown & Line-by-Line Annotations
> *"There is an integer array `nums` sorted in ascending order (with distinct values)."*
- Originally sorted: $nums[0] < nums[1] < \dots < nums[n-1]$. Distinct elements eliminate ambiguity.

> *"Prior to being passed to your function, `nums` is possibly rotated at an unknown pivot index $k$ ($1 \le k < \text{nums.length}$)..."*
- Array split into two sorted subarrays: $[nums[k], \dots, nums[n-1], nums[0], \dots, nums[k-1]]$.
- Key Property: For any partition at `mid`, **at least one of the two halves (`[low, mid]` or `[mid, high]`) is guaranteed to be strictly sorted**!

> *"Given the array `nums` after the possible rotation and an integer `target`, return the index of `target` if it is in `nums`, or `-1` if it is not in `nums`."*
- Goal: Find index in $O(\log n)$ runtime.

---

### 3.2 Constraints & Complexity Analysis
- $n = \text{nums.length} \in [1, 5000]$
- $-10^4 \le \text{nums}[i], \text{target} \le 10^4$
- All values of `nums` are **unique**.
- **Constraint Mandate**: Explicitly requires $O(\log n)$ runtime. Linear scan $O(n)$ is rejected.

---

### 3.3 Direction Exploration & Invariant Proof
- **Direction 1 (Find Pivot then Binary Search)**:
  - Find rotation pivot in $O(\log n)$, then run standard binary search on whichever sorted half contains target.
  - Takes two binary search passes.
- **Direction 2 (One-Pass Binary Search)**:
  - At any `mid`, evaluate which half is normally sorted:
    - **Case A**: If `nums[low] <= nums[mid]`, the left half `[low, mid]` is strictly sorted.
      - Check if `target` lies within `[nums[low], nums[mid])`:
        - If yes: search left half $\implies \text{high} = \text{mid} - 1$.
        - If no: search right half $\implies \text{low} = \text{mid} + 1$.
    - **Case B**: Otherwise, the right half `[mid, high]` is strictly sorted (`nums[mid] < nums[high]`).
      - Check if `target` lies within `(nums[mid], nums[high]]`:
        - If yes: search right half $\implies \text{low} = \text{mid} + 1$.
        - If no: search left half $\implies \text{high} = \text{mid} - 1$.

---

### 3.4 Step-by-Step Dry Run Trace Table
Input: `nums = [4, 5, 6, 7, 0, 1, 2]`, `target = 0`

| Step | `low` | `high` | `mid` | `nums[mid]` | Sorted Half Condition | Target Range Check | Action |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | 0 | 6 | 3 | 7 | `nums[0] (4) <= nums[3] (7)` $\implies$ Left is sorted | Is $0 \in [4, 7)$? No. | `low = mid + 1 = 4` |
| 2 | 4 | 6 | 5 | 1 | `nums[4] (0) <= nums[5] (1)` $\implies$ Left is sorted | Is $0 \in [0, 1)$? **Yes**! | `high = mid - 1 = 4` |
| 3 | 4 | 4 | 4 | 0 | `nums[4] == target (0)` $\implies$ **Match Found!** | - | **Return index 4** |

---

### 3.5 Complete Python Implementation

```python
class Solution:
    def search(self, nums: list[int], target: int) -> int:
        low, high = 0, len(nums) - 1
        
        while low <= high:
            mid = low + (high - low) // 2
            
            if nums[mid] == target:
                return mid
                
            # Determine which half is sorted
            if nums[low] <= nums[mid]:
                # Left half [low...mid] is sorted
                if nums[low] <= target < nums[mid]:
                    high = mid - 1
                else:
                    low = mid + 1
            else:
                # Right half [mid...high] is sorted
                if nums[mid] < target <= nums[high]:
                    low = mid + 1
                else:
                    high = mid - 1
                    
        return -1
```

- **Time Complexity**: $O(\log n)$ — Interval is halved every step.
- **Space Complexity**: $O(1)$ — Constant memory.

---

### 3.6 Live Verbalization Script

> *"Because the array was originally sorted and rotated, dividing the array at any midpoint will always leave at least one half strictly sorted.
> 
> At each step, I first check if `nums[mid] == target`. If not, I determine whether the left half is sorted by evaluating `nums[low] <= nums[mid]`.
> 
> If the left half is sorted, checking if `target` falls between `nums[low]` and `nums[mid]` is trivial. If it does, I discard the right half; otherwise, I discard the left half.
> 
> If the left half is not sorted, the right half is guaranteed to be sorted, and I perform the mirror check against `nums[mid]` and `nums[high]`.
> 
> This maintains our $O(\log n)$ invariant in a single pass without needing a separate pivot-finding step."*
