# Domain 05: Trees — Path & Diameter Problems (Bottom-Up Tree DP)

---

## 1. The Bottom-Up Tree DP Paradigm

Tree path problems frequently require computing an optimal turnaround path while communicating with ancestral nodes. This establishes a dual responsibility in post-order DFS:

```
                  Node (X)
                 /        \
   [Left Subtree]          [Right Subtree]
         │                        │
         ▼                        ▼
    Left Gain                Right Gain
    
1. Global Candidate Path through X (Turns at X):
   Path = Left Gain + Right Gain + X.val  (Updates global maximum)

2. Value Returned to Parent of X (Cannot branch both ways):
   Return = X.val + max(Left Gain, Right Gain)
```

---

## 2. Benchmark Problem Deep Dive: LeetCode 124 — Binary Tree Maximum Path Sum

### 2.1 Problem Statement Breakdown & Line-by-Line Annotations
> *"A path in a binary tree is a sequence of nodes where each pair of adjacent nodes in the sequence has an edge connecting them."*
- A path does not need to pass through the root.
- A path cannot branch into multiple forks (it is a simple path).

> *"A node can only appear in the sequence at most once."*
- No cycles or retraced edges.

> *"The path sum of a path is the sum of the node's values in the path."*
- Nodes can have negative values! If a subtree contributes a negative sum, it is optimal to **discard it entirely** (`max(0, subtree_gain)`).

> *"Given the `root` of a binary tree, return the maximum path sum of any non-empty path."*
- Goal: $\max \sum_{\text{path}} \text{node.val}$.

---

### 2.2 Constraints & Complexity Analysis
- Number of nodes $\in [1, 3 \times 10^4]$
- Node values in range $[-1000, 1000]$
- Maximum possible path sum: $3 \times 10^4 \times 1000 = 3 \times 10^7$ (fits in standard 32-bit signed integer).
- Target Time Complexity: $O(N)$.
- Target Auxiliary Space: $O(H)$ recursion stack.

---

### 2.3 Direction Exploration & Invariant Proof
- **The Negative Gain Pruning Invariant**:
  If a child subtree's maximum contribution is negative, adding it to the path will strictly reduce the total sum. Therefore, we clamp the child gain to zero:
  $$\text{effective\_gain} = \max(0, \text{dfs}(\text{child}))$$
- **Local Turnaround Calculation**:
  At current node `curr`, the maximum path that peaks at `curr` (using both left and right branches) is:
  $$\text{local\_max} = \text{curr.val} + \text{left\_gain} + \text{right\_gain}$$
  We update our global maximum: $\text{global\_max} = \max(\text{global\_max}, \text{local\_max})$.
- **Return Value to Parent**:
  The parent can only extend the path through *one* of `curr`'s subtrees:
  $$\text{return\_to\_parent} = \text{curr.val} + \max(\text{left\_gain}, \text{right\_gain})$$

---

### 2.4 Complete Python Implementation

```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

class Solution:
    def maxPathSum(self, root: TreeNode | None) -> int:
        # Initialize with negative infinity to handle trees consisting of all negative values
        max_sum = float('-inf')
        
        def max_gain(node: TreeNode | None) -> int:
            nonlocal max_sum
            if not node:
                return 0
                
            # Post-order: compute left and right subtree gains
            # Clamp to 0 to ignore paths that would decrease the total sum
            left_gain = max(0, max_gain(node.left))
            right_gain = max(0, max_gain(node.right))
            
            # 1. Evaluate candidate path turning at current node
            current_path_sum = node.val + left_gain + right_gain
            if current_path_sum > max_sum:
                max_sum = current_path_sum
                
            # 2. Return maximum single-branch path extendable by parent
            return node.val + max(left_gain, right_gain)
            
        max_gain(root)
        return int(max_sum)
```

- **Time Complexity**: $O(N)$ — Every node is visited once during post-order traversal.
- **Space Complexity**: $O(H)$ — Call stack bounded by tree height $H$.

---

### 2.5 Live Verbalization Script

> *"To find the maximum path sum across any non-empty path in a binary tree:
> 
> A path cannot fork; therefore, if a path uses both the left and right children of a node, that node must be the highest peak of that path.
> 
> I formulate a post-order recursive function `max_gain(node)` that calculates the maximum single-branch contribution this node can offer to its parent.
> 
> In post-order order, we first compute `left_gain` and `right_gain`. Because node values can be negative, if a subtree produces a negative sum, we clamp it to 0 using `max(0, gain)`.
> 
> At the current node, the maximum path that turns through it is `node.val + left_gain + right_gain`. We update our global tracker with this value.
> 
> Finally, we return `node.val + max(left_gain, right_gain)` to the caller, since the parent can only connect to one branch.
> 
> This post-order DP strategy processes every node in $O(N)$ time with $O(H)$ auxiliary stack space."*
