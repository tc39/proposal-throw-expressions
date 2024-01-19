# ECMAScript `throw` expressions

This proposal defines new syntax to throw exceptions from within an expression context.

## Status

**Stage:** 2  
**Champion:** Ron Buckton (@rbuckton)

_For more information see the [TC39 proposal process](https://tc39.github.io/process-document/)._

## Authors

* Ron Buckton (@rbuckton)

# Proposal

A `throw` expression allows you to throw exceptions in expression contexts. For example:

* Parameter initializers
  ```js
  function save(filename = (throw new TypeError("Argument required"))) {
  }
  ```
* Arrow function bodies
  ```js
  lint(ast, { 
    with: () => (throw new Error("avoid using 'with' statements."))
  });
  ```
* Conditional expressions
  ```js
  function getEncoder(encoding) {
    const encoder = encoding === "utf8" ? new UTF8Encoder() 
                  : encoding === "utf16le" ? new UTF16Encoder(false) 
                  : encoding === "utf16be" ? new UTF16Encoder(true) 
                  : (throw new Error("Unsupported encoding"));
  }
  ```
* Logical operations
  ```js
  class Product {
    get id() { return this._id; }
    set id(value) { this._id = value || (throw new Error("Invalid value")); }
  }
  ```

In order to avoid a difference in precedence between `throw` expression and a _ThrowStatement_, we want to ensure that
operators within the expression are parsed in the same way in order to avoid ambiguity and confusion:

```js
throw a ? b : c; // evaluates 'a' and throws either 'b' or 'c'
(throw a ? b : c);

throw a, b; // evaluates 'a', throws 'b'
(throw a, b);

throw a && b; // throws 'a' if 'a' is falsy, otherwise throws 'b'
(throw a && b);

throw a || b; // throws 'a' if 'a' is truthy, otherwise throws 'b'
(throw a || b);

// ... etc.
```

As a result, `throw` expressions are only parsed inside of a _ParenthesizedExpression_, and thus will always require
outer parentheses to be parsed correctly:

```js
(throw a, b); // evaluates 'a', throws 'b'
(throw a ? b : c); // evaluates 'a' and throws either 'b' or 'c'
a ? (throw b) : c;

// mostly nonsensical uses, requires extra parenthesis:
if ((throw a)) { }
while ((throw a)) { }
do { } while ((throw a));
for ((throw a);;) { }
for (; (throw a);) {}
for (;; (throw a)) {}
for (let x in (throw a)) { }
switch ((throw a)) { }
switch (a) { case (throw b): }
return (throw a);
throw (throw a);
`${(throw a)}`;
a[(throw b)];
a?.[(throw b)];
```

# Grammar

```diff grammarkdown
  CoverParenthesizedExpressionAndArrowParameterList[Yield, Await] :
    `(` Expression[+In, ?Yield, ?Await] `)`
    `(` Expression[+In, ?Yield, ?Await] `,` `)`
+   `(` ThrowExpression[?Yield, ?Await] `)`
    `(` `)`
    `(` `...` BindingIdentifier[?Yield, ?Await] `)`
    `(` `...` BindingPattern[?Yield, ?Await] `)`
    `(` Expression[+In, ?Yield, ?Await] `,` `...` BindingIdentifier[?Yield, ?Await] `)`
    `(` Expression[+In, ?Yield, ?Await] `,` `...` BindingPattern[?Yield, ?Await] `)`

  ParenthesizedExpression[Yield, Await] :
    `(` Expression[+In, ?Yield, ?Await] `)`
+   `(` ThrowExpression[?Yield, ?Await] `)`

+ ThrowExpression[Yield, Await] :
+   `throw` Expression[+In, ?Yield, ?Await]
```

# Other Notes

A `throw` expression can be approximated in ECMAScript using something like the following definition:

```js
const __throw = err => { throw err; };

// via helper...
function getEncoder1(encoding) {
  const encoder = encoding === "utf8" ? new UTF8Encoder() 
                : encoding === "utf16le" ? new UTF16Encoder(false) 
                : encoding === "utf16be" ? new UTF16Encoder(true) 
                : __throw(new Error("Unsupported encoding"));
}

// via arrow...
function getEncoder2(encoding) {
  const encoder = encoding === "utf8" ? new UTF8Encoder() 
                : encoding === "utf16le" ? new UTF16Encoder(false) 
                : encoding === "utf16be" ? new UTF16Encoder(true) 
                : (() => { throw new Error("Unsupported encoding"); })();
}
```

However, this has several downsides compared to a native implementation:
* The `__throw` helper will appear in `err.stack` in a host environment.
  * This can be mitigated in *some* hosts that have `Error.captureStackTrace`
* Hosts require more information for optimization/deoptimization decisions as the `throw` is not local to the function.
* Not ergonomic for debugging as the frame where the exception is raised is inside of the helper.
* Inline invoked arrow not ergonomic (at least 10 more symbols compared to native).

# Resources

- [Overview Slides](https://tc39.github.io/proposal-throw-expressions/ThrowExpressions-tc39.pptx)

# TODO

The following is a high-level list of tasks to progress through each stage of the [TC39 proposal process](https://tc39.github.io/process-document/):

### Stage 1 Entrance Criteria

* [x] Identified a "[champion][Champion]" who will advance the addition.  
* [x] [Prose][Prose] outlining the problem or need and the general shape of a solution.  
* [x] Illustrative [examples][Examples] of usage.  
* [x] ~High-level API~ _(proposal does not introduce an API)_.  

### Stage 2 Entrance Criteria

* [x] [Initial specification text][Specification].  
* [x] _Optional_. [Transpiler support][Transpiler].  

### Stage 3 Entrance Criteria

* [x] [Complete specification text][Specification].  
* [x] Designated reviewers have [signed off][Stage3ReviewerSignOff] on the current spec text.  
* [x] The ECMAScript editor has [signed off][Stage3EditorSignOff] on the current spec text.  

### Stage 4 Entrance Criteria

* [ ] [Test262](https://github.com/tc39/test262) acceptance tests have been written for mainline usage scenarios and [merged][Test262PullRequest].  
* [ ] Two compatible implementations which pass the acceptance tests: [\[1\]][Implementation1], [\[2\]][Implementation2].  
* [ ] A [pull request][Ecma262PullRequest] has been sent to tc39/ecma262 with the integrated spec text.  
* [ ] The ECMAScript editor has signed off on the [pull request][Ecma262PullRequest].  

<!-- The following are shared links used throughout the README: -->

[Champion]: #status
[Prose]: #proposal
[Examples]: #proposal
[Specification]: https://tc39.github.io/proposal-throw-expressions
[Transpiler]: https://github.com/Microsoft/TypeScript/pull/18798
[Stage3ReviewerSignOff]: https://github.com/tc39/proposal-throw-expressions/issues/7
[Stage3EditorSignOff]: https://github.com/tc39/proposal-throw-expressions/issues/8
[Test262PullRequest]: #todo
[Implementation1]: #todo
[Implementation2]: #todo
[Ecma262PullRequest]: #todo