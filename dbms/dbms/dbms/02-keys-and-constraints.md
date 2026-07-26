# Keys and Constraints in DBMS

Keys and constraints are fundamental to relational database design.

They help databases answer two important questions:

1. **How do we uniquely identify data?**
2. **How do we prevent invalid or inconsistent data from being stored?**

In interviews, simply listing different types of keys is usually not enough. You should understand how they relate to each other, how constraints enforce data integrity, and what trade-offs appear in real database design.

---

## 1. What Is a Key?

A **key** is an attribute, or combination of attributes, used to identify rows or establish relationships between data in a relational database.

Consider a `candidates` table:

| candidate_id | email | name |
|---:|---|---|
| 101 | alex@example.com | Alex |
| 102 | sam@example.com | Sam |
| 103 | priya@example.com | Priya |

Potential identifiers include:

```text
candidate_id
email
```

If both values are guaranteed to be unique, either could potentially identify a candidate.

Different key terms describe different roles these identifiers can play.

---

# 2. Super Key

A **super key** is any set of one or more attributes that can uniquely identify a row in a relation.

Suppose:

```text
candidate_id
```

is unique.

Then:

```text
{candidate_id}
```

is a super key.

But these are also super keys:

```text
{candidate_id, name}

{candidate_id, email}

{candidate_id, name, email}
```

Why?

Because they still contain `candidate_id`, which is already sufficient to uniquely identify the row.

### Important Point

A super key may contain **unnecessary attributes**.

That leads us to candidate keys.

---

# 3. Candidate Key

A **candidate key** is a minimal super key.

It uniquely identifies a row, and no proper subset of that key can still uniquely identify the row.

Suppose both:

```text
candidate_id
```

and:

```text
email
```

are guaranteed to be unique.

Then both can be candidate keys:

```text
{candidate_id}

{email}
```

But:

```text
{candidate_id, email}
```

is not a candidate key if either attribute alone already uniquely identifies the row.

It is a super key, but it is not minimal.

---

## Super Key vs Candidate Key

This distinction is frequently asked in interviews.

### Super Key

A set of attributes that uniquely identifies a row.

### Candidate Key

A **minimal** super key.

Example:

```text
{candidate_id}
```

Candidate key and therefore also a super key.

```text
{candidate_id, name}
```

Super key, but not a candidate key if `candidate_id` alone is unique.

### Key Relationship

```text
Candidate Key ⊆ Super Key
```

Every candidate key is a super key.

Not every super key is a candidate key.

---

# 4. Primary Key

A **primary key** is the candidate key selected as the main identifier for rows in a table.

Example:

```sql
CREATE TABLE candidates (
    candidate_id INTEGER PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    name VARCHAR(100) NOT NULL
);
```

Here:

```text
candidate_id
```

is the primary key.

If `email` is also guaranteed unique and non-null, it may represent another candidate key conceptually, but `candidate_id` has been selected as the primary key.

---

## Properties of a Primary Key

A primary key must uniquely identify each row.

Conceptually, it must be:

- Unique
- Non-null

A table has one primary key constraint, although that primary key may contain multiple columns.

---

# 5. Alternate Key

Candidate keys that are not selected as the primary key are commonly called **alternate keys**.

Suppose:

```text
candidate_id
email
```

are candidate keys.

If:

```text
candidate_id
```

becomes the primary key, then:

```text
email
```

can be considered an alternate key.

Conceptually:

```text
Candidate Keys
├── candidate_id → Primary Key
└── email        → Alternate Key
```

---

# 6. Composite Key

A **composite key** is a key consisting of multiple attributes.

Consider a table storing candidate skills:

| candidate_id | skill_id | proficiency |
|---:|---:|---|
| 1 | 10 | Advanced |
| 1 | 20 | Intermediate |
| 2 | 10 | Beginner |

Neither:

```text
candidate_id
```

nor:

```text
skill_id
```

uniquely identifies a row.

But together:

```text
(candidate_id, skill_id)
```

can identify each candidate-skill relationship.

SQL:

```sql
CREATE TABLE candidate_skills (
    candidate_id INTEGER,
    skill_id INTEGER,
    proficiency VARCHAR(50),

    PRIMARY KEY (candidate_id, skill_id)
);
```

The primary key contains two columns, so it is a **composite primary key**.

---

## Composite Key vs Primary Key

These concepts describe different things.

**Primary key** describes the role of the key.

**Composite key** describes the fact that multiple attributes form the key.

Therefore, a primary key can be:

```text
Single-column primary key
```

or:

```text
Composite primary key
```

---

# 7. Natural Key

A **natural key** uses data that already has business meaning.

Examples might include:

```text
email address
country code + phone number
ISBN
```

provided the chosen value is actually guaranteed to satisfy the required uniqueness and stability rules for the system.

Example:

```text
email = alex@example.com
```

could potentially be used as an identifier if the application's business rules guarantee its uniqueness.

---

# 8. Surrogate Key

A **surrogate key** is an artificial identifier introduced primarily for identifying records rather than representing business meaning.

Examples:

```text
candidate_id = 1024
```

or a generated UUID.

Example:

```sql
CREATE TABLE candidates (
    id BIGINT PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    name VARCHAR(100) NOT NULL
);
```

Here:

```text
id
```

is a surrogate identifier.

---

## Natural Key vs Surrogate Key

Suppose email addresses are unique.

You could theoretically use:

```text
email
```

as the primary key.

Or introduce:

```text
candidate_id
```

as a surrogate key.

### Natural Key

Potential advantages:

- Already meaningful
- May avoid introducing another identifier

Potential disadvantages:

- Business values can change
- May be large
- May involve multiple columns
- Business rules around uniqueness can change

### Surrogate Key

Potential advantages:

- Usually stable
- Independent of business meaning
- Often convenient for relationships
- Can provide a compact identifier depending on type

Potential disadvantages:

- Adds another attribute
- Does not itself enforce uniqueness of business identifiers

For example, using:

```text
candidate_id
```

as the primary key does not automatically prevent duplicate emails.

You may still need:

```sql
UNIQUE (email)
```

---

# 9. Foreign Key

A **foreign key** creates a referential relationship between tables.

Consider:

```text
candidates
interviews
```

Each interview belongs to a candidate.

### Candidates

| candidate_id | name |
|---:|---|
| 1 | Alex |
| 2 | Sam |

### Interviews

| interview_id | candidate_id | score |
|---:|---:|---:|
| 101 | 1 | 82 |
| 102 | 1 | 91 |
| 103 | 2 | 76 |

Here:

```text
interviews.candidate_id
```

references:

```text
candidates.candidate_id
```

SQL:

```sql
CREATE TABLE candidates (
    candidate_id INTEGER PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

CREATE TABLE interviews (
    interview_id INTEGER PRIMARY KEY,
    candidate_id INTEGER NOT NULL,
    score INTEGER,

    FOREIGN KEY (candidate_id)
        REFERENCES candidates(candidate_id)
);
```

This establishes a relationship between the two tables.

---

# 10. Referential Integrity

**Referential integrity** ensures that relationships between referenced data remain valid according to the foreign key rules.

Suppose:

```text
candidate_id = 999
```

does not exist in `candidates`.

Then an interview referencing candidate `999` would violate the foreign key constraint:

```sql
INSERT INTO interviews (
    interview_id,
    candidate_id,
    score
)
VALUES (
    200,
    999,
    90
);
```

With the foreign key enforced, the database should reject the operation.

This prevents orphaned references.

---

# 11. What Happens When Referenced Data Is Deleted?

Suppose:

```text
Candidate 1
├── Interview 101
└── Interview 102
```

What should happen if Candidate 1 is deleted?

Different applications require different behavior.

Foreign keys can define referential actions.

Common examples include:

```text
RESTRICT
NO ACTION
CASCADE
SET NULL
SET DEFAULT
```

Exact behavior can depend on the database system.

---

## ON DELETE CASCADE

Example:

```sql
FOREIGN KEY (candidate_id)
    REFERENCES candidates(candidate_id)
    ON DELETE CASCADE
```

Deleting the candidate can automatically delete referencing interview rows.

This can be useful when child records should not exist without the parent.

But it should be used intentionally because one delete can affect many rows.

---

## ON DELETE SET NULL

Example:

```sql
FOREIGN KEY (candidate_id)
    REFERENCES candidates(candidate_id)
    ON DELETE SET NULL
```

The referencing column becomes `NULL` when the referenced row is deleted.

For this to work, the column must allow `NULL`.

Example:

```sql
candidate_id INTEGER NULL
```

---

## RESTRICT / NO ACTION

These options generally prevent deletion when referencing rows would violate the foreign key.

However, the exact timing and semantics of `RESTRICT` and `NO ACTION` can differ across database systems.

Avoid claiming they are universally identical.

---

# 12. What Is a Constraint?

A **constraint** is a rule enforced by the database on stored data.

Constraints help protect data integrity.

Common SQL constraints include:

```text
PRIMARY KEY
FOREIGN KEY
UNIQUE
NOT NULL
CHECK
```

Different database systems may support additional constraints or database-specific behavior.

---

# 13. PRIMARY KEY Constraint

Example:

```sql
CREATE TABLE candidates (
    candidate_id INTEGER PRIMARY KEY,
    name VARCHAR(100)
);
```

The primary key ensures each row has a unique identifier.

Conceptually:

```text
candidate_id
1
2
3
```

is valid.

But duplicate primary key values such as:

```text
1
1
```

are not allowed.

Primary key columns also cannot contain `NULL`.

---

# 14. UNIQUE Constraint

A `UNIQUE` constraint ensures that values satisfy a uniqueness requirement.

Example:

```sql
CREATE TABLE candidates (
    candidate_id INTEGER PRIMARY KEY,
    email VARCHAR(255) UNIQUE
);
```

This prevents duplicate non-null email values according to the database's uniqueness semantics.

---

## PRIMARY KEY vs UNIQUE

Both can enforce uniqueness, but they serve different roles.

### PRIMARY KEY

- Main identifier for the table
- Must be unique
- Cannot be NULL
- One primary key constraint per table
- Can contain multiple columns

### UNIQUE

- Enforces additional uniqueness rules
- Multiple UNIQUE constraints can exist
- NULL handling can vary by database system and constraint definition

### Important Interview Trap

Do not say:

> UNIQUE always allows exactly one NULL.

That is not universally correct.

NULL behavior for unique constraints differs between database systems.

For example, some systems allow multiple rows containing `NULL` under a unique constraint because `NULL` represents an unknown value.

Always consider the specific DBMS.

---

# 15. NOT NULL Constraint

`NOT NULL` ensures that a column must contain a non-null value.

Example:

```sql
CREATE TABLE candidates (
    candidate_id INTEGER PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);
```

This would reject a row that attempts to store:

```text
name = NULL
```

---

## NULL Does Not Mean Empty String

These are different:

```text
NULL
```

and:

```text
''
```

An empty string is a string value with zero characters.

`NULL` represents the absence of a value or an unknown value, depending on context.

This distinction matters in SQL.

---

# 16. CHECK Constraint

A `CHECK` constraint ensures that stored values satisfy a condition.

Example:

```sql
CREATE TABLE interview_scores (
    interview_id INTEGER PRIMARY KEY,
    score INTEGER CHECK (score >= 0 AND score <= 100)
);
```

This expresses the rule:

```text
0 <= score <= 100
```

Attempting:

```sql
INSERT INTO interview_scores (
    interview_id,
    score
)
VALUES (
    1,
    150
);
```

would violate the constraint.

---

# 17. Multiple Constraints Together

Real tables usually combine several constraints.

Example:

```sql
CREATE TABLE candidates (
    candidate_id BIGINT PRIMARY KEY,

    email VARCHAR(255)
        UNIQUE
        NOT NULL,

    name VARCHAR(100)
        NOT NULL,

    experience_years INTEGER
        CHECK (experience_years >= 0)
);
```

And:

```sql
CREATE TABLE interviews (
    interview_id BIGINT PRIMARY KEY,

    candidate_id BIGINT NOT NULL,

    score INTEGER
        CHECK (score >= 0 AND score <= 100),

    FOREIGN KEY (candidate_id)
        REFERENCES candidates(candidate_id)
        ON DELETE CASCADE
);
```

These rules protect several invariants directly at the database layer.

---

# 18. Why Database Constraints Matter

Suppose an API validates:

```text
score must be between 0 and 100
```

but the database does not.

Another application, migration, script, admin operation, or buggy service might insert:

```text
score = 500
```

Application validation alone does not necessarily protect the database from every write path.

A database constraint:

```sql
CHECK (score >= 0 AND score <= 100)
```

protects that invariant at the database layer.

---

## Should All Business Logic Be Database Constraints?

No.

Some rules are:

- Too complex
- Context-dependent
- External-service dependent
- Workflow-specific
- Better handled by application logic

Strong systems often use multiple layers:

```text
Frontend validation
        ↓
Application validation
        ↓
Database constraints
```

Each layer solves different problems.

---

# 19. Foreign Key vs Primary Key

Another common interview question.

### Primary Key

Identifies rows within its own table.

Example:

```text
candidates.candidate_id
```

### Foreign Key

References a candidate key or otherwise eligible unique key in another table, depending on the database system and schema rules.

Example:

```text
interviews.candidate_id
```

Conceptually:

```text
candidates
----------------
candidate_id PK
      ↑
      │
      │
candidate_id FK
----------------
interviews
```

---

# 20. Can a Foreign Key Contain NULL?

Yes, a foreign key column can often contain `NULL` if the column itself is nullable.

Example:

```sql
candidate_id INTEGER NULL
```

A `NULL` foreign key can represent the absence of a relationship.

Whether that makes sense depends on the domain.

For example:

```text
interview.candidate_id
```

probably should not be nullable if every interview must belong to a candidate.

Then:

```sql
candidate_id INTEGER NOT NULL
```

would better represent the business rule.

---

# 21. Can a Foreign Key Reference a UNIQUE Column?

In many relational database systems, a foreign key can reference a column or set of columns protected by a `PRIMARY KEY` or an eligible `UNIQUE` constraint.

Example:

```sql
CREATE TABLE users (
    id INTEGER PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL
);
```

Another table could potentially reference:

```text
users.email
```

depending on the DBMS and schema definition.

However, using a stable identifier such as:

```text
users.id
```

is often more convenient because business attributes such as email can change.

This is a design decision, not an absolute rule.

---

# 22. Can a Table Have Multiple Primary Keys?

A table has **one primary key constraint**.

But that primary key can contain multiple columns.

Correct:

```sql
PRIMARY KEY (candidate_id, skill_id)
```

Incorrect conceptual statement:

```text
candidate_id is primary key #1
skill_id is primary key #2
```

Those two columns together form one composite primary key.

---

# 23. Can a Table Have Multiple Candidate Keys?

Yes.

Suppose a user table guarantees uniqueness for:

```text
user_id
email
username
```

Each could potentially be a candidate key if each individually satisfies the necessary uniqueness and minimality requirements.

One is selected as the primary key.

The others can be considered alternate keys.

---

# 24. Key Hierarchy

A useful mental model:

```text
Super Keys
│
├── Minimal Super Keys
│      │
│      └── Candidate Keys
│             │
│             ├── Selected Candidate Key
│             │        └── Primary Key
│             │
│             └── Remaining Candidate Keys
│                      └── Alternate Keys
│
└── Non-minimal Super Keys
```

A composite key can appear at different levels depending on its role.

For example:

```text
(candidate_id, skill_id)
```

could be both:

```text
Candidate Key
+
Primary Key
+
Composite Key
```

at the same time.

---

# 25. Example Interview Schema

Consider an interview platform.

```sql
CREATE TABLE candidates (
    id BIGINT PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    name VARCHAR(100) NOT NULL
);
```

```sql
CREATE TABLE interviews (
    id BIGINT PRIMARY KEY,

    candidate_id BIGINT NOT NULL,

    status VARCHAR(30)
        CHECK (
            status IN (
                'scheduled',
                'in_progress',
                'completed',
                'cancelled'
            )
        ),

    score INTEGER
        CHECK (score >= 0 AND score <= 100),

    FOREIGN KEY (candidate_id)
        REFERENCES candidates(id)
        ON DELETE CASCADE
);
```

This schema expresses several rules.

### Candidate

```text
id
```

uniquely identifies a candidate.

```text
email
```

must be unique and non-null.

```text
name
```

must exist.

### Interview

```text
id
```

uniquely identifies an interview.

```text
candidate_id
```

must reference a valid candidate.

```text
status
```

must satisfy the allowed status values when non-null under this definition.

```text
score
```

must satisfy the allowed range when non-null under this definition.

---

# 26. Common Interview Questions

Make sure you can explain these clearly.

### Keys

1. What is a key in DBMS?
2. What is a super key?
3. What is a candidate key?
4. What is a primary key?
5. What is an alternate key?
6. What is a composite key?
7. What is a natural key?
8. What is a surrogate key?
9. What is a foreign key?
10. Can a table have multiple candidate keys?
11. Can a table have multiple primary keys?
12. Can a primary key contain multiple columns?

### Constraints

13. What is a database constraint?
14. What is the difference between PRIMARY KEY and UNIQUE?
15. What does NOT NULL do?
16. What does CHECK do?
17. What is referential integrity?
18. Can a foreign key contain NULL?
19. Can a foreign key reference a UNIQUE column?
20. What happens when a referenced row is deleted?

### Design

21. Natural key vs surrogate key?
22. When would you use a composite primary key?
23. Why enforce constraints in the database if validation already exists in the application?
24. Should every foreign key use ON DELETE CASCADE?
25. Why might email be a poor primary key even when it is unique?

---

# 27. Common Mistakes

## Mistake 1: Every super key is a candidate key

Incorrect.

Candidate keys are **minimal** super keys.

---

## Mistake 2: A table can have multiple primary keys

Incorrect.

A table has one primary key constraint, although that key can contain multiple columns.

---

## Mistake 3: PRIMARY KEY and UNIQUE are identical

They both enforce uniqueness-related rules, but they serve different purposes and have different semantics.

Primary keys are the table's primary identifier and cannot contain NULL.

UNIQUE constraints represent additional uniqueness requirements, and NULL behavior may depend on the database system.

---

## Mistake 4: Foreign key means the column must be unique

Incorrect.

Many child rows can reference the same parent row.

Example:

```text
Candidate 1
├── Interview 101
├── Interview 102
└── Interview 103
```

All three interviews can contain:

```text
candidate_id = 1
```

---

## Mistake 5: Foreign keys can never be NULL

Incorrect.

A foreign key may be nullable unless the schema also specifies:

```sql
NOT NULL
```

Whether it should be nullable depends on the relationship being modeled.

---

## Mistake 6: ON DELETE CASCADE should always be used

Incorrect.

Cascade deletion is useful when child records should disappear with the parent, but it can also cause large or unintended deletions.

Deletion behavior should represent domain requirements.

---

## Mistake 7: Surrogate keys remove the need for UNIQUE constraints

Incorrect.

Suppose:

```text
id
```

is a surrogate primary key.

If emails must also be unique, you still need to enforce that requirement:

```sql
email VARCHAR(255) UNIQUE NOT NULL
```

---

# 28. Interview Answer Framework

Suppose the interviewer asks:

**What is the difference between a super key and a candidate key?**

Weak answer:

> A super key uniquely identifies a row and a candidate key also uniquely identifies a row.

Technically incomplete.

Better answer:

> A super key is any set of attributes capable of uniquely identifying a row. A candidate key is a minimal super key, meaning none of its attributes can be removed while preserving uniqueness. Therefore every candidate key is a super key, but a super key containing unnecessary attributes is not a candidate key.

That answer demonstrates the actual distinction:

```text
Minimality
```

---

# 29. Practical Design Question

Suppose you have:

```text
users
```

with:

```text
id
email
name
```

and email is unique.

The interviewer asks:

**Would you use email or id as the primary key?**

Do not answer:

> Always use id.

A stronger answer discusses trade-offs.

For example:

> I would usually prefer a stable surrogate identifier such as `id` as the primary key and enforce email uniqueness separately. Email has business meaning and may change, while internal relationships benefit from a stable identifier. However, the correct choice depends on the domain, database, access patterns, and guarantees around the natural key.

This shows engineering judgment instead of memorized rules.

---

# 30. Quick Revision

Before moving forward, you should understand:

```text
Super Key
Candidate Key
Primary Key
Alternate Key
Composite Key
Natural Key
Surrogate Key
Foreign Key
```

And:

```text
PRIMARY KEY
FOREIGN KEY
UNIQUE
NOT NULL
CHECK
```

Most importantly, understand these relationships:

```text
Candidate Key = Minimal Super Key
```

```text
Primary Key = Selected Candidate Key
```

```text
Alternate Key = Candidate Key not selected as Primary Key
```

```text
Composite Key = Key containing multiple attributes
```

```text
Foreign Key = Attribute(s) representing a referential relationship
```

---

## Interview Tip

For key-related interview questions, do not memorize eight isolated definitions.

Understand the hierarchy:

```text
Can it uniquely identify the row?
        ↓
      Yes
        ↓
    Super Key
        ↓
Is it minimal?
        ↓
      Yes
        ↓
   Candidate Key
        ↓
Was it selected as the main identifier?
        ↓
      Yes
        ↓
    Primary Key
```

Once this relationship is clear, most key questions become much easier to reason about.

---

## Previous

**[← Database Fundamentals](./01-database-fundamentals.md)**

## Next

**[Normalization →](./03-normalization.md)**

---

Maintained by **InterviewEraHQ**.
