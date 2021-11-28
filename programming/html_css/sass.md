# nesting

## classes or tags
```css
.banner {
    font-family: 'Pacifico', cursive;

    .slogan {
        position: absolute;

        span {
            font-size: 24px;
        }
    }
}
```

## properties
```css
.banner {
  font-family: 'Pacifico', cursive;
  /*border is a property with subproperties which you can nest like this*/
  border: {
    top: 4px solid black;
    bottom: 4px solid black;
  }
}
```

## inserting the parent selector
Use the following character to insert the parent selector in nested blocks, eg when you want to specify a pseudo-class.
```css
.notecard {
  &:hover   { /* selects .notecard:hover */}
  &.simple  { /* selects .notecard.simple */}
}
```

# combining classes

## extending classes
Use the following code to apply all styles from class1 to class2. This way it is unecessary for HTML elements to have both classes.

```css
.class1 {
  border: 1px yellow;
  background-color: #fdd;
}
.class2 {
  @extend .class1;
  border-color: pink;
}
```

## placeholders
Using placeholders you can create a class that is only used to extend other classes.

```css
a%drink {
  font-size: 2em;
  background-color: $lemon-yellow;
}

.lemonade {
  @extend %drink;
}

/* results in */
a.lemonade {
  font-size: 2em;
  background-color: $lemon-yellow;
}
```

### mixins vs placeholders
Mixins insert their content wherever they you @include them. Placeholders reduce repetition in the compiled CSS since they combine selectors. Rule of thumb: use mixins when they are parameterized, otherwise use @extend.

```css
/* definitions */
@mixin no-variable {
  font-size: 12px;
}
%placeholder {
  font-size: 12px;
}

/* SCSS */
span {
  @extend %placeholder;
}
div {
  @extend %placeholder;
}
p {
  @include no-variable;
}
h1 {
  @include no-variable;
}

/* result */
span, div{  /* no repetition */
  font-size: 12px;
}
p {
  font-size: 12px;
  //rules specific to ps
}
h1 {
  font-size: 12px;
  //rules specific to h1s
}

```

# variables

## naming
Using dashes "-"  vs underscores "_"

> SCSS does not differentiate between dashes and underscores, they are compiled to the same thing.

## declaring and using
```css
/* declaring */
$translucent-white: rgba(255,255,255,0.3);

/* referencing */
.banner {
    background-color: $translucent-white;
}
```

## types of variables
```css
$translucent-white: rgb(255,255,255); /* color */
$width: 10px;                         /* number (px does not matter) */
$tag: span                            /* string, with or wout quotes */
$use_smt: false                       /* boolean */
$nothing: null                        /* empty value */

$font: Helvetica, Arial, sans-serif;  /* list, space or comma delimited */
$map_ex: (key1: val1, key2: val2);    /* map */
```

## groups of properties
Create a group of css properties using the following code.

```css
@mixin backface-visibility {
  backface-visibility: hidden;
  -webkit-backface-visibility: hidden;
  -moz-backface-visibility: hidden;
}

.banner {
    @include backface-visibility; /* this is how you use them */
}
```

### parameterized
Create a group of css properties that use the given parameters.

```css
@mixin dashed-border($width, $color: #FFF) {  /* color has a default */
  border: {
     color: $color;
     width: $width;
     style: dashed;
  }
}

.banner {
    @include dashed-border(140px, color: #111);  /* naming is optional */
}
```

### use map as a parameter
Syntax for passing a list or map as a parameter. A list should have parameters in the right order, whereas a map should have keys that match the parameter names!

```css
.banner {
    @include dashed-border(parameter-map...);
}
```

### string concatenation
Include a parameter as a part of a string inside the mixin

```css
/* syntax is begin{string_to_concat}end */ 
@mixin photo-content($file) {
  content: url(#{$file}.jpg);
  object-fit: cover;
}
```

### using &
What does "&" stand for in mixins?

```css
@mixin text-hover($color){
  /* & will get replaced with the parent selector */
  &:hover {
      color: $color; 
  }
}

.text {
    @include text-hover(red);
}
/* compiles to */
.text:hover {
    color: red;
}
```

# functions
reference: http://sass-lang.com/documentation/Sass/Script/Functions.html

## arithmetic
> The Sass arithmetic operations are:
 - addition +
 - subtraction -
 - multiplication *
 - division /
 - modulo %
Keep in mind that the result depends on the unit and the resulting unit needs to be meaningful.

## color functions

### fade-out
Make a color more transparent by taking a number between 0 and 1 and decreasing opacity, or the alpha channel, by that amount

```css
$color: rgba(39, 39, 39, 0.5);
$amount: 0.1;
$color2: fade-out($color, $amount); /* rgba(39, 39, 39, 0.4) */
$color3: fade-in($color, $amount);  /* rgba(55,7,56, 0.6) */
```

### adjust-hue
Change the hue of a color by taking color and a number of degrees (usually between -360 degrees and 360 degrees), and rotate the color wheel by that amount.

```css
$color: rgb(39, 39, 39, 0.5);
$color2: adjust-hue($color, 120deg);
```

### arithmetic operators
You can add colors
```css
.text {
    color: red + blue  /* result is purple */
}
```

# control flow

## looping lists
```css
$list: (orange, purple, teal);
@each $item in $list {
  /* html elements have classes eg .orange, .purple... */
  /* hash tag replaced by variable in curly braces, so a class name*/
  .#{$item} {  
    background: $item;
  }
}
```

## for loop
```css
@for $i from 1 through 10 {
  .ray:nth-child(#{$i}) { /* hash tag replaced by variable in curly braces*/
    background: adjust-hue(blue, $i * $step);
   }
}
```

## inline if conditional
Setting width based on condition
```css
width: if( $condition, $value-if-true, $value-if-false);
```

## full if conditional
A mixin defining color based on the parameter $suit.
```css
@mixin deck($suit) {
 @if($suit == hearts || $suit == spades){ color: blue; }
 @else-if($suit == clovers || $suit == diamonds){ color: red; }
 @else{ //some rule }
}
```

# code organization

## folder organization
> you have a main file at root for @import everything, and additional folders
> 
> one option is to split by functionality
>  - components (buttons, carousel, ...)
>  - layout (grid, header, footer, ...)
>  - pages (SCSS specific for pages)
>  - helpers (variables, functions, mixins, placeholders... stuff that does not create direct CSS)
>  - vendors (CSS or Sass from other projects)

[useful link](http://thesassway.com/beginner/how-to-structure-a-sass-project)

## @import statements
> Sass extends the existing CSS @import rule to allow including other SCSS and Sass files. Typically, all imported SCSS files are imported into a main SCSS file which is then combined to make a single CSS output file.
The main/global SCSS file has access to any variables or mixins defined in its imported files. The @import command takes a filename to import.
>
> By default, @import looks for a Sass file in the same or otherwise specified directory but there are a few circumstances where it will behave just like a CSS @import rule:
 - If the file’s extension is .css.
 - If the filename begins with http://.
 - If the filename is a url().
 - If the @import has any media queries.
If you use a _ prefix that tells Sass to hold off on compiling the file individually and instead import it. When importing, you can omit the underscore and the extension: @import "filename".

