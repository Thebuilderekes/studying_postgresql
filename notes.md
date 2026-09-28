## definitions in postgresql


## NEED TO KNOW
- Postgresql stores all its data in a file system called `PGDATA`
- Only superusers can create tables in postgresql
- In postgresql databases are directories
- You can create  tables for testing but that table only exists within
that session and disappears after you close that session

## Managing user roles and connections
You can have multiple super users each with their own unique access?
`CREATE ROLE name[ [with] option [...]]`
- if you forget to add an option at the time of creation or you want to  change
an exist option you can use the `ALTER ROLE` statement.


## VALID UNTIL
VALID UNTIL allows you to create a role with access that expires at ta
specified time
`create role role_name with login password 'supersecret' in role group_name valid
until '2025-17-12'`


## ADMIN
After adding a role to a group_role you can also assign that role the ability
to add other roles to that group using the ADMIN statement
`Create ROLE group_role with nologin ADMIN role_name`

## select a particular role
`SET ROLE role_name`

## changing name of a role
`ALTER ROLE role_name  RENAME to new_role_name`

## using role for group
A group is a role that contains other roles
to add a role to a group you simply create the roile with the ``WITH  NOLOGIN``
statement and then when creating the role use the ``IN ROLE role_name``  to end
the statement

`CREATE ROLE group_name with nologin`
then
`Create ROLE role_name with login password 'supersecret' in ROLE group_role`

### Creating a "Group" of Superusers
To manage multiple administrators easily, you can create a dedicated group role and grant it creation permissions.Create the managing group role:sql

``````CREATE ROLE db_admins WITH CREATEROLE INHERIT;``````

## GRANT
You can also give a role to another role like so:
`GRANT group_role to role_name`

-- Add users to this group:
````sql
GRANT db_admins TO user_1;
GRANT db_admins TO user_2;

```````
If you want a group where every member inherits true SUPERUSER status directly without using SET ROLE, you must manually apply
`` ALTER ROLE username WITH SUPERUSER; `` to each individual user.

## How to Remove a User from a Group
To remove a user's membership from a group or role, use the REVOKE
command.

`REVOKE group_role_name FROM user_role_name;`

## Giving role managing access to roles and groups
-- to give role access to manage and create roles and manage groups
``ALTER ROLE your_role_name WITH CREATEROLE;``

## Assign superuser access to a role
`ALTER ROLE existing_role_name WITH SUPERUSER;`

## DROP ROLE
-- to drop a role
`DROP ROLE role_name`

-- tries to drop a role but  returns a message if a role does not exist
`DROP ROLE IF EXISTS role_name`

## Inspect existing roles
-- To get full details of all existing roles
``\du``

-- To see the role that you currently accessing a cluster from
`SELECT current_role;`

-- To check for the access information on a particular role
`SELECT rolname, rolcanlogin, rolconnlimit, rolpassword FROM pg_roles WHERE
rolname = 'role_name';`
*only cluster superusers can use pg_authid, every other user can use pg_roles*

-- for more information
`SELECT rolname, rolcanlogin, rolconnlimit, rolpassword FROM pg_authid WHERE
rolname = 'role_name';`
*only cluster superusers can use pg_authid*

# GRANT
You can grant select on only specific columns of a table to a role while
taking away permission to do any other kind of modification on all columns.
`GRANT SELECT (column_name1, column_name) ON table_name TO role_name`

- You can also grant elect to a group_role and share that permission to another
  role
  `SELECT ON group_role to role_name`

## find out
- Find out about RLS policy and replication
- session_user vs current_user
- what is client_min_messages
- what are ACLs in postgresql


## Working with DATABASE

## DROP database
`drop database database_name`

## copy database
`create database newdatabase template source_database`

## confirm database size
``\l+ database_name``



## Working with Tables

## The SELECT statement
- The select statement is used to search and return data based on conditions.
You do not need the FROM statement for the SELECT statement to work but using it
make the select statement flexible to work with


-- This will return the word in upper case like 'WORD'
`select upper('word)`

-- To select a column from a table in a database
``SELECT column_name FROM table_name``

-- to select specific columns from a table
 ``SELECT col1,col2 FROM table_name``

-- to return all the data from table_name
-- better not to use this when there are alot of data as it will slow down query
 ``SELECT * FROM table_name``

## SELECT DISTINCT statement
``SELECT DISTINCT col2 FROM table_name``
 - returns the unique entries based on the column


## COUNT function
- Get the size of a table, number of rows it contains
- Get summary about data
- Can be used with DISTINCT statement


``SELECT COUNT(*) FROM table_name``
- this counts the number of rows with both null and non-null values,

``COUNT(col_name)``
- counts only non-null rows ignoring rows with NULL


``SELECT COUNT(DISTINCT col_name) FROM table_name``
- counts the unique rows in the column name


## WHERE statement
It is used to get data based on a condition, and can be used with the following
statements:
``BETWEEN,
EXTRACT()
<, >,
<> or !=,
IN('row_value1', 'row_value2'),
LIKE '[letter]%',
AND,
OR
IS NULL
``
### Important
`WHERE` is computed before aggregate functions and `GROUP BY` statement
`HAVING` is computed after aggregate functions and `GROUP BY` statement

``SELECT * FROM table_name WHERE column_name KEYWORD``

### Types of tables
- Unlogged tables
- Temporary tables
- Logged tables

## CREATE
- Create a table with all the data fromanotehr datbel
`CREATE table new_table as SELECT * from another_table`

## INSERT
-- To get all the data in one table and insert into another table
`INSERT into new_table SELECT * from  other_table`

## UPDATE
-- To update a table's column value to a new value
`` UPDATE table_name set updated_property="value" where another_property='another_value';``

## DELETE
--  delete  all data in a table
`DELETE from table_name`

-- the best way to delete all data from a table
``TRUNCATE table table_name``



### Using transactions
- Find out the purpose of transactions, why you need them and wy you use them in
  postgresql

### NULL values
- Read up on everything that is important about NULL values and how
to handle them

-- To sort the NULL values to the last of the table values
`SELECT * FROM table_name order by column_name NULLS last` -  default is ASC
`SELECT * FROM table_name order by column_name NULLS first` - default is DESC

-- To see all values where the value of a column is null
``SELECT * from table_name where column_name is NULL;``


## DISTINCT


## ORDER BY

-- Order the table the second column
`SELECT  * FROM table_name ORDER BY 2 `

-- Order the table the third column
`SELECT  * FROM table_name ORDER BY 3 `

-- Order the table reverse-order
`SELECT  * FROM table_name ORDER BY DESC `

-- Order alphabetically the column_name1
`SELECT  * FROM table_name ORDER BY column_name1 `

`SELECT  * FROM table_name ORDER BY column_name1, column_name2 `

`SELECT column_name1, column_name2 FROM table_name
ORDER BY column_name1 DESC, column_name2`
- used of sorting
- used for data presentation based on what you want presented.
- Handles pagination

## LIMIT
Used to display a certain limit of data


## BETWEEN
- used to get date between a certain range off a condition
- can be `NOT BETWEEN` to exclude values that are not between the stated values
e.g
 ``SELECT column_name FROM table_name WHERE column_name BETWEEN value1 and
 value2``

 ``SELECT * FROM table_name WHERE column_name NOT BETWEEN value1 and
 value2``

 ## Subqueries
 ## IN
 - fetches column rows that are equal to the values stated within the ``IN`` parenthesis
 ``SELECT * FROM table_name WHERE column_name IN (value1,
 value2)``

## NOT IN

- Fetches column rows that do not equal the values in the parenthesis
``SELECT * FROM table_name WHERE column_name NOT IN (value1,
value2)``

## JOIN
This can be used to access rows across multiple tables and join them together  based on a condition
- It is good practice to pick the exact column in the `SELECT` statement used
with the join so you dont have situations where same name columns appear
multiple times after you use `JOIN`


## INNER JOIN
Inner join is what you are looking for when you what a column of one table to
continue adding another column from another table with a condition there is a
match on both tables. Don't show me any row that doesn't have a match.

``SELECT a.column_name1, a.column_name2, b.column_name1, b.column_name2 from
table table_name1 INNER JOIN table_name2 b on a.column_name1 = b.column_name2 ``


## LEFT JOIN
Show me everything from the left table, and whatever matches from the right
table even if it is NULL on the right table.

``SELECT a.column_name1, a.column_name2, b.column_name1, b.column_name2 from
table table_name1 lEFT JOIN table_name2 b on a.column_name1 = b.column_name2 ``

- You can put a place holder in place of a null value you can use the
``COALESCE(expected_null_column::TEXT, 'placeholder text')``after the `SELECT` for text placeholder or ``COALESCE(expected_null_column, 0)`` for number place holder

## RIGHT JOIN
Keep every row from the table on the right, even if there is no matching row in the table on the left.


## FULL OUTER JOIN
- If there is a match between the tables based on your join condition, the rows are combined.
- If there is a row in the left table but no match in the right table, it includes the left table's data and fills the right table's columns with NULL.
- If there is a row in the right table but no match in the left table, it includes the right table's data and fills the left table's columns with NULL

## EXISTS/NOT EXISTS



## AGGREGATE functions
- cannot be used with another column
common aggregate functions:
``
MIN()
MAX()
AVG()
SUM()
COUNT()
``
- You can use these with sub queries like
`SELECT column_name1, MIN(column_name2)  from table_name where column_name2 =
(SELECT MIN(column_name2) from table_name);`

## GROUP BY
used to group aggregated values in a query
`SELECT column, aggregate_function() from table_name GROUP BY column, `

## HAVING
- used to filter results in a row after grouping and aggregation
- follows GROUP BY
`SELECT column, aggregate_function() from table_name GROUP BY column HAVING
condition`

## LIKE
Looks for character or characters within a quote.
- It is case sensitive
- pattern matching within text columns

 ``SELECT * FROM table_name WHERE column_name LIKE pattern``

 ``SELECT * FROM table_name WHERE column_name LIKE 'char%'`` - search containing char
 ``SELECT * FROM table_name WHERE column_name LIKE 'word%'`` - search for word
 ``SELECT * FROM table_name WHERE column_name LIKE '%word%'`` - search if
 contains word
 ``SELECT * FROM table_name WHERE column_name LIKE '%w_rd%'`` - underscore for
 position character search
 ``SELECT * FROM table_name WHERE column_name LIKE '%colo[u]r'`` - for spelling
 variation where a character within the [] could be used or omitted

 ``SELECT * FROM table_name WHERE column_name LIKE '%example[_].[cno]m'``
 - matches any email address that ends with example or any TLD that starts with
'c','n' or 'o'

## ILIKE
- Its just LIKE but case insensitive

``LIKE and ILIKE`` are used with wild card characters

## EXTRACT
- used to extract specific part from value e.g year
`select `

## check foreign key relationship across all tables
``````
SELECT
    conrelid::regclass AS source_table,
    conname AS foreign_key_name,
    pg_get_constraintdef(oid) AS connection_details
FROM
    pg_constraint
WHERE
    contype = 'f'
    AND connamespace = 'public'::regnamespace
ORDER BY
    source_table;
    ``````


## Views
Views are a way to store a query into a table like structure so you can easily
reuse it again without having to retype the query:

`CREATE VIEW myview as
select colum_name1 from table_name p join table_name2 t on t.column_name1 =
p.column_name1`

This would store the select query part into ``myview``

##* **Use views with caution**
Although views are considered useful to avoid repetition and for reuse of
queries they can make it easy to hide complex queries that contain aggregate
functions that can slow down execution so they should be used with action

 ## Foreign keys
 These are column names used by child tables to reference parent tables. They are usually named the same with the parent column name that they are referencing



## Transactions, MVCC
- **what is a transaction?**
A transaction is a group of queries put together in such a way that they must
all run or none them runs.
A transaction is done by surrounding the group of queries with `BEGIN` and
`COMMIT` commands
- Postgresql actually treats all SQL statements as being part of a transaction
  so if do not state it explicitly, it is implicitly declared.
 - Transactions have an id that can be gotten with `txid_current()`
 every statement has their own unique ``xid``
 -  There's a hidden column in every table called `xmin` that you can use to see the common ``xid`` that every row of data has to indicate that they were created within the same transaction.

## Relevant Study topics
- savepoint using  `SAVEPOINT`
Allows you to rollback to a state you create in your transaction
After declaring it like so: ``SAVEPOINT my_savepoint``
You use the `ROLLBACK TO my_savepoint` to get to your savepoint

## window functions
A window function lets you perform a calculation across a group of related rows without collapsing those rows into one row.
-- Ranking
``
ROW_NUMBER()
RANK()
DENSE_RANK()
``
-- aggregates
``
SUM() OVER (...)
AVG() OVER (...)
COUNT() OVER (...)
MIN() OVER (...)
MAX() OVER (...)
``
-- looking at neigbouring rows
``
LAG()
LEAD()
``

## INHERTANCE
Using `INHERITS` statement one can create a table and have that table inherit
from another.


# what is MVCC, WAL, checkpoint?

These four concepts are tightly connected. The easiest way to understand them is to imagine **multiple people using the same PostgreSQL database at the same time**, while the machine could also crash at any moment.


They solve different problems:

```text
Transaction → "These operations belong together."
MVCC        → "Users shouldn't interfere with each other's reads/writes."
WAL         → "Record changes so committed data can survive a crash."
Checkpoint  → "Periodically make the actual data files catch up with the WAL."
```

Let's build this from the ground up.

---

# 1. Transactions — "all of this work belongs together"

Imagine a bank transfer:

```text
Alice:   $1,000
Bob:       $500
```

Alice sends Bob $200.

The database needs to perform **two operations**:

```text
1. Remove $200 from Alice
2. Add $200 to Bob
```

You don't want this:

```text
Alice → $800
Bob   → $500
```

because Alice lost the money but Bob didn't receive it.

And you don't want:

```text
Alice → $1,000
Bob   → $700
```

because Bob received money that didn't come from Alice.

A **transaction** groups the operations together:

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 200
WHERE id = 1;

UPDATE accounts
SET balance = balance + 200
WHERE id = 2;

COMMIT;
```

PostgreSQL treats that as **one unit of work**.

If something goes wrong:

```sql
ROLLBACK;
```

PostgreSQL undoes the transaction.

### The problem transactions solve

> **How do I make several database operations behave as one indivisible unit?**

This is the **Atomicity** part of ACID.

---

# 2. MVCC — Multiple Version Concurrency Control

This is probably the most initially confusing concept, but it's one of the most important PostgreSQL concepts to understand.

Imagine you have:

```text
products

id | name       | stock
---+------------+------
1  | Keyboard   | 10
```

User A starts updating the stock:

```sql
BEGIN;

UPDATE products
SET stock = 9
WHERE id = 1;
```

At approximately the same time, User B executes:

```sql
SELECT stock
FROM products
WHERE id = 1;
```

What should B see?

Should B:

* see `10`?
* see `9`?
* wait for A?
* get some half-written state?

PostgreSQL uses **MVCC** to deal with this.

### The basic idea

MVCC = **Multi-Version Concurrency Control**.

Rather than thinking:

> "There is one row, and everyone fights over that row."

think:

> "PostgreSQL maintains versions of rows so transactions can work with a consistent view of the database."

Conceptually:

```text
Before update:

Row version A
stock = 10

        ↓ UPDATE

Row version B
stock = 9
```

PostgreSQL keeps information about these row versions internally.

This allows readers and writers to interact much more efficiently than a system where every read has to wait for every write.

---

# 3. What problem does MVCC solve?

**Concurrency.**

Imagine 10,000 users accessing your application simultaneously.

You don't want:

```text
User A reading
      ↓
Everyone else waits
      ↓
User A finishes
      ↓
User B reads
      ↓
Everyone else waits
```

Instead, PostgreSQL can allow transactions to work concurrently while maintaining a consistent view of the data.

For example:

```text
              PostgreSQL
                  │
       ┌──────────┼──────────┐
       ↓          ↓          ↓
   User A      User B      User C
    UPDATE      SELECT      SELECT
       │          │          │
       └────── MVCC ─────────┘
```

This is a huge reason PostgreSQL can support many concurrent users.

### But there's a catch

Old row versions eventually need to be cleaned up.

That's where **VACUUM** comes into the picture.

For example:

```sql
UPDATE products
SET stock = 9;
```

doesn't simply mean:

```text
destroy old row
create new row
```

PostgreSQL's MVCC machinery creates a new row version and eventually needs to reclaim versions that are no longer visible to any transaction.

This is one reason **VACUUM and autovacuum are so important in PostgreSQL**.

---

# 4. WAL — Write-Ahead Log

Now imagine this:

You run:

```sql
UPDATE accounts
SET balance = 500
WHERE id = 1;
```

PostgreSQL needs to make that change durable.

But writing directly to the database's data files every time something changes would be expensive.

Instead, PostgreSQL uses a **Write-Ahead Log (WAL)**.

The basic principle is:

> **Record the change in the WAL before considering the change safely committed.**

Conceptually:

```text
SQL
 │
 ▼
Transaction
 │
 ▼
WAL
 │
 ▼
"Account 1 changed from X → 500"
 │
 ▼
COMMIT
```

The WAL is essentially a durable record of changes that PostgreSQL can use to recover the database.

---

# 5. Why do we need WAL?

Imagine your server has:

```text
RAM
 ↓
PostgreSQL
 ↓
Disk
```

Your application executes:

```sql
UPDATE accounts
SET balance = 500;
```

The change might initially exist in PostgreSQL's memory/buffer system before the corresponding data page is written to its final location on disk.

Then:

💥 **Power failure.**

What happens?

Without some sort of recovery mechanism, PostgreSQL could potentially have:

```text
Transaction said "COMMIT"
             ↓
Machine crashed
             ↓
Data files weren't completely updated
             ↓
😬
```

WAL gives PostgreSQL a recovery trail.

After restarting, PostgreSQL can essentially say:

> "The data files weren't completely up to date, but I have the WAL describing the changes that had been committed. I can replay the necessary changes."

This is called **crash recovery**.

---

# 6. Why is it called "Write-Ahead"?

Because the log is written **ahead of the corresponding data-file modification**.

Conceptually:

```text
        WAL
         │
         ▼
   "Change X → Y"
         │
         ▼
      COMMIT
         │
         ▼
Data pages eventually written
```

The important rule is:

> **The WAL describing a change must be safely recorded before PostgreSQL considers the change durable.**

This is fundamental to PostgreSQL's durability guarantees.

---

# 7. Checkpoints

Now we've got another problem.

Imagine PostgreSQL has been running for days.

The WAL could contain:

```text
WAL
│
├── change 1
├── change 2
├── change 3
├── change 4
├── change 5
├── ...
├── change 10,000,000
└── change 10,000,001
```

If the server crashes, PostgreSQL could potentially have to replay a huge amount of WAL.

That's where **checkpoints** come in.

A checkpoint is essentially PostgreSQL saying:

> **"Let's make sure the changes up to this point have been flushed from memory to the actual data files."**

Conceptually:

```text
WAL
───────────────────────────────────────
       │
       │
       ▼
   CHECKPOINT
       │
       ▼
Data files catch up
```

After a checkpoint, PostgreSQL has a known point from which recovery can continue.

---

# 8. Why do we need checkpoints?

Primarily to make **crash recovery more manageable**.

Imagine:

```text
Server crashes
      ↓
Find latest checkpoint
      ↓
Read WAL after checkpoint
      ↓
Replay necessary changes
      ↓
Database recovered
```

Without checkpoints, PostgreSQL could potentially need to process an enormous amount of WAL during recovery.

So checkpoints provide a kind of **recovery boundary**.

---

# 9. How all four work together

This is the part I'd memorize.

Suppose you run:

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

### Transaction

Groups these operations:

```text
Withdraw $100
+
Deposit $100
=
ONE transaction
```

If something fails:

```text
ROLLBACK
```

---

### MVCC

Allows PostgreSQL to handle this transaction **alongside other transactions** while maintaining appropriate consistent views of the rows.

```text
Transaction A ──┐
                ├── PostgreSQL MVCC
Transaction B ──┤
                │
Transaction C ──┘
```

---

### WAL

PostgreSQL records the necessary changes in the write-ahead log so committed changes can be recovered after a crash.

```text
Transaction
     ↓
WAL
     ↓
COMMIT
```

---

### Checkpoint

Periodically PostgreSQL makes sure modified data pages have been flushed to the database files.

```text
WAL
 │
 │ changes
 │
 ▼
CHECKPOINT
 │
 ▼
data files
```

If the machine crashes:

```text
             CRASH
               ↓
        Find checkpoint
               ↓
      Replay relevant WAL
               ↓
        Database recovers
```

---

# 10. The simplest analogy

Imagine you're writing a very important paper.

### Transaction

You say:

> "I'm going to make these five edits as one operation. Either all five count or none count."

### MVCC

Several people are editing/reviewing the document simultaneously, but they shouldn't see inconsistent intermediate states.

### WAL

Before modifying the final document, you keep a **journal of every change**.

```text
10:01 — changed paragraph 1
10:02 — changed paragraph 3
10:03 — deleted paragraph 5
```

If your computer crashes, you can reconstruct what happened from the journal.

### Checkpoint

Every so often you save the actual document completely:

```text
SAVE COMPLETE
```

Now, if the computer crashes, you don't have to replay the entire journal from the beginning. You start from the latest complete save and apply the changes afterward.

---

# 11. How this connects to what you're reading

This is why I'd **not skip Chapter 11** of Ferrari's book.

You don't necessarily need to memorize the internals immediately. Aim to understand this chain:

```text
             CONCURRENT USERS
                    │
                    ▼
                  MVCC
                    │
                    ▼
              TRANSACTIONS
                    │
                    ▼
                   WAL
                    │
                    ▼
              CHECKPOINTS
                    │
                    ▼
             CRASH RECOVERY
```

And then add:

```text
MVCC
 │
 └── old row versions
          │
          ▼
       VACUUM
          │
          ▼
   reclaim dead tuples
```

Once that picture makes sense, PostgreSQL stops looking like a black box. You start understanding **why** things such as transactions, `VACUUM`, WAL, indexes, and checkpoints exist rather than simply memorizing that they exist.
