# functions
A function assigns one number *in its domain* to another number (always one number!). A domain is a set of numbers for which the function "makes sense". A function need not only contain an algebraic function, it could, e.g., contain an if statement.

If two functions produce the same output for the same input, they are the same. But functions can be algebraically the same, but still different functions. Eg, $f(x) = (x^2) / x$ is not the same as $g(x) = x$ even if you can turn each into another with algebra. The reason for that is that the domain of `f(x)` does not include 1 (because that would entail dividing by zero), whereas the domain of `g(x)` does.

Some functions

- f(x) = x (identity)
- f(x) = c (constant)
- f(x) = a * x + b (linear)
- f(x) = x ^ n (power function)
- f(x) = a ^ x (exponential)
- f(x) = a*x^3 + b*x^2 + c*x (polynomials)
- f(x) = sqrt(x) (root, an example of a polynomial)
- absolute value
- cos, sin, tan
- logarithm

LIMITS
------------------------------------------------------------------------------------------------------------------------------------------------------------------------

to say that a number L is a limit of a function in relation to some number a, this means that you can the function gets close to L when x gets close to (but not equal to) a

- more formal definition: if you want to get f(x) close to L so that f(x) is no more than N far from L, there exists a number M so that if x is no more than M far from a, you will get f(x) within the required range. In simpler words, for every range that you want f(x) to be close to L, there exists a range for which x needs to be close to a. This is actually the formal definition of limits and you can find the mathematical formulation in the tidbits section
- limits allow you to access "forbidden information" (what happens when x gets close to a number which is not in the domain of the function)
- in some cases the limit is not defined (does not exist)

you also have one-sided limits, which imply that x gets close to a only from one side

- sometimes you have a one-sided limit even if the two-sided limit does not exist - in fact, if the two one-sided limits are not equal, then the two sided limit does not exist

limit laws (see also link)

- limit of a sum is the sum of the limits (provided the limits exists)  =  near the limit of a sum is the sum of the limits
- same goes for a product (limit of a product is the product of the limits, provided the limits exist)
- same goes for quotients (division), with an additional condition - that the limit of the denominator is not zero
    - here is how to think about this addition. When the limit of the denominator is zero, this means that it gets very close to zero, when x gets close to a. Numbers close to zero are either very small positive or very small negative numbers. Dividing anything by the former gives you a huge positive number, whereas dividing by the latter gives you a huge negative number. In other words, the whole expression (the quotient) is not getting near to anything
- the squeeze theorem is used to confirm a limit of a function when you can compare it to two other functions whose limit is easily calculated. At the link there are some famous squeeze theorem results

trick for calculating some limits: you are allowed to algebraically change the function to calculate the limits. This is allowed even if it's true that algebraically equal functions are not the same. Their outputs near the limit, though, might still be the same (even if the output exactly at the limit is not the same).

- use factoring
    - remember that "x" stands for anything. If the limit of sin(x) / x is 1, then it is also so for sin(9x) / 9x, or any other x.

a definition of continuity

- informal: a function is continuous if inputs that are close to each other give outputs that are also close to each other
- formal, at point a: f is continuous at point a if the limit of the function when x gets close to a is equal to f (a)
    - for an open interval (a, b) (this means that it includes the extremes): f is continuous if for every value in the interval the above is true
    - for a closed interval: f is continuous if the same thing holds as for the open interval, only that for a and b the one sided limit has to approach f (a/b); the one sided limit is approaching from the "center" of the interval (it is right sided for a, left sided for b)
- intermediate value theorem says that if a function is continuous for an interval (a, b), then within that interval it has to output all values that are between f(a) and f(b). In a different formulation, if f(a) > c > f(b), and f is continuous in (a, b), there exists an x, such that a > x > b, and f(x) = c
    - this theorem allows you to claim that if f(a) < 0 < f(b), then there has to exist an x so that f(x) = 0. Using this fact, you can find (actually approximate) roots if your function is equal to zero when x is equal to some root. Example: f(x) = x^2 - 2. An x such that f(x) = 0 is actually sqrt(2). You approximate this value by finding an x such that f(x) is an extremely small value, and you do that by "wiggling" the input to f so that it gives ever smaller outputs.

when a limit is equal to infinity, this means that you can make f(x) as big as you like, if you're willing to use x that is close enough to a

- infinity is not a number, which means that you can't do arithmetic with it. If you use limit laws and get infinity as one of the results, you can't work with that. What you have to do is use various algebraic tricks: rewrite the function in a way so that the limits of its constituent parts are not equal to infinity
- some tidbits:
    - you can have negative infinity
    - you can't include infinity in a closed interval
    - potential infinity means you can get a number as large as you like when x -> a, but each actual f(x) is a finite number. Limits are like that. Actual infinity is a actually limitless number, and is impossible to reason about since it implies that there is something that is both definite/complete and infinite (link)

DERIVATIVES: DEFINITIONS
------------------------------------------------------------------------------------------------------------------------------------------------------------------------

starting idea: the slope of a straight line tells you how much does y (the output) change with a unit change of x

derivative:

- definition: a derivative of f at point x is the limit with h going to zero of this expression: f (x + h) - f (x) / h
    - basically, it measures how much the output changes (f (x + h)), for a very small change in the input (the denominator, h)
    - more illustratively, how much does the output f(x) change when you "wiggle" x (= slightly change it)
    - geometrically, the derivative at point a is the slope of the tangent line on the function f, at point a
    - when the derivative is positive, the function is increasing, and it is decreasing when the derivative is negative. This allows you to get an insight into how the function looks like
- if there is a derivative at that point, then we say that f is differentiable at x; f is also in that case continuous at x
    - illustratively, this means that if you "zoom in" on the function at x, it looks like a straight line
    - f can also be differentiable for an open interval, but not for a closed interval. You're "wiggling" x, and you can't wiggle it beyond the range
- the derivative changes for different values of x, so it is actually a function in itself that is derived from the original function. You can have 2nd order derivatives (derivative of a derivative) and higher.
    - derivatives allow you to turn complex functions into straight lines which are easier to understand

Theorem: if a function is differentiable at a, it is also continuous at a. Proven by using the derivative definition

Second order derivative measures how changing the input affects how changing the input affects the output. It's a measure of the change of change.

- An example is acceleration. If velocity is change of location with respect to time, then acc. is the change of velocity, or a change of change.
- 2nd order derivative is a measure of concavity of the function (it's shape).
    - if f''(x) > 0, the function is concave up ("U" shaped) and the rate of change is getting bigger.
    - if f''(x) < 0,  the function is concave down (reverse "U"), the rate of change is getting smaller
    - if f''(x) = 0, the function is at it's inflection point
- The 2nd order derivative can tell you interesting info. At points where the first order der is zero, the second order might be nonzero. The rate of change might be becoming more positive with x, or negative

DERIVATIVES: RULES
------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Rules for derivatives of rational functions
A rational function is the one that can be expressed as a ratio of two polynomial functions

- Constant multiple rule
if g(x) = k * f(x), then g'(x) = k * f'(x). Since the output of g is changing k times faster (or slower) for the same input change, this is also evident in the derivative

- Sum rule
if h(x) = f(x) + g(x), then h'(x) = f'(x) + g'(x)

- Power rule
if f(x) = x ^ n, then f'(x) = n * x ^ (n - 1)

- Product rule (f and g need to be differentiable at the points of interest)
- if h(x) = f(x) * g(x), then h'(x) = f'(x) * g(x) + g'(x) * f(x)

- Quotient rule (f and g need to be differentiable and g(x) != 0 at the points of interest)
- if h(x) = f(x) / g(x), then h'(x) = h'(x) = (f'(x) * g(x) - g'(x) * f(x)) / g(x)^2

DERIVATIVES: USAGE
------------------------------------------------------------------------------------------------------------------------------------------------------------------------

use derivatives to find extreme values (maxima and minima)

- local extrema
f(c) is a local maximum if whenever x is near c, f(c) >= f(x). "Near c" can be more formally defined as: there exists a quantity E > 0, such that whenever x is in (c - E, c + E) -> f(c) >= f(x)
there are also local minima
- global extrema: f(c) >= f(x), or f(c) <=  f(x), whenever x is in the domain of f