# Indexing in DBMS

Indexes are one of the most important database performance concepts in backend and database interviews.

A weak explanation is:

> An index makes queries faster.

That is incomplete.

A strong candidate should understand:

- What problem an index solves
- How indexes reduce data access
- Why B-trees are widely used
- How composite indexes work
- Why column order matters
- What selectivity means
- What covering indexes are
- Why indexes can make writes more expensive
- Why an index may exist but still not be used
- How query execution plans help diagnose performance

The central idea is:

> An index is an additional data structure that trades storage and write overhead for potentially faster access to data.

---

# 1. Why Do We Need Indexes?

Suppose an `interviews` table contains:

```text
10,000,000 rows
```

and we run:

```sql
SELECT *
FROM interviews
WHERE candidate_id = 12345;
```

Without a useful access path, the database may need to examine a large portion of the table to find matching rows.

Conceptually:

```text
Row 1
Row 2
Row 3
...
Row 9,999,999
Row 10,000,000
```

This can be expensive as the dataset grows.

An index can provide a more efficient way to locate qualifying rows.

---

# 2. What Is an Index?

An **index** is an auxiliary data structure maintained by a database to help locate data efficiently for certain operations.

For example:

```sql
CREATE INDEX idx_interviews_candidate_id
ON interviews(candidate_id);
```

The database now has an index associated with:

```text
candidate_id
```

A query such as:

```sql
SELECT *
FROM interviews
WHERE candidate_id = 12345;
```

may use that index instead of scanning the entire table.

The keyword is:

```text
may
```

Creating an index does not guarantee the optimizer will use it.

---

# 3. Book Index Analogy

Imagine a 1,000-page textbook.

You want to find:

```text
Database Normalization
```

Without an index, you might inspect pages sequentially.

With a book index:

```text
Database Normalization → 342
```

you can navigate much closer to the desired location.

A database index serves a similar conceptual purpose.

However, database indexes are dynamic data structures that must also be maintained as data changes.

---

# 4. Table Scan

A **table scan** or equivalent full scan means the database examines the table broadly to determine which rows satisfy the query.

Example:

```sql
SELECT *
FROM interviews
WHERE candidate_id = 12345;
```

Conceptually:

```text
Read row
   ↓
Check candidate_id
   ↓
Match?
   ↓
Continue
```

A scan is not automatically bad.

For a small table, or when a query needs a large percentage of the rows, scanning may be cheaper than using an index.

---

# 5. Index Lookup

With an appropriate index:

```text
Index
  ↓
Find candidate_id = 12345
  ↓
Locate matching entries
  ↓
Retrieve required data
```

This can dramatically reduce the amount of data that must be examined.

But performance depends on:

- Index structure
- Query
- Selectivity
- Data distribution
- Table size
- Statistics
- Storage
- Database optimizer

---

# 6. Basic Index Creation

Example:

```sql
CREATE INDEX idx_candidates_email
ON candidates(email);
```

A query:

```sql
SELECT *
FROM candidates
WHERE email = 'alex@example.com';
```

may benefit from this index.

---

# 7. UNIQUE Index

Some database systems implement uniqueness constraints using unique indexes internally.

Example:

```sql
CREATE UNIQUE INDEX idx_candidates_email
ON candidates(email);
```

This can enforce uniqueness while also providing an indexed access path.

However, the relationship between:

```text
UNIQUE constraint
```

and:

```text
unique index
```

is DBMS-specific.

At the schema level, use constraints to express data integrity requirements rather than assuming indexes and constraints are universally interchangeable.

---

# 8. Common Index Data Structures

Database systems can support multiple index types.

Common examples include:

```text
B-tree
Hash
Bitmap
Full-text
Spatial
Specialized inverted indexes
```

Availability and behavior depend on the database system.

For general-purpose relational database interviews, **B-tree indexes** are especially important.

---

# 9. B-Tree Index

B-tree family structures organize sorted keys in a balanced tree structure.

A simplified representation:

```text
                 [40]
               /      \
           [20]        [60]
          /   \        /   \
       [10]   [30]  [50]   [70]
```

Real database B-trees are more complex and typically have many keys and child pointers per page.

The important property is that the tree remains balanced.

This enables efficient navigation through large indexed datasets.

---

# 10. Why B-Trees Are Useful

B-tree indexes can support operations such as:

```text
Equality search
Range search
Ordered traversal
Prefix-based access in composite indexes
```

Example:

```sql
SELECT *
FROM interviews
WHERE score >= 80
  AND score <= 90;
```

An appropriate B-tree index on:

```text
score
```

can potentially support the range lookup.

---

# 11. Why Not Use a Hash Table for Everything?

Hash-based indexes can be effective for equality-style lookups in systems that support them.

Conceptually:

```text
key
 ↓
hash(key)
 ↓
bucket
```

But ordered range queries such as:

```sql
WHERE score BETWEEN 80 AND 90
```

require order-aware access.

A B-tree maintains key ordering, making it suitable for many equality, range, and ordering workloads.

Exact index capabilities depend on the DBMS.

---

# 12. Single-Column Index

Example:

```sql
CREATE INDEX idx_interviews_candidate_id
ON interviews(candidate_id);
```

This index is primarily organized around:

```text
candidate_id
```

Potentially useful queries include:

```sql
SELECT *
FROM interviews
WHERE candidate_id = 100;
```

But simply having the column somewhere in a query does not guarantee the index will help.

---

# 13. Composite Index

A **composite index** contains multiple columns.

Example:

```sql
CREATE INDEX idx_interviews_candidate_status
ON interviews(candidate_id, status);
```

The index is ordered first by:

```text
candidate_id
```

and then, within matching candidate IDs, by:

```text
status
```

Conceptually:

```text
candidate_id
     ↓
   status
```

Column order matters.

---

# 14. Composite Index Ordering

Suppose we create:

```sql
CREATE INDEX idx_interviews_candidate_status
ON interviews(candidate_id, status);
```

This can often help:

```sql
WHERE candidate_id = 10
```

and:

```sql
WHERE candidate_id = 10
  AND status = 'completed'
```

But a query using only:

```sql
WHERE status = 'completed'
```

may not be able to use this index as effectively for direct lookup because the leading indexed column is not constrained.

This is commonly explained using the **leftmost-prefix principle**.

---

# 15. Leftmost-Prefix Principle

For an index:

```text
(A, B, C)
```

the ordered prefixes are:

```text
A
A, B
A, B, C
```

Queries using those leading portions can often benefit from the index.

For example:

```sql
WHERE A = ?
```

```sql
WHERE A = ?
  AND B = ?
```

```sql
WHERE A = ?
  AND B = ?
  AND C = ?
```

A query using only:

```sql
WHERE B = ?
```

does not have the leading column `A`.

Therefore the index may be much less useful for direct lookup.

---

# 16. Leftmost Prefix Is Not an Absolute "Cannot Use" Rule

Avoid saying:

> An `(A, B)` index can never be used for B alone.

That is too absolute.

A database optimizer may still choose to scan the index, use specialized access techniques, or exploit it for other reasons.

The better statement is:

> Composite B-tree indexes are primarily organized by their leading columns, so queries that do not constrain the leading columns generally cannot exploit the index as effectively for direct search.

Actual behavior depends on the DBMS and execution plan.

---

# 17. Column Order Matters

Suppose your common query is:

```sql
SELECT *
FROM interviews
WHERE candidate_id = 100
  AND status = 'completed';
```

Possible indexes include:

```text
(candidate_id, status)
```

and:

```text
(status, candidate_id)
```

Which one is better?

There is no universal answer.

Consider:

- Query patterns
- Equality predicates
- Range predicates
- Selectivity
- Sorting requirements
- Other queries using the index
- Data distribution

Index design should follow workload requirements, not arbitrary rules.

---

# 18. Equality and Range Predicates

Consider:

```sql
SELECT *
FROM interviews
WHERE candidate_id = 100
  AND created_at >= '2026-01-01';
```

An index such as:

```sql
CREATE INDEX idx_interviews_candidate_created
ON interviews(candidate_id, created_at);
```

can be useful because the database can conceptually locate:

```text
candidate_id = 100
```

and then inspect the relevant:

```text
created_at
```

range within that portion of the index.

---

# 19. Range Conditions and Later Columns

Suppose the index is:

```text
(A, B, C)
```

and the query is:

```sql
WHERE A = ?
  AND B > ?
  AND C = ?
```

Once a range condition is used on `B`, the ability to use the ordering of later column `C` for narrowing the same index range may be limited.

However, database engines can differ in how they use later columns for:

- Filtering
- Index condition evaluation
- Skip scans
- Other optimizations

Do not reduce this to:

> Columns after a range condition are completely useless.

That is too broad.

---

# 20. What Is Selectivity?

**Selectivity** describes how effectively a condition narrows the dataset.

Consider:

```text
email
```

where almost every row has a different value.

A query:

```sql
WHERE email = 'alex@example.com'
```

might match one row.

This is highly selective.

Now consider:

```text
status
```

with values:

```text
active
inactive
```

If 90% of rows are:

```text
active
```

then:

```sql
WHERE status = 'active'
```

matches a large portion of the table.

That predicate has relatively low selectivity.

---

# 21. Cardinality vs Selectivity

These terms are related but not identical.

**Cardinality** often refers to the number of distinct values or the number of rows in a relation, depending on context.

Example:

```text
email → many distinct values
status → few distinct values
```

Selectivity refers to how much a predicate reduces the candidate rows.

Do not treat the terms as universally interchangeable.

---

# 22. Are Low-Cardinality Columns Bad Indexes?

Not automatically.

An index on:

```text
status
```

might still be useful when:

- One status value is rare
- Queries frequently target that rare value
- The index is part of a composite index
- The DBMS supports index structures suited to the workload
- The index can satisfy other query requirements

Example distribution:

```text
completed → 9,900,000 rows
failed    → 100,000 rows
```

A query for:

```sql
WHERE status = 'failed'
```

could still be selective enough to benefit.

---

# 23. Covering Index

Suppose a query needs:

```sql
SELECT candidate_id, status
FROM interviews
WHERE candidate_id = 100;
```

and an index contains:

```text
(candidate_id, status)
```

The database may be able to satisfy the query using data available from the index without fetching all required values from the underlying table representation.

This concept is commonly called a **covering index**.

---

# 24. Covering Is Query-Specific

An index is not simply:

```text
covering = true
```

for every query.

Suppose the index contains:

```text
(candidate_id, status)
```

It may cover:

```sql
SELECT candidate_id, status
FROM interviews
WHERE candidate_id = 100;
```

but not:

```sql
SELECT candidate_id, status, score
FROM interviews
WHERE candidate_id = 100;
```

if `score` is unavailable from the relevant index representation.

Covering is relative to the query.

---

# 25. Index-Only Scan

Some database systems can perform an **index-only scan** when the required query data can be obtained from the index and other engine-specific conditions are satisfied.

This can reduce table access.

However:

```text
Covering index
```

and:

```text
Guaranteed index-only execution
```

are not identical concepts.

Whether an index-only strategy is possible or beneficial depends on the DBMS and runtime conditions.

---

# 26. Included Columns

Some database systems support adding non-key columns to an index.

Conceptually:

```text
Index key:
(candidate_id)

Included data:
(status, score)
```

This can help cover queries without making every included column part of the index's search ordering.

Syntax and capabilities vary by DBMS.

Do not assume every database supports identical `INCLUDE` semantics.

---

# 27. Clustered Index

The term **clustered index** is highly DBMS-specific.

Conceptually, clustering refers to storing or organizing table data in a way related to an index's key order.

This can improve locality for certain access patterns.

However, PostgreSQL, SQL Server, MySQL/InnoDB, and other systems differ substantially in how table storage and clustering work.

Avoid universal claims such as:

> Every table has exactly one clustered index.

That statement depends on the database architecture.

---

# 28. Clustered vs Non-Clustered: Conceptual Difference

A simplified interview-level distinction:

### Clustered Organization

The table's physical or primary storage organization is closely associated with the key order.

### Secondary / Non-Clustered Index

A separate index structure points to or identifies underlying rows.

But exact terminology and implementation vary.

Always mention the DBMS when discussing storage internals.

---

# 29. Primary Keys and Indexes

Many relational database systems automatically create an index or equivalent indexed structure to enforce a primary key.

Example:

```sql
CREATE TABLE candidates (
    id BIGINT PRIMARY KEY,
    name VARCHAR(100)
);
```

You usually should not immediately create another identical index on:

```text
id
```

without checking what the DBMS already created.

Otherwise, you may create redundant indexes.

---

# 30. Foreign Keys and Indexes

A common misconception is:

> Creating a foreign key automatically creates an index on the foreign key column.

This is not universally true.

Example:

```sql
FOREIGN KEY (candidate_id)
REFERENCES candidates(id)
```

Whether an index is automatically created on:

```text
candidate_id
```

depends on the DBMS.

Even when it is not automatically created, indexing foreign-key columns can often be useful for:

- Joins
- Parent-row updates/deletes
- Child-row lookups

But it should still be based on workload requirements.

---

# 31. Indexes Improve Reads but Cost Writes

Suppose a table has five indexes.

When inserting a row:

```sql
INSERT INTO interviews (...)
VALUES (...);
```

the database may need to update:

```text
Table data
Index 1
Index 2
Index 3
Index 4
Index 5
```

Indexes therefore introduce write overhead.

Operations affected can include:

```text
INSERT
UPDATE
DELETE
```

depending on which indexed values or rows change.

---

# 32. Why UPDATE Can Become More Expensive

Suppose we have:

```sql
CREATE INDEX idx_candidates_email
ON candidates(email);
```

and run:

```sql
UPDATE candidates
SET email = 'new@example.com'
WHERE id = 100;
```

Because the indexed value changes, the database may need to modify the relevant index entries.

Updating a non-indexed column may have different costs.

Exact behavior depends on the storage engine.

---

# 33. Index Storage Cost

Indexes consume storage.

Suppose a large table has:

```text
100 GB table data
```

Indexes may add significant additional storage depending on:

- Indexed columns
- Number of indexes
- Data types
- Index type
- Included columns
- Engine overhead

Therefore:

> More indexes are not automatically better.

---

# 34. Over-Indexing

A table with many overlapping indexes can suffer from:

```text
Higher write latency
More storage usage
More maintenance
Longer index creation/rebuild operations
Additional optimizer choices
Operational complexity
```

Index design should be deliberate.

---

# 35. Redundant Indexes

Suppose we have:

```text
INDEX (candidate_id)
```

and:

```text
INDEX (candidate_id, status)
```

Is the first index redundant?

Possibly.

The second index begins with:

```text
candidate_id
```

so it may support some queries that use only `candidate_id`.

But that does not automatically mean the single-column index should always be removed.

Consider:

- Index size
- Query frequency
- Covering requirements
- Write overhead
- DBMS behavior
- Execution plans

Evaluate actual workload before deleting indexes.

---

# 36. Why an Index May Not Be Used

Suppose an index exists:

```sql
CREATE INDEX idx_interviews_status
ON interviews(status);
```

but the database chooses a table scan for:

```sql
SELECT *
FROM interviews
WHERE status = 'completed';
```

Possible reason:

```text
Almost every row has status = 'completed'
```

Using the index could require:

```text
Read many index entries
        +
Fetch many table rows
```

A sequential scan may be cheaper.

The optimizer chooses based on estimated cost.

---

# 37. Small Tables

For a table containing only:

```text
100 rows
```

scanning the entire table can be cheaper than navigating an index and fetching rows through it.

Therefore:

> Table scan does not automatically mean a performance problem.

Context matters.

---

# 38. Functions on Indexed Columns

Suppose:

```sql
CREATE INDEX idx_candidates_email
ON candidates(email);
```

Query:

```sql
SELECT *
FROM candidates
WHERE LOWER(email) = 'alex@example.com';
```

A plain index on:

```text
email
```

may not directly support the expression:

```text
LOWER(email)
```

in some systems.

Possible solutions may include:

- Expression/function-based index
- Case-insensitive data type or collation
- Different query design

depending on the DBMS.

Example where supported:

```sql
CREATE INDEX idx_candidates_lower_email
ON candidates(LOWER(email));
```

---

# 39. SARGability

A predicate is often called **SARGable** when it can effectively use an index search argument.

Consider:

```sql
WHERE created_at >= '2026-01-01'
  AND created_at < '2027-01-01'
```

This can often use an index on:

```text
created_at
```

Compare:

```sql
WHERE YEAR(created_at) = 2026
```

Applying a function to every stored value may make direct use of a plain index more difficult, depending on the DBMS.

A range predicate can often be more index-friendly.

---

# 40. Leading Wildcard Search

Suppose:

```sql
CREATE INDEX idx_candidates_name
ON candidates(name);
```

This query:

```sql
WHERE name LIKE 'Alex%'
```

may be able to exploit ordered index access depending on collation and DBMS behavior.

But:

```sql
WHERE name LIKE '%Alex'
```

starts with an unrestricted prefix.

A normal B-tree index is generally much less useful for directly locating such matches.

Full-text or specialized indexes may be more appropriate for certain search workloads.

---

# 41. Sorting With an Index

Suppose we frequently run:

```sql
SELECT *
FROM interviews
WHERE candidate_id = 100
ORDER BY created_at DESC;
```

An index such as:

```sql
CREATE INDEX idx_interviews_candidate_created
ON interviews(candidate_id, created_at DESC);
```

may help with both:

```text
Filtering
+
Ordering
```

depending on the database and query.

This can reduce or eliminate a separate sorting step.

---

# 42. Index for GROUP BY

Indexes can sometimes help aggregation queries.

Example:

```sql
SELECT candidate_id, COUNT(*)
FROM interviews
GROUP BY candidate_id;
```

An index beginning with:

```text
candidate_id
```

may provide useful ordering or access characteristics.

However, whether it improves performance depends on the optimizer and workload.

Do not assume `GROUP BY` automatically benefits from any index on the grouped column.

---

# 43. Partial Index

Some databases support **partial indexes**, where only rows satisfying a condition are indexed.

Suppose most interviews are completed, but the application frequently queries pending interviews.

Conceptually:

```sql
CREATE INDEX idx_pending_interviews
ON interviews(candidate_id)
WHERE status = 'pending';
```

This index stores only rows satisfying:

```text
status = 'pending'
```

Potential benefits:

```text
Smaller index
Lower storage
Efficient targeted queries
```

Support and syntax are DBMS-specific.

---

# 44. Expression Index

Some systems support indexing an expression.

Example:

```sql
CREATE INDEX idx_candidates_lower_email
ON candidates(LOWER(email));
```

Then:

```sql
WHERE LOWER(email) = 'alex@example.com'
```

may benefit from that index.

Again, exact support varies by database.

---

# 45. Query Execution Plan

How do you know whether an index is actually being used?

Inspect the **query execution plan**.

Many database systems provide tools such as:

```sql
EXPLAIN
```

Example:

```sql
EXPLAIN
SELECT *
FROM interviews
WHERE candidate_id = 100;
```

The output can show how the database plans to execute the query.

---

# 46. EXPLAIN

A simplified execution plan might indicate:

```text
Index Scan
```

or:

```text
Sequential Scan
```

depending on the DBMS.

You may also see:

```text
Estimated rows
Estimated cost
Join methods
Sort operations
Filter conditions
Index conditions
```

Exact terminology varies.

---

# 47. EXPLAIN ANALYZE

Some systems support:

```sql
EXPLAIN ANALYZE
```

which executes the query and reports runtime information.

Example:

```sql
EXPLAIN ANALYZE
SELECT *
FROM interviews
WHERE candidate_id = 100;
```

This can provide information such as:

```text
Actual execution time
Actual rows
Loops
Plan operations
```

Because the query actually runs, use it carefully with operations that modify data or expensive production workloads.

---

# 48. Estimated Rows vs Actual Rows

Suppose the optimizer estimates:

```text
100 rows
```

but execution returns:

```text
1,000,000 rows
```

That large difference may indicate problems involving:

```text
Statistics
Data distribution
Correlation
Predicate estimation
```

Bad cardinality estimates can lead to poor plan choices.

---

# 49. Database Statistics

Query optimizers use statistics to estimate:

```text
Number of rows
Value distribution
Distinct values
Selectivity
```

These estimates help choose between strategies such as:

```text
Index scan
Table scan
Nested loop
Hash join
Merge join
```

An index alone does not determine the plan.

---

# 50. Index Design Starts With Queries

Do not begin with:

> Which columns should I index?

Begin with:

```text
What queries does the application run?
```

For each important query, inspect:

```text
WHERE conditions
JOIN conditions
ORDER BY
GROUP BY
Selected columns
Frequency
Rows returned
Data distribution
```

Then design indexes around the workload.

---

# 51. Example Workload

Suppose InterviewEra frequently runs:

```sql
SELECT
    id,
    score,
    created_at
FROM interviews
WHERE candidate_id = ?
  AND status = 'completed'
ORDER BY created_at DESC
LIMIT 10;
```

Potential index:

```sql
CREATE INDEX idx_interviews_candidate_status_created
ON interviews(
    candidate_id,
    status,
    created_at DESC
);
```

Why might this be useful?

The query uses:

```text
candidate_id → equality filter
status       → equality filter
created_at   → ordering
```

The index aligns with the access pattern.

But it should still be verified with an execution plan and realistic data.

---

# 52. Should We Add score to the Index?

Suppose the query selects:

```text
id
score
created_at
```

Should we create:

```text
(candidate_id, status, created_at, score)
```

Not automatically.

Adding more columns can:

```text
Increase index size
Increase write cost
Reduce cache efficiency
```

Covering a query can be useful, but every extra column has a cost.

Index design is a trade-off.

---

# 53. Indexing JOIN Columns

Consider:

```sql
SELECT
    c.name,
    i.score
FROM candidates AS c
JOIN interviews AS i
    ON i.candidate_id = c.id
WHERE c.id = 100;
```

Indexes may be relevant on:

```text
candidates.id
```

and:

```text
interviews.candidate_id
```

The primary key may already provide an index-like access path for:

```text
candidates.id
```

But:

```text
interviews.candidate_id
```

may require a separate index depending on the DBMS and workload.

---

# 54. Composite Index vs Multiple Single-Column Indexes

Suppose a query uses:

```sql
WHERE candidate_id = ?
  AND status = ?
```

Options include:

```text
INDEX(candidate_id)

INDEX(status)
```

or:

```text
INDEX(candidate_id, status)
```

Some database systems can combine multiple indexes for certain queries.

However, a well-designed composite index may provide a more direct access path.

Do not assume:

```text
INDEX(A) + INDEX(B)
```

is equivalent to:

```text
INDEX(A, B)
```

They have different structures and capabilities.

---

# 55. Index Intersection / Combination

Some optimizers can combine multiple indexes.

Conceptually:

```text
Index on A
    ↓
Matching row identifiers

Index on B
    ↓
Matching row identifiers

Combine
    ↓
Rows satisfying both
```

This can be useful, but it does not eliminate the need for thoughtful composite indexes.

Implementation differs across databases.

---

# 56. Index Maintenance

Indexes require maintenance over time.

Depending on the DBMS, operational concerns may include:

```text
Page splits
Fragmentation
Bloat
Statistics updates
Rebuilding
Reorganizing
Vacuuming
```

The exact maintenance model differs substantially between database engines.

Avoid applying maintenance advice from one DBMS blindly to another.

---

# 57. Index Fragmentation

Some database systems expose index fragmentation as an important maintenance concept.

As data changes, physical index organization can become less optimal for certain workloads.

But:

> Rebuild every index regularly.

is not good universal advice.

Measure actual database behavior and follow DBMS-specific operational practices.

---

# 58. Index Bloat

In systems such as PostgreSQL, indexes can accumulate unused space under certain workloads and MVCC behavior.

This can increase:

```text
Storage
I/O
Cache usage
```

Maintenance may involve database-specific operations.

Again, terminology and solutions depend on the engine.

---

# 59. Indexes and Caching

Frequently accessed index pages may remain in memory.

A smaller, focused index can sometimes be more cache-friendly than a very large index containing many unnecessary columns.

Therefore:

```text
Cover everything with one huge index
```

is usually not a sound indexing strategy.

---

# 60. Common Interview Question: Why Not Index Every Column?

Because indexes have costs.

Every additional index may increase:

```text
Storage
INSERT cost
UPDATE cost
DELETE cost
Maintenance
Operational complexity
```

And many indexes may never be useful for important queries.

A strong answer:

> Indexes trade additional storage and write overhead for potentially faster access. I would index based on important query patterns, selectivity, joins, sorting requirements, and measured execution plans rather than indexing every column.

---

# 61. Common Interview Question: When Might an Index Not Help?

Possible cases include:

```text
Very small table

Query returns a large percentage of rows

Low-selectivity predicate

Function or transformation prevents effective lookup

Leading wildcard search

Composite index does not align with query

Outdated or inaccurate statistics

Optimizer estimates a scan is cheaper
```

The exact reason should be verified using the execution plan.

---

# 62. Common Interview Question: What Columns Should Be Indexed?

There is no universal list.

Candidates may consider columns involved in:

```text
Frequent selective filters
JOIN conditions
Ordering
Grouping
Uniqueness constraints
Frequent lookup patterns
```

But index design must consider the complete workload.

A column being present in:

```text
WHERE
```

does not automatically mean it needs an index.

---

# 63. Common Interview Question: B-Tree vs Hash Index

A simplified comparison:

### B-Tree

Suitable for many:

```text
Equality lookups
Range lookups
Ordered traversal
Prefix access
```

### Hash

Primarily useful for:

```text
Equality-based lookup
```

depending on the database implementation.

B-tree indexes are more general-purpose because they preserve ordering.

---

# 64. Common Interview Question: Composite Index

Suppose:

```text
INDEX(A, B)
```

A strong answer should mention:

- The index is ordered primarily by A
- B is ordered within A values
- Queries using A can often benefit
- Queries using A and B can often benefit
- Queries using B alone may not get efficient direct lookup
- Column order should reflect workload
- Actual behavior must be verified with the DBMS

---

# 65. Common Interview Question: Covering Index

A strong answer:

> A covering index contains the data required to satisfy a particular query, allowing the database potentially to avoid fetching additional columns from the underlying table. Whether this results in an index-only execution depends on the database engine and runtime conditions.

---

# 66. Common Interview Questions

Be prepared to answer:

1. What is an index?
2. Why do databases use indexes?
3. How does an index improve query performance?
4. What is a B-tree index?
5. B-tree vs hash index?
6. What is a composite index?
7. Why does composite index column order matter?
8. What is the leftmost-prefix principle?
9. What is selectivity?
10. What is cardinality?
11. What is a covering index?
12. What is an index-only scan?
13. What is a clustered index?
14. What is a secondary index?
15. Does a primary key automatically create an index?
16. Does a foreign key automatically create an index?
17. Why not index every column?
18. How do indexes affect INSERT?
19. How do indexes affect UPDATE?
20. How do indexes affect DELETE?
21. Why might an optimizer ignore an index?
22. What is a partial index?
23. What is an expression index?
24. What is SARGability?
25. Can indexes help ORDER BY?
26. Can indexes help GROUP BY?
27. Composite index vs multiple single-column indexes?
28. What is EXPLAIN?
29. What is EXPLAIN ANALYZE?
30. How would you investigate a slow query?

---

# 67. Common Mistakes

## Mistake 1: Indexes always make queries faster

Incorrect.

Indexes help particular access patterns.

For some queries, scans are cheaper.

---

## Mistake 2: More indexes always mean better performance

Incorrect.

Indexes increase write, storage, and maintenance costs.

---

## Mistake 3: Every WHERE column should be indexed

Incorrect.

Consider:

```text
Selectivity
Query frequency
Data size
Existing indexes
Write workload
```

before creating an index.

---

## Mistake 4: Foreign keys always create indexes

Not universally true.

Foreign key enforcement and indexing are separate concerns.

---

## Mistake 5: A composite index works equally well in any column order

Incorrect.

```text
(A, B)
```

and:

```text
(B, A)
```

are different indexes.

---

## Mistake 6: An index exists, so the database must use it

Incorrect.

The optimizer chooses the estimated cheapest plan.

---

## Mistake 7: Table scans are always bad

Incorrect.

They can be optimal for:

```text
Small tables
Large result sets
Low-selectivity queries
```

---

## Mistake 8: Covering indexes have no cost

Incorrect.

Adding columns increases index size and can increase maintenance overhead.

---

## Mistake 9: EXPLAIN shows what definitely happened

A normal `EXPLAIN` commonly shows the planned execution strategy.

To inspect actual runtime behavior, systems may provide tools such as:

```text
EXPLAIN ANALYZE
```

depending on the DBMS.

---

## Mistake 10: Index advice is identical across databases

Incorrect.

PostgreSQL, MySQL, SQL Server, Oracle, SQLite, and other systems differ in:

```text
Storage
Index types
Optimizers
Clustering
Statistics
Maintenance
Execution plans
```

Always consider the actual database.

---

# 68. Index Design Framework

When designing an index, reason in this order:

```text
What query is slow?
        ↓
What rows does it need?
        ↓
What predicates are used?
        ↓
What is the relationship cardinality?
        ↓
How selective are the predicates?
        ↓
Are joins involved?
        ↓
Is sorting required?
        ↓
Is grouping required?
        ↓
What indexes already exist?
        ↓
What write cost will the new index add?
        ↓
Create candidate index
        ↓
Inspect execution plan
        ↓
Measure with realistic data
```

Do not design indexes from schema names alone.

---

# 69. Example Interview Scenario

Suppose we have:

```text
interviews(
    id,
    candidate_id,
    status,
    score,
    created_at
)
```

There are:

```text
50,000,000 rows
```

The application frequently runs:

```sql
SELECT
    id,
    score,
    created_at
FROM interviews
WHERE candidate_id = ?
  AND status = 'completed'
ORDER BY created_at DESC
LIMIT 10;
```

A reasonable candidate index to evaluate is:

```sql
CREATE INDEX idx_interviews_candidate_status_created
ON interviews(
    candidate_id,
    status,
    created_at DESC
);
```

Reasoning:

```text
candidate_id
    ↓
Restrict to one candidate
    ↓
status
    ↓
Restrict to completed interviews
    ↓
created_at DESC
    ↓
Provide useful ordering
    ↓
LIMIT 10
```

But the interview answer should not end there.

Add:

> I would verify the index using the execution plan and realistic production-like data because selectivity, distribution, existing indexes, and database-specific optimizer behavior can change the best design.

That demonstrates engineering judgment.

---

# 70. Slow Query Investigation

Suppose:

```sql
SELECT *
FROM interviews
WHERE candidate_id = 123;
```

takes five seconds.

Do not immediately say:

> Add an index.

Investigate:

```text
1. Execution plan
2. Number of rows in the table
3. Number of rows returned
4. Existing indexes
5. Statistics
6. I/O
7. Locks / waits
8. Query frequency
9. Data distribution
10. Application/network latency
```

The query may not even be slow because of indexing.

---

# 71. Interview Answer: What Is an Index?

Weak answer:

> An index makes database queries faster.

Better answer:

> An index is an additional data structure maintained by a database to provide efficient access paths for particular queries. For example, a B-tree index can help with equality, range, and ordered lookups. The trade-off is additional storage and maintenance cost on writes, so indexes should be designed around workload patterns rather than added to every column.

---

# 72. Interview Answer: Why Not Index Everything?

A strong answer:

> Every index consumes storage and must be maintained when relevant data changes. While indexes can reduce read cost for certain queries, excessive indexing can increase insert, update, and delete overhead and create redundant structures. I would design indexes around important query patterns and verify their value through execution plans and measurements.

---

# 73. Interview Answer: Why Is My Index Not Used?

A strong answer:

> The optimizer chooses a plan based on estimated cost, not simply on whether an index exists. It may choose a scan when the table is small, the predicate matches a large percentage of rows, the index does not align with the query, statistics produce different estimates, or accessing many table rows through the index would cost more than scanning. I would inspect the execution plan before changing the index.

---

# 74. Quick Revision

Remember:

```text
Index
  ↓
Additional data structure
  ↓
Potentially faster reads
  +
Storage cost
  +
Write cost
```

For composite indexes:

```text
(A, B, C)
```

think:

```text
A
A + B
A + B + C
```

but avoid turning the leftmost-prefix principle into an absolute rule.

For performance:

```text
Query
  ↓
Execution Plan
  ↓
Measure
  ↓
Optimize
  ↓
Measure Again
```

---

# 75. Final Takeaway

Indexing is not:

```text
Slow query
   ↓
Add index
   ↓
Done
```

A better mental model is:

```text
Understand workload
        ↓
Understand data distribution
        ↓
Inspect execution plan
        ↓
Identify access bottleneck
        ↓
Design appropriate index
        ↓
Measure improvement
        ↓
Evaluate write/storage cost
```

The strongest interview answers discuss both:

```text
Benefit
+
Trade-off
```

because an index is never free.

---

## Previous

**[← SQL and Joins](./04-sql-and-joins.md)**

## Next

**[Transactions and ACID →](./06-transactions-and-acid.md)**

---

Maintained by **InterviewEraHQ**.
