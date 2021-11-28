
# OOP in Javascript
Three main pillars of OOP: encapsulation, polymorphism and inheritance.

Objects are elements with named parts called properties (these names can be numbers as well). Object.part1 or object["part1"] gets you to the property. 

Methods are properties that hold functions.

http://javascriptissexy.com/oop-in-javascript-what-you-need-to-know/
http://www.crockford.com/javascript/private.html
http://eloquentjavascript.net/06_object.html

## this
The variable this has special meaning in Javascript. Each function call gets its own this binding. When calling object.method, this inside the method scope refers to the object. If not, this will refer to the global object.

This creates issues when calling functions inside constructors or object methods since *this* will not refer to the parent object. There are several ways to solve this:

 - create a new variable which will point to the object, as in *var self = this*
 - use func.bind(this) to set its *this* variable to the one you want. Other higher-order functions also allow you to set *this*, functions like map or forEach

## prototypes
All objects have a prototype (except the Object object), which is itself another object. Prototypes are used to extend objects with methods and properties.

If you try obj.method(), and the obj does not have that method, then its prototype will be searched for that method. Then its prototype's prototype and so on until you get to Object.prototype which is (I guess) everybodies ancestor. 

Object.getPrototypeOf(obj) returns the prototype of obj. Object.create(obj) will return a new object with the prototype of obj. You can also pass null as the argument and get an object without a prototype. This is useful if you want an object "without the baggage", eg a pure name-value map.

From http://stackoverflow.com/questions/9959727/proto-vs-prototype-in-javascript

So __proto__ is the actual object that is saved and used as the prototype while Myconstructure.prototype is just a blueprint for __proto__ which, is infact the actual object saved and used as the protoype. Hence myobject.prototype wouldnt be a property of the actual object because its just a temporary thing used by the constructor function to outline what myobject.__proto__ should look like.

newCar.__proto__ IS Car.prototype, not an instance of Car.prototype. While Car.protoype IS an instance of an object. Car.prototype is not something that gives newCar any properties or structure, it simply IS the next object in newCar's prototype chain. Car.prototype is not a temporary object. It is the object that is set as the value of the __proto__ property of any new objects made using Car as a constructor. If you want to think of anything as a blueprint object, think of Car as a blueprint for new car-objects

## constructors
Constructors are functions called with the new func() call. They return an instance of itself with all the methods and properties, as well as its prototype. These functions typically do not return anything explicitly, and I don't know what happens to new calls if they do.

Constructors are supposed to contain code relevant to *constructing* the object, whatever that is supposed to mean.

By default, constructors get a blank object as their prototype in func.prototype. If you want to add a method to it, you set its func.prototype.method.

There is a confusing distinction between the prototype property and the actual prototype. Func.prototype gets passed to instances created by new. But Object.getPrototypeOf(func) is the actual prototype that (I suppose) has the methods and properties of func as an individual object that do not get passed. 

## overriding methods
Each object has access to its own properties, the properties of its prototype and its prototype's prototype, etc. 

You can override methods on the prototype level obj.prototype.method1 = function(). These get passed to instances of this object. 

Or you can override on the instance level with
obj.method1 = function()
These are only available to this particular instance.

## getters and setters
Generally, you don't put attributes in the constructor except the ones that are relevant for constructing the object (this seems to be the bare bones only). For example, you don't want to access an attribute with *obj.attr*, if the attr will change later since the constructor is only fired at the beginning.

Some people use getter and setter methods that re-compute the desired attribute, as in *obj.getAttribute()*. But you can also define special get and set methods and be able to use ```obj.attr``` (invokes the getter) or ```obj.attr = x``` (invokes the setter):
```js
var pile = {
  elements: ["eggshell", "orange peel", "worm"],
  set height(value) {
    console.log("Ignoring attempt to set height to", value);
  }
};

// or

Object.defineProperty(TextCell.prototype, "heightProp", {
  get: function() { return this.text.length; }
});
```

## the in operator
"method1" in obj will tell you if obj has that method. You can also iterate over method names with for name in obj. 

The issue is, the in operator will detect methods from all prototypes. If you want to evade those, use obj.hasOwnProperty("method1") which will evaluate to True.

The for loop will only iterate over enumerable properties. The methods from Object.prototype are all nonenumerable so you won't see them. If you want to add an nonenumerable property to an object use the Object.defineProperty method.

## creating objects
There are many ways to create objects in JS

 - object literals, similar to dict in Python but values can also be functions
 - using *function* keyword to define functionalities, then creating instances with new functionName(). You define the constructor when creating the function (similar to __init__ in Python), and add new methods by overriding the object.prototype property with an object literal. You need to include the constructor when overriding object.prototype or you'll lose it (constructor: Class_name)

**private properties**: in the constructor declare with var

**re-usable methods**, or however I should call these. I do not remember seeing this in Python. You define a function but with a call to *this* (equal to self). Then set the function as a method for an object. Where *this* is used, it will call that object.

## inheritance
A straightforward option: inherit the parent constructor by calling it directly in the child constructor, and inherit the prototype with Object.create.

function Child(param) {
  Parent.call(this, param);
}
Child.prototype = Object.create(Parent.prototype);.

**constructor inheritance**: Parent.call(this, params) will call the Parent constructor, but with *this* being the child.

inheritance using Object.create(parent), this will inherit everything from the parent, including the constructor

inheritance using inheritPrototype(child, parent). You inherit everything from the parent, but the constructor is still from child.

**classical inheritance**
Children will correctly inherit attributes from the constructor

https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/create

## should you use inheritance?
Inheritance tends to make your code more complex since new objects depend on old objects. Use it sparingly.

An alternative is to use composition, such that the new object has the old object as a parameter in the call and saved as a property. You can then call the old object methods from within the new object.

## instanceof
If obj.prototype = Object.create(OldObj.prototype), then *obj instanceof OldObj* is true.

## defineProperty
Use `Object.defineProperty(obj, key, descriptor)` to define a property on the object, and to define the parameters for this property. Properties added through Object.defineProperty, in contrast, default to being read-only, write-only, non-deletable, and non-enumerable, but you can customize this.

> - configurable: false disables most changes to the property descriptor and makes the property undeletable
> - enumerable: false hides the property from for..in loops and Object.keys
value: undefined is the initial value for the property
> - writable: false makes the property value immutable
> - get: undefined is a method that acts as the getter for the property
> - set: undefined is a method that receives the new value and updates the property’s value

> Note that when defining a property you’ll have to choose between using value and writable or get and set. When choosing the former you’re configuring a *data descriptor* – this is the kind you get when declaring properties like foo.bar = 'baz', it has a value and it may or may not be writable. When choosing the latter you’re creating an *accessor descriptor*, which is entirely defined by the methods you can use to get() or set(value) the value for the property.

# interacting with the DOM
The Document Object Model (DOM) is exposed via the *document* object. With methods like *.getElementByID* or *.querySelector* you can access HTML elements. The *window* object exposes the global environment.

The DOM is represented as a tree with document.documentElement as the root node. Each node has these properties (not exhaustive)
 - nodeType, a numeric code. 1 is document.ELEMENT_NODE, 3 is TEXT_NODE, 8 is COMMENT_NODE
 - childNodes, contains a nodeList object type (unfortunately not an array). nodeList is live, meaning it changes as the DOM changes. You should loop it from the back to the front because of this.
 - parentNode
 - nextSibling and previousSibling
 - firstChild and lastChild
 - nodeValue
 - getElementByTagName('tag_name'), ...ClassName, ...Id
 - setAttribute, getAttribute
 - querySelectorAll, similar to jQuery()

Browsers do not update their display while a JavaScript program is running. For example, this is why you requestAnimationFrame() when animating

## performance
The browser will not redraw the layout everytime it is changed, but wait as long as it can in order to improve speed. If your js program is switching between changing the layout and reading its information (which requires re-drawing), then the whole thing will run slow.

## event capturing / event-driven programming
The idea is that you do something when an event occurs. You run a callback function when an event fires. The function is passed an event object which contains useful data.

Most of the event capturing in Javascript happens on the DOM (clicking on DOM elements, hovering, etc), but there are also "DOM-less" events like "load".

### event bubbling / propagation through the DOM
An event propagates from child nodes to parents. Event.target stays the same. This allows you, eg, to register a handler on the parent if you have many children, and use the target to do something on the child. You can stop propagation using event.stopPropagation().

The capturing of the event (whatever that is) has the reverse order of propagation: it starts from the highest parent and continues downward. You can use this event trigger order by using `elem.addEventListener(event_type, handler_function, phase=true`, where `phase` is the capture phase.

http://javascript.info/tutorial/bubbling-and-capturing

### prevent default
E.g., when you click on a link, you are taken to that link. But the click event handler fires before you are taken to the link.

Using `event.preventDefault()` you can prevent default behavior. From my limited experience, this seems to stop the child element from doing anything if the event is handled on its parent. Useful if you want to override hotkeys.

 - http://www.w3schools.com/jquery/event_preventdefault.asp

### interesting bits

 - register the button pressed during **mouse** movement using `event.which` or `event.buttons` (depending on the browser)
 - mouseover / mouseout is fired whenever you enter / leave a node, no matter if it is a parent or a child node. This means you have to be clever (check Eloquent Javascript) if you want to know when you enter and leave the parent (or just use `:hover` if that suffices)
 - preventDefault does not prevent **scrolling** since the scroll handler is called after the scrolling takes place
 - **focus** determines which node receives keyboard events. Only specific html types can be focused (eg forms or links) unless you add a tabindex attribute. If none is focused, document.body serves as the target of the event. 
 - focus (and "blur", which means unfocus) events do not propagate
 - **load** event fires after the page has loaded. Useful for scheduling stuff for which you need to load the page for (such as waiting for all DOM events to load then registering event handlers for them)
 - **beforeunload** fires when a page is closed or navigate away from. It enables you to ask a "are you sure you want to close" dialog by returning a string in the .on("beforeunload") handler 
 - use **timers to speed up handlig events that fire too many times**. The idea is that you do not fire the callback every time that the event fires, but wait for the timer to run out.

### web workers
Web workers create a background thread with `worker = new Worker(script_location)`. This thread communicates with the main thread using messages. The worker uses a 'message' handler to respond to main thread messages, and vice versa. Workers have their own environment and have limited access to js functionality (e.g., they do not have DOM access).


# asynchronous HTTP requests
This book has a good intro: http://eloquentjavascript.net/17_http.html

## promises
Promises allow you to chain asynchronous requests to make the code more readable.

https://developers.google.com/web/fundamentals/getting-started/primers/promises
https://www.promisejs.org/

```javascript
// this is a nodejs script so you have to load xmlhttprequest

URL = 'https://httpbin.org/get'
var XMLHttpRequest = require("xmlhttprequest").XMLHttpRequest;

function get(url) {
  return new Promise(function(succeed, fail) {
    var req = new XMLHttpRequest();
    req.open("GET", url, true);
    req.addEventListener("load", function() {
      if (req.status < 400)
        succeed(req.responseText);
      else
        fail(new Error("Request failed: " + req.statusText));
    });
    req.addEventListener("error", function() {
      fail(new Error("Network error"));
    });
    req.send(null);
  });
}

x = get(URL)

// the promise acts as a handle to the request outcome
// It has a then method that you call with two functions
// one to handle success and one to handle failure.

// this will just return a Promise with the string 'success'
y = x.then(function(text) {return 'success'},
       function(text) {return 'failure'})

// this will return a Promise with the parsed JSON
z = x.then(function(text) {return JSON.parse(text)},
           function(text) {return 'failure'})

// calling then returns a new promise object
// so you can chain them

```

Calling .then() produces a new promise, whose result (the value passed to success handlers) depends on the return value of the first function we passed to then. This function may return another promise to indicate that more asynchronous work is being done. In this case, the promise returned by then itself will wait for the promise returned by the handler function, succeeding or failing with the same value when it is resolved. When the handler function returns a nonpromise value, the promise returned by then immediately succeeds with that value as its result. 

The catch method is similar to then, except that it only expects a failure handler and will pass through the result unmodified in case of success.

Similar to AJAX calls, you do not return a value directly from a promise, but you can either assing to a global variable inside a promise, or you do stuff via side-effects. (Or maybe there is another way, but I haven't found it.)

You can wrap FileReader calls in a Promise object and in that way use the same Promise interface for all asynchronous code.

# loading code inside HTML
http://javascript.crockford.com/script.html

Any js code you load inside html with a `<script>` tag will be available to all the later scripts you load. You can e.g. load the d3 library first, then place your d3 code after that.

# Debugging
https://developer.chrome.com/devtools/docs/javascript-debugging

Place 'debugger;' anywhere in the code and it will stop the execution at that point. You need to have the console open. 

## strict mode
'"use strict mode";' will return more errors. Useful for most situations.

 - throws error if not declaring variable
 - if not calling a function as method, this will be equal to undefined. Outside strict mode it is equal to the global object
 - does not allow multiple function parameters with the same name

## exceptions
When an error occurs, an exception is returned. What is special about them is that they return to the bottom of the stack, effectively halting your program.

### handling exceptions
You can place "obstacles" on their way by using try-catch-finally blocks. You put the code you are "testing" in try, the error handling in catch and "clean-up" (something you have to do anyway) in finally.

```javascript
try {
  console.log("You see", look());
} catch (error) { // if an error occurs
  console.log("Something went wrong: " + error);
} finally {}
```

### raising exceptions
To raise an exception, use the keyword throw and give it an exception value. Usually that is done using the Error constructor, as in `throw new Error(message)`.

It is useful to create your own error objects that inherit from Error. That way you can selectively handle exceptions: do something only if this exception type occurs. This is preferrable to "blanket handling"

```javascript
function InputError(message) {
  this.message = message;
  this.stack = (new Error()).stack; // stack call
}
InputError.prototype = Object.create(Error.prototype); InputError.prototype.name = "InputError";
```

Now you can use *err instance of InputError*.

## assertions
Assertions throw an error if a condition is not met. Javascript does not have a native assert function.

# functions
https://javascriptweblog.wordpress.com/2010/07/06/function-declarations-vs-function-expressions/

http://www.2ality.com/2012/09/expressions-vs-statements.html

## expressions and statements
An expression produces a value and can be written wherever a value is expected, for example as an argument in a function call. Roughly, a statement performs an action. Loops and if statements are examples of statements. A program is basically a sequence of statements (we’re ignoring declarations here). Wherever JavaScript expects a statement, you can also write an expression. Such a statement is called an expression statement. The reverse does not hold: you cannot write a statement where JavaScript expects an expression. For example, an if statement cannot become the argument of a function.

a block {} is a sequence of statements. It can be named and you can break from it by calling break block_name

### function expression
function () { }
function foo () { } // name foo only available inside closure

In order to prevent ambiguity, JS forbids expression statements to start with a curly brace or with the keyword function. So what do you do if you want to write an expression statement that starts with either of those two tokens? You can put it in parentheses so that it is interpreted as an expression.

**eval** expects a statement
    > eval("{ foo: 123 }")
    123
    > eval("({ foo: 123 })")
    { foo: 123 }

**immediately invoked function expression**
using parentheses:
    > (function () { return "abc" }())
    'abc'
    
or using a unary operator like void:
void function () { console.log("hello") }()

## the arguments object
JS functions can always take more or less arguments than specified in their definition, so we need a way to access those.

Whenever a function is called, a special variable named `arguments` is added to the environment in which the function body runs. This variable can be used to loop over the argument (eg, this is how console.log works).

The arguments object is an array-like object. The only other standard property of arguments is `callee`. This always refers to the function currently being executed.

```javascript
var myfunc = function(one) {
  arguments.callee === myfunc;
  arguments[0] === one;
  arguments[1] === 2;
  arguments.length === 3;
}

myfunc(1, 2, 3);
```

## apply method: calling func with arbitrary N of args
f.apply(this, array) will call f with an array of args. The first of the args is the context which translates to the this variable.

Since the array can be of arbitrary size, this is how you call the function with an arbitrary number of arguments. 

## call method: similar to apply
func.call(this, arg1, arg2) will call the function with the provided arguments. 

## bind method: return func with some args set
Bind is like a partial apply. function.bind(this, arg1)  will return the function with arg1 set, but you can pass the other arguments. Useful in callbacks.

## Closures and callback functions
A function that closes over local variables is a closure. If you create a reference to that function (ie, store it as a variable), the value of the local var is "frozen". This is how you can, eg, create decorators. 

http://javascriptissexy.com/understand-javascript-closures-with-ease/

http://javascriptissexy.com/understand-javascript-callback-functions-and-use-them/

# SVG
https://www.sitepoint.com/closer-look-svg-path-data/

# efficiency
Very generally: pretty code with abstractions is inefficient. More specifically, functions that beneath the surface **build new objects** (filter, map, reduce) or **call functions are slower**.

# regular expressions
In JS, a regexp is a type of object.

```js
// regexp object
re = new RegExp('abc');
re = /abc/; // alternative, special backslash interpretation
re = /abc/i; // case insensitive

// testing
re.test('abcde'); // testing for match, returns true/false
x = re.exec('abcde'); // returns null if no match, object otherwise
console.log(x) // array equal to the matched string, useful
// array will also contain patterns in parentheses, if they exist

'abcd'.match(/abc/); // string method that does the same
```

## date
`new Date()` returns the current datetime. You can also specify the numbers.

# various tidbits

## global scope is an object named window
var x = 1;
window.x // prints 1

## immutable values are not proper objects
You cannot add a property to an, eg., string.

## equality
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Equality_comparisons_and_sameness

## property (attribute) access
You access properties either by naming them directly after a dot as in object.property. Or you put an expression that evaluates to the correct name in square brackets as in object["property"]. The reason why some properties are always accessed throught brackets even if you know their name is because they are not valid names, e.g. numbers as in array[1].

## short notes
Curly braces for body, similar to R

semicolon ; after every statement seems optional in most cases, but recommended

methods called same as Python `object.method()`, and attributes

x == (null || undefined) iff x equal to one of those two values. Useful since js does conversions when comparison values are not of the same type. If you want to check if x is a real value, compare to that. 

Use || to set a default value: in 'x || y', if x converts to false (eg, is null), the the expression will return y. The && expr will return left if it evaluates to false, otherwise right. This property of not evaluation is what defines **short-circuit evaluation**. Not only JS has this.

**Nested scope** means that vars declared with var inside a function are visible to it and all functions contained within it, but not to higher level functions or the top level. But code blocks defined by {} do not have local scope except if you use the **let** keyword.

**function square(nr) vs var square = function(nr)**. The first is a *function declaration* and this puts the function at the top of its scope. This means you can use the function before you declared it.

## declaring vars
use *var* to declare new objects (numbers, strings, functions...). If you do not use it, the interpreter will search for those objects in the available scopes (local, global, idk if you can have nesting)

## accurate timers using system time
Cannot use `window.setInterval(callback, interval_in_ms)` as this will be slowed down by other processes and thus give inaccurate time. Use system time via `new Date()`.

https://www.sitepoint.com/creating-accurate-timers-in-javascript/
https://github.com/Falc/Tock.js/tree/master

## wait for DOM rendering
https://swizec.com/blog/how-to-properly-wait-for-dom-elements-to-show-up-in-modern-browsers/swizec/6663