# Module 01: DBMS — ACID Properties & Concurrency Control

---

## 1. The ACID Pillars Explained

A **transaction** is a logical unit of database processing that includes one or more database access operations.

```
       ┌────────────────────────────────────────────────────────┐
       │                  A C I D   P I L L A R S               │
       └────────────────────────────────────────────────────────┘
          │                 │                 │               │
     [Atomicity]      [Consistency]      [Isolation]     [Durability]
   "All or Nothing"   "Integrity &     "Independence"   "Survives
   (Undo Logs / WAL)   Constraints"    (Locks & MVCC)     Crashes"
                                                         (Redo Logs)
```

### 1.1 Atomicity ("All or Nothing")
- **Definition**: Either all operations in a transaction complete successfully, or none of them are applied.
- **Real-World Example**: Transferring $500 from Account A to Account B. If the system crashes after deducting $500 from A but before adding $500 to B, the system rolls back Account A to its original state.
- **Implementation**: Managed via the **Undo Log** and **Write-Ahead Logging (WAL)**.

### 1.2 Consistency ("Preserving Invariants")
- **Definition**: A transaction brings the database from one valid state to another valid state, satisfying all explicit constraints (e.g., `FOREIGN KEY`, `CHECK`, `UNIQUE`) and implicit business invariants.
- **Example**: If a constraint requires that account balances cannot be negative, any transaction attempting to withdraw more than the balance is rejected and aborted.

### 1.3 Isolation ("Concurrent Independence")
- **Definition**: The execution of multiple concurrent transactions results in a system state that would be obtained if transactions were executed serially (one after the other).
- **Implementation**: Managed via **Locks** (Shared, Exclusive) and **MVCC (Multi-Version Concurrency Control)**.

### 1.4 Durability ("Permanence")
- **Definition**: Once a transaction commits, its effects persist permanently, even in the event of an immediate power outage or hardware crash.
- **Implementation**: Written to non-volatile disk via the **Redo Log** before reporting success to the client (WAL protocol).

---

## 2. Concurrency Anomalies (What Goes Wrong?)

When transactions run concurrently without proper isolation, four distinct phenomena can occur:

### 2.1 Dirty Read
Transaction A modifies a row but has **not yet committed**. Transaction B reads that uncommitted modified row. If Transaction A subsequently **rolls back**, Transaction B processed data that never officially existed!

### 2.2 Non-Repeatable Read (Fuzzy Read)
Transaction A reads a row. Transaction B **modifies or deletes** that row and **commits**. Transaction A reads the exact same row again and observes different column values.

### 2.3 Phantom Read
Transaction A queries a **range of rows** (e.g., `WHERE age > 30` returns 5 rows). Transaction B **inserts** a new row with `age = 35` and **commits**. Transaction A repeats the query and discovers a 6th row that wasn't there before (a "phantom").

### 2.4 Lost Update
Transaction A and Transaction B read balance = $100 simultaneously. Transaction A adds $50 (writes $150). Transaction B subtracts $20 (writes $80), completely overwriting Transaction A's deposit without knowing it.

---

## 3. The 4 ANSI SQL Isolation Levels

To balance data consistency with throughput, databases offer 4 isolation levels:

| Isolation Level | Dirty Read | Non-Repeatable Read | Phantom Read | Mechanism / Lock Overhead |
| :--- | :---: | :---: | :---: | :--- |
| **Read Uncommitted** | ❌ Allowed | ❌ Allowed | ❌ Allowed | Zero read locks; reads dirty data. |
| **Read Committed** | ✅ Prevented | ❌ Allowed | ❌ Allowed | Reads only committed data. (Default in PostgreSQL, Oracle, SQL Server). |
| **Repeatable Read** | ✅ Prevented | ✅ Prevented | ⚠️ Partially Prevented | Guarantees repeated reads see identical data. (Default in MySQL InnoDB). |
| **Serializable** | ✅ Prevented | ✅ Prevented | ✅ Prevented | Strict order. Full range locks or serialization failure detection. |

---

## 4. MVCC (Multi-Version Concurrency Control)

Modern database engines (MySQL InnoDB, PostgreSQL) achieve high throughput using **MVCC**.
> **Golden Rule of MVCC**: *"Readers never block writers, and writers never block readers."*

### How MVCC Works Under the Hood:
1. Every transaction is assigned a monotonically increasing **Transaction ID (XID / TxID)**.
2. Every row in the table contains hidden metadata columns:
   - `DB_TRX_ID`: The ID of the transaction that inserted or last modified this row.
   - `DB_ROLL_PTR`: A pointer pointing to the previous version of the row stored in the **Undo Log**.
3. When a transaction performs a `SELECT`:
   - It creates a **Read View (Snapshot)**.
   - It only sees row versions created by transactions that committed **before** its snapshot was created.
   - If a row was modified after the snapshot, the engine follows the `DB_ROLL_PTR` into the Undo Log to reconstruct the older, consistent version.

---

## 5. Lock-Based Concurrency & Two-Phase Locking (2PL)

When MVCC alone cannot prevent write-write conflicts, locks are used.

### Shared Lock (S-Lock) vs. Exclusive Lock (X-Lock):
- **Shared Lock (`LOCK IN SHARE MODE` / `FOR SHARE`)**: Read lock. Multiple transactions can hold S-locks on the same row concurrently.
- **Exclusive Lock (`FOR UPDATE`)**: Write lock. Only one transaction can hold an X-lock. All other S-locks and X-locks must wait.

### Two-Phase Locking (2PL) Protocol:
Guarantees serializability through two strict phases:
1. **Growing Phase**: The transaction may acquire locks, but cannot release any.
2. **Shrinking Phase**: The transaction may release locks, but cannot acquire any new locks.

```
       Number of Locks
             ▲
             │       /═════════════\
             │      /               \
             │     /                 \
             │    /                   \
             │   /                     \
             └──/───────────────────────\─────► Time
                Growing Phase        Shrinking Phase
             (Acquiring Locks)      (Releasing Locks)
```

---

## 6. Deadlocks & Resolution

A **Deadlock** occurs when two or more transactions are permanently blocked because each holds a lock that the other needs.

```
Tx 1: Holds Lock on Row A  ──(wants Lock on Row B)──► Waiting for Tx 2
Tx 2: Holds Lock on Row B  ──(wants Lock on Row A)──► Waiting for Tx 1
```

### Detection and Recovery:
1. **Wait-For Graph**: The database engine maintains a directed graph where nodes represent transactions and edges represent wait conditions. A cycle in the graph indicates a deadlock.
2. **Deadlock Victim Selection**: The engine terminates and rolls back the transaction with the smallest undo log footprint (lowest cost to abort), allowing the other transaction to proceed.
3. **Deadlock Prevention Strategy in Code**:
   - Always acquire locks in the **exact same global order** across all application services (e.g., always sort row IDs before locking: `SELECT * FROM accounts WHERE id IN (1, 2) ORDER BY id FOR UPDATE`).
