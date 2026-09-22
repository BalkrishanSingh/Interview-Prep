# Framework: Non-DP Thinking Blueprint (Pointers, Trees, Graphs, Search)

---

## 1. Property Observation & Invariant Identification

Unlike Dynamic Programming (which relies on state transitions and overlapping subproblems), non-DP algorithms exploit **structural invariants and data properties** to eliminate redundant search spaces.

```
PROPERTY OBSERVED IN PROBLEM                      ALGORITHMIC INVARIANT SELECTED
─────────────────────────────────────────────     ──────────────────────────────────────────
• Sorted array / Monotonic function         ───►  Binary Search / Two Pointers
• Contiguous subarray under validity check  ───►  Sliding Window (Expand right, shrink left)
• Hierarchical / Recursive dependencies     ───►  Tree Traversals / DFS Post-order
• Shortest path on unweighted graph         ───►  Breadth-First Search (Queue FIFO)
• Shortest path on weighted non-neg graph   ───►  Dijkstra's Algorithm (Min-Heap Priority Queue)
• Dependency resolution / Pre-requisites    ───►  Topological Sort (Kahn's / Indegree 0)
• Dynamic connectivity / Cycle detection    ───►  Disjoint Set Union (DSU / Union-Find)
• Locally optimal choice yields global best ───►  Greedy (Interval sorting)
```

---

## 2. The 7-Step Non-DP Problem Solving Blueprint

### Step 1: Problem Dissection & Property Extraction
Read every line of the problem statement:
- Is the sequence sorted or unsorted?
- Are elements positive, negative, or distinct?
- Does the problem require contiguous elements (subarray/substring) or non-contiguous elements (subsequence)?
- Are we asked for an exact value, an index, or an aggregate property?

### Step 2: Constraint & Complexity Envelope
- $N \le 10^5 \implies O(N^2)$ brute force will trigger **Time Limit Exceeded (TLE)** ($10^{10}$ operations).
- Target complexity: $O(N \log N)$ or $O(N)$.
- Required auxiliary space: $O(1)$ in-place or $O(N)$ with auxiliary hash map/set.

### Step 3: Brute Force Baseline & Bottleneck Identification
- What is the naive solution? (e.g., nested double loop checking every pair $\implies O(N^2)$).
- **Identify the Bottleneck**: What repeated or redundant work is the naive algorithm doing? (e.g., repeatedly scanning backwards, or recomputing window sums from scratch).

### Step 4: Invariant Formulation
Define the mathematical invariant maintained by the algorithm:
- *Two Pointers*: *"Since the array is sorted, if `arr[left] + arr[right] > target`, no element to the right of `left` can pair with `right` to equal `target`. Therefore, `right` must decrement."*
- *Sliding Window*: *"The window `[left, right]` always maintains at most $K$ distinct characters."*
- *Binary Search*: *"The target value is guaranteed to lie within the search interval `[low, high]`."*

### Step 5: Generic Algorithmic Template
Implement using parameterized, robust boilerplate code with clear boundary condition handling.

### Step 6: Step-by-Step Dry Run (Trace Table)
Walk through a concrete testcase row-by-row tracking:
- Pointer positions (`left`, `right`, `mid`)
- State variables (`curr_sum`, `max_len`, `visited`)
- Termination conditions

### Step 7: Boundary Edge Case Audit
Verify behavior on extreme edge inputs:
- Empty array / single element ($N = 0, N = 1$)
- All duplicate elements
- Target not found / unreachable state
- Negative numbers or zeroes

---

## 3. Generic Algorithmic Boilerplate Patterns

### 3.1 Two Pointers (Converging from Opposite Ends)
```python
def two_pointers_converging(arr: list[int], target: int) -> list[int]:
    left = 0
    right = len(arr) - 1
    
    while left < right:
        curr_sum = arr[left] + arr[right]
        if curr_sum == target:
            return [left, right]
        elif curr_sum < target:
            left += 1   # Need larger sum -> move left pointer forward
        else:
            right -= 1  # Need smaller sum -> move right pointer backward
            
    return []
```

### 3.2 Dynamic Sliding Window
```python
def sliding_window_dynamic(s: str, k: int) -> int:
    from collections import defaultdict
    counts = defaultdict(int)
    left = 0
    max_len = 0
    
    for right in range(len(s)):
        # 1. Expand window by incorporating right element
        counts[s[right]] += 1
        
        # 2. Shrink window from left until invariant condition is satisfied
        while len(counts) > k:  # Invariant violation
            counts[s[left]] -= 1
            if counts[s[left]] == 0:
                del counts[s[left]]
            left += 1
            
        # 3. Update result with valid window [left, right]
        max_len = max(max_len, right - left + 1)
        
    return max_len
```

### 3.3 Binary Search on Sorted Range
```python
def binary_search_standard(nums: list[int], target: int) -> int:
    low = 0
    high = len(nums) - 1
    
    while low <= high:
        # Prevent integer overflow in languages with fixed integer sizes
        mid = low + (high - low) // 2
        if nums[mid] == target:
            return mid
        elif nums[mid] < target:
            low = mid + 1
        else:
            high = mid - 1
            
    return -1
```

---

## 4. Live Verbalization Framework

When presenting a non-DP problem live:

1. **Acknowledge Brute Force & Explain Why It Fails**:
   > *"The naive approach is to generate all possible subarrays using two nested loops and evaluate each one. With $N \le 10^5$, an $O(N^2)$ solution requires roughly $10^{10}$ operations, which will exceed our 1-second time limit."*
2. **State Invariant & Optimization Logic**:
   > *"Notice that as we expand our right boundary, the window size only grows, and character counts are monotonic. Instead of restarting our search from index `left + 1`, we can slide `left` forward only when an invalid character count occurs. This ensures every element enters and leaves the window at most once, reducing our time complexity to $O(N)$."*
3. **Walk Through Dry Run**:
   > *"Let's trace this on input `s = 'eceba'`, $k = 2$. At index 0, window is `['e']`, distinct count is 1. At index 2, window is `['e', 'c', 'e']`, distinct count is still 2. At index 3, 'b' makes 3 distinct characters, so our `while` loop triggers, advancing `left` until distinct count returns to 2..."*
4. **State Final Time & Space Complexity**:
   > *"Time complexity is $O(N)$ because each pointer advances at most $N$ times. Space complexity is $O(K)$ where $K$ is the number of unique characters stored in our frequency map."*
