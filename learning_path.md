## Learning path
core for backend 4, 5, 6, 11, 13
Once Chapters 1, 2, 4, 5 and 6 make sense, I'd jump to:

- Chapter 10
roles
permissions
GRANT
REVOKE
authentication
authorization
pg_hba.conf

- chapter 11
transactions
ACID
isolation levels
MVCC
concurrency
deadlocks
VACUUM
WAL
checkpoints

- Chapter 13
indexes
B-tree
query plans
EXPLAIN
EXPLAIN ANALYZE
sequential scans
index scans
query optimization
statistics

chapter 16
postgresql.conf
server configuration
monitoring
statistics
connections
resource usage

- Chapter 15
pg_dump
pg_restore
backups
restoration
point-in-time recovery


 ## What to focus on
What becomes valuable is being able to look at a problem and immediately understand **what the database should be doing, why, and whether the AI-generated SQL is actually good**.

For an interview or real development, I'd divide PostgreSQL knowledge into three levels.

## 1. Know these almost by heart

These are the things where you should be able to write SQL without asking AI.

### A. Joins

You should be extremely comfortable with:

```sql
INNER JOIN
LEFT JOIN
```

and understand:

```text
INNER JOIN → only matching rows
LEFT JOIN  → everything from the left + matches
```

Using `dvdrental`:

```sql
SELECT
    c.first_name,
    c.last_name,
    SUM(p.amount) AS total_paid
FROM customer c
JOIN payment p
    ON p.customer_id = c.customer_id
GROUP BY c.customer_id, c.first_name, c.last_name;
```

You should be able to explain **why the join exists**, not merely produce it.

Also know why this can produce duplicate-looking rows:

```sql
customer → payment
```

because one customer can have many payments.

That understanding is more important than memorizing syntax.

---

# 2. Aggregation should be second nature

Know:

```sql
COUNT()
SUM()
AVG()
MIN()
MAX()
GROUP BY
HAVING
```

For example:

```sql
SELECT
    c.customer_id,
    c.first_name,
    COUNT(r.rental_id) AS rental_count
FROM customer c
LEFT JOIN rental r
    ON r.customer_id = c.customer_id
GROUP BY
    c.customer_id,
    c.first_name
ORDER BY rental_count DESC;
```

And immediately understand the difference between:

```sql
WHERE
```

and:

```sql
HAVING
```

**WHERE filters rows before aggregation.**

**HAVING filters groups after aggregation.**

That's an interview question worth being able to answer without hesitation.

---

# 3. Understand relationships deeply

This is probably more important than memorizing SQL.

You should look at:

```text
film
film_actor
actor
```

and immediately recognize:

```text
film ↔ actor
```

is a **many-to-many relationship** implemented through:

```text
film_actor
```

Likewise:

```text
customer
   ↓
rental
   ↓
inventory
   ↓
film
```

Understanding these relationships lets you construct queries instead of guessing them.

If an interviewer says:

> "Find the actors who appeared in the most films."

You should mentally construct:

```text
actor
  ↓
film_actor
  ↓
film
```

before writing a single line of SQL.

That's the kind of database thinking AI can't replace very well.

---

# 4. Subqueries and CTEs

You should understand both, but don't obsess over memorizing every variation.

Know how to reason about:

```sql
SELECT ...
FROM ...
WHERE x IN (
    SELECT ...
);
```

and:

```sql
WITH something AS (
    SELECT ...
)
SELECT ...
FROM something;
```

For example, suppose you want customers who have spent more than the average customer:

```sql
SELECT
    c.customer_id,
    c.first_name,
    SUM(p.amount) AS total_paid
FROM customer c
JOIN payment p
    ON p.customer_id = c.customer_id
GROUP BY c.customer_id, c.first_name
HAVING SUM(p.amount) > (
    SELECT AVG(total_paid)
    FROM (
        SELECT SUM(amount) AS total_paid
        FROM payment
        GROUP BY customer_id
    ) x
);
```

You don't necessarily need to memorize this exact query.

You need to understand **what each level is doing**.

---

# 5. Window functions — learn these seriously

This is one area where I'd deliberately become strong.

Know:

```sql
ROW_NUMBER()
RANK()
DENSE_RANK()
LAG()
LEAD()
SUM() OVER()
AVG() OVER()
PARTITION BY
ORDER BY
```

For example:

```sql
SELECT
    customer_id,
    payment_date,
    amount,
    SUM(amount) OVER (
        PARTITION BY customer_id
        ORDER BY payment_date
    ) AS running_total
FROM payment;
```

You should understand why this is different from:

```sql
GROUP BY customer_id
```

`GROUP BY` **collapses rows**.

A window function generally **keeps the rows while calculating information across related rows**.

That's a very useful mental distinction.

---

# 6. Constraints — know these extremely well

This is one area where I would absolutely expect a backend developer to understand the fundamentals without AI.

Know:

```sql
PRIMARY KEY
FOREIGN KEY
UNIQUE
NOT NULL
CHECK
DEFAULT
```

For example:

```sql
CREATE TABLE customer (
    customer_id BIGSERIAL PRIMARY KEY,
    email TEXT NOT NULL UNIQUE,
    age INTEGER CHECK (age >= 18)
);
```

And understand **why constraints belong in the database**, rather than relying entirely on application validation.

If your PHP application says:

```text
email must be unique
```

but the database doesn't enforce uniqueness, two simultaneous requests can potentially violate your assumption.

That's database thinking.

---

# 7. Indexes — VERY important

This is probably one of the biggest things I'd want you to understand for real-world backend work.

Know:

```sql
CREATE INDEX ...
```

but more importantly understand:

> **Why does an index make some queries faster, and what does it cost?**

For example:

```sql
CREATE INDEX idx_payment_customer_id
ON payment(customer_id);
```

Then understand why this query could benefit:

```sql
SELECT *
FROM payment
WHERE customer_id = 123;
```

But also understand:

> Indexes aren't free.

They consume storage and have maintenance/write costs.

You should also become comfortable with:

```sql
EXPLAIN
```

and eventually:

```sql
EXPLAIN ANALYZE
```

If an AI gives you:

```sql
SELECT ...
```

and you can say:

> "Let's see what PostgreSQL's query planner actually does."

you're already operating at a different level from someone who simply accepts generated SQL.

---

# 8. Transactions

Know this without hesitation:

```sql
BEGIN;

UPDATE ...;

INSERT ...;

COMMIT;
```

and:

```sql
ROLLBACK;
```

Understand **atomicity**.

For example, imagine a payment operation requires multiple changes:

```text
BEGIN
   ↓
create payment
   ↓
update account balance
   ↓
create transaction record
   ↓
COMMIT
```

If something fails halfway:

```text
ROLLBACK
```

You don't want half the operation persisted.

Also learn:

```text
ACID
```

and eventually PostgreSQL's:

```text
MVCC
```

These become particularly important when you're building real applications.

---

# 9. NULL

This sounds trivial.

It isn't.

You should deeply understand:

```sql
NULL
```

and why this doesn't work:

```sql
WHERE email = NULL
```

You need:

```sql
WHERE email IS NULL
```

Also understand:

```sql
COALESCE()
```

and how `NULL` propagates through expressions.

A surprising amount of SQL confusion comes from misunderstanding `NULL`.

---

# 10. Data modeling

This is where I'd spend **more time than memorizing obscure SQL syntax**.

Given a requirement like:

> Users can create posts. Posts can have comments. Users can like posts.

You should naturally start thinking:

```text
users
  │
  ├── posts
  │     │
  │     └── comments
  │
  └── likes
```

Then:

```text
users
posts
comments
post_likes
```

You should know when something should be:

* a separate table
* a foreign key
* a many-to-many junction table
* a nullable relationship
* constrained with `UNIQUE`
* indexed

This is exactly the kind of reasoning that remains valuable when AI can generate the SQL for you.

---

# 11. Learn enough PostgreSQL internals to debug AI

You don't need to become a PostgreSQL internals researcher.

But understand these concepts:

```text
Database
Schema
Table
Index
Query planner
Transactions
MVCC
Locks
Vacuum
WAL
Connection
```

Especially:

### MVCC

Understand roughly why PostgreSQL can have:

```text
Transaction A → reading
Transaction B → updating
```

without every reader simply blocking every writer.

### VACUUM

Understand why PostgreSQL needs vacuuming and what dead tuples are.

### WAL

Understand that PostgreSQL doesn't simply overwrite database files and hope for the best; **Write-Ahead Logging** is fundamental to durability and recovery.

These concepts make production debugging much easier.

---

# 12. Know PostgreSQL-specific features

Once you're comfortable with SQL, learn things that distinguish PostgreSQL from generic SQL.

I'd prioritize:

```text
JSONB
ARRAY
UUID
ENUM
RETURNING
ON CONFLICT
ILIKE
FILTER
DISTINCT ON
GENERATE_SERIES()
```

For example:

```sql
INSERT INTO customer (...)
VALUES (...)
RETURNING customer_id;
```

That's extremely useful in application development.

And:

```sql
INSERT INTO ...
VALUES (...)
ON CONFLICT (...)
DO UPDATE SET ...;
```

is something you'll encounter frequently in real applications.

---

# What I would NOT memorize

Don't waste mental energy memorizing things like:

```text
every CREATE INDEX variation
every ALTER TABLE variation
every obscure PostgreSQL configuration parameter
every date function
every JSON operator
every system catalog
```

AI is excellent at retrieving syntax.

Your job is to know **what you want PostgreSQL to accomplish and whether the generated solution is correct.**

---

# Your "AI-era PostgreSQL brain"

If I were preparing you specifically for modern AI-assisted backend development, I'd aim for this:

| Area                      | Target          |
| ------------------------- | --------------- |
| SELECT / WHERE / ORDER BY | **Know cold**   |
| JOINs                     | **Know cold**   |
| GROUP BY / HAVING         | **Know cold**   |
| Aggregates                | **Know cold**   |
| NULL                      | **Know cold**   |
| Constraints               | **Know cold**   |
| Relationships             | **Know cold**   |
| Data modeling             | **Know cold**   |
| Transactions              | **Know cold**   |
| Index fundamentals        | **Know cold**   |
| EXPLAIN / EXPLAIN ANALYZE | **Know well**   |
| CTEs                      | **Know well**   |
| Subqueries                | **Know well**   |
| Window functions          | **Know well**   |
| PostgreSQL data types     | **Know well**   |
| JSONB                     | **Know well**   |
| MVCC                      | **Understand**  |
| Locks                     | **Understand**  |
| VACUUM                    | **Understand**  |
| WAL                       | **Understand**  |
| Query planner             | **Understand**  |
| Replication               | **Learn later** |
| Partitioning              | **Learn later** |
| Sharding                  | **Learn later** |

### And there's a particularly valuable skill I'd add:

**Be able to look at an AI-generated query and interrogate it.**

For example, AI gives you:

```sql
SELECT *
FROM customer c
JOIN rental r ON r.customer_id = c.customer_id
JOIN inventory i ON i.inventory_id = r.inventory_id
JOIN film f ON f.film_id = i.film_id
WHERE f.title = 'ACADEMY DINOSAUR';
```

Don't just think:

> "Looks right."

Think:

> Why these joins?

> What is the cardinality at each step?

> Could this duplicate rows?

> Which columns are indexed?

> What happens if the title isn't unique?

> Is `SELECT *` appropriate?

> What does `EXPLAIN ANALYZE` say?

> Would another query be cheaper?

> Is there a constraint guaranteeing the assumption this query makes?

**That is the skill I'd want you to develop.**

And `dvdrental` is actually excellent for this because you can use the same database to practice everything from basic joins → aggregation → window functions → indexing → query plans → transactions → data modeling without constantly switching datasets.




## Breaking changes from PostgreSQL 16 to PostgreSQL 18


MD5 Password Deprecation (v18):
MD5 password authentication is now officially deprecated in favor of SCRAM-SHA-256. While MD5 still functions, it will be entirely removed in the next major version release. You should migrate legacy user accounts to SCRAM before moving past v18.


Strict Maintenance search_path (v17):
Critical utility commands like VACUUM, ANALYZE, REINDEX, and CREATE INDEX now force a safe, empty search_path during execution. If you have custom functional indexes or triggers that rely on implicit schema resolution (i.e., calling objects without prefixing schema_name.), they will fail during maintenance routines unless rewritten with explicit schema references.🛑


SQL, Query & Data Changes
VACUUM and ANALYZE Inheritance Change (v18): Running VACUUM or ANALYZE on a parent table now automatically targets all partition and inheritance children by default. If your maintenance scripts targeted strictly parent shells to save resources, you must explicitly rewrite them using the new ONLY keyword (e.g., VACUUM ONLY parent_table;).

Disallowed Unlogged Partitioned Tables (v18): Creating partitioned tables as UNLOGGED is no longer supported.

CSV Parsing Alteration (v18):
The COPY FROM command will no longer interpret \. as a hard End-Of-File marker when parsing a standard CSV data stream, correcting a legacy bug but potentially breaking custom automated data pipelines that relied on that EOF formatting.

Interval Value Constraint (v17):
In time fields, the modifier ago is strictly
restricted to appearing at the very end of an interval string expression.🛠️ ]

Configuration & System View ChangesData Checksums Default (v18):
When initializing new database clusters using initdb, data checksums are now enabled by default. While highly recommended for corruption detection, it adds a slight performance overhead; you must pass --no-data-checksums if you want to bypass this behavior.

Removal of old_snapshot_threshold (v17): The old_snapshot_threshold server parameter has been entirely removed. If this configuration exists inside your legacy postgresql.conf, the database server will refuse to start up until it is deleted.

System View Column Re-naming (v17): Several key metrics in tracking tables were modified. For instance, buffers_backend and buffers_backend_fsync were stripped out of pg_stat_bgwriter. Additionally, the I/O block timing columns in pg_stat_statements were completely renamed. Custom monitoring configurations (such as Datadog or Prometheus metrics) will break until they are updated to match v17/v18 schema definitions.🧩

Ecosystem & ExtensionsExtension Deprecations (v17): The legacy server side adminpack extension has been removed. Also, check with your cloud provider; some managed platforms (like Supabase) have dropped support for older, complex extensions like plv8 or legacy versions of timescaledb on their v17+ tracks.

Proactive Upgrade Path

To safely transition your workflow, it is highly recommended to use pg_upgrade. Version 18 significantly optimized this path by keeping optimizer statistics intact during the transition, meaning you will not face the traditional immediate post-upgrade query slowdowns.If you'd like to dive deeper, let me know:Are you executing this upgrade on a managed cloud provider (like AWS RDS) or a self-hosted server?Do you use any third-party extensions (e.g., PostGIS, TimescaleDB)?Do you have automated scripts or monitoring dashboards hooked into system metrics?I can give you a targeted testing strategy for your specific environment.






## What You Must Know vs. What You Can Let AI Handle1. What You Must Know (Mental Framework & Concepts)

- The Query Planner (EXPLAIN & EXPLAIN ANALYZE): You must know how to read execution plans. When AI writes a slow query, you need to look at the plan to spot missing indexes, expensive sequential scans, or bad join choices.

- Indexing Strategies: You don’t need to memorize the exact syntax for creating a GIN or B-tree index, but you must know when to use them (e.g., GIN for JSONB or arrays, B-tree for standard lookups, partial or expression indexes).

- Concurrency and MVCC (Multi-Version Concurrency Control): Understand how PostgreSQL handles simultaneous reads and writes without locking tables. Knowing how isolation levels affect race conditions prevents subtle data corruption bugs that AI won't catch.

- The VACUUM Process: Understand bloat, autovacuum tuning, and why dead tuples need cleanup. If left unmanaged, performance degrades silently.PostgreSQL-Specific Data Types: Know that powerful native types like JSONB, arrays, UUID, and extensions like pgvector exist. You need to recognize when a problem is better solved with a native JSON array or a vector embedding rather than a rigid relational table.



## retention strategy

The best way to start trading your memory for web development knowledge is to use active recall and spaced repetition software (SRS). 
These techniques trick your brain into retaining complex syntax, concepts, and logic by forcing you to pull information from your memory just as you are about to forget it.Instead of passively re-reading tutorials, you actively "pay" with mental effort to build permanent knowledge.
🧠 The Core Strategy: Spaced Repetition (Anki)Download Anki, a free, open-source flashcard app used heavily by developers to memorize language syntax, API methods, and architectural patterns.

The Rules of the Trade:Never memorize what you don't understand: Code a small project or watch a tutorial first. Only make cards for things you have successfully implemented or understood.Keep cards atomic: One card should test one small piece of information. Do not put an entire script on a card.
Use Cloze Deletions: Use "fill-in-the-blank" cards to memorize syntax.
📋 How to Structure Your Coding Flashcards1. Language Syntax & Methods (JavaScript/Python)Front: How do you add an element to the end of an array in JavaScript?Back: array.push(element)

2. Concept & TheoryFront: What is the main difference between let and const in JavaScript?Back: let can be reassigned; const cannot be reassigned.3. Fill-in-the-Blank (Cloze Deletion)Front: The [...] hook in React is used to run side effects in functional components.Back: useEffect🛠️ Hands-on Code "Trading" SystemsIf you want to move beyond digital flashcards, use these practical methods to convert your working memory into muscle memory:The "Build it Twice" Method: Build a small app (like a calculator) following a tutorial. The next day, build the exact same app completely from memory. When you get stuck, look at your first project, memorize the fix, close it, and keep writing.The Flipped Tutorial: Watch a 10-minute coding tutorial without touching your keyboard. Take zero notes. When it ends, open your code editor and try to reconstruct the core logic from memory.Daily Code Katas: Use platforms like Codewars or LeetCode. Once you solve a problem, copy the core logic pattern into Anki so you recognize the algorithmic structure the next time you see it.I can help you build your first set of study materials if you tell me:What programming language or framework (HTML/CSS, JavaScript, React, Python, etc.) are you learning right now?What is your current experience level (absolute beginner, intermediate)?Are you studying for a specific goal (a job interview, building a personal project)?
