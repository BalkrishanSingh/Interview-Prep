# Domain 08: Greedy Algorithms — Intervals & Greedy Choice Invariants

---

## 1. Greedy vs. Dynamic Programming: Proof Requirements

A **Greedy Algorithm** builds up a solution piece by piece, always choosing the next piece that offers the most immediate (locally optimal) benefit, under the premise that local optimality leads to global optimality.

| Dimension | Greedy Algorithm | Dynamic Programming |
| :--- | :--- | :--- |
| **Choice Timing** | Commits to a choice irrevocably before solving subproblems. | Evaluates all possible choices and commits to the best after solving subproblems. |
| **Backtracking** | Never reconsiders prior decisions. | Retains subproblem states for trade-off evaluation. |
| **Proof Requirement** | Must prove **Greedy Choice Property** (e.g., via exchange argument) + Optimal Substructure. | Must prove **Optimal Substructure** + Overlapping Subproblems. |
| **Failure Mode** | Suboptimal local choices lead to dead-ends (e.g., 0/1 knapsack with fractional greedy heuristic). | High polynomial or exponential complexity if state space is unconstrained. |

---

## 2. Interval Scheduling: Start Time vs. End Time Sorting

Interval problems universally hinge on sorting. Choosing the correct sorting dimension is the critical design choice:

```
Pattern A: Maximize non-overlapping intervals (Activity Selection)
           ──> Sort by END TIME ascending.
           Invariant: Finishing earliest leaves maximum remaining time for subsequent intervals.

Pattern B: Merge overlapping intervals
           ──> Sort by START TIME ascending.
           Invariant: Overlapping intervals appear consecutively in the sorted order.
```

---

## 3. Benchmark Problem 1: LeetCode 435 — Non-Overlapping Intervals

### 3.1 Problem Statement Breakdown & Line-by-Line Annotations
> *"Given an array of intervals `intervals` where `intervals[i] = [start_i, end_i]`, return the minimum number of intervals you need to remove to make the rest of the intervals non-overlapping."*
- Let $N = \text{len(intervals)}$.
- Minimizing removals is mathematically equivalent to:
  $$\text{Min Removals} = N - \text{Max Compatible Non-Overlapping Intervals}$$

---

### 3.2 Constraints & Complexity Analysis
- $1 \le \text{intervals.length} \le 10^5$
- $\text{intervals}[i]\text{.length} == 2$
- $-5 \times 10^4 \le \text{start}_i < \text{end}_i \le 5 \times 10^4$
- Target Time: $O(N \log N)$ driven by sorting.
- Target Space: $O(1)$ auxiliary space (in-place sort).

---

### 3.3 Greedy Invariant & Exchange Argument Proof
- Sort intervals by their **end time** in non-decreasing order: `intervals.sort(key=lambda x: x[1])`.
- Pick the first interval (the one that ends earliest).
- For each subsequent interval $[s, e]$:
  - If $s \ge \text{end}_{\text{prev}}$: Compatible! Accept interval, update $\text{end}_{\text{prev}} = e$.
  - If $s < \text{end}_{\text{prev}}$: Overlap! Must remove one. Because the current interval ends after or at $\text{end}_{\text{prev}}$, greedily dropping the current interval preserves the earliest finish time. Increment removal count.

---

### 3.4 Dry Run Trace Table

Consider `intervals = [[1, 2], [2, 3], [3, 4], [1, 3]]`:
- After sorting by end time: `[[1, 2], [2, 3], [1, 3], [3, 4]]`.

| Interval `[s, e]` | `prev_end` | Overlap Condition ($s < \text{end}_{\text{prev}}$)? | Action | Removals |
| :--- | :--- | :--- | :--- | :--- |
| `[1, 2]` | $-\infty$ | $1 < -\infty$ (False) | Keep interval, `prev_end = 2` | 0 |
| `[2, 3]` | 2 | $2 < 2$ (False) | Keep interval, `prev_end = 3` | 0 |
| `[1, 3]` | 3 | $1 < 3$ (**True**) | Overlap! Drop interval | 1 |
| `[3, 4]` | 3 | $3 < 3$ (False) | Keep interval, `prev_end = 4` | 1 |

Result: `1` removal.

---

### 3.5 Complete Python Implementation

```python
class SolutionNonOverlapping:
    def eraseOverlapIntervals(self, intervals: list[list[int]]) -> int:
        if not intervals:
            return 0
            
        # Sort intervals by their end time ascending
        intervals.sort(key=lambda x: x[1])
        
        removals = 0
        prev_end = intervals[0][1]
        
        for i in range(1, len(intervals)):
            start, end = intervals[i]
            
            if start < prev_end:
                # Overlap detected: greedily drop the current interval
                removals += 1
            else:
                # No overlap: retain current interval and advance boundary
                prev_end = end
                
        return removals
```

---

## 4. Benchmark Problem 2: LeetCode 56 — Merge Intervals

### 4.1 Invariant: Sort by Start Time
To merge overlapping intervals, sort by `start` time ascending. This ensures that any interval that could merge with interval $i$ must appear immediately adjacent to it.

```python
class SolutionMergeIntervals:
    def merge(self, intervals: list[list[int]]) -> list[list[int]]:
        if not intervals:
            return []
            
        # Sort intervals by their start time
        intervals.sort(key=lambda x: x[0])
        
        merged = [intervals[0]]
        
        for curr_start, curr_end in intervals[1:]:
            last_end = merged[-1][1]
            
            if curr_start <= last_end:
                # Overlap exists: merge in-place by extending end boundary
                merged[-1][1] = max(last_end, curr_end)
            else:
                # Disjoint: push new interval to result
                merged.append([curr_start, curr_end])
                
        return merged
```
- **Time Complexity**: $O(N \log N)$ for sorting.
- **Space Complexity**: $O(N)$ for the merged output.

---

## 5. Benchmark Problem 3: LeetCode 134 — Gas Station

### 5.1 Problem Statement Breakdown & Line-by-Line Annotations
> *"There are `n` gas stations along a circular route, where the amount of gas at the $i$-th station is `gas[i]`."*
- Circular array of size $N$.

> *"You have a car with an unlimited gas tank and it costs `cost[i]` of gas to travel from the $i$-th station to its next $(i + 1)$-th station."*
- Net gas gained/spent at station $i$: $\Delta_i = \text{gas}[i] - \text{cost}[i]$.

> *"Return the starting gas station's index if you can travel around the circuit once in the clockwise direction, otherwise return `-1`. If there exists a solution, it is guaranteed to be unique."*

---

### 5.2 The Two Fundamental Invariants
1. **Global Invariant**: If $\sum_{i=0}^{N-1} \text{gas}[i] < \sum_{i=0}^{N-1} \text{cost}[i]$, total energy is insufficient to complete a loop, regardless of starting station $\implies$ return `-1`.
2. **Local Greedy Invariant**: If starting at station $A$ and running out of gas at station $B$ ($\sum_{i=A}^B (\text{gas}[i] - \text{cost}[i]) < 0$), then **no station between $A$ and $B$ inclusive** can serve as a valid starting point.
   - *Proof*: The tank accumulated a non-negative balance from $A$ up to any intermediate station $k$ ($A \le k < B$). Starting fresh at $k$ with tank 0 only worsens the deficit at $B$.
   - Therefore, the next potential starting point must be $B + 1$.

---

### 5.3 Complete Python Implementation

```python
class SolutionGasStation:
    def canCompleteCircuit(self, gas: list[int], cost: list[int]) -> int:
        total_tank = 0
        curr_tank = 0
        start_station = 0
        
        for i in range(len(gas)):
            diff = gas[i] - cost[i]
            total_tank += diff
            curr_tank += diff
            
            # If current tank drops below zero, station i cannot be traversed
            if curr_tank < 0:
                # Greedily reset starting station to i + 1
                start_station = i + 1
                curr_tank = 0
                
        # If total gas is less than total cost, completion is impossible
        return start_station if total_tank >= 0 else -1
```

- **Time Complexity**: $O(N)$ — Single linear pass.
- **Space Complexity**: $O(1)$ — Constant memory.

---

### 5.4 Live Verbalization Script

> *"For Gas Station, two mathematical conditions determine the solution:
> 
> First, if the total sum of `gas` is strictly less than the total sum of `cost`, completing the circuit is impossible, and we return `-1`.
> 
> Second, if total gas is sufficient, there is guaranteed to be a unique starting station. To find it in a single pass, I maintain `curr_tank` representing fuel along the current candidate trip.
> 
> I iterate through each station $i$. At each station, I add `gas[i] - cost[i]` to both `total_tank` and `curr_tank`. 
> If `curr_tank` falls below zero, starting anywhere from the current `start_station` up through $i$ is guaranteed to fail before reaching $i + 1$, because any intermediate station entered with a non-negative fuel balance already failed to bridge the gap.
> 
> Thus, I greedily reset `start_station` to $i + 1$ and reset `curr_tank` to 0.
> 
> At the end of the loop, if `total_tank >= 0`, `start_station` is our verified answer. This runs in $O(N)$ time and $O(1)$ auxiliary space."*
