Namespaces make it easier to reuse code by importing it and to navigate existing code. In Javascript everything is either global or local to a function. You have to use hacks like objects for public, and functions for private namespacing.

# links

 - DONE great overview and understandable. Only it doesn't go into details with AMD, which is the hardest to understand. [Link](https://medium.freecodecamp.com/javascript-modules-a-beginner-s-guide-783f7d7a5fcc)
 - https://medium.freecodecamp.com/javascript-modules-part-2-module-bundling-5020383cf306
 - https://carldanley.com/js-module-pattern
 - http://www.adequatelygood.com/JavaScript-Module-Pattern-In-Depth.html
 - https://addyosmani.com/resources/essentialjsdesignpatterns/book/#modulepatternjavascript

# functions for namespacing
Idea is that you wrap the code in a function that you immediately run. The code therefore has its own private namespace.

The following module logs a value to the console but does not actually provide any values for other modules to use:
(function() {
function square(x) { return x * x; } var hundred = 100;
console.log(square(hundred)); })();
// → 10000

Why did we wrap the namespace function in a pair of parentheses? This has to do with a quirk in JavaScript’s syntax. If an expression starts with the keyword function, it is a function expression. However, if a statement starts with function, it is a function declaration, which requires a name and, not being an expression, cannot be called by writing parentheses after it. You can think of the extra wrapping parentheses as a trick to force the function to be interpreted as an expression.

# using functions to create namespaces
var dayName = function () {
var names = ["Sunday", "Monday", "Tuesday", "Wednesday",
"Thursday", "Friday", "Saturday"];
return function(number) {
  return names[number]; 
  };
}();

The array names in the above code is inside the outer function's scope and is available to the inner, returned function. I wonder why is the outer function necessary? Is it because names will not get instantiated every time dayName is called?

The same goes for the code below where you pass an object to the function (would it work with only weekDay = {} or var weekDay?) and append named functions to it. Thus the functions from the module are wrapped inside a global object which serves as the namespace.

(function(exports) {
var names = ["Sunday", "Monday", "Tuesday", "Wednesday",
"Thursday", "Friday", "Saturday"];
exports.name = function(number) { return names[number];
};
exports.number = function(name) {
return names.indexOf(name); };
})(this.weekDay = {});

But this approach will lead to problems if two modules claim the same global object. To resolve that issue you need to use require.

# require (CommonJS)
This is a very ingenious namespacing pattern that resolves the issue of namespacing. It does so by wrapping all modules inside a single global variable, require.

I will not go into details, but the idea is this. The module needs to conform to a standard: it needs to wrap its methods and objects into an object named exports. Exports is defined in require as an empty object and is filled with modules.

module:
var names = ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"];
exports.name = function(number) { return names[number];  // exports is global
};
exports.number = function(name) {
  return names.indexOf(name)
};

require:
function require(name) {
var code = new Function("exports", readFile(name));  // readFile is made up but a kind exists in Node.js
var exports = {};  // defines exports in own scope
code(exports);
return exports;
}

A more sophisticated version will store the exports module in require.cache[module_name] so that it can prevent multiple reloads of the same library. Also, it will allow for export variables that are not an object. Details in the book Eloquent Javascript.

# Solving the issue of loading modules: AMD & Browserify
On the web, loading scripts is slow. Therefore, using require calls would freeze your webpage.

One option is to use Browserify which goes through your code when building, resolves all require calls and builds a big file with all your code and dependencies.

AMD uses Ajax to load modules. A particular module with a dependency is defined as a callback function that runs after its dependency is loaded. The details are involved and I couldn't understand it sufficiently. I'd like to find a better resource for it.