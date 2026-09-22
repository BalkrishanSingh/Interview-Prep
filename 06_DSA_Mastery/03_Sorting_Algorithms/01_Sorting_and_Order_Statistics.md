# Domain 03: Sorting Algorithms & Order Statistics

---

## 1. Classical Sorting Algorithms Taxonomy

Sorting organizes elements in monotonic order. Modern systems leverage comparisons or distribution properties to optimize cache lines and runtimes.

```
┌────────────────────────────────────────────────────────────────────────┐
│                        SORTING ALGORITHMS MATRIX                       │
├──────────────────┬────────────┬────────────┬────────────┬──────────────┤
│ Algorithm        │ Best Time  │ Avg Time   │ Worst Time │ Space (Aux)  │
├──────────────────┼────────────┼────────────┼────────────┼──────────────┤
│ Merge Sort       │ O(N log N) │ O(N log N) │ O(N log N) │ O(N) (Stable)│
│ Quick Sort       │ O(N log N) │ O(N log N) │ O(N^2)     │ O(log N)     │
│ Heap Sort        │ O(N log N) │ O(N log N) │ O(N log N) │ O(1) (In-Plc)│
│ Counting Sort    │ O(N + K)   │ O(N + K)   │ O(N + K)   │ O(K)         │
└──────────────────┴────────────┴────────────┴────────────┴──────────────┘
```

### Critical Concept: Stability in Sorting
- A sort is **Stable** if elements with identical keys maintain their relative pre-existing input order.
- *Why it matters*: When sorting multi-column records (e.g., sorting employees by Name first, and then by Department), an unstable sort scrambles the alphabetical name order within departments.
- **Merge Sort is Stable**; **QuickSort and HeapSort are Unstable**.

---

## 2. QuickSelect: Finding the K-th Order Statistic in O(N)

When finding the $K$-th largest or smallest element, sorting the entire array is wasteful ($O(N \log N)$). 
**QuickSelect** (Hoare's selection algorithm) applies the partitioning step of QuickSort, but recursively explores **only the one partition** containing the target index:

$$T(N) = T(N/2) + O(N) = O(N + N/2 + N/4 + \dots) = O(2N) = \mathbf{O(N) \text{ Expected Time}}$$

---

## 3. Benchmark Problem Deep Dive: LeetCode 215 — Kth Largest Element in an Array

### 3.1 Problem Statement Breakdown & Line-by-Line Annotations
> *"Given an integer array `nums` and an integer `k`, return the $k$-th largest element in the array."*
- $k=1$ means the maximum element.
- $k=2$ means the second largest.
- Notice: The problem asks for the $k$-th largest in **sorted order**, not the $k$-th distinct element!

> *"Can you solve it without sorting?"*
- Explicit constraint directing away from trivial $O(N \log N)$ `nums.sort()`.
- Target: $O(N \log K)$ via Min-Heap or expected $O(N)$ via QuickSelect.

---

### 3.2 Constraints & Complexity Analysis
- $1 \le k \le \text{nums.length} \le 10^5$
- $-10^4 \le \text{nums}[i] \le 10^4$
- For $N = 100,000$:
  - Full Sort: $10^5 \log_2(10^5) \approx 1.7 \times 10^6$ operations ($O(N \log N)$).
  - Min-Heap of size $k$: $N \log k$ operations ($O(N \log K)$).
  - QuickSelect: Expected $\approx 2 \times 10^5$ operations ($O(N)$).

---

### 3.3 Direction Exploration & Invariant Proof
- **Target Index Conversion**:
  The $k$-th largest element in an array of length $N$ resides at index:
  $$\text{target\_idx} = N - k \quad \text{(in 0-indexed ascending order)}$$
- **Lomuto / Hoare Partition Invariant**:
  After partitioning around a pivot value $P$:
  - All elements at indices $< \text{pivot\_idx}$ are $\le P$.
  - Element at $\text{pivot\_idx}$ is in its **final, permanent sorted position**.
  - All elements at indices $> \text{pivot\_idx}$ are $\ge P$.
- **Decision Branch**:
  - If $\text{pivot\_idx} == \text{target\_idx} \implies$ **Found!**
  - If $\text{pivot\_idx} < \text{target\_idx} \implies$ Discard left half, search $[\text{pivot\_idx} + 1, \text{high}]$.
  - If $\text{pivot\_idx} > \text{target\_idx} \implies$ Discard right half, search $[\text{low}, \text{pivot\_idx} - 1]$.

---

### 3.4 Python Implementations

#### Approach A: Min-Heap of Size K (Production Standard, $O(N \log K)$ Time, $O(K)$ Space)
```python
import heapq

class SolutionHeap:
    def findKthLargest(self, nums: list[int], k: int) -> int:
        min_heap = []
        for num in nums:
            heapq.heappush(min_heap, num)
            if len(min_heap) > k:
                heapq.heappop(min_heap)  # Evict smallest, leaving top k largest
        return min_heap[0]
```

#### Approach B: Randomized QuickSelect ($O(N)$ Expected Time, $O(1)$ Space)
```python
import random

class SolutionQuickSelect:
    def findKthLargest(self, nums: list[int], k: int) -> int:
        target_idx = len(nums) - k  # Kth largest is (N - k)th smallest
        
        def partition(low: int, high: int) -> int:
            # Random pivot selection prevents worst-case O(N^2) on sorted inputs
            rand_pivot = random.randint(low, high)
            nums[rand_pivot], nums[high] = nums[high], nums[rand_pivot]
            
            pivot = nums[high]
            i = low
            for j in range(low, high):
                if nums[j] <= pivot:
                    nums[i], nums[j] = nums[j], nums[i]
                    i += 1
            nums[i], nums[high] = nums[high], nums[i]
            return i

        low, high = 0, len(nums) - 1
        while low <= high:
            p_idx = partition(low, high)
            if p_idx == target_idx:
                return nums[p_idx]
            elif p_idx < target_idx:
                low = p_idx + 1
            else:
                high = p_idx - 1
                
        return -1
```

---

### 3.5 Live Verbalization Script

> *"To find the $k$-th largest element without performing a full $O(N \log N)$ sort, there are two primary approaches:
> 
> 1. **Min-Heap Approach**: Maintain a min-heap of size $k$. As we stream through the array, whenever the heap exceeds size $k$, we pop the smallest element. At the end, the heap contains the $k$ largest elements, and the root is the $k$-th largest. This runs in $O(N \log K)$ time and $O(K)$ space.
> 
> 2. **QuickSelect Approach**: We can find the element in expected $O(N)$ time with $O(1)$ space. The $k$-th largest element corresponds to index $N - k$ in an ascending sorted array. 
> 
> We choose a random pivot and partition the array such that all elements smaller than the pivot precede it, and larger elements follow it. 
> 
> Unlike QuickSort which recurses into both partitions, QuickSelect inspects the pivot's finalized index and recurses only into the single partition containing $N - k$. By eliminating half the array at each step, the geometric series $N + N/2 + N/4 + \dots$ sums to $O(N)$ expected time."*
