# JavaScript Done Right!<br>#

# 0. Introduction #

JavaScript is one of the top programming languages today. It can also be confusing for developers coming from C, Java or PHP backgrounds. Worse still are the numerous frameworks (such as Angular and React) that make it difficult to decide what to learn and how to use the language.

In this short talk/demo, we'll look at the best practices in JavaScript programming. We'll explain using simple examples. This is for beginners and intermediate JS programmers. You must be familiar with JS syntax. We'll not cover DOM access, jQuery, Node.js or JS frameworks.

The recommendations here assume that you use a transpiler such as Babel so that code can work on older browsers. Therefore we freely use syntax from recent ECMAScript standards.

# 1. Basic syntax, typing, scoping, strict mode #

## Style guide ##

Use ample whitespaces, curly braces and indentation for readability.
```javascript
// Bad
if(condition) doSomething();

// Good
if ( condition ) {
  doSomething();
}
```

Multiline strings contain unnecessary extra spaces. Use string concatenation instead.
```javascript
// Bad
var badline = 'hello \
      world';
console.log(badline);

// Good
var goodline = 'hello ' +
      'world';
console.log(goodline);
```

Mainly for readability some recommend object literals rather than object constructors.
```javascript
// Not recommended
var cat = new Object();

cat.name = "kitty";
cat.owner = "Harry";
cat.age = 3;

// Recommended
var cat = {
  name : "kitty",
  owner : "Harry",
  age : 3
}
```

Declare one variable per line. Declare all variables in one scope together, to avoid use of undeclared variables.
```javascript
// Bad
var foo = 123, bar = "abc", baz = [];

// Good
var foo = 123,
  bar = "abc",
  baz = [];
```

All variable declarations should be at the start of their scope.
```javascript
// Bad
function foo () {
  // do something
  var abc = 0;
}

// Good
function foo () {
  var abc = 0;

  // do something
}
```

## Typing ##

Note that `==` compares only value after type coercion whereas `===` is a stricter comparison that compares both type and value. Sometimes comparing different types can introduce bugs and therefore `===` is more suitable. At other times, do explicit conversion of types.
```javascript
console.log(33 == '33')
console.log(33 === '33')

if (0) {
  console.log("foo zero");
}
if ('0') { // non-empty string
  console.log("foo zero string");
}
if (+'0') { // convert to number: commonly used to convert DOM element value (string to number)
  console.log("foo zero string coerced");
}
if ('0' == 0) { // string coerced to number
  console.log("foo zero string == 0");
}
if ('0' === 0) {
  console.log("foo zero string === 0");
}

console.log(false == 0)
console.log(false === 0)

console.log("" == false)
console.log("" === false)

console.log(null == undefined)
console.log(null === undefined)

// NaN is not equal to anything, even itself: use isNaN()
console.log(NaN == NaN)
console.log(NaN == NaN)
console.log(isNaN(NaN))
```

Here are some interesting conversions.
```javascript
var greet = '1';
console.log(typeof greet, greet); // '1'
console.log(typeof greet, typeof +greet, +greet); // 1
console.log(typeof greet, typeof +greet, +greet++); // 1, type change and post-increment
console.log(typeof greet, greet); // 2

var foo = false;
console.log(typeof foo, foo); // false
console.log(typeof foo, typeof +foo, +foo); // 0
console.log(typeof foo, typeof (''+foo), ''+foo); // false
```

While there's `parseInt` function, shift operator is often used to convert floats to integers. Zero-filled right shift on negative numbers will cause problems.
```javascript
var num = 21.6;
var neg = -43.7;

console.log(parseInt(num, 10));
console.log(parseInt(neg, 10));

console.log(num >> 0);
console.log(neg >> 0);

console.log(num >>> 0);
console.log(neg >>> 0); // wrong
```

## Primitives vs objects ##

Be aware of limitations when mixing primitives and objects.
```javascript
var s1 = 'foo';
var s2 = new String('foo');

console.log(typeof s1); // string
console.log(typeof s2);  // object
console.log(s1 == s2); // true
console.log(s1 === s2); // false

// eval works differently
var s1 = '2 + 2';
var s2 = new String('2 + 2');
console.log(eval(s1)); // 4
console.log(eval(s2));
console.log(eval(s2.valueOf())); // 4
```

Objects are created only when `new` keyword is used.
```javascript
var n1 = new Number(11);
var n2 = Number(22);
console.log(typeof n1, n1);
console.log(typeof n2, n2);

if (new Boolean(false)) { // Boolean object
  // will be wrongly entered
  console.log("Inside if 1");
}
if (Boolean(false)) { // boolean primitive
  console.log("Inside if 2");
}
```

Primitives become objects when they are accessed as objects.
```javascript
var n1 = new Number(11);
var n2 = n1 + 2;
console.log(typeof n1, n1);
console.log(typeof n2, n2);
```

## Arrays ##

Iterating through an array can be problematic with some implementations.
```javascript
var a = [11, 22, 33],
  b = new Array(10),
  c = document.getElementsByTagName('body');

// Bad
function badder(arr) {
  for (var key in arr) { // not recommended
    console.log(arr[key]);
  }
}

badder(a);
badder(b);
badder(c);

// Good
function gooder(arr) {
  for (var i = 0; i < arr.length; i++) {
    console.log(arr[i]);
  }
}

gooder(a);
gooder(b);
gooder(c);
```

There are many ways to check if an element is present in an array but `includes` method is the easiest.
```javascript
a = [11, 22, 33];

// Not recommended (using bit operations: -1 in bits is 111...)
console.log(!!~a.indexOf(11))
console.log(!!~a.indexOf(44))

// Not recommended
console.log(a.find((x) => x == 11) === 11)
console.log(a.find((x) => x == 44) === 44)
console.log(a.findIndex((x) => x == 11) >= 0)
console.log(a.findIndex((x) => x == 44) >= 0)

// Recommended
console.log(a.indexOf(11) >= 0)
console.log(a.indexOf(44) >= 0)

// Best
console.log(a.includes(11))
console.log(a.includes(44))
```

To convert from array to string, use of `join` method results in shorter and cleaner code. However, performance depends on browser implementations.
```javascript
var foo = new Array(1000).fill('abcd');

// tested at http://jsben.ch/
// join: faster in Firefox Quantum 61.0.1 (969 ms)
{
  let bar = foo.join("");
  console.log(bar.length);
}

// tested at http://jsben.ch/
// string concatenation: faster in Chrome 69.0.3497.100 (562 ms)
{
  let bar = "";
  for (let i = 0, len = foo.length; i < len; i++){
      bar += foo[i];
  }
  console.log(bar.length);
}
```

## const ##

Note that an object can be declared as constant but it's members can change.
```javascript
const n1 = 12;
n1 = 22; // error

const m1 = {length: 22, width: 9};
m1.width = 4; // okay
m1 = {}; // error
```

## var vs let ##

Use `var` to avoid corrupting global variables.
```javascript
// Bad
abc = 123;
function inner () {
  abc = 567;
  console.log("inner:", abc)
}
inner();
console.log("outer:", abc);

// Good
abc = 123;
function inner () {
  var abc = 567;
  console.log("inner:", abc)
}
inner();
console.log("outer:", abc);
```

Use `let` to limit variable scope only to where it's used.
```javascript
for ( var i = 0; i < 100; i++ ) {
  // statements
}
console.log(i);
for ( let j = 0; j < 100; j++ ) {
  // statements
}
console.log(j); // j is out of scope
```

We can't declare a variable more than once within the same scope using `let`.
```javascript
let abc = 123;
{
  let abc = 567;
  console.log(abc);
}
console.log(abc);

let abc = 789; // error
```

## Security ##

Avoid `eval` since it can be a security risk. Assume a feed coming in JSON format and study the following code.
```javascript
// Bad
var userInfo = eval(feed); // execute any JS code
var email = userInfo['email'];

// Good
var userInfo = JSON.parse(feed); // will throw exception for arbitrary JS code
var email = userInfo['email'];
```

## Strict mode ##

With strict mode, only a restricted JS syntax is permitted. This can result in less bugs. Strict mode applies to entire files or functions. It's not possible to mix strict and non-strict code.

TBD

# 2. Functions, function expressions, arrows, binding #

## Default arguments ##

```
function multiply(a = 3, b = 1) {
  return a * b;
}

console.log(multiply());
console.log(multiply(10));
console.log(multiply(10, 2));
console.log(multiply(b = 10)); // this won't work: order must be maintained
```

## Functions as objects ##

Functions are objects. They can passed into other functions as arguments.
```javascript
function square(f, a, b) {
  return f(a*a, b*b);
}

function add(a, b) {
  return a + b;
}

cube(add, 2, 3);
```

## Function declaration vs expression ##

A function expression is stored in a variable. Function itself has no name and is invoked using the variable's name. With a function declaration, function is named.
```javascript
// Function expression
var foo1 = function() {
  return true;
};  // semicolon here

// Function declaration
function foo2() {
  return true;
}  // no semicolon here but having one is not an error
```

Always use semicolons with function expressions to avoid problems.
```javascript
// Error
Array.prototype.emptyone = function() {
  return [];
}

(function() {
  console.log("Hello");
})();

// Good
Array.prototype.emptyone = function() {
  return [];
};

(function() {
  console.log("Hello");
})();

// What's happening with the first example
Array.prototype.emptyone = function(f) {
  f();
  return alert;
}

(function() {
  console.log("Hello");
})("xxx");
```

However, function expressions can have an identifier, which is necessary when it's recursive.
```javascript
var factorial = function factorial( number ) {
  if ( number < 2 ) {
    return 1;
  }

  return number * factorial( number - 1 );
};
```

All declarations are parsed first. This is called **hoisting**. Therefore, function expressions are useful because they can be defined at runtime, passed in/out of other functions and used in closures.
```javascript
console.log(foo2()); // no error
function foo2() {
  return 456;
}

console.log(foo1()); // an error
var foo1 = function() {
  return 123;
};
```

Here's an example of using a function expression in a nested manner.
```javascript
var friend = {
  country: function (name, city) {
    var name = name;

    var print = function () {
      var message = function () {
        console.log(`I have a friend in ${name}, ${city}.`);
      };

      message();
    };

    print();
  }
};

friend.country("India", "Delhi");
```

The following example illustrates `(function(){})();`, a syntax that's confusing for beginners. This is nothing more than a function expression that's immediately invoked. Hence, these are called **An Introduction to IIFEs - Immediately Invoked Function Expressions (IIFE)**. It's primary advantage is data privacy. Data within the function cannot be accessed outside the function.
```javascript
// Defined but not called
var calllater = function () {
  console.log("Hello world");
};
calllater();

// Example without arguments
// Combines function definition with immediate invocation
(function () {
  console.log("Hello world");
})();

// Example with a single argument
(function (name) {
  console.log(`Hello ${name}`);
})("India");
```

## Arrow functions ##

Arrow functions and arrow function expressions have a shorter syntax.
```javascript
fruits = ["Apple", "Orange", "Pear", "Banana"];

// Traditional for loop
fruits.forEach(function(item, index) {
  console.log(item, index);
});

// Arrow syntax
fruits.forEach((item, index) => {
  console.log(item, index);
});

// Curly braces and semicolon can be omitted for single statements
fruits.forEach((item, index) => console.log(item, index));
```

Here's a function expression in arrow syntax.
```javascript
var double = num => num * 2

console.log(double(4));
```

More importantly, arrow functions don't bind to `this` at invocation: they preserve the original binding.
```javascript
// Bad
function Counter() {
  this.num = 0;

  this.timer = setInterval(function add() {
    this.num++;
    console.log(this.num);
  }, 1000);
}

var c = new Counter();
// clearInterval(c.timer); // to stop timer

// Good
function Counter() {
  this.num = 0;

  this.timer = setInterval(() => {
    this.num++;
    console.log(this.num);
  }, 1000);

  this.stopper = setTimeout(() => {
    clearInterval(this.timer);
  }, 5000);
}

var cc = new Counter();
```

Exactly why does an arrow function preserve the original binding? This can be understood with the following example.
```javascript
// Bad
var user = {
  firstName: "John",
  sayHi() {
    alert(`Hello, ${this.firstName}!`);
  }
};

// this is set to window
setTimeout(user.sayHi, 1000);

// Good
var user = {
  firstName: "John",
  sayHi() {
    alert(`Hello, ${this.firstName}!`);
  }
};

// this is set to user
setTimeout(function() {
  user.sayHi();
}, 1000);

// Arrow syntax
setTimeout(() => user.sayHi(), 1000);
```

## Binding ##

The above example can be extended via binding.
```javascript
var user = {
  firstName: "John",
  sayHi() {
    alert(`Hello, ${this.firstName}!`);
  }
};

let sayHi = user.sayHi.bind(user);
setTimeout(sayHi, 1000);
```

# 3. Object-oriented JS, mixins, closures, prototypal inheritance #

## Extending built-in prototypes ##

This is often not recommended but here's an example of extending the `Array` prototype. It has no method named `first` but we create one. We'll see why this works even when the prototype is updated after the array instance has been created.
```javascript
a = [11, 22, 33];

Array.prototype.first = function () {
  return this[0];
}

console.log(a.first());

delete(Array.prototype.first);

console.log(a.first()); // error!
```

## Checking object properties ##

Here are some ways to check if a property exists.
```javascript
function Counter() {
  this.num = 0;
}

console.log( "num" in Counter ) // false

var cc = new Counter();
console.log( cc.num !== undefined ) // true
console.log( cc.hasOwnProperty( "num" ) ) // true
console.log( "num" in cc ) // true
```

## Mixins ##

TBD

## Closures ##

TBD

## Prototypal inheritance ##

A variable is also an object that has properties. Instances can be created using `Object.create`.
```javascript
var dog = {
  sound: function () {
    console.log('woof!');
  }
};

var spotty = Object.create(dog);

spotty.sound();

console.log(dog.isPrototypeOf(spotty)); // true
console.log(Object.getOwnPropertyNames(dog)) // sound
console.log(dog.hasOwnProperty('sound')); // true
console.log(spotty.hasOwnProperty('sound')); // false
```

All functions are objects that have properties and a prototype. Constructor functions can be invoked using the `new` keyword to make instances.
```javascript
function Cat () {
}

Cat.prototype.sound = function () {
  console.log('meow!');
};

var garfield = new Cat();

garfield.sound();

console.log(Cat.prototype.isPrototypeOf(garfield)); // true
console.log(Object.getOwnPropertyNames(Cat.prototype)) //  contructor, sound
console.log(Cat.prototype.hasOwnProperty('sound')); // true
console.log(Cat.hasOwnProperty('sound')); // false
console.log(garfield.hasOwnProperty('sound')); // false
```

We are essentially creating a new object based on a prototype and then calling the constructor with the `this` bound to the new object. The following code clarifies this.
```javascript
var garfield = new Cat();

// Equivalent to the above line
var garfield = Object.create(Cat.prototype);
Cat.call(garfield)
```

Here's a complete example of prototypal inheritance.
```javascript
function Person(first, last, age, gender, interests) {
  this.name = {
    first,
    last
  };
  this.age = age;
  this.gender = gender;
  this.interests = interests;
}

Person.prototype.bio = function() {
  console.log('Hi! My interests are ' + this.interests + '.');
}

Person.prototype.greeting = function() {
  console.log('Hi! I\'m ' + this.name.first + '.');
};

function Teacher(first, last, age, gender, interests, subject) {
  Person.call(this, first, last, age, gender, interests);

  this.subject = subject;
}
//Teacher.prototype = Object.create(Person.prototype);
//Teacher.prototype.constructor = Teacher;

Teacher.prototype.greeting = function() {
  console.log('Hello! I\'m your teacher. You can call me ' + this.name.first + '.');
};

var teacher1 = new Teacher('Dave', 'Griffiths', 31, 'male', ['football', 'cookery'], 'mathematics');
console.log(teacher1.name.first, teacher1.interests[0], teacher1.subject)
teacher1.greeting();

console.log(Object.getOwnPropertyNames(Teacher.prototype));
teacher1.bio(); // won't work unless the prototype is also inherited
```

![image](https://jordankasper.com/images/preso/JS_prototype_chain.png "Prototypal inheritance chain")

_Image source: https://jordankasper.com/object-oriented-javascript-part-the-second/_

For those from C++ or Java backgrounds, ES6 introduces a more familiar syntax that uses keywords `class`, `extends`, `constructor`, and `super`. Note the use of template literals rather than string concatenation.
```javascript
class Person {
  constructor(first, last, age, gender, interests) {
    this.name = {
      first,
      last
    };
    this.age = age;
    this.gender = gender;
    this.interests = interests;
  }

  greeting() {
    console.log(`Hi! I'm ${this.name.first}.`);
  };

  bio() {
    console.log(`Hi! My interests are ${this.interests}.`);
  };
}

class Teacher extends Person {
  constructor(first, last, age, gender, interests, subject, grade) {
    super(first, last, age, gender, interests);

    // subject and grade are specific to Teacher
    this.subject = subject;
    this.grade = grade;
  }

  greeting() {
    console.log(`Hello! I\'m your teacher. You can call me ${this.name.first}.`);
  }
}

var teacher1 = new Teacher('Dave', 'Griffiths', 31, 'male', ['football', 'cookery'], 'mathematics');
console.log(teacher1.name.first, teacher1.interests[0], teacher1.subject)
teacher1.greeting();

console.log(Object.getOwnPropertyNames(Teacher.prototype));
teacher1.bio(); // will work
```

Finally, an example of implementing private methods using `Symbol` that is an immutable data type.
```javascript
const privateMethod = Symbol('privateMethod');

class Service {
  constructor () {
    this.say = "Hello";
  }
  
  [privateMethod] () {
    console.log(this.say);
  }
  
  publicMethod () {
    this[privateMethod]()
  }
}

new Service().publicMethod()

// Uncaught TypeError: (intermediate value).privateMethod is not a function
new Service().privateMethod()

// Uncaught TypeError: (intermediate value)[Symbol(...)] is not a function
new Service()[Symbol('privateMethod')]();
```

# 4. Async programming, promises #

## JS Browser Execution Model ##

Here are a few things to note:

* All JS code of an app runs in a single browser thread.
* If the thread is blocked, such as in an infinite loop, nothing else can run.
* Each browser tab will have its own thread.
* Web APIs are implemented by browsers and hide low-level complexities such as timeouts, AJAX calls or DOM callbacks. Web APIs are asynchronous. Once they are done, a specified callback function is scheduled for execution.
* Two essentials are the **Event Loop** and the **Task/Message/Callback Queue**. A new task enters the loop when the loop is empty.
* Tasks from the queue are processed in a FIFO order. There's no priority.

![image](https://cdn-images-1.medium.com/max/1600/1*lZ-KXoVNUSOwaq7q8zUBDg.png "JS Browser Execution Model")

_Image Source: https://itnext.io/how-javascript-works-in-browser-and-node-ab7d0d09ac2f_

Can you now explain the log output from the following code? Would it be different if the delay is non-zero?
```javascript
function main() {
  console.log('Hello');
  setTimeout(
    function display () {
      console.log('World');
    }
  , 0);
  console.log('India');
}

main();
```

What about this example? Is the behaviour same in different browsers?
```javascript
function main() {
  console.log('Hello');

  setTimeout( () => alert('World') , 1000); // don't dismiss this alert for about 5 seconds
  setTimeout( () => alert('Mars') , 5000);

  console.log('India');
}

main();
```

## Async vs Defer ##

The following figures show different ways of loading/parsing JS code. Attributes are used to specify async or defer modes. Example, `<script defer src="example.js"></script>`

Async mode is useful on slow connections since DOM parsing can continue while download happens in parallel. Defer mode is useful when executing the JS code makes sense only after the DOM has loaded fully. This is also useful when a file is downloaded first but uses definitions from a file downloaded later.

### Normal execution ###
![image](https://bitsofco.de/content/images/2017/02/Normal-Execution.png "Normal execution")

### Async execution ###
![image](https://bitsofco.de/content/images/2017/02/Async-Execution.png "Async execution")

### Defer execution ###
![image](https://bitsofco.de/content/images/2017/02/Defer-Execution.png "Defer execution")

_Image Source: https://bitsofco.de/async-vs-defer/_

## Promises ##

Refer to [Promises page on Devopedia](https://devopedia.org/promises) for the basics.

## Async / Await ##

TBD

# 5. Useful JS frameworks & libraries #

Frameworks define and guide how developers should write their apps. Libraries on the other hand are called directly by application code or frameworks to perform specific tasks. In general, libraries that have zero dependencies on other libraries must be preferred.

Here are some popular JS frameworks and libraries that can help you develop your app quickly:

* **Frameworks**: [Angular](https://angular.io/), [Ember](https://emberjs.com/), [React](https://reactjs.org/), [Vue](https://vuejs.org/), [Meteor](https://www.meteor.com/), [Node](https://nodejs.org/en/), [Express](http://expressjs.com/), [DoneJS](https://donejs.com/)
* **DOM**: [jQuery](http://jquery.com/)
* **Utility**: [Underscore](https://underscorejs.org/) or [Ramda](https://github.com/ramda/ramda) for functional programming, [Lodash](https://github.com/lodash/lodash), [RxJS](https://github.com/Reactive-Extensions/RxJS) for reactive programming, [Backbone](http://backbonejs.org/) for separating business logic from UI, [Redux](https://github.com/reactjs/redux) for state management, [Moment](https://github.com/moment/moment/) for handling dates and times, [Chance](https://github.com/chancejs/chancejs) for random generation of data, [Mout](http://moutjs.com/) of modular utilities, [RequireJS](https://requirejs.org/) for optimized JS file/module loading, [Voca](https://github.com/panzerdp/voca) for string manipulations
* **Templating**: [Mustache](http://mustache.github.com/), [Handlebars](http://handlebarsjs.com/)
* **UI Elements**: [jQuery UI](http://jqueryui.com/), [Slick](http://kenwheeler.github.io/slick/) for carousels, [jQuery Modal](http://jquerymodal.com/) for modal dialogs, [Polished](https://github.com/styled-components/polished) UI styling using JS
* **Animations**: [Anime](http://anime-js.com/), [Typed](https://github.com/mattboldt/typed.js) for animating text typing, [Three.js](https://github.com/mrdoob/three.js/) for 3D animations, [ScrollReveal](https://github.com/jlmakes/scrollreveal) for scrolling animations
* **Forms**: [Parsley](http://parsleyjs.org/) for form validation, [Garlic](http://garlicjs.org/) for local caching of form data until submitted, [Algolia Places](https://community.algolia.com/places/) to autocomplete fields, [Cleave](http://nosir.github.io/cleave.js/) for formatting fields, [Multiple](https://multiple.js.org/) for sharing backgrounds across multiple elements
* **Backgrounds**: [Bideo](https://rishabhp.github.io/bideo.js/) for video backgrounds, [Granmin](https://sarcadass.github.io/granim.js/) for colourful and animated gradient backgrounds
* **Data Visualization**: [D3.js](https://d3js.org/), [Chart.js](http://www.chartjs.org/)
* **Scientific**: [MathJS](https://github.com/josdejong/mathjs) for computations, [MathJAX](https://www.mathjax.org/) for rendering equations
* **Database**: [TaffyDB](http://taffydb.com/) for DB functionality on the browse
* **Testing**: [Mocha](https://mochajs.org) with assertions using [Chai](chaijs.com), [Puppeteer](https://developers.google.com/web/tools/puppeteer/) or [Cypress](https://www.cypress.io/) for end-to-end app testing, [QUnit](https://qunitjs.com/) for JS unit testing, [Jest](https://facebook.github.io/jest/) for mainly React apps, [Jasmine](https://jasmine.github.io/) for behaviour-driven testing
* **Automation**: [Gulp](https://gulpjs.com/), [Grunt](https://gruntjs.com/)
* **Tools**: [npm](https://www.npmjs.com/) or [Yarn](https://yarnpkg.com/) for package management, [Webpack](https://webpack.js.org/) as a module bundler, [JSHint](http://jshint.com) or [ESLint](https://eslint.org/) for linting, [Babel](https://babeljs.io/) for transpiling ES6+ for older browsers, [TypeScript](https://www.typescriptlang.org/) or [Flow](https://flow.org/) for static type checking
* **Online Editors**: [JS Bin](http://jsbin.com), [JSFiddle](https://jsfiddle.net/), [CodePen](https://codepen.io)
* **Benchmarking**: [jsPerf](https://jsperf.com/) or [JSBen.ch](http://jsben.ch/) to run benchmark performance tests
