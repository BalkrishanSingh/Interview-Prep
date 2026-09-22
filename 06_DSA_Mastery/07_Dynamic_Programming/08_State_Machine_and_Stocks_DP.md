# Domain 07: Dynamic Programming — State Machine DP & Stock Trading

---

## 1. State Machine DP Mental Model

In **State Machine Dynamic Programming**, the subproblem cannot be described by an array index alone because future decisions depend on the **current discrete state** of the system (e.g., holding a stock vs. empty-handed vs. cooldown).

We model the problem as a **Finite State Machine (FSM)**:
- **Nodes**: Mutually exclusive states.
- **Edges**: Directed transitions representing decisions (Buy, Sell, Rest/Hold), each carrying a cost or profit delta.
- **Optimal Substructure**: The maximum profit in state $S$ at day $i$ is the maximum of all valid incoming transitions from day $i-1$.

---

## 2. Benchmark Problem 1: LeetCode 309 — Stock with Cooldown

### 2.1 Problem Statement Breakdown & Line-by-Line Annotations
> *"You are given an array `prices` where `prices[i]` is the price of a given stock on the $i$-th day."*
- Price series $P_0, P_1, \dots, P_{N-1}$.

> *"Find the maximum profit you can achieve. You may complete as many transactions as you like with the following restrictions:"*
- Can only hold at most 1 share at any time.

> *"After you sell your stock, you cannot buy stock on the next day (i.e., cooldown one day)."*
- Mandatory 1-day cooldown state between a Sell action and the next Buy action.

---

### 2.2 The 3-State Transition Graph

```
           ┌─────────── Hold (Rest) ───────────┐
           ▼                                   │
      ┌─────────┐       Sell (+price)     ┌─────────┐
      │  HELD   │ ──────────────────────> │  SOLD   │
      └─────────┘                         └─────────┘
           ▲                                   │
           │ Buy (-price)                      │ Cooldown (1 day)
           │                                   ▼
      ┌─────────┐ <─── Rest (Stay Ready) ── ┌─────────┐
      │  RESET  │                           │ (cooldown)
      └─────────┘ ───────────────────────── └─────────┘
```

1. **State `held`**: Currently owns 1 share of stock.
   - Enter by: Continuing to hold from yesterday, OR buying today from `reset` state.
   $$\text{held}[i] = \max(\text{held}[i-1], \text{reset}[i-1] - \text{price})$$
2. **State `sold`**: Just sold stock today; enters mandatory cooldown.
   - Enter by: Selling the share owned in `held` yesterday.
   $$\text{sold}[i] = \text{held}[i-1] + \text{price}$$
3. **State `reset`**: Empty-handed and ready to buy (cooldown finished or resting).
   - Enter by: Resting from yesterday's `reset`, OR recovering from yesterday's `sold` state.
   $$\text{reset}[i] = \max(\text{reset}[i-1], \text{sold}[i-1])$$

**Base Cases (Day 0)**:
- $\text{held}[0] = -\text{price}[0]$ (buying first stock).
- $\text{sold}[0] = -\infty$ (cannot sell on day 0).
- $\text{reset}[0] = 0$ (resting on day 0 with 0 profit).

---

### 2.3 Complete Python Implementation ($O(1)$ Space)

```python
class SolutionStockWithCooldown:
    def maxProfit(self, prices: list[int]) -> int:
        if not prices:
            return 0
            
        held = -prices[0]
        sold = float('-inf')
        reset = 0
        
        for price in prices[1:]:
            prev_held = held
            prev_sold = sold
            prev_reset = reset
            
            held = max(prev_held, prev_reset - price)
            sold = prev_held + price
            reset = max(prev_reset, prev_sold)
            
        # Maximum profit cannot end in held state
        return max(sold, reset)
```

- **Time Complexity**: $O(N)$ — Single linear pass through prices.
- **Space Complexity**: $O(1)$ — 3 rolling state variables.

---

### 2.4 Live Verbalization Script

> *"Because selling a stock imposes a 1-day cooldown before the next purchase, the decision at day $i$ depends on discrete system states. I model this as a 3-state finite state machine:
> 
> 1. `held`: We own a stock. We either carried it over from yesterday (`held`), or we bought today using funds from the `reset` state (`reset - price`).
> 2. `sold`: We sold our stock today (`held + price`). We must enter cooldown.
> 3. `reset`: We hold no stock and are eligible to buy. We either rested from yesterday's `reset`, or we transitioned out of yesterday's `sold` cooldown state.
> 
> On day 0, `held = -prices[0]`, `sold = -inf`, and `reset = 0`.
> For each subsequent day, each state updates simultaneously using yesterday's values.
> 
> At termination, the maximum profit is $\max(\text{sold}, \text{reset})$. This achieves $O(N)$ time with $O(1)$ auxiliary memory."*

---

## 3. Benchmark Problem 2: LeetCode 714 — Stock with Transaction Fee

### 3.1 State Formulation
With no cooldown but a fixed transaction fee $F$ applied per round-trip:
1. `hold`: Maximum profit holding a stock.
2. `cash`: Maximum profit not holding stock.

```python
class SolutionStockWithFee:
    def maxProfit(self, prices: list[int], fee: int) -> int:
        cash = 0
        hold = -prices[0]
        
        for price in prices[1:]:
            cash = max(cash, hold + price - fee)
            hold = max(hold, cash - price)
            
        return cash
```
- **Time Complexity**: $O(N)$.
- **Space Complexity**: $O(1)$.

---

## 4. Benchmark Problem 3: LeetCode 188 — Best Time to Buy and Sell Stock IV ($K$ Transactions)

### 4.1 State Invariant with $K$ Transactions
When at most $K$ transactions are allowed, each transaction consists of a Buy-Sell pair:
- If $K \ge N / 2$: We can execute unlimited transactions whenever prices rise ($O(N)$ greedy pass).
- Otherwise: Maintain arrays `buy[k]` and `sell[k]` for $k \in [1 \dots K]$:
  - `buy[k] = max(buy[k], sell[k-1] - price)`
  - `sell[k] = max(sell[k], buy[k] + price)`

```python
class SolutionStockIV:
    def maxProfit(self, k: int, prices: list[int]) -> int:
        n = len(prices)
        if n <= 1 or k == 0:
            return 0
            
        # Optimization: unlimited transactions if k >= n // 2
        if k >= n // 2:
            return sum(max(0, prices[i] - prices[i - 1]) for i in range(1, n))
            
        buy = [float('-inf')] * (k + 1)
        sell = [0] * (k + 1)
        
        for price in prices:
            for t in range(1, k + 1):
                buy[t] = max(buy[t], sell[t - 1] - price)
                sell[t] = max(sell[t], buy[t] + price)
                
        return sell[k]
```

- **Time Complexity**: $O(N \times K)$.
- **Space Complexity**: $O(K)$ — Space optimized from $O(N \times K)$ to $O(K)$.
