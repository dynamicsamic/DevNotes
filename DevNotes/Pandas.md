#### Read files quick reference
```python
from pathlib import Path
import pandas as pd

WORKDIR = Path(__file__).parent

# Most popular pandas functions for reading different file types
read_excel, read_csv, read_xml, read_json, read_sql, read_parquet, read_pickle, read_feather, read_html

# Read parquet file
df = pd.read_parquet(WORKDIR / 'tables/table1.parquet')
print(df)

# Read csv file
df = pd.read_csv(WORKDIR / 'tables/table2.csv', sep=';')
print(df)

# Read xml file 
df = pd.read_xml(WORKDIR / 'tables/table3.xml', parser="etree")
print(df)

# Read excel file
df = pd.read_excel(WORKDIR / 'tables/table4.xlsx')
print(df)

# Read only the first and second sheets
df = pd.read_excel(WORKDIR / 'tables/table4.xlsx', sheet_name=[0,1])

# Read only named sheets 
df = pd.read_excel(WORKDIR / 'tables/table4.xlsx', sheet_name="first_sheet")

# Read mixed named and positional sheets
df = pd.read_excel(WORKDIR / 'tables/table4.xlsx', sheet_name=[0, "second_sheet"])

# Print only specific sheets
print(df[0])
print(df["second_sheet"])

# Excel sheets often contain complex column layout with horizontal and vertical headers.
# For example you can have this kind of layout
1	                       высшее          среднее        
2	                       город деревня   город деревня
3	молеже 35 лет мужчины     12      34       2      14
4	              женщины     32      34       4      14
5	страше 35 лет мужчины      6      23       1       7
6	              женщины     11      22       0       8

# Here the first two COLUMNS are virtical header, meaning they don't contain data, they serve more like additional shape to the column layout.
# On the other hand the first two ROWS are horizontal headers, they shape the layout in its own way.

# Sometimes source files may contain non utf-8 encoding.
# To overcome this you can either provide a value to `encoding` parameter, or if you don't know the encoding beforehand provide value ro `encoding_errors`.
df = pd.read_csv('file.csv', sep=";", encoding_errors='ignore')
# Malformed values will be set to NaN, and you can filter them out or replace.
df.info()

# In pandas read_csv method virtical headers can be 
```
---
#### Limiting output data
```python
import pandas as pd

df = pd.read_xml('path/to/file')

# Return a dataframe copy limited to 5 rows
df.head()

# You can set manual limit on head() method
df.head(2)

# Return single column (Series object)
df["product_name"]

# Return multiple columns (DataFrame object)
df[["product_name", "brand"]]

# Return single row (Series object) by its index position
df.iloc[0]

# Return a range of rows (DataFrame object)
df.iloc[1:3]

# Return arbitrary number of rows by passing a list of column nums
df.iloc[[4,1,3]]

# Return a range of rows with step (DataFrame object)
df.iloc[1:5:2]

# Return specific column (Series object)
df.iloc[:, 1] # the rist column with all values
# OR
df.loc[:, "product_name"]

# Return arbitrary number of columns
df.iloc[:, [1,4,3]]
df.loc[:, ["product_name", "brand"]]

# Return a range of columns
df.iloc[:, 1:4:2]
df.loc[:, "product_id":"purchase_date"]

# Return specific columns with specific rows (DataFrame object)
df.iloc[1:3, ::2]
# OR
df.loc[1:3, "product_name": "product_id"] # product_id will be included

# Return scalar value of a cell
df.iloc[0,0]

# You can convert a Series object to DataFrame
df["product_name"].to_frame()
```
---
#### Manipulating columns and values
```python
import pandas as pd

df = pd.read_xml('path/to/file')

# Update column values (inplace)
df["amount"] = df["amount"] + 1

# Add a new column with single value for every row (inplace)
df["new_col"] = "NewVal"

# Add a new column with different values (inplace).
# Number of values should match number of rows
df["new_col"] = ["one", "two", "three"]

# Use Series object for adding new column inplace.
# Series object values will be matched to dataframe rows by its index
df["new_col"] = pd.Series(["one", "two", "three"])

# Add custom index to series object to map new values to different rows
df["new_col"] = pd.Series(["one", "two", "three"], index=[2, 0, 1])

# If you don't create series object yourself and not sure, if it has the right index, you can reset it, to map its values to dataframe rows from the first row
df["new_col"] = series_obj.reset_index(0)

# Or you can convert the series object to python list with `to_list` method
df["new_col"] = series_obj.to_list()

# You can avoid the original data mutation by creating a copy of a dataframe or a column
new_df = df.copy()
col = df["name"].copy()

# A more idiomatic way to add columns is to use dataframes's assing method.
# It can both create new columns and overwrite old ones. It does not mutate the dataframe inplace, returning an updated copy of the old dataframe.

df.assign(another_col = 1) # add new column, named `another_col` with single value = 1 assigned to all rows.

# Add column with multiple values
df.assign(another_col = [1,2,3,6,4])

# You can reference the dataframe in assing method. It is usually accomplished using lambda functions. Lambda functions receive dataframe as argument.
df.assing(another_col = lambda frame: frame["new_col"].str.lower())

# You can chain assigns to mutate columns step by step. This may be useful, when you want to copy a column with rename that uses non-ascii characters and then modify it.
(
 df
	 .assign(new_col=lambda frame: frame["Новая колонка"])
	 .assign(another_col=lambda frame: frame["new_col"].str.lower())
)

# Chaining assignments with rename is also possible
(
 df
	 .assign(new_col=[1,2,3,4])
	 .rename({"new_col": "Новая колонка"}, axis="columns")
)

# Conditional assignment also possible.
# Here we use numpy's `.where` method. This method receives a condition in a form of sequence of values (usually a 1 or more dimentional vectors), a value or sequence of values for rows satisfying condition and a value or a sequence of values for those not satisfying the condition. 
# `.where` also produces a sequence (vector), thats why pandas dataframes and series coopearte with it greatly.
# The main constraint when using the `.where` method - all arguments must of the same dimention. It means that when you work with two columns, you need to provide a list or tuple of two values. 
# In this example we add new column named `another_col` to dataframe and assign it values depending on the the `new_col` values.
import numpy as np
df.assign(
	another_col=lambda frame: np.where(
		frame["new_col"] == "hello world", # a condition
		"greeting",                     # value when condition true
		"something else"                # value when condition false
	)
)

# We can use dataframe columns in for all parameters in `.where` method.
df.assign(
	bonus=lambda frame: np.where(
		frame["evaluation_mark"] > 6,
		frame["salary"] * 0.2,
		frame["salary"] * 0.01
	)
)

# Conditional assignment with np `.where` and `.mask` methods are a less rich.
df.assign(
	bonus=lambda frame: frame["salary"].where(
		frame["evaluation_mark"] > 6, # condition to evaluate
		frame["salary"] * 0.2,        # update values if condition is false
	)
)

df.assign(
	bonus=lambda frame: frame["salary"].mask(
		frame["evaluation_mark"] > 6, # condition to evaluate
		frame["salary"] * 0.2,        # update values if condition is true
	)
)

# A more advanced functionality for conditional assignment is provided by the `np.select` function.
# np.select receives a sequence of conditions, a sequence of values for each condition to be used when the condition is true, and a default value to be used when none of condtitions is true.
# We add a rank column depending on the score column. If the score does not match any of the conditions, it means it is higher than 90, then we assign 'grand master' to it
df.assign(
	rank=lambda frame: np.select(
		[frame["score"] < 20, frame["score"] < 50, frame["score"] < 90],
		["beginner", "medium", "pro"],
		"grand master"
	)
)


# It is also possible to replace columns with loc and iloc dataframe methods.
# Additionally the loc method will create a new column, if it does not exist.
df.loc[:, 'new_col'] = [1,2,3,4]
df.iloc[:, -1] = [1,2,3,4]

# You can update a column value by accessing it with square brackets [].
# In this case a warning would arise signaling that you have mutated the dataframe inplace.
df["amount"][0] = 1

# A better way to acomplish this task is to use the loc or iloc dataframe methods
df.loc[1, "amount"] = 1
df.iloc[1, 6] = 1

# Drop a column. You can use either of these method calls
df.drop(columns="new_col")
df.drop("new_col", axis="columns")
df.drop("new_col", axis=1)

# Drop mulitple columns
df.drop(columns=["new_col", "another_col"])
df.drop(["new_col", "another_col"], axis="columns")
df.drop(["new_col", "another_col"], axis=1)

# Drop duplicate rows with `.drop_duplicates()` method.
# Drop rows if entire row is duplicate.
df.drop_duplicates()

# Drop rows if `col1` column has duplicate values
df["col1"].drop_duplicates()
# OR
df.drop_duplicates("col")

# Drop duplicate rows keeping ones that come last
df.drop_duplicates(keep='last')

# Drop both original and duplicate rows
df.drop_duplicates(keep='last')
```
#### Increase csv file processing with `pyarrow`
```python
import pandas as pd

# It is 2-6 times faster than default C implementation, but uses additional dependency, which can be installed wtih `pip install pyarrow`  
df = pd.read_csv('path/to/file', sep=',', engine="pyarrow")
```
---
#### Dataframe filtering and comparison
```python
import pandas as pd

df = pd.read_xml(...)

# Dataframes support different comparison methods. You can use builtin python operators, like >, <, == and others as well as specific methods like .isna or .notna

# Find if a role column contains values that equal to "developer"
df["role"] == "developer"

# The above code returns a series object with two columns - 1)index and 2)boolean result of comparison every column row value to scalar value "developer".
0        True
1        True
2        True
3        True
4        True
         ... 
29367    True
29368    True
29369    True
29370    True
29371    True
Name: role, Length: 29372, dtype: boolean

# You then can take this series object and use its index and True values as indexes of valid rows. You need pass the series as index of the dataframe.
df[df["role"] == "developer"]
	employee_id    role        ...
0   1              "developer" ...
2   123            "developer" ...
14  2345           "developer" ...
22  94             "developer" ...
...

# All builtin comparison operators have their analougs as dataframe methods.
# I.e.
df["role"].eq("developer")
df["salary"].gt(123_445)

# You can also use the `not` operator to negate the result by prefixing your expression with `~` sign.
# The expression below reads as `role not eqaul to 'developer'`
df[~df["role"] == "developer"]

# To compare a date value to a date column you need to wrap your value in quotes
df["report_date"] > '2022-01-01'

# It is also possible to compare date and datetime columns with date and datetime objects. This way the code is a bit more involved.
# If your datetimes are timezone aware (i.e. `2024-11-07 23:46:56.243000+00:00`) comparing it to timezone naive datetimes will result in error. So you need to handle this issue manually.
df['report_date'] >= pd.Timestamp("2024-11-08", tz="UTC")

# You may need to convert the 'date' column to datetime type first
df['report_date'] = pd.to_datetime(df['report_date'])

# You can also filter <datetime> column by <date> if the time part is not relevant to you
import datetime as dt
df['date'].dt.date >= dt.date(2024, 11, 8)

# To check if a container contains values from a dataframe column the .isin() method used.
df["role"].isin(["developer", "admin", "product manager", "sales manager"])

# The `not in` variant. 
df[~df["role"].isin(["product manager", "sales manager"])]

# Trying to check if column contains a value with builtin python `in` operator results in dataframe checking a value among index.
"developer" in df["role"] # This returns false, if there is no such value in the index.
0 in df["role"] # This returns true, because 0 index exists in the dataframe (it' the first row) 

# To perform search with python builtin `in` operator, you can convert column to list.
"developer" in df["role"].to_list()

# String columns have .str attribute that provides multiple convinient methods that simulate python builtin string methods.
# You can use str.contains to check if a row value contains a substring.
df[df["status"].str.contains("cancel")]

# The `not contains` option.
df[~df["status"].str.contains("wait")]

# Check if values start with substring
df[df["status"].str.startswith("ca")]

# Check if values have prefix
df[df["status"].str.isalpha()]

# To find rows with missing values (such as Null, pd.NA, np.nan) use the `.isna` method. All null rows will be marked as True. This method returns a series object with index and boolean column, that can be used with main dataframe to return only truthy rows.
df["age"].isna()

# Find rows that have not null values with `.notna` method. All not-null rows will be marked as True.
df["age"].notna()

# You can also use `.isnull` and `.notnull` aliases
df["age"].isnull()
df["age"].notnull()

# This four methods are also implemented as pd methods
pd.isna(df["age"])
pd.notna(df["age"])
pd.isnull(df["age"])
pd.notnull(df["age"])

# Find duplicate rows in a dataframe or a column with `.duplicated()` method.
# By default it scans the dataframe/series object from top to bottom and marks every duplicate row as True. If you set the `keep` argument to 'last', it will start scanning from the bottom, thus marking higher values as duplicates. If you set `keep` parameter to False, this will mark both original and duplicate columns as duplicates.

# Select complete dataframe duplicate rows.
df[df.duplicated()]

# Select duplicate rows only within `salary` column.
df[df["salary"].duplicated()]
# OR
df[df.duplicated("salary")]

# Select duplicate rows within `salary` and `age`, mark the last ones as unique and the first ones as duplicates (here row is a combination between values from both columns)
df[df.duplicated(["salary", "age"], keep='last')]

# Select duplicate rows within `salary` and `age`, mark both oroginal and duplicate rows as duplicates.
df[df.duplicated(["salary", "age"], keep='last')]

# If you have homogeneous data it maybe usefull to compare the whole dataframe against a value.
# For example, if you have table with student marks, you can compare every column's row to 5.
marks = pd.read_excel('path/to/file')
(marks.loc[:, "math":"reading"] == 5).head()

# Exhaustive and short-circuit comparisons with `.all` and `.any` methods.
# `.all()` returns true if all values in one or many columns satisfy some condition.

# Say we want to know if all students have 4's and 5's marks for all disciplines.
marks.loc[:, "math":].isin([4,5]).all()
math       True
language   True
reading    True
painting   True
history    True
dtype: bool

# We can narrow the scope and choose only some students and some disciplines 
marks.loc[6:8, ["math", "history"]].isin([4,5]).all()
math       True
history    True
dtype: bool

# `.any()` method returns true if at least of the values in one or many columns satisfy some condition.
marks.loc[:, "math":].isin([4,5]).any()

# Instead of evaluating the whole dataframe with `.all` method, you can evaluate single rows if you set the `axis` parameter = 1.
# Here True marks a row having in all its columns only 4 and 5.
marks.loc[:, "math":].isin([4,5]).all(axis=1)
0      False
1       True
2       True
3       True
4      False
       ...  
140     True
141     True
142     True
143     True
144     True
Length: 145, dtype: bool

# Comparing columns between each other is also possible, since it's pd.Series objects.
marks["math"] == marks["physics"]

# Compound comparison is also possible with `&` and `|` operators.
# In standard python these operators are used for bitwise operations. Their priority (order of precedence) is higher than comparison operators. Since pandas uses operator overloading, operator priority does not change, and `&` in dataframe filtering will be evaluted before the `==` operator.
# For example, in a comparison like 
marks[marks["math"] == 5 & marks["physics"].notnull()]
# the first thing that would be evaluated is the `marks["physics"].notnull()` part; after you will make `and` operation with `5`, which will is a truthy value, so the result would not change; and the last part would be comparing the `math` column to `physics` which will produce unexpected result.
# Always isolate your comparisons with parens.
marks[(marks["math"] == 5) & marks["physics"].notnull()]
marks[(marks["math"] == 5) & (marks["physics"] == 4)]
marks[(marks["math"] == 5) | marks["reading"].notnull()]

# Comparison and filtering is also possible with the `.query()` method. It receives a comparison string, evaluates it and returns result as if was constructed using other dataframe comparison mechanics.
# This method returns the dataframe, not series object. No need to use original dataframe indexing.
marks.query("math == 5")
# is equivalent to 
marks[marks["math"] == 5]

# Date example with `.query` method
df.query("report_date == '2022-12-31'").head(3)

# You can use standart python `and`, `or`, `not` and `in` operators
df.query('(report_dt >= "2021-12-31") and (pos_name == "Кассир")').head(3)
marks.query('Математика not in [3, 4]').head(3)

# You can chain multiple .query methods
(df
    .query('report_dt >= "2021-12-31"')
    .query('pos_name == "Кассир"')
    .head(3)
)

# If your column names contain spaces surrond those with backticks
marks.query('`Физическая культура` == 5').head(3)

# It is also possible to use variables in the query expression
students = [
    "Бычкова Александра Борисовна",
    "Кулешов Семен Богданович",
    "Павловскиая Алена Данииловна",
    "Курочкин Андрей Семенович",
    "Лапин Юрий Тимурович"
]
marks.query("ФИО in @students")
# OR using f-strings
marks.query(f"ФИО in {students}")

# The same is possible with dates
date = '2015-01-31'
df.query('report_dt >= @date')

# When using dates with f-strings in query, you need to enclose the date variable in quotes (single or double) to hint pandas that this is variable should be parsed as string.
date = '2015-01-31'
df.query(f'report_dt == "{date}"')

# If you want to use the `.query` method with Series object, first convert it into dataframe.
pd.Series(list(range(10))).to_frame('nums').query("nums in [1,2,3]")

# It is also possible to filter rows and columns with `.loc` method passing lambda functions in it.
# Here we select only rows that are present once (does not have duplicates) 
df.loc[~lambda dframe: dframe[["name", "position"]].duplicated(keep=False)]

# Conditional filtering with dataframe `.where` and `.mask` methods.
# These methods workd similar. They receive a condition (usually a column) and a value or values. 
# `.where` method uses the value(s) when condition is false, and if condition is true values from original column are used.
# `.mask` method uses the value(s) when condition is true, and if condition is false it uses values from original column.
# In this example we update salary column depending on position name. If position name equals to "Директор магазина", we leave it as is, otherwise we raise the salary by 20%. 
empl["salary"].where(
	empl["pos_name"] == "Директор магазина", 
	empl["salary"] * 1.2
)
# Scalar value can also be used.
empl["salary"].where(empl["pos_name"] == "Директор магазина", 100000)

# With mask we update only those who have position name == "Директор магазина". Others will keep their values.
empl["salary"].mask(
	empl["pos_name"] == "Директор магазина", 
	empl["salary"] * 1.2
)
# Scalar value can also be used.
empl["salary"].mask(empl["pos_name"] == "Директор магазина", 100000)
```
---
#### Concatenating two dataframes
```python
df1 = pd.concat([df1, df2], axis=0)

# Resulting dataframe will probably contain non unique index value. This will prevent you from saving you dataframe into a file resulting in error: ValueError: DataFrame index must be unique for orient='columns'.
# To overcome this you can:
# - drop index column at all, if you don't need it.
df1.reset_index(drop=True, inplace=True)

# - reset the index to recalculate it from scratch
df.reset_index(inplace=True)
```
---
#### Working with values
```python
import pandas as pd
import numpy as np

df = pd.read_xml('path/to/file')

# Find the minimum value from every row of 3 columns
np.minimum(df["col1"], df["col2"], df["col3"])

# Find maximum value from every row of 3 columns
np.maximum(df["col1"], df["col2"], df["col3"])

# Find minimum value from row of a column and a constant value
np.minimum(df["col1"], 295)

# Find minimum and maximum with pd.series.min and max methods
df["col1"].min()
df["col2"].max()

# Return a constant instead of lower values
df["col1"].clip(10_000) # return 10_000 for every row value lower than 10_000

# Return constants instead of lower and higher values
df["col1"].clip(10_000, 50_000) # return 10_000 for every row value lower than 10_000 and 50_000 for every row value higher than 50_000
```
---
#### Mathematical operations
```python
# Mathematical operations in pandas are vectorized, which means that operations are applied on whole series or dataframes not individual cells.
df = pd.DataFrame(
    {
        "names": ["joe", "bob", "helen", "gill", "alice"],
        "math": [2, 4, 4, 5, 3],
        "history": [4, 3, 5, 5, 4],
    }
)

# Numeric columns support all python mathematic operators.
df["math"] + 10 # Add 10 to all rows in `math` columns
df["math"] - 10 # Subtract 10 to all rows in `math` columns
df["math"] * 2 # Multiply all rows in `math` column by 2
df["math"] / 2 # Divide all rows in `math` column by 2
df["math"] // 2 # Divide without remaining all rows in `math` column by 2
df["math"] ** 2 # Square all rows in `math` column
df["math"] % 2 # Find division remaining by 2 for all rows in `math` column

# String columns support `+` and `*` math operators.
df["names"] + ' student' # Add ' student' to all strings in `name` column
df["names"] * 3 # Copy all strings in `name` column 3 times

# You can apply mathematic operations to the whole dataframe.
df * 2 # Numeric values will be doubled, string values will be copied 2 times

# Or sub dataframes
df.loc[:, "math": "history"] * 2
df.iloc[1:3, 1:] // 2

# OR use multiple columns
df[["math", "history"]] // 2

# OR use derived dataframe
df.drop(columns=["names"]) // 2

# Columns can be added together to produce new series object
df["math"] + df["history"]

# If you have empty value in you column (NaN, None...) it won't be coerced to 0 and the result of mathematical operation would be empty.
df["math"] + pd.Series([None, 1, 2, 3, 4])
0    NaN  # Wasn't coerced to 0
1    5.0
2    6.0
3    8.0
4    7.0
dtype: float64
# To overcome this behaviour you should explicitly convert empty values with `fillna()` method. Pass a value (in this case it's 0)
df["math"] + pd.Series([None, 1, 2, 3, None]).fillna(0)
0    2.0
1    5.0
2    6.0
3    8.0
4    7.0
dtype: float64

# The same rule workd when dividing empty values. Dividing a value by empty value or dividing an empty value by other value results in NaN. Dividing by 0 results in `inf`.
print(pd.Series([None, 10, 12]) / 2) # [NaN, 5.0, 6.0]
print(pd.Series([None, 10, 12]) / 0) # [NaN, inf, inf]

# Important note when adding dataframe and series objects - their indexes should be the same to produce predictable results.
# Here we add two series objects with opposite values, which will produce 0 series
pd.Series([1,2,3,4]) + pd.Series([-1,-2,-3,-4]) # => pd.Series([0,0,0,0])
# Here we mess up the second series' index which yeilds an unexpected outcome.
pd.Series([1,2,3,4]) + pd.Series([-1,-2,-3,-4], index=[3,2,1,0])
# => pd.Series([0,-1,1,3])

# Integers can not be multiplied to negative powers. To overcome this use `astype(float)` method.
df["math"].astype(float) ** -2

# Dataframe operations with lists are applied in this way: the first item in list is applied to the first column, second item - to second column and so on. Number of items in list and number of columns in the dataframe must be equal.
df = pd.DataFrame(
	{"col1": range(0, 5), "col2": range(100, 105), "col3": range(1000, 1005)}
)
df + [1, 10, 100]

   col1  col2  col3
0     1   110  1100
1     2   111  1101
2     3   112  1102
3     4   113  1103
4     5   114  1104

# Mixing dataframes and series in one operation produces unexpected results. This happens because of index conflict, but I did not dig enough to explain this.
# To add a flat series object to a dataframe use `to_numpy().reshape(-1, 1)`.
df + pd.Series([10,11,12,13,14]).to_numpy().reshape(-1, 1)
# This way all series elements will be added to every column according to their indexes.
```
#### Matrix operations
```python
```
---
#### Mathematical functions
```python
# Popular numpy functions
import numpy as np
import pandas as pd

# With numpy you can perform vectorized calculations.
np.log(pd.Series([1,2,3,]))
np.sin(pd.Series([1,2,3,]))
np.cos(pd.Series([1,2,3,]))
...

# Get absolute values with `abs` pandas method
pd.Series([1,-2,-3,4]).abs() # [1,2,3,4]

# Round values to N decimal places.
pd.Series([6.22, 1.22344, 4.2, 5]).round(2) # [6.22, 1.22, 4.20, 5.00]

# Round values to N integer places with negative N
pd.Series([201.22, 42231.22344, 9874.2, 58733]).round(-2) # [200.0, 42200.0, 9900.0, 58700.0]

# Round up with `np.ceil` and round down with `np.floor`
np.ceil(pd.Series([6.22, 1.22344, 4.2, 5])) # [7.0, 2.0, 5.0, 5.0]
np.floor(pd.Series([6.22, 1.22344, 4.2, 5])) # [6.0, 1.0, 4.0, 5.0]

# There are also functions that count cumulative results
pd.Series([1, 2, 3, 4]).cumsum() # [1, 3, 6, 10]
pd.Series([1, 2, 3, 4]).cumprod() # [1, 2, 6, 24]
pd.Series([1, 2, 3, 0, 4, -1]).cummin() # [1, 1, 1, 0, 0, -1]
pd.Series([1, 2, 3, 0, 4, -1]).cummax() # [1, 2, 3, 3, 4, 4]

# By default all empty values are ignored during calculation and returned as is
pd.Series([1, None, 3, 4]).cumsum() # [1, NaN, 4, 8]
pd.Series([1, 2, None, 0, 4, -1]).cummin() # [1, 1, NaN, 0, 0, -1]

# You can change this by setting the `skipna` parameter to False.
# This way all results starting from the empty value will also be marked as empty
pd.Series([1, 2, None, 0, 4, -1]).cummin(skipna=False) # [1, 1, NaN, NaN, NaN, NaN]

# This functions can be applied to dataframes if all the columns have numeric or empty values.
pd.DataFrame({"col1": [1, 2, 3, 4], "col2": [6, None, 7, 8]}).cumsum()
   col1  col2
0     1   6.0
1     3   NaN
2     6  13.0
3    10  21.0

# Per-row cumulative calculations are also possible.
pd.DataFrame(
    {
        "col1": [1, 2, 3, 4],
        "col2": [6, None, 7, 8],
        "col3": [100, 10, 20, 1]}
)
.cumsum(axis=1)
   col1  col2   col3
0   1.0   7.0  107.0
1   2.0   NaN   12.0
2   3.0  10.0   30.0
3   4.0  12.0   13.0

# Columns have properties that can tell about value sorting.
# `is_monotonic_increasing` returns true if values are in non-decreasing order.
pd.Series([1, 2, 3, 4]).is_monotonic_increasing # True
pd.Series([1, 2, 3, 1]).is_monotonic_increasing # False
pd.Series([0, 0, 0, 0]).is_monotonic_increasing # True
pd.Series([None, None, None, None]).is_monotonic_increasing # False

# `is_monotonic_decreasing` returns true if value are in non-increasing order.
pd.Series([5, 4, 4, 4, 2]).is_monotonic_decreasing # True
pd.Series([1, 2, 3, 4]).is_monotonic_increasing # False
pd.Series([0, 0, 0, 0]).is_monotonic_decreasing # True

# Python builtin funcitons min, max, sum, len, abs, also work with columns.
max(pd.Series([1, 2, 3, 4])) # 4
len(pd.Series([1, 2, 3, 4])) # 4

# Note that only functions that expect iterables work with series natively.
# For example functions from math module such as `log` or `sin` expect a single value, using series objects with these functions will result in error.
```
---
#### Replacement and transformation functions
```python
import pandas as pd

df = pd.DataFrame(
	{
	"role": ["client", "admin", "tutor", "client"],
	"occupation": ["supplier", "worker", "client", "other"]
	}
)

# Pandas dataframe and series objects have `replace` method that replaces old values with new ones.
# Replace every 'client' value with `user`.
df.replace({"client": "user"})

# Replace multiple old values with multiple new ones by providing list of old values and list of new values.
df.replace(["client", "other"], ["user", "undefined"])

# Preivous notaiton is equivalent to the one below providing dictionary.
df.replace({"admin": "ADMIN", "client":"CLIENT"})

# Replace multiple old values with one new value by providing a list of old values and a value to be used for raplacing every old value.
df.replace(["client", "other"], "undefined")

# The same work for single columns
df["role"].replace({"admin": "ADMIN"})

# Another option for working with single columns is by naming it in dataframe
df.replace({"role":{"client": "user"}})

# A powerful feature of replace can be regex usage
df.replace({"^c.+t&": "user"}, regex=True)

# You can apply replacing or transformation funcion with `map` method.
# Map will apply provided function to all values.
df.map(lambda x: "user" if x == "client" else x) # Replace all occurances of `client` with `user`.

# When applied to Series object map can take a mapping instead of a function.
# In this case all values, that are not in the mapping will be replaced with NaN
df["role"].map({"client": "user"})
0    user
1     NaN
2     NaN
3    user

# To change this behaviour you can use the hack below.
# Here lambda function provides values for the dictionary .get method.
df["role"].map(lambda x: {"client": "user"}.get(x, x))

# A much better choice would be to use a normal lambda function.
df["role"].map(lambda x: "user" if x == "client" else x)

# A somewhat similar function for applying functions to dataframes and columns is `aplly`.
# By default when applied to dataframe it works on by-column basis.
df.apply(lambda col: col.str.upper()) # Here lambda receivec a column
df.apply(lambda col: "hello " + col)

# To make it work on by-row basis use `axis=1` argument. This way you are able to engage columns in transformations.
# Here lambda receives a row.
df.apply(lambda row: row["role"] + " is better that " + row["occupation"], axis=1)

# This can be useful in conjunction with `.iloc` method.
df.iloc[:10, :].apply(
	lambda row: row["role"] + " is better that " + row["occupation"], 
	axis=1
)

# `apply` also works with separate columns.
# lambda receives individual value.
df["role"].apply(lambda val: val.upper())
pd.Series([1,2,3,4]).apply(lambda val: val**2)
pd.Series([1,2,3,4]).apply(lambda val: 0) # you can skip original values

# Another convinient function that mutates values by applying functions is `transform`. It is applied to each value in a dataframe or series.
# It's main purpose - make calculations based on existing values.
df.transform(lambda val: val + 'hello' + val[0])
pd.Series([1, 2,3, 4]).apply(lambda val: val ** 2)

# Unlike `apply` or `map` you can't just skip original values and replace them with random ones when working with dataframes.
df.transform(lambda val: 'new_val') # ValueError: Function did not transform

# You can also use per-row transformation with axis=1. As in `apply` you can refer to columns by indexing them, but again you should reference your original value.
df.transform(lambda val: val + val["occupation"] * len(val), axis=1) # valid
df.transform(lambda val: val["role"] + val["occupation"], axis=1) # invalid

# When transforming columns (series), there is no restriction on using original values.
df["role"].transform(lambda val: 'random') # replace everything with 'random'
pd.Series([1,2,3,4]).transform(lambda val: 0) # replace everything with `0`

# If you have multiple functions that work as a pipeline, you can do that in usual way - by manually passing each function result to next function.
func3(
	  func2(
		  func1(df, arg1, arg2),
		  arg1,
		  arg2
	  ),
	  arg1,
	  arg2
)

# Or you can use the `.pipe` method to chain those functions in a more readable way
(
	 df
	 .pipe(func1, arg1=..., arg2=...)
	 .pipe(func2, arg1=..., arg2=...)
	 .pipe(func3, arg1=..., arg2=...)
)

# For example, instead of writing this
((((1 / pd.Series(range(1, 1000001))) ** 2) * 6).cumsum()) ** -2
# You can write this
(
    pd.Series(range(1, 1000001))
    .pipe(lambda col: 1 / col)
    .pipe(lambda col: col**2)
    .pipe(lambda col: col * 6)
    .pipe(lambda col: col.cumsum())
    .pipe(lambda col: col**-2)
)
# It may be a bit slower to run multiple functions, but it certainly easier to read and debug.
# One main point you need to pay attention what each function returns. If your goal to work with dataframes only return dataframes from your functions.
```
---
#### Converting values
```python
# Nullifying coruppted data during type convertion.
df["col1"] = pd.to_numeric(df["col1"], errors="coerce").fillna(0)
```
---
#### Sorting DataFrames and Series
```python
import pandas as pd

df = pd.DataFrame(
    {
        "dates": pd.to_datetime(["2021-10-09", "2022-01-01", "2024-05-23"]),
        "names": ["first", "second", "third"],
        "values": [1, 2, 3],
    }
)

# Values are sorted with `.sort_values` method for dataframes or series.
# Sort dataframe by 'dates' column.
df.sort_values(by="dates")

# Sort dataframe in descending order.
df.sort_values(by="dates", ascending=False)

# Sort dataframe by multiple columns.
df.sort_values(by=["dates", "values"])

# Sort dataframe by multiple columns with one direction.
df.sort_values(by=["dates", "values"], ascending=False)

# Sort dataframe by multiple columns with multiple directions.
df.sort_values(by=["dates", "values"], ascending=[False, True])

# Sort single column.
df["dates"].sort_values(ascending=False)

# It is also possible to sort by index.
df.sort_index(ascending=False)

# If you have comples index, you can sort by multiple index rows.
# You can pass index names or their positions 
df.sort_index(level=["names", 0], ascending=[True, False])

# There is also `.rank` function, if you need to rank values.
df.iloc[:5, -1].rank(method="average")
# the `method` parameter determines how to treat duplicate values.
# "average" - is the default behaviour, it calculates the average rank and assigns it to all duplicates.
pd.Series([1,3,3,4,5]).rank(method="average") # [1.0, 2.5, 2.5, 4.0, 5.0]

# "min" - calculates the minimal position and assigns it to all duplicates.
pd.Series([1,3,3,4,5]).rank(method="min") # [1.0, 2.0, 2.0, 4.0, 5.0]

# "max" - calculates the maximum position and assigns it to all duplicates.
pd.Series([1,3,3,4,5]).rank(method="max") # [1.0, 3.0, 3.0, 4.0, 5.0]

# "first" - assigns unique values to all duplicates.
pd.Series([1,3,3,4,5]).rank(method="first") # [1.0, 2.0, 3.0, 4.0, 5.0]

# "dense" - works like "min", but does skip ranks for duplicate values.
pd.Series([1,3,3,4,5]).rank(method="dense") # [1.0, 2.0, 2.0, 3.0, 4.0]

# `rank` also provides the the `ascending` parameter
pd.Series([1,3,3,4,5]).rank(method="dense", ascending=False) # [4.0, 3.0, 3.0, 2.0, 1.0]
```
---
