@level=3

[great source](https://babeljs.io/learn-es2015/)
[another](https://ponyfoo.com/articles/tagged/es6-in-depth)

# template strings
ES6 template strings are a new string syntax that give you more powerful abilities for constructing strings using javascript

```javascript
var person = 'you'
`Hello, ${person}! 1 + 1 = ${1 + 1}`

// templates can also be multiline
`Hello,
world!`
```

## tagged template strings
You can apply a function to template strings.

```javascript
function transform(list, ...args) {
    // params passed are a list of the non-template parts of the string
    // and the templated parts
    let result = ''
    for(var i=0; i < (list.length - 1); i++) {
        result += list[i] + str_tf(args[i]);
    }
    result += list[i];
    return result;

    function str_tf(string) {
        string = string.replace(/&/g, '&amp;');
        // ... and so on
        return string;
    }
};
transform `this ${here} is a template ${s}`
// list will be equal to ['this ', ' is a template', '']
// args will be equal to [here, s]
```

# arrow functions
Arrow functions reduce the amount of typing for anonymous functions. Similar to lambda in Python.

```javascript
var f = (x, y) => x + y;
// is equal to
var f = function(x, y) {return x + y};
// or with curly braces
var f = (x, y) => {
    return x + y;
};
```

## this in arrow functions
Arrow functions lexically bind the value of `this`: the value of `this` in arrow functions is the same as it was in the enclosing scope.
```javascript
var foot = {
    kick: function () {
        this.yelp = "Ouch!";
        setTimeout(function () {
            console.log(this.yelp);  // this is equal to window
        }, 0);
    }
}

var foot = {
    kick: function () {
        this.yelp = "Ouch!";
        // this is equal to enclosing scope, ie the foot object
        setTimeout(() => console.log(this.yelp), 0);
    }
}
```

## lexical arguments
Arrow functions get their `arguments` object from the parent function.

```javascript
function square() {
  let example = () => {
    let numbers = [];
    for (let number of arguments) {
      numbers.push(number * number);
    }

    return numbers;
  };

  return example();
}

square(2, 4, 7.5, 8, 11.5, 21); // returns: [4, 16, 56.25, 64, 132.25, 441]
```

# destructuring
The destructuring assignment syntax is a JavaScript expression that makes it possible to extract data from arrays or objects into distinct variables. It is a useful technique that allows extracting necessary data from complex elements.

```javascript
// destructuring arrays: elements bound to scope
var numbers = [1, 2, 3];
var [foo, bar] = numbers;  // foo=1, bar=2

// assigning to object properties
var data = {};
[data.foo, data.bar] = numbers;  // data.foo=1, data.bar=2

// ommiting values
var [foo, , bar] = numbers;  // foo=1, bar=3

// destructuring objects
var box = {width: 10, height: 5, depth: 4};
var {width: x, depth} = box;
// x=10, depth=4, width is not defined

// Fail-soft destructuring
var [a] = [];
a === undefined;

// Fail-soft destructuring with defaults
var [a = 1] = [];
a === 1;

// object destructuring and defaults useful for passing objects as params
function g({name: x, intro='hello, my name is '}) {
  console.log(intro + x);
}
g({name: 5}) // prints 'hello my name is 5'


```

# changes in functions

## passing a list of args: spread
An example of a *variadic function* would be Math.max, which you can call with any number of arguments: Math.max(1, 2) or Math.max(1, 2, 3) or ...

In ES6, you can use the ...args syntax to "spread" an array out when calling such a function. This replaces the old `function.apply(this, args)` call.

```javascript
var args = [1, 5, 8];
var max = Math.max(...args);

// with a non-variadic func
function test(a, b, c) {
    console.log(a + 2*b + c);
}
test(...args)  // 19

// also works in arrays
args_copy = [...args];  // args_copy = [1, 5, 8]
```

## creating a variadic function: rest 
Rest parameters are used when you want to write a function that accepts a variadic number of arguments, but acts on them as if they were an array. Replaces the `Array.prototype.slice.call(arguments)` call.

```javascript
function sum(...args) {
    let result = 0;
    args.forEach((x) => {result = result + x});
    return result;
};
```

## default values
ES6 supports default values. They can even be expressions and depend on earlier arguments.

```javascript
var identity   = function(x=1) {return x};
var transform  = function(arg, transform=x => x) {return transform(arg);};
var equal_to_5 = function(arg, error=`${arg} not equal to 5`) {
    assert.strictEqual(arg, 5, error);
};
```

# iterators and generators

## iterator
Iterators are objects with a certain interface: a single method `.next()` which returns an object with two keys:

  * `value`: the next item in the Iterator's collection.
  * `done`: true when the Iterator's collection is exhausted.

Every time next() is called, the Iterator's collection is advanced one more
item, and that item is returned as the value. Once exhausted, value will be the
Iterator's "return value", and done will be true (you may also have infinite sets, like "all even numbers"). All subsequent calls to next() will have done set to true. You are allowed to define next with parameters.

Built in iterators include `Array.forEach()` and the for...in construct for Objects as in `for(key in {1: 1, 2: 2}) {}`.

In ES6 collections like Arrays, TypedArrays and Map and Set all implement the iterator interface. You can access their `.next()` function with `[1, 2][Symbol.iterator]().next()`. To implement that interface you need to define an `object[Symbol.iterator]` function on the object

```javascript
// a function that makes an object into an iterator
function makeCounter(someObj) {
    let counter = 0,
        done    = false;

    someObj.next = function() {
        if(counter < 10)  // simple iteration from 1 to 10
            counter++;
        else
            done = true;

        return {value: counter, done: done}
    }
};
```

## for...of loop
Use this loop to iterate collections like Arrays, TypedArrays, etc. Does not work on any object as not everybody implements the iterator interface (has the `object[Symbol.iterator]()` function.

```javascript
var obj = {elem1: 1, elem2: 2};
for(elem of obj) {
    console.log(elem);
};
```

## generator
A generator is a factory for making an iterator. It is "syntactic sugar" since you do not need to create an object with a `.next` method, it does that automatically and uses the iterator interface with `Symbol.iterator`.

```javascript
function* generate() {  // asterisk after function call
    var num = 0;

    while(true) {
        num += 1;
        // yield call, .next() returns this and freeze execution
        let reset = yield num;
        // to pass an argument to .next, store a val like this
        // execution freezes here, so reset will always be false on 1st call
        if(reset)
            num = 0;
    };
};

var generator = generate();
generator.next();  // 1
for(let x of generator) {
    console.log(x)  // 2, 3, 4, 5...
}
```

## iterator interface
This is the iterator interface: if you want `for...of` to work you need to implement this.

```javascript
// using typescript to describe interfaces

// result has to contain done and value
interface IteratorResult {
  done: boolean;
  value: any;
}

// iterator is an object with a next() method
interface Iterator {
  next(): IteratorResult;
}

// iterable is an object with a Symbol.iterator function that returns an Iterator
interface Iterable {
  [Symbol.iterator](): Iterator
}
```

## defining an iterator
This is defining an object with a function `obj[Symbol.iterator` and this function returns an iterator object that implements the `next()` function.
```javascript
var fibonacci = {
  // Make a method that has the Symbol.iterator function.
  [Symbol.iterator]: function() {
    let pre = 0, cur = 1;
    // The resulting iterator object has to have a next method:
    return {
      next() {
        // The result of next has to be an object with the property `done` that states whether or not the iterator is done.
        [pre, cur] = [cur, pre + cur];
        if (pre < 1000)  return { done: false, value: pre };
        return { done: true };
      }
    }
  }
}

for(let x of fibonacci) {
    console.log(x);
};
```


# classes

```javascript
class Character {
    constructor(health) {
        this.health = health;
    }
    damage(amount) {
        this.health = this.health - amount;
    }
}
var henchman = new Character(100);
```

## extending classes
```javascript
class Character {
    constructor(health) {
        this.health = health;
    }
    get_health() {
        return this.health;
    }
    damage(amount) {
        this.health = this.health - amount;
    }
}

class Armored_character extends Character{
    constructor(health, armor_reduction) {
        super(health);  // calls the parent class constructor
        // must be used before the this keyword can be used
        this.armor_reduction = armor_reduction;
    }
    get_health() {
        super.get_health();  // calls the parent get_health method, passes this
    }
    damage(amount) {
        this.health = this.health - this.armor_reduction * amount;
    }
}

// you can extend built-ins like Array, Date and (DOM) Element
class myArray extends Array {}
```

## getters
```javascript
class Rectangle {
    constructor(height, width) {
        this.height = height;
        this.width = width;
    }

    calcArea() {
      return this.height * this.width;
    }

    get area() {
        return this.calcArea();
    };
};
```

## static methods
Can be called without instantiating a class, but *are not accessible using the `this` keyword from an instance!* (This is different from Python). If you want to call them from an instance, use `classname.static_method_name()` or by calling the method as a property of the constructor `this.constructor.static_method_name()`.

```javascript
class Employee {
    constructor(name) {
        this._name = name;
    }
 
    static convert(thing) {
        if(thing.name) {
            return new Employee(thing.name);
        }
    }
}
```

# namespacing using "export"
```javascript
// export.js
export const message = 'hello'
// use function declarations
export function print(message) {console.log(message);};

// importing
import {message, print} from './export.js';  // I think this is object desctructuring
// or
import * as export from './export.js';
export.message;  // hello
```

## export default
With the `export default` option you do not need to know the name of the objects in the module.

```javascript
// export.js
export default {  // default exposes an object directly
    message: 'hello',
    name:    'greeting'
}

// import.js
import message from './export.js';
console.log(message.message)  // hello

// you can also use curly braces
import {message, print} from './export.js';
```

# block scope
Two new keywords allow the declaration of variables in block scope.

```javascript
{
  var _var = 10;
  let _let = 20;
  const _const = a;
  _var = _let;
  _let = _const;
  _const = 30;  // SyntaxError
}
// a = 20、a is defined with `var` so it is accessible outside of the scope
console.log(_var);
// vars defined with `let` not available: ReferenceError
console.log(b);
// Same goes for const
console.log(tmp);
```

# object literals enhancements
Look [here](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Object_initializer) for a comprehensive intro to all object initialization options

## computed property
Using computed properties you define dynamic property names. Because of this, duplicate property names are possible at runtime. These will not trigger an error under `'use strict';` for this reason.

```javascript
var prop = 'name';
var obj = {
    [name]: 'John',
    [(() => name * 2)()]: 'Peter'  // can be a call to a function
};
```

## shorthands
```javascript
// shorthand property names
var a = 'foo', b = 42, c = {};
var o = {a, b, c};

// shorthand method definition
var o = {
  property([parameters]) {}
};
```

## setting the prototype
```javascript
var obj = {
    // Sets the prototype. "__proto__" or '__proto__' would also work.
    __proto__: theProtoObj,
    // Computed property name does not set prototype or trigger early error for
    // duplicate __proto__ properties.
    ['__proto__']: somethingElse,
};
```

# maps and sets
More info on their API [here](http://2ality.com/2015/01/es6-maps-sets.html)

## sets
WeakSets allow their values to be garbage collected, so no memory leaks.

```javascript
// Sets
var s = new Set();
s.add("hello").add("goodbye").add("hello");
s.size === 2;
s.has("hello") === true;

// Weak Sets
var ws = new WeakSet();
ws.add({ data: 42 });
// the object has no other references so it will not be held in the set
console.log(ws)  // WeakSet {}

```

## maps
Map allows you to map arbitrary values (keys) to other values. Unlike objects that allow you to map strings to values.

WeakMaps allow their values to be garbage collected, so no memory leaks. They do not have the `clear()` method which allows for added security: The mapping from weakmap/key pair value can only be observed or affected by someone who has both the weakmap and the key. With clear(), someone with only the WeakMap would’ve been able to affect the WeakMap-and-key-to-value mapping.

```javascript
// Maps
var s = {1: 1};
var m = new Map();

m.set("hello", 42);
m.set(s, 34);
m.get(s) == 34;

m.size  // 2
m.clear()
m.size  // 0

m..entries() // returns an iterable with one [key,value] pair for each entry in this map

// Weak Maps
var wm = new WeakMap();
wm.set(s, { extra: 42 });
wm.size === undefined
```

# proxies
Proxies allow you to extend or change the default behavior of objects. Useful [link](https://ponyfoo.com/articles/es6-proxies-in-depth).

> You use proxies to to set up "traps" to intercept interactions with the object.

> The target object (the object being proxied) should often be completely hidden from accessors in proxying scenarios. Effectively preventing direct access to the target and instead forcing access to target exclusively through proxy. You can do that using factories.

> Not supported by Babel as it is too complex.

## getter and setter proxies

### implementing private properties
`get` and `set` proxies allow you to implement private properties.

You could implement private properties using function scopes, but proxies allow you to “privatize” property access on different layers. Imagine an underlying underling object that already has several “private” properties, which you still access in some other middletier module that has intimate knowledge of the internals of underling. The middletier module could return a proxied version of underling without having to map the API onto an entirely new object in order to protect those internal variables. Just locking access to any of the “private” properties would suffice!

```javascript
// a private object factory
function private_prop() {
    function invariant (key, action) {
      if (key[0] === '_') {
        throw new Error(`Invalid attempt to ${action} private "${key}" prop!`)
      }
    }
    var handler = {
      get (target, key) {
        // returns the value except when prefixed by '_'
        invariant(key, 'get')
        return target[key]
      },
      set (target, key, value) {
        // sets the value except when prefixed by '_'
        invariant(key, 'set')
        target[key] = value;
        // if 'use strict' the setter needs to return a "truish" value
        return true
      }
    }
    var target = {};
    return new Proxy(target, handler);
};

var x = private_prop();
x._private = 'secret';  // throws an error
```

### implement validation
`get` and `set` proxies allow you to implement validation.

While, yes, you could set up schema validation on the target object itself, doing it on a Proxy means that you separate the validation concerns from the target object, which will go on to live as a Plain Old JavaScript Object happily ever after. Similarly, you can use the proxy as an intermediary for access to many different objects that conform to a schema, without having to rely on prototypal inheritance or ES6 class classes.

```javascript
var validator = {
  set (target, key, value) {
    if (key === 'age') {
      if (typeof value !== 'number' || Number.isNaN(value)) {
        throw new TypeError('Age must be a number')
      }
      if (value <= 0) {
        throw new TypeError('Age must be a positive number')
      }
    }
    return true
  }
};

var person = new Proxy({}, validator);
person.age = 0;  // Error: Age must be a positive number
```

## revocable proxies
We can use Proxy.revocable in a similar way to Proxy. The main differences are that the return value will be `{proxy, revoke}`, and that once `revoke()` is called the proxy will throw on any operation.

This type of Proxy is particularly useful because you can now completely cut off access to the proxy granted to a consumer. You start by passing of a revocable Proxy and keeping around the revoke method (hey, maybe you can use a WeakMap for that), and when its clear that the consumer shouldn’t have access to the target anymore.

Furthermore, since revoke is available on the same scope where your handler traps live, you could set up extremely paranoid rules such as “if a consumer attempts to access a private property more than once, revoke their proxy entirely”.

```javascript
var target = {}
var handler = {}
var {proxy, revoke} = Proxy.revocable(target, handler)
proxy.a = 'b'
console.log(proxy.a)  // 'b'
revoke()
revoke()
console.log(proxy.a)  // TypeError!
```

## other traps
What other operators can you trap with Proxies beside `get` and `set`? More info on [this link](https://ponyfoo.com/articles/es6-proxy-traps-in-depth).

> * `has`, for the `in` operator. Eg you can hide private properties.
> * `deleteProperty` handles the `delete` operator. Eg, you can stop private properties from being deleted.
> * `defineProperty` traps Object.defineProperty as well as `obj.prop = 'value'`.
> * `enumerate` – traps `for..in` loops. Should return an iterator that conforms to the iterator protocol.
> * `ownKeys` traps `Object.keys` and related methods.
> * `apply` traps function calls, eg to log every one of them. Need to use `Reflect.apply(...arguments)`, not `target.apply(...arguments)`.

> and other even more esoteric traps:
> * construct – traps usage of the new operator
 * getPrototypeOf – traps internal calls to [[GetPrototypeOf]]
 * setPrototypeOf – traps calls to Object.setPrototypeOf
 * isExtensible – traps calls to Object.isExtensible
 * preventExtensions – traps calls to Object.preventExtensions
 * getOwnPropertyDescriptor – traps calls to Object.getOwnPropertyDescriptor

## uses for proxies
>
- Add validation rules – and enforce them – on plain old JavaScript objects
- Keep track of every interaction that goes through a proxy
- Decorate objects without changing them at all
- Make certain properties on an object completely invisible to the consumer of a proxy
- Revoke access at will when the consumer should no longer be able to use a proxy
- Modify the arguments passed to a proxied method
- Modify the result produced by a proxied method
- Prevent deletion of specific properties through the proxy
- Prevent new definitions from succeeding, according to the desired property descriptor
- Shuffle arguments around in a constructor
- Return a result other than the object being new-ed up in a constructor
- Swap out the prototype of an object for something else

# Symbol
Symbol is a new primitive in Javascript that gives you a unique value whenever you instantiate it with `Symbol('name')`. They are typically used as a unique value for object properties, especially when you want to have a global unique name for something like iterators in `Symbol.iterator`. They are akin to Python's reserved property names like `__dict__` and they are not enumerable and do not show up in `Object.getOwnProperties`.

# Reflect
ES6's Reflect object is a single ordinary object that contains various methods useful when changing objects or calling their methods.

Has methods similar to the methods defined in Object but on of the major difference is that the Reflect object's method does return meaningful data rather than throwing an error in case of failure.

There is a one-to-one correspondence between "reflect" methods and Proxy traps which ensures that the return type to be compatible with the return type of the Proxy traps as in `Reflect.get(target, name, [receiver])`.

available methods: 

 - Reflect.get(target, name, [receiver])
 - Reflect.set(target, name, value, [receiver])
 - Reflect.has(target, name)
 - Reflect.apply(target, receiver, args)
 - Reflect.construct(target, args)
 - Reflect.getOwnPropertyDescriptor(target, name)
 - Reflect.defineProperty(target, name, desc)
 - Reflect.getPrototypeOf(target)
 - Reflect.setPrototypeOf(target, newProto)
 - Reflect.deleteProperty(target, name)
 - Reflect.enumerate(target)
 - Reflect.preventExtensions(target)
 - Reflect.isExtensible(target)
 - Reflect.ownKeys(target)

# additions to standard library

## Array methods
```javascript
// takes an array-like object or iterator and an optional map func
Array.from(document.querySelectorAll("*"), x => x)

// fills an array with a static value
[0, 0, 0].fill(7, 1) // arr.fill(value, start, end)

// returns index of the first elem that satisfies func
[1,2,3].findIndex(x => x == 2) // 1

// iterating across elems, valid for all iterators
["a", "b", "c"].entries() // iterator [0, "a"], [1,"b"], [2,"c"]
["a", "b", "c"].keys() // iterator 0, 1, 2
["a", "b", "c"].values() // iterator "a", "b", "c"
```

## Object methods
```javascript
// copies all enumerable and own properties to target and returns it
// useful for making a copy of an object
Object.assign({}, { origin: new Point(0,0) }) // (target, ...sources)
```

# promises
What is a promise?

> A Promise is a Javascript object that (typically) does something asynchronous and calls the functions you pass to it when the async work is done. They represent a value that we can handle at some point in the future.

> And, better than callbacks here, Promises give us guarantees about that future value, specifically:
 - No other registered handlers of that value can change it (the Promise is immutable)
 - We are guaranteed to receive the value, regardless of when we register a handler for it, even if it's already resolved (in contrast to events, which can incur race conditions).

> A Promise's state can only change once: it can be fulfilled/resolved or rejected. It can also be "pending".

## constructor
Constructor takes a func as a param. This func describes the async work that  Promise does. It does not need to be async, but why would you use a Promise then? The function takes resolve and reject handlers as params, both are optional. It is most common to use only the resolve and to catch errors downstream.

Unhandled rejections will be reported as warnings.

```javascript
// constructor takes as param a func that does async work 
var promise = new Promise(function(resolve, reject) {
    // resolve and reject should be handlers for a successful and unsuccessful Promise
    setTimeout(function() {
        resolve('resolved!')
    }, 300)
});
// call .then() to specify what happens when the Promise has done its work
promise.then(console.log, null);  // params are resolve and reject respectively
```

## always asynchronous
Promises are always async, ie they must not fire their
resolution/rejection function on the same turn of the event loop that they are
created on.

```javascript
var promise = new Promise(function(fulfill, reject) {
    fulfill('Promise fired!');
});
// the outcome of the promise is already known, but it does not fire yet
promise.then(console.log);
console.log('main program loop') // this is printed first
```

## short notation
```javascript
// CONSTRUCT PROMISE WITH ONLY REJECT
// static method that returns a rejected Promise
var promise = Promise.reject(new Error('error!'));

// CONSTRUCT PROMISE WITH ONLY RESOLVE
var promise = Promise.resolve('success!');

// ONLY RESOLVE AFTER PROMISE CONTRUCTION
// reject is second param to then
promise2.then(null, (err) => console.log(err.message));
// that is same as this
promise2.catch((err) => console.log(err.message));
```

## chaining
Promises can return other Promises, or if they return values these will be wrapped in Promises automatically. These allow you to chain Promises one after another.

The cool thing is that you can catch any errors downstream since errors get passed down the chain. You typically put a call to `catch()` at the end to capture any rejections or exceptions. This is very similar to ordinary `try ... catch` blocks for dealing with errors in sync code. This is actually advisable since any errors in your Promise or even in their reject functions will otherwise not be caught! Remember, a resolve/reject function is for the *previous* Promise.

```javascript
var p = Promise.resolve(4);
p.then((val) => val + 2)  
 .then((val) => { throw new Error("step 2 failed!") }) 
 .then((val) => console.log("got", val))
 .catch((err) => console.log("error: ", err.message));
// => error: step 2 failed!

// but you can still use the resolved promise p that you defined at the beginning
p.then(console.log);  // 2
```

## composing Promises
Use this to do something after all Promises in a list have been resolved.

```javascript
// a wrapper for jQuery.getJSON that returns a Promise
var fetchJSON = function(url) {  
  return new Promise((resolve, reject) => {
    $.getJSON(url)
      .done((json) => resolve(json))
      .fail((xhr, status, err) => reject(status + err.message));
  });
};

// fetch for each URL
var itemUrls = {  
    'http://www.api.com/items/1234',
    'http://www.api.com/items/4567'
  },
itemPromises = itemUrls.map(fetchJSON);  // a list of Promises

// process results
Promise.all(itemPromises)  
  .then(function(results) {
     // we only get here if ALL promises fulfill
     results.forEach(function(item) {
       // process item
     });
  })
  .catch(function(err) {
    // Will catch failure of first failed promise
    console.log("Failed:", err);
  });

```

## race
Get results of first Promise that resolves.

```javascript
// A Promise that times out after ms milliseconds
function delay(ms) {  
  return new Promise((resolve, reject) => {
    setTimeout(resolve, ms);
  });
};

// Which ever Promise fulfills first is the result passed to our handler
Promise.race([  
  fetchJSON('http://www.api.com/profile/currentuser'),  // try to fetch user
  delay(5000).then(() => { user: 'guest' })  // if not, pass 'guest' after 5sec
])
.then(function(json) {
   // this will be 'guest' if fetch takes longer than 5 sec.
   console.log("user:", json.user);  
})
.catch(function(err) {
  console.log("error:", err);
});
```

## serial execution
If you want Promises to run in series, each Promise chained to the previous and receiving its results.

```javascript
// factories: Promise returning functions
// cannot create Promises since they immediately execute
function doFirstThing(){ return Promise.resolve(1); }  
function doSecondThing(res){ return Promise.resolve(res + 1); }  
function lastThing(res){ console.log("result:", res); }

var fnlist = [ doFirstThing, doSecondThing, doThirdThing, lastThing];

// Execute a list of Promise return functions in series
function pseries(list) {  
  var p = Promise.resolve();
  return list.reduce(function(pacc, fn) {
    // will execute each Promise with the results of previous one
    return pacc = pacc.then(fn);
  }, p);
}

pseries(fnlist);  // prints 2
```



## anti patterns

### using a reject function
`then(resolve).catch(reject)` vs `then(resolve, reject)`.

```javascript
// BAD
somethingAsync().then(function(result) {}, function(err) {});

// BETTER
somethingAsync()  
  .then(function(result) { /*handle success*/ })
  .catch(function(err) { /*handle error*/ });
// 
```

### nested promises
What to do if you have more Promises and need to do with the results from all of them?

```javascript
// BAD
firstThingAsync().then(function(result1) {  
  secondThingAsync().then(function(result2) {
    // do something with result1 and result2
  });
},
function(err) {  
  // Errors from secondThingAsync() don't end up here!
});

// BETTER
Promise.all([firstThingAsync, secondThingAsync])  
  .then(function(results) {
    // do something with result1 and result2
    // available as results[0] and results[1] respectively
  })
  .catch(function(err) { /* ... */ });

// if Promise #2 needs the results of #1, use this
firstThingAsync()  
    .then(function(result1) {
        return Promise.all([result1, secondThingAsync(result1)]); 
    })
    .then(function(results) {
        // do something with results array: results[0], results[1]
    })
    .catch(function(err){ /* ... */ });
```

### avoid side effects
> always return something from a Promise: another Promise, a sync value or throw an Error