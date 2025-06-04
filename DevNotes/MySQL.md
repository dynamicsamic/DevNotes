#### Install MySQL on Ubuntu
```bash
# Update all packages
$ sudo apt update && sudo apt upgrade -y

# Install mysql
$ sudo apt install mysql-server
 
# MySQL was installed, but there is no users except the root.
# Need to create a user.
# Run mysql in sudo mode
sudo mysql
>>> Welcome to the MySQL monitor.  Commands end with ; or \g.
```
```sql
-- Create a user
mysql> CREATE USER 'user_name'@'localhost' IDENTIFIED BY 'password';
>>> Query OK, 0 rows affected (0,01 sec)
-- Grant all privileges to enable new user create databases
mysql> GRANT ALL PRIVILEGES ON * . * TO 'user_name'@'localhost';
>>> Query OK, 0 rows affected (0,01 sec)
mysql> FLUSH PRIVILEGES; -- this maybe optional
>>> Query OK, 0 rows affected (0,01 sec)
```
---
#### Connect to MySQL using command-line arguments
```bash
mysql <database_name> -h <database_host> -u <database_user> -p
# promts to enter your password
```
---
#### Execute a SQL query without entering the MySQL CLI
```bash
mysql <database_name> -h <database_host> -u <database_user> -p -e "SELECT * FROM my_table LIMIT 100"
```
---
#### Redirect a SQL query output to a file
```bash
mysql <database_name> -h <database_host> -u <database_user> -p -e "SELECT * FROM my_table LIMIT 100" > output.csv
```
---
#### Dump query results in  a `.tsv` file
Very useful, when:
- the database and the server that makes query aren't on the same host (container, pod), like on the cloud;
- MySQL server doesn't have file access privileges;
- no other query tools are available for you, only MySQL installed. 
```bash
mysql <database_name> -h <database_host> -u <database_user> --default-character-set=utf8mb4 --password -e "SELECT * FROM my_table LIMIT 10" > /path/to/output-file.tsv

# Now you can convert this file into pandas dataframe with
df = pd.read_csv(file_name, sep='\t')

# Or convert it into csv using Linux `tr` utility
tr '\t' ',' < original.tsv > transformed.csv

# You can provide not sql query from a file by redirecting its contents to  Mysql
mysql <database_name> -h <database_host> -u <database_user> --default-character-set=utf8mb4 -p < sql_query.sql > /path/to/output-file.tsv
```
---
#### Store query results in a `.csv` file
- If you have file privileges you can use MySQL builtin mechanisms to store a query in a `.csv` file.
```sql
SELECT order_id,product_name,qty
FROM orders
WHERE foo = 'bar'
INTO OUTFILE '/var/lib/mysql-files/orders.csv'
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n';
```
---
#### Prettify query output in CLI
```mysql
-- Finish your query with `\G` instead of `;`
SELECT * FROM my_table\G

-- To enable a scrolling mode
pager less -S
-- Here run your query
SELECT ...
-- Turn of the scrolling mode
nopager
```
---
#### Create a database dump
```bash
mysqldump -u <db_user> -h <db_host> -p <db_name> <table1> <table2> <table3> --default-character-set=utf8mb4 --no-tablespaces > mysql_dump.sql 
# --no-tablespaces - without this option dump will fail because of absent PROCESS priviliges
# To exclude multiple tables you can pass --ignore-table=Table1 --ignore-table=Table2 --ignore-table=Table3
```
---
#### Restore database from a dump
```bash
mysql <db_name> -u <db_user> -h <db_host> -p < mysql-dump.sql
```
---
#### Change user password
```mysql
mysql> ALTER USER 'user_name'@'localhost' IDENTIFIED BY 'NEW_USER_PASSWORD';
mysql> FLUSH PRIVILEGES;

-- OR
mysql> UPDATE mysql.user SET authentication_string = PASSWORD('NEW_USER_PASSWORD') WHERE User = 'user_name' AND Host = 'localhost';
mysql> FLUSH PRIVILEGES;
```
---
#### mysql CLI
- Run `mysql CLI`
```bash 
$ mysql -u user_name -p
# or directly connecting to a database
$ mysql db_name -u user_name -p
```
- Enter your password and start using the database
```mysql
-- Connect to database, if you haven't done this in the previous step
USE <db_name>;
 
>>> SHOW DATABASES; -- show all databases
>>> SHOW DATABASES LIKE 'patter'; -- show specific databases
 
-- Connect to database
>>> USE <db_name>;

-- Show all tables
>>> SHOW TABLES;
+----------------+
| Tables_in_test |
+----------------+
| orders         |
| user           |
+----------------+
2 rows in set (0.01 sec)
 
-- Show table schema
DESCRIBE <table_name>; -- also possible EXPLAIN <table_name>
+-------+-------------+------+-----+---------+-------------------+
| Field | Type        | Null | Key | Default | Extra             |
+-------+-------------+------+-----+---------+-------------------+
| id    | int         | NO   | PRI | NULL    | auto_increment    |
| name  | varchar(50) | NO   | UNI | NULL    |                   |
| date  | timestamp   | YES  |     | now()   | DEFAULT_GENERATED |
+-------+-------------+------+-----+---------+-------------------+
 
-- Show create statement that was used to crete the table 
SHOW CREATE TABLE <table_name>;
 
| Table  | Create Table                                                            orders | CREATE TABLE `orders` (
  `id` int NOT NULL AUTO_INCREMENT,
  `order_sum` int NOT NULL,
  `user_id` int NOT NULL,
  PRIMARY KEY (`id`),
  KEY `user_id` (`user_id`),
  CONSTRAINT `orders_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `user` (`id`) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci 
```
---
#### Use MySQL from Python with `mysql-connector-python`
```python
# Installation
(venv)$ pip install mysql-connector-python

# Create connection
from contextlib import contextmanager
from mysql import connector

def connect_to_mysql(config):
	try:
		con = connector.connect(**config)
		yield con
	except Exception as err:
		logger.error(...)
		con = None
		raise
	finally:
		if con:
			con.close()

config = {"user": "username", "database": "db_name", "password": "passw"}

def make_query(query: str, args: tupe[Any] | dict[str, Any]):
	with connect_to_mysql(config) as con:
		with con.cursor() as cursor:
			result = cursor.execute(query, args)
			for item in result:
				print(item) # return tuples of data

""" Query examples """
# Select all rows at once with `cursor.fetchall`
with con.cursor() as cursor:
	query = "SELECT * FROM mytable WHERE id > %s AND salary > %s"
	cursor.execute(query, (1, 50000)) # id=1, salary=50000
	res = cursor.fetchall()
	for row in cursor:
		print(row) # (col1, col2, col3) ... many times

# Select only N rows with `cursor.fetchmany`
with con.cursor(buffered=True) as cursor:
# Without `buffered` mysql will raise error, if not all rows processed
	query = "SELECT * FROM mytable WHERE id > %s AND salary > %s"
	cursor.execute(query, (1, 50000)) # id=1, salary=50000
	first_five = cursor.fetchmany(5) # make batches with 5 rows in each
	for row in first_five:
		print(row) # (col1, col2, col3) ... 5 times

	# You can query multiple times, while there are unprocessed rows in buffer
	next_six = cursor.fetchmany(6)
	for row in next_six:
		print(f"{row[0]}-{row[1]}-{row[2]}") # col1-col2-col3 ... 6 times  
		
# Select one row with `cursor.fetchone`
with con.cursor() as cursor:
	query = "SELECT * FROM mytable WHERE id = %s"
	cursor.execute(query, (1,))
	item = cursor.fetchone()
	print(one) # (col1, col2, col3)

# Return rows as dictionaries instead of tuples
with con.cursor(buffered=True, dictionary=True) as cursor:
	query = "SELECT * FROM mytable WHERE id > %s AND salary > %s"
	cursor.execute(query, (1, 50000)) # id=1, salary=50000
	first_five = cursor.fetchmany(5) # make batches with 5 rows in each
	for row in first_five:
		print(row) # {col1: val1, col2: val2, col3: val3} ... 5 times
		print(row["col1"], row["col2"]) # work with keys, not idexes
		
# Return rows as namedtuples (deprecation warning!)
with con.cursor(buffered=True, named_tuple=True) as cursor:
	query = "SELECT * FROM mytable WHERE id > %s AND salary > %s"
	cursor.execute(query, (1, 50000)) # id=1, salary=50000
 	first_five = cursor.fetchmany(5) # make batches with 5 rows in each
	for row in first_five:
		print(row) # Row(col1=val1, col2=val2, col3=val3) ... 5 times
		print(row.col1, row.col2) # work with attributes, not indexes

# Provide named query arguments
with con.cursor(buffered=True) as cursor:
	# Name your column, enclose it in parens, prefix with `%`, append `s`
	# like %(id)s, %(name)s, %(created_at)s
	query = "SELECT * FROM mytable WHERE id > %(id)s AND salary > %s"

	# Supply dictionary with keys names as columns in query
	cursor.execute(query, {"id":1, "salary":50000})
	first_five = cursor.fetchmany(5) # make batches with 5 rows in each
	...

# Make insert query. MySQL does not support returning
with con.cursor() as cursor:
	query = "INSERT INTO mytable (name, salary) VALUES (%s, %s)"
	cursor.execute(query, ("name", 10000)) # return None
	# OR
	query = "INSERT INTO mytable (name, salary) VALUES (%(name)s, %(salary)s)"
	cursor.execute(query, {"name": "name", "salray": 10000}) # return None

	con.commit()
	print(cursor.rowcount) # => 1, how many rows were affected
	print(cursor.lastrowid) # => 10, id of the last inserted row. Works only
	                       # with AUTO_INCREMENT PKs  

# Insert many with `cursor.executemany`
with con.cursor() as cursor:
	query = "INSERT INTO mytable (name, salary) VALUES (%s, %s)"
	cursor.executemany(query, [("name1", 10000), ("name2", 20000)])
	con.commit()
	print(cursor.rowcount) # => 2, 2 rows were inserted	

# Update query
with con.cursor() as cursor:
	query = "UPDATE mytable SET salary = %s WHERE name LIKE 'name%'"
	cursor.execute(query, (0,))
	con.commit()
	print(cursor.rowcount) # => 1, one row affected
	print(cursor.lastrowid) # => 0, MySQL can't retrieve last updated id

# Delete query
with con.cursor() as cursor:
	query = "DELETE FROM mytable WHERE name LIKE 'name%'"
	cursor.execute(query
	con.commit()
	print(cursor.rowcount) # => 2, two rows deleted 
	print(cursor.lastrowid) # => 0, MySQL can't retrieve last deleted id
```
---
#### Types of keys in MySQL (`MUL`, `KEY` and so on)
- `MUL` key means either this column has an index or this column references other table's primary key. `MUL` does not imply `UNIQUE` or `NOT NULL` constraints on tables. `MUL` is shown when you describe a table with `DESC`.

- Take this example, where we don't create any key.
```mysql
>>> CREATE TABLE users (id int);
>>> desc users; -- The Key column is empty
... 
+-------+---------+------+-----+---------+-------+
| Field | Type    | Null | Key | Default | Extra |
+-------+---------+------+-----+---------+-------+
|   id  | int(11) | YES  |     | NULL    |       |
+-------+---------+------+-----+---------+-------+
```

- Now we create a `MUL` key via creating ad index 
```mysql
>>> CREATE TABLE users (id int, index(id));
>>> desc users; 
... 
+-------+---------+------+-----+---------+-------+
| Field | Type    | Null | Key | Default | Extra |
+-------+---------+------+-----+---------+-------+
|   id  | int(11) | YES  | MUL | NULL    |       |
+-------+---------+------+-----+---------+-------+
```

- And if we would have two tables one of which would reference the other's `PK` field, we would see the `MUL` key again.
```mysql
>>> CREATE TABLE users(id int PRIMARY KEY);
>>> CREATE TABLE accounts(id int, FOREIGN KEY(id) REFERENCES users(id));
>>> desc users; -- users have primary key on its `id` column
... 
+-------+---------+------+-----+---------+-------+
| Field | Type    | Null | Key | Default | Extra |
+-------+---------+------+-----+---------+-------+
|   id  | int(11) | YES  | PRI | NULL    |       |
+-------+---------+------+-----+---------+-------+
>>> desc accounts; -- accounts have `MUL` key on its `id` column
... 
+-------+---------+------+-----+---------+-------+
| Field | Type    | Null | Key | Default | Extra |
+-------+---------+------+-----+---------+-------+
|   id  | int(11) | YES  | MUL | NULL    |       |
+-------+---------+------+-----+---------+-------+
```

- `KEY` without `PRIMARY` or `FOREIGN` is nearly a synonim for `index`.
```mysql
>>> ALTER TABLE users ADD COLUMN uin int;
>>> ALTER TABLE users ADD KEY(uin);
--- OR WE CAN
>>> CREATE INDEX uin_idx ON users(uin);
>>> desc users;
+-------+-------------+------+-----+---------+-------------------+
| Field | Type        | Null | Key | Default | Extra             |
+-------+-------------+------+-----+---------+-------------------+
| id    | int         | NO   | PRI | NULL    | auto_increment    |
| name  | varchar(50) | NO   | UNI | NULL    |                   |
| date  | timestamp   | YES  |     | now()   | DEFAULT_GENERATED |
| uin   | int         | NO   | MUL | 0       |                   |
+-------+-------------+------+-----+---------+-------------------+
```
---
#### Read current session's settings
```mysql
SELECT @@session.sql_mode;
+--------------------------------------------------------------------+
| @@SESSION.sql_mode                                                 |    
+--------------------------------------------------------------------+
| STRICT_TRANS_TABLES,STRICT_ALL_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,|
| ERROR_FOR_DIVISION_BY_ZERO,TRADITIONAL,NO_ENGINE_SUBSTITUTION      |
+--------------------------------------------------------------------+
1 row in set (0.01 sec)
```
---
#### Show active connections
```mysql
SHOW FULL processlist;
```
---
#### Kill a process
```mysql
KILL <process_id>;
```
---
#### Show current session's warnings
```mysql
SHOW WARNINGS;
```
---
#### Get a character from its ASCII numeric value
```mysql
-- The CHAR function takes multiple numeric values and returns a string. You need  to explicitly provide character set to get values from. Otherwise the result will be in hexadeciaml encoding.
SELECT CHAR(67,72,65,82 USING ASCII)
-- OR
SELECT CHAR(67,72,65,82 USING utf8mb4)
=> 'CHAR'
```
---
#### `POSITION`function to find first substring in a string and return its start index with
```mysql
-- The POSITION function takes an expression in form of {substring} IN {string} and returns the starting index of the substring in string. If no match found, 0 is returned.
SELECT POSITION('hello' IN 'hello world'); -- => 1, indexing is 1-based
SELECT POSITION('world' IN 'hello world'); -- => 7
SELECT POSITION('not_found' IN 'hello world'); => 0
```
---
#### `LOCATE` function to find substring in a string from arbitrary index and return substring's start index
```mysql
-- LOCATE works similar to `POSITION` but it lets you to begin substring search from arbitrary index.
SELECT LOCATE('hello', 'hello world'); => 1, indexing is 1-based
SELECT LOCATE('hello', 'hello world hello', 3); => 13, first substring skipped
SELECT LOCATE('not_found', 'hello world'); => 0
```
---
#### `STRCMP` function to compare strings
```mysql
-- STRCMP recieves two strings and return 0 if strings are equal, 1 if second string is preceeding the first in lexicographical order and -1 otherwise. It is case insensitive.
SELECT STRCMP('hello', 'hello'); => 0, strings equal
SELECT STRCMP('hello', 'HELLO'); => 0, strings equal
SELECT STRCMP('hello', 'aaa'); => 1, 'aaa' preceeds 'hello'
SELECT STRCMP('hello', 'zzz'); => -1, 'zzz' goes after 'hello'
```
---
#### `INSERT` function to modify strings
```mysql
-- INSERT modifies a string by inserting a a new string into current one. It is also possible to substitute characters by string inserted.
-- You should provide target string, index at which to insert new string, number of characters to replace in target string and the string to insert.
SELECT INSERT('hello world', 6, 0, ' happy'); => 'hello happy world', the new substring was inserted at index 6 of the old string, no characters substituted
 
SELECT INSERT('hello world', 1, 5, 'goodbye'); => 'goodbye world', the new substring was inserted at index 1 of the old string, and 5 characters were replaced with this string.
 
SELECT INSERT('hello world', 1, 5, ''); => 'world', we replaced first 5 characters with an empty string.
```
---
#### `SUBSTRING` function to create substrings
```mysql
-- SUBSTRING recieves target string, target string's index to start substring from and the length of the substring.
SELECT SUBSTRING('massachusetts', 1, 5); => 'massa'
SELECT SUBSTRING('massachusetts', 9, 3); => 'set'
SELECT SUBSTRING('massachusetts', 1); => 'massachusetts'
-- Start index may be negative. This way -1 is the last char's index, -2 is prelast and so on.
SELECT SUBSTRING('massachusetts', -5); => 'setts'
SELECT SUBSTRING('massachusetts', -5, 3); => 'set'
```

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
#### Current session `timezone`
```sql
SELECT @@SESSION.time_zone;
+---------------------+
| @@session.time_zone |
+---------------------+
| SYSTEM              |
+---------------------+

-- Change the timezone
SET @@SESSION.time_zone = '+03:00' -- set session time to Moscow time
```
---
#### `EXTRACT` function to extract datetime parts
```sql
-- You can extract different parts from datetime string such as year, month, hours ...
SELECT EXTRACT(YEAR FROM '2023-01-01'); => 2023
SELECT EXTRACT(HOUR FROM '2024-01-01 20:00:59'); => 20

-- String must be ISO formatted
SELECT EXTRACT(HOUR FROM 'JAN, 01 2024 20:00:59'); => NULL
```
---
#### Mutate a table values (update, delete) while querying it
```sql
-- MySQL does not allow to mutate a table simulataneously with querying itself.
-- For example this query will produce `You can't specify target table for update in FROM clause` error.
DELETE FROM table1
	WHERE id IN (
		SELECT t1.id
		FROM table1 t1
			INNER JOIN table2 t2 ON t1.id = t2.ref_id
			WHERE t1.column1 = 'value1' AND
				  t2.column3 = 'value3' AND
				  t2.column8 = 'value8'	
	)

-- To overcome this you can wrap your subquery with another 'dummy' select subquery. Wrapping the subquery will cause MySQL server to implicitly create a temproray table, which will avoid the error, since technically you're not querying the same table you're modifying.
-- So what you need is a little add on.
DELETE FROM table1
	WHERE id IN (
		SELECT * FROM ( -- this was added
			SELECT t1.id
			FROM table1 t1
				INNER JOIN table2 t2 ON t1.id = t2.ref_id
				WHERE t1.column1 = 'value1' AND
					  t2.column3 = 'value3' AND
					  t2.column8 = 'value8'	
		) AS subq      -- this was added
	);
```
---
#### MySQL Shell (mysqlsh)
```bash
# Run mysqlsh in sql mode. (from CLI, cmd or shell)
mysqlsh --sql

# Switch to js and back to SQL
\js
\sql

# Connect to database.
\connect user@host
\connect root@localhost
# ... you will be prompted to enter your password

```
---
#### Reset root password (Windows)
```bash
# Prepare a file with sql statement for updating the password (named `mysql-init.txt` for example). Store in the root C: path.
ALTER USER 'root'@'localhost' IDENTIFIED BY 'password';

# Deactivate mysqld daemon
Open start panel -> enter `services` -> find `mysqld` service on the page -> click stop

# Navigate to the path where MySQL Server is installed (for me its C:\Program Files\MySQL\MySQL Server 8.0).
# Delete all files from the `data` directory. If there is no such directory - craete and empty one.
# Open `cmd` as administrator and navigate to where the mysql executables located.
cd "C:\Program Files\MySQL\MySQL Server 8.0\bin"

# Run
mysqld --install
mysqld --initialize
mysqld --init-file=C:\\mysql-init.txt --console

# This will install mysql daemon, initialize with a fresh install data, and then additionaly initialize it with commands from the prepared file.

# Now open second cmd instance and type
mysql -u root -p
# ... enter your new password when prompted
```
---
