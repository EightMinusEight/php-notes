

### Syntax and Usage

Attributes are declared with #[ and ] braces.

```
#[ExampleAttribute('foo', 'bar')]
function example() {}
```

Attributes are kinds of classes that can be used to add metadata to other classes, functions, class methods, class properties, constants, and parameters. Attributes do nothing during runtime.

Attributes have no impact on the code, but available for the reflection API. Attributes in PHP 8 allow other code to examine the class properties and methods.

We can have more than one attribute to a declaration.

It may resolve the class name.

Attributes can be namespaced.

It may have zero or more parameters

### PHP 8 Attribute Syntax

In PHP 8, #[ ] (# and square brackets) is used for an attribute declaration.

We can declare multiple attributes inside #[ ], separated by a comma.

Arguments are optional to use but need to be enclosed inside parenthesis ().

Arguments can be literals values or constant expressions.
