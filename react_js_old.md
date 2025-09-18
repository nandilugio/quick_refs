REACT JS
========

JSX templates
=============

Converted to JS during compilation.

```jsx
<SomeJsx withSomeParam={ "that can be " + "dynamic" }>
	<NestedThing prop="ok" />
  	<ul>
  		{ [1,2,3].map((n) => <li key={n}>{n}</li>) }
  	</ul>
    This is regular text
    { 3 }
</SomeJsx>
```

becomes:

```js
import { jsx as _jsx } from "react/jsx-runtime";
import { jsxs as _jsxs } from "react/jsx-runtime";

/*#__PURE__*/_jsxs(SomeJsx, {
    withSomeParam: "that can be " + "dynamic",
    children: [
        /*#__PURE__*/_jsx(NestedThing, { prop: "ok" }),
        /*#__PURE__*/_jsx("ul", {
            children: [1, 2, 3].map(
                n => /*#__PURE__*/_jsx("li", { children: n }, n)  // Last n is the "key", a unique prop required by React to manipulate these DOM elements
            )
        }),
        "This is regular text",
        3  // TODO: This was not casted as a string in compilation-time, but I think it is casted on render 
    ]
});
```

Try it in babeljs.io!

This conversion inplies you need to:
- `return` only one element. If you need more, wrap them in eg. a `<div>`, `<section>` or a React fragment (`<>...</>`)
- Include all JS in `{}`, and place it only where it makes sense: attribute values, children lists (TODO: lists of lists I think are flattened. Check!). Note double-curlies `{{}}` are just JS objects passed as a dynamic JS value!
- Avoid JS naming issues
    - Almost everything should be camelCased
        - Attribute names become JS object keys: no dashes! -> camelCase
    - Some names are reserved, eg. `class`. Use `className` instead
- Uppercased tags are interpreted as components, lowercased as HTML elements


Components
==========

Functions returning some React DOM, normally generated using JSX. They encapsulate the UI, the logic and the state of the component.

```jsx
function MyComponent({ prop1, prop2 }) {
    const result = of(some_logic(prop1));
    return (<SomeJsx withSomeParam={result}>{prop2 * 2}</SomeJsx>);
}

export default MyComponent;
```

is used as:

```jsx
import MyComponent from './MyComponent';

<MyComponent prop1="asd" prop2={3}>
```

**Properties:** The function receives an object with all passed properties, which can be either
- static _strings_ (no numbers, etc.) enclosed in quotes, or
- dynamic JS expressions enclosed in `{}`

State (with `useState`) and Events
----------------------------------

```jsx
import { useState } from 'react';  // useState is a React Hook

export default function Counter({ startFrom = 0 }) {
    const [count, setCount] = useState(startFrom);  // useState generates a piece of "reactive" state, and returns the getter and setter
    
    function handleIncrementClick(e) {
        console.log(e)  // `e` is the event here, not used in the example. Has target, input value, click coords, etc.
        setCount(count + 1);  // Setters make the component be re-rendered
    }

    return (
        <>  {/* This empty tag is a React fragment, used to enclose multiple items and return only one */}
            { !!count && <p>Clicked {count} times.</p> }  {/* Conditional rendering with a simple JS `&&` operator. Note the `!!` is there to avoid rendering `0` when the expression short-circuits! */}
            <button onClick={handleIncrementClick}>
                Increment!
            </button>
        </>
    );
}
```
