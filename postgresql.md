PostgreSQL
++++++++++

Docs at https://www.postgresql.org/docs/current/index.html

Easy test environment
=====================

Fast Docker setup
-----------------

Tested with postgres 12 and 16. Modify as needed.

```sh
# Download and run the image as `pg_test`
#   - Sets auth to be trust (no password needed)
#   - Mounts current directory as /host
#   - Add --rm to remove the container on exit
docker run -e POSTGRES_HOST_AUTH_METHOD=trust -v ${PWD}:/host -p5432:5432 --name pg_test postgres:latest

# Manage the container as usual
docker start pg_test -a
docker stop pg_test

# Create a database
docker exec -it pg_test psql postgres://postgres@localhost:5432/postgres -c 'create database pg_test'

# Optionally load some data
docker exec -it pg_test psql postgres://postgres@localhost:5432/pg_test -f /host/test_schema_and_data.sql

# Connect using your local psql
psql postgres://postgres@localhost:5432/pg_test

# Connect using psql from Postgres' Docker container
docker exec -it pg_test psql postgres://postgres@localhost:5432/pg_test
```

Test data
---------

* [My data analyst SQL test data](https://github.com/nandilugio/data_analysts_sql_test/blob/master/test_schema_and_data.sql)
* [PostgreSQL Wiki sample databases](https://wiki.postgresql.org/wiki/Sample_Databases)
* postgresqltutorial.com [tutorial database](https://www.postgresqltutorial.com/postgresql-sample-database/)
* PostgreSQL [tutorial](https://www.postgresql.org/docs/current/tutorial.html) has a [sample database](https://www.postgresql.org/docs/current/tutorial-sql-intro.html) (in [the repo](https://github.com/postgres/postgres/tree/master/src/tutorial)).
* PostgreSQL [regression tests setup schema](https://www.postgresql.org/docs/current/regress.html) sometimes used in the docs.

For the last, [test_setup.sql](https://github.com/postgres/postgres/blob/master/src/test/regress/sql/test_setup.sql) requires the [data files](https://github.com/postgres/postgres/blob/master/src/test/regress/data/) to be downloaded. For help on how, see this [SO answer](https://stackoverflow.com/questions/7106012/download-a-single-folder-or-directory-from-a-github-repo). Then, assuming you have the above Docker setup:

```sh
# Load the schema. Set PG_ABS_SRCDIR to the abs path within the container where the `data/` dir is available.
wget -qO- https://raw.githubusercontent.com/postgres/postgres/master/src/test/regress/sql/test_setup.sql | docker exec -e PG_ABS_SRCDIR=/host -i pg_test psql postgres://postgres@localhost:5432/pg_test

# Run any other sql files you need straight from the PostgreSQL repo piping wget output to psql
wget -qO- https://raw.githubusercontent.com/postgres/postgres/master/src/test/regress/sql/create_index.sql | docker exec -i pg_test psql postgres://postgres@localhost:5432/pg_test        
```

`psql` client
=============

See [psql docs](https://www.postgresql.org/docs/current/app-psql.html).

Useful options and usage
------------------------

```sh
# Connect to a database
psql postgres://user:password@host:port/database # postgres URL
psql -h host -p port -U user -W database

psql pgurl -c 'SELECT * FROM users'
psql pgurl -f /path/to/file
psql pgurl < /path/to/file
```

Basic commands
--------------

```sql
\?              -- psql help
\h              -- SQL help
\q              -- quit

\l              -- list databases
\c database     -- connect to database

\dt             -- list tables, views and sequences
\d table        -- describe table, view, sequence or index

\df             -- list functions
\d function     -- describe function

\e              -- edit query in editor
\i file         -- execute query from file

\copy (query) from 'file.csv' DELIMITER ',' CSV HEADER  -- import CSV data (file on client)
\watch [SEC]                                            -- run last query every SEC seconds (def. 2)
```

Data types
==========

See [all data types](https://www.postgresql.org/docs/current/datatype.html).

```sql
BOOLEAN     -- true/false

SMALLINT/INTEGER/BIGINT -- 16/32/64-bit signed integer
DECIMAL|NUMERIC         -- arbitrary precision decimal
REAL/DOUBLE|FLOAT       -- 32/64-bit floating point number
SERIAL                  -- Integer. Not only a type but also a sequence (like MySQL's autoincrement)

CHAR(n)                 -- fixed-length string
TEXT|VARCHAR(_)         -- variable-length string

DATE/TIME               -- date/time
TIMESTAMP               -- date and time
TIMESTAMPTZ             -- date and time with timezone

ARRAY                   -- array of values (kind of deprecated in favor of JSONB right?)
ENUM                    -- enumerated (enum) type
JSON                    -- JSON data as string (kind of deprecated too right?)
JSONB                   -- JSON data with binary representation
UUID                    -- universally unique identifier

-- Row types (composite types)
CREATE TYPE type_name AS (col1 type, col2 type, ...);
[ROW](value, [ , ... ]) -- row constructor

-- Domain types (custom types)
CREATE DOMAIN type_name AS type [DEFAULT value] [NOT NULL] [CHECK (condition)];
```

Sequences
---------

```sql
-- Create sequence
-- See https://www.postgresql.org/docs/current/sql-createsequence.html
CREATE SEQUENCE sequence_name
    [AS { SMALLINT | INTEGER | BIGINT }]
    [INCREMENT/MINVALUE/MAXVALUE/START]
    [CACHE/CYCLE]
    [OWNED BY { table_name.column_name | NONE }];

-- Drop sequence
-- See https://www.postgresql.org/docs/current/sql-dropsequence.html
DROP SEQUENCE sequence_name;

-- Use sequence
-- See https://www.postgresql.org/docs/current/functions-sequence.html
sequence_name.NEXTVAL
sequence_name.CURRVAL

-- Set sequence value
-- See https://www.postgresql.org/docs/current/functions-sequence.html
SELECT SETVAL(sequence_name, value, is_called); -- is_called = true to increment sequence? TODO
SELECT SETVAL(sequence_name, 1, false); -- Reset sequence to 1

-- Use sequence in table
-- See https://www.postgresql.org/docs/current/sql-createtable.html
CREATE TABLE table_name (
    id SERIAL PRIMARY KEY, -- table_name_id_seq INTEGER
    sequence_name  default nextval('my_sequence_name')
);
```

Indexes
-------

Read [all about indexes](https://www.postgresql.org/docs/current/indexes.html).

* __B-tree:__ Defalut. Searches and updates are O(log(n)). Stores order: good for equality and range queries, ORDER BY, UNIQUE constraints, etc.
* __Hash:__ Searches and updates are O(1). Smaller than b-trees since values are not stored. Doesn't store order so only good for equality queries. Small values (say <25 chars) doesn't benefit from hash indexes => prefer b-trees.
* __GIN:__ Generalized Inverted Index. Good for searches within the value (full-text search, JSON props, etc.).

```sql
-- Create index
-- See https://www.postgresql.org/docs/current/sql-createindex.html
CREATE [UNIQUE] -- Enforces uniqueness
    INDEX [CONCURRENTLY] -- Create index without locking table. Will take longer to be usable. Quirky for unique indexes.
    index_name
    ON table_name (col1, col2, ...) -- or expressions like `lower(col1)`, etc.
    USING [BTREE|HASH|GIN|...];

-- Maintenance
REINDEX TABLE table_name; -- Rebuild all indexes on table. Can shrink indexes.

-- Drop index
-- See https://www.postgresql.org/docs/current/sql-dropindex.html
DROP INDEX index_name;
```

TODO: GIN, tsvector and text search

JSON
----

See [JSON types](https://www.postgresql.org/docs/current/datatype-json.html) in the docs.
[This article](https://scalegrid.io/blog/using-jsonb-in-postgresql-how-to-effectively-store-index-json-data-in-postgresql/) is also interesting.

```sql
CREATE TABLE users ( ... config JSONB DEFAULT '{}'::JSONB, ... );

-- Querying
config->'lang' = 'ES'; -- -> returns JSON(B)
config->>'lang' = 'ES'; -- ->> returns TEXT

-- Querying with indexes
-- See https://www.postgresql.org/docs/current/datatype-json.html#JSON-INDEXING
CREATE INDEX users_config_gin ON users USING GIN (config);
config ? 'ui'; -- test key existence
config @> '{"ui": { "theme": "dark" }}'::jsonb; -- test leaf inclusion
-- See all functions and operators: https://www.postgresql.org/docs/current/functions-json.html

-- Also useful: format results as JSON
SELECT
    json_build_object(
        'id', users.id,
        'follower_ids', coallesce(json_agg(followers.id), '[]'::json)
    )
FROM users JOIN users AS followers ...;
```

Tables
======

Creation
--------

```sql
-- Create table
-- See https://www.postgresql.org/docs/current/sql-createtable.html
CREATE TABLE table_name (
    -- column_name data_type [column_constraints] [defau],
    id SERIAL PRIMARY KEY,
    other_id INTEGER REFERENCES other_table (other_id) ON DELETE CASCADE,
    created_at TIMESTAMP DEFAULT NOW(),
    first_name TEXT NOT NULL,
    last_name TEXT NOT NULL,
    full_name TEXT GENERATED ALWAYS AS (first_name || ' ' || last_name) STORED,
    age INTEGER CHECK (age > 0),
    -- ...
);

-- Create table from another table
CREATE TABLE table_name AS SELECT ... FROM other_table WHERE ...;

-- Create table from CSV data (file on server)
CREATE TABLE table_name (col, ...) FROM 'path/to/file.csv' DELIMITER ',' CSV HEADER;

-- Constraints
-- See https://www.postgresql.org/docs/current/ddl-constraints.html
PRIMARY KEY (col, ...) -- unique and not null
FOREIGN KEY (col, ...) REFERENCES other_table (col, ...) ON DELETE CASCADE
UNIQUE (col, ...)
NOT NULL (col, ...)
CHECK (condition)
```

Deletion
--------

```sql
-- Drop table
-- See https://www.postgresql.org/docs/current/sql-droptable.html
DROP TABLE table_name;
```

Populating data
===============

```sql
-- Insert rows
-- See https://www.postgresql.org/docs/current/sql-insert.html
INSERT INTO table_name (col1, col2, ...) VALUES -- column names are optional
    (value1, value2, ...),
    (value1, value2, ...),
    ...;

-- Insert rows from another table
INSERT INTO table_name (col1, col2, ...)
    SELECT col1, col2, ... FROM other_table WHERE ...;

-- Import CSV data (file on server)
COPY table_name (col1, col2, ...) FROM 'path/to/file.csv' DELIMITER ',' CSV HEADER;
```

Querying data
=============

```sql
-- Select rows
-- See https://www.postgresql.org/docs/current/sql-select.html
SELECT [DISTINCT] *, or, columns, or, expressions ...
    FROM table_name
        [LEFT|RIGHT|OUTER] JOIN other_table ON table_name.id = other_table.id
    WHERE condition -- on pre-grouped rows
    GROUP BY columns, ...
    HAVING condition -- on group rows
    ORDER BY columns, ...
    LIMIT n OFFSET m;

-- Subqueries
SELECT * FROM table_name WHERE id = (SELECT max(id) FROM other_table WHERE ...); -- single value
SELECT * FROM table_name WHERE id IN (SELECT id FROM other_table WHERE ...); -- multiple values

-- Common table expressions (CTE)
-- See https://www.postgresql.org/docs/current/queries-with.html
WITH cte_1 AS (
    SELECT ...
), cte_2 AS (
    SELECT ...
)
SELECT * FROM cte_1 JOIN cte_2 ON ...; -- Or INSERT, UPDATE, DELETE, ...
```

Updating data
=============

```sql
-- Update rows
-- See https://www.postgresql.org/docs/current/sql-update.html
UPDATE table_name SET col1 = value1, col2 = value2, ... WHERE ...;

-- Update rows from another table
UPDATE table_name SET col1 = other_table.col1, col2 = other_table.col2, ...
    FROM other_table -- like USING on DELETEs: full separate table expression joined in the WHERE clause
        [JOIN third_table ON other_table.id = third_table.id]
    WHERE table_name.id = other_table.id;

-- Update rows from a subquery
UPDATE table_name SET col1 = subquery.col1, col2 = subquery.col2, ...
    FROM (SELECT id, col1, col2, ... FROM other_table WHERE ...) AS subquery
    WHERE table_name.id = subquery.id;
```

Deleting data
=============

```sql
-- Delete rows
-- See https://www.postgresql.org/docs/current/sql-delete.html
DELETE FROM table_name WHERE ...;

-- Delete rows from another table
DELETE FROM table_name
    USING other_table -- like FROM on UPDATEs: full separate table expression joined in the WHERE clause
        [JOIN third_table ON other_table.id = third_table.id]
    WHERE table_name.id = other_table.id;

-- Delete rows from a subquery
DELETE FROM table_name
    USING (SELECT id FROM other_table WHERE ...) AS subquery
    WHERE table_name.id = subquery.id;

-- Delete all rows
-- See https://www.postgresql.org/docs/current/sql-truncate.html
TRUNCATE TABLE table_name;
```

Views
=====

```sql
-- Create view
-- See https://www.postgresql.org/docs/current/sql-createview.html
CREATE [MATERIALIZED] VIEW view_name AS
    SELECT ...;

-- Refresh materialized view
-- See https://www.postgresql.org/docs/current/sql-refreshmaterializedview.html
REFRESH MATERIALIZED VIEW view_name;

-- Drop view
-- See https://www.postgresql.org/docs/current/sql-dropview.html
DROP VIEW view_name;
```

It is possible to INSERT, UPDATE and DELETE rows in a view if it is defined on a single table and has all the required columns.

Useful functions
================

See [all functions](https://www.postgresql.org/docs/current/functions.html).

Scalar functions and expressions
--------------------------------

```sql
-- Operators
exp = exp  -- Also <> or !=, <, <=, >, >=
exp IS [NOT] NULL
exp [NOT] BETWEEN exp AND exp
exp [NOT] IN (exp, exp, ...)
exp AND/OR exp
NOT exp

[NOT] EXISTS (subquery)
[NOT] IN (row|array|subquery)
ANY/ALL (row|array|subquery) = exp -- = or any operator. ANY = SOME

-- Conditional expressions
CASE WHEN condition THEN value
     WHEN condition THEN value
     ELSE value END
COALESCE(value, value, ...)         -- first non-null value
NULLIF(value, value)                -- null if values are equal, else first value
GREATEST/LEAST(value, value, ...)   -- greatest/least value

-- Type expressions
value::type                     -- cast to type
CAST(value AS type)             -- cast value to type
TO_CHAR(value, format)          -- cast to string using format
TO_NUMBER(value, format)        -- cast to number using format
ISNULL(value [ , default ] )    -- true if value is null

-- Pattern matching
exp [NOT] LIKE 'pattern'        -- Wildcards: '%' any sequence of characters, '_' any single character
exp [NOT] ILIKE 'pattern'       -- case insensitive LIKE
exp [NOT] SIMILAR TO 'regex'    -- regex with POSIX syntax plus '%' and '_' wildcards
exp ~ 'regex'                   -- POSIX regex
exp ~* 'regex'                  -- case insensitive POSIX regex
exp !~ 'regex'                  -- POSIX regex does not match
exp !~* 'regex'                 -- case insensitive POSIX regex does not match

-- Numeric
RANDOM()                -- random number between 0 and 1
FLOOR/ROUND/CEIL(n)     -- round to integer
GREATEST/LEAST(n, m)    -- greatest of n and m
SQRT(n)                 -- square root
EXP(n)                  -- e to the power of n

-- String
s1 || s2                    -- concatenate strings
CONCAT(s1, s2, ...)         -- concatenate strings
LENGTH(s)                   -- length of string
TRIM(s)                     -- remove leading and trailing whitespace
SPLIT_PART(s, delim, n)     -- split string on delimiter and return nth part
LEFT/RIGHT(s, n)            -- leftmost/rightmost n characters
LOWER/UPPER(s)              -- lowercase
INITCAP(s)                  -- capitalize first letter of each word
SUBSTRING(s, start, length) -- substring
REPLACE(s, from, to)        -- replace substring
REGEXP_REPLACE(s, pattern, replacement) -- replace substring using regex

-- Date and time
NOW()
CURRENT_TIMESTAMP
LOCALTIMESTAMP(precision)
-- See https://www.postgresql.org/docs/current/functions-datetime.html#FUNCTIONS-DATETIME-CURRENT
DATE_TRUNC('day', d)    -- truncate date to day
EXTRACT('day' FROM d)   -- extract day from date
DATE_PART('day', d)     -- extract day from date
AGE(d1, d2)             -- difference between dates
```

Set returning functions
-----------------------

Used in FROM clauses like tables.

```sql
-- Generate series
SELECT * FROM generate_series(1, 10, 2); -- 1, 3, 5, 7, 9

-- Generate random numbers
SELECT * FROM generate_series(1, 10) AS i, random() -- AS r; TODO
```

Aggregate functions
-------------------

Used with GROUP BY.

```sql
COUNT([DISTINCT] column)    -- count distinct non-null values in column
SUM/AVG/MIN/MAX(exp)        -- column values simle descriptive stats
STRING_AGG(exp, delim)      -- concatenate values in column using delimiter
ARRAY_AGG(exp)              -- aggregate values in column into array
JSONB_AGG(exp)              -- aggregate values in column into JSON array

-- FILTER clauses
SELECT
    COUNT(*) as total_users,
    COUNT(*) FILTER (WHERE age < 21) as underage_users,
    COUNT(*) FILTER (WHERE last_login > NOW() - INTERVAL '1 day') as active_users_24h
FROM users;
```

Window functions
----------------

Groups rows for calculations but results are not aggregated into a single row. All rows show a result that can be a full window aggregate or a different value calculated depending on eg. its position on the window, etc.

They're executed after the WHERE, GROUP BY and HAVING clauses, so they're only allowed in SELECT and ORDER BY.

```sql
-- Partitions: rows are grouped by the expression
WINDOW_FN(exp) OVER ([ partition ] [ order ] [ frame ])
    -- Where:
    partition := PARTITION BY exp1 [ , ... ]            -- one window per distinct value of the exp(s)
    order := ORDER BY exp1 [ ASC | DESC ] [ , ... ]     -- rows are ordered within each partition.
                                                        -- /!\ WARNING! ordering implies the frame is
                                                        --   ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    frame := ROWS BETWEEN pre AND post                  -- pre and post can be CURRENT ROW or UNBOUNDED/n PRECEDING/FOLLOWING

COUNT/SUM/AVG/...(exp) OVER ()                                  -- regular aggregates work as window functions

ROW_NUMBER/RANK/DENSE_RANK() OVER (ORDER BY exp)                -- ranks in ordered window
LAG/LEAD(exp, n) OVER (ORDER BY exp)                            -- values of surrounding rows in ordered window
FIRST_VALUE/LAST_VALUE/NTH_VALUE(exp, n) OVER (ORDER BY exp)    -- values of rows in ordered window
NTILE(n) OVER (ORDER BY exp)                                    -- row number divided into n buckets
PERCENT_RANK() OVER (ORDER BY exp)                              -- percentile rank of row
CUME_DIST() OVER (ORDER BY exp)                                 -- cumulative distribution of row
```

User-defined functions
======================

```sql
-- Create function
CREATE FUNCTION function_name (arg1 type, arg2 type, ...) RETURNS type AS $$
    -- statements
$$ LANGUAGE sql;

-- Drop function
DROP FUNCTION function_name (type, type, ...);

-- Call function
SELECT function_name (arg1, arg2, ...);
```

Procedural Language (PL/*) functions
------------------------------------

| Language   | Availability | Trusted |
|------------|--------------|---------|
| sql        | core         | yes     |
| plpgsql    | core         | yes     |
| plpython3u | core         | no      |
| plsh       | core         | no      |
| plv8       | 3rd party    | yes     |

See [all languages](https://wiki.postgresql.org/wiki/PL_Matrix).

Trusted languages can't expose data unintendedly (can't access the filesystem, etc.).

PL/PgSQL
--------

```sql
$$
BEGIN
    TODO -- statements
END;
$$ LANGUAGE plpgsql;

```

Triggers
========

```sql
-- Create trigger
-- See https://www.postgresql.org/docs/current/sql-createtrigger.html
CREATE TRIGGER trigger_name
    [BEFORE|AFTER|INSTEAD OF] [INSERT|UPDATE|DELETE|TRUNCATE] -- INSTEAD OF for views
    ON table_name
    REFERENCING OLD|NEW TABLE AS name
    [FOR EACH ROW|STATEMENT]
    [WHEN (condition)] -- on ROW triggers, condition can reference OLD.col or NEW.col
    EXECUTE FUNCTION function_name('some arg');
    -- Params can be passed to the function, but it receives them in TG_ARGV. It can also access
    -- the OLD and NEW records, and other data through TG_* variables.

-- Drop trigger
-- See https://www.postgresql.org/docs/current/sql-droptrigger.html
DROP TRIGGER trigger_name ON table_name;

-- Example: `updated_at` column
CREATE FUNCTION trigger_set_timestamp()
    RETURNS trigger -- trigger functions must return a row (OLD, NEW, NULL or a custom one)
    LANGUAGE plpgsql AS
$$
BEGIN
  NEW.updated_at = NOW();
  RETURN NEW;
END;
$$;
CREATE TRIGGER set_timestamp BEFORE UPDATE ON users
    FOR EACH ROW EXECUTE FUNCTION trigger_set_timestamp();
```

Concurrency Control
===================

Transactions
------------

```sql
BEGIN;
COMMIT;
ROLLBACK;

SAVEPOINT savepoint_name;
ROLLBACK TO savepoint_name;
RELEASE savepoint_name;
```

Explicit Locking
----------------

```sql
-- Lock row
SELECT * FROM table_name WHERE id = 1 FOR [UPDATE|SHARE] [NOWAIT]; -- NOWAIT to fail if row is locked

-- Lock table
LOCK TABLE table_name IN [ACCESS SHARE|ROW SHARE|ROW EXCLUSIVE|SHARE UPDATE EXCLUSIVE|SHARE|SHARE ROW EXCLUSIVE|EXCLUSIVE|ACCESS EXCLUSIVE] MODE;
```

Pessimistic Concurrency Control
-------------------------------

Locks data to prevent other transactions from modifying it. Can cause deadlocks. Locks are released when the transaction ends.

```sql
-- Lock row
SELECT * FROM table_name WHERE id = 1 FOR [UPDATE|SHARE] [NOWAIT]; -- NOWAIT to fail if row is locked

-- Lock table
LOCK TABLE table_name IN [ACCESS SHARE|ROW SHARE|ROW EXCLUSIVE|SHARE UPDATE EXCLUSIVE|SHARE|SHARE ROW EXCLUSIVE|EXCLUSIVE|ACCESS EXCLUSIVE] MODE;
```

Optimistic Concurrency Control
------------------------------

Allows other transactions to modify data, but checks for conflicts before committing. Can cause serialization failures. Locks are released when the transaction ends. Requires a version column. See [this article](https://www.citusdata.com/blog/2018/02/15/optimistic-locking-in-postgresql/) for more info.

```sql
-- Add a version column
ALTER TABLE table_name ADD COLUMN version INTEGER NOT NULL DEFAULT 0;

-- Update row
UPDATE table_name SET version = version + 1 WHERE id = 1 AND version = 0;

-- Check if update was successful
SELECT * FROM table_name WHERE id = 1 AND version = 1;
```

Transactions and Concurrency Control
------------------------------------
    
```sql
-- Transaction isolation levels
-- See https://www.postgresql.org/docs/current/transaction-iso.html
SET TRANSACTION ISOLATION LEVEL READ COMMITTED; -- Default
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

The rule system
===============

TODO

System Info and Administration
==============================

Troubleshooting
---------------

TODO: check!

```sql
-- Check for deadlocks
SELECT * FROM pg_locks WHERE NOT GRANTED;

-- Check for serialization failures
SELECT * FROM pg_stat_database WHERE xact_commit + xact_rollback > 0;

-- Check for long running transactions
SELECT * FROM pg_stat_activity WHERE state = 'active' AND xact_start < NOW() - INTERVAL '5 minutes';

-- Kill a connection
SELECT pg_terminate_backend(pid);

-- Check for locks
SELECT * FROM pg_locks;

-- Check for table bloat
SELECT
    schemaname || '.' || relname AS table,
    indexrelname AS index,
    pg_size_pretty(pg_total_relation_size(C.oid)) AS total_size,
    pg_size_pretty(pg_relation_size(C.oid)) AS internal_size,
    pg_size_pretty(pg_table_size(C.oid)) AS table_size,
    pg_size_pretty(pg_indexes_size(C.oid)) AS indexes_size,
    pg_size_pretty(pg_total_relation_size(reltoastrelid)) AS toast_size

```
