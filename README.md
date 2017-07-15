# ECMAScript `throw` expressions

This proposal defines new syntax to throw exceptions from within an expression context.

# Proposal

A `throw` expression allows you to throw exceptions in expression contexts. For example:

* Parameter initializers
  ```js
  function save(filename = throw new TypeError("Argument required")) {
  }
  ```
* Arrow function bodies
  ```js
  lint(ast, { 
    with: () => throw new Error("avoid using 'with' statements.")
  });
  ```
* Conditional expressions
  ```js
  function getEncoder(encoding) {
    const encoder = encoding === "utf8" ? new UTF8Encoder() 
                  : encoding === "utf16le" ? new UTF16Encoder(false) 
                  : encoding === "utf16be" ? new UTF16Encoder(true) 
                  : throw new Error("Unsupported encoding");
  }
  ```
* Logical operations
  ```js
  class Product {
    get id() { return this._id; }
    set id(value) { this._id = value || throw new Error("Invalid value"); }
  }
  ```

A `throw` expression *does not* replace a `throw` statement due to the difference 
in the precedence of their values. To maintain the precedence of the `throw` statement,
*ThrowStatement* must be parsed ahead of *ExpressionStatement* in the grammar.

# Grammar

```grammarkdown
AssignmentExpression[In, Yield, Await]:
  ...
  ThrowExpression[?In, ?Yield, ?Await]
  ...

ThrowExpression[In, Yield, Await]:
  `throw` [no LineTerminator here] AssignmentExpression[?In, ?Yield, ?Await]
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