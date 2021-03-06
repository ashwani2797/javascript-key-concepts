# JavaScript Done Right!<br> #

# 0. Introduction #

In this meetup, we'll talk about DOM access and manipulation. This can be done in plain JavaScript using the new APIs introduced in ES6. It can also be done using jQuery. We will look at both approaches. We leave it to developers to choose what suits them.

As a study reference, look at the following cheat sheets:
* [https://htmlcheatsheet.com/jquery/](https://htmlcheatsheet.com/jquery/)
* [https://oscarotero.com/jquery/](https://oscarotero.com/jquery/)

# 1. DOM access and manipulation #

The fastest way to select an element is using `getElementById()` because all IDs on a page have to be unique. But this approach is restrictive since many elements will not have IDs. Previously, `getElementsByClassName()` and `getElementsByTagName()` were common approaches. Since 2013, we can use `querySelector()` and `querySelectorAll()` without ever needing jQuery.

Let's start with simple DOM access and compare jQuery syntax with modern JS syntax. We'll use the following example.

```html
<div id="title">List of bugs</div>
<ul>
  <li class="assigned">Bug #233: Lorem ipsum dolor sit amet</li>
  <li class="resolved">Bug #144: Duis aute irure dolor in reprehenderit</li>
  <li class="new">Bug #576: Excepteur sint occaecat cupidatat</li>
  <li class="new">Bug #587: Quis nostrud exercitation ullamco</li>
</ul>
<p class="assigned count">1</p>
```

```javascript
// jQuery
let title = $('#title').text();
let firstNew = $('.new')[0].textContent;
let numAssigned = $('p.assigned').text(); 
console.log(`${title} ${firstNew} ${numAssigned}`);

// JavaScript
let title = document.getElementById('title').textContent;
let firstNew = document.querySelector('.new').textContent; // returns first element
let numAssigned = document.querySelectorAll('p.assigned')[0].textContent; // returns a list
console.log(`${title} ${firstNew} ${numAssigned}`);
```

What exactly is `$` in jQuery? In fact, `$` is a valid name for a variable in JavaScript. `jQuery` library maps such a variable to `jQuery`:
```javascript
window.jQuery = window.$ = jQuery;
```
Either of the following two approaches is valid to access jQuery:
```javascript
// ready() implies that DOM is loaded: doesn't imply other JS scripts have executed

// jQuery accessed from global window scope
$(document).ready(function() {
    let title = $('#title').text();
    console.log(title);
});

// jQuery passed in as an argument
jQuery(document).ready(function($) {
    let title = $('#title').text();
    console.log(title);
});
```

To loop through a list of DOM elements, we can do this:
```javascript
// jQuery with function expression
$('li').each( function () {
    console.log($(this).text());
});

// jQuery with arrow function
$('li').each( (i, item) => console.log($(item).text()) );

// JavaScript
document.querySelectorAll('li').forEach( item => console.log(item.textContent) );
```

The following example shows how to update existing DOM content. In this example, we change the CSS class.
```html
<div class="hello friendly">
  Hello World!
</div>
```
```css
.friendly {
    color: green;
}
.unfriendly {
    color: red;
}
```
```javascript
// jQuery: select all class=hello elements
$('.hello').removeClass('friendly').addClass('unfriendly');

// JavaScript: select first class=hello element
const elem = document.querySelector('.hello');
elem.classList.remove('friendly');
elem.classList.add('unfriendly');
```

Let's see how to add new elements to the DOM.
```html
<div id="title">
  Hello World!
</div>
```
```javascript
// jQuery
$('#title').prepend("<div>JS is for JavaScript!</div>");

// JavaScript
let newDiv = document.createElement("div"); 
let newContent = document.createTextNode("JS is for JavaScript!"); 
newDiv.appendChild(newContent);

var titleDiv = document.getElementById("title"); 
document.body.insertBefore(newDiv, titleDiv); 
```

Here's another example involving a "local form submission". Notice how `const` variables of DOM elements can be modified (but they can't be reassigned to another DOM element). This is how the page will look:
![New comment added](https://gist.githubusercontent.com/arvindpdmn/9fb4c8860bdb9c8d0c7694640965d46e/raw/420f397ff6a7fca6fc5f3a18ed7eae2806588921/comment-form.png)
```html
<div id="comments">
  <p class="comment">I liked the article.</p>
  <p class="comment">Your article is misleading!</p>
</div>
<form>
  <textarea id="commentContent"></textarea>
  <button>Post comment</button>
</form>
```
```javascript
// jQuery
$('button').click(function () {
    const commentContent = $('#commentContent').val();
    $('#comments').append(`<p class="comment">${commentContent}</p>`);
    $('#commentContent').val(''); // clear
    return false; // prevent form submission for this example
});

// JavaScript
$('button').click(function () {
    const commentElem = document.getElementById('commentContent');
    const comments = document.getElementById('comments');
    comments.insertAdjacentHTML('beforeend', `<p class="comment">${commentElem.value}</p>`);
    commentElem.value = ''; // clear
    return false; // prevent form submission for this example
});
```

# 2. Pitfalls and best practices #

Avoid repeat DOM searches. Retrieve once and store the element's access in a variable.
```javascript
// Bad
$('#button').click(function() {
    $('#msg').f1();
    $('#msg').f2();
    $('#msg').f3();
}

// Good
$('#button').click(function() {
    var msgelem = $('#msg');
    msgelem.f1();
    msgelem.f2();
    msgelem.f3();
}
```

Avoid excessive specificity for better performance.
```javascript
// Okay
$( ".data table.attendees td.organizer" );
 
// Better, if the middle class can be dropped
$( ".data td.organizer" );
```

If using jQuery, be aware of how the selectors are translated to DOM access.
```javascript
// Fast since getElementById() is invoked
$("#wrapper");

// Slower since we need to get to inner element
$("#wrapper").find(".inner");

// Probably slowest since querySelectorAll() is invoked
$("#wrapper .inner");
```

Avoid [jQuery Extensions](https://api.jquery.com/category/selectors/jquery-selector-extensions/) since jQuery cannot make use of the native `querySelectorAll()`. In such a case, jQuery will use its own **Sizzle** selector engine.

Avoid doing universal selections since they will be slower.
```javascript
$( ".buttons > *" ); // Extremely expensive.
$( ".buttons" ).children(); // Much better.
 
$( ":radio" ); // Implied universal selection.
$( "*:radio" ); // Same thing, explicit now.
$( "input:radio" ); // Much better.
```

Here's an counter-intuitive example.
```javascript
$( ".danger" );

// Slower! Used to faster in older implementations.
$( "div.danger" );
```

One common error is to assume that a code will be skipped if there are no matches:
```javascript
// Wrong
if ( $( '#subtitle' ) ) { // returns a valid jQuery object
    // this code will always run
}

// Right
if ( $( '#subtitle' ).length ) {
    // runs only if there's a match
}
```

Switching between jQuery objects and DOM elements can lead to hard to debug bugs. Use with caution.
```javascript
// jQuery + DOM
let items = $( 'li' );
let firstone = items[0]; // conversion from jQuery object to DOM element
let html = firstone.innerHTML; // access DOM element property

// jQuery: option 1
let items = $( 'li' );
let firstone = $(items[0]); // convert to DOM element and back to jQuery object
let html = firstone.html();

// jQuery: option 2
let items = $( 'li' );
let firstone = items.eq(0); // remains as jQuery object
let html = firstone.html();
```

In fact, it's common to do chaining of calls in jQuery:
```javascript
let html = $('li').eq(0).html();
```

The function `html()` can be used as a getter or setter. Likewise, we have `text()` function.
```javascript
// Getter
let oldhtml = $('li').eq(0).html();

// Setter
$('li').eq(0).html('<div class="err">This is <b>new</b>!!</div>');
```

Here's an unexpected result. Assuming that the document has four `div` elements, clicking on any `div` will print 3. This is because the function doesn't execute until later, when `i` is already 3.
```javascript
var elements = document.getElementsByTagName('div');
var n = elements.length;
for (var i = 0; i < n; i++) {
    elements[i].addEventListener("click", function () {
        console.log("This is element #" + i);
    });
}
```

There are two ways to solve:
```javascript
// Using closure
var elements = document.getElementsByTagName('div');
var n = elements.length;
function showMsg(i) {
    return function () {
        console.log("This is element #" + i);
    };
}
for (var i = 0; i < n; i++) {
    elements[i].addEventListener("click", showMsg(i));
}

// Using let!
// Because let variables have block-level scope, 
// using them within a function expression enforces closure!
var elements = document.getElementsByTagName('div');
var n = elements.length;
for (let i = 0; i < n; i++) {
    elements[i].addEventListener("click", function () {
        console.log("This is element #" + i);
    });
}
```

Let's look at some cases of memory leaks.
```javascript
// Memory for elem is not reclaimed since it's global and referenced by listener
// even after elem is removed from DOM tree
var btn = document.getElementById("btn");
var elem = document.getElementById("msg");
btn.addEventListener("click", function () {
    elem.remove();
});

// Simple fix
var btn = document.getElementById("btn");
btn.addEventListener("click", function () {
    var elem = document.getElementById("msg");
    elem.remove();
});
```

Here's a memory leak from within a closure, about 10 MB every 10 ms!
```javascript
// function(){} shares the same context as inner() that refers to largeData
// so gc cannot reclaim largeData
var res;

function outer() {
	var largeData = new Array(10000000);	
	var oldRes = res;

    /* Unused but leaks? */
	function inner() {
		if (oldRes) return largeData;
	}

	return function(){};
}

setInterval(function() {
	res = outer();
}, 10);

// Solution is to actually call the function and not just keep its reference
...
setInterval(function() {
	res = outer()();
}, 10);
```

# 3. Event listeners and event processing #

A key press, a form getting submitted, a page or a DOM element getting loaded: all these are events. An executable block of code associated with an event is called an **event handler** or **event listener**. Such handlers can be added to one or more DOM elements. This helps us to add dynamic behaviour to an otherwise static web page content.

It's not generally a good practice to mix HTML (presentation) and event handling (behaviour) by inlining the handlers:
```html
<!-- Bad -->
<button onclick="alert('Hello, world!');">Press me</button>

<!-- Not recommended -->
<script>
    function sayHello() {
        alert('Hello, world!');
    }
</script>
<button onclick="sayHello()">Press me</button>

<!-- Recommended -->
<script>
    let btn = document.querySelector('button');
    
    function sayHello() {
        alert('Hello, world!');
    }
    btn.onclick = sayHello;    
</script>
<button>Press me</button>
```

It's also better to use new function `addEventListener()` instead because we can define multiple handlers:
```javascript
// won't work since only sayHi() will be called
let btn = document.querySelector('button');

function sayHello() {
    alert('Hello, world!');
}

function sayHi() {
    alert('Hi, world!');
}

btn.onclick = sayHello;    
btn.onclick = sayHi;    

// works, sayHello() called before sayHi()
btn.addEventListener('click', sayHello);
btn.addEventListener('click', sayHi);
```

We can also remove handlers at a later point if so required:
```javascript
btn.removeEventListener('click', sayHello);
```

We have seen an example of `click` event but there are lots of event types: focus, submit, resize, scroll, keydown, mouseup, dblclick, drop, drop, load, etc. A longer [list of events](https://developer.mozilla.org/en-US/docs/Web/Events) is available on MDN.

In this example, we'll use a free API to obtain currency exchange rates from INR to other currencies. We'll call this API via AJAX. From the response, we'll display the rate for from INR to a random entry in the response. Let's see the HTML and JS files for this example.

_example1a.htm_
```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <script src="example1a.js"></script> 
        <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.3.1/jquery.min.js"></script> 
    </head>
    <script>
</script>
<body>
    <div id="outer">
        <input id="request" type="button" value="Request">
    </div>
</body>
</html>
```

_example1a.js_
```javascript
$('#request').click(function() {
    $.ajax({
        url: 'http://www.floatrates.com/daily/inr.json',
        method: 'GET',
        complete: function(response) {
            var obj = JSON.parse(response);

            // Select a random entry
            var others = Object.keys(obj);
            var other = obj[others[Math.floor(Math.random()*others.length)]];
            var text = "INR to " + other['code'] + "(" + other['name'] + ") rate is: " + other['rate'] + " at " + other['date'];
            var btn = '<input id="request" type="button" value="Request">';
            $(this).parent().html(text + btn);
        }
    });
});
```

The above will not work for the following reasons:
* jQuery library is not loaded but it's used first: change the order of script inclusion.
* When script executes, DOM is not ready: hence click event listener will not get added.
* Are you sure the response needs to be parsed? Solution is to check the type of response first.
* Is `complete` the right one to use inside `$.ajax`?
* What can you say about using `$(this)` inside `$.ajax`? Either access DOM element directly or better still use arrow syntax.

We make these modifications to have a partly working code. Note that we have also made the text string more maintainable. We should now see an output like the following:

![INR rates](https://gist.githubusercontent.com/arvindpdmn/9fb4c8860bdb9c8d0c7694640965d46e/raw/a1a73883c2aa601963dadc7d2f3e3e58da086445/inr-rates.png)

_example1b.htm_
```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.3.1/jquery.min.js"></script> 
        <script src="example1b.js"></script> 
    </head>
    <script>
</script>
<body>
    <div id="outer">
        <input id="request" type="button" value="Request">
    </div>
</body>
</html>
```

_example1b.js_
```javascript
jQuery(document).ready(function($){
    $('#request').click(function() {
        $.ajax({
            url: 'http://www.floatrates.com/daily/inr.json',
            method: 'GET',
            success: function(response, textStatus, xhr) {
                if (response) {
                    if (xhr.getResponseHeader("content-type").indexOf("application/json") === -1) {
                        var obj = JSON.parse(response);
                    }
                    else {
                        var obj = response;
                    }

                    // Select a random entry
                    var others = Object.keys(obj);
                    var other = obj[others[Math.floor(Math.random()*others.length)]];
                    var text = `INR to ${other['code']} (${other['name']}) rate is: ${other['rate']} at ${other['date']}`;
                    var btn = '<input id="request" type="button" value="Request">';
                    $('#outer').html(text + btn);
                }
            },
            error: function() {},
            complete: function() {}
        });
    });
});
```

But there's still a problem. The first click works but subsequent clicks fail. This is because when we get the first response, we replaced the entire contents of `#outer` element, which unfortunately got rid of the event listener too! Below is one way to solve this by adding a new element to contain the response.

_example1c.htm_
```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.3.1/jquery.min.js"></script> 
        <script src="example1c.js"></script> 
    </head>
    <script>
</script>
<body>
    <div id="outer">
        <div id="randomCurr"></div>
        <input id="request" type="button" value="Request">
    </div>
</body>
</html>
```

_example1c.js_
```javascript
jQuery(document).ready(function($){
    $('#request').click(function() {
        $.ajax({
            url: 'http://www.floatrates.com/daily/inr.json',
            method: 'GET',
            success: function(response, textStatus, xhr) {
                if (response) {
                    if (xhr.getResponseHeader("content-type").indexOf("application/json") === -1) {
                        var obj = JSON.parse(response);
                    }
                    else {
                        var obj = response;
                    }

                    // Select a random entry
                    var others = Object.keys(obj);
                    var other = obj[others[Math.floor(Math.random()*others.length)]];
                    var text = `INR to ${other['code']} (${other['name']}) rate is: ${other['rate']} at ${other['date']}`;
                    $('#randomCurr').html(text);
                }
            },
            error: function() {},
            complete: function() {}
        });
    });
});
```

What if you don't have the luxury to modify the HTML by adding a new `div` to contain the response? The following is the preferred way to solve this. The event handler is still associated with the button but jQuery will reassociate.

_example1d.htm_
```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.3.1/jquery.min.js"></script> 
        <script src="example1d.js"></script> 
    </head>
    <script>
</script>
<body>
    <div id="outer">
        <input id="request" type="button" value="Request">
    </div>
</body>
</html>
```

_example1d.js_
```javascript
jQuery(document).ready(function($){
    $('#outer').on("click", "input#request", function() {
        $.ajax({
            url: 'http://www.floatrates.com/daily/inr.json',
            method: 'GET',
            success: function(response, textStatus, xhr) {
                if (response) {
                    if (xhr.getResponseHeader("content-type").indexOf("application/json") === -1) {
                        var obj = JSON.parse(response);
                    }
                    else {
                        var obj = response;
                    }

                    // Select a random entry
                    var others = Object.keys(obj);
                    var other = obj[others[Math.floor(Math.random()*others.length)]];
                    var text = `INR to ${other['code']} (${other['name']}) rate is: ${other['rate']} at ${other['date']}`;
                    var btn = '<input id="request" type="button" value="Request">';
                    $('#outer').html(text + btn);
                }
            },
            error: function() {},
            complete: function() {}
        });
    });
});
```

In the above example, we used `$('#outer')` inside the success event handler. A better alternative is using arrow syntax (code snippet). Note that `$(this)` refers to the button.
```javascript
...
            success: (response, textStatus, xhr) => {
                if (response) {
                    ...
                    $(this).parent().html(text + btn);
                }
            }
...
```

Let's now look at the above script without using jQuery. Note that we have to add the listener after processing every response.
```javascript
document.addEventListener("DOMContentLoaded", function(event) {
    function showRandomRate () {
        fetch("http://www.floatrates.com/daily/inr.json")
        .then(response => response.json()) // json() returns a Promise
        .then((obj) => {
            // Select a random entry
            var others = Object.keys(obj);
            var other = obj[others[Math.floor(Math.random()*others.length)]];
            var text = `INR to ${other['code']} (${other['name']}) rate is: ${other['rate']} at ${other['date']}`;
            var btn = '<input id="request" type="button" value="Request">';
            this.parentElement.innerHTML = text + btn;
            document.querySelector('input#request').addEventListener("click", showRandomRate);
        });
    }

    document.querySelector('input#request').addEventListener("click", showRandomRate);
});
```

We can also pass an argument to the event handler. This argument gives the full context of the event. Consider the following example of four red squares. When a square is clicked, it's colour changes. At the same time, the new colour is displayed. Take note of how we use `e.target`. We will also see a CSS error (at least in Firefox) since DOM property `backgroundColor` will overwrite CSS `background-color`. To modify the CSS directly, we can use jQuery.

![Four squares](https://gist.githubusercontent.com/arvindpdmn/9fb4c8860bdb9c8d0c7694640965d46e/raw/a1a73883c2aa601963dadc7d2f3e3e58da086445/foursqrs.png)

```html
<div class="wrapper">
    New colour: <span>-</span>
    <br>
    <div class="inner"></div>
    <div class="inner"></div>
    <div class="inner"></div>
    <div class="inner"></div>
</div>
```
```css
.inner {
    float: left;
    width: 25%;
    height: 100px;
    background-color: red;
}
```
```javascript
function getRandomColour() {
    return Math.floor((Math.random()*255) + 1);
}
function bgChange(e) {
    let r = getRandomColour();
    let g = getRandomColour();
    let b = getRandomColour();
    return `rgb(${r},${g},${b})`;
}  

var divs = document.querySelectorAll('div');

for (var i = 0; i < divs.length; i++) {
    divs[i].addEventListener('click', function (e) {
        console.log(this.className);
        if (e.target === this) {
            // jQuery(e.target).css('background-color', bgChange(e));
            e.target.style.backgroundColor = bgChange(e);    
        }
        else {
            this.querySelector('span').textContent = e.target.style.backgroundColor;
        }
    });
}
```

With every click, on the console we'll see "inner", then "wrapper" printed out. This shows that the events of the innermost element are triggered first. This order can be reversed in the above code by passing a third argument to `addEventListener()`:
```javascript
    divs[i].addEventListener('click', function (e) {
        ...
    }, true);
```

To understand how this works, let's look under the hood. Event handlers go through three phases: capturing, targeting, and bubbling. Bubbling the events from the target to its parents is the common approach. However, handlers can be triggered during the capturing phase by setting the third argument to true when calling `addEventListener()`. There are also `e.stopPropagation()` (to stop propagation to children or ancestors) and `e.stopImmediatePropagation()` (to stop propagation plus skip other handlers of current element).
![Phases of event handling](https://mdn.mozillademos.org/files/14075/bubbling-capturing.png)

There are times when you wish to allow the events to propagate but you want to skip the default action. In the following example, the checkbox is not read only and yet it can't be checked.
```html
<form>
  <label for="id-checkbox">Checkbox:</label>
  <input type="checkbox" id="id-checkbox"/>
</form>
<div id="output-box"></div>
```
```javascript
document.querySelector("#id-checkbox").addEventListener("click", function (e) {
    document.getElementById("output-box").innerHTML += 
        "Sorry! <code>preventDefault()</code> won't let you check this!<br>";
    e.preventDefault(); // has effect even if we stop propagation
    // event can still propagate to ancestors
});
```

The third argument to `addEventListener()` can also be an object of options:
```javascript
myelem.addEventListener("click", function (e) {
    ...
}, {
    capture: false,    // for bubbling phase
    once: true,        // handler will be triggered once and then removed
    passive: true      // function is not allowed to call preventDefault()
});
```

The following code using arrow syntax will not work. Can you explain why?
```javascript
    divs[i].addEventListener('click', (e) => {
        console.log(this.className);
        ...
    });
```

Let's recall that `this` is bound to `window` at the time of declaration. This is one place where we don't want to use arrow funtions. Instead, use a function expression where `this` can change depending on the target element.

Can we pass data via events, from one handler to the next? For this, we can use custom events.
```javascript
window.addEventListener("MyEventType", function(evt) {
    alert(evt.msg);
});

// Dispatch an event
var evt = new CustomEvent("MyEventType", {msg: "some data to be passed"});
window.dispatchEvent(evt);
```
