# Domain 05: Trees — Binary Search Tree (BST) Properties & Validation

---

## 1. BST Structural Invariants

A **Binary Search Tree** is a binary tree where every node satisfies the global binary search property:
$$\forall u \in \text{LeftSubtree}(x), u.\text{val} < x.\text{val} \qquad \text{and} \qquad \forall v \in \text{RightSubtree}(x), v.\text{val} > x.\text{val}$$

```
VALID BST:                           INVALID BST:
        10                                  10
       /  \                                /  \
      5    15                             5    15
     / \                                      /  \
    2   8                                    6    20
                                             ▲
                                (6 < 10! Violates root's left boundary)
```

> [!WARNING]
> **The Local Check Fallacy**:
> Checking only immediate children (`node.left.val < node.val < node.right.val`) is **incorrect**! In the invalid tree above, node 15 satisfies $6 < 15 < 20$ locally; but globally, 6 violates the ancestral condition that all nodes in 10's right subtree must exceed 10.

---

## 2. Lowest Common Ancestor (LCA) in a BST

In a BST, finding the LCA of nodes $p$ and $q$ runs in $O(H)$ time without hashing or backtracking:

```python
def lowestCommonAncestor(root: TreeNode, p: TreeNode, q: TreeNode) -> TreeNode:
    curr = root
    while curr:
        if p.val < curr.val and q.val < curr.val:
            curr = curr.left   # Both nodes reside in left subtree
        elif p.val > curr.val and q.val > curr.val:
            curr = curr.right  # Both nodes reside in right subtree
        else:
            return curr        # Split point reached: curr is the LCA!
    return None
```

---

## 3. Benchmark Problem Deep Dive: LeetCode 98 — Validate Binary Search Tree

### 3.1 Problem Statement Breakdown & Line-by-Line Annotations
> *"Given the `root` of a binary tree, determine if it is a valid binary search tree (BST)."*
- A valid BST requires:
  - The left subtree of a node contains only nodes with keys **less than** the node's key.
  - The right subtree of a node contains only nodes with keys **greater than** the node's key.
  - Both the left and right subtrees must also be binary search trees.
- Note: Strict inequality ($<$ and $>$). Duplicate keys are invalid.

---

### 3.2 Constraints & Complexity Analysis
- Number of nodes $\in [1, 10^4]$
- Node values in range $[-2^{31}, 2^{31} - 1]$ (Requires 64-bit integer limits: $-\infty$ to $+\infty$).
- Target Time: $O(N)$ (every node visited once).
- Auxiliary Space: $O(H)$ where $H$ is tree height (stack space).

---

### 3.3 Direction Exploration & Invariant Proof
- **Direction 1 (Valid Range Interval Propagation)**:
  - Every node $x$ is constrained by an open interval: $(\text{low}, \text{high})$.
  - Root interval: $(-\infty, +\infty)$.
  - When branching left: node value becomes the new upper bound $\implies (\text{low}, \text{node.val})$.
  - When branching right: node value becomes the new lower bound $\implies (\text{node.val}, \text{high})$.
  - If any node violates $\text{low} < \text{node.val} < \text{high}$, the tree is invalid.
- **Direction 2 (Inorder Monotonicity Check)**:
  - Inorder traversal of a valid BST must yield a strictly increasing sequence.
  - Maintain `prev_val`. If current node $\le \text{prev\_val}$, return `False`.

---

### 3.4 Complete Python Implementation (Range Propagation)

```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

class Solution:
    def isValidBST(self, root: TreeNode | None) -> bool:
        def validate(node: TreeNode | None, low: float, high: float) -> bool:
            # An empty node is trivially a valid BST
            if not node:
                return True
                
            # Current node must lie strictly within inherited bounds
            if not (low < node.val < high):
                return False
                
            # Left child bounded by (low, node.val)
            # Right child bounded by (node.val, high)
            return (validate(node.left, low, node.val) and 
                    validate(node.right, node.val, high))
                    
        return validate(root, float('-inf'), float('inf'))
```

- **Time Complexity**: $O(N)$ — Inspects each node exactly once.
- **Space Complexity**: $O(H)$ — Recursion call stack bounded by tree height ($O(\log N)$ balanced, $O(N)$ worst-case degenerate linked list).

---

### 3.5 Live Verbalization Script

> *"To validate a Binary Search Tree, simply checking whether a node is greater than its left child and smaller than its right child is insufficient because BST invariants apply globally across all subtrees.
> 
> Instead, I define a recursive helper `validate(node, low, high)` where `(low, high)` represents the allowable open interval for the current node's value.
> 
> The root begins with interval `(-inf, +inf)`. When recursing into the left child, its value must still exceed `low`, but can never reach or exceed `node.val`, yielding `(low, node.val)`. Conversely, the right child inherits `(node.val, high)`.
> 
> If any node violates `low < node.val < high`, we immediately prune and return `False`.
> 
> This guarantees complete validation across all $N$ nodes in $O(N)$ time with $O(H)$ memory stack depth."*
