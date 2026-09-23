# Domain 05: Trees — Traversals, Views & BFS/DFS Patterns

---

## 1. Tree Traversal Taxonomy & Mental Models

A binary tree is a hierarchical, non-linear data structure. Navigating every node in a tree requires systematic traversal strategies classified into **Depth-First Search (DFS)** and **Breadth-First Search (BFS)**:

```
TREE TRAVERSALS:
├── DEPTH-FIRST SEARCH (DFS) [LIFO / Stack: Recursion or Explicit Stack]
│   ├── Pre-Order  (Root -> Left -> Right)  [Cloning, Serialization, Prefix expressions]
│   ├── In-Order   (Left -> Root -> Right)  [Sorted BST values, BST validation]
│   └── Post-Order (Left -> Right -> Root)  [Bottom-up computation, Heights, Tree deletion]
└── BREADTH-FIRST SEARCH (BFS) [FIFO / Queue]
    ├── Level-Order Traversal               [Layer-by-layer grouping, Shortest path in unweighted trees]
    └── Coordinate BFS                      [Vertical Order, Top View, Bottom View]
```

### When to Use Which Traversal:
| Traversal | Order of Visiting | Practical Applications |
| :--- | :--- | :--- |
| **Pre-Order** | Root $\to$ Left $\to$ Right | Copying/cloning a tree, prefix serialization/deserialization, directory tree printing. |
| **In-Order** | Left $\to$ Root $\to$ Right | Retrieving sorted keys from a Binary Search Tree (BST), checking if a tree is a valid BST. |
| **Post-Order** | Left $\to$ Right $\to$ Root | Bottom-up calculations: computing subtree size, tree height/diameter, deleting trees safely (children freed before parent). |
| **Level-Order** | Level by Level (Top to Bottom, Left to Right) | Finding shortest path/distance, layer grouping, connecting horizontal level siblings. |

---

## 2. Common TreeNode Definition

```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right
```

---

## 3. Canonical Recursive DFS Traversals

Recursion leverages the runtime's implicit **Call Stack**. Each recursive invocation pushes a stack frame containing the local node reference and returns to the parent frame upon completion.

### 3.1 Implementations

```python
# 1. Pre-Order Traversal: Root -> Left -> Right
def preorder_recursive(root: TreeNode | None) -> list[int]:
    result = []
    def dfs(node: TreeNode | None):
        if not node:
            return
        result.append(node.val)  # 1. Visit Root
        dfs(node.left)           # 2. Traverse Left Subtree
        dfs(node.right)          # 3. Traverse Right Subtree
    dfs(root)
    return result


# 2. In-Order Traversal: Left -> Root -> Right
def inorder_recursive(root: TreeNode | None) -> list[int]:
    result = []
    def dfs(node: TreeNode | None):
        if not node:
            return
        dfs(node.left)           # 1. Traverse Left Subtree
        result.append(node.val)  # 2. Visit Root
        dfs(node.right)          # 3. Traverse Right Subtree
    dfs(root)
    return result


# 3. Post-Order Traversal: Left -> Right -> Root
def postorder_recursive(root: TreeNode | None) -> list[int]:
    result = []
    def dfs(node: TreeNode | None):
        if not node:
            return
        dfs(node.left)           # 1. Traverse Left Subtree
        dfs(node.right)          # 2. Traverse Right Subtree
        result.append(node.val)  # 3. Visit Root
    dfs(root)
    return result
```

### 3.2 Complexity Analysis
- **Time Complexity**: $O(N)$ — Every node is visited exactly once.
- **Space Complexity**: $O(H)$ auxiliary stack space, where $H$ is tree height:
  - Best / Balanced Case: $H = O(\log N)$.
  - Worst / Skewed Case: $H = O(N)$ (e.g. linked-list degenerated tree).

---

## 4. Canonical Iterative DFS Traversals (Explicit Stack)

When tree depth is extreme ($N > 10^4$), recursion can cause a `RecursionError` (call stack overflow). Using an **explicit LIFO heap stack** provides total memory control.

### 4.1 Iterative Pre-Order Traversal (Root $\to$ Left $\to$ Right)
- **Invariant**: The stack maintains pending nodes. Because a stack is LIFO, pushing `right` before `left` guarantees that `left` is popped and processed first.

```python
def preorder_iterative(root: TreeNode | None) -> list[int]:
    if not root:
        return []
        
    result = []
    stack = [root]
    
    while stack:
        node = stack.pop()
        result.append(node.val)  # Process Root immediately
        
        # Push right FIRST so left is popped and processed FIRST
        if node.right:
            stack.append(node.right)
        if node.left:
            stack.append(node.left)
            
    return result
```

---

### 4.2 Iterative In-Order Traversal (Left $\to$ Root $\to$ Right)
- **Invariant**: Drill down as far left as possible, pushing every node onto the stack. When hitting `None`, the top of the stack is the leftmost unvisited node. Pop it, process it, and transition to its `right` child.

```python
def inorder_iterative(root: TreeNode | None) -> list[int]:
    result = []
    stack = []
    curr = root
    
    while curr or stack:
        # 1. Drill down to the leftmost leaf
        while curr:
            stack.append(curr)
            curr = curr.left
            
        # 2. Pop and process the current leftmost node
        curr = stack.pop()
        result.append(curr.val)
        
        # 3. Transition to the right subtree
        curr = curr.right
        
    return result
```

---

### 4.3 Iterative Post-Order Traversal (Left $\to$ Right $\to$ Root)
Post-order is the trickiest to write with an explicit stack because a parent node must be visited **after** both its left and right subtrees are fully traversed.

#### Method: Reverse-Modified Preorder (Optimal & Cleanest)
- Standard Pre-Order visits: `Root -> Left -> Right`.
- If we modify the push order to visit: `Root -> Right -> Left`.
- Reversing that result produces: `Left -> Right -> Root` (exact Post-Order)!

```python
def postorder_iterative(root: TreeNode | None) -> list[int]:
    if not root:
        return []
        
    result = []
    stack = [root]
    
    # Generate Root -> Right -> Left sequence
    while stack:
        node = stack.pop()
        result.append(node.val)
        
        # Push left FIRST so right is popped and processed FIRST
        if node.left:
            stack.append(node.left)
        if node.right:
            stack.append(node.right)
            
    # Reverse to obtain Left -> Right -> Root
    return result[::-1]
```

---

## 5. The Universal Iterative Template (Call-Stack Simulation)

Remembering 3 distinct while-loop architectures for iterative pre, in, and post order can be error-prone. The **Universal Visited Flag (Call-Stack Simulation)** pattern unifies all three into one identical loop structure.

### 5.1 The Concept
Each element in the stack is a tuple: `(node, visited: bool)`.
- If `visited == True`: The node is ready to be processed (`result.append(node.val)`).
- If `visited == False`: The node has not been expanded. Pop it and push its components **in reverse order of the desired traversal** (because a stack reverses order).

### 5.2 Unified Code

```python
def universal_traversal(root: TreeNode | None, order: str = "inorder") -> list[int]:
    if not root:
        return []
        
    result = []
    stack = [(root, False)]
    
    while stack:
        node, visited = stack.pop()
        if not node:
            continue
            
        if visited:
            result.append(node.val)
        else:
            if order == "preorder":
                # Desired: Root -> Left -> Right
                # Stack Push (Reverse): Right -> Left -> Root
                stack.append((node.right, False))
                stack.append((node.left, False))
                stack.append((node, True))
                
            elif order == "inorder":
                # Desired: Left -> Root -> Right
                # Stack Push (Reverse): Right -> Root -> Left
                stack.append((node.right, False))
                stack.append((node, True))
                stack.append((node.left, False))
                
            elif order == "postorder":
                # Desired: Left -> Right -> Root
                # Stack Push (Reverse): Root -> Right -> Left
                stack.append((node, True))
                stack.append((node.right, False))
                stack.append((node.left, False))
                
    return result
```

---

## 6. Level-Order Traversal (BFS with Queue)

Breadth-First Search processes nodes horizontal layer by horizontal layer using a **FIFO Queue**.

### Snapshotting Pattern for Layer Grouping
By recording `level_size = len(queue)` at the start of each level loop, we process exactly one tree depth per outer iteration:

```python
from collections import deque

def level_order(root: TreeNode | None) -> list[list[int]]:
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

- **Time Complexity**: $O(N)$ — Every node is enqueued and dequeued once.
- **Space Complexity**: $O(W)$ — Where $W$ is the maximum width of the tree ($W \le N/2$ for a complete binary tree $\implies O(N)$).

---

## 7. Benchmark Problem Deep Dive: Binary Tree Vertical / Top View

### 7.1 Coordinate Invariant Breakdown
We map the tree onto a 2D Cartesian coordinate plane:
- Root is positioned at `(row = 0, col = 0)`.
- Left child is at `(row + 1, col - 1)`.
- Right child is at `(row + 1, col + 1)`.

```
          1 (col 0)
        /   \
(col -1) 2     3 (col 1)
         \
      (col 0) 4
Top View visible from above: [2, 1, 3] (Node 4 is hidden beneath 1)
```

**The Deciding Invariant**:
A node is visible in the Top View if and only if it is the **first node encountered in its column** when scanning from top to bottom.
Because BFS visits nodes in strictly increasing row order (`row = 0, 1, 2, ...`), the first time BFS encounters any column `col`, that node is guaranteed to have the minimum depth.

---

### 7.2 Complete Python Implementation (Top View)

```python
from collections import deque

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
        
        # Invariant: BFS explores smaller row levels first.
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
- **Space Complexity**: $O(N)$ — Queue and hash map store at most $N$ elements.

---

### 7.3 Technical Articulation Script

> *"To determine the top view of a binary tree, we model the tree on a 2D Cartesian coordinate plane. The root begins at column 0. Branching to a left child decrements the column by 1, and branching to a right child increments the column by 1.
> 
> A node is visible from above if and only if it has the minimum row index for its column.
> 
> Because BFS explores nodes in strictly non-decreasing row order, the first node that our BFS reaches in any given column is guaranteed to be the topmost node. We record this first occurrence in a hash map, track the minimum and maximum column bounds, and reconstruct the result from leftmost to rightmost column in $O(N)$ time and $O(N)$ auxiliary space."*
