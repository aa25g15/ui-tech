# UI Tech

## Frontend Security
* SSL - Secure Sockets Layer - An encryption technique used to safeguard the communication between the server and the client, without it, sensitive information is subject to theft. This is pretty much a standard requirement now.
* Cross Site Scripting

## Web and Browser Fundamentals
### Explain the entire process (what actually happens in backend) starting from sending request from client to server and then back to client.
* https://developer.mozilla.org/en-US/docs/Web/Performance/How_browsers_work

### Event Loop and How Promises and setTimeout are Queued

### Web Performance
* Key Web Performance Metrics (Can calculate them from tools such as lighthouse)
	* Cummulative Layout Shift (CLS) - How much the layout shifts until the render is complete 
		* If images load and move elements around them for example, this can lead to a bad CLS score, you should always set height and width on image elements on their containers
	* Largest Contentful Paint (LCP) - Time to load the largest asset
		* Use next generation formats such as webp
		* Compress asset size
		* Preload LCP asset
	* Time to Interactive (TTI) - The point in time when the long task has finished (tasks that occupy the UI thread for 50ms or more) followed by a 5 second period of network and main thread inactivity. It helps identify scenarios where a page might appear interactive but actually isn't cause the resources needed to respond to user interaction adequately have not been loaded.
		* Preconnect to required origins
		* Reduce main thread work
		* Minify JS
		* Preload critical requests
		* Keep request count low and transfer sizes small
		* Reduce JS execution time
		* Reduce the impact of third party code
	* Total Blocking Time (TBT) - The total time the render process is blocked due to for example, loading of scripts etc.
		* Use defer on scripts or move them at the end of the body tag
		* async and defer are different - async will start loading the script and execute it as soon as it is loaded, defer will start loading the script but will not execute it before the render is complete, also deferred scripts are executed in the order in which they appear in document
	* First Contentful Paint (FCP) - The time taken for for the user to see ANY content on the browser
		* This can be improved using a preloader which shows much before your LCP
		* Inline critical CSS
		* Improve server response time - TTFB (Time to first byte)
		* Avoid script based elements above the fold
		* Avoid lazy loading images above the fold
		* Ensure text remains visible during webfont load
			* Browsers might not show any font until all font files are loaded, this can cause FLash of Invisible Text (FOIT), to prevent this add font-display: swap in your font-face or &display=swap if loading from a CDN such as google fonts. This will show a system font until your fonts load.
		* Eliminate render blocking resources
	* Speed Index - The speed at which above the fold content visually loads
		* Minimize main thread work
		* Reduce JavaScript execution time
		* Ensure text remains visible during webfont load
	* First Input Delay (FID) - This refers to the delay between first input received by the browser and the actual time after which the browser is able to respond to the input. It is a measure of responsiveness of the site. After, FCP and before the main thread is idle (TTI point), if a user clicks or does some other such action, the main thread might be busy (it might be executing some JS files it just downloaded etc.), this might cause the browser to exhibit a delay in responding to the user action.
		* Minimize main thread work
		* Keep requests low and transfer sizes small
		* Reduce JS execution time

<img src="web-performance.png" alt="web performance" />

### Service workers and web workers

## HTML
### Questions
* What does DOCTYPE do?
* What is Charset Meta Tag?

### Semantic HTML

### Accessibility

## CSS
### Position Property
* Static - Default. The elements is positioned as in natural flow of the document.
* Absolute - The element is positioned w.r.t the closest positioned (non-staic) ancestor
* Relative - The element is positioned w.r.t its natural position
* Fixed - The element is positioned w.r.t the window
* Sticky - The element is positioned relative initially and then when user scrolls and a certain offset is passed, the element takes the position fixed

### Selectors
* ID selector
```css
#submit-button {
  color: red;
}
```
* Class selector
```css
.submit-button {
  color: red;
}
```
* Element selector
```css
button {
  color: red;
}
```
* Immediate child selectors
* Selecting all nested children
```css
.parent-class .child-class {
  color: red;
}
```
* Sibling selectors
* Attribute selectors
```css
[data-component="a3-icon"]{
  color: red;
}
```

### Specificity

### Box Model
<img src="box-model.png" alt="box model" width="400"/>

## JavaScript
* Remember JS is a synchronous single-threaded language but the browser event-loop and promises makes it exhibit async functionality
* Synchronous means that the code is executed in the sequence in which it is written, waiting for the previous instruction to finish executing
* Remeber that JS is not static typed but dynamically which means that a variable is not attached to a particular data type, you can do this in JS:
```javascript
const a = "Abhinav";
a = 12; // No error
```
* Remember you can pass excessive arguments to functions in JS and it will not throw and error unlike other languages such as Java
```javascript
function printName(first, last){
	console.log(first + " " + last);
}

printName("Rahul", "Chabra", "Friend"); // no error, output - Rahul Chabra
```

### Some general things to remember
* array.slice(start, end) // Will return a new array without modifying the original, end is non-inclusive
* array.splice(index, howmany, item1, ....., itemX) // modifies the original array, itemN are items to be added

### Data Types
* Primitive - Can store only 1 type at a time
  * number - Integers and floating points both
    * infinity and NaN are special type of numbers
  * big int - When numbers are not enough, rarely needed
  * string
    * Remember strings are immutable in JS!!!!!!! I have made this silly mistake before too, if they were not then any hacker could alter them and change references causing security breaches.
    * There is practically no difference between single and double quotes in JS
    * Backticks (feature of ES6) however allow us to define template strings with variables inside strings such as:
    ```javascript
    const info = `${name} is ${age} years old`;
    ```
  * boolean - true or false
  * null - Means a value is not known
  * undefined - Means a variable has been declared but its value has not been defined yet
    * null and undefined are different in this sense that null is still an assigned value, it is just that the value is unknown
* objects - Can store collection of data and more complex entities
* symbols - Are used to create unique identifiers for objects (probably not that important)

### Difference between var, let and const - let and const are features of ES6
* var
  * Var is either global scoped or function scoped, if declared outside function, it is global scoped, if declared inside function, it is function scoped and it will not be accessible outside that function
  * var can be used to redelcare the same variable!
  * Variables declared using var can be reassigned
  * Variables declared with var can be accessed in their scope before they are declared although the value is always undefined (hoisting)
* let and const
  * Both are block scoped and cannot be accessed outside the block
  ```javascript
  let a = 10;
    function f() {
        if (true) {
            let b = 9
 
            // It prints 9
            console.log(b);
        }
 
        // It gives error as it
        // defined in if block
        console.log(b);
    }
    f()
 
    // It prints 10
    console.log(a)
  ```
  * But as they are only block scoped, you can declare same variable name in different blocks using let and const - DO NOT GET CONFUSED HERE!
  ```javascript
  let a = 10
  if (true) {
    let a = 9
    console.log(a) // It prints 9
  }
  console.log(a) // It prints 10
  ```
  * Let variables can be reassigned but const cannot, however for const objects we can alter object property values but not the properties themselves, we can even add properties! Again do not confused here.
  ```javascript
  const dog = {
  	name: "oreo"
  }
  dog.age = 12;

  console.log(dog); // age will be added as a property
  ```
  * You cannot access let and cost variables before they are declared, you will get ReferenceError

### Promises, callbacks, async, await
* How does javascript figures out that a promise is resolved?
* Promise chaining
* Implement Promise.all
* Implement an array of promises in parallel
* Implement an array of promises one after the other

### Function call, bind, apply
```javascript
const printInfo = function(state, country){
  console.log(this.name + " is from " + this.city + " and " + state + " and " + country);
}

const person1 = {
  name: "Abhinav"
  city: "Amritsar"
}

const person2 = {
  name: "Rahul"
  city: "Chandigarh"
}

printInfo.call(person1, "Punjab", "India"); // Output - Abhinav is from Amritsar and Punjab and India
printInfo.apply(person1, ["Punjab", "India"]); // Output - Abhinav is from Amritsar and Punjab and India

const boundedFunc = printInfo.bind(person1, "Punjab");
boundedFunc("India"); // Output - Abhinav is from Amritsar and Punjab and India
```
* call is used to invoke a function with a given context to which "this" points to and function arguments are passed as comma separated values
* apply is the SAME as call, the only difference is that in apply, function arguments are passed as an array
* bind returns a function which has its "this" pointing towards the provided scope, additional function parameters can be passed as comma separated values which will be passed to the bounded function on invoking it. All arguments need not be there when doing a bind. Rest of them can be passed in the bounded function upon invoking it. See above code.

### Dynamic Dispatch
* Just a fancy term which means calling methods on objects, the origin of this term is from older languages where calling methods on objects was called as sending messages to the object
* In JS we say Dynamic dispatch cause when we call the method, for example, obj.method(), the engine does not know exactly which function in memory to call, it has to first search the right function and might have to look down the prototype chain
* In some other lower level languages such as C++, static dispatch is found as the program knows exactly which function to call in memory

### Currying
* Currying is a concept where a function is broken down into smaller functions each of which takes one argument and returns a function which waits for the second argument and so on till all arguments are provided.
* It is a way to make sure that we have everything beforehand
* So with currying - func(a, b, c) will be split into func(a)(b)(c)
* There can be 2 ways of creating a curried function:
* Closures:
```javascript
const sum = (a,b,c) => console.log(a + b + c);

const cs = (a) => {
	return (b) => {
		return (c) => {
			console.log(a + b + c);
		}
	}
}

cs(1)(2)(3);
```
* bind method:
```javascript
// const add = (a, b, c) => a+b+c
// Implement curriedSum such that all invokations to cs should return answer as 9
// Gave a hint: fn.length gives no of arguments that function takes . eg: in our case add.length = 3
// const cs = curriedSum(add);
// cs(2)(3, 4) 
// cs(2, 3, 4)
// cs(2, 3)(4)
// cs(2)(3)(4)

const curry = (func) => {
	return curried = (...args) => {
  	if(args.length < func.length){
    	return curried.bind(this, ...args);
    }
    func(...args);
  }
}

const sum = (a,b,c) => console.log(a + b + c);

const curriedSum = curry(sum);
curriedSum(1)(5)(3); // 9
curriedSum(1, 5)(3); // 9
curriedSum(1)(5, 3); // 9
curriedSum(1, 5, 3); // 9
```
* Another question related to currying (asked in Amazon) - create a function "sum" which can be called sum(1)(2)(3)(4).....() to return the sum of all the numbers used as inputs
```javascript
const sum = (num1) => {
	return (num2) => {
		if(num2){
			return sum(num1 + num2);
		}
		return num1;
	}
}
```

### Hoisting
* In hoisting, it appears that the interpretor moves the declarations of functions, classes and variables to the top of their scope before executing the code such that their value or declaration is accessible before the line at which they are defined.
* There are 3 types of hoisting:
  1. Value hoisting - The value of the declaration is accessible before it is actually declared. This is true for functions, generator functions, async functions and async generator functions.
  ```javascript
  calc(5,10); // Will log 15

  function calc(a,b) {
    console.log(a + b);
  }

  // Note - If you use const calc = () => {}, this will throw referenceError
  ```
  2. Declaration hoisting - The declaration is accessible before it is actually declared although its value is always undefined. This is true for "var" declarations.
  ```javascript
  console.log(a); // Output - undefined

  var a = 10;
  ```
  3. When a hoisted declarations taints the scope in which it is declared such that accessing it before it is declared throws a ReferenceError - This is true for const, let and class. You cannot access variables declared with const and let and classes before they are declared in their scope.
  ```javascript
  const dog = new Dog(); // ReferenceError

  class Dog {}
  ```

### Closures
* Functions form closures in JS through which the variables available in their lexical scope are available to the functions themselves.
```javascript
const name = "Abhinav";

function printName(){
  console.log(name);
}

printName(); // Output - Abhinav
```
* How do closures work?

### Debounce
* Debounce is a technique through which multiple function calls are combined to a single call. The target function is called only after a certain time has passed after the last call was made. Its common applications include a search bar.
```javascript
const debounce = (func, time = 200) => { // ms
  let timerId;
  return (...args) => { // Rest operator - Collects values and puts them in an array
    clearTimeout(timerId);
    timerId = setTimeout(() => {
      func.apply(this, args);
    }, time);
  }
}
```

### Throttle
* Throttle is a technique by which the target function is called at a certain rate only. The target function is called and then a certain time must pass after this before it is allowed to be called again.
```javascript
const throttle = (func, time = 200) => { // ms
  let timerId;
  return (...args) => {
    if(!timerId){
      // timer is cleared, we can call function
      func.apply(this, args);
      timerId = setTimeout(() => {
        timerId = null;
      }, time)
    }
  }
}
```

### Memoization
* A technique used to store computed results in a cache (temporary data store such as a Map) so that they can be retrieved and returned instead of doing a fresh calculation everytime
```javascript
const memoize = (func) => {
	const map = new Map();  // Cache

	return (...args) => {
		const stringified = JSON.stringify(args);
    
		if(map.has(stringified)){
			// We have already computed this result
			console.log("From cache");
			return map.get(stringified);
		}
		const result = func.apply(this, args);
		map.set(stringified, result);
		return result;
	}
}

const sum = (a, b, c) => a + b + c;

const memoizedSum = memoize(sum);

console.log(memoizedSum(10, 20, 30));
console.log(memoizedSum(9, 312, 45));
console.log(memoizedSum(8, 756, 43));
console.log(memoizedSum(10, 20, 30)); // From cache
console.log(memoizedSum(9, 312, 30));
console.log(memoizedSum(9, 312, 45)); // From Cache
```

### The this Keyword

### Destructuring in JS

### Microtasks/Macrotasks

### Polyfill for bind method
```javascript
Function.prototype.myBind = function(...args){
  const obj = this;
  const params = args.slice(1);
  return function(...args2) {
    obj.apply(args[0], [...params, ...args2]);
  }
}
```

### Difference Between ES5 and ES6
* ES5 was released in 2009 and ES6 in 2015
* ES6 is a major improvement on ES5 with features such as:
	* Classes
	* Arrow functions
	* Let, const
	* Better performance
	* Template literals
	* Object literals
	* Generator functions
	* Modules
	* Promises
	* Rest and spread operators
	* Destructuring
	* Default parameters
	* And a lot more...

### Modules in JS

### Classes - This is an ES6 Feature
```javascript
class Dog {
	#name; // private
  age // public
  
  constructor(name, age){
	  this.#name = name;
    this.age = age;
  }

	printName = () => { // public method
  	console.log(this.#getString());
  }
  
  #getString = () => { // Private method
  	return `${this.#name} is ${this.age} years old.`
  }
}

const dog1 = new Dog("oreo", 7);
const dog2 = new Dog("ramsy", 6);

dog1.printName(); // oreo
dog2.printName(); // ramsy

console.log(dog1.age); // 7

console.log(dog1.#name); // Uncaught SyntaxError: Private field '#name' must be declared in an enclosing class"
```

### Implement own eventmanager

### Implement own redux like lib

### Difference Between Arrow Functions and Normal Functions
* The this keyword in an arrow function points to its lexical scope but in a normal function, it points to the function's object

### Difference between Typescript & Javascript
* As the popularity of JS grew from client side to server side it could not fulfill the enterprise level requirement of OOPs
* As a solution to it, a superset of JS, called TS was introduced
* TS is an OOPS programming language while JS is a prototype based language
* TS supports static typing and strong typing but JS does not
* TS supports interfaces and JS does not
* All this helps point out errors during development (pre-compile stage) which reduces chances of errors during runtime
* Also, it provides intellisense in IDEs such as VS Code which makes development a lot easier and consistent
* TS makes code more maintainable and scalable
* All JS code is compatible with TS and TS can use JS packages 
* TS is trans-piled to JS and for compatibility with older browsers, compilers like Babel can be used

### Prototype and Prototypal Inheritance
* Forget about how inheritance works in other languages as the way it works in JS is a whole new concept
* Inheritance still means the same thing i.e. that one object is able to access the properties and methods of other object
* Whenever we create a function/array/object/{almost anything} in JS, the JS engine attaches a new object to the object of your declaration
* This new object has some hidden properties and methods which are now available to the object you just created without you ever defining them
* This new attached object is called the prototype of your object
```javascript
const arr = ["abhinav", "oreo"];
console.log(arr.__proto__); // Same as Array.prototype which is also an object
console.log(arr.__proto__.__proto__); // Same as Object.prototype which is also an object
console.log(arr.__proto__.__proto__.__proto__); // null - End of the prototype chain
```
* This is called the prototype chain, the Array.prototype has its own methods and parameters such as push() etc. and it inherits the methods and parameters of Object.prototype too which further has null prototype
* This is why it is called that everything in JS is actually an object as anything you create besides primitive types inherit the Object.prototype
* The reason why it is written like __proto__ is so that you do not accidentally alter it, it is just a naming convention
* A function will have Function.prototype which has Object.prototype
* A string will have String.prototype which has Object.prototype
* How can one object inherit the properties and methods of another object by using its prototype?
```javascript
const obj1 = {
  name: "abhinav",
  city: "amritsar",
  getInfo: function(){
    console.log(this.name + " is from " + this.city);
  }
}

const obj2 = {
  name: "rahul",
}

// NEVER DO THIS - ONLY FOR DEMONSTRATION
obj2.__proto__ = obj1;

console.log(obj2.name) // rahul
console.log(obj1.name) // abhinav
console.log(obj2.city) // amritsar - will try to find city on the object itself, if not found will look in prototype and so on...
console.log(obj2.getInfo()) // rahul is from amritsar - Note that now 'this' points to obj2 in the function getInfo
```

## OS
### Threads
* An application has multiple processes, those processes can then either be single threaded or multi-threaded.
* A process is an application being executed
* Each process has code, data, files
  * For a multi-threaded process, each thread does a different task but these threads can run concurrently increasing efficiency and responsiveness of the app
  * The code, data and files are shared between threads which saves system resources and is economical
<img src="threads.png" alt="threads" width="400"/>

## OOPS
* What I learned in university was procedural programming in which variables and functions are decoupled and variables are on one side and functions operate on those variables and are interdependent on each other.
* This creates a spaghetti code problem where there is so much interdependence that altering one functions breaks multiple others.
* OOPs comes to rescue!
* 4 pillars of OOPS:
  * Encapsulation - Related variables (parameters) and methods that operate on them are combined into a single unit called an Object
    * Reduces complexity
    * Reduces number of parameters needed for methods (less the better)
    * Increases reusability
    * Eg - localStorage object in browser
  * Astraction - When we have combined these parameters and methods into a single unit, there is no need for everyone to know exactly what complicated stuff is going on inside the unit as a whole. Several methods and parameters can be hidden from outside access.
    * Reduces complexity since unnecessary information is not available (imagine a DVD player, do you care what happens inside it?)
    * Reduces impact of change as now less things are exposed and so the rest of the code is less dependent on the object as a whole
  * Inheritance - Classes defining objects can inherit the properties and methods of other classes which reduces unnecessary code
    * Reduces unnecessary code - The DRY principle
    * Reduces complexity
    * Increases maintainability
    * Eg - All elements in the DOM inherit the HTMLElement class and thus implement all the methods defined in that class
  * Polymorphism - Poly means many and morph means form. This technique helps reduce unnecessary and complicated if else and switch statements.
    * The same method will behave differently depending on the object being referenced
    * Imagine HTMLElements such as Select, Radio, Checkbox. They all will render differently. Without polymorphism, you would need complicated switch statements to handle each type of element but with polymorphism, you can write element.render() and render will behave differently based on which type of element is being referenced.
