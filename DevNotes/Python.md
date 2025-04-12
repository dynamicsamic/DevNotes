---
Created: 2024-07-23T18:09
---
#### Install `Python` on `Ubuntu`
```bash
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository ppa:deadsnakes/ppa # repository that held the most recent compiled python bundles 
sudo apt update
sudo apt install python3.13
```
---
#### Install `pip` on `Ubuntu`
```bash
sudo apt update
sudo apt install python3.13-pip
```
---
#### Install Python `venv` on `Ubuntu`
```bash
sudo apt install python3-venv
```
---
#### Create virtual environment with custom prompt in shell
```bash
python3 -m venv venv --prompt {your_prompt}
source venv/bin/activate
(your_prompt)$ ...
```
---
#### Create unexisting file while write or append
```python
# Add the `+` plus sign to the mode argument
with open('file.txt', 'w+') as f:
	...

with open('file.txt', 'a+') as f:
	...
```
---
#### Fastest substring match
```python
# Three are multiple ways to detect if a substring is present in a string.
# The most simpple and popular are str.find(), re.match() and str in str.
import re

'hello world'.find('hello') # => 0, found at index zero
'hello world'.find('not found') # => -1, not found

pattern = re.compile('hello')
pattern.match('hello world') # => <re.Match object; span=(0, 5), match='hello'>
pattern.match('not found') # => None

'hello' in 'hello world' # => True
'not found' in 'hello world' # => False

# Out of theese 3 methods the `in` method is the fastest, second - str.find, third - re.match
```
---
#### `__aexit__` method signature for async context managers
```python
from typing import Optional, Type
from types import TracebackType

class SomeManager:
...
async def __aexit__(
	self, 
	exc_type: Optional[Type[BaseException]],
	exc_val: Optional[BaseException],
	exc_tb: Optional[TracebackType]
) -> bool:
		...
```
---
#### `timeit.timeit` usage from python interface
```python
import timeit
    
def my_func(...):
	...
    
timeit.timeit("my_func(...)", setup="from __main__ import my_func", numer=...)
# or
timeit.timeit("my_func(...)", globals=globals(), number=...)
```
---
#### Convert `int` to `bytes` and vice-versa
```python
"""int to bytes"""

# <int> objects have a method `to_bytes`
int_obj.to_bytes(
	length: int = 1, 
	byte_order: Literal["big", "little"] = "big", 
	signed: bool = False
) -> bytes
# `length` instructs how many bytes future `byte-integer` will occupy
# e.g. length=4 means it will occupy 4 bytes, hence you'll be able 
# to store numbers up until 2^32.
# `byte_order` instructs where the most signigicant bytes will be stored
# default is the left most byte will be the highest.
i = 1_000_000
i.to_bytes(3) # b'\x0fB@'

"""bytes to int"""

# user the the int class method `from_bytes`
int.from_bytes(
	bytes_obj: bytes,
	byte_order: Literal["big", "little"] = "big", 
	signed: bool = False
) -> int
int.from_bytes(b'\x0fB@') # 1_000_000
```
---
#### Generate `UUID`
```python
import uuid
    
# Generate a random UUID4
uuid.uuid4()

# Generate UUID5 based on a UUID namespace and some string
uuid.uuid5(namespace=uuid.NAMESPACE_DNS, name="username")

# Grom commandline
python -m uuid -u uuid4

# UUIDs can't be decoded, so you can not extract the name or namespace
# values out of UUID.
```
---
#### Asyncio
```python
# Combine execution of multiple coroutines in a task queue style and catch all errors
import asyncio

async def coro1():
		...

async def coro2():
		...
		raise Exception

async def coro3():
		...

async def main():
		coros = [coro1(...), coro2(...)]
		r = await asyncio.gather(*coros, return_exceptions=True)
asyncio.run(main())

# In normal conditions the execution of all tasks in gather would be
# cancelled after the coro2 raises the exception and the exception
# would be propagated up until the start point of the function.
# With  `return_exceptions` however all tasks except the error one
# will complete their execution and return their results in a list.
# In the result list we also can find an Exception object for the 
# second coroutine.

### Using locks in asyncio ###
# Locks are useful when you want to limit the access to a resource that is being used by multiple coroutines. Such as writing to a file. For the lock to work properly it should be shared among courintes that access the resource.

# This code sample simulates file access by multiple coroutines.
import asyncio
from concurrent.futures import ThreadPoolExecutor

def _sync_write_data(file_path: str, data: str):
	with open(file_path, 'a') as f:
		f.write(data)

async def async_write_data(
	file_path: str, 
	data: str, 
	executor: ThreadPoolExecutor, 
	lock: asyncio.Lock,
	wait: int,
):
	# Here the await lock.acquire() method called.
	# Every courouting using this lock object will wait until lock released.
	async with lock:
		loop = asyncio.get_running_loop()
		asyncio.sleep(wait)
		await loop.run_in_executor(executor, _sync_write_data, file_path, data)

async def main():
	executor = ThreadPoolExecutor()
	lock = asyncio.Lock()
	await async_write_data('path/to/file', '\nNew line', executor, lock, 5)
	await async_write_data('path/to/file', '\nAnother line', executor, lock, 2)
	# Without using the lock, the second corountine would be executed before the first one, because of the first one waiting longer. Lock will synchronize the execution of these two coroutines.
```
----
#### Generate hashes of given length
```python
from hashlib import blake2b
    
h = blake2b(digest_size=20, key="SECRET")
return h.hexidigest()
```
---
#### Create custom Exception with `@classmethod`
```python
class MyException(Exception):
	def __init__(self, *args, **kwargs):
		self.__dict__.update(kwargs)
		Exception.__init__(self, *args)
		...

	@classmethod
	def create_from_other(cls, *args, arg1=None, arg2=None):
		...
		return cls(*args, arg1=arg1, arg2=arg2)
```
---
#### JSON Serialize `Enum`
```python
# If a model inherits from generic `enum.Enum` it can’t be used with `json.dumps`.
# To avoid this use `enum.StrEnum` or `enum.IntEnum`
import json
from enum import Enum, StrEnum

class InvalidEnum(Enum):
	arg = "Invalid"
class ValidEnum(StrEnum):
	arg = "Valid!"

json.dumps(ValidEnum.arg) # '"Valid!"'
json.dumps(InvalidEnum.arg) # Traceback (most recent call last): ... TypeError: Object of type InvalidEnum is not JSON serializable
```
---
#### Ternary operator in list comprehension:
```python
# The `else` syntax is not supported in list comprehensions.
# You can define `if` at the end of the comprehension
[i for i in range(10) if i % 2 == 0] # [0, 2, 4, 6, 8]

# But adding `else` after `if` raises error
[i for i in range(10) if i % 2 == 0 else i * 10] # SyntaxError: invalid syntax

# To overcome this use expression with ternary operator.
[i if i % 2 == 0 else i * 10 for i in range(10)] # [0, 10, 2, 30, 4, 50, 6, 70, 8, 90]
(i if i % 2 == 0 else i * 10) # - the important part
```
---
#### Difference between `__getattr__` and `__getattribute__`
```python
 # __getattr__ used for missing attribute lookup
class LookupMissing:
	attr = 42
	
	def __getattr__(self, attr_name: str) -> Any:
		print(f"__getattr__ called for attr name: {attr_name}")
		return 77

m = LookupMissing()
m.attr  # => 42
m.other_attr    # prints '__getattr__ called for attr name: other_attr'
				# returns 77

# __getattribute__ used for general attribute lookup 
class LookupAlways:
	attr = 42
		
	def __getattribute__(self, attr_name: str) -> Any:
		print(f"__getattribute__ called for attr name: {attr_name}")
		return 77
				
a = LookupAlways()
a.attr  # prints '__getattribute__ called for attr name: attr'
		# returns 77 			

a.other_attr  # prints '__getattribute__ called for attr name: other_attr'
			  # returns 77
```
---
#### Flatten a list of lists
```python
# Aprroaches listed in order of their efficiency
list_of_lists = [[1,2,3], [4,5,6], [7,8]]

# use builtin sum with empty list
flatten = sum(list_of_lists, [])

# use nested list comprehension
flatten = [i for j in list_of_lists for i in j]

# use reduce with with sum operator
from functools import reduce
from operator import add
flatten = reduce(add, list_of_lists)

# user chain from itertools
from itertools import chain
flatten = list(chain.from_iterable(list_of_list))
```
---
#### Create `timezone aware` datetimes
```python
from datetime import datetime
from zoneinfo import ZoneInfo

timezone = ZoneInfo("Europe/Moscow")
now = datetime.now(tz=timezone)
```
---
#### Create datetimes from strings
```python
from datetime import datetime

# If you have an ISO formatted datetime string
datetime.fromisoformat('2024-11-21 20:12:44')

# If you have string with custom formatting
datetime.strptime('21/11/24 21.22.11', '%d/%m/%y %H.%M.%S')
```
---
#### How to work with `namedtuple`
```python
from collections import namedtuple

# First create your `type`. First str arg is the name of the type.
# Usually it is identical to the type name itself.
# Next provide attributes either in one string separated by spaces
Person = namedtuple("Person", "name age email city")
# or in a tuple or list of strings.
Country = namedtuple("Country", ["name", "continent", "region", "capital"])

# Create an instance passing positional arguments
r = Country("Russia", "Europe", "East Europe", "Moscow")
# or by passing keyword arguments
r = Country(name="Russia", continent="Europe", region="East Europe", capital="Moscow")
# or by unpacking a dictionary
data = {"name":"Russia", "continent":"Europe", "region":"East Europe", "capital":"Moscow"}
r = Country(**data)

# Access namedtuple instance's attributes via attribute lookup
r.name  # -> Russia
# or via index lookup
r[0]  # -> Russia

# Iterate over namedtuple instance like it's a simple tuple
list(r)
print(*r)

# Convert namedtuple to dict
r._asdict()

# It is also possible to add docstring to namedtuple
Country.__doc__ = "A row of data about particular country"
```
---
#### Run `CPU-bound` tasks in separate process in `async functions`
```python
# Running cpu bound tasks in async function will block the event loop,
# disrupting the whole async execution model. To prevent this run cpu bound
# tasks in separate processes with ProcessPoolExecutor
import asyncio
from concurrnt.futures import ProcessPoolExecutor

def blocking_fun(*args):
	...

async def main():
	executor = ProcessPoolExecutor(workers=2)
	loop = asyncio.get_running_loop()
	awaitable_obj = loop.run_in_executor(executor, blocking_fun, *args)
	result = await awaitable
	return result

if __name__ == "__main__":
	asyncio.run(main())

# Since ProcessPoolExecutor creation has an overhead, the instance is typically craeted once and shared across different coroutines. 
```
---
#### `maketrans` and `translate` string functions
```python
# This functions used toghether to convert strings into new ones with 
# characters replaced according to provided mapping.  

# `maketrans` creates a mapping <old>: <new>
my_str = "hello world"
table = str.maketrans({'h': 'a', 'd': 'k'})

# `translate` produces new string with replaces chars
new_str = my_str.translate(table)
print(new_str)  # -> 'aello worlk'
```
---
#### Creating CLIs with `argparse`
```python
# argparse - is a powerfull builtin Python library for creating complex command line tools.
# Its main API is delivered via `ArgumentParser` class. 
# All arguments for ArgumentParser are optional.

from argparse import ArgumentParser

cli = ArgumentParser(
	prog = "MyCLI App", # Name of your CLI application
	description = "MyCLI App - ...", # First message
	epilog = "Thank's for using the app", # Final message
	add_help = True, # Add default -h --help option 
	...
)

# cli arguments and options (flags) are added via `add_argument` method
cli.add_argument("argname") # this is a cli argument
cli.add_argument("-a") # this is an option

# The difference between two types are
python myapp.py argname # this is how you provide a cli argument
python myapp.py -a argvalue # this is how you provide an option

# Usefull options for positional arguments
cli.add_argument(
	"argname", # name of the argument
	nargs = [ # how many args should cli take 
		int # like 2, `argname` argument would store 2 values in a list
		"?" # one or none value
		"*" # zero or more values
		"+" # at least one value
	],
	default = Literal, # like 42 or "hello", a value that would be used if 
					   # no value provided, usefull with nargs="*"
	type = <type>, # cast provided value to a type
	choices=[Literal_1, ...Literal_N], # list of valid values
	help="Some help message", # a help message for argument
	metavar="ArgName", # a more user-friendly argument name in help message
	dest="InnerArgName", # the arg name which would be used when accessing it
						 # after parser parsed args
	deprecated=bool, # flag that generates the deprecated warning

)

# Usefull options for options (flags), options from above not included
cli.add_argument(
	"-a", # short name of the option
	'--argument', # verbose name of the option
	action= [
		"store" # stores provided value, default
		"store_const" # assign a constant, if no valu provided
					  # should be used with const=...
		"store_true" # assign True, if no value provided
		"store_false" # assign False, if no value provided
		"append" # captures all values if an option appeared multiple times, 
				 # like myapp -f 123 -l -s -l 345
		"count" # counts how many times an option occurs
				# like pytest -vvv, would be 3 for -v option
	],
	required=bool # allows empty values for arguments
)

# You can test, how your cli app works by using parse_args method
cli.parse_args(['-a', 'some_val']) # either returns instance of argparse.Namespace or exits with error.

# It is also possible to add subcommands, to avoid arguments shadowing and conflicting each other. An app with subcommands would work like this
python myapp <my_command> argument -o option_arg

# A subcommand cli flow:
# 1. Create a cli:
cli = ArgumentParser(prog = "MyCLI App", description = "MyCLI App - ...")

# 2. Create a subcommands entity, using the main cli's `add_subparsers` method.
## This entity would gather all following subcommands:
subcommands = cli.add_subparsers(title="subcommands", help="main commands")

# 3. Create a subcommand parser for you command
first_parser = subcommands.add_parser("first", help="my help message")

# 4. Add arguments for your parser as usual
first_parser.add_argument(...)
first_parser.add_argument(...)

# 5. Optionally you can add arbitary kwargs to you parsers
first_parser.set_default(key=value)

# When runing your app, use `parse_args` method to parse arguments from console
def main():
	cli = ArgumentParser()
	cli.parse_args()

# You can also handle app crashing using ArgumentParser's builtin method
def main():
	cli = ArgumentParser()
	try:
		cli.parse_args()
	except Exception as err:
		logger.error(err)
		cli.print_help() # print help message to user
		cli.exit(status=1, message="Error occured")


# If you need to collect arbitrary amount of options that may contain dashes and double dashes (which in argparse is considered an option itself), you can use the REMAINDER constant.
cli = ArgumetnParser()
cli.add_argument("required") # this is a regular positional argument
cli.add_argument("-args", nargs=argparse.REMAINDER)

# Now you are able to do this
cli.parse_arguments(['value', '-a', '--d', 'optional']) # the -args option will receive ['-a', '--d', 'optional']

# To add multiline formatting to description or usage use `RawTextHelpFormatter`
from argparse import ArgumentParser, RawTextHelpFormatter
cli = ArgumentParser(
	description="Some long description", 
	formatter_class=RawTextHelpFormatter
)
```
---
#### `uv` dependency tool
```bash
# Update uv
uv self update


```
---
#### Parsing `XML` files
```python
# XML is a hierarchical strucutre, it consists of the root node and child nodes, which in turn can contain more child nodes.
# Python has builtin xml package.
import xml.etree.ElementTree as ET

# Construct a tree object from file 
tree = ET.parse('/path/to/file')

# Get tree's root.
root = tree.getroot()

# To iterate over root's children use a for loop.
for element in root:
	print(element) # Each element is an instance of ET.Element class

# To traverse through all nodes in a tree use iter() method
for element in root.iter():
	print(element)

# Like HTML tags xml tree's elements can also have attributes. 
# For example: <country name="Russia"> and <neighbor name="Belarus">, here `county` is the element (tag) and `name` is attribute.
# To get elements attrbutes use `attrib` property
for element in root.iter():
	print(element.tag, element.attrib)

# It is also possible to construct xml tree from string instead of file.
# Sample XML string
s = """<?xml version="1.0"?>
<data>
    <country name="Liechtenstein">
        <rank>1</rank>
        <year>2008</year>
        <gdppc>141100</gdppc>
        <neighbor name="Austria" direction="E"/>
        <neighbor name="Switzerland" direction="W"/>
    </country>
</data>"""

element = ET.fromstring(s)
# In this case ET a tree's root element. It does not have `getroot` method.
```
---
#### Write `remote file's` contents to local file
```python
# Suppose we have a file on a remote server. We can read its contents, but do not have the `Download` file functionality. In this case we can read remote file's contents and write it to our local file.
import requests

response = requests.get('some.url')
with open('path/to/file.ext', 'wb+') as file:
	file.write(response.contents)
```
---
#### Traverse directory recursively
```python
# The `walk` method from `os` package does the thing. It accepts a string-path and returns a generator which for every subdirectory yields a 3-tuple, consisting of current_directory, list of child directories, list of child files.

# Printing all files can be done using this flow.
import os

for path, _, files in os.walk('/path/to/dir/'):
	for file in files:
		print(os.path.join(path, file))
```
---
#### Raise exceptions when receiving bad status codes in `Requests`
```python
# You can avoid manually checking response statuses with `raise_for_status` method.
# In case of a 4XX answer it will raise an error.
import requests

try:
	response = requesets.get('unexisting_url_return_404.com')
	response.raise_for_status()
except Exception as e:
	... # do something
	raise
```
---
#### Allow importing from parent and sibling directories by creatin a python package
```python
# Importing from sibling or parent directories isn't trivial in Python. It requires manipulating PYTHONPATH env variable and looks a bit hacky.
# To overcome this limitation you can turn your working directory into a python package and install it as usual pip package.

# Create a root directory for your package with descriptive name.
mkdir codebase

# Inside your root directory create a virtual environment
python -m venv .venv --prompt codebase

# Activate venv and install external dependecies
source .venv/bin/activate
(codebase)$ pip install pandas ...

# Inside root directory create pyproject.toml file with contents like this
[project]
name = "codebase"
version = "0.1.0"
description = "My small project"

[build-system]
build-backend = "flit_core.buildapi"
requires = ["flit_core >=3.2,<4"]

# Now install your package via pip in editable mode. Editable mode is needed to reflect changes made to `installed` python files.
(codebase)$ pip install -e .

# Create your source code structure
mkdir codebase/lib # `lib` stores source code that other modules will import 
touch codebase/lib/__init__.py codebase/lib/funcs.py

mkdir codebase/client # `client` - application code that will import from `lib`
touch codebase/client/__init__.py codebase/client/client_mod.py

# Inside codebase/lib/func.py
def echo(arg):
	print(arg)

# Inside codebase/client/client_mod.py
from codebase.lib.func import echo # You need to include import path from the root directory

echo("Hi, my name is John")


# Return to terminal
(codebase)$ python codebase/client/client_mod.py
... Hi, my name is John
```
---
#### Create custom types for dictionaries with certain contents
```python
# If you need to restrict type of dictionaries your code works with, you can create a custom dictionary with certain keys.
from typing import TypedDict

class MyStrictDict(TypedDict):
	key1: str
	key2: int

# Alternative syntax
MyStrictDict = TypedDict('MyStrictDict', {'key1': str, 'key2': int})

# Mark certain key as not required
from typing import NotRequired

class NotSoStrictDict(TypedDict):
	strict: str
	loose: NotRequired[int]

# It is also possible to mark all keys as not required
class LooseDict(TypedDict, total=False):
	key1: str
	key2: int

# It is also possible to mark a filed as required when all fields are not required
from typing import Required

class MixedDict(TypedDcit, total=False):
	strict: Required[str]
	loose: int
```
---
#### Save list of dictionaries as JSON file
```python
import json

items = [
	{"name":"John", "age": 24}, 
	{"name": "Bob", "age": 44},
]

with open('file.json', 'w+', encoding='utf-8') as f: 
	json.dump(items, f, ensure_ascii=False, indent=4) # Not `dumps` but `dump`
```
---
>[!summary]- Working with CSV files
>The standard csv library provides two ways for reading a csv file - using the `reader` function and the `DictReader` class.
>`reader` function is a more basic tool that is written in `C` , and `DictWriter` is a Python-based abstraction over this function with some convenient methods and attributes.
>1. The `reader` function takes any iterable object (including file objects) and returns a generator-like `reader_` object that yields one row at a time when iterated over.
>```python
>with open("test_file.csv", newline="") as csv_file:
    reader = csv.reader(csv_file)
    for row in reader:
        print(row) # each row is a list of values
>									# ['Fruit', 'Quantity', 'Cost']
>									# ['Strawberry', '1000', '$2.200']
>									# ['Apple', '500', '$1.100']
>```
>Be sure to consume the reader while the file is open. Closing the file context will prevent the reader from further file parsing.
>The `reader` function accepts several arguments for customizing the csv file reading process. B**e sure to provide keyword arguments, not positional ones.**
>The most frequently used are:
>- `delimiter: str = ","` - a character that separates columns from each other.
>- `quotechar: str | None = '"'` - character that indicates borders of a value.
>	E.G. if you use `,` as delimiter and your file contains values such as `$2,000`, the csv parser will split this value in two fields, first - `$2`, second - `000`. To prevent this surround such values with a character and pass this character as `quotechar` argument. For `quotechar = "` value `"$2,000"` will work, for `quotechar = |` value `|$2,000|` will work.
> There are also multiple arguments for formatting quotes and escaping special characters, but they are used not quite often.
> There is also the `lineterminator: str = "\r\n"` argument which is in fact hardcoded, so you can't the the lines are terminated in any way.
> The `reader_` is a pretty basic object, there isn't much you can do with it except for iterating over it.
> 

#### Working with xml files
```python
import xml.etree.ElementTree as ET

# Create a tree object
tree = ET.parse('path/to/file.xml') # via reading from disc
root_element = ET.fromstring(               # via parsing a string
	'''<?xml version="1.0" encoding="utf-8" ?>
	<orders date="2025-01-16 03:34:47">
	<order>
	<id>8-296-09658</id>
	</order>
	</orders>'''
)
root_element = ET.fromstringlist(          # Via parsing a list of string
	[
		'<?xml version="1.0" encoding="utf-8" ?>',
		'<orders date="2025-01-16 03:34:47">',
		'<order>',
		'<id>8-296-09658</id>',
		'</order>',
		'</orders>'
	]
)

# Every xml tree consists of elements that may contain subelements. 
# Every element has 3 main properties.
element.tag  # name of the element's tag (str)
element.text  # text value of the element; for parent elements it is usually empty string or '\n' (str)
element.attrib # value of a tag's attribute dict[str, str]; for <orders date="2025-01-16 03:34:47" .attrib would return {"date": "2025-01-16 03:34:47"}

# Elements also have methods for retrieving data, adding and removing elements, and iterating the element tree.

### Methods fo iteration ###
element.find('tag_name') # Find the first sublement by its tag name.
element.findall('tag_name') # Find all sublements by a tag name and store this subelements in a list.
element.findtext('tag_name') # Return the value of an enclosed tag. Only returns value of the topmost tag. Won't work with nested subelements.
element.iter() # Create an iterator for all elements and subelements 
element.itertext() # Creates an iterator for text values of all elements and subelements
element.iterfind('tag_name') # A generator version of the `.findall` method

### Methods for working with attributes ###
element.keys() # get all attributes' names
element.get('attrib_name') # get the attribute's value
element.set('update': True) # add or update an attribute
element.values() # get tuples of all <attribute:value> pairs  

### Methods for mutating xml tree ###
# Append a subelement to the end of the list of subelements.
element.append(ET.fromstring('<order><id>12323</id></order>'))

# Insert an element at certain index
element.insert(1, ET.fromstring('<order><id>12323</id></order>'))

# Append multiple subelements
element.append(
	[
		ET.fromstring('<order><id>12323</id></order>'),
		ET.fromstring('<order><id>7224</id></order>')
	]
)

# Remove a particular element. Works only for element objects, not tag names.
elem_to_remove = element.find('order')
element.remove(elem_to_remove)

### Save or inspect xml files ###
# Print tree object to stdout
ET.dump(element)

# Convert tree to byte-string
ET.tostring(element)

# Convert tree to list of byte-strings
ET.tostringlist(element)

# Write contents of a tree into file. Only works with trees not elements
tree.write('path/to/file.xml')

# You can use .tostring or .tostringlist with `open` function in binary mode to write contents of a tree object to a file.
with open('my_file.xml', 'wb') as f:
	f.writelines(ET.tostringlist(tree))

### General workflow for xml files ###
# Create a tree
tree = ET.parse('path/to/file.xml')

# Get the root element (node)
orders = tree.getroot()

# Iterating over top level subelements
for order in orders:
	order.findtext('id')

# Iterating over nested subelements
for order in orders:
	for prop in order:
		...

# Iterate over particular subelements
singles = 0
for order in orders:
	for products in order.findall('products'):
		for product in products.findall('product'):
			if product.findtext('quantity') == '1':
				singles += 1

# Use generators for memory-efficient iteration
for order in orders.iterfind('order'):
	for products in order.iterfind('products'):
		for product in products.iterfind('product'):
			if product.findtext('quantity') == '1':
				singles += 1

# Modify element values inplace
for product in products.iterfind('product'):
	qty = int(product.findtext('quantity'))
	if qty == 0:
		product.find('quantity').text = '1'
	
# Root - is the top element, that contains other elements.
orders = tree.getroot()
```
---
#### Pretty print contents of a `defaultdict`
```python
from collections import defaultdict
from pprint import pprint

d = defaultdict(list)
d["list"].append("values")
pprint(dict(d))
```
---
#### Format dates with f-strings
```python
from datetime import datetime as dt

# Format datetime timestamp
print(f"{dt.now():%Y%m%d%H%M%S%f}") # 20250325174350195095

# Format with separators
print(f"{dt.now():%Y-%m-%-d %H:%M:%S.%f}") # 2025-03-25 17:43:50.195095
```
---
#### Type annotations
```python
# Annotate function that can accept arbitrary arguments (including none) of arbitrary type (Any).
from typing import Callable
Callable[..., ReturnType] # Here elipsis means none or many arguments of arbitrary type.
# This annotation feets a broad variety of functions, it is loose.

# Strictly annotate funcion's arguments' names and types.
from typing import Protocol

# Functions that follow the protocol below should implement parameter names and types as defined in the protocol definition.
class MyFunctionProtocol(Protocol):
	__init__(self, *vals: int, sep: str | None = None) -> list[str]:
		...
def valid_func(*vals: int, sep: str | None = None) -> list[str]: ...

def invalid_func(*vals: int, delim: str | None = None) -> list[str]: ...
```
---
#### Test note
I would like to have the spell checking on my notes
```python
# As well as seemless code blocks
def test_function(...):
	pass
```
This is possible but It looks like it is not contained I guess

---
#### This is another item


