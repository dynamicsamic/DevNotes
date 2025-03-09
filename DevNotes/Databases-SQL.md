---
Created: 2024-07-24T23:59
---
#### Order of execution of `select` query
```SQL
FROM  + JOIN -- define the table(s)
WHERE  -- filter rows
GROUP BY  -- group rows
HAVING  -- filter groupped rows
SELECT  -- select target columns, make aliases
ORDER BY  -- provide final ordering
LIMIT  -- limit the result
```
---
#### List active connections to `Postgres` db
```SQL
SELECT * FROM pg_stat_activity WHERE datname = 'dbname';
```
---
####  SQL command to `create table only if does not exist`
```SQL
CREATE TABLE IF NOT EXISTS table_name (...);
```
---
#### `Postgres` connection string schema
```bash
{protocol}://{user}:{password}@{host}:{port}/{db_name}

# Example 
postgres://test_user:admin@localhost:5432/test_db
```
---
#### Connect to `Postgres` server in a docker container
```bash
# Run psql directly
sudo docker exec -it {container} psql -U postgres -d db_name

# Run bash and run pqsl from it
sudo docker exec -it {container} bash
pqsl -U postgres -d db_name # postgres_connection_string also works

# Run bash, run postgres as super user, run psql
sudo docker exec -it {container} bash
su postgres
psql
```
---
#### `UUID` as `index` ind DB: drawbacks and advantages
```markdown
- Drawbacks:
	- Takes up more disc space.
	- Slower selects, inserts and updates because its harder to build and maintain balanced B-tree.
    - Also their generation by DB is slower than integers.
    - Uniqueness not guaranteed, may encounter a collision.
- Advantages:
    - More suitable for distributed systems as UUIDs are mostly globally unique.
```
---
#### `Dump` into and `load` `Postgres` data from a file
```bash
# Dump database into .sql file
pg_dump -cC --inserts -U {user_name} {db_name} > {path_to_file}

# Load dumped database into postgres
psql -U {username} < {path_to_file}
```
----
#### `Postgres` create custom type ONLY if it does not exist
```SQL
DO $$ BEGIN
		CREATE TYPE my_type AS ENUM ('op1', 'op2');
EXCEPTION
		WHEN duplicate_object THEN null;
END $$;
```
---
#### Set default for `created_at` type of field
```SQL
CREATE TABLE table_name(
	created_at timestamptz NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```
---
#### `Asyncpg` create database only if it does not exist
```python
# Try to connect to your database, if it does not exist asyncpg raises
# InvalidCatalogNameError, we then connect to the default postgres
# `template1` database and create our database from there.

async def init_db(user: str, password: str, database: str) -> None:
	try:
		conn = await asyncpg.connect(
			user=user,
			password=password,
			database=database,
			host="localhost",
			port=5432,
		)
	except asyncpg.InvalidCatalogNameError:
		conn = await asyncpg.connect(
			user=user,
			password=password,
			host="localhost",
			port=5432,
			database="template1",
		)
		await conn.execute(f"CREATE DATABASE {database};")
```
---
#### `Asyncpg` main execution functions
```python
# All functions that start with `fetch` are also capable of inserting
# new data with `RETURNING` query instruction in them.

# Fetch one row from the db the result is dict-like instance which attributes
# may be accessed with ["attr"] notation. For empty query returns None.
conn.fetchrow(statement: str, values: Any)

# Fetch one more rows in a list. For empty query returns [].
conn.fetch(stmt: str, values: Any)

# Fetch a scalar value. If there are multiple columns returned by the query
# takes the value of the first column. 
conn.fetchval(stmt: str, values: Any)

# Execute the query and return a string result of the operation e.g. `0 1`
# meaning the opeation for one row was successfull.
conn.execute(stmt: str, values: Any)

# Execute the query with multiple sets of data. Returns None.
conn.executemany(stmt: str, values: list[tuple[Any]])
```
---
#### Useful SQL `queries` syntax
```SQL
-- Create database
CREATE DATABASE my_db

-- Drop database
DROP DATABASE my_db [CASCADE]

-- Rename database
ALTER DATABASE RENAME my_db TO new_db

-- Create empty table
CREATE TABLE my_table()

-- Create table with columns
CREATE TABLE my_table(
	table_id serial NOT NULL PRIMARY KEY,
	name varchar(60) UNIQUE NOT NULL)
)

-- Primary key and foreing key may be set after column definition
CREATE TABLE my_table(
	table_id INT AUTO_INCREMENT,
	other_id INT NOT NULL,
	PRIMARY KEY (table_id),
	FOREIGN KEY (other_id) REFERENCES other_table(other_id)
);

-- Create table from another table
CREATE TABLE my_table AS
SELECT 
	col1, 
	col2, 
	(SELECT AVG(col3) FROM other_table) AS col3
FROM other_table
WHERE col4 = (
	SELECT MAX(col5)
	FROM other_table
);

-- `ON DELETE` instruction in create table schema.
-- ON DELETE determins how a row in referencing table will change if the row in
-- referenced table will be deleted. 
-- Options are: `CASCADE`, `SET NULL`, `SET DEFAULT` and `RESTRICT`.
CREATE TABLE my_table(
	table_id INT PRIMARY KEY,
	other_id INT NOT NULL
	FOREIGN KEY (other_id) REFERENCES other_table(other_id) ON DELETE CASCADE
)

-- Create index
CREATE INDEX index_name ON table_name(col_name);

-- Create ENUM type
CREATE TYPE my_type AS ENUM ('op1', 'op2')

-- Add new column
ALTER TABLE my_table ADD COLUMN new_col NUMERIC(8,1);

-- Add new column before the first column (MySQL)
ALTER TABLE my_table ADD COLUMN col_name COL_TYPE FIRST;

-- Add new column after a column (MySQL)
ALTER TABLE my_table ADD COLUMN col_name COL_TYPE AFTER col_name;

-- Rename column
ALTER TABLE my_table RENAME COLUMN old_col TO new_col

-- Change column name or type
ALTER TABLE my_table CHANGE col_name new_name new_COL_TYPE;

-- Drop column
ALTER TABLE my_table DROP COLUMN old_col;

-- Shorthand drop column 
ALTER TABLE my_table DROP col_name;

-- Drop multiple columns
ALTER TABLE my_table DROP COLUMN col1, col2;

-- Add column with foreign key
ALTER TABLE my_table ADD COLUMN f_key int NOT NULL REFERENCES old_table(pkey)

-- Add foreign key constraint to existing column
ALTER TABLE my_table ADD FOREIGN KEY(f_key) REFERENCES old_table(pkey)

-- Add primary key constraint to existing column
ALTER TABLE my_table ADD PRIMARY KEY(pkey)

-- Drop constraint
ALTER TABLE my_table DROP CONSTRAINT pkey_constraint -- name of constraint

-- Add not null constraint to column
ALTER TABLE my_table ALTER COLUMN name SET NOT NULL;

-- Add unique consrtaint to column
ALTER TABLE my_table ADD UNIQUE(name);

-- Add default value to existing column
ALTER TABLE table_name ALTER COLUMN col_name SET DEFAULT <val>;

-- Insert values
INSERT INTO my_table (col1, col2, col3) values \
(val1_1, val2_1, val3_1), (val1_2, val2_2, val3_3) \
RETURNING pkey, col1, col2;

-- Insert values from a select query
INSERT INTO my_table (col1, col2, col3)
SELECT col1, col2, col3 FROM other_table;

-- Insert values from a select query with subquery
INSERT INTO my_table (col1, col2, col3)
SELECT col1, col2, col3
FROM other_table
WHERE col1 NOT IN (
	SELECT col1
	FROM my_table
);

-- Update rows
UPDATE my_table SET col1 = val1, col2 = val2 WHERE pkey = 2;

-- Update rows by several conditions with IF expression
UPDATE my_table
SET col1 = IF(col1 > 10, col1*1.1, col1),
	col2 = IF(col2 = "name", "bob", col2);

-- Update row using two tables
UPDATE 
	my_table
	INNER JOIN other_table USING(my_id)
SET 
	my_table.col1 = my_table.col1 + other_table.col3
	my_table.col2 = my_table.col2 - other_table.col5
WHERE 
	my_table.col3 = other_table.col8;

-- Update rows while quering the same table.
-- Updating and selecting rows in from one table single query is prohibited.
UPDATE 
	my_table
	INNER JOIN (
	SELECT col1
	FROM my_table
	WHERE col1 = 12324
	) AS query_in ON query_in.col1=my_table.col1
SET 
	my_table.col1 = 12451
WHERE 
	my_table.col1 = query_in.col1;

-- Delete rows
DELETE FROM my_table WHERE pkey = 11;

-- Delete rows with WHERE clause depending on data from other tables
DELETE FROM
	my_table
USING
	my_table
	INNER JOIN other_table ON my_table.id = other_table.my_id
	INNER JOIN third_table ON other_table.id = third.other_id
WHERE
	other_table.col1 = 11
	AND third_table.col4 = 'hello world';

-- Select unique values
SELECT DISTINCT col1, col2 FROM my_table;

-- Select not null values
SELECT col1, col2 FROM my_tab WHERE col1 IS NOT NULL;

-- `LIKE` operator usage
-- select names which have 'R' as second letter 
SELECT name FROM my_tab WHERE name LIKE '_R%';
-- the same but case insensitive match (will match `r` and `R`)
SELECT name FROM my_tab WHERE name ILIKE '_r%';
-- select names that have letter 'b' or 'B' somewhere
SELECT name FROM my_tab WHERE name ILIKE '%b%';
-- select names that don't start with 'A' and end with 'b' or 'B
SELECT name FROM my_tab WHERE name NOT LIKE 'A%' AND name NOT ILIKE '%b';
```
---
#### Set `CASCADE` delete for a `FOREIGN KEY`
```SQL
CREATE TABLE my_table (
	fk int NOT NULL REFERENCES another_table(pk) ON DELETE CASCADE
)
```
---
#### `Postgres` efficient `IN` lookup
```SQL
-- If your `in` values live in a table do.
SELECT col1, col2
FROM my_table
WHERE col1 IN (SELECT col3 FROM another_table); 

-- If you supply random values do.
SELECT col1, col2
FROM my_table
WHERE col1 = any (values (val1), (val2), (val3));
```
---
#### Set `Postgres` database `timezone`
```SQL
-- List all available timezones
SELECT * FROM pg_timezone_names;

-- Set timezone for your db
ALTER DATABASE my_db SET timezone TO 'Europe/Moscow'; 
```
---
#### `Truncate` data in table
```SQL
-- Truncate lets you empty your tables.
TRUNCATE my_table;

-- If tables references another table you need to truncate all these 
-- table at once.
TRUNCATE my_table, ref_table, other_table; 
```
---
#### Start `postgresql service`, create a user and database
```bash
sudo service postgresql start && echo "SELECT 'CREATE USER <username> WITH \
CREATEDB' WHERE NOT EXISTS (SELECT FROM pg_catalog.pg_roles WHERE rolname=\
'<rolename>')\gexec" | psql -U postgres -X
```
---
#### `Asyncpg` add query listener to connection
```python
# The asyncpg.connection.Connection class has the `add_query_logger` method
# which adds a callable to the set of the connection's query loggers.
# The callable should receive one argument, which is an instance of 
# a namedtuple type called `LoggedQuery`.

def logger(record: asyncpg.connection.LoggedQuery):
	print(record.query, record.args, record.elapsed)

async def fun(...):
	con = await db_pool.acquire()
	con.add_query_logger(logger)
	await con.execute(...)
	...
	
# It's also possible to use context manager to add and remove query loggers
# after each connection expiration.
with con.query_logger(logger):
	await con.execute(...)

# We can modify the asyncpg `Connection` class to add query loggers on each
# connection instantiation made by connection pool
from asyncpg.connection import Connection

class LoggedConnection(Connection):
	def __init__(self, *args, **kwargs):
		super().__init__(*args, **kwargs)
		self.add_query_logger(logger)
				
db_pool = await asyncpg.create_pool(..., connection_class=LoggedConnection)
```
---
#### Get efficient `estimated` count in `Postgres` with `asyncpg`
```python
# Builtin `count` function gets inefficient on large databases.
# If the precise count is not required we can run an estimated count query
# which uses interntal Postgres tools to made for its query planner.
# It relies on the internal statistics machinery, and if there has not
# been issued the `ANALYZE tablename` query, it will yield an incorrect result.
estimated_count = """
	SELECT  reltuples::bigint AS estimate
	FROM    pg_class
	WHERE   oid = 'public.{table_name}'::regclass;
"""

# We can run this query, and if it yields -1, it means we need to analyze the 
# table first. All the subsequent queries will not need the run analyze again 
# too soon. With asyncpg we can do.

async def estimate(table_name: str) -> int:
	qry = estimated_count.format(table_name=table_name)
	async with db_pool.acquire() as con:
		async with con.transaction:
			count = await con.fetchval(qry)
			if count < 0: # need to run analyze
				await con.execute("ANALYZE {table_name};")
				count = await con.fetchval(qry)
		
	return count
```
---
#### Referencing same column in multiple tables with `USING`
```SQL
-- When we join tables using the same column name we can use `USING` clause.
SELECT t1.col1 FROM table1 t1 INNER JOIN table2 t2 USING(item_id);
```
#### Add column with `NOT NULL` constraint  to existing DB 
```SQL
/* Since the column can not be empty we must provide a default value.
But we don't want this default value be present in the future, so we
delete it soon after the changes. */

ALTER TABLE my_table ADD COLUMN name VARCHAR(50) NOT NULL DEFAULT 'Any name';
ALTER TABLE my_table ALTER COLUMN name DROP DEFAULT;
```
---
#### Remove trailing zeros in a NUMERIC column in Postgres
```SQL
-- You can change the type from `NUMERIC` to REAL.
ALTER TABLE my_table ALTER COLUMN name TYPE REAL;
```
---
#### `Capitalize` (make first letters uppercase) strings in `Postgres`
```SQL
UPDATE my_table SET column_name = initcap(column_name);
```
---
#### `IF` function usage
```SQL
-- IF function signature: IF(condition, resul_if_true, result_if_false)
SELECT 
	author, 
	name,
	ROUND(IF(name='John', price*1.2, price*1.1), 2) AS new_price 
FROM table_name;

-- It is possible to place another IF function in the result_if_false place
SELECT 
	author, 
	name,
	ROUND(IF(name='John', price*1.2, IF(name='Helen', price*1.3, price*1.2)), 2)
FROM table_name;
```
---
#### `GROUP BY` and `HAVING` usage and examples
```SQL
-- GROUP BY is used to group rows according to the same value in a column
-- and conduct some aggregations on them.
-- For example this query
SELECT author, COUNT(title)
FROM books
GROUP BY author;
-- will select data from books table, group rows by same values in 
-- `author` column, count every title that was groped, 'glue' the author column
-- with counted values and return result.

-- Every column that is not being aggretated by the function should be 
-- placed into `GROUP BY` part of the query.

-- If you want to filter query results according to some aggregation, you
-- are not able to place it into the `WHERE` clause. You would need HAVING.
-- HAVING allows to perform filtering according to a result of an aggregation.

SELECT author, COUNT(title)
FROM books
GROUP BY author
HAVING COUNT(title) > 10;
-- This query will perform everything in previous query and additionally
-- filter out those authors that have less than 10 books in the db.
```
---
#### `Subqueries` usage and examples
```sql
-- Subqueries are usefull when:
--    - you need to filter by one or multiple computed values;
--    - you need to use a computed value in select clause.

-- Example with one computed value.
-- Select only rows with minimal price
SELECT author, title, price
FROM book
WHERE price = (
		SELECT MIN(price) 
		FROM book
	);

-- Example with multiple computed values.
-- Select only rows with authors that have more than 10 books in stock.
SELECT author, title, price
FROM book
WHERE author IN (
		SELECT author
		FROM book
		GROUP BY author
		HAVING SUM(amount) > 10
	);

-- Combine subqueries with ANY and ALL.
-- Select rows with price higher than all the average prices for every author.
SELECT author, title, price
FROM book
WHERE price > ALL (
		SELECT AVG(price)
		FROM book
		GROUP BY author
	);

-- Select rows with price higher than any of the average prices for every author.
SELECT author, title, price
FROM book
WHERE price < ANY (
		SELECT AVG(price)
		FROM book
		GROUP BY author
	);

-- You may need to make computations against a constant computed value to 
-- calculate a difference or deviation from target value.
SELECT 
	author, 
	price, 
	amount,
	(SELECT MAX(amount) FROM book) - amount as Difference
FROM book;
```
---
#### Date and time functions in MySQL
```SQL
-- Set default DATE and DATETIME for column
colname DATE DEFAULT (CURRENT_DATE);
colname DATE DEFAULT (NOW());
colname DATETIME DEFAULT (CURRENT_TIMESTAMP);
colname DATETIME DEFAULT (NOW());
colname TIMESTAMP DEFAULT (CURRENT_TIMESTAMP);
colname TIMESTAMP DEFAULT (NOW());

-- DATEDIFF(date_higher, date_lower) returns difference in days between two dates
SELECT DATEDIFF('2022-01-22', '2022-01-19') -- => 3

-- TIMESTAMPDIFF(time_unit, datetime_lower, datetime_higher) returns difference
-- between two datetimes measured in provided unit. Return value is int.
-- Valid units are: SECOND, MINUTE, HOUR, DAY, MONTH, YEAR.
SELECT TIMESTAMPDIFF(HOUR,'2023-05-18 10:15:30','2023-06-18 10:15:32');
-- => 744

SELECT TIMESTAMPDIFF(SECOND,'2023-06-18 10:15:30','2023-06-18 10:15:32');
-- => 2

-- YEAR(date) returns year from date
SELECT YEAR('2022-01-19'); -- => 2022

-- MONTH(date) returns positional number of month
SELECT MONTH('2022-01-19'); -- => 1 (first month in year)

-- MONTHNAME(date) return the name of the month
SELECT MONTHNAME('2022-01-19'); -- => 'January'

-- Generate date by adding interval to some date.
-- This will add 42 days. Also possible `MONTH` or `YEAR` instead of `DAY`.
SELECT DATE_ADD('2020-01-01', INTERVAL 42 DAY); -- => 2020-02-12

-- Generate random date with `RAND` function. `RAND` generates a float number
-- from 0 to 1 excluding. After we multiply it with 365 days in year and round
-- the number to the nearest whole number.
SELECT DATE_ADD('2020-01-01', INTERVAL FLOOR(RAND() * 365) DAY);

-- FROM_UNIXTIME(timestamp_, [format_str]) returns datetime from unix timestamp.
SELECT FROM_UNIXTIME(1727811071); -- => 2024-10-01 19:31:11
SELECT FROM_UNIXTIME(1727811071, '%Y/%d/%m %h:%i:%s'); -- => 2024/01/10 07:31:11
SELECT FROM_UNIXTIME(1727811071, '%Y %D %M %h:%i:%s %x'); 
-- => 2024 1st October 07:31:11 2024

-- Convert datetime to unix timestamp
SELECT UNIX_TIMESTAMP('2024-10-01 19:31:11'); -- => 1727811071

-- Convert seconds to time with SEC_TO_TIME(seconds)
SELECT SEC_TO_TIME(120); -- => 00:02:00, which is two minutes
SELECT SEC_TO_TIME(3600); -- =: 01:00:00, which is one hour
```
---
#### `UNION` operator
```SQL
-- UNION operator glues together results from multiple select queries.
-- It is useful when we have queries from two separate tables that
-- don't have a common table.

-- For example we have two tables with reports from last and 
-- second to last years: report1 and report2. We could combine queries to
-- provide year-to-year analytics.

SELECT
	YEAR(transaction_date) report_year,
	MONTHNAME(transaction_date) report_month,
	SUM(transaction.sum) month_sum
FROM
	report1
GROUP BY
	report_year,
	report_month

UNION ALL
-- ALL will show all rows including duplicates

SELECT
	YEAR(transaction_date) report_year,
	MONTHNAME(transaction_date) report_month,
	SUM(transaction.sum) month_sum
FROM
	report2
GROUP BY
	report_year,
	report_month

ORDER BY
	report_month,
	report_year

-- Important note!
-- Both query results must have equal number of columns and equal value types.
-- Sorting is provided with `ORDER BY` command placed in the last query.
```
---
#### Views
```SQL
-- Views are queries that do not execute emmediately, but rather postponed
-- until there is a need in them. Views can be executed as many times as it
-- needs to.
CREATE VIEW view_name AS
/*   QUERY GOES HERE   */

-- Views are useful when you need to:
-- store a repeated query;
-- return a specific subset of a table;
-- return a specific subset of a table with columns modified or added.


/* Create a subset of a table */
CREATE VIEW vw_active_children AS
SELECT
	name,
	age,
	score
FROM
	subscribers
WHERE
	age < 18
	AND status = 'ACTIVE';

-- Then query your view.
SELECT * FROM vw_active_children;

/* Create a subset with modifed info.
	Resulting query will return emails in the form of `box****.ru`
*/
CREATE VIEW vw_active_adults_public AS
SELECT
	name,
	score,
	concat(left(email, 3), '****', right(email, 3))
FROM
	subscribers
WHERE
	age >= 18;

-- Views do not store data, that's why it does not get stale when the
-- main table gets updated.

-- Views are also suitable for table creation and row insertion
CREATE TABLE active_adults_public AS
SELECT * FROM vw_active_adults_public;

-- If you're not sure whether the view exists use REPLACE in combination with CREATE.
CREATE OR REPLACE VIEW my_view AS ...;
DROP VIEW my_view;
```
---
#### Variables
```SQL
-- Variables declared using this syntax `@var_name := expression `;
-- You can declare a variable as a query and then use it in other expesssions.

SELECT
	@bad_id := customer_id
FROM 
	customers
	INNER JOIN orders USING(customer_id)
WHERE
	orders.payment IS NULL;

UPDATE
	customers
SET
	status = 'Freeze'
WHERE
	id IN (@bad_id);

-- Scalar variables can be declared upfront using `declare` or `set`
-- keywords.
set @my_id := 0;

SELECT  *,
		(@my_id := @my_id + 1) my_id
FROM    my_table
-- This query will select all columns from my_table and add column `my_id`
-- which will be incremented for every row;

-- Update query with variables
set @my_id := 0;
set @row_num := 1;

UPDATE  my_table
SET     col1 = IF(
			col2 = @my_id,
			@row_num := @row_num + 1,
			@row_num := 1 AND @my_id := col2)
);
-- This query will update col1 updating variables in IF expression body.

-- A slightly different way of declaring a select-variable
SET @my_var = (
	SELECT  col1,
			col2
	FROM    my_table
);
```
---
#### Random sorting in MySQL
```SQL
SELECT
	...
FROM
	...
ORDER BY
	RAND();
```
---
#### Functions to check for `NULL`ness
```sql
-- Function coalesce(A, B, C) returns first non null value.
SELECT  COALESCE(SUM(col1), 0) -- If SUM(col1) is NULL, returns 0.

-- Function IFNULL(A, B), returns B if a is NULL, A otherwise
SELECT IFNULL(SUM(col1), 1) -- if SUM(col1) is NULL, returns 1.
```
---
#### String functions in MySQL
```sql
-- `concat(*args)`- concatenates provided arguments
concat(col1, ' ', concat(col2, '...'), '!');

-- 'left(s: str, take_chars: int)' - takes n chars from string's left side.
left('Hello world!', 5); -- => Hello

-- `right(s: str, take_chars: int)` - takes n chars from string's right side.
right('Hello world!', 6); -- => world!

-- `length(s: str)` - returns the number of chars in a string.
length('Hello'); -- => 5

-- `substr(s: str, start_from: int, take_chars: int)` - takes n chars starting 
-- from specifed char. The first char has index 1, not 0.
substr('Hello world!', 1, 2); -- => He
substr('Hello world!', 1); -- => Hello world!

-- instr(s1: str, s2: str): int - return 1 if s2 is a substring of or equalt to
-- s1, return 0 otherwise.
instr('Hello world', 'Hello'); -- => 1
instr('Hello world', 'Helloh'); -- => 0

-- lpad(val: str|int, final_len:int, 'fill_val:str'):str - prefix a string or
-- number with fill value to reach the final length.
lpad(22, 4, '0'); -- => 0022
lpad('A', 10, 'B'); --=> BBBBBBBBBA
-- rpad does the same but from the end of the mutated value
```
---
#### `CASE` statement syntax and use cases
```sql
-- General syntax
CASE    WHEN <condition1> THEN <value_or_expression1>
		WHEN <condition2> THEN <value_or_expression2>
		WHEN <condition3> THEN <value_or_expression3>
		ELSE <value_or_expression4>
END

-- CASE statement can be used in SELECT, UPDATE, SET, DELETE, WHERE, HAVING
-- and ORDER BY blocks.
/**** EXAMPLES ****/
-- In SELECT block. Add new column which values depend on some coditions.
SELECT    city,
		  location = CASE
						  WHEN region = 'xyz' THEN 'North-East'
						  WHEN region = 'abc' THEN 'South-West'
						  ELSE 'Not-Known'
					END
FROM ...    -- Also possible CASE ... END AS location.

-- In ORDER BY block. Apply conditional sorting.
SELECT    city
FROM      ....
ORDER BY  CASE  WHEN region = 'xyz' THEN latitude
				ELSE longitude END ASC,
		  CASE WHEN sea_level = 22 THEN longitude END DESC;

-- In UPDATE block. Set various values according to some logic.
UPDATE   cities
SET      description = (CASE    WHEN population < 150 THEN 'SMALL'
								WHENE population < 800 THEN 'MEDIUM'
								ELSE 'LARGE'
						END);	

-- In HAVING block. Provide extra filtering on aggregated calculations. 
SELECT    city,
		  SUM(population)
FROM      cities
GROUP BY  city
HAVING    CASE  WHEN SUM(population) > 200000 THEN 'LARGE'
			    WHEN SUM(population) > 3000000 THEN 'EXTRA LARGE'
		  END = 'EXTRA LARGE';
```
---
#### `REGEXP` in queries
```sql
-- Regular expressions in SQL work via the REGEXP operator.
-- Regexps can be used in SELECT block to produce new columns with calculated
-- values or in WHERE block to filter rows.
SELECT  username,
	    description REGEXP 'teenager' AS is_teenager
FROM    ...

-- Filtering rows in WHERE block
SELECT   name,
		 id
FROM     users
WHERE    description REGEXP 'teenager|adult';

-- To separate a pattern by its boundaries use '\\b' modificator
SELECT   name
FROM     users
WHERE    description REGEXP '\\bmin'; 
-- This way all occurrances of exactly `min` word would be found.

-- It is also possible to costruct patterns with the `CONCAT` function.
SELECT   name
FROM     users
WHERE    emai REGEXP CONCAT('^', name, '@', '.+', '.com'); 
```
---
#### Table Expression and the `WITH` statement
```sql
-- WITH construction is used to simplify usage of subqueries and 
-- calculated (derived) tables.

-- WITH construct has following syntax
WITH alias_for_query (alias_colname1, alias_colname2)
AS (SELECT ...)
SELECT alias_colname1,
	   alias_colname2
  FROM alias_for_query
 WHERE ...

-- You can create multiple tables in one WITH block
WITH 
	alias_query1 (alias_colname1, alias_colname2)
	AS (
		SELECT ...
	),
	alias_query2 (alias_colname1, alias_colname2)
	AS (
		SELECT ...
	)
SELECT aq1.alias_colname1,
	   aq2.alias_colname1
  FROM alias_query1 AS aq1
	   INNER JOIN alias_query2 AS aq2
		     ON aq1.alias_colname1 = aq2.alias_colname2

-- A realworld example
WITH
	 sum_total (step_name, num_total_answers)
	 AS (
		    SELECT step_name,
				   COUNT(*)
		      FROM step
			       INNER JOIN step_student USING(student_id)
		  GROUP BY step_name
	 ),
	 sum_correct (step_name, num_correct_answers)
	 AS (
		    SELECT step_name,
				   SUM(IF(result = 'correct', 1, 0))
		      FROM step
			       INNER JOIN step_student USING(student_id)
		  GROUP BY step_name
	 )
  SELECT sc.step_name AS Step,
	     ROUND(sc.num_correct_answers / st.num_total_answers * 100) AS Rate
    FROM sum_correct AS sc
		 INNER JOIN sum_total AS st
		 	   ON sc.step_name = st.step_name
ORDER BY Rate DESC,
         Step ASC
;

/* To combine queries that use table expresssions with UNION operator, first declare all table expressions, separated by comma, after that declare queries and combine them with UNION as usual. */

WITH query1 (col1, col2) AS -- table experssion block start
	 (
	 SELECT ...
	 ),
	 query2 (col1, col2) AS
	 (
	 SELECT ...
	 ),
	 query3 (col1, col2) AS
	 (
	 SELECT ...
	 )
     -- table expression block ended
SELECT ...
FROM query1

UNION ALL

SELECT ...
FROM query2

UNION ALL

SELECT ...
FROM query3

ORDER BY ...;
```
---
#### Window functions: overview
```sql
/* Window functions are analytical functions that are applied to all rows in a query and added to query results as additional columns. The name `window` comes from that fact, that this functions can partition query in defferent subsets, which makes them look like windows. 
*/
-- To apply a window function use next syntax
SELECT 
	function_name[(args)] OVER ([PARTITION BY column] [ORDER BY columns]);

/*Window functions can be divided into three main categories.
They are:*/
--  Ranking functions:
ROW_NUMBER() -- assign a unique ordinal number to each row in window;
RANK() -- assign a rank to each row; if, for example, three rows have the same 
-- #1 rank, the next row after these rows will get #4.
DENSE_RANK() -- assign a rank to each row; if, for example, three rows have  
-- the same #1 rank, the next row after these rows will get #2.
NTILE(num_buckets: int) -- evenly distribute rows into number of buckets.

-- Offset functions:
LEAD(expression, [offset_num], [value_if_null]) -- for every row in window returns its next value.
LAG(expression, [offset_num], [value_if_null]) -- for every row in window returns its previous value.
FIRST_VALUE(expression) -- returns the first value in window
LAST_VALUE(expression) -- returns the last value in window
NTH_VALUE(expression, position) -- returns provided row postiion value

-- Analytical functions:
CUME_DIST() -- cumulative distribution -- calculates the number of rows that are less than or equal to current row divided by amount of all rows in window.
... -- to be continued

-- Aggregate functions:
	MAX, MIN, SUM, COUNT, AVG -- do their job but instead of folding to one value they add a column with calculated values.  
```
---
#### Window functions: Ranking functions
```sql
                                   __________
__________________________________|TEST_TABLE|_________________________________
order_id	client_id	created_at	         amount
	1	        1	   2024-10-04 19:22:36	  10
	2	        1	   2024-10-02 19:22:36	  12
	3	        1	   2024-10-04 19:22:36	   3
	4	        2	   2024-10-02 19:22:36	   1
    5	        2	   2024-10-04 19:22:36	   3
_______________________________________________________________________________

/*--------------------------------ROW_NUMBER----------------------------------*/
-- ROW_NUMBER() adds unique ordinal number to each row; 
-- does not take any arguments.
-- you may provide ORDER BY to determine order of numbers;
-- you may provide PARTITION BY to determine whether you need to start numbering
-- with every new group (window).
  SELECT *,
		 ROW_NUMBER() OVER (ORDER BY order_id) AS row_num
    FROM orders
ORDER BY order_id;
							||
							\/
order_id	client_id	created_at	         amount    row_num
	1	        1	   2024-10-04 19:22:36	  10        1
	2	        1	   2024-10-02 19:22:36	  12        2
	3	        1	   2024-10-04 19:22:36	   3        3
	4	        2	   2024-10-02 19:22:36	   1        4
    5	        2	   2024-10-04 19:22:36	   3        5
-- row_num assigned consequtive numbers to every row following order_id ordering.

-- We could change the ordering to client_id and amount instead of order_id
  SELECT *,
		 ROW_NUMBER() OVER (ORDER BY client_id, amount) AS row_num
    FROM orders
ORDER BY client_id, 
		 amount;
							||
							\/
order_id	client_id	created_at	         amount    row_num
	3	        1	   2024-10-04 19:22:36	   3        1
	1	        1	   2024-10-04 19:22:36	  10        2
	2	        1	   2024-10-02 19:22:36	  12        3
	4	        2	   2024-10-02 19:22:36	   1        4
    5	        2	   2024-10-04 19:22:36	   3        5
-- This time query sorted first  by client_id, after by amount.

/* The ORDER BY block in the main query is executed after ORDER BY block
in window function. In case the two ORDER BY blocks do not match, the 
row numbers will be assigned first following their ORDER BY block, after that all the resulting query would be sorted accodring to main ORDER BY */
  SELECT *,
		 ROW_NUMBER() OVER (ORDER BY client_id) AS row_num
    FROM orders
ORDER BY amount;
							||
							\/
order_id	client_id	created_at	         amount    row_num
	4	        2	   2024-10-02 19:22:36	   1          4
	3	        1	   2024-10-04 19:22:36	   3          3
    5	        2	   2024-10-04 19:22:36	   3          5
	1	        1	   2024-10-04 19:22:36	  10          1
	2	        1	   2024-10-02 19:22:36	  12          2
/*We can see that row numbers are calculated depending on client id ordering
but the final result is sorted by amount, so row number go in random order.*/

/* Add partitioning to start row numbering separately for every client-based
window */
  SELECT *,
		 ROW_NUMBER() 
		 OVER (PARTITION BY client_id ORDER BY client_id, amount) AS row_num
    FROM orders
ORDER BY client_id,
         amount;
							||
							\/
order_id	client_id	created_at	         amount    row_num
	3	        1	   2024-10-04 19:22:36	   3          1
	1	        1	   2024-10-04 19:22:36	  10          2
	2	        1	   2024-10-02 19:22:36	  12          3
	4	        2	   2024-10-02 19:22:36	   1          1
    5	        2	   2024-10-04 19:22:36	   3          2
-- This time row numbering starts from 1 for every distinct client_id.

/*-----------------------------------RANK-------------------------------------*/
-- RANK() adds rank to each row; Does not assign consequtive ranks for equal values.
-- Does not take any arguments.
  SELECT *,
		 RANK() OVER (ORDER BY amount) AS ranq
    FROM orders
ORDER BY amount
							||
							\/
order_id	client_id	created_at	         amount     ranq
	4	        2	   2024-10-02 19:22:36	   1          1
	3	        1	   2024-10-04 19:22:36	   3          2
    5	        2	   2024-10-04 19:22:36	   3          2
	1	        1	   2024-10-04 19:22:36	  10          4
	2	        1	   2024-10-02 19:22:36	  12          5
/* Every row recieved its rank depending on the amount column. Orders with
order_id 3 and 5 both received rank 2 because thier amounts are equal, the next row got 4-th rank instead of 5-th */

-- Use PARTITION BY to calculate separate ranks for every group
  SELECT *,
		 RANK() OVER (PARTITION BY client_id ORDER BY created_at) AS ranq
    FROM orders
ORDER BY client_id,
         created_at
							||
							\/
order_id	client_id	created_at	         amount     ranq
	2	        1	   2024-10-02 19:22:36	  12          1         
	1	        1	   2024-10-04 19:22:36	  10          2
	3	        1	   2024-10-04 19:22:36	   3          2
	4	        2	   2024-10-02 19:22:36	   1          1
    5	        2	   2024-10-04 19:22:36	   3          2
-- Every row with equal created_at column value received similar ranks and
-- we recalculate ranks for every distinct client_id wih PARTITION BY clause

/*---------------------------------DENSE_RANK---------------------------------*/
-- Works the same way as the RANK function, but does not skip ranks
  SELECT *,
		 DENSE_RANK() OVER (ORDER BY amount) AS ranq
    FROM orders
ORDER BY amount
							||
							\/
order_id	client_id	created_at	         amount     ranq
	4	        2	   2024-10-02 19:22:36	   1          1
	3	        1	   2024-10-04 19:22:36	   3          2
    5	        2	   2024-10-04 19:22:36	   3          2
	1	        1	   2024-10-04 19:22:36	  10          3
	2	        1	   2024-10-02 19:22:36	  12          4

/*-----------------------------------NTILE-----------------------------------*/
-- NTILE(N) -- evenly distributes rows into one or more buckets.
-- the N parameter is the number (integer) of buckets;
-- missing argument or zero value raise error; 
-- minimal N is 1, which means all rows will be in one bucket.

-- Distrubte 5 rows into 3 buckets sorted by amount
  SELECT *,
		 NTILE(3) OVER (ORDER BY amount) AS bucket
    FROM orders
ORDER BY amount
							||
							\/
order_id	client_id	created_at	         amount     bucket
	4	        2	   2024-10-02 19:22:36	   1          1
	3	        1	   2024-10-04 19:22:36	   3          1
    5	        2	   2024-10-04 19:22:36	   3          2
	1	        1	   2024-10-04 19:22:36	  10          2
	2	        1	   2024-10-02 19:22:36	  12          3

-- Distrubte 5 rows into 4 buckets sorted by created_at
  SELECT *,
		 NTILE(4) OVER (ORDER BY created_at) AS bucket
    FROM orders
ORDER BY created_at
							||
							\/
order_id	client_id	created_at	         amount     bucket
	2	        1	   2024-10-02 19:22:36	  12          1
	4	        2	   2024-10-02 19:22:36	   1          1
	1	        1	   2024-10-04 19:22:36	  10          2
	3	        1	   2024-10-04 19:22:36	   3          3
    5	        2	   2024-10-04 19:22:36	   3          4

-- Partition 5 rows into groups by client_id and for every group distribute
-- rows into 2 buckets.
  SELECT *,
		 NTILE(2) OVER (PARTITION BY client_id ORDER BY client_id) AS bucket
    FROM orders
ORDER BY client_id
							||
							\/
order_id	client_id	created_at	         amount     bucket
	1	        1	   2024-10-04 19:22:36	  10          1
	2	        1	   2024-10-02 19:22:36	  12          1
	3	        1	   2024-10-04 19:22:36	   3          2
	4	        2	   2024-10-02 19:22:36	   1          1
    5	        2	   2024-10-04 19:22:36	   3          2
```
---
#### Window functions: Offset functions
```sql
NTH_VALUE(expression, position) -- returns provided row postiion value

/*-----------------------------------LAG-------------------------------------*/
/* LAG(expression, [offset], [value_if_null]) -- takes a column or expression and forms a new column in which every row has value of the previous row from the target column;
-- `expression` param determines the target column to take values from;
-- `offset` param determines how many rows it should look back to find the value.
Defaults to 1, meaning that it should look for the previous row. If, for example we set offset to 3, it would mean looking back for 3 rows.
-- `value_if_null` param determines a value to be used, when there is no value for the previous row. Defaults to NULL, often it is useful to set it to 0;
-- ORDER BY may be skipped ;
-- PARTITION BY makes LAG to restart previous row definition for every group*/

-- Simplest possible LAG function example. It uses the natural ordering to generate previous values for every amount value. The first row does not have a previous value, so it defaults to NULL.
  SELECT *,
		 LAG(amount) OVER () AS prev_val
    FROM orders
ORDER BY order_id;
							||
							\/
order_id	client_id	created_at	         amount    prev_val
	1	        1	   2024-10-04 19:22:36	  10        NULL
	2	        1	   2024-10-02 19:22:36	  12        10
	3	        1	   2024-10-04 19:22:36	   3        12
	4	        2	   2024-10-02 19:22:36	   1        3
    5	        2	   2024-10-04 19:22:36	   3        1

-- We can change the offset parameter to look for 2 previous rows. This way first two rows would not have a value, because there is no 2 rows prior to them.
-- Other rows would recieve values as expected.
  SELECT *,
		 LAG(amount, 2) OVER () AS prev_val
    FROM orders
ORDER BY order_id;
							||
							\/
order_id	client_id	created_at	         amount    prev_val
	1	        1	   2024-10-04 19:22:36	  10        NULL
	2	        1	   2024-10-02 19:22:36	  12        NULL
	3	        1	   2024-10-04 19:22:36	   3        10
	4	        2	   2024-10-02 19:22:36	   1        12
    5	        2	   2024-10-04 19:22:36	   3        3

-- We can set default missing value to 0 and change the ordering to take created_at and amount alongside; Here we can see, that first value is no loger a null, but 0. Also the ordering has changed as expected.
  SELECT *,
		 LAG(amount, 1, 0) OVER (ORDER BY created_at, amount) AS prev_val
    FROM orders
ORDER BY created_at, amount;
							||
							\/
order_id	client_id	created_at	         amount    prev_val
	4	        2	   2024-10-02 19:22:36	   1        0
	2	        1	   2024-10-02 19:22:36	  12        1
	3	        1	   2024-10-04 19:22:36	   3        12
    5	        2	   2024-10-04 19:22:36	   3        3
	1	        1	   2024-10-04 19:22:36	  10        3

-- We can also use partition by client_id. This way previous value generation restarts for every client_id. We also sort rows by client_id and descending amount.
  SELECT *,
		 LAG(amount) OVER (PARTITION BY client_id ORDER BY client_id, amount DESC) AS prev_val
    FROM orders
ORDER BY client_id, amount DESC;
							||
							\/
order_id	client_id	created_at	         amount    prev_val
	2	        1	   2024-10-02 19:22:36	  12        NULL
	1	        1	   2024-10-04 19:22:36	  10        12
	3	        1	   2024-10-04 19:22:36	   3        10
    5	        2	   2024-10-04 19:22:36	   3        NULL
	4	        2	   2024-10-02 19:22:36	   1        3

-- It is also possible to use value generated by window functions in different expressions. Here we can look at the previous amount and if current amount is less by 2 units mark it as `DROPPED`. The generated `status` column makes it obvious that we experienced a drop in our sales with order id #3.
  SELECT *,
		 CASE 
             WHEN LAG(amount, 1, 0) OVER (ORDER BY amount DESC) - amount > 2 
	             THEN 'DROPPED'
             ELSE 'NORMAL'
         END AS status
    FROM orders
ORDER BY amount DESC;
							||
							\/
order_id	client_id	created_at	         amount    status
	2	        1	   2024-10-02 19:22:36	  12        NORMAL
	1	        1	   2024-10-04 19:22:36	  10        NORMAL
	3	        1	   2024-10-04 19:22:36	   3        DROPPED
    5	        2	   2024-10-04 19:22:36	   3        NORMAL
	4	        2	   2024-10-02 19:22:36	   1        NORMAL

/*----------------------------------LEAD-------------------------------------*/
/* LEAD(expression, [offset], [value_if_null]) -- is similar to the `LAG` function but it forms a new column in which every row has value of the next row from the target column;
-- `expression` param determines the target column to take values from;
-- `offset` param determines how many rows it should look forward to find the value.
Defaults to 1, meaning that it should look for the next row. If, for example we set offset to 3, it would mean looking foward for 3 rows.
-- `value_if_null` param determines a value to be used, when there is no value for the previous row. Defaults to NULL, often it is useful to set it to 0;
-- ORDER BY may be skipped;
-- PARTITION BY makes LEAD to restart next row definition for every group*/

-- We want to find the next amount value for every row sorting by created_at column id desceding order and client_id in ascending order. We also want to replace missing values with 0. We can see that every next_val column value equals to the next row value from corresponding `amount` column. And the last value in `next_val` column has value of 0, because it does not have a next value. 
  SELECT *,
		 LEAD(amount, 1, 0) OVER (ORDER BY created_at DESC, client_id) AS next_val
    FROM orders
ORDER BY created_at DESC, client_id;
							||
							\/
order_id	client_id	created_at	         amount    next_val
	1	        1	   2024-10-04 19:22:36	  10        3
	3	        1	   2024-10-04 19:22:36	   3        3
    5	        2	   2024-10-04 19:22:36	   3        12
	2	        1	   2024-10-02 19:22:36	  12        1
	4	        2	   2024-10-02 19:22:36	   1        0

/*--------------------------------FIRST_VALUE---------------------------------*/
/* FIRST_VALUE(expression) -- takes in an expression or column and returns a column with the window's first row value.
-- ORDER BY may be skipped. This way the ordering from main query ORDER BY would be used.
-- PARTITION BY -- determines first value for each group. */

-- Let's find the first value from the entire query. If we set the ordering by amount DESC we would get the highest amount displayed for every row. 
  SELECT *,
		 FIRST_VALUE(amount) OVER (ORDER BY amount DESC, client_id) AS min_val
    FROM orders
ORDER BY amount DESC, client_id;
							||
							\/
order_id	client_id	created_at	         amount    min_val
	2	        1	   2024-10-02 19:22:36	  12        12
	1	        1	   2024-10-04 19:22:36	  10        12
	3	        1	   2024-10-04 19:22:36	   3        12
    5	        2	   2024-10-04 19:22:36	   3        12
	4	        2	   2024-10-02 19:22:36	   1        12

-- If we add paritioning by client_id we would get the highest amount for every client displayed at every row. This would be 12 and 3 for clients #1 and #2 respectively.
  SELECT *,
		 FIRST_VALUE(amount) OVER (PARTITION BY client_id ORDER BY amount DESC, client_id) AS max_val
    FROM orders
ORDER BY amount DESC, client_id;
							||
							\/
order_id	client_id	created_at	         amount    min_val
	2	        1	   2024-10-02 19:22:36	  12        12
	1	        1	   2024-10-04 19:22:36	  10        12
	3	        1	   2024-10-04 19:22:36	   3        12
    5	        2	   2024-10-04 19:22:36	   3        3
	4	        2	   2024-10-02 19:22:36	   1        3

-- We can use the maximum value to calculate the difference in sales between the highest and current amounts for each client. 
  SELECT *,
		 FIRST_VALUE(amount) 
			 OVER (PARTITION BY client_id ORDER BY amount DESC, client_id) 
			 - amount AS sales_diff
    FROM orders
ORDER BY amount DESC, client_id;
							||
							\/
order_id	client_id	created_at	         amount    sales_diff
	2	        1	   2024-10-02 19:22:36	  12        0
	1	        1	   2024-10-04 19:22:36	  10        2
	3	        1	   2024-10-04 19:22:36	   3        9
    5	        2	   2024-10-04 19:22:36	   3        0
	4	        2	   2024-10-02 19:22:36	   1        2

/*---------------------------------LAST_VALUE---------------------------------*/
/* LAST_VALUE(expression) -- takes in an expression or column and returns a column with the window's last row value.
-- ORDER BY may be skipped. This way the ordering from main query ORDER BY would be used.
-- PARTITION BY -- determines last value for each group. */

-- The simplest query - get the last value from entire window. We get the last value from amount column (which is at order_id #5) and get the ordering from main ORDER BY clause.
  SELECT *,
		 LAST_VALUE(amount) OVER () AS last_value
    FROM orders
ORDER BY client_id;
							||
							\/
order_id	client_id	created_at	         amount    last_value
	1	        1	   2024-10-04 19:22:36	  10        3
	2	        1	   2024-10-02 19:22:36	  12        3
	3	        1	   2024-10-04 19:22:36	   3        3
	4	        2	   2024-10-02 19:22:36	   1        3
    5	        2	   2024-10-04 19:22:36	   3        3

-- We can also get the minimum amount value, by changing the ordering. The LAST_VALUE function looking for the last row's amount column, finds its value and attaches it to the `last_value` column. 
  SELECT *,
		 LAST_VALUE(amount) OVER () AS min_value
    FROM orders
ORDER BY amount DESC;
							||
							\/
order_id	client_id	created_at	         amount    min_value
	2	        1	   2024-10-02 19:22:36	  12        1
	1	        1	   2024-10-04 19:22:36	  10        1
	3	        1	   2024-10-04 19:22:36	   3        1
    5	        2	   2024-10-04 19:22:36	   3        1
	4	        2	   2024-10-02 19:22:36	   1        1

-- LAST_VALUE may lead to unexpected behavior when used with several ORDER BY clauses in its OVER part. Here it seems to just copy the amount column. Here we want ordering by client_id first and by amount after it.
  SELECT *,
		 LAST_VALUE(amount) OVER (ORDER BY client_id, amount) AS min_value
    FROM orders
ORDER BY client_id, amount;
							||
							\/
order_id	client_id	created_at	         amount    min_value
	3	        1	   2024-10-04 19:22:36	   3        3
	1	        1	   2024-10-04 19:22:36	  10        10
	2	        1	   2024-10-02 19:22:36	  12        12
	4	        2	   2024-10-02 19:22:36	   1        1
    5	        2	   2024-10-04 19:22:36	   3        3
```
---
#### Test table
```sql

CREATE TABLE orders (
    order_id INT AUTO_INCREMENT,
    client_id INT NOT NULL,
    created_at DATETIME NOT NULL DEFAULT(NOW()),
    amount INT NOT NULL,
    PRIMARY KEY (order_id)
);

INSERT INTO orders (client_id, amount)
VALUES (1, 10), (1, 12), (1, 3), (2, 1), (2,3);
UPDATE orders
SET created_at = DATE_ADD(NOW(), INTERVAL 2 DAY)
WHERE order_id % 2 <> 0;
```
---
#### PSQL (Postgres CLI)
```sql
--- Connect to database
psql postgres://test_user:admin@localhost:5432/test_db
-- OR more explicitly
psql -h my_host.yandexcloud.net -p 6432 -U my_username -d my_database sslmode=verify-full
-- OR super verbosely
psql host=some_host.yandexcloud.net port=6432 sslmode=verify-full dbname=my_database user=my_username

\l -- List all databases
\l+ -- more verbose output

\c <db_name> -- connect to another database
\d -- list all tables in a database (including sequences) 
\dt -- list tables only
\dt+
\d <table_name> -- Describe table
\d+ <table_mame>

-- Select database names
SELECT datname FROM pg_database;

\dn -- list available schemas
\df -- list avaialable functions
\dv -- list avaialable views
\du -- list users
\s -- show history
\timing -- activate and deactivate timing 
```