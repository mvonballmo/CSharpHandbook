# Documentation

## Files

* Include a `README.md` file at the root of the project that includes the following information:
  * Dependencies
  * Basic configuration
  * Basic command line
  * Links to other documentation
* Include a `LICENSE` file at the root of the project that describes licensing restrictions.

## Language

* Use U.S. English spelling and grammar.
* Use full sentences or clauses; do not use lists of keywords or short phrases.
* Use the prepositional possessive for code elements (i.e. write "the value of `<paramref name="prop">`" instead of "`<paramref name="prop">`’s value").

## Style

* An API should document itself. Code documentation can sometimes be very obvious and simple. This indicates to the caller that it really _is_ that simple.
* Document similar members consistently; it’s better to repeat yourself or to use the same structure for all members as long as the documentation is useful for each member.
* Include conceptual documentation for each concept/component to provide an overview and examples of how to use the product.
* Move longer documentation out of the code and into higher-level conceptual documentation or examples.

## XML Documentation

* Include XML documentation to enhance code-completion.
* Document `public` and `protected` elements.
* Do not document `private` or `internal` members.
* Include references to important members from class documentation.

### Dependencies

* Do not introduce dependencies for documentation.
* Do not add `using` statements for documentation; if necessary, include the required namespace in the documentation reference itself.

### Tags

* Format block tags onto separate lines. E.g. `<summary>`, `<param>`, `<remarks>` and `<returns>`
* Use `<c>` tags for the keywords `null`, `false` and `true`.
* Use `<see>` tags to refer to properties, methods and classes.
* Use `<paramref>` and `<typeparamref>` tags to refer to method parameters.
* Use the `<inheritdoc/>` tag for method overrides or interface implementations.
* Use a `<remarks>` section to indicate usage and to link to related members. It’s sometimes good to include references to other types or methods in descriptive sentences in addition to listing them in the `<seealso>` section.

### Examples

1. [Examples](examples.md)