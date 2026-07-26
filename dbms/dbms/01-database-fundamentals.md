# Database Fundamentals

Database fundamentals are among the most common topics in software engineering interviews. Before learning SQL, indexing, transactions, or normalization, you should understand what a database is, why a DBMS exists, and what problems it solves.

---

## 1. What Is Data?

**Data** represents raw facts or information that can be stored, processed, and interpreted.

Examples:

- Candidate name
- Email address
- Interview score
- Product price
- Order status
- Login timestamp

Applications organize related data so it can be stored, retrieved, updated, and analyzed efficiently.

---

## 2. What Is a Database?

A **database** is an organized collection of data designed for efficient storage, retrieval, modification, and management.

For example, an interview preparation platform might store information about:

```text
Users
Resumes
Interviews
Questions
Responses
Scores
Reports
```

Instead of keeping this information across unrelated files, a database provides a structured way to organize and work with it.

### Simple Example

Consider a `candidates` table:

| id | name | role |
|---:|---|---|
| 1 | Alex | Backend Developer |
| 2 | Sam | Frontend Developer |
| 3 | Priya | Data Analyst |

Each row represents a candidate, while each column represents an attribute of that candidate.

---

## 3. What Is a DBMS?

A **Database Management System (DBMS)** is software that allows applications and users to create, access, modify, organize, and manage databases.

A DBMS commonly provides mechanisms for:

- Data storage and retrieval
- Query processing
- Data modification
- Constraints and validation
- Concurrent access
- Transactions
- Authentication and permissions
- Backup and recovery
- Data consistency and integrity

Examples include:

- PostgreSQL
- MySQL
- SQLite
- Microsoft SQL Server
- Oracle Database
- MongoDB

The exact capabilities and implementation differ between database systems.

---

## 4. Database vs DBMS

A database and a DBMS are related, but they are not the same thing.

| Database | DBMS |
|---|---|
| Organized collection of data | Software used to manage databases |
| Represents stored information | Provides mechanisms to access and modify information |
| Contains records and relationships | Handles operations, constraints, transactions, permissions, and more |

### Interview Question

**What is the difference between a database and a DBMS?**

A strong answer:

> A database is an organized collection of stored data, while a DBMS is the software system used to create, access, modify, secure, and manage that data.

---

## 5. Why Use a DBMS Instead of Files?

Applications can store data directly in files.

For a very small application, you might have:

```text
users.txt
interviews.txt
questions.txt
scores.txt
```

This can work for simple use cases, but complexity increases as the application grows.

The application may need to handle:

- Efficient searching
- Relationships between records
- Duplicate data
- Concurrent reads and writes
- Access control
- Data validation
- Partial failures
- Recovery
- Consistency across multiple operations

A DBMS provides abstractions and mechanisms for handling many of these concerns.

### Example

Suppose two requests attempt to update the same candidate's interview result simultaneously.

With basic file storage, the application must carefully coordinate those writes to avoid corrupting or overwriting data.

Database systems provide concurrency-control and transaction mechanisms that can help applications handle these situations correctly.

### Important Interview Point

Using a DBMS does **not** automatically guarantee:

- Fast queries
- Perfect database design
- No downtime
- No data loss
- Unlimited scalability
- Correct application behavior

Schema design, indexes, queries, transaction boundaries, infrastructure, backups, and application architecture still matter.

---

## 6. Major Types of Databases

Databases can be categorized according to their data models and architecture.

Two broad categories frequently discussed in interviews are relational and NoSQL databases.

---

### Relational Databases

Relational databases organize data using relations, commonly represented as tables containing rows and columns.

Examples include:

- PostgreSQL
- MySQL
- SQLite
- Microsoft SQL Server
- Oracle Database

Example:

| id | name | role |
|---:|---|---|
| 1 | Alex | Backend Developer |
| 2 | Sam | Frontend Developer |

Relationships between data can be represented using keys and constraints.

For example:

```text
Candidate
    |
    └── Interviews
```

One candidate may have multiple interview records.

Relational databases are commonly useful when:

- Relationships between entities matter
- Structured querying is important
- Constraints need to be enforced
- Transactional consistency is important

This is not an exhaustive rule. Database selection should be based on workload requirements.

---

### NoSQL Databases

**NoSQL** refers to a broad family of database systems that use data models other than, or beyond, the traditional relational table model.

Common categories include:

- Document databases
- Key-value databases
- Wide-column databases
- Graph databases

A document might look like:

```json
{
  "id": 1,
  "name": "Alex",
  "role": "Backend Developer",
  "skills": ["Python", "PostgreSQL"]
}
```

Different NoSQL systems make different trade-offs around:

- Data modeling
- Relationships
- Consistency
- Distribution
- Query capabilities
- Scaling
- Schema flexibility

Therefore:

> SQL vs NoSQL should not be treated as "which one is better?"

The better question is:

> Which database model and system best match the workload and requirements?

---

## 7. What Is a Schema?

A **schema** describes the logical structure and organization of data.

In a relational database, a schema may define:

- Tables
- Columns
- Data types
- Constraints
- Relationships
- Indexes
- Other database objects

Example:

```sql
CREATE TABLE candidates (
    id INTEGER PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL
);
```

This definition specifies that:

- `id` identifies a candidate
- `name` is required
- `email` is required
- `email` must be unique

Schemas help establish predictable structures and enforce important rules about stored data.

---

## 8. What Are CRUD Operations?

A large portion of application database interaction can be described using four basic operations:

| Operation | Meaning |
|---|---|
| Create | Add new data |
| Read | Retrieve existing data |
| Update | Modify existing data |
| Delete | Remove data |

Together, these are commonly called **CRUD**.

### Create

```sql
INSERT INTO candidates (id, name, email)
VALUES (1, 'Alex', 'alex@example.com');
```

### Read

```sql
SELECT *
FROM candidates;
```

### Update

```sql
UPDATE candidates
SET name = 'Alex Kumar'
WHERE id = 1;
```

### Delete

```sql
DELETE FROM candidates
WHERE id = 1;
```

CRUD is a useful abstraction, but real database applications also involve transactions, aggregation, joins, constraints, indexing, concurrency, and other operations.

---

## 9. What Is Data Integrity?

**Data integrity** means maintaining the validity, consistency, and reliability of stored data according to the rules of the system.

Consider these requirements:

```text
Candidate email must be unique.

Every interview must reference a valid candidate.

A required candidate name cannot be NULL.

A score must remain within the application's permitted range.
```

Relational databases can enforce many such rules using constraints.

Examples include:

- PRIMARY KEY
- FOREIGN KEY
- UNIQUE
- NOT NULL
- CHECK

Example:

```sql
CREATE TABLE interview_scores (
    id INTEGER PRIMARY KEY,
    candidate_id INTEGER NOT NULL,
    score INTEGER CHECK (score >= 0 AND score <= 100)
);
```

The `CHECK` constraint prevents values outside the allowed range from satisfying that constraint.

### Application Validation vs Database Constraints

Application-level validation is useful for:

- Better user feedback
- Business workflows
- Complex application rules

Database constraints are valuable for protecting important invariants at the storage layer.

For critical rules, relying only on frontend validation is usually insufficient.

---

## 10. What Is Data Redundancy?

**Data redundancy** occurs when the same information is unnecessarily stored in multiple places.

Consider:

| interview_id | candidate_id | candidate_name | candidate_email |
|---:|---:|---|---|
| 101 | 1 | Alex | alex@example.com |
| 102 | 1 | Alex | alex@example.com |
| 103 | 1 | Alex | alex@example.com |

The candidate's name and email are repeated for every interview.

This can create problems.

For example, if the email changes, multiple rows may need to be updated.

Failure to update all relevant copies can create inconsistent data.

Normalization is one technique used in relational database design to reduce certain forms of redundancy and prevent anomalies.

We will cover normalization separately.

---

## 11. What Is Data Independence?

**Data independence** refers to the ability to change aspects of how data is structured or stored without requiring corresponding changes at every other layer of the system.

It is commonly discussed at two levels.

### Physical Data Independence

Changes to physical storage should ideally not require changes to the logical model used by applications.

Examples may include changes to:

- Storage organization
- Indexes
- Physical access methods

### Logical Data Independence

Changes to the logical schema should ideally have limited impact on external views or applications.

Logical data independence is generally more difficult to achieve than physical data independence.

### Interview Point

You do not usually need a long theoretical explanation.

Understand the central idea:

> Database systems provide abstraction between how data is used and how it is internally represented or stored.

---

## 12. DBMS Architecture: Basic Idea

A simplified application flow might look like:

```text
User
  ↓
Application
  ↓
DBMS
  ↓
Database Storage
```

The application does not usually manipulate raw database files directly.

Instead, it communicates with the database system using:

- SQL
- Drivers
- Database protocols
- ORMs
- Database APIs

The DBMS handles the underlying database operations.

---

## 13. Relational Example

Suppose an interview platform stores candidates and their interviews.

### Candidates

| id | name |
|---:|---|
| 1 | Alex |
| 2 | Sam |

### Interviews

| id | candidate_id | score |
|---:|---:|---:|
| 101 | 1 | 82 |
| 102 | 1 | 91 |
| 103 | 2 | 76 |

Here:

```text
candidates.id
```

identifies a candidate.

And:

```text
interviews.candidate_id
```

can reference the corresponding candidate.

Conceptually:

```text
Candidate 1
├── Interview 101
└── Interview 102

Candidate 2
└── Interview 103
```

This illustrates how relational databases can model relationships between entities.

Keys and constraints are covered in the next guide.

---

## 14. Common Interview Questions

Make sure you can answer these without simply memorizing definitions.

### Fundamentals

1. What is data?
2. What is a database?
3. What is a DBMS?
4. What is the difference between a database and a DBMS?
5. Why would you use a DBMS instead of plain files?
6. What is a database schema?
7. What are CRUD operations?
8. What is data integrity?
9. What is data redundancy?
10. What is data independence?

### Relational vs NoSQL

11. What is a relational database?
12. What is a NoSQL database?
13. Does NoSQL mean there is no schema?
14. When might a relational database be appropriate?
15. What factors would you consider when choosing a database?

### Practical Reasoning

16. Why shouldn't important validation exist only in the frontend?
17. Does using a DBMS automatically make an application scalable?
18. Can relational databases scale?
19. Is NoSQL always faster than SQL?
20. Why can duplicated data become a problem?

---

## 15. Common Mistakes

### Mistake 1: Database and DBMS are the same thing

They are related but distinct.

A database represents the stored data, while the DBMS provides the software mechanisms for managing it.

---

### Mistake 2: NoSQL means no schema

Many NoSQL databases support flexible data models, but applications still have data structures, expectations, and validation requirements.

Some NoSQL systems also provide explicit schema-validation capabilities.

---

### Mistake 3: NoSQL is always faster than SQL

There is no universal performance winner.

Performance depends on factors such as:

- Workload
- Query patterns
- Data model
- Indexes
- Distribution
- Consistency requirements
- Hardware
- Configuration
- Database implementation

---

### Mistake 4: SQL databases cannot scale

Relational databases can scale substantially.

Possible strategies include:

- Better indexing
- Query optimization
- Caching
- Read replicas
- Partitioning
- Vertical scaling
- Horizontal architectures where appropriate

The right approach depends on the system.

---

### Mistake 5: A DBMS guarantees data will never be lost

Database systems provide durability and recovery mechanisms, but real reliability also depends on:

- Configuration
- Storage
- Replication
- Backups
- Infrastructure
- Failure model
- Operational practices

Avoid absolute claims.

---

## 16. Interview Answer Framework

For many DBMS questions, use this structure:

**Definition → Why it exists → Example → Trade-off**

Suppose the interviewer asks:

**Why do we use a DBMS?**

A weak answer:

> A DBMS is used to manage databases.

A stronger answer:

> A DBMS is software that manages access to databases and provides mechanisms for querying, constraints, transactions, concurrency control, permissions, and recovery. For example, PostgreSQL can manage relational data while enforcing keys and transactional guarantees. A DBMS solves many data-management problems, but good performance and reliability still depend on schema design, queries, indexes, infrastructure, and application architecture.

The second answer demonstrates understanding rather than memorization.

---

## 17. Quick Revision

Before moving forward, you should be able to explain:

- Database
- DBMS
- Database vs DBMS
- File storage vs DBMS
- Relational databases
- NoSQL databases
- Schema
- CRUD
- Data integrity
- Data redundancy
- Data independence

You should also understand that database engineering involves **trade-offs rather than universal rules**.

---

## Interview Tip

Do not stop after defining a database concept.

When possible, explain:

```text
What is it?
      ↓
Why does it exist?
      ↓
How does it work?
      ↓
Where would we use it?
      ↓
What trade-offs does it introduce?
```

This makes your answer much closer to what technical interviewers expect from candidates who genuinely understand the topic.

---

## Next

Continue with:

**[Keys and Constraints →](./02-keys-and-constraints.md)**

---

Maintained by **InterviewEraHQ**.
