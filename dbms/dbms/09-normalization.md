# Normalization in DBMS

Normalization is the process of organizing relational data to reduce unnecessary redundancy and prevent data anomalies while preserving meaningful relationships.

It is one of the most frequently tested DBMS topics because it connects:

```text
Relational model
Keys
Functional dependencies
Schema design
Data integrity
Decomposition
Redundancy
Update anomalies
```

A strong candidate should understand:

- Why normalization is needed
- Functional dependencies
- Super keys
- Candidate keys
- Primary keys
- Prime and non-prime attributes
- Partial dependencies
- Transitive dependencies
- 1NF
- 2NF
- 3NF
- BCNF
- Lossless decomposition
- Dependency preservation
- Denormalization
- When normalization helps
- When deliberate denormalization may be justified

The central idea is:

> Store each fact in a structure where its dependencies are represented clearly and unnecessary duplication is minimized.

---

# 1. Why Do We Need Normalization?

Suppose we store interview data like this:

```text
Interview(
    interview_id,
    candidate_id,
    candidate_name,
    candidate_email,
    role_id,
    role_name,
    interviewer_id,
    interviewer_name,
    score
)
```

Example:

```text
1 | 101 | Alex | alex@example.com | 10 | Backend Engineer | 501 | Sam | 82
2 | 101 | Alex | alex@example.com | 20 | System Engineer  | 502 | Mia | 88
3 | 102 | John | john@example.com | 10 | Backend Engineer | 501 | Sam | 76
```

Notice repeated values:

```text
Alex
alex@example.com
Backend Engineer
Sam
```

The same facts are stored multiple times.

This creates redundancy.

---

# 2. Why Is Redundancy a Problem?

Redundancy increases:

```text
Storage duplication

Update complexity

Risk of inconsistent data

Maintenance cost
```

Suppose:

```text
candidate_id = 101
```

changes email from:

```text
alex@example.com
```

to:

```text
alex@newmail.com
```

If candidate data appears in 100 interview rows, all those rows may need updating.

If one row is missed:

```text
same candidate
+
different email values
```

now exist.

This is an **update anomaly**.

---

# 3. Database Anomalies

Poorly designed schemas commonly produce:

```text
Insertion anomaly

Update anomaly

Deletion anomaly
```

Normalization helps reduce these problems.

---

# 4. Update Anomaly

Suppose:

```text
Candidate 101
```

appears in:

```text
50 interview rows
```

with:

```text
email = alex@example.com
```

The email changes.

If only 49 rows are updated:

```text
49 rows → alex@newmail.com

1 row   → alex@example.com
```

The database now contains conflicting values for the same fact.

---

# 5. Insertion Anomaly

Suppose the table stores:

```text
Interview + Candidate + Role
```

and we want to add a new role:

```text
Machine Learning Engineer
```

but no interview exists for that role yet.

If role information can only exist inside an interview row, we may be unable to represent the new role without inventing unrelated interview data or introducing nulls.

That is an insertion anomaly.

---

# 6. Deletion Anomaly

Suppose:

```text
Role 10 = Backend Engineer
```

appears only in one interview row.

If that interview is deleted:

```text
DELETE interview
```

we may accidentally lose the only stored information about:

```text
Role 10
```

even though deleting an interview should not logically delete the role itself.

That is a deletion anomaly.

---

# 7. Normalized Design

Instead of one large table:

```text
Interview
Candidate
Role
Interviewer
```

facts can be separated according to their logical dependencies.

Example:

```text
Candidate(
    candidate_id,
    candidate_name,
    candidate_email
)
```

```text
Role(
    role_id,
    role_name
)
```

```text
Interviewer(
    interviewer_id,
    interviewer_name
)
```

```text
Interview(
    interview_id,
    candidate_id,
    role_id,
    interviewer_id,
    score
)
```

Now:

```text
candidate email
```

is stored with the candidate rather than duplicated across interviews.

---

# 8. Functional Dependency

Normalization is built around **functional dependencies**.

A functional dependency:

```text
X → Y
```

means:

> For each value of X, there is at most one corresponding value of Y in the relation.

Example:

```text
candidate_id → candidate_email
```

means a candidate ID determines one candidate email value within the modeled relation.

Example:

```text
role_id → role_name
```

means a role ID determines its role name.

---

# 9. Functional Dependency Example

Consider:

```text
Student(
    student_id,
    student_name,
    department_id,
    department_name
)
```

Possible dependencies:

```text
student_id → student_name

student_id → department_id

department_id → department_name
```

Therefore:

```text
student_id → department_name
```

can also be inferred transitively.

Functional dependencies describe relationships between attributes.

---

# 10. Functional Dependency Is About Semantics

Suppose current data happens to be:

```text
candidate_name → candidate_email
```

because every current candidate has a unique name.

That does not necessarily mean this is a valid functional dependency.

Two candidates may have the same name.

Functional dependencies should reflect the intended rules of the data model, not accidental properties of the current dataset.

---

# 11. Determinant

In:

```text
X → Y
```

`X` is called the **determinant**.

Example:

```text
candidate_id → candidate_email
```

Here:

```text
candidate_id
```

is the determinant.

A determinant can contain one or multiple attributes.

---

# 12. Trivial Functional Dependency

A functional dependency is trivial when the right-hand side is already contained in the left-hand side.

Example:

```text
(A, B) → A
```

Since:

```text
A
```

is already part of:

```text
(A, B)
```

the dependency is trivial.

---

# 13. Non-Trivial Functional Dependency

Example:

```text
candidate_id → candidate_email
```

where:

```text
candidate_email
```

is not part of:

```text
candidate_id
```

This is non-trivial.

Normalization discussions mainly focus on meaningful non-trivial dependencies.

---

# 14. Attribute Closure

The **closure** of an attribute set helps determine what attributes can be functionally derived from it.

Suppose:

```text
A → B
B → C
C → D
```

Then:

```text
A+
```

contains:

```text
A
B
C
D
```

because:

```text
A → B
A → C
A → D
```

can be inferred.

Attribute closure is especially useful for finding candidate keys.

---

# 15. Closure Example

Relation:

```text
R(A, B, C, D)
```

Dependencies:

```text
A → B
B → C
A → D
```

Start:

```text
A+ = {A}
```

Using:

```text
A → B
```

we get:

```text
A+ = {A, B}
```

Using:

```text
B → C
```

we get:

```text
A+ = {A, B, C}
```

Using:

```text
A → D
```

we get:

```text
A+ = {A, B, C, D}
```

Therefore A determines every attribute in R.

So A is a super key.

---

# 16. Super Key

A **super key** is a set of attributes that uniquely identifies a tuple.

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
```

and:

```text
email
```

are unique.

Possible super keys include:

```text
{candidate_id}

{email}

{candidate_id, email}

{candidate_id, name}

{email, name}
```

A super key may contain unnecessary attributes.

---

# 17. Candidate Key

A **candidate key** is a minimal super key.

Minimal means:

> Removing any attribute from the key would make it no longer uniquely identify the tuple.

If:

```text
candidate_id
```

alone uniquely identifies the candidate, then:

```text
{candidate_id}
```

can be a candidate key.

But:

```text
{candidate_id, name}
```

is not a candidate key if `candidate_id` alone is sufficient.

It is still a super key.

---

# 18. Super Key vs Candidate Key

### Super Key

Uniquely identifies a tuple.

May contain extra attributes.

### Candidate Key

A minimal super key.

Contains no unnecessary attribute.

Therefore:

```text
Every candidate key is a super key.
```

But:

```text
Not every super key is a candidate key.
```

---

# 19. Primary Key

A **primary key** is the candidate key selected as the primary identifier of tuples in the relation.

Suppose candidate keys are:

```text
candidate_id

email
```

We may choose:

```text
candidate_id
```

as:

```text
PRIMARY KEY
```

while:

```text
email
```

remains another candidate key and may be enforced with:

```text
UNIQUE
```

---

# 20. Alternate Key

Candidate keys not selected as the primary key are often called **alternate keys**.

Example:

```text
candidate_id → primary key

email → alternate candidate key
```

Both may uniquely identify a candidate.

Only one is designated as the primary key.

---

# 21. Composite Key

A key containing multiple attributes is a **composite key**.

Example:

```text
Enrollment(
    student_id,
    course_id,
    grade
)
```

Suppose one student can enroll in many courses, and one course can have many students.

Then:

```text
(student_id, course_id)
```

may uniquely identify an enrollment.

This is a composite key.

---

# 22. Prime Attribute

A **prime attribute** is an attribute that belongs to at least one candidate key.

Suppose candidate key:

```text
(student_id, course_id)
```

Then:

```text
student_id
course_id
```

are prime attributes.

---

# 23. Non-Prime Attribute

A **non-prime attribute** does not belong to any candidate key.

Example:

```text
Enrollment(
    student_id,
    course_id,
    grade
)
```

with candidate key:

```text
(student_id, course_id)
```

Then:

```text
grade
```

is non-prime.

---

# 24. Full Functional Dependency

Suppose:

```text
(student_id, course_id) → grade
```

and neither:

```text
student_id → grade
```

nor:

```text
course_id → grade
```

holds.

Then `grade` depends on the entire composite determinant.

This is a **full functional dependency**.

---

# 25. Partial Dependency

A partial dependency occurs when an attribute depends on only part of a composite candidate key.

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

---

# 26. Transitive Dependency

Suppose:

```text
student_id → department_id

department_id → department_name
```

Then:

```text
student_id → department_name
```

through:

```text
department_id
```

This is a transitive dependency in the common normalization context.

Conceptually:

```text
student_id
     ↓
department_id
     ↓
department_name
```

---

# 27. Normal Forms

The commonly discussed normal forms are:

```text
1NF

2NF

3NF

BCNF
```

Additional normal forms include:

```text
4NF

5NF
```

but 1NF through BCNF are the most common in general software-engineering interviews.

---

# 28. First Normal Form — 1NF

A relation is in **First Normal Form (1NF)** when its attributes contain values consistent with the relational model's atomic-domain requirement rather than repeating groups or nested collections in a single attribute.

A common practical explanation:

> Each row-column intersection should represent one value from the attribute's domain, rather than a list of repeated values.

---

# 29. 1NF Violation Example

Bad design:

```text
Candidate(
    candidate_id,
    name,
    skills
)
```

Row:

```text
101 | Alex | Java, Python, SQL
```

The `skills` field stores multiple logical skill values in one column.

This makes operations such as:

```text
Find all Python candidates
```

more difficult and weakens relational modeling.

---

# 30. 1NF Design

Instead:

```text
Candidate(
    candidate_id,
    name
)
```

and:

```text
CandidateSkill(
    candidate_id,
    skill
)
```

Rows:

```text
101 | Java

101 | Python

101 | SQL
```

Now each relationship is represented separately.

---

# 31. Atomic Does Not Mean "Cannot Be Split Physically"

A common misunderstanding is:

> Atomic means a value cannot contain smaller components.

For example:

```text
2026-07-26
```

contains:

```text
year
month
day
```

but can still be one valid date value.

Atomicity in 1NF concerns the modeled domain and relational representation, not whether a string can theoretically be split into characters.

---

# 32. Second Normal Form — 2NF

A relation is in **Second Normal Form (2NF)** when:

```text
It is in 1NF
```

and:

```text
No non-prime attribute is partially dependent on a candidate key.
```

2NF primarily matters when candidate keys contain multiple attributes.

---

# 33. 2NF Violation Example

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

depends only on part of the key.

And:

```text
course_name
```

depends only on another part of the key.

Therefore the relation violates 2NF.

---

# 34. Convert to 2NF

Decompose into:

```text
Student(
    student_id,
    student_name
)
```

```text
Course(
    course_id,
    course_name
)
```

```text
Enrollment(
    student_id,
    course_id,
    grade
)
```

Now:

```text
student_name
```

depends on:

```text
student_id
```

inside Student.

```text
course_name
```

depends on:

```text
course_id
```

inside Course.

And:

```text
grade
```

depends on the full enrollment key.

---

# 35. Important 2NF Interview Point

If every candidate key contains only one attribute, partial dependency on part of a key cannot occur.

Therefore such a relation, if already in 1NF, satisfies the partial-dependency requirement of 2NF.

This is why 2NF problems are commonly demonstrated with composite keys.

---

# 36. Third Normal Form — 3NF

A formal definition:

A relation is in **3NF** if for every non-trivial functional dependency:

```text
X → A
```

at least one of the following is true:

```text
X is a super key
```

or:

```text
A is a prime attribute
```

The common interview intuition is:

> Non-key facts should not depend on other non-key facts in a way that creates transitive dependency from a key.

---

# 37. 3NF Violation Example

Consider:

```text
Employee(
    employee_id,
    employee_name,
    department_id,
    department_name
)
```

Dependencies:

```text
employee_id → employee_name

employee_id → department_id

department_id → department_name
```

Therefore:

```text
employee_id
     ↓
department_id
     ↓
department_name
```

`department_name` depends transitively on `employee_id`.

The relation violates the common 3NF pattern.

---

# 38. Convert to 3NF

Decompose:

```text
Employee(
    employee_id,
    employee_name,
    department_id
)
```

and:

```text
Department(
    department_id,
    department_name
)
```

Now:

```text
department_id → department_name
```

is represented in Department.

Employee references the department through:

```text
department_id
```

---

# 39. 2NF vs 3NF

### 2NF

Focuses on:

```text
Partial dependency on candidate keys
```

especially composite keys.

### 3NF

Addresses problematic dependencies where non-key information determines other information, subject to the formal 3NF condition.

Simplified:

```text
2NF
→ Remove problematic partial dependencies

3NF
→ Remove problematic transitive dependencies
```

But in interviews, know the formal 3NF definition too.

---

# 40. Boyce-Codd Normal Form — BCNF

BCNF provides a stricter condition than 3NF.

A relation is in **BCNF** if for every non-trivial functional dependency:

```text
X → Y
```

`X` is a super key.

In simple terms:

> Every determinant of a non-trivial functional dependency must be a super key.

---

# 41. 3NF vs BCNF

3NF allows:

```text
X → A
```

when either:

```text
X is a super key
```

or:

```text
A is prime
```

BCNF requires:

```text
X is a super key
```

for every non-trivial dependency.

Therefore:

```text
BCNF is stricter than 3NF.
```

Every BCNF relation is in 3NF.

But not every 3NF relation is in BCNF.

---

# 42. BCNF Example

Consider relation:

```text
Teaching(
    student,
    subject,
    teacher
)
```

Suppose the business rules are:

```text
(student, subject) → teacher

teacher → subject
```

Candidate keys include:

```text
(student, subject)

(student, teacher)
```

Therefore:

```text
subject
teacher
student
```

are prime attributes because each belongs to at least one candidate key.

Now consider:

```text
teacher → subject
```

`teacher` is not a super key.

Therefore BCNF is violated.

But because `subject` is a prime attribute, the dependency can satisfy the 3NF condition.

So this relation can be:

```text
3NF
```

but not:

```text
BCNF
```

---

# 43. Why BCNF Exists

3NF permits certain dependencies to preserve useful dependency structures.

BCNF removes more redundancy by requiring every determinant to be a super key.

But decomposing to BCNF can sometimes sacrifice:

```text
Dependency preservation
```

This is why:

```text
BCNF is stricter
```

does not mean:

```text
BCNF is automatically the best design in every practical case.
```

---

# 44. Normal Form Progression

A useful mental model:

```text
Unnormalized design
       ↓
1NF
       ↓
2NF
       ↓
3NF
       ↓
BCNF
```

Each level adds stronger structural requirements.

Generally:

```text
BCNF → 3NF → 2NF → 1NF
```

A BCNF relation satisfies the lower normal forms under the usual definitions.

---

# 45. Decomposition

**Decomposition** means splitting one relation into multiple relations.

Example:

```text
Employee(
    employee_id,
    employee_name,
    department_id,
    department_name
)
```

becomes:

```text
Employee(
    employee_id,
    employee_name,
    department_id
)
```

and:

```text
Department(
    department_id,
    department_name
)
```

But decomposition should not be arbitrary.

Two important properties are:

```text
Lossless join

Dependency preservation
```

---

# 46. Lossless Decomposition

A decomposition is **lossless** if joining the decomposed relations reconstructs exactly the original valid relation without introducing spurious tuples.

Conceptually:

```text
Original Relation
       ↓
Decompose
       ↓
R1 + R2
       ↓
Join
       ↓
Original Relation
```

No information is lost or incorrectly invented.

---

# 47. Lossy Decomposition

Suppose decomposition produces tables that, when joined, generate combinations that were not present in the original relation.

These extra rows are called:

```text
spurious tuples
```

That decomposition is lossy.

Normalization should aim for lossless decomposition.

---

# 48. Lossless Join Condition for Binary Decomposition

Suppose relation:

```text
R
```

is decomposed into:

```text
R1
R2
```

A common lossless-join test states that the decomposition is lossless with respect to functional dependencies if:

```text
R1 ∩ R2 → R1
```

or:

```text
R1 ∩ R2 → R2
```

in the closure of the dependencies.

In other words, the common attributes should functionally determine all attributes of at least one decomposed relation.

---

# 49. Lossless Example

Original:

```text
Employee(
    employee_id,
    employee_name,
    department_id,
    department_name
)
```

Dependencies:

```text
employee_id → employee_name, department_id

department_id → department_name
```

Decompose:

```text
Employee(
    employee_id,
    employee_name,
    department_id
)
```

and:

```text
Department(
    department_id,
    department_name
)
```

Common attribute:

```text
department_id
```

and:

```text
department_id → department_name
```

so `department_id` determines Department.

This decomposition is lossless under the given dependencies.

---

# 50. Dependency Preservation

A decomposition is **dependency preserving** if the original functional dependencies can be enforced by checking the decomposed relations individually, without requiring joins to verify them.

This matters because constraints should ideally be enforceable efficiently.

---

# 51. Dependency Preservation Example

Suppose:

```text
R(A, B, C)
```

dependencies:

```text
A → B

B → C
```

Decompose:

```text
R1(A, B)

R2(B, C)
```

Then:

```text
A → B
```

can be checked in R1.

And:

```text
B → C
```

can be checked in R2.

The dependencies are preserved directly.

---

# 52. Why Dependency Preservation Matters

Suppose a dependency can only be checked by joining multiple tables.

Then every insert or update enforcing that rule may require more complicated coordination.

Dependency-preserving decomposition can make integrity enforcement simpler.

However, sometimes achieving BCNF requires a decomposition that does not preserve every dependency directly.

This creates a design trade-off.

---

# 53. Lossless vs Dependency Preserving

These are different properties.

### Lossless

Question:

```text
Can the original relation be reconstructed correctly?
```

### Dependency Preserving

Question:

```text
Can the original dependencies be enforced locally on decomposed relations?
```

A decomposition can be:

```text
Lossless but not dependency preserving
```

These concepts should not be confused.

---

# 54. Finding Candidate Keys

Given:

```text
R(A, B, C, D)
```

and:

```text
A → B

B → C

A → D
```

Compute:

```text
A+
```

Start:

```text
{A}
```

Apply:

```text
A → B
```

Result:

```text
{A, B}
```

Apply:

```text
B → C
```

Result:

```text
{A, B, C}
```

Apply:

```text
A → D
```

Result:

```text
{A, B, C, D}
```

A determines all attributes.

Therefore A is a super key.

Since A contains only one attribute and is minimal:

```text
A is a candidate key.
```

---

# 55. Composite Candidate Key Example

Relation:

```text
Enrollment(
    student_id,
    course_id,
    semester,
    grade
)
```

Suppose:

```text
(student_id, course_id, semester) → grade
```

and no proper subset uniquely identifies an enrollment.

Then:

```text
(student_id, course_id, semester)
```

is a candidate key.

This matters when checking partial dependencies for 2NF.

---

# 56. Armstrong's Axioms

Functional dependencies can be inferred using rules known as **Armstrong's axioms**.

Core rules:

```text
Reflexivity

Augmentation

Transitivity
```

These inference rules are sound and complete for functional dependencies.

---

# 57. Reflexivity

If:

```text
Y ⊆ X
```

then:

```text
X → Y
```

Example:

```text
(A, B) → A
```

because:

```text
A ⊆ (A, B)
```

---

# 58. Augmentation

If:

```text
X → Y
```

then:

```text
XZ → YZ
```

Example:

If:

```text
A → B
```

then:

```text
AC → BC
```

---

# 59. Transitivity

If:

```text
X → Y
```

and:

```text
Y → Z
```

then:

```text
X → Z
```

Example:

```text
student_id → department_id

department_id → department_name
```

therefore:

```text
student_id → department_name
```

---

# 60. Derived Rules

Other useful rules can be derived from Armstrong's axioms.

Examples:

```text
Union

Decomposition

Pseudotransitivity
```

These are useful when solving functional-dependency problems.

---

# 61. Union Rule

If:

```text
X → Y
```

and:

```text
X → Z
```

then:

```text
X → YZ
```

Example:

```text
candidate_id → name

candidate_id → email
```

therefore:

```text
candidate_id → name, email
```

---

# 62. Decomposition Rule

If:

```text
X → YZ
```

then:

```text
X → Y
```

and:

```text
X → Z
```

Example:

```text
candidate_id → name, email
```

implies:

```text
candidate_id → name
```

and:

```text
candidate_id → email
```

---

# 63. Pseudotransitivity

If:

```text
X → Y
```

and:

```text
WY → Z
```

then:

```text
WX → Z
```

This is less frequently needed in general software interviews but may appear in formal DBMS questions.

---

# 64. Minimal Cover

A **minimal cover** or **canonical cover** is an equivalent set of functional dependencies simplified so that unnecessary attributes and redundant dependencies are removed.

Typical goals include:

```text
Single attribute on RHS

No extraneous attributes

No redundant dependencies
```

Minimal covers are useful in normalization algorithms such as 3NF synthesis.

---

# 65. Why Minimal Cover Matters

Suppose dependencies contain redundant information.

Normalization directly from an unnecessarily complex dependency set makes reasoning harder.

A minimal cover provides a simpler equivalent representation.

For many software-engineering interviews, understanding the concept is enough unless the interviewer explicitly asks you to compute one.

---

# 66. Normalization Example

Suppose we have:

```text
InterviewResult(
    interview_id,
    candidate_id,
    candidate_name,
    role_id,
    role_name,
    score
)
```

Dependencies:

```text
interview_id → candidate_id, role_id, score

candidate_id → candidate_name

role_id → role_name
```

Primary key:

```text
interview_id
```

Since the key contains one attribute, there is no partial dependency on part of that key.

So the relation can satisfy 2NF.

But:

```text
interview_id → candidate_id → candidate_name
```

and:

```text
interview_id → role_id → role_name
```

create transitive dependency patterns.

Therefore the design violates 3NF under these assumptions.

---

# 67. Normalize the Interview Example

Decompose:

```text
Candidate(
    candidate_id,
    candidate_name
)
```

```text
Role(
    role_id,
    role_name
)
```

```text
InterviewResult(
    interview_id,
    candidate_id,
    role_id,
    score
)
```

Now each fact is stored in its logical relation.

```text
candidate_id → candidate_name
```

belongs in Candidate.

```text
role_id → role_name
```

belongs in Role.

InterviewResult stores relationships and interview-specific facts.

---

# 68. Many-to-Many Relationships

Suppose:

```text
Candidate
```

can have many skills.

And:

```text
Skill
```

can belong to many candidates.

Do not store:

```text
skills = "Java,Python,SQL"
```

Instead:

```text
Candidate(
    candidate_id,
    name
)
```

```text
Skill(
    skill_id,
    skill_name
)
```

```text
CandidateSkill(
    candidate_id,
    skill_id
)
```

This models the many-to-many relationship explicitly.

---

# 69. Junction Table

A table such as:

```text
CandidateSkill(
    candidate_id,
    skill_id
)
```

is commonly called:

```text
Junction table

Bridge table

Associative table
```

Its key may be:

```text
(candidate_id, skill_id)
```

or it may use a surrogate key plus an appropriate uniqueness constraint.

---

# 70. Surrogate Key vs Natural Key

A **natural key** comes from meaningful domain data.

Examples:

```text
email
ISBN
country_code
```

when the domain guarantees the required uniqueness and stability.

A **surrogate key** is an artificial identifier.

Examples:

```text
candidate_id
order_id
interview_id
```

often implemented using:

```text
integer
UUID
```

Normalization does not require every table to use either natural or surrogate keys exclusively.

---

# 71. Surrogate Keys Do Not Remove Functional Dependencies

Suppose:

```text
Employee(
    id,
    employee_code,
    department_id,
    department_name
)
```

Adding:

```text
id
```

as a surrogate primary key does not make:

```text
department_id → department_name
```

disappear.

Therefore:

> Adding an ID column does not automatically normalize a schema.

This is a common practical mistake.

---

# 72. Normalization Is Not "Split Everything"

Another misconception:

> More tables means more normalized.

Incorrect.

Normalization follows:

```text
Functional dependencies
Keys
Data semantics
```

Randomly splitting columns into separate tables can make the schema worse.

The goal is not maximum table count.

The goal is correct representation of dependencies with controlled redundancy.

---

# 73. What Is Denormalization?

**Denormalization** intentionally introduces selected redundancy or combines data to optimize particular workloads.

Example normalized design:

```text
Candidate

Interview

InterviewScore
```

A reporting system may maintain:

```text
candidate_interview_summary
```

containing precomputed values such as:

```text
candidate_id
total_interviews
average_score
last_interview_at
```

This duplicates information derivable from base tables.

But it may make frequent reporting much faster.

---

# 74. Why Denormalize?

Possible reasons:

```text
Reduce expensive joins

Speed up read-heavy workloads

Precompute aggregates

Support analytics

Reduce repeated computation

Optimize specific query patterns
```

But denormalization introduces trade-offs.

---

# 75. Denormalization Costs

Duplicated data must remain synchronized.

Potential problems:

```text
Stale data

More complex writes

Synchronization bugs

Extra storage

Harder integrity management
```

Therefore:

> Denormalization should solve a measured problem, not an imagined one.

---

# 76. Normalize First, Measure Later

A good default for transactional relational systems is:

```text
Model data correctly
      ↓
Normalize to a sensible level
      ↓
Create appropriate indexes
      ↓
Measure workload
      ↓
Identify bottleneck
      ↓
Denormalize selectively if justified
```

Do not begin with:

```text
Joins might be slow
```

and duplicate everything.

---

# 77. Are Joins Bad?

No.

Relational databases are designed to join relations.

A query such as:

```sql
SELECT
    c.name,
    r.role_name,
    i.score
FROM interviews AS i
JOIN candidates AS c
    ON c.candidate_id = i.candidate_id
JOIN roles AS r
    ON r.role_id = i.role_id;
```

is not automatically a performance problem.

Performance depends on:

```text
Data volume

Indexes

Query plan

Join strategy

Selectivity

Hardware

Workload
```

Do not denormalize merely to avoid every join.

---

# 78. Normalization vs Performance

Normalization can:

```text
Reduce redundant writes

Improve integrity

Simplify updates
```

But highly normalized designs may require more joins for some read patterns.

Denormalization can:

```text
Reduce read-time joins
```

but increases:

```text
Write complexity

Consistency burden
```

Schema design is an engineering trade-off.

---

# 79. OLTP and Normalization

**OLTP** systems typically handle:

```text
Frequent inserts

Frequent updates

Small transactions

Operational workflows
```

Normalization is often valuable because it reduces redundancy and supports consistent updates.

Examples:

```text
Banking

Orders

User accounts

Interview sessions

Subscriptions
```

---

# 80. Analytics and Denormalization

Analytical workloads often prioritize:

```text
Large scans

Aggregations

Reporting

Read performance
```

Models such as:

```text
Star schema
```

may intentionally include denormalized dimensions or structures optimized for analytics.

Therefore the ideal schema depends on workload.

---

# 81. Normalization Does Not Guarantee Good Performance

A perfectly normalized schema can still perform poorly because of:

```text
Missing indexes

Bad queries

Poor execution plans

Excessive network calls

N+1 queries

Bad partitioning

Lock contention
```

Normalization primarily addresses:

```text
Data organization
Redundancy
Dependencies
Integrity
```

It is not a universal performance optimization.

---

# 82. Normalization Does Not Guarantee Integrity

Suppose a schema is normalized but the application allows:

```text
negative credits
```

when business rules forbid them.

The schema may still need:

```text
CHECK constraints

Transactions

Concurrency control

Application validation
```

Normalization is one component of database correctness.

---

# 83. Common Interview Question: What Is Normalization?

Weak answer:

> Normalization removes duplicate data.

Better answer:

> Normalization is the systematic organization of relational data based on keys and dependencies to reduce unnecessary redundancy and prevent insertion, update, and deletion anomalies while preserving the relationships and integrity required by the data model.

---

# 84. Common Interview Question: Why Normalize?

A strong answer:

> Normalization reduces duplicated facts and makes each fact easier to maintain consistently. This helps prevent update, insertion, and deletion anomalies. The schema can later be selectively denormalized if measured workload requirements justify the additional consistency complexity.

---

# 85. Common Interview Question: Explain 1NF

Strong answer:

> 1NF requires a relational representation without repeating groups or collection-like values inside a single attribute. Each attribute value should belong to its defined domain, and multi-valued relationships should normally be represented using separate rows or relations.

---

# 86. Common Interview Question: Explain 2NF

Strong answer:

> A relation is in 2NF when it is in 1NF and no non-prime attribute depends on only a proper subset of a candidate key. This issue is especially relevant when candidate keys are composite.

---

# 87. Common Interview Question: Explain 3NF

Strong answer:

> Formally, a relation is in 3NF when for every non-trivial dependency X → A, either X is a super key or A is a prime attribute. Practically, 3NF removes problematic transitive dependencies where non-key facts determine other non-key facts.

---

# 88. Common Interview Question: Explain BCNF

Strong answer:

> A relation is in BCNF when every determinant of a non-trivial functional dependency is a super key. BCNF is stricter than 3NF because 3NF permits certain dependencies whose right-hand side is a prime attribute even when the determinant is not a super key.

---

# 89. Common Interview Question: 3NF vs BCNF

### 3NF

For:

```text
X → A
```

allows:

```text
X is super key
```

or:

```text
A is prime
```

### BCNF

Requires:

```text
X is super key
```

for every non-trivial dependency.

Therefore:

```text
Every BCNF relation is 3NF.

Not every 3NF relation is BCNF.
```

---

# 90. Common Interview Question: What Is Partial Dependency?

A partial dependency occurs when a non-prime attribute depends on only part of a composite candidate key.

Example:

```text
(student_id, course_id) → grade

student_id → student_name
```

`student_name` depends only on part of the composite key.

This violates 2NF.

---

# 91. Common Interview Question: What Is Transitive Dependency?

Example:

```text
employee_id → department_id

department_id → department_name
```

Therefore:

```text
employee_id → department_name
```

through another attribute.

This is the common transitive-dependency pattern addressed when moving toward 3NF.

---

# 92. Common Interview Question: What Is Lossless Decomposition?

Strong answer:

> A decomposition is lossless when joining the decomposed relations reconstructs the original valid relation without losing information or generating spurious tuples.

---

# 93. Common Interview Question: What Is Dependency Preservation?

Strong answer:

> A decomposition is dependency preserving when the original functional dependencies can be enforced through the decomposed relations without needing joins to validate those dependencies.

---

# 94. Common Interview Question: Lossless vs Dependency Preserving

### Lossless

Concerned with:

```text
Preserving information
```

### Dependency Preserving

Concerned with:

```text
Preserving efficient enforcement of dependencies
```

A good decomposition ideally provides both, but some normalization decisions involve trade-offs.

---

# 95. Common Interview Question: Why Not Normalize Everything to BCNF?

A strong answer:

> BCNF removes more dependency-based redundancy than 3NF, but a BCNF decomposition may not preserve all functional dependencies directly. In practice, schema design also considers integrity enforcement, query patterns, complexity, and workload requirements. Higher normal form is not automatically better in every system.

---

# 96. Common Interview Question: When Would You Denormalize?

Possible answer:

> I would consider denormalization after identifying a real read-performance bottleneck where normalized joins or repeated aggregation are materially expensive. I would first measure the workload and evaluate indexing and query optimization. If denormalization is justified, I would explicitly design how duplicated data remains consistent.

---

# 97. Common Interview Questions

Be prepared to answer:

1. What is normalization?
2. Why is normalization needed?
3. What is redundancy?
4. What is an insertion anomaly?
5. What is an update anomaly?
6. What is a deletion anomaly?
7. What is a functional dependency?
8. What is a determinant?
9. What is a trivial functional dependency?
10. What is attribute closure?
11. How do you find a candidate key?
12. What is a super key?
13. What is a candidate key?
14. Super key vs candidate key?
15. Candidate key vs primary key?
16. What is an alternate key?
17. What is a composite key?
18. What is a prime attribute?
19. What is a non-prime attribute?
20. What is a full functional dependency?
21. What is a partial dependency?
22. What is a transitive dependency?
23. What is 1NF?
24. What is 2NF?
25. What is 3NF?
26. What is BCNF?
27. 2NF vs 3NF?
28. 3NF vs BCNF?
29. Can a relation be 3NF but not BCNF?
30. What is decomposition?
31. What is lossless decomposition?
32. What are spurious tuples?
33. What is dependency preservation?
34. Lossless vs dependency preserving?
35. What are Armstrong's axioms?
36. What is a minimal cover?
37. What is denormalization?
38. When should you denormalize?
39. Are joins bad?
40. Does adding a surrogate ID normalize a table?

---

# 98. Common Mistakes

## Mistake 1: Normalization means removing every duplicate value

Incorrect.

Repeated values can be legitimate.

Normalization addresses redundancy caused by dependencies and poor representation of facts.

---

## Mistake 2: More tables means more normalized

Incorrect.

Normalization is based on dependencies, not table count.

---

## Mistake 3: Every table with a primary key is normalized

Incorrect.

Example:

```text
id
department_id
department_name
```

can still contain:

```text
department_id → department_name
```

and problematic redundancy.

---

## Mistake 4: Adding an auto-increment ID fixes normalization

Incorrect.

Surrogate keys do not remove existing dependencies.

---

## Mistake 5: 2NF means no transitive dependencies

Incorrect.

2NF focuses on partial dependencies of non-prime attributes on candidate keys.

3NF addresses the relevant transitive-dependency pattern.

---

## Mistake 6: 3NF and BCNF are the same

Incorrect.

BCNF is stricter.

---

## Mistake 7: BCNF is always the best production schema

Incorrect.

Dependency preservation and workload requirements can matter.

---

## Mistake 8: Normalization automatically makes queries faster

Incorrect.

Normalization primarily improves data organization and integrity.

Query performance must be measured separately.

---

## Mistake 9: Denormalization is bad database design

Incorrect.

Intentional, measured denormalization can be appropriate.

Uncontrolled redundancy is the problem.

---

## Mistake 10: Store arrays as comma-separated strings to avoid another table

Usually poor relational modeling for queryable many-valued relationships.

Model the relationship explicitly when individual values need relational semantics.

---

# 99. Interview Scenario

Suppose:

```text
Interview(
    interview_id,
    candidate_id,
    candidate_name,
    candidate_email,
    role_id,
    role_name,
    score
)
```

Dependencies:

```text
interview_id → candidate_id, role_id, score

candidate_id → candidate_name, candidate_email

role_id → role_name
```

Ask:

```text
What is the key?
```

Assume:

```text
interview_id
```

is the candidate key.

Then:

```text
candidate_name
candidate_email
role_name
```

depend transitively on the interview key through other identifiers.

Better decomposition:

```text
Candidate(
    candidate_id,
    candidate_name,
    candidate_email
)
```

```text
Role(
    role_id,
    role_name
)
```

```text
Interview(
    interview_id,
    candidate_id,
    role_id,
    score
)
```

Now each relation represents a clearer logical fact.

---

# 100. Normalization Problem-Solving Framework

When given a relation:

```text
1. List all attributes

2. Identify functional dependencies

3. Find candidate keys

4. Identify prime and non-prime attributes

5. Check 1NF

6. Check partial dependencies

7. Check transitive dependencies

8. Check every determinant for BCNF

9. Decompose where necessary

10. Verify lossless join

11. Check dependency preservation

12. Evaluate practical workload requirements
```

Do not jump directly to:

```text
Split this into three tables.
```

Show the dependency reasoning.

---

# 101. Quick Revision

Remember:

```text
Functional Dependency

X → Y

means

X determines Y
```

Keys:

```text
Super Key
→ Uniquely identifies tuple

Candidate Key
→ Minimal super key

Primary Key
→ Selected candidate key

Prime Attribute
→ Part of some candidate key
```

Normal forms:

```text
1NF
→ Proper relational values / no repeating groups

2NF
→ 1NF + no partial dependency of non-prime attributes on candidate keys

3NF
→ For X → A, X is super key or A is prime

BCNF
→ Every non-trivial determinant is a super key
```

Decomposition:

```text
Lossless
→ No information loss or spurious reconstruction

Dependency Preserving
→ Dependencies enforceable without joining decomposed relations
```

---

# 102. Final Takeaway

Normalization is not about blindly creating more tables.

It is about understanding:

```text
What fact does this attribute represent?

What determines that fact?

Where should that fact live?
```

A poor schema often looks like:

```text
One giant table
      ↓
Repeated facts
      ↓
Update anomalies
      ↓
Inconsistent data
```

A normalized design aims for:

```text
Clear dependencies
      ↓
Logical relations
      ↓
Reduced unnecessary redundancy
      ↓
Safer updates
```

But production database design also considers:

```text
Query patterns

Indexes

Read/write ratio

Operational complexity

Analytics requirements
```

Therefore:

```text
Normalize intentionally

Measure performance

Denormalize selectively
```

The strongest interview candidate does not merely memorize:

```text
1NF → 2NF → 3NF → BCNF
```

They can take a schema, identify its dependencies, explain exactly why a normal form is violated, decompose it correctly, and discuss the practical trade-offs.

---

## Previous

**[← Deadlocks](./08-deadlocks.md)**

## Next

**[ER Model and Schema Design →](./10-er-model-and-schema-design.md)**

---

Maintained by **InterviewEraHQ**.
