# Transactions and ACID in DBMS

Transactions are one of the most important concepts in database systems.

Applications frequently perform operations where multiple database changes belong to one logical unit of work.

Examples:

```text
Transfer money between accounts
Create an order and reserve inventory
Register a user and create their profile
Complete an interview and store its result
Create a payment and update subscription status
```

The challenge is that failures can happen between individual operations.

A transaction provides a mechanism for treating related database operations as a logical unit.

A strong interview candidate should understand:

- What a transaction is
- Why transactions are needed
- COMMIT and ROLLBACK
- ACID properties
- What atomicity actually guarantees
- What consistency means
- Why isolation matters
- What durability does and does not mean
- Transaction boundaries
- Savepoints
- Autocommit
- Failure scenarios
- Why long-running transactions can become problematic

---

# 1. What Is a Transaction?

A **transaction** is a sequence of database operations treated as one logical unit of work.

Consider transferring:

```text
$100
```

from Account A to Account B.

The application needs to perform:

```text
1. Subtract $100 from Account A
2. Add $100 to Account B
```

These operations belong together.

The desired outcome is:

```text
Both operations succeed
```

or:

```text
Neither operation takes effect
```

We do not want:

```text
Account A loses $100
```

while:

```text
Account B receives nothing
```

A transaction helps the database manage this unit of work correctly.

---

# 2. Basic Transaction Example

A simplified SQL transaction might look like:

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 100
WHERE id = 1;

UPDATE accounts
SET balance = balance + 100
WHERE id = 2;

COMMIT;
```

Conceptually:

```text
BEGIN
  ↓
Operation 1
  ↓
Operation 2
  ↓
COMMIT
```

After a successful commit, the transaction's changes become committed according to the database's transactional semantics.

---

# 3. What Is COMMIT?

`COMMIT` completes the current transaction.

Example:

```sql
BEGIN;

UPDATE candidates
SET status = 'interviewed'
WHERE id = 100;

COMMIT;
```

Conceptually:

```text
Transaction starts
       ↓
Changes occur
       ↓
COMMIT
       ↓
Transaction successfully completes
```

After commit, the database treats the transaction's changes as committed data.

---

# 4. What Is ROLLBACK?

`ROLLBACK` aborts a transaction and reverses changes made by that transaction that have not been committed.

Example:

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 100
WHERE id = 1;

-- Some condition fails

ROLLBACK;
```

Conceptually:

```text
BEGIN
  ↓
Changes
  ↓
Problem
  ↓
ROLLBACK
  ↓
Discard transaction changes
```

This allows the database to return to the appropriate state from before the transaction's uncommitted changes.

---

# 5. Why Are Transactions Needed?

Consider an interview platform.

When a candidate completes an interview, the application might need to:

```text
Mark interview as completed
Store final score
Store evaluation
Deduct one interview credit
Update candidate statistics
```

Suppose the first three operations succeed but deducting the credit fails.

Without appropriate transactional design, the database might contain a partially completed workflow.

For example:

```text
Interview = completed
Score = stored
Evaluation = stored
Credit = not deducted
Statistics = not updated
```

Whether these operations should all belong to one database transaction depends on the architecture and business requirements.

But when several database changes must succeed or fail as one unit, transactions are the primary mechanism for expressing that requirement.

---

# 6. Transaction Lifecycle

A simplified transaction lifecycle can be represented as:

```text
BEGIN
  ↓
Execute operations
  ↓
Everything valid?
  ├── Yes → COMMIT
  │
  └── No  → ROLLBACK
```

Real database systems have more detailed internal transaction states, but this model is sufficient for many interviews.

---

# 7. What Is ACID?

ACID represents four important transaction properties:

```text
A → Atomicity
C → Consistency
I → Isolation
D → Durability
```

These properties describe important guarantees and design goals provided by transactional database systems.

Understanding the meaning of each property is much more important than memorizing the acronym.

---

# 8. Atomicity

**Atomicity** means a transaction is treated as an indivisible logical unit with respect to its database changes.

Conceptually:

```text
All required transaction changes happen
```

or:

```text
None of them take effect
```

Consider:

```text
Transfer $100 from A to B
```

Operations:

```text
A.balance -= $100
B.balance += $100
```

If the second operation fails, we should not leave only the first operation applied.

Atomicity ensures the transaction does not leave a partial set of its intended changes committed.

---

# 9. Atomicity Example

Suppose:

```text
Account A = $500
Account B = $300
```

We transfer:

```text
$100
```

Expected result:

```text
Account A = $400
Account B = $400
```

Now suppose the system performs:

```text
Account A = $400
```

and then the second update fails.

Without atomicity:

```text
Account A = $400
Account B = $300
```

The $100 has effectively disappeared from the represented balances.

With an atomic transaction, the failed transaction can be rolled back:

```text
Account A = $500
Account B = $300
```

---

# 10. Atomicity Does Not Mean Every Application Workflow Is Automatically Atomic

Suppose an application performs:

```text
1. Update PostgreSQL
2. Send email
3. Charge external payment API
4. Publish message to another service
```

Wrapping the PostgreSQL operation in a database transaction does not automatically make the external operations part of the same atomic transaction.

A local database transaction controls operations participating in that transaction.

Distributed workflows may require additional patterns such as:

```text
Outbox pattern
Saga
Idempotency
Compensation
Distributed transactions
```

depending on the architecture.

Do not assume:

> BEGIN and COMMIT make the entire application workflow atomic.

---

# 11. Consistency

**Consistency** means a transaction should move the database from one valid state to another valid state while preserving the integrity rules that the database and transaction are responsible for enforcing.

Examples of integrity rules might include:

```text
Primary key uniqueness

Foreign key validity

CHECK constraints

NOT NULL constraints

Application invariants enforced transactionally
```

Suppose a rule says:

```text
score must be between 0 and 100
```

and the database has:

```sql
CHECK (score >= 0 AND score <= 100)
```

A transaction attempting:

```text
score = 150
```

cannot successfully commit that invalid value while the constraint is enforced.

---

# 12. Consistency Is Commonly Misunderstood

A weak explanation is:

> Consistency means all users always see the same data.

That is not what the **C in ACID** primarily means.

That idea is closer to concerns involving:

```text
Isolation
Replication consistency
Distributed systems consistency
```

ACID consistency concerns preserving defined correctness rules and invariants across transactions.

---

# 13. Who Defines Consistency?

The database cannot understand every business rule automatically.

Suppose your business rule is:

```text
A candidate gets one free interview per month.
```

The database does not know this unless the schema, transaction logic, or application design expresses and enforces the rule.

Consistency depends on mechanisms such as:

```text
Schema constraints
Transaction logic
Application logic
Correct concurrency control
```

Therefore:

> ACID consistency does not automatically make incorrect business logic correct.

---

# 14. Isolation

**Isolation** concerns how concurrently executing transactions interact and what intermediate effects they can observe.

Suppose:

```text
Transaction A
```

and:

```text
Transaction B
```

execute at the same time.

Without appropriate isolation, one transaction might observe data in ways that produce incorrect application behavior.

A simplified mental model is:

> Transactions should behave with controlled interference from concurrently executing transactions.

The exact guarantees depend on the isolation level and database implementation.

---

# 15. Isolation Does Not Mean Transactions Literally Run One at a Time

A common misconception is:

> Isolation means the database executes transactions sequentially.

Not necessarily.

Database systems support concurrency.

Conceptually:

```text
Transaction A ──────────────►

       Transaction B ──────────────►

Transaction C ─────────►
```

The database uses mechanisms such as:

```text
Locks
MVCC
Serialization checks
Snapshots
```

to provide isolation guarantees while allowing concurrency.

Exact mechanisms vary by DBMS.

---

# 16. Isolation Levels

Common SQL isolation-level names include:

```text
READ UNCOMMITTED
READ COMMITTED
REPEATABLE READ
SERIALIZABLE
```

Some database systems also provide:

```text
Snapshot isolation
```

or database-specific variants.

Different isolation levels trade stronger concurrency guarantees against factors such as:

```text
Blocking
Retries
Concurrency
Implementation cost
```

We will cover isolation levels, dirty reads, non-repeatable reads, phantom reads, locking, and MVCC in:

```text
07-concurrency-and-isolation.md
```

---

# 17. Durability

**Durability** means that once a transaction successfully commits, its changes should survive subsequent failures covered by the database's durability guarantees.

Conceptually:

```text
Transaction
    ↓
COMMIT succeeds
    ↓
System crashes
    ↓
Database recovers
    ↓
Committed transaction remains
```

Database systems commonly use mechanisms such as:

```text
Write-ahead logging
Transaction logs
Persistent storage
Recovery procedures
```

to provide durability.

Implementation details differ by database.

---

# 18. Durability Does Not Mean "Data Can Never Be Lost"

This is a major interview trap.

Do not say:

> ACID durability guarantees committed data can never be lost under any circumstance.

Real systems can experience:

```text
Storage failure
Misconfiguration
Corruption
Operator mistakes
Region failure
Broken backups
Software defects
Catastrophic infrastructure failure
```

Durability guarantees operate within the failure model and configuration of the database system.

Reliable systems additionally use mechanisms such as:

```text
Backups
Replication
Point-in-time recovery
Disaster recovery
Monitoring
```

depending on requirements.

---

# 19. ACID Using One Example

Consider:

```text
Transfer $100 from Account A to Account B
```

Starting balances:

```text
A = $500
B = $300
```

Transaction:

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 100
WHERE id = 1;

UPDATE accounts
SET balance = balance + 100
WHERE id = 2;

COMMIT;
```

Expected result:

```text
A = $400
B = $400
```

Now map ACID to the example.

---

## Atomicity

Either both balance changes commit or the transaction does not commit the partial transfer.

```text
Debit A
+
Credit B
```

belong to one unit.

---

## Consistency

Defined invariants should remain satisfied.

For example, if the system enforces appropriate rules around account balances and transfer records, a successful transaction must respect those rules.

---

## Isolation

Other concurrent transactions should interact with this transfer according to the configured isolation guarantees.

They should not be allowed to create results that violate the guarantees promised by the chosen isolation model.

---

## Durability

Once the transfer commits successfully, the committed result should survive failures covered by the database's durability model.

---

# 20. Transaction Failure Scenarios

Transactions can fail for many reasons.

Examples:

```text
Constraint violation
Deadlock victim selection
Serialization failure
Timeout
Application exception
Connection failure
Database crash
Storage failure
Explicit rollback
```

Applications must be designed to handle transaction failures correctly.

---

# 21. Constraint Failure

Suppose:

```sql
CREATE TABLE candidates (
    id BIGINT PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL
);
```

A transaction attempts to insert a duplicate email.

The database may reject the statement because it violates:

```text
UNIQUE
```

Depending on the database and transaction behavior, the application may need to:

```text
Handle the error
Rollback
Retry appropriately
Return a meaningful response
```

---

# 22. Application Exception

Consider pseudocode:

```text
BEGIN

update interview
deduct credit

application exception

ROLLBACK
```

If the exception occurs before commit and the transaction is rolled back correctly, its database changes should not remain committed.

This is why transaction error handling matters.

---

# 23. Transaction Boundaries

One of the most important design decisions is:

> What operations should belong to the same transaction?

Consider:

```text
Create order
Reserve inventory
Create payment record
```

If these database changes must remain consistent together, they may belong to one transaction when they exist in the same transactional database and architecture.

But avoid putting unrelated work inside the transaction.

---

# 24. Keep Transactions Focused

Consider:

```text
BEGIN

Update database

Call external AI API
Wait 8 seconds

Send email

Call another service

COMMIT
```

This is often problematic.

The transaction remains open while waiting for external systems.

Possible consequences include:

```text
Locks held longer
Old snapshots retained longer
More contention
More open connections
Higher deadlock risk
Reduced throughput
```

Exact effects depend on the database and isolation model.

A better architecture often keeps the database transaction focused on the database changes that require atomicity.

---

# 25. Long-Running Transactions

A transaction that remains open for a long time can create operational problems.

Potential effects include:

```text
Lock contention
Blocking
Version retention
Resource usage
Delayed cleanup
Higher conflict probability
```

The specific impact depends on the DBMS.

General principle:

> Keep transactions as short as practical while still preserving the required correctness.

---

# 26. Transaction Scope Example

Suppose an interview completion flow does:

```text
1. Mark interview completed
2. Save score
3. Deduct credit
4. Update usage record
```

If these changes belong to the same transactional database and must remain consistent together:

```text
BEGIN

mark interview completed
save score
deduct credit
update usage

COMMIT
```

can be reasonable.

But suppose generating the interview report requires a 15-second AI request.

Holding the database transaction open while waiting for that request may be undesirable.

The workflow could instead be designed around separate stages depending on consistency requirements.

---

# 27. What Is Autocommit?

Many database clients and systems operate in an **autocommit** mode by default.

Conceptually:

```sql
UPDATE candidates
SET name = 'Alex'
WHERE id = 1;
```

may execute as its own transaction when no explicit transaction is open.

Conceptually:

```text
BEGIN
UPDATE
COMMIT
```

is handled automatically.

Exact behavior depends on:

```text
DBMS
Driver
Framework
Connection configuration
```

---

# 28. Why Autocommit Matters

Suppose the application executes:

```sql
UPDATE accounts
SET balance = balance - 100
WHERE id = 1;
```

and then separately:

```sql
UPDATE accounts
SET balance = balance + 100
WHERE id = 2;
```

If each statement commits independently:

```text
Transaction 1 → debit
COMMIT

Transaction 2 → credit
COMMIT
```

then the first statement could commit even if the second fails.

For operations that must be atomic together, use an explicit transaction boundary.

---

# 29. What Is a SAVEPOINT?

A **savepoint** creates a point inside a transaction to which part of the transaction can potentially be rolled back.

Example:

```sql
BEGIN;

UPDATE candidates
SET status = 'active'
WHERE id = 1;

SAVEPOINT candidate_updated;

UPDATE interview_credits
SET credits = credits - 1
WHERE candidate_id = 1;

ROLLBACK TO SAVEPOINT candidate_updated;

COMMIT;
```

Conceptually:

```text
BEGIN
  ↓
Operation A
  ↓
SAVEPOINT
  ↓
Operation B
  ↓
Problem
  ↓
Rollback B
  ↓
Keep A
  ↓
COMMIT
```

Syntax and exact semantics vary across database systems.

---

# 30. SAVEPOINT vs ROLLBACK

`ROLLBACK` commonly aborts the current transaction.

```sql
ROLLBACK;
```

A rollback to a savepoint can undo only the portion after the savepoint:

```sql
ROLLBACK TO SAVEPOINT my_savepoint;
```

This can be useful in complex transactions where partial recovery is intentionally designed.

---

# 31. Savepoints Do Not Mean Partial Commit

Suppose:

```text
Operation A
SAVEPOINT
Operation B
```

Rolling back B to the savepoint does not mean A is already committed.

A remains part of the current transaction until the transaction itself commits.

Conceptually:

```text
A executed
    ↓
Savepoint
    ↓
B rolled back
    ↓
A still uncommitted
    ↓
COMMIT
    ↓
A becomes committed
```

---

# 32. Transaction vs Savepoint

### Transaction

Defines the main atomic unit.

```text
BEGIN
...
COMMIT / ROLLBACK
```

### Savepoint

Defines an internal recovery point within that transaction.

```text
BEGIN
...
SAVEPOINT
...
ROLLBACK TO SAVEPOINT
...
COMMIT
```

A savepoint does not create an independent committed transaction.

---

# 33. Nested Transactions

Developers sometimes write application code that appears like:

```text
BEGIN outer

    BEGIN inner

    COMMIT inner

COMMIT outer
```

True independent nested transactions are not universally supported by relational databases.

Frameworks may simulate nested behavior using:

```text
Savepoints
```

or transaction propagation rules.

Never assume:

> Calling transaction() inside transaction() creates an independent transaction.

Check the DBMS and framework semantics.

---

# 34. Transactions and External APIs

Consider:

```text
BEGIN

Create payment record

Call payment provider

COMMIT
```

Problems can occur.

Suppose:

```text
External payment succeeds
```

but:

```text
Database transaction fails
```

Now:

```text
Customer charged
Database says no payment
```

A normal database rollback cannot undo an external payment API call.

---

# 35. Distributed Workflows

When operations span multiple systems:

```text
Database
Payment provider
Message broker
Email service
Another microservice
```

a local database transaction is insufficient to make the entire workflow atomic.

Common architectural patterns include:

```text
Transactional outbox
Saga
Idempotency keys
Retries
Compensating actions
Distributed transaction protocols
```

The correct pattern depends on the system.

This distinction is valuable in backend and system design interviews.

---

# 36. Transactional Outbox: Basic Idea

Suppose the application must:

```text
Update order
+
Publish OrderCreated event
```

A dangerous flow is:

```text
Update database
COMMIT
        ↓
Publish message
        ↓
Publish fails
```

Now the database says the order exists, but downstream systems never receive the event.

With an outbox pattern:

```text
BEGIN

Create order
Create outbox event

COMMIT
```

Both database records commit together.

A separate process later publishes the outbox event.

Conceptually:

```text
Database transaction
├── Order
└── Outbox Event
        ↓
Background publisher
        ↓
Message broker
```

This does not make the broker part of the database transaction, but it helps coordinate reliable event publication.

---

# 37. Idempotency

Suppose a transaction fails because of a transient concurrency error and the application retries.

You must consider:

> Is repeating the operation safe?

An **idempotent** operation can be repeated without producing unintended additional effects.

For example, payment and order workflows often use unique operation identifiers or idempotency keys to prevent duplicate effects.

Transactions and retries should be designed together.

---

# 38. Transactions and Retries

Some transaction failures are transient.

Examples may include:

```text
Serialization failure
Deadlock victim
Temporary connection issue
```

An application may retry certain operations.

But:

```text
Retry everything blindly
```

is dangerous.

Consider:

```text
Was an external API already called?
Could duplicate rows be created?
Could a payment happen twice?
Is the operation idempotent?
```

Retry logic should understand the failure mode.

---

# 39. Transactions and Locks

Transactions often interact with locking mechanisms.

Example:

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 100
WHERE id = 1;
```

The database may acquire locks associated with the affected data.

Those locks may remain until a transaction boundary depending on the DBMS and operation.

Long transactions can therefore increase contention.

Locking is covered more deeply in:

```text
07-concurrency-and-isolation.md
```

---

# 40. Transactions and MVCC

Many modern relational databases use **Multi-Version Concurrency Control (MVCC)** or related versioning mechanisms.

Instead of making every reader wait for every writer, the database may allow transactions to observe appropriate versions of rows.

Conceptually:

```text
Old version
    ↓
Reader

New version
    ↓
Writer
```

The exact visibility rules depend on:

```text
DBMS
Isolation level
Transaction snapshot
```

MVCC is covered in the concurrency guide.

---

# 41. Transaction Logs

Databases commonly maintain transaction-related logs.

A simplified idea:

```text
Database change
      ↓
Record recovery information
      ↓
Persist according to durability protocol
      ↓
Commit
```

These logs can help recover committed changes after failures.

Different systems use terminology such as:

```text
WAL
Redo log
Transaction log
```

Exact implementations differ.

---

# 42. Write-Ahead Logging

**Write-Ahead Logging (WAL)** is a technique where information needed to recover a change is written to a log before the corresponding data-page changes are considered safely persisted.

Simplified:

```text
Change requested
      ↓
Write log record
      ↓
Persist required log information
      ↓
Commit can succeed
      ↓
Data pages may be persisted later
```

This allows recovery mechanisms to reconstruct committed state after a crash.

Exact durability behavior depends on database configuration.

---

# 43. Why Doesn't COMMIT Require Every Data Page to Be Written Immediately?

Writing every modified table page to its final storage location before returning from every commit could be expensive.

With logging mechanisms, the database can often persist sufficient recovery information first.

Conceptually:

```text
Transaction changes
       ↓
Durable log
       ↓
COMMIT
       ↓
Data pages written later
```

After a crash, recovery can use the log to restore the required committed state.

Implementation details vary by DBMS.

---

# 44. Atomicity vs Durability

These are often confused.

### Atomicity

Question:

```text
Did the transaction happen completely or not?
```

Concern:

```text
Partial transaction effects
```

### Durability

Question:

```text
After successful commit, will the result survive covered failures?
```

Concern:

```text
Persistence after commit
```

They solve different problems.

---

# 45. Consistency vs Isolation

Also frequently confused.

### Consistency

Concerned with:

```text
Preserving defined database invariants
```

### Isolation

Concerned with:

```text
Interaction between concurrent transactions
```

Isolation can be necessary to preserve certain application invariants under concurrency, but the concepts are distinct.

---

# 46. Atomicity vs Isolation

### Atomicity

Controls whether one transaction's changes are applied as a complete unit.

### Isolation

Controls interactions between multiple concurrent transactions.

Example:

```text
Transaction A partially fails
```

is primarily an atomicity concern.

```text
Transaction A and B interfere concurrently
```

is primarily an isolation concern.

---

# 47. Transaction Example: Interview Credits

Suppose a candidate has:

```text
credits = 1
```

Two requests arrive at almost the same time:

```text
Request A → Start interview
Request B → Start interview
```

A naive application might do:

```text
Read credits
If credits > 0:
    Start interview
    credits = credits - 1
```

Both requests could potentially read:

```text
credits = 1
```

before either writes the new value.

This is a concurrency problem.

Simply saying:

> Use a transaction.

may not be sufficient.

The transaction must also use an appropriate concurrency-control strategy.

Possible techniques include:

```text
Atomic conditional UPDATE
Row locking
Serializable transaction
Optimistic concurrency control
```

depending on the system.

This is why **ACID isolation** matters.

---

# 48. Atomic Conditional Update

Instead of:

```text
Read credits
Check credits
Update credits
```

some systems can express the condition directly in one statement:

```sql
UPDATE candidate_credits
SET credits = credits - 1
WHERE candidate_id = 100
  AND credits > 0;
```

Then the application checks whether a row was updated.

This can reduce race conditions by moving the condition and update into one database operation.

The full correctness still depends on the surrounding workflow and DBMS semantics.

---

# 49. Transactions Do Not Fix Bad Logic

Suppose:

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 1000
WHERE id = 1;

COMMIT;
```

If the application should prevent negative balances but does not check or constrain them, the transaction can successfully commit incorrect business behavior.

ACID does not mean:

```text
Business logic is automatically correct.
```

Transactions provide mechanisms for correctness.

Developers still need to define and enforce the correct rules.

---

# 50. Transactions Do Not Automatically Prevent Race Conditions

Another major misconception:

> If I use BEGIN and COMMIT, concurrency problems disappear.

Incorrect.

Two transactions can execute concurrently.

Correctness may depend on:

```text
Isolation level
Locks
MVCC behavior
Atomic statements
Optimistic concurrency
Constraints
Retry logic
```

Transactions provide the boundary.

Concurrency control determines how those transactions interact.

---

# 51. Transaction Boundary Anti-Pattern

Avoid transactions that contain unnecessary work:

```text
BEGIN

SELECT data

Call AI service
Wait 10 seconds

Process large file

Send HTTP request

UPDATE database

COMMIT
```

Potential result:

```text
Transaction open for 15+ seconds
```

This can consume database resources unnecessarily.

Prefer to determine:

```text
Which database operations actually require one atomic boundary?
```

and keep that boundary focused.

---

# 52. But Don't Make Transactions Too Small Either

Suppose:

```text
Operation A
Operation B
Operation C
```

must either all succeed or all fail.

Creating three independent transactions:

```text
Transaction 1 → A → COMMIT
Transaction 2 → B → COMMIT
Transaction 3 → C → COMMIT
```

breaks the atomic relationship.

If B fails:

```text
A remains committed
```

Therefore:

> Keep transactions short, but not shorter than the consistency requirement.

---

# 53. Transaction Boundary Framework

Ask:

```text
What invariant am I protecting?
        ↓
Which database operations must succeed together?
        ↓
Can they participate in one local transaction?
        ↓
What concurrency problems can occur?
        ↓
What external operations are involved?
        ↓
Can external work happen outside the transaction?
        ↓
How will failures and retries be handled?
```

This is a much stronger approach than wrapping random functions in transactions.

---

# 54. Common Interview Question: What Is a Transaction?

Weak answer:

> A transaction is a group of queries.

Better answer:

> A transaction is a logical unit of database work containing one or more operations that should be handled together according to transactional guarantees. It establishes a boundary within which changes can be committed or rolled back, while properties such as atomicity, isolation, and durability govern important aspects of correctness and failure handling.

---

# 55. Common Interview Question: Explain ACID

A strong answer:

> ACID describes four important transaction properties. Atomicity ensures a transaction does not commit only part of its intended database changes. Consistency means transactions preserve defined database invariants. Isolation controls interactions between concurrent transactions according to the chosen isolation guarantees. Durability means successfully committed changes survive failures covered by the database's durability model.

Then give a concrete example.

---

# 56. Common Interview Question: Atomicity vs Consistency

### Atomicity

```text
All-or-nothing transaction behavior
```

### Consistency

```text
Preservation of defined invariants
```

Example:

Atomicity can ensure both sides of a transfer commit together.

Consistency concerns whether the resulting database state satisfies the system's enforced rules.

---

# 57. Common Interview Question: Isolation vs Consistency

A useful answer:

> Consistency describes whether defined invariants remain valid, while isolation describes how concurrent transactions interact. Stronger isolation can help applications preserve certain invariants under concurrency, but isolation does not define the business invariants itself.

---

# 58. Common Interview Question: Durability vs Backup

They are not the same thing.

### Durability

Concerned with preserving successfully committed transactions across failures covered by the database's durability mechanism.

### Backup

Creates recoverable copies of data for scenarios such as:

```text
Accidental deletion
Corruption
Infrastructure failure
Disaster recovery
```

A durable database still needs a backup and recovery strategy.

---

# 59. Common Interview Question: COMMIT vs ROLLBACK

### COMMIT

Successfully completes the transaction and makes its changes committed.

### ROLLBACK

Aborts the transaction and discards its uncommitted changes.

Simple:

```text
COMMIT   → Keep transaction
ROLLBACK → Abort transaction
```

---

# 60. Common Interview Question: Why Use SAVEPOINT?

A savepoint provides a recovery point inside a transaction.

Instead of aborting the entire transaction, the application may roll back work performed after a savepoint while retaining earlier uncommitted work.

The remaining transaction still needs to eventually:

```text
COMMIT
```

or:

```text
ROLLBACK
```

---

# 61. Common Interview Question: Why Are Long Transactions Bad?

Potential reasons include:

```text
Locks held longer
Higher contention
More version retention
More resource usage
Higher deadlock/conflict risk
Lower throughput
```

The exact impact depends on the DBMS.

A good answer should not claim every long transaction causes all of these effects.

---

# 62. Common Interview Question: Can a Transaction Include an API Call?

Technically, application code can call an API while a database transaction remains open.

The better question is:

> Should it?

Often, holding a transaction open while waiting for an external service is undesirable.

And the external operation is usually not automatically rolled back if the database transaction fails.

This requires architectural handling rather than assuming local transaction atomicity extends across systems.

---

# 63. Common Interview Questions

Be prepared to answer:

1. What is a transaction?
2. Why do databases need transactions?
3. What is COMMIT?
4. What is ROLLBACK?
5. What is ACID?
6. What is atomicity?
7. What is consistency?
8. What is isolation?
9. What is durability?
10. Atomicity vs consistency?
11. Consistency vs isolation?
12. Atomicity vs isolation?
13. Durability vs backup?
14. What happens if a transaction fails before commit?
15. What is autocommit?
16. When should you use an explicit transaction?
17. What is a savepoint?
18. Savepoint vs transaction?
19. Do savepoints commit data?
20. What are long-running transactions?
21. Why can long transactions be problematic?
22. Can database transactions include external APIs?
23. Does ACID guarantee no data loss?
24. Does using a transaction prevent race conditions?
25. Can transactions span multiple services?
26. What is write-ahead logging?
27. Why are transaction logs useful?
28. How would you design a money-transfer transaction?
29. How would you safely deduct limited credits?
30. How should transaction retries be handled?

---

# 64. Common Mistakes

## Mistake 1: ACID means data can never be lost

Incorrect.

Durability operates within the database's configured guarantees and failure model.

Backups and disaster recovery are still necessary.

---

## Mistake 2: Consistency means every user sees identical data immediately

Incorrect in the ACID context.

ACID consistency primarily concerns preserving defined invariants.

---

## Mistake 3: Isolation means transactions run sequentially

Incorrect.

Databases can execute transactions concurrently while providing isolation guarantees.

---

## Mistake 4: BEGIN and COMMIT prevent every race condition

Incorrect.

Concurrency behavior depends on:

```text
Isolation
Locks
MVCC
Constraints
Atomic operations
```

---

## Mistake 5: ROLLBACK can undo an external API call

Incorrect.

A database rollback affects operations participating in that database transaction.

It does not automatically undo:

```text
Email
Payment API
HTTP request
Message already published
```

---

## Mistake 6: Savepoint means partial commit

Incorrect.

Changes before the savepoint remain uncommitted until the transaction itself commits.

---

## Mistake 7: Longer transactions are safer

Not necessarily.

Keeping a transaction open unnecessarily can increase contention and resource usage.

---

## Mistake 8: Smaller transactions are always better

Incorrect.

Operations that must succeed together require an appropriate shared transaction boundary.

---

## Mistake 9: ACID makes bad application logic correct

Incorrect.

The database cannot enforce business rules that have not been correctly expressed.

---

## Mistake 10: Transactions are only needed for financial systems

Incorrect.

Transactions are useful whenever multiple database operations must preserve a consistency requirement.

Examples include:

```text
Orders
Inventory
Credits
Subscriptions
Interview attempts
User registration
Reservations
```

---

# 65. Interview Scenario: Money Transfer

**Question:**

How would you transfer $100 from Account A to Account B?

A weak answer:

```sql
UPDATE accounts
SET balance = balance - 100
WHERE id = 1;

UPDATE accounts
SET balance = balance + 100
WHERE id = 2;
```

Problem:

The first statement may succeed while the second fails.

Better:

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 100
WHERE id = 1;

UPDATE accounts
SET balance = balance + 100
WHERE id = 2;

COMMIT;
```

But a strong interview answer goes further.

Ask:

```text
Does Account A have enough balance?
Can two transfers happen concurrently?
Can the balance become negative?
What isolation/locking strategy is needed?
Should a transfer ledger be recorded?
What happens on deadlock or serialization failure?
How are retries made idempotent?
```

The transaction boundary is only one part of the solution.

---

# 66. Interview Scenario: Limited Interview Credits

Suppose:

```text
candidate_id = 100
credits = 1
```

Two interview-start requests arrive concurrently.

A naive flow:

```text
Read credits
Check > 0
Start interview
Deduct credit
```

can race.

A better database operation might be:

```sql
UPDATE candidate_credits
SET credits = credits - 1
WHERE candidate_id = 100
  AND credits > 0;
```

Then verify that exactly one row was affected before proceeding according to the workflow.

This demonstrates an important principle:

> Put critical state transitions as close as possible to the database operation that protects them.

---

# 67. Interview Scenario: Order Creation

Suppose creating an order requires:

```text
Create order
Create order items
Reserve inventory
```

These database changes may need to succeed together.

Conceptually:

```text
BEGIN

Create order
Create items
Reserve inventory

COMMIT
```

But payment through an external provider introduces another system.

Then the architecture must consider:

```text
Retries
Idempotency
Compensation
Outbox events
Workflow state
```

A local database transaction alone cannot provide global atomicity.

---

# 68. ACID Mental Model

Remember:

```text
Atomicity
    ↓
Did the transaction happen completely?

Consistency
    ↓
Are defined invariants preserved?

Isolation
    ↓
How do concurrent transactions interact?

Durability
    ↓
Does committed state survive covered failures?
```

Do not memorize only:

```text
A C I D
```

Understand the question each property answers.

---

# 69. Transaction Design Mental Model

For a real workflow:

```text
Business invariant
        ↓
Identify database state changes
        ↓
Determine atomic boundary
        ↓
Choose concurrency strategy
        ↓
Keep transaction focused
        ↓
COMMIT / ROLLBACK
        ↓
Handle failures
        ↓
Retry safely when appropriate
```

If external systems are involved:

```text
Local transaction
        +
Outbox / Saga / Idempotency / Compensation
```

may be required depending on the architecture.

---

# 70. Final Takeaway

Transactions are not merely:

```text
BEGIN
...
COMMIT
```

They are a mechanism for protecting logical units of database work.

A strong database engineer reasons about:

```text
Atomicity
Consistency
Isolation
Durability
Transaction boundaries
Concurrency
Failure handling
Retries
External side effects
```

The key principle is:

> Define the invariant first, then design the transaction that protects it.

And remember:

```text
Transaction ≠ automatic correctness

ACID ≠ no possible data loss

Isolation ≠ no concurrency

ROLLBACK ≠ undo external side effects

COMMIT ≠ backup
```

Understanding these distinctions is far more valuable in interviews than memorizing the ACID acronym.

---

## Previous

**[← Indexing](./05-indexing.md)**

## Next

**[Concurrency and Isolation →](./07-concurrency-and-isolation.md)**

---

Maintained by **InterviewEraHQ**.
