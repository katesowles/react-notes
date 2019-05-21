# React Notes
#LearnReact

## Conventions
### Code
- Don’t mix expressions and strings (curly brackets and quotes) in a JSX attribute — use one or the other, but never both
- When breaking the markup onto different lines for readability, contain with parenthesis to avoid issues with semicolons
- `camelCase` property names
- Elements without children end with `/>` as with XML
- Always start component names with a capital letter (they represent classes and are declared with functions), lower case names are reserved for DOM tags

### Verbiage
- “Element” refers to individual React elements/objects, “components” are made up of multiple elements.
---
## Good to Know
### JSX
- Compiled by Babel
- JSX prevents injection attacks by converting everything to a string before being rendered
- JSX is compiled down to `React.createElement()` calls, which produces an object, which is what was available pre-JSX
- Two ways in JSX to declare the same React element:
```
const element = (
  <h1 className="greeting">
    Hello World!
  </h1>
);
```
```
// NOTE: no one really uses this...we'll see why later...

const element = React.createElement(
  'h1',                     // type
  {className: 'greeting'},  // props, or array of objects
  'Hello World!'            // children / tag contents
                            // children also an array
);
```

### Rendering Elements
- Rendering an element into the root DOM node:
```
const element = <h1>Hello World!</h1>;
ReactDOM.render(element, document.getElementById('root);
```
- React elements are immutable, they cannot be updated, only replaced:
```
function tick () {
  const element = (
    <div>
      <p>It is {new Date().toLocalTimeString()}.</p>
    </div>
  );
}

ReactDOM.render(
  element,
  document.getElementById('root')
);

setInterval(tick, 1000);	// re-renders 'tick' function every second
```
- React only updates what is strictly necessary. Does only one tag update? Then that’s the only tag to re-render. Does only the child content of an element change? Then only the contents of the element (not the structure of the element itself) is updated

### Components and Props
- Two ways to write a React component:
```
// using a function definition:

function Welcome (props) {
  return <h1>Hello, {props.name}</h1>
}
```
```
// using an ES6 class definition:
// do note that class definitions have additional features
// was prominent about a year ago when you needed to pass state -- no longer the case as of v16.5 when Hooks were introduced

class Welcome extends React.Component {
  render () {
    return <h1>Hello, {this.props.name}</h1>
  }
};
```
- React components rendered the same way as React elements:
```
function Welcome (props) {
ret urn <h1>Hello, {props.name}</h1>;
}

const element = <Welcome name="Kate" />

ReactDOM.render(
  element,
  document.getElementById('root')
);
```
- We can use the same component multiple times with different props:
```
...

<Welcome name="Alyssa" />
<Welcome name="Kate" />
<Welcome name="Tory" />
<Welcome name="Will" />

...
```
- Props are read-only, no matter how you declare a component, it must never modify it’s own props (aka it must be a “pure” function)

### State and Lifecycle
- State is not available in component function definitions (without Hooks).
- We can make the Clock component example a reusable, self-contained, stateful component:
```
class Clock extends React.Component {
  constructor (props) {
    super(props);

    this.state = {
      date: new Date();
    }
  }

  componentDidMount () {
    this.timerID = setInterval( () => this.tick(), 1000 );
  }

  componentWillUnmount () {
    clearInterval(this.timerId);
  }

  tick () {
    this.setState((state, props) => {
      counter: state.counter + props.increment
    });
  }

  render() {
    return (
      <div>
        <p>It is {this.state.date.toLocaleTimeString()}.</p>
      </div>
    );
  }
}

ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```
- Remember to never update `this.state._____` directly, only use `this.setState(...)`
- Remember that the only place `this.state` can be set is in the constructor
	
### Handling Events
- React events are named using `camelCase`
- Pass a function as the event handler, rather than a string (ex. `onClick={activateLasers}` as opposed to `onCLick="activateLasers()"`)
-  Don’t forget to `e.preventDefault()` on event handlers that are not using the default click action!
- It is necessary to bind the callback function to `this` otherwise it will be `undefined` when the function is called, this is a JS feature, not React/JSX…Generally, if you refer to a method without `()` after it, such as with our click handler below, you should bind that method:
```
class Toggle extends React.component {
  constructor (props) {
    super(props);
    this.state = { isToggleOn: true };

    // binding here is necessary!
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick () {
    this.setState(state => ({
      isToggleOn: !state.isToggleOn
    }));
  }

  render () {
    return (
      <button onClick={this.handleClick}>
        { this.state.isToggleOn ? 'ON' : 'OFF' }
      </button>
    );
  }
}

ReactDOM.render(
  <Toggle />,
  document.getElementById('root')
);
```
- Alternatives to having to manually bind with the format above:
```
// public class fields syntax (experimental)

class Toggle extends React.Component {
  handleClick = () => { ... };

	...
}
```
```
// arrow function in the callback

class Toggle extends React.Component {
  handleClick () { ... }

  render () {
    return (
      <button onClick={(e) => this.handleClick(e)}>
        { this.state.isToggleOn ? 'ON' : 'OFF' }
      </button>
    );
  };
}
```
- It’s common to want to pass an extra parameter to an event handler, either of the following would work:
```
<button onClick={(e) => this.deleteRow(id, e)}>Delete Row</button>
```
```
<button onClick={this.deleteRow.bind(this, id)}>Delete Row</button>
```

### Conditional Rendering
-We’ll create a `Greeting` component that displays one of two components, depending on whether or not a user is logged in:
```
function UserGreeting (props) {
  return <h1>Welcome back!</h1>
}

function GuestGreeting (props) {
  return <h1>Please sign up.</h1>
}

function Greeting(props) {
  const isLoggedIn = props.isLoggedIn;

  if (isLoggedIn) {
    return <UserGreeting />;
  }

  return <GuestGreeting />;
}

ReactDOM.render(
  // try changing the below to `true`
  <Greeting isLoggedIn={false} />,
  document.getElementById('root')
);
```
- Can use variables to store elements, which can be helpful when conditionally rendering part of a component:
```
function LoginButton (props) {
  return (
    <button onClick={props.onClick}>    
      Login
    </button>
  );
}

function LogoutButton (props) {
  return (
    <button onClick={props.onClick}>    
      Logout
    </button>
  );
}

class LoginControl **extends** React.Component {
  constructor (props) {
    super(props);

    this.handleLoginClick = this.handleLoginClick.bind(this);
    this.handleLogoutlick = this.handleLogoutlick.bind(this);

    this.state = { isLoggedIn: false };
  }

  handleLoginClick () {
    this.setState({ isLoggedIn: true });
  }

  handleLogoutClick () {
    this.setState({ isLoggedIn: false });
  }

  render () {
    const isLoggedIn = this.state.isLoggedIn;
    let button;

    if (isLoggedIn) {
      button = <LogoutButton onClick={ this.handleLogoutClick} />;
    } else {
      button = <LoginButton onClick={ this.handleLoginClick } />;
    }

    return (
      <div>
        <Greeting isLoggedIn={ isLoggedIn } />
        { button }
      </div>
    );
  }
}

ReactDOM.render(
  <LoginControl />,
  document.getElementById('root')
);
```
- Inline expressions: Inline `If` with Logical `&&` Operator; these work because `true && expression` always evaluates to `expression`, and conversely `false && expression` always evaluates to `false`
```
function Mailbox (props) {
  const unreadMessages = props.unreadMessages;

  return (
    <div>
      <h1>Hello!</h1>
      {unreadMessages.length > 0 &&
        <h2>
          You have { unreadMessages.length } unread messages;
        </h2>
      }
    </div>
  );
}

const messages = [ 'One', 'Two', 'Three' ];

ReactDOM.render(
  <Mailbox unreadMessages={messages} />,
  document.getElementById('root');
);
```
- Inline `If-Else` with Ternary Operator:
```
render () {
  const isLoggedIn = this.state.isLoggedIn;

  return (
    <div>
      This user is <b>{ isLoggedIn ? 'currently' : 'not' }</b> logged in.
    </div>
  );
}
```
- Could also be used for larger expressions, although it’s less obvious what’s going on:
```
render () {
  const isLoggedIn = this.state.isLoggedIn;

  return (
    <div>
      { isLoggedIn ? (
        <LogoutButton onClick={this.handleLogoutClick} />
      ) : (
        <LoginButton onClick={this.handleLoginClick} />
      )}
    </div>
  );
}
```
- Can also be used to prevent a component from rendering:
```
function WarningBanner (props) {
  if (!props.warn) {
    // will stop render of component
    return null;
  }

  return (
    <div className="warning">
      Warning!
    </div>
  );
}
```

### Lists and Keys
- We can build collections  of elements:
```
const numbers = [ 1, 2, 3, 4, 5 ];
const listItems = numbers.map(number => {
  <li>{ number }</li>
};

...

ReactDOM.render(
  <ul>{listItems}</ul>,
  document.getElementByid('root')
);
```
- Or, alternatively, if you want to render a list inside a component:
```
function ListItem (props) {
  return <li>{props.value}</li>;
};

function NumberList (props) {
  const numbers = props.numbers;
  const listItems = numbers.map(numbers => {
    <ListItem key={number.toString()}
    value={number} />
  };

  return (
    <ul>{listItems}</ul>
  );
}

const numbers = [ 1, 2, 3, 4, 5 ];

ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
);
```
- Keys allow React to detect which elements/components have changed, have been added, or have been removed
- Most commonly, a data ID will be used as a key. An index can be used if no data was available, though do note that using indexes for keys if the order of items may change can negatively impact performance
- Keys must be unique among siblings, but don’t have to be unique on a global scale
- Keys serve as a hint to React, but don’t get passed to your component. If your component needs the same value, pass it as a prop with a different name
- Map can also be inlined into the JSX:
```
function NumberList (props) {
  const numbers = props.numbers;

  return (
    <ul>
      {numbers.map(number => 
        <ListItem key={number.toString()}
                  value={number}>
      )}
    </ul>
  );
}
```
### Forms
- Since form elements naturally hold some internal state, they're a little different form other DOM elements in React
- `<input>`, `<textarea>`, and `<select>` typically maintain their own state, in React state must be kept mutable, so these form fields are updated with `setState()`
#### Input Tag
- An input whose value is controlled by React (with React being the single source of truth) is called a "controlled component"
- An example of React managing state for a text input form element:
```
class NameForm extends React.Component {
  constructor (props) {
    super(props);
    this.state = { value: '' };

    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleChange (event) {
    this.setState({ value: event.target.value });
  }

  handleSubmit (event) {
    alert('A name was submitted: ' + this.state.value);
    event.preventDefault();
  }

  render () {
    return (
      <form onSubmit={this.handleSubmit}> 
        <label>
          Name:
          <input type="text" value={this.state.value} onChange={this.handleChange} />
        </label>

        <input type="submit" value="Submit" />
      </form>
    );
  }
}
```
- Setting the input's value on our form element ensures that it will always reflect the value of `this.state.value`, making React state the single source of truth
- Since `handleChange` runs on every keystroke, the React state is also updated as the user types
#### Text Area Tag
- The `<textarea>` element's value is definied by it's children:
```
<textarea>
  This is the child of the textarea.
</textarea>
```
- In React, the `<textarea>` will behave much like the `<input>` and receive it's value from the `value` attribute instead of targeting it's child:
```
class EssayForm extends React.Component {
  constructor (props) {
    super(props);

    this.state = {
      value: 'This is the value of the textarea.'
    }

    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleChange (event) {
    this.setState({ value: event.target.value });
  }

  handleSubmit (event) {
    alert('A name was submitted: ' + this.state.value);
    event.preventDefault();
  }

  render () {
    return (
      <form onSubmit={this.handleSubmit}> 
        <label>
          Essay:
          <textarea value={this.state.value} onChange={this.handleChange} />
        </label>

        <input type="submit" value="Submit" />
      </form>
    );
  }
}
```
#### Select Tag
- Instead of setting `selected` on the `option` within the `select`, React sets the `selected` value by setting a value attribute on the root `select` tag
```
class FlavorForm extends React.Component {
  constructor (props) {
    super(props);
    this.state = { value: 'coconut' };

    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleChange (event) {
    this.setState({ value: event.target.value });
  }

  handleSubmit (event) {
    alert('Your favorite flavor is: ' + this.state.value);
    event.preventDefault();
  }

  render () {
    return (
      <form onSubmit={this.handleSubmit}> 
        <label>
          Pick your favorite flavor:
          <select value={this.state.value} onChange={this.handleChange}>
            <option value="apple">Apple</option>
            <option value="banana">Banana</option>
            <option value="cherry">Cherry</option>
          </select>
        </label>

        <input type="submit" value="Submit" />
      </form>    )
  }
}
```
- You can pass an array into the `value` attribute in order to select multiple options in a select tag
#### File Input Tag
-Because the value of the file input tag is read-only, it is an uncontrolled component in React
#### Handling Multiple Inputs
- When you need to handle multiple controlled `input` elements, you can add a `name` attribute to each element and let the handler function choose what to do based on the value of `event.target.name`:
```
class Reservation extends React.Component {
  constructor (props) {
    super(props);

    this.state = {
      isGoing: true,
      numberOfGuests: 2
    };

  this.handleInputChange = this.handleInputChange.bind(this);
  }

  handleInputChange (event) {
    const target = event.target;
    const value = target.type === 'checkbox' ? target.checked : target.value;
    const name = target.name;

    this.setState({
      [name]: value
    });
  }


  render () {
    return (
      <form>
        <label>
          Is going:
          <input name="isGoing"
                type="checkbox"
                checked={this.state.isGoing}
                onChange={this.handleInputChange} />
          </input>
        </label>

        <label>
          Number of guests:
          <input name="numberOfGuests"
                type="number"
                value={this.state.numberOfGuests}
                onChange={this.handleInputChange} />
        </label>
      </form>
    );
  }
}
```
#### Handling Null Values
- Keep in mind that specifying the value prop on a controlled component prevents the user from changing the input unless you want them to. If you've specified a value but the input is still editable, you may have accidentally set `value` to `undefined` or `null`.
#### Alternatives to Controlled Components
- In some cases, using controlled components can be tedious. An alternative is to use Uncontrolled Components: https://reactjs.org/docs/uncontrolled-components.html
#### Fully-Fledged Solutions
- `Formik` is one of the most popular solutions, but it is built upon the same principles of controlled components and managed state, so make sure you understand the basics.
### Lifting Up State
- It's common for several components to reflect the same changing data, in which case we'll "lift" the shared state up to their closest common ancestor:
```
const scaleNames = {
  c: "Celsius",
  f: "Fahrenheit"
};

function toCelsius (fahrenheit) {
  return (fahrenheit - 32) * 5 / 9;
}

function toFahrenheit (celsius) {
  return (celsius * 9 / 5) + 32;
}

function tryConvert(temperature, conver) {
  const input = parseFloat(temperature);

  if (Number.isNaN(input)) {
    return '';
  }

  const output = convert(input);
  const rounded = Math.round(output * 1000) / 1000;

  return rounded.toString();
}

function BoilingVerdict (props) {
  if (props.celsius >= 100) {
    return <p>Water would boil.</p>
  }

  return <p>Water would not boil.</p>
}

class TemperatureInput extends React.Component {
  constructor (props) {
    super(props);

    this.handleChange = this.handleChange.bind(this);
  }

  handleChange (event) {
    this.props.onTemperatureChange(event.target.value);
  }

  render () {
    const temperature = this.props.temperature;
    const scale = this.props.scale;

    return (
      <fieldset>
        <legend>Enter Temp in {scaleNames[scale]}:</legend>

        <input value={temperature}
              onChange={this.handleChange} />
      </fieldset>
    );
  }
}

class Calculator extends React.Component {
  constructor (props) {
    super(props);

    this.state = {
      temperature: '',
      scale: 'c'
    };


    this.handleCelsiusChange = this.handleCelsiusChange.bind(this);
    this.handleFahrenheitChange = this.handleFahrenheitChange.bind(this);
  }

  handleCelsiusChange (temperature) {
    this.setState({ temperature, scale: 'c' });
  }

  handleFahrenheitChange (temperature) {
    this.setState({ temperature, scale: 'f' });
  }

  render () {
    const scale = this.state.scale;
    const temperature = this.state.temperature;
    const celsius = scale === 'f' ? tryConvert(temperature, toCelsius) : temperature;
    const fahrenheit = scale === 'c' ? tryConvert(temperature, toFahrenheit) : temperature;

    return (
      <div>
        <TemperatureInput scale="c"
                          temperature={celsius}
                          onTemperatureChange={this.handleCelsiusChange} />

        <TemperatureInput scale="f"
                          temperature={fahrenheit}
                          onTemperatureChange={this.handleFahrenheitChange} />

        <BoilingVerdict celsius={parseFloat(celsius)} />
      </div>
    );
  }
}
```
- Instead of trying to sync the state between different components, you should rely on the top-down data flow
- While lifting state does involve more "boilerplate" code than two-way binding, it takes less work to find and isolate bugs.
### Composition vs Inheritance
- The composition model is recommended over iheritance in order to increase reusability of code between components
#### Containment
- Some components don't know their children ahead of time. This is especially common for components like `Sidebar` or `Dialog` that represent generic "boxes", but we can use the `children` prop to pass the child elements. This allows other components to pass arbitrary children to them by nesting the JSX. Anything inside the `<FancyBorder>` tags get passed into the FancyBorder component as a `children` prop:
```
function FancyBorder (props) {
  return (
    <div className={'FancyBorder FancyBorder-' + props.color}>
      {props.children}
    </div>
  );
}

function WelcomeDialog (props) {
  return (
    <FancyBorder color="blue>
      <h1 className="Dialog-title"
        Welcome
      </h1>
      <p className="Dialog-message>
        Thank you for visiting!
      </p>
    </FancyBorder>
  );
}
```
- You might sometimes need multiple "holes" in a component, in such cases you can come up with your own convention instead of `children`. This is less common, but entirely possible:
```
function SplitPane (props) {
  return (
    <div className="SplitPane">
      <div className="SplitPane-left>
        {props.left}
      </div>

      <div className="SplitPane-right>
        {props.right}
      </div>
    </div>
  );
}

function App () {
  return (
    <SplitPane left={<Contacts />}>
    <SplitPane right={<Chat />}>
  );
}
```
#### Specialization
- There are cases where we think about components as being "special cases" of another, more generic component. For instance, WelcomeDialog is a special case of Dialog. The best practice here is to achieve this by composition where more "specific" component renders a more "generic" one and configures it with props:
```
function Dialog (props) {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title>
        {props.title}
      </h1>
      <p className="Dialog-message>
        {props.message}
      </p>
    </FancyBorder>
  );
}

function WelcomeDialog () {
  return (
    <Dialog title="Welcome"
            message="Thank you for visiting!" />
  );
}
```
#### What about inheritance?
- Even at Facebook, where they have thousands of components, they haven't found anyuses cases where they would recommend creating component inheritance hierarchies. Props and composition give all the needed flexibility.
- If you want to reuse non-UI functionality between components, we suggest extracting it into a separate JavaScript module and import and use that function/object/class without extending it.
### Thinking in React
1. Start with a mock -- imagine we have a JSON API and mock from the designer...
2. Break the UI into a component hierarchy
    - Break the mock up by drawing boxes around every component (and sub-component) in the mock and give them all names
    - How to know how small a component should be? Keep single responsibility principle in mind: a component should ideally only do one thing, if it grows, it should be broken out into two or more smaller components
    - Often you'll find that, if built correctly, your UI and your data model will map nicely since they should both adhere to the same information architecture
3. Build a static version in React
    - At this stage it's best to decouple the building of the UI and the incorporation of interactivity; building a static version requires a lot of typing and very little critical thinking, whereas adding interactivity requries a lot of critical thinking and not a lot of typing.
    - Ignore state in this static version, the implementation of state is reserved for interactivity. Just pass data using props at this point.
    - At the end of this step you'll have a library of reusable components that render your data model. The components will only have `render()` methods. The component at the top of the hierarchy will take your data model as a prop
    - NOTE: remember the differences between the props and state models of data, remember why props are the default, and state is implemented when and for the data that is expected to change as the user interacts with components.
4. Identify the minimal (but complete) representation of UI state
    - To start, think about the minimal set of mutable state that your app needs, compute the rest on-demand. For example, in a to-do app, you likely only need to maintain the list of to-dos, but not a second variable for tracking the length of the list.
    - If having a hard time determining which elements of the app require state, consider these:
      - Is it passed from parent via props? If so, probably not state.
      - Does it remain unchanged over time? If so, probably not state.
      - Can you compute it based on any other state or props in your component? If so, probably not state.
5. Identify where your state should live
    - For each piece of state in your application, consider the following:
      - Identify every component that renders somethign based on that state, find a common owner component (or another component higher up the hierarchy) can hold this state.
      - If you can't find a component where it makes sense to own the state, create a new component simply for holding the state and add it somewhere in the hierarchy above the common owner component.
6. Add inverse data flow
    - Now it's time to consider how data will flow out of the deeply-nested components in order to update the state. This usually takes the form of an onChange prop that's passed down from the state-holding component.
