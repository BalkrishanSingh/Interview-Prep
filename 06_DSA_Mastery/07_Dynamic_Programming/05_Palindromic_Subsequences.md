# Domain 07: Dynamic Programming — Palindromes & Interval DP

---

## 1. Palindrome Paradigms: Substrings vs. Subsequences

When addressing palindrome problems, the structure of the allowed operations dictates the optimal algorithmic paradigm:

| Problem Type | Contiguity | Paradigm | Optimal Complexity | Typical Benchmark |
| :--- | :--- | :--- | :--- | :--- |
| **Palindromic Substrings** | Contiguous | Two-Pointer Expansion from Centers | $O(N^2)$ time, $O(1)$ space | LC 5, LC 647 |
| **Palindromic Subsequences** | Non-contiguous | Interval DP / LCS Reduction | $O(N^2)$ time, $O(N)$ space | LC 516 |

---

## 2. Benchmark Problem 1: LeetCode 647 — Palindromic Substrings (Expand Around Center)

### 2.1 Problem Statement & Annotations
> *"Given a string `s`, return the number of palindromic substrings in it."*
- Contiguous substrings only.
- A single character is always a palindrome.
- Every substring is uniquely identified by its center.

### 2.2 Expanding Around Centers ($O(N^2)$ Time, $O(1)$ Space)
A string of length $N$ has $2N - 1$ potential centers:
- $N$ odd-length centers: centered at index $i$ ($i, i$).
- $N - 1$ even-length centers: centered between index $i$ and $i+1$ ($i, i+1$).

```python
class SolutionPalindromicSubstrings:
    def countSubstrings(self, s: str) -> int:
        n = len(s)
        count = 0
        
        def expand(left: int, right: int) -> int:
            pal_count = 0
            while left >= 0 and right < n and s[left] == s[right]:
                pal_count += 1
                left -= 1
                right += 1
            return pal_count
            
        for i in range(n):
            count += expand(i, i)      # Odd length palindromes
            count += expand(i, i + 1)  # Even length palindromes
            
        return count
```
- **Time Complexity**: $O(N^2)$ — Each expansion takes $O(N)$, evaluated at $2N - 1$ centers.
- **Space Complexity**: $O(1)$ — No extra memory or recursion stack.

---

## 3. Benchmark Problem 2: LeetCode 516 — Longest Palindromic Subsequence

### 3.1 Problem Statement Breakdown & Line-by-Line Annotations
> *"Given a string `s`, find the longest palindromic subsequence's length in `s`."*
- A subsequence is not required to be contiguous.
- Characters can be deleted without changing relative order.

---

### 3.2 Interval DP Formulation

Let $\text{dp}[i][j]$ be the length of the longest palindromic subsequence in substring $s[i \dots j]$ ($0 \le i \le j < N$).

#### Recurrence Relation
1. **Endpoints match** (`s[i] == s[j]`):
   - Both characters are included at opposite ends of the palindrome:
     $$\text{dp}[i][j] = 2 + \text{dp}[i+1][j-1]$$
   - Special boundary when $i == j$: $\text{dp}[i][i] = 1$.
2. **Endpoints do not match** (`s[i] != s[j]`):
   - We must discard either $s[i]$ or $s[j]$:
     $$\text{dp}[i][j] = \max(\text{dp}[i+1][j], \text{dp}[i][j-1])$$

#### Diagonal Evaluation Order
Because $\text{dp}[i][j]$ depends on $\text{dp}[i+1][j-1]$ (inner shorter substring), we must evaluate subproblems in increasing order of substring length $L \in [1 \dots N]$ or by decreasing index $i$ from $N-1$ down to 0.

---

### 3.3 Dry Run Trace Table

Consider $s = \text{"bbbab"}$ (length $N = 5$):

```
       b    b    b    a    b
       0    1    2    3    4
  0  [ 1 ][ 2 ][ 3 ][ 3 ][ 4 ]
  1  [   ][ 1 ][ 2 ][ 2 ][ 3 ]
  2  [   ][   ][ 1 ][ 1 ][ 3 ]
  3  [   ][   ][   ][ 1 ][ 1 ]
  4  [   ][   ][   ][   ][ 1 ]
```
Result: $\text{dp}[0][4] = 4$ (subsequence `"bbbb"`).

---

### 3.4 Complete Python Implementations

#### Method A: Interval DP Bottom-Up Tabulation (1D Space-Optimized)
```python
class Solution:
    def longestPalindromeSubseq(self, s: str) -> int:
        n = len(s)
        # dp[j] represents the longest palindromic subsequence in s[i...j]
        dp = [0] * n
        
        # Traverse i backwards from n-1 to 0
        for i in range(n - 1, -1, -1):
            dp[i] = 1  # Base case: single character
            prev_diag = 0  # Represents dp[i+1][j-1]
            
            for j in range(i + 1, n):
                temp = dp[j]  # dp[i+1][j] before being overwritten
                
                if s[i] == s[j]:
                    dp[j] = 2 + prev_diag
                else:
                    dp[j] = max(dp[j], dp[j - 1])
                    
                prev_diag = temp
                
        return dp[n - 1]
```

#### Method B: Equivalence Reduction to LCS
A string's Longest Palindromic Subsequence is identical to the **Longest Common Subsequence of $s$ and its reverse $s^R$**:
$$\text{LPS}(s) = \text{LCS}(s, s[::-1])$$

```python
class SolutionLCS:
    def longestPalindromeSubseq(self, s: str) -> int:
        rev_s = s[::-1]
        n = len(s)
        prev = [0] * (n + 1)
        
        for i in range(1, n + 1):
            curr = [0] * (n + 1)
            for j in range(1, n + 1):
                if s[i - 1] == rev_s[j - 1]:
                    curr[j] = 1 + prev[j - 1]
                else:
                    curr[j] = max(prev[j], curr[j - 1])
            prev = curr
            
        return prev[n]
```

- **Time Complexity**: $O(N^2)$ — Nested loops over string length $N$.
- **Space Complexity**: $O(N)$ — 1D array of length $N$.

---

### 3.5 Live Verbalization Script

> *"For Longest Palindromic Subsequence, we want to find the maximum length subsequence in string $s$ that reads the same forwards and backwards.
> 
> I define `dp[i][j]` as the length of the longest palindromic subsequence in the substring `s[i...j]`. 
> 
> The base case is a single character where `dp[i][i] = 1`.
> 
> For any substring with endpoints $i$ and $j$:
> If `s[i] == s[j]`, both matching outer characters contribute 2 to the palindrome, transitioning to `2 + dp[i+1][j-1]`.
> If `s[i] != s[j]`, the optimal palindrome must either exclude `s[i]` or exclude `s[j]`, so `dp[i][j] = max(dp[i+1][j], dp[i][j-1])`.
> 
> Because each state `(i, j)` depends on states with shorter length or row $i+1$, I iterate the outer pointer $i$ backwards from $N-1$ down to 0 and the inner pointer $j$ forwards from $i+1$ up to $N-1$. This allows condensing the 2D grid into a 1D array of size $N$ using a temporary variable to preserve the diagonal `dp[i+1][j-1]`.
> 
> Alternatively, the problem can be solved by computing the Longest Common Subsequence between `s` and its reverse `s[::-1]`.
> 
> Both formulations run in $O(N^2)$ time and $O(N)$ auxiliary space."*
