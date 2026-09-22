# Domain 05: Trees — Traversals, Views & BFS/DFS Patterns

---

## 1. Tree Traversal Taxonomy

```
TREE TRAVERSALS:
├── DEPTH-FIRST SEARCH (DFS) [Call Stack / LIFO]
│   ├── Pre-Order  (Root -> Left -> Right)  [Cloning, Serialization]
│   ├── In-Order   (Left -> Root -> Right)  [Sorted BST retrieval]
│   └── Post-Order (Left -> Right -> Root)  [Bottom-up deletion, Subtree math]
└── BREADTH-FIRST SEARCH (BFS) [Queue / FIFO]
    ├── Level-Order Traversal               [Shortest distance, layer grouping]
    └── Coordinate BFS                      [Top View, Vertical Order View]
```

---

## 2. Reusable Traversal Templates

### 2.1 Level-Order Traversal (BFS with Queue)
```python
from collections import deque

def level_order_template(root) -> list[list[int]]:
    if not root:
        return []
        
    result = []
    queue = deque([root])
    
    while queue:
        level_size = len(queue)  # Snapshot queue length for current level
        current_level = []
        
        for _ in range(level_size):
            node = queue.popleft()
            current_level.append(node.val)
            
            if node.left:
                queue.append(node.left)
            if node.right:
                queue.append(node.right)
                
        result.append(current_level)
        
    return result
```

### 2.2 Iterative Inorder Traversal (Using Explicit Stack)
```python
def inorder_iterative(root) -> list[int]:
    result = []
    stack = []
    curr = root
    
    while curr or stack:
        # Traverse to the leftmost node
        while curr:
            stack.append(curr)
            curr = curr.left
            
        curr = stack.pop()
        result.append(curr.val)
        curr = curr.right  # Visit right subtree
        
    return result
```

---

## 3. Benchmark Problem Deep Dive: Binary Tree Vertical / Top View

### 3.1 Problem Statement Breakdown & Coordinate Invariant
- Assign coordinates to tree nodes:
  - Root at `(row = 0, col = 0)`.
  - Left child at `(row + 1, col - 1)`.
  - Right child at `(row + 1, col + 1)`.
- **Top View Invariant**: For every column `col`, the node with the **minimum row index** (the first node encountered during BFS) is visible from above.

```
          1 (col 0)
        /   \
(col -1) 2     3 (col 1)
         \
      (col 0) 4
Top View visible from above: [2, 1, 3] (Node 4 is hidden beneath 1)
```

---

### 3.2 Complete Python Implementation (Top View)

```python
from collections import deque

class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

def top_view(root: TreeNode | None) -> list[int]:
    if not root:
        return []
        
    # Maps col -> node.val (Stores strictly the FIRST node reaching this col)
    top_view_map = {}
    queue = deque([(root, 0)])  # (node, col)
    
    min_col = 0
    max_col = 0
    
    while queue:
        node, col = queue.popleft()
        
        # Invariant: BFS visits smaller row levels first.
        # First node recorded at 'col' is guaranteed the topmost visible node.
        if col not in top_view_map:
            top_view_map[col] = node.val
            min_col = min(min_col, col)
            max_col = max(max_col, col)
            
        if node.left:
            queue.append((node.left, col - 1))
        if node.right:
            queue.append((node.right, col + 1))
            
    # Output nodes ordered from leftmost column to rightmost column
    return [top_view_map[c] for c in range(min_col, max_col + 1)]
```

- **Time Complexity**: $O(N)$ — Every node is enqueued and processed once.
- **Space Complexity**: $O(N)$ — Queue and hash map hold $O(N)$ elements.

---

### 3.3 Live Verbalization Script

> *"To determine the top view of a binary tree, we model the tree on a 2D Cartesian coordinate plane. The root starts at horizontal column 0. A left child decrements the column by 1 (`col - 1`), while a right child increments it by 1 (`col + 1`).
> 
> A node is visible from the top if and only if it is the first node encountered in its vertical column as viewed from top to bottom.
> 
> Because Breadth-First Search (BFS) explores nodes strictly level-by-level (increasing depth `row`), the first time our BFS encounters any column `col`, that node is guaranteed to have the minimum depth.
> 
> We record `top_view_map[col] = node.val` only on the first visit, track the min and max column bounds, and return the values from `min_col` to `max_col` in $O(N)$ time and $O(N)$ space."*
