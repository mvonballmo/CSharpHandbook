C# 6:

Auto-Property Initializers: initialize a property in the declaration rather than in the constructor or on an otherwise unnecessary local variable.

Out Parameter Declaration: An out parameter can now be declared inline with var or a specific type. This avoids the ugly variable declaration outside of a call to a Try* method.

Using Static Class: using can now be used with with a static class as well as a namespace. Direct access to methods and properties of a static class should clean up some code considerably.

- I haven't encountered this. I wouldn't know which classes to include as static. Extension methods are already interpreted correctly, by nature. Anything else seems odd.

String Interpolation: Instead of using string.Format() and numbered parameters for formatting, C# 6 allows expressions to be embedded directly in a string (á la PHP): e.g. “{Name} logged in at {Time}”

nameof(): This language feature gets the name of the element passed to it; useful for data-binding, logging or anything that refers to variables or properties.

Null-conditional operator: This feature reduces conditional, null-checking cruft by returning null when the target of a call is null. E.g. company.People?[0]?.ContactInfo?.BusinessAddress.Street includes three null-checks

C# 7:

out variables: use these always; use var

patterns and pattern variables: use with abandon; no special formatting rules; use inline variables

switch statements with patterns: also cool, same formatting rules as for switch statements

tuple types and tuple literals: use these instead of Try with out where possible. Always name the parameters. You can use short names since the tuple type is very transient and localized.

If the method returns a single constant tuple, then specify the names there (to keep the method declaration shorter). If there are several exit points, don't repeat the names in the literal tuples; instead, include the names only in the return-type declaration.

Deconstruction: Use deconstruction where appropriate to consume tuples. If the tuple has named members, then you can just use it; otherwise, use deconstruction to assign the members to variables with logical names.

Prefer the external var (first, second, third) declaration to the internal one (var first, var second, var third)

Declare Deconstructors only where they can be reasonably expected to be used. (e.g. for smaller pure-data classes or structs).

Discards (_): use this instead of declaring useless local variables during deconstruction.

Local functions:

- always use when a private method is used only once
- do not redeclare if the method is used more than once
- do not nest unreasonably; always consider composition instead to allow customization/mocking
- Use where you can to avoid constant re-checking of null values

Literal _ in numbers: go ahead, but be reasonable

Ref returns and locals: you probably don't need it. Don't go nuts with it without reason.

Expression-bodied members: use for simple, one-liners. Use good judgment.

Throw in expression-bodied members: of course.

Exception conditions: also totally fine.

From https://www.infoq.com/articles/Patterns-Practices-CSharp-7:

✔ CONSIDER using tuple returns instead of out parameters when the list of fields is small and will never change.
✔ DO use PascalCase for descriptive names in the return tuple. This makes the tuple fields look like properties on normal classes and structs.
✔ DO use var when reading a tuple return without deconstructing it. This avoids accidentally mislabeling fields.
✘ AVOID returning value tuples with a total size of more than 16 bytes. Note, reference variables always count as 4 bytes on a 32-bit OS and 8 bytes on a 64-bit OS.
✘ AVOID returning value tuples if reflection is expected to be used on the returned value.
✘ DO NOT use tuple returns on public APIs if there is a chance additional fields will need to be returned in future versions. Adding fields to a tuple return is a breaking change.


 CONSIDER using deconstruction when reading tuple return values, but be aware of mislabeling mistakes.
✔ DO provide a custom deconstruct method for structs.
✔ DO match the field order in a class's constructor, ToString override, and Deconstruct method.
✔ CONSIDER providing secondary deconstruct methods if the struct has multiple constructors.
✘ DO NOT expose Deconstruct methods on classes when it isn't obvious what order the fields should appear in.
✘ DO NOT expose multiple Deconstruct methods with the same number of parameters.


 CONSIDER providing a tuple return alternative to out parameters.
✘ AVOID using out or ref parameters. [See Framework Design Guidelines]
✔ CONSIDER providing overloads that omit the out parameters so wildcards are not needed.



 DO use local functions instead of anonymous functions when a delegate is not needed, especially when a closure is involved.
✔ DO use local iterators when returning an IEnumerator when parameters need to be validated.
✔ CONSIDER placing local functions at the very beginning or end of a function to visually separate them from their parent function.
✘ AVOID using closures with delegates in performance sensitive code. This applies to both anonymous and local functions.


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