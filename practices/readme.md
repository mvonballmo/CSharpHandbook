# Patterns & Best Practices

## Using base and this

* Use `this` only when referring to other constructors.
* Do not use `this` to resolve name-clashes; instead, change one of the conflicting names.
* Use `base` only from a constructor or to call a predecessor method.
* You may only call the `base` of the method being executed; do not call other `base` methods. In the following example, the call to `CheckProcess()` is not allowed, whereas the call to `RunProcess()` is.
  ```csharp
  public override void RunProcess()
  {
    base.CheckProcess();  // Not allowed
    base.RunProcess();
  }
  ```

##	Using Value Types

* Always use the lower-case primitive type.
  * Use `int` instead of `Int32`
  * Use `string` instead of `String`
  * Use `bool` instead of `Boolean`
  * Use `short` instead of `Int16`
  * Use `byte` instead of `Byte`
  * Use `long` instead of `Int64`
  * And so on.

## Using Strings

* Prefer string-interpolation for C#6 or higher.
* Prefer `string.Format` for C#5 or lower.
* Use string-concatenation or `string.Concat` only if you have identified a performance bottleneck.
* Use a `StringBuilder` for more complex situations or when concatenation occurs over multiple statements.
* Use `string.Empty` instead of `“”`.
* Use `string.IsNullOrEmpty` instead of `s == null` or `s.Length == 0`.

## Using Checked

* Enable range-checking during development and debugging.
* Disable range-checking in release builds only if there is a valid performance reason for doing so.
* Overflow- and underflow-prone operations should have explicit `checked` blocks (i.e. if there was a range-checking problem at some point, the block should be marked with a `checked` block). This way, even if range-checking is disabled, these blocks will still be checked.

## Using Floating Point and Integral Types

* Be careful when comparing floating-point constants; unless you are using `decimals`, the representation will have limited precision and can lead to subtle bugs.
* One exception to this rule is when you are comparing constants of fixed, known value (like `0.0` or `1.0`, but not `3.14`).
* Be careful when casting representations with different sizes (e.g. `long` to `int`).

## Using Generics

* Use generic collection types (e.g. use `IList<T>` instead of `IList`).
* Declare generic constraints instead of casting or using the `is`-operator.
* Generic constraints should operate on interfaces wherever possible.
* When inheriting from both a generic and non-generic interface (e.g. `IEnumerable` and `IEnumerable<T>`), implement the non-generic version explicitly and implement it using the generic interface.

## Using Partial Classes

To control file size, partial classes can be useful to separate

* `private` or `protected` inner classes.
* larger blocks of `private` or `protected` methods.

## `readonly` vs. `const`

* Use `const` only when the value really is constant (e.g. `NumberDaysInWeek`); otherwise, use `readonly`.
* Use `readonly` as much as possible.

## Resource strings

* Use resources for strings that will be translated (e.g. user-facing messages).
* Do not use resources for catastrophic error messages or log messages.
* Extract strings to resource only if the code using them is final. It's wasted effort to translate strings that might be removed.

## Explicit Interface Implementation

* Do not use explicit interface-implementation except in cases outlined in other sections of this manual. It's a neat trick, but clever code is more often confusing code.



## Using System.Linq

* Pay attention to the order of your Linq expressions to improve performance.
  * Filter before sorting
  * Apply filters that are likely to remove more items first
  * Apply filters with low performance impact first
* Do not use `List.Foreach()`.
* Use multiple lines and indenting to to make expressions more legible.
  ```csharp
  var result = elements
    .Where(e => e.Enabled)
    .Where(e => LastUsed > clock.Now.AddWeeks(-2))
    .OrderBy(e => e.LastUsed)
    .ThenBy(e => e.Name);
  ```
  Here `Enabled` is tested first because it's cheaper to check.
* Use Linq syntax to share temporary variables instead of re-declaring them in several lambdas.
  ```csharp
  var result =
    from e in elements
    where e.Enabled
    where e.LastUsed > clock.Now.AddWeeks(-2)
    orderby e.LastUsed, e.Name;
  ```

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

##	Using Optional Parameters

* Do not use optional parameters in constructors.
* Do not use optional parameters that might change in public APIs; instead, use overloaded methods.
* Do not use more than one or two optional parameters, even for internal APIs.

This blog post [Optional argument corner cases, part four](http://blogs.msdn.com/b/ericlippert/archive/2011/05/19/optional-argument-corner-cases-part-four.aspx) by Eric Lippert discusses the problem in more detail.

> [Optional arguments can lead to] fairly serious versioning issue[s]. [...] The lesson here is to think carefully about the scenario with the long term in mind. If you suspect that you will be changing a default value and you want the callers to pick up the change without recompilation, don't use a default value in the argument list; make two overloads, where the one with fewer parameters calls the other.

## Using “var”

* Use `var` everywhere.

The justification is that the rest of this handbook encourages a style where

* methods are small
* parameters and variables are well-named

So the types, where relevant, will be obvious from the relatively small and local context.

The following examples show calls without using `var`.

```csharp
IList<Airplane> planes1 = new List<Airplane>();
IDataList<Airplane> planes2 = connection.GetList<Airplane>();
IDataList<Airplane> planes3 = hanger.GetAirplanes(connection);
```

In which cases is the type relevant or non-obvious? The following version using `var` only gains in clarity.

```csharp
var planes1 = new List<Airplane>();
var planes2 = connection.GetList<Airplane>();
var planes3 = hanger.GetAirplanes(connection);
```

## Using out and ref parameters

* Use `out` and `ref` parameters as little as possible.
* Instead of using many `out` and `ref` parameters, consider defining a `struct` instead; this improves the maintainability of the API.

TODO Add recommendations from chapter 9 here.


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



