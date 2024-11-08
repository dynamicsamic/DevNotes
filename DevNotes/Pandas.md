#### Read files quick reference
```python
from pathlib import Path
import pandas as pd

WORKDIR = Path(__file__).parent

# Most populat pandas functions for reading different file types
read_excel, read_csv, read_xml, read_json, read_sql, read_parquet, read_pickle, read_feather, read_html

# Read parquet file
df = pd.read_parquet(WORKDIR / 'tables/table1.parquet')
print(df)

# Read csv file
df = pd.read_csv(WORKDIR / 'tables/table2.csv', sep=';')
print(df)

# Read xml file 
df = pd.read_xml(WORKDIR / 'tables/table3.xml')
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

# In pandas read_csv method virtical headers can be 
```

#### Increase csv file processing with `pyarrow`
```python
import pandas as pd

# It is 2-6 times faster than default C implementation, but uses additional dependency, which can be installed wtih `pip install pyarrow`  
df = pd.read_csv('path/to/file', sep=',', engine="pyarrow")
```
---
#### Dataframe filtering
```python
import pandas as pd

df = pd.read_xml(...)

# Filter by <str> `status` column which has `cancel` in its value
df[df["status"].str.contains("cancel")]
# Note that here `df["status"].str.contains("cancel")` returns a pd.Series object with two columns - "index" and "status". This Series object is wrapped by a dataframe and its index column `selects` the wrapper dataframe rows.

# Filter by <str> `status` column which has not `wait` in its value
df[~df["status"].str.contains("wait")]

# Filter by <int> "id" column which equals to 1
df[df["id"].eq(1)]
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
