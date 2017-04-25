# Patterns & Best Practices


## Using Generics

* Use generic collection types (e.g. use `IList<T>` instead of `IList`).
* Declare generic constraints instead of casting or using the `is`-operator.
* Generic constraints should operate on interfaces wherever possible.
* When inheriting from both a generic and non-generic interface (e.g. `IEnumerable` and `IEnumerable<T>`), implement the non-generic version explicitly and implement it using the generic interface.


## Defining Extension Methods

* Do not extend `object` or `string` in commonly used namespaces.
* Use extension methods to extend system or third-party library types.
* Do not mix extension methods with other static methods. If a class contains extension methods, it should contain only extension methods and private support methods.

### Interface Design

* Use extension methods for methods that can be defined in terms of public members of that interface.
* Do so only for methods for which the implementation is certain to be the same for all implementations. That is, do not restrict an implementation’s efficiency because an extension instead of interface method was used.
* Define useful, but more rarely used extension methods in a separate namespace to force callers to "opt in".
* Do not mix extension methods for different types in one class.
* Do not _make decisions_ in extension methods. Instead, declare components and inject them where needed.
* Do not use static code that does _make decisions_ in extension methods.
* Do not use a global service locator in extension methods.
* Do not pass an IOC to extension methods.

## Generated code

* Use partial classes for generated code wherever possible.
* Do not commit manual changes to generated code. Temporary changes for debugging are fine, but always be aware that your changes will be overwritten when code is re-generated.

## Setting Timeouts

* Use `TimeSpans` to encapsulate timeout values.
* Prefer passing the timeout value as a parameter to the method that will use it rather than offering it as a property on the class. This more closely associates the timeout value with the operation that uses it.
* The method can decide which values of `TimeSpan` are valid (including `TimeSpan.Zero` and `TimeSpan.MaxValue`).

## Configuration & File System

* An assembly should not assume its location.
* Do not use the registry to store application information; save user settings to a user-accessible file instead.

## Logging and Tracing

* Do not log directly to any output (e.g. a file or the console).
* Avoid logging directly to global or static constructs.
* Instead, inject an interface into the method where needed (e.g. `ILogger`).



