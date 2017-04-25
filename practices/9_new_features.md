C# 7:

patterns and pattern variables: use with abandon; no special formatting rules; use inline variables

switch statements with patterns: also cool, same formatting rules as for switch statements


Local functions:

- always use when a private method is used only once
- do not redeclare if the method is used more than once
- do not nest unreasonably; always consider composition instead to allow customization/mocking
- Use where you can to avoid constant re-checking of null values

 DO use local functions instead of anonymous functions when a delegate is not needed, especially when a closure is involved.
✔ DO use local iterators when returning an IEnumerator when parameters need to be validated.
✔ CONSIDER placing local functions at the very beginning or end of a function to visually separate them from their parent function.
✘ AVOID using closures with delegates in performance sensitive code. This applies to both anonymous and local functions.




Ref returns and locals: you probably don't need it. Don't go nuts with it without reason.

Expression-bodied members: use for simple, one-liners. Use good judgment.

Throw in expression-bodied members: of course.

Exception conditions: also totally fine.


 CONSIDER providing a tuple return alternative to out parameters.
✘ AVOID using out or ref parameters. [See Framework Design Guidelines]
✔ CONSIDER providing overloads that omit the out parameters so wildcards are not needed.





CONSIDER using ref returns instead of index values in functions that work with arrays.
✔ CONSIDER using ref returns instead of normal returns for indexers on custom collection classes that hold structs.
✔ DO expose properties containing mutable structs as ref properties.
✘ DO NOT expose properties containing immutable structs as ref properties.
✘ DO NOT expose ref properties on immutable or read-only classes.
✘ DO NOT expose ref indexers on immutable or read-only collection classes.

✔ CONSIDER using ValueTask<T> in performance sensitive code when results will usually be returned synchronously.
✔ CONSIDER using ValueTask<T> when memory pressure is an issue and Tasks cannot be cached.
✘ AVOID exposing ValueTask<T> in public APIs unless there are significant performance implications.
✘ DO NOT use ValueTask<T> when calls to Task.WhenAll or WhenAny are expected.

DO use expression bodied members for simple properties.
✔ DO use expression bodied members for methods that just call other overloads of the same method.
✔ CONSIDER using expression bodied members for trivial methods.
✘ DO NOT use more than one conditional (a ? b : c) or null-coalescing (x ?? y) operator in an expression bodied member.
✘ DO NOT use expression bodied members for constructors and finalizers.

 CONSIDER placing throw expressions on the right side of conditional (a ? b : c) and null-coalescing (x ?? y) operators in assignments/return statements.
✘ AVOID placing throw expressions on the middle slot of a conditional operator.
✘ DO NOT place throw expressions inside a function's parameter list.