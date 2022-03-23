# JSX/TSX Cheatsheet

JSX/TSX is the special syntax on top of JavaScript and TypeScript to support
React UI declarations in a easier to type fashion.

Instead of typing `React.createElement('div', {}, 'My element!')`, JSX/TSX makes
it possible to type `<div>My element!</div>` and have it be compiled to the code
mentioned prior.

[React Without JSX]: https://reactjs.org/docs/react-without-jsx.html

React _can_ be used without JSX/TSX, see [React Without JSX], but using this
syntactic sugar makes it easier to build and read deeply nested component trees.

JSX is different from HTML, HTML is interpreted by the browser directly and is
independent of JavaScript. JSX is a declarative way using managing the JS DOM
which is a JavaScript representation of the parsed HTML document. This is as
opposed to an imperative approach, that would be the React Without JSX sample
above.

## Interpolate value

As a part of the element's content:

```tsx
const name = 'Tom';

<div>Hello, {name}!</div>
```

As a value of an element's attribute:

```tsx
const id = 'mainDiv';
<div id={id}>content</div>
```

[`do` expression proposal]: https://babeljs.io/docs/en/babel-plugin-proposal-do-expressions

The code in the curly braces needs to be an expression, not a statement. Check
out [`do` expression proposal] for a possible future way for running statements
in JSX/TSX interpolation operator. Or just run the code outside of the JSX/TSX
tree.

## Wrapping JSX/TSX trees in parentheses to avoid ASI problems

ASI stands for automatic semicolon insertion. Refer to the ECMA spec to see its
rules: https://262.ecma-international.org/7.0/#sec-rules-of-automatic-semicolon-insertion

An example of ASI in action:

```ts
return
1 + 1;
```

It seems we just inserted a line break for formatting, but ASI will actually
make the resulting code be interpreted as if we wrote this:

```ts
return;
1 + 1;
```

[TypeScript Playground]: https://www.typescriptlang.org/play

You can verify this yourself in the [TypeScript Playground].

In terms of JSX/TSX, ASI could lead to a bad time if we placed the JSX/TSX on
its own line like the expression above:

```tsx
return
  <div>
    content
  </div>;
```

This will compile into a standalone `return` statement making the function
return `undefined` and the JSX/TSX bit will just be a free-standing value.

To fix the problem, it is generally recommended to wrap any JSX/TSX expression
in parentheses:

```tsx
return (
  <div>
    content
  </div>
);
```

[Understanding Automatic Semicolon Insertion in JavaScript by Bradley Braithwaite]: http://www.bradoncode.com/blog/2015/08/26/javascript-semi-colon-insertion

ASI impacts JSX/TSX pretty much just in this way. The other situations where it
might kick in don't affect JSX/TSX, see
[Understanding Automatic Semicolon Insertion in JavaScript by Bradley Braithwaite]
for details.

One could also not use parentheses and just remove the newline like so:

```tsx
return <div>
         content
       </div>;
```

However, the automated code formatter in your IDE of choice is likely going to
mess with the indentation and it is overall IMO less readable than the paren
wrap.

## Storing JSX/TSX in a variable

JSX/TSX tree doesn't have to be just a return value of a component function and
it doesn't need to be built up in a single piece. Check out this example where
the tree is made up of multiple subtrees all built independently and some even
conditionally.

```tsx
const user; // undefined or { name: string; } object
const header = <div>Hi, user!</div>;
const footer = <div>Copyright Tomas Hubelbauer</div>;
let main = user ? <div>Hi, {user.name}!</div> : <button>Sign in</button>;

return (
  <main>
    {header}
    {main}
    {footer}
  </main>
);
```

## Attributes

Attribute names in JSX/TSX are based on the members of the classing backing the
HTML elements when represented by DOM. Most HTML attributes will have the same
name in DOM, but there are exceptions, such as `class` becomes `className`,
`tabindex` becomes `tabIndex`, `data-test` becomes `dataset.test` and so on.

Also not all attibute types are what you would expect from HTML, for example,
the `checked` attribute of an `input` element is a flag, if present, it is
checked regardless of its value, if missing, it is not. In the DOM version of
the `input` element, `HTMLInputElement`, the `checked` attribute is a `boolean`
value.

- HTML: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/checkbox
- DOM: https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement

In JSX/TSX you always need to use either quotes (single or double) or curly
braces to set the attribute's value. HTML will allow you to omit them in simple
cases.

Also, the behavior of an attribute with no specified value is different. A sole
`checked` attribute on an `input` will mean the input is checked. A sole
`checked` attribute in JSX/TSX will mean the value of the attribute is literal
`true` boolean value.

[Spread Attributes]: https://reactjs.org/docs/jsx-in-depth.html#spread-attributes

There is also a way to pass an object with key value pairs to be used as element
attributes and their values in JSX/TSX. This is called [Spread Attributes]:

```tsx
const props = { id: 'id', className: 'class' };
return <div {...props} />;
```

## Self-closing any element

In HTML, not all elements can be self-closing. For example, the `canvas` element
always needs to have an explicit closing tag, but will likely never have any
content of its own. In JSX/TSX, every element is self-closing. This is valid:

```tsx
const div = <div />;
```

It is the same as typing `<div></div>`.

## Interpolating raw HTML into JSX/TSX

By default, JSX/TSX will escape all strings so that raw HTML cannot be directly
inserted. This is to prevent XSS attacks.

```tsx
const html = '<img src=x onerror=alert(1)>';

// <div>&lt;img src=x onerror=alert(1)&gt;</div>
return <div>{html}</div>;
```

[`dangerouslySetInnerHTML`]: https://reactjs.org/docs/dom-elements.html#dangerouslysetinnerhtml

To actually put raw HTML into JSX/TSX, use [`dangerouslySetInnerHTML`]:

```tsx
const html = '<img src=x onerror=alert(1)>';

// <div><img src="x" onerror="alert(1)"></div>
return <div dangerouslySetInnerHTML={{ __html: html }}>{html}</div>;
```

## `React.createElement`

JSX/TSX elements do not get transformed into representations or calls on the DOM
APIs representing HTML elements. Instead, they are transformed into nested
`React.createElement` function calls. More on this in [React Without JSX].

This is so that React can do what's called a reconciliation where the tree
represented by the JSX/TSX code gets compared to the current state of the DOM
(simplifying, it is actually compared to the previous version of that tree) and
JS calls to the DOM API are issued to make the changes between the two trees
reflect in the DOM in the least amount of DOM API calls possible.

These trees are called the VDOM, or "virtual DOM".

## Sources

- [Introducing JSX](https://reactjs.org/docs/introducing-jsx.html)

## To-Do

### Work through more sources and update the cheat sheet accordingly

- [JSX In Depth](https://reactjs.org/docs/jsx-in-depth.html)
- [JSX (TypeScript - TSX)](https://www.typescriptlang.org/docs/handbook/jsx.html)
- [React JSX (W3Schools)](https://www.w3schools.com/react/react_jsx.asp)

There are more bits worth highlighting in these articles. The W3Schools one I
don't think adds too much on top of the official documentation, but it does
mention all elements must be closed which I want to make sure I don't forget to
add; not sure if the other documents mention this.

### Rewrite for brevity and remove unnecessary fluff, rely on examples over text

This document as it is currently version is my v1, I have mostly wrote what I
could recall would be worth mentioning and helped myself with the official JSX
documentations, but in time I want to add tips and tricks (like fragments with
keys, stuff about returning arrays, null, undefined etc.) so it should start
looking less like a rewrite of the documentation page in other words and more
like a set of snippets for quick reference.

I have also wrote some tips up already in another repository, but I failed to
look it up and I think I ended up deleting it at some point, which is a shame
because it already had a bunch of good tips. :(
