# React Advanced Guides

### Accessibility

- Accessibility compliance checklists:
  - https://www.wuhcag.com/wcag-checklist/
  - https://webaim.org/standards/wcag/checklist
  - https://a11yproject.com/checklist/
- `aria-*` HTML attributes are fully supported in JSX; naming convention is counter to typical React camelCase and maintain their hyphen-cased format
- Reference for Semantic HTML Elements: https://developer.mozilla.org/en-US/docs/Web/HTML/Element
- Rather than use `<div>` to hold together groups of elements, it's preferred that we use the React Fragment:

```javascript
import React, { Fragment } from "react";

function ListItem({ item }) {
  return (
    <Fragment>
      <dt>{item.term}</dt>
      <dd>{item.description}</dd>
    </Fragment>
  );
}

function Glossary(props) {
  return (
    <dl>
      {props.items.map(item => {
        <ListItem item={item} key={item.id} />;
      })}
    </dl>
  );
}
```

- `<></>` is also shorthand for Fragment tags that don't need any props (may require extra tooling)
- Each HTML form control needs to be labeled accessibly, this includes proper labeling:

```javascript
<label htmlFor="namedInput">Name:</label> // htmlFor is the react way of notating the `for` attribute
<input id="namedInput" type="text" name="name" />
```

- How and where submission or validation errors are shown is also important, keep these in mind and implement as many as possible/make sense:

  - Include success/error messages in the main page heading; ideally this would be the most prominently displayed heading element, updated to reflect something like "3 Errors - Billing Address" or "Thank you for submitting your order"

  - Include success/error messages in the page title; similar to including messages in page headings, swap the title text for a descriptive message about the state of the submission

  - Use dialogs (`alert()`); this is commonly used if updating the title or primary heading is easily missed. Do keep in mind that dialogs are more obtrusive, which may or may not be desired. The alert dialog provides proper keyboard navigation and respects the user's defualt settings, including font size, colors, and language. I custom dialog would have to match that functionality

  - Listing errors; ideally at the top of the page or just before the form. The list should have a distinct heading so that it's easy to identify. Additionally, each error listed should reference the label of the corresponding form control for recognizability, provide a concise, easy to understand description of the error, provide an indication of how to correct the mistake(s), and include an in-page link (`href="#firstname"`) to the corresponding form control to make access easier for the users

  - In-line feedback; should be in addition to overall feedback, specfic feedback at or near the form controls can better help users to navigate your form; this can be a text indicator or symbolic (green checkmark next to valid inputs, red X or exclamation mark and a red border on the input for invalid inputs); this should include instructions on how to correct the error(s)

- There are a number of different approaches you can take to form validation:

  - Show validation after submission, appropriate success and error messages are displayed for each input field to help the user identify fields to be correct; if the submitted data includes error(s) it is convenient to set the focus to the first input element that contains an error

  - Show validation while the user is typing, for example checking the availability of a username on signup for a service as opposed to making the user resubmit the form multiple times; keep in mind that not all situations may be appropriate for such immediate, persistent feedback. Scaled feedback is another example of validation on typing, password strength meters are a good example of this, this visual gauge should also be accompanied by a text label

  - Show validation on focus changes, for example this waits to validate the form field until focus is moved off of that field (to the next field or by clicking the page background). This is useful in cases where validation on typing would display an error message most of the time

  - NOTE : the code examples found (here)[https://www.w3.org/WAI/tutorials/forms/notifications/] show the use of `aria-live` attributes with a value of both `polite` and `assertive`. As you might imagine, polite displays the error without interrupting a screen reader's progress through the page, assertive would cause the errors to interrupt the screen reader in order to be read by the user as soon as possible.

- It's also important to make sure users can navigate through and fully operate you form with keyboard only, typically through use of the `tab` or `shift-tab` to navigate forward and back. Whichever form field has focus should have a visual indicator of this status, that can be applied with `outline` in the CSS (only ever set `outline: 0` on an element where you're replacing that visual representation of active, focused state). Navigation order is also important to appropriately navigating a form, this is accomplished with the `tabindex` attribute.

- `a` elements are only keyboard-accessible or presented to screen readers as a link when it has a non-empty `href` value. Links without an `href` attribute (or an empty value) should not be used.

- (This)[https://webaim.org/techniques/keyboard/] page includes a table showing the expected navigation functions of different keystroke combinations

- left off here: https://reactjs.org/docs/accessibility.html#mechanisms-to-skip-to-desired-content

### Code Splitting

- https://reactjs.org/docs/code-splitting.html

### Context

- Context is essentially a way to maintain globally-scoped value instead of having to pass the props down the component tree, this is especially useful for things like themeing elements

```javascript
const ThemeContext = React.createContext("light");

function App(props) {
  return (
    // Use a Provider to pass the current theme to the tree below.
    // Any component can read it, no matter how deep it is.
    // In this example, we're passing "dark" as the current value.
    <ThemeContext.Provider value="dark">
      <Toolbar />
    </ThemeContext.Provider>
  );
}

function Toolbar(props) {
  return (
    // A component in the middle doesn't have to
    // pass the theme down explicitly anymore.
    <div>
      <ThemedButton />
    </div>
  );
}

function ThemedButton(props) {
  // Assign a contextType to read the current theme context.
  // React will find the closest theme Provider above and use its value.
  // In this example, the current theme is "dark".
  const context = ThemeContext;

  return <Button theme={context} />;
}
```

- Context is primarily used when some data needs to be accessible by many components at different nesting levels. Apply it sparingly because it makes component reuse more difficult. If you only want to avoid passing some props through many levels, component composition (passing the deeply-nested, affected component through the component tree instead of the props) is often a simpler solution than context. This inversion of control can yield cleaner code as it reduces the number of props that must be passed from component to component through the tree. In situations where the child needs to communciate with the parent before rendering, check out Render Props (below)

- Example of dynamic context: https://reactjs.org/docs/context.html#dynamic-context

- Example of nested context: https://reactjs.org/docs/context.html#updating-context-from-a-nested-component

- Example of multiple contexts: https://reactjs.org/docs/context.html#updating-context-from-a-nested-component

- Context Gotchyas: https://reactjs.org/docs/context.html#caveats

### Error Boundaries

- https://reactjs.org/docs/error-boundaries.html

### Forwarding Refs

- https://reactjs.org/docs/forwarding-refs.html

### Fragments

- Fragments let you group a list of children without adding extra nodes to the DOM, this reduces the depth of the nested component tree structure.

- Fragments may also have keys if they were created via a looping or map function

### Higher-Order Components

- Simply put, a higher-order component is a function that takes a component and returns a new component

- Higher-order components are the preferred method of code sharing between components (previously mixins were relatively common).

### Integrating with Other Libraries

-

### JSX In Depth

-

### Optimizing Performance

-

### Portals

-

### React Without ES6

-

### React Without JSX

-

### Reconciliation

-

### Refs and the DOM

-

### Render Props

-

### Static Type Checking

-

### Strict Mode

-

### Typechecking with PropTypes

-

### Uncontrolled Components

-

### Web Components

-
