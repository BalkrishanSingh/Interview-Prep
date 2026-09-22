# Domain 04: Recursion & Backtracking — Constraint Satisfaction (N-Queens)

---

## 1. Constraint Satisfaction & Pruning Mechanics

In Constraint Satisfaction Problems (CSPs), the state space is too vast to search without pruning. We prune invalid states immediately upon placing a piece by evaluating mathematical invariants:

```
N-QUEENS CHESSBOARD DIAGONAL INVARIANTS:
Positive Diagonals (/) : row + col is CONSTANT
Negative Diagonals (\) : row - col is CONSTANT

       col: 0   1   2   3
row 0:    [ 0,  1,  2,  3 ]  --> row + col values
row 1:    [ 1,  2,  3,  4 ]
row 2:    [ 2,  3,  4,  5 ]
row 3:    [ 3,  4,  5,  6 ]
```

By maintaining three boolean sets (`cols`, `pos_diag`, `neg_diag`), we verify whether square `(r, c)` is under attack in **$O(1)$ time** rather than scanning rows, columns, and diagonals ($O(N)$).

---

## 2. Benchmark Problem Deep Dive: LeetCode 51 — N-Queens

### 2.1 Problem Statement Breakdown & Line-by-Line Annotations
> *"The n-queens puzzle is the problem of placing `n` queens on an `n x n` chessboard such that no two queens attack each other."*
- A queen can attack any piece in the same row, column, or diagonal.
- Invariant 1: Exactly 1 queen must be placed in each row.
- Invariant 2: Exactly 1 queen must be placed in each column.
- Invariant 3: No two queens can share a positive (`r + c`) or negative (`r - c`) diagonal.

> *"Given an integer `n`, return all distinct solutions to the n-queens puzzle."*
- Formatted as a list of boards, where `'Q'` indicates a queen and `'.'` represents an empty space.

---

### 2.2 Constraints & Complexity Analysis
- $1 \le n \le 9$
- **State Space Reduction**:
  - Naive selection of $N$ squares from $N^2$: $\binom{N^2}{N} \implies \binom{81}{9} \approx 2.6 \times 10^{11}$ states.
  - One Queen per row constraint: $N^N = 9^9 \approx 3.8 \times 10^8$ states.
  - Backtracking with $O(1)$ diagonal pruning: Explores at most $O(N!)$ candidate leaves. For $N=9$: $9! = 362,880$ operations $\implies$ **Executes in $< 10$ milliseconds**.

---

### 2.3 Step-by-Step Dry Run for $N = 4$

```
row 0: Place Q at col 0 -> board = [(0,0)]
  row 1:
    col 0: Invalid (col 0 taken)
    col 1: Invalid (pos_diag: 1+0=1 vs 0+1=1 taken)
    col 2: Valid! Place Q at (1,2)
      row 2:
        col 0: Invalid (col 0 taken)
        col 1: Invalid (neg_diag: 2-1=1 vs 1-2=-1, pos_diag: 2+1=3 vs 1+2=3 taken)
        col 2: Invalid (col 2 taken)
        col 3: Invalid (pos_diag: 2+3=5, but attacks (0,0) neg_diag: 2-3=-1 vs 0-0=0)
        -> All columns in row 2 invalid! BACKTRACK to row 1.
    col 3: Valid! Place Q at (1,3)
      row 2:
        col 1: Valid! Place Q at (2,1)
          row 3:
            col 2: Valid! Place Q at (3,2)
            -> row == 4 == N: SOLUTION 1 FOUND! [".Q..", "...Q", "Q...", "..Q."]
```

---

### 2.4 Complete Python Implementation

```python
class Solution:
    def solveNQueens(self, n: int) -> list[list[str]]:
        cols = set()
        pos_diag = set()  # (r + c)
        neg_diag = set()  # (r - c)
        
        result = []
        board = [["."] * n for _ in range(n)]
        
        def backtrack(r: int):
            # Base Case: All n rows successfully populated with valid queens
            if r == n:
                solution = ["".join(row) for row in board]
                result.append(solution)
                return
                
            for c in range(n):
                # O(1) conflict validation
                if c in cols or (r + c) in pos_diag or (r - c) in neg_diag:
                    continue
                    
                # 1. Choose: Place Queen and record attacked lines
                cols.add(c)
                pos_diag.add(r + c)
                neg_diag.add(r - c)
                board[r][c] = "Q"
                
                # 2. Explore: Advance to next row
                backtrack(r + 1)
                
                # 3. Unchoose: Backtrack state
                cols.remove(c)
                pos_diag.remove(r + c)
                neg_diag.remove(r - c)
                board[r][c] = "."
                
        backtrack(0)
        return result
```

- **Time Complexity**: $O(N!)$ — The first row has $N$ choices, the second at most $N-2$, the third $N-4$, etc.
- **Space Complexity**: $O(N)$ — Sets and recursion stack bounded by $N$.

---

### 2.5 Live Verbalization Script

> *"To solve N-Queens without exploring invalid configurations:
> 
> Because each row must contain exactly one queen, I proceed row-by-row, reducing our search space from $O(N^2)$ board combinations to $O(N!)$.
> 
> For any square `(r, c)`, checking if it is under attack can be optimized to $O(1)$ time using three sets:
> - A `cols` set tracking occupied column indices.
> - A `pos_diag` set tracking occupied `r + c` sums.
> - A `neg_diag` set tracking occupied `r - c` differences.
> 
> When recursing on row `r`, we iterate through columns `c = 0` to `n - 1`. If `c`, `r + c`, or `r - c` is in our sets, we prune the branch immediately.
> 
> Otherwise, we place `'Q'`, record the sets, recurse on `r + 1`, and remove them upon backtrack. Reaching `r == n` indicates a complete, non-attacking board configuration."*
