# Modern JavaScript

* During development: simply use the latest Google Chrome!
* During production: use **Babel** to transpile and polyfill your code (converting back to ES5 to ensure browser compatibility for all users)
* to check which feature is compatible with modern browser, you can check ES6 compat-table

### Strict Mode

```javascript
function f1() {
	'use strict';
    return this;
}
console.log(f1())  // undefined

function f1() {
    return this;
}
console.log(f1())  // global object

```



# Tools

## vscode 

### vscode configuration

* install monokai pro theme

* file icon theme: seti

* install auto close tag extension

### vscode Setting

autosave: on focus change

multi cursor modifier: alt

word wrap: on

format on save: checked

### vscode tricks

1. select the content, ctrl + d, will selected all the same content
2. move one line up, alt + up arrow
3. move current line down, alt + down arrow
4. open another vertical window ctrl +\
5. open terminal: ctrl + shift + `

## Chrome

* inspect window Ctrl + Shift + i

# Fundamentals

## Javascript Engine



### link to javascript

```html
<body>
    <script src="script.js"></script>
</body>
```

### Comments

```javascript
// one line comment
/* 
multi-line comments
*/
```



### Variables

* There are two types of variable in JavaScript: Primitive and Object

* JavaScript has dynamic typing: we do not have to manually define the data type of the value stored in a variable. Instead, data types are determined automatically

#### value types

```javascript
// show type of variable 
console.log(typeof true)

// primitive
// Datatype:Number -- Floating point numbers 
let a = 10; 
const b = 10.34;
// Datatype: String -- Sequence of Characters, wrapped by '' or ""
let firstName = 'Jonas'; 
// Datatype: Boolean -- true or false
let fullAge = true;
// Datatype: Undefined -- Value taken by a variable that is not defined (empty value)
let children;
// Datatype: Null -- Also means 'empty value'
// null refer to nothing
var x = null;
// undefined refers to absense of value
var x;
// NaN not a number
Math.sqrt(-10)

// Datatype: Symbol (ES2015) -- Value that is unique and cannot be changed 
// Datatype: Bigint (ES2020) -- Larger integers than the Number type can hold


// object
let me = {
    name: 'Jonas'
};
```

### Destructuring

```javascript
const point = [10, 25, -34];
const [x, , z] = point;
console.log(x, y, z);

// you could ignore values 
const [x, ,z]= point;
```



```javascript
const gemstone = {
  type: 'quartz',
  color: 'rose',
  carat: 21.29
};

const {type, color, carat} = gemstone;
console.log(type, color, carat);
let {color} = gemstone;
```



#### typeof bug

```javascript
typeof null // is object
```

### scope & const, let, var

1. Global scope - When a particular variable is visible (can be used) anywhere in the code. Such a variable is generally called as **Global variable**.
2. Function scope - When a particular variable is visible (can be used) within ***a\*** particular function only. Such a variable is generally called as **Local variable**.
3. Block scope - When a particular variable is visible (can be used) within ***a\*** pair of `{ . . . }` only.

```javascript
/*
 * Global scope. 
 * This variable declared outside of any function is called Global variable. 
 * Hence, you can use this anywhere in the code
 */
var opinion = "This nanodegree is amazing";

// Function scope
function showMessage() {
    // Local variable, visible within the function `showMessage`
    var message = "I am an Udacian!"; 

    // Block scope
    {
          let greet = "How are you doing?";
        /*
         * We have used the keyword `let` to declare a variable `greet` because variables declared with the `var` keyword can not have Block Scope. 
         */
    } // block scope ends

    console.log( message ); // OK
    console.log( greet ); // ERROR. 
    // Variable greet can NOT be used outside the block

    console.log( opinion ); // OK    to use the gobal variable anywhere in the code

} // function scope ends

```

### Variable Declaration

There are three ways to declare a variable:

1. `let` - It a new way to declare a variable in any scope - Global, Local, or Block. The value of this variable can be changed or reassigned anytime within its scope.
2. `const` - It is also a way to declare *constants* in any scope - Global, Local, or Block. Once you are assigned a value to a `const` variable, the value of this variable CANNOT be changed or reassigned throughout the code.
3. `var` - This is the old way of declaring variables in only two scope - Global, or Local. Variables declared with the `var` keyword can not have Block Scope. The value of this variable can be changed or reassigned anytime within its scope.

Bad practices

```javascript
// it is illegal to declare an empty const variable
const a; // error

// assign a value to a variable without declare it is a bad practice
// javascript actually does not create a variable but create a property on the global object
lastName = 'Fan'

```

### hoist

- avaScript hoists function declarations and variable declarations to the top of the current scope.
- Variable *assignments* are not hoisted.
- Declare functions and variables at the top of your scripts, so the syntax and behavior are consistent with each other.

```javascript
function sayGreeting() {
    // hoist var greeting, but not assign value
    console.log(greeting);
   	var greeting = 'hello';
}
syaGreeting(); // undefined
```



### Escaping Characters

```javascript
console.log("The man whispered, \"please speak to me.\"")
"\\" // backslash
"\"" // double quote""
"\'" //single quote
"\n" // new line
"\t" // tab
```



### Symbol

* symbol object is always unique and cannot be changed

```javascript
let sym1 = Symbol();
let sym2 = Symbol('foo');
let sym3 = Symbol('foo')

// get false
sym2 === sym3
```

### Operator

```javascript
// float division
a / 10
// power
2 ** 3
// string combination
str1 + str2
// increment
i++
// assignment
const isFullAge = ageSearch >= 18
// assign two variable together
let x, y;
x = y = 10

// operator precedence
console.log( now - 190 > now - 188) 
```

Logical operator

```javascript
// logical operator
true && true // and
true || false // or
!true // not true

// equal 
3 === 3
// not equal
4 !== 5
// not null
token != null
// or
false || true
// and
true && true
// not 
!true
```



### Spread operator

```javascript
const primes = new Set([2, 3, 5, 7, 11, 13, 17, 19, 23, 29]);
console.log(...primes);
```

### ...rest parameter

```javascript
printPackageContents('cheese', 'eggs', 'milk', 'bread');
function printPackageContents(...items) {
    for(const item of items) {
        console.log(item);
    }
}
```

### Variadic functions

can have indefinite number of arguments

```javascript
function sum(...nums) {
  let total = 0;  
  for(const num of nums) {
    total += num;
  }
  return total;
}
```



### format

```javascript
// remind ` is for the template literals
let name = "xiao"
const jonas = `I'm ${name}, ${2+3}`
// multiline strings from ES6
console.log(`String
multiple
lines`);

```

### Object Literal Shorthand

```javascript
let type = 'quartz';
let color = 'rose';
let carat = 21.29;
let gemstone = {type,color, carat};
// {type: "quartz", color: "rose", carat: 21.29}
```



### expression and statement

* expression produce a value
* statement not produce a value but an action. For example: if else statement
* Ternary operator is expression and can be used in template literal

```javascript
// you can use expression the template literal
console.log(`this is ${200-100}`)
// you cannot use statemnet in the template literal
// but you can use tenary operator
const age = 19
console.log(`I like to drink ${age > 18 ? 'wine' : 'no wine'}`)
```

### condition

```javascript
// one line condition 
let js = 'amazing';
if (js === 'amazing') console.log('javascript is Fun');
// if else
if (a === 0) {
    console.log(a);
} else if (a ===1) {
    console.log('this is else if');
} else {
    console.log('this is else');
}

// ternary operator
const age = 34;
age >= 18 ? console.log('I can drink wine') : console.log('you are not allowed to drink');
// embeded ternary expression
var category = eatsPlants ? (eatsAnimals ? "omnivore" : "herbivore") : (eatsAnimals ? "carnivore" : "undefined");

// assignment with tenary operator
const bill = 275
const tip = bill <= 300 && bill >= 50 ? bill*0.15 : bill * 0.2
```

### switch

```javascript
var month = 2;

switch(month) {
  case 4:
  case 6:
  case 9:
  case 11:
    days = 30;
    break;
  case 2:
    days = 28;
    break;
  default:
  	days = 31;
    break;
}

console.log("There are " + days + " days in this month.");
```

### type conversion, type coercion

```javascript
// convert string to number
let year = '1990';
year = Number(year);
typeof year

// convert number to string implicitly
let num1 = 1
let num2 = '2'
console.log(num1 + num2) // return 3

// convert something cannot be converted to number
const a = number('fa') // return NaN: not a number
typeof a // NaN is Number type

// convert a number to string
String(23);

// type coercion
console.log('I am ' + 2 +'years old') // add operator will turn number to string
console.log(1+2+4+'2') // return 72
console.log('10'-'4'-'3'-2+'5')  // return 15
// Except add operator, other operators will turn string to number
console.log('23' - '10' -3) // got 10, turn to number
console.log('36'/ '3') // return 12
console.log('23' > '18') // return true
```

### Truthy and Falsy Value

```javascript
// falsy value: 0, '', undefined, null, NaN, false
// anything else is Falsy value
console.log(Boolean({})) // empty object is truthy
console.log(Bollean([]))
console.log(Boolean('0')) // 0 string is true
console.log(Boolean(0)) // false
```

### == vs ===

* always use triple equals and pretend the doubt equals does not exist

```javascript
// === is strict comparison
18 === 18 // return true
18 === '18' // return false
// == loose equal operator, does implicit type coercion
18 == '18' // return true
0 == false // return true
" " == false // return true
1 == true // return true

// not stric equal
typeof 'a' !== 'string'
```

### Scope

*block* scope vs. *function* scope

A function's runtime scope describes the variables available for use inside a given function. The code inside a function has access to:

1. The function's arguments.
2. Local variables declared within the function.
3. Variables from its parent function's scope.
4. Global variables.

**runtime scope**. When a function is run, it creates a new runtime scope. This scope represents the *context* of the function, or more specifically, the set of variables available for the function to use.

```javascript
const myName = 'Andrew';
// Global variable

function introduceMyself() {

  const you = 'student';
  
  // refer to variable contained in its parent function
  // refer to variable in global
  function introduce() {
    console.log(`Hello, ${you}, I'm ${myName}!`);
  }

  return introduce();
}
```

### Scope Chain

Whenever your code attempts to access a variable during a function call, the JavaScript interpreter will always start off by looking within its own local variables. If the variable isn't found, the search will continue looking up what is called the **scope chain**

### closure

* closure is the combination of a function and the environment within which that function was declared. The environment consists of any local variables that were in scope at the time that the closure was created.
* In JavaScript, all the function form closures.

```javascript
// JS Nuggets: Closures

function makeFunc() {
  var name = "JS Nuggets";
  function displayName() {
    console.log(name);
  }
  return displayName;
}

var myFunc = makeFunc();
myFunc();


var counter = (function() {
  var privateCounter = 0;
  function changeBy(val) {
    privateCounter += val;
  }
  return {
    increment: function() {
      changeBy(1);
    },
    decrement: function() {
      changeBy(-1);
    },
    value: function() {
      return privateCounter;
    }
  };   
})();

console.log(counter.value());
counter.increment();
counter.increment();
console.log(counter.value()); 
counter.decrement();
console.log(counter.value());
```

```javascript
// constructor
function IceCream() {
  this.scoops = 0;
}

// adds scoop to ice cream
IceCream.prototype.addScoop = function() {
  console.log("inside addScoop", this);
  setTimeout(function() {
    console.log("inside setTimeout: ", this)
	this.scoops++;
    console.log('scoop added!');
  }, 500);
};

let dessert = new IceCream();
dessert.addScoop();
```



## This

```javascript
/* THIS */

console.log(this.document === document);

console.log(this === window);

this.a = 37;
console.log(window.a); 


function f1() {
  'use strict';
  return this;
}
console.log(f1() === window);



function add(c, d) {
  return this.a + this.b + c + d;
}

var o = {a: 1, b: 3};
console.log(add.call(o, 5, 7));
console.log(add.apply(o, [10, 20]));


function f() {
  return this.a;
}

var g = f.bind({a: 'unicycle'});
console.log(g());

var h = g.bind({a: 'cereal'}); // won’t work a second time
console.log(h());

var o = {a: 8, f: f, g: g, h: h};
console.log(o.f(), o.g(), o.h());


var o = {
 traditionalFunc: function () {
   console.log('traditionalFunc this === o?', this === o);
 },
 arrowFunc: () => {
   console.log('arrowFunc this === o?', this === o);
   console.log('arrowFunc this === window?', this === window);
 }
};

o.traditionalFunc();

o.arrowFunc();


var o = {
  prop: 37,
  f: function() {
    return this.prop;
  }
};

console.log(o.f()); // logs 37

var o = {prop: 23};

function independent() {
  return this.prop;
}

o.f = independent;

console.log(o.f());
```

## Loops

### for loop

```javascript
// old school style
for (let i =1; i< 5; i++) {
	console.log(i);    
}

// ES5
function compressAllBoxes(boxes) {
    boxes.forEach(function(box) {
       console.log(box); 
    });
}

// ES6
function compressAllBoxes(boxes) {
    boxes.forEach(box => console.log(box));
}

// ES6 for in loop, iterate with index
const digits = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9];
for (const index in digits) {
  console.log(digits[index]);
}
// ES6 for of loop, iterate with element
const digits = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9];
for (let digit of digits) {
  console.log(digit);
}
```

### while loop

```javascript
let i = 0
while (i<5) {
    console.log(i);
    i++;
}
```

#### do while Loop

```javascript
var count = 4;
do {
    console.log(my_list[count]);
    count--;
} while (count>=0);
```

### break & continue

# Array

```javascript
const my_arr = ['1', '2']
const years = new Array(1992,1993,1994)

// element is zero based
years[0]
// len of array
years.length
// you can put different type of value in the array
my_arr = [1,'2', my_name]
```

### methods

```javascript
const my_arr = [1,2,3]
// push(): add element to the end
len_of_new_arrar = my_arr.push(4) // return the length of new array

// unshift(): add element to the beginning
len_of_new_arrar = my_arr.unshift('0') // return the length of new array

// remove the last element in the array
friends.pop(); // return the removed element

// remote the first element in the array
friend.shift(); // return the removed element

// find the index of one element
friend.indexOf('steven') // return the index

// check if element exists
friend.includes('steven') // return boolean

// reverse
friend.reverse()

// sort
friend.sort()

//join
['a', 'b'].join(" ")

// transfrom array to string
['a', 'b'].toString()

```

### splice()

**Following is the syntax of `splice()` method**: `arrayName.splice(arg1, arg2, item1, ....., itemX);` where,

- `arg1` = Mandatory argument. Specifies the starting index position to add/remove items. You can use a negative value to specify the position from the end of the array e.g., -1 specifies the last element.
- `arg2` = Optional argument. Specifies the count of elements to be removed. If set to 0, no items will be removed.
- `item1, ....., itemX` are the items to be added at index position arg1

```javascript
var donuts = ["glazed", "chocolate frosted", "Boston creme", "glazed cruller"];
donuts.splice(1, 1, "chocolate cruller", "creme de leche"); 
// removes "chocolate frosted" at index 1 and adds "chocolate cruller" and "creme de leche" starting at index 1

Returns: ["chocolate frosted"]
donuts array after calling the splice() method: ["glazed", "chocolate cruller", "creme de leche", "Boston creme", "glazed cruller"]
```

### fill()

```javascript
new Array(100).fill('nemo')
```

### filter()

like with `map()`, the `filter()` method returns a *new* array instead of modifying the original array

```javascript
const array = [1,2,3,4,5];
// filter the list with the element greater than 2
const filterArray = array.filter(num=> {
    return num > 2;
})
// take off return without using return
const filterArray = array.filter(num => num>2)
```

### forEach()

```javascript
words = ["cat", "in", "hat"];
words.forEach(function(word, num, all) {
  console.log("Word " + num + " in " + all.toString() + " is " + word);
});
// not to use all
words = ["cat", "in", "hat"];
words.forEach(function(word, num) {
  console.log("Word " + num + " in " + words.toString() + " is " + word);
});
```

### map()

Remember that the key difference between `forEach()` and `map()` is that `forEach()` doesn't return anything, while `map()` returns a new array with the values that are returned from the function

**the `map()` method returns a new array; it does not modify the original array**

```javascript
// map function will return a new array
[1,2,3].map(v=> v*2)

//improvedDonuts ["JELLY DONUT HOLE", "CHOCOLATE DONUT HOLE", "GLAZED DONUT HOLE"]
var donuts = ["jelly donut", "chocolate donut", "glazed donut"];
var improvedDonuts = donuts.map(function(donut) {
  donut += " hole";
  donut = donut.toUpperCase();
  return donut;
});
```

### reduce

you can do filter and map with reduce

```javascript
const my_list = [1,2,3,4,5];
// get the sum of all the elements in the list
const reduceArray = my_list.reduce((acc, num) => {
    return acc + num;
}, 10) // 10 is the initiate value, the result is 25

// take off bracket without return 
const reduceArray = my_list.reduce((acc, num) => acc + num, 10)
console.log(reduceArray)
```

### concat

```javascript
const fruits = ["apples", "bananas", "pears"];
const vegetables = ["corn", "potatoes", "carrots"];
const produce = fruits.concat(vegetables);
console.log(produce);

// or
let produce = [...fruits, ...vegetables]
```



# Number

```javascript
// transform string to number
Number(12)
// round 2 decimal string
let n = 10.0001
n.toFixed(2) 

// local price format
let price = 10000
a.tolocalString("en-US")
```



# String

```javascript
// Pick a string. Your string can have any number of characters.
var my_string = "a";

// Calculate the ASCII value of the first character, i.e. the character at the position 0. 
var ASCII_value = my_string.charCodeAt(0);

// uppercase
my_string.toUpperCase();

```



# Function

* the difference between function declaration and function expression is hoisting

### first-class function

In JavaScript, functions are *first-class* functions. This means that you can do with a *function* just about anything that you can do with other elements, such as numbers, strings, objects, arrays, etc. JavaScript functions can:

1. Be stored in variables
2. Be returned from a function.
3. Be passed as arguments into another function.

### function is object

```javascript
function average(n1, n2, n3) {
    return (n1 + n2 + n3)/3;
}
average.length // 3
average.name // average
```



### function declaration

* you can call the function before you define it. This process is called hoisting.

```javascript
// declare the funtion
function log() {
    console.log('this is log function')
}
// invoke function 
log();

// with default argument
function greet(name='fanxiao', age=10) {
    console.log(`hello ${name}, with ${age} ages`)
}
```

### function expression

* you can see function like a value stored in an variable
* you can only call the function after you define it

```javascript
// declare an anoynmous function
const calcAge2 = function (birthYear) {
    return 2037-birthYear;
}
// call the function
calcAge2(1994)
```

### arrow function

* arrow function get rid of the `function`, `return`, `{}` 
* arrow function do not have `this` keyword

```javascript
// arrow function in one line
const calcAge3 = birthYear => 2037 - birthYear;
calcAge3(1992)

// arrow function in multiple lines
const yearsUtilRetirement = birthYear => {
  const age = 2038 - birthYear; 
  return age;
};
yearsUtilRetirement(1994)

// arrow function with multiple parameters 
const yearsUtilRetirement = (birthYear, thisYear) => {
  const age = thisYear - birthYear; 
  return age;
};
yearsUtilRetirement(1994)
```

### arrow function and this keyword

this: The object that is executing the current function

* if the function is a method of an object, this in this method refer to the object
* if the function is a regular function, which means it is not a part of an object, this refer to the global object, it is the window object in browser, and global in node 
* With regular functions, the value of `this` is set based on *how the function is called*. With arrow functions, the value of `this` is based on *the function's surrounding context*. In other words, the value of `this` *inside* an arrow function is the same as the value of `this` *outside* the function.

```javascript
// function is a method of an object
const video = {
    title: 'a', 
    play() {
        console.log(this);
    }
};
video.play();
```

```javascript
//regular function
function playVideo() {
    console.log(this);
}
playVideo();
// Window {0: global, 1: global, window: Window, self: Window, document: document, name: "", location: Location, …}
```

```javascript
// constructor function
function Video(title){
    this.title = title; // this is an empty object 
    console.log(this);
}
const video = new Video('a')// create an empty objec with new operator
```

```javascript
// function saw as regular function
const video = {
    title: 'a', 
    tags: ['a', 'b', 'c'],
    showTags() { // callback function insdie forEach saw as regular function
        this.tags.forEach(function(tag) {
            console.log(this);
        });
    }
}
video.showTags(); // this is windows object
```

```javascript
// forEach function all you pass other argument other than call back function
const video = {
    title: 'a', 
    tags: ['a', 'b', 'c'],
    showTags() { // pass this into the callback function
        this.tags.forEach(function(tag) {
            console.log(this);
        }, this);
    }
}
video.showTags(); // video object
```

arrow functions inherit `this` value from the surrounding context

```javascript
// use arrow function could solve the problem above
const video = {
    title: 'a', 
    tags: ['a', 'b', 'c'],
    showTags() {
        this.tags.forEach(tag=>console.log(tag));
    }
}
video.showTags();
```

```javascript
////////////////////////////////////////

let icecream = {
	name:'icecream', 
	price:11,
	print_name: function(){
		console.log(this)
	}};
	
icecream.print_name();
/////////////////////////////////////////
let icecream = {
	name:'icecream', 
	price:11,
	print_name: ()=>console.log(this),
}
icecream.print_name();

///////////////////////////////////////////////
let icecream = {
	name:'icecream', 
	price:11,
	print_name: function() {
		setTimeout(() => console.log(this), 500);
	},
}
icecream.print_name();
```



### Factory function

* each time we call the function, we create an object
* like regular function using camel notation

```javascript
function createCircle(radius) {
    return {
        radius, 
        draw() {
            console.log('draw');
        }
    };
}
const circle = createCircle(1);
```



### Constructor function

* the job of constructor function is to construct or create an object
* use Pascal Notation: OneTwoThreeFour
* the difference to factory function is that Pascal Notation, implicitly return object, new operator. 

```javascript
function Circle(radius) {
    this.radius = radius; // add property to an empty object
    this.draw = function() {
        console.log('draw');
    }
}
// with new operator, create an empty javascript object
const circle = new Circle(1); 
```



### Callback function

A function that takes other functions as arguments (and/or *returns* a function, as we learned in the previous section) is known as a **higher-order function**

Being able to store a function in a variable makes it really simple to pass the function into another function. A function that is passed into another function is called a **callback**.

```javascript
// function expression catSays
var catSays = function(max) {
  var catMessage = "";
  for (var i = 0; i < max; i++) {
    catMessage += "meow ";
  }
  return catMessage;
};

// function declaration helloCat accepting a callback
function helloCat(callbackFunc) {
  return "Hello " + callbackFunc(3);
}

// pass in catSays as a callback function
helloCat(catSays);
```



### Inline function

```js
// Function declaration that takes in two arguments: a function for displaying
// a message, along with a name of a movie
function movies(messageFunction, name) {
  messageFunction(name);
}

// Call the movies function, pass in the function and name of movie
movies(function displayFavorite(movieName) {
  console.log("My favorite movie is " + movieName);
}, "Finding Nemo");
```

### Function expressions and hoisting

Deciding when to use a function expression and when to use a function declaration can depend on a few things, and you will see some ways to use them in the next section. But, one thing you'll want to be careful of is hoisting.

All *function declarations are hoisted* and loaded before the script is actually run. *Function expressions are not hoisted*, since they involve variable assignment, and only variable declarations are hoisted. The function expression will not be loaded until the interpreter reaches it in the script.

### IIFE

immediately invoke function execution

* reduce the global name space
* the order of file is also important

```javascript
// inside js file
let myApp = [];
// the following function can be executed immediately but only once
(function(){
    myApp.add = function(a, b) {
        return a * b
    }
})();

```

### Curring

* you can have function return by another function

```javascript
// this is normal function
const welcome = (hello, name) => hello + " " + name;

// curring function 
const welcome_i8n = (hello) => (name) => hello + " " + name;
// define a new function by curried function
const english = welcome_i8n('Hi');
const spanish = welcome_i8n('Hola');
const german = welcome_i8n('tag');

console.log(english('Xiao'));
console.log(spanish('Xiao'));
console.log(german('Xiao'));

```

basically `const welcome_i8n = (hello) => (name) => hello + " " + name ` equals

```javascript
const welcome_i8n = function(hello) {
  return function(name) {
      hello + " " + name;
  }
};
```

### Compose

* combine two functions

```javascript
// compose two functions
const compose = (f, g) => (arg) =>f(g(arg));
const sum=(num) => num + 1;
compose(sum, sum)(5) ; // return 7

// compose two functions which get different input
const add_two = (f, g) => (arg1, arg2) =>f(arg1) + g(arg2);
const square=(num) => num ** 2;
add_two(square, square)(3,4); // return 25
```

### Destructuring with Object Defaults Parameter

```javascript
function buildHouse({floors=1,color='red'}={}) {
  return `Your house has ${floors.toString()} floor(s) with ${color} brick walls.`
}

console.log(buildHouse()); // Your house has 1 floor(s) with red brick walls.
console.log(buildHouse({})); // Your house has 1 floor(s) with red brick walls.
console.log(buildHouse({floors: 3, color: 'yellow'})); // Your house has 3 floor(s) with yellow brick walls.

```

### Scope

A function's runtime scope describes the variables available for use inside a given function. The code inside a function has access to:

1. The function's arguments.
2. Local variables declared within the function.
3. Variables from its parent function's scope.
4. Global variables.

```javascript
const init = function () {
  let playing = true; // playing can only be referred inside the init function

// playing is not accessible outside the function
console.log(playing)
```

if the variable is declared outside of function, we can access it inside the function

```javascript
let playing;
const init = function (){
    playing = true;
}
```



### Closure

child scope can access the parent scope, but parent scope cannot access the child scope

### script tags

* the order to import could be a problem
* lack of Dependency Resolution
* poluting the global name space

```html
<script type="text/javascript" src="./1.js"></script>
<script type="text/javascript" src="./2.js"></script>
<script type="text/javascript" src="./3.js"></script>
<script type="text/javascript" src="./4.js"></script>
```

### Inline script

inside a html body tag, the problem is 

* the code does not have code reusability
* the pollution of global namespace

```html
<script type="text/javascript">
    function () {
        alert('a')
    }
</script>
```

# Object

There are two method to create an object: literal notation, or constructor function. 

```javascript
// literal notation
const myObject {};

// constructor function 
const myObject = new Object();
```

we can add and delete property to object

```javascript
// add and delete
myObject.name = 'xiao'
// delete property
delete myObject.name
```



we can create a object without class and define method inside of an object.

Using `sister["parents"]` is called **bracket notation** (because of the brackets!) and using `sister.parents` is called **dot notation** (because of the dot!).

```javascript
const user = {
    name: "John", 
    age: 18, 
    hobby: "Soccer", 
    isMarried: false,
    //define the method
    thisMethod: function() {
        alert(this.age); 
    },
};
// don't use number at the beginning of property
user.age  // dot notation
user['age'] // bracket notation
user['a' + 'ge'] // you can use expresion 

// to expand the user object
user.married = true

// call the method
user.thisMethod()
user['thisMethod']()
```

### reference type

```javascript
// false
[] === []

obj00 = { value: 10};
// obj01 is a reference of obj00
obj01 = obj00;
// change made in obj00 will impact on obj01
// true
obj01 === obj00;
// obj02 is a new object
obj02 = { value: 10 };
// false 
obj02 === obj00
```

### this

**How the function is invoked determines the value of `this` inside the function**

context tell you what object we are inside of

```javascript
// in console , this is window
console.log(this === window)
// alert is a method of window object
this.alert('hello world')

// if this is in an object, this is the object
const obj04 = {
    a: function(){
        console.log(this.b);
    }, 
    b: 11,
}
```

```javascript
const car = {
  numberOfDoors: 4,
  drive: function () {
     console.log(`Get in one of the ${this.numberOfDoors} doors, and let's go!`);
  }
};

const letsRoll = car.drive;

letsRoll(); // this is window object
```

## Object.keys() and Object.values()

```javascript
const dictionary = {
  car: 'automobile',
  apple: 'healthy snack',
  cat: 'cute furry animal',
  dog: 'best friend'
};

Object.keys(dictionary); // ['car', 'apple', 'cat', 'dog']
// Object.values() is quite new
Object.values(dictionary); // ['automobile', 'healthy snack', 'cute furry animal', 'best friend']

// get keys by for in
const result = []
for (const word in dictionary) {
    result.push(word)
}
```



# Class

when you reuse the code

```javascript
class Player {
    // construct
    constructor(name, type) {
        this.name = name;
        this.type = type;
    }
    introduce() {
        console.log(`Hi I am ${this.name}, I'am a ${this.type}`)
    }
}

let obj05 = new Player('fanxiao', 'human');

// inheritance
class Wizard extends Player {
    constructor(name, type) {
        // access the construtor of parent class
        super(name, type);
    }
    play() {
        console.log(`i'am a ${this.type}`)
    }
}

let obj06 = new Wizard('xiao', 'wizard')
```

```javascript
class Dessert {
  constructor(calories = 250) {
    this.calories = calories;
  }
}

class IceCream extends Dessert {
  constructor(flavor, calories, toppings = []) {
    super(calories);
    this.flavor = flavor;
    this.toppings = toppings;
  }
  addTopping(topping) {
    this.toppings.push(topping);
  }
}
```

## Subclass

```javascript
class Vehicle {
	constructor(color = 'blue', wheels = 4, horn = 'beep beep') {
		this.color = color;
		this.wheels = wheels;
		this.horn = horn;
	}

	honkHorn() {
		console.log(this.horn);
	}
}

class Bicycle extends Vehicle {
  constructor(color, wheels=2, horn='honk honk') {
    super(color, wheels, horn);
  }
}


const myVehicle = new Vehicle();
myVehicle.honkHorn(); // beep beep
const myBike = new Bicycle();
myBike.honkHorn(); // honk honk

```



# Browser function

```javascript
// log in the browser console
console.log('hello world')
// log two variables 
console.log(a, b)
// prompt up an alert
alert('this is an alert')
// prompt up a confirm window to user
confirm('do you want to delete this ?')
// prompt an input window and allow user to input 
prompt('this is a prompt')
// timer 
setTimeout(callback, time_miliseconds) // asynchronous function
```



# Windows Object

This `window` object has access to a ton of information about the page itself, including:

- The page's URL (`window.location;`)
- The vertical scroll position of the page (`window.scrollY'`)
- Scrolling to a new location (`window.scroll(0, window.scrollY + 200);` to scroll 200 pixels down from the current location)
- Opening a new web page (`window.open("https://www.udacity.com/");`)

# Math

```javascript
Math.floor()
Math.random() // return number [0,1]
Math.trunc(1.55) // return 1 
```

# ES7

### include

```javascript
// o is in the hello
'hello'.include('o')
```

### power operator

```javascript
const cube = (y) => y**3
```

# ES8

### .padStart() and .padEnd()

```javascript
// add pad at the beginning of string
'Turtle'.padStart(10)
// add pad at the end of string
'Turtle'.padEnd(10)
```

### line

easy to git change

```javascript
const fun = (
             a, 
             b, 
             c, 
            ) => {
    console.log(a);
}
```

### Object.values, Object.entries, Object.keys

```javascript
let obj = {
    username0: 'Santa',
    usernane1: 'Rudolf', 
    username2: 'Mr.Grinch'
}
// Object.keys()
Object.keys(obj).forEach((key, index) => {
    console.log(key, obj[key], index);
})

// Object.values()
Object.values(obj).forEach(value => {
    console.log(value);
})

// Object.entries() get [key, value]
Object.entries(obj).forEach(entry => {
    console.log(entry)
})

// replace 
Object.entries(obj).map(entry => {
    return entry[0].replace('username', '') + entry[1]
})
```

# ES10

### flat

we can choose how many layers to distribute of flat

```javascript
const my_list = [1, 2, [2, 4], [[5, 7], 9]]
// default argument is 1
my_list.flat(2)

// get rid of empty element
const entries = ['a', 'b',,,,,'c']
entries.flat()
```

### flatmap

```javascript
const my_list = [1, 2, [2, 4], [[5, 7], 9]]
// add 10 to each element in all layers
// if the element has more than one layer, 10 will add in the last element as a string
my_list.flatMap(num=>num + 10)
```

### trimStart(), trimEnd()

```javascript
let userEmail1 = '    tigerfanxiao@gmail.com'
let userEmail2 = 'tigerfanxiao@gmail.com   '
console.log(userEmail.trimStart())
console.log(userEmail.trimEnd())
```

### fromEntries

easily transfer a list of key-value to an object

```javascript
let my_list = [['name', 'fanxiao'], ['age', 19]]
Object.fromEntries(my_list)
```

### try-catch

```javascript
// from ES10 no need to create error object
try {
    true + 'hi'
} catch {
    
    console.log('Wrong input')
}

// before ES10 it is needed to create error object
try {
    bob + 'a';
} catch (error) {
    console.log('yuo messed up', error)
}
```

### for of and for in 

```javascript
// work with iterable array, string
basket = ['apple', 'orange', 'grapes']
for (item of basket) {
    console.log(item)
}

// work with enumerable with keys
let obj00 = {
    'name': 'fanxiao',
    'age': 19,
}
for (key in obj00) {
    console.log(key)
}
```



# DOM

### html class and id

```html
<!-- class is not unique -->
<p class="first">
    content
</p>
<!-- id is not unique -->
<img id="course-image" src="http"/>
```



## DOM Selector

```javascript
document.body.addEventListener('click', function () {
     var myParent = document.getElementsByTagName("h1")[0]; 
     var myImage = document.createElement("img");
     myImage.src = 'https://thecatapi.com/api/images/get?format=src&type=gif';
     myParent.appendChild(myImage); 
     myImage.style.marginLeft = "160px";
});
```

### querySelector

```javascript
// querySelector is only return the first one matched tag
tag_obj = document.querySelector('tag name') // return a tag object
tag_obj = document.querySelector('.classname')
tag_obj = document.querySelector('#id')
tag_obj = document.getElementById('id') // a little faster than querySelector


// set attribute
tag_obj.style.color = 'red' // set style attribute
body_obj.style.backgroundColor = 'green';
tag_obj.style.width = '300px';
tag_obj.style.display = 'block' // none: hide, blcok: show

tag_obj.textContent = 'this is xiao' // set content

// input tag
tag_obj.value // get the input value


```

### querySelectorAll

```javascript
// return NodeList
document.querySelectorAll('btn')
```

### mange class List

Javascript will not remove what does not exist and add what already exists

```javascript
tag_obj.classList.add('fadeIn') // add class fadeinn
tag_obj.classList.remove('classname1', 'classname2') // not use .classname here
tag_obj.classList.contains('classname'); 
tag_obj.classList.toogle('classname') // if exists, remove; if not exists, add
```



## DOM event

### Mouse event

```javascript
// click event
tag_obj.addEventListener('click', callback)

img.src = 'dog.jpg' // load img is asynchronous
img.addEventListener('load', callback)
```

### Keyboard event

keyboard event is global event, so it belongs to document object

* There is `keydown`, `keypress`, `keyup`

```javascript
document.addEventListener('keydown', callback)
```



# Thread

Thread of Execution: Part of execution context that actually execute the code in computer's CPU

# Debuging

```javascript
let flattened = [[0, 1], [2, 3], [4, 5]].reduce(
    (acc, array) => {
        debugger;
        return acc.concat(array);
    }, []
);
```

# Javascript Runtime

Javascript is a single thread programming language, so it has only one stack and one heap
except from Javascript engine, we have Javascript Runtime, 

[demo](http://latentflip.com/loupe/?code=ZnVuY3Rpb24gcHJpbnRIZWxsbygpIHsNCiAgICBjb25zb2xlLmxvZygnSGVsbG8gZnJvbSBiYXonKTsNCn0NCg0KZnVuY3Rpb24gYmF6KCkgew0KICAgIHNldFRpbWVvdXQocHJpbnRIZWxsbywgMzAwMCk7DQp9DQoNCmZ1bmN0aW9uIGJhcigpIHsNCiAgICBiYXooKTsNCn0NCg0KZnVuY3Rpb24gZm9vKCkgew0KICAgIGJhcigpOw0KfQ0KDQpmb28oKTs%3D!!!PGJ1dHRvbj5DbGljayBtZSE8L2J1dHRvbj4%3D) for runtime

setTimeout(callback_function, time)
the callback_function is be putted in callback queue. the function inside callback function will loaded in call stack. 

Javascript engine will run all the lines in the Javascript file. when it found webapi, it will put it in webapi zoon and continue to put function into call stack, the event loop will keep check if the call stack is empty it means all the function is pop up from the call stack. the call stack is empty. 

then it will run webapi



# ajax

AJAX = Asynchronous Javascript And XML: Allow us to communiate with remote web servers in an asynchronous way. With Ajax calls, we can request data from web servers dynamically.



```javascript
// example of synchronous call
const p = document.querySelector('.p');  // get p tag as an object
alert('Text set');
p.style.color = 'red'; // change the color of p tag
```

* Synchronous code is executed line by line
* Each line of code waits for previous line to finish, so if there is an alert, the execution will be blocked.

```javascript
const p = document.querySelector('.p');
setTime(function () {
    p.textContent = 'My name is Jonas'; // callback function
}, 5000);
p.style.color = 'red';
```

* Asynchronous code is executed after a task that runs in the background finishes. 

  `setTimeout (callback, timer)` will trigger a timer at background, the callback function will be executed when the timer is finished. 

* Asynchronous code is non-blocking

* Execution doesn't wait for an asynchronous task to finish its work

* Callback functions alone do Not make code aynchronous!

### Asynchronous behavior

```javascript
setTimeout(callback, milisecond)
img.src = 'dog.jpg'
```

### Example of asynchronous

```javascript
const img = document.querySelector('.dog');
img.src = 'dog.jpg'  // asynchronous
// add event listener is synchronous
// this callback function will be called after the img is loaded
img.addEventListener('load', function() {
    img.classList.add('fadeIn'); 
});
p.style.width = '300px';
```

