#### Installation on Ubuntu
```bash
# First install ngnix
sudo apt update && sudo apt upgrade -y
sudo apt install ngnix

# Ubuntu is configured to automatically add nginx as service.
# Check if nginx is running as service
sudo service nginx status
# You should see
... Active: active (running) since Wed 2024-10-09 12:51:49 MSK; 13min ago ...

# If you see
... Active: failed (Result: exit-code) since Wed 2024-10-09 12:40:39 MSK; 7min>
# It means Ubuntu could not start nginx service.
# One possible reason is the default 80 port is busy.

# Check 80 port
sudo lsof -i:80
# Example output
COMMAND   PID     USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
apache2 24580     root    4u  IPv6 112398      0t0  TCP *:http (LISTEN)
apache2 24582 www-data    4u  IPv6 112398      0t0  TCP *:http (LISTEN)
apache2 24583 www-data    4u  IPv6 112398      0t0  TCP *:http (LISTEN)
apache2 24584 www-data    4u  IPv6 112398      0t0  TCP *:http (LISTEN)
apache2 24585 www-data    4u  IPv6 112398      0t0  TCP *:http (LISTEN)
apache2 24586 www-data    4u  IPv6 112398      0t0  TCP *:http (LISTEN)
# Here we see that apache2 listen on 80's port

# Check apache2 service status
sudo service apache2 status
>>>  apache2.service - The Apache HTTP Server
     Loaded: loaded (/lib/systemd/system/apache2.service; enabled; vendor prese>
     Active: active (running) since Wed 2024-10-09 12:38:38 MSK; 8min ago ...

# Stop it to free up the port
sudo service apache2 stop

# Start nginx service
sudo service nginx start

# Install php and php plugins
sudo apt update && sudo apt upgrade -y

# Install some utility to manage software and packages
sudo apt install -y software-properties-common

# Add new apt respository to search for PHP-specific packages 
sudo add-apt-repository ppa:ondrej/php

# Update index to include new packages
sudo apt update

# Install PHP
sudo apt install -y php8.3

# Check the insatllation completed successfully
php -v
>>> PHP 8.3.12 (cli) (built: Sep 27 2024 03:53:05) ...

# Optionally search for PHP-related packages
sudo apt-cache search php8.3
>>> php8.3-amqp - AMQP extension for PHP
... php8.3-apcu - APC User Cache for PHP
... php8.3-ast - AST extension for PHP 7
...

# Install PHP plugin to adjust it to work with nginx and MySQL
sudo apt install -y php8.3-fpm php8.3-mysql

# Update nginx config file
sudo nano /etc/nginx/sites-available/default
>>> # You should look at the following ...
# Find the server {} section.
>>> server {
...     listen 80 default_server;
...     listen [::]:80 default_server;
...
# Find the location{} sections
# Add a new location{} or uncomment existing php-related one
# The new location{} section should look like this 
>>> location ~ \.php$ {
...             include snippets/fastcgi-php.conf;
...             fastcgi_pass unix:/run/php/php8.3-fpm.sock;
...     }
# Save and close the file.

# Restart nginx
sudo systemctl restart nginx

# Reload the PHP-nginx plugin
sudo systemctl reload php8.3-fpm

# Test that nginx can serve PHP files
sudo bash -c 'echo "<?php phpinfo();?>" > /var/www/html/info.php' # creates a PHP test file with system info inside /var/www/html/ directory.

# Open your browser at localhost:80/info.php. 
>>> PHP Version 8.3.12|
>>> |System|Linux 6.8.0-45-generic #45~22.04.1-Ubuntu SMP PREEMPT_DYNAMIC Wed Sep 11 15:25:05 UTC 2 x86_64|
... 

# Delete this file
sudo rm /var/www/html/info.php

# List installed PHP plugins
php -m

# To uninstall PHP
sudo apt-get purge php8.3

# Remove the orphaned packages with
sudo apt-get autoremove

# Lastly, check the PHP version to confirm the uninstall worked
php -v
```
#### Basics
```php
// One-line comment like in golang, javascript
 # One-line comment like in Python, Bash
/* Multy
	line
	comments */

// Every program in PHP should start with opening tag `<?php` and end with
// closing tag `?>`.
<?php  
  // PHP is lously typed, it tryes to coerce types when they differ.
  $a = '50';
  $b = 100;
  echo $a + $b; //=> produces 150.
  echo '150' + '120'; //=> produces 270.
  echo 150 + '15ffde'; //=> produces 165 and throws warning.
?>
```
---
#### Stidin/Stdout
```PHP
// To print a scalar value use `echo` or `print` commands.
$var = 12;
echo $var;
print $var;

// To print more complex data use `print_r`.
$arr = array(1,3,4);
print_r($arr); //=> Array([0]=>1 [1]=>3 [2]=>4)

// To print a more detailed variable information of arbitrary type use `var_dump` function.
$var_dump($arr);

// To read input from stdin use `readline`.
// Input type is inferred automatically.
$name = readline();
```
---
#### Variables and constants
```PHP
// Variables are declared with `$` sign prefix with no type definition.
// Every expression should be separated from others with semicolon `;`.
$variable = 12;

// Constants shuld be uppercase by convention and can be declared in two ways.
define("CONSTANT_NAME", "constant_value"); // using `define` function.
const HELLO_CONSTANT = 11; // using `const` instruction.

// To check if a constant was defined use `defined` function
if (!defined("MY_CONSTANT")) {
	define("MY_CONSTANT", 22);
}

// Delete a variable (does not work with constants).
unset($variable);

// Check if a variable exists and is not null.
isset($variable);

// Global variables are not seen inside functions.
$a = 10;
function printa() {
	echo $a;
}
printa(); //=> Warning: Undefined variable $a.

// To get access to global variables the `global` keyword is used.
// This way you can also mutate the variable.
function printa() {
  global $a;
  echo $a;
}
printa(); //=> prints 10;

# Type casting is done by enclosing the resulting type in parens and providing a value, like this: (type) val.
// Booleans 
var_dump((bool) "");             // -> false
var_dump((bool) "Some Text");    // -> true
var_dump((boolean) "0");         // -> false
var_dump((bool) "false");        // -> true
var_dump((bool) 0);              // -> false
var_dump((bool) 1);              // -> true
var_dump((bool) -1);             // -> true
var_dump((bool) null);           // -> false
var_dump((bool) []);             // -> false
var_dump((bool) ["hello"]);      // -> true

// Integers
var_dump((int) false);        // -> 0
var_dump((integer) true);     // -> 1
var_dump((int) "-1");         // -> -1
var_dump((int) "Hello");      // -> 0
var_dump((int) "12 months");  // -> 12, collects all digits until first non-digit 
var_dump((int) 12.7);         // -> 12
var_dump((int) null);         // -> 0

// Float
var_dump((float) false);      // -> 0
var_dump((float) true);       // -> 1
var_dump((float) "-1");       // -> -1
var_dump((float) "Hello");    // -> 0
var_dump((float) "2.5 Hour"); // -> 2.5
var_dump((float) null);       // -> 0

// Strings
var_dump((string) false);     // -> ""
var_dump((string) true);      // -> "1"
var_dump((string) 0);         // -> "0"
var_dump((string) 1.353);     // -> "1.353"
var_dump((string) []);        // -> "Array"
var_dump((string) ["John"]);  // -> "Array"
var_dump((string) null);      // -> ""

// Arrays
var_dump((array) false);      // -> [false]
var_dump((array) true);       // -> [true]
var_dump((array) 0);          // -> [0]
var_dump((array) 1.353);      // -> [1.353]
var_dump((array) "John");     // -> ["John"]
var_dump((array) null);       // -> []
```
---
#### Mathematical operators
```PHP
// Mathematical operators generally have higher priority comparing to
// other types of operators.

		/*Mathematical operators in order of precedence */

**      // `POW` operator; 2 ** 3 => 8
+$var   // unary positivity
-$var   // unary negativity
++$var  // prefix increment; increment value and return it
--$var  // prefix decrement; decrement value and return it
$var++  // postfix increment; return value and increment it
$var--  // postfix decrement; return value and decrement it
*       // `MULT` operator; 2 * 2 => 4
/       // `DIV` operator; 4 / 2 => 2; 5 / 2 => 2.5
%       // `REMAIN` opeartor; 5 % 2 => 1; -6 % 4 => -2
+       // `ADD`; 1 + 1 => 2
```
---
#### Comparison operators
```php
// Comparison operators have lower precendece than mathematical operators.
// (Except the logical `!` operator)

	  /* Comparison operators in order of precedence */

<    // less_than, loose comparison
<=   // less_or_equal, loose comparison
>    // greater_than, loose comparison
>=   // greater_or_equal, loose comparison
==   // loose is_equal, string(11) == int(11) => True
===  // strict is_equal, string(11) === int(11) => False
!=   // loose not_equal, string(11) != int(11) => False 
!==  // strict not equal, string(11) !== int(11) => True
<>   // same as !=
<=>  // `spaceship` operator:  if $a > $b => 1;
     //                        if $a == $b => 0; 
     //                        if $a < $b => -1  
??   // compare-to-null operator. If left argument is null,
     // returns right argument, otherwise return left argument
     //                       null ?? 42 => 42; 
     //                       array() ?? 42 => array(); 
     //                       array(1,2,3)[99] ?? 42 => 42. 
? :  // ternary operator. Has syntax: condition ? value1 : value2.
	 //                       $var = 12; 
     //                       $var > 10 ? $var * 2 : $var / 2; => 24.
     //                       Reads like 'if $var > 10 then return $var * 2,
     //                       else return $var / 2.
```
---
#### Logical operators
```PHP
// Logical operators have lower precendece than comparison operators.
// (Except the `!` operator)

	/* Logical operators in order of precedence */

!    // `NOT` (negate) operator; !True => False;
&&   // `AND` operator; True && True => True;
||   // `OR` operator; True || False => True; True || True => True;
and  // same effect as `&&` but with lower precendece;
xor  // `XOR` operator; True xor False => True; True xor True => False;
or   // same effect as `||` but with lower precendence;
```
---
#### Assignment operators
```php
// Assignment operators have lower precendece than symbolic logical operators
// but have higher precedence than word-like logical operators.
// All assignment operators have equal precedence.

$var = 42       // simple assign; `$var2 = $var` will make a copy
                // of $var's value;
$bar = &$var    // assign a reference to a value which $var points to.
	            // Now the two variables point to single value.
	            // Reassignment of one variable, will reassign the other.
	            // $var = 12=> $bar = 12 as well.
$var *= 42      // multiply by value and assign;
$var /= 42      // divide by value and assign;
$var %= 42      // get remainder from dividing by value and assign it to variable
$var += 42      // add value and assign;
$var -= 42      // substract value and assign;
$var .= 'hello' // concat two strings and assign to variable

// Bitwise assignment opeartors.
&=, |=, ^=, <<=, >>=
```
#### Useful constants
```php
__FILE__ // path to current file
__DIR__ // path to directory, where current file located
__LINE__ // line of code

PHP_VERSION // outputs the php version

```
---
#### Commenting PHP code
```php
// Documenting PHP code is done using specified format named `docblocks`.
/**
 * Sums two numbers, adds to array if provided.
 * 
 * Always returns a value. Array is passed by reference if provided.
 * 
 * @param int|float $a   The first number.
 * @param int|float $b   The second number.
 * @param ?array &$array Optional. The array that the sum will be appended to.
 *                       Defaults to null.
 * 
 * @return int|float     Sum of two numbers.
 */
function sum(int|float $a, int|float $b, ?array &$array = null): int|float {
    $result = $a + $b;
    
    if ($array) {
        $array[] = $result;
    }
    return $result;
}

```
---
#### Strings
```PHP
// Strings can be enclosed in either single or double qoutes.
// Single quotes are `raw` strings - meaning they don't recognize 
// special characters, like `\n`. Double qoutes can handle this.
echo 'Hello\nWorld!' //=> just prints 'Hello\nWorld!'.
echo "Hello\nWorld!" //=> prints 'Hello' and 'World!' on separate lines.

// String concatenation is done using the `.` operator.
$hello = "Hello";
echo $hello." World!"; //=> `Hello World`

// Also possible to concatenate random strings.
$hello." "."World"."!";

// Concatenation with `+` operator raises error.
echo "Hello" + " " + "World!";

// Displaying variables inside strings is very straight forward.
echo "I would like to say $hello!"

// When you to add something to variables it is better to use braces `{}`.
$f = 'fruit'
echo "I like {$fruit}s!"

// Calling a function iside string is not possible.
echo "I would like to call my function {myFunc()}"; //=> prints bare string.

// You can inject object variables and methods inside stirngs.
$obj = new SomeObject();
echo "My object {$obj->name}, whants to say something: {$obj->tellSmth()}"

// Strings support indexing
echo "hello"[0]; //=> 'h'
echo "hello"[-1]; //=> 'o'

// You can produce string slices with `substr` function.
$st = "Hello"
substr($st, 2); //=> from index 2 until the end "llo"
substr($st, 2, 2); //=> take two chars starting from index 2, "ll"
substr($st, -3, 2); //=> take two chars starting from index -3, "lo"

// Convert int to string
$var = strval(123);
echo gettype($var); //=> string

// Get string length
echo strlen("hello"); //=> 5

// Convert string into a list of chars with `str_split()` function.
// Function signature: str_split(string $string, int $chunkSize = 1)
str_split('hello world'); // ['h', 'e', 'l', 'l', 'o' ..., 'd']
str_split('hello world', 2); // ['he', 'll', 'o ', 'wo', 'rl', 'd']
```
---
#### Object type info
```php
// Get object's type
$var = 12;
gettype($var); //=> integer

// Get more detailed type info
var_dump($var); //=> int(125).
```
---
#### Conditional statements
```php
// If block definition
if ($var != 42) {
	echo "not equal";
} elseif ($bar === 'World') {
	echo "Hello!";
} else {
	echo "The ELSE!";
}

// You may omit braces if you use the `endif` operator at the end of the block.
if ($var != 42):
	echo "not equal";
elseif ($bar === 'World'):
	echo "Hello!";
else:
	echo "The ELSE!";
endif;

// When using the `switch` statement it evalueates all the cases until it 
// reaches `break` or `return` operators, so you need to use either of them  
// at the end of each case short-curcuit the evaluation of `switch` statement.
switch ($var) {
	case 42:
		echo "Not bad";
		break;
		
	// We can chain cases that have the same behavior.
	case 33:
	case 44:
		return "Hooray!";
	case 22:
		$var = 42;
		echo "To low"; // This will proceed to the next case.
	default:
		$var = 11; // This will override $var's value from previous case.
}

// There is also second variation of switch syntax with `endswitch` operator.
switch ($var):
	case ... :
	case ... :
		return 1;
	default:
		...;
endswitch;

// switch satement can also be used to evaluate arbitrary expressions.
$userId = 22;

switch (getPermission($userId)) {
	case "admin":
		...
	case "moderator":
		...
	default:
		...
}
```
---
#### Loops
```php
                /************** FOR LOOP **************/
               /**************************************/
// for loop syntax is similar to JS and Go
for (<init_expression>; <exit_expresssion>; <each_step_expression>) {
	do_some_stuff;
}

// simple example
for ($i = 0; $i < 10; i++) {
	echo $i;
}

// All of the expressions can be ommited which converts the for loop into 
// while loop making it the developer's responsibility to control the loop flow. 
$i = 0;
for (; ; ;) {
	if ($i > 5) {
		break;
	}
	echo $i;
	$i++;
}

                /************** WHILE LOOP **************/
               /****************************************/
// while loop also has a predictable syntax
while (<exit_expression>) {
	do_some_stuff;
}

// simple example
$i = 10;
while ($i > 0) {
	echo $i;
	$i--;
}

                /************** DO WHILE LOOP **************/
               /*******************************************/
// do-while loop is similar to `while` but it executes the `do` part 
// at least once, no matter what is written in the `while` part.
do {
	echo "Hello world!";
} while (false); // Prints `Hello world!` once and finishes.

                /************** FOREACH LOOP **************/
               /******************************************/
// foreach loop iterates through array's elements giving access to 
// one element at a time.
$arr = [1,2,3];
foreach ($arr as $elem) {
	echo " $elem"; //=> 1 2 3
};

// You can also get acces to array's idexes.
foreach ($arr as $idx => $elem) {
	echo "Idx=$idx, elem=$elem"; // Idx=0, elem=1 Idx=1, elem=2 Idx=2, elem=3
}

// The `break` keyword is used to exit for, while, do-while and foreach loops 
foreach ($arr as $val) {
	if ($arr === 42) {
		break; // terminate the loop completely if condition met.
	}
	do_some_stuff;
}

// The `continue` keyword is used to skip the rest of one loop iteratrion 
// works the same way as in Python.
foreach ($arr as $val) {
	if ($arr === 42) {
		continue; // only skip this particular iteration
	}
	do_some_stuff;
}

// Alternative syntax for `for`, `while` and `foreach` loops, that uses
// colon opearator.
for ($i = 0; $i < 10; $i++):
	echo $i;
endfor;

while ($i > 10):
	echo $i;
endwhile;

foreach ($arr as $val):
	echo $val;
endforeach;
```
---
#### Math
```php
// Integer division (no remainder) functions
intdiv(8, 3); //=> 2

// Round numbers (int, float).
$low = 22.000001;
$mid = 22.545;
$high = 22.999999;

// `floor` rounds down to nearest integer.
floor($high); //=> 22

// `ceil` rounds up to nearest integer.
ceil($low); //=> 23

// `round` allows to specify how many numbers to save.
round($mid); //=> 22
round($mid, 2); //=> 22.55
round($mid, -1); //=> 20

// `round` also allows to specify how to treat 4 and 5 when rounding.
PHP_ROUND_HALF_UP // the last 5 is rounded up, default
PHP_ROUND_HALF_DOWN // the last 6 is rounded up
PHP_ROUND_HALF_EVEN // 1.5 and 2.5 round to 2
PHP_ROUND_HALF_ODD // 1.5 rounds to 1 and 2.5 rounds to 3
```
---
#### Arrays
```php
// Arrays in PHP are in reality ordered maps, but they can be used to act like
// lists, dictionaries, stacks and queues.
$arr = array(
	"one" => 1,
	2 => "two",
);
/* OR */
$arr = ["one" => 1, 2 => "two"];

// Get array value
$arr["one"]; //=> 1;

// Get missing value
$arr["missing"]; //=> raises warning

// Append a key:value pair
$arr["new"] = 42;

// Delete key:value pair
unset($arr["new"]); //=> removes "new" key from array.

// Array keys must be strings or integers. All other types will be coerced
// to this two types. Values for overlapping keys will be erased down to
// the last key:value pair.
$arr = [
	1 => "one",
	"1" => "two",
	1.5 => "three",
];
print_r($arr); //=> [1 => three], coerced everything and chose the last key.

// You can add two array-maps to form a union.
// Overlapping keys will be overriden. The exact way HOW they will be replaced
// depends on the position of operands.
$arr1 = [1 => "five"];
$arr2 = [1 => "SIX", "two" => 3];
$arr3 =  $arr1 + $arr2;
print_r($arr3); //=> [1 => "five", "two" => 3]

$arr3 = $arr2 + $arr1;
print_r($arr3); //=> [1 => "SIX", "two" => 3]

// List-like arrays are created by omitting the keys definition. 
$arr = array(1, "hello", 3.6); 
/* OR */ 
$arr = [1, 2, 3];

// This way keys are assigned implicitly as idecies.
print_r($arr); //=> [0 => 1, 1 => 2, 2 => 3]

// Arrays support indexing. Negative indexing not allowed.
[1, 15, 17][0]; //=> 1;

// Append a value. Multiple values not allowed.
$arr = [1, 2, 3];
$arr[] = 15;
print_r($arr); //=> [1, 2, 3, 15]

// Append multiple values with `array_push` function.
array_push($arr, 22, 23); // => returns new size = 6.
print_r($arr); //=> [1, 2, 3, 15, 22, 23]

// Push new values at the front with `array_unshift` function.
array_unshift($arr, "hello", "not"); // => returns new size = 8
print_r($arr); //=> ["hello", "not", 1, 2, 3, 15, 22, 23]

// Pop the last value
$val = array_pop($arr); //=> $val = 23

// Shift the first value;
$val = array_shift($arr); //=> $val = "hello"

// Remove value from index
unset($arr[1]); //=> removes value at index(key) 1.

// Reindex array (start indexes from 0 and so on). 
arary_values($arr);

// When new values added they get key value that is greater than previous
// maximum key.
$arr = [7 => 1];
array_push($arr, 99);
print_r($arr); //=> [7 => 1, 8 => 99];

// Array destructuring
$arr = [1, 2, 3];
[$a, $b, $c] = $arr;
echo "$a-$b-$c"; //=>1-2-3

// Array destructuring does not rquire to use all array items, you can assign some of the items (but sequentially).
$arr = [1, 2, 3, 4];
[$a, $b] = $arr; // $a = 1, $b = 2
[$a, $_, $_, $b]; // $a= 1, $b = 4

// The same result can be achieved with `list` function.
list($a, $_, $b, $_); // $a = 1, $b = 3 

// For destructuring string-key arrays you need to reference its keys.
$arr = ['a' => 22, 'b' => 59, 'c' => '441'];
['a' => $a, 'b' => $b, 'c' => $c] = $arr; // $a = 22, $b = 59, $c = '441'

// Referencing string-key arrays without key names raises warning.
[$a, $b, $c] = $arr; // THIS WON'T WORK!

// Mixing key names and just variables raises fatal error.
[$a, 'b' => $b, 'c' => $c] = $arr; // THIS WON'T WORK!

// Join array values into one string with `implode`.
// Implode signature `implode(delim:str, arr:array)`.
$arr = [1,2,3,4];
echo implode(', ', $arr); //=> `1, 2, 3, 4`

// Split a string into array with `explode`.
// Explode signature `explode(delim:str, str:string)`
$str = "1 2 3 4"
$arr = explode(' ', $str); //=> [1, 2, 3, 4]

// Count elements in array
count($arr); // => 4

// Sum elements in array
array_sum([1,2,3,4]); //=> 10

// Check if array key exists with array_key_exists($key, $array) function.
array_key_exists(1, [1,2,3,4]) // true
array_key_exists(99, [1,2,3,4]) // false
array_key_exists(4, [1,2,3,4,null]) // true, works even for nulls
array_key_exists('a' ['a'=>1,'b'=>2,'c'=>3, 'd'=>null]) // works with char keys

// Filter array items with `array_filter` function.
// Function signature: 
// array_filter(array $array, callable $filterFunc = null, int $mode = 0): array.
// If `filterFunc` is not provided then function filters out all falsy values.
array_filter([0, 1, 2, 3, null, false, []]); //=> [1,2,3]

// if provided array_filter will apply `filterFunc` to all array elements, creating new array that does not contain any element that that `filterFunc` returned true for.
array_filter([0, 1, 2, 3, null], fn($elem) => $elem > 1); // [2, 3]
array_filter([0, 1, 2, 3, null], fn($elem) => $elem % 2 !== 1); // [2, 4, null]

// Reset array keys after filtering.
array_values(array_filter([1,3,5,7,2,4], fn($elem) => $elem % 2 !== 1));

// The `mod` parameter allows to change the behaviour of the function and apply filtering to key or both keys and values. Defaults to 0, which means - only values.

// Apply a function to array items with `array_map`.
// Function signature:
// array_map(callable $mapFunc, array $array, array ...$arrays): array.
array_map(fn($item) => $item ** 2, [1,2,3,4]); // [1, 4, 9, 16]
array_map(fn($elem) => strtoupper($elem), ["hello", "world"]); // ['HELLO', 'WORLD']

// Use multiple values from multiple arrays in your mapFunc.
$firstNames = ["Sam", "Bob", "Jane"];
$lastNames = ["Smith", "Dow", "Jackson"];
array_map(
	fn($firstName, $lastName) => $firstName." ".$lastName, 
	$firstNames, 
	$lastNames
); // [Sam Smith, Bob Dow, Jane Jackson]

// Merge multiple arrays into single one with `array_merge.
// Function signature: array_merge(array ...$arrays): array.
// Values with numeric keys are appended to the first array.
array_merge([1,2,3], [4,5], [6]);// [1, 2, 3, 4, 5, 6]

// Values with identical string keys are overwritten by the latter ones.
array_merge(["one"=>1, "two"=>2], ["one"=>"ONE", "three"=>3], ["one"=>"ENO"]);
// [one => ENO, two => 2, three => 3]

// Create array slices with `array_slice` function.
// Function signature: array_slice(array $array, int $startIndex, ?int $sliceSize = null, bool $preserveKeys = false)
$arr = [1,2,3,4,5,6];
array_slice($arr, 0, 3); // [1,2,3], start from index 0, collect 3 items
array_slice($arr, 1, 2); // [2,3], start from index 1, collect 2 items
array_slice($arr, 3); // [4,5,6], start from index 3, collect all
array_slice($arr, 3, null, true); // start from index 3, collect all, copy corresponding indexes to resulting array. 

// PHP has multiple functions for sorting arrays. Arrays are sorted inplace.
// sort() and rsort() sort arrays in ascending or descending order, but reindex the array.
$arr = [5, 1, 2];
sort($arr);
print_r($arr); // [0=>1, 1=>2, 2=>5]

// This approach can cause problems, with string keys.
$arr = ["a"=>23, "b"=>2];
sort($arr);
print_r($arr); // [0=>2, 1=>23]

// To avoid reindexing use asort() or arsort()
$arr = ["a"=>23, "b"=>2];
asort($arr);
print_r($arr); // [b=>2, a=>23]

// If you want to sort by keys use ksort() or krsort().

// Generate sequences with `range()` function.
range('A', 'Z'); // ['A', 'B', 'C'..., 'Z']
range(1, 10); // [1, 2, 3..., 10]
range(1, 10, 3); // [1, 4, 7, 10]
range(10, 1, -1); // [10, 9, 8..., 1]

# Extract associated array into current scope variables with extract function.
# Won't work with digit based keys.
$arr = ['name'=>'John', 'age'=>23];
extract($arr);
echo "User name is {$name}, age {$age}"; // $name and $age derived from array keys
# If theere already is a variable with such a name you can either overwrite it (which is default or skip wit EXTR_SKIP global constant) 
extract($arr, EXTR_SKIP);
```
---
#### Pass by reference
```php
// If you assign a variable to a variable, the copy of its value
// gets passed to this variable. This new variable is independent
// from the original variable.
$var = 12;
$val = $var;
$var = 333;

echo $var; //=> 333
echo $val; //=> 12

// In reality values are not always get copied.
// PHP uses copy-on-write optimization, so value gets copied only if it
// is being mutated during the program execution.

// In contrast pass by reference creates an alias to a single object in
// memory. So two variables get bound to each other.

$var = 42;
$val =& $var;
$var = 333;

echo $var; //=> 333
echo $val; //=> 333

$val = 444;
echo $var; //=> 444
echo $val; //=> 444

// Arrays like most of the types are passed by value. Each time an array
// is passed to a function, the copy of it is created. Arrays can be passed
// by reference, this way two entities will point to single object in memory.
$arr = ["one" => "hello", "two" => "world"];
$copy = $arr;
$alias =& $arr;

$arr["two"] = "Joe";
echo $arr["two"]; // => 'Joe'
echo $alias["two"]; // => 'Joe'
echo $copy["two"]; //=> 'world'

// Function that recieves a copy.
function addItem($arr)
{
	$arr[] = "new val";
	return $arr;
}

$copy = addItem($arr); // Mutate a copy and return it.
print_r($copy); //=> ["one" => "hello", "two" => "Joe", 0 => "new val"]
print_r($arr); // ["one" => "hello", "two" => "Joe"]

// Function that receives a reference.
function addItemByRef(&$arr)
{
	$arr[] = "new val";
	return $arr;
}

$arr = ["one" => "hello", "two" => "world"];
$alias = addItemByRef($arr); // Mutate the $arr and assign reference to $alias
print_r($alias); //=> ["one" => "hello", "two" => "world", 0 => "new val"];
print_r($arr); //=> ["one" => "hello", "two" => "world", 0 => "new val"];
```
---
#### Functions
```php
// Functions defined with `function` keyword
function myFunc()
{
	function_body goes here;
}

// Function with required arguments
function myFunc($a, $b, $c)
{
	return "$a $b $c";
}

// Unlike JS, calling a function without required arguments raises Fatal Error
myFunc(); //=> Uncaught ArgumentCountError: Too few arguments to function my_func(), 0 passed

// Calling a function with extra arguments is OK.
myFunc(1, 2,3,4); //=> no warning.

// Function with required and variadic parameters.
// Variadic parameters stored in an array. If no args passed, array is empty.
function myFunc($a, ...$args)
{
	print_r($args);
}
myFunc(12, 11, 12); //=> [0 => 11, 1 => 12]

// Parameters with default value. All default values go after required ones.
function myFunc($required, $def = 11, ...$args)

// Pass arguments by reference
function myFunc(&$arg)
{
	$arg += 11; // $arg will be mutated after function call;
}

// Pass variadic arguments by reference.
// Note that `arg` and `...args` cannot be passed as values, only variables.
function myFunc(&$arg, &...$args)

// Null and array as default values
function myFunc($arr = ["one", "two"], $mod = NULL)

// Function parameter and return value type definition.
// Types are evaluated at run time, so type mismatch will trigger TypeError.
function myFunc(array $arr): integer

// Union types are allowed since version 8.3
function myFunc(float|int $var): float|int

// Type hinting for variadic functions also available
function myFunc(int|float ...$args)

// Array unpacking in function call
function myFunc($a, $b)
{
	return $a + $b;
}

$c = myFunc(...[1, 2]); //=> c=3 

// You can call a function with named parameters.
// Prefix a value with parameter name and a colon `:`.
function myFunc($param1, $param2)...
$var = myFunc(param1: 22, param2: 11);
// Be carefull with named parameters
$var = myFunc(param1: 22, 11); // Throws error, named argument cannot be 
                                // listed before positional.
$var = myFunc(11, param1: 22); // Throws error, value 11 has already been
// assigned to parameter `param1`, so calling `param: 22` will try to redefine
// this assignment which is prohibited.

// You can save local variables' states between function calls.
// This is called `static varaible`. To preserve its value, you should always return it.
function myFunc() 
{
	static $myVar = 22;
	return $myVar++;
}

echo myFunc(); # 22
echo myFunc(); # 23
echo myFunc(); # 24

// Unlike Python, in PHP regular functions cannot be used directly as values. 
// They need to be wrapped in quotes.
function foo($val) {
	...
}

function bar($val, $callback) {
	...
}

echo var(1, 'foo'); // function name wrapped in single quoters.

// Not super convinient. To overcome this inconvinience anonymous functions can be used.
// Anonymous functions can be created similar to JavaScript.
// You assign a function expression to a variable. Hence you need to end you function expression with semi-colon.
$sum = function($a, $b) {
	return $a + $b;
};

// Now you can call this variable or pass it to another function.
$sum(1,2);

// Anonymous funcions can have limited access to global scope by the use of `use` keyword. This keyword prevents functions from mutatint global variables.
// List global variables in use keyword parens.
$modificator = 32;

$sum = function($a, $b=1) use($modificator) {
	return $a + $b + $modificator; 
};

// Use $sum variable as a callback in another function.
function bar($val, $callback) {
	return $callback($val);
}

bar(1, $sum);

// The ultimate form of anonymous function is arrow functions.
$sum = fn ($a, $b=1) => $a + $b;

// Arrow functions have builtin limited access to global scope.
$sum = fn ($a, $b=1) => $a + $b + $glob; 

// Funcions as arguments can be type annotated with `callable` type.
function applyCallaback(int|float $val, callable $callback): int|float {
	return $callaback($val);
}
```
---
#### Classes and objects
```php
// Classes in PHP are defined with `class` keyword and curly braces.
class MyClass
{};

// Attributes and methods in classes can be public, protected and private:
//     - public - available for everyone
//     - protected - available for reading and modifying inside this class and its subclasses
//     - private - only available inside this class for modifying and available for reading for subclasses.
// By defalut all attributes and methods are public.
class Car
{
	public $maker = 'Car Inc.';
	private $engine = 'Engine Ltd.';
	protected $vin = '123df43df3112005';

	public function drive()
	{
		echo "driving ...";
	}
}

// Class instances are created with `new` keyword.
$myCar = new Car();

// Accessing attrbiutes and methods is done via arrow `->` operator.
echo $myCar->maker; //=> 'Car Inc.'
$myCar->drive(); //=> 'driving ...'
echo $myCar->engine; //=> Error
echo $myCar->vin; //=> Error

// Mutating the attributes is done the same way.
$myCar->maker = 'NewCar Inc.';
echo $myCar->maker; //=> 'NewCar Inc.'

// Static attributes, constants and methods are avaialable via double-colon `::` operator.
class Car
{
	public const WEHICLE_TYPE = 'Car';
	public static $maker = 'Car Inc.';
	public static function blink();
	{
		echo "blink-blink";
	}

}

echo Car::$maker; //=> 'Car Inc.'. Note that here we should use `$` operator
Car::blink(); //=> "blink-blink"

// Get the class name
echo Car::class;

// In most cases classes are initialized via the constructor method.
// Methods can access the object via `this` keyword.
// You need to declare instance variables first before use them in constructor; 
class Car
{
	public $maker;
	public $engine;
	
	function __construct($maker, $engine)
	{
		$this->maker = $maker;
		$this->engine = $engine;
	}
	public function drive()
	{
		echo "driving $this->maker car with $this->engine engine.";
	}
}

$myCar = new Car('Car Inc.', 'Engine Inc.');
$myCar->drive(); //=> `driving Car Inc. car with Engine Inc. engine.`

// Providing not enough arguments to constructor results in Fatal Error.

// It is also possible to provide default values for constructor parameters
class Car
{
	public string $maker;
	public stirng $engine;
	
	function __construct($maker = 'Car Inc.', $engine = 'Engine Inc.')
	{
		$this->maker = $maker;
		$this->engine = $engine;
	}
}

// Along with __construct special method there is a complementary method
// named __destruct. It is most usefull when you need to make some cleanup, 
// free up resources or close files and connections.
class FileHandler
{
	function __construct($fileName)
	{
		$this->fileObj = fopen($fileName, 'r');
	}

	function __destruct()
	{
		fclose($this->fileObj);
	} 

}

// PHP 8 introduced a shorthand class definition similar to Python's dataclasses.
// You need to add access modifiers to function parameters.
// For simple cases function body may be left empty.
class Client
{
	public function __construct(
		public string $name,
		public int $age
	) {
	}

}

// You can use a technique called `method chaining` to chain method calls.
// The main part of this technique is to return the instance in a method.
class Account
{
	public function __construct(
		public string $name,
		public int $balance
	) {
	}

	public function deposit(int $amount)
	{
		$this->balance += $amount;
		return $this;
	}

	public function sendUpdatedBalanceNotification()
	{
		echo "Your balance was updated to: $this->balance, $this->name";
	}
}

$johnAccount = new Account('john', 200);
$johnAccount
  ->deposit(100)
  ->deposit(100)
  ->sendUpdatedBalanceNotification(); // "Your balance was updated to: 400, john"


// Subclasses are defined via the `extends` keyword.
// A subclass can only inherit from one superclass. Multiple inheritance prohibited.
class Animal
{
	public $name;
	public $breatheSystem = "complex";
	private $privateVar = "private info";

	function __constructor($name)
	{
		$this->name = $name;
	}
	
	public function raiseVoice()
	{
		echo "Making sound";
	}
}

class Dog extends Animal
{
	public function raiseVoice()
	{
		echo "Bark!";
	}
}

$myDog = new Dog("beam");

// Dog inherits __constructor method and $breatheSystem variable from animal.
echo $myDog->breatheSystem; //=> 'complex'
$myDog->raiseVoice(); //=> 'Bark!'

// $privateVar variable is not available in subclass;
echo $myDog->privateVar; // => Raises `Undefined property` warning

// You can access parent methods via `parent` keyword
class Dog extends Animal
{
	public function __construct(string $name, string $familyRole)
	{
		parent::__construct($name); // calling parent constructor after child constructor code will override it.
		$this->$familyRole = $familyRole;
	}

	public function raiseVoice()
	{
		parent::raiseVoice();
		echo "Bark!";
	}
}
// Note: if you overriding a parent method (except for the __construct()), you have to keep child method's signature the same as parent method's signature.

// To prevent a class from modifying or inheriting from use `final` keyword.
final class MyClass... // Can't inherit from it.

// When used with methods `final` keyword keeps parent methods from overriding by children.
class MyClass
{
	final public function bar()
	{
	}
}
// Now overriding it in subclasses will be prohibited.

// Abstract classes are created by prefixing the access modifier with the `abstract` keyword.
// Abstract classes serve as blueprints for its subclasses. They can have both abstract and concrete methods. All abstract methods should be implemented by subclasses. You can't instatiate an abstract class.
// Abstract classes are used, when there is a clear parent->child relationship between classes and subclasses. If you want to inherit properties for unrelated classes it is better to use interfaces.
abstract class Electronics
{
	// class const will be passed to subclass directly
    public const TYPE = "Electronics";
    public string $producer;

	// regular __construct() method will be available for overriding 
    public function __construct($producer) {
        $this->producer = $producer;
    }

	// this abstact function should be implemented by subclasses
    abstract public function turnOn();
}

class MicrowaveOven extends Electronics
{
	// Concrete implementation of an abstract method
    public function turnOn()
    {
        echo "Microvawe was turned on";
    }
}

$m = new MicrowaveOven("LG");
$m->turnOn(); // "Microvawe was turned on"
echo $m::TYPE; // "Electronics"


// Abstract classes or generics can be implemented with interfaces.
// Interface is a collection of methods that define the behaviour of all
// the classes that implement this iterface without method details.
// Interface can not have variables, but can have constants.
interface Shape
{
	public function draw();
	public function calculateArea();
}

interface Radius
{
	public function setRadius($radius);
	public function getRadius();
}

class Circle implements Shape, Radius
{
	public $radius = NULL;
	
	public function draw()
	{
		echo 'Drawing a circle';
	}

	public function calculateArea()
	{
		echo 'Calculate area';
	}

	public function setRadius($radius) // any parameter name would work
	{
		$this->radius = $radius;
		echo "set radius to $radius";
}

	public function getRadius()
	{
		return $this->radius;
	}

}

$myCircle = new Circle();
$myCircle->draw(); //=> 'Drawing a circle'
$myCircle->calculateArea(); //=> 'Calculate area'
$myCircle->setRadius(11); //=> 'set radius to 11;'
echo $myCircle->getRadius(); //=> 11

// If you start with declaring an null stub instead of a class instance, you can use null-safety operator.
class MyClass
{
	public function hello()
	{
		return 'hello';
	}
}

function foo(mixed $arg)
{
	$instance = null; // start with empty stub
	
	if ($arg == 'create') {
		$instance = new MyClass(); // assign a new class instance
	}

	// Now we can have both null and not-null $instance variable. To avoid warning when calling the `hello` method use the `?` operator.
	$result = $instance?->hello();
	if (!$result) {
		throw new Error('$result not set');
	}
}

// Create aliases for classes
use DateTime as DT
$d = new DT();

// PHP have a rare used feature - Anonymous classes.
// It is used, when you have a throw away single-use class, typically to be supplied to another class. It is not a feature someone would use to often.
// Consider a case, when we have a FileProcessor class that expets a FileWriter iterface. If this interface implementation is used only once you may omit creation of a dedicated class and supply an anonymous class to the FileProcessor.
// This may be usefull for testing.
interface FileWriter
{
    public function write(string $file, string $data);
}

class FileProcessor
{
    public FileWriter $writer;
    public function __construct($writer) {
        $this->writer = $writer;
    }
    
    public function process(string $file, string $data) {
        echo "Processing a file...\n";
        echo $this->writer->write($file, $data);
    }
}

$procr = new FileProcessor(
	// Create a class and an instance at the same time and supply it to caller.
	new class() implements FileWriter {
    public function write(string $file, string $data) {
		    echo "Writing data `{$data}` to file `{$file}`...";
	    }
    }
);
$procr->process("users.txt", "Id=123, Name='Bob'"); // Processing a file...
// Writing data `Id=123, Name='Bob'` to file `users.txt`...


// You can iterate through object's public properties.
class MultiProp {
	public int $intProp;
	public string $strProp;
	public bool $boolProp;
	public array $arProp;
	
	public function __construct() {
		$this->intProp = 4;
		$this->strProp = 'Hello World!';
		$this->boolProp = true;
		$this->arProp = [74, 21, 85];
	}
}

$m = new MultiProp();
foreach($m as $prop) {
	var_dump($prop);
}
/*
	int(4)
	string(12) "Hello World!"
	bool(true)
	array(3) {
	  [0]=>
	  int(74)
	  [1]=>
	  int(21)
	  [2]=>
	  int(85)
	}
*/
```
---
#### Iterator interface
```php
// PHP has a builtin interface for creating iterable classes called `Iterator`.
/* A class that implements the Iterator interface is called `iterable`.
   Every iterable should implement 5 required methods from Iterator interface:
  - key(): mixed - returns current key;
  - current(): mixed - returns current element;
  - next(): void - switch state to the next element;
  - valid(): bool - determine should it iterate next or stop;
  - rewind(): void - return iterator to its initial state;

When you iterate through iterable object PHP performs next steps:
  1. Calls `rewind()` method to start from intial state.
  2. Calls `valid()` method before each iteration to determine if current iteration should be conducted.
  3.1. If valid() returns `false`, terminates the loop.
  3.2. If valid() returns `true`, calls the `key()` and `current()` methods.
  4. Evaluates loop body.
  5. Calls `next()` method to switch state.
  6. Repeats the process from step 2.
*/

// A simple example of ListIterator that lets you iterate through array elements.
class ListIterator implements Iterator
{
	protected array $items;
	protected int $idx;
	
	public function __construct(...$items) {
		$this->items = $items;
		$this->idx = 0;
	}
	
	public function key(): mixed {
		return $this->idx;
	}
	
	public function current(): mixed {
		return $this->items[$this->idx];
	}
	
	public function next(): void {
		++$this->idx;
	}
	
	public function valid(): bool {
		return $this->idx < count($this->items);
	}
	
	public function rewind(): void {
		$this->idx = 0;
	}
	
}

$it = new ListIterator('a', 'b', 'c');
foreach($it as $key=>$value) {
	echo "$key => $value\n";
}
/*
0 => a
1 => b
2 => c
*/
```
---
#### Working with files
```php
// You can open a file with `fopen` funciton.
// fopen has signature fopen(string fileName, string mode), where:
// fileName parameter its the name of the file like `reqs.txt` or `tmpl.html`
// mode can be one of `r`, `r+`, `w`, `w+`, `a`, `a+`;
// `r` - read file, `r+` - read and write;
// `w` - write into file, clean previous contents, creates file if it does not exist
// `w+` - write + read
// `a` - append to a file, `a+` - append and read, creates a file.

$requiremets = fopen('requirements.txt', 'r');

// To write data into file use `fwrite()` function.
// fwrite signature: fwrite(fileObj, data).
$todoList = fopen("todo-list.txt", "a");
fwrite($todoList, "[] Buy bread.\n")
fwrite($todoList, "[] Go to barber.\n")

// To close a file use fclose function;
fclose($todoList); //=> Returns true or false (if closing failed)

// To iterate over file contents the file function may be used.
// It accepts a file name and returns an array of lines, delimited by
// the EOL operator `\n`.
$contents = file('todo-list.txt');
foreach ($contents as $task) {
	echo $task;
}

// There are other functions for working with files and directories.

// View directory contents.
scandir('dir_name');
scandir(__DIR__); // current directory

// Create a directory
mkdir('new_directory');

// Remove empty directory
rmdir('new_directory');

// Create a file
touch('my_file.txt');

// Check if a file exists
file_exists('my_file.txt');

// Add contents to a file
file_put_contents('my_file.txt', 'These are the contents!\nGoodgye');

// View file contents
file_get_contents('my_file.txt');

// View file size in bytes
filesize('my_file.txt')

// Sometimes the file size may be cached in previous operations. To get actual file size it is better to drop cache before measuring the file size.
clearstatcache();

// Delete file
unlink('my_file.txt')
```
---
#### Error handling
```php
# You can supress non-fatal errors and warnings by prefixing a statement with the `@` operator.
[1,2,3][10] // - raises warning
@[1,2,3][10] // - does not raise warning
```
---
#### Mixing HTML with PHP code
```php
// Sometimes to get html highlighting we can mix it with PHP code like this.
<?php
$value = 10;

if($value === 11): ?>
	<h1>Hello world!</h1>
<?php elseif($value === 12): ?>
	<h1>Hello guys!</h1>
<?php else: ?>
	<h1>Hello srtanger!</h1>
<?php endif; ?>
```
---
#### Sleeping
```php
sleep(10) // result in seconds
```
---
#### Using other files from within current file
```php
// nav.php
<nav>
	<a href = "/">Home Page</a>
	<a href = "/about.php">About Page</a>
</nav>

// index.php
<?php include "nav.php" ?> 
Home Page

// If you reference one file in multiple parts of your php code use `include_once` to not load the contents twice.

// Also `require` and `require_once` are possible.
// about.php
<?php require "nav.php" ?>
About Page
```
---
#### Random values
```php
// Generate arbitrary random number.
rand();
# OR
mt_rand();

// Generate random number between boundaries (both included).
rand(0, 10);
# OR
mt_rand(0, 10);

// Shuffle an array inplace.
$ar = range(0, 10);
shuffle($ar);
print_r($ar); // [8, 9, 1, 3, 2...] 
```
---
#### Handling exceptions
```php
// Generating a new exception is done by using the `throw` keyword and supplying it a new Exception object (or its subclasses).
// Here we throw exception if array length is 0.
class ArrayPrinter
{
	public static function print_ar(array $array): void {
		if (count($array) === 0) {
			throw new \Exception('Array must not be empty.');
		}
		print_r($array);
	} 
}

ArrayPrinter::print_ar([]);

// There are also more specific exceptions that the parent \Exception class.
// Here we could use the `InvalidArgumentException`.
...
throw new \InvalidArgumentException('Array must not be empty.');
...
// Other builtin exceptions
- BadFunctionCallException
- DomainException
- InvalidArgumentException
- LengthException
- LogicException
- OutOfBoundsException
- OutOfRangeException
- OverflowException
- RangeException
- RuntimeException
- UnderflowException
- UnexpectedValueException

// To create a custom exception you can subclass from generic Exception or any other exception class.
// We can create a custom exception to handle the previous empty array case.
class EmptyArrayException extends \Exception {}

// Now we can use it just like previous exceptions.
...
throw new EmptyArrayException('Array must not be empty.');
...

// If the error message won't change depending on the situation, it can be embeded into the class definition.
// This will eliminate the need to pass the same message every time the error would occur, making exception handling more DRY.
class EmptyArrayException extends \Exception 
{
	protected $message = "Array must not be empty."; // No type annotation!
}

// Now we omit the exception message but the result stays the same.
...
throw new EmptyArrayException();
...

// Exception handling is done by try->catch operators.
// Code that is expected to fail is placed in the body of try operator.
// catch block accepts an exception class and a variable name to store the created exception object. 
try {
	ArrayPrinter::print_ar([]);
} catch (EmptyArrayException $e) {
	echo "Returning back to main menu";	
}

// Multiple exception classes are combined using the pipe operator.
try {
	ArrayPrinter::print_ar([]);
} catch (EmptyArrayException|\InvalidArgumentException $e) {
	echo "Returning back to main menu";	
}

// There may be multiple catch blocks.
try {
	ArrayPrinter::print_ar([]);
} catch (EmptyArrayException|\InvalidArgumentException $e) {
	echo "Returning back to main menu";	
} catch (\Exception $e) {
	$e->getMessage(); // you can access exception object properties.
	echo "Terminating session.";
}

// There also a `finally` statement to execute code no matter if the exception would be thrown or not.
try {
	ArrayPrinter::print_ar([]);
} catch (EmptyArrayException|\InvalidArgumentException $e) {
	echo "Returning back to main menu";	
} catch (\Exception $e) {
	$e->getMessage(); // you can access exception object properties.
	echo "Terminating session.";
} finally {
	echo "Program finished.";
}
```
---
#### Working with dates and time
```php
// Get current timestamp.
time(); // int(1736431499)

// Create a timestamp with `mktime` function.
// `mktime` signature: mktime(int $hours, ?int minutes, ?int seconds, ?int $month, ?int $date, ?int $year): int.
mktime(20, 15, 10, 1, 15, 2025); // 1736972110 === 2024-01-15 20:15:10

// Convert a timestamp to a string-based date representation with `date` function.
/* `date` function signature: date(string $format, ?int $timestamp=null): string.
- `$format` parametr defines the output format and must contain specific modifiers (https://www.php.net/manual/en/datetime.format.php).
- `$timestamp` parametr should contain a valid timestamp. If no timestamp provided, current timestamp (aka `time()`) will be used.*/

date('Y-m-d'); // '2025-01-09', current date, ISO formatted
date('D F y'); // 'Thu January 25', current date, American style
date('D F y', 1735321499); // 'Fri December 24', a moment in the past, American

/* Creating datetime objects with `DateTime` class. */
// A datetime object for current timestamp can be created without arguments.
$date = new DateTime();
var_dump($date); /* 
object(DateTime)#3 (3) {
  ["date"]=>
  string(26) "2025-01-09 14:48:31.320372"
  ["timezone_type"]=>
  int(3)
  ["timezone"]=>
  string(3) "UTC"
}
-- Note! DateTime object cannot be converted to string out of the box, so using `echo` will cause an exception. To print a datetime use `var_dump` function.
*/

// To create a specific datetime object pass a datetime-formatted string.
// The formatting will be implicitly calculated by PHP.
new DateTime("2025-01-09"); // "2025-01-09 00:00:00.000000", ISO formatted
new DateTime("01/09/2025"); // "2025-01-09 00:00:00.000000", American style

// To provide your own format use the DateTime::createFromFormat() method.
$weirdFormat = 'Y__m..d'
DateTime::createFromFormat($weirdFormat, '2025_01.09'); // "2025-01-08 15:09:16.000000" NOTE: the time part is taken from current time, it's not 0
// Also note, that if datetime does not follow the format string `false` is returned.

// By default a datetime object is created with the UTC timezone.
// To change the timezone, create a `DateTimeZone` object and pass it when creating a DateTime object.
$tz = new DateTimeZone('Europe/Moscow');
$date = new DateTime("2025-01-09 10:00:25", $tz);
// OR
$date = DateTime::createFromFormat('Y--m--d H:i:s', "2025--01--09 10:00:25", $tz);
var_dump($date); // "2025-01-09 10:00:z25.000000", timezone="Europe/Moscow"

// List of timezones is predefined in PHP.

// To adjust datetime to a different timezone use the `setTimeZone()` method.
$tz = new DateTimeZone('Europe/Moscow');
$date = new DateTime("2025-01-09 10:00:25", $tz);
$date->setTimeZone(new DateTimeZone('Europe/Berlin')); // "2025-01-09 08:00:25.000000", timezone="Europe/Berlin"

// It is also possible to change the date or time parts of a datetime object.
$date = new DateTime("2025-01-09 10:00:25");
$date->setDate(2024, 12, 24); // "2024-12-24 10:00:25.000000"
$date
	->setDate(2024, 12, 24)a
	->setTime(22, 10, 15, 100); // "2024-12-24 22:10:15.000100"

// Get a datetime object's string representation with the `format` method.
$date = new DateTime("2025-01-09 10:00:25");
echo $date->format('d/m/Y'); // "09/01/2025"

// Find difference between two datetime objects.
$date1 = new DateTime("2025-01-09 10:00:25");
$date2 = new DateTime("2025-01-09 11:00:25");
var_dump($date1->diff($date2)); // object(DateInterval), which has properties like `y`-for year, `h`-for hour and so on.
echo $date1->diff($date2)->h; // 1, means the difference = 1 hour
echo $date1->diff($date2)->y; // 0, means the difference = 0 years

// Another usefull method for modifying the datetime object is the `modify` method.
// You can pass a string modifier, and PHP will try to parse it and change the datetime object based on the parsed string.
$date = new DateTime("2025-01-09 10:00:25");
$date->modify('tomorrow'); // "2025-01-10 00:00:00.000000", transform to tomorrow midnight.
$date->modify('+1 day'); // "2025-01-10 10:00:25.000000", add one day to current datetime.
$date->modify('-1 month');
```
---
#### Composer
```bash
# Generate composer.json file
composer init

# Add directories for autoload
-- open composer.json and add new properties
"autoload": {
	"psr-4": {
		"Framework\\": "src/Framework", # namespace: path
		"App\\": "src/App"              # namespace: path
	}
}
-- Go to terminal and type
composer dump-autoload # Composer will generate the vendor directory and special autoload files

-- After that go to your bootstrap.php file and add
require __DIR__ . '/../../vendor/autoload.php';
```
---
#### Capturing string output with Output Buffer
```php
// Sometimes you need to capture output in order to use it later on.
// You can use output buffer to capture and release output on demand.
$count = performSomeCalculations($args); // create a variable to use in string
ob_start(); // We created an output buffer that captures all output.
echo "The result = {$count}\n";  // Output strings
echo "Congratulations! My friend!";
$contents = ob_get_contents(); // Everything that was captured dupmped into variable
ob_end_clean(); // stop the buffer interception and clean the buffer.
echo $contents; // The result = 1000\nCongratulations! My friend!
```
---
