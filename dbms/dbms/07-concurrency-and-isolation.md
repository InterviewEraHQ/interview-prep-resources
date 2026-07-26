# Concurrency and Isolation in DBMS

Modern databases rarely execute only one transaction at a time.

Real applications have many users and services accessing the same data concurrently:

```text
Two users purchasing the last item

Two requests deducting the same interview credit

Multiple transfers updating the same account

Two admins editing the same record

Thousands of users reading while other transactions write
```

Concurrency improves throughput, but it introduces correctness problems.

A strong interview candidate should understand:

- Why databases support concurrent transactions
- Race conditions
- Dirty reads
- Non-repeatable reads
- Phantom reads
- Lost updates
- Write skew
- Isolation levels
- Serializable execution
- Locks
- Shared and exclusive locking
- Pessimistic concurrency control
- Optimistic concurrency control
- MVCC
- Snapshots
- Deadlocks
- Retries
- How to choose a concurrency-control strategy

The central problem is:

> How can multiple transactions operate concurrently without producing unacceptable results?

---

# 1. What Is Database Concurrency?

**Concurrency** means multiple transactions can make progress during overlapping periods of time.

Conceptually:

```text
Transaction A ───────────────►

      Transaction B ───────────────►

Transaction C ─────────►
```

They do not necessarily execute one after another.

Concurrency allows databases to serve many requests efficiently.

But transactions may interact with the same data.

That is where concurrency control becomes necessary.

---

# 2. Why Do Databases Need Concurrency?

Imagine a production application with:

```text
10,000 active users
```

If every transaction had to wait for all previous transactions to finish:

```text
Transaction 1
      ↓
Transaction 2
      ↓
Transaction 3
      ↓
Transaction 4
```

throughput could become severely limited.

Instead, databases allow overlapping work where possible.

Example:

```text
User A reads candidate 10

User B updates candidate 500

User C inserts an interview

User D reads interview history
```

These operations may not conflict with one another.

Concurrency lets the system utilize resources more effectively.

---

# 3. The Concurrency Problem

Concurrency becomes difficult when transactions interact with the same logical data.

Suppose:

```text
Candidate credits = 1
```

Two requests arrive:

```text
Request A → Start interview

Request B → Start interview
```

Both perform:

```text
Read credits
Check credits > 0
Deduct credit
```

Possible execution:

```text
A reads 1

B reads 1

A decides interview is allowed

B decides interview is allowed

A writes 0

B writes 0
```

Two interviews were allowed using one credit.

The final database value:

```text
credits = 0
```

looks reasonable.

But the business outcome is incorrect.

This is a **race condition**.

---

# 4. What Is a Race Condition?

A race condition occurs when the correctness of an operation depends on the timing or interleaving of concurrent operations.

Conceptually:

```text
Transaction A
     ↘
      Shared State
     ↗
Transaction B
```

If different execution timing can produce different results, the system may contain a concurrency bug.

Race conditions are particularly dangerous because they may:

```text
Work correctly during testing

Fail only under load

Appear intermittently

Be difficult to reproduce
```

---

# 5. Transactions Alone Do Not Eliminate Race Conditions

Suppose both requests use transactions:

```text
Transaction A:

BEGIN
Read credits = 1
Write credits = 0
COMMIT
```

and:

```text
Transaction B:

BEGIN
Read credits = 1
Write credits = 0
COMMIT
```

The existence of:

```text
BEGIN
COMMIT
```

does not automatically prevent incorrect concurrent behavior.

Correctness also depends on:

```text
Isolation level
Locking
MVCC semantics
Atomic statements
Constraints
Concurrency-control strategy
```

A transaction defines the unit of work.

Isolation determines how concurrent transactions interact.

---

# 6. What Is Isolation?

Isolation is the ACID property concerned with interaction between concurrently executing transactions.

A useful question is:

> What effects of another transaction is this transaction allowed to observe?

Different isolation levels provide different answers.

Stronger isolation can prevent more concurrency anomalies, but may also introduce:

```text
More blocking

More transaction aborts

More retries

Lower concurrency in some workloads
```

The exact trade-offs depend on the DBMS.

---

# 7. Serial Execution

The simplest conceptual execution model is:

```text
Transaction A
    ↓
COMMIT
    ↓
Transaction B
    ↓
COMMIT
```

Transactions execute one after another.

This makes reasoning easier.

But forcing every transaction to execute globally one at a time would significantly reduce concurrency.

Databases therefore attempt to execute transactions concurrently while preserving appropriate correctness guarantees.

---

# 8. Serializable Execution

**Serializable** does not necessarily mean transactions physically execute one at a time.

It means the outcome is equivalent to some valid serial ordering of those transactions, according to the database's serializability model.

For example:

```text
Actual execution:

A1
B1
A2
B2
```

may still be valid if its observable outcome is equivalent to:

```text
A
then
B
```

or:

```text
B
then
A
```

Serializable isolation is therefore about correctness equivalence, not necessarily literal sequential execution.

---

# 9. Concurrency Anomalies

Important anomalies include:

```text
Dirty Read

Non-Repeatable Read

Phantom Read

Lost Update

Write Skew
```

Understanding these is more useful than memorizing an isolation-level table.

---

# 10. Dirty Read

A **dirty read** occurs when one transaction reads data written by another transaction that has not committed.

Example:

Initial value:

```text
balance = $500
```

Transaction A:

```text
UPDATE balance → $400
```

but has not committed.

Transaction B reads:

```text
balance = $400
```

Then Transaction A rolls back.

Final value:

```text
balance = $500
```

Transaction B observed a value that never became committed state.

---

# 11. Dirty Read Timeline

```text
Transaction A              Transaction B

BEGIN

balance = $400

                           READ balance
                           → $400

ROLLBACK

balance returns to $500
```

Transaction B observed:

```text
$400
```

even though that change was later rolled back.

That is a dirty read.

---

# 12. Why Dirty Reads Are Dangerous

Suppose Transaction B uses the dirty value to make another decision:

```text
Read temporary balance
      ↓
Approve another operation
      ↓
Original transaction rolls back
```

Now the decision was based on data that never actually committed.

For this reason, many production database configurations prevent dirty reads.

Exact behavior depends on the DBMS and isolation level.

---

# 13. Non-Repeatable Read

A **non-repeatable read** occurs when a transaction reads the same logical row more than once and gets different committed values because another transaction modifies the row between those reads.

Example:

Transaction A:

```text
Read score → 70
```

Transaction B:

```text
Update score → 90
COMMIT
```

Transaction A reads again:

```text
Read score → 90
```

Within Transaction A:

```text
First read  = 70
Second read = 90
```

The same row produced different values.

---

# 14. Non-Repeatable Read Timeline

```text
Transaction A               Transaction B

BEGIN

READ score
→ 70

                            UPDATE score = 90
                            COMMIT

READ score
→ 90

COMMIT
```

Transaction A sees two different committed versions during the same transaction.

---

# 15. Dirty Read vs Non-Repeatable Read

### Dirty Read

Reads:

```text
Uncommitted data
```

from another transaction.

### Non-Repeatable Read

Reads:

```text
Different committed values
```

for the same logical row during one transaction.

These are different anomalies.

---

# 16. Phantom Read

A **phantom read** occurs when repeating a predicate-based query produces a different set of qualifying rows because another transaction inserts, deletes, or changes rows affecting that predicate.

Example:

Transaction A:

```sql
SELECT *
FROM interviews
WHERE score >= 80;
```

returns:

```text
10 rows
```

Transaction B inserts:

```text
score = 95
```

and commits.

Transaction A runs the same query again:

```text
11 rows
```

A new qualifying row has appeared.

That row is conceptually a phantom relative to the earlier result set.

---

# 17. Phantom Read Timeline

```text
Transaction A                  Transaction B

BEGIN

SELECT score >= 80
→ 10 rows

                               INSERT score = 95
                               COMMIT

SELECT score >= 80
→ 11 rows

COMMIT
```

The result set changed.

---

# 18. Non-Repeatable Read vs Phantom Read

Simplified distinction:

### Non-Repeatable Read

A previously read row changes.

```text
Row 10:
score 70 → 90
```

### Phantom Read

The set of rows matching a predicate changes.

```text
10 matching rows → 11 matching rows
```

In practice, standards, implementations, and MVCC behavior can make the boundaries more nuanced.

For interviews, understand the conceptual distinction first.

---

# 19. Lost Update

A **lost update** occurs when concurrent operations read the same state, compute updates independently, and one update overwrites the effect of another.

Initial value:

```text
credits = 10
```

Transaction A:

```text
Read 10
Subtract 1
Plan to write 9
```

Transaction B:

```text
Read 10
Subtract 1
Plan to write 9
```

Then:

```text
A writes 9
B writes 9
```

Final result:

```text
credits = 9
```

But two credits were consumed.

Expected:

```text
credits = 8
```

One update was lost.

---

# 20. Lost Update Timeline

```text
Transaction A              Transaction B

READ credits = 10

                           READ credits = 10

calculate 9

                           calculate 9

WRITE 9

                           WRITE 9
```

Final:

```text
9
```

Expected:

```text
8
```

This is a classic read-modify-write race.

---

# 21. Avoiding Lost Updates With Atomic Operations

Instead of:

```text
Read credits
Calculate credits - 1
Write new value
```

perform the state transition directly:

```sql
UPDATE candidate_credits
SET credits = credits - 1
WHERE candidate_id = 100
  AND credits > 0;
```

The database performs the update against the row under its concurrency-control rules.

The application can inspect the affected-row count.

Conceptually:

```text
UPDATE succeeded → credit consumed

UPDATE affected 0 rows → no available credit
```

This can be significantly safer than application-side read-modify-write logic.

---

# 22. Write Skew

**Write skew** is a concurrency anomaly where transactions read overlapping state but modify different rows, allowing a shared invariant to be violated.

Suppose a hospital requires:

```text
At least one doctor must remain on call.
```

Current state:

```text
Doctor A = on call
Doctor B = on call
```

Transaction A:

```text
Checks B is on call
Sets A off call
```

Transaction B:

```text
Checks A is on call
Sets B off call
```

If both decisions are made from compatible snapshots:

```text
A → off
B → off
```

Final result:

```text
No doctor on call
```

Each transaction updated a different row, so simple same-row conflict detection may not be enough.

---

# 23. Why Write Skew Matters

Write skew demonstrates that:

```text
No lost update
```

does not necessarily mean:

```text
No concurrency bug
```

The invariant spans multiple rows:

```text
At least one doctor remains on call.
```

Protecting such invariants may require stronger isolation, explicit locking, schema redesign, or other concurrency-control mechanisms.

---

# 24. SQL Isolation Levels

The SQL standard commonly describes isolation levels such as:

```text
READ UNCOMMITTED

READ COMMITTED

REPEATABLE READ

SERIALIZABLE
```

A useful conceptual progression is:

```text
Weaker isolation
      ↓
Potentially more concurrency anomalies

Stronger isolation
      ↓
Fewer allowed anomalies
```

However, real DBMS behavior differs.

Do not assume that two databases implementing:

```text
REPEATABLE READ
```

provide identical semantics.

---

# 25. READ UNCOMMITTED

Conceptually, `READ UNCOMMITTED` provides weak isolation.

It may permit transactions to observe changes that have not yet committed.

This can allow:

```text
Dirty reads
```

along with other anomalies.

However, some databases do not actually expose dirty reads even when this isolation-level name is requested.

Always check the specific DBMS.

---

# 26. READ COMMITTED

`READ COMMITTED` generally prevents dirty reads.

A transaction reads committed data.

However, depending on the implementation, separate statements within the same transaction may observe different committed states.

Conceptually:

```text
Statement 1 → committed snapshot/state A

Another transaction commits

Statement 2 → committed snapshot/state B
```

Therefore non-repeatable reads and changing predicate results may be possible.

---

# 27. READ COMMITTED Example

Transaction A:

```text
BEGIN

Read score → 70
```

Transaction B:

```text
Update score → 90
COMMIT
```

Transaction A:

```text
Read score again → 90
```

This behavior can occur under implementations where each statement sees the committed state appropriate to that statement.

Exact semantics depend on the DBMS.

---

# 28. REPEATABLE READ

`REPEATABLE READ` provides stronger guarantees than `READ COMMITTED`.

A common conceptual goal is:

```text
Rows read during the transaction should remain stable for repeated reads.
```

Many MVCC databases accomplish this through a transaction-level snapshot or related mechanisms.

But phantom behavior and other anomalies vary significantly between database systems.

Do not memorize one universal behavior for every DBMS.

---

# 29. SERIALIZABLE

`SERIALIZABLE` aims to provide behavior equivalent to serial transaction execution.

Conceptually:

```text
Concurrent execution
        ↓
Result equivalent to
        ↓
Some serial order
```

This provides strong protection against concurrency anomalies.

But it may require:

```text
Blocking

Locking

Conflict detection

Transaction aborts

Retries
```

depending on the implementation.

---

# 30. Isolation Level Comparison

A simplified conceptual table:

| Isolation Level | Dirty Read | Non-Repeatable Read | Phantom Read |
|---|---|---|---|
| READ UNCOMMITTED | May occur | May occur | May occur |
| READ COMMITTED | Prevented | May occur | May occur |
| REPEATABLE READ | Prevented | Prevented | DBMS-dependent |
| SERIALIZABLE | Prevented | Prevented | Prevented under serializable semantics |

This table is useful for interviews, but it is not a substitute for understanding the actual database.

Real implementations differ.

---

# 31. Stronger Isolation Is Not Automatically Better

Suppose every transaction runs at:

```text
SERIALIZABLE
```

Correctness reasoning may become easier for certain workflows.

But depending on the database and workload, stronger isolation can increase:

```text
Blocking

Transaction conflicts

Serialization failures

Retries

Latency
```

The correct isolation level depends on the invariant and workload.

Use the weakest level that still provides the required correctness when combined with the rest of the design.

---

# 32. What Is a Lock?

A **lock** is a concurrency-control mechanism used to coordinate access to database resources.

Conceptually:

```text
Transaction A
      ↓
Acquire lock
      ↓
Access resource
      ↓
Release lock
```

If another transaction requests an incompatible lock:

```text
Transaction B
      ↓
Wait / fail / retry
```

depending on the DBMS and operation.

---

# 33. Shared Lock

A **shared lock** is conceptually associated with reading.

Multiple transactions may be able to hold compatible shared locks on the same resource.

Conceptually:

```text
Transaction A → Shared Lock ┐
                            ├→ Row
Transaction B → Shared Lock ┘
```

Both can read.

Exact lock behavior depends on the database and isolation model.

MVCC systems may not require traditional shared row locks for ordinary reads.

---

# 34. Exclusive Lock

An **exclusive lock** is conceptually associated with modification.

If Transaction A holds an exclusive lock on a resource:

```text
Transaction A → Exclusive Lock → Row
```

another transaction requesting an incompatible lock may need to wait.

This prevents conflicting modifications from proceeding simultaneously in unsafe ways.

---

# 35. Lock Compatibility

A simplified conceptual model:

```text
Shared + Shared
→ Often compatible

Shared + Exclusive
→ Usually incompatible

Exclusive + Exclusive
→ Incompatible
```

Real databases support many lock modes beyond these basic categories.

Examples may include:

```text
Intent locks
Update locks
Predicate locks
Key-range locks
Metadata locks
```

depending on the DBMS.

---

# 36. Lock Granularity

Locks may operate at different granularities:

```text
Database
Table
Page
Row
Key
Key range
Metadata object
```

Coarser locks:

```text
Less lock-management overhead
+
Potentially less concurrency
```

Finer locks:

```text
More concurrency
+
Potentially more lock-management overhead
```

Database systems choose and expose locking strategies differently.

---

# 37. SELECT FOR UPDATE

When an application needs to read a row and then safely modify it, some relational databases provide:

```sql
SELECT ...
FOR UPDATE;
```

Example:

```sql
BEGIN;

SELECT credits
FROM candidate_credits
WHERE candidate_id = 100
FOR UPDATE;

-- Validate credits

UPDATE candidate_credits
SET credits = credits - 1
WHERE candidate_id = 100;

COMMIT;
```

The selected row is locked for an update-oriented workflow according to the DBMS semantics.

---

# 38. SELECT FOR UPDATE Example

Suppose:

```text
credits = 1
```

Transaction A:

```text
SELECT ... FOR UPDATE
```

acquires the relevant lock.

Transaction B attempts the same operation.

Conceptually:

```text
A holds lock
      ↓
B waits
      ↓
A updates credits to 0
      ↓
A commits
      ↓
B continues
      ↓
B now observes 0
```

This can prevent both transactions from independently consuming the same credit.

---

# 39. SELECT FOR UPDATE Is Not Always Necessary

Suppose the operation can be expressed as:

```sql
UPDATE candidate_credits
SET credits = credits - 1
WHERE candidate_id = 100
  AND credits > 0;
```

Then an explicit:

```sql
SELECT ... FOR UPDATE
```

may not be necessary.

Prefer a single atomic state transition when it cleanly expresses the business rule.

Use read-then-lock workflows when the application genuinely needs them.

---

# 40. Pessimistic Concurrency Control

**Pessimistic concurrency control** assumes conflicts are sufficiently likely or costly that access should be coordinated before making changes.

Conceptually:

```text
Acquire lock
     ↓
Read / modify
     ↓
Commit
     ↓
Release lock
```

Example:

```sql
SELECT ...
FOR UPDATE;
```

Potential advantages:

```text
Conflicts handled before modification

Useful for highly contested resources
```

Potential disadvantages:

```text
Blocking

Deadlocks

Long lock durations

Reduced concurrency
```

---

# 41. Optimistic Concurrency Control

**Optimistic concurrency control** assumes conflicts are relatively uncommon.

Instead of holding a lock throughout the workflow:

```text
Read current version
      ↓
Modify locally
      ↓
Update only if version unchanged
```

If another transaction changed the row first:

```text
Update fails
      ↓
Retry / report conflict
```

---

# 42. Version Column Example

Suppose:

```text
candidate_profiles

id
name
headline
version
```

Current row:

```text
id      = 100
version = 5
```

Application reads version 5.

Then attempts:

```sql
UPDATE candidate_profiles
SET
    headline = 'Backend Engineer',
    version = version + 1
WHERE id = 100
  AND version = 5;
```

If another transaction already changed the row:

```text
version = 6
```

then:

```text
0 rows updated
```

The application detects a concurrency conflict.

---

# 43. Pessimistic vs Optimistic Concurrency

### Pessimistic

```text
Prevent conflicting access first
```

Often uses:

```text
Locks
```

Useful when:

```text
Conflicts are frequent
Conflict cost is high
Critical sections are short
```

### Optimistic

```text
Allow work
Detect conflict before commit/update
```

Often uses:

```text
Version
Timestamp
Conditional UPDATE
```

Useful when:

```text
Conflicts are uncommon
Blocking is undesirable
Retries are acceptable
```

There is no universal winner.

---

# 44. What Is MVCC?

**MVCC** stands for:

```text
Multi-Version Concurrency Control
```

Instead of maintaining only one immediately visible version of a row, an MVCC system can maintain multiple row versions associated with transaction visibility.

Conceptually:

```text
Row Version 1
Row Version 2
Row Version 3
```

Different transactions may observe different versions according to their snapshots and isolation rules.

---

# 45. Why MVCC?

Without versioning, readers and writers might frequently block each other.

MVCC can allow patterns such as:

```text
Reader → old visible version

Writer → creates/modifies newer version
```

Conceptually:

```text
Reader ─────► Version 1

Writer ─────► Version 2
```

The reader may continue without waiting for the writer in many situations.

This can improve read/write concurrency.

---

# 46. MVCC Does Not Mean No Locks

A common misconception:

> MVCC databases do not use locks.

Incorrect.

MVCC reduces some reader/writer conflicts, but databases may still use locks for:

```text
Concurrent writes

DDL

Explicit locking

Foreign-key operations

Unique constraints

Metadata

Other internal coordination
```

MVCC and locking can coexist.

---

# 47. Snapshot

A **snapshot** represents the database state visible to a transaction or statement according to MVCC visibility rules.

Conceptually:

```text
Transaction A snapshot:

Row X → Version 3
Row Y → Version 7
Row Z → Version 2
```

Another transaction may create newer versions, but Transaction A may continue seeing versions appropriate to its snapshot.

Snapshot lifetime depends on the isolation level and DBMS.

---

# 48. Statement-Level vs Transaction-Level Snapshot

A database may provide behavior conceptually similar to:

### Statement-Level Snapshot

Each statement receives a fresh view of committed data.

```text
Statement 1 → Snapshot A

Statement 2 → Snapshot B
```

### Transaction-Level Snapshot

The transaction uses a stable snapshot for multiple operations.

```text
Transaction
├── Statement 1 → Snapshot A
├── Statement 2 → Snapshot A
└── Statement 3 → Snapshot A
```

Exact behavior depends on the database and isolation level.

---

# 49. MVCC Example

Initial committed row:

```text
score = 70
```

Transaction A begins and sees:

```text
Version 1 → score 70
```

Transaction B updates:

```text
score = 90
```

and commits.

Now:

```text
Version 1 → 70
Version 2 → 90
```

Depending on Transaction A's isolation and snapshot rules, it may continue seeing:

```text
70
```

while a newer transaction sees:

```text
90
```

This is the basic idea behind version-based concurrency.

---

# 50. MVCC Trade-Off

Old row versions cannot necessarily be removed immediately while transactions may still need them.

Long-running transactions can therefore interfere with cleanup in some MVCC databases.

Potential effects include:

```text
Version accumulation

Storage growth

Cleanup delays

Higher maintenance overhead
```

Exact behavior depends on the DBMS.

This is another reason long-running transactions deserve attention.

---

# 51. What Is a Deadlock?

A **deadlock** occurs when transactions wait on each other in a cycle and none can proceed.

Example:

Transaction A:

```text
Locks Row 1
Needs Row 2
```

Transaction B:

```text
Locks Row 2
Needs Row 1
```

Now:

```text
A waits for B

B waits for A
```

Neither can continue.

---

# 52. Deadlock Diagram

```text
Transaction A
    │
    │ holds Row 1
    │
    ▼
  Row 1

Transaction A needs Row 2
         ▲
         │
         │ held by
         │
Transaction B


Transaction B
    │
    │ holds Row 2
    │
    ▼
  Row 2

Transaction B needs Row 1
         ▲
         │
         │ held by
         │
Transaction A
```

Cycle:

```text
A waits for B
B waits for A
```

That is a deadlock.

---

# 53. Deadlock Example

Transaction A:

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 100
WHERE id = 1;

-- later

UPDATE accounts
SET balance = balance + 100
WHERE id = 2;
```

Transaction B:

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 50
WHERE id = 2;

-- later

UPDATE accounts
SET balance = balance + 50
WHERE id = 1;
```

Possible sequence:

```text
A locks Account 1

B locks Account 2

A requests Account 2 → waits

B requests Account 1 → waits
```

Deadlock.

---

# 54. How Databases Handle Deadlocks

Many database systems detect deadlocks.

When a cycle is detected:

```text
Choose one transaction
        ↓
Abort it
        ↓
Release its locks
        ↓
Other transaction continues
```

The aborted transaction may receive an error.

The application may need to retry it safely.

---

# 55. Preventing Deadlocks With Consistent Lock Ordering

Suppose every transfer always locks accounts in ascending ID order.

Instead of:

```text
Transfer A:

Lock 10
Lock 20
```

and:

```text
Transfer B:

Lock 20
Lock 10
```

both use:

```text
Lock smaller ID first
Lock larger ID second
```

Then:

```text
Transaction A:
Lock 10
Lock 20

Transaction B:
Lock 10
Lock 20
```

This can eliminate that particular lock-order cycle.

Consistent lock ordering is an important deadlock-reduction technique.

---

# 56. Deadlocks Cannot Always Be Eliminated

Even carefully designed systems can encounter deadlocks.

Therefore production applications should often be prepared for:

```text
Deadlock error
      ↓
Rollback
      ↓
Retry when safe
```

Retries should generally include:

```text
Limited attempts
Backoff
Idempotent operation design
Observability
```

Do not implement infinite immediate retry loops.

---

# 57. Deadlock vs Blocking

These are different.

### Blocking

Transaction B waits for Transaction A.

```text
A holds resource

B waits

A commits

B continues
```

This can be normal.

### Deadlock

A waits for B while B waits for A, directly or through a larger cycle.

```text
A waits for B

B waits for A
```

No transaction in the cycle can naturally proceed.

---

# 58. Deadlock vs Starvation

### Deadlock

Transactions are stuck in a cyclic wait.

### Starvation

A transaction repeatedly fails to obtain the resources or scheduling opportunity needed to make progress because other work keeps winning.

They are distinct concurrency problems.

---

# 59. Lock Timeout

Some databases or applications use lock timeouts.

Conceptually:

```text
Wait for lock
     ↓
Timeout exceeded
     ↓
Abort / return error
```

A timeout prevents indefinite waiting.

But a timeout does not solve the root cause of excessive contention.

Investigate:

```text
Long transactions
Hot rows
Poor lock ordering
Large updates
Missing indexes
Workload design
```

---

# 60. Hot Rows

A **hot row** is a row that many concurrent transactions attempt to update.

Example:

```text
global_counter
```

where thousands of requests perform:

```sql
UPDATE counters
SET value = value + 1
WHERE id = 1;
```

All requests contend for the same logical row.

Even if the SQL is correct, throughput may be limited by contention.

---

# 61. Hot Row Example: Credits

Per-candidate credits:

```text
candidate 100 → credits
candidate 101 → credits
candidate 102 → credits
```

distribute updates across many rows.

But a single global row:

```text
total_interviews_started
```

updated on every request can become a contention point.

Schema and architecture influence concurrency.

---

# 62. Constraints as Concurrency Tools

Concurrency correctness should not rely only on application checks.

Suppose usernames must be unique.

Unsafe approach:

```text
SELECT username

if not exists:
    INSERT username
```

Two concurrent requests may both observe:

```text
username does not exist
```

and both attempt insertion.

Use a database constraint:

```sql
UNIQUE (username)
```

Then handle the conflict.

The database becomes the final authority for the invariant.

---

# 63. Check-Then-Act Race

This pattern is dangerous:

```text
Check condition
      ↓
Time passes
      ↓
Act based on old condition
```

Example:

```text
Check seat available
      ↓
Another transaction books seat
      ↓
Book seat
```

This is called a:

```text
check-then-act race
```

Possible solutions include:

```text
Atomic conditional operation

Locking

Unique constraint

Serializable transaction

Optimistic concurrency
```

depending on the problem.

---

# 64. Atomic Reservation Example

Suppose:

```text
interview_slots

id
status
```

To reserve an available slot:

```sql
UPDATE interview_slots
SET status = 'reserved'
WHERE id = 100
  AND status = 'available';
```

Then check affected rows.

If:

```text
1 row updated
```

reservation succeeded.

If:

```text
0 rows updated
```

someone else may already have reserved it.

This avoids a separate:

```text
SELECT
then
UPDATE
```

race.

---

# 65. Unique Constraint Example

Suppose only one booking is allowed per slot:

```sql
CREATE TABLE bookings (
    id BIGINT PRIMARY KEY,
    slot_id BIGINT NOT NULL UNIQUE,
    candidate_id BIGINT NOT NULL
);
```

Concurrent requests may both try:

```sql
INSERT INTO bookings (...)
```

but the database uniqueness constraint prevents both from committing the same:

```text
slot_id
```

This is often stronger than application-side:

```text
Does booking already exist?
```

checks.

---

# 66. Isolation Does Not Replace Constraints

Even with strong transaction isolation, schema constraints remain important.

Use constraints for invariants naturally expressible as:

```text
UNIQUE

FOREIGN KEY

CHECK

NOT NULL
```

Concurrency control and constraints solve related but different problems.

A robust system often uses both.

---

# 67. Read-Modify-Write Pattern

Common application logic:

```text
1. Read current value
2. Modify in application
3. Write new value
```

Example:

```text
credits = SELECT credits

credits = credits - 1

UPDATE credits = new_value
```

Under concurrency, this can create lost updates.

Prefer:

```sql
UPDATE candidate_credits
SET credits = credits - 1
WHERE candidate_id = 100
  AND credits > 0;
```

when the state transition can be expressed directly.

---

# 68. Optimistic Update Pattern

Suppose the application must edit a complex profile.

Read:

```text
id = 100
version = 7
```

Then:

```sql
UPDATE candidate_profiles
SET
    name = 'Alex',
    headline = 'Backend Engineer',
    version = version + 1
WHERE id = 100
  AND version = 7;
```

If:

```text
affected rows = 0
```

the record changed since it was read.

Then:

```text
Reload
Retry
Merge
or
Report conflict
```

depending on the product behavior.

---

# 69. Isolation and Performance

Isolation is a correctness decision, but it has performance implications.

Potential effects of stronger isolation include:

```text
More blocking

More lock retention

More conflict detection

More aborted transactions

More retries
```

Potential effects of weaker isolation include:

```text
Higher concurrency
+
More responsibility on application design
```

The goal is not:

```text
Maximum isolation
```

The goal is:

```text
Required correctness
+
Acceptable performance
```

---

# 70. Choosing an Isolation Level

Ask:

```text
What invariant must hold?

What concurrent operations can occur?

Which anomalies would violate correctness?

Can atomic statements solve the problem?

Can constraints enforce the invariant?

Do we need explicit locks?

Can optimistic retries handle conflicts?

Would serializable isolation simplify the workflow?
```

Choose isolation based on the answers.

Do not choose isolation based only on habit.

---

# 71. Example: Interview Credits

Requirement:

```text
A candidate cannot start an interview without available credit.
```

Naive:

```text
SELECT credits

if credits > 0:
    UPDATE credits
```

Race possible.

Better candidate solution:

```sql
UPDATE candidate_credits
SET credits = credits - 1
WHERE candidate_id = ?
  AND credits > 0;
```

Check:

```text
affected rows = 1
```

before considering the credit consumed.

If starting the interview requires multiple database changes, place the required state changes inside an appropriate transaction.

---

# 72. Example: Money Transfer

Requirement:

```text
Transfer $100 from A to B.
```

A robust design must consider:

```text
Atomicity

Concurrent transfers

Balance validation

Lock ordering

Constraints

Deadlock handling

Retries

Ledger/history
```

A possible approach:

```text
BEGIN

Lock accounts in deterministic order

Validate source balance

Debit source

Credit destination

Create transfer record

COMMIT
```

The exact SQL and locking strategy depend on the database.

---

# 73. Example: Last Available Seat

Suppose:

```text
seat 42 = available
```

Two users attempt booking simultaneously.

Unsafe:

```text
SELECT status

if available:
    UPDATE status = booked
```

Both may read:

```text
available
```

Better:

```sql
UPDATE seats
SET
    status = 'booked',
    candidate_id = ?
WHERE id = 42
  AND status = 'available';
```

Only one transaction should successfully transition the row from:

```text
available
```

to:

```text
booked
```

under normal transactional semantics.

---

# 74. Example: Duplicate Registration

Unsafe:

```text
SELECT email

if not found:
    INSERT user
```

Two requests can race.

Better:

```sql
UNIQUE(email)
```

Then:

```text
INSERT
```

and handle uniqueness violation.

Principle:

> Let the database enforce invariants that the database can express reliably.

---

# 75. Example: Concurrent Profile Editing

Two admins open:

```text
Candidate Profile version 3
```

Admin A changes:

```text
Name
```

Admin B changes:

```text
Headline
```

If both write the entire old object, one update may overwrite the other.

Optimistic concurrency:

```sql
UPDATE candidate_profiles
SET
    ...,
    version = version + 1
WHERE id = ?
  AND version = 3;
```

Only the first compatible update succeeds.

The second detects the conflict and can reload or merge.

---

# 76. Isolation Interview Trap

Question:

> Does REPEATABLE READ prevent phantom reads?

Do not immediately answer:

```text
Yes
```

or:

```text
No
```

A stronger answer is:

> The SQL isolation-level model distinguishes repeatable reads from phantom protection, but actual REPEATABLE READ semantics vary across database systems. For example, MVCC and locking implementations can provide stronger or different behavior than the simplified standard table suggests. I would specify the DBMS before making an implementation-level claim.

This demonstrates actual understanding.

---

# 77. PostgreSQL Example

PostgreSQL uses MVCC.

At a high level:

```text
READ COMMITTED
```

typically uses a new snapshot for each statement.

```text
REPEATABLE READ
```

uses a stable transaction snapshot and provides snapshot-isolation-like behavior.

```text
SERIALIZABLE
```

adds mechanisms designed to detect serialization anomalies and may abort transactions that must be retried.

Exact behavior depends on PostgreSQL's implementation and version.

---

# 78. MySQL/InnoDB Example

MySQL's InnoDB storage engine also uses MVCC and locking.

Its default isolation level has historically commonly been:

```text
REPEATABLE READ
```

and InnoDB can use mechanisms such as:

```text
Record locks

Gap locks

Next-key locks
```

for certain operations and isolation levels.

Its behavior should not be assumed identical to PostgreSQL merely because both use the same isolation-level names.

---

# 79. SQL Server Example

SQL Server supports lock-based concurrency and optional row-versioning-based isolation features.

Depending on configuration, behaviors associated with:

```text
READ COMMITTED
```

can differ.

This reinforces the rule:

> Isolation-level names alone are not enough to understand production behavior.

Know the DBMS.

---

# 80. Common Interview Question: What Is a Dirty Read?

Strong answer:

> A dirty read occurs when a transaction reads another transaction's uncommitted changes. If the writer later rolls back, the reader has observed data that never became committed state.

---

# 81. Common Interview Question: What Is a Non-Repeatable Read?

Strong answer:

> A non-repeatable read occurs when a transaction reads the same logical row multiple times and sees different committed values because another transaction modified that row between the reads.

---

# 82. Common Interview Question: What Is a Phantom Read?

Strong answer:

> A phantom read occurs when repeating a predicate-based query within a transaction returns a different set of qualifying rows because another transaction changed the set of rows satisfying that predicate.

---

# 83. Common Interview Question: What Is a Lost Update?

Strong answer:

> A lost update occurs when concurrent transactions derive new values from the same earlier state and one write overwrites the effect of another. It commonly appears in application-side read-modify-write logic and can often be prevented with atomic updates, locking, optimistic version checks, or stronger concurrency control.

---

# 84. Common Interview Question: What Is MVCC?

Strong answer:

> Multi-Version Concurrency Control allows a database to maintain multiple versions of data so transactions can read versions appropriate to their visibility rules while concurrent updates occur. This can reduce reader-writer blocking, although MVCC databases still use locks and other synchronization mechanisms where necessary.

---

# 85. Common Interview Question: Optimistic vs Pessimistic Locking

A concise comparison:

### Pessimistic

```text
Assume conflict may occur
Lock before modification
Other transactions may wait
```

### Optimistic

```text
Assume conflict is uncommon
Perform work without holding the conflicting lock
Detect stale state during update
Retry or reject on conflict
```

Choose based on:

```text
Conflict frequency
Transaction duration
Retry cost
Contention
Business requirements
```

---

# 86. Common Interview Question: What Is a Deadlock?

Strong answer:

> A deadlock occurs when transactions form a cyclic dependency while waiting for resources held by one another. Databases commonly resolve it by aborting one transaction, so applications should be prepared to handle and safely retry appropriate deadlock failures.

---

# 87. Common Interview Question: How Do You Prevent Deadlocks?

Possible strategies include:

```text
Consistent lock ordering

Short transactions

Avoid unnecessary locks

Index queries appropriately

Reduce hot-resource contention

Access resources predictably
```

But do not claim deadlocks can always be completely prevented.

Production systems should also handle them correctly.

---

# 88. Common Interview Question: Does MVCC Eliminate Deadlocks?

No.

MVCC can reduce some reader-writer blocking, but write-write conflicts and explicit locking can still produce deadlocks.

Example:

```text
Transaction A locks Row 1
Transaction B locks Row 2

A needs Row 2
B needs Row 1
```

MVCC does not make this cycle disappear.

---

# 89. Common Interview Question: Transaction vs Lock

### Transaction

Defines a logical unit of work.

```text
BEGIN
...
COMMIT
```

### Lock

Coordinates concurrent access to a resource.

A transaction may hold multiple locks during its lifetime.

They are related but not interchangeable concepts.

---

# 90. Common Interview Questions

Be prepared to answer:

1. What is concurrency in DBMS?
2. Why do databases support concurrency?
3. What is a race condition?
4. Does using a transaction eliminate race conditions?
5. What is isolation?
6. What is serial execution?
7. What is serializability?
8. What is a dirty read?
9. What is a non-repeatable read?
10. What is a phantom read?
11. What is a lost update?
12. What is write skew?
13. What are SQL isolation levels?
14. What is READ UNCOMMITTED?
15. What is READ COMMITTED?
16. What is REPEATABLE READ?
17. What is SERIALIZABLE?
18. Why not always use SERIALIZABLE?
19. What is a lock?
20. Shared vs exclusive lock?
21. What is lock granularity?
22. What is SELECT FOR UPDATE?
23. What is pessimistic concurrency control?
24. What is optimistic concurrency control?
25. What is a version column?
26. What is MVCC?
27. Why does MVCC improve concurrency?
28. Does MVCC eliminate locks?
29. What is a snapshot?
30. What is a deadlock?
31. Deadlock vs blocking?
32. Deadlock vs starvation?
33. How are deadlocks resolved?
34. How can deadlocks be reduced?
35. What is a hot row?
36. What is a check-then-act race?
37. How can constraints help with concurrency?
38. How would you safely deduct a limited credit?
39. How would you prevent double booking?
40. How would you handle concurrent profile updates?

---

# 91. Common Mistakes

## Mistake 1: Transactions automatically prevent race conditions

Incorrect.

Transactions need appropriate isolation and concurrency-control mechanisms.

---

## Mistake 2: Isolation means only one transaction executes at a time

Incorrect.

Transactions can execute concurrently while preserving isolation guarantees.

---

## Mistake 3: READ COMMITTED means all reads in a transaction see the same snapshot

Not universally.

In some implementations, each statement can see a newer committed state.

---

## Mistake 4: REPEATABLE READ means identical behavior across databases

Incorrect.

Implementation semantics vary significantly.

---

## Mistake 5: SERIALIZABLE means transactions literally run one by one

Incorrect.

Concurrent execution is possible as long as the result satisfies serializable semantics.

---

## Mistake 6: MVCC means no locking

Incorrect.

MVCC and locks can coexist.

---

## Mistake 7: SELECT then UPDATE is always safe inside a transaction

Incorrect.

Another transaction may modify the state between the logical check and action depending on isolation and locking.

---

## Mistake 8: Deadlock and blocking are the same

Incorrect.

Blocking may resolve naturally when the lock holder completes.

Deadlock contains a cyclic wait.

---

## Mistake 9: Deadlocks mean the database is broken

Incorrect.

Deadlocks are a normal possibility in concurrent systems.

The database and application should detect and handle them.

---

## Mistake 10: Application validation is enough for uniqueness

Incorrect under concurrency.

Use database uniqueness constraints when uniqueness is a database invariant.

---

# 92. Concurrency Problem-Solving Framework

When an interview presents a concurrency problem, reason in this order:

```text
1. Identify shared mutable state
        ↓
2. Identify the invariant
        ↓
3. Identify concurrent operations
        ↓
4. Construct a failing interleaving
        ↓
5. Determine the anomaly
        ↓
6. Check whether a constraint can enforce the rule
        ↓
7. Check whether one atomic statement can solve it
        ↓
8. Consider optimistic or pessimistic control
        ↓
9. Determine required isolation
        ↓
10. Design failure/retry handling
```

This is much stronger than immediately saying:

```text
Use a lock.
```

---

# 93. Interview Scenario: One Credit, Two Requests

Initial:

```text
credits = 1
```

Two requests:

```text
A → Start interview
B → Start interview
```

Unsafe:

```text
A reads 1
B reads 1

A allows interview
B allows interview

A writes 0
B writes 0
```

Correctness requirement:

```text
At most one request can consume the final credit.
```

Possible solution:

```sql
UPDATE candidate_credits
SET credits = credits - 1
WHERE candidate_id = ?
  AND credits > 0;
```

Then:

```text
affected rows = 1
    ↓
Credit acquired

affected rows = 0
    ↓
No credit available
```

If other state changes must happen atomically with credit consumption, include them in the appropriate transaction.

---

# 94. Interview Scenario: Booking the Last Slot

Requirement:

```text
Only one candidate can reserve slot 500.
```

Schema-level protection:

```sql
UNIQUE(slot_id)
```

Then two concurrent inserts cannot both successfully commit the same slot reservation.

Alternative state-transition design:

```sql
UPDATE interview_slots
SET
    status = 'reserved',
    candidate_id = ?
WHERE id = 500
  AND status = 'available';
```

Check affected rows.

The best design depends on the schema and business model.

---

# 95. Interview Scenario: Two Account Transfers

Transaction A:

```text
Transfer from Account 10 → 20
```

Transaction B:

```text
Transfer from Account 20 → 10
```

If each transaction locks its source first:

```text
A locks 10
B locks 20

A needs 20
B needs 10
```

Deadlock.

Better:

```text
Always lock lower account ID first.
```

Both attempt:

```text
Lock 10
then
Lock 20
```

One waits before creating a cycle.

This reduces deadlock risk.

---

# 96. Interview Scenario: Concurrent Resume Editing

Two sessions read:

```text
resume.version = 12
```

Both edit.

Use:

```sql
UPDATE resumes
SET
    content = ?,
    version = version + 1
WHERE id = ?
  AND version = 12;
```

First update:

```text
version 12 → 13
```

Second update:

```text
WHERE version = 12
```

matches zero rows.

Conflict detected.

The application can:

```text
Reload
Compare
Merge
Retry
Ask user to resolve conflict
```

depending on product requirements.

---

# 97. Interview Scenario: Reporting Query

Suppose a reporting transaction performs several queries and requires a stable view of data throughout the report.

Under statement-level committed snapshots:

```text
Query 1 → database state A

Concurrent commits occur

Query 2 → database state B
```

The report may combine observations from different points in time.

A transaction-level snapshot or stronger isolation may be appropriate depending on the correctness requirement.

The important question is:

> Does the report require each statement to see the latest committed data, or one internally consistent snapshot?

---

# 98. Choosing a Concurrency Strategy

A practical mental model:

```text
Can a database constraint enforce it?
        │
        ├── Yes → Use constraint
        │
        ▼
Can one atomic SQL statement express it?
        │
        ├── Yes → Prefer atomic operation
        │
        ▼
Is conflict rare?
        │
        ├── Yes → Consider optimistic concurrency
        │
        ▼
Must state be protected before acting?
        │
        ├── Yes → Consider pessimistic locking
        │
        ▼
Does the invariant span complex reads/writes?
        │
        └── Consider stronger isolation / serializable design
```

These mechanisms can also be combined.

---

# 99. Production Checklist

For concurrency-sensitive workflows, verify:

```text
What state is shared?

What invariant must never break?

Can multiple requests reach this code concurrently?

Is there a check-then-act sequence?

Can the operation be made atomic?

Are database constraints present?

What isolation level is used?

Are explicit locks necessary?

Can transactions deadlock?

Are deadlock/serialization failures retried?

Are retries idempotent?

Are transactions kept short?

Are contention and failures observable?
```

Concurrency bugs are usually architectural problems, not syntax problems.

---

# 100. Quick Revision

Remember the anomalies:

```text
Dirty Read
→ Read uncommitted data

Non-Repeatable Read
→ Same row, different committed value

Phantom Read
→ Predicate returns different row set

Lost Update
→ One concurrent update overwrites another

Write Skew
→ Different-row writes violate shared invariant
```

Remember the main tools:

```text
Atomic SQL operations

Constraints

Isolation levels

Locks

Optimistic concurrency

MVCC

Serializable transactions

Retries
```

And remember:

```text
Transaction ≠ no race conditions

MVCC ≠ no locks

Blocking ≠ deadlock

REPEATABLE READ ≠ identical semantics everywhere

SERIALIZABLE ≠ physically sequential execution
```

---

# 101. Final Takeaway

Concurrency control is about preserving correctness while allowing useful parallel work.

The wrong mental model is:

```text
Concurrency problem
      ↓
Add transaction
      ↓
Solved
```

The better model is:

```text
Define invariant
      ↓
Identify concurrent operations
      ↓
Find possible race
      ↓
Choose database guarantee
      ↓
Use constraint / atomic update / lock / version / isolation
      ↓
Handle conflicts and retries
      ↓
Measure contention
```

The strongest engineers do not begin by asking:

```text
Which isolation level should I use?
```

They begin with:

```text
What must remain true even when multiple transactions execute concurrently?
```

Once that invariant is clear, the correct concurrency-control strategy becomes much easier to design.

---

## Previous

**[← Transactions and ACID](./06-transactions-and-acid.md)**

## Next

**[Deadlocks →](./08-deadlocks.md)**

---

Maintained by **InterviewEraHQ**.
