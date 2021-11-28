# materials
https://swizec.com/blog/livecoding-34-map-global-migrations-part-3/swizec/7455

react + d3
https://egghead.io/lessons/react-creating-a-d3-force-layout-in-react-c65fa37c
http://bl.ocks.org/sxywu/fcef0e6dac231ef2e54b

useful, both React and other new tech like ES6 and Redux
https://github.com/markerikson/react-redux-links

# the React library

## principles
sources:
 - https://www.youtube.com/watch?v=DgVS-zXgMTk

Write HTML, CSS and Javascript together. Components, not templates. Templates are tightly coupled with display logic (Javascript) anyway, so we might put them together. *I felt this myself, often going from JS to Handlebars.* Templates separate technologies, not concerns.

Re-renders entire app on every update. But does it on the virtual DOM. React components are *idempotent functions* (funcs with no additional effect if called more than once with the same params), just like a server rendered app. The mutable state is not in the render function, it is separated from rendering.

Does event delegation for free! Event delegation is when you have small objects that are part of a bigger one, but attach only one event listener to the bigger object and read `event.target` to conserve memory.

Since React `render` is a function it is easily testable.

React keeps state out of the DOM!

## simple React setup
https://www.codecademy.com/articles/react-setup-iv

## React library parts
What react subcomponents are there? This is probably incomplete, only what I know so far.

> React: Methods returned by require('react') don't deal with the DOM at all. They do not engage directly with anything that isn't part of React. They are for pure React purposes, such as creating components or writing JSX elements.

> ReactDOM: The methods returned by require('react-dom') are for interacting with the DOM. Example: ReactDOM.render

# JSX
> JSX is the HTML-like markup that React uses. The React compiler transforms any JSX into `React.createElement()` calls. Therefore, to use JSX you need to import the React library.

## multi-line JSX
> If a JSX expression takes up more than one line, then you should wrap the multi-line JSX expression in parentheses.

## one wrapper
> A JSX expression must have exactly one outermost element.

```jsx
// This code will work
var paragraphs = (
  <div id="i-am-the-outermost-element">
    <p>I am a paragraph.<p>
    <p>I, too, am a paragra-ph.</p>
  </div>
);

// But this code will not work
var paragraphs = (
  <p>I am a paragraph.</p> 
  <p>I, too, am a paragraph.</p>
);
```

## rendering
This is how you render JSX expressions
```javascript
var ReactDOM = require('react-dom');

// takes an expression that evals to JSX and an HTML node
ReactDOM.render(<h1>Hello world<h1>, document.getElementById('app'));
```

### virtual DOM definition
> For every DOM object, there is a corresponding virtual DOM object. This is a a lightweight copy that lacks power to directly change what's on the screen. The point of the vDOM is speed: manipulating the real DOM is slow, whereas the vDOM is much faster.

> When you render a JSX element, every single vDOM object gets updated, which happens very quickly. React then compares the vDOM with a vDOM snapshot that was taken right before the update. By comparing the new vDOM with a pre-update version, React figures out exactly which virtual DOM objects have changed. Once React knows which vDOM objects have changed, React updates those objects, and only those objects, on the real DOM.

## syntax

### classes
JSX cannot use `class` because it is a reserved word in Javascript.

```html
<h1 className="big">Hey</h1>
```

### closing tags
> In JSX you have to include a forward slash in self-closing tags like `<br />` and `<img />`

### javascript inside JSX
Use curly braces to insert a Javascript expression. This expression will have access to variables in the top-level environment. If you want to mix JS and text, put the text outside of the braces.

```jsx
var x = 2;
<div>Two plus two equals {x + x}</div>

// statements are not permissible, this is not valid <div>{if(x==2) 1;}</div>
```

### usage of short-circuit evaluation
Use the `&&` operator to display some HTML tags only if an expression is true.

```html
<ul>
  <li>Applesauce</li>
  { age > 15 && <li>Brussels Sprouts</li> }
  { age > 20 && <li>Oysters</li> }
  { age > 25 && <li>Grappa</li> }
</ul>
```

### usage of .map
A list of JSX elements will transform into the right HTML elements.

```javascript
var people = ['Rowe', 'Prevost', 'Gare'];
var peopleLIs = people.map(function(person, i) {
  return <li key={'person_' + i}>{person}<li>
});
var list = <ul>{peopleLIs}<ul>
```

### key attribute
Use the `key` attribute to keep track of the elements.

```html
<ul>
  <li key="li-01">Example1</li>
  <li key="li-02">Example2</li>
  <li key="li-03">Example3</li>
</ul>
```

### event listeners
In JSX, event listeners names are written in camelCase.

```html
<div onClick={handle_click} onMouseOver={handle_hover}</div>
```

#### event switches
If you have many handlers, it is common to put them in a switch statement for readability.

```javascript
handleEvent({type}) {
  switch(type) {
    case "click":
      return require("./actions/doStuff")(/* action dates */)
    case "mouseenter":
      return this.setState({ hovered: true })
    case "mouseleave":
      return this.setState({ hovered: false })
    default:
      return console.warn(`No case for event type "${type}"`)
  }
}
```

### styles
Styles are usually injected as object literals. They have a slightly different naming convention than CSS styles.

```javascript
var styles = {
  background: 'lightblue',
  color:      'darkred',
  marginTop:  100,
  fontSize:   50
};
var styleMe = <h1 style={styles}>Please style me!  I am so bland!<h1>;
```

#### an interesting solution
This is how a React component can have default, custom and state-dependent styles

```javascript
function m() {
  // returns Object that is a combination of all Object arguments
  var result = {};
  for (var i = 0; i < arguments.length; i++) {
    if (arguments[i]) {
      Object.assign(res, arguments[i])
    }
  }
  return result;
}

styles = {
  container: {
    borderRadius: 2, 
    border: '1px solid'
  }
}

propTypes: {
    depressed: React.PropTypes.bool,
    style    : React.PropTypes.object
}

<div style={m(
  styles.container,  // default style
  this.props.style,  // custom style
  // style when depressed, never overwritten with custom style
  this.props.depressed && styles.depressed
  )}><div>
```

#### style components
It is common to use functional stateless components for reusable style, and to wrap those in other components if you want to apply additional style.

```javascript
// main button of class 'btn'
const Btn = ({ className, primary, ...props }) =>
  <button
    type="button"
    className={classnames(
      "btn",
      primary && "btn-primary",
      className
    )}
    {...props}
 />

// a primary btn
const PrimaryBtn = props =>
  <Btn {...props} primary />

```

### boolean HTML attributes
> If you write disabled="true" or disabled="false" in raw HTML it won't work - in raw HTML, you need to remove the disabled attribute to enable the button. But React is not raw HTML - it does the following magic behind the scenes: if you do disabled={true} in JSX, it gets converted to just <button ... disabled> in HTML. If you do disabled={false} in JSX, the disabled attribute is removed from the button tag in HTML. This works with other boolean attributes like checked.

# components

## React components vs React elements
> A ReactElement is a light, stateless, immutable, virtual representation of a DOM Element. They are created by React.createElement('div') or more often using JSX which compiles HTML-like tags to ReactElements.

> ReactComponent is a stateful (dynamic) representation of a ReactElement that can be converted to it. Whenever the state changes, it is re-rendered.

> Whenever a ReactComponent is changing the state, we want to make as little changes to the “real” DOM as possible. So this is how React deals with it. The ReactComponent is converted to the ReactElement. Now the ReactElement can be inserted to the virtual DOM. vDOM sees if there is a difference btw the new View and the old one and updates the actual DOM accordingly.

## createClass()
Creates a component class which is a factory that creates components. Takes one argument, an object that must at a minimum include a render function that will usually return a JSX expression.

Always use UpperCamelCase for React classes or they will not be recognized in JSX code!

```javascript
var MyName = React.createClass({
  name: "marko",

  render: function () {
    return <h1>My name is {this.name}.<h1>;  // this refers to the component
  }
});
```

## rendering components
Render components in a similar way to ordinary JSX.

```javascript
ReactDOM.render(<MyName />, document.getElementById('container'));
```

## props
What are props? Props vs state?

> Every component has something called props that is similar to a function's parameters. props "come from above", when instantiating a component, although a component can have default props.

> State comes from inside the component: there is an initial state which is changed during the lifecycle of the component, unlike props which should stay the same. props should not change during a component's lifecycle. Components that have only props are "pure", ie they always give the same output.

### passing props to a component
You pass props by using HTML-like attributes. Also, you can use the ES6 spread operator.

```javascript
<MyComponent foo="bar" myInfo={["top", "secret", "lol"]}>

// props "spread" from object
var FancyDiv = function(props) {
  return <div {...props} className="fancy" /> // order matters: leter props overwrite
}

// a more complicated example: merging classnames
var FancyDiv = function(props) {
  <div
    className={["fancy", className].join(' ')}
    {...props}
  >
};
```

#### passing handlers
Component properties are often used to pass handlers. There is a naming convention you should follow with this.

```javascript
var Parent = React.createClass({
  handleClick: function () {  // handlers are prefixed with "handle"
    alert('yo');
  },
  render: function () {
    return <Button onClick={this.handleClick} >;  // passed as attributes prefixed with "on"
  }
});

var Button = React.createClass({
  render: function () {
    return <button onClick={this.props.onClick}>Click me!<button>;  // onClick here is a reserved name
  }
});
```

### children
`this.props.children` contains children that were passed to the React component. If a component has more than one child, this.props.children will return an array. If a component has only one child, then it will return the single child.

```javascript
<List type='Living Cat Musician'>
  <li>Nora the Piano Cat<li>
  <li>Lora the Piano Cat<li>
<List>

var List = React.createClass({
  render: function () {
    // this.props.children will return a list of the JSX elements defined above
    return <div><ul>{this.props.children}</ul></div> ;
  }
});
```

### accessing props inside a component
Props is accessible through `this.props`.

```javascript
var Greeting = React.createClass({
  render: function () {
    return <h1>Hi there, {this.props.firstName}<h1>;
  }
});
```

### automatic binding
> Automatic binding allows you to pass functions as props, and any this values in the functions' bodies will automatically refer to whatever they referred to when the function was defined. No binding to worry about!
> Also, handlers inside components have automatic binding of `this` to the object. But this does not work in ES6 classes!

### propTypes
propTypes are used for validating and documenting properties. It is an object defined on a React component.

These are deprecated and migrated to a different library. Need to check it out.

```javascript
React.createClass({ 
  propTypes: {
    title: React.PropTypes.string.isRequired,
    author: React.PropTypes.string.isRequired,
    weeksOnList: React.PropTypes.number.isRequired
  },
  render: function () {
    return ();
  }
});
```

## component functions
> basic functions
>   - `render`

> mounting lifecycle (in order)
>   - `getDefaultProps` should return the default props object
>   - `getInitialState` should return the initial state object, use the constructor if using ES6 classes
>   - `componentWillMount`: before it renders for the first time. `setState` will not cause re-render
>   - `render`
>   - `componentDidMount`: after it renders for the first time. Good place to connect a React app to external applications, such as web APIs or JavaScript frameworks or to start timers.

> unmounting
>   - `componentWillUnmount` gets called right before a component is removed from the DOM

> updating lifecycle
>   - `componentWillReceiveProps(nextProps)` if a component updates and will receive props
>   - `shouldComponentUpdate(nextProps, nextState)` has to return true or false and will control whether the component wil re-render
>   - `componentWillUpdate(nextProps, nextState)`. You cannot call `this.setState` here. The main purpose is to interact with things outside of the React architecture like checking the window size or interacting with an API.
>   - render
>   - `componentDidUpdate(prevProps, prevState)`, also used for interacting with things outside of React

> functions that are available to you
>   - `setState` takes an object (does not need to have all the state values!) and updates the state. You can't call this.setState from inside of the render function since changing the state automatically triggers the render function: it would create an infinite loop.

## state
> State attributes are available to you through `this.state.attr_name`. You can change them using the `this.setState` function. Everytime you change the state, the `render` function is automatically called. While state is there to be changed, props should never be changed from inside the component.

# React patterns

## use state instead of the DOM
React uses the component state instead of DOM attributes to make changes. Eg, you would not add/remove a class, but instead change the component state!

The state is an intermediary thing which sits in between the event handlers and `render()`:

 - event handlers don't need to worry about which part of the DOM changes. They just need to set the state.
 - `render` needs to worry about is what the current state is.

```javascript
// jQuery style uses CSS
$('#my-button').on('click', function() {$(this).toggleClass('active')});
/*CSS
#my-button.active {
    color: red
}
*/

// React style
var MyButton = React.createClass({
    // skipping the getInitialState function
    handleClick: function() {
        this.setState({color: this.state.color === 'blue' ? 'red' : 'blue'});
    },
    render: function() {
        return (<button
            color={this.state.color}
            onClick={this.handleClick}
            ><button>);
    }
});
```

## stateful containers and stateless children (presentational components)
> The basic idea: if a component has to have state, make calculations based on props, or manage any other complex logic, then that component shouldn't also have to render HTML-like JSX. This separates business logic from presentational logic.

> The idea is that some components are concerned with how things look (presentational) and others with how things work (containers). Usually the 1st are stateless and the 2nd stateful.

> The parent passes its state to the children as props, as well as any functions that the children will use in handlers to change the state of the parent. This state can then be passed to the child's siblings, or whatever is needed.

> A looser rule is that if you need to aggregate data from children, always take care of their state inside the parent. You "lift the state up"! Another word for it is state hoisting. This is done by passing handlers to stateless components so that they update their stateful parents.

### stateful containers
An example

```javascript
class CommentListContainer extends React.Component {
  constructor() {
    super()
    this.state = { comments: [] }
  }

  componentDidMount() {
    $.ajax({
      url: "/my-comments.json",
      dataType: 'json',
      success: comments =>
        this.setState({comments: comments});
    })
  }

  render() {
    return <CommentList comments={this.state.comments} >
  }
}
```

#### higher order components
Components that return other components. Useful when you want to wrap a stateless component so that it receives a state that is common to your app (eg fetching smt from the database).

This is a replacement for many mixins usage ([link](https://facebook.github.io/react/blog/2016/07/13/mixins-considered-harmful.html)).

```javascript
// stateless comp: displays name if available
const Greeting = ({ name }) => {
  if (!name) { return <div>Connecting...<div> }
  return <div>Hi {name}!<div>
}

// Connect will turn a component into stateful by adding name fetching
const Connect = ComposedComponent =>
  class extends React.Component {
    constructor() {
      super()
      this.state = { name: "" }
    }

    componentDidMount() {
      // this would fetch or connect to a store
      this.setState({ name: "Michael" })
    }

    render() {
      return (
        <ComposedComponent
          {...this.props}
          name={this.state.name}
        >
      )
    }
  }
```

### stateless functional components
To emphasize and simplify stateless components, you can define them as simple functions that output JSX elements. Such a component takes two optional inputs, props and state.

```javascript
function GuineaPigs (props) {
  var src = props.src;
  return (
    <div>
      <h1>Cute Guinea Pigs<h1>
      <img src={src} >
    <div>
  );
}
```

#### layout components
Layout components have children but are not their owner and should not interrupt their lifecycle.

```javascript
class HorizontalSplit extends React.Component {
  shouldComponentUpdate() {  // never updates, never will interrupt children
    return false
  }

  render() {
    <FlexContainer>
      <div>{this.props.leftSide}<div>
      <div>{this.props.rightSide}<div>
    <FlexContainer>
  }
}
```

#### propTypes for stateless functional components
To write propTypes for a stateless functional component, you define a propTypes object, as a property of the stateless functional component itself.

```javascript
function Example (props) {
  return <h1>{props.message}<h1>;
}

Example.propTypes = {
  message: React.PropTypes.string.isRequired
};
```

#### destructuring props
ES6 destructuring works great with functional components.

```javascript
// These examples are equivalent
const Greeting = props => <div>Hi {props.name}!<div>
const Greeting = ({ name }) => <div>Hi {name}!<div>

// The rest parameter syntax (...) allows you to collect all the remaining properties in a new object and you can use JSX Spread Attributes to forward props to the composed component
const Greeting = function({ name, ...props }) {
  return <div {...props}>Hi {name}!<div>
}
```

## controlled and uncontrolled components
> An uncontrolled component is a component that maintains its own internal state. A controlled component is a component that does not maintain any internal state. Since a controlled component has no state, it must be controlled by someone else. This distinction comes up with forms, where React prefers to make forms controlled.

### React way of doing forms
1. HTML forms are uncontrolled, they keep track of their state in the value attribute. But in React, devs prefer to strongly separate stateful and stateless components. To turn an input field into a controlled (stateless) component, give it a value attribute.
2. Whenever an form changes, the state should change. Not only when the user clicks 'submit'.

```javascript
var Input = React.createClass({
  getInitialState: function() {
    return { userInput: '' }
  },
  handleUserInput: function(e) {
    this.setState({
      userInput: e.target.value
    });
  },
  render: function () {
    return (
      <div>
        <input
          value={this.state.userInput}       // turn input into a controlled component
          type="text"
          onChange={this.handleUserInput} >  // you handle each change
        <h1>{this.state.userInput}<h1>
      <div>
    );
  }
});
```

## immutable state
There are generaly two ways of changing the state: directly, or first making a copy then making changes on that. The second option (immutable state) is preferrable: 

 - keep a reference to the old state, useful for undoing
 - easier to compare old vs new. If old != new, there is a change
 - determining when to re-render in React and using `shouldComponentUpdate`

```javascript
// ...
class Example extends React.Component {
    handleClick() {
        // copying the state property
        const squares = this.state.squares.slice();
        squares[i] = 'X';
        this.setState({squares: squares});  // passing the new object
    }
    render() {}
};
```

## principles
> - If a component does not own a datum, then that datum should not influence it’s state
> - Store the simplest possible values to describe a component’s state, eg `isActive` and calculate anything inside the render function, eg `var class = isActive ? 'active' : ''`
> - Whenever possible, make decisions and do calculations at the last possible moment: in the render function. This is slightly slower, but ensures the least amount of redirection in the component. Eg: concatenate the firstName and lastName prop, decide on which classes to use, decide whether to use placeholder text... If there is a lot of code, extract helper functions, possibly prefixing them wih "render". If the code is computationally expensive, use a memoization function.

## using refs to modify a child
To modify a child typically, you re-render it with new props. However, there are a few cases where you need to imperatively modify a child outside of the typical dataflow. The child to be modified could be an instance of a React component, or it could be a DOM element.

The ref attribute takes a callback function, and the callback will be executed immediately after the component is mounted or unmounted. When the ref attribute is used on an HTML element, the ref callback receives the underlying DOM element as its argument. When the ref attribute is used on a custom component declared as a class, the ref callback receives the mounted instance of the component as its argument.

Avoid using refs for anything that can be done declaratively. Some use-cases:

 - managing focus, text selection, or media playback.
 - triggering imperative animations.
 - integrating with third-party DOM libraries.

https://facebook.github.io/react/docs/refs-and-the-dom.html

```javascript
class CustomTextInput extends React.Component {
  focus = () => {
    // Explicitly focus the text input using the raw DOM API
    this.textInput.focus();
  }

  render() {
    // Use the `ref` callback to store a reference to the text input DOM
    // element in an instance field (for example, this.textInput).
    return (
      <div>
        <input type="text"
          ref={(input) => { this.textInput = input; }} />
        // when clicking button, focus will move to text input
        <input type="button" value="Focus the text input" onClick={this.focus} />
      </div>
    );
  }
}
```

## strategies for component communication
These are non Flux or Redux

> PARENT TO CHILD:
> - props (most common)
> - through refs attribue: parent runs child instance methods

> CHILD TO PARENT:
> - callbacks passed through props, most common
> - event bubbling, non-specific to React, catches events on any child
 
> SIBLING TO SIBLING:
> - through any of the above methods, usually callbacks

> ANY TO ANY:
> - observer pattern
> - global variables (or not)
> - context, which is passed through all children recursively (need to check this, but it's uncommon)

# React + D3

## option 1: React provides the container
The way most people use D3 with React is to use React to build the structure of the application, and to render traditional HTML elements, and then when it comes to the data visualization section, they pass a DOM container (typically an <svg>) over to D3 and use D3 to create and destroy and update elements

The benefit of this method of integrating React and D3 is that you can use all the same kind of code you see in all the core D3 examples. The main difficulty is that you need to create functions in various React lifecycle events to make sure your viz updates.

```javascript
import React, { Component } from 'react' // ...
import './App.css'
import { scaleLinear } from 'd3-scale’
import { max } from 'd3-array'
import { select } from 'd3-selection'

class BarChart extends Component {
   constructor(props){
      super(props)
      this.createBarChart = this.createBarChart.bind(this)
   }

   // after mount or receiving new props/state, redraw
   componentDidMount() {
      this.createBarChart()
   }
   componentDidUpdate() {
      this.createBarChart()
   }

   // the D3 part
   createBarChart() {
      const node = this.node  // a reference to the SVG node
      const dataMax = max(this.props.data)
      const yScale = scaleLinear()
         .domain([0, dataMax])
         .range([0, this.props.size[1]])
      select(node)
         .selectAll('rect')
         .data(this.props.data)
         .enter()
         .append('rect')
   
      select(node)
         .selectAll('rect')
         .data(this.props.data)
         .exit()
         .remove()
   
      select(node)
         .selectAll('rect')
         .data(this.props.data)
         .style('fill', '#fe9922')
         .attr('x', (d,i) => i * 25)
         .attr('y', d => this.props.size[1] — yScale(d))
         .attr('height', d => yScale(d))
         .attr('width', 25)
   }

   // D3 will draw after render (...DidMount), which just places an SVG node        
   render() {
         return <svg ref={node => this.node = node}  // store ref to SVG node
         width={500} height={500}>
         <svg>
      }
   }
export default BarChart
```

## option 2: React for element creation, D3 as the visualization kernel
Not enough info on [this link](https://medium.com/@Elijah_Meeks/interactive-applications-with-react-d3-f76f7b3ebc71).

## option 3: using D3 sub-libraries that do not change the DOM
E.g. use scales to calculate the positions of the elements and React to render. On changing data (props), the component will update.

Only issue is when you need to use a DOM changing D3 function, then you use refs.

[link](https://hackernoon.com/how-and-why-to-use-d3-with-react-d239eb1ea274)

