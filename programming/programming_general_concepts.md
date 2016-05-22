### input in the command line
Use the input() or raw_input() functions. The first will take a string or smt that can be converted into it and return that string. The second will turn a literal into a string. I.e., you can write text without surrounding it in quotation marks.

### program running time
Running time is calculated as a function of input and takes the form of arithmetic sums. You are executing something each time, from 0 to n. These "somethings" get added up.  You can express the whole thing as a sum that is a function of n. 

An empirical way of approaching this is to implement a counter inside the inner loop of your code. Then plot the counter in relation to the input n. Log-log plots (see math_notes) are very useful here to make the distiction between polynomial and exponential relationships.

![Common arithmetic sums](IMG_0103.PNG)

### search
Many applications use search. Examples:
* Google Maps searches spatially for your location
* FPS games NPCs search for you in the 3D world
* 1D search of a student's file based on student ID

### breadth-first search
A search strategy for 2D (and I guess and dimensional) space where you start with a particular location and expand radially outward from it. Think for example of a chessboard and starting at the top left corner. It is implemented by designating some locations as the boundary and placing those locations in a queue. The algorithm works by marking the next location in the queue as visited and marking the unvisited locations surrounding it as the new boundaries.

For grids, BFS takes max m x n order of statements.

###Python: generators
Generator expression: use a list comprehension, but put it in normal parenthesis. Does not store all of the elements in memory, but generates them when needed.

Generator function: define a function and return the values using the keyword *yield*. The function will return the values one by one and will not exit like it would with *return*. You can still use *return* to exit the function, but don't return anything. These are typically used with while loops. 

I guess generators are useful for when you don't actually need the whole sequence. That way you conserve memory.

### stacks and queues
Stacks are "last in, first out" (LIFO) data types. You stack elements "on top" of one another, and you always process the last one that got in. The last one you *pushed* is the first one to *pop*.

Queues, on the contrary, are "first in, first out" (FIFO) structures.

In Python, you can use lists natively as stacks, since list.append() will push the item to a stack, and list.pop() will pop the last element you added. For queues, you should use something else, like making your own class.

### classes in Python
A class encapsulates data and the methods you can do with that data.

specific methods for classes
* __init__ initializes the class instance
* __len__ returns the length of the class instance
* __str__ the print output
* __iter__ the iterator, the thing you get when you use the instance in a for loop. Use generator functions here

### object oriented style
build reusable code. Example: if you build a card class for Blackjack, don't put a value method in because cards have different values, or no values, depending on the game.

### class inheritance
Using *class class_name(inherits_from)* you can inherit all of the methods from the old class in the new class. 

### duck typing
Python does not force you to declare types, nor does it force your functions to only accept a given type. Because of this, duck typing is used: if it quacks like a duck, and walks like a duck, then it's a duck. By this, it means that if your object has the required methods, then you can use it in the given context.

Classes and inheritance are used to achieve duck typing.