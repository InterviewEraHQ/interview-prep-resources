# SQL and Joins

SQL is one of the most important skills tested in database and backend interviews.

Knowing SQL syntax is only the starting point. Strong candidates should understand:

- How queries logically operate
- How tables are combined
- How aggregation works
- How `NULL` affects comparisons
- How to solve common interview query patterns
- How query design can affect correctness and performance

This guide covers the SQL concepts most commonly expected in software engineering interviews.

---

# 1. What Is SQL?

**SQL (Structured Query Language)** is a language used to interact with relational database systems.

SQL can be used to:

```text
Create database objects
Read data
Insert data
Update data
Delete data
Define constraints
Manage transactions
Control permissions
```

Example:

```sql
SELECT name, email
FROM candidates
WHERE experience_years >= 2;
```

This retrieves selected candidates whose experience satisfies the condition.

---

# 2. SQL Command Categories

SQL commands are commonly grouped into categories.

## DDL — Data Definition Language

Used to define or modify database structures.

Examples:

```sql
CREATE
ALTER
DROP
TRUNCATE
```

Example:

```sql
CREATE TABLE candidates (
    id BIGINT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL
);
```

---

## DML — Data Manipulation Language

Used to modify stored data.

Common examples:

```sql
INSERT
UPDATE
DELETE
```

Example:

```sql
INSERT INTO candidates (id, name, email)
VALUES (1, 'Alex', 'alex@example.com');
```

---

## DQL — Data Query Language

`SELECT` is commonly described as DQL because it retrieves data.

```sql
SELECT *
FROM candidates;
```

Terminology varies between educational sources and database documentation, so focus on what the statements actually do rather than memorizing category labels.

---

## TCL — Transaction Control Language

Used to control transactions.

Examples:

```sql
COMMIT
ROLLBACK
SAVEPOINT
```

Transactions are covered separately in the transactions guide.

---

## DCL — Data Control Language

Used for access and permission management.

Examples include:

```sql
GRANT
REVOKE
```

Exact capabilities and syntax depend on the DBMS.

---

# 3. SELECT

`SELECT` retrieves data.

Example:

```sql
SELECT id, name, email
FROM candidates;
```

To retrieve all columns:

```sql
SELECT *
FROM candidates;
```

For production code, explicitly selecting required columns is often preferable to blindly using `SELECT *`.

For example:

```sql
SELECT id, name
FROM candidates;
```

can make the intended data requirements clearer.

---

# 4. WHERE

`WHERE` filters rows.

```sql
SELECT id, name
FROM candidates
WHERE experience_years >= 2;
```

Multiple conditions:

```sql
SELECT id, name
FROM candidates
WHERE experience_years >= 2
  AND status = 'active';
```

Using `OR`:

```sql
SELECT id, name
FROM candidates
WHERE role = 'Backend Developer'
   OR role = 'Frontend Developer';
```

---

# 5. Comparison Operators

Common operators include:

```text
=
<>
!=
>
<
>=
<=
```

Support for particular syntax can vary by DBMS.

Example:

```sql
SELECT *
FROM interviews
WHERE score >= 80;
```

---

# 6. IN

Instead of:

```sql
WHERE role = 'Backend Developer'
   OR role = 'Frontend Developer'
   OR role = 'Data Engineer'
```

you can often write:

```sql
WHERE role IN (
    'Backend Developer',
    'Frontend Developer',
    'Data Engineer'
)
```

---

# 7. BETWEEN

Example:

```sql
SELECT *
FROM interviews
WHERE score BETWEEN 70 AND 90;
```

`BETWEEN` is inclusive of both boundaries in SQL.

Conceptually:

```text
score >= 70
AND
score <= 90
```

---

# 8. LIKE

`LIKE` performs pattern matching.

Example:

```sql
SELECT *
FROM candidates
WHERE name LIKE 'A%';
```

This matches values beginning with `A`.

Common wildcard characters:

```text
%  → zero or more characters
_  → one character
```

Example:

```sql
WHERE name LIKE '_lex'
```

could match:

```text
Alex
```

Pattern-matching behavior, including case sensitivity, can vary by DBMS and collation.

---

# 9. ORDER BY

Used to sort results.

```sql
SELECT name, score
FROM interview_results
ORDER BY score DESC;
```

Ascending:

```sql
ORDER BY score ASC;
```

Multiple columns:

```sql
ORDER BY score DESC, name ASC;
```

---

# 10. LIMIT

Many database systems support `LIMIT` for restricting returned rows.

Example:

```sql
SELECT *
FROM interviews
ORDER BY score DESC
LIMIT 5;
```

This retrieves five rows after sorting.

However, row-limiting syntax varies across database systems.

For example, some systems use constructs such as:

```text
LIMIT
TOP
FETCH FIRST
```

Do not assume every DBMS uses identical syntax.

---

# 11. DISTINCT

`DISTINCT` removes duplicate rows from the selected result.

Example:

```sql
SELECT DISTINCT role
FROM candidates;
```

Suppose the data contains:

```text
Backend Developer
Backend Developer
Frontend Developer
```

The result becomes:

```text
Backend Developer
Frontend Developer
```

---

# 12. Aggregate Functions

Aggregate functions summarize multiple rows.

Common examples:

```text
COUNT()
SUM()
AVG()
MIN()
MAX()
```

Example:

```sql
SELECT COUNT(*)
FROM candidates;
```

Average score:

```sql
SELECT AVG(score)
FROM interviews;
```

Highest score:

```sql
SELECT MAX(score)
FROM interviews;
```

---

# 13. COUNT(*) vs COUNT(column)

This distinction is frequently tested.

```sql
COUNT(*)
```

counts rows.

```sql
COUNT(score)
```

counts rows where `score` is not `NULL`.

Suppose:

| id | score |
|---:|---:|
| 1 | 80 |
| 2 | NULL |
| 3 | 90 |

Then:

```sql
SELECT COUNT(*)
FROM interviews;
```

returns:

```text
3
```

while:

```sql
SELECT COUNT(score)
FROM interviews;
```

returns:

```text
2
```

---

# 14. GROUP BY

`GROUP BY` groups rows so aggregates can be calculated per group.

Example:

```sql
SELECT candidate_id, COUNT(*) AS interview_count
FROM interviews
GROUP BY candidate_id;
```

Possible result:

| candidate_id | interview_count |
|---:|---:|
| 1 | 3 |
| 2 | 1 |
| 3 | 5 |

---

# 15. HAVING

`HAVING` filters groups after grouping.

Example:

```sql
SELECT candidate_id, COUNT(*) AS interview_count
FROM interviews
GROUP BY candidate_id
HAVING COUNT(*) >= 3;
```

This returns candidates having at least three interview rows.

---

# 16. WHERE vs HAVING

A common interview question.

### WHERE

Filters rows before grouping.

```sql
WHERE score >= 50
```

### HAVING

Filters groups after aggregation.

```sql
HAVING AVG(score) >= 80
```

Example:

```sql
SELECT candidate_id, AVG(score) AS avg_score
FROM interviews
WHERE status = 'completed'
GROUP BY candidate_id
HAVING AVG(score) >= 80;
```

Conceptually:

```text
Filter completed interviews
        ↓
Group by candidate
        ↓
Calculate average
        ↓
Keep groups with average >= 80
```

---

# 17. Logical Query Processing Order

A SQL query may be written as:

```sql
SELECT candidate_id, AVG(score) AS avg_score
FROM interviews
WHERE status = 'completed'
GROUP BY candidate_id
HAVING AVG(score) >= 80
ORDER BY avg_score DESC;
```

A useful conceptual logical processing order is:

```text
FROM
 ↓
WHERE
 ↓
GROUP BY
 ↓
HAVING
 ↓
SELECT
 ↓
DISTINCT
 ↓
ORDER BY
 ↓
LIMIT / FETCH
```

This is a conceptual model for reasoning about queries.

It should not be confused with the database engine's physical execution plan, which the optimizer may transform significantly.

---

# 18. What Is a JOIN?

A **JOIN** combines related rows from multiple tables.

Suppose we have:

### Candidates

| id | name |
|---:|---|
| 1 | Alex |
| 2 | Sam |
| 3 | Priya |

### Interviews

| id | candidate_id | score |
|---:|---:|---:|
| 101 | 1 | 82 |
| 102 | 1 | 91 |
| 103 | 2 | 76 |

The relationship is:

```text
candidates.id
      ↓
interviews.candidate_id
```

---

# 19. INNER JOIN

`INNER JOIN` returns rows where the join condition matches between both sides.

```sql
SELECT
    c.name,
    i.score
FROM candidates AS c
INNER JOIN interviews AS i
    ON c.id = i.candidate_id;
```

Result:

| name | score |
|---|---:|
| Alex | 82 |
| Alex | 91 |
| Sam | 76 |

Priya does not appear because she has no matching interview.

Conceptually:

```text
Candidates ∩ Matching Interviews
```

More precisely, it returns combinations satisfying the join predicate.

---

# 20. LEFT JOIN

`LEFT JOIN` returns all rows from the left table and matching rows from the right table.

```sql
SELECT
    c.name,
    i.score
FROM candidates AS c
LEFT JOIN interviews AS i
    ON c.id = i.candidate_id;
```

Result:

| name | score |
|---|---:|
| Alex | 82 |
| Alex | 91 |
| Sam | 76 |
| Priya | NULL |

Priya remains because every candidate from the left side is preserved.

---

# 21. Find Candidates With No Interviews

This is a common interview pattern.

Using `LEFT JOIN`:

```sql
SELECT
    c.id,
    c.name
FROM candidates AS c
LEFT JOIN interviews AS i
    ON c.id = i.candidate_id
WHERE i.id IS NULL;
```

Why?

Candidates without matching interviews receive:

```text
NULL
```

for columns from the right table.

---

# 22. RIGHT JOIN

`RIGHT JOIN` preserves all rows from the right table and matching rows from the left table.

Example:

```sql
SELECT
    c.name,
    i.score
FROM candidates AS c
RIGHT JOIN interviews AS i
    ON c.id = i.candidate_id;
```

Conceptually, it is the directional counterpart of a left join.

In many cases, the same query can be rewritten as a `LEFT JOIN` by reversing table order.

Support can vary by DBMS.

---

# 23. FULL OUTER JOIN

`FULL OUTER JOIN` returns:

- Matching rows
- Unmatched rows from the left side
- Unmatched rows from the right side

Example:

```sql
SELECT
    c.name,
    i.score
FROM candidates AS c
FULL OUTER JOIN interviews AS i
    ON c.id = i.candidate_id;
```

Not every database system supports `FULL OUTER JOIN` directly.

Always consider the target DBMS.

---

# 24. CROSS JOIN

A `CROSS JOIN` returns the Cartesian product of two inputs.

Suppose:

```text
Candidates = 3 rows
Roles = 4 rows
```

Then:

```sql
SELECT *
FROM candidates
CROSS JOIN roles;
```

can produce:

```text
3 × 4 = 12 rows
```

Cross joins are useful when every combination is intentionally required.

Accidental Cartesian products can create unexpectedly huge results.

---

# 25. SELF JOIN

A self join joins a table to itself.

Consider employees:

| employee_id | name | manager_id |
|---:|---|---:|
| 1 | Maya | NULL |
| 2 | Alex | 1 |
| 3 | Sam | 1 |

To retrieve employee and manager names:

```sql
SELECT
    e.name AS employee,
    m.name AS manager
FROM employees AS e
LEFT JOIN employees AS m
    ON e.manager_id = m.employee_id;
```

The same table plays two logical roles:

```text
e → employee
m → manager
```

Aliases make those roles explicit.

---

# 26. JOIN vs UNION

These operations solve different problems.

### JOIN

Combines columns from related rows.

Conceptually:

```text
Table A + Table B
        ↓
More columns
```

### UNION

Combines compatible result sets vertically.

Conceptually:

```text
Result A
Result B
   ↓
More rows
```

Example:

```sql
SELECT email
FROM candidates

UNION

SELECT email
FROM interviewers;
```

---

# 27. UNION vs UNION ALL

`UNION` removes duplicate rows from the combined result.

```sql
SELECT email FROM candidates
UNION
SELECT email FROM interviewers;
```

`UNION ALL` retains duplicates.

```sql
SELECT email FROM candidates
UNION ALL
SELECT email FROM interviewers;
```

If duplicate elimination is unnecessary, `UNION ALL` can avoid the work required to remove duplicates.

Actual performance depends on the DBMS and execution plan.

---

# 28. Subqueries

A **subquery** is a query nested inside another SQL statement.

Example:

```sql
SELECT name
FROM candidates
WHERE id IN (
    SELECT candidate_id
    FROM interviews
    WHERE score >= 90
);
```

The inner query identifies candidate IDs meeting the score condition.

---

# 29. Scalar Subquery

A scalar subquery returns a single value.

Example:

```sql
SELECT name
FROM candidates
WHERE id = (
    SELECT candidate_id
    FROM interviews
    ORDER BY score DESC
    LIMIT 1
);
```

This example assumes the subquery returns one row.

If a scalar subquery produces multiple rows where only one value is allowed, the database generally raises an error.

---

# 30. Correlated Subquery

A correlated subquery refers to values from the outer query.

Example:

```sql
SELECT
    c.id,
    c.name
FROM candidates AS c
WHERE EXISTS (
    SELECT 1
    FROM interviews AS i
    WHERE i.candidate_id = c.id
      AND i.score >= 90
);
```

The inner query depends on:

```text
c.id
```

from the outer query.

---

# 31. EXISTS

`EXISTS` checks whether a subquery produces at least one row.

Example:

```sql
SELECT
    c.id,
    c.name
FROM candidates AS c
WHERE EXISTS (
    SELECT 1
    FROM interviews AS i
    WHERE i.candidate_id = c.id
);
```

This finds candidates with at least one interview.

---

# 32. NOT EXISTS

To find candidates with no interviews:

```sql
SELECT
    c.id,
    c.name
FROM candidates AS c
WHERE NOT EXISTS (
    SELECT 1
    FROM interviews AS i
    WHERE i.candidate_id = c.id
);
```

This is an alternative to the `LEFT JOIN ... IS NULL` pattern.

---

# 33. IN vs EXISTS

Candidates often memorize:

> EXISTS is faster than IN.

That is not a safe universal rule.

Modern optimizers can transform queries in different ways.

Choose based on:

- Correct semantics
- Readability
- `NULL` behavior
- Execution plan
- Actual DBMS
- Data distribution

Then measure when performance matters.

---

# 34. Common Table Expressions — CTEs

A CTE defines a named query expression, commonly using `WITH`.

Example:

```sql
WITH candidate_scores AS (
    SELECT
        candidate_id,
        AVG(score) AS avg_score
    FROM interviews
    GROUP BY candidate_id
)
SELECT
    candidate_id,
    avg_score
FROM candidate_scores
WHERE avg_score >= 80;
```

CTEs can improve readability for complex queries.

Whether a CTE is materialized or optimized into the surrounding query depends on the DBMS, version, query, and configuration.

Do not assume a CTE always creates a temporary table.

---

# 35. NULL

`NULL` represents the absence of a known value.

It is not equivalent to:

```text
0
''
false
```

To test for `NULL`, use:

```sql
IS NULL
```

Example:

```sql
SELECT *
FROM interviews
WHERE score IS NULL;
```

Not:

```sql
WHERE score = NULL
```

---

# 36. Three-Valued Logic

SQL comparisons can conceptually produce:

```text
TRUE
FALSE
UNKNOWN
```

For example:

```sql
NULL = 10
```

does not evaluate to `TRUE`.

And:

```sql
NULL = NULL
```

also does not behave like ordinary equality.

This is why SQL provides:

```sql
IS NULL
```

and:

```sql
IS NOT NULL
```

---

# 37. NOT IN and NULL Trap

Suppose:

```sql
SELECT id
FROM candidates
WHERE id NOT IN (
    SELECT candidate_id
    FROM blocked_candidates
);
```

If the subquery can return `NULL`, SQL's three-valued logic can make the result surprising.

A common safer pattern for anti-matching is:

```sql
SELECT
    c.id
FROM candidates AS c
WHERE NOT EXISTS (
    SELECT 1
    FROM blocked_candidates AS b
    WHERE b.candidate_id = c.id
);
```

You should understand the `NULL` semantics rather than blindly replacing every `NOT IN`.

---

# 38. CASE

`CASE` provides conditional expressions.

Example:

```sql
SELECT
    candidate_id,
    score,
    CASE
        WHEN score >= 90 THEN 'Excellent'
        WHEN score >= 75 THEN 'Good'
        WHEN score >= 60 THEN 'Average'
        ELSE 'Needs Improvement'
    END AS performance
FROM interviews;
```

---

# 39. COALESCE

`COALESCE` returns the first non-null expression.

Example:

```sql
SELECT
    name,
    COALESCE(phone, 'Not provided') AS phone
FROM candidates;
```

If `phone` is `NULL`, the fallback value is returned.

---

# 40. Common Interview Query: Second Highest Salary

Suppose:

```text
employees(
    id,
    name,
    salary
)
```

One approach:

```sql
SELECT MAX(salary) AS second_highest_salary
FROM employees
WHERE salary < (
    SELECT MAX(salary)
    FROM employees
);
```

This finds the second **distinct** highest salary.

Another approach using ranking:

```sql
WITH ranked_salaries AS (
    SELECT
        salary,
        DENSE_RANK() OVER (
            ORDER BY salary DESC
        ) AS salary_rank
    FROM employees
)
SELECT DISTINCT salary
FROM ranked_salaries
WHERE salary_rank = 2;
```

Interviewers may specifically ask how duplicates should be treated.

Clarify whether they mean:

```text
second row after sorting
```

or:

```text
second distinct highest salary
```

Those are different problems.

---

# 41. Window Functions

Window functions calculate values across related rows without collapsing them into one row per group.

Example:

```sql
SELECT
    candidate_id,
    score,
    AVG(score) OVER (
        PARTITION BY candidate_id
    ) AS candidate_average
FROM interviews;
```

Unlike:

```sql
GROUP BY candidate_id
```

the original interview rows remain visible.

---

# 42. ROW_NUMBER

Example:

```sql
SELECT
    candidate_id,
    score,
    ROW_NUMBER() OVER (
        PARTITION BY candidate_id
        ORDER BY score DESC
    ) AS row_num
FROM interviews;
```

Each row receives a unique sequence number within its candidate partition.

---

# 43. RANK vs DENSE_RANK

Suppose scores are:

```text
100
90
90
80
```

Using `RANK()`:

```text
100 → 1
90  → 2
90  → 2
80  → 4
```

Using `DENSE_RANK()`:

```text
100 → 1
90  → 2
90  → 2
80  → 3
```

`RANK()` leaves gaps after ties.

`DENSE_RANK()` does not.

---

# 44. Top Score Per Candidate

Using `ROW_NUMBER()`:

```sql
WITH ranked_interviews AS (
    SELECT
        id,
        candidate_id,
        score,
        ROW_NUMBER() OVER (
            PARTITION BY candidate_id
            ORDER BY score DESC
        ) AS row_num
    FROM interviews
)
SELECT
    id,
    candidate_id,
    score
FROM ranked_interviews
WHERE row_num = 1;
```

But consider ties.

If two interviews have the same score, `ROW_NUMBER()` selects one according to the full ordering available to the database.

For deterministic results, provide a tie-breaker:

```sql
ORDER BY score DESC, id ASC
```

If all tied top scores should be returned, `RANK()` or `DENSE_RANK()` may be more appropriate.

---

# 45. Find Duplicate Emails

A classic interview query:

```sql
SELECT
    email,
    COUNT(*) AS occurrences
FROM candidates
GROUP BY email
HAVING COUNT(*) > 1;
```

This identifies email values appearing more than once.

---

# 46. Delete Duplicates Carefully

Deleting duplicates is more complicated than finding them.

First define:

- What makes rows duplicates?
- Which row should survive?
- Are foreign keys referencing them?
- Is deletion reversible?
- Is there a transaction and backup strategy?

A common pattern for identifying duplicates is:

```sql
WITH ranked AS (
    SELECT
        id,
        email,
        ROW_NUMBER() OVER (
            PARTITION BY email
            ORDER BY id
        ) AS row_num
    FROM candidates
)
SELECT *
FROM ranked
WHERE row_num > 1;
```

Before turning such logic into a `DELETE`, verify the result set.

Production data cleanup should not be treated as a casual interview one-liner.

---

# 47. Nth Highest Salary

Using `DENSE_RANK()`:

```sql
WITH ranked AS (
    SELECT
        salary,
        DENSE_RANK() OVER (
            ORDER BY salary DESC
        ) AS salary_rank
    FROM employees
)
SELECT DISTINCT salary
FROM ranked
WHERE salary_rank = 3;
```

For a parameterized N:

```text
salary_rank = N
```

This solves the Nth **distinct** highest salary problem.

---

# 48. Employees Earning More Than Their Manager

Given:

```text
employees(
    id,
    name,
    salary,
    manager_id
)
```

Use a self join:

```sql
SELECT
    e.name AS employee,
    e.salary AS employee_salary,
    m.name AS manager,
    m.salary AS manager_salary
FROM employees AS e
JOIN employees AS m
    ON e.manager_id = m.id
WHERE e.salary > m.salary;
```

This is a common self-join interview pattern.

---

# 49. Candidates Above Their Own Average

Suppose we want interview attempts whose score is greater than that candidate's average.

Using a window function:

```sql
WITH scored AS (
    SELECT
        id,
        candidate_id,
        score,
        AVG(score) OVER (
            PARTITION BY candidate_id
        ) AS avg_score
    FROM interviews
)
SELECT
    id,
    candidate_id,
    score,
    avg_score
FROM scored
WHERE score > avg_score;
```

This demonstrates why window functions can be useful.

---

# 50. JOIN Duplication Trap

Suppose:

```text
Candidate 1 → 3 interviews
Candidate 2 → 2 interviews
```

Then:

```sql
SELECT c.*
FROM candidates AS c
JOIN interviews AS i
    ON c.id = i.candidate_id;
```

can return Candidate 1 three times and Candidate 2 twice.

This is not necessarily a duplicate-data problem.

It is a consequence of the relationship cardinality.

If you only need candidates that have interviews, `EXISTS` may express the intent more directly:

```sql
SELECT
    c.*
FROM candidates AS c
WHERE EXISTS (
    SELECT 1
    FROM interviews AS i
    WHERE i.candidate_id = c.id
);
```

---

# 51. LEFT JOIN Filter Trap

Consider:

```sql
SELECT
    c.name,
    i.score
FROM candidates AS c
LEFT JOIN interviews AS i
    ON c.id = i.candidate_id
WHERE i.score >= 80;
```

The `WHERE` condition removes rows where:

```text
i.score IS NULL
```

so candidates without interviews disappear.

For some use cases, the intended query may instead be:

```sql
SELECT
    c.name,
    i.score
FROM candidates AS c
LEFT JOIN interviews AS i
    ON c.id = i.candidate_id
   AND i.score >= 80;
```

Now all candidates remain, while only qualifying interviews match.

The correct version depends on the requirement.

This distinction is frequently useful in SQL interviews.

---

# 52. GROUP BY Trap

Suppose:

```sql
SELECT
    candidate_id,
    name,
    AVG(score)
FROM interviews
GROUP BY candidate_id;
```

Whether this is valid depends on the schema and DBMS rules.

In standard SQL reasoning, selected non-aggregated expressions generally need to be grouped or functionally dependent in a way recognized by the database.

Do not rely on permissive DBMS behavior when explaining portable SQL.

---

# 53. Query Correctness Before Optimization

Suppose two queries return the same result for your current sample data.

That does not prove they are semantically equivalent.

Test cases should include:

```text
NULL values
Duplicate values
Empty tables
Ties
Missing relationships
Multiple child rows
Boundary values
```

In interviews, correctness comes before micro-optimization.

---

# 54. SQL Performance Basics

When a query is slow, do not immediately rewrite random syntax.

Investigate factors such as:

```text
Execution plan
Indexes
Rows scanned
Join strategy
Cardinality estimates
Filtering
Sorting
Aggregation
Data distribution
Table size
```

Example:

```sql
SELECT *
FROM interviews
WHERE candidate_id = 123;
```

An appropriate index on:

```text
candidate_id
```

may help depending on selectivity, table size, query shape, and optimizer decisions.

Indexes are covered in the next guide.

---

# 55. Common SQL Interview Questions

You should be comfortable answering:

1. What is SQL?
2. What is the difference between DDL and DML?
3. What is `WHERE`?
4. What is `GROUP BY`?
5. What is `HAVING`?
6. `WHERE` vs `HAVING`?
7. `COUNT(*)` vs `COUNT(column)`?
8. What is a join?
9. `INNER JOIN` vs `LEFT JOIN`?
10. What is a self join?
11. What is a cross join?
12. `JOIN` vs `UNION`?
13. `UNION` vs `UNION ALL`?
14. What is a subquery?
15. What is a correlated subquery?
16. What is `EXISTS`?
17. `IN` vs `EXISTS`?
18. What is a CTE?
19. What is `NULL`?
20. Why does `column = NULL` not work as expected?
21. What is a window function?
22. `ROW_NUMBER` vs `RANK` vs `DENSE_RANK`?
23. How would you find duplicate values?
24. How would you find the second highest salary?
25. How would you find rows with no related records?

---

# 56. Common Mistakes

## Mistake 1: WHERE and HAVING are interchangeable

Incorrect.

`WHERE` filters rows before grouping.

`HAVING` filters groups after grouping.

---

## Mistake 2: COUNT(*) and COUNT(column) always return the same result

Incorrect when the column contains `NULL`.

---

## Mistake 3: LEFT JOIN always preserves every left row regardless of WHERE

Incorrect.

A condition on right-table columns in `WHERE` can remove unmatched rows.

---

## Mistake 4: UNION and UNION ALL are identical

Incorrect.

`UNION` performs duplicate elimination.

`UNION ALL` retains duplicates.

---

## Mistake 5: NULL equals NULL

Ordinary equality does not treat `NULL` like a regular value.

Use the appropriate `IS NULL` semantics.

---

## Mistake 6: EXISTS is always faster than IN

Incorrect.

Performance depends on the optimizer, data, indexes, query semantics, and DBMS.

---

## Mistake 7: CTEs always improve performance

Incorrect.

A CTE primarily provides a way to structure a query.

Its execution behavior depends on the database system and query.

---

## Mistake 8: DISTINCT is the correct fix for every duplicate result

Often wrong.

Unexpected repeated rows may indicate:

- One-to-many relationships
- Incorrect join predicates
- Missing join conditions
- Misunderstood requirements

Adding:

```sql
DISTINCT
```

can hide the underlying issue.

---

# 57. SQL Problem-Solving Framework

When given a SQL interview problem, use this process:

```text
1. Understand the required output
        ↓
2. Identify required tables
        ↓
3. Understand relationships/cardinality
        ↓
4. Determine filtering
        ↓
5. Determine grouping/aggregation
        ↓
6. Handle NULL and duplicates
        ↓
7. Determine ordering/ranking
        ↓
8. Write the query
        ↓
9. Test edge cases
        ↓
10. Consider performance
```

Do not start typing SQL before understanding the expected result.

---

# 58. Example Interview Problem

**Question:**

Find candidates who completed at least three interviews and have an average score of at least 80.

Schema:

```text
interviews(
    id,
    candidate_id,
    status,
    score
)
```

Solution:

```sql
SELECT
    candidate_id,
    COUNT(*) AS interview_count,
    AVG(score) AS avg_score
FROM interviews
WHERE status = 'completed'
GROUP BY candidate_id
HAVING COUNT(*) >= 3
   AND AVG(score) >= 80;
```

Reasoning:

```text
Completed interviews only
        ↓
WHERE
        ↓
Group by candidate
        ↓
GROUP BY
        ↓
Calculate count and average
        ↓
Keep qualifying groups
        ↓
HAVING
```

This reasoning is more important than memorizing the final query.

---

# 59. Example With Multiple Tables

**Question:**

Return candidate names and their average completed-interview score, but only for candidates whose average is at least 80.

```sql
SELECT
    c.id,
    c.name,
    AVG(i.score) AS avg_score
FROM candidates AS c
JOIN interviews AS i
    ON i.candidate_id = c.id
WHERE i.status = 'completed'
GROUP BY
    c.id,
    c.name
HAVING AVG(i.score) >= 80;
```

This combines:

```text
JOIN
WHERE
GROUP BY
HAVING
```

Understanding how those stages interact is a core SQL interview skill.

---

# 60. Final Takeaway

Strong SQL knowledge is not about memorizing dozens of queries.

It is about understanding how relational data is transformed.

Think in stages:

```text
FROM / JOIN
      ↓
WHERE
      ↓
GROUP BY
      ↓
HAVING
      ↓
SELECT
      ↓
ORDER BY
      ↓
LIMIT
```

And always reason about:

```text
Relationships
Cardinality
NULL
Duplicates
Aggregation
Correctness
Performance
```

Once those concepts are clear, unfamiliar SQL interview problems become much easier to solve.

---

## Previous

**[← Normalization](./03-normalization.md)**

## Next

**[Indexing →](./05-indexing.md)**

---

Maintained by **InterviewEraHQ**.
