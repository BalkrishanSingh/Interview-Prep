# Domain 07: Dynamic Programming — DP on Trees & Directed Acyclic Graphs

---

## 1. Tree DP Mental Model & Post-Order Invariants

Dynamic Programming on Trees operates under a key structural guarantee: **subtrees rooted at different children are completely disjoint**. A decision made in the left subtree does not directly restrict choices in the right subtree except through their common parent.

```
                    Root (u)
                   /        \
             Left (v1)     Right (v2)
             [Disjoint]    [Disjoint]
```

### The Post-Order Tuple Pattern
Instead of allocating a global memoization table keyed by node references, the idiomatic tree DP approach uses a **Post-Order DFS** where each node returns a tuple of optimal values corresponding to mutually exclusive states:
$$\text{dfs}(\text{node}) \to (\text{state}_1, \text{state}_2, \dots)$$

---

## 2. Benchmark Problem Deep Dive: LeetCode 337 — House Robber III

### 2.1 Problem Statement Breakdown & Line-by-Line Annotations
> *"The thief has found himself a new place for his thievery again. There is only one entrance to this area, called `root`."*
- Tree-structured hierarchy.

> *"Besides the `root`, each house has one and only one parent house. After a tour, the smart thief realized that all houses in this place form a binary tree."*
- Connected acyclic graph where each node has at most two children (`left`, `right`).

> *"It will automatically contact the police if two directly-linked houses were broken into on the same night."*
- **Constraint**: If node $u$ is robbed, neither `u.left` nor `u.right` can be robbed.

> *"Return the maximum amount of money the thief can rob without alerting the police."*
- Objective: Find the Maximum Independent Weight Set on a tree.

---

### 2.2 Constraints & Complexity Analysis
- Number of nodes $N \in [1, 10^4]$
- $0 \le \text{Node.val} \le 10^4$
- Target Time: $O(N)$ — Every tree node visited exactly once in post-order traversal.
- Target Space: $O(H)$ where $H$ is tree height ($O(\log N)$ balanced, $O(N)$ degenerate).

---

### 2.3 State Tuple Definition & Recurrence

For every node `u`, define the state tuple:
$$(\text{with\_root}, \text{without\_root})$$
1. **$\text{with\_root}$**: The maximum money obtainable from the subtree rooted at `u` **including** `u`.
   - If `u` is robbed, neither `left` nor `right` can be robbed:
     $$\text{with\_root} = u.\text{val} + \text{left.without\_root} + \text{right.without\_root}$$
2. **$\text{without\_root}$**: The maximum money obtainable from the subtree rooted at `u` **excluding** `u`.
   - If `u` is skipped, its children may either be robbed or skipped (we choose the maximum independently for each child):
     $$\text{without\_root} = \max(\text{left.with}, \text{left.without}) + \max(\text{right.with}, \text{right.without})$$

**Base Case**:
- If `node is None`: return `(0, 0)`.

---

### 2.4 Dry Run Trace Table

Consider the tree:
```
       3
      / \
     2   3
      \   \
       3   1
```

| Node | Left Subtree `(with, without)` | Right Subtree `(with, without)` | `with_root` ($u.\text{val} + L_0 + R_0$) | `without_root` ($\max(L) + \max(R)$) | Output Tuple |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Leaf (3) under 2 | `(0, 0)` | `(0, 0)` | $3 + 0 + 0 = 3$ | $0 + 0 = 0$ | `(3, 0)` |
| Leaf (1) under 3 | `(0, 0)` | `(0, 0)` | $1 + 0 + 0 = 1$ | $0 + 0 = 0$ | `(1, 0)` |
| Node (2) | `(0, 0)` | `(3, 0)` | $2 + 0 + 0 = 2$ | $0 + \max(3, 0) = 3$ | `(2, 3)` |
| Node (3, right) | `(0, 0)` | `(1, 0)` | $3 + 0 + 0 = 3$ | $0 + \max(1, 0) = 1$ | `(3, 1)` |
| Root (3) | `(2, 3)` | `(3, 1)` | $3 + 3 + 1 = 7$ | $\max(2, 3) + \max(3, 1) = 3 + 3 = 6$ | `(7, 6)` |

Final answer: $\max(7, 6) = 7$.

---

### 2.5 Complete Python Implementation

```python
from typing import Optional

class TreeNode:
    def __init__(self, val: int = 0, left: Optional['TreeNode'] = None, right: Optional['TreeNode'] = None):
        self.val = val
        self.left = left
        self.right = right

class Solution:
    def rob(self, root: Optional[TreeNode]) -> int:
        # Returns tuple: (max_money_with_root, max_money_without_root)
        def dfs(node: Optional[TreeNode]) -> tuple[int, int]:
            if not node:
                return (0, 0)
                
            left_with, left_without = dfs(node.left)
            right_with, right_without = dfs(node.right)
            
            # Case 1: Rob current node -> cannot rob children
            with_root = node.val + left_without + right_without
            
            # Case 2: Do not rob current node -> children can be robbed or skipped
            without_root = max(left_with, left_without) + max(right_with, right_without)
            
            return (with_root, without_root)
            
        with_root, without_root = dfs(root)
        return max(with_root, without_root)
```

- **Time Complexity**: $O(N)$ — Exactly one post-order visit per node in the tree.
- **Space Complexity**: $O(H)$ — Recursion call stack proportional to tree height $H$ ($O(\log N)$ best case, $O(N)$ worst case).

---

### 2.6 Live Verbalization Script

> *"In House Robber III, the houses form a binary tree, and directly linked parent-child houses cannot both be robbed.
> 
> Because each subtree is an independent subproblem, I solve this using a post-order tree traversal that returns a 2-element tuple for each node: `(with_root, without_root)`.
> 
> The base case is a null node, which returns `(0, 0)`.
> 
> For any given node:
> First, I recursively solve its left and right children to obtain their respective tuples.
> If I choose to rob the current node, I earn its value, but I am forbidden from robbing its immediate children. Therefore, `with_root = node.val + left_without + right_without`.
> If I choose to skip the current node, I am free to either rob or skip each child independently. Therefore, `without_root = max(left_with, left_without) + max(right_with, right_without)`.
> 
> I return this pair up the recursion stack. At the tree root, the overall answer is simply `max(with_root, without_root)`.
> 
> This eliminates redundant recalculations, running in $O(N)$ time with $O(H)$ call stack space."*
