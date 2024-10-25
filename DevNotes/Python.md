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
sudo apt install python3-pip
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
