# Domain 04: Recursion & Backtracking — Core Patterns

---

## 1. The Backtracking Mental Model

Backtracking is an exhaustive depth-first search (DFS) over a **State Space Tree**. When the algorithm determines that a partial candidate cannot possibly lead to a valid solution, it **prunes** the branch by undoing the most recent choice (**backtracking**).

```
                      ROOT []
              /          |          \
          [1]           [2]          [3]
         /   \           |
     [1,2]   [1,3]     [2,3]
      /
   [1,2,3]
   
   Invariant: Choose -> Explore (Recurse) -> Unchoose (Pop/Backtrack)
```

### The Universal Backtracking Boilerplate:
```python
def backtrack(candidate_path, start_index, state):
    if is_solution(candidate_path):
        results.append(list(candidate_path))
        return
        
    for i in range(start_index, len(elements)):
        if is_valid_choice(elements[i]):
            # 1. Choose
            candidate_path.append(elements[i])
            
            # 2. Explore
            backtrack(candidate_path, i + 1, state)  # (i for reuse, i+1 for distinct)
            
            # 3. Unchoose (Backtrack)
            candidate_path.pop()
```

---

## 2. The Big 3 Paradigms: Subsets vs. Combinations vs. Permutations

| Pattern | Order Matters? | Element Re-use? | Loop Start Index | Pruning Condition for Duplicates |
| :--- | :--- | :--- | :--- | :--- |
| **Subsets** | No (`[1,2] == [2,1]`) | No | `start_index = i + 1` | `if i > start and nums[i] == nums[i-1]: continue` |
| **Combinations** | No (`k` length subset) | Problem dependent | `start_index = i + 1` | `if i > start and nums[i] == nums[i-1]: continue` |
| **Permutations** | **Yes** (`[1,2] != [2,1]`) | No (all used) | `start_index = 0` + `visited` set | `if visited[i] or (i > 0 and nums[i] == nums[i-1] and not visited[i-1])` |

---

## 3. Benchmark Problem Deep Dive: LeetCode 78 & 90 — Subsets (Power Set)

### 3.1 Problem Statement Breakdown & Line-by-Line Annotations
> *"Given an integer array `nums` of unique elements, return all possible subsets (the power set)."*
- A power set contains $2^N$ elements (each element is either included or excluded).

> *"The solution set must not contain duplicate subsets. Return the solution in any order."*
- Uniqueness invariant: Combinations of identical elements must not be generated twice.

---

### 3.2 Constraints & Complexity Analysis
- $1 \le \text{nums.length} \le 10$ (For Subsets I: up to $10$, for Subsets II: up to $20$).
- Elements in range $[-10, 10]$.
- **Mathematical Space**:
  - $N = 10 \implies 2^{10} = 1,024$ subsets.
  - $N = 20 \implies 2^{20} \approx 10^6$ subsets.
  - Generating and copying each subset takes $O(N)$ time.
  - Expected Total Time Complexity: $O(N \cdot 2^N)$.
  - Expected Space Complexity: $O(N)$ recursion stack depth.

---

### 3.3 Direction Exploration & Invariant Proof
- **Branching Decision**:
  At each index $i$ in `range(start, len(nums))`:
  1. Add `nums[i]` to `current_subset`.
  2. Record `current_subset` into `result`.
  3. Recurse to generate all downstream subsets starting at `i + 1`.
  4. Pop `nums[i]` to reset state before the next sibling iteration.
- **Handling Duplicates (Subsets II)**:
  Sort `nums` first. If `nums[i] == nums[i - 1]` and `i > start`, skipping this iteration ensures duplicate branches are pruned at the current recursion level.

---

### 3.4 Step-by-Step Dry Run Trace Table
Input: `nums = [1, 2, 3]`

```
backtrack([], start=0) -> Appends []
  ├── i=0: Pick 1 -> path=[1], append [1]
  │   └── backtrack([1], start=1)
  │       ├── i=1: Pick 2 -> path=[1,2], append [1,2]
  │       │   └── backtrack([1,2], start=2)
  │       │       └── i=2: Pick 3 -> path=[1,2,3], append [1,2,3]
  │       │           └── backtrack([1,2,3], start=3) -> Returns (start==len)
  │       │       Pop 3 -> path=[1,2]
  │       │   Pop 2 -> path=[1]
  │       └── i=2: Pick 3 -> path=[1,3], append [1,3]
  │           Pop 3 -> path=[1]
  │   Pop 1 -> path=[]
  ├── i=1: Pick 2 -> path=[2], append [2]
  │   └── i=2: Pick 3 -> path=[2,3], append [2,3]
  └── i=2: Pick 3 -> path=[3], append [3]
```
**Total Generated Subsets**: `[[], [1], [1, 2], [1, 2, 3], [1, 3], [2], [2, 3], [3]]` (Exactly $2^3 = 8$).

---

### 3.5 Complete Python Implementation (Subsets II with Duplicate Handling)

```python
class Solution:
    def subsetsWithDup(self, nums: list[int]) -> list[list[int]]:
        nums.sort()  # Sorting brings duplicates adjacent for O(1) pruning
        result = []
        path = []
        
        def backtrack(start: int):
            # Every reached state represents a valid unique subset
            result.append(list(path))
            
            for i in range(start, len(nums)):
                # Prune duplicate branches at the same tree depth
                if i > start and nums[i] == nums[i - 1]:
                    continue
                    
                path.append(nums[i])       # Choose
                backtrack(i + 1)           # Explore
                path.pop()                 # Unchoose (Backtrack)
                
        backtrack(0)
        return result
```

- **Time Complexity**: $O(N \cdot 2^N)$ — $2^N$ states, each taking $O(N)$ to copy to the result list.
- **Space Complexity**: $O(N)$ — Maximum recursion stack depth is $N$.

---

### 3.6 Live Verbalization Script

> *"To generate all power set subsets while preventing duplicates:
> 
> A brute-force iteration without pruning would generate redundant combinations whenever duplicate values exist.
> 
> First, I sort the input array so identical elements are adjacent.
> 
> I formulate a recursive backtracking helper `backtrack(start)`. At the start of each invocation, the current path is copied into our results because every prefix in the tree is a valid subset.
> 
> In our loop from `start` to `len(nums) - 1`, we prune duplicates by checking `if i > start and nums[i] == nums[i - 1]`. This ensures we only branch on the first occurrence of a duplicate at any given recursion level, while still permitting duplicates across deeper levels (e.g., `[2, 2]`).
> 
> We append the number, recurse with `start = i + 1`, and pop the number to backtrack cleanly. This runs in $O(N \cdot 2^N)$ time and $O(N)$ auxiliary stack space."*
