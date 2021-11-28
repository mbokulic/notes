@level=2

# main terms

## compiling / transpiling
Compile ES6 or Typescript or something else into ordinary Javascript understood by most browsers. Transpiling is simply a specific type of compiling: taking source code written in one language and transforming into another language *that has a similar level of abstraction*.

## bundles
Requiring each JavaScript dependency as part of your page, script tag by script tag, is slow. Therefore, most sites use so-called JavaScript bundles. The bundling process takes all of your dependencies and "bundles" them together into a single file for inclusion on your page.

## minifying
A step where all unnecessary characters are stripped out of your JavaScript without changing its functionality. This reduces the amount of data that the client will have to download.

## live reloading / watching
Watch source files for changes, recompile, possibly reload the entire page

## hot reloading
Hot reloading, which will live-update your project in your browser when you save a file. Swapping of recompiled modules on the fly, allows live editing. Works best with functional programming.

## sourcemaps
Sourcemaps make debugging easier by providing a mapping from your bundled JavaScript back to its original form.

## code splitting
split certain features into separate bundles to be loaded on demand