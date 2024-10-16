#### Types
```js
// Primitive types
Number: 120, -220
BigInt: 10n // don't know what is this for
Boolean: true, false
String: 'hello'
Symbol: Symbol('hello') // don't know what is this for
Null Values: null, undefined, NaN

// Composite types
Array: [1,2,3,4]
Object: {a: 11, b: 22}

// Convert String to Number and vice-versa
console.log(Number('12')); //=> 12
console.log(Number('jehehs')); //=> NaN
console.log(String(823487743)); //=> '823487743'

// JS tries to convert strings into numbers in mathematical operations.
console.log("42" * 3); //=> 126
console.log("42" * "3"); //=> 126
console.log("42bwert" * 3); //=> NaN

// Falsy values
"" // empty string
-- null
-- 0, -0
-- NaN // when a conversion to number failed
-- false
-- undefined

// Some usefull constants
Number.MAX_VALUE; //=> 1.7976931348623157e+308 the largest number possible in JS.
Number.MIN_VALUE; //=> 5e-324 the lowest number possible in JS
Number.MAX_SAFE_INTEGER; //=> 2**53 - 1;
Number.MIN_SAFE_INTEGER; //=> -(2**53 - 1).
Number.POSITIVE_INFINITY; //=> is higher than any other number
Number.NEGATIVE_INFINITY; //=> is lower than any other number

// Be carefull, that array and null are objects too
typeof([1,2,3]); //=> 'object'
typeof(null); //=> 'object'
```
---
#### Variables
```js
/* Variables can be declared with `let` and `var` keywords. Type of variable is not declared. `let ` is the preferable way of declaring variables, unlike the `var` keyword declared variables they are available in the scope they were declared. `var`-defined variables can pollute the global scope. */
let val = 12;
val = [1,2,3,4];
val = "Hello world";

// Constants are declared with `const` keyword. Mutating them raises TypeError.
const val = 255;

// Primitive type variables are passed by value.
let a = 42;
let b = a;
a = "Hello";
console.log(`a = ${a}, b = ${b}`); //=> 'a = Hello, b = 42'

// Complex types like arrays and objects are passed by reference.
a = [1,2,3,4];
b = a;
a.push("Hello");
console.log(b); //=> [ 1, 2, 3, 4, "hello" ]
```
#### Comparison operators
```js
==   // loose equality, 12 == '12' => true
==== // strict equality, 12 === '12' => false
!=   // loose inequality, 12 != '12' => false
!==  // strict inequality, 12 !== '12' => true
<    // loose less than, 12 < '13' => true
<=   // loose less or equal, 12 <= '13' => true
>    // loose greater than, 12 > '11' => true
>=   // loose greater or equal, 12 >= '11' => true 
```
---
#### Strings
```js
// Strings in JS are immutable.

// Get string length
console.log('hello'.length); //=> 5

// Replacing characters is done via .replace(old, new) string method.
const str = "hello";
str.replace("l", "L"); //=> heLlo

// By default .replace impacts only the first occurance of target substring.
// To change this behaviour you can provide a regex with `g` flag
str.replace(/l/g, "L"); //=> heLLo

/* String literals allows variable interpolation into strings.
Works like f-strings in Python. 
Allows not only scalar values, but expressions too.
Use backticks `` to denote this string and inject variables with ${var} syntax.
*/
let name = "Bob";
console.log(`Hello, ${name}!`); //=> Hello, Bob
let num = 42;
console.log(`42 divided by 2 equals to: ${num / 2}`); //=> 21

// Spread string into array
[..."hello"]; // ["o", "l", "l", "e", "h"]

```
#### Arrays
```js
// Create array
let ar = [1, 2, 3, 4];
let ar = Array(1, 2, 3, 4);
let ar = new Array(1, 2, 3, 4);

// Create arrays with `spare` slots
let ar = [1, ,3, 4, 5]; //=> [1, <empty slot>, 3, 4, 5 ]
let ar = [ , , , , 5]; //=> [<empty slot>, <empty slot>, <empty slot>, <empty slot>, 5]
// Indexing
console.log(ar[0]); //=> 1
console.log(ar[99]); //=> undefined
console.log(ar[-1]); //=> undefined
console.log(ar[ar.length - 1]); //=> 4

// Push multiple elements
let ar = [1];
ar.push(2, 3, 4); //=> returns 3 - new array size
console.log(ar); //=> [1,2,3,4]

// Enqueue multiple elements
let ar = [1];
ar.unshift(-1, 0); //=> returns 3 - new array size
console.log(ar); //=> [-1, 0, 1]

// Pop the last element from array
let ar = [1,2,3];
ar.pop(); //=> returns 3
ar.pop(0); //=> pop ignores provided args, returns the next value = 2

// Shift the first element from array
let ar = [1,2,3];
ar.shift(); //=> returns 1
ar.shift(1); //=> shift ignores provided args, returns the next value = 2

// Check if array has a value
let ar = [11,22,33];
ar.includes(11); //=> true
ar.includes(1); //=> false

// Convert array into string
[1,2,3,4].join("-"); // "1-2-3-4"

// To check an object is array or not use `Array.isArray`
console.log(Array.isArray([])); //=> true
console.log(Array.isArray({})); //=> false

// Get index of array item
let ar = [9, 86, 15, 9];
ar.indexOf(9); // 0 - index of the first 9
ar.lastIndexOf(9); // 3 - index of the last 9
ar.indexOf(-141); // -1 for unexisting item

/* Mutate arrays with `splice` method. It removes elements from one arrays, and creates another array with removed items. It can add new items in original array simultaneously, thus acting like replacement function.
`splice` has following signature: */
array.splice([startIndex], [removeCount], [insertItem1, ...insertItemN])

// splice without arguments does nothing
let ar = [1,2,3,4];
ar.splice(); // => []
console.log(ar); // [1,2,3,4]

// splice with only the first argument will remove all elements from original 
// array starting from provided startIndex.
ar.splice(1); //=> [2,3,4] all items from index 1
console.log(ar); //=> [1]
ar.splice(29); // => [], to high index will return empty array 

// splice with negative index will count from right, starting from -1
ar = [1,2,3,4,5];
ar.splice(-1); //=> [5] - remove all elements starting from last element 
console.log(ar); //=> [1,2,3,4]
ar.splice(-10); //=> [1,2,3,4] to low negative index will remove all items
console.log(ar); //=> []

// Providing the second argument will limit the number of removed items
ar = [1,2,3,4,5];
ar.splice(1, 2); //=> [2,3], remove 2 elements starting from index 1
console.log(ar); //=> [1,4,5]

// To high `removeCount` will remove all items
ar.splice(0, 10); //=> [1,4,5]
console.log(ar); //=> []

// Negative `removeCount` has no effect
ar = [1,2,3,4,5];
ar.splice(0, -10); //=> []

// Using insert elements with `removeCount` set to 0 will insert new elements
// before the `startIndex.
ar = [1,2,3,4,5];
ar.splice(0, 0, 99); //=> []
console.log(ar); //=> [99,1,2,3,4,5]

// Replace items
ar = [1,2,3,4,5];
ar.splice(1, 2, 22, 33); //=>[2,3]
console.log(ar); //=> [1,22,33,4,5]

/* Create shallow slices of arrays with `slice` method. `slice` does not mutate the original array. `slice` has following signature: */
array.slice([startIndex], [endIndex]) // where endIndex not included

let ar = [1,2,3,4];
ar.slice(); //=> [1,2,3,4]
ar.slice(0, 2); //=> [1,2]
ar.slice(2, 5); //=> [3,4]
ar.slice(0, -1); //=> [1,2,3]
ar.slice(0, -10); //=> []

// Concatenate arrays with `concat` method.
[1].concat([2]); //=> [1,2]
[1].concat([11, 12], [11, 15]); //=> [1, 11, 12, 11, 15]
[1].concat([12, 13], 5); //=> [1, 12, 13, 5]

// Concatenation also possible with the `...spread` operator
[5, ...[1,2,3], [6,7,9]]; //=> [5, 1, 2, 3, 6, 7, 9]
```
---
#### Array Iteration
```js
// Iterate over array idecies
let ar = [11, 12, 13];
for (let i in ar) {
	console.log(ar[i]); // prints 11 12 13
}

// Iterate over array values
let ar = [5, 7, 8];
for (let a of ar) {
	console.log(a); //=> prints 5, 7, 8 on separe lines
}

/* `forEach` array method lets you apply a callable for each array element by attaching a callback. `forEach` has this syntax:
array.forEach(Callable[(item, ?index, ?array), Any], ?this): Any */
let ar = [11, 22, 33];
ar.forEach(
	function(item, index, arr) {
		console.log(item * 2, index, arr);
	},
	this
);
// The ouput prints array element first, element's index - second and the array itself third. This does not seem to have any impact on this.
	22, 0, [11, 22, 33]
	44, 1, [11, 22, 33]
	66, 2, [11, 22, 33]

// This method has a shorthand notation, which is preferably used
ar.forEach((item) => console.log(item * 2));

// It is also possible to mutate an array during iteration
ar.forEach((item, idx, arr) => (arr[idx] = item * 3));
console.log(ar); //=> [33, 66, 99]

/* `filter` method generates new array containing only values from original array that satisfy given filtering function. It has following syntax:
arr.filter(Callable[(item, ?index, ?arr), Any], ?this): []Any */

// Extract even numbers from original array.
let arr = [1, 2, 3, 4, 5, 6, 7, 8];
let evens = arr.filter((item) => item % 2 == 0);
console.log(evens); //=> [2, 4, 6, 8]
console.log(arr); //=> [1, 2, 3, 4, 5, 6, 7, 8], original arr not mutated

/* `map` method generates new array which contains all values from the original array with callable applies to each element. It has following syntax:
arr.map(Callable[(item, ?idx, ?arr), Any], ?this): []Any */

// Round every float down to the nearest integer
let arr = [98.11, .42, 11, 2.5677];
let rounded = arr.map((item) => Math.floor(item));
console.log(rounded); //=> [98, 0, 11, 2]

/* `every` and `some` array methods are similar to Python's `all` and `any` functions. `every` returns true if ALL elements satisfy a callable, `some` returns true if ANY of elements satisfy a callable.
Both function have signatures as the previous array methods. The receive a callable and optional this argument. Callable has 3 parameters, of whic the first one - the current item - is requiered.
*/

// Check if all elements in the array are positive
console.log([1, 2, 3, 4].every((item) => item > 0)); //=> true

// Check if at least one of the elements is even
console.log([1, 3, 5].some((item) => item % 2 == 0)); //=> false

/* `reduce` and `reduceRight` fold array values into single value.
These function have following signature:
arr.reduce(Callable[(accum, currentVal, ?index, ?array), Any], ?initVal): Any

Little breakdown of how this funcitons work: when the function starts, it checks for `initVal` parameter value. If initVal is set, it is used as the value for `accum` parameter, and all the array elements are treated as values for `currentVal` parameters. If the `initVal` is not set then the first array element is chosen as `accum` and the iteration start from the second element.
After that the function iterates over the array and on every step the current value is chosen as `currentVal` and `accum` is the result of all the previous operations. 
*/

// Sum all elements in the array
console.log([1,2,3,4].reduce((acc, val) => acc + val)); //=> 10

// Sum all elements in the array and add to 10
console.logt([1,2,3,4].reduce((acc, val) => acc + val, 10)); //=> 20

// `reduceRidght` does the same work, but it starts folding from the right 
// side of the array.
```
#### Objects
```js
// Object in JS are data containers much like maps or dictionaries.
// It can contain multiple data types, but its keys must be strings.
let obj = {
	num: 12,
	str: "string",
	arr: [1,2,3],
	bool: true,
	obj1: {
		one: 1,
		two: "two",
		},
	fn: function(a,b) {
		return a*b;
		},
};

// If object key contains only letter and digits, quotes may be ommited.
// Otherwise quotes are required.
let obj = {
	"Key with space": 42,
};

// To get value use bracket `[]` or dot `.` notation.
let a = obj["key"]
let a = obj.key

// For complex keys only bracket notation is available
let a = obj["Super Long"]

// To assign new value to existing key or to add new key:value pair 
// use the assignment operator `=`.
let ob = {key1: 42}
ob.key1 = 11;
ob["key1"] = 11;
ob.key2 = 22;
ob["key2"] = 22

// To delete a key: value pair use `delete` keyword.
delete obj.key1
delete obj["key2"]

// To check if a key exist use `hasOwnProperty` object method.
ob.hasOwnProperty("key1"); //=> true;
ob.hasOwnProperty["Do not exist"]; //=> false;

/* To unpack an object into variables define your variables inside curly braces and perfix this with let or const keyword. 
IMPORTANT!!!
    - names of variables must match names of keys in the object;
    - if you declare an unpacking variable with name, that was previously
      declared with a `let` or `const` keywords you will get an error 
*/

let obj = { one: 1, two: 2, three: 3 }
let { one, two, three } = obj;
console.log(one, two, three); //=> 1 2 3

let { some, another, third } = obj;
console.log(some, another, third); //=> undefined undefined undefined

let one = 3874884;
let obj = { one: 1, two: 2, three: 3 };
let { one, two, three } = obj; //=> raises SyntaxError: redeclaration of let one

/* Merge two dictionaries with Object.assign(target, source1, ...sourceN).
Object.assign() mutates the target object inplace, but also returns the newly created object. When source and target have same keys, source will override the target value. Later source will override prior source.
*/

let o1 = {one: 1, two: 2};
let o2 = {one: 3, three: 5};
Object.assign(o1, o2); // => Object { one: 3, two: 2, three: 5 }
console.log(o1); // => Object { one: 3, two: 2, three: 5 }

let o1 = {one: 1, two: 2};
let o2 = {one: 11, three: 3};
let o2 = {one: 111, three: 33};
Object.assign(o1, o2, o3); //=> Object { one: 111, two: 2, three: 33 }
console.log(o1); //=> Object { one: 111, two: 2, three: 33 }
```
---
#### Object iteration
```js
let obj = { one: 1, two: 2 } 
// Iterating object's keys is done via `for .. in` construct.
for (let key in obj) {
	console.log(obj[key]); // 1 2
}

// Also possible to iterate over an object using Object's methods
for (const key of Object.keys(obj)) {
	console.log(key); // one two
}

// Iterate over values
for (const value of Object.values(obj)) {
	console.log(value); // 1 2
}

// Iterate over key:value pairs
for (const [key, value] of Object.entries(obj)) {
	console.log(key, ': ', value); // one: 1 two: 2
}
```
#### Classes and OOP
```js
/* Classes in JS were added relatevely late. Syntactically they look similar to PHP's classes */

// Declare a class
class Animal {
	#privateProperty; // Private proerties must be declared upfront

	static classVariable = "Global class variable";

	static staticMethod() {
		return "This is static";
	}

	instanceMethod() {
		console.log("Instance method declared with parens and without keywords");
	}

	constructor(name, numLegs, countDown=10) {
		this.name = name;       // accessing the object itself is done via
		this.numLegs = numLegs; // the `this` keyword
		this.countDown = countDown;
		this.#privProp = "private";
	}

	["computed" + "Property"] = "computedProp";

	["computedInstance" + "Method"]() {
		console.log("Object properties can be generated dynamically");
	}

	*instanceGenerator() {
		for (let i = 0; i < this.counter; i++) {
			yield i;
		}
	}
}

// Class variables and methods must be declared with as static.
// Static are available without instantiation.
// Static properties are not available from class instance.
console.log(Animal.classVariable); //=> "Global class variable"
console.log(Animal.staticMethod()); //=> "This is static"

// Instaces are created with the `new` keyword and parens.
let anim = new Animal(4, "bob"); // let the countDown be default 10
console.log(anim.name); //=> 4
console.log(anim.numLegs); //=> 'bob'
console.log(anim.countDown); //=> 10
anim.instanceMethod(); //=> "Instance method declared with parens and without keywords"
console.log(anim.computedProperty); //=> "computedProp"
anim.computedInstanceMethod(); //=> "Object properties can be generated dynamically"
let gener = new Animal().instanceGenerator(); //=> a generator created
console.log(anim.privProp); //=> undefined (no access to private property)

// JS will not complain, you do not provide arguments required in constructor.
// All the corresponding properties with missing values will be set to undefined.
let undef = new Animal();
console.log(undef.name, undef.numLegs, undef.countDown); //=> unedfined, undef...

/* Sometimes it is suitable to declare constructor variables beforehand, to indicate about some maybe hidden or not so obvious properties. This is done by declaring variables in the class body. This variables won't be accessbile from anywhere, and will get their values when the constructor method will be invoked*/
class MyClass {
	name = 12;
	numLegs;

	constructor(name, numLegs) {
		this.name = name;
		this.numLegs = numLegs;
	}
}

// Classes can also be declared as anonymous, bound to a varaible.
// This is usefull for debugging purposes.
let Animal = class {
	...;
};

/* One class can inherit from the other using the `extends` keyword */
class Dog extends Animal {
	...;
}

// Access to parent class is done via the super keyword
class Dog extends Animal {
	constructor(name, numLegs, newArg, countDown=20) {
		super(name, numLegs, countDown);
		this.newArg = newArg;
	}
}

// Instantiating a Dog class now add everything from parent Animal, plus its own `newArg` property.
let d = new Dog('Bob', 4, 'Secret');
console.log(d.name, d.numLegs, d.newArg, d.countDown);// => 'Bob 4 Secret 20'


// Super can also call parent class methods
class Dog extends Animal {
	instanceMethod() {
		super.instanceMethod();
		console.log("Dog added to the Animal method");
	}
}
```
#### Null values
```js
// JS has two null values - `null` and `undefined`;
// null - is an empty value, it is used intentional to mark a variable 
// has no meaningful value.
let val = null;
val === null; //=> true
val > 0; //=> false
val < 0; //=> true
val > -1; //=> true

// unefined - means complete absence of value. 
// Undefined occurs when:
// a) you set an empty variable;
	let val; //=> val's value is undefined.
// b) access an object's missing key;
	let obj = {key: 11};
	obj.missing; //=> undefined;
// c) access invalid array or string index
	[1,2,3][22]; //=> undefined;
	"hello"[22]; //=> undefined;
// d) no argument provided to function;
	function func(arg){return arg;}
	func(); //=> undefined;

// You can check the undefined variable
val === undefined;

// Accessing object's nested missing keys raises TypeError.
let obj = {key: 42};
obj.key.key; //=> returns undefined, because `key` is in obj.
obj.missing.key; //=> raises TypeError.

// To safely access object's nested missing keys use question mark
// operator `?` after the first key in the chain.
obj.missing?.key; //=> returns undefined;
obj.missing?.key.key2.key3; //=> returns undefined;

// There is a shorthand for checking a variable against null values
// and using a default value instead if necessary. Kind of short form 
// for ternary operator. It is called null coalescing, and its syntax looks
// like val1 ?? val2. It returns the second value if the first is null or
// undefined, and returns first value otherwise.
null ?? 1; //=> returns 1;
undefined ?? 1; //=> returns 1;
0 ?? 1; //=> 0;
```
---
#### Switch statement
```js
// Switch statemnt is similar to php's switch statement.
let myVar = 22;
switch (myVar) {
	case 12:
	case 20:
		console.log("hello");
		break;
	case 22:
		return "bye";
	default:
		console.log('finish');
}

// If you want more conplex logic than just comparing values for equality
// you can use `true` in switch statement initial clause.
switch (true) {
	case myVar > 22 && myVar < 15:
		console.log(...);
		break;
	case myVar % 5 == 0:
		return ...;
}
```
---
#### Functions
```js
// Immutable primitive types such as numbers or strings will not be modified
// when passed to a function. Complex types, such as objects, arrays or functions
// will mutate the original entity if mutated during function execution.
let a = 12;
function myFunc(arg1) {
	arg1 /= 6;
	return arg1;
}
myFunc(a); //=> 2
console.log(a); //=> 12

// Global scope vaiables are available in function scope.
let a = 12;
function myFunc() {
	a /= 2;
	return a;
}
myFunc(); //=> 6
console.log(a); //=> 6

// By default all parameters in function signature are optional.
// If you supply arguments than parameters, missing values will be filled
// with undefined.
function myFunc(arg1, arg2, arg3) {
	console.log(arg1, arg2, arg3);
}
myFunc(1); //=> 1 undefined undefined

// Default values provede with assignment `=` operator.
function myFunc(arg1, arg2 = 12, arg3 = 'Hello') {
	console.log(arg1, arg2, arg3);
}
myFunc(1); //=> 1 12 Hello
myFunc(); //=> undefined 12 Hello

// Result of a function execution without return statement is undefined.

// Function can return only one value. For multiple values use objects or arrays.
function myFunc(arg1, arg2) {
	return {
		name: arg1,
		age: arg2,
	};
}

// Anonymous functions are declared without a name. They can be usefull, when
// declaring a callback in another function, or when creating an object with
// callables that will be invoked by accessing a key. This way of function
// declaration is called `function expression`.
function complexFunction(function (param1) {
	...
})

const funcStore = {
	func1: function (param1) {
		...
	},
	func2: function (param) {
	...
	}
};

const helper = function (param) {
	...
};

// Anonymous functions are useful as closures.
function applyFunc(func) {
	return function (a, b) {
		return func(a, b);
	};
}

function adder(a, b) {
	return a + b;
}

const addTwoNums = applyFunc(adder);
console.log(addTwoNums(1, 2)); //=> 3

// Variadic parameters in function signature is declared with spread operator.
function printer(...args) {
	console.log(args);
}

printer(1,2,3,4); //=> [1, 2, 3, 4]

// Spread operator is also used for unpacking function results into variables.
function myFunc(a, b, c) {
	return [a, b, c];
}

let [x, y, z] = myFunc(1, 2, 3);
console.log(x, y, z); //=> 1, 2, 3

// Functions can use other function that were declared after this function declaration. This is called `hoisting` (going up from bottom)
function foo(a) {
	printer(a);
}

function printer(a) {
	console.logt(a);
}

foo("hello world"); //=> prints 'hello world'

/* Arrow anonymous functions are declared without the `function` keyword.
The use parens to denote function parameters and declare the returning result after the arrow `=>` opeartor.*/

// These three functions do the same task.
function adder(a, b) {
	return a + b;
}

const adder = function (a, b) {
	return a + b;
} 

const adder = (a, b) => a + b;

// By default arrow function allow only one line before semicolon in the return part. This can be leveraged by enclosing return code in braces.

const func = (a, b, c) => {
	a = b % 2 + c;
	b = a / c;
	c /= 14.5;
	return a + b +c;
}

/* Immediately Invoked Function Expression (IIFE) - is the when you execute the function immediately after its definition. It can be usefull, when you need to declare variables (for example to make some calculations), but don't want to pollute current scope with this varaiables. */

// Imagine we have a main function, and inside of it we produce some calculation.
function main(...args) {
	let myVar = 42;
	let x = (
		function() {
			let x = 11;
			let y = 4522;
			const z = 1;
			y *= 22;
			x += y;
			x /= z;
			return x;
		}
	)(); // -- here we call the function passing its return value to x variable
	console.log(`myVar = ${myVar}, x = ${x}`);
}
console.log(main(112, 445, 0)); //=> 'myVar = 42, x = 99495'

// Let's try to access any of x's anonymous function local variables.
function main(...args) {
	let myVar = 42;
	let x = (
		function() {
			let x = 11;
			let y = 4522;
			const z = 1;
			y *= 22;
			x += y;
			x /= z;
			return x;
		}
	)();
	console.log(`y = ${y}, z = ${z}, x = ${x}`);
	console.log(`myVar = ${myVar}, x = ${x}`);
}
console.log(main(112, 445, 0)); //=> Uncaught ReferenceError: y is not defined

/* Generator-functions are be declared by postfixing the `function` keyword with an asterisk `*` operator. Thiese functions create generators when called. After creating generators can be iterated over. */
function* numbers(n) {
	for (let i = 0; i < n; i++) {
		yield i;
	}
}

// To iterate over the generator use `for...of` syntax
for (let a of numbers(10)) {
	console.log(a); //=> prints numbers from 0 to 9;
}

// Functions inherit from objects, so it is possible to treat the as objects
function letter(message) {
  this.message = message;
}

letter.prototype = {
  sendLowerCase: function () {
    return this.message.toLowerCase();
  },
  sendUpperCase: function () {
    return this.message.toUpperCase();
  }
}

let letr = new letter("This Is A Message");
letr.sendLowerCase(); // "this is a message"
letr.sendUpperCase(); // "THIS IS A MESSAGE"
```
---
#### Working with document
```js
/* Document is the object that represents current html page. It is called so because of the DOM term which stands for Document Object Model. It represents a tree of html objects and their attributes. The root object in the tree is <document> itself which contains one child - the <html> tag, which has two children - <head> and <body> tags.
*/

// To get an element from page by its `id` property use `getElementById` method
const mainTitleElement = document.getElementById('title');

// To get its contents use the `textContent` property
mainTitleElement.textContent

// You can also get a value for a CSS selector using querySelector method. 
// Here we get and element with html id person
let person = document.querySelector("#person");

// Get css clsass .pet
let pet = document.querySelector(".pet");

// Get an element with class `pet` and id `dog`
let dog = document.querySelector("#dog .pet");

// Get all the <li> elements enclosed in <ul> tag
const listItems = document.querySelectorAll("ul > li");

// Working with buttons
let button = document.querySelector("#button1");

// Change the button text
button.innerText = "New text";

// Add callable that will be invoked if button is clicked
button.onclick = function () {button.innerText = "Old Text"};

// Get the inline style of an element
let paragrapgh = document.querySelector("p");
paragraph.style; //=> get access to the style objectd
paragraph.style.display = "block"; //=> set paragraph's display property to block

// Modify HTML text inside tags
let demo = document.querySelector("#demo");
demo.innerHTML = "Something else";

// Create new element
let uList = document.createElement("ul");
uList.textContent = "Purchase List";
document.appendChild(uList);

let listItem1 = document.createElement("li");
listItem1.textContent = "Chicken - 2 kilos";
uList.appendChild(listItem1)

let listItem2 = document.createElement("li");
listItem2.textContent = "Pork - 1 kilo";
uList.appendChild(listItem2)

/* Event Listener - is a callabale that can be `attached` to a document element and invoked when a specific action is performed on this element. To add an event listener first specify the target action, like `click` and provide the callback after. */
let button = document.getElementById("btn1");
button.addEventListener("click", () => alert("You voice was counted"));
```
---
#### Math object
```js
// Math global builtin object contains functions and constants for producing
// mathematical operations.
Math.random(); //=> produces a float from 0 (inclusive) to 1 (exlusive)
Math.floor(N); //=> round down to the nearest integer
Math.ceil(N); //=> round up to the nearest integer
Math.abs(N); //=> return absolute form of a value
Math.min(...args); //=> return the minimum value
Math.max(...args); //=> return the maximum value
```
---
#### Control flow
```js
/* ---------------------------Ternary operator--------------------------------*/
<condition> ? valueIfTrue : valueIfFalse;
1 > 0 ? console.log(11) : console.log(-1);

/* ----------------------------if_else if_else--------------------------------*/
if (condition1) {
	...
} else if (condition2) {
	...
} else {
	...
};

/* ---------------------------Switch statement--------------------------------*/
// Switch statment check multiple cases. All cases are not strict, meaning a match case does not terminate the following execution of other cases. So you need to add `break` or `return` operators, if you wish to short circuit.  
let a = 11;
switch (a - 1) {
	case 9:
	  console.log("This is 9!");
	  break;
	case 11:
	case 12:
	  console.log("This is 11 or 12");
	  break;
	case 10:
	  console.log("This is 10");
	  break;
	default:
	  console.log("This is something else");
}

// By default `let` and `const` keywords are not allowed in case blocks.
// To fix this use curly braces in case statement.
switch (a - 11) {
	case 9: {
	  const message = "hello";
	  break;
	}
	case 10: {
	  const message = "world";
	  break;
	}
	default: {
	  const message = "something"
	}
}

// It is also possible to simulate if-else block with `switch(true)`
let a = 11;
let b = 12;
switch (true) {
	case a > 12 && a % 2 != 1:
		//
		break;
	case b - 11 == a -0:
		//
		break;
}
```
---
#### Loops
```js
// While loop
while (condition) {
	///
};

// Infinite while loop
while (true) {
	///
};

// For loop
for (initExpressions; endExpression; onEachStepExpression) {
	///
};

// Example
for (let i = 0; i < 20; i++) {
	...
};

// All three parts can be omitted
let i = 0;
for ( ; ; ) {
	if (i > 3) {
	  break;
	}
	
	console.log(i);
	i++;
};

```
---
#### Regular Expressions
```js
// Regular expressions are declared with enclosed in slashes `/pattern/`
const regex = /hello/;

// Escaping special characters is done by prefixing the char with one backslash
const regex = /\+hello\./; // find a string `+hello.`

// Shorthand charachters are preceeded with a backslash
// They can be `\s` - for spaces, or `\d` - for digits
const regex = / hello /; // find a string `hello` surrounded with spaces

// Brackets are used to list characters that must be matched.
// No need to use escaping insed brackets. 
const regex = /[a,b,Z,_]/; // match either of a, b, Z or _

// Flags are added after the closing slash
const regex = /[+-]/g; // g - means global search, it matches all the occurances 
                      // of pattern
const regex = /[z]/i; // i - means insensitive, matches lower and uppercase

// Use `match(pattern)` string method to find matches. It returns a list
// of a matched occurance with some additional info or null if no occurance found
const words = "hello, world, hello, people, HELLO";
words.match(/hello/); //=> ['hello', index: 0, input: "hello, world, hello, people, HELLO", groups: undefined].
// first argument - what was matched;
// second argument - first index in original string where the match occured;
// third argument - the original string;
// fourth argument - the captured groups.

// We receive `bare` arrays with values when using  the `global` 
words.match(/hello/g); //=> ["hello", "hello"], return all lowercase "hello"
words.match(/hello/ig); //=> ["hello", "hello", "HELLO"], return all "hello"
words.match(/123123/g); //=> null
```
---
#### Get info about entity's type
```js
// Use `typeof(entity)` funciton
console.log(typeof('a')); //=> "string"
console.log(typeof(2)); //=> "number"
console.log(typeof([1,2,3,4])); //=> "object" ?? Very odd
console.log(typeof({a: 1})); //=> "object" 

// Also possible `typeof X`
```
---
#### Working with dates
```js
/* The `Date` class represents a datetime object. It can accept a string or numbers that represent each fraction of datetime. */

// Define Date with numbers. Note that month count start with 0.
let d1 = new Date(YYYY, MM, DD, HH, mm, SS);
let d1 = new Date(2024, 9, 5, 11, 25, 22); // 2024-10-05 11:25:22

// Defining dates with strings accepts several formats.
// ISO format
let d1 = new Date('2024-10-05 11:25:22 GMT+0300'); // 2024-10-05 11:25:22

// American format
let d1 = new Date('01/13/2024'); // 2024-01-13 00:00:00
let d1 = new Date('01-13-2024'); // 2024-01-13 00:00:00

// Get difference between two dates
let d1 = new Date(2024, 9, 5);
let d2 = new Date(2024, 9, 4);

let diffMillSec = d1.getTime() - d2.getTime(); // find difference in milliseconds
let diff = Math.round(diffMillsec / (1000 * 3600 * 24));

// Usefull Date methods
d1.getFullYear(); // return date's full year
d1.getMonth(); // return the month number counting from 0
d1.getDate(); // return the day part from the date

// Get current date. Create new Date object without args 
new Date(); // Date Mon Oct 07 2024 15:18:21 GMT+0300 (Moscow Standard Time)

// Dates can be compared directely
new Date() > new Date(1999, 1, 1); // true
```
---
#### Error Handling
```js
// Error handling is done via `try {catch}` construct
try {
	// your code
} catch (err) {
	// handle error
}

// To throw an error use `throw`
throw Error("This is a generic error");
throw(TypeError("This is a type error"));
```
---
#### Sets
```js
/* Sets behave much like in Python. Thye store unique value and provide a faster lookup. */

// Creating a set
let newSet = new Set([1,2,3,4]);
console.log(newSet); // Set(4) {1, 2, 3, 4}

// Create a set from array
let ar = [1,2,3,4];
new Set(ar); // Set(4) {1, 2, 3, 4}

// Convert set to array with spread operator
let ar = [...newSet]; // Array(4) [1,2,3,4]

// Add new item
newSet.add(5); // Set(5) {1, 2, 3, 4, 5}

// Add existing item. Nothing happens
newSet.add(5); // Set(5) {1, 2, 3, 4, 5}

// Check if value present in set
newSet.has(1); // true

// Remove existing element
newSet.delete(5); // true // newSet is now Set(4) {1, 2, 3, 4}

// Remove element does not exist
newSet.delete(5); // false // newSet remains the same

// Get the size of a set
newSet.size; // 4

// Iterating over set
for (const item of newSet) {
	console.log(item); // 1 2 3 4
}

// The same result possible with
for (const key of newSet.key()) {
	console.log(key);
}

// The same result possible with
for (const value of newSet.values()) {
	console.log(value);
}

// The same result possible with
for (const [key, value] of newSet.entries()) {
	console.log(key, '---', value);
}

// Remove all values
newSet.clear();

// Sets also have comparing methods 
-- difference
-- symmetricDifference
-- union
-- intersection
-- isSubsetOf
-- isSupersetOf
-- isDisjointFrom
```
---
#### Get properties of JS data types
```js
// Get properties of type Array
Object.getOwnPropertyNames(Array) // ["isArray", "from", "fromAsync", "of", "prototype", "length", "name" ]

// Get properties of array instance
Object.getOwnPropertyNames(Array.prototype) // [ "length", "toString", "toLocaleString", "join", "reverse", "sort", "push", "pop", "shift", "unshift", … ] and 30 more

// Get all the Object's methods
for (const m of Object.getOwnPropertyNames(Object)) {
	if (typeof(Object[m]) == "function") {
	  console.log(m);
	}  
};

```
---
#### Destructuring of arrays and objects
```js
// Array desctructuring
let [a, b] = [1,2];
console.log(`a=${a}, b=${b}`); // a=1, b=2
let [a, b, ...rest] = [1,2,3,4,5];
console.log(`a=${a}, b=${b}, rest=${rest}`); // a=1, b=2, rest=3,4,5

// Object destructuring. Need to remember to name vars as object keys
let obj = {name: "John", age: 23};
let { name, age } = obj;
console.log(`name=${name}, age=${age}`); // name=John, age=23

let obj = {name: "John", age: 23, email: "gmail", random: 11};
let {name, age, ...rest} = obj;
// name=John, age=23, rest={email:"gmail", random:11}

// If you do this on the fly (without variable declaration) enclose the whole expression in parens
({ a, b } = { a: 1, b: 2 });
```
---
#### Asynchronous Code Execution
```js
/* Initially JS was a single threaded with synchronous execution runtime. As code was getting more complicated the need to execute asynchrounous code emerged. First functions delay code execution were `setTimeout` and  `setInterval`. They have the same API and allow to delay code execution without blocking the main thread. This function have the following signature:
setTimeout|setInterval(function, milliSec, arg1, ...argN).
Both functions receive function to execute, delay time in milliseconds and optional arguments to pass to the function. `setTimeout` executes the function once with a delay, while `setTimeout` executes the function continiously after specified delay.

Both mehods are handled by the browser (or other runtime) WebAPI. They are added to the Task Queue and pushed onto the call stack once the timer has expired. This way the main program thread is not blocked while waiting for the execution of timed functions.

The API of this functions is pretty limited. The result of calling this functions - is a unique timerId. There is no way of returning the result. You can only stop the function execution, by calling the `clearTimeoun|Interval` functions.
*/

// Here some basic examples
// Call console.log function with "Hello world!" argument after 2 seconds
let timeoutId = setTimeout(console.log, 2 * 1000, "Hello", "world!") //=> 216

// Delete the timeout
clearTimeout(timeoutId);

// Call console.log function with "Hello world!" argument every 5 seconds
let intervId = setInterval(console.log, 5 * 1000, "hello", "world");

// Delete the interval
clearInterval(intervId);

/* Execution flow of synchronous and timed functions.
The JS runtime (browser, NodeJS or others) has additional constructs, such as browser's WebAPI, to facilitate async function exection. These constructs usually have additional data structures, such as queues and stacks, and mechanisms to track timing, etc.

The overall runtime architecture consists of the call stack, the Microtask Queue for Promise execution and the Task Queue for delayed function execution.
The call stack has the highest priority and executes first, after the call stack the execution loop checks the Microtask Queue and pushes all its members onto the call stack. After that the execution loop checks the Task Queue and pushes its members onto the call stack. This concludes one iteration of the execution loop. Execution loop repeats this algorithm while there is anything in the stack and queues. 

The execution of a program goes secquentually from top to bottom.
  - Plain functions are pushed to the call stack and executed as soon as possible. All nested functions are also pushed to the stack and the last function executed first. 
  - Timed functions (`setTimout` and `setInterval)` are pushed onto the call stack too. The runtime will recognize a timed function, delegete its execution to WebAPI and pop it out of the call stack.
  - WebAPI will track the timer expiration using its inner mechanics. After the timer has expired, the function call with arguments from timed function will be put into WebAPI's Task Queue. 
  - The execution loop will first execute everytihng that's in the call stack, then check the Task Queue and push its memebers onto the call stack. */

// The async execution of timed functions can be seen in this example.
setTimeout(console.log, 5*1000, "hello");
console.log("goodbye");
// => 'goodbye' ... 5 sec later ... 'hello'

/* First the `setTimeout` is pushed to the call stack, then its execution is delegated to the WebAPI and it's popped from the stack. After the `console.log` is pushed onto the call stack and executed, producing "goodbye". Then the `console.log` is popped from the stack. The execution loop checks for Microtask Queue and the Task Queue, and since 5 seconds haven't passed yet, it produces nothing. After 5 seconds have passed the WebAPI puts the timed function in the Task Queue. The execution loop checks the Task Queue and pushes the `console.log("hello")` from the Task Queue onto the call stack. With new iteration the execution loop checks the call stack and executes its only member. Which produces "hello".
*/
```
---
#### Promises
```js
/* Promise - is another way of executing functions asynchronously.
Promise - is an absraction, that represents some code that will be executed asynchronously. Promises are executed through the Microtask Queue. New promises are put into the Microtask Queue and their execution is done after the call execution. That's why promises are executed before timed functions.

Promise require the main function  two callbacks to handle the result of the code execution, one callback - for the successfull execution, another - for handling the error.
*/

// Promise API looks like following
const promise = new Promise((resolve, reject) => {
	const result = myFunction(myVar);
	if (result == "hello") {
		resolve("It was greeting");
	} else if (result == "Goodbye") {
		resolve("It was goodbye");
	} else {
		reject("It was something else");
	} // For the if branches the `resolve` callback will be called.
}    //  For the else branch the `reject` callback will be called. 
); 

// Executing the promise is done via `then` and `catch` methods
promise
      .then(message => console.log(message))
      .catch(message => console.log(message));


/* A promise can be in of three states:
   - pending -> it has not been executed yet;
   - fulfilled -> it has been ececuted and resolved
   - rejected -> it has been executed and rejected

/* The argument to the `resolve` callback in the promise body will be passed to the callback in `then` method. The same with `reject->catch` binding.
Promise can have muiltiple `then` and `catch` methods producing a chain of callback that can use the result in their own way. The only requirement - is to always return the value being passed, to keep the chain going.
*/

// Consider this example
const promise = new Promise((resolve, reject) => {
	const result = myFunction(myVar);
	if (result == "hello") {
		resolve(42);
	} else if (result == "Goodbye") {
		resolve(22);
	} else {
		reject(-1);
	}
});

promise
	  .then(value => value *= 2)
	  .then(value => value /= 2)
	  .then(value => value += 2)
	  .then(value => ++value)
	  .catch(value => console.log(`invalid value ${value}`))
	  .finally(() => console.log("Promise completed")) // optional block
	  // will return 45 if result == 'hello'

/*`then` callbacks are being executed sequentially. If an error occurs during result handling it is passed to the proper error handler, if it defined, and the resolving of then calbback continues starting after the error handler. If the error handler concludes the chain, no code is executed after. Important note that if error occurs in one of the `then` handlers the value won't be passed down the chain. */

// A real world example using the `fetch` function. `fetch` returns a promise, so to handle its results you need to use `then` and `catch` methods.
let result = fetch('https://api.example.com)
			 .then(response => response.json())
			 .then(data => console.log(data))
			 .catch(err => console.log(err));

/* It is also possible to group multiple (usually related) promises and apply some additional logic to their execution results. */
  - Promise.all(...promises) /* - accepts multiple promises, executes them concurrently and returns their results in array, if all of them executed successfully. If any of the promises is rejected, then execution of other promises is aborted and the first error is returned. It is usefull, when you have interrelated promises, and you need all of them to be executed, not just part. */
Promise.all([promise1, promise2, promise3])
       .then(messages => console.log(messages))
       .catch(error => console.log(error));

  - Promise.allSettled(...promises) /* - acts the same way as `Promise.all`, but executes all the promises no matter how they resolved. It returns an array of executed promises, that may be in `fulfilled` or `rejected` state. Since this function deals with errors, there is no need to use `catch` method. */
Promise.allSettled([promise1, promise2, promise3])
	   .then(messages => console.log(messages));
// [
//   { status: 'rejected', reason: 'Error From Promise One' },
//   { status: 'fulfilled', value: 200 },
//   { status: 'fulfilled', value: 'Finished' }
// ]

- Promise.any(...promises) /* - returns the first fulfilled promise's result or the first rejected promise's error if all promises were rejected. */
Promise.any([promise1, promise2, promise3])
       .then(message => console.log(message));

- Promise.race(...promises) /* - returns the first executed promise result no matter if was fulfilled or rejected. */
Promise.race([promise1, promise2, promise3])
       .then(message => console.log(message))
       .catch(error => console.log(error));
```
---
#### Asynchronous functions
```js
/* Asynchrounous functions - is another API for exectuting asynchronous concurrent code. Async function always returns a promise. If an async function does not explicitly return a promise it will be wrapped by the runtime. */
async function foo() {
	return 1;
}

// is the same as 

function foo() {
	return Promise.resolve(1);
}

/* Since async function returns promise, getting its result is possible either by using the `await` keyword, using the `then...catch` promise methods or using the Promise coposition methods like `all` or `allSettled`. */

// Imagine we have a function that returns a timedout promise. 
function timedPromise(res, timeoutSec) {
	console.log("timedFunc started with timeout", timeoutSec, "sec" ) // #3
	return new Promise((resolve, reject) => {
		setTimeout(() => {
			resolve(res);                                            // #6
			console.log("timedFunc returned result: ", res);          // #7
		}, timeoutSec * 1000)
	})
}

// We can use it in async function like this
async function someFunc(res, timeoutSec) {
	console.log("Async function started");                           // #1
	let p = timedPromise(res, timeoutSec);                           // #2
	console.log("A promise was created and called");                 // #4
	let result = await p;                                            // #5
	console.log("Promise returned result:", result);                 // $8
	return result;
}

// Async function can be called just like sync function.
let r = someFunc("slow", 3); 

/* Let's inspect how this code gets executed.
  - #1 console.log inside someFunc called with string "Async function started";
  - #2 timedPromise function called with `res` parameter = "slow" and `timeoutSec` parameter = 3;
  - #3 console.log inside timedPromise called with string "timedFunc started with timeout 3 sec", promise is created and waits for resolving;
  - #4 console.log inside someFunc called with string "A promise was created and called";
  - #5 someFunc halts further execution until the promise is fulfilled, someFunc yields the control back to the execution context it was called from;
  - #6 the timer in timedPromise expires and promise resolves the string "slow";
  - #7 console.log inside timedPromise called with string 'timedFunc returned result: slow';
  - #8 someFunc resumes its execution and calls console.log with string 'Promise returned result: slow';
*/

// If we would call this function two times with slow and fast timers, like so
let slow = someFunc("slow", 20);
let fast = someFunc("fast", 2);

/* The global execution context would first call the `slow` someFunc, which in turn would execute all its code sequentially until it reaches the await part. This command would instruct the `slow` someFunc to give control back (yield) to the global execution context. The global execution context after that calls the `fast` someFunc version. The `fast` someFunc takes control and executes all the code before the await part. It yields control to the global execution context. 

The global execution context checks the `slow` version - it haven't finished its waiting. Then it checks the `fast` version, which has finished its waiting and is ready to take control and resume execution of the remaining code. 
It prints 'timedFunc returned result: fast' and 'Promise returned result: fast'. At this point the `fast` variable's promise is in `fulfilled` state.

The `fast` someFunc finished its execution and returned the control to the global execution context, which checks the `slow` someFunc. Since it is not ready yet to take control and there is no more code to execute the global execution context is idling not doing anything.

At some point `slow` someFunc had finished its waiting and the global execution context gives control to it to proceed with its execution. The `slow` someFunc prints 'timedFunc returned result: slow' and 'Promise returned result: slow'. Since there is no more code to execute in `slow` someFunc it finishes its execution and control is returned to the global execution context.

The `slow` and `fast` variables are both point to Promises with fulfilled states. The program has finished. */


```
---
#### `fetch` function
```js
/* `fetch` is used to make http requests mainly to use this data in page updates. It has the following signature: fetch(url, [, options]) */

// Options can be:
  - method // http method
  - headers // http headers
  - body // request body
  - mode // same-origin || no-cors || cors
  - credentials // omit || same-origin || include, instructs whether to pass cookies and auth headers along with request
  - cache // default || no-store || reload || no-cache || force-cache || only-if-cached, instructs how to cache the request
  - redirect // follow will allow redirections, error will forbid

// Basic `fetch` usage
const getUsers = async () => {
  let res = await fetch("https://jsonplaceholder.typicode.com/users");

// `fetch` has several methods to interact with its return result
  - response.ok
  - response.status
  - response.headers
  - response.json()
  - response.text()
  - response.arrayBuffer()
  - response.blob()
  - response.formData()

// More examples
const getUsers = async () => {
  let res = await fetch("https://jsonplaceholder.typicode.com/users");
  let responseJson = await res.json();
  console.log(responseJson);
};

/* Dealing with request errors */
// Using `ok` property
const getUsers = async () => {
  let res = await fetch("https://jsonplaceholder.typicode.com/users");
  if (!res.ok) {
    console.log(`Что-то пошло не так, статус: ${res.status}`);
  } else {
    console.log(await res.json());
  }
};

// Using `try...catch` syntax
const getUsers = async () => {
  try {
	let res = await fetch("https://jsonplaceholder.typicode.com/users");
	console.log(await res.json());
  } catch (err) {
	    console.log(err.message)
  }
}

// Using promise chain
const getUsers = async () => {
res = fetch("https://jsonplaceholder.typicode.com/users")
	  .then(res => res.json())
	  .then(res => console.log(res))
	  .catch(err => console.log(err.message));
}

```