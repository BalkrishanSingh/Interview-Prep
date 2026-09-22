# Module 01: DBMS — Window Functions & Ranking In-Depth

---

## 1. Window Functions: Core Mental Model

In traditional SQL, when you need an aggregate statistic (such as `AVG(salary)` or `COUNT(*)`), using `GROUP BY` **collapses** individual rows into summary buckets. You lose the individual row identity.

A **Window Function** performs calculations across a set of table rows that are related to the current row, but **maintains the individual identity of every single row**.

```
GROUP BY:
[Row 1: 100] \
[Row 2: 200]  -> [Group Summary: AVG = 200]  (Individual rows destroyed)
[Row 3: 300] /

WINDOW FUNCTION:
[Row 1: 100] -> [Row 1: 100 | AVG = 200]
[Row 2: 200] -> [Row 2: 200 | AVG = 200]     (Individual rows preserved!)
[Row 3: 300] -> [Row 3: 300 | AVG = 200]
```

### Complete Syntax Anatomy

```sql
FUNCTION_NAME([arguments]) OVER (
    [PARTITION BY partition_column_1, partition_column_2, ...]
    [ORDER BY sort_column_1 [ASC | DESC], ...]
    [ROWS | RANGE frame_specification]
)
```

1. **`PARTITION BY`**: Divides the rows into subsets (like mini-tables). If omitted, the entire result set is treated as one partition.
2. **`ORDER BY`**: Determines the logical evaluation order within each partition.
3. **Frame Specification (`ROWS | RANGE`)**: Defines the sliding window subset of rows relative to the current row (e.g., preceding 2 rows, unbounded preceding).

---

## 2. The Big 3 Ranking Functions: ROW_NUMBER vs RANK vs DENSE_RANK

Ranking functions assign an integer rank to each row within a partition based on an `ORDER BY` clause. Below is a side-by-side evaluation using a salary dataset containing duplicate values.

### Sample Data & Execution Comparison

Suppose we have salaries: `[10000, 9000, 9000, 8000, 7000]`.

```sql
SELECT 
    emp_name,
    salary,
    ROW_NUMBER() OVER (ORDER BY salary DESC) AS row_num,
    RANK()       OVER (ORDER BY salary DESC) AS rnk,
    DENSE_RANK() OVER (ORDER BY salary DESC) AS dense_rnk
FROM Employees;
```

### Output Table:
| emp_name | salary | `ROW_NUMBER()` | `RANK()` | `DENSE_RANK()` | Explanation |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Alice | 10000 | **1** | **1** | **1** | Highest distinct value |
| Bob | 9000 | **2** | **2** | **2** | Tie for 2nd |
| Charlie | 9000 | **3** | **2** | **2** | Tie for 2nd (`RANK` repeats 2, `DENSE_RANK` repeats 2) |
| David | 8000 | **4** | **4** | **3** | `RANK` skipped 3 (1,2,2 -> 4). `DENSE_RANK` has **NO gaps** (1,2,2 -> 3). |
| Emma | 7000 | **5** | **5** | **4** | `RANK` continues with 5; `DENSE_RANK` continues with 4. |

### Summary Comparison Table

| Function | Handling of Ties | Gap in Sequence After a Tie? | Typical Use Case |
| :--- | :--- | :--- | :--- |
| `ROW_NUMBER()` | Assigns arbitrary sequential unique integers (1, 2, 3...) | **No gaps** (strictly incrementing) | Deduplication (`WHERE row_num = 1`), Pagination. |
| `RANK()` | Tied items get identical rank | **YES (Skips numbers)**. Next rank = current rank + number of ties | Olympic medals (Gold=1, Silver=2, Silver=2, Bronze=4). |
| `DENSE_RANK()` | Tied items get identical rank | **NO (Zero gaps)**. Next rank = current rank + 1 | Nth Highest Salary, Top-K leaderboards without rank gaps. |

---

## 3. Positional Functions: LEAD and LAG

`LEAD` and `LAG` allow querying data from a subsequent or preceding row without performing an expensive self-join.

### Syntax
```sql
LAG(expression [, offset [, default_value]]) OVER (PARTITION BY ... ORDER BY ...)
LEAD(expression [, offset [, default_value]]) OVER (PARTITION BY ... ORDER BY ...)
```
- `offset`: Number of rows back (LAG) or forward (LEAD). Default is `1`.
- `default_value`: Value returned when offset falls outside partition boundaries. Default is `NULL`.

### Real-World Example 1: Month-over-Month (MoM) Revenue Growth
```sql
WITH MonthlySales AS (
    SELECT 
        DATE_FORMAT(order_date, '%Y-%m') AS sales_month,
        SUM(amount) AS current_month_revenue
    FROM Orders
    GROUP BY DATE_FORMAT(order_date, '%Y-%m')
)
SELECT 
    sales_month,
    current_month_revenue,
    LAG(current_month_revenue, 1, 0) OVER (ORDER BY sales_month) AS prev_month_revenue,
    ROUND(
        (current_month_revenue - LAG(current_month_revenue, 1) OVER (ORDER BY sales_month)) 
        / LAG(current_month_revenue, 1) OVER (ORDER BY sales_month) * 100, 2
    ) AS mom_growth_pct
FROM MonthlySales;
```

---

## 4. The Framing Clause: ROWS vs RANGE

When doing rolling calculations (e.g. running totals, moving averages), the frame determines the subset of rows evaluated.

```
BETWEEN <frame_start> AND <frame_end>
```
Common options:
- `UNBOUNDED PRECEDING`: First row of partition.
- `CURRENT ROW`: The row currently being processed.
- `N PRECEDING`: $N$ rows prior to current row.
- `N FOLLOWING`: $N$ rows after current row.
- `UNBOUNDED FOLLOWING`: Last row of partition.

### The Hidden Default Trap!
If you write `OVER (ORDER BY transaction_date)` without specifying a frame, the SQL standard defaults to:
```sql
RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
```
> [!WARNING]
> **Difference between `ROWS` and `RANGE`**:
> - `ROWS` treats every row physically and individually.
> - `RANGE` treats duplicate values in `ORDER BY` as a single peer group! If 3 transactions occur on the exact same date, `RANGE` calculates the sum including **all 3 at once**, causing unexpected jumps instead of a smooth running total!
> - **Best Practice**: For running totals and moving averages, always explicitly use `ROWS`:
>   ```sql
>   SUM(amount) OVER (ORDER BY trans_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)
>   ```

### 7-Day Moving Average Example
```sql
SELECT 
    trans_date,
    daily_revenue,
    AVG(daily_revenue) OVER (
        ORDER BY trans_date 
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS rolling_7day_avg
FROM DailyFinancials;
```

---

## 5. Practical Implementation Scenarios

### Problem 1: Find the Top 3 Highest Earners in EACH Department
```sql
WITH RankedEmployees AS (
    SELECT 
        emp_id,
        emp_name,
        dept_id,
        salary,
        DENSE_RANK() OVER (
            PARTITION BY dept_id 
            ORDER BY salary DESC
        ) AS salary_rank
    FROM Employees
)
SELECT emp_id, emp_name, dept_id, salary
FROM RankedEmployees
WHERE salary_rank <= 3;
```
*Rationale for `DENSE_RANK()` over `RANK()`*:
If two employees in Engineering tie for the #2 highest salary, both are included, and the next earner is ranked #3. `DENSE_RANK` guarantees that exactly 3 distinct highest salary tiers are selected per department.

### Problem 2: Identify Users with at Least 3 Consecutive Active Days
```sql
-- Table: Logins (user_id, login_date)
WITH DistinctLogins AS (
    SELECT DISTINCT user_id, login_date
    FROM Logins
),
GroupedSequences AS (
    SELECT 
        user_id,
        login_date,
        -- Subtracting row_number days from login_date yields a constant group date for consecutive streaks
        DATE_SUB(login_date, INTERVAL ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY login_date) DAY) AS streak_group
    FROM DistinctLogins
)
SELECT user_id, MIN(login_date) AS streak_start, COUNT(*) AS consecutive_days
FROM GroupedSequences
GROUP BY user_id, streak_group
HAVING COUNT(*) >= 3;
```

---

## 6. Mathematical & Semantic Distinction: DENSE_RANK() vs RANK()

Both functions assign rankings based on an `ORDER BY` clause and award the same rank to identical values.

The operational distinction lies in sequence handling following a tie:
- **`RANK()`**: Leaves gaps in the sequence. If two rows tie for rank 1, the next row receives rank 3 (rank 2 is skipped). Formula: $\text{Next Rank} = \text{Current Rank} + \text{Number of Ties}$.
- **`DENSE_RANK()`**: Never skips numbers. If two rows tie for rank 1, the next row receives rank 2. Formula: $\text{Next Rank} = \text{Current Rank} + 1$.

When querying for the "$N$-th distinct value" (such as the second highest salary), `DENSE_RANK()` is mandatory; if duplicate top values exist, `RANK()` skips rank 2 entirely, causing a filter `WHERE rnk = 2` to return zero rows.
