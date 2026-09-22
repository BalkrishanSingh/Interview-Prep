# Domain 07: Dynamic Programming — 2D String DP & Longest Common Subsequence (LCS)

---

## 1. 2D Sequence DP Mental Model

When solving optimization problems comparing two strings $S_1$ (length $M$) and $S_2$ (length $N$), state parameters naturally align with prefixes or suffixes of both strings:
- State $(i, j)$ represents subproblem on $S_1[0 \dots i-1]$ and $S_2[0 \dots j-1]$.
- Table dimension: $(M + 1) \times (N + 1)$ with 1-based indexing to simplify empty string base cases.

```
       ""    a    c    e
  ""  [ 0 ][ 0 ][ 0 ][ 0 ]
  a   [ 0 ][ 1 ][ 1 ][ 1 ]
  b   [ 0 ][ 1 ][ 1 ][ 1 ]
  c   [ 0 ][ 1 ][ 2 ][ 2 ]
  d   [ 0 ][ 1 ][ 2 ][ 2 ]
  e   [ 0 ][ 1 ][ 2 ][ 3 ]
```

---

## 2. Benchmark Problem 1: LeetCode 1143 — Longest Common Subsequence

### 2.1 Problem Statement Breakdown & Line-by-Line Annotations
> *"Given two strings `text1` and `text2`, return the length of their longest common subsequence. If there is no common subsequence, return 0."*
- A subsequence is derived by deleting zero or more characters without changing relative order.
- Objective: Maximize length of subsequence present in both strings.

---

### 2.2 Constraints & Complexity Analysis
- $1 \le \text{text1.length}, \text{text2.length} \le 1,000$
- Target Time: $O(M \times N) \le 1,000 \times 1,000 = 10^6$ operations (instantaneous in ~50 ms).
- Target Space: $O(M \times N)$ for 2D table, or $O(\min(M, N))$ using two rolling 1D rows.

---

### 2.3 Recurrence Relation & Transition Logic
Let $\text{dp}[i][j]$ be the LCS length for prefixes $\text{text1}[0 \dots i-1]$ and $\text{text2}[0 \dots j-1]$:
1. **Characters Match** (`text1[i-1] == text2[j-1]`):
   - Include this character in LCS and transition diagonally:
     $$\text{dp}[i][j] = 1 + \text{dp}[i-1][j-1]$$
2. **Characters Do Not Match** (`text1[i-1] != text2[j-1]`):
   - Best result comes from either dropping `text1[i-1]` or dropping `text2[j-1]`:
     $$\text{dp}[i][j] = \max(\text{dp}[i-1][j], \text{dp}[i][j-1])$$

**Base Cases**:
- $\text{dp}[0][j] = 0$ for all $0 \le j \le N$ (LCS with empty string is 0).
- $\text{dp}[i][0] = 0$ for all $0 \le i \le M$.

---

### 2.4 Complete Python Implementations

#### Top-Down (Memoization)
```python
from functools import lru_cache

class SolutionTopDown:
    def longestCommonSubsequence(self, text1: str, text2: str) -> int:
        m, n = len(text1), len(text2)
        
        @lru_cache(maxsize=None)
        def dp(i: int, j: int) -> int:
            if i >= m or j >= n:
                return 0
                
            if text1[i] == text2[j]:
                return 1 + dp(i + 1, j + 1)
            else:
                return max(dp(i + 1, j), dp(i, j + 1))
                
        return dp(0, 0)
```

#### Bottom-Up (2D Tabulation)
```python
class Solution2D:
    def longestCommonSubsequence(self, text1: str, text2: str) -> int:
        m, n = len(text1), len(text2)
        dp = [[0] * (n + 1) for _ in range(m + 1)]
        
        for i in range(1, m + 1):
            for j in range(1, n + 1):
                if text1[i - 1] == text2[j - 1]:
                    dp[i][j] = 1 + dp[i - 1][j - 1]
                else:
                    dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])
                    
        return dp[m][n]
```

#### Space-Optimized ($O(\min(M, N))$ Space)
```python
class Solution:
    def longestCommonSubsequence(self, text1: str, text2: str) -> int:
        # Ensure text2 is the shorter string to minimize row memory
        if len(text1) < len(text2):
            text1, text2 = text2, text1
            
        m, n = len(text1), len(text2)
        prev = [0] * (n + 1)
        
        for i in range(1, m + 1):
            curr = [0] * (n + 1)
            for j in range(1, n + 1):
                if text1[i - 1] == text2[j - 1]:
                    curr[j] = 1 + prev[j - 1]
                else:
                    curr[j] = max(prev[j], curr[j - 1])
            prev = curr
            
        return prev[n]
```

---

## 3. Benchmark Problem 2: LeetCode 72 — Edit Distance

### 3.1 Problem Statement Breakdown
> *"Given two strings `word1` and `word2`, return the minimum number of operations required to convert `word1` to `word2`."*
Allowable operations:
1. **Insert** a character.
2. **Delete** a character.
3. **Replace** a character.

---

### 3.2 Decision Recurrence
Let $\text{dp}[i][j]$ be the minimum operations to convert `word1[0...i-1]` to `word2[0...j-1]`:
- If `word1[i-1] == word2[j-1]`:
  $$\text{dp}[i][j] = \text{dp}[i-1][j-1]$$
- If `word1[i-1] != word2[j-1]`:
  $$\text{dp}[i][j] = 1 + \min \begin{cases}
  \text{dp}[i][j-1] & \text{(Insert into word1)} \\
  \text{dp}[i-1][j] & \text{(Delete from word1)} \\
  \text{dp}[i-1][j-1] & \text{(Replace in word1)}
  \end{cases}$$

**Base Cases**:
- $\text{dp}[i][0] = i$ (deleting all $i$ characters).
- $\text{dp}[0][j] = j$ (inserting all $j$ characters).

```python
class SolutionEditDistance:
    def minDistance(self, word1: str, word2: str) -> int:
        m, n = len(word1), len(word2)
        dp = [[0] * (n + 1) for _ in range(m + 1)]
        
        for i in range(m + 1):
            dp[i][0] = i
        for j in range(n + 1):
            dp[0][j] = j
            
        for i in range(1, m + 1):
            for j in range(1, n + 1):
                if word1[i - 1] == word2[j - 1]:
                    dp[i][j] = dp[i - 1][j - 1]
                else:
                    dp[i][j] = 1 + min(
                        dp[i][j - 1],      # Insert
                        dp[i - 1][j],      # Delete
                        dp[i - 1][j - 1]   # Replace
                    )
                    
        return dp[m][n]
```

---

### 3.3 Live Verbalization Script

> *"For Longest Common Subsequence, we want to find the length of the longest sequence of characters appearing in the same relative order in both `text1` and `text2`.
> 
> I model this with a 2D dynamic programming grid where `dp[i][j]` represents the LCS length between prefix `text1[0...i-1]` and prefix `text2[0...j-1]`.
> 
> The base cases are when either prefix is empty, which corresponds to row 0 and column 0, all initialized to 0.
> 
> For each pair of characters $(i, j)$:
> If `text1[i-1] == text2[j-1]`, both strings share this character, so the LCS increases by 1 over the diagonal subproblem: `1 + dp[i-1][j-1]`.
> If they do not match, the character in one string cannot pair with the current character of the other. The best answer is the maximum between dropping `text1[i-1]` (`dp[i-1][j]`) and dropping `text2[j-1]` (`dp[i][j-1]`).
> 
> Because computing the current row requires only the previous row, we can space-optimize from $O(M \times N)$ to $O(\min(M, N))$ using two rolling 1D arrays.
> 
> The time complexity is $O(M \times N)$ and the space complexity is $O(\min(M, N))$."*

---

## 4. Benchmark Problem Deep Dive: LeetCode 115 — Distinct Subsequences

### 4.1 Problem Statement Breakdown & Line-by-Line Annotations
> *"Given two strings `s` and `t`, return the number of distinct subsequences of `s` which equals `t`."*
- $S$ is the source string (length $M$), $T$ is the target pattern (length $N$).
- We are counting the number of ways to form $T$ by deleting characters from $S$.

---

### 4.2 Recurrence Relation & Branching Invariants

Let $\text{dp}[i][j]$ be the number of distinct subsequences of prefix $s[0 \dots i-1]$ that equal prefix $t[0 \dots j-1]$:
1. **Characters Match** (`s[i - 1] == t[j - 1]`):
   - We have two options:
     - **Option A (Match)**: Use $s[i-1]$ to match $t[j-1]$. Remaining problem is matching $s[0 \dots i-2]$ with $t[0 \dots j-2]$: $\text{dp}[i-1][j-1]$.
     - **Option B (Skip)**: Do NOT use $s[i-1]$, and look for other matches for $t[0 \dots j-1]$ earlier in $s[0 \dots i-2]$: $\text{dp}[i-1][j]$.
     $$\text{dp}[i][j] = \text{dp}[i-1][j-1] + \text{dp}[i-1][j]$$
2. **Characters Do Not Match** (`s[i - 1] != t[j - 1]`):
   - We are forced to skip $s[i-1]$:
     $$\text{dp}[i][j] = \text{dp}[i-1][j]$$

**Base Cases**:
- $\text{dp}[i][0] = 1$ for all $0 \le i \le M$: An empty target string $T$ can always be formed in exactly 1 way (by deleting all characters of $S$).
- $\text{dp}[0][j] = 0$ for $j > 0$: A non-empty target $T$ cannot be formed from an empty source $S$.

---

### 4.3 Complete Python Implementation ($O(N)$ Space Optimized)

```python
class SolutionDistinctSubseq:
    def numDistinct(self, s: str, t: str) -> int:
        m, n = len(s), len(t)
        
        # dp[j] stores count of distinct subsequences of s seen so far matching t[0...j-1]
        dp = [0] * (n + 1)
        dp[0] = 1  # Empty target string has 1 match
        
        for i in range(1, m + 1):
            # Iterate j backwards to preserve values from previous row i-1
            for j in range(n, 0, -1):
                if s[i - 1] == t[j - 1]:
                    dp[j] += dp[j - 1]
                    
        return dp[n]
```

- **Time Complexity**: $O(M \times N)$.
- **Space Complexity**: $O(N)$ — 1D array of length $|T| + 1$.

---

### 4.4 Live Verbalization Script

> *"For Distinct Subsequences, we want to count how many subsequences of string $S$ match string $T$.
> 
> I define `dp[j]` as the number of ways prefix $S$ matches prefix $T[0...j-1]$.
> 
> Base case: `dp[0] = 1` because there is exactly one way to form an empty string $T$.
> 
> For each character in $S$:
> I iterate through $T$ backwards from index $N$ down to 1 (mirroring the 0/1 knapsack space optimization).
> If `s[i-1] == t[j-1]`, we have two independent choices: match the current character (adding `dp[j-1]` combinations) or skip it (retaining the existing `dp[j]` combinations from earlier in $S$). Therefore, `dp[j] += dp[j-1]`.
> If they do not match, the character cannot be used, so `dp[j]` carries over unchanged.
> 
> This runs in $O(M \times N)$ time with $O(N)$ space."*

