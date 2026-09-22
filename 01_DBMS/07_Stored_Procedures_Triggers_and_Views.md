# Module 01: DBMS — Stored Procedures, Triggers, and Views

---

## 1. Stored Procedures vs. User-Defined Functions (UDFs)

Stored procedures and user-defined functions represent pre-compiled procedural logic stored directly within the database catalog.

```
┌────────────────────────────────────────────────────────────────────────┐
│                        STORED ROUTINES                                 │
└──────────────────────────────────┬─────────────────────────────────────┘
                                   │
         ┌─────────────────────────┴─────────────────────────┐
         ▼                                                   ▼
┌─────────────────────────────────┐         ┌─────────────────────────────────┐
│        STORED PROCEDURE         │         │      USER-DEFINED FUNCTION      │
├─────────────────────────────────┤         ├─────────────────────────────────┤
│ • Executed via CALL / EXEC      │         │ • Executed inside SELECT / WHERE│
│ • Can modify database state     │         │ • Pure computation / Read-only  │
│   (INSERT, UPDATE, DELETE)      │         │ • Must return exactly one value │
│ • Can return 0, 1, or N results │         │   (scalar or table)             │
│ • Supports full transactions    │         │ • Cannot manage transactions    │
│   (COMMIT / ROLLBACK)           │         │   (NO COMMIT / ROLLBACK)        │
└─────────────────────────────────┘         └─────────────────────────────────┘
```

### 1.1 Detailed Architectural Comparison
| Feature | Stored Procedure | User-Defined Function (UDF) |
| :--- | :--- | :--- |
| **Invocation** | Standalone statement (`CALL proc_name(args)`) | Embedded within SQL expressions (`SELECT fn(col) FROM t`) |
| **Return Value** | Optional. Can return 0, 1, or multiple result sets | **Mandatory**. Must return exactly one scalar value or a table |
| **DML Operations** | Allowed (`INSERT`, `UPDATE`, `DELETE`) | **Forbidden** in most engines (cannot mutate table state) |
| **Transaction Control** | Fully supported (`START TRANSACTION`, `COMMIT`, `ROLLBACK`) | Not supported (cannot initiate or terminate transactions) |
| **Performance** | Pre-compiled execution plan cached in memory | Can cause severe performance degradation if row-by-row (RBAR) in `WHERE` |

### 1.2 Example: Stored Procedure with Transaction & Error Handling (MySQL syntax)
```sql
DELIMITER $$

CREATE PROCEDURE TransferFunds(
    IN sender_account_id INT,
    IN receiver_account_id INT,
    IN transfer_amount DECIMAL(10, 2)
)
BEGIN
    DECLARE sender_balance DECIMAL(10, 2);
    
    -- Error handler: Rollback on any SQL exception
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
        RESIGNAL;
    END;

    START TRANSACTION;

    -- Lock the sender row for update to prevent race conditions
    SELECT balance INTO sender_balance 
    FROM Accounts 
    WHERE account_id = sender_account_id 
    FOR UPDATE;

    IF sender_balance < transfer_amount THEN
        SIGNAL SQLSTATE '45000' 
        SET MESSAGE_TEXT = 'Insufficient balance for transfer';
    END IF;

    -- Perform transfer
    UPDATE Accounts 
    SET balance = balance - transfer_amount 
    WHERE account_id = sender_account_id;

    UPDATE Accounts 
    SET balance = balance + transfer_amount 
    WHERE account_id = receiver_account_id;

    COMMIT;
END$$

DELIMITER ;
```

---

## 2. Triggers: Mechanisms and Caveats

A **Trigger** is a specialized stored program that automatically executes (fires) in response to a specific event on a particular table or view.

### 2.1 Trigger Dimensions
1. **Timing**: `BEFORE` vs. `AFTER`
   - `BEFORE`: Ideal for data validation, sanitization, or calculating default values before writing to disk.
   - `AFTER`: Ideal for audit logging, synchronizing cache tables, or updating aggregated counters.
2. **Event**: `INSERT`, `UPDATE`, or `DELETE`.
3. **Granularity**: `FOR EACH ROW` (Row-level) vs. `FOR EACH STATEMENT` (Statement-level).

### 2.2 Pseudo-Records: `OLD` vs. `NEW`
| Event | `OLD` Values | `NEW` Values |
| :--- | :--- | :--- |
| `INSERT` | All fields are `NULL` | Contains the incoming values to be inserted |
| `UPDATE` | Contains values before update | Contains values after update |
| `DELETE` | Contains values before deletion | All fields are `NULL` |

### 2.3 Example: Audit Logging Trigger
```sql
CREATE TRIGGER audit_employee_salary_update
AFTER UPDATE ON Employees
FOR EACH ROW
BEGIN
    IF OLD.salary <> NEW.salary THEN
        INSERT INTO SalaryAuditLog (
            employee_id, 
            old_salary, 
            new_salary, 
            changed_by, 
            changed_at
        ) VALUES (
            OLD.emp_id, 
            OLD.salary, 
            NEW.salary, 
            USER(), 
            NOW()
        );
    END IF;
END;
```

> [!WARNING]
> **Enterprise Caveat on Triggers**:
> While triggers enforce integrity at the database layer, excessive triggers lead to "hidden side effects" that make debugging application failures notoriously difficult. They also introduce overhead on bulk `INSERT`/`UPDATE` statements.

---

## 3. Standard Views vs. Materialized Views

```
STANDARD (VIRTUAL) VIEW:
[ Query Request ] ──► [ View Definition (Saved SQL Query) ] ──► [ Executed on Underlying Tables on the Fly ]
(Stores NO data on disk; purely a logical abstraction)

MATERIALIZED VIEW:
[ Query Request ] ──► [ Physical Table on Disk containing Pre-computed Results ]
(Fast read; requires periodic REFRESH to sync with base tables)
```

### 3.1 Comparison Table
| Dimension | Standard View | Materialized View |
| :--- | :--- | :--- |
| **Data Storage** | Zero disk space (only stores query metadata) | Stored physically as a table on disk |
| **Query Performance** | Same as underlying query (re-executed every time) | **Extremely fast** (pre-calculated data read directly) |
| **Data Freshness** | 100% real-time (reflects current table state) | Stale until refreshed (`REFRESH MATERIALIZED VIEW`) |
| **Index Support** | Cannot create indexes on standard views | **Can be indexed** directly on materialized columns |
| **Best Use Case** | Security masking (hiding sensitive columns), simplifying complex joins | Complex analytical aggregation, dashboards, OLAP reporting |

### 3.2 Materialized View Refresh Strategies
1. **Complete Refresh**: Drops all rows in the view and re-runs the entire base query from scratch.
2. **Fast / Incremental Refresh**: Uses materialized view logs on base tables to apply only delta changes (`INSERT`, `UPDATE`, `DELETE`) since the last refresh.
3. **On-Demand vs. On-Commit**: Refresh triggered manually/by cron vs. triggered automatically when base table transaction commits.
