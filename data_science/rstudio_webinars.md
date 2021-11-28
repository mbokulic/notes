# debugging

## standard library
`browser()` is similar to `debugger` in Javascript. In RStudio you get code highlighting and an overview of the environment. Press 'c + enter' to exit debug mode. [Link](http://rfunction.com/archives/2633). RStudio can use breakpoints instead of `browser()`.

`debug(function_name)` is the same, but it puts the whole function in debugging mode. `undebug(function_name)` turns that off. There is also `debugonce(function_name)`.

`traceback()` prints the call stack of the last uncaught error

`options(error=browser)` will always open the browser on error

For **Shiny** apps, use `options(shiny.error=browser)`.

[debugging with RStudio](https://support.rstudio.com/hc/en-us/articles/205612627-Debugging-with-RStudio)

# libraries

## visualizations
dygraph: javascript
htmlwidgets
diagrammer

# shiny

## input functions
[useful gallery](https://shiny.rstudio.com/gallery/widget-gallery.html), seems to contain all of them

## HTML functions

### static HTML elements
`tags$tag_name` contains functions that create HTML tags. The parameters are HTML attributes (like 'href', 'class', etc), whereas unnamed arguments will go inside the tag as text or other nested tags. Good thing is specifying classes will not erase the original classes necessary for the UI.

You can also pass raw HTML with the `HTML('html_string')` function. There are other functions, check [here](https://shiny.rstudio.com/reference/shiny/latest/).

### layout functions
Organize the layout in a grid using `fluidPage()` with `fluidRow()` and `fluidColumn()`. Columns have 12 points. [Link](https://shiny.rstudio.com/articles/layout-guide.html). There are also templates for layouts like `sidebarLayout()`.

Besides `fluidPage()` you also have

 - `fixedPage()` for fixed width apps
 - `navbarPage()` is a collection of panels and you have a navbar at the top
 - `dashboardPage()` in the `shinydashboard` package ([link](https://rstudio.github.io/shinydashboard/)). Very useful since it has templates, icons, valueboxes, etc.

Use panels for grouping elements like `wellPanel()` or `tabPanel()`. I couldn't find a gallery for them, but you can check all functions [here](https://shiny.rstudio.com/reference/shiny/latest/). [An article](https://shiny.rstudio.com/articles/layout-guide.html) about layout functions.

### external HTML, CSS, JS, Markdown
You can use an external HTML file (with Javascript!) to create the app. Check it [here](https://shiny.rstudio.com/articles/html-ui.html). The IDs of HTML elements acting as output will serve as `output$id` names, whereas the name tag will be used for input elements. You also need to use Shiny CSS classes for outputs: shiny-text-output, shiny-plot-output, or shiny-html-output.

Link to an external CSS file using the `theme` parameter in layout functions, or create a `head` tag with a link to the CSS sheet. You can also use inline CSS, but I would prefer not to. Put CSS in the /www directory. [Link](https://shiny.rstudio.com/articles/css.html).

Check `?includeHTML` to see that one and other functions that allow you to include external files in your app.

## output functions
`plotOutput()`, `tableOutput()`, etc. They reserve space for outputs.

[Link](https://shiny.rstudio.com/reference/shiny/latest/)

### HTML templates
Templates similar to Handlebars and others. You create a template and process it using `htmlTemplate(filename='index.html', ...)` where the named `...` arguments will populate the double handlebars in the template.

You should always call `{{headContent()}}` and `{{bootstrapLib()}}` in the header.

[This](https://beta.rstudioconnect.com/jcheng/scorecard-app/) is just amazing! [An article](https://shiny.rstudio.com/articles/templates.html) with more details.

### htmlwidgets()
Embed Javascript visualizations inside Shiny (also works with RMarkdown and in the Viewer). They act similar to other Shiny code: you have an UI component that returns HTML and a server component where you bind data. [Check here](http://rstudio.github.io/dygraphs/shiny.html) for an example.

## render functions
`renderPlot()`, `renderTable()`, etc. Used in the server function to specify a function for drawing the output. Only these functions accept reactive values (input values) as their parameters. This looks like assigment but it's not (more info in [Chang's presentation](https://www.rstudio.com/resources/videos/effective-reactive-programming/)). (I think it's a handler.)

```r
server = function(input, output) {
    output$plot = renderPlot(expr={
        plot(input$x, input$y)
        })
}
```

### interactive plots
You can add interactivity to plots using `click`, `dblclick`, `hover` and `brush` parameters. These allow you to setup handlers for plot interactions in the server function. [Example](https://shiny.rstudio.com/gallery/plot-interaction-basic.html). Works with ggplot2 best, but also base graphics.

"Opts" functions allow you to set more options. Eg, `plotOutput(brush=brushOpts(direction='x'))` will set up a brush interactivity only on the x-axis.

Useful functions for interactive plots:

 - `nearPoints()` will return N nearest rows of the data frame
 - `stopApp(value)` will stop the Shiny app and return the value. Useful for local exploratory work.

## functions for manipulating reactivity
[How reactivity works](https://shiny.rstudio.com/articles/understanding-reactivity.html). The gist of it is that the Shiny app is pinging every N msecs to see if there are changes in the reactive values (= onchange event dispatchers). If yes, then it runs the callbacks for the reactive values.

Most reactive expressions / handlers will run once on start except observeEvent, but you can override this.

### reactive()
The `reactive()` function runs an expression that fires when input values inside it change. Returns a function that you can use in render functions, this function will return the result of that expression. This code will only be run once when inputs change, meaning you can use this to cache data. Will not execute until it is needed.

You can use reactive expressions inside other reactive expressions. But if you need a specific order of execution think of another way of expressing that since you cannot guarantee when they will execute (bcz they are "lazy").

use cases

 - storing a result of a calculation which will be used by more than one render function (e.g., plot + table)

### isolate()
`isolate(input$value)` unsubscribes the render function from the input value. The returned value of `isolate()` is used as a parameter in the render function.

### observeEvent()
Register an event handler. Takes an input value or reactive expression call as the first argument. Usually used with `actionButton`. The reactive values in the handler are isolated. Check [this link](https://shiny.rstudio.com/articles/action-buttons.html) for action buttons.

`observe()` is the same, but runs code whenever any of the reactive values in the block of code change. You do not need to specify the event you are listening.

Both functions do not return a value but a reference to the handler. You can suspend this handler (make it inactive) and resume using this reference.

use cases:

 - download data from the database or API
 - save to disk
 - update inputs

#### observe() vs reactive()
These two have a very different purpose:

 - `observe()` and `observeEvent()` are for actions / side effects. They are eager: they execute whenever the events they are observing fire. This means that they can lead to performance issues.
 - `reactive()` is for calculating values, *without side effects*. It is lazy, it will not calculate when it is not needed. Eg, when an output is hidden.

### eventReactive()
Runs the given expression after the specified event fires. Returns a reactive function (similar to `reactive()`). Useful to isolate some input values (the ones in the expression), meaning nothing changes if you update them.

use cases:

 - "update" button which redraws plots

### reactiveValues()
Used to manage the state of the app. Creates a list of reactive values you can manipulate, *unlike input values which you cannot*.

### other

 - `invalidateLater()` (delay)
 - `validate()`
 - `req()` to require input inside `reactive()` when reactive needs to return a value for the downstream functions to work. It will stop the reactive from calculating if the parameter is NULL. Useful when you have empty inputs (so plots return errors) or are waiting for data to load.
 - `shinySignals()` is smt advanced

## app structure
Two options: the ui object and server function in "app.R" or separate scripts: "ui.R" and "server.R". This way RStudio or shinyapps will know how to run the application.

If you use app.R, everything that is outside of the server function will be run only once per R session, no matter how many people connect to it. Reactive functions will run multiple times, so don't put more computation in them than you have to.

For static content like images, save them in a folder named 'www'. This will act as root.

### scoping
read: https://shiny.rstudio.com/articles/scoping.html

### modules
Create functions for re-using functionality.

UI module:
```r
gapModuleUI <- function(id) {
  ns <- NS(id)  # creates a "namespace". It just prepends the id to a string
  tagList(  # returns HTML containing the stuff you put in as params
    plotOutput(ns("plot")),
    sliderInput(ns("year"), "Select Year", value = 1952, 
                min = 1952, max = 2007, step = 5,  
                animate = animationOptions(interval = 500))
  )
}
```

The server module has an input and output parameter, as the top-level Shiny function. You also need a session param and can have additional ones. You call these with `callModule()`. Shiny does some magic with the passed ID so that it matches the IDs in the UI modules. You do need to use the `NS()` function for that to work, I believe.

```r
shiny_module <- function(input, output, session, data) {...}

server <- function(input, output) {
  callModule(shiny_module, id="all", all_data)  # all_data is the data param
}
```

You can pass reactive values to server modules/functions by passing reactive expressions, ie output of `reactive()`. And you can return reactive values from them.

```r
shiny_module <- function(input, output, session, data) {
    # ...
    return(reactive({input$num}))
}

server <- function(input, output) {
  num = callModule(shiny_module, id="all", all_data)  # storing the reactive value
}
```

Check the module [video](https://www.rstudio.com/resources/webinars/understanding-shiny-modules/) and [article](https://shiny.rstudio.com/articles/modules.html).

## specific uses

### DataTables: interactive tables
A lot of options, I did not bother to list them all. Check links below.

https://yihui.shinyapps.io/DT-selection/

https://blog.rstudio.org/2015/06/24/dt-an-r-interface-to-the-datatables-library/

http://rstudio.github.io/DT/shiny.html

https://www.rstudio.com/resources/videos/interfacing-datatables/

### leaflet: interactive maps
R interface for leafletjs. [Link](https://rstudio.github.io/leaflet/)

### Shiny gadgets
Small Shiny apps that are intended for exploratory data analysis during your local R session. Typically you will use `stopApp()` to return a value from it. Cancel and done buttons added automatically.

You use the `miniPage()` UI from the "miniUI" library for a small UI with few margins.

You would typically package the whole code (UI and server) into a function that uses  `runGadget()` at the end to run the app. It will open in the Viewer if you're using RStudio. You can override the `viewer` parameter of `runGadget()` if you want to display it somewhere else (popup or browser).

Check [this webinar](https://www.rstudio.com/resources/webinars/interactive-graphics-with-shiny/) for the basics or [this article](https://shiny.rstudio.com/articles/gadgets.html).

use cases:

 - cleaning data
 - leverage analysis when modelling
 - UI for a database
 - filtering data
 - building plots

### dashboards
Important qualities of dashboards are that they a) update automatically or on a schedule and b) have a clean UI.

Updating dashboards

 - `shiny::reactiveFileReader(intervalMilis, filePath)` will read the file if it changes on the specified interval
 - `shiny::reactivePoll(intervalMilis, checkFunc, valueFunc)` will run the check function every N msecs and if the output changes run the value function that will refresh data

Special UI in the `shinydashboard` package ([link](https://rstudio.github.io/shinydashboard/)). Very useful since it has templates, icons, valueboxes, etc.

### bookmarkable state
Insert a `bookmarkButton()` inside the UI and enable bookmarking in the `shinyApp()` function or in the `global.R` script.

Two options: 

 - 'url': encodes the state in the URL, longer URLs
 - 'server': saves state on server, shorter URLs, requires Shiny Server

If there is a state variable other than the input variables that you want to save (eg, a counter), then you need to

 - create an `onBookmark()` handler that will save that state in a reactive variable
 - create an `onRestore()` handler that will restore the state
 - if needed, exclude vars with `setBookmarkExclude()`

Links: [basic](https://shiny.rstudio.com/articles/bookmarking-state.html), [advanced](https://shiny.rstudio.com/articles/bookmarking-state.html).

## other

### options
https://shiny.rstudio.com/reference/shiny/latest/shiny-options.html

### debugging
breaking into code

 - breakpoints (only server) or `browser()`
 - `options(shiny.error=browser)` runs the browser function on error

**tracing ("showing stuff")**

 - `options(shiny.trace=TRUE)` displays the data passed between the client and server
 - `options(shiny.reactlog=TRUE)` then `showReactLog()` afterwards. This is *not* real-time.
 - `runApp(display.mode='showcase')` is "debug mode"
 - printing with `cat(file=stderr())`. Use standard error output since Shiny will always print this.

### code profiling
Run the Shiny app inside the `profvis()` function which will give you a visualization of where processing time is spent.

# misc

## saveRDS
`saveRDS()` saves only one R object and you can unserialize it to another filename (that seems better than the `save()` function). [Link](http://www.fromthebottomoftheheap.net/2012/04/01/saving-and-loading-r-objects/).

```R
saveRDS(a, 'a.RDS')
b = readRDS('a.RDS')
```

## dir()
`dir()` is equal to `list.files()`