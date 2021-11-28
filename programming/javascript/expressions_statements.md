# types

## logical

### NaN equality
Is NaN equal to itself?
```javascript
NaN == NaN  // false
```

### type conversion 1
What is `0 == false` and `'' == false` equal to?
```javascript
0  == false  // true
'' == false  // true
// if you want to avoid type conversion, use '==='
```

### conditional (ternary) operator
Returns one or the other value based on the logical value of the first.
```javascript
// if x evals to true, z will never be evaluated (short-circuit evaluation)
x     ? y : z
true  ? 1 : 2  // returns 1
false ? 1 : 2  // returns 2
```

### using || and && for short-circuit evaluation 
What do `true || y` and `true && y` return?
```javascript
true || y  // true
true && y  // y

/*
 - || returns left value when it can be converted to true, right value otherwise. Use for defaults: if left can smt be empty, fall back on right
 - && returns left if false, right otherwise
*/
```

## strings

### repeat method
```javascript
'foal '.repeat(3)  // 'foal foal foal ', just one param
```

## misc

### naming the type of value
```javascript
x = "this is a string"
typeof x  // "string"
```

# expressions and statements

## expression
> A fragment of code that produces a value is called an expression. Every value that is written literally (such as 22 or "psychoanalysis") is an expression. An expression between parentheses is also an expression, as is a binary operator applied to two expressions or a unary operator applied to one.

## statement
> Code that performs an action. Should end with semicolor ";". Every statement can be replaced by an expression (ie, expressions *are* statements), but not vice versa.

## examples of statements
> loops and if statements, variable declarations 

## control flow and statements
Control flow tools expect statements. If you have only one statement you can avoid using curly braces.

```javascript
if(true)
    console.log('its true')  // will print
if(false)
    console.log('its false')  // won't print
console.log('out of the loop')  // it will print
```

# control flow

## do loop
`do` loop will run at least once

```javascript
var count = 0;
do {
  count ++;
} while (count < 10);
```

## switch
Use switch as a replacement for a lot of if statements. Switch runs statements based on what the given expression is equal to.
```javascript
switch (weather) {
  case "rainy":
    console.log("Remember to bring an umbrella.");
    break;
  case "sunny":
    console.log("Dress lightly.");  // if there is no break, it will continue
  case "cloudy":
    console.log("Go outside.");
    break;
  default:
    console.log("Unknown weather type!");
    break;
}
```

# operators

## shorthand update of values
Using these shorthands you can update a value based on that variable’s previous value.

```javascript
counter += 1; // adds 1
result *= 2;  // multiplies by two
// even shorter!
counter++;    // adds one
```

