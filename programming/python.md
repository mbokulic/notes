## getting started testing ([youtube](https://www.youtube.com/watch?v=FxSsnHeWQBY))

running unittest on CL: "-m" means, instead of running a Python program in a file, run the importable module "unittest" as the main program. The unittest module accepts a number of forms of command-line arguments telling it what to run. In this case we give it the module name "test_port1". It will import that module, find the test classes and methods in it, and run them all.

```python -m unittest test_port1 ```

Use **assert helpers** that are methods in the base TestCase class that let you test specific assertions (e.g., TestCase.assertEqual). They provide more useful exception output.

Use your **own base class** since you will probably create your own methods that extend the base methods.

## python class development toolkit[video](https://www.youtube.com/watch?v=HTLu2DFOdTg&index=97&list=WL)

**new-style class for python**: in py2 you have to inherit from object

```python
class Circle(object): # new-style class
    version = 1.0 # class variable, same for all instances

    def __init__(self, radius): # init not a constructor, self already exists
        self.radius = radius # instance variable, has to be unique to inst
```

In Python, it is not necessary to construct get() and set() methods. You can access and modify instance variables directly.

**Overriding vs extending methods in child classes**: in overriding, you replace the parent method. In extending, you call the parent method and then change its output.

```python
Class Tire(Circle):

    def perimeter(self):
        return Circle.perimeter(self) * 1.25 # extending the parent method
```

**Alternative constructors** mean that you construct an instance, but not using the standard method. You should create these by using **class methods** which take the class, not the instance (self) as their parameter. Thus, they will work in all of the child classes. [SO link](http://stackoverflow.com/questions/12179271/python-classmethod-and-staticmethod-for-beginner). A common use case is if you want alternative initialization methods (see below).

```python
@classmethod
def from_bounding_box(cls, diagonal): # cls is the passed class
    '''create circle from bounding box diagonal'''
    radius = bb / 2.0 / math.sqrt(2.0)
    return cls(radius) # return class instance with the calculated radius
```

Similar to class methods, **static methods** are functions attached to classes and every child inherits them. But they do not take the class as the first parameter. Use this for functions you want to keep in the class namespace, but you don't want to create a class instance (nor need to) to call them. You do this so that a) it is easier to find the function and b) you suggest the context in which it should be used.

```python
@staticmethod
def angle_to_grade(angle):
    '''convert angle degrees to percentage grade'''
    pass
```

**Class local reference** means that you have an object that is specific to your class, and does not get overwritten or extended in the child classes. The syntax is to use double underscores. That object then looks like this: _classname__objectname. 

```python
class Parent():

    def my_method(self):
        return(1)

    __my_method = my_method # __my_method will stay the same in children
```

**Single underscores** are used as a convention for "private" methods or variables, though they are not really private. These will not get imported when using ```from x import *```. ([link](https://shahriar.svbtle.com/underscores-in-python))

**Convert attribute call to method call** using ```@property```. 

```python
class Circle(object):

    def __init__(self, radius):
        self.radius = radius

    @property # you will be able to access radius as self.radius 
    def radius(self):
        return self.diameter / 2.0

    @radius.setter # I guess this is what gets run first
    def radius(self, radius):
        self.diameter = radius * 2.0
```

**Memory optimization using __slots__ (Flyweight Design Pattern)**. [Docs link](https://docs.python.org/2/reference/datamodel.html#slots). You don't save the whole dictionary (no dir method!), just the objects that you specify. Only use when you must. Subclasses do not inherit slots.

## Beyond PEP-8: write Pythonic code ([youtube](https://www.youtube.com/watch?v=wf-BqAjZb8M))

magic methods:
 - __len__ for length
 - __getitem__ and __setitem__ for making your object iterable
 - __repr__ is a representation of the class that captures everything in it. Useful for debugging (http://stackoverflow.com/questions/1436703/difference-between-str-and-repr-in-python)

Use **context managers**, the _with_ statement.

Create **custom exceptions** since they will tell you exactly what is happening.

Use **named tuples** if you have a collection of values that you want to be able to access by name ([documentation link](https://docs.python.org/2/library/collections.html#collections.namedtuple)).