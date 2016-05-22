# various tidbits

## equality
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Equality_comparisons_and_sameness

## short shit
Curly braces for body, similar to R

semicolon ; after every statement seems optional in most cases, but recommended

methods called same as Python `object.method()`, and attributes

## declaring vars
use *var* to declare new objects (numbers, strings, functions...). If you do not use it, the interpreter will search for those objects in the available scopes (local, global, idk if you can have nesting)

## case syntax
funny case syntax, it doesn't use the curly braces.
switch(var) {
    case 'crap':
        do stuff;
        break;
    default:
        do stuff;
}

# OOP in Javascript
http://javascriptissexy.com/oop-in-javascript-what-you-need-to-know/
http://www.crockford.com/javascript/private.html

There are many ways to create objects in JS

 - object literals, similar to dict in Python but values can also be functions
 - using *function* keyword to define functionalities, then creating instances with new functionName(). You define the constructor when creating the function (similar to __init__ in Python), and add new methods by overriding the object.prototype property with an object literal. You need to include the constructor when overriding object.prototype or you'll lose it (constructor: Class_name)
 - inheritance using Object.create(parent), this will inherit everything from the parent, including the constructor
 - inheritance using inheritPrototype(child, parent). You inherit everything from the parent, but the constructor is still from child.

**private properties**: in the constructor declare with var

**constructor inheritance**: Parent.call(this, params) will call the Parent constructor, but with *this* (inside Child this refers to the child instance).

**re-usable methods**, or however I should call these. I do not remember seeing this in Python. You define a function but with a call to *this* (equal to self). Then set the function as a method for an object. Where *this* is used, it will call that object.

# interacting with the DOM
The Document Object Model (DOM) is exposed via the *document* object. With methods like *.getElementByID* or *.querySelector* you can access HTML elements.

The *window* object exposes the...window.

# asynchronous HTTP requests
This book has a good intro: http://eloquentjavascript.net/17_http.html

# loading code inside HTML
http://javascript.crockford.com/script.html

Any js code you load inside html with a `<script>` tag will be available to all the later scripts you load. You can e.g. load the d3 library first, then place your d3 code after that.

# Debugging
https://developer.chrome.com/devtools/docs/javascript-debugging

Place 'debugger;' anywhere in the code and it will stop the execution at that point. You need to have the console open. 

# Closures and callback functions
http://javascriptissexy.com/understand-javascript-closures-with-ease/

http://javascriptissexy.com/understand-javascript-callback-functions-and-use-them/

# strict mode
'"use strict mode";' turns strict mode on which turns some silent errors into throws (useful) and restricts some variable definition possibilities which are too loose