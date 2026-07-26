# Normalization in DBMS

Normalization is a database design technique used to organize relational data, reduce unnecessary redundancy, and prevent anomalies caused by duplicated or poorly structured data.

In interviews, normalization is often taught as:

```text
1NF → 2NF → 3NF → BCNF
```

But memorizing definitions is not enough.

The important questions are:

- Why does redundancy become a problem?
- What is a functional dependency?
- What exactly changes between 1NF, 2NF, 3NF, and BCNF?
- How do we decompose a relation correctly?
- When might denormalization be useful?

This guide builds those concepts progressively.

---

# 1. Why Do We Need Normalization?

Consider an interview platform storing candidate interview information in one table:

| candidate_id | candidate_name | candidate_email | interview_id | interviewer | interviewer_email |
|---:|---|---|---:|---|---|
| 1 | Alex | alex@example.com | 101 | Maya | maya@example.com |
| 1 | Alex | alex@example.com | 102 | Rahul | rahul@example.com |
| 2 | Sam | sam@example.com | 103 | Maya | maya@example.com |

Several values are repeated.

For Candidate 1:

```text
Alex
alex@example.com
```

appear multiple times.

For interviewer Maya:

```text
Maya
maya@example.com
```

also appear multiple times.

This is **data redundancy**.

Redundancy is not automatically wrong, but uncontrolled redundancy can create consistency problems.

---

# 2. What Is Data Redundancy?

**Data redundancy** means storing the same fact in multiple places when that duplication is unnecessary for the intended design.

Example:

```text
candidate_id = 1
candidate_name = Alex
candidate_email = alex@example.com
```

appears in every interview row for Alex.

Suppose Alex completes 100 interviews.

The same candidate information might then appear 100 times.

This increases:

- Storage duplication
- Update complexity
- Risk of inconsistent values

Normalization helps organize such facts into appropriate relations.

---

# 3. What Are Database Anomalies?

Poorly structured redundant data can produce several types of anomalies.

The three commonly discussed in interviews are:

```text
Update Anomaly
Insertion Anomaly
Deletion Anomaly
```

---

## 3.1 Update Anomaly

Suppose Alex changes email from:

```text
alex@example.com
```

to:

```text
alex.kumar@example.com
```

If Alex appears in many rows, every relevant row must be updated.

If only some rows are updated:

| candidate_id | candidate_name | candidate_email | interview_id |
|---:|---|---|---:|
| 1 | Alex | alex.kumar@example.com | 101 |
| 1 | Alex | alex@example.com | 102 |

Now the database contains conflicting information about the same candidate.

This is an **update anomaly**.

---

## 3.2 Insertion Anomaly

Suppose we want to store a new candidate before they have completed any interview.

If the table design requires:

```text
interview_id
interviewer
```

to create a row, we may be unable to store the candidate independently.

The design has incorrectly tied candidate existence to interview existence.

This is an **insertion anomaly**.

---

## 3.3 Deletion Anomaly

Suppose Candidate 2 has exactly one interview:

| candidate_id | candidate_name | interview_id |
|---:|---|---:|
| 2 | Sam | 103 |

If we delete interview `103`, we might accidentally delete the only stored information about Sam.

This is a **deletion anomaly**.

---

# 4. Goal of Normalization

Normalization attempts to organize data so that individual facts are stored in appropriate relations.

Instead of:

```text
Candidate + Interview + Interviewer
```

all in one table, we might model:

```text
Candidates
Interviews
Interviewers
```

with relationships between them.

Conceptually:

```text
Candidates
    |
    |
Interviews
    |
    |
Interviewers
```

This can reduce unnecessary duplication and make dependencies clearer.

---

# 5. Functional Dependency

Functional dependency is one of the most important concepts behind normalization.

Suppose we have attributes:

```text
candidate_id
candidate_name
candidate_email
```

If each `candidate_id` identifies exactly one candidate name and email, we can write:

```text
candidate_id → candidate_name
candidate_id → candidate_email
```

This means:

> Given a candidate ID, the corresponding candidate name and email are determined.

The left side is called the **determinant**.

---

## Functional Dependency Notation

```text
X → Y
```

means:

> If two tuples agree on X, they must also agree on Y.

Example:

```text
candidate_id → candidate_email
```

If:

```text
candidate_id = 10
```

always corresponds to:

```text
candidate_email = alex@example.com
```

then candidate email is functionally dependent on candidate ID.

---

# 6. Functional Dependency Is About Constraints, Not Coincidence

Consider:

| candidate_id | city |
|---:|---|
| 1 | Delhi |
| 2 | Delhi |
| 3 | Noida |

From this small dataset, you should not conclude:

```text
city → candidate_id
```

because multiple candidates can belong to the same city.

Functional dependencies describe rules that must hold for valid states of the relation, not patterns that happen to exist in one sample.

This distinction is important in normalization questions.

---

# 7. Trivial Functional Dependency

A functional dependency:

```text
X → Y
```

is trivial when:

```text
Y ⊆ X
```

Example:

```text
(candidate_id, email) → candidate_id
```

This is trivially true because `candidate_id` is already part of the determinant.

---

# 8. Full Functional Dependency

Suppose a relation has composite key:

```text
(student_id, course_id)
```

and:

```text
(student_id, course_id) → grade
```

If neither:

```text
student_id → grade
```

nor:

```text
course_id → grade
```

holds, then `grade` depends on the **entire composite key**.

This is a full functional dependency.

---

# 9. Partial Dependency

A **partial dependency** occurs when a non-prime attribute depends on only part of a composite candidate key.

Consider:

```text
Enrollment(
    student_id,
    course_id,
    student_name,
    course_name,
    grade
)
```

Suppose:

```text
(student_id, course_id)
```

is the candidate key.

Dependencies:

```text
student_id → student_name

course_id → course_name

(student_id, course_id) → grade
```

Here:

```text
student_name
```

depends only on:

```text
student_id
```

which is part of the composite key.

Similarly:

```text
course_name
```

depends only on:

```text
course_id
```

These are partial dependencies.

Partial dependencies are central to understanding **Second Normal Form (2NF)**.

---

# 10. Transitive Dependency

Consider:

```text
Employee(
    employee_id,
    employee_name,
    department_id,
    department_name
)
```

Suppose:

```text
employee_id → department_id
```

and:

```text
department_id → department_name
```

Therefore:

```text
employee_id → department_name
```

through `department_id`.

Conceptually:

```text
employee_id
      ↓
department_id
      ↓
department_name
```

This type of dependency is relevant when understanding **Third Normal Form (3NF)**.

---

# 11. Prime and Non-Prime Attributes

A **prime attribute** is an attribute that belongs to at least one candidate key.

A **non-prime attribute** does not belong to any candidate key.

Example:

```text
Enrollment(
    student_id,
    course_id,
    grade
)
```

If:

```text
(student_id, course_id)
```

is the only candidate key, then:

```text
student_id
course_id
```

are prime attributes.

And:

```text
grade
```

is non-prime.

This terminology becomes important in formal definitions of 2NF and 3NF.

---

# 12. First Normal Form — 1NF

A relation is generally considered to be in **First Normal Form (1NF)** when each attribute contains atomic values appropriate to the relational model and there are no repeating groups represented as multiple values inside a single attribute.

Consider:

| candidate_id | name | skills |
|---:|---|---|
| 1 | Alex | Java, Python, SQL |
| 2 | Sam | React, TypeScript |

The `skills` column contains multiple skill values in a single field.

A more relational representation would be:

### Candidates

| candidate_id | name |
|---:|---|
| 1 | Alex |
| 2 | Sam |

### Candidate Skills

| candidate_id | skill |
|---:|---|
| 1 | Java |
| 1 | Python |
| 1 | SQL |
| 2 | React |
| 2 | TypeScript |

Each skill is now represented as its own value in a tuple.

---

## What Does Atomic Mean?

Atomic does not mean:

> The value cannot possibly be divided further.

For example:

```text
Alex Kumar
```

could technically be split into:

```text
Alex
Kumar
```

But whether it should be depends on what the application treats as the value.

Atomicity in this context is about the domain and relational representation, not physical indivisibility.

---

## 1NF Interview Summary

A simplified interview answer:

> First Normal Form requires a relational structure without repeating groups or multiple independent values packed into a single attribute. Each attribute value should represent a single value from its domain for that tuple.

---

# 13. Second Normal Form — 2NF

A relation is in **Second Normal Form (2NF)** when:

1. It is in 1NF.
2. Every non-prime attribute is fully functionally dependent on every candidate key.

In practical interview examples, 2NF is mainly about eliminating **partial dependencies** on composite candidate keys.

---

## Example Before 2NF

Consider:

```text
Enrollment(
    student_id,
    course_id,
    student_name,
    course_name,
    grade
)
```

Candidate key:

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

depends only on `student_id`.

And:

```text
course_name
```

depends only on `course_id`.

These are partial dependencies.

---

# 14. Convert the Relation to 2NF

Separate student information:

```text
Students(
    student_id,
    student_name
)
```

Separate course information:

```text
Courses(
    course_id,
    course_name
)
```

Keep relationship-specific information:

```text
Enrollments(
    student_id,
    course_id,
    grade
)
```

Now:

```text
student_id → student_name
```

belongs in `Students`.

```text
course_id → course_name
```

belongs in `Courses`.

And:

```text
(student_id, course_id) → grade
```

remains in `Enrollments`.

The partial dependencies have been removed from the enrollment relation.

---

## Important 2NF Interview Point

If every candidate key consists of a single attribute, partial dependency on a proper subset of a candidate key cannot occur.

Therefore, such a relation that is already in 1NF is automatically in 2NF.

---

# 15. Third Normal Form — 3NF

A formal definition of **Third Normal Form (3NF)** is:

For every non-trivial functional dependency:

```text
X → A
```

at least one of the following must hold:

1. `X` is a super key, or
2. `A` is a prime attribute.

A common interview simplification is:

> A relation should be in 2NF and should not contain problematic transitive dependencies of non-key attributes on candidate keys through other non-key attributes.

The formal definition is more accurate.

---

# 16. Example Before 3NF

Consider:

```text
Employees(
    employee_id,
    employee_name,
    department_id,
    department_name
)
```

Suppose:

```text
employee_id → employee_name

employee_id → department_id

department_id → department_name
```

Therefore:

```text
employee_id → department_name
```

through:

```text
department_id
```

Conceptually:

```text
employee_id
      ↓
department_id
      ↓
department_name
```

Department name is being stored repeatedly for employees in the same department.

---

# 17. Convert the Relation to 3NF

Create:

```text
Employees(
    employee_id,
    employee_name,
    department_id
)
```

and:

```text
Departments(
    department_id,
    department_name
)
```

Now department information is stored independently.

Instead of:

| employee_id | employee_name | department_id | department_name |
|---:|---|---:|---|
| 1 | Alex | 10 | Engineering |
| 2 | Sam | 10 | Engineering |
| 3 | Priya | 20 | Product |

we have:

### Employees

| employee_id | employee_name | department_id |
|---:|---|---:|
| 1 | Alex | 10 |
| 2 | Sam | 10 |
| 3 | Priya | 20 |

### Departments

| department_id | department_name |
|---:|---|
| 10 | Engineering |
| 20 | Product |

This reduces repeated department information.

---

# 18. Boyce-Codd Normal Form — BCNF

**Boyce-Codd Normal Form (BCNF)** is stricter than 3NF.

A relation is in BCNF if, for every non-trivial functional dependency:

```text
X → Y
```

`X` is a super key.

Simplified:

> Every determinant of a non-trivial functional dependency must be a super key.

---

# 19. 3NF vs BCNF

This is one of the most common normalization interview questions.

### 3NF

For:

```text
X → A
```

3NF allows the dependency when either:

```text
X is a super key
```

or:

```text
A is a prime attribute
```

### BCNF

BCNF requires:

```text
X must be a super key
```

for every non-trivial functional dependency.

Therefore:

```text
BCNF ⇒ 3NF
```

but:

```text
3NF ⇏ BCNF
```

A relation can satisfy 3NF while violating BCNF.

---

# 20. Example: 3NF but Not BCNF

Consider:

```text
Teaching(
    student,
    course,
    instructor
)
```

Assume these business rules:

1. Each course has one instructor.
2. Each instructor teaches only one course.
3. A student may take multiple courses.

Functional dependencies:

```text
course → instructor

instructor → course
```

Candidate keys include:

```text
(student, course)

(student, instructor)
```

Therefore:

```text
course
instructor
```

are prime attributes because they appear in candidate keys.

Now consider:

```text
course → instructor
```

`course` alone is not a super key because many students can take the same course.

But `instructor` is a prime attribute.

Therefore this dependency can satisfy 3NF.

However, it violates BCNF because:

```text
course
```

is not a super key.

This demonstrates how BCNF can be stricter than 3NF.

---

# 21. Normal Form Hierarchy

Conceptually:

```text
BCNF
 ↓
3NF
 ↓
2NF
 ↓
1NF
```

Meaning:

```text
BCNF ⇒ 3NF ⇒ 2NF ⇒ 1NF
```

A BCNF relation is also in 3NF.

A 3NF relation is also in 2NF.

A 2NF relation is also in 1NF.

The reverse implications do not generally hold.

---

# 22. What Is Decomposition?

**Decomposition** means splitting a relation into smaller relations.

Example:

```text
Employees(
    employee_id,
    employee_name,
    department_id,
    department_name
)
```

can be decomposed into:

```text
Employees(
    employee_id,
    employee_name,
    department_id
)
```

and:

```text
Departments(
    department_id,
    department_name
)
```

But splitting tables arbitrarily is not enough.

A useful decomposition should preserve important properties.

Two concepts frequently discussed are:

```text
Lossless Join
Dependency Preservation
```

---

# 23. Lossless Join Decomposition

A decomposition is **lossless** when joining the decomposed relations reconstructs exactly the original relation, without introducing spurious tuples or losing valid information represented by the original relation.

Suppose:

```text
Employees(
    employee_id,
    employee_name,
    department_id
)
```

and:

```text
Departments(
    department_id,
    department_name
)
```

Joining them through:

```text
department_id
```

should reconstruct the intended employee-department information.

Conceptually:

```text
Employees
    +
Departments
    ↓
JOIN
    ↓
Original information
```

---

## Why Lossless Decomposition Matters

Imagine decomposing a table in a way that causes joins to generate combinations that never existed.

Those extra combinations are called **spurious tuples**.

Normalization should not destroy the semantics of the original relation.

---

# 24. Dependency Preservation

A decomposition is **dependency preserving** when the original functional dependencies can be enforced by checking constraints on the decomposed relations individually, without requiring joins to verify them.

This matters because enforcing dependencies across multiple joined relations can be more complicated.

---

# 25. Lossless Join vs Dependency Preservation

They are different properties.

### Lossless Join

Concerned with:

```text
Can we reconstruct the original relation correctly?
```

### Dependency Preservation

Concerned with:

```text
Can we enforce the original functional dependencies directly on the decomposed relations?
```

A decomposition can be lossless but not preserve every dependency.

This becomes particularly relevant when comparing 3NF and BCNF decomposition strategies.

---

# 26. Does Normalization Always Improve Performance?

No.

Normalization primarily improves **data organization and integrity characteristics**.

It can reduce:

- Redundancy
- Update anomalies
- Insertion anomalies
- Deletion anomalies

But normalized schemas may require additional joins.

For some workloads:

```text
More joins
    ↓
More query complexity
    ↓
Potential additional execution cost
```

Performance depends on:

- Queries
- Indexes
- Data volume
- Join strategies
- Caching
- Database engine
- Hardware
- Access patterns
- Distribution

Normalization should not be described as a universal performance optimization.

---

# 27. What Is Denormalization?

**Denormalization** is the intentional introduction of redundancy or combining of data that could otherwise be represented in a more normalized form, usually to satisfy particular read, performance, or operational requirements.

Example:

Suppose an analytics dashboard frequently needs:

```text
candidate_name
interview_count
average_score
```

Computing aggregates repeatedly across large datasets might be expensive.

A system might maintain precomputed or duplicated information such as:

```text
candidate_statistics
```

containing:

```text
candidate_id
interview_count
average_score
```

This introduces additional synchronization requirements but can improve certain read workloads.

---

# 28. Normalization vs Denormalization

### Normalization

Often prioritizes:

- Reduced redundancy
- Clear dependencies
- Data integrity
- Easier consistency management

Potential cost:

- More relations
- More joins
- More complex read queries

### Denormalization

May prioritize:

- Faster specific reads
- Fewer joins
- Precomputed information
- Workload-specific access patterns

Potential cost:

- Redundant data
- More complex writes
- Synchronization requirements
- Risk of inconsistency

Neither approach is universally better.

The design should reflect the workload.

---

# 29. Practical Example

Suppose we begin with:

```text
InterviewRecords(
    candidate_id,
    candidate_name,
    candidate_email,
    interview_id,
    interviewer_id,
    interviewer_name,
    interviewer_email,
    score
)
```

Possible dependencies:

```text
candidate_id → candidate_name, candidate_email

interviewer_id → interviewer_name, interviewer_email

interview_id → candidate_id, interviewer_id, score
```

Candidate information repeats across interviews.

Interviewer information also repeats across interviews.

A more normalized design could be:

```text
Candidates(
    candidate_id,
    candidate_name,
    candidate_email
)
```

```text
Interviewers(
    interviewer_id,
    interviewer_name,
    interviewer_email
)
```

```text
Interviews(
    interview_id,
    candidate_id,
    interviewer_id,
    score
)
```

Relationships:

```text
Candidates
    |
    | candidate_id
    ↓
Interviews
    ↑
    | interviewer_id
    |
Interviewers
```

Each entity's attributes are now associated with the entity they describe.

---

# 30. Normalization Is Not "Create More Tables"

A common misconception is:

> More tables means the database is more normalized.

Incorrect.

Normalization is based on:

- Functional dependencies
- Candidate keys
- Normal-form rules
- Correct decomposition

Splitting:

```text
Users
```

into 20 arbitrary tables does not automatically improve normalization.

Normalization is about representing dependencies correctly.

---

# 31. Is 3NF Always Enough?

There is no universal answer.

Many practical relational schemas aim for structures around:

```text
3NF
```

or:

```text
BCNF
```

where appropriate.

But production database design also considers:

- Query patterns
- Performance
- Data volume
- Operational complexity
- Reporting requirements
- Caching
- Distributed architecture
- Consistency requirements

Normal forms are design tools, not a command to normalize every schema as far as theoretically possible.

---

# 32. Common Interview Questions

Make sure you can explain these clearly.

### Fundamentals

1. What is normalization?
2. Why is normalization needed?
3. What is data redundancy?
4. What is an update anomaly?
5. What is an insertion anomaly?
6. What is a deletion anomaly?
7. What is a functional dependency?
8. What is a determinant?
9. What is a partial dependency?
10. What is a transitive dependency?

### Normal Forms

11. What is 1NF?
12. What is 2NF?
13. What is 3NF?
14. What is BCNF?
15. What is the difference between 2NF and 3NF?
16. What is the difference between 3NF and BCNF?
17. Can a relation be in 3NF but not BCNF?
18. If a relation is in BCNF, is it also in 3NF?
19. If all candidate keys are single-column keys, can partial dependency occur on a candidate key?

### Decomposition

20. What is decomposition?
21. What is lossless decomposition?
22. What is dependency preservation?
23. What are spurious tuples?
24. Can a decomposition be lossless but not dependency preserving?

### Practical Design

25. Does normalization always improve performance?
26. What is denormalization?
27. When might denormalization be useful?
28. Should every production database be normalized to BCNF?
29. Can normalized databases still have performance problems?
30. How would you decide between normalization and denormalization?

---

# 33. Common Mistakes

## Mistake 1: Normalization means removing all duplicate values

Incorrect.

Some values can naturally repeat.

For example:

| candidate_id | city |
|---:|---|
| 1 | Delhi |
| 2 | Delhi |

The repeated value `Delhi` does not automatically indicate a normalization problem.

Normalization is about dependencies and redundancy of facts, not eliminating every repeated string.

---

## Mistake 2: 1NF means every value is physically indivisible

Incorrect.

Atomicity is relative to the domain and how the application models the attribute.

A name does not necessarily need to be split into individual characters or components to satisfy 1NF.

---

## Mistake 3: 2NF means there are no composite keys

Incorrect.

Composite keys are perfectly valid.

2NF addresses partial dependencies of non-prime attributes on candidate keys.

---

## Mistake 4: 3NF means there are no functional dependencies

Incorrect.

Functional dependencies continue to exist.

3NF restricts which functional dependencies are acceptable based on determinants, super keys, and prime attributes.

---

## Mistake 5: 3NF and BCNF are the same

Incorrect.

BCNF is stricter.

3NF permits certain dependencies where the determinant is not a super key if the dependent attribute is prime.

BCNF does not provide that exception.

---

## Mistake 6: BCNF is always the best production design

Incorrect.

Database design involves practical trade-offs.

BCNF may be desirable, but dependency preservation, query patterns, operational complexity, and performance requirements also matter.

---

## Mistake 7: Normalization always makes queries faster

Incorrect.

Normalization can require additional joins.

Performance must be evaluated for the actual workload.

---

## Mistake 8: Denormalization means bad database design

Incorrect.

Intentional denormalization can be a valid engineering decision when its trade-offs are understood and controlled.

Accidental redundancy and deliberate denormalization are not the same thing.

---

# 34. Interview Answer: What Is Normalization?

A weak answer:

> Normalization removes duplicate data from databases.

This is incomplete.

A stronger answer:

> Normalization is a relational database design process that organizes attributes and relations based on keys and functional dependencies. Its goals include reducing unnecessary redundancy and preventing update, insertion, and deletion anomalies. Normal forms such as 1NF, 2NF, 3NF, and BCNF progressively impose stronger structural conditions, although practical database design also considers performance and workload requirements.

---

# 35. Interview Answer: 2NF vs 3NF

A useful explanation:

> Second Normal Form requires a relation to be in 1NF and removes partial dependencies of non-prime attributes on candidate keys. Third Normal Form goes further by restricting dependencies where non-key determinants can cause attributes to depend indirectly on candidate keys. Formally, for every non-trivial dependency X → A in 3NF, X must be a super key or A must be a prime attribute.

---

# 36. Interview Answer: 3NF vs BCNF

A strong answer:

> Both 3NF and BCNF use functional dependencies to restrict problematic redundancy. In 3NF, for every non-trivial dependency X → A, either X must be a super key or A must be a prime attribute. BCNF is stricter because it requires X to be a super key for every non-trivial functional dependency. Therefore every BCNF relation is in 3NF, but a 3NF relation may not satisfy BCNF.

---

# 37. Quick Revision

Remember the progression:

```text
Unstructured / poorly designed relation
            ↓
           1NF
            ↓
Remove partial dependencies
            ↓
           2NF
            ↓
Address dependencies violating 3NF
            ↓
           3NF
            ↓
Require every non-trivial determinant
to be a super key
            ↓
          BCNF
```

And remember:

```text
BCNF ⇒ 3NF ⇒ 2NF ⇒ 1NF
```

---

# 38. Normalization Decision Framework

When examining a relation, ask:

```text
What are the candidate keys?
        ↓
What functional dependencies exist?
        ↓
Are values represented appropriately for 1NF?
        ↓
Do non-prime attributes depend on only
part of a candidate key?
        ↓
Check 2NF
        ↓
Do functional dependencies violate
the formal 3NF condition?
        ↓
Check 3NF
        ↓
Is every determinant of a non-trivial
functional dependency a super key?
        ↓
Check BCNF
```

Do not start by mechanically splitting tables.

Start with **keys and dependencies**.

---

# 39. Interview Tip

When an interviewer gives you a normalization problem, write down:

```text
Relation
Candidate Keys
Functional Dependencies
```

first.

Then determine the highest normal form.

For example:

```text
R(A, B, C)

Candidate Key:
(A, B)

Dependencies:
A → C
```

Since:

```text
C
```

depends only on:

```text
A
```

which is a proper subset of the composite candidate key:

```text
(A, B)
```

there is a partial dependency.

Therefore, assuming the relation is in 1NF, it violates 2NF.

This reasoning is far stronger than guessing the normal form from the table structure.

---

# 40. Final Takeaway

Normalization is fundamentally about **dependencies and correct data modeling**, not merely reducing table size or creating more tables.

A strong interview candidate should be able to reason about:

```text
Redundancy
    ↓
Anomalies
    ↓
Functional Dependencies
    ↓
Candidate Keys
    ↓
Normal Forms
    ↓
Decomposition
    ↓
Trade-offs
```

Once these relationships are understood, normalization questions become much easier than memorizing isolated definitions.

---

## Previous

**[← Keys and Constraints](./02-keys-and-constraints.md)**

## Next

**[SQL and Joins →](./04-sql-and-joins.md)**

---

Maintained by **InterviewEraHQ**.
