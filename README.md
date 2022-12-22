# UI Tech

## Frontend Security

## Web and Browser Fundamentals
### What happens when you type a website in your browser?
* 

## HTML
### What is Charset Meta Tag?

### Semantic HTML

### Accesibility

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

## JavaScript
* Remember JS is a synchronous single-threaded language but the browser event-loop and promises makes it exhibit async functionality
* Synchronous means that the code is executed in the sequence in which it is written, waiting for the previous instruction to finish executing

### Data Types
* Strings
  * Remember strings are immutable in JS!!!!!!! I have made this silly mistake before too, if they were not then any hacker could alter them and change references causing security breaches.

### Difference between var, let and const
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

### Function call, bind, apply

### Currying

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

### Debounce
* Debounce is a technique through which multiple function calls are combined to a single call. The target function is called only after a certain time has passed after the last call was made. Its common applications include a search bar.

### Throttle
* Throttle is a technique by which the target function is called at a certain rate only. The target function is called and then a certain time must pass after this before it is allowed to be called again.

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
