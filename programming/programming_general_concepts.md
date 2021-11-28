# memoization
Memoisation is an optimization technique used primarily to speed up computer programs by storing the results of expensive function calls and returning the cached result when the same inputs occur again.

# greed in regular expressions
Repetition operators * + ? {} are greedy, which means they match the biggest expression they can. This occurs because of backtracking: a repetition means that the regexp engine stays in the repetition loop (match this, match this, ...) until it arrives at a non-match and the goes back one character (backtracks).

You can turn the operators into non-greedy ones by putting ? behind them. 

# map and reduce
Map applies a function to all elements and returns those elements. This function needs to take in one parameter: the element which is transformed. 

Reduce combines all elements into one value. Sometimes you give it a special starting value. The function that goes into reduce takes two variables, the total and the next value that are combined at each step. E.g., for summing, the steps would be: 1+2=3, 3+3=6, etc.

# immutable vs mutable values
An interesting consequence of mutability is in equality evaluations. Since immutable values are always the same, their references will always evaluate to the same value.

With immutable values, two references can evaluate to the same value (or more precisely, the same content), but still be different. E.g.:

```javascript
object1 = {nr: 1};
object2 = object1; // object1 == object2 evals to true
object3 = {nr: 1}; // object3 == object1 evals to false
```

# functions: side effects vs return values
Functions can create side effects (eg, print in the console, change global var) or return values. Generally, those that return values are more reusable since they can be combined.

Pure functions do not depend on context and always give the same result with the same input. These are easier to test. 

# call stack
When a function runs, the computer must "remember" where it was called so that it can return to that place. It does this by putting the call context onto the call stack. In certain cases (eg infinite recursion) this stack can become larger than the memory allows and thus lead to a stack overflow error. 

# program running time
Running time is calculated as a function of input and takes the form of arithmetic sums. You are executing something each time, from 0 to n. These "somethings" get added up.  You can express the whole thing as a sum that is a function of n. 

An empirical way of approaching this is to implement a counter inside the inner loop of your code. Then plot the counter in relation to the input n. Log-log plots (see math_notes) are very useful here to make the distiction between polynomial and exponential relationships.

![Common arithmetic sums](IMG_0103.PNG)

# breadth-first search
A search strategy for 2D (and I guess and dimensional) space where you start with a particular location and expand radially outward from it. Think for example of a chessboard and starting at the top left corner. It is implemented by designating some locations as the boundary and placing those locations in a queue. The algorithm works by marking the next location in the queue as visited and marking the unvisited locations surrounding it as the new boundaries.

For grids, BFS takes max m x n order of statements.

#Python: generators
Generator expression: use a list comprehension, but put it in normal parenthesis. Does not store all of the elements in memory, but generates them when needed.

Generator function: define a function and return the values using the keyword *yield*. The function will return the values one by one and will not exit like it would with *return*. You can still use *return* to exit the function, but don't return anything. These are typically used with while loops. 

I guess generators are useful for when you don't actually need the whole sequence. That way you conserve memory.

# stacks and queues
Stacks are "last in, first out" (LIFO) data types. You stack elements "on top" of one another, and you always process the last one that got in. The last one you *pushed* is the first one to *pop*.

Queues, on the contrary, are "first in, first out" (FIFO) structures.

In Python, you can use lists natively as stacks, since list.append() will push the item to a stack, and list.pop() will pop the last element you added. For queues, you should use something else, like making your own class.

# classes in Python
A class encapsulates data and the methods you can do with that data.

specific methods for classes
* __init__ initializes the class instance
* __len__ returns the length of the class instance
* __str__ the print output
* __iter__ the iterator, the thing you get when you use the instance in a for loop. Use generator functions here

# object oriented style
build reusable code. Example: if you build a card class for Blackjack, don't put a value method in because cards have different values, or no values, depending on the game.

# class inheritance
Using *class class_name(inherits_from)* you can inherit all of the methods from the old class in the new class. 

# duck typing
Python does not force you to declare types, nor does it force your functions to only accept a given type. Because of this, duck typing is used: if it quacks like a duck, and walks like a duck, then it's a duck. By this, it means that if your object has the required methods, then you can use it in the given context.

Classes and inheritance are used to achieve duck typing.