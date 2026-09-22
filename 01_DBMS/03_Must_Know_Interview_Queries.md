# Module 01: DBMS — Advanced SQL Queries & Edge-Case Playbook

---

## 1. Finding the Second Highest (and N-th Highest) Salary

Finding the $N$-th highest or lowest value requires robust handling of duplicate values and boundary edge cases (such as datasets containing fewer than $N$ records).

### Schema: `Employee`
| id | salary |
| :--- | :--- |
| 1 | 100 |
| 2 | 200 |
| 3 | 300 |
| 4 | 300 |

---

### Approach 1: Window Function (`DENSE_RANK`) — Standard & Scalable
```sql
WITH RankedSalaries AS (
    SELECT 
        salary,
        DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
    FROM Employee
)
SELECT 
    (SELECT salary FROM RankedSalaries WHERE rnk = 2 LIMIT 1) AS SecondHighestSalary;
```
*Why the outer `SELECT (...) AS SecondHighestSalary`?*
If the table contains only 1 row (no 2nd highest salary exists), a bare `WHERE rnk = 2` returns an **empty result set** (0 rows). Wrapping the query in a scalar subquery ensures SQL evaluates to `NULL` when no matching record exists.

---

### Approach 2: Using `MAX()` with Subquery — Engine Agnostic
```sql
SELECT MAX(salary) AS SecondHighestSalary
FROM Employee
WHERE salary < (
    SELECT MAX(salary) FROM Employee
);
```
- **Mechanism**: The subquery finds the global maximum ($300$). The outer query evaluates the maximum salary strictly smaller than $300$ ($200$).
- **Handling Edge Cases**: If there is only 1 employee, `salary < (MAX)` yields no rows, and `MAX()` on an empty set evaluates to `NULL`.
- **Limitation**: Finding the $N$-th highest salary requires nesting $N-1$ subqueries, which scales poorly.

---

### Approach 3: `LIMIT` and `OFFSET` with `DISTINCT`
```sql
SELECT (
    SELECT DISTINCT salary
    FROM Employee
    ORDER BY salary DESC
    LIMIT 1 OFFSET 1
) AS SecondHighestSalary;
```
- For the $N$-th highest salary, set `OFFSET N - 1`.
- **Requirement for `DISTINCT`**: If duplicate values exist at the top rank, omitting `DISTINCT` causes `OFFSET 1` to return duplicate values of rank 1 instead of the genuine second highest tier.

---

## 2. Duplicate Records: Detection and Safe Deletion

Managing duplicate records requires identifying duplicate keys and safely pruning them while preserving canonical entries.

### Schema: `Person`
| id | email |
| :--- | :--- |
| 1 | john@example.com |
| 2 | bob@example.com |
| 3 | john@example.com |

---

### 2.1 Finding Duplicates
```sql
SELECT email, COUNT(*) AS occurrences
FROM Person
GROUP BY email
HAVING COUNT(*) > 1;
```

---

### 2.2 Deleting Duplicates (Keeping the row with smallest `id`)

#### Method A: Using Common Table Expression (CTE) & `ROW_NUMBER()` (PostgreSQL, SQL Server, MySQL 8+)
```sql
WITH RankedRows AS (
    SELECT 
        id,
        email,
        ROW_NUMBER() OVER (PARTITION BY email ORDER BY id ASC) AS row_num
    FROM Person
)
DELETE FROM Person
WHERE id IN (
    SELECT id FROM RankedRows WHERE row_num > 1
);
```

#### Method B: Self-Join (Compatible with MySQL 5.7+ and standard ANSI SQL)
```sql
DELETE p1
FROM Person p1
INNER JOIN Person p2
    ON p1.email = p2.email
    AND p1.id > p2.id;
```
- **Mechanism**: Pairs every record `p1` with `p2` having the same email. If `p1.id > p2.id`, then `p1` is a duplicate of an earlier record and gets deleted.

---

## 3. Employees Earning More Than Their Managers

### Schema: `Employee`
| id | name | salary | managerId |
| :--- | :--- | :--- | :--- |
| 1 | Joe | 70000 | 3 |
| 2 | Henry | 80000 | 4 |
| 3 | Sam | 60000 | NULL |
| 4 | Max | 90000 | NULL |

```sql
SELECT e.name AS Employee
FROM Employee e
INNER JOIN Employee m
    ON e.managerId = m.id
WHERE e.salary > m.salary;
```
- **Key Insight**: Use an `INNER JOIN` rather than `LEFT JOIN`. Employees without managers (`managerId IS NULL`) can never earn more than their manager, so they are filtered out immediately.

---

## 4. Consecutive Numbers / Streaks (Gaps and Islands)

### Problem: Find all numbers that appear at least three times consecutively.
### Schema: `Logs`
| id | num |
| :--- | :--- |
| 1 | 1 |
| 2 | 1 |
| 3 | 1 |
| 4 | 2 |
| 5 | 1 |
| 6 | 2 |
| 7 | 2 |

---

### Approach 1: Using `LEAD` and `LAG`
```sql
WITH NeighborCheck AS (
    SELECT 
        num,
        LAG(num, 1)  OVER (ORDER BY id) AS prev_num,
        LEAD(num, 1) OVER (ORDER BY id) AS next_num
    FROM Logs
)
SELECT DISTINCT num AS ConsecutiveNums
FROM NeighborCheck
WHERE num = prev_num AND num = next_num;
```

---

### Approach 2: Self-Join on `id` (Assuming IDs are strictly consecutive)
```sql
SELECT DISTINCT l1.num AS ConsecutiveNums
FROM Logs l1
JOIN Logs l2 ON l1.id = l2.id - 1
JOIN Logs l3 ON l1.id = l3.id - 2
WHERE l1.num = l2.num AND l2.num = l3.num;
```

---

## 5. Relational Division: "Entities Associated with All Items"

Relational division represents queries that search for entities associated with all members of another set (the universal quantifier $\forall$ in relational calculus).

### Schema:
- `Customer` (customer_id, product_key)
- `Product` (product_key)

```sql
SELECT customer_id
FROM Customer
GROUP BY customer_id
HAVING COUNT(DISTINCT product_key) = (
    SELECT COUNT(*) FROM Product
);
```
> [!NOTE]
> **Cardinality Verification**:
> A customer could purchase the same product multiple times. Without `DISTINCT`, a customer purchasing product `1` multiple times would falsely satisfy the total count condition.

---

## 6. Cumulative Running Total

Calculate running total of transaction amounts per customer over time.

```sql
SELECT 
    customer_id,
    transaction_date,
    amount,
    SUM(amount) OVER (
        PARTITION BY customer_id 
        ORDER BY transaction_date 
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total
FROM Transactions;
```

---

## 7. SQL Edge Cases & Core Nuances Reference

| Scenario | Behavior & Standard Solution |
| :--- | :--- |
| **NULL Handling in ORDER BY** | Use `ORDER BY col ASC NULLS LAST` (ANSI/PostgreSQL/Oracle) or `ORDER BY ISNULL(col), col ASC` (MySQL). |
| **UNION vs. UNION ALL** | `UNION` eliminates duplicate rows by sorting in memory ($O(N \log N)$). `UNION ALL` preserves duplicates and concatenates sets directly without sorting ($O(N)$). |
| **COUNT(*), COUNT(1), and COUNT(column)** | `COUNT(*)` and `COUNT(1)` count all rows in the partition, including rows with NULLs. `COUNT(column)` counts only rows where `column IS NOT NULL`. |
| **Aggregate Nesting** | Direct nesting (e.g. `MAX(AVG(salary))`) is invalid in standard SQL. Layered aggregations require subqueries or Common Table Expressions (CTEs). |
