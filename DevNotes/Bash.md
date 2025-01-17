---
Created: 2024-08-07T23:43
---
#### While loop
```Bash
while <statement>
do
	...
done

# Example: read csv file in a loop
echo file.csv | while IFS="," VAR1 VAR2
# here IFS="," changes word separator from default single space to comma
# VAR1 and VAR2 are variables in which every line from file.csv will
# unpack to. So these vars hold two values on one line, it means we expect 
# two columns in our file. If we'd had more than two columns, we would need
# more variables.
do
	echo $VAR1 $VAR2 # we access the variable by prefixing them with '$' sign 
done
```
---
#### Add reference to `psql` and use it in a script
```Bash
PSQL="psql -X --username=<username> --dbname=<dbname> --no-align --tuples-only -c"
# -c flag lets you run a single command and exit psql
MYID=1
RESULT=$($PSQL "SELECT * FROM my_table WHERE id = $MYID;")
```
---
#### `if` statement syntax
```Bash
if [[ condition ]]
then
	# do something
fi

# chech for None variables
if [[ -z $MYVAR ]]
then
	# set the MYVAR variable
fi
```
---
#### Make new line characters visible in the output of an `echo` command in a script
```bash
# use `-e` flag and supply the `\n` character, it will be visible
echo -e "\nHello world!\n"
```
---
#### `Intercept a variable` passed to your script
```bash
use `$<int:num>` to capture arguments. 
cat $1 # will cat the string passed to script
```
---
#### `Function declaration` in bash
```Bash
# Functions are declared in uppercase with parenthesis and curly braces
MAIN_FUNCTION() {} # this is an empty function

# To call a function just place its name without parens
MAIN_FUNCTION
```
---
#### `Case` statement in bash
```Bash
# The case block consist of the `case` keyword, an expression to be evaluated,
# the `in` keyword, several cases delimited by a closing paren and end with
# double semicolon, which also may have a default action and the 
# final `esac` keyword.
case EXPRESSION in
option1) custom_action1 ;;
option2) custom_action2 ;;
*) default_action ;;
esac


case $USER_ID in
1) do_something ;;
2) do_other_thing ;;
*) do_default ;;
esac
```
---
#### Creating ranges
```bash
for n in {1..100}; do
	echo "Current number: ${n}"
done
```
---
#### The role of brackets, braces and parenthesis
```bash
# Brackets [], [[]] are syntactic sugar for the Bash builtin `test` command.
# Test command provdes different tests for string values, like comaprisons,
# pattern matching, etc.
# Double brackets is a modern version of single brackets, it lets you use
# regular expressions and other extended string comparison techniques.

# Examples.
# Test if two variables equal.
if [[$VAR1 == $VAR2 ]] then ... ; fi;

# Test if a variable follows a pattern
h=hello
if [[ $h =~ ^h[a-z]*$ ]] then ... ; fi ;

# Test does not follow pattern
if ! [[ $h =~ ^h[a-z]*$ ]] then ... ; fi ;

# To treat matching pattern as literal string enclose it in double quotes
if [[ $h =~ "[hello]" ]] then ... ; fi ;

```