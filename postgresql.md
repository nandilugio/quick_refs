PostgreSQL
++++++++++

Docs at https://www.postgresql.org/docs/current/index.html

Easy test environment
=====================

Tested with postgres12, modify as needed.

```sh
# Download and run the image (add --rm to remove the container on exit)
docker run -e POSTGRES_HOST_AUTH_METHOD=trust -v ${PWD}:/host -p5432:5432 --name sql_test postgres:12

# Manage the container as usual
docker start sql_test -a
docker stop sql_test

# Create a database
docker exec -it sql_test psql postgres://postgres@localhost:5432/postgres -c 'create database sql_test'

# Optionally load some data
docker exec -it sql_test psql postgres://postgres@localhost:5432/sql_test -f /host/test_schema_and_data.sql

# Connect using your local psql
psql postgres://postgres@localhost:5432/sql_test

# Connect using psql from Postgres' Docker container
docker exec -it sql_test psql postgres://postgres@localhost:5432/sql_test
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
SERIAL                  -- auto-incrementing integer

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

TODO

JSON
----

TODO

Text search
-----------

TODO

Indexes
-------

TODO

Tables
======

Creation
--------

```sql
-- Create table
-- See https://www.postgresql.org/docs/current/sql-createtable.html
CREATE TABLE table_name (
    -- column_name data_type [column_constraint],
    id SERIAL PRIMARY KEY,
    other_id INTEGER REFERENCES other_table (other_id) ON DELETE CASCADE,
    name TEXT NOT NULL,
    age INTEGER CHECK (age > 0),
    created_at TIMESTAMP DEFAULT NOW()  
    ...
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
    TOOD -- statements
END;
$$ LANGUAGE plpgsql;

```

Triggers
========

TODO

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

Locking
-------

TODO

The rule system
===============

TODO

System Info and Administration
==============================

TODO