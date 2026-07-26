# Deadlocks in DBMS

Deadlocks are one of the most important concurrency topics in database interviews.

They appear when transactions compete for resources and create a circular dependency in which none of the involved transactions can continue.

A strong candidate should understand:

- What a deadlock is
- Deadlock vs normal blocking
- How deadlocks form
- Coffman conditions
- Wait-for graphs
- Deadlock detection
- Deadlock prevention
- Deadlock avoidance
- Victim selection
- Transaction rollback
- Lock ordering
- Lock duration
- Lock granularity
- Timeouts
- Retry strategies
- Idempotency
- How indexes can affect deadlocks
- How to diagnose production deadlocks

The central idea is:

> A deadlock is not simply one transaction waiting for another. It is a cyclic dependency where every participant is waiting for another participant in the cycle.

---

# 1. What Is a Deadlock?

A **deadlock** occurs when two or more transactions wait indefinitely for resources held by one another.

Example:

```text
Transaction A
holds Resource 1
needs Resource 2

Transaction B
holds Resource 2
needs Resource 1
```

Now:

```text
A waits for B
B waits for A
```

Neither transaction can proceed.

Conceptually:

```text
A → waits for → B
↑               ↓
└──── waits for ┘
```

That cycle is the defining characteristic of a deadlock.

---

# 2. Simple Database Example

Suppose we have two accounts:

```text
Account 1
Account 2
```

Transaction A updates Account 1 first:

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 100
WHERE id = 1;
```

Transaction B updates Account 2 first:

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 50
WHERE id = 2;
```

At this point:

```text
Transaction A holds a lock related to Account 1

Transaction B holds a lock related to Account 2
```

Now Transaction A tries:

```sql
UPDATE accounts
SET balance = balance + 100
WHERE id = 2;
```

but Account 2 is locked by B.

So:

```text
A waits for B
```

Then B tries:

```sql
UPDATE accounts
SET balance = balance + 50
WHERE id = 1;
```

but Account 1 is locked by A.

Now:

```text
B waits for A
```

We have:

```text
A → B
B → A
```

Deadlock.

---

# 3. Deadlock Timeline

```text
Transaction A                 Transaction B

BEGIN                         BEGIN

Lock Account 1

                              Lock Account 2

Request Account 2
→ WAIT

                              Request Account 1
                              → WAIT
```

Current state:

```text
A holds Account 1
A waits for Account 2

B holds Account 2
B waits for Account 1
```

Neither transaction can proceed without intervention.

---

# 4. Deadlock vs Blocking

This distinction is fundamental.

## Blocking

Suppose:

```text
Transaction A holds Row 1

Transaction B needs Row 1
```

Then:

```text
B waits
```

Eventually:

```text
A commits
```

and releases the lock.

Then:

```text
B continues
```

This is ordinary blocking.

---

## Deadlock

Now suppose:

```text
A holds Row 1
A needs Row 2

B holds Row 2
B needs Row 1
```

Then:

```text
A waits for B
B waits for A
```

Neither can naturally finish.

That is a deadlock.

---

# 5. Visual Difference

Blocking:

```text
B
│
│ waits for
▼
A
│
│ eventually commits
▼
B continues
```

Deadlock:

```text
A ─── waits for ───► B
▲                    │
│                    │
└──── waits for ─────┘
```

The second structure contains a cycle.

---

# 6. Deadlock vs Lock Wait

A lock wait does not automatically mean deadlock.

Example:

```text
Transaction A
    ↓
holds lock

Transaction B
    ↓
waits
```

If A can continue independently:

```text
A commits
    ↓
lock released
    ↓
B continues
```

there is no deadlock.

A deadlock requires a cyclic dependency.

---

# 7. Deadlock vs Starvation

Another important distinction:

### Deadlock

Transactions cannot proceed because they form a cyclic dependency.

```text
A waits for B
B waits for A
```

### Starvation

A transaction repeatedly fails to obtain the resources or scheduling opportunity required to make progress.

Example:

```text
Transaction A repeatedly loses access
while other transactions keep succeeding.
```

Starvation does not require a circular wait.

---

# 8. Deadlock vs Livelock

### Deadlock

Participants are stuck waiting.

```text
A waits
B waits
```

### Livelock

Participants continue changing behavior but still make no useful progress.

Conceptually:

```text
A backs off for B

B backs off for A

Both retry

Both conflict again

Repeat
```

The system is active but not progressing.

Livelock is less commonly discussed in basic DBMS interviews but is useful in broader concurrency discussions.

---

# 9. Why Do Deadlocks Happen?

Deadlocks commonly arise because transactions:

```text
Acquire multiple resources

Acquire them in different orders

Hold some resources while requesting others
```

Example:

```text
Transaction A:
Lock X
Lock Y

Transaction B:
Lock Y
Lock X
```

This creates the possibility of:

```text
A owns X
B owns Y

A waits for Y
B waits for X
```

---

# 10. Coffman Conditions

Four conditions are classically associated with deadlock.

They are called the **Coffman conditions**:

```text
1. Mutual Exclusion

2. Hold and Wait

3. No Preemption

4. Circular Wait
```

A deadlock requires all four conditions to hold simultaneously in the classical model.

Breaking at least one of these conditions prevents deadlock under that model.

---

# 11. Mutual Exclusion

At least one resource must be non-shareable in the conflicting mode.

Conceptually:

```text
Transaction A
     ↓
Exclusive access
     ↓
Resource X
```

Transaction B cannot simultaneously acquire an incompatible lock on the same resource.

Database write locks commonly create this type of exclusivity.

---

# 12. Hold and Wait

A transaction holds one resource while waiting for another.

Example:

```text
Transaction A:

holds Row 1
waits for Row 2
```

This is:

```text
Hold
+
Wait
```

If transactions never held resources while requesting additional resources, this condition would be broken.

But enforcing that universally would often be impractical.

---

# 13. No Preemption

Resources cannot simply be forcibly taken away from a transaction while preserving normal execution.

Conceptually:

```text
A owns lock X
```

The system does not normally say:

```text
Give lock X to B
and let A continue as if nothing happened.
```

Instead, databases may resolve a deadlock by aborting a transaction.

After rollback:

```text
locks are released
```

This is different from transparently stealing the lock while keeping the transaction intact.

---

# 14. Circular Wait

There is a cycle of transactions waiting for one another.

Example:

```text
A waits for B

B waits for C

C waits for A
```

Graphically:

```text
A → B → C
↑       ↓
└───────┘
```

This condition directly captures the deadlock cycle.

---

# 15. Coffman Conditions Summary

Deadlock requires:

```text
Mutual Exclusion
       +
Hold and Wait
       +
No Preemption
       +
Circular Wait
```

Conceptually:

```text
All four present
      ↓
Deadlock becomes possible
```

A prevention strategy attempts to ensure at least one condition cannot occur.

---

# 16. Two-Transaction Deadlock

The simplest deadlock is:

```text
A waits for B

B waits for A
```

But deadlocks can involve more than two transactions.

Example:

```text
A waits for B

B waits for C

C waits for D

D waits for A
```

The important property is still:

```text
Cycle
```

not the number of participants.

---

# 17. Wait-For Graph

A **wait-for graph** models transaction dependencies.

Each node represents a transaction.

An edge:

```text
A → B
```

means:

```text
Transaction A is waiting for a resource held by Transaction B.
```

Example:

```text
A → B
B → C
```

No cycle exists yet.

If:

```text
C → A
```

appears:

```text
A → B → C
↑       ↓
└───────┘
```

a cycle exists.

That indicates a deadlock in the basic wait-for model.

---

# 18. Wait-For Graph Example

Suppose:

```text
T1 waits for T2

T2 waits for T3

T3 waits for T1
```

Graph:

```text
T1 ───► T2
▲        │
│        ▼
└────── T3
```

Cycle:

```text
T1 → T2 → T3 → T1
```

Therefore:

```text
Deadlock
```

---

# 19. Why Wait-For Graphs Matter

A database can conceptually inspect transaction dependencies:

```text
Who is waiting?

What resource are they waiting for?

Who owns that resource?
```

From those relationships it can build dependency information.

If the dependency graph contains a cycle:

```text
deadlock detected
```

The database can then choose a transaction to abort.

Real implementations may use more specialized algorithms and internal representations.

---

# 20. Deadlock Detection

One strategy is to allow deadlocks to occur and detect them afterward.

Conceptually:

```text
Transactions execute
      ↓
Locks create waits
      ↓
Database examines dependencies
      ↓
Cycle detected
      ↓
Deadlock identified
```

Then the database resolves the deadlock.

This is called:

```text
Deadlock detection
```

---

# 21. Deadlock Resolution

After detecting a deadlock, the system must break the cycle.

Common strategy:

```text
Select one transaction
      ↓
Abort / rollback it
      ↓
Release its locks
      ↓
Other transaction proceeds
```

Example:

```text
A waits for B
B waits for A
```

Database aborts B.

Then:

```text
B releases locks
      ↓
A acquires required lock
      ↓
A continues
```

---

# 22. Deadlock Victim

The transaction chosen for rollback is often called the:

```text
deadlock victim
```

Example:

```text
T1
T2
T3
```

form a deadlock.

Database chooses:

```text
T2
```

Then:

```text
ROLLBACK T2
```

The cycle disappears.

The application running T2 receives an error and may need to retry.

---

# 23. Victim Selection

How does a database choose which transaction to abort?

Possible considerations can include:

```text
Transaction cost

Amount of work already completed

Number of locks held

Rollback cost

Transaction priority

Internal DBMS heuristics
```

Exact behavior is database-specific.

Do not assume:

> The newest transaction is always aborted.

or:

> The smallest transaction is always aborted.

Different systems use different policies.

---

# 24. Deadlock Detection Is Not the Same as Timeout

Suppose:

```text
Transaction B waits 5 seconds
```

and then receives a timeout.

That does not necessarily mean a deadlock occurred.

It could simply be:

```text
Transaction A held a lock for too long.
```

Deadlock detection identifies a cyclic dependency.

Timeout detects:

```text
waiting exceeded some duration
```

They solve different problems.

---

# 25. Lock Timeout

A lock timeout sets a maximum amount of time a transaction will wait for a lock.

Conceptually:

```text
Request lock
    ↓
Resource unavailable
    ↓
Wait
    ↓
Timeout threshold reached
    ↓
Error
```

This prevents indefinite waiting.

But it does not identify whether the wait was caused by:

```text
Deadlock

Long transaction

Slow query

Hot row

Large update
```

---

# 26. Deadlock Timeout

Some systems use timing mechanisms as part of deadlock detection.

Instead of checking every wait immediately, the database may wait for a configured period before running expensive deadlock checks.

This should not be confused with:

```text
Abort every transaction after N seconds.
```

Exact behavior is DBMS-specific.

---

# 27. Deadlock Prevention

**Deadlock prevention** designs resource acquisition rules so that at least one necessary deadlock condition cannot occur.

Possible approaches include:

```text
Consistent lock ordering

Acquire required locks in a predefined order

Avoid holding locks while requesting unrelated resources

Reduce transaction scope
```

The most practical database technique is often:

```text
Consistent resource ordering
```

---

# 28. Consistent Lock Ordering

Suppose transactions need to lock accounts:

```text
10
20
```

Unsafe strategy:

```text
Transfer A:
Lock source
Lock destination

Transfer B:
Lock source
Lock destination
```

For opposite-direction transfers:

```text
A locks 10 then needs 20

B locks 20 then needs 10
```

Deadlock possible.

---

# 29. Deterministic Ordering

Instead, define:

> Always lock accounts in ascending account ID order.

Then both transactions use:

```text
Lock 10
then
Lock 20
```

Even if the transfer directions differ.

Possible execution:

```text
A locks 10

B tries 10
→ waits

A locks 20

A completes

A commits

B gets 10

B gets 20
```

There is blocking.

But no circular wait is created by this lock sequence.

---

# 30. Why Lock Ordering Works

Suppose every transaction must acquire resources according to a global order:

```text
R1 < R2 < R3 < R4
```

A transaction may acquire:

```text
R1 → R3
```

or:

```text
R2 → R4
```

but never:

```text
R4 → R2
```

This prevents circular resource acquisition.

A cycle would require some transaction eventually to wait "backward" in the ordering.

The global ordering rule prevents that.

---

# 31. Ordering Multiple Rows

Suppose an operation updates candidate IDs:

```text
42
11
70
5
```

Instead of locking in request-dependent order:

```text
42 → 11 → 70 → 5
```

sort them:

```text
5 → 11 → 42 → 70
```

Then every transaction touching overlapping candidates uses the same ordering rule.

This can significantly reduce deadlock risk.

---

# 32. Keep Transactions Short

Consider:

```text
BEGIN

Lock row

Call external API
Wait 8 seconds

Perform computation
Wait 4 seconds

Update row

COMMIT
```

The transaction holds resources for a long period.

This increases the window during which other transactions can conflict.

Better:

```text
Perform unrelated work first

BEGIN

Perform required database operations

COMMIT
```

when business correctness allows it.

---

# 33. Short Transactions Reduce Risk, Not Guarantee Elimination

A shorter transaction:

```text
holds locks for less time
```

which reduces the probability of overlapping conflicts.

But even two very short transactions can deadlock if they acquire resources in conflicting orders.

Therefore:

```text
Short transactions
```

are helpful but are not a complete deadlock-prevention strategy.

---

# 34. Avoid External Calls Inside Lock-Holding Transactions

Potentially problematic:

```text
BEGIN

SELECT ... FOR UPDATE

Call payment API

Wait for response

UPDATE ...

COMMIT
```

While waiting:

```text
Database locks remain held
```

depending on the operation and DBMS.

This can increase:

```text
Blocking
Deadlock probability
Connection usage
Latency
```

External side effects also introduce failure-handling complexity.

Keep database critical sections focused where possible.

---

# 35. Deadlock Avoidance

**Deadlock avoidance** differs conceptually from prevention.

Prevention:

```text
Design rules that make deadlock structurally impossible
```

Avoidance:

```text
Evaluate whether granting a resource request could move the system into an unsafe state
```

A classical operating-systems example is:

```text
Banker's Algorithm
```

General-purpose relational databases more commonly rely on combinations of:

```text
Locking policies
Detection
Abort/retry
Application-level ordering
```

rather than requiring applications to declare all future resource needs.

---

# 36. Prevention vs Avoidance vs Detection

### Prevention

Break a necessary deadlock condition.

Example:

```text
Global lock ordering
```

### Avoidance

Grant resources only when the resulting state remains safe.

Classical example:

```text
Banker's Algorithm
```

### Detection

Allow deadlocks to form.

Then:

```text
Detect cycle
Abort victim
```

Databases commonly use detection together with application design that reduces deadlock frequency.

---

# 37. Banker's Algorithm

Banker's Algorithm is a classical deadlock-avoidance algorithm.

It reasons about:

```text
Available resources

Maximum resource requirements

Currently allocated resources

Remaining needs
```

and grants requests only when the system can remain in a safe state.

It is important academically.

However, it is not the normal mechanism used for ordinary row-lock management in most transactional application databases.

Do not answer:

> Databases solve deadlocks using Banker's Algorithm.

as a universal claim.

---

# 38. Safe State

In deadlock-avoidance theory, a **safe state** means there exists some ordering in which all processes or transactions can eventually complete using available resources.

Conceptually:

```text
Current allocations
      ↓
Can we find completion order?
      ↓
Yes
      ↓
Safe state
```

An unsafe state does not necessarily mean a deadlock already exists.

It means the system may no longer be able to guarantee avoidance.

---

# 39. Deadlock Prevention by Acquiring Everything First

One theoretical approach is:

```text
Request all required resources before beginning work.
```

This can break:

```text
Hold and Wait
```

because a transaction does not hold some resources while waiting for others.

But in databases this can be impractical because:

```text
Resource needs may depend on query results

Large lock sets reduce concurrency

Transactions may not know all resources in advance
```

So this is more useful as theory than as a universal application strategy.

---

# 40. Deadlock Prevention by Preemption

Another theoretical strategy is to allow resources to be preempted.

But database transactions cannot usually have arbitrary locks removed while continuing normally.

Instead:

```text
Abort transaction
      ↓
Rollback
      ↓
Release locks
```

This effectively frees resources by terminating the transaction.

The transaction may then be retried.

---

# 41. SQL Deadlock Scenario

Consider:

```sql
-- Transaction A

BEGIN;

UPDATE candidates
SET status = 'active'
WHERE id = 1;

UPDATE candidates
SET status = 'active'
WHERE id = 2;

COMMIT;
```

At the same time:

```sql
-- Transaction B

BEGIN;

UPDATE candidates
SET status = 'inactive'
WHERE id = 2;

UPDATE candidates
SET status = 'inactive'
WHERE id = 1;

COMMIT;
```

Possible execution:

```text
A locks row 1

B locks row 2

A requests row 2
→ waits

B requests row 1
→ waits
```

Deadlock.

---

# 42. Fixing the SQL Scenario

Ensure both transactions access rows in the same order.

For example:

```text
Always process candidate IDs ascending.
```

Then:

```sql
-- Transaction A

BEGIN;

UPDATE candidates
SET status = 'active'
WHERE id = 1;

UPDATE candidates
SET status = 'active'
WHERE id = 2;

COMMIT;
```

and Transaction B should also access:

```text
1
then
2
```

rather than:

```text
2
then
1
```

---

# 43. SELECT FOR UPDATE Deadlock

Explicit locking can also create deadlocks.

Transaction A:

```sql
BEGIN;

SELECT *
FROM accounts
WHERE id = 1
FOR UPDATE;

SELECT *
FROM accounts
WHERE id = 2
FOR UPDATE;
```

Transaction B:

```sql
BEGIN;

SELECT *
FROM accounts
WHERE id = 2
FOR UPDATE;

SELECT *
FROM accounts
WHERE id = 1
FOR UPDATE;
```

Possible result:

```text
A locks 1

B locks 2

A waits for 2

B waits for 1
```

Deadlock.

`FOR UPDATE` does not make deadlocks impossible.

---

# 44. Deterministic FOR UPDATE

Suppose a transfer touches accounts:

```text
source_id = 20
destination_id = 10
```

Instead of locking:

```text
source
then destination
```

derive:

```text
first_id  = MIN(source_id, destination_id)
second_id = MAX(source_id, destination_id)
```

Then lock:

```text
10
then
20
```

for every transfer.

This creates deterministic resource ordering.

---

# 45. Multiple-Row Locking

Some applications need to lock multiple rows.

Conceptually:

```sql
SELECT *
FROM accounts
WHERE id IN (10, 20)
ORDER BY id
FOR UPDATE;
```

This expresses the intended ordering clearly.

However, exact lock acquisition behavior and optimizer behavior should be verified for the specific DBMS.

Do not assume SQL text ordering always provides every lock-order guarantee you need across all systems.

---

# 46. Indexes and Deadlocks

Indexes are usually discussed as performance structures.

But indexing can also influence concurrency.

Suppose:

```sql
UPDATE interviews
SET status = 'expired'
WHERE candidate_id = 100;
```

Without an appropriate index, the database may need to inspect a much larger portion of the table.

Depending on the DBMS and execution strategy, this can:

```text
Take longer

Touch more pages/rows

Hold resources longer

Increase overlap with concurrent transactions
```

Appropriate indexes can reduce the amount of work and shorten transactions.

That may reduce contention and deadlock probability.

---

# 47. Missing Indexes Do Not Directly "Cause" Every Deadlock

Avoid saying:

> Missing indexes cause deadlocks.

A better statement:

> Poor access paths can increase query duration and the number of resources touched, which can increase contention and make certain deadlock patterns more likely.

Deadlocks still require cyclic waiting.

---

# 48. Large Batch Updates

Suppose:

```sql
UPDATE interviews
SET status = 'archived'
WHERE created_at < '2025-01-01';
```

affects millions of rows.

This transaction may:

```text
Run for a long time

Hold many locks/resources

Conflict with user traffic

Increase rollback cost
```

Large maintenance operations should be designed carefully.

Possible strategies include:

```text
Batching

Scheduling during lower traffic

Appropriate indexes

Shorter transactions
```

depending on correctness requirements.

---

# 49. Batching

Instead of updating:

```text
5,000,000 rows
```

in one transaction, a system may process smaller batches where business semantics allow it.

Conceptually:

```text
10,000 rows
COMMIT

10,000 rows
COMMIT

10,000 rows
COMMIT
```

Potential benefits:

```text
Shorter transactions

Reduced lock duration

Smaller rollback scope

Lower contention
```

But batching changes atomicity.

Do not split a transaction when the entire operation must succeed or fail as one unit.

---

# 50. Hot Rows and Deadlocks

Suppose many transactions update:

```text
candidate_usage_summary
```

for the same candidate.

This creates a hot row.

Hot rows commonly cause:

```text
Contention
Waiting
Latency
```

They do not automatically create deadlocks.

But if transactions also lock other resources in varying orders, hot rows can participate in deadlock cycles.

---

# 51. Hot Row Example

Transaction A:

```text
Lock candidate credits
Lock interview record
```

Transaction B:

```text
Lock interview record
Lock candidate credits
```

If both touch the same logical resources:

```text
A holds credits

B holds interview

A needs interview

B needs credits
```

Deadlock.

Better architecture should define a consistent resource-access order.

---

# 52. Foreign Keys and Deadlocks

Foreign-key operations can introduce locking interactions that developers may overlook.

Example:

```text
Transaction A updates parent

Transaction B inserts child

Transaction C deletes related rows
```

The database may need locks to preserve referential integrity.

The exact lock behavior depends heavily on the DBMS.

When investigating a deadlock, inspect:

```text
Foreign keys

Triggers

Cascades

Indexes

Implicit database operations
```

not only the visible application statements.

---

# 53. Unique Constraints and Deadlocks

Concurrent transactions inserting or updating values covered by uniqueness constraints can interact through internal index and locking mechanisms.

For example:

```text
Transaction A inserts unique key X

Transaction B interacts with related unique keys or rows
```

Complex sequences can produce waits and occasionally deadlocks.

The lesson is:

> Deadlocks can involve indexes and constraint enforcement, not only explicit row locks.

---

# 54. Triggers and Hidden Work

Suppose application code executes:

```sql
UPDATE orders
SET status = 'completed'
WHERE id = 100;
```

but the table has a trigger that updates:

```text
inventory
audit_log
customer_stats
```

The transaction may acquire resources that are not obvious from the original SQL statement.

This can create unexpected lock ordering.

When diagnosing deadlocks, include:

```text
Triggers

Cascades

Stored procedures

Constraint checks
```

in the investigation.

---

# 55. Deadlocks Across Tables

Deadlocks do not require transactions to update the same table.

Transaction A:

```text
Lock users row
Lock subscriptions row
```

Transaction B:

```text
Lock subscriptions row
Lock users row
```

Possible cycle:

```text
A holds users

B holds subscriptions

A waits for subscriptions

B waits for users
```

Deadlock.

The resources can belong to different tables.

---

# 56. Deadlocks Across Many Transactions

Suppose:

```text
T1 holds A, waits for B

T2 holds B, waits for C

T3 holds C, waits for D

T4 holds D, waits for A
```

Cycle:

```text
T1 → T2 → T3 → T4 → T1
```

A deadlock detector must detect cycles of arbitrary size, not only two-transaction cases.

---

# 57. Application Retry

Suppose the database aborts Transaction B as the deadlock victim.

The application receives an error.

A common strategy is:

```text
Deadlock detected
      ↓
Transaction rolled back
      ↓
Wait briefly
      ↓
Retry entire transaction
```

The important phrase is:

```text
entire transaction
```

Do not blindly retry only the final failed SQL statement if earlier transaction work was rolled back.

---

# 58. Why Retry the Whole Transaction?

Suppose:

```text
BEGIN

Read A
Update B
Update C
```

Deadlock occurs while updating C.

Database rolls back the transaction.

The earlier:

```text
Update B
```

may no longer exist.

Retrying only:

```text
Update C
```

would produce incorrect state.

The application should rerun the logical transaction from a valid starting point.

---

# 59. Retry With Backoff

Avoid:

```text
Failure
↓
Retry immediately
↓
Failure
↓
Retry immediately
↓
Failure
```

If competing transactions retry simultaneously, they may collide repeatedly.

Use controlled retry behavior such as:

```text
Attempt 1
    ↓
Deadlock
    ↓
Short randomized backoff
    ↓
Attempt 2
```

Randomization can reduce synchronized retries.

---

# 60. Bounded Retries

Do not retry forever.

Example strategy:

```text
Attempt 1
Attempt 2
Attempt 3
```

Then:

```text
Return failure
Log context
Raise alert if appropriate
```

Exact retry limits depend on the workload.

Infinite retry loops can hide serious contention problems and increase system load.

---

# 61. Idempotency and Retries

Retries become especially important when transactions interact with external systems.

Suppose:

```text
Charge payment provider

Then database transaction deadlocks
```

If the entire application operation is blindly retried:

```text
payment may be charged twice
```

Therefore retry-safe workflows may require:

```text
Idempotency keys

Operation IDs

Transactional outbox

Workflow state

Compensation
```

depending on architecture.

---

# 62. Database Retry vs External Side Effects

A transaction rollback can undo:

```text
Database changes participating in that transaction
```

It cannot automatically undo:

```text
Email sent

HTTP request

Payment charge

SMS sent

Message already published
```

Therefore:

```text
Deadlock retry
```

must consider external side effects.

This is a system-design issue, not only a SQL issue.

---

# 63. Retry Pseudocode

Conceptually:

```text
for attempt in 1..MAX_RETRIES:

    try:
        BEGIN

        perform transaction

        COMMIT

        return success

    catch deadlock:
        ROLLBACK

        if attempt == MAX_RETRIES:
            raise

        wait with backoff
```

Production code should use DBMS-specific error detection rather than matching generic error strings.

---

# 64. Do Not Retry Every Database Error

A deadlock may be transient.

But:

```text
UNIQUE constraint violation

Invalid SQL

Missing table

Permission error

CHECK constraint violation
```

usually will not be fixed by blindly retrying the same operation.

Retry logic should classify errors.

Example:

```text
Deadlock
→ maybe retry

Serialization failure
→ maybe retry

Temporary connection failure
→ maybe retry depending on commit uncertainty

Constraint violation
→ usually handle as application/data error
```

---

# 65. Commit Uncertainty

A difficult failure case is:

```text
Application sends COMMIT

Connection fails before response arrives
```

The application may not know whether:

```text
COMMIT succeeded
```

or:

```text
COMMIT failed
```

Blindly repeating externally visible operations can cause duplication.

Systems requiring strong reliability often use:

```text
Idempotency

Unique operation identifiers

Reconciliation

Durable workflow state
```

to handle ambiguous outcomes.

---

# 66. Deadlock Logging

Production systems should capture enough information to diagnose deadlocks.

Useful information includes:

```text
Transaction identifiers

SQL statements

Locked resources

Waiting resources

Tables

Indexes

Execution plans

Transaction duration

Application operation

Request identifiers
```

Without observability, recurring deadlocks become difficult to fix.

---

# 67. Diagnosing a Deadlock

When a deadlock occurs:

```text
1. Identify transactions in the cycle

2. Identify resources each transaction holds

3. Identify resources each transaction requests

4. Determine acquisition order

5. Inspect query plans

6. Check indexes

7. Check transaction duration

8. Check hidden work such as triggers

9. Check whether resource order can be standardized

10. Verify retry handling
```

Do not solve deadlocks by simply increasing timeouts.

---

# 68. Deadlock Graph

Some database systems expose deadlock reports or graphs.

Conceptually:

```text
Transaction A
    holds X
    waits Y

Transaction B
    holds Y
    waits X
```

A useful deadlock graph answers:

```text
Who waited?

For what?

Who owned it?

Which statement was executing?
```

This is much more useful than only knowing:

```text
Deadlock occurred.
```

---

# 69. PostgreSQL Deadlock Behavior

PostgreSQL detects deadlocks involving lock waits.

When a deadlock is detected, PostgreSQL aborts one of the transactions and reports an error.

Applications should treat the transaction as failed and roll back/retry appropriately.

PostgreSQL also exposes configuration and logging options useful for investigating lock waits and deadlocks.

Exact operational configuration should be based on the deployed PostgreSQL version.

---

# 70. MySQL/InnoDB Deadlock Behavior

InnoDB can detect transactional deadlocks and roll back a transaction to break the cycle.

Deadlocks can involve:

```text
Record locks

Index records

Gap/next-key locking

Foreign-key checks

Concurrent updates
```

depending on isolation level and query pattern.

InnoDB diagnostic information can help identify recent deadlocks.

Exact behavior is version- and configuration-dependent.

---

# 71. SQL Server Deadlock Behavior

SQL Server can detect deadlock cycles and choose a victim transaction.

Victim selection can be influenced by factors such as:

```text
Deadlock priority

Rollback cost
```

SQL Server provides deadlock graph diagnostics through its observability tooling.

Again, implementation details differ from PostgreSQL and MySQL.

---

# 72. Deadlock Handling Is DBMS-Specific

Do not memorize one database's behavior and apply it universally.

Different systems vary in:

```text
Lock types

Lock granularity

MVCC behavior

Deadlock detection timing

Victim selection

Diagnostics

Error codes

Retry recommendations
```

A strong interview answer separates:

```text
General deadlock theory
```

from:

```text
DBMS-specific implementation
```

---

# 73. Can READ COMMITTED Have Deadlocks?

Yes.

Deadlocks are not exclusive to:

```text
SERIALIZABLE
```

Any isolation level where transactions acquire incompatible resources in cyclic order can potentially experience deadlocks.

Example:

```text
A updates Row 1
B updates Row 2

A needs Row 2
B needs Row 1
```

Even ordinary write operations can deadlock.

---

# 74. Can SERIALIZABLE Cause Deadlocks?

Depending on implementation, stronger isolation may involve additional locking or conflict mechanisms.

Therefore stronger isolation does not mean:

```text
deadlocks become impossible
```

Some serializable implementations may abort transactions for serialization conflicts that are not traditional lock deadlocks.

Applications should distinguish error types when appropriate.

---

# 75. Deadlock vs Serialization Failure

These concepts are related but different.

### Deadlock

```text
Cyclic resource wait
```

Example:

```text
A waits for B
B waits for A
```

### Serialization Failure

The database determines concurrent transactions cannot all commit while preserving required serializable semantics.

There may not be a traditional lock cycle.

Both can require:

```text
Rollback
Retry
```

but their underlying causes differ.

---

# 76. Deadlock vs Optimistic Conflict

Optimistic concurrency might produce:

```text
UPDATE ... WHERE version = 5
```

with:

```text
0 rows affected
```

because another transaction changed the row.

That is a concurrency conflict.

It is not necessarily a deadlock.

No cyclic waiting is required.

---

# 77. Deadlock vs Unique Constraint Conflict

Two transactions attempt:

```text
INSERT same unique email
```

One eventually fails because:

```text
UNIQUE(email)
```

is violated.

This is not automatically a deadlock.

It is a uniqueness conflict.

Deadlock terminology should be reserved for cyclic waiting.

---

# 78. Common Interview Question: What Is a Deadlock?

Weak answer:

> A deadlock happens when two transactions block each other.

Better answer:

> A deadlock occurs when transactions form a cyclic dependency while waiting for incompatible resources held by one another, so none of the transactions in the cycle can proceed without intervention. Databases typically detect the cycle and abort one transaction to break it.

---

# 79. Common Interview Question: What Are the Coffman Conditions?

The four conditions are:

```text
Mutual Exclusion

Hold and Wait

No Preemption

Circular Wait
```

A classical deadlock requires all four simultaneously.

A prevention strategy ensures at least one cannot occur.

---

# 80. Common Interview Question: How Does a Database Detect Deadlocks?

A strong conceptual answer:

> The database tracks lock waits and transaction dependencies. These can be modeled as a wait-for graph where an edge from transaction A to transaction B means A is waiting for a resource held by B. A cycle indicates a deadlock, after which the database can choose a victim transaction to abort.

---

# 81. Common Interview Question: How Do You Prevent Deadlocks?

A strong answer:

> I first try to reduce the possibility of circular waits by acquiring shared resources in a deterministic order, keeping transactions short, minimizing unnecessary locking, and ensuring queries use efficient access paths. I still assume deadlocks can occur in production, so retryable transactions must handle deadlock errors safely.

This is stronger than:

> Always lock rows.

---

# 82. Common Interview Question: Why Does Lock Ordering Help?

Because circular wait requires transactions to acquire resources in conflicting orders.

If every transaction follows:

```text
R1 → R2 → R3
```

then a transaction cannot hold a higher-ordered resource while waiting for a lower-ordered resource.

That removes the circular acquisition pattern under the ordering scheme.

---

# 83. Common Interview Question: Can Deadlocks Be Completely Prevented?

For narrowly controlled workflows, resource ordering and other design constraints can eliminate specific deadlock patterns.

But in complex production databases:

```text
Many queries

Multiple tables

Constraints

Triggers

ORM behavior

Background jobs

Different services
```

it is safer to design both for:

```text
Deadlock reduction
+
Deadlock recovery
```

rather than assume deadlocks will never occur.

---

# 84. Common Interview Question: What Happens After a Deadlock?

Typical sequence:

```text
Deadlock detected
      ↓
Database chooses victim
      ↓
Victim transaction aborted
      ↓
Its locks released
      ↓
Other transaction proceeds
      ↓
Application receives error
      ↓
Application may retry
```

Exact behavior depends on the database.

---

# 85. Common Interview Question: Should You Retry a Deadlock?

Often yes, when:

```text
The operation is safe to retry

The entire failed transaction is restarted

Retries are bounded

Backoff is used where appropriate
```

But retries should not hide persistent contention.

If deadlocks happen frequently:

```text
Investigate the design.
```

---

# 86. Common Interview Question: Deadlock vs Timeout?

### Deadlock

Detected cyclic dependency.

### Timeout

A wait exceeded a configured duration.

A timeout can happen without a deadlock.

A deadlock can sometimes be detected before a general lock timeout.

---

# 87. Common Interview Question: Does MVCC Prevent Deadlocks?

No.

MVCC can reduce some reader/writer blocking, but concurrent writes and explicit locks can still create cycles.

Example:

```text
A locks X

B locks Y

A needs Y

B needs X
```

MVCC does not eliminate that conflict.

---

# 88. Common Interview Question: Can Indexes Reduce Deadlocks?

A strong answer:

> Appropriate indexes can reduce the amount of data a query scans or modifies and can shorten transaction duration, which may reduce contention and the probability of some deadlocks. But indexes do not inherently eliminate circular waits, and index structures can themselves participate in locking behavior depending on the database.

---

# 89. Common Interview Questions

Be prepared to answer:

1. What is a deadlock?
2. Deadlock vs blocking?
3. Deadlock vs starvation?
4. Deadlock vs livelock?
5. What causes a deadlock?
6. What are the Coffman conditions?
7. What is mutual exclusion?
8. What is hold and wait?
9. What is no preemption?
10. What is circular wait?
11. What is a wait-for graph?
12. How is a deadlock detected?
13. What is a deadlock victim?
14. How does a database resolve a deadlock?
15. How is the victim selected?
16. Deadlock detection vs prevention?
17. Prevention vs avoidance?
18. What is a safe state?
19. What is Banker's Algorithm?
20. Do databases normally use Banker's Algorithm?
21. Why does lock ordering prevent certain deadlocks?
22. How do short transactions help?
23. Can SELECT FOR UPDATE deadlock?
24. Can READ COMMITTED deadlock?
25. Can SERIALIZABLE deadlock?
26. Does MVCC eliminate deadlocks?
27. Deadlock vs serialization failure?
28. Deadlock vs optimistic conflict?
29. Deadlock vs lock timeout?
30. How should an application retry a deadlock?
31. Why should retries use backoff?
32. Why should retries be bounded?
33. Why does idempotency matter?
34. How can indexes influence deadlocks?
35. How would you diagnose a production deadlock?
36. Can foreign keys participate in deadlocks?
37. Can triggers contribute to deadlocks?
38. How would you prevent transfer deadlocks?
39. How would you handle large batch updates?
40. Should every deadlock simply be retried?

---

# 90. Common Mistakes

## Mistake 1: Every blocked transaction is deadlocked

Incorrect.

Blocking is normal.

Deadlock requires a cyclic dependency.

---

## Mistake 2: Deadlocks happen only between two transactions

Incorrect.

Any number of transactions can participate in the cycle.

---

## Mistake 3: Deadlocks happen only when updating the same row

Incorrect.

Deadlocks can involve:

```text
Different rows

Different tables

Indexes

Foreign keys

Metadata

Ranges

Other lockable resources
```

---

## Mistake 4: Transactions prevent deadlocks

Incorrect.

Transactions are the entities that often hold locks involved in deadlocks.

---

## Mistake 5: MVCC prevents deadlocks

Incorrect.

Write conflicts and explicit locks can still deadlock.

---

## Mistake 6: Increasing lock timeout fixes deadlocks

Incorrect.

Timeout configuration does not remove circular dependencies.

---

## Mistake 7: Retry forever

Incorrect.

Retries should be:

```text
Bounded
Controlled
Observable
```

---

## Mistake 8: Retry only the failed SQL statement

Often incorrect.

If the database aborted the transaction, retry the logical transaction from the correct boundary.

---

## Mistake 9: Deadlock means database bug

Incorrect.

Deadlocks are a normal possibility in concurrent transactional systems.

Frequent deadlocks may indicate poor workload or transaction design.

---

## Mistake 10: Lock ordering means no transaction ever waits

Incorrect.

Lock ordering can prevent certain cycles.

Transactions may still block.

Example:

```text
A locks Row 1

B waits for Row 1

A commits

B continues
```

That is normal blocking.

---

# 91. Interview Scenario: Opposite Transfers

Transaction A:

```text
Transfer $100:

Account 10
→
Account 20
```

Transaction B:

```text
Transfer $50:

Account 20
→
Account 10
```

Naive locking:

```text
A:
Lock source 10
Lock destination 20

B:
Lock source 20
Lock destination 10
```

Possible:

```text
A locks 10

B locks 20

A waits for 20

B waits for 10
```

Deadlock.

---

# 92. Better Transfer Strategy

Define:

```text
first_account  = MIN(source, destination)

second_account = MAX(source, destination)
```

Both transactions lock:

```text
10
then
20
```

regardless of transfer direction.

Then:

```text
One transaction gets 10

Other waits

First gets 20

First completes

Second continues
```

Blocking can occur.

Circular wait does not arise from opposite lock ordering because the order is now consistent.

---

# 93. Interview Scenario: Interview Completion

Suppose completing an interview updates:

```text
interviews

candidate_credits
```

Flow A:

```text
Lock interview
Update interview

Lock credits
Update credits
```

Another workflow uses:

```text
Lock credits
Update credits

Lock interview
Update interview
```

These workflows can deadlock.

Define one resource order:

```text
candidate_credits
      ↓
interview
```

or:

```text
interview
      ↓
candidate_credits
```

and use it consistently across workflows where practical.

---

# 94. Interview Scenario: Batch Processing

Worker A processes:

```text
Candidate IDs:
1 → 2 → 3 → 4
```

Worker B processes:

```text
Candidate IDs:
4 → 3 → 2 → 1
```

Potential conflict:

```text
A owns lower IDs

B owns higher IDs

Both move toward each other
```

Better:

```text
All workers process shared resources using a deterministic ordering strategy.
```

Additional techniques may include:

```text
Partitioning work

SKIP LOCKED-style queues

Smaller batches
```

where supported and semantically appropriate.

---

# 95. Production Deadlock Framework

When deadlocks appear in production:

```text
Deadlock
   ↓
Capture diagnostic graph/report
   ↓
Identify participants
   ↓
Map SQL to application workflows
   ↓
Identify resource acquisition order
   ↓
Check transaction duration
   ↓
Check query plans/indexes
   ↓
Check triggers/FKs/cascades
   ↓
Standardize ordering where possible
   ↓
Reduce transaction scope
   ↓
Verify safe retry
   ↓
Monitor deadlock frequency
```

Do not begin by randomly adding indexes or increasing timeouts.

---

# 96. Deadlock Prevention Mental Model

Think:

```text
What resources can this transaction acquire?
             ↓
In what order?
             ↓
Can another transaction acquire them in reverse order?
             ↓
Yes
             ↓
Circular wait possible
             ↓
Define deterministic ordering
```

Then also ask:

```text
Can transaction duration be reduced?

Can the operation use fewer resources?

Can a single atomic statement replace multiple steps?
```

---

# 97. Deadlock Recovery Mental Model

Even after prevention work:

```text
Assume deadlock can happen
        ↓
Database detects cycle
        ↓
One transaction aborted
        ↓
Application detects retryable error
        ↓
Rollback transaction state
        ↓
Backoff
        ↓
Retry complete operation
        ↓
Stop after bounded attempts
```

This is the production-grade mindset.

---

# 98. Quick Revision

Remember:

```text
Blocking
→ One transaction waits

Deadlock
→ Cyclic waiting

Starvation
→ Transaction repeatedly fails to progress

Livelock
→ Transactions remain active but make no useful progress
```

Coffman conditions:

```text
Mutual Exclusion

Hold and Wait

No Preemption

Circular Wait
```

Wait-for graph:

```text
A → B
```

means:

```text
A waits for B
```

Cycle:

```text
A → B → C → A
```

means:

```text
Deadlock
```

---

# 99. Practical Strategy

For application development:

```text
1. Keep transactions focused

2. Acquire shared resources consistently

3. Prefer deterministic lock ordering

4. Use appropriate indexes

5. Avoid unnecessary external waits inside transactions

6. Use atomic database operations where possible

7. Expect occasional deadlocks

8. Retry complete transactions safely

9. Use bounded retries and backoff

10. Monitor recurring deadlock patterns
```

The objective is not:

```text
Never see a deadlock error.
```

The objective is:

```text
Make deadlocks uncommon
+
Recover correctly when they occur.
```

---

# 100. Final Takeaway

A deadlock is fundamentally a dependency-cycle problem.

```text
Transaction A
holds X
needs Y

Transaction B
holds Y
needs X
```

creates:

```text
A → B → A
```

The database can break the cycle by aborting a participant.

But application design should reduce how often such cycles occur.

The strongest engineering approach combines:

```text
Deterministic resource ordering

Short transactions

Efficient queries

Minimal lock scope

Deadlock detection

Safe retries

Idempotency

Observability
```

Remember:

```text
Blocking ≠ Deadlock

Timeout ≠ Deadlock detection

MVCC ≠ No deadlocks

Retry ≠ Fix

Lock ordering reduces circular waits

Deadlocks are expected possibilities in concurrent systems
```

A strong interview answer does not stop at:

> The database rolls back one transaction.

It explains:

```text
Why the cycle formed
How the database detects it
How the cycle is broken
How the application recovers
How the design can prevent the same pattern from recurring
```

That is the difference between knowing the definition of a deadlock and understanding database concurrency.

---

## Previous

**[← Concurrency and Isolation](./07-concurrency-and-isolation.md)**

## Next

**[Normalization →](./09-normalization.md)**

---

Maintained by **InterviewEraHQ**.
