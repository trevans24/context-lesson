
# ![](https://ga-dash.s3.amazonaws.com/production/assets/logo-9f88ae6c9c3871690e33280fcf557f33.png) React Context API

### Learning Objectives
*After this lesson, you will be able to:*
- Define Context and distinguish a `Context.Provider` from a `Context.Consumer`
- Use the React Context API to pass data and functions across the Component Tree
- Identify appropriate use cases for React Context

## What is Context in React

From the [React Docs](https://reactjs.org/docs/context.html):
>Context provides a way to pass data through the component tree without having to pass props down manually at every level.

In short, the Context API gives as a method to work around prop-drilling in certain situations.

We have learned that React has a uni-directonal flow of data, only passing data or functions from parent to child via props. Using `Context` allows us to use data or functions that we would consider _global_ within our Component tree without passing them through each level of a branch.

A `Context` is composed of three parts: the definition, the Provider and the Consumer. Aptly named, the Provider will provide access to the data that we want to make available, and the Consumer will be used where we want to use the data provided.

You may have already been using Context whether you knew it or not. React Router is an example of the Context API in use! Behind the scenes, React Router has created a Context that includes `location`, `history`, `match` and helper functions to deal with navigation. It grants access via the`<BrowserRouter />` and `<Link />` components.
```js
/* App.js */
<BrowserRouter> // Context Provider
  <Router exact path='/' component={Home} />
  <Router ... />   
  <Router ... />  
</BrowserRouter>


/* NavComponent.jsx */
<Link to='/'>Home</Link> // Context Consumers
<Link to='/issues'>Issues</Link>
<Link to='/pullrequests'>Pull Requests</Link>
```

## Why Context was developed
Before React 15, developers could only use prop drilling to manage their application’s state. Then between 15 and 16.3 developers started reaching for other toolboxes, like [Redux](https://redux.js.org/) to help with more complicated state management and for use cases with a globabl state object. Redux and other toolboxes offered alternative solutions to managing state, but they introduced complexity, increased the bundle size, and required learning another library in addition to React.

Like all abstractions, these tools came burdened with tradeoffs that felt worth it for large applications, but seemed like overkill for small and mid-sized applications. In short, you spent about an extra hour or so to set up an application just to use tools like Redux.

### Why should I use Context and not Redux?
To be fair if you already know how to use Redux and enjoy what it brings to the table then all means go ahead and keep using it. However, if you're looking to decrease your bundle sizes, improve performance, and get into developing right away then Context is the right solution for you.

![cool kid](https://media.giphy.com/media/xT1R9YYeCajBym9Gfu/giphy.gif)

## Appropriate Use Cases
- Authenticated User Data
- Theming
- Localization data

>Context is primarily used when some data needs to be accessible by many components at different nesting levels. Note from React: Apply it sparingly because it makes component reuse more difficult.

___
## Using Context

### Create a Context
Touch a new file in `src/Context` to house your Context code. You can define and export data and function stubs to be used by Context. When creating your context make sure that the shape of your Context data matches what you will use be using in your Context Providers and Consumers.

```js
// ThemeContext.js
import React, { Component, createContext } from 'react';

// Create a context
// Note here that the name of the context needs to match when you want to access it
export const ThemeContext = createContext();

...
```

### Context Consumers
Within the child component where data should be displayed, import the Context and wrap the portion of the component with a `Context.Consumer`. Your context data will be accessible through an anonymous function.
```js
// This is the Context consumer of the values (state)
export const ThemeConsumer = ThemeContext.Consumer;
```

### Context Providers
```js
// Create a provider (wrapper that handles the logic)
// This also allows communication to context and update
// state values throughout app
export default ThemeProvider extends Component {
	state = {
		colors: ['black', 'blue', 'green', 'rebeccapurple', 'red', 'whitesmoke'],
		color: ''
	}

	changeColor = (color) => {
		this.setState({
			color
		})
	}

	render() {
		return (
			// When creating the Provider you must return a value
			// this allows you to access the Context State
			<ThemeContext.Provider value={{ theme: this.state, changeColor: this.changeColor }} >
				// When you wrap your component (App)
				// this way the children can access the state
				{this.props.children}
			</ThemeContext.Provider>
		)
	}
}
```

### Putting it all together
```js
import React, { Component, createContext } from 'react';

export const ThemeContext = createContext();

export default ThemeProvider extends Component {
	state = {
		colors: ['black', 'blue', 'green', 'rebeccapurple', 'red', 'whitesmoke'],
		color: ''
	}

	changeColor = (color) => {
		this.setState({
			color
		})
	}

	render() {
		return (
			<ThemeContext.Provider value={{ theme: this.state, changeColor: this.changeColor }} >
				{this.props.children}
			</ThemeContext.Provider>
		)
	}
}
```

### Getting the Context state to the rest of the components
```js
// index.js
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';
// Import your Context to wrap the application
import { ThemeProvider } from './context/ThemeContext';

ReactDOM.render(
	<ThemeProvider>
      <App />
    </ThemeProvider>
  , document.getElementById('root'));

```
**Why do you put it in the index.js and wrap the App and not just around your components you need to provide it to?**
The code then becomes messy and difficult to read.
```js
//SomeComponent.js
import { ThemeProvider, ThemeConsumer } from './context/ThemeContext';

...

return (
	// First you need the provider to supply the state
	<ThemeProvider>
	// Then you need the consumer to pass the state to the component needing the state
    <ThemeConsumer>
      {({ theme }) => (
        <ComponentUsingContext theme={theme} />
      )}
    </ThemeConsumer>
	</ThemeProvider>
)
```
Imagine doing that in a number of other components in a large application..
![Crazy Person](https://media.giphy.com/media/11ahZZugJHrdLO/giphy.gif)

### Supplying the Context State
You need to wrap the component needing the state to pass as props
```js
//SomeComponent.js
import { ThemeConsumer } from './context/ThemeContext';

...

return (
	<ThemeConsumer>
    {(context) => (
      <ColorPane theme={context.theme} changeColor={context.changeColor} />
    )}
  </ThemeConsumer>
)


... // OR if you want use object destructuring for easier access
return (
	<ThemeConsumer>
    {({ theme: { color, colors }, changeColor }) => (
      <ColorPane color={color} colors={colors} changeColor={changeColor} />
    )}
  </ThemeConsumer>
)
```

### Accessing Context in Components
Passing the context state into the component using it now the state and functions are accessable in props
```js
import React, { Component } from 'react';

export default class ColorPane extends Component {
  render() {
    let style = {
      background: this.props.theme.color, // We can access data in the this.context object
      height: "200px",
      width: "200px"
    }

  	//vs object destructuring
  	let style = {
      background: this.props.color, // We can access data in the this.context object
      height: "200px",
      width: "200px"
    }

  	// Using the Context function
  	setColor = () => {
  		// randomize the colors
  		let newColorIndex = Math.ceil(Math.random()*5);
  		this.props.changeColor(this.props.colors[newColorIndex])
  	}

    return (
      <div style={style}>
	      <button onClick={setColor} >Change Color</button>
      </div>
    )
  }
}
```

### Accessing Multiple Contexts
You can use data from multiple Contexts in the following manner
```js
...
<ThemeContext.Consumer>
  { ({color}) => (
    <UserContext.Consumer>
    { ({user}) => (
      <MyComponent user={user} themeColor={color} />
    )}
    </UserContext.Consumer>
  )}
</ThemeContext.Consumer>

...

```

## Enter Hooks

The [new Hooks API*](https://reactjs.org/docs/hooks-reference.html#usecontext) has provided a `useContext` hook to allow access to your Contexts within function components. So what does that change?

Now we don't need to wrap any component using the Context with a consumer.
```js
//SomeComponent.js
import { ThemeConsumer } from './context/ThemeContext';

...
return (
	<ThemeConsumer>
	{({ theme: { color, colors }, changeColor }) => (
    <ColorPane color={color} colors={colors} changeColor={changeColor} />
  )}
  </ThemeConsumer>
)
```
This makes our components become a lot cleaner and easier to read
```js
//SomeComponent.js
...
return (
	<ColorPane />
)
```
Now our component using the Context handles it all
```js
//ColorPane.js
import React, { useContext } from 'react';
import { ThemeContext } from '../Path/To/ThemeContext';

export function ColorPane() {
  const { theme: { color, colors }, changeColor } = useContext(ThemeContext);

	//vs object destructuring
	let style = {
    background: color, // We can access data in the this.context object
    height: "200px",
    width: "200px"
  }

	// Using the Context function
	function setColor() {
		// randomize the colors
		let newColorIndex = Math.ceil(Math.random()*5);
		changeColor(colors[newColorIndex])
	}

  return (
    <div style={style}>
      <button onClick={setColor} >Change Color</button>
    </div>
  )

}
```

___
## Alternatives to Context (if your a crazy person)
- [Component Composition Patterns](https://reactjs.org/docs/composition-vs-inheritance.html)

Containment
```js
// ParentContainer.jsx
export const ParentContainer = props => {
  return(
    <div>
      {props.children}
    </div>
  )
}


// App.jsx
...
render() {
  return(
    <ParentContainer some={someProps}>
        <ChildComponent different={Props} />
        <AnotherChild someOther={Props} />
    </ParentContainer>`
  )
}
...
 ```
Passed
```js
//Parent.jsx
import React from 'react';
import { ChildOne } from './ChildOne'

export function Parent(props) {
  return (
    <ChildOne {...props} />
  )
}

// ChildOne.jsx
import React from 'react';

export function ChildOne(props) {
  return (
    <div>
      {props.usedDownTheTree}
    </div>
    )
}

// UsedDownTheTree.jsx
import React from 'react';

export function UsedDownTheTree(props) {
  return(
    <>
      <h1>Hello, {props.name ? props.name : 'World!'}</h1>
      <h3>{props.message ? props.message : '' }</h3>
    </>
  )
}

//App.js
import React, { Component } from 'react';
import {Parent} from './Parent'
import {UsedDownTheTree} from './UsedDownTheTree';
import './App.css';

class App extends Component {
  constructor(props) {
    super(props)
    this.state = {
      user: 'GLaDOS',
      message: "I don't like lemonade"
    }
  }


  render() {
    let usedDownTheTree = <UsedDownTheTree name={this.state.user} message={this.state.message}/>
    return (
      <div className="App">
          <Parent usedDownTheTree={usedDownTheTree}/>
      </div>
    );
  }
}

export default App;


```
