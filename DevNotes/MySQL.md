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
#### Change user password
```sql
mysql> ALTER USER 'user_name'@'localhost' IDENTIFIED BY 'NEW_USER_PASSWORD';
mysql> FLUSH PRIVILEGES;

-- OR
mysql> UPDATE mysql.user SET authentication_string = PASSWORD('NEW_USER_PASSWORD') WHERE User = 'user_name' AND Host = 'localhost';
mysql> FLUSH PRIVILEGES;
```
---
#### mysql CLI
```sql
-- Run mysql CLI
$ mysql -u user_name -D db_name -p

-- Show databases;
SHOW DATABASES; -- show all databases
SHOW DATABASES LIKE 'patter'; -- show specific databases

-- Connect to database
USE <db_name>;


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
