# Query Processing and Optimization

When an application sends SQL to a database, the DBMS does not simply read the SQL statement and execute it line by line.

It must determine **how** to retrieve the requested data efficiently.

A simplified flow is:

```text
SQL Query
    ↓
Parsing
    ↓
Validation
    ↓
Query Rewrite
    ↓
Optimization
    ↓
Execution Plan
    ↓
Execution
    ↓
Result
```

A strong candidate should understand:

- Query processing
- Parsing and validation
- Logical and physical plans
- Query optimizer
- Cost-based optimization
- Execution plans
- Table scans
- Index scans
- Index-only scans
- Selectivity
- Cardinality estimation
- Statistics
- Join algorithms
- Nested loop join
- Hash join
- Sort-merge join
- Predicate pushdown
- Projection pruning
- Sargability
- Composite indexes
- Covering indexes
- Query-plan analysis
- Common SQL performance mistakes

The central idea is:

> SQL describes the result you want; the query optimizer decides how the database should obtain it.

---

# 1. Declarative Nature of SQL

SQL is primarily declarative.

When you write:

```sql
SELECT name
FROM candidates
WHERE email = 'alex@example.com';
```

you specify:

```text
What data you want
```

not necessarily:

```text
Exactly how the database should retrieve it
```

The DBMS may choose:

```text
Sequential scan

Index scan

Index-only scan
```

depending on the schema, statistics, indexes, and estimated cost.

---

# 2. Query Processing

Query processing is the sequence of operations used by a DBMS to transform a SQL query into executable database operations.

Conceptually:

```text
SQL
 ↓
Parse
 ↓
Analyze
 ↓
Rewrite
 ↓
Optimize
 ↓
Execute
```

Different DBMS implementations vary, but this model is useful for interviews.

---

# 3. Parsing

The parser checks SQL syntax and builds an internal representation.

Invalid:

```sql
SELECT FROM candidates;
```

The DBMS detects a syntax error before normal query execution.

Valid:

```sql
SELECT name
FROM candidates;
```

can proceed to semantic analysis.

---

# 4. Semantic Analysis

Syntax can be correct while the query is still invalid.

Example:

```sql
SELECT unknown_column
FROM candidates;
```

The SQL grammar may be valid, but:

```text
unknown_column
```

does not exist.

The DBMS resolves objects such as:

```text
Tables

Columns

Functions

Types

Operators
```

and checks whether the query is semantically valid.

---

# 5. Query Rewrite

Before physical optimization, the DBMS may transform the query into an equivalent form.

Possible transformations include:

```text
Predicate simplification

View expansion

Subquery transformation

Constant folding

Redundant expression elimination
```

The exact rewrite rules depend on the database engine.

---

# 6. Logical Query Plan

A logical plan represents relational operations without committing to exact physical algorithms.

Example query:

```sql
SELECT c.name
FROM candidates AS c
JOIN interviews AS i
    ON i.candidate_id = c.candidate_id
WHERE i.score >= 80;
```

Conceptually:

```text
Scan Candidate
      │
      │
      ├── Join
      │
Scan Interview
      │
Filter score >= 80
      │
Project candidate name
```

A logical plan describes operations such as:

```text
Selection

Projection

Join

Aggregation

Sorting
```

---

# 7. Physical Query Plan

A physical plan chooses concrete execution methods.

Example:

```text
Index Scan on interviews
        ↓
Hash Join
        ↓
Sequential Scan on candidates
        ↓
Projection
```

The optimizer chooses physical operators based on estimated cost.

---

# 8. Query Optimizer

The **query optimizer** evaluates possible execution strategies and attempts to select an efficient plan.

It may decide:

```text
Which table to access first

Whether to use an index

Which index to use

Which join algorithm to use

Join order

Whether sorting is required

Whether predicates can be pushed down
```

Optimization is essential because equivalent SQL queries may have drastically different execution costs.

---

# 9. Cost-Based Optimization

Modern relational databases commonly use **cost-based optimization**.

The optimizer estimates the cost of candidate plans based on factors such as:

```text
Number of rows

Data distribution

Selectivity

Available indexes

I/O

CPU

Memory

Join cardinality

Sort requirements
```

Then it chooses a plan expected to have relatively low cost.

---

# 10. Estimated Cost Is Not Runtime

An optimizer cost such as:

```text
cost = 153.42
```

does not necessarily mean:

```text
153.42 milliseconds
```

Cost is typically an internal estimate used to compare execution plans.

Exact meaning depends on the DBMS.

---

# 11. Statistics

The optimizer needs information about stored data.

Database statistics may describe:

```text
Row counts

Distinct values

Value distributions

Null fractions

Histograms

Correlation
```

These statistics help estimate how many rows each operation will produce.

---

# 12. Why Statistics Matter

Suppose:

```text
1,000,000 users
```

exist.

Query:

```sql
SELECT *
FROM users
WHERE country = 'US';
```

If:

```text
900,000 users
```

match, an index may be less attractive than when:

```text
500 users
```

match.

The optimizer needs distribution information to estimate this.

---

# 13. Stale Statistics

Suppose a table grows from:

```text
10,000 rows
```

to:

```text
100,000,000 rows
```

but optimizer statistics do not reflect the current distribution.

The optimizer may make poor estimates and select inefficient plans.

Depending on the DBMS, statistics may be:

```text
Automatically maintained

Manually analyzed

Updated periodically
```

---

# 14. Cardinality

In query optimization, **cardinality** often refers to the number of rows produced by an operation.

Example:

```text
Table rows = 1,000,000
```

Filter:

```text
status = 'active'
```

Estimated output:

```text
120,000 rows
```

Then estimated cardinality after filtering is:

```text
120,000
```

---

# 15. Cardinality Estimation

Cardinality estimation predicts how many rows will pass through each stage of a query plan.

This affects decisions such as:

```text
Join order

Join algorithm

Index usage

Memory allocation

Parallelism
```

Poor cardinality estimates can produce poor plans.

---

# 16. Selectivity

Selectivity describes how strongly a condition filters rows.

Suppose:

```text
1,000,000 rows
```

exist.

Condition:

```text
email = 'alex@example.com'
```

returns:

```text
1 row
```

This predicate is highly selective.

Condition:

```text
is_active = true
```

returns:

```text
950,000 rows
```

This predicate has low filtering power.

---

# 17. Why Selectivity Matters

Indexes are especially useful when they allow the DBMS to locate a relatively small subset of rows efficiently.

Example:

```sql
WHERE email = ?
```

with:

```text
UNIQUE INDEX(email)
```

may quickly locate one row.

But:

```sql
WHERE is_active = true
```

when nearly every row is active may not benefit much from a simple index.

---

# 18. Sequential Scan

A sequential scan reads the table pages and checks rows against the predicate.

Conceptually:

```text
Row 1 → check
Row 2 → check
Row 3 → check
...
Row N → check
```

This is often described as:

```text
Full table scan
```

though terminology varies by DBMS.

---

# 19. Sequential Scan Is Not Automatically Bad

Suppose:

```text
90%
```

of a table must be returned.

Using an index might require many random lookups.

A sequential scan may be cheaper.

Therefore:

> Seeing a sequential scan in an execution plan does not automatically mean the query is poorly optimized.

Context matters.

---

# 20. Index Scan

An index scan uses an index to locate qualifying entries and then retrieves the required table rows when necessary.

Example:

```sql
SELECT *
FROM candidates
WHERE email = 'alex@example.com';
```

with:

```text
INDEX(email)
```

Conceptually:

```text
Index
  ↓
Locate email
  ↓
Find row reference
  ↓
Fetch candidate row
```

---

# 21. Index-Only Scan

Sometimes all required data can be obtained from the index without fetching the base table row.

Example index:

```text
(email, name)
```

Query:

```sql
SELECT name
FROM candidates
WHERE email = 'alex@example.com';
```

Potentially:

```text
Index
  ↓
email + name available
  ↓
Return result
```

Whether an actual index-only scan is possible depends on the DBMS and visibility/storage details.

---

# 22. Covering Index

A **covering index** contains the columns required to satisfy a particular query.

Example:

```sql
SELECT name, created_at
FROM candidates
WHERE email = ?;
```

An index containing:

```text
email
name
created_at
```

may cover the query.

A covering index is a query-specific concept.

---

# 23. More Indexes Are Not Always Better

Every additional index can increase:

```text
Storage

INSERT cost

UPDATE cost

DELETE cost

Maintenance work
```

Indexes must be selected according to workload.

Do not create indexes on every column.

---

# 24. Composite Index

A composite index contains multiple columns.

Example:

```sql
CREATE INDEX idx_interviews_candidate_status
ON interviews(candidate_id, status);
```

This may help queries such as:

```sql
WHERE candidate_id = ?
  AND status = ?
```

and potentially:

```sql
WHERE candidate_id = ?
```

depending on the DBMS.

Column order matters.

---

# 25. Composite Index Column Order

Suppose index:

```text
(candidate_id, created_at)
```

Query:

```sql
WHERE candidate_id = ?
ORDER BY created_at DESC;
```

This index may be useful because rows for a candidate can be located and ordered according to the index structure.

But:

```sql
WHERE created_at = ?
```

may not benefit in the same way from that index.

Exact behavior depends on the database engine and query.

---

# 26. Leftmost Prefix Concept

For common B-tree composite indexes:

```text
(A, B, C)
```

queries using leading columns often benefit most naturally:

```text
A

A, B

A, B, C
```

A query using only:

```text
C
```

typically cannot exploit the index in the same direct way.

This is a useful rule of thumb, not a universal statement for every index type or optimizer.

---

# 27. Sargability

A predicate is **sargable** when it can effectively use an index search operation.

Example:

```sql
WHERE email = 'alex@example.com'
```

with an index on:

```text
email
```

is typically sargable.

---

# 28. Non-Sargable Predicate

Suppose:

```sql
WHERE LOWER(email) = 'alex@example.com'
```

A normal index on:

```text
email
```

may not be usable for direct lookup because a function is applied to the indexed column.

Possible solutions depend on the DBMS:

```text
Normalize values before storage

Functional/expression index

Case-insensitive data type

Appropriate collation
```

---

# 29. Date Filtering Mistake

Potentially inefficient:

```sql
WHERE DATE(created_at) = '2026-07-26'
```

because a function is applied to:

```text
created_at
```

A more index-friendly range may be:

```sql
WHERE created_at >= '2026-07-26 00:00:00'
  AND created_at <  '2026-07-27 00:00:00'
```

assuming the timestamp and timezone semantics make this range correct.

---

# 30. Implicit Type Conversion

Suppose:

```text
candidate_id
```

is numeric but a query compares it using an incompatible type.

Some databases may perform implicit conversion.

Depending on which side is converted, this can affect:

```text
Index usage

Performance

Correctness
```

Use appropriate parameter types.

---

# 31. Predicate Pushdown

Suppose:

```sql
SELECT *
FROM candidates AS c
JOIN interviews AS i
    ON i.candidate_id = c.candidate_id
WHERE i.status = 'completed';
```

Instead of joining all interviews and filtering later, the engine may filter:

```text
status = completed
```

earlier.

Conceptually:

```text
Interview
   ↓
Filter completed
   ↓
Join Candidate
```

This can reduce the number of rows entering the join.

---

# 32. Projection Pruning

Suppose a table has:

```text
50 columns
```

but the query needs:

```text
name
email
```

The engine can avoid carrying unnecessary columns through parts of the plan.

This is called:

```text
Projection pruning
```

It reduces unnecessary data processing.

---

# 33. Why SELECT * Can Be Problematic

Example:

```sql
SELECT *
FROM candidates;
```

Potential problems:

```text
Transfers unnecessary columns

Increases network payload

May prevent covering-index opportunities

Creates tighter coupling to schema

Processes larger rows
```

Use only required columns where practical.

---

# 34. Join Processing

A join combines rows from multiple relations according to a condition.

Example:

```sql
SELECT c.name, i.score
FROM candidates AS c
JOIN interviews AS i
    ON i.candidate_id = c.candidate_id;
```

The DBMS must decide how to execute this join efficiently.

Common algorithms:

```text
Nested Loop Join

Hash Join

Sort-Merge Join
```

---

# 35. Nested Loop Join

Conceptually:

```text
For each row in A:
    Find matching rows in B
```

Naive form:

```text
A row 1
   ↓
scan B

A row 2
   ↓
scan B

...
```

A naive nested loop can be expensive for two large unindexed relations.

But nested loops can be extremely efficient in the right conditions.

---

# 36. Indexed Nested Loop Join

Suppose:

```text
A contains 10 rows
```

and B has an index on the join key.

Then:

```text
For each A row
    ↓
Index lookup in B
```

can be efficient.

Nested loop joins are often useful when:

```text
Outer input is small

Inner lookup is indexed

Join is selective
```

---

# 37. Hash Join

A hash join typically works by building a hash table from one input and probing it using rows from the other input.

Conceptually:

```text
Smaller Input
     ↓
Build Hash Table
     ↓
Larger Input
     ↓
Probe Hash Table
     ↓
Matches
```

Hash joins are commonly effective for equality joins.

---

# 38. Hash Join Example

Join condition:

```sql
i.candidate_id = c.candidate_id
```

Possible process:

```text
Build hash table from Candidate IDs
        ↓
Scan Interview
        ↓
Hash each candidate_id
        ↓
Find matching candidate
```

Actual build/probe choices depend on the optimizer.

---

# 39. Hash Join Trade-Offs

Hash joins can be efficient for large equality joins but may require substantial memory.

If the hash structure does not fit in available memory, the database may need:

```text
Partitioning

Disk spill
```

which can make execution slower.

---

# 40. Sort-Merge Join

A merge join operates on inputs ordered by the join key.

Conceptually:

```text
Sort A by key

Sort B by key

Walk through both ordered inputs
```

Example:

```text
A: 1 2 4 7

B: 1 3 4 5

Matches:
1
4
```

---

# 41. When Merge Join Can Be Useful

Merge joins can be attractive when:

```text
Inputs are already ordered

Indexes provide useful ordering

Large inputs are joined

Join condition supports merge processing
```

If sorting is required first, sort cost must be considered.

---

# 42. Join Algorithm Comparison

### Nested Loop

Often effective when:

```text
One side is small

Indexed lookup exists
```

### Hash Join

Often effective for:

```text
Large equality joins
```

### Merge Join

Often effective when:

```text
Inputs are ordered

Large ordered datasets are involved
```

These are heuristics, not fixed rules.

---

# 43. Join Order

Suppose:

```text
Candidate

Interview

QuestionResponse

Evaluation
```

are joined.

The optimizer must decide which joins to perform first.

Starting with a highly selective filter may dramatically reduce intermediate rows.

Join order can significantly affect performance.

---

# 44. Intermediate Results

Imagine:

```text
Table A = 10 million rows

Table B = 5 million rows

Table C = 100 rows
```

If C contains a highly selective filter, joining C earlier may reduce the amount of data processed later.

Optimizers attempt to minimize expensive intermediate results.

---

# 45. Join Explosion

Suppose a query accidentally joins:

```text
Candidate

Interview

Question

Skill
```

without proper relationship conditions.

Result cardinality may explode.

Example:

```text
10 candidates
×
20 interviews
×
30 questions
×
15 skills
```

can produce huge intermediate results.

Incorrect joins are both:

```text
Correctness bugs

Performance bugs
```

---

# 46. Cartesian Product

A Cartesian product pairs every row from one relation with every row from another.

Example:

```sql
SELECT *
FROM candidates
CROSS JOIN roles;
```

If:

```text
1,000 candidates

100 roles
```

result:

```text
100,000 rows
```

Sometimes this is intentional.

Often an accidental Cartesian product indicates a missing join condition.

---

# 47. Aggregation

Query:

```sql
SELECT candidate_id, AVG(score)
FROM interviews
GROUP BY candidate_id;
```

The database must:

```text
Read rows

Group by candidate

Compute average
```

Possible implementation strategies include:

```text
Hash aggregation

Sort-based aggregation
```

depending on the DBMS and plan.

---

# 48. Hash Aggregation

Conceptually:

```text
candidate_id
     ↓
Hash table
     ↓
running sum + count
```

For each row:

```text
Update aggregate state
```

At the end:

```text
Produce result per group
```

Memory requirements depend on the number and size of groups.

---

# 49. Sort-Based Aggregation

Another strategy:

```text
Sort rows by candidate_id
        ↓
Rows with same candidate become adjacent
        ↓
Compute aggregate
```

This can be useful when input is already sorted or sorting is beneficial for other plan operations.

---

# 50. Sorting

Query:

```sql
SELECT *
FROM interviews
ORDER BY created_at DESC;
```

Without useful ordering from an index, the DBMS may need to sort rows.

Sorting large datasets can require:

```text
CPU

Memory

Temporary disk space
```

---

# 51. Index and ORDER BY

Index:

```text
(candidate_id, created_at)
```

Query:

```sql
SELECT *
FROM interviews
WHERE candidate_id = ?
ORDER BY created_at;
```

may be able to obtain rows in useful order directly from the index.

This can reduce or eliminate a separate sort operation.

---

# 52. LIMIT

Query:

```sql
SELECT *
FROM interviews
ORDER BY created_at DESC
LIMIT 10;
```

with an appropriate index may allow the DBMS to retrieve a small number of rows efficiently.

Without an appropriate access path, the DBMS may still need to process or sort a large number of rows before returning 10.

---

# 53. OFFSET Pagination

Example:

```sql
SELECT *
FROM interviews
ORDER BY created_at DESC
LIMIT 20 OFFSET 100000;
```

The DBMS may still need to traverse or process many rows before reaching the requested page.

Large offsets can become expensive.

---

# 54. Keyset Pagination

Instead of:

```sql
OFFSET 100000
```

use the last seen ordering key.

Example:

```sql
SELECT *
FROM interviews
WHERE created_at < ?
ORDER BY created_at DESC
LIMIT 20;
```

For stable ordering, a tie-breaker may be required:

```text
(created_at, interview_id)
```

---

# 55. Stable Keyset Pagination

Example:

```sql
SELECT *
FROM interviews
WHERE
    (created_at, interview_id) < (?, ?)
ORDER BY created_at DESC, interview_id DESC
LIMIT 20;
```

Support for row-value comparisons varies by DBMS.

Equivalent boolean conditions can also be used.

The key idea is:

```text
Continue from the last seen key
```

rather than skipping a huge number of rows.

---

# 56. Subqueries

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

A subquery is not automatically slow.

Modern optimizers may transform it into an efficient form such as:

```text
Semi join
```

depending on semantics and DBMS capabilities.

---

# 57. EXISTS

Example:

```sql
SELECT *
FROM candidates AS c
WHERE EXISTS (
    SELECT 1
    FROM interviews AS i
    WHERE i.candidate_id = c.candidate_id
      AND i.score >= 90
);
```

`EXISTS` expresses:

```text
Return candidate when at least one matching interview exists
```

This can communicate intent clearly.

Do not memorize:

```text
EXISTS is always faster than IN
```

That claim is not universally correct.

---

# 58. Correlated Subquery

Example:

```sql
SELECT c.name,
       (
           SELECT COUNT(*)
           FROM interviews AS i
           WHERE i.candidate_id = c.candidate_id
       ) AS interview_count
FROM candidates AS c;
```

The subquery references the outer row.

Depending on the optimizer, this may be transformed efficiently or may result in repeated work.

Inspect the execution plan.

---

# 59. N+1 Query Problem

Application code:

```text
Query all candidates

Then for each candidate:
    query interviews
```

For:

```text
1 candidate query
+
N interview queries
```

you get:

```text
N + 1 queries
```

This can cause severe application-level latency.

---

# 60. N+1 Example

Suppose:

```text
1 query = 5 ms
```

and:

```text
500 candidates
```

Naively:

```text
1 + 500 queries
```

Even before considering server load, network and connection overhead can make this much slower than a well-designed batched or joined approach.

---

# 61. Fixing N+1

Possible solutions:

```text
JOIN

Batch query

IN query

ORM eager loading

DataLoader-style batching

Precomputed relationship
```

The correct approach depends on:

```text
Result size

Data shape

Caching

ORM behavior
```

---

# 62. Query Optimization Starts With Measurement

Do not optimize based on guesses such as:

```text
Joins are slow

Subqueries are slow

CTEs are slow

Indexes make everything fast
```

Instead:

```text
Measure query latency

Inspect execution plan

Compare estimated vs actual rows

Find expensive operators

Identify I/O or CPU bottleneck

Change one thing

Measure again
```

---

# 63. EXPLAIN

Many databases provide an:

```sql
EXPLAIN
```

command.

It shows the planned execution strategy.

Depending on the DBMS, output may include:

```text
Scan type

Join type

Estimated rows

Estimated cost

Sort operations

Index usage
```

---

# 64. EXPLAIN ANALYZE

Many systems also provide functionality similar to:

```sql
EXPLAIN ANALYZE
```

which executes the query and reports actual runtime information.

Possible information:

```text
Actual rows

Actual time

Loops

Memory

Buffers

Disk spill
```

Exact syntax and metrics depend on the DBMS.

Be careful with write queries because analysis modes may actually execute them.

---

# 65. Estimated vs Actual Rows

Suppose:

```text
Estimated rows: 10

Actual rows: 1,000,000
```

This is a major estimation error.

The optimizer may have selected a plan appropriate for:

```text
10 rows
```

but poor for:

```text
1,000,000 rows
```

Large estimate errors are important clues during plan analysis.

---

# 66. What to Inspect in a Query Plan

Look for:

```text
Unexpected sequential scans

Large estimated/actual row mismatch

Repeated loops

Large sorts

Disk spills

Huge intermediate results

Missing index usage

Expensive join operators

Rows filtered after expensive work
```

Do not focus only on the top-level total cost.

---

# 67. Filter Early

Suppose only completed interviews are required.

Better logical approach:

```text
Filter completed interviews
        ↓
Join required candidate data
```

rather than:

```text
Join all interviews
        ↓
Process huge intermediate result
        ↓
Filter completed
```

Optimizers often perform predicate pushdown automatically, but query structure and semantics still matter.

---

# 68. Fetch Only Required Rows

Bad:

```sql
SELECT *
FROM interviews;
```

then application:

```text
Keep only completed interviews
```

Better:

```sql
SELECT interview_id, candidate_id, score
FROM interviews
WHERE status = 'completed';
```

Do filtering close to the data when practical.

---

# 69. Fetch Only Required Columns

Suppose Interview contains:

```text
video_url

transcript

raw_audio_metadata

large_json_report
```

but dashboard needs only:

```text
interview_id

score

created_at
```

Do not fetch large columns unnecessarily.

This reduces:

```text
Database I/O

Network transfer

Application memory
```

---

# 70. Large TEXT / JSON Columns

Large values can make:

```text
SELECT *
```

particularly expensive.

For list pages, fetch summary fields.

Retrieve heavy payloads only when required.

Schema design and query design should work together.

---

# 71. DISTINCT

Example:

```sql
SELECT DISTINCT candidate_id
FROM interviews;
```

`DISTINCT` may require:

```text
Sorting

Hashing

Deduplication
```

Do not add DISTINCT merely to hide duplicate rows caused by an incorrect join.

Fix the relationship logic first.

---

# 72. UNION vs UNION ALL

`UNION` removes duplicates.

`UNION ALL` retains them.

Therefore `UNION` may require additional deduplication work.

Use:

```text
UNION ALL
```

when duplicate removal is not required by the semantics.

---

# 73. OR Conditions

Query:

```sql
WHERE email = ?
   OR phone = ?
```

may be harder to optimize than a simple indexed equality depending on indexes and DBMS.

Possible plans include:

```text
Bitmap operations

Multiple index scans

Sequential scan
```

Do not automatically rewrite OR without examining the plan.

---

# 74. LIKE Queries

Index usefulness depends on the search pattern and DBMS.

Example:

```sql
WHERE name LIKE 'Alex%'
```

may use a suitable index in some systems.

But:

```sql
WHERE name LIKE '%Alex%'
```

cannot generally use a normal B-tree index for a direct prefix lookup.

Specialized indexes or full-text search may be appropriate.

---

# 75. Full-Text Search

Searching:

```text
Millions of interview transcripts
```

with:

```sql
LIKE '%distributed systems%'
```

may not scale well.

Depending on requirements, consider:

```text
Database full-text search

Inverted indexes

Dedicated search systems
```

Choose technology based on actual search semantics.

---

# 76. Function on Indexed Column

Potential issue:

```sql
WHERE YEAR(created_at) = 2026
```

A range predicate can often be more index-friendly:

```sql
WHERE created_at >= '2026-01-01'
  AND created_at <  '2027-01-01'
```

Again, exact timestamp semantics and DBMS behavior matter.

---

# 77. ORDER BY Function

Example:

```sql
ORDER BY LOWER(name)
```

A normal index on:

```text
name
```

may not directly provide the requested ordering.

An expression index may help if the DBMS supports it and the query is common enough to justify the index.

---

# 78. Indexing Foreign Keys

Suppose:

```text
Interview.candidate_id
```

is frequently used for:

```text
JOIN

WHERE

DELETE/UPDATE referential checks
```

An index may be useful.

But whether foreign-key columns are automatically indexed depends on the DBMS.

Always verify rather than assume.

---

# 79. Index Selectivity Trap

Suppose:

```text
status
```

contains only:

```text
active

inactive
```

and:

```text
99% = active
```

A normal index on status alone may provide little benefit for:

```sql
WHERE status = 'active';
```

But it could still be useful as part of:

```text
(status, created_at)
```

for a specific workload.

Index design is query-driven.

---

# 80. Partial Index

Some databases support indexes containing only rows matching a condition.

Example concept:

```sql
CREATE INDEX ...
ON interviews(created_at)
WHERE status = 'pending';
```

If:

```text
pending interviews
```

are a small subset frequently queried, a partial index may be efficient.

Support and syntax vary by DBMS.

---

# 81. Expression Index

Some databases allow indexing an expression.

Example concept:

```text
INDEX(LOWER(email))
```

Then query:

```sql
WHERE LOWER(email) = ?
```

may use the expression index.

Use such indexes only when they match actual query patterns.

---

# 82. Query Plan Can Change

The same SQL query may use different plans over time because:

```text
Data volume changes

Distribution changes

Statistics change

Indexes change

Parameters change

DBMS version changes
```

A query that performs well with:

```text
10,000 rows
```

may behave differently with:

```text
100 million rows
```

---

# 83. Parameter Sensitivity

Suppose:

```sql
WHERE country = ?
```

For:

```text
country = rare value
```

an index may be excellent.

For:

```text
country = extremely common value
```

a scan may be better.

Some DBMSs must balance plan reuse with parameter-specific selectivity.

This can lead to parameter-sensitive plan issues.

---

# 84. Memory and Query Performance

Operators such as:

```text
Sort

Hash join

Hash aggregate
```

may need working memory.

Insufficient memory can cause:

```text
Disk spill
```

which can dramatically increase execution time.

But increasing memory globally without understanding concurrency can also be dangerous.

---

# 85. Parallel Query Execution

Some databases can execute parts of a query using multiple workers.

Potentially parallel operations include:

```text
Scans

Aggregations

Joins
```

Parallelism can reduce latency for expensive queries but adds coordination overhead.

Small queries may not benefit.

---

# 86. Query Optimization vs Schema Optimization

Suppose a query is slow because:

```text
candidate_email
```

is stored inside:

```text
large unstructured JSON
```

and frequently searched.

No clever SQL rewrite may fully compensate for poor modeling.

Performance optimization may require:

```text
Schema changes

Indexes

Materialized structures

Partitioning
```

Query optimization and schema design are connected.

---

# 87. Materialized View

A materialized view stores the result of a query physically.

Example:

```text
Candidate Interview Summary
```

with:

```text
candidate_id

total_interviews

average_score

last_interview_at
```

Instead of recomputing expensive aggregates repeatedly, the database can read precomputed results.

---

# 88. Materialized View Trade-Off

Benefits:

```text
Faster reads

Precomputed joins/aggregates
```

Costs:

```text
Storage

Refresh complexity

Potentially stale data
```

Use when the workload justifies it.

---

# 89. Partitioning

Large tables can sometimes be divided into partitions.

Example:

```text
Interview
```

partitioned by:

```text
created_at
```

or another appropriate key.

A query targeting a specific partition range may avoid scanning unrelated partitions.

This is often called:

```text
Partition pruning
```

---

# 90. Partitioning Is Not a Replacement for Indexing

Partitioning solves different problems.

A badly indexed query inside one large partition can still be slow.

Partitioning can help with:

```text
Data management

Pruning

Maintenance

Very large datasets
```

but should not be applied blindly.

---

# 91. Caching vs Query Optimization

Suppose a query takes:

```text
3 seconds
```

and the application caches its result.

Users may see fast responses after the first request.

But the underlying query remains expensive.

Caching can be useful, but it should not automatically replace understanding the database bottleneck.

---

# 92. Application Latency

Database latency includes more than query execution.

Conceptually:

```text
Application
    ↓
Acquire connection
    ↓
Network
    ↓
Database execution
    ↓
Network
    ↓
Deserialize
    ↓
Application processing
```

A query executing in:

```text
20 ms
```

can still produce:

```text
500 ms
```

application latency because of surrounding overhead.

---

# 93. Connection Pooling

Opening a new database connection for every request can be expensive.

Applications commonly use:

```text
Connection pools
```

which maintain reusable database connections.

But oversized pools can overload the database.

Pool size should reflect:

```text
Database capacity

Application concurrency

Query duration

Number of application instances
```

---

# 94. Long-Running Transactions

A slow query inside a long transaction may cause more than latency.

It can contribute to:

```text
Locks

Blocked transactions

MVCC cleanup pressure

Connection exhaustion
```

Query optimization can therefore improve concurrency as well as response time.

---

# 95. Lock Wait vs Slow Query

Suppose a query takes:

```text
10 seconds
```

It may not be executing for 10 seconds.

It could spend:

```text
9.8 seconds waiting for a lock

0.2 seconds executing
```

Diagnose:

```text
Execution time

Wait time

Locking

I/O

CPU
```

separately.

---

# 96. Common Performance Workflow

When a database endpoint is slow:

```text
1. Measure total endpoint latency

2. Measure database query latency

3. Identify slow query

4. Inspect execution plan

5. Compare estimated vs actual rows

6. Inspect scans and joins

7. Check indexes

8. Check statistics

9. Check lock waits

10. Check rows/columns returned

11. Make targeted change

12. Benchmark again
```

Do not begin by randomly adding indexes.

---

# 97. Common Interview Question: What Is Query Optimization?

Strong answer:

> Query optimization is the process by which a DBMS evaluates alternative execution strategies for a SQL query and selects a physical plan expected to execute efficiently based on available indexes, statistics, estimated cardinalities, operator costs, and other engine-specific factors.

---

# 98. Common Interview Question: What Is an Execution Plan?

Strong answer:

> An execution plan is the DBMS's physical strategy for executing a query. It describes operations such as scans, joins, sorts, and aggregations, along with their ordering and often estimated cost and cardinality.

---

# 99. Common Interview Question: Logical vs Physical Plan?

### Logical Plan

Describes relational operations:

```text
Filter

Join

Project

Aggregate
```

### Physical Plan

Chooses implementation:

```text
Index scan

Hash join

Merge join

Hash aggregate
```

The logical plan describes what operations are required.

The physical plan describes how they will be performed.

---

# 100. Common Interview Question: What Is Selectivity?

Answer:

> Selectivity describes how strongly a predicate narrows the rows. Highly selective predicates return a small fraction of rows, while low-selectivity predicates match a large fraction.

This affects index usefulness.

---

# 101. Common Interview Question: What Is Cardinality Estimation?

Answer:

> Cardinality estimation predicts the number of rows produced by scans, filters, joins, and other operators. The optimizer uses these estimates to compare plans and choose join order, access paths, algorithms, and resource allocations.

---

# 102. Common Interview Question: Why Might an Index Not Be Used?

Possible reasons:

```text
Predicate matches too many rows

Table is small

Function applied to indexed column

Implicit conversion

Wrong composite-index order

Statistics suggest scan is cheaper

Required columns make lookup expensive

Different index is better
```

An unused index does not automatically mean the optimizer is wrong.

---

# 103. Common Interview Question: Nested Loop vs Hash Join?

### Nested Loop

Often suitable when:

```text
Outer input is small

Inner side has efficient lookup
```

### Hash Join

Often suitable when:

```text
Large inputs

Equality join
```

Actual choice depends on cost estimates and DBMS capabilities.

---

# 104. Common Interview Question: Hash Join vs Merge Join?

### Hash Join

Builds a hash structure and probes it.

Usually associated with equality joins.

### Merge Join

Processes ordered inputs together.

Can be efficient when ordering already exists or can be obtained cheaply.

---

# 105. Common Interview Question: What Is Sargability?

Answer:

> Sargability describes whether a predicate can be used efficiently as an index search condition. Applying transformations or functions to an indexed column can sometimes prevent direct index lookup unless an appropriate expression index or equivalent mechanism exists.

---

# 106. Common Interview Question: Why Is SELECT * Bad?

Answer:

> SELECT * can fetch unnecessary data, increase I/O and network transfer, make index-only execution less likely, and couple application code to the complete table schema. It is not inherently invalid, but production queries should generally retrieve only the columns they require.

---

# 107. Common Interview Question: Why Can OFFSET Be Slow?

Answer:

> Large OFFSET values may require the database to traverse or process many preceding rows before returning the requested page. Keyset pagination instead continues from the last ordering key and can scale better for large ordered datasets.

---

# 108. Common Interview Question: What Is N+1?

Answer:

> The N+1 problem occurs when an application fetches a collection with one query and then issues another query for each returned row, producing one initial query plus N additional queries. It increases database round trips and can cause severe latency.

---

# 109. Common Interview Question: Is a Full Table Scan Bad?

Correct answer:

> Not necessarily. A sequential scan can be optimal when the table is small or a query needs a large fraction of its rows. The execution plan and workload determine whether the scan is problematic.

---

# 110. Common Interview Question: Does an Index Always Improve Performance?

No.

Indexes improve certain reads but add:

```text
Storage

Write overhead

Maintenance cost
```

An index is useful only when it supports actual access patterns enough to justify those costs.

---

# 111. Common Interview Question: How Would You Optimize a Slow Query?

A strong answer:

> I would first measure the query and inspect its actual execution plan. I would compare estimated and actual cardinalities, identify expensive scans, joins, sorts, spills, or repeated loops, verify indexes and statistics, reduce unnecessary rows and columns, and then benchmark a targeted change. I would not start by blindly adding indexes.

---

# 112. Common Interview Questions

Be prepared to answer:

1. What is query processing?
2. What happens after SQL reaches the DBMS?
3. What is parsing?
4. What is semantic analysis?
5. What is query rewriting?
6. What is a logical plan?
7. What is a physical plan?
8. What is a query optimizer?
9. What is cost-based optimization?
10. What are database statistics?
11. What is cardinality?
12. What is cardinality estimation?
13. What is selectivity?
14. Sequential scan vs index scan?
15. When is a sequential scan good?
16. What is an index-only scan?
17. What is a covering index?
18. What is a composite index?
19. Why does index column order matter?
20. What is sargability?
21. What is predicate pushdown?
22. What is projection pruning?
23. What are common join algorithms?
24. How does nested loop join work?
25. How does hash join work?
26. How does merge join work?
27. Why does join order matter?
28. What is a Cartesian product?
29. What is hash aggregation?
30. How can ORDER BY become expensive?
31. How can an index help ORDER BY?
32. Why can OFFSET pagination be slow?
33. What is keyset pagination?
34. Are subqueries always slow?
35. EXISTS vs IN?
36. What is a correlated subquery?
37. What is the N+1 problem?
38. What does EXPLAIN show?
39. What does EXPLAIN ANALYZE show?
40. Estimated rows vs actual rows?
41. Why might an index not be used?
42. What is a partial index?
43. What is an expression index?
44. What is partition pruning?
45. What is a materialized view?
46. How would you debug a slow database endpoint?
47. How can lock waits look like query slowness?
48. Why is connection pooling important?
49. Why is SELECT * often undesirable?
50. How do you decide whether to add an index?

---

# 113. Common Mistakes

## Mistake 1: Full table scan always means bad query

Incorrect.

For small tables or low-selectivity queries, a scan may be optimal.

---

## Mistake 2: Index every column

Incorrect.

Indexes have write and storage costs.

---

## Mistake 3: Joins are always slow

Incorrect.

Properly designed joins with appropriate indexes and plans are fundamental relational operations.

---

## Mistake 4: Subqueries are always slower than joins

Incorrect.

Optimizers can transform many subqueries into efficient execution strategies.

---

## Mistake 5: EXISTS is always faster than IN

Incorrect.

Semantics, data distribution, null behavior, optimizer transformations, and DBMS implementation matter.

---

## Mistake 6: SELECT * has no performance impact

Incorrect.

It can increase:

```text
I/O

Network payload

Memory usage
```

and prevent some covering-index opportunities.

---

## Mistake 7: Add an index before looking at the plan

Bad debugging methodology.

Measure first.

---

## Mistake 8: Estimated cost equals milliseconds

Incorrect.

Optimizer cost is usually an internal planning metric.

---

## Mistake 9: Database query time equals endpoint latency

Incorrect.

Endpoint latency can include:

```text
Connection wait

Network

Serialization

Application processing

Lock waits
```

---

## Mistake 10: Slow query means database needs more hardware

Not necessarily.

The root cause may be:

```text
Missing index

N+1 queries

Bad join

Huge payload

Stale statistics

Lock contention

Poor schema design
```

---

# 114. Interview Scenario

Suppose this query is slow:

```sql
SELECT *
FROM interviews
WHERE candidate_id = 101
ORDER BY created_at DESC
LIMIT 20;
```

Table:

```text
100 million rows
```

Current index:

```text
INDEX(candidate_id)
```

Possible improvement:

```text
INDEX(candidate_id, created_at)
```

Why?

The database needs:

```text
Rows for one candidate

ordered by created_at
```

A composite index matching that access pattern may reduce both filtering and sorting work.

But verify using:

```text
EXPLAIN / actual execution plan
```

before and after the change.

---

# 115. Interview Scenario: Function on Column

Query:

```sql
SELECT *
FROM interviews
WHERE DATE(created_at) = '2026-07-26';
```

Index:

```text
INDEX(created_at)
```

Potential issue:

```text
Function applied to indexed column
```

Potential rewrite:

```sql
SELECT *
FROM interviews
WHERE created_at >= '2026-07-26 00:00:00'
  AND created_at <  '2026-07-27 00:00:00';
```

This may allow an efficient range scan.

Always account for timezone semantics.

---

# 116. Interview Scenario: N+1

Application:

```text
SELECT all candidates
```

Then:

```text
for each candidate:
    SELECT interviews
```

Suppose:

```text
1,000 candidates
```

Total:

```text
1,001 queries
```

Possible fix:

```text
Batch interviews by candidate IDs
```

or:

```text
Join / eager load
```

depending on required result shape.

The key issue is excessive round trips.

---

# 117. Interview Scenario: Wrong Index

Query:

```sql
SELECT interview_id, created_at
FROM interviews
WHERE candidate_id = ?
  AND status = 'completed'
ORDER BY created_at DESC
LIMIT 10;
```

Indexes:

```text
INDEX(status)

INDEX(created_at)
```

Neither necessarily matches the complete access pattern well.

A possible composite index might begin with:

```text
candidate_id
```

and include columns based on:

```text
Filtering

Ordering

Selectivity
```

For example:

```text
(candidate_id, status, created_at)
```

could be evaluated.

But index design should be confirmed using the real workload and execution plan.

---

# 118. Query Optimization Framework

When asked:

> How would you optimize this query?

Use this structure:

```text
1. Understand expected result

2. Check table sizes

3. Inspect execution plan

4. Compare estimates vs actuals

5. Identify scan/filter behavior

6. Inspect join order and algorithms

7. Check existing indexes

8. Evaluate predicate selectivity

9. Check sargability

10. Reduce unnecessary columns

11. Reduce unnecessary rows

12. Check sorting/aggregation

13. Check N+1 at application layer

14. Check lock/wait behavior

15. Make targeted change

16. Benchmark again
```

This is stronger than immediately saying:

```text
Add an index.
```

---

# 119. Quick Revision

Remember:

```text
SQL
 ↓
Parser
 ↓
Analyzer
 ↓
Rewriter
 ↓
Optimizer
 ↓
Physical Plan
 ↓
Executor
```

Optimizer considers:

```text
Statistics

Cardinality

Selectivity

Indexes

Join order

Join algorithms

I/O

CPU

Memory
```

Common scans:

```text
Sequential Scan

Index Scan

Index-Only Scan
```

Common joins:

```text
Nested Loop

Hash Join

Merge Join
```

Common optimization ideas:

```text
Filter early

Fetch fewer columns

Use appropriate indexes

Keep predicates sargable

Avoid N+1

Use scalable pagination

Inspect actual plans

Measure before optimizing
```

---

# 120. Final Takeaway

Query optimization is not:

```text
Slow query
    ↓
Add random index
```

It is:

```text
Slow operation
    ↓
Measure
    ↓
Identify database contribution
    ↓
Inspect execution plan
    ↓
Understand cardinality and selectivity
    ↓
Find expensive operator
    ↓
Change query / index / schema
    ↓
Benchmark
```

The strongest interview candidate understands that:

```text
SQL specifies what

Execution plan specifies how
```

and can explain why the optimizer may choose:

```text
Sequential scan instead of index

Hash join instead of nested loop

Composite index instead of multiple unrelated indexes

Keyset pagination instead of huge OFFSET
```

Most importantly:

> Query optimization is evidence-driven engineering. Measure the real bottleneck, understand the execution plan, and optimize the operation that is actually expensive.

---

## Previous

**[← ER Model and Schema Design](./10-er-model-and-schema-design.md)**

## Next

**[DBMS Interview Revision →](./12-dbms-interview-revision.md)**

---

Maintained by **InterviewEraHQ**.
