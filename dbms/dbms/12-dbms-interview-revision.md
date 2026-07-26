# DBMS Interview Revision Guide

This guide is the **final revision layer** for the DBMS section.

It is not intended to replace the detailed topic guides. Use it after completing them to rapidly revise the concepts, comparisons, SQL patterns, and interview questions most likely to matter in technical interviews.

---

# 1. DBMS in One Definition

A **Database Management System (DBMS)** is software that allows applications and users to store, retrieve, update, organize, secure, and manage data while providing mechanisms for:

```text
Persistence

Querying

Integrity

Concurrency

Transactions

Recovery

Security

Data abstraction
```

A relational DBMS organizes data primarily into relations represented as tables.

Examples include:

```text
PostgreSQL

MySQL

Oracle Database

Microsoft SQL Server

SQLite
```

---

# 2. DBMS vs RDBMS

## DBMS

General category of software used to manage databases.

## RDBMS

A DBMS based on the relational model.

It represents data using relations and supports concepts such as:

```text
Rows

Columns

Keys

Constraints

Relationships

Relational operations
```

Therefore:

```text
RDBMS ⊂ DBMS
```

Every RDBMS is a DBMS, but not every DBMS is relational.

---

# 3. Core Relational Terms

Remember:

```text
Relation
→ Table

Tuple
→ Row

Attribute
→ Column

Domain
→ Allowed set/type of values

Degree
→ Number of attributes

Cardinality
→ Number of tuples
```

Example:

```text
Candidate

candidate_id | name | email
-------------|------|-----------------
101          | Alex | alex@example.com
102          | Mia  | mia@example.com
```

Here:

```text
Degree = 3

Cardinality = 2
```

---

# 4. Schema vs Instance

## Schema

Defines database structure.

Example:

```text
Candidate(
    candidate_id,
    name,
    email
)
```

## Instance

The actual data stored at a particular point in time.

```text
101 | Alex | alex@example.com
102 | Mia  | mia@example.com
```

Think:

```text
Schema
→ Structure

Instance
→ Current data
```

---

# 5. Three-Schema Architecture

Conceptually:

```text
External Level
      ↓
Conceptual Level
      ↓
Internal Level
```

## External Level

User/application-specific views.

## Conceptual Level

Overall logical database structure.

## Internal Level

Physical storage representation.

The architecture helps explain:

```text
Data abstraction

Data independence
```

---

# 6. Data Independence

## Physical Data Independence

Physical storage changes without requiring changes to the logical schema.

Examples:

```text
Index changes

Storage layout changes

File organization changes
```

## Logical Data Independence

Logical schema changes without requiring every external view/application to change.

Generally:

```text
Logical data independence
```

is harder to achieve than physical data independence.

---

# 7. Keys Quick Revision

## Super Key

Any attribute set that uniquely identifies a row.

## Candidate Key

A minimal super key.

## Primary Key

The candidate key selected as the primary row identifier.

## Alternate Key

Candidate keys not selected as primary.

## Composite Key

A key containing multiple attributes.

## Foreign Key

Attribute or attributes referencing a candidate/primary key in another or the same relation.

---

# 8. Super Key vs Candidate Key

Suppose:

```text
Candidate(
    candidate_id,
    email,
    name
)
```

and both:

```text
candidate_id

email
```

are unique.

Possible super keys:

```text
{candidate_id}

{email}

{candidate_id, name}

{email, name}
```

Candidate keys:

```text
{candidate_id}

{email}
```

because they are minimal.

---

# 9. Primary Key vs UNIQUE

Primary key:

```text
Primary relational identifier
```

UNIQUE:

```text
Enforces additional uniqueness
```

Example:

```sql
CREATE TABLE candidates (
    candidate_id BIGINT PRIMARY KEY,
    email TEXT NOT NULL UNIQUE
);
```

Here:

```text
candidate_id
```

is the primary key.

```text
email
```

is another business uniqueness constraint.

---

# 10. Foreign Key

Example:

```text
Candidate

candidate_id PK
```

```text
Interview

interview_id PK
candidate_id FK
```

Relationship:

```text
Candidate 1 ─── N Interview
```

Foreign keys help enforce:

```text
Referential integrity
```

---

# 11. Integrity Constraints

Important constraints:

```text
PRIMARY KEY

FOREIGN KEY

UNIQUE

NOT NULL

CHECK
```

Example:

```sql
CREATE TABLE interviews (
    interview_id BIGINT PRIMARY KEY,
    candidate_id BIGINT NOT NULL,
    score INTEGER CHECK (score BETWEEN 0 AND 100),

    FOREIGN KEY (candidate_id)
        REFERENCES candidates(candidate_id)
);
```

Constraints protect database invariants independently of application code.

---

# 12. SQL Command Categories

Common interview classification:

```text
DDL

DML

DQL

DCL

TCL
```

## DDL

```sql
CREATE
ALTER
DROP
TRUNCATE
```

## DML

```sql
INSERT
UPDATE
DELETE
```

## DQL

```sql
SELECT
```

## DCL

```sql
GRANT
REVOKE
```

## TCL

```sql
COMMIT
ROLLBACK
SAVEPOINT
```

Classification can vary slightly between references and DBMS documentation.

---

# 13. DELETE vs TRUNCATE vs DROP

## DELETE

Removes selected rows.

```sql
DELETE FROM interviews
WHERE status = 'cancelled';
```

Can use:

```text
WHERE
```

## TRUNCATE

Removes all rows from a table using DBMS-specific bulk semantics.

```sql
TRUNCATE TABLE interviews;
```

## DROP

Removes the database object itself.

```sql
DROP TABLE interviews;
```

Do not memorize universal claims about rollback or logging behavior because these differ between DBMSs.

---

# 14. WHERE vs HAVING

## WHERE

Filters rows before grouping.

```sql
SELECT *
FROM interviews
WHERE score >= 80;
```

## HAVING

Filters groups after aggregation.

```sql
SELECT candidate_id, AVG(score)
FROM interviews
GROUP BY candidate_id
HAVING AVG(score) >= 80;
```

Remember:

```text
WHERE
→ Rows

HAVING
→ Groups
```

---

# 15. GROUP BY

Example:

```sql
SELECT candidate_id, COUNT(*) AS total
FROM interviews
GROUP BY candidate_id;
```

Conceptually:

```text
Rows
 ↓
Group by candidate
 ↓
Aggregate each group
```

Common aggregates:

```sql
COUNT()
SUM()
AVG()
MIN()
MAX()
```

---

# 16. SQL Logical Processing Order

A useful conceptual order:

```text
FROM / JOIN

WHERE

GROUP BY

HAVING

SELECT

DISTINCT

ORDER BY

LIMIT / OFFSET
```

This explains why a SELECT alias often cannot be referenced in WHERE in standard query semantics.

Actual internal execution is optimizer-dependent.

---

# 17. INNER JOIN

Returns matching rows from both sides.

```sql
SELECT c.name, i.score
FROM candidates AS c
INNER JOIN interviews AS i
    ON i.candidate_id = c.candidate_id;
```

Conceptually:

```text
Candidate ∩ Interview matches
```

---

# 18. LEFT JOIN

Returns:

```text
All rows from left table

+

matching rows from right table
```

Example:

```sql
SELECT c.name, i.interview_id
FROM candidates AS c
LEFT JOIN interviews AS i
    ON i.candidate_id = c.candidate_id;
```

Candidates without interviews still appear.

Right-side columns become:

```text
NULL
```

for unmatched rows.

---

# 19. RIGHT JOIN

Returns:

```text
All rows from right table

+

matching rows from left table
```

Often the same logic can be expressed by swapping table order and using LEFT JOIN.

Support varies by DBMS.

---

# 20. FULL OUTER JOIN

Returns:

```text
Matched rows

+

unmatched left rows

+

unmatched right rows
```

Not every DBMS supports FULL OUTER JOIN directly.

---

# 21. CROSS JOIN

Returns the Cartesian product.

If:

```text
A = 100 rows

B = 20 rows
```

then:

```text
A CROSS JOIN B
```

can produce:

```text
2,000 rows
```

when all combinations are retained.

---

# 22. SELF JOIN

A table can be joined to itself.

Example:

```text
Employee

employee_id
manager_id
```

Query:

```sql
SELECT
    e.name AS employee,
    m.name AS manager
FROM employees AS e
LEFT JOIN employees AS m
    ON e.manager_id = m.employee_id;
```

---

# 23. UNION vs UNION ALL

## UNION

Combines results and removes duplicates.

## UNION ALL

Combines results and retains duplicates.

Because UNION performs deduplication, it can require additional processing.

Use UNION ALL when duplicate removal is unnecessary.

---

# 24. Subquery

A query inside another query.

Example:

```sql
SELECT *
FROM candidates
WHERE candidate_id IN (
    SELECT candidate_id
    FROM interviews
    WHERE score >= 90
);
```

Subqueries may be:

```text
Scalar

Multi-row

Correlated

Uncorrelated
```

---

# 25. Correlated Subquery

A correlated subquery references the outer query.

```sql
SELECT c.name
FROM candidates AS c
WHERE EXISTS (
    SELECT 1
    FROM interviews AS i
    WHERE i.candidate_id = c.candidate_id
);
```

Conceptually, the inner expression depends on the current outer row.

Optimizers may transform correlated queries into more efficient plans.

---

# 26. EXISTS

Use EXISTS when the question is:

```text
Does at least one matching row exist?
```

Example:

```sql
SELECT *
FROM candidates AS c
WHERE EXISTS (
    SELECT 1
    FROM interviews AS i
    WHERE i.candidate_id = c.candidate_id
);
```

Do not claim:

```text
EXISTS is always faster than IN
```

Performance depends on semantics, optimizer behavior, data, and indexes.

---

# 27. NULL

NULL represents absence or unknownness according to the schema semantics.

It is not the same as:

```text
0

false

''
```

Incorrect:

```sql
WHERE score = NULL
```

Correct:

```sql
WHERE score IS NULL
```

or:

```sql
WHERE score IS NOT NULL
```

---

# 28. Three-Valued Logic

SQL conditions can conceptually evaluate to:

```text
TRUE

FALSE

UNKNOWN
```

NULL comparisons often produce:

```text
UNKNOWN
```

Example:

```sql
NULL = NULL
```

does not behave like ordinary equality.

Use:

```sql
IS NULL
```

when checking for NULL.

---

# 29. COUNT and NULL

```sql
COUNT(*)
```

counts rows.

```sql
COUNT(column)
```

counts non-NULL values of that expression.

Example:

```text
score

80
NULL
90
```

Then:

```text
COUNT(*) = 3

COUNT(score) = 2
```

---

# 30. Views

A view is a stored query definition presented like a relation.

Example:

```sql
CREATE VIEW completed_interviews AS
SELECT *
FROM interviews
WHERE status = 'completed';
```

A regular view typically does not store the complete result physically.

---

# 31. Materialized View

A materialized view stores query results physically.

Useful for expensive:

```text
Aggregations

Joins

Reporting queries
```

Trade-off:

```text
Faster reads

vs

Refresh complexity and potentially stale data
```

---

# 32. Stored Procedure vs Function

Exact semantics vary by DBMS.

Generally:

## Function

Often:

```text
Accepts parameters

Returns a value or relation

Can sometimes be used in expressions
```

## Procedure

Often:

```text
Performs a sequence of database operations

May support transaction-related operations depending on DBMS
```

Do not make universal syntax claims across database systems.

---

# 33. Trigger

A trigger executes automatically when a configured database event occurs.

Examples:

```text
INSERT

UPDATE

DELETE
```

Potential uses:

```text
Audit logging

Derived data

Validation
```

Potential problems:

```text
Hidden side effects

Debugging difficulty

Unexpected write amplification
```

Use deliberately.

---

# 34. Normalization

Normalization organizes relations to reduce undesirable redundancy and anomalies.

Main interview forms:

```text
1NF

2NF

3NF

BCNF
```

The goal is not:

```text
Maximum number of tables
```

The goal is:

```text
Correct representation of dependencies and facts
```

---

# 35. Functional Dependency

Notation:

```text
X → Y
```

means:

> If two tuples have the same X value, they must have the same Y value.

Example:

```text
candidate_id → candidate_email
```

if candidate ID uniquely determines candidate email.

Functional dependencies drive normalization.

---

# 36. 1NF

A relation in 1NF has values represented according to the relational model without repeating groups inside a single field.

Bad conceptual design:

```text
candidate_id | skills
101          | Java, Python, SQL
```

when skills need to be independently queried and related.

Better:

```text
Candidate

Skill

CandidateSkill
```

1NF is more precise than simply saying:

```text
Every value must be atomic
```

because atomicity depends on the modeled domain.

---

# 37. 2NF

A relation is in 2NF when:

```text
It is in 1NF

and

Every non-prime attribute is fully dependent on every candidate key
```

2NF mainly matters when composite candidate keys exist.

It removes problematic:

```text
Partial dependencies
```

---

# 38. 2NF Example

Suppose:

```text
Enrollment(
    student_id,
    course_id,
    student_name,
    course_name,
    grade
)
```

Key:

```text
(student_id, course_id)
```

Dependencies:

```text
student_id → student_name

course_id → course_name

(student_id, course_id) → grade
```

Problem:

```text
student_name
```

depends only on part of the key.

Similarly:

```text
course_name
```

depends only on another part.

Decompose into:

```text
Student

Course

Enrollment
```

---

# 39. 3NF

A relation is in 3NF when it satisfies 2NF and avoids problematic transitive dependencies of non-key attributes on keys.

Example:

```text
Employee(
    employee_id,
    department_id,
    department_name
)
```

Dependencies:

```text
employee_id → department_id

department_id → department_name
```

Therefore:

```text
employee_id → department_name
```

transitively.

Better:

```text
Employee(
    employee_id,
    department_id
)
```

```text
Department(
    department_id,
    department_name
)
```

---

# 40. BCNF

A relation is in BCNF when for every non-trivial functional dependency:

```text
X → Y
```

X is a super key.

BCNF is stricter than 3NF.

Remember:

```text
BCNF
→ Every determinant must be a super key
```

---

# 41. 3NF vs BCNF

Every BCNF relation is in 3NF.

But every 3NF relation is not necessarily in BCNF.

BCNF can require further decomposition where 3NF still permits certain dependencies involving prime attributes.

---

# 42. Anomalies

Poor schema design can cause:

```text
Insertion anomaly

Update anomaly

Deletion anomaly
```

Example:

```text
CandidateInterview(
    candidate_id,
    candidate_email,
    interview_id,
    score
)
```

Candidate email is repeated across many interviews.

Changing the email requires multiple updates.

That creates an:

```text
Update anomaly
```

---

# 43. Denormalization

Denormalization intentionally introduces controlled redundancy to improve particular workloads.

Possible reasons:

```text
Reduce expensive joins

Precompute aggregates

Improve read latency

Analytics workloads
```

Trade-off:

```text
Read performance

vs

Consistency and update complexity
```

Denormalize based on evidence, not assumptions.

---

# 44. ER Model

The ER model represents:

```text
Entities

Attributes

Relationships

Cardinality

Participation
```

Example:

```text
Candidate 1 ─── N Interview
```

means one candidate may be associated with many interviews.

---

# 45. Cardinality

Common relationships:

```text
1:1

1:N

M:N
```

Implementation:

```text
1:N
→ Foreign key on many side

M:N
→ Junction table

1:1
→ Foreign key + UNIQUE or shared primary key
```

---

# 46. Many-to-Many

Example:

```text
Candidate M ─── N Skill
```

Implementation:

```text
CandidateSkill(
    candidate_id,
    skill_id
)
```

Possible key:

```text
(candidate_id, skill_id)
```

---

# 47. Weak Entity

A weak entity depends on an owner entity's key for identification.

Example:

```text
Employee

Dependent
```

Possible dependent identity:

```text
(employee_id, dependent_name)
```

Not every child table is a weak entity.

---

# 48. Generalization and Specialization

Generalization:

```text
Candidate
Recruiter
Admin
    ↓
User
```

Specialization:

```text
User
 ↓
Candidate
Recruiter
Admin
```

Related concepts:

```text
Disjoint

Overlapping

Total

Partial
```

---

# 49. Transaction

A transaction is a logical unit of database work.

Example:

```text
Deduct credits

Create interview

Record credit transaction
```

These operations may need to succeed or fail together.

---

# 50. ACID

Remember:

```text
A → Atomicity

C → Consistency

I → Isolation

D → Durability
```

---

# 51. Atomicity

A transaction is treated as an indivisible unit with respect to commit/rollback semantics.

Conceptually:

```text
All required operations commit

or

Transaction is rolled back
```

Example:

```text
Debit account

Credit account
```

The system should not commit only half of the transfer.

---

# 52. Consistency

A committed transaction should preserve database invariants, assuming correct transaction logic and constraints.

Examples:

```text
Primary-key uniqueness

Foreign-key validity

CHECK constraints

Business invariants
```

Consistency does not mean:

```text
All replicas always contain identical data at every instant
```

That is a different distributed-systems concept.

---

# 53. Isolation

Concurrent transactions should behave according to the guarantees of the configured isolation level.

Isolation controls anomalies such as:

```text
Dirty reads

Non-repeatable reads

Phantoms

Serialization anomalies
```

---

# 54. Durability

Once a transaction commits, its effects should survive failures according to the DBMS's durability guarantees.

Mechanisms may include:

```text
Write-ahead logging

Persistent storage

Recovery protocols

Replication
```

---

# 55. Transaction Commands

Typical commands:

```sql
BEGIN;

UPDATE ...;

INSERT ...;

COMMIT;
```

or:

```sql
ROLLBACK;
```

Savepoint:

```sql
SAVEPOINT step1;
```

allows partial rollback within a transaction where supported.

---

# 56. Concurrency Problems

Important anomalies:

```text
Dirty Read

Non-Repeatable Read

Phantom Read

Lost Update

Write Skew
```

Know the difference.

---

# 57. Dirty Read

Transaction T1 modifies data but has not committed.

T2 reads that uncommitted value.

Then T1 rolls back.

T2 observed data that never became committed.

That is a:

```text
Dirty read
```

---

# 58. Non-Repeatable Read

T1 reads row:

```text
score = 70
```

T2 changes it to:

```text
score = 90
```

and commits.

T1 reads the same row again and sees:

```text
90
```

The same row produced different committed values during one transaction.

---

# 59. Phantom Read

T1 executes:

```sql
SELECT *
FROM interviews
WHERE score >= 90;
```

T2 inserts another matching row and commits.

T1 executes the predicate query again and sees an additional row.

That new matching row is a:

```text
Phantom
```

---

# 60. Lost Update

Two transactions read:

```text
credits = 10
```

T1 computes:

```text
9
```

T2 also computes:

```text
9
```

Both write:

```text
9
```

Two deductions occurred logically, but final value reflects only one.

This is a:

```text
Lost update
```

---

# 61. Write Skew

Two transactions read overlapping data and update different rows while jointly violating a business invariant.

This can occur under some snapshot-based isolation models.

Example invariant:

```text
At least one interviewer must remain on duty.
```

Two transactions each observe two interviewers and independently disable a different one.

Final result:

```text
0 interviewers on duty
```

even though neither transaction directly overwrote the other's row.

---

# 62. Isolation Levels

Standard conceptual levels:

```text
Read Uncommitted

Read Committed

Repeatable Read

Serializable
```

Actual behavior differs across DBMS implementations.

Do not assume identical semantics from the isolation-level name alone.

---

# 63. Read Uncommitted

Provides weak isolation.

Conceptually may allow:

```text
Dirty reads
```

Some DBMS implementations provide stronger behavior even when this level is requested.

---

# 64. Read Committed

Prevents dirty reads.

But depending on implementation, repeated reads may observe newly committed changes.

Commonly possible:

```text
Non-repeatable reads

Phantoms
```

---

# 65. Repeatable Read

Provides stronger consistency for repeated reads within a transaction.

Exact behavior regarding:

```text
Phantoms

Write skew
```

depends on the DBMS implementation.

---

# 66. Serializable

Strongest standard isolation level.

Goal:

> Concurrent transactions behave as though they executed in some serial order.

Implementation may use:

```text
Locks

Predicate locking

Serializable snapshot isolation

Conflict detection
```

Serializable transactions may require retries after serialization failures.

---

# 67. Lock

Locks coordinate concurrent access to database resources.

Common conceptual categories:

```text
Shared lock

Exclusive lock
```

## Shared

Generally permits compatible readers.

## Exclusive

Used for operations requiring exclusive modification rights.

Exact lock modes vary by DBMS.

---

# 68. Two-Phase Locking

Two-phase locking conceptually contains:

```text
Growing Phase
→ Acquire locks

Shrinking Phase
→ Release locks
```

Strict variants retain important locks until commit or rollback.

2PL is a concurrency-control protocol.

---

# 69. Deadlock

Example:

```text
T1 holds A
T1 waits for B

T2 holds B
T2 waits for A
```

Neither can proceed.

This is a deadlock.

DBMSs may:

```text
Detect cycles

Abort one transaction

Release its locks
```

The application should be prepared to retry when appropriate.

---

# 70. Deadlock Prevention

Useful strategies:

```text
Access resources in consistent order

Keep transactions short

Avoid unnecessary locks

Index queries appropriately

Retry aborted transactions safely
```

You cannot assume all deadlocks can be eliminated.

---

# 71. Optimistic Concurrency Control

Optimistic approaches assume conflicts are relatively uncommon.

Typical pattern:

```text
Read

Modify

Validate version

Write if unchanged
```

Example:

```sql
UPDATE candidates
SET credits = 9,
    version = version + 1
WHERE candidate_id = 101
  AND version = 7;
```

If:

```text
0 rows updated
```

another transaction may have changed the row.

---

# 72. Pessimistic Concurrency Control

Pessimistic approaches lock resources before conflicting operations occur.

Example concept:

```sql
SELECT ...
FOR UPDATE;
```

Useful when conflicts are likely or a transaction needs protected access to rows.

Trade-off:

```text
Stronger coordination

vs

Blocking and contention
```

---

# 73. MVCC

**Multi-Version Concurrency Control** maintains multiple row versions so readers and writers can often proceed with less direct blocking.

Conceptually:

```text
Old version
New version
Transaction visibility rules
```

Transactions see versions according to:

```text
Snapshot

Isolation level

Commit state
```

Implementation varies across DBMSs.

---

# 74. MVCC Does Not Mean No Locks

Incorrect:

```text
MVCC eliminates locking.
```

MVCC can reduce reader-writer blocking.

But databases may still use locks for:

```text
Writes

DDL

Constraints

Explicit locking

Internal coordination
```

---

# 75. Index

An index is an auxiliary data structure that improves certain data-access operations.

Trade-off:

```text
Faster reads

vs

Additional storage and write cost
```

Common index structures include:

```text
B-tree family

Hash

Bitmap

Specialized text/spatial indexes
```

Support varies by DBMS.

---

# 76. B-Tree Index

B-tree-family indexes support ordered lookup.

Useful for operations such as:

```text
Equality

Range queries

Ordering

Prefix access in composite indexes
```

Typical complexity is often described as approximately:

```text
O(log n)
```

for tree navigation, though real database performance also depends heavily on I/O, caching, selectivity, and row retrieval.

---

# 77. Hash Index

Hash indexes are designed around hash-based lookup.

Conceptually strong for:

```text
Equality
```

but not naturally ordered for:

```text
Range scans

ORDER BY
```

Exact capabilities depend on the database.

---

# 78. Clustered vs Non-Clustered

Conceptually:

## Clustered

Data organization follows the index's ordering or structure.

## Non-Clustered

Index is separate from the base data and references rows.

Exact terminology and implementation differ substantially across DBMSs.

Avoid claiming every database supports exactly one clustered index in the same way.

---

# 79. Composite Index

Example:

```text
(candidate_id, created_at)
```

Useful query pattern:

```sql
WHERE candidate_id = ?
ORDER BY created_at DESC;
```

Column order matters.

---

# 80. Covering Index

An index covers a query when it contains all data required by that query.

Potential benefit:

```text
Avoid base-table row lookup
```

Whether the engine can perform a true index-only scan depends on implementation details.

---

# 81. Index Selectivity

Highly selective condition:

```text
email = unique value
```

Low-selectivity condition:

```text
is_active = true
```

when almost everyone is active.

Indexes tend to be more valuable when they significantly reduce the rows that must be processed.

---

# 82. Query Processing Pipeline

Remember:

```text
SQL
 ↓
Parsing
 ↓
Semantic Analysis
 ↓
Rewrite
 ↓
Optimization
 ↓
Execution Plan
 ↓
Execution
```

---

# 83. Query Optimizer

The optimizer evaluates possible execution strategies.

It considers factors such as:

```text
Statistics

Indexes

Cardinality

Selectivity

Join order

Join algorithms

I/O

CPU

Memory
```

The objective is to choose a relatively low-cost execution plan.

---

# 84. Logical vs Physical Plan

Logical:

```text
Scan

Filter

Join

Project

Aggregate
```

Physical:

```text
Sequential Scan

Index Scan

Hash Join

Nested Loop

Merge Join

Hash Aggregate
```

Think:

```text
Logical
→ What relational operations?

Physical
→ How will those operations run?
```

---

# 85. Sequential Scan

Reads table data and evaluates rows.

Good when:

```text
Table is small

Large percentage of rows is required
```

A sequential scan is not automatically a performance problem.

---

# 86. Index Scan

Uses an index to locate qualifying rows.

Often effective for:

```text
Selective predicates
```

Example:

```sql
WHERE email = ?
```

with a suitable email index.

---

# 87. Join Algorithms

Three major algorithms to remember:

```text
Nested Loop Join

Hash Join

Merge Join
```

---

# 88. Nested Loop Join

Conceptually:

```text
For each outer row
    find matching inner rows
```

Often effective when:

```text
Outer input is small

Inner lookup is indexed
```

---

# 89. Hash Join

Conceptually:

```text
Build hash table from one input

Probe with the other input
```

Commonly effective for:

```text
Equality joins

Large datasets
```

subject to memory and cost considerations.

---

# 90. Merge Join

Conceptually:

```text
Ordered input A

Ordered input B

Walk through both
```

Can be effective when inputs are already ordered or ordering can be obtained efficiently.

---

# 91. EXPLAIN

Use:

```sql
EXPLAIN
```

to inspect the planned execution strategy.

Look for:

```text
Scan types

Join types

Estimated rows

Estimated costs

Sorts

Index usage
```

Exact output varies by DBMS.

---

# 92. EXPLAIN ANALYZE

Where supported, an analyze mode executes the query and provides actual runtime information.

Compare:

```text
Estimated rows

vs

Actual rows
```

Large differences may indicate:

```text
Poor statistics

Data correlation

Parameter sensitivity

Difficult predicates
```

Be careful because analyzing write statements may actually modify data.

---

# 93. Sargability

Sargable:

```sql
WHERE created_at >= ?
```

Potentially less sargable:

```sql
WHERE DATE(created_at) = ?
```

Applying functions to indexed columns can prevent efficient direct index access unless the database supports an appropriate expression index or equivalent strategy.

---

# 94. N+1 Problem

Bad application pattern:

```text
1 query for candidates

+

1 interview query per candidate
```

For:

```text
1,000 candidates
```

this becomes:

```text
1,001 queries
```

Possible fixes:

```text
JOIN

Batch loading

Eager loading

IN query
```

depending on the required data shape.

---

# 95. OFFSET vs Keyset Pagination

OFFSET:

```sql
LIMIT 20 OFFSET 100000;
```

can become expensive for deep pages.

Keyset:

```sql
WHERE created_at < ?
ORDER BY created_at DESC
LIMIT 20;
```

continues from the previous ordering key.

For stable ordering, include a unique tie-breaker when necessary.

---

# 96. Database Recovery

Recovery mechanisms protect database state after failures.

Important concepts:

```text
Transaction log

Write-ahead logging

Checkpoint

Undo

Redo
```

Exact algorithms differ across DBMSs.

---

# 97. Write-Ahead Logging

Core principle:

> Log information required for recovery is persisted before the corresponding modified data page is considered safely persisted.

This allows recovery mechanisms to reconstruct committed changes after failures.

---

# 98. Undo vs Redo

## Undo

Reverses effects that should not remain.

## Redo

Reapplies effects that should remain but may not yet be reflected in persisted data pages.

Recovery algorithms determine which operations need undo or redo.

---

# 99. Checkpoint

A checkpoint gives the recovery system a known progress point.

Its purpose is to reduce recovery work.

It does not necessarily mean:

```text
Every database page is immediately written to disk at that exact moment.
```

Implementation varies.

---

# 100. Backup vs Replication

Replication is not a replacement for backup.

Why?

If someone executes:

```sql
DELETE FROM candidates;
```

that deletion may replicate to replicas.

Backup provides a historical recovery point.

Think:

```text
Replication
→ Availability / redundancy

Backup
→ Recovery from loss or corruption
```

---

# 101. OLTP vs OLAP

## OLTP

Optimized for transactional workloads.

Characteristics:

```text
Many short transactions

Frequent inserts/updates

Low latency

High concurrency
```

Example:

```text
Creating an interview session
```

## OLAP

Optimized for analytical workloads.

Characteristics:

```text
Large scans

Aggregations

Historical analysis

Complex queries
```

Example:

```text
Average interview performance by role over 12 months
```

---

# 102. Row Store vs Column Store

## Row-Oriented

Stores values of a row together.

Often suitable for:

```text
OLTP

Point lookups

Whole-row operations
```

## Column-Oriented

Stores values by column.

Often suitable for:

```text
Analytics

Aggregations

Compression

Scanning selected columns
```

These are workload-oriented design choices.

---

# 103. Partitioning

Partitioning divides a logical table into smaller physical partitions according to a partitioning strategy.

Possible keys:

```text
Date

Range

List

Hash
```

Potential benefits:

```text
Partition pruning

Maintenance

Data lifecycle management
```

Partitioning is not automatically a performance improvement.

---

# 104. Sharding

Sharding distributes data across multiple database nodes or independent database instances.

Example:

```text
Shard 1
→ Users A–M

Shard 2
→ Users N–Z
```

or hash-based distribution.

Sharding introduces complexity:

```text
Routing

Cross-shard queries

Rebalancing

Transactions

Operational overhead
```

Do not shard prematurely.

---

# 105. Partitioning vs Sharding

Conceptually:

## Partitioning

Divides data into partitions, often within one database system.

## Sharding

Distributes data across independent nodes/databases.

Sharding is a distributed architecture decision.

---

# 106. Replication

Replication maintains copies of data across nodes.

Common patterns:

```text
Primary → Replica

Multi-primary

Leaderless
```

depending on the system.

Possible goals:

```text
Availability

Read scaling

Disaster recovery support

Geographic distribution
```

---

# 107. Replication Lag

In asynchronous replication:

```text
Primary receives write
       ↓
Commit
       ↓
Replica receives change later
```

A read from the replica immediately after a write may return stale data.

This matters for:

```text
Read-after-write consistency
```

---

# 108. CAP Theorem

For a distributed data system experiencing a network partition, CAP says you cannot simultaneously guarantee both:

```text
Strong consistency

and

Availability
```

while also tolerating the partition.

CAP concerns:

```text
Consistency

Availability

Partition tolerance
```

Do not summarize it as:

```text
Choose any two all the time.
```

The meaningful trade-off occurs during partitions.

---

# 109. SQL vs NoSQL

Do not answer:

```text
SQL is old.

NoSQL is faster.
```

Instead compare:

```text
Data model

Consistency requirements

Query patterns

Relationships

Transactions

Scale

Operational requirements
```

Relational databases are excellent for structured data with relationships and strong transactional requirements.

NoSQL systems cover several different models:

```text
Document

Key-value

Wide-column

Graph
```

---

# 110. Database Design Workflow

When designing a database:

```text
1. Understand requirements

2. Identify entities

3. Identify attributes

4. Define relationships

5. Determine cardinalities

6. Choose keys

7. Define constraints

8. Normalize

9. Identify historical requirements

10. Understand query patterns

11. Add indexes

12. Consider transactions/concurrency

13. Consider lifecycle and deletion

14. Measure production behavior
```

Do not start with random tables.

Start with business facts.

---

# 111. Scenario: Candidate and Interview

Requirement:

```text
One candidate can take many interviews.

Each interview belongs to exactly one candidate.
```

Model:

```text
Candidate 1:N Interview
```

Schema:

```text
Candidate(
    candidate_id PK
)
```

```text
Interview(
    interview_id PK,
    candidate_id FK NOT NULL
)
```

Foreign key belongs on the:

```text
many side
```

---

# 112. Scenario: Candidate Skills

Requirement:

```text
One candidate has many skills.

One skill belongs to many candidates.
```

Model:

```text
Candidate M:N Skill
```

Schema:

```text
CandidateSkill(
    candidate_id FK,
    skill_id FK,

    PRIMARY KEY(candidate_id, skill_id)
)
```

---

# 113. Scenario: Credit Deduction

Suppose candidate has:

```text
credits = 1
```

Two requests simultaneously start interviews.

Bad logic:

```text
Read credits

Check credits > 0

Subtract 1

Save
```

Both requests may observe:

```text
credits = 1
```

and both succeed.

Use concurrency-safe database logic such as an atomic conditional update:

```sql
UPDATE candidates
SET credits = credits - 1
WHERE candidate_id = ?
  AND credits > 0;
```

Then verify whether a row was updated.

---

# 114. Scenario: Slow Candidate History

Query:

```sql
SELECT interview_id, score, created_at
FROM interviews
WHERE candidate_id = ?
ORDER BY created_at DESC
LIMIT 20;
```

Potential index:

```text
(candidate_id, created_at)
```

Reason:

```text
Filter by candidate

+

order by created_at
```

But confirm using the execution plan and actual workload.

---

# 115. Scenario: Duplicate Email

Bad:

```text
Application checks whether email exists

Then inserts user
```

Two concurrent requests can both pass the check.

Correct protection includes a database constraint:

```sql
UNIQUE(email)
```

The application should handle the resulting constraint violation appropriately.

---

# 116. Scenario: Transfer

Transfer:

```text
Account A → Account B
```

Operations:

```text
Debit A

Credit B
```

Need:

```text
Transaction
```

because partial completion is invalid.

Also consider:

```text
Concurrent updates

Balance constraints

Deadlocks

Retry behavior
```

---

# 117. Scenario: Dashboard Analytics

Dashboard repeatedly calculates:

```text
Total interviews

Average score

Completion rate

Last interview date
```

over hundreds of millions of rows.

Possible solutions:

```text
Indexes

Precomputed aggregates

Materialized views

Analytics database

Incremental summary tables
```

Do not immediately denormalize without measuring the workload.

---

# 118. Scenario: Deadlock

T1:

```text
Lock Candidate 1

Then Candidate 2
```

T2:

```text
Lock Candidate 2

Then Candidate 1
```

Possible deadlock.

Improve by using consistent ordering:

```text
Always lock lower candidate_id first
```

and keeping transactions short.

---

# 119. Scenario: Deep Pagination

Bad at scale:

```sql
LIMIT 20 OFFSET 500000;
```

Potentially better:

```text
Keyset pagination
```

using the last seen:

```text
(created_at, interview_id)
```

especially for feed-like ordered navigation.

---

# 120. Scenario: Interview History

Suppose a question's text changes after an interview.

Requirement:

```text
Historical interview must show exactly what candidate was asked.
```

Simply referencing a mutable Question row may be insufficient.

Possible solution:

```text
Question versioning

or

Question snapshot stored with interview attempt
```

This is a data-modeling problem, not merely a UI problem.

---

# 121. Rapid-Fire Interview Questions

### What is DBMS?

Software that manages storage, retrieval, modification, integrity, concurrency, and access to database data.

### What is RDBMS?

A DBMS based on the relational model.

### What is a primary key?

The selected candidate key used as the primary identifier of tuples.

### What is a foreign key?

An attribute set that references a key in another or the same relation and enforces referential integrity.

### What is normalization?

A process of organizing relations based on dependencies to reduce problematic redundancy and anomalies.

### What is denormalization?

Intentional introduction of controlled redundancy for workload-specific reasons.

### What is ACID?

Atomicity, Consistency, Isolation, Durability.

### What is a transaction?

A logical unit of database work with commit/rollback semantics.

### What is an index?

An auxiliary data structure that improves selected access patterns at the cost of storage and write overhead.

### What is a deadlock?

A cycle of transactions waiting for resources held by each other.

---

# 122. Rapid-Fire SQL

### Second highest salary

One possible solution:

```sql
SELECT MAX(salary)
FROM employees
WHERE salary < (
    SELECT MAX(salary)
    FROM employees
);
```

This returns the second highest **distinct** salary.

---

### Duplicate emails

```sql
SELECT email, COUNT(*)
FROM users
GROUP BY email
HAVING COUNT(*) > 1;
```

---

### Candidates without interviews

```sql
SELECT c.*
FROM candidates AS c
LEFT JOIN interviews AS i
    ON i.candidate_id = c.candidate_id
WHERE i.candidate_id IS NULL;
```

Alternative:

```sql
SELECT c.*
FROM candidates AS c
WHERE NOT EXISTS (
    SELECT 1
    FROM interviews AS i
    WHERE i.candidate_id = c.candidate_id
);
```

---

### Interview count per candidate

```sql
SELECT
    candidate_id,
    COUNT(*) AS interview_count
FROM interviews
GROUP BY candidate_id;
```

---

### Top 3 scores

```sql
SELECT DISTINCT score
FROM interviews
ORDER BY score DESC
LIMIT 3;
```

Syntax varies across DBMSs.

---

# 123. Window Functions

Window functions compute values across related rows without collapsing them into one row per group.

Example:

```sql
SELECT
    candidate_id,
    score,
    RANK() OVER (
        ORDER BY score DESC
    ) AS score_rank
FROM interviews;
```

Unlike GROUP BY:

```text
Individual rows remain visible
```

---

# 124. ROW_NUMBER vs RANK vs DENSE_RANK

Scores:

```text
100

90

90

80
```

## ROW_NUMBER

```text
1

2

3

4
```

## RANK

```text
1

2

2

4
```

## DENSE_RANK

```text
1

2

2

3
```

Remember this distinction.

---

# 125. Top Interview per Candidate

Example:

```sql
WITH ranked AS (
    SELECT
        interview_id,
        candidate_id,
        score,
        ROW_NUMBER() OVER (
            PARTITION BY candidate_id
            ORDER BY score DESC
        ) AS rn
    FROM interviews
)
SELECT *
FROM ranked
WHERE rn = 1;
```

A deterministic tie-breaker may be added if required.

---

# 126. CTE

A Common Table Expression improves query organization.

Example:

```sql
WITH completed AS (
    SELECT *
    FROM interviews
    WHERE status = 'completed'
)
SELECT candidate_id, AVG(score)
FROM completed
GROUP BY candidate_id;
```

Do not assume CTEs always improve or harm performance.

Optimizer behavior differs across databases and versions.

---

# 127. Database Interview Red Flags

Avoid statements like:

```text
Indexes always make queries faster.

Joins are slow.

Subqueries are bad.

NoSQL is faster than SQL.

Serializable means one transaction physically runs at a time.

MVCC means databases do not use locks.

Foreign keys automatically create indexes everywhere.

TRUNCATE can never be rolled back.

Every table must have an auto-increment ID.

Normalization always improves performance.

Full table scans are always bad.

EXISTS is always faster than IN.
```

These are oversimplifications or incorrect universal claims.

---

# 128. How to Answer DBMS Questions

For conceptual questions:

```text
1. Define concept

2. Explain purpose

3. Give example

4. Mention trade-off or caveat
```

Example:

> An index is an auxiliary structure used to accelerate selected access patterns. A B-tree index on candidate_id can make selective lookups faster, but indexes consume storage and increase write maintenance, so I would choose them based on query patterns rather than indexing every column.

That is stronger than:

> Index makes queries fast.

---

# 129. How to Answer Comparison Questions

Use:

```text
Definition

Core difference

Use case

Trade-off
```

For:

```text
Optimistic vs Pessimistic Locking
```

explain:

```text
How conflicts are handled

When conflicts are expected

Blocking behavior

Retry behavior
```

Do not merely expand terminology.

---

# 130. How to Answer SQL Questions

Before writing SQL:

```text
Understand schema

Understand expected output

Clarify duplicate behavior

Clarify NULL behavior

Identify grouping level

Identify ordering

Then write query
```

After writing:

```text
Test edge cases mentally
```

such as:

```text
No rows

Duplicate values

NULL

Ties

Multiple matches
```

---

# 131. How to Answer Database Design Questions

Use:

```text
Requirements
 ↓
Entities
 ↓
Relationships
 ↓
Cardinality
 ↓
Keys
 ↓
Constraints
 ↓
Normalization
 ↓
Transactions
 ↓
Indexes
 ↓
Scale
```

Do not jump directly to:

```text
We need PostgreSQL + Redis + Kafka + sharding.
```

Start with the data model and workload.

---

# 132. DBMS Revision Checklist

Before an interview, verify that you can explain without notes:

```text
DBMS vs RDBMS

Schema vs instance

Keys

Constraints

SQL command categories

JOIN types

WHERE vs HAVING

GROUP BY

Subqueries

NULL

Views

Normalization

Functional dependencies

1NF

2NF

3NF

BCNF

ER modeling

Cardinality

Weak entities

Transactions

ACID

Isolation levels

Concurrency anomalies

Locks

Deadlocks

MVCC

Indexes

Composite indexes

Query optimizer

Execution plans

Join algorithms

N+1

Recovery

WAL

Partitioning

Replication

Sharding

OLTP vs OLAP
```

If one concept cannot be explained in:

```text
30–60 seconds
```

review its detailed guide.

---

# 133. Final 20 Questions

Try answering these without looking at the notes.

### 1.

Why is every candidate key a super key, but every super key is not a candidate key?

### 2.

Why does adding a surrogate primary key not eliminate the need for business UNIQUE constraints?

### 3.

How would you implement a many-to-many relationship?

### 4.

What is the difference between 3NF and BCNF?

### 5.

Why can denormalization improve performance?

### 6.

Explain all four ACID properties using one transaction example.

### 7.

What is the difference between a dirty read and a non-repeatable read?

### 8.

What is a phantom read?

### 9.

How does a lost update occur?

### 10.

What is the difference between optimistic and pessimistic concurrency control?

### 11.

How can a deadlock occur?

### 12.

What problem does MVCC solve?

### 13.

Why can an index make writes slower?

### 14.

Why might a database choose a sequential scan even when an index exists?

### 15.

Nested loop vs hash join?

### 16.

What does cardinality estimation influence?

### 17.

Why can `OFFSET 500000` become expensive?

### 18.

How would you diagnose a slow SQL query?

### 19.

Why is replication not a backup?

### 20.

How would you design a database for an interview platform where candidates can make multiple attempts and historical questions must remain unchanged?

---

# 134. Final 10 SQL Challenges

Practice writing SQL for:

```text
1. Find second highest distinct salary.

2. Find Nth highest salary.

3. Find duplicate emails.

4. Find candidates with no interviews.

5. Find interview count per candidate.

6. Find candidate with highest average score.

7. Find latest interview for each candidate.

8. Rank candidates by score.

9. Find candidates with more than five completed interviews.

10. Find the top three scores for each role.
```

Do not memorize only final queries.

Understand:

```text
GROUP BY

HAVING

JOIN

Subquery

CTE

Window functions

Ranking

NULL handling
```

---

# 135. Interview-Day Revision

Immediately before a DBMS interview, prioritize:

```text
ACID

Transactions

Isolation levels

Concurrency anomalies

Normalization

Keys

Joins

Indexes

Execution plans

SQL practice
```

Then revise:

```text
ER modeling

Recovery

MVCC

Deadlocks

Partitioning

Replication
```

The first group appears more frequently in general software-engineering interviews.

---

# 136. Complete DBMS Mental Model

Think of a database system as several layers:

```text
Application
      ↓
SQL / Query Interface
      ↓
Parser
      ↓
Optimizer
      ↓
Execution Engine
      ↓
Transaction / Concurrency Manager
      ↓
Buffer / Storage Manager
      ↓
Logging / Recovery
      ↓
Persistent Storage
```

Across those layers:

```text
Schema
→ defines structure

Constraints
→ protect invariants

Transactions
→ group operations

Isolation
→ controls concurrency

Indexes
→ optimize access

Optimizer
→ selects execution strategy

Logging
→ supports recovery

Replication
→ copies data across nodes
```

Understanding how these pieces interact is more valuable than memorizing isolated definitions.

---

# 137. Final Takeaway

DBMS interviews are not primarily about memorizing terminology.

You should be able to reason through:

```text
How should the data be modeled?

Which constraints protect correctness?

Which operations require transactions?

What happens when two requests execute concurrently?

Which index matches this query?

Why did the optimizer choose this plan?

How will the system recover after failure?

What happens when the dataset becomes large?
```

A strong DBMS foundation connects:

```text
Data Modeling
      ↓
SQL
      ↓
Transactions
      ↓
Concurrency
      ↓
Indexes
      ↓
Query Optimization
      ↓
Recovery
      ↓
Scalability
```

The objective is not:

```text
Memorize 500 database definitions.
```

The objective is:

> Understand how a database preserves correctness while storing and retrieving data efficiently under concurrent workloads.

---

## Previous

**[← Query Processing and Optimization](./11-query-processing-and-optimization.md)**

## Back to DBMS

**[DBMS Overview →](./README.md)**

---

Maintained by **InterviewEraHQ**.
