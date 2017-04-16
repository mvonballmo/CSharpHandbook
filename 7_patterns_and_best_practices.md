# Patterns & Best Practices

## Safe Programming

* Use static typing wherever possible.
* Make data immutable wherever possible.
* Make references non-nullable wherever possible.
* Make methods pure wherever possible.
* Do not use the `new` keyword to force overrides; use `override` instead or restructure the code to avoid it.
* Do not use `goto`.
* Do not use `unsafe`. An exception is when writing external libraries or high-performance code.
* Always test parameters, local variables and fields that can be `null`.
* Do not re-use local variable names, even though the scoping rules are well-defined and allow it. This prevents surprising effects when the variable in the inner scope is removed and the code continues to compile because the variable in the outer scope is still valid.
* Do not modify a variable with a prefix or suffix operator more than once in an expression. The following statement is not allowed:
  ```csharp
  items[propIndex++] = ++propIndex;
  ```
* Use the `[NotNull]` attribute for parameters, fields and results. Enforce it with a runtime check.

## Side Effects

A side effect is a change in an object as a result of reading a property or calling a method that causes the result of the property or method to be different when called again.

* Prefer pure methods.
* Void methods have side effects by definition.
* Writing a property must cause a side effect.
* Reading a property should not cause a side effect. An exception is lazy-initialization to cache the result.
* Avoid writing methods that return results _and_ cause side effects. An exception is lazy-initialization to cache the result.

## Null-handling

* Instead of allowing `null` for a parameter, avoid null-checks with a null implementation.
  ```csharp
  interface ILogger
  {
    bool Log(string message);
  }

  class NullLogger : ILogger
  {
    void Log(string message)
    {
      // NOP
    }
  }
  ```

## Casting

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

## `return` statements

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

### `continue` statements

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

## Object Lifetime

* Use a `using` statement to precisely delineate the lifetime of `IDisposable` objects.
* Use `try`/`finally` blocks to manage other state (e.g. executing a matching `EndUpdate()` for a `BeginUpdate()`)
* Do not set local variables to `null`. They will be automatically de-referenced and cleaned up.

### `IDisposable`

* Implement `IDisposable` if your object uses disposable objects or system resources.
* Be careful about making your interfaces `IDisposable`; that pattern is a leaky abstraction and can be quite invasive.
* Implement `Dispose()` so that it can be safely called multiple times (see example below).
* Don't hide `Dispose()` in an explicit implementation. It confuses callers unnecessarily. If there is a more appropriate domain-specific name for `Dispose`, feel free to make a synonym.

### `Finalize`

* There are performance penalties for implementing `Finalize`. Implement it only if you actually have costly external resources.
* Call the `GC.SuppressFinalize` method from `Dispose` to prevent `Finalize` from being executed if `Dispose()` has already been called.
* `Finalize` should include only calls to `Dispose` and `base.Finalize()`.
* `Finalize` should never be `public`

### Example

The following class expects the caller to use a non-standard pattern to avoid holding open a file handle.

```csharp
public class Weapon
{
  public Load(string file)
  {
    _file = File.OpenRead(file);
  }

  // Aim(), Fire(), etc.

  public EjectShell()
  {
    _file.Dispose();
  }

  private File _file;
}
```
Instead, make `Weapon` disposable. The following implementation follows the recommended pattern. R# can help you create this pattern.

> _Note that `_file` is set to `null` so that `Dispose` can be called multiple notes, not to clear the reference._

```csharp
public class Weapon : IDisposable
{
  public Weapon(string file)
  {
    _file = File.OpenRead(file);
  }

  // Aim(), Fire(), etc.

  public Dispose()
  {
    Dispose(true);
    GC.SuppressFinalize(this);
  }

  protected virtual void Dispose(bool disposing)
  {
    if (disposing)
    {
      if (_file != null)
      {
        _file.Dispose();
        _file = null;
      }
    }
  }

  private File _file;
}
```

Using the standard pattern, R# and Code Analysis will detect when an `IDisposable` object is not disposed of properly.

### Destructors

* Avoid using destructors because they incur a performance penalty in the garbage collector.
* Do not access other object references inside the destructor as those objects may already have been garbage-collected (there is no guaranteed order-of-destruction in the IL or .NET runtime).

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

## Using Event Handlers

Be aware of the following when raising events.

* Event handlers can affect performance.
* Event handlers can change the calling object.
* Event handlers can throw exceptions.
* Event handlers are not guaranteed to run in the calling thread.
* To avoid null-reference exceptions, get a reference to the handler in a local variable before checking it and calling it. For example:
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
  The following code is an example of a simple event handler and receiver with all of the Encodo style conventions applied.
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

###	Lazy Evaluation

`IEnumerable<T>` sequences are evaluated lazily. ReSharper will warn of multiple enumeration.

###	Capturing Unstable Variables/”Access to Modified Closure”

You can accidentally change the value of a captured variable before the sequence is evaluated. Since _ReSharper_ will complain about this behavior even when it does not cause unwanted side-effects, it is important to understand which cases are actually problematic.

```csharp
var data = new[] { "foo", "bar", "bla" };
var otherData = new[] { "bla", "blu" };
var overlapData = new List<string>();

foreach (var d in data)
{
  if (otherData.Where(od => od == d).Any())
  {
    overlapData.Add(d);
  }
}

Assert.That(overlapData.Count, Is.EqualTo(1)); // "bla"
```

The reference to the variable `d` will be flagged by _ReSharper_ and marked as an _“access to a modified closure”_. This indicates that a variable referenced—or “captured”—by the lambda expression—closure—will have the last value assigned to it rather than the value that was assigned to it when the lambda was created.

In the example above, the lambda is created with the first value in the sequence, but since we only use the lambda once, and then always before the variable has been changed, we don’t have to worry about side-effects. _ReSharper_ can only detect that a variable referenced in a closure is being changed within its scope.

Even though there isn’t a problem in this case, rewrite the `foreach`-statement above as follows to eliminate the _access to modified closure_ warning.

```csharp
var data = new[] { "foo", "bar", "bla" };
var otherData = new[] { "bla", "blu" };
var overlapData = data.Where(d => otherData.Where(od => od == d).Any()).ToList();

Assert.That(overlapData.Count, Is.EqualTo(1)); // "bla"
```

Finally, use library functionality wherever possible. In this case, we should use `Intersect` to calculate the overlap (intersection).

```csharp
var data = new[] { "foo", "bar", "bla" };
var otherData = new[] { "bla", "blu" };
var overlapData = data.Intersect(otherData).ToList();

Assert.That(overlapData.Count, Is.EqualTo(1)); // "bla"
```

Remember to be aware of how items are compared. The `Intesects` method above compares using `Equals`, not reference-equality.

The following example does not yield the expected result:

```csharp
var data = new[] { "foo", "bar", "bla" };

var threshold = 2;
var twoLetterWords = data.Where(d => d.Length == threshold);

threshold = 3;
var threeLetterWords = data.Where(d => d.Length == threshold);

Assert.That(twoLetterWords.Count(), Is.EqualTo(0));
Assert.That(threeLetterWords.Count(), Is.EqualTo(3));
```

The lambda in `twoLetterWords` _references_ `threshold`, which is then changed before the lambda is evaluated with `Count()`. There is nothing wrong with this code, but the results can be surprising. Use `ToList()` to evaluate the lambda in `twoLetterWords` _before_ the threshold is changed.

```csharp
var data = new[] { "foo", "bar", "bla" };
var threshold = 2;
var twoLetterWords = data.Where(d => d.Length == threshold).ToList();

threshold = 3;
var threeLetterWords = data.Where(d => d.Length == threshold);

Assert.That(twoLetterWords.Count(), Is.EqualTo(0));
Assert.That(threeLetterWords.Count(), Is.EqualTo(3));
```

###	Enumeration Run-time Errors

Changing a sequence during enumeration causes a runtime error.

The following code will fail whenever `data` contains an element for which `IsEmpty` returns `true`.

```csharp
foreach (var d in data.Where(d => d.IsEmpty))
{
  data.Remove(d);
}
```

To avoid this problem, use an in-memory copy of the sequence instead. A good practice is to use `ToList()` to create the copy and to call it in the `foreach` statement so that it's clear why it's being used.

```csharp
foreach (var d in data.Where(d => d.IsEmpty).ToList())
{
  data.Remove(d);
}
```

###	More Unexpected Results

Suppose, in the example above, that we also want to know how many elements were empty. Let's start by extracting `emptyElements` to a variable.

```csharp
var emptyElements = data.Where(d => d.IsEmpty);
foreach (var d in emptyElements.ToList())
{
  data.Remove(d);
}

return emptyElements.Count();
```

Since `emptyElements` is evaluated lazily, the call to `Count()` to return the result will evaluate the iterator again, producing a sequence that is now empty—because the `foreach`-statement removed them all from data. The code above will always return zero.

A more critical look at the code above would discover that the `emptyElements` iterator is triggered twice: by the call to `ToList()` and `Count()` (ReSharper will helpfully indicate this with an inspection). Both `ToList()` and `Count()` logically iterate the entire sequence.

To fix the problem, we lift the call to `ToList()` out of the `foreach` statement and into the variable.

```csharp
var emptyElements = data.Where(d => d.IsEmpty).ToList();
foreach (var d in emptyElements)
{
  data.Remove(d);
}

return emptyElements.Count;
```

We can eliminate the `foreach` by directly re-assigning `data`, as shown below.

```csharp
var dataCount = data.Count;
var data = data.Where(d => !d.IsEmpty).ToList();

return dataCount – data.Count;
```

The first algorithm is more efficient when the majority of item in `data` are empty. The second algorithm is more efficient when the majority is non-empty.

## Designing Types

Design types so that the caller cannot use them incorrectly.

The example below shows a typical class indicates usage in documentation rather than the API.

```csharp
/// <remarks>
/// Make sure to set the <see cref="ServerName">, <see cref="UserName">
/// and <see cref="Password"> before calling <see cref="Connect">. Call <see cref="Connect"> before calling <see cref="LogIn"> and <see cref="RunTask">.
/// </remarks>
public interface IBackEnd
{
  string ServerName { get; set; }
  string UserName { get; set; }
  string Password { get; set; }
  void Connect();
  void LogIn();
  void RunTask([NotNull] ITask task);
}

internal static Main()
{
  var backEnd = CreateBackEnd();
  var tasks = GetTasks();

  backEnd.ServerName = GetServerName();
  backEnd.UserName = GetUserName();
  backEnd.Password = GetPassword();
  backEnd.Connect();
  backEnd.LogIn();

  foreach (var task in tasks)
  {
    backEnd.RunTask(task);
  }
}
```

The block in `Main()` _should_ always look the same, but the pattern is not enforced.

Instead of a single `IBackEnd` type with multiple methods, move the configuration parameters to separate objects and define _three_ interfaces, each of which has a single responsibility:

```csharp
interface IDisconnectedBackEnd
{
  [NotNull]
  IConnectedBackEnd Connect([NotNull] IConnectionSettings settings);
}

interface IConnectedBackEnd
{
  [NotNull]
  ILoggedInBackEnd LogIn([NotNull] IUser user);
}

interface ILoggedInBackEnd
{
  void RunTask([NotNull] ITask task);
}

internal static Main()
{
  var settings = GetConnectionSettings();
  var user = GetUser();
  var tasks = GetTasks();

  var backEnd =
    CreateDisconnectedBackEnd()
    .Connect(settings)
    .LogIn(user);

  foreach (var task in tasks)
  {
    backEnd.RunTask(task);
  }
}
```

In this case, the code in `Main()` will still always look the same, _but it can now no longer take any other shape_. The caller _cannot_ call `LogIn()` without having called `Connect()` first. With these types, a method can require a `IConnectedBackEnd` or `ILoggedInBackEnd` rather than a generic `IBackEnd` in an unknown state.

For example, we can extract a method for running tasks where the type of the parameter enforces the state of the back end.

```csharp
internal static Main()
{
  var settings = GetConnectionSettings();
  var user = GetUser();
  var tasks = GetTasks();

  var backEnd =
    CreateDisconnectedBackEnd()
    .Connect(settings)
    .LogIn(user);

  RunTasks(backEnd, tasks);
}

private static void RunTasks([NotNull] ILoggedInBackEnd backEnd, [NotNull] IEnumerable<ITask> tasks)
{
  if (backEnd != null) { throw new ArgumentNullException(nameof(backEnd)); }
  if (tasks != null) { throw new ArgumentNullException(nameof(tasks)); }

  foreach (var task in tasks)
  {
    backEnd.RunTask(task);
  }
}
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

* Use `var` when the type or intent is already clear from the context; otherwise, specify the type to improve understandability.
* Removing too many types not only reduces readability, but also reduces navigability (i.e. one can no longer navigate to related types using <kbd>F12</kbd>).

usage of `var`:

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

## Restricting Access with Interfaces

One advantage in using interfaces over abstract classes is that interfaces can enforce immutability even though the backing implementation is mutable.

The following example illustrates this principle for properties:

```csharp
interface IMaker
{
  bool Enabled { get; }
}

class Maker : IMaker
{
  bool Enabled { get; set; }
}
```

The principle extends to making read-only sequences as well, using `IEnumerable<T>` in the interface and exposing `IList<T>` in the backing class, using explicit interface implementation, as shown below:

```csharp
interface IProcessedCommands
{
  IEnumerable<ISwitchCommand> SwitchCommands { get; }
}

class ProcessedCommands
{
  IEnumerable<ISwitchCommand> IProcessedCommands.SwitchCommands
  {
    get { return SwitchCommands.ToList(); }
  }

  IList<ISwitchCommand> SwitchCommands { get; private set; }
}
```

In this way, the client of the interface cannot call `Add()` or `Remove()` on the list of commands, but the provider has the convenience of doing so as it prepares the backing object.

Using Linq, the client is free to convert the result to a list using `ToList()`, but will create its own copy, leaving the `SwitchCommands` on the object represented by the interface unaffected.

Note, though, that the implementation of `IProcessedCommands.SwitchCommands` calls `ToList()` to avoid passing its internal representation back to the caller. If it didn't do this, then caller could alter the internal state with the following code.

```csharp
var processedCommands = GetProcessedCommands();
var switches = processedCommands.SwitchCommands as IList<ISwitchCommand>;
if (switches != null)
{
  switches.Clear();
}
```

## Error Handling

###	Strategies

* Prefer exceptions to return codes.
* Use a return code only where all results are valid.
* In those cases, prefer `Tuple` results to pure return codes.
* Unless you have a public API, where the `Tuple` should be replaced with a concrete type.
* Use the `Try*`-pattern (illustrated below) to encapsulate methods that can fail.
* If errors/warnings are expected, then use an `ILogger` or similar construct to record those warnings rather than throwing and multiple catching exceptions.

###	Error Messages

* Be as specific as possible when throwing exceptions. Let the caller/catcher elide detail as needed.
* If data included in a message _could_ be empty, consider wrapping it in braces so that the message is clear even when the data is empty. For example, the following code might produce a confusing error message:
  ```csharp
  var message = $"The following expression {data} could not be parsed.";
  ```
  If `data` is empty, the caller (user or developer) sees only _The following expression could not be parsed._ If the message was instead defined as follows:
  ```csharp
  var message = $"The following expression {data} could not be parsed.";
  ```
  Then the caller sees _The following expression [] could not be parsed._ In this case it's more obvious that the expression was empty.
* The message should include as much information as possible, though it is highly recommended to provide both a longer, low-level message (for logging and debugging) and a shorter, high-level message for presenting to the user.
* Error messages should describe how the user or developer can avoid the exception.
* The standards for error messages are the same as for any other text that might be shown to a user; use grammatically correct English (though translations may also be provided).
* Messages should be complete sentences and end in a period.
* Do not use question marks or exclamation points.
* Use the error level or exception type to indicate severity or let the handler of the exception determine what level of urgency to assign.
* Lower-level, developer messages should be logged to sources that are available only to those with permission to view lower-level details.
* Applications should avoid showing sensitive information to end-users. This applies especially to web applications, which must never show exception traces in production code. The exact message returned by an exception can vary depending on the permission level of the executing code.

###	The Try* Pattern

The Try\* pattern is used by the .NET framework. Generally, Try\*-methods accept an `out` parameter on which to attempt an operation, returning `true` if successful.

* If you provide a method using the Try* pattern (), you should also provide a non-try-based, exception-throwing variant as well. The exception-throwing variant should call the Try* variant, never the other way around.
  ```csharp
  public IExpression Parse(string text)
  {
    if (TryParse(text, var out expression))
    {
      return expression;
    }

    throw new InvalidOperationException($"The expression [{text}] contains a syntax error.");
  }

  public bool TryParse(string text, out IExpression result)
  {
    if (text == "true")
    {
      expression = BooleanExpression(true);

      return true;
    }

    if (text == "false")
    {
      expression = BooleanExpression(false);

      return true;
    }

    expression = null;

    return false;
  }
  ```

## Exceptions

### Categories

Exceptions come in two categories: exceptions and bugs. Bugs should result in aborting the application because it is not possible to reason about software that has a bug.

Almost all exceptions in C# are actually bugs.

The following exceptions are bugs:

* `ArgumentException` and descendants
* `NullReferenceException`
* `ClassCastException`
* `OutOfMemoryException`
* `StackOverflowException`
* `InvalidOperationException`

###	Defining Exceptions

* Re-use exception types wherever possible.
* Do not simply create an exception type for every different error.
* Create a new type only if you want to expose additional properties or catch a specific class of exception.
* Don't expose additional properties unless you're actually going to use them.
* Use a custom exception to hold any information that more completely describes the error (e.g. error codes or structures).
  ```csharp
  throw new DatabaseException(errorInfo);
  ```
* Custom exceptions should always inherit from `Exception`.
* Custom exceptions should be `public` so that other assemblies can catch them and extract information.
* Avoid constructing complex exception hierarchies; use your own exception base-classes only if you actually will have code that needs to catch all exceptions of a particular sub-class.
* An exception should provide the two standard constructors and should use the given parameter names:
  ```csharp
  public class ConfigurationException : Exception
  {
    public ConfigurationException(string message)
      : base(message)
    { }

    public ConfigurationException(string message, Exception innerException)
      : base(message, innerException)
    { }
  }
  ```
* Only implement serialization for exceptions if you're going to use it.
* If an exception must be able to work across application domain and remoting boundaries, then it must be serializable.
* Do not cause exceptions during the construction of another exception (this sometimes happens when formatting custom messages) as this will subsume the original exception and cause confusion.

###	Throwing Exceptions

* If a member cannot satisfy its post-condition (or, absent a post-condition, fulfill the promise implied in its name or specified in its documentation), it should throw an exception.
* Use standard exceptions where possible.
* Never throw `Exception`. Instead, use one of the standard .NET exceptions when possible. These include `InvalidOperationException`, `NotSupportedException`, `ArgumentException`, `ArgumentNullException` and `ArgumentOutOfRangeException`.
* When using an `ArgumentException` or descendent thereof, make sure that the `ParamName` property is non-empty.
* Your code should not explicitly or implicitly throw `NullReferenceException`, `System.AccessViolationException`, `System.InvalidCastException`, or `System.IndexOutOfRangeException` as these indicate implementation details and possible attack points in your code. These exceptions are to be avoided with pre-conditions and/or argument-checking and should never be documented or accepted as part of the contract of a method.
* Do not throw `StackOverflowException` or `OutOfMemoryException`; these exceptions should only be thrown by the runtime.
* Do not explicitly throw exceptions from `finally` blocks (implicit exceptions are fine).

###	Catching Exceptions

* Do not catch any errors unless you are at a system boundary (e.g. you want to control how the error response from a web server is constructed.)
* Do not catch and suppress bugs.
* You may only catch all exceptions if you re-throw bugs.
* Be as specific as possible as to which exceptions are caught (even if this means you have to repeat some lines of handling code).
* Catch and re-throw an exception in order to reset the internal state of an object to satisfy a post-condition.
* Do not catch exceptions and reroute them to an event; this practice separates the point-of-failure from the logging/collection point, increasing the likelihood that an exception is ignored and making debugging very difficult.
* The _only time_ it is appropriate to handle a bug is when third-party code misbehaves _and you are sure that possibly corrupt state will not be re-used_. That is, if the component is long-lived, you should not continue to use it after it has thrown a "bug" exception. If the component is short-lived, it will be recycled and you can more-or-less safely ignore the bug. In the case of a misbehaving third-party component, catch the specific, known exception that is causing the problem and note it.
  ```csharp
  try
  {
    return new BuggyComponent().GenerateReport(data);
  }
  catch (NullReferenceException exception)
  {
    logger.Log("Buggy Component encountered known bug when processing expression.", exception);
  }
  ```

###	Wrapping Exceptions

* Only catch an exception to wrap it in another exception, log it or set an internal state
* Use an empty throw statement to re-throw the original exception in order to preserve the stack-trace.
* Wrapped exceptions should _always_ include the original exception in order to preserve the stack-trace.
* Lower-level exceptions from an implementation-specific subsection should be caught and wrapped before being allowed to bubble up to implementation-independent code (e.g. when handling database exceptions).

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

##	Marking Members as Obsolete

* Avoid introducing breaking changes. Retain old overloads/names wherever possible.
* Include the version number in the message.
  ```csharp
  [Obsolete(""Since 4.0: Use IMetaElementSearchAspect instead.")]
  ```

### Refactoring Names and Signatures

The whole point of having an agile process and lots of automated tests is to be able to quickly improve designs and accommodate new functionality. Very often, this involves quite aggressive refactoring. Aggressive refactoring means that code that compiled with a previous version of framework may no longer compile with the latest version.
In each case where such a change is to be made, the following points must be considered:

* Will customer code be affected? A “customer” in this case is _anyone_ who consumes your code: both internal and external developers.
* Do you have access to all uses of the code in order to be able to update it?
* Is all of the code that depends on code integrated at least daily in order to catch any usages you might have forgotten?
* Is the area you are changing covered by automated tests?
* Just how important is the change?
* Can you make the change in a non-destructive manner by introducing an optional parameter or an overload instead?
* Are you willing to introduce a compile error into customer code?

With an agile methodology, the answer to the question “should you?” is quite often “yes”. Cleaner, tighter and more logical code is more maintainable and self-explanatory code.

###	Safe Obsolescence

If the change is localized, you can of course make it right away. If not, you may need to go the long route described below:

1.	Mark the old version of the feature as obsolete.
2.	Create the new feature with a different name so that it does not collide with the existing feature.
3.	Rewrite the code so that the main code path uses the new feature but also incorporates the old version as well.
4.	Add an issue to complete the next stage of refactoring in the next release.
5.	Make and distribute a point release.
6.	In the next release, remove the obsolete feature and all handling for it.
7.	Rename the new feature to the desired name but retain a copy with the temporary name as well, marking it as obsolete.
8.	Update the issue to indicate that it should be completed in the next release.
9.	Make and distribute a point release.
10.	In the next release, remove the obsolete version with the temporary name and the refactoring is complete.
11.	Close the issue.

Because the steps outlined above require the patience of a saint, they should really only be used for features that absolutely _must_ be refactored but that absolutely _cannot_ break customer code. In all other cases, refactor away and make sure that repair instructions for the compile error are included in the release notes.

## Loose vs. Tight Coupling

Whether to use loose or tight coupling for components depends on several factors. If a component on a lower-level must access functionality on a higher level, then you're doing something wrong.

If the component on the higher level needs to be coupled to a component on a lower level, then it’s possible to have them be more tightly coupled by using an interface. The advantage of using an interface over a set or one or more callbacks is that changes to the semantics of how the coupling should occur can be enforced. The example below should make this much clearer.
Imagine a class that provides a single event to indicate that it has received data from somewhere.

```csharp
public class DataTransmitter
{
  public event EventHandler<DataBundleEventArgs> DataReceived;
}
```

This is the class way of loosely coupling components; any component that is interested in receiving data can simply attach to this event, like this:

```csharp
public class DataListener
{
  public DataListener(DataTransmitter transmitter)
  {
    transmitter.DataReceived += TransmitterDataReceived;
  }

  private TransmitterDataReceived(object sender, DataBundleEventArgs args)
  {
    // Do something when data is received
  }
}
```

Another class could combine these two classes in the following, classic way:

```csharp
var transmitter = new DataTransmitter();
var listener = new DataListener(transmitter);
```

The transmitter and listener can be defined in completely different assemblies and need no dependency on any common code (other than the .NET runtime) in order to compile and run. If this is an absolute _must_ for your component, then this is the pattern to use for all events. Just be aware that the loose coupling may introduce _semantic_ errors—errors in usage that the compiler will not notice.
For example, suppose the transmitter is extended to include a new event, NoDataAvailableReceived.

```csharp
public class DataTransmitter
{
  public event EventHandler<DataBundleEventArgs> DataReceived;
  public event EventHandler NoDataAvailableReceived;
}
```

Let’s assume that the previous version of the interface threw a timeout exception when it had not received data within a certain time window. Now, instead of throwing an exception, the transmitter triggers the new event instead. The code above will no longer indicate a timeout error (because no exception is thrown) nor will it indicate that no data was transmitted.

One way to fix this problem (once detected) is to hook the new event in the `DataListener` constructor. If the code is to remain highly decoupled—or if the interface cannot be easily changed—this is the only real solution.

Imagine now that the transmitter becomes more sophisticated and defines more events, as shown below.

```csharp
public class DataTransmitter
{
  public event EventHandler<DataBundleEventArgs> DataReceived;
  public event EventHandler NoDataAvailableReceived;
  public event EventHandler ConnectionOpened;
  public event EventHandler ConnectionClosed;
  public event EventHandler<DataErrorEventArgs> ErrorOccurred;
}
```

Clearly, a listener that attaches and responds appropriately to _all_ of these events will provide a much better user experience than one that does not. The loose coupling of the interface thus far requires all clients of this interface to be proactively aware that something has changed and, once again, the compiler is no help at all.

If we can change the interface—and if the components can include references to common code—then we can introduce tight coupling by defining an interface with methods instead of individual events.

```csharp
public interface IDataListener
{
  void DataReceived(IDataBundle bundle);
  void NoDataAvailableReceived();
  void ConnectionOpened();
  void ConnectionClosed();
  void ErrorOccurred(Exception exception, string message);
}
```

With a few more changes, we have a more tightly coupled system, but one that will enforce changes on clients:

* Add a list of listeners on the `DataTransmitter`
* Add code to copy and iterate the listener list instead of triggering events from the `DataTransmitter`.
* Make `DataListener` implement `IDataListener`
* Add the listener to the transmitter’s list of listeners.

Now when the transmitter requires changes to the `IDataListener` interface, the compiler will required that all listeners also be updated.
