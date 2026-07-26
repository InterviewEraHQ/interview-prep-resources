# ER Model and Schema Design

The **Entity-Relationship (ER) Model** is a conceptual approach for designing databases before translating business requirements into relational tables.

It answers questions such as:

```text
What entities exist?

What attributes describe them?

How are entities related?

What uniquely identifies each entity?

What cardinality exists between relationships?

Which constraints must the database enforce?
```

A strong candidate should understand:

- Entity and entity set
- Attributes
- Keys
- Relationships
- Cardinality
- Participation constraints
- Weak entities
- Recursive relationships
- Ternary relationships
- Generalization and specialization
- ER diagrams
- ER-to-relational mapping
- One-to-one relationships
- One-to-many relationships
- Many-to-many relationships
- Junction tables
- Foreign keys
- Natural vs surrogate keys
- Schema constraints
- Practical schema-design trade-offs

The central idea is:

> First model the business domain correctly, then translate that model into relational structures and enforce its invariants.

---

# 1. What Is an Entity?

An **entity** is a distinguishable object or concept about which the system stores information.

Examples:

```text
Candidate

Interview

Company

Job

Question

Subscription
```

For an interview platform:

```text
Candidate
```

could represent one user preparing for interviews.

An:

```text
Interview
```

could represent one interview session.

---

# 2. Entity Instance vs Entity Type

Consider:

```text
Candidate
```

This describes an entity type.

A particular candidate:

```text
candidate_id = 101
name = Alex
```

is an entity instance.

Conceptually:

```text
Entity Type
    ↓
Candidate

Entity Instances
    ↓
Candidate 101
Candidate 102
Candidate 103
```

---

# 3. Entity Set

An **entity set** is a collection of entities of the same type.

Example:

```text
Candidate Set

101 | Alex
102 | John
103 | Mia
```

Similarly:

```text
Interview Set
```

contains interview entities.

---

# 4. What Is an Attribute?

An **attribute** describes a property of an entity.

Example:

```text
Candidate
```

may have:

```text
candidate_id

name

email

created_at
```

Conceptually:

```text
Candidate
   │
   ├── candidate_id
   ├── name
   ├── email
   └── created_at
```

---

# 5. Types of Attributes

ER modeling commonly discusses:

```text
Simple attributes

Composite attributes

Single-valued attributes

Multi-valued attributes

Derived attributes

Key attributes
```

Understanding these helps translate conceptual models into relational schemas.

---

# 6. Simple Attribute

A simple attribute is treated as one value in the model.

Example:

```text
email
```

or:

```text
score
```

The model does not decompose it into smaller modeled attributes.

---

# 7. Composite Attribute

A composite attribute can be represented using meaningful subparts.

Example:

```text
Address
```

could contain:

```text
street

city

state

postal_code

country
```

Conceptually:

```text
Address
   │
   ├── street
   ├── city
   ├── state
   ├── postal_code
   └── country
```

Whether it should actually be decomposed depends on how the application needs to query and validate it.

---

# 8. Single-Valued Attribute

A single-valued attribute has one value for an entity under the modeled business rules.

Example:

```text
candidate_id
```

A candidate has one candidate identifier.

---

# 9. Multi-Valued Attribute

Suppose a candidate can have multiple skills:

```text
Java

Python

SQL
```

Conceptually:

```text
Candidate
    ↓
Skills
    ↓
Java
Python
SQL
```

In a relational design, queryable multi-valued relationships are usually modeled separately.

Example:

```text
Candidate

Skill

CandidateSkill
```

rather than:

```text
skills = "Java, Python, SQL"
```

---

# 10. Derived Attribute

A derived attribute can be calculated from other stored data.

Example:

```text
total_interviews
```

could be derived from:

```sql
COUNT(interviews)
```

Similarly:

```text
average_score
```

may be calculated from interview results.

Conceptually:

```text
Interview rows
      ↓
Aggregation
      ↓
average_score
```

Derived values may sometimes be materialized for performance, but then synchronization becomes a design concern.

---

# 11. Stored vs Derived Attributes

Suppose:

```text
date_of_birth
```

is stored.

Then:

```text
age
```

can be derived relative to the current date.

Storing both can create inconsistency:

```text
date_of_birth = 2000-01-01

age = 21
```

years later.

Prefer storing stable source facts where practical and deriving volatile values when needed.

---

# 12. Key Attribute

A key attribute helps uniquely identify an entity.

Example:

```text
Candidate

candidate_id
```

If:

```text
candidate_id
```

uniquely identifies each candidate, it can serve as a key.

In relational implementation it may become:

```sql
PRIMARY KEY
```

---

# 13. What Is a Relationship?

A **relationship** describes an association between entities.

Example:

```text
Candidate
    ↓
takes
    ↓
Interview
```

Another:

```text
Company
    ↓
posts
    ↓
Job
```

Relationships are central to ER modeling.

---

# 14. Relationship Degree

The **degree** of a relationship is the number of entity types participating in it.

Common forms:

```text
Unary

Binary

Ternary
```

---

# 15. Binary Relationship

A binary relationship involves two entity types.

Example:

```text
Candidate
    ↓
takes
    ↓
Interview
```

This is the most common relationship type in application schemas.

---

# 16. Unary / Recursive Relationship

A recursive relationship relates an entity type to itself.

Example:

```text
Employee
    ↓
manages
    ↓
Employee
```

One employee may manage another employee.

Relational implementation:

```text
Employee(
    employee_id,
    name,
    manager_id
)
```

where:

```text
manager_id
```

references:

```text
employee_id
```

in the same table.

---

# 17. Recursive Foreign Key

Example:

```sql
CREATE TABLE employees (
    employee_id BIGINT PRIMARY KEY,
    name TEXT NOT NULL,
    manager_id BIGINT,
    FOREIGN KEY (manager_id)
        REFERENCES employees(employee_id)
);
```

Conceptually:

```text
Employee
   │
   └── manager_id
          ↓
       Employee
```

The root manager may have:

```text
manager_id = NULL
```

depending on business rules.

---

# 18. Ternary Relationship

A ternary relationship involves three entity types.

Example:

```text
Interviewer

Candidate

InterviewRound
```

Suppose the business fact is:

> An interviewer evaluates a candidate in a particular interview round.

This relationship may require all three participants to identify the fact correctly.

---

# 19. Do Not Blindly Replace Ternary Relationships

A ternary relationship cannot always be replaced by three independent binary relationships without changing its meaning.

Suppose:

```text
Supplier
Part
Project
```

and the fact is:

> Supplier S supplies Part P to Project J.

The combination:

```text
(S, P, J)
```

is meaningful.

Separate binary relationships may fail to capture which supplier supplied which part to which project.

---

# 20. Cardinality

Cardinality describes how many entities can participate in a relationship.

Common cardinalities:

```text
One-to-One

One-to-Many

Many-to-One

Many-to-Many
```

Often written:

```text
1:1

1:N

N:1

M:N
```

---

# 21. One-to-One Relationship

Example:

```text
User
    ↓
has
    ↓
UserProfile
```

Suppose each user has at most one profile and each profile belongs to exactly one user.

Then:

```text
User 1 ───── 1 UserProfile
```

This is a one-to-one relationship.

---

# 22. Implementing One-to-One

One common implementation:

```text
User(
    user_id PK
)
```

```text
UserProfile(
    profile_id PK,
    user_id FK UNIQUE
)
```

The important constraint is:

```text
UNIQUE(user_id)
```

Without it:

```text
multiple profiles
```

could reference the same user, turning the relationship into one-to-many.

---

# 23. Shared Primary Key One-to-One

Another implementation:

```text
User(
    user_id PK
)
```

```text
UserProfile(
    user_id PK FK,
    bio,
    avatar_url
)
```

Here:

```text
UserProfile.user_id
```

is both:

```text
PRIMARY KEY
```

and:

```text
FOREIGN KEY
```

This naturally allows at most one profile per user.

---

# 24. One-to-Many Relationship

Example:

```text
Candidate
    ↓
has
    ↓
Interviews
```

One candidate can have many interviews.

Each interview belongs to one candidate.

Conceptually:

```text
Candidate 1
   │
   ├── Interview A
   ├── Interview B
   └── Interview C
```

This is:

```text
1:N
```

---

# 25. Implementing One-to-Many

Place the foreign key on the many side.

Example:

```text
Candidate(
    candidate_id PK
)
```

```text
Interview(
    interview_id PK,
    candidate_id FK
)
```

Conceptually:

```text
Candidate
    1
    │
    │
    N
Interview
```

---

# 26. Why Is the Foreign Key on the Many Side?

Suppose one candidate has 100 interviews.

Bad design:

```text
Candidate(
    candidate_id,
    interview_1,
    interview_2,
    interview_3,
    ...
)
```

This creates repeating groups.

Instead:

```text
Interview
```

stores:

```text
candidate_id
```

for each interview.

This supports any number of interviews without changing the Candidate schema.

---

# 27. Many-to-Many Relationship

Suppose:

```text
Candidate
```

can have many:

```text
Skills
```

and each:

```text
Skill
```

can belong to many candidates.

Conceptually:

```text
Candidate M ───── N Skill
```

This is a many-to-many relationship.

---

# 28. Implementing Many-to-Many

Relational databases normally represent it using a junction table.

```text
Candidate(
    candidate_id PK
)
```

```text
Skill(
    skill_id PK
)
```

```text
CandidateSkill(
    candidate_id FK,
    skill_id FK
)
```

Possible primary key:

```text
(candidate_id, skill_id)
```

---

# 29. Junction Table

A junction table represents the relationship itself.

Example:

```text
CandidateSkill
```

Rows:

```text
101 | 1
101 | 2
102 | 1
```

Meaning:

```text
Candidate 101 has Skill 1

Candidate 101 has Skill 2

Candidate 102 has Skill 1
```

---

# 30. Relationship Attributes

Sometimes the relationship itself has attributes.

Suppose:

```text
Candidate
```

and:

```text
Job
```

have an application relationship.

The relationship may contain:

```text
applied_at

status

source
```

Then:

```text
Application(
    candidate_id,
    job_id,
    applied_at,
    status,
    source
)
```

represents more than a simple junction.

It becomes an important domain entity in its own right.

---

# 31. Association Entity

A many-to-many relationship with meaningful attributes is often modeled as an **associative entity**.

Example:

```text
Student
Course
```

Relationship:

```text
Enrollment
```

with:

```text
enrolled_at

grade

status
```

Then:

```text
Enrollment
```

is not merely plumbing.

It represents a real business concept.

---

# 32. Cardinality vs Participation

These concepts are related but different.

### Cardinality

Answers:

```text
How many?
```

Example:

```text
One candidate → many interviews
```

### Participation

Answers:

```text
Must the entity participate?
```

Example:

```text
Must every interview belong to a candidate?
```

These should not be confused.

---

# 33. Total Participation

An entity has total participation when every entity instance must participate in the relationship.

Suppose:

> Every interview must belong to a candidate.

Then Interview has total participation in the candidate-interview relationship.

Relationally, this may be represented with:

```text
candidate_id NOT NULL
```

plus a foreign key.

---

# 34. Partial Participation

Suppose:

> A candidate may exist before completing any interview.

Then Candidate participation in:

```text
Candidate → Interview
```

is optional.

A candidate can exist with:

```text
0 interviews
```

Therefore Candidate has partial participation in that relationship.

---

# 35. Minimum and Maximum Cardinality

A useful notation expresses:

```text
(min, max)
```

Example:

```text
Candidate:

0..N Interviews
```

means:

```text
minimum = 0

maximum = many
```

For Interview:

```text
1..1 Candidate
```

means:

```text
exactly one candidate
```

---

# 36. Optional vs Mandatory Relationships

Example:

```text
Interview
```

may optionally have:

```text
reviewer_id
```

If review is optional:

```text
reviewer_id NULL
```

may be valid.

If every interview must have a reviewer:

```text
reviewer_id NOT NULL
```

may be appropriate.

Nullability should reflect business semantics.

---

# 37. Strong Entity

A **strong entity** can be uniquely identified using its own attributes.

Example:

```text
Candidate(
    candidate_id
)
```

where:

```text
candidate_id
```

independently identifies a candidate.

---

# 38. Weak Entity

A **weak entity** cannot be uniquely identified solely by its own partial attributes and depends on an owner entity for identification.

Classic example:

```text
Employee

Dependent
```

Suppose a dependent has:

```text
dependent_name
```

but that name is only unique within one employee's dependents.

Then identification may require:

```text
(employee_id, dependent_name)
```

---

# 39. Weak Entity Example

```text
Employee(
    employee_id PK
)
```

```text
Dependent(
    employee_id,
    dependent_name,
    date_of_birth
)
```

Possible key:

```text
(employee_id, dependent_name)
```

The dependent's identity includes the owner's key.

---

# 40. Weak Entity vs Ordinary Child Table

Not every table containing a foreign key is a weak entity.

Example:

```text
Interview(
    interview_id PK,
    candidate_id FK
)
```

Interview has its own identifier:

```text
interview_id
```

Therefore it is not weak merely because it references Candidate.

Weakness concerns identification dependency, not just existence of a foreign key.

---

# 41. Existence Dependency

Some entities may depend on another entity for meaningful existence.

Example:

```text
OrderItem
```

may not make sense without:

```text
Order
```

This is an existence dependency.

But conceptual existence dependency and weak-entity identification are not always identical.

Be precise when discussing them.

---

# 42. Identifying Relationship

The relationship connecting a weak entity to its owner is called an:

```text
Identifying relationship
```

Example:

```text
Employee
    ↓
has
    ↓
Dependent
```

The owner's key contributes to identifying the weak entity.

---

# 43. Generalization

**Generalization** combines multiple lower-level entity types into a more general entity.

Example:

```text
Candidate

Recruiter

Admin
```

may share:

```text
user_id

name

email
```

These common attributes can be generalized into:

```text
User
```

Conceptually:

```text
Candidate ──┐
Recruiter ──┼──► User
Admin ──────┘
```

---

# 44. Specialization

**Specialization** starts with a general entity and creates more specific subtypes.

Example:

```text
User
   │
   ├── Candidate
   ├── Recruiter
   └── Admin
```

Subtypes may have additional attributes.

Example:

```text
Candidate
    resume_url

Recruiter
    company_id
```

---

# 45. Generalization vs Specialization

### Generalization

Bottom-up:

```text
Candidate
Recruiter
Admin
      ↓
User
```

### Specialization

Top-down:

```text
User
 ↓
Candidate
Recruiter
Admin
```

They describe related abstraction processes viewed from opposite directions.

---

# 46. ISA Relationship

Subtype relationships are often described as:

```text
ISA
```

Example:

```text
Candidate ISA User

Recruiter ISA User
```

Meaning:

```text
Every Candidate is a User
```

but:

```text
Not every User is necessarily a Candidate.
```

---

# 47. Disjoint Specialization

Suppose a user can be exactly one of:

```text
Candidate

Recruiter
```

Then the specialization may be:

```text
Disjoint
```

An entity cannot simultaneously belong to both subtypes under that rule.

---

# 48. Overlapping Specialization

Suppose a user may be both:

```text
Candidate
```

and:

```text
Interviewer
```

Then specialization is:

```text
Overlapping
```

Subtype membership is not mutually exclusive.

---

# 49. Total Specialization

Suppose every User must belong to at least one subtype:

```text
Candidate

Recruiter

Admin
```

Then specialization is total.

Conceptually:

```text
Every User
    ↓
must belong to subtype
```

---

# 50. Partial Specialization

Suppose a User may exist without belonging to any specialized subtype.

Then specialization is partial.

Example:

```text
User
```

may represent a generic account before onboarding determines the user's role.

---

# 51. Mapping Inheritance to Tables

There are multiple strategies.

One approach:

```text
User(
    user_id,
    name,
    email
)
```

```text
Candidate(
    user_id FK PK,
    resume_url
)
```

```text
Recruiter(
    user_id FK PK,
    company_id
)
```

This stores common attributes in User and subtype-specific attributes separately.

---

# 52. Single-Table Inheritance

Another strategy:

```text
User(
    user_id,
    type,
    name,
    email,
    resume_url,
    company_id
)
```

Example:

```text
type = candidate
```

uses:

```text
resume_url
```

while:

```text
company_id
```

may be null.

Advantages:

```text
Simple reads

Fewer joins
```

Potential disadvantages:

```text
Many nullable columns

Subtype constraints harder to express

Table becomes broad
```

---

# 53. Concrete Table Strategy

Another strategy:

```text
Candidate(
    user_id,
    name,
    email,
    resume_url
)
```

```text
Recruiter(
    user_id,
    name,
    email,
    company_id
)
```

Common fields are duplicated.

This can simplify subtype-specific access but creates duplication and may complicate cross-user operations.

There is no universal inheritance mapping strategy.

---

# 54. ER Diagram

An ER diagram visually represents:

```text
Entities

Attributes

Relationships

Cardinalities

Participation
```

A simplified conceptual model:

```text
Candidate
    │
    │ 1
    │
    │ N
Interview
    │
    │ N
    │
    │ 1
Role
```

Meaning:

```text
One candidate can have many interviews.

Each interview belongs to one role.
```

---

# 55. Crow's Foot Notation

A common ER notation uses crow's-foot symbols to represent cardinality.

Conceptually:

```text
Candidate |────< Interview
```

where the crow's foot represents:

```text
many
```

Different modeling tools may use slightly different symbols.

Focus on the cardinality meaning rather than memorizing one drawing tool's appearance.

---

# 56. ER Model to Relational Schema

A typical mapping process:

```text
Entity
   ↓
Table

Attribute
   ↓
Column

Key Attribute
   ↓
Primary / Candidate Key

1:N Relationship
   ↓
Foreign Key on N side

M:N Relationship
   ↓
Junction Table

Weak Entity
   ↓
Table including owner key

Multi-Valued Attribute
   ↓
Separate Relation
```

---

# 57. Mapping a Strong Entity

ER entity:

```text
Candidate

candidate_id
name
email
```

Relational schema:

```sql
CREATE TABLE candidates (
    candidate_id BIGINT PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT NOT NULL UNIQUE
);
```

The conceptual entity becomes a table.

---

# 58. Mapping One-to-Many

Conceptual:

```text
Candidate 1 ─── N Interview
```

Relational:

```sql
CREATE TABLE interviews (
    interview_id BIGINT PRIMARY KEY,
    candidate_id BIGINT NOT NULL,
    FOREIGN KEY (candidate_id)
        REFERENCES candidates(candidate_id)
);
```

The foreign key is placed on:

```text
Interview
```

because it is the many side.

---

# 59. Mapping Many-to-Many

Conceptual:

```text
Candidate M ─── N Skill
```

Relational:

```sql
CREATE TABLE candidate_skills (
    candidate_id BIGINT NOT NULL,
    skill_id BIGINT NOT NULL,

    PRIMARY KEY (candidate_id, skill_id),

    FOREIGN KEY (candidate_id)
        REFERENCES candidates(candidate_id),

    FOREIGN KEY (skill_id)
        REFERENCES skills(skill_id)
);
```

The junction table converts:

```text
M:N
```

into two:

```text
1:N
```

relationships.

---

# 60. Mapping One-to-One

Conceptual:

```text
User 1 ─── 1 Profile
```

Possible relational implementation:

```sql
CREATE TABLE profiles (
    user_id BIGINT PRIMARY KEY,
    bio TEXT,

    FOREIGN KEY (user_id)
        REFERENCES users(user_id)
);
```

The primary key prevents multiple profile rows for the same user.

---

# 61. Primary Key Design

A primary key should provide stable tuple identity.

Common choices:

```text
Auto-increment integer

BIGINT

UUID

Natural identifier
```

The correct choice depends on:

```text
Domain

Distribution requirements

Data lifecycle

Security concerns

Storage/index characteristics
```

Do not choose keys only because one style is fashionable.

---

# 62. Natural Key

A natural key comes from the business domain.

Example:

```text
country_code
```

may naturally identify a country under the modeled standard.

Potential benefits:

```text
Meaningful

Existing domain uniqueness
```

Potential risks:

```text
Business rules change

Value changes

Large keys

External semantics
```

---

# 63. Surrogate Key

A surrogate key is created primarily for internal identification.

Example:

```text
candidate_id = 845123
```

or:

```text
UUID
```

Benefits can include:

```text
Stable internal identity

Simple references

Independence from mutable business attributes
```

But surrogate keys do not replace business uniqueness constraints.

---

# 64. Surrogate Key Trap

Suppose:

```text
users(
    id PRIMARY KEY,
    email
)
```

Business rule:

```text
Each email must be unique.
```

Adding:

```text
id
```

does not enforce that rule.

You still need:

```sql
UNIQUE(email)
```

when the business rule requires uniqueness.

---

# 65. Composite Key

Sometimes identity naturally consists of multiple values.

Example:

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

This prevents duplicate relationships such as:

```text
Candidate 101 → Python

Candidate 101 → Python
```

being inserted twice.

---

# 66. Composite Key vs Surrogate Key

Alternative:

```text
CandidateSkill(
    id,
    candidate_id,
    skill_id
)
```

If `id` is the primary key, the schema may still need:

```text
UNIQUE(candidate_id, skill_id)
```

to preserve the actual business invariant.

A surrogate key should not erase domain constraints.

---

# 67. Foreign Key

A foreign key enforces referential integrity.

Example:

```text
Interview.candidate_id
```

references:

```text
Candidate.candidate_id
```

This prevents an interview from referencing a candidate that does not exist, subject to the defined constraint behavior.

---

# 68. Referential Integrity

Suppose:

```text
interviews.candidate_id = 999
```

but no candidate:

```text
999
```

exists.

Without a foreign key, the database may accept the orphaned reference.

With a foreign key, the database can reject invalid references.

This protects structural consistency.

---

# 69. Foreign Key Actions

When referenced rows change or are deleted, common actions include:

```text
RESTRICT

NO ACTION

CASCADE

SET NULL

SET DEFAULT
```

Exact semantics can vary by DBMS.

Choose actions according to domain behavior, not convenience.

---

# 70. ON DELETE CASCADE

Example:

```text
Order
    ↓
OrderItems
```

If order items have no meaning without their order:

```text
ON DELETE CASCADE
```

may be appropriate.

Deleting the order automatically deletes its items.

But cascade behavior can be dangerous when deletion has large or unexpected reach.

---

# 71. CASCADE Is Not Always Correct

Suppose:

```text
Candidate
```

has:

```text
InterviewHistory
```

and interview history must be retained for compliance or analytics.

Then deleting Candidate should perhaps:

```text
anonymize candidate data
```

rather than delete all interview history.

Database actions should reflect lifecycle requirements.

---

# 72. ON DELETE SET NULL

Suppose an interview optionally references:

```text
reviewer_id
```

If the reviewer account is deleted but the interview should remain:

```text
ON DELETE SET NULL
```

may be appropriate.

This requires the foreign-key column to permit:

```text
NULL
```

---

# 73. NOT NULL Constraint

Use:

```text
NOT NULL
```

when a value is mandatory.

Example:

```text
Interview.candidate_id
```

if every interview must belong to a candidate.

Do not rely only on application validation.

Database constraints protect integrity across all writers.

---

# 74. UNIQUE Constraint

Suppose:

```text
email
```

must be unique.

Use:

```sql
email TEXT NOT NULL UNIQUE
```

rather than:

```text
SELECT email first

If absent:
    INSERT
```

The application-side check alone has a race condition.

The database should enforce the invariant.

---

# 75. CHECK Constraint

Suppose interview scores must be:

```text
0 to 100
```

Use:

```sql
CHECK (score >= 0 AND score <= 100)
```

This prevents invalid values regardless of which application path writes the data.

---

# 76. Constraints Are Part of Schema Design

Good schema design does not stop at:

```text
Tables

Columns

Foreign keys
```

It also considers:

```text
PRIMARY KEY

UNIQUE

NOT NULL

CHECK

FOREIGN KEY

Defaults

Indexes
```

The schema should encode important invariants where practical.

---

# 77. Schema Design From Requirements

Suppose requirements are:

```text
Candidates can take many interviews.

Each interview targets one role.

An interview contains many questions.

Questions may appear in many interviews.

Each interview-question pair stores the candidate's answer and score.
```

Start by identifying:

```text
Candidate

Interview

Role

Question
```

Then identify relationships.

---

# 78. Candidate to Interview

Requirement:

```text
Candidates can take many interviews.
```

Relationship:

```text
Candidate 1 ─── N Interview
```

Implementation:

```text
Interview.candidate_id
```

---

# 79. Role to Interview

Requirement:

```text
Each interview targets one role.
```

Assuming a role can be used by many interviews:

```text
Role 1 ─── N Interview
```

Implementation:

```text
Interview.role_id
```

---

# 80. Interview to Question

Requirement:

```text
An interview contains many questions.

A question may appear in many interviews.
```

Relationship:

```text
Interview M ─── N Question
```

Therefore create an associative entity.

Example:

```text
InterviewQuestion
```

---

# 81. InterviewQuestion

Possible schema:

```text
InterviewQuestion(
    interview_id,
    question_id,
    sequence_number,
    answer,
    score
)
```

This relationship contains attributes specific to:

```text
Question inside a particular interview
```

such as:

```text
answer

score

position
```

---

# 82. Better Interview Schema

Conceptually:

```text
Candidate
    │
    │ 1:N
    ▼
Interview
    │
    ├──────── N:1 ───────► Role
    │
    │
    │ 1:N
    ▼
InterviewQuestion
    ▲
    │ N:1
    │
Question
```

This separates:

```text
Question definition
```

from:

```text
Question usage in an interview
```

---

# 83. Example Relational Schema

```sql
CREATE TABLE candidates (
    candidate_id BIGINT PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT NOT NULL UNIQUE
);

CREATE TABLE roles (
    role_id BIGINT PRIMARY KEY,
    name TEXT NOT NULL UNIQUE
);

CREATE TABLE questions (
    question_id BIGINT PRIMARY KEY,
    question_text TEXT NOT NULL
);

CREATE TABLE interviews (
    interview_id BIGINT PRIMARY KEY,
    candidate_id BIGINT NOT NULL,
    role_id BIGINT NOT NULL,
    created_at TIMESTAMP NOT NULL,

    FOREIGN KEY (candidate_id)
        REFERENCES candidates(candidate_id),

    FOREIGN KEY (role_id)
        REFERENCES roles(role_id)
);

CREATE TABLE interview_questions (
    interview_id BIGINT NOT NULL,
    question_id BIGINT NOT NULL,
    sequence_number INTEGER NOT NULL,
    answer TEXT,
    score INTEGER,

    PRIMARY KEY (interview_id, question_id),

    FOREIGN KEY (interview_id)
        REFERENCES interviews(interview_id),

    FOREIGN KEY (question_id)
        REFERENCES questions(question_id),

    CHECK (score IS NULL OR (score >= 0 AND score <= 100))
);
```

This is only one possible design.

Actual requirements may require different identifiers, ordering constraints, versioning, or history tables.

---

# 84. Hidden Requirement: Repeated Questions

Suppose the same question can appear twice in one interview.

Then:

```text
PRIMARY KEY(interview_id, question_id)
```

would prevent that.

A better model might use:

```text
interview_question_id
```

or:

```text
(interview_id, sequence_number)
```

as identity.

This demonstrates an important principle:

> Schema design depends on business semantics, not generic templates.

---

# 85. Hidden Requirement: Question Versioning

Suppose a question is edited after an interview.

Should historical interviews show:

```text
the new question text
```

or:

```text
the exact question asked at interview time?
```

If historical accuracy matters, simply referencing the mutable Question row may be insufficient.

Possible designs include:

```text
Question versions

Immutable questions

Snapshot text in InterviewQuestion
```

This is a real schema-design issue.

---

# 86. Snapshot Data

Suppose:

```text
Role.name
```

changes from:

```text
Software Engineer
```

to:

```text
Software Engineer II
```

Should an interview from last year display the original role title?

If yes, the schema may need:

```text
versioning
```

or:

```text
snapshot attributes
```

Deliberate historical duplication is not automatically bad normalization.

It can represent a different fact:

```text
Role name at interview time
```

---

# 87. Current State vs Historical Fact

These are different facts:

```text
Candidate's current email

Candidate's email when application was submitted
```

The first belongs to current candidate state.

The second may belong to an application snapshot.

Do not normalize away historical facts merely because the values look duplicated.

---

# 88. Status Modeling

Suppose Interview can have:

```text
created

in_progress

completed

cancelled
```

A simple schema might use:

```text
status
```

with a CHECK constraint.

But if status history matters:

```text
InterviewStatusHistory(
    interview_id,
    status,
    changed_at
)
```

may also be needed.

Current state and event history are different modeling concerns.

---

# 89. Boolean Explosion

Bad schema:

```text
is_created

is_started

is_completed

is_cancelled

is_failed
```

This allows contradictory states:

```text
is_completed = true

is_cancelled = true

is_failed = true
```

If states are mutually exclusive, a single status field may model the domain more accurately.

---

# 90. Nullable Column Explosion

Suppose:

```text
Payment(
    card_number,
    upi_id,
    bank_account,
    wallet_id,
    crypto_address,
    ...
)
```

with most columns null depending on payment type.

This may indicate multiple concepts are being forced into one table.

Subtype modeling or separate payment-method structures may provide a clearer schema.

---

# 91. JSON vs Relational Columns

Modern relational databases often support JSON.

JSON is useful for:

```text
Flexible metadata

External payload snapshots

Rarely queried configuration

Evolving attributes
```

But do not use JSON merely to avoid schema design.

If the application frequently needs to:

```text
filter

join

validate

index

enforce relationships
```

individual relational columns or tables may be more appropriate.

---

# 92. JSON Example

Potentially reasonable:

```text
interview_metadata JSON
```

containing:

```text
model version

experimental flags

provider metadata
```

Potentially poor:

```text
candidate JSON
```

containing the entire candidate domain while the application frequently queries:

```text
candidate.email

candidate.subscription

candidate.company
```

Use JSON intentionally.

---

# 93. Indexes Are Part of Physical Schema Design

Logical design may say:

```text
Interview.candidate_id FK
```

But common query:

```sql
SELECT *
FROM interviews
WHERE candidate_id = ?;
```

may benefit from:

```text
INDEX(candidate_id)
```

Foreign-key constraints and indexes are related concepts but are not universally the same thing.

Check your DBMS behavior.

---

# 94. Do Not Index Every Column

Indexes improve some reads but add:

```text
Storage

Write cost

Maintenance

Planner complexity
```

Index based on:

```text
Query patterns

Join columns

Filtering

Ordering

Selectivity

Measured workload
```

not because a column exists.

---

# 95. Schema Naming

Prefer consistent names.

For example:

```text
candidate_id

interview_id

created_at

updated_at
```

Avoid mixing:

```text
candidateId

candidate_id

CandidateID

candidateid
```

inside the same schema without a deliberate convention.

Consistency reduces cognitive overhead.

---

# 96. Avoid Ambiguous Names

Bad:

```text
data

value

type

info

status2
```

Better:

```text
interview_status

question_text

candidate_email

subscription_plan
```

Names should communicate domain meaning.

---

# 97. Avoid Encoding Meaning Into IDs

Example:

```text
USR-IND-PRO-2026-0001
```

may encode:

```text
country

plan

year

sequence
```

If those properties change, the identifier becomes misleading.

Identifiers should usually remain stable.

Store mutable domain properties separately.

---

# 98. Schema Evolution

Production schemas change.

Examples:

```text
Add column

Add table

Change constraint

Backfill data

Create index

Rename field

Migrate relationship
```

Schema design should consider how changes can be deployed safely.

---

# 99. Adding NOT NULL Safely

Suppose a table has millions of rows and you introduce:

```text
timezone NOT NULL
```

Existing rows have no value.

A migration may need:

```text
1. Add nullable column

2. Backfill values

3. Update application writers

4. Verify data

5. Add NOT NULL constraint
```

Exact deployment strategy depends on the DBMS and migration system.

---

# 100. Schema Design and Concurrency

Suppose credits are modeled as:

```text
candidate.credit_balance
```

Many concurrent operations update the same row.

This may create:

```text
Hot-row contention
```

Schema design influences concurrency behavior.

Alternatives might include:

```text
Credit ledger

Reservation model

Append-only events
```

depending on requirements.

Data modeling affects more than storage.

---

# 101. Ledger Model

Instead of only storing:

```text
credit_balance = 7
```

a system might store:

```text
CreditTransaction

+10 purchase

-1 interview

-1 interview

-1 interview
```

Balance can be derived:

```text
SUM(amount)
```

or maintained separately as a cached aggregate.

A ledger provides:

```text
Auditability

History

Reconciliation
```

but may require more sophisticated querying and concurrency design.

---

# 102. Soft Delete

Instead of physically deleting:

```text
Candidate
```

a system may use:

```text
deleted_at
```

or:

```text
status = deleted
```

This is called soft deletion.

Potential benefits:

```text
Recovery

Auditability

Historical references
```

Potential costs:

```text
More filtering

Unique constraint complexity

Storage growth

Accidental exposure of deleted rows
```

Use it only when lifecycle requirements justify it.

---

# 103. Audit Tables

For sensitive or important state transitions, the system may need:

```text
who changed it

what changed

when it changed
```

Possible design:

```text
InterviewAudit(
    audit_id,
    interview_id,
    action,
    actor_id,
    created_at,
    metadata
)
```

Audit requirements should be considered during schema design rather than added blindly later.

---

# 104. Timestamps

Common timestamps:

```text
created_at

updated_at

deleted_at
```

But semantics must be defined.

For example:

```text
updated_at
```

could mean:

```text
last business-data update

last database write

last synchronization

last user edit
```

Ambiguous timestamps create unreliable analytics.

---

# 105. Time Zones

For globally used systems, storing timestamps with clear timezone semantics is important.

A common approach is:

```text
Store an absolute instant

Display using user's timezone
```

Avoid ambiguous local timestamps when the actual instant matters.

Exact timestamp types vary by database.

---

# 106. Schema Design Mistake: Duplicate Source of Truth

Suppose:

```text
Candidate.interview_count
```

and actual interview rows both represent the same current count.

Now two sources must remain synchronized.

This may be justified as a cached aggregate, but ownership must be explicit.

Ask:

```text
Which value is authoritative?

How is derived data updated?

How is inconsistency repaired?
```

---

# 107. Schema Design Mistake: Missing Constraints

Suppose application code assumes:

```text
one active subscription per user
```

but database allows:

```text
5 active subscriptions
```

because no constraint protects the invariant.

Application assumptions are not database guarantees.

Important business invariants should be enforced at the strongest practical layer.

---

# 108. Schema Design Mistake: Wrong Cardinality

Suppose schema assumes:

```text
User 1:1 Company
```

by storing:

```text
users.company_id
```

But future requirement says:

```text
One user may belong to multiple companies.
```

Now the schema must change significantly.

During design, clarify cardinality rather than guessing it.

---

# 109. Schema Design Mistake: Premature Generalization

Suppose you create:

```text
Entity(
    entity_id,
    entity_type,
    data JSON
)
```

for:

```text
Candidate

Interview

Question

Payment

Company
```

because:

> Everything is an entity.

This destroys useful relational structure and constraints.

Generalize only where the domain actually shares meaningful behavior or structure.

---

# 110. Schema Design Mistake: One Giant Table

Example:

```text
CandidateInterviewQuestionResultRoleCompanySubscription
```

containing everything.

Problems:

```text
Redundancy

Nulls

Anomalies

Complex updates

Poor ownership of facts
```

Separate concepts according to their semantics and dependencies.

---

# 111. Schema Design Mistake: Table Per Tiny Attribute

The opposite extreme is also bad.

Example:

```text
CandidateName

CandidateEmail

CandidateCreatedAt
```

with separate tables for simple attributes without a domain reason.

Normalization does not mean every attribute deserves a table.

---

# 112. Common Interview Question: What Is an ER Model?

Strong answer:

> The ER model is a conceptual data-modeling approach that represents entities, their attributes, relationships, cardinalities, and constraints before those concepts are translated into a relational schema.

---

# 113. Common Interview Question: Entity vs Attribute?

### Entity

Represents a distinguishable domain object or concept.

Example:

```text
Candidate
```

### Attribute

Describes a property of that entity.

Example:

```text
candidate_email
```

---

# 114. Common Interview Question: Entity vs Table?

They are related but not identical.

An:

```text
Entity
```

belongs to the conceptual model.

A:

```text
Table
```

belongs to the relational implementation.

Often an entity maps to a table, but mapping can be more complex for:

```text
Inheritance

Weak entities

Multi-valued attributes

Relationships
```

---

# 115. Common Interview Question: What Is Cardinality?

Strong answer:

> Cardinality describes the maximum number of instances of one entity that may be associated with instances of another entity through a relationship, such as 1:1, 1:N, or M:N. Modeling often also includes minimum participation such as 0..1 or 1..N.

---

# 116. Common Interview Question: 1:N Implementation?

Answer:

> Put a foreign key referencing the one-side entity on the many-side relation.

Example:

```text
Candidate 1:N Interview
```

becomes:

```text
Interview.candidate_id
```

---

# 117. Common Interview Question: M:N Implementation?

Answer:

> Introduce a junction or associative relation containing foreign keys to both participating relations. Its key usually includes the referenced identifiers or an equivalent uniqueness constraint.

Example:

```text
CandidateSkill(
    candidate_id,
    skill_id
)
```

---

# 118. Common Interview Question: What Is a Weak Entity?

Strong answer:

> A weak entity lacks a complete identifying key of its own and depends on an owner entity's key, combined with a partial key or discriminator, for identification.

Do not answer:

> A weak entity is any table with a foreign key.

That is incorrect.

---

# 119. Common Interview Question: Natural vs Surrogate Key?

### Natural Key

Identity comes from domain data.

### Surrogate Key

Artificial internal identifier.

Strong answer:

> I choose based on stability, domain semantics, reference patterns, and operational requirements. Even when using a surrogate primary key, I still enforce natural business uniqueness separately where required.

---

# 120. Common Interview Question: Primary Key vs Unique Constraint?

Both can enforce uniqueness, but they serve different schema roles.

A primary key:

```text
identifies the row as its primary relational identifier
```

while UNIQUE constraints enforce additional candidate/business uniqueness.

A table can have:

```text
one primary key
```

and:

```text
multiple UNIQUE constraints
```

---

# 121. Common Interview Question: Foreign Key vs Join?

A foreign key is:

```text
an integrity constraint
```

A join is:

```text
a query operation combining related data
```

You can technically join columns without a foreign-key constraint.

But the foreign key protects referential integrity.

---

# 122. Common Interview Question: Does a Foreign Key Automatically Create an Index?

Correct answer:

> It depends on the DBMS. Some systems may automatically index certain related columns while others do not. Foreign-key constraints and indexes solve different problems, so index requirements should be verified for the specific database and workload.

Avoid universal claims.

---

# 123. Common Interview Question: Why Use a Junction Table?

Because relational tables do not represent an M:N relationship using a single foreign key.

The junction table converts:

```text
M:N
```

into:

```text
1:N

+

1:N
```

and can also store attributes belonging to the relationship.

---

# 124. Common Interview Question: What Is Participation?

Answer:

> Participation describes whether an entity instance must participate in a relationship. Total participation means participation is mandatory; partial participation means an entity may exist without participating.

---

# 125. Common Interview Question: Generalization vs Specialization?

Answer:

> Generalization is a bottom-up abstraction that combines common properties of multiple entity types into a supertype. Specialization is a top-down process that divides a general entity into more specific subtypes.

---

# 126. Common Interview Question: Should Everything Be Normalized?

Answer:

> The schema should first represent business facts and dependencies correctly. Normalization is useful for reducing unnecessary redundancy and anomalies, but production design also considers query patterns, historical requirements, analytics, performance, and operational complexity. Deliberate denormalization can be valid when its consistency model is explicit.

---

# 127. Common Interview Questions

Be prepared to answer:

1. What is an ER model?
2. What is an entity?
3. Entity type vs entity instance?
4. What is an entity set?
5. What is an attribute?
6. What are attribute types?
7. What is a composite attribute?
8. What is a multi-valued attribute?
9. What is a derived attribute?
10. What is a relationship?
11. What is relationship degree?
12. What is a recursive relationship?
13. What is a ternary relationship?
14. What is cardinality?
15. Explain 1:1.
16. Explain 1:N.
17. Explain M:N.
18. How do you implement 1:1?
19. How do you implement 1:N?
20. How do you implement M:N?
21. What is a junction table?
22. What is an associative entity?
23. What is participation?
24. Total vs partial participation?
25. What is a weak entity?
26. What is an identifying relationship?
27. Weak entity vs child table?
28. What is generalization?
29. What is specialization?
30. Disjoint vs overlapping specialization?
31. Total vs partial specialization?
32. How do you map inheritance to relational tables?
33. What is a natural key?
34. What is a surrogate key?
35. What is a composite key?
36. Why doesn't a surrogate key replace UNIQUE constraints?
37. What is referential integrity?
38. What does ON DELETE CASCADE do?
39. When might SET NULL be appropriate?
40. How do you design a schema from requirements?
41. When should JSON be used?
42. What are schema-design constraints?
43. How do historical requirements affect schema design?
44. What is soft deletion?
45. How would you model status history?

---

# 128. Common Mistakes

## Mistake 1: Every entity must map to exactly one table

Incorrect.

Mapping can depend on relationships, inheritance, weak entities, and implementation strategy.

---

## Mistake 2: Every foreign key means 1:1

Incorrect.

A normal foreign key commonly represents many-to-one.

A UNIQUE constraint may be needed to enforce one-to-one.

---

## Mistake 3: Many-to-many can be represented with two foreign-key columns in either parent table

Incorrect.

Use an associative/junction relation.

---

## Mistake 4: Every child table is a weak entity

Incorrect.

Weakness concerns identification dependency.

---

## Mistake 5: A surrogate ID enforces business uniqueness

Incorrect.

You may still need:

```text
UNIQUE(email)

UNIQUE(candidate_id, skill_id)
```

or other constraints.

---

## Mistake 6: CASCADE should be used everywhere

Incorrect.

Deletion semantics must reflect business and retention requirements.

---

## Mistake 7: NULL means empty string

Incorrect.

NULL generally represents absence or unknownness according to the schema semantics.

It is distinct from:

```text
''
```

---

## Mistake 8: JSON removes the need for schema design

Incorrect.

Moving poorly modeled data into JSON can simply hide schema problems.

---

## Mistake 9: ER diagram is the database

Incorrect.

ER modeling is conceptual.

The relational schema is an implementation derived from that model.

---

## Mistake 10: Schema design is only about tables

Incorrect.

Good schema design includes:

```text
Relationships

Keys

Constraints

Indexes

Lifecycle

History

Concurrency

Query patterns
```

---

# 129. Schema Design Interview Framework

When given a system-design problem:

```text
1. Identify core entities

2. Define their responsibilities

3. Identify stable identifiers

4. Identify relationships

5. Determine cardinality

6. Determine optional vs mandatory participation

7. Identify relationship attributes

8. Resolve M:N relationships

9. Define keys and foreign keys

10. Add business constraints

11. Check normalization

12. Identify historical/versioning requirements

13. Identify high-frequency query patterns

14. Add indexes deliberately

15. Consider lifecycle and deletion

16. Consider concurrency

17. Validate against example workflows
```

Do not start by immediately writing SQL tables.

Understand the domain first.

---

# 130. Example: Interview Platform Design

Requirements:

```text
A candidate can create multiple interviews.

Each interview targets one role.

An interview has multiple attempts.

Each attempt contains multiple questions.

Each question receives one candidate response per attempt.

Each response may receive an AI evaluation.
```

Possible entities:

```text
Candidate

Role

Interview

InterviewAttempt

Question

AttemptQuestion

Evaluation
```

Relationships:

```text
Candidate 1:N Interview

Role 1:N Interview

Interview 1:N InterviewAttempt

InterviewAttempt 1:N AttemptQuestion

Question 1:N AttemptQuestion

AttemptQuestion 1:0..1 Evaluation
```

This model separates:

```text
Interview configuration

Attempt execution

Question definition

Question occurrence

Evaluation
```

which are different business concepts.

---

# 131. Why Separate Interview and Attempt?

Suppose:

```text
Interview
```

represents:

```text
Backend Engineer mock interview configuration
```

A candidate may attempt it multiple times.

If attempts are stored directly in Interview:

```text
score

started_at

completed_at
```

then a second attempt overwrites the first.

Instead:

```text
Interview
    ↓
InterviewAttempt
```

preserves attempt history.

This is schema design driven by lifecycle semantics.

---

# 132. Final Takeaway

Good schema design starts with:

```text
Business facts
```

not:

```text
Tables
```

The reasoning flow is:

```text
Requirements
     ↓
Entities
     ↓
Attributes
     ↓
Relationships
     ↓
Cardinality
     ↓
Participation
     ↓
Keys
     ↓
Constraints
     ↓
Relational schema
     ↓
Indexes and physical optimization
```

Remember:

```text
1:N
→ Foreign key on many side

M:N
→ Junction table

1:1
→ Foreign key + uniqueness or shared key

Weak entity
→ Identification depends on owner

Surrogate key
→ Does not replace business constraints

Foreign key
→ Integrity

Index
→ Access optimization

ER model
→ Conceptual design

Relational schema
→ Database implementation
```

The strongest interview answer does not merely draw boxes and arrows.

It explains:

```text
Why an entity exists

Why a relationship has that cardinality

Where the foreign key belongs

Which constraints enforce the business rule

How history is preserved

How the design changes when requirements change
```

That is practical database modeling.

---

## Previous

**[← Normalization](./09-normalization.md)**

## Next

**[Query Processing and Optimization →](./11-query-processing-and-optimization.md)**

---

Maintained by **InterviewEraHQ**.
