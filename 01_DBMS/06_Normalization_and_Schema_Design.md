# Module 01: DBMS — Normalization & Schema Design

---

## 1. Why Normalization? The Three Anomalies

**Normalization** is the systematic process of structuring relational database tables to reduce data redundancy and eliminate data anomalies.

Consider an unnormalized table: `StudentCourse(student_id, student_name, course_id, course_fee, instructor_name)`

```
┌────────────┬──────────────┬───────────┬────────────┬─────────────────┐
│ student_id │ student_name │ course_id │ course_fee │ instructor_name │
├────────────┼──────────────┼───────────┼────────────┼─────────────────┤
│ 101        │ Alice        │ CS101     │ 500        │ Prof. Smith     │
│ 102        │ Bob          │ CS101     │ 500        │ Prof. Smith     │
│ 103        │ Charlie      │ CS102     │ 600        │ Prof. Jones     │
└────────────┴──────────────┴───────────┴────────────┴─────────────────┘
```

### The Three Deadly Anomalies:
1. **Insertion Anomaly**: If the university introduces a new course `CS103` before any student enrolls, we cannot insert it without setting `student_id = NULL` (which fails if `student_id` is part of the primary key).
2. **Update Anomaly**: If the fee for `CS101` changes from 500 to 550, we must update multiple rows. If one row is missed, data becomes corrupt and contradictory.
3. **Deletion Anomaly**: If Charlie drops out and we delete row 3, we inadvertently delete all record of course `CS102` and Prof. Jones!

---

## 2. Fundamental Keys & Functional Dependencies

- **Functional Dependency ($X \to Y$)**: Attribute $Y$ is functionally dependent on $X$ if each value of $X$ determines exactly one value of $Y$.
- **Super Key**: Any set of attributes that uniquely identifies a row.
- **Candidate Key**: A minimal super key (no attribute can be removed without losing uniqueness).
- **Primary Key**: The candidate key chosen by the database designer.
- **Prime Attribute**: An attribute that is part of ANY candidate key.
- **Non-Prime Attribute**: An attribute that is NOT part of any candidate key.

---

## 3. Step-by-Step Normal Forms: 1NF to BCNF

```
┌────────────────────────────────────────────────────────┐
│ 1NF: Atomic values only (no repeating groups / arrays) │
└───────────────────────────┬────────────────────────────┘
                            │
┌───────────────────────────▼────────────────────────────┐
│ 2NF: 1NF + No Partial Dependency (Key -> Non-Key)       │
└───────────────────────────┬────────────────────────────┘
                            │
┌───────────────────────────▼────────────────────────────┐
│ 3NF: 2NF + No Transitive Dependency (Non-Key -> Non-Key)│
└───────────────────────────┬────────────────────────────┘
                            │
┌───────────────────────────▼────────────────────────────┐
│ BCNF: For every X -> Y, X MUST be a Super Key          │
└────────────────────────────────────────────────────────┘
```

---

### 3.1 First Normal Form (1NF)
**Rule**:
1. Each column must contain atomic (indivisible) values.
2. No repeating groups or arrays.
3. Each row must be uniquely identifiable.

#### Non-1NF Table:
| emp_id | emp_name | phone_numbers |
| :--- | :--- | :--- |
| 1 | Alice | `9876543210, 9123456780` |

#### Refactored 1NF Table:
Split into atomic rows or a separate phone table:
| emp_id | emp_name | phone_number |
| :--- | :--- | :--- |
| 1 | Alice | 9876543210 |
| 1 | Alice | 9123456780 |

---

### 3.2 Second Normal Form (2NF)
**Rule**: Must be in 1NF **AND** contain **no partial functional dependencies**.
- A partial dependency exists when a non-prime attribute depends on only a *portion* of a composite candidate key.
- *Note*: If a table's candidate key consists of only a single column, it is **automatically in 2NF**.

#### Non-2NF Table:
Candidate Key: `(student_id, course_id)`
| student_id | course_id | course_name | grade |
| :--- | :--- | :--- | :--- |
| 101 | CS101 | Intro CS | A |
| 101 | CS102 | Algorithms | B |

- `grade` depends on `(student_id, course_id)` — **Full dependency**.
- `course_name` depends *only* on `course_id` — **Partial dependency**!

#### Refactoring to 2NF:
Decompose into two tables:
1. `CourseRegistration(student_id, course_id, grade)`
2. `Course(course_id, course_name)`

---

### 3.3 Third Normal Form (3NF)
**Rule**: Must be in 2NF **AND** contain **no transitive dependencies**.
- A transitive dependency exists when a non-prime attribute determines another non-prime attribute ($A \to B$ and $B \to C$, where $A$ is the primary key).

#### Non-3NF Table:
Primary Key: `emp_id`
| emp_id | emp_name | zip_code | city | state |
| :--- | :--- | :--- | :--- | :--- |
| 1 | Alice | 560001 | Bangalore | Karnataka |
| 2 | Bob | 560001 | Bangalore | Karnataka |

- `emp_id -> zip_code`
- `zip_code -> city, state`
- `emp_id -> city, state` is a **transitive dependency**! If the postal boundary changes, multiple employee rows must be updated.

#### Refactoring to 3NF:
1. `Employee(emp_id, emp_name, zip_code)`
2. `ZipCodeLocation(zip_code, city, state)`

---

### 3.4 Boyce-Codd Normal Form (BCNF)
**Rule**: A stricter version of 3NF. For every functional dependency $X \to Y$, **$X$ must be a Super Key**.

#### When does 3NF fail BCNF?
When a table has multiple overlapping composite candidate keys.

#### Classic Example:
Consider student counseling: `(student_id, subject, advisor)`
- A student can have multiple subjects.
- For each subject, a student has one advisor: `(student_id, subject) -> advisor`.
- Each advisor specializes in **only one subject**: `advisor -> subject`.

Here, candidate keys are `(student_id, subject)` and `(student_id, advisor)`.
- It is in 3NF because `subject` is a prime attribute.
- However, in `advisor -> subject`, `advisor` is **NOT a super key**!
- To reach BCNF, decompose into:
  1. `AdvisorSubject(advisor, subject)`
  2. `StudentAdvisor(student_id, advisor)`

---

## 4. When & Why to Denormalize?

In large-scale production architectures (such as high-throughput banking and e-commerce platforms), strictly normalized schemas (3NF) can introduce high query latency due to multi-table joins.

### Denormalization Trade-offs:
| Dimension | Normalized (3NF) | Denormalized |
| :--- | :--- | :--- |
| **System Profile** | **OLTP** (Online Transaction Processing) | **OLAP** (Data Warehouses / Analytics) |
| **Write Performance** | **Fast** (single row update, zero duplicate data) | **Slower** (multiple copies must be updated) |
| **Read Performance** | Slower (requires 5-10 table joins) | **Blazing fast** (pre-joined summary tables) |
| **Storage** | Minimal | Higher (redundant columns stored) |
| **Integrity Risks** | Enforced by foreign keys | Requires application-level reconciliation |

---

## 5. Relational (SQL) vs. NoSQL Architectural Comparison

| Dimension | RDBMS (PostgreSQL, MySQL, Oracle) | NoSQL (MongoDB, DynamoDB, Cassandra) |
| :--- | :--- | :--- |
| **Data Model** | Relational tables with strict fixed schema | Document (JSON), Key-Value, Columnar, Graph |
| **Scaling** | Vertical scaling (Scale-up: more CPU/RAM) | Horizontal scaling (Scale-out: sharding across clusters) |
| **Transactions** | Strong ACID compliance | BASE (Basically Available, Soft state, Eventual consistency) |
| **Join Support** | Rich, declarative, optimizer-driven joins | Joins typically done in application logic or denormalized |
| **Best Used For** | Financial transactions, inventory, ERP | High-volume streaming, clickstreams, social feeds, dynamic schema |
