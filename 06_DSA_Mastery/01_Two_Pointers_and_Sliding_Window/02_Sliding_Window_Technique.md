# Domain 01: Sliding Window Technique

---

## 1. Algorithmic Blueprint & Invariant

The **Sliding Window Technique** transforms problems involving contiguous subarrays or substrings from $O(N^2)$ or $O(N^3)$ brute force down to $O(N)$ linear time by maintaining a window that expands and contracts monotonically.

```
DYNAMIC SLIDING WINDOW:
Expand 'right' pointer to include new elements:
[ left ] ──────────────► [ right ]
   "p"     "w"     "w"     "k"     "e"     "w"
            │
            ▼ (When invariant is violated, shrink 'left' forward)
          [ left ] ─────► [ right ]
```

### The Two Sliding Window Paradigms:
1. **Fixed-Size Window ($K$)**: Window size is invariant (`right - left + 1 == K`). Slide by incrementing both `left` and `right` simultaneously.
2. **Dynamic / Variable-Size Window**:
   - **Expansion Phase**: Move `right` pointer forward one step at a time, incorporating the new element into state (hash map, frequency counter, running sum).
   - **Contraction Phase**: While the window state violates the problem constraint, advance `left` pointer forward, removing `arr[left]` from the state.
   - **Update Result**: When the window is valid, record the optimum metric (`max_len = max(max_len, right - left + 1)`).

---

## 2. Reusable Python Boilerplate Template

```python
def sliding_window_dynamic_template(s: str) -> int:
    """
    Template for dynamic sliding window.
    Time Complexity: O(N) - Each character processed at most twice (left and right)
    Space Complexity: O(min(N, Alphabet_Size))
    """
    window_state = {}
    left = 0
    best_result = 0
    
    for right, char in enumerate(s):
        # 1. Expand: Include s[right] in current window state
        window_state[char] = window_state.get(char, 0) + 1
        
        # 2. Contract: Shrink from left while constraint is violated
        while condition_violated(window_state):
            left_char = s[left]
            window_state[left_char] -= 1
            if window_state[left_char] == 0:
                del window_state[left_char]
            left += 1
            
        # 3. Update optimal result with valid window
        best_result = max(best_result, right - left + 1)
        
    return best_result
```

---

## 3. Benchmark Problem Deep Dive: LeetCode 3 — Longest Substring Without Repeating Characters

### 3.1 Problem Statement Breakdown & Line-by-Line Annotations
> *"Given a string `s`,"*
- Input string of arbitrary ASCII characters (letters, digits, symbols, spaces).

> *"find the length of the longest substring"*
- **Substring** requires strictly contiguous characters (unlike a subsequence which can omit characters).

> *"without repeating characters."*
- Every character within the window $[i, j]$ must be distinct: $\forall a, b \in [i, j], a \ne b \implies s[a] \ne s[b]$.
- Goal: $\max (j - i + 1)$ subject to the uniqueness invariant.

---

### 3.2 Constraints & Complexity Analysis
- $s\text{.length} \in [0, 5 \times 10^4]$
- $s$ consists of English letters, digits, symbols, and spaces.
- **Complexity Envelope**:
  - $N = 50,000 \implies N^2 = 2.5 \times 10^9$ operations $\implies$ $O(N^2)$ will trigger **TLE**.
  - A valid solution must run in $O(N)$ time.

---

### 3.3 Direction Exploration & Invariant Proof
- **Direction 1 (Brute Force)**:
  - Generate all possible substrings $O(N^2)$ and check uniqueness with a hash set $O(N)$.
  - Total Time: $O(N^3)$. Space: $O(\min(N, \Sigma))$.
- **Direction 2 (Basic Sliding Window with Set)**:
  - Maintain `left` and `right`. If $s[\text{right}]$ is in set, increment `left` and remove $s[\text{left}]$ until duplicate is gone.
  - Total Time: $O(2N) = O(N)$.
- **Direction 3 (Optimized Sliding Window with Last Seen Index Hash Map)**:
  - Store the most recent index where each character appeared: $\text{last\_seen}[\text{char}] = \text{index}$.
  - When $s[\text{right}]$ repeats at index `prev_idx`, we can jump `left` directly to $\max(\text{left}, \text{prev\_idx} + 1)$ in $O(1)$ without incrementally popping elements!
  - Total Time: Exactly $N$ iterations ($O(N)$). Space: $O(\min(N, \Sigma))$.

---

### 3.4 Step-by-Step Dry Run Trace Table
Input: `s = "abcabcbb"`

| Step | `right` | `char` | `last_seen` before step | Action / `left` update | Current Window | Window Len ($r - l + 1$) | `max_len` |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | 0 | 'a' | `{}` | Not in window $\implies$ `left = 0` | `"a"` | $0 - 0 + 1 = 1$ | **1** |
| 2 | 1 | 'b' | `{'a': 0}` | Not in window $\implies$ `left = 0` | `"ab"` | $1 - 0 + 1 = 2$ | **2** |
| 3 | 2 | 'c' | `{'a': 0, 'b': 1}` | Not in window $\implies$ `left = 0` | `"abc"` | $2 - 0 + 1 = 3$ | **3** |
| 4 | 3 | 'a' | `{'a': 0, 'b': 1, 'c': 2}` | Seen at 0 $\ge$ `left` $\implies$ `left = 0 + 1 = 1` | `"bca"` | $3 - 1 + 1 = 3$ | **3** |
| 5 | 4 | 'b' | `{'a': 3, 'b': 1, 'c': 2}` | Seen at 1 $\ge$ `left` $\implies$ `left = 1 + 1 = 2` | `"cab"` | $4 - 2 + 1 = 3$ | **3** |
| 6 | 5 | 'c' | `{'a': 3, 'b': 4, 'c': 2}` | Seen at 2 $\ge$ `left` $\implies$ `left = 2 + 1 = 3` | `"abc"` | $5 - 3 + 1 = 3$ | **3** |
| 7 | 6 | 'b' | `{'a': 3, 'b': 4, 'c': 5}` | Seen at 4 $\ge$ `left` $\implies$ `left = 4 + 1 = 5` | `"cb"` | $6 - 5 + 1 = 2$ | **3** |
| 8 | 7 | 'b' | `{'a': 3, 'b': 6, 'c': 5}` | Seen at 6 $\ge$ `left` $\implies$ `left = 6 + 1 = 7` | `"b"` | $7 - 7 + 1 = 1$ | **3** |

**Final Result**: `3` (represented by substrings `"abc"`, `"bca"`, `"cab"`).

---

### 3.5 Complete Python Implementation

```python
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        last_seen = {}
        left = 0
        max_length = 0
        
        for right, char in enumerate(s):
            # If char was seen within current window, jump left boundary
            if char in last_seen and last_seen[char] >= left:
                left = last_seen[char] + 1
            else:
                # Valid unique window: update maximum length
                window_len = right - left + 1
                if window_len > max_length:
                    max_length = window_len
                    
            # Record/update the most recent index of current character
            last_seen[char] = right
            
        return max_length
```

- **Time Complexity**: $O(N)$ — Single pass over string of length $N$.
- **Space Complexity**: $O(\min(N, \Sigma))$ — Hash map stores at most $\Sigma$ entries where $\Sigma$ is the alphabet size (128 for standard ASCII).

---

### 3.6 Live Verbalization Script

> *"To find the length of the longest substring with unique characters, a naive approach of checking every substring takes $O(N^3)$ time, which is too slow for $N = 50,000$.
> 
> Instead, I maintain a sliding window defined by boundaries `[left, right]`. As `right` advances, I track the most recent index of each character in a hash map `last_seen`.
> 
> If the incoming character at `right` has already been seen and its previous index lies within our current window (`last_seen[char] >= left`), our uniqueness invariant is violated. Rather than incrementing `left` by 1 repeatedly, I jump `left` directly past the prior occurrence: `left = last_seen[char] + 1`.
> 
> Because each character is inspected once by the `right` pointer and `left` only moves forward, the algorithm completes in strictly $O(N)$ time with $O(\min(N, \Sigma))$ auxiliary space."*
