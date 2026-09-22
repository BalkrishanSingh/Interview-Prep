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

---

## 4. Benchmark Problem Deep Dive: LeetCode 39 & 40 — Combination Sum I & II

### 4.1 Problem Statement Breakdown & Line-by-Line Annotations
> *"Given an array of distinct integers `candidates` and a target integer `target`, return a list of all unique combinations where the chosen numbers sum to `target`."*
- Unlike Subsets, we only record paths whose accumulated sum equals `target`.
- In **Combination Sum I (LC 39)**: The same number may be chosen from `candidates` an **unlimited number of times**.
- In **Combination Sum II (LC 40)**: `candidates` contains **duplicates**, and each number may only be used **once**.

---

### 4.2 Invariant Comparison: Unbounded Reuse vs. Duplicate Pruning

| Problem | Element Re-use | Input Duplicates | Recursive Call Index | Loop Pruning Condition |
| :--- | :--- | :--- | :--- | :--- |
| **LC 39 (Comb Sum I)** | Yes (Unlimited) | No | `backtrack(i, rem - candidates[i])` | `if candidates[i] > rem: break` (with sorted array) |
| **LC 40 (Comb Sum II)**| No (Once each) | Yes | `backtrack(i + 1, rem - candidates[i])` | `if i > start and candidates[i] == candidates[i-1]: continue` |

---

### 4.3 Complete Python Implementations

#### Combination Sum I (Optimal Solution with Early Exit)
```python
class SolutionCombSumI:
    def combinationSum(self, candidates: list[int], target: int) -> list[list[int]]:
        # Sorting enables breaking early when candidates[i] > remaining
        candidates.sort()
        results = []
        path = []
        
        def backtrack(start: int, remaining: int):
            if remaining == 0:
                results.append(list(path))
                return
                
            for i in range(start, len(candidates)):
                # Pruning: all subsequent candidates are even larger
                if candidates[i] > remaining:
                    break
                    
                path.append(candidates[i])
                # Stay at index i because elements can be reused
                backtrack(i, remaining - candidates[i])
                path.pop()
                
        backtrack(0, target)
        return results
```

#### Combination Sum II (Optimal Solution with Duplicate Pruning)
```python
class SolutionCombSumII:
    def combinationSum2(self, candidates: list[int], target: int) -> list[list[int]]:
        candidates.sort()
        results = []
        path = []
        
        def backtrack(start: int, remaining: int):
            if remaining == 0:
                results.append(list(path))
                return
                
            for i in range(start, len(candidates)):
                if candidates[i] > remaining:
                    break
                # Skip duplicate elements at the same tree level
                if i > start and candidates[i] == candidates[i - 1]:
                    continue
                    
                path.append(candidates[i])
                # Advance to i + 1 because elements cannot be reused
                backtrack(i + 1, remaining - candidates[i])
                path.pop()
                
        backtrack(0, target)
        return results
```

- **Time Complexity**: $O(2^N)$ worst-case for combinations where $N = \text{len(candidates)}$.
- **Space Complexity**: $O(\text{target} / \min(\text{candidates}))$ stack depth for Combination Sum I, $O(N)$ for Combination Sum II.

---

### 4.4 Live Verbalization Script

> *"For Combination Sum, the search space is a decision tree where each level chooses the next element to add to the running sum.
> 
> In Combination Sum I, because elements can be reused without limit, when I pick `candidates[i]`, I recurse passing the same index `i` rather than `i + 1`. To optimize pruning, I first sort the array. If `candidates[i] > remaining`, every subsequent number in the sorted list will also exceed the remaining target, allowing an immediate loop break.
> 
> In Combination Sum II, each number can only be picked once, and the input contains duplicates. I recurse with `i + 1` to forbid reuse. To prevent duplicate combination sets, I sort the input and apply the sibling pruning rule: `if i > start and candidates[i] == candidates[i - 1]: continue`. This guarantees that across identical numbers at any recursion depth, only the first candidate branches out."*

---

## 5. Benchmark Problem Deep Dive: LeetCode 46 & 47 — Permutations I & II

### 5.1 The Permutations Invariant: Order Matters

In permutations, `[1, 2]` is distinct from `[2, 1]`. Therefore:
1. Every recursive step must scan elements starting from **index 0**, not `start`.
2. An element cannot be placed if it has already been consumed in the current branch. We track this using a boolean array `visited` of size $N$.

---

### 5.2 Handling Duplicates in Permutations II: The `not visited[i-1]` Proof

Given `nums = [1, 1', 2]`, we want to generate `[1, 1', 2]` but avoid generating the identical permutation `[1', 1, 2]`.

**Mathematical Pruning Rule**:
```python
if visited[i] or (i > 0 and nums[i] == nums[i - 1] and not visited[i - 1]):
    continue
```

**Why `not visited[i - 1]`?**
- If `visited[i - 1] == True`: We are currently deeper down the recursive path of `nums[i - 1]`. Using `nums[i]` here is valid because `nums[i - 1]` was already chosen earlier in the prefix (e.g., placing `1'` after `1`).
- If `visited[i - 1] == False`: `nums[i - 1]` was just examined, completed its entire subtree, and was unchosen (backtracked). Choosing `nums[i]` now at the same position would explore the exact same permutations that `nums[i - 1]` already explored. Thus, we prune.

---

### 5.3 Complete Python Implementations

#### Permutations I (Optimal Visited Array Approach)
```python
class SolutionPermutationsI:
    def permute(self, nums: list[int]) -> list[list[int]]:
        results = []
        path = []
        visited = [False] * len(nums)
        
        def backtrack():
            if len(path) == len(nums):
                results.append(list(path))
                return
                
            for i in range(len(nums)):
                if visited[i]:
                    continue
                    
                visited[i] = True
                path.append(nums[i])
                backtrack()
                path.pop()
                visited[i] = False
                
        backtrack()
        return results
```

#### Permutations II (Optimal Duplicate Pruning)
```python
class SolutionPermutationsII:
    def permuteUnique(self, nums: list[int]) -> list[list[int]]:
        nums.sort()
        results = []
        path = []
        visited = [False] * len(nums)
        
        def backtrack():
            if len(path) == len(nums):
                results.append(list(path))
                return
                
            for i in range(len(nums)):
                if visited[i]:
                    continue
                # Enforce sequential order among duplicates:
                # If the previous identical element was NOT chosen in this branch,
                # choosing the current one would yield a duplicate permutation.
                if i > 0 and nums[i] == nums[i - 1] and not visited[i - 1]:
                    continue
                    
                visited[i] = True
                path.append(nums[i])
                backtrack()
                path.pop()
                visited[i] = False
                
        backtrack()
        return results
```

- **Time Complexity**: $O(N \cdot N!)$ — There are $N!$ permutations, and each takes $O(N)$ to copy to the output array.
- **Space Complexity**: $O(N)$ — For the recursion stack, `path` list, and `visited` boolean array.

---

### 5.4 Live Verbalization Script

> *"For Permutations, order matters, so every recursive step considers all numbers by looping from index 0. To prevent using the same number twice in a single permutation, I maintain a boolean array `visited`.
> 
> When the input contains duplicates (Permutations II), sorting the array is our first step.
> 
> To prune duplicate permutations, we check:
> `if visited[i] or (i > 0 and nums[i] == nums[i-1] and not visited[i-1]): continue`.
> 
> The condition `not visited[i-1]` is key: it means that the previous identical element was already processed and backtracked at this position. Branching on `nums[i]` now would redundantly recreate the subtree already covered by `nums[i-1]`. Enforcing that `nums[i-1]` must be actively marked as visited guarantees that duplicate elements are selected strictly in their original index order.
> 
> This generates all unique permutations in $O(N \cdot N!)$ time and $O(N)$ auxiliary space."*

