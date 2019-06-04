### React as a UI Runtime [(link)](https://overreacted.io/react-as-a-ui-runtime/)

- "Some programs output numbers. Other programs output poems...React programs usually output a **tree that may change over time**. It might be a DOM tree, an iOS hierarchy, a tree of PDF Primitives, or even of JSON objects."
- React makes a bet on two basic principles:
  - Stability (Most updates don't radically change the host tree's overall structure)
  - Regularity (UI can be broken down into UI patterns that look and behave consistently)

#### Host Trees / Host Instances

- Each node on a host tree is known as a host instance; they're regular DOM nodes when used on the web, and they have their own properties as well as other host intances as children. And because these instances are DOM elements, they come with their own APIs like appendChild, removeChild, and setAttribute, etc.

#### Renderers

- The vast majority of renderers are written using the "mutating" mode, which is how the DOM works: can create a node, set it's properties, and later add/remove children form it; these host instances are completely mutable.
- React can also work in a "persistent" mode, which is the mode for host environments that don't provide methods like appendChild, but instead clone the parent tree and always replace the top-level child. Immutability on the host tree level makes multi-threading easier. As a React user, you never need to think about these modes, but it's important to know that it can work either way

#### React Elements

- React elements are just plain JS objects that _describe_ a host instance. They're lightweight and have no host instance tied to them. As with host instances, React elements can form a tree by nesting elements within the child property of their "parent" element
- React elements are immutable and don't have their own persistent identity, they're meant to be re-created and thrown away all the time

#### Entry Point

- this is the portion of `ReactDOM.render` that passes `document.getElementById('_____')`, which is what tells React where to render a particular React element tree

#### Reconciliation

- Since React's job is to make the host tree match the provided React element tree, there are two ways to go about reconciling differing commands:

  1. React could blow away the existing tree and re-create it, but in DOM this is slow and loses important information like field focus, selection, scroll state, etc.
  2. React decideds when to _update_ a host intance and when to create a _new_ one

  - While this can be simple, as with rendering a button as a first (and only) child, React considers whether **an element type in the same place in the tree "matches up" between previous and next renders, if so, React reuses the existing host instance**:

  ```javascript
  ReactDOM.render(
    <button className="blue />,
    document.getElementById('root')
  );

  ReactDOM.render(
    <button className="red" />,
    document.getElementById('root')
  );
  ```

  - When we update a host isntance with child host instances, React first decides whether to re-use the parent instance, then repeats the decision proceedure for each child

#### Conditions

- If React only reuses host instances when the element types "match up" between updates, how can we render conditional content?:

```javascript
// First render
ReactDOM.render(
  <parent>
    <child-one />
  </parent>,
  document.getElementById("root")
);

// Second render
ReactDOM.render(
  <parent>
    <conditional-child /> // has to update since type changed from `child-one`
    to `conditional-child`
    <child-one /> // has to create a new input since there was previously no second
    element
  </parent>,
  document.getElementById("root")
);

// UNLESS

function Form({ showConditional }) {
  let conditionalChild = null;
  if (showConditional) {
    conditionalChild = <p>This is a message</p>;
  }

  return (
    <parent>
      {conditionalChild}
      <child-one /> // regardless of if `showConditional` is true or false, `child-one`
      doesn't change position between renders
    </parent>
  );

  // now the input state isn't lost!
}
```

### Lists

- Comparing element type at the same position in the tree is usually enough to decide whether to reuse or re-create the cooresponding host instance. But this only works well if child positions are static and don't re-order.
- With dynamic lists, we can't be sure the order is ever the same, this is where object keys come in. A key tells React that it should consider an item to be _conceptually_ the same even if it has a different _position_ within a parent element between renders. This is why unique identifiers (like product IDs) are used.
- Keys are only relevant within a particular parent React element. React won't try to "match up" elements with the same keys between different parents.

### Component Purity

- In general, mutation isn't common within React, however, _local mutation_ is absolutely fine. This means that within a Component function we can set a variable `items` and push items into this array variable within the component function before passing it to the `render` method, which is what "locks" it in.
- As long as calling a component multiple times is safe and doens't affect the rendering of other components, React doesn't care if it's 100% pure. Idempotence (completes the same mutation on a component, even if it's run more than once, the end result is the same) is more important to React than purity.

### Recursion

- How do we use components from within other components?:

```javascript
// this IS NOT the idiomatic way
let reactElement = Form({ showMessage: true });
ReactDOM.render(reactElement, document.getElementById("root"));

// this IS the idiomatic way
let reactElement = <Form showMessage={true} />;
ReactDOM.render(reactElement, document.getElementById("root"));
```

- What does React do when an element type is a function? It calls your component and asks what element _that_ component wants to render within it.

```javascript
// This...
ReactDOM.render(<App />, document.getElementById("root"));

// Turns into this...
<div>
  <article>
    Some text
    <footer>More text</footer>
  </article>
</div>;

// When App contains Layout
// And Layout is a `div` that holds Content
// And Content is an `article` that holds some text and Footer
// And Footer is a `footer` that holds more text
```

### Inversion of Control

- Why write `<Form />` instead of `Form()`? React can do its job better if it "knows" about your components rather than if it only sees the React element tree after recursively calling them.
- This is a classic example of _inversion of control_ (instead of with traditional programming, where the custom code that expresses the purpose of a program calls into a reusible library to take care of generic tasks, but with inversion of control, it is the framework that calls into the custom or task-specific code)
- The benefits of relinquishing this control to React of:
  - Components become more than functions (React can augment component functions with features like local state that are tied to the component identity in the tree, if you called components directly you'd have to build these features yourself)
  - Component types participate in the reconciliation (By letting React call your compnents, you are allowing it more information about the conceptual structure of your tree. With this information React won't attempt to re-use host instances when changing from rendering ComponentOne to ComponentTwo, which wipes all stored state)
  - React can delay the reconciliation (If React takes control over calling your components, it can let the browser do some work between the component calls so that re-rendering a large component tree doesn't block the main thread --as well as many other interesting things)
  - Better debugging (If components are first-class citizens, the library can build rich dev tools into it)

### Lazy Evaluation

- If we call components as functions, they would execute immediately whether or not we want to actually render them or not (as with conditional components), but if we pass a React element, it doesn't execute until it's sure it's needed. This reduced unnecessary rendering work that would otherwise just be thrown away and makes the code less fragile

### State

- Host instances can have all kinds of local state: focus, selection, input, etc
- Components are still functions but React augments them with features that are useful for UIs; these additional features are called _Hooks_
- `useHooks` is a hook and it returns a pair of values: the current state and a function that updates it'

### Consistency

- Even if we want to split the reconciliation process into non-blocking chunks, we should still perform the actual host tree operations all at once in order to enxure users don't see a half-updated UI and the browser doesn't perform any unnecessary layout and style recalculation for intermediate states the user won't see

### Memoization

- When a parent component calls `setState`, by default React reconciles it's whole child subtree. This isn't a problem until trees get too deep or wide, at which point you can tell React to memoize a subtree and reuse previous render results during shallow equal prop changes, this causes the setState method in a parent compnent to skip over reconciling `Row`s whose item is referentially equal to the item rendered last time:

```javascript
function Row({ item }) {
  // ...
}

export default React.memo(Row);
```

- You can get fine-grained memoization at the level of individual expressions with the `useMemo` hook. The cache is local to component tree position and will be destroyed together with its local state. It only holds the (one) last item. React does not memoize components by default since they often receive different props so memoizing would be a net loss

### Raw Models

- React intentionally doesn't use a "reactivity" (RxJS) system for fine-grained updates. Any update at the top triggers reconciliation instead of updating just the components affected by the changes
- The primary reason is because traversing models to set up those fine-grained listeners eats up precious time, increasing Time to Interactive

### Batching

- (come back to this after reading about Hooks)

### Call Tree

-

### Context

-

### Effects

-

### Custom Hooks

-

### Status Use Order

-

### What's Left Out

-
