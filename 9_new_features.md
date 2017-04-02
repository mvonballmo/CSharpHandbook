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

