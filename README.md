# UI Tech

## Resources
* https://alphaayush.notion.site/alphaayush/2e13395deff94a428d45b3aa88dc7ee7?v=06b5c5617b8442bc878bd210257786ad
* https://github.com/lydiahallie/javascript-questions/blob/master/README.md

## Frontend Security
* SSL - Secure Sockets Layer - An encryption technique used to safeguard the communication between the server and the client, without it, sensitive information is subject to theft. This is pretty much a standard requirement now.
* Cross Site Scripting (XSS)
* CSRF
* Clickjacking
* CSP - Content Security Policies
* CORS - Cross Origin Resource Sharing
* Auth

## Web and Browser Fundamentals
### Explain the entire process (what actually happens in backend) starting from sending request from client to server and then back to client.
* https://developer.mozilla.org/en-US/docs/Web/Performance/How_browsers_work

### Event Capture, Target, Bubbling, Delegation
* When you click for example, the browser does not straight away know which element received the event
* It knows that a click event occured, so it starts from the root, all the way down to the element that received the event - This is called event capturing phase
* The target element is isolated and the browser checks if an event handler is registered with it - This is called the target phase
* If an event handler is registered, the event starts to bubble from the target all the way up to the root and all parental event listeners are also triggered unless the event propagation is manually stopped
* This bubbling of the events up the DOM can be used to implement a technique called delegation, in delegation a common parent is delegated to deal with events triggered on the children, for example:
* Imagine you have several buttons inside a div and you want to log the button text when each button is clicked
* One way is to attach a click event listener on each button, this will lead to several event handlers doing the same thing!!
* Instead we can add a single event listener to the parent like this:
```html
<div class="buttons-container">
	<button>Button 1</button>
	<button>Button 2</button>
	<button>Button 3</button>
	<button>Button 4</button>
</div>
```
```javascript
const container = document.querySelector(".buttons-container");
container.addEventListener("click", (event) => {
	if(event.target.tagName === "BUTTON"){
		console.log(event.target.innerText);
	}
});
```
* Now a single event listener handles all the buttons!!! Thus, event delegation is an efficient way of handling similar events

### Event Loop and How Promises and setTimeout are Queued
* JS is a synchronous, single-threaded language which means it can only do 1 thing at a time, so how can it perform concurrent async operations?
* The answer is the Event Loop and Web APIs (in browsers) or C++ APIs in Node which handle concurrent operations in separate threads
* The JS runtime environment has the call stack and any function/method called is pushed on to the call stack and popped after it has been executed
* The JS V8 engine is unable to perform any other operation while the call stack is executing a function
* This means that if we add a lot of slow code to the call stack (slow code could even be something like a while loop from 0 to 100000), the UI will essentially freeze and our clicks and actions will seem to not work, then when the call stack becomes empty again, the browser will execute all our clicks or actions which earlier did not seem to respond!
* Imagine we are getting data from the network which is a slow operation:
```javascript
getSyncData1(); // 5 seconds
getSyncData2(); // 3 seconds
getSyncData3(); // 10 seconds

// If this was done synchronously, the call stack would be busy for 18 seconds!!!! All inputs/clicks/actions will seem like frozen and delayed
// Even the render will not occur, cause even the render function needs an empty call stack to work, remember usual FPS is 60, so browser needs to
// paint every 16ms approximately for a smooth UI and smooth animation experience, otherwise the user will experience jank
```
* Remember that APIs like setTimeout, ajax, DOM etc. are not part of the V8 Engine but they are provided by the browser and run on other threads, this enables concurrency
* The browser maintains 2 queues - Microtask queue and Macrotask queue
	* Microtask queue - Promise and mutation observer callbacks, even the fetch API since it returns a promise
	* Macrotask queue - Browser API callbacks such as DOM event handlers, setTimeout, ajax etc.
* The event loop is responsible to check the call stack, when it is empty, it will dequeue and push all microtasks to the call stack first since they have higher priority and then push macrotasks onto the call stack
* In this way, every loop of the event loop a callback (micro or macro if micro are finished) is pushed to the stack and executed
* In this way, JS is non-blocking and this leads to smooth UI experiences
* If we have slow synchronous code too, for example:
```javascript
function sleep(milliseconds) {
  const date = Date.now();
  let currentDate = null;
  do {
    currentDate = Date.now();
  } while (currentDate - date < milliseconds);
}

for(let i = 0; i < 10; i++){
	console.log("Processing")
	sleep(1000);
}
```
* It is better to create an async version of it and utilise the non-blocking power of the Event Loop, this will not block the render() and will not lead to jank or low responsiveness, remember that render() is not blocked cause it is given priority, so between event loop iterations, it can be called when the stack is momentarily empty
```javascript
function sleep(milliseconds) {
  const date = Date.now();
  let currentDate = null;
  do {
    currentDate = Date.now();
  } while (currentDate - date < milliseconds);
}

for(let i = 0; i < 10; i++){
	// This will push the slow sync callback to the macrotask queue
	setTimeout(() => {
		console.log("Processing");
		sleep(1000);
	}, 0);
}
```
* Full fledged effect of the Event Loop and the difference between micro and macro tasks can be understood here:
```javascript
setTimeout(() => {
	console.log(1);
}, 0)

new Promise(resolve => resolve(2)).then(res => console.log(res));

new Promise(resolve => resolve(3)).then(res => console.log(res));

setTimeout(() => {
	console.log(4);
}, 0)

new Promise(resolve => resolve(5)).then(res => setTimeout(() => console.log(res), 0));

new Promise(resolve => resolve(6)).then(res => console.log(res));

/*
	Output - 2 3 6 1 4 5
  
  Explanation:
	Macrotask Queue - log(1), log(4)
  Microtask Queue - log(2), log(3), 
  callback with setTimeout for log(5), log(6)
  
  Now the event loop in 1 cycle tries to finish all microtasks and 
  then takes 1 macrotask and adds to callstack when it is empty
  
  After finishing all microtasks:
	Output - 2 3 6
  Note that the third callback is executed and its result was
  to add log(5) with a setTimeout, this setTimeout is now 
  pushed to macrotask queue
  
  Final macrotask queue - log(1), log(2), log(5)
  Now microtask queue is empty, event loop will take each log
  function in macrotask queue and execute it in each loop
  
  Final console output - 2 3 6 1 4 5
*/
```

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

## REST API Methods
* GET - Idempotent, safe, no body, used only to retrieve information from the server
* POST - Non-idempotent, unsafe, has body, used to create new entry to the DB, or change in state or cause side effects on the server
* PUT - Idempotent, unsafe, has body, used to replace all target resource values with the data in the request body
* DELETE - Idempotent, unsafe, used to delete a particular resource from DB
* OPTIONS - Idempotent, safe, is used to check permissible options for communication with the server, client can mention a URL or use * to refer to the entire server
* PATCH - Non-idempotent, unsafe, is used to carry out partial modifications to a resource
* Idempotent means that no matter how many times a request is carried out, there are no additional side effects than which occured the first time
* PATCH is not always idempotent, however, it can be but PUT is always idempotent, PATCH has instructions to modify a resource whereas PUT is the whole modification itself
* PUT is idempotent but POST is not. Calling POST multiple times can have multiple side-effects like duplicate entries getting created in the DB.

## HTML
### Questions
* What does DOCTYPE do?
	* DOCTYPE -> Document Type Definition (DTD)
	* Defines the structure rules of the document
	* Browsers trigger no-quirks mode for particular DTD
	* For HTML5:
	```html
	<!DOCTYPE html>
	```
* What is Charset Meta Tag?
* Serve page in multiple languages
	* lang attribute in html tag
	* hreflang in link to tell search engines that page exists in multiple languages
	* i18n placeholders
	* Allow to change language easily
	* Dates and time could change
	* Colors could change
	* Text direction and even width could change
	* May need to swap images with embedded text
	* Accept-Language header is sent to server
	* Show locale in url
	* Do not concatenate translated strings as this may not be gramatically correct
* data- attributes
	* Can be changes easily in inspect
	* Available in dataset
	* Better to keep data in JS binded using framework or library
	* Could be useful for testing libs
* Building blocks of HTML5
	* Audio/Video
	* 2D/3D graphics
	* Better performance
	* Better ways of connecting with the server
	* More semantic elements
	* More web APIs
	* Offline and storage
	* More verbose styling and themes
	* Device access
* Cookie, sessionStorage and localStorage
	* Session, local - 5mbs, not sent to server
	* Cookie - 4kb, sent to server
	* You know the rest
* Describe the difference between <script>, <script async> and <script defer>
	* script - block render
	* async - loads script in parallel, executes immediately
	* defer - loads script in parallel, executes after HTML parsing in order of appearance
* Why is it generally a good idea to position CSS <link>s between <head></head> and JS <script>s just before </body>? Do you know any exceptions?
	* HTML -> DOM and CSS -> CSSOM
	* CSSOM generation is delayed if stylesheets are not in head
* Progressive rendering - Techniques to improve perceived load time
	* Lazy loading images
	* Prioritising content above fold
	* Using onload event or DOMContentLoaded event to load below fold content after above fold is loaded
	* Async HTML fragments - Flushing parts of the HTML to the browser as the page is constructed on the back end
* srcset attribute in images
	* To serve different images for different device widths and pixel density
	```html
	<img srcset="small.jpg 500w, medium.jpg 1000w, large.jpg 2000w" src="..." alt="">
	```
	* Pixel density of retina displays could be 2
	* Suppose device width is 320px
		* 500 / 320 = 1.5625
		* 1000 / 320 = 3.125
		* 2000 / 320 = 6.25
	* For normal display small.jps is chosen, for retina display (2x), medium.jpg is chosen

### Semantic HTML
* Semantic HTML is using elements which clearly indicate the purpose of the content within, for example, <article>, <section>, <heading> etc.
* It makes it easier for assistive technologies to understand your page

### Accessibility
* a11y rules for accessibility
* In my experience, using proper aria-label attributes for buttons, using proper alt text for images, using semantic HTML, improves accessibility

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
* Element or tag selector
```css
button {
  color: red;
}
```
* Child selectors
```css
.test > p {
  color: red;
}
	
.test:nth-child(2) { // also first-child and last-child, nth child is considered slow
  color: red;
}
```
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
* a b c d (left overpowers right)
	* a - inline styling, 0 or 1
	* b - number of id selectors
	* c - number of class, attribute and pseudo-class selectors
	* d - number of element and pseudo-element selectors
* If specificity is the same, the rule which appears more at the bottom is considered more specific

### Box Model
* Box model is responsible for calulating width and height and collapsing of any borders or margins and how elements are positioned next to each other
* A box with no width will take all horizontal space
* width and height depend on the content
* Default width and height only include content-box but with box-sizing: border-box this can be overridden to include padding and borders
<img src="box-model.png" alt="box model" width="400"/>
	
### Floats
* https://css-tricks.com/all-about-floats/
	
### Questions
* What's the difference between "resetting" and "normalizing" CSS? Which would you choose, and why?
	* Resetting - remove all default styles
	* normalizing - keep some useful styles and even fixes known bugs for common browser dependencies
* Describe z-index and how stacking context is formed.
	* z-index stacks w.r.t parent not root
	* B is on top of A, A and B are siblings, no way that a child of A can ever be on top of B
	* Stacking contexts are independent and self-contained
	* z-index only works on positioned elements (not static)
* Describe Block Formatting Context (BFC) and how it works. - Need more clarity
* What are the various clearing techniques and which is appropriate for what context?
	* class:
	```css
	.clearfix::after{
		content: '';
		height: 0;
		visibility: hidden;
		display: block;
		clear: both;
	}
	```
	* empty div with style="clear: both;"
	* using overflow: hidden or overflow: auto on parent
* Explain CSS sprites, and how you would implement them on a page or site.
	* Multiple images stitched into 1 and used with a class having background-image, background-position and background-size
* How would you approach fixing browser-specific styling issues?
	* Use a UI library like Bootstrap which already handle these issues
	* Use reset or normalize css
	* use autoprefixer for auto prefixing of vendor prefixes
	* Use caniuse for compatiblity check
	* Load a separate stylesheet for that particular browser
	* Use a preprocessor
* How do you serve your pages for feature-constrained browsers? What techniques/processes do you use?
	* Graceful degradation - Develop for modern browsers, make sure backward compatibility is there
	* Progressive development - Develop for basic features, add features as they are supported
	* Use caniuse
	* Use feature checking with modernizr
	* Use @support css rule to check for css feature support
* What are the different ways to visually hide content (and make it available only for screen readers)
	* Position it outside screen using absolute - Method of choice
	* Make its height and width 0
	* Use W3C specifications
	* Use meta tags
	* Use text-indent (only works for text inside blocks) - Performance considerations, use text-indent: 100%;
* Can you give an example of an @media property other than screen?
	* all
	* print
	* speech
	* screen
* What are some of the "gotchas" for writing efficient CSS?
	* Do not use tag selectors or global selectors
	* Keep the selector chain short
	* Use specific classes for all elements and if there is heirarchy, embed it in the classname according to the Block Element Modifier Methodology
	* Avoid rules that change layout or trigger reflow or repaint
	* Do not use !important
	* DRY principle
	* Use multiple stylesheets
	* Be consistent with naming and reusing classes
	* Reuse classes in the class attribute
	* Use shorthands
	* Avoid unnecessary CSS
	* Do not position elements until absolutely needed to
* Can use @font-face for custom fonts, remember to use font-display: swap;
* Explain how a browser determines what elements match a CSS selector
	* Browser will match elements from right to left of the selector chain, they will shortlist all elements with the rightmost key selector and then move to parents to determine if the element matches the entire selector chain
	* For example "p span", first browser will collect all span elements then look if each span has a p ancestor up until root
	* That is why avoid global or tag selectors
* pseudo-elements - Keywords addd after selectors to style a particular part of an element, eg - ::first-line, ::first-letter, can even be used to add elements to the DOM using ::after and ::before, tooltip arrows use these only
* pseudo-classes - Used to style a particular state of an element, :hover, :active, :visited etc.
* What is the CSS display property and can you give a few examples of its use?
	* none (removes from DOM), block, inline-block, inline, flex, grid, table, table-cell, table-row, list-item
* What's the difference between inline and inline-block and block?
	* inline and inline-block and height and width based on content
	* block has width of entire parent minus padding
	* block and inline-block allow height and width to be defined, inline doesnt
	* block and inline-block respect all margins, paddings, inline only respects horizontal
	* Can be aligned with vertical-align? - yes for both inline ones and no for block
* Can you explain the difference between coding a website to be responsive versus using a mobile-first strategy?
	* Both are related
	* Mobile-first is more performant for mobiles
* How is responsive design different from adaptive design?
	* Responsive is one ball resizes for different hoops - Media Queries to adjust to widths
	* Adaptive is different balls for different hoops - Client sniffing and serving different content for different clients
* Have you ever worked with retina graphics? If so, when and what techniques did you use?
	* Marketing name for displays with pixel density greater than 1
	* Use img srcset and sizes attributes
* Is there any reason you'd want to use translate() instead of absolute positioning, or vice-versa? And why?
	* translate does not cause reflow or repaint but causes composition, faster paint and better performance than absolute positioning which causes reflow
	* Absolute uses CPU, translate uses GPU

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
* Promises are objects which represent the eventual success or failure of an asynchronous operation
* Three states:
	* Pending
	* Fulfilled
	* Rejected
* Promises solve the issue of the callback hell which was created when multiple operations had to be done one after the other, several callbacks had to be sent inside nested functions which looks ugly and confusing
```javascript
doTask1(function(res1){
	doTask2(res1, function(res2){
		doTask3(res2, function(finalRes){
			console.log(finalResult);
		}, errorCallback)
	}, errorCallback)
}, errorCallback)
```
* Promises aim to solve this problem using promise chaining
```javascript
doTask1()
.then((res1) => doTask2(res1))
.then((res2) => doTask3(res2))
.then((finalRes) => console.log(finalRes))
.catch((error) => console.log(error));
```
* Remember to return the next promise from the callback else you will break the promise chain!
* Remember promises are guaranteed to be asynchronous and their callbacks will only be called once the call stack is empty even when a callback is attached to an already resolved promise!
```javascript
Promise.resolve(3).then((res) => console.log(res));
console.log(4);
// Output - 4 3
```
* Remember that the rejection of a promise in the chain will go to the next catch handler in the chain
* Always terminate your promise chain with a catch handler otherwise you will get unhandledRejection
* You can also nest promises but this should be avoided unless there is good reason to do so, keep the promise chain flat!
```javascript
doTask1()
.then((res) => 
	doLessImportantTask(res)
	.then((lessImpRes) => doMoreLessImportantTask(lessImpRes))
	.catch((error) => console.log("Less important had error but it does not matter."))
	// Remember the scope of a nested catch handler is always limited as is intentional here, it can only catch errors from these less important
	// tasks, we do not want these less important tasks to fail the main chain
)
.then((res2) => doCriticalTask(res2))
.catch((error) => console.log("Critical failure occured!"));
```
* Remember you can even continue the chain after a catch statement
```javascript
New Promise((resolve) => {
	console.log("Initial")
	resolve();
})
.then(res => {
	throw new Error("Something went wrong");
	console.log("Do this");
})
.catch((error) => console.log("Do this now"))
.then(() => console.log("Do this no matter what happened before"))
.catch((error) => console.log("Do this again"))
	
/*
Output:
Initial
Do this now
Do this no matter what happened before
*/
```
* Async/Await syntax
```javascript
async function execute(initia̵lVal){
	try {
		const res1 = await doTask1(initia̵lVal);
		const res2 = await doTask2(res1);
		const finalResult = await doTask3(res2);
		console.log(finalResult);
	} catch(error){
		console.log(error);
	}
}
```
* Promise callbacks are handled as a microtask whereas setTimeout() callbacks are handled as task queues.
* 4 Concurrency methods
	* Promise.all - Fulfills when all fulfill, rejects if any 1 rejects
	* Promise.allSettled - Settles when all settle whether any fulfills or rejects
	* Promise.any - Fulfills if any 1 fulfills, rejects if all reject
	* Promise.race - Settles when any 1 first settles whether it settles as fulfilled or rejected

### Polyfill for Promise.all
```javascript
function promiseAll(promises) {
  return new Promise((resolve, reject) => {
    const resArray = [];
    let settled = 0;
    for (let i = 0; i < promises.length; i++) {
      Promise.resolve(promises[i])
        .then(res => {
          resArray[i] = res;
          settled++;
          if (settled === promises.length) resolve(resArray);
        })
        .catch(error => reject(error));
    }
  });
}
```
	
### Polyfill for Promise.allSettled
```javascript
function promiseAllSettled(promises) {
  return new Promise((resolve) => {
    const resArray = [];
    let settled = 0;
    for (let i = 0; i < promises.length; i++) {
      Promise.resolve(promises[i])
        .then(res => {
          resArray[i] = res;
          settled++;
          if (settled === promises.length) resolve(resArray);
        })
        .catch(error => {
          resArray[i] = error;
          settled++;
          if (settled === promises.length) resolve(resArray);
        });
    }
  });
}
```
	
### pollyfill for Promise.race
```javascript
function racePromises(promises) {
  return new Promise((resolve, reject) => {
    for (let i = 0; i < promises.length; i++) {
      Promise.resolve(promises[i])
        .then(res => resolve(res))
        .catch(error => reject(error));
    }
  });
}
```

* How does javascript figures out that a promise is resolved?
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
* Memoization can only be done on pure functions whose output depends only on the inputs and not on the "outside world". In other words, for the same inputs, the output will always be the same.
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
* this keyword points to the current scope of execution and is different for normal and arrow functions, see the differences in both in another section

### Destructuring in JS
* Desctructuring allows us to easily extract parts of arrays or objects and assign them to new variables
```javascript
const [firstName, lastName] = ["Abhinav", "Aggarwal"]; // Array desctructuring
console.log(firstName); // Abhinav
console.log(lastName); // Aggarwal

const obj = {
	address: {
		city: "Amritsar",
		state: "Punjab"
	},
	name: "Rahul",
	age: 26
}

const {age, name} = obj; // Object desctructuring
console.log(age); // 26
console.log(name); // Rahul
```

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

### Polyfill for map and filter method
```javascript
Array.prototype.myMap = function(func) {
	const res = [];
	const arr = this;
	for(let i = 0; i < arr.length; i++){
		if(arr[i] !== null && arr[i] !== undefined){
			res.push(func.apply(this, [arr[i], i]));
		}
	}
	return res;
}

Array.prototype.myFilter = function(func){
	const res = [];
	const arr = this;
	for(let i = 0; i < arr.length; i++){
		if(arr[i] !== null && arr[i] !== undefined && func.apply(this, [arr[i], i])){
			res.push(arr[i]);
		}
	}
	return res;
}

const arr = [2, 3, 5, 44, 33, 64];

const mappedArr = arr.myMap((num, index) => {
	return num * index;
});

const filteredArr = arr.myFilter((num, index) => {
	return (index + 1) % 2 == 0; // array of even places
});

console.log(mappedArr);
console.log(filteredArr);
```

### Polyfill for reduce method and reduceRight method
```javascript
Array.prototype.myReduce = function(func, initialVal){
	let res = initialVal ?? null;
	const arr = this;
	for(let i = 0; i < arr.length; i++){
		if(arr[i] !== null && arr[i] !== undefined){
			if(res === null){
				res = arr[i];
				continue;
			}
			res = func.apply(this, [res, arr[i]]);
		}
	}
	return res;
}

Array.prototype.myReduceRight = function(func, initialVal){
	let res = initialVal ?? null;
	const arr = this;
	
	for(let i = arr.length - 1; i >= 0; i--){ // From right to left
		if(arr[i] !== null && arr[i] !== undefined){
			if(res === null){
				res = arr[i];
				continue;
			}
			res = func.apply(this, [res, arr[i]]);
		}
	}
	return res;
}
```

### Pollyfill for find method
```javascript
Array.prototype.myFind = function(func){
	let res = null;
	const arr = this;
	// linear search
	for(let i = 0; i < arr.length; i++){
		if(arr[i] !== null && arr[i] !== undefined && func.apply(this, [arr[i]])){
			res = arr[i];
			break; // Only return first instance
		}
	}
	return res;
}
```
				      
### Snake Case to Camel Case
```javascript
function convertSnakeCaseToCamel(s){
	const arr = s.split("-");
	for(let i = 1; i < arr.length; i++){
  	arr[i] = arr[i].charAt(0).toUpperCase() + arr[i].substring(1);
  }
  return arr.join("");
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

### Immediately Invoked Function Expressions (IIFE)

### Implement own eventmanager

### Implement own redux like lib

### Difference Between Arrow Functions and Normal Functions
* The this keyword
	* In regular functions, the behaviour of this keyword changes based on how the function is invoked
	* Normal invocation - this points to the global scope
	```javascript
	function func(){
		function func2(){
		console.log(this); // Still points to window object only
	  }
	  func2();
	}

	func();
	```
	* Method invocation - this points to the object that owns the method
	```javascript
	const obj = {
		printName: function(){
			console.log(this); // points to obj object
		}
	}
	```
	* Invocation using apply or bind or call - this points to the context provided in apply, call or bind as is their purpose
	* Using new keyword or as constructors - this points to the instance of the function created using new keyword
	```javascript
	function person(name){
		this.name = name;
		console.log(this);
	}

	const p1 = new person("Abhinav");
	/*
	Output:
	{
	  name: "Abhinav"
	}
	*/
	```
	* However in arrow functions, the this keyword will always point towards the lexical scope
	```javascript
	var name = "Abhinav";
	const obj = {
		name: "rahul",
		printName: () => {
			console.log(this.name); // Abhinav
		},
		printName2: function(){
			console.log(this.name); // rahul
    		}
	}
  
	obj.printName(); // Abhinav
	obj.printName2(); // rahul
	```
* arguments keyword
	* In normal functions, arguments passed to the function are accessible using the arguments keyword
	```javascript
	function test(){
		console.log(arguments[0]);
		console.log(arguments[1]);
	}
	test(1, 2);
	```
	* However, much like this keyword, arguments keyword also points to the lexical scope in arrow functions
* As constructors or using new keyword
	* As this keyword in arrow functions points to the lexical scope, they cannot be used with new keyword or as contructors
* Implicit return - Just a fancy way of saying that if the return is immediately followed by the curly braces, the curly braces can be omitted altogether and the function will return what is written
```javascript
const sum = (a,b,c) => a + b + c; // returns the sum
```

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
