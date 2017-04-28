# Usage

## Structure

### Assemblies

* Use a separate assembly to improve decoupling and reduce dependencies.
* Top-level application assemblies should have as little code as possible. Most logic goes in class libraries.

The example below illustrates the projects for a solution called “Calculator” with a _WPF_ application, a web-API application and a console application.
* `Calculator.Core`
* `Calculator.Core.Web`
* `Calculator.Core.Wpf`
* `Calculator.Web.Api`
* `Calculator.Wpf`
* `Calculator.Console`

The first three define libraries of functionality that is used by the next four applications. The server and console only use the `Calculator.Core` library whereas the _Winform_ and _WPF_ applications use their respective libraries. Separating the renderer-dependent code into a separate library makes it much easier to add another application using the same renderer but performing a slightly different task. Only highly application-dependent code should be defined directly in an application project.

### Files

* Place each type (classes, interfaces, enums, etc.) in a separate file.
* Each file should include a header, with the following form:
  ```csharp
  // <copyright file="Filename.cs" company="Encodo Systems AG">
  //   Copyright (c) 2017 Encodo Systems AG. All rights reserved.
  // </copyright>
  // <license>
  //   This file is subject to the terms and conditions defined in the 'LICENSE' file.
  // </license>
  ```
  A solution should include licensing conditions in a file named `LICENSE`.
* Namespace `using` statements should go at the very top of the file, just after the header and just before the namespace declaration.
* The namespaces at the top of the file should be in alphabetical order, with the exception that `System.*` assemblies come first.

### Namespaces

#### Referencing

* Do not use the global namespace.
* Avoid fully-qualified type names; use the `using` statement instead.
* Use aliases to resolve ambiguities.
* Use static classes where they improve clarity of code.

#### Defining

* Put abstract/high-level types in outer namespaces (e.g. `Encodo.Quino.Data`).
* Put concrete types in inner ones (e.g. `Encodo.Quino.Data.Ado`).
* Group types in specific namespaces.
* Avoid deep hierarchies, as they are more difficult to browse and understand.
* For general-use types, it's OK to use “Utilities”, “Core” or “General”.
* Refactor types into new namespaces if a clear presents itself.

## Types

### Classes

* Declare at most one field per line.
* Do not use public or protected fields; use properties instead.

#### Abstract Classes

* Define constructors for abstract classes as `protected`.
* Consider providing a partial implementation of an abstract class that handles some of the abstraction in a standard way; implementors can use this class as a base and avoid having to repeat code in their own implementations.

#### Static Classes

* Use static classes only for extension methods _or_ constants, but not both.
* Group functionality into logical static classes.

#### Inner Classes

* Inner classes should be `private` or `protected`.
* Inner types should not replace namespaces for organization.
* Use nested types if the inner type is logically within the other type (e.g. a `TableOfContents` class may have an `Options` inner class or a `Builder` inner class).
* Use an inner class to group private or protected constants.

Alternatives to consider:

* Consider using composition to inject the class instead, to allow customization and testing.
* Consider _local methods_ as an alternative to a `private` class.

#### Partial Classes

To control file size, partial classes can be useful to separate

* Generated code.
* `private` or `protected` inner classes.
* larger blocks of `private` or `protected` methods.

### Interfaces

#### Design

* Use interfaces to clearly define your API surface, separate from implementation.
* Use interfaces so that tests can mock behavior and test more precisely.
* Remove interfaces that are not used outside of testing code.
* Always create an interface for composed objects (i.e. those that are injected into other constructors) to ease testing and mocking.
* An interface should be as concise as possible. This allows consumers to precisely mock or override functionality.
* Remember the single-responsibility principle.
* If the standard implementation of a method can always be written in terms of other interface methods, consider defining an extension method for the interface instead. This is a nice way of providing a default implementation for all implementors.
* Avoid _marker_ interfaces. Attributes are a more appropriate way to mark types without _changing_ the type.

#### Usage

* Avoid similar interfaces that cause confusion as to which one should be used where. Re-use interfaces wherever  possible and appropriate.
* Provide a standard implementation or an abstract base. This provides both an implementation example and some protection from future changes to the interface.
* Use explicit interface implementation where appropriate to avoid expanding a class API unnecessarily.

### `structs`

Consider defining a structure instead of a class if most of the following conditions apply:

* Instances of the type are small (16 bytes or less) and commonly short-lived.
* The type is commonly embedded in other types.
* The type logically represents a single value and is similar to a primitive type, like an `int` or a `double`.
* The type is immutable.
* The type will not be boxed frequently.

Use the following rules when defining a `struct`.

* Avoid methods; at most, have only one or two methods other than equality overrides and operator overloads.
* Provide parameterized constructors for initialization.
* Overload operators and equality as expected; implement `IEquatable` instead of overriding `Equals` in order to avoid the negative performance impact of boxing and un-boxing the value.
* A `struct` should be valid when uninitialized so that consumers can declare an instance without calling a constructor.
* Public fields are allowed (even encouraged) for structures used to communicate with external APIs through unmanaged code.

## Generics

* Use generic collection types (e.g. use `IList<T>` instead of `IList`).
* Use generic constraints instead of casting or using the `is`-operator.
* Use interfaces as generic constraints wherever possible.
* When inheriting from both a generic and non-generic interface (e.g. `IEnumerable` and `IEnumerable<T>`), implement the non-generic version explicitly and implement it using the generic interface.

### `enums`

#### Design

* Use enumerations for strongly typed sets of values
* Use a singular name (e.g. `MigrationPhase` instead of `MigrationPhases`)
* Use enumerations for a list of constants that is not logically open-ended; otherwise, use a `static` class with constants so that consumers can extend the list.
* Use the default type of `Int32` whenever possible.
* Do not include sentinel values, such as `FirstValue` or `LastValue`.
* The first value in an enumeration is the default; make sure that the most appropriate simple enumeration value is listed first.
* Do not assign explicit values except to enforce specific values for storage in a database or to match an external API.

#### Usage

* Enumerations are like interfaces; be extremely careful of changing them when they are already included in code that is not under your control (e.g. used by a framework that is, in turn, used by external application code). If the enumeration must be changed, use the `ObsoleteAttribute` to mark members that are no longer in use.

#### Bit-sets

* Use the `[Flags]` attribute to make a bit-set instead of a simple enumeration.
* Use plural names for bit-sets.
* Assign explicit values for bit-sets in powers of two; use hexadecimal notation.
* The first value of a bit-set should always be `None` and equal to `0x00`.
* In bit-sets, feel free to include commonly-used aliases or combinations of flags to improve readability and maintainability. One such common value is `All`, which includes all available flags and, if included, should be defined last. For example:
  ```csharp
  [Flags]
  public enum QuerySections
  {
    None = 0x00,
    Select = 0x01,
    From = 0x02,
    Where = 0x04,
    OrderBy = 0x08,
    NotOrderBy = All & ~OrderBy,
    All = Select | From | Where | OrderBy,
  }
  ```
  The values `NotOrderBy` and `All` are aliases defined in terms of the other values. Note that the elements here are not aligned because it is expected that they will be documented, in which case column-alignment won’t make a difference in legibility.
* Avoid designing a bit-set when certain combinations of flags are invalid; in those cases, consider dividing the enumeration into two or more separate enumerations that are internally valid.

## Members

### Modifiers

* The visibility modifier is required for all types, methods and fields; this makes the intention explicit and consistent.
* The visibility keyword is always the first modifier.
* The `const` or `readonly` keyword, if present, comes immediately after the visibility modifier.
* The static keyword, if present, comes after the visibility modifier and readonly modifier.
  ```csharp
  private readonly static string DefaultDatabaseName = "admin";
  ```

#### `sealed`

* Do not declare protected or virtual members on sealed classes
* Avoid sealing classes unless there is a very good reason for doing so (e.g. to improve reflection performance).
* Consider sealing only selected members instead of sealing an entire class.
* Consider sealing members that you have overridden if you don’t want descendants to avoid your implementation.

#### `internal`

* Wherever possible, make implementation classes `internal` to reduce the surface area of the API.
* Prefer `private` and `protected` to `internal` methods.
* Instead of using `internal` to mean _assembly-local_, use composition to provide access to shared functionality

Most historical uses for `internal` methods can be implemented with other, better patterns. One use is to allow overriding of a method inside an assembly, but not outside. Two questions: why are you overriding instead of composing? And, if it's useful for your library or framework to be able to override, why deny this to consumers?

### Declaration Order

* Constructors, in descending order of complexity
* public constants
* public properties
* public methods
* protected constants
* protected properties
* protected methods
* private constants
* private properties
* private methods
* private fields

### Constants

* Declare all constants other than `0`, `1`, `true`, `false` and `null`.
* Use `true` and `false` only for assignment, never for comparison.
* Avoid passing `true` or `false` for parameters; use an `enum` or constants to impart meaning instead.
* If there is a logical connection between two constants, indicate this by making the initialization of one dependent on the other.
  ```csharp
  public const int DefaultCacheSize = 25;
  public const int DefaultGranularity = DefaultCacheSize / 5;
  ```
* Use `const` only when the value really is constant (e.g. `NumberDaysInWeek`); otherwise, use `readonly`.
* Use `readonly` as much as possible.

### Constructors

* Do not include a call to the default `base()`
* Avoid doing more than setting properties in a constructor; provide a well-named method on the class to perform any extra work after the object has been constructed.
* Avoid calling virtual methods from a constructor. The initialization order for constructors can lead to crashes. The example below illustrates this problem, where the override `CaffeineAddict.GoToWork()` uses Coffee before it has been initialized.
  ```csharp
  public abstract class Employee
  {
    public Employee()
    {
      Notify();
    }

    protected abstract void Notify();
  }

  public class CaffeineAddict : Employee
  {
    public CaffeineAddict([NotNull] Employee boss)
      : base()
    {
      if (boss == null) { throw new ArgumentNullException(nameof(beverage)); }

      Boss = boss;
    }

    [NotNull]
    public Employee Boss { get; }

    protected override void Notify()
    {
      // Crashes when called from the constructor
      Boss.Notify();
    }
  }
  ```
* Constructors should "funnel" so that initialization code is written only once. For example:
  ```csharp
  protected Query()
  {
    _restrictions = new List<IRestriction>();
    _sorts = new List<ISort>();
  }

  public Query([NotNull] IMetaClass model)
    : this()
  {
    if (model == null) { throw new ArgumentNullException(nameof(model)); }

    Model = model;
  }

  public Query([NotNull] IDataRelation relation)
    : this()
  {
    if (relation == null) { throw new ArgumentNullException(nameof(relation)); }

    Relation = relation;
  }
  ```

### Properties

* Prefer immutable properties.
* Prefer automatic properties.
* Prefer getter-only auto-properties.
* Prefer auto-property initializers.

#### Declaration

For example, the first version below is not only verbose, but B is mutable (within the class):

```csharp
// Do not use this style
public class A
{
  A()
  {
    B = 1;
  }

  int B { get; private set; }
}
```

This is also quite verbose, but B is now read-only:

```csharp
public class A
{
  int B
  {
    get { return _b; }
  }

  private readonly int _b = 1;
}
```

Finally, this is the recommended syntax (C#6 and higher):

```csharp
public class A
{
  int B { get; } = 1;
}
```

#### Commutativity

Properties should be commutative; that is, it should not matter in which order you set them. Avoid enforcing an ordering by using a method to execute code that you would want to execute from the property setter. The following example is incorrect because setting the password before setting the name causes a login failure.

```csharp
class SecuritySystem
{
  private string _userName;

  public string UserName
  {
    get { return _userName; }
    set { _userName = value; }
  }

  private int _password;

  public int Password
  {
    get { return _password; }
    set
    {
      _password = value;
      LogIn();
    }
  }

  protected void LogIn()
  {
    IPrincipal principal = Authenticate(UserName, Password);
  }

  private IPrincipal Authenticate(string UserName, int Password)
  {
    // Authenticate the user
  }
}
```

Instead, you should take the call LogIn() out of the setter for Password and make the method public, so the class can be used like this instead:

```csharp
var system = new SecuritySystem()
{
  Password = "knock knock";
  UserName = "Encodo";
}
system.LogIn();
```

In this case, Password can be set before the UserName without causing any problems.

### Indexers

* Avoid indexers. They are difficult to navigate, even with good tools. Use well-named methods instead (e.g. `Get*()` and `Set*()`).
* Provide an indexed property only if it really makes sense.
* Indexes should be 0-based.

### Methods

* Methods should not exceed a cyclomatic complexity of 20.
* Prefer private methods.
* Use the same names and positions for parameters shared by similar methods.
* Avoid returning `null` for methods that return collections or strings. Instead, return an empty collection (declare a static empty list) or an empty string (`String.Empty`).
* Methods that are explicitly left empty should be marked with NOP:
  ```csharp
  protected override void DoBeforeSave()
  {
    // NOP
  }
  ```

### Extension Methods

* Use extension methods for methods that can be defined in terms of public members of that interface.
* Do so only for methods for which the implementation is certain to be the same for all implementations. That is, do not restrict an implementation’s efficiency because an extension instead of interface method was used.
* Do not extend `object` or `string` in commonly used namespaces.
* Do not mix extension methods with other static methods. If a class contains extension methods, it should contain only extension methods and private support methods.
* Do not mix extension methods for different types in one class.
* Define useful, but more rarely used extension methods in a separate namespace or assembly to force callers to "opt in".

#### Bodies

* Do not _make decisions_ in extension methods. Instead, declare components and inject them where needed.
* Do not use static code that does _make decisions_ in extension methods.
* Do not use a global service locator in extension methods.
* Do not pass an IOC to extension methods.

### Parameters

* Use at most 5 parameters per method. Otherwise, use a `class`, `struct` or `tuple`.
* Use at most 1 `out` or `ref` parameter. Otherwise, return a `class`, `struct` or `tuple`.
* A `ref` and `out` parameter belongs at the end of the list of non-optional parameters.
* Use the same name for parameters in interface implementations or overrides.
* Avoid re-assigning the value of a parameter. Instead, use a local variable.
* A valid re-assignment is for nullable parameters, of the form shown below:
  ```csharp
  void Apply([NotNull] IMigrationPlan plan, [CanBeNull] ILogger logger = null)
  {
    if (plan != null) { throw new ArgumentNullException(nameof(plan)); }

    logger = logger ?? NullLogger.Default;

    // ...
  }
  ```

### Optional Parameters

* Do not use optional parameters in constructors.
* Do not use optional parameters that might change in public APIs; instead, use overloaded methods.
* Do not use more than one or two optional parameters, even for internal APIs.

This blog post [Optional argument corner cases, part four](http://blogs.msdn.com/b/ericlippert/archive/2011/05/19/optional-argument-corner-cases-part-four.aspx) by Eric Lippert discusses the problem in more detail.

> [Optional arguments can lead to] fairly serious versioning issue[s]. [...] The lesson here is to think carefully about the scenario with the long term in mind. If you suspect that you will be changing a default value and you want the callers to pick up the change without recompilation, don't use a default value in the argument list; make two overloads, where the one with fewer parameters calls the other.

### Expression-bodied Members

* Use expression-bodied members for simple properties and methods.
* The same rules apply as for any other expression; use a standard property or method body for complex logic.
* Do not use expression-bodied members for constructors and finalizers.

### `tuples`

* Do not use tuples with more than 3 fields.
* In C# 7 or higher, consider using a `Tuple<T, bool>` instead of an `out` parameter.
* In C# 6 or lower, use the Try* pattern because tuple fields cannot have names.
* Avoid `tuple` return types for public APIs

#### Naming

* Always name the parameters in a tuple.
* If the method returns a single constant tuple, then specify the names there (to keep the method declaration shorter). If there are several exit points, don't repeat the names in the literal tuples; instead, include the names only in the return-type declaration.
* Prefer external `var (first, second, third)` declaration to the internal one `(var first, var second, var third)`

#### Deconstruction

* Provide a custom `deconstruct` method for structs.
* Match the field order in a class's constructor, ToString override, and Deconstruct method.
* Use deconstruction where appropriate to consume tuples. If the tuple has named members, then you can just use it; otherwise, use deconstruction to assign the members to variables with logical names.

### Overloads

* Use overloads for methods that have similar behavior. Do not include parameter names in the method name. For example, the following is incorrect
  ```csharp
  void Update();
  void UpdateUsingQuery(IQuery query);
  void UpdateUsingSql(string sql);
  ```
  The overloaded version below reduces the perceived size of the API and makes it easier to understand.
  ```csharp
  void Update();
  void Update(IQuery query);
  void Update(string sql);
  ```
* Avoid putting a lot of logic in overloaded methods.
* Try to make overloads "funnel" to a single overload or other method.
* Make at most one overload `virtual`. For example:
  ```csharp
  public void Update()
  {
    Update(QueryTools.NullQuery);
  }

  public void UpdateUsingSql(string sql)
  {
    Update(new Query(sql));
  }

  public virtual void Update(IQuery query)
  {
    // Perform update
  }
  ```

### Virtual

* `virtual` is a code smell; consider _composition_ as an alternative.
* Avoid `public virtual` methods, but do not create an extra layer of method call either.
* If a method has logical pre-conditions or post-conditions, consider wrapping a `protected virtual` method in a `public` method, as shown below:
  ```csharp
  public void Update(IQuery query)
  {
    if (query == null) { throw new ArgumentNullException(nameof(query)); }
    if (!query.Valid) { throw new ArgumentException("Query is not valid.",  nameof(query)); }
    if (!query.Updatable) { throw new ArgumentException("Query is not updatable.",  nameof(query)); }

    DoUpdate(query);

    if (!query.UpToDate) { throw new ArgumentException("Query should have been updated.",  nameof(query)); }
  }

  protected virtual void DoUpdate(IQuery query)
  {
    // Perform update
  }
  ```
  Use a `Do` or `Internal` prefix for these methods.
* Wrap multiple parameters in an "arguments" class to avoid changing the signature when more data is needed in future versions.

### `new` Properties

* Do not use the `new` keyword to force overrides; use `override` instead or restructure the code to avoid it.

### Event Handlers

Be aware of the following when raising events.

* Event handlers can affect performance.
* Event handlers can change the calling object.
* Event handlers can throw exceptions.
* Event handlers are not guaranteed to run in the calling thread.

#### Alternatives

* Do not use event handlers other than in legacy code (e.g. Winform)
* For user interfaces, use the MVVM pattern instead
* For back-end objects, use messaging or event-aggregator patterns

#### Rules

* Declare events using the `event` keyword.
* Do not use delegate members.
* Use delegate inference instead of writing `new EventHandler(…)`.
* Declare events with `EventHandler<T>`.
* An event has two parameters named `sender` of type `object` and `args` with an event-specific type.
* Do not allow `null` for either the `sender` or the `args` parameters.
* Use `EventArgs` as the base class for custom arguments.
* Use `CancelEventArgs` as the base class if you need to be able to cancel an event.
* Custom arguments should include only properties, but no logic.

#### Race conditions

To avoid null-reference exceptions, get a reference to the handler in a local variable before checking it and calling it. The so-called "elvis" operator in C# 6 and higher is recommended.

```csharp
protected virtual void RaiseMessageDispatched()
{
  MessageDispatched?.(this, EventArgs.Empty);
}
```

For C# 5 and lower, use:

```csharp
protected virtual void RaiseMessageDispatched()
{
  EventHandler handler = MessageDispatched;
  if (handler != null)
  {
    handler(this, EventArgs.Empty);
  }
}
```

The following code is an example of a simple event handler and receiver.

```csharp
public class Safe
{
  public event EventHandler Locked;

  public void Lock()
  {
    // Perform work

    RaiseLocked(EventArgs.Empty);
  }

  protected virtual void RaiseLocked(EventArgs args)
  {
    Locked?.(this, args);
  }
}

public static class StoreManager
{
  private static void SendMailAboutSafe(object sender, EventArgs args)
  {
    // Respond to the event
  }

  public static void TestSafe()
  {
    Safe safe = new Safe();
    safe.Locked += SendMailAboutSafe;
    safe.Lock();
  }
}
```

### Operators

#### Caveats

* Avoid overloading operators in general; it's not appropriate for most problem domains.
* Do not override the `==`-operator for reference types; instead, override the `Equals()` method to avoid redefining reference equality.
* Do not provide a conversion operator unless it can be logically expected by consumers of the API.

#### Recommendations

* If an operator is needed, re-use operator conventions from other languages or the problem domain of the API (e.g. mathematical operators)
* If you do override Equals(), you must also override `GetHashCode()`.
* If you do override the == operator, consider overriding the other comparison operators (!=, <, <=, >, >=) as well.
* You should return `false` from the `Equals()` function if the objects cannot be compared. However, if they are different types, you may throw an exception.
* Do not mix and match conversion operators and types. The type `DynamicString` can convert to `System.String` but should not convert to `System.Int32`. Instead, use a constructor to initialize from types that are not in the same domain.
* Use `implicit` operators _only_ where the conversion _cannot_ result in a loss of data or an exception.
* Use an `explicit` operator where data-loss or exceptions are possible.

### `ref` Returns and Properties

`ref` returns are a feature that can improve memory-management in applications that uses larger structures. Taking references avoids copying values where not necessary.

* Use ref properties for mutable structs.
* Do not use ref properties for immutable structs or read-only classes.

## Statements and Expressions

### `base`

* Use `base` only from a constructor or to call a predecessor method.
* You may only call the `base` of the method being executed; do not call other `base` methods. In the following example, the call to `CheckProcess()` is not allowed, whereas the call to `RunProcess()` is.
  ```csharp
  public override void RunProcess()
  {
    base.CheckProcess();  // Not allowed
    base.RunProcess();
  }
  ```

### `this`

* Use `this` only when referring to other constructors.
* Do not use `this` to resolve name-clashes; instead, change one of the conflicting names.

### Value Types

* Always use the lower-case primitive type.
  * Use `int` instead of `Int32`
  * Use `string` instead of `String`
  * Use `bool` instead of `Boolean`
  * Use `short` instead of `Int16`
  * Use `byte` instead of `Byte`
  * Use `long` instead of `Int64`
  * And so on.

### Strings

* Use `string.Empty` instead of `“”`.
* Use `string.IsNullOrEmpty` instead of `s == null` or `s.Length == 0`.

#### Concatenation

* Prefer string-interpolation for C#6 or higher.
* Prefer `string.Format` for C#5 or lower.
* Use string-concatenation or `string.Concat` only if you have identified a performance bottleneck.
* Use a `StringBuilder` for more complex situations or when concatenation occurs over multiple statements.

### Interpolation

* Prefer embedding variables rather than expressions.
* Avoid embedding longer expressions.

The following example uses short expressions and is legible.

```csharp
var s = $"The total [{total + shipping}] is higher than the balance [{balance - fees}].";
```

However, the following interpolated string isn't very easy to read.

```csharp
var s = $"The total [{allOrders.Sum(o => o.Total + (o.Tax * o.Vat.Rate)) + shipping}] is higher than the balance [{accounts.Sum(a => a.Balance) - fees}].";
```

Instead, extract variables to reduce complexity to the previous formulation.

```csharp
var total = allOrders.Sum(o => o.Total + (o.Tax * o.Vat.Rate));
var balance = accounts.Sum(a => a.Balance);
var s = $"The total [{total + shipping}] is higher than the balance [{balance - fees}].";
```

### `nameof`

* Use `nameof` for passing names to `ArgumentExceptions`.
* Use `nameof` wherever possible to avoid constant strings.

### Resource Strings

* Use resources for all localizable strings.
* Do not use resources for strings that will not be localized (e.g. log messages)
* Resource identifiers follow the same rules as all other identifiers.
* Do not waste time localizing strings until code is reviewed and stable.

### Floating Point and Integral Types

* Be careful when comparing floating-point constants; unless you are using `decimals`, the representation will have limited precision and can lead to subtle bugs.
* One exception to this rule is when you are comparing constants of fixed, known value (like `0.0` or `1.0`, but not `3.14`).
* Be careful when casting representations with different sizes (e.g. `long` to `int`).
* Use the literal `_` as a reasonable separator (e.g. to delineate hex groups or as a thousands-separator).

### Local Variables

* Use `var` and initialization wherever possible.
* Declare local variables individually.
* Initialize a local variable on the same line as the declaration
* Use standard line-breaking rules outline elsewhere for longer, fluent initialization.

### Local Functions

* Use local functions for short private methods that are used only once.
* If a local function body must be repeated, then use a private method instead.
* Use local functions instead of anonymous functions.
* Use local iterators when returning an IEnumerator when parameters need to be validated.
* Put local functions at the beginning or end of its containing body.

### `var`

* Use `var` everywhere.

The justification is that the rest of this handbook encourages a style where:

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

## `out` variables

* Use `out`-parameter declaration to define parameters.
  ```csharp
  if (list.TryGetValue("One", var out item))
  {
    // use 'item'
  }
  ```
  For C# 5 and older, use the standalone variable declaration.
  ```csharp
  Item item;
  if (list.TryGetValue("One", out item))
  {
    // use 'item'
  }
  ```

### Loops

* Do not change the loop variable of a for-loop.
* Update while loop variables either at the beginning or the end of the loop.
* Keep loop bodies short; avoid excessive nesting.

### Conditions

* Do not compare to `true` or `false`.
* Use parentheses only if the precedence isn't relatively obvious
* Use _StyleCop_ or _ReSharper_ to indicate where parentheses are needed and stick to it.
* Initialize Boolean values with simple expressions rather than using an if-statement.
  ```csharp
  bool needsUpdate = Count > 0 && Objects.Any(o => o.Modified);
  ```
* Always use brackets for flow-control blocks (`switch`, `if`, `while`, `for`, etc.)
* Do not add useless `else` blocks. An `if` statement may stand alone and an `else if` statement may be the last condition.
  ```csharp
  if (a == b)
  {
    // Do something
  }
  else if (a > b)
  {
    // Do something else
  }
  // No final "else" required
  ```
* Do not force really complicated logic into an `if` statement; instead, use local variables to make the intent clearer. For example, imagine we have a lesson planner and want to find all unsaved lessons that are either unscheduled or are scheduled within a given time-frame. The following condition is too long and complicated to interpret quickly:
  ```csharp
  if (!lesson.Stored && ((StartTime <= lesson.StartTime && lesson.EndTime <= EndTime) || !lesson.Scheduled))
  {
    // Do something with the lesson
  }
  ```
  Even trying to apply the line-breaking rules results in an unreadable mess:
  ```csharp
  if (!lesson.Stored &&
    ((StartTime <= lesson.StartTime && lesson.EndTime <= EndTime) ||
    ! lesson.Scheduled))
  {
    // Do something with the lesson
  }
  ```
  Even with this valiant effort, the intent of the ||-operator is difficult to discern. With local variables, however, the logic is much clearer:
  ```csharp
  bool lessonInTimeSpan = StartTime <= lesson.StartTime && lesson.EndTime <= EndTime;
  if (!lesson.Stored && (lessonInTimeSpan || !lesson.Scheduled))
  {
    // Do something with the lesson
  }
  ```

### `switch` Statements

* Use `throw new UnexpectedEnumException(value)` in the `default` branch. This is more semantically correct than `InvalidEnumArgumentException`, which does not allow you to indicate the unexpected value _and_ erroneously suggests that the value was invalid.
* The following code is correct:
  ```csharp
  switch (type)
  {
    case DatabaseType.PostgreSql:
      return new PostgreSqlMetaDatabase();
    case DatabaseType.SqlServer:
      return new SqlServerMetaDatabase();
    case DatabaseType.SQLite:
      return new SQLiteMetaDatabase();
    default:
      throw new UnexpectedEnumException(value);
  }
  ```
* The `default` label must always be the last label in the statement. In C# 7, the default label is always interpreted last anyway.

### `continue` Statements

* Do not use `continue`.
* The following example is not allowed.
  ```csharp
  foreach (var search in searches)
  {
    if (!search.Path.Contains("CN="))
    {
      continue;
    }

    // Work with valid searches
  }
  ```
  Instead, use a condition to filter elements.
  ```csharp
  foreach (var search in searches)
  {
    if (search.Path.Contains("CN="))
    {
      // Work with valid searches
    }
  }
  ```
  Even better, use `Where()` to filter elements.
  ```csharp
  foreach (var search in searches.Where(s => s.Path.Contains("CN=")))
  {
    // Work with valid searches
  }
  ```

### `return` Statements

* Prefer multiple return statements to local variables and nesting.
  ```csharp
  if (specialTaxRateApplies)
  {
    return CalculateSpecialTaxRate();
  }

  return CalculateRegularTaxRate();
  ```
* Compose smaller methods to avoid local variables for return values. For example, the following method uses a local variable rather than multiple returns.
  ```csharp
  bool result;

  if (SomeConditionHolds())
  {
    PerformOperationsForSomeCondition();

    result = false;
  }
  else
  {
    PerformOtherOperations();

    if (SomeOtherConditionHolds())
    {
      PerformOperationsForOtherCondition();

      result = false;
    }
    else
    {
      PerformFallbackOperations();

      result = true;
    }
  }

  return result;
  ```
  This method can be rewritten to return the value instead.
  ```csharp
  if (SomeConditionHolds())
  {
    PerformOperationsForSomeCondition();

    return false;
  }

  PerformOtherOperations();

  if (SomeOtherConditionHolds())
  {
    PerformOperationsForOtherCondition();

    return false;
  }

  PerformFallbackOperations();

  return true;
  ```

* The only code that may follow the last `return` statement is the body of an exception handler.
  ```csharp
  try
  {
    // Perform operations

    return true;
  }
  catch (Exception exception)
  {
    throw new DirectoryAuthenticatorException(exception);
  }
  ```

### Pattern-matching

* The same rules apply for `when` expressions as for other conditions: short expressions are fine; extract more complex logic to local or private methods.

### `goto` Statements

* Do not use `goto`.

### `unsafe` Blocks

* Do not use `unsafe`.

### Ternary and Coalescing Operators

* Use these operators for simple expressions and results.
* Do not use these operators with long conditions and values. Instead, use local variables and/or standard conditional statements.

### Null-conditional Operator

* Use the `?.`-operator only when no handling for `null` cases is required.
* Most code should not use this operator; instead, enforce non-null values.

For example, the following example is only allowed when the data comes from a dynamic source (e.g. JSON).

```csharp
company.People?[0]?.ContactInfo?.BusinessAddress.Street
```

If the data is not dynamic, then `People`, `ContactInfo` and `BusinessAddress` should never be `null`.

#### `throw`-Expressions

* Only use `throw`-expressions at the end of an expression.
* Do not use `throw`-expressions anywhere else in an compound expression.
* Do not use `throw`-expressions as actual arguments.

### Lambdas

* Do not make overly-complex lambda expressions; instead, define a method or use a delegate.

### System.Linq

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

### Casting

* Use a direct cast if you are sure of the type.
  ```csharp
  ((IWeapon)item).Fire();
  ```
* Use the `is`-operator when _testing_ but not _using_ the result of the cast.
  ```csharp
  return item is IWeapon;
  ```
* To use the result of the cast, use an `is`-expression in C# 7 and higher.
  ```csharp
  if (item == null) { throw new ArgumentNullException(nameof(item)); }

  if (item is IWeapon weapon)
  {
    return weapon.Fire();
  }

  return NullTurn.Default;
   ```
* Use a `switch` statement to match more than one or two patterns. Keep the argument precondition separate (even though it _could_ be the penultimate `case`).
  ```csharp
  if (item == null) { throw new ArgumentNullException(nameof(item)); }

  switch (item)
  {
    case IWeapon weapon:
      return weapon.Fire();
    case IMagic magic:
      return magic.Cast();
    default:
      return NullTurn.Default;
  }
  ```
* In C# 6 and lower, use the `as`-operator.
  ```csharp
  if (item == null) { throw new ArgumentNullException(nameof(item)); }

  var weapon = item as IWeapon;
  if (weapon != null)
  {
    return weapon.Fire();
  }

  var magic = item as IMagic;
  if (magic != null)
  {
    return magic.Cast();
  }

  return NullTurn.Default;
  ```

### `checked`

* Enable range-checking during development and debugging.
* Disable range-checking in release builds only if there is a valid performance reason for doing so.
* Use explicit `checked` blocks for overflow- and underflow-prone operations. I.e. if there was a range-checking problem at some point, the block should be marked with a `checked` block. This guarantees checking for these blocks even when range-checking is disabled.

### Compiler Variables

* Avoid using compiler variables.
* Avoid using `#define` in the code; use a compiler define in the project settings instead.
* Avoid suppressing compiler warnings.

#### The [Conditional] Attribute

Use the `ConditionalAttribute` instead of the `#ifdef`/`#endif` pair wherever possible (i.e. for methods or classes).

```csharp
public class SomeClass
{
  [Conditional("TRACE_ON")]
  public static void Msg(string msg)
  {
    Console.WriteLine(msg);
  }
}
```

#### \#if/#else/#endif

For other conditional compilation, use a static method in a static class instead of scattering conditional options throughout the code.

```csharp
public static class EncodoCompilerOptions
{
  public static bool DeveloperBuild()
  {
#if ENCODO_DEVELOPER
    return true;
#else
    return false;
#endif
  }
}
```

This approach has the following advantages:

* The compiler checks all code paths instead of just the one satisfying the current options; this avoids unknowingly retaining incompatible code in a library or application.
* Code formatting and indenting is not broken up by (possibly overlapping) compile conditions; the name `EncodoCompilerOptions` makes the connection to the compiler obvious enough.
* The compiler option is referenced only once, avoiding situations in which some code uses one compiler option (e.g. `ENCODO_DEVELOPER`) and other code uses another, misspelled option (e.g. `ENCODE_DEVELOPER`).

### Comments

#### Styles

* Use the single-line comment style—`//`—to indicate a comment.
* Use four slashes —`////`—to indicate a single line of code that has been temporarily commented.
* Use the multi-line comment style—`/*` … `*/`—to indicate a commented-out block of code. Do not push these comments to the master branch.
* Consider using a compiler variable to define a non-compiling block of code; this practice avoids misusing a comment.
  ```csharp
  #if FALSE
        // commented code block
  #endif
  ```
* Use the single-line comment style with `TODO` to indicate an issue that must be addressed. Before a check-in, these issues must either be addressed or documented in the issue tracker, adding the URL of the issue to the TODO as follows:
  ```csharp
  // TODO http://issue-tracker.encodo.com/?id=5647: Title of the issue in the issue tracker
  ```

#### Placement

* Longer comments should always precede the line being commented. Separate multi-line comments with an additional newline before the code.
* Short comments may appear to the right of the code being commented, but only for lines ending in semicolon (i.e. marking the end of a statement). For example:
  ```csharp
  int Granularity = Size / 5; // More than 50% is not valid!
  ```
* Comments on the same line as code should _never_ be wrapped to multiple lines.

#### Use Cases

* Use comments to explain algorithms or tricky bits that aren't immediately obvious from a quick read.
* Use comments to indicate where a hard-won bug-fix was added; if possible, include a reference to a URL in an issue tracker.
* Use comments to indicate assumptions not already evident from assertions or thrown exceptions.
* Comments are in US-English; prefer a short style that gets right to the point.
* A comment need not be a full, grammatically-correct sentence. For example, the following comment is too long
  ```csharp
  // Using a granularity that is more than 50% of the size is not valid!
  int Granularity = Size / 5;
  ```
  Instead, you should stick to the essentials so that the warning is immediately clear:
  ```csharp
  int Granularity = Size / 5; // More than 50% is not valid!
  ```
* Comments should not explain the obvious. In the following example, the comment is superfluous.
  ```csharp
  public const int Granularity = Size / 5; // granularity is 20% of size
  ```
