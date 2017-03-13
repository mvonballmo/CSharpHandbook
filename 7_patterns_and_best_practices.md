# Patterns & Best Practices

## Safe Programming

* Use early-binding—which can be checked by the compiler—wherever possible; use late-binding only when necessary.
* Avoid using `new` whenever possible; use `override` instead or restructure the code to avoid it.
* Do not use `goto` unless you have gotten the approval of a senior member of your team.
* Use `unsafe` code only for integrating external libraries.
* If a reference may be `null`, test it before using it; otherwise, use `Debug.Assert` to verify that it is not `null`.
* Do not re-use local variable names, even though the scoping rules are well-defined and allow it. This prevents surprising effects when the variable in the inner scope is removed and the code continues to compile because the variable in the outer scope is still valid.
* Do not modify a variable with a prefix or suffix operator more than once in an expression. The following statement is not allowed:

      ```c#
      items[propIndex++] = ++propIndex;
      ```

## Side Effects

A side effect is defined as a change in an object as a result of reading a property or calling a method that causes the result of the property or method to be different when called again.

* Reading a property may not cause a side effect.
* Writing a property may cause a side effect.
* Methods have side effects by definition.
* Avoid writing methods that return results and cause side effects. A method should either retrieve information or it should execute an operation, but not both.

## Null Handling

* If a value of `null` is allowed for a parameter of an interface type, consider making an empty implementation of that interface (named with the prefix `Null`).

      ```c#
      interface IWeapon
      {
        bool Ready { get; }
        void Aim();
        void Fire();
      }

      class NullWeapon : IWeapon
      {
        bool Ready
        {
          get { return false; }
        }

        void Aim()
        {
          // NOP
        }

        void Fire()
        {
          // NOP
        }
      }
      ```
    Additionally, you should make an instance of this empty implementation available via a global static. This makes it easier to write code that uses the interface, as it can assert that the parameter is non-`null` and callers can simply pass in the empty implementation to satisfy the pre-condition.

## Casting

* Use the `as`-operator when testing and using a type.  If you are just testing a type, use the is-operator.

      ```c#
      Class1 other = obj as Class1;
      if (other == null) { return false; }
      ```
* If you are using the type of an object to route to a type-specific method, then you can use the `is`-operator and the casting operator, as shown below.

      ```c#
      if (this is IMetaProperty)
      {
        HandleProperty((IMetaProperty)this);
      }
      else if (this is IMetaMethod)
      {
        HandleMethod((IMetaMethod)this);
      }
      else
      {
        HandleDefault(this);
      }
      ```
* If you are sure of the type, use the casting operator to assert the type because it will throw an `InvalidCastException` if it fails; this avoids having to test for `null`.

      ```c#
      ((IMetaClass)obj).RunShow();
      ```

## Conversions

C# types can define `explicit` and `implicit` conversions to and from other types.

* Provide conversions only where they are logically justified; this goes double for implicit conversions. For example, the Quino library object `MetaString` enhances a string with multiple language representations; it can be freely converted to and from a string using the default language. However, you should not be able to implicitly or explicitly cast a control that displays a web page to a `String` (representing either the page content or the URL).
* Generally, you should only provide implicit conversions between types that are in the same domain (like converting between string representations).
* Do not provide implicit conversions if the conversion would cause data-loss.
* Implicit conversions cannot throw exceptions; explicit conversions can. Use `InvalidCastException` for such errors.

## Exit points (continue and return)
* Multiple return statements are allowed if all exit points are relatively close to one another. For example, the following code has two, clearly visible exit points.

      ```c#
      if (a == 1) { return true; }

      return false;
      ```
* Use `return` statements to avoid additional indentation for easy-to-catch conditions.

      ```c#
      if (String.IsNullOrEmpty(key)) { return null; }
      ```
* Do not spread return statements out throughout a longer method; instead, create a local variable named result and return it from the end of the method.

      ```c#
      bool result = parameters.Count == 0;

      if (someConditionHolds())
      {
        // Perform operations
        result = false;
      }
      else
      {
        // Perform other operations

        if (someOtherConditionHolds())
        {
          // Perform operations
          result = false;
        }
        else
        {
          // Perform other operations
          result = true;
        }
      }

      return result;
      ```
* The only code that may follow the last `return` statement is the body of an exception handler.

      ```c#
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
* Avoid the use of `continue`; rewrite the program logic instead by reversing the condition. Instead of the following logic:

      ```c#
      foreach (SearchResult search in searches)
      {
        if (!search.Path.Contains("CN=")) { continue; }

        // Work with valid searches
      }
      ```
    Use the following logic:

      ```c#
      foreach (SearchResult search in searches)
      {
        if (search.Path.Contains("CN="))
        {
          // Work with valid searches
        }
      }
      ```

## Object Lifetime

* Always use the `using` statement with objects implementing `IDisposable` to limit their lifetimes.
* Set objects that are no longer needed to `null` so that the garbage collector can collect them.
* Avoid using destructors because they incur a performance penalty in the garbage collector.
* Do not access other object references inside the destructor as those objects may already have been garbage-collected (there is no guaranteed order-of-destruction as in other languages).
* Use `try`/`finally` blocks (or related constructs, like `using` or `lock`) to clean up objects allocated in a method. Local variables are automatically de-referenced when exiting the method, so there’s no need to set them to `null`.

## Using Dispose and Finalize

`Dispose` and `Finalize` provide control over how an object is garbage-collected. `Finalize` is called when an object is reclaimed by the garbage collector; overriding it prevents resources from leaking if a consumer of your class fails to call `Dispose`.

* If you implement `Dispose`, use the `IDisposable` interface to allow explicit recovery of resources.
* Make sure that `Dispose` can be called multiple times safely; all other methods should raise an `ObjectDisposedException` if `Dispose` has already been called.
* If there is a more appropriate domain-specific name for Dispose, implement the IDisposable interface explicitly and provide a method with the new name that calls Dispose.
* Call the `GC.SuppressFinalize` method to prevent `Finalize` from being executed if `Dispose()` has already been called.
* If you implement `Dispose`, implement `Finalize` only if you actually have costly external resources.
* `Finalize` should simply include a call to `Dispose`.
* There are performance penalties for implementing `Finalize`, so proceed with caution.
* `Finalize` should never be `public`; call only the `base.Finalize()` method directly.

## Using base and this

* Use `this` only when referring to other constructors.
* Do not use `this` to resolve name-clashes; instead, change one of the conflicting names.
* Use `base` only from a constructor or to call a predecessor method.
* You may only call the `base` method of the method being executed; do not call other `base` methods. In the following example, the call to `CheckProcess()` is not allowed, whereas the call to `RunProcess()` is.

      ```c#
      public override void RunProcess()
      {
        base.CheckProcess();  // Not allowed
        base.RunProcess();
      }
      ```
##	Using Value Types

Some value types have both Pascal- and camel-case versions. Though `string` is simply an alias of `String`, you should not mix and match them everywhere. Instead, follow the rules below.

* Use types from the `System` namespace when calling static functions (e.g. `String.Format` or `String.IsNullOrEmpty`).
* Use the value types when declaring variables or fields (e.g. `string` instead of `String`).

## Using Strings

* Use the “+”-operator for concatenating up to three values; otherwise, use `String.Format`;
* Use a `StringBuilder` for more complex situations or when concatenation occurs over multiple statements.
* In comparisons, use `“”` instead of `String.Empty` as it is clearer and shorter. The generated code is the same.
* For function results, use `String.Empty`.
* When checking whether a string is empty, use `String.IsNullOrEmpty` instead of `(s == null)` or `(s.Length == 0)`.

## Using Checked

* Applications should always have range-checking during development and debugging.
* Range-checking may be disabled in release builds if there is a valid performance reason for doing so.
* Overflow- and underflow-prone operations should have explicit `checked` blocks (i.e. if there was a range-checking problem at some point, the block should be marked with a `checked` block). This way, even if range-checking is disabled, these blocks will still be checked.

## Using Floating Point and Integral Types

* Be extremely careful when comparing floating-point constants; unless you are using `decimals`, the representation will have limited precision and lead to subtle bugs.
* One exception to this rule is when you are comparing constants of fixed, known value (like `0.0` or `1.0`, but not `3.14`).
* Be extremely careful when casting representations with different sizes (e.g. `Int64` to `Int32`); always `Assert` that the value fits within the new representation so as to localize the point-of-failure.

## Using Generics

* Use the generic version of a class if available (e.g. use `IList<T>` instead of `IList`).
* Do not use casting in generic classes; use a generic constraint (`where`) instead.
* Use generic parameters and constraints instead of forcing a base type.
* Avoid constraints in delegates.
* Avoid constraints in generic methods; consider whether the problem can be solved another way.
* If inheriting from both a generic and non-generic interface, implement the non-generic version explicitly and implement it using the generic interface (e.g. when implementing `IEnumerable` and `IEnumberable<T>`).

## Using Partial Classes

* Use partial classes to separate `private` or `protected` inner classes away from the “main” implementation.
* Use partial classes to separate larger blocks of `private` or `protected` methods away from the `public` interface (this helps reduce file size).

## Explicit Interface Implementation

*** TODO ***

## Using Event Handlers

* Be careful with events in performance-sensitive code; handlers can affect performance in non-predictable ways.
* Assume that the state of the object triggering an event has changed in unpredictable ways (i.e. that the code executed in the event handler may have changed the state of the calling object).
* Assume that code in an event handler may trigger exceptions and act accordingly in the `Raise*` triggering method.
* Be aware of which thread is triggering the event handler and which thread is handling the event. Events handled in the main (UI) thread must be synchronized (this is a limitation of Windows UI programming).
* To avoid multi-threading problems, get a reference to the handler in a local variable before checking it and calling it. For example:

      ```c#
      protected virtual void RaiseMessageDispatched()
      {
        EventHandler handler = MessageDispatched;
        if (handler != null) { handler(this, new EventArgs()); }
      }
      ```
    The following code is an example of a simple event handler and receiver with all of the Encodo style conventions applied.

      ```c#
      public class Safe
      {
        public event EventHandler Locked;

        public void Lock()
        {
          // Perform work

          RaiseLocked(new EventArgs());
        }

        protected virtual void RaiseLocked(EventArgs args)
        {
          EventHandler handler = Locked;
          if (handler != null) { handler(this, args); }
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

*** TODO ***

* Pass-through event handlers (best practices)

## Using System.Linq

When using Linq expressions, be careful not to sacrifice legibility or performance simply in order to use Linq instead of more common constructs. For example, the following loop sets a property for those elements in a list where a condition holds.

```c#
foreach (var pair in Data)
{
  if (pair.Value.Property is IMetaRelation)
  {
    pair.Value.Value = null;
  }
}
```

This seems like a perfect place to use Linq; assuming an extension method `ForEach(this IEnumerable<T>)`, we can write the loop above using the following Linq expression:

```c#
Data.Where(pair => pair.Value.Property is IMetaRelation).ForEach(pair => pair.Value.Value = null);
```

This formulation, however, is more difficult to read because the condition and the loop are now buried in a single line of code, but a more subtle performance problem has been introduced as well. We have made sure to evaluate the restriction (`Where`) first so that we iterate the list (with `ForEach`) with as few elements as possible, but we still end up iterating twice instead of once. This could cause performance problems in border cases where the list is large and a large number of elements satisfy the condition.

###	Lazy Evaluation

Linq is mostly a blessing, but you always have to keep in mind that Linq expressions are evaluated lazily. Therefore, be very careful when using the `Count()` method because it will iterate over the entire collection (if the backing collection is of base type `IEnumerable<T>`). Linq is optimized to check the actual backing collection, so if the `IEnumerable<T>` you have is a list and the count is requested, Linq will use the `Count` property instead of counting elements naively.

A few concrete examples of other issues that arise due to lazy evaluation are illustrated below.

###	Capturing Unstable Variables/”Access to Modified Closure”

You can accidentally change the value of a captured variable before the sequence is evaluated. Since _ReSharper_ will complain about this behavior even when it does not cause unwanted side-effects, it is important to understand which cases are actually problematic.

```c#
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

// We expect one element in the overlap, “bla”
Assert.AreEqual(1, overlapData.Count);
```

The reference to the variable `d` will be flagged by _ReSharper_ and marked as an “access to a modified closure”. This is a reminder that a variable referenced—or “captured”—by the lambda expression—closure—will have the last value assigned to it rather than the value that was assigned to it when the lambda was created. In the example above, the lambda is created with the first value in the sequence, but since we only use the lambda once, and then always before the variable has been changed, we don’t have to worry about side-effects. _ReSharper_ can only detect that a variable referenced in a closure is being changed within the scope that it checks and letting you know so you can verify that there are no unwanted side-effects.
Even though there isn’t a problem, you can rewrite the `foreach`-statement above as the following code, eliminating the “Access to modified closure” warning.

```c#
var overlapData = data.Where(d => otherData.Where(od => od == d).Any()).ToList();
```

The example above was tame in that the program ran as expected despite capturing a variable that was later changed. The following code, however, will not run as expected:

```c#
var data = new[] { "foo", "bar", "bla" };
var otherData = new[] { "bla", "blu" };
var overlapData = new List<string>();

var threshold = 2;
var results = data.Where(d => d.Length == threshold);
var overlapData = data.Where(d => otherData.Where(od => od == d).Any());
if (overlapData.Any())
{
  threshold += 1;
}

// All elements are three characters long, so we expect no matches
Assert.AreEqual(0, results.Count());
      ```

Here we have a problem because the closure is evaluated _after_ a local variable that it captured has been modified, resulting in unexpected behavior. Whereas it’s possible that this is exactly what you intended, it’s not a recommended coding style. Instead, you should move the calculation that uses the lambda after any code that changes variables that it capture:

```c#
var threshold = 2;
var overlapData = data.Where(d => otherData.Where(od => od == d).Any());
if (overlapData.Any())
{
  threshold += 1;
}
var results = data.Where(d => d.Length == threshold);
```

This is probably the easiest way to get rid of the warning and make the code clearer to read.

###	Enumeration Run-time Errors

You can run into a problem changing a list that is being enumerated

```c#
var emptyElements = data.Where(d => d.IsEmpty);
foreach (var d in emptyElements)
{
  data.Remove(d);
}
```

The `emptyElements` sequence above is evaluated in the `foreach`-statement and pulls its elements from data. After the first element is removed, however, the call to retrieve the next item from `emptyElements` will cause an iteration error because the underlying sequence has changed.

###	Unexpected Results

You can get unexpected behavior that result in neither a run-time nor a compile-time error. Suppose we’ve fixed the error in the code above by forcing evaluation of the `emptyElements` sequence by putting its elements in a list. However, we also want to know how many elements were empty, so we return the `Count()`.

```c#
var emptyElements = data.Where(d => d.IsEmpty);
foreach (var d in emptyElements.ToList())
{
  data.Remove(d);
}

return emptyElements.Count();
```

Since `emptyElements` is evaluated lazily, the call to `Count()` to return the result will evaluate the iterator again, producing a sequence that is now empty—because the `foreach`-statement removed them all from data. The code above will always return zero.

A more critical look at the code above would discover that the `emptyElements` iterator is triggered three times: by the call to `Any()`, `ToList()` and `Count()`. Any sane implementation for the `Any()` method iterates only the first element of the list, but calls to `ToList()` and `Count()` must logically iterate the entire sequence.

Before trying to solve the problem, you have to consider how many items will commonly be in data. The next question will commonly be whether memory or speed is more important, but the code above includes a required call to `ToList()`, which renders the point moot because the algorithm already requires all code to be copied to another list. The least-wasteful solution then is:

```c#
var emptyElements = data.Where(d => d.IsEmpty).ToList();
foreach (var d in emptyElements)
{
  data.Remove(d);
}

return emptyElements.Count;
```

This restricts the amount of copied references to the number of empty elements in the list. If that’s not the common case, then a better algorithm would be to find all non-empty elements instead:

```c#
var nonEmptyElements = data.Where(d => !d.IsEmpty).ToList();
var dataCount = data.Count();
data.Clear();
foreach (var d in nonEmptyElements)
{
  data.Add(d);
}

return dataCount – nonEmptyElements;
```

The point is, you can’t be lazy with Linq; get it?

## Using Extension Methods

_This section applies to .NET 3.5 and newer._

Extension methods are a way of adding available methods to a type without actually defining those methods on the type itself. Though these methods may contribute to API bloat, they are encouraged as long as they are used often (more than once, please) and provide a clear service—either making calling code much cleaner or handling a tricky task, or both.

* Use extension methods to extend system or third-party library types.
* Use extension methods to avoid cluttering interfaces with methods that are not central to the interface and can be implemented using other members of the interface.
* Use extension methods to provide a common implementation of some functionality for all implementations of an interface, so that each implementation is not required to implement functionality that can be provided centrally (i.e. poor-man’s multiple-inheritance).
* Do not use extension methods when an interface implementation could possibly provide an optimized version of the functionality were the method declared on the interface instead. That is, do not restrict an implementation’s efficiency because an extension instead of interface method was used.
* Do not mix extension methods with other static methods. If a class contains extension methods, it should contain only extension methods and private support methods.
* Each extended type should have its own extension methods class; do not mix extension methods for different types in one class.

##	Using Optional Parameters

***TODO***
<http://blogs.msdn.com/b/ericlippert/archive/2011/05/19/optional-argument-corner-cases-part-four.aspx>:

This is a fairly serious versioning issue, and one of the main reasons why we pushed back for so long on adding default arguments to C#. The lesson here is to think carefully about the scenario with the long term in mind. If you suspect that you will be changing a default value and you want the callers to pick up the change without recompilation, don't use a default value in the argument list; make two overloads, where the one with fewer parameters calls the other.

## Using “var”

_This section applies to .NET 3.5 and newer._

The introduction of the keyword `var` for implicitly-typed variables is a boon to writing legible code. Using `var` can eliminate a lot of repeated text from code and make the intent much clearer. However, the goal is to make code more legible, not just to use implicit typing as much as possible. [\[1\]](#footnote_1)

* It is not sufficient that the code compile; a human reader must also be able to (relatively) easily understand the code.
* You should always use semantically relevant names; this goes doubly so for local variables using `var`.
* Do not use `var` when the return type is a basic type, like `int` or `string`.
* Use of `var` in larger scopes requires more care; within smaller scopes, its use is almost always ok.
* Use `var` when the type or intent is already clear from the context; otherwise, specify the type to improve understandability.
* Removing too many types not only reduces readability, but also reduces navigability (i.e. one can no longer navigate to related types using <kbd>F12</kbd>).

usage of `var`:

* Use `var` when you have to; when you are using anonymous types.
* Use `var` when the type of the declaration is obvious from the initializer, especially if it is an object creation. This eliminates redundancy.
* Consider using `var` if the code emphasizes the semantic "business purpose" of the variable and downplays the "mechanical" details of its storage.
* Use explicit types if doing so is necessary for the code to be correctly understood and maintained.
* Use descriptive variable names regardless of whether you use "var". Variable names should represent the semantics of the variable, not details of its storage; `decimalRate` is bad; `interestRate` is good.

###	Examples

* The classic case involves direct instantiation using the new-operator.

      ```c#
      IList<Airplane> planes = new List<Airplane>();
      ```
    This type of declaration needlessly clutters the code and should be replaced:

      ```c#
      var planes = new List<Airplane>();
      ```
* The rule does not state that the full type must be stated—simply that the code be understandable. The following example can use `var` because it also uses appropriate variable- and method-naming conventions.

      ```c#
      IDataList<Airplane> planes = connection.GetList<Airplane>();
      ```
    Naming the variable `planes` in the plural and using the method name `GetList` is sufficient to get the point across that the variable is a list of `Airplane` objects.

      ```c#
      var planes = connection.GetList<Airplane>();
      ```
* In the following example, the right-hand side offers no hint as to the type (other than that implied by the plural name GetAirplanes).

      ```c#
      IDataList<Airplane> planes = hanger.GetAirplanes(connection);
      ```
    However, you can use var here because the name of the variable planes is semantically relevant.

      ```c#
      var planes = hangar.GetAirplanes(connection);
      ```

## Using out and ref parameters

One case in which you may return error codes instead of throwing exceptions is when writing a library that will be used external code that does not support passing exceptions across process or domain boundaries.

* If you must use error codes, do not `return` them; use `out` and `ref` parameters instead.
* Use `out` and `ref` parameters as little as possible.
* Instead of using many `out` and `ref` parameters, consider defining a `struct` instead; this improves the maintainability of the API.

## Restricting Access with Interfaces

One advantage in using interfaces over abstract classes is that interfaces can restrict write-access to properties or lists. That is, the interface declares a getter, but no setter so that clients of the interface can only read the property. However, the backing implementation is free to add a setter as well, allowing the creator of the backing object to set the property externally, if desired.

The following example illustrates this principle for properties:

```c#
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

```c#
interface IProcessedCommands
{
  IEnumerable<ISwitchCommand> SwitchCommands { get; }
}

class ProcessedCommands
{
  IEnumerable<ISwitchCommand> IProcessedCommands.SwitchCommands
  {
    get { return SwitchCommands; }
  }

  IList<ISwitchCommand> SwitchCommands { get; private set; }
}
```

In this way, the client of the interface cannot call `Add()` or `Remove()` on the list of commands, but the provider has the convenience of doing so as it prepares the backing object. Using Linq, the client is free to convert the result to a list using `ToList()`, but will create its own copy, leaving the `SwitchCommands` on the object represented by the interface unaffected.

## Error Handling

###	Strategies

* Exceptions are the primary means of signaling errors (see 7.24.3 – Exceptions).
* Consider carefully whether an error is truly exceptional or whether the component should handle the error and set a property to indicate a status instead. This is especially true of code that performs an asynchronous task (e.g. a communications component), where the initiation point is separated from the completion point.
* Do not design methods that change error-handling strategy depending on a parameter; use the Try*-pattern instead.
* If a method is expected to encounter one or more errors, don’t use exceptions; use an `IMessageRecorder` instead.
* Reserve the result for semantically relevant data. If there is no semantically relevant result, use `void`. The following function is incorrect because -1 is not a semantically relevant result for the method name.

      ```c#
      public int GetNumberOfPeopleInFile(string fileName)
      {
        if (CanLoadFile(fileName))
        {
          // Return actual number of people.
        }

        return -1;
      }
      ```
    Instead, you should use something like the following code, throwing exceptions for situations that the API is not meant to handle.

      ```c#
      public int GetNumberOfPeopleInFile(string fileName)
      {
        File people = LoadFile(fileName); // throws exception

        // Return actual number of people.
      }
      ```

###	Error Messages

* The standards for error messages are the same as for any other text that might be shown to a user; use grammatically correct English (though translations may also be provided).
* Avoid question marks and exclamation points in messages (even assertion messages). Use the error level or exception type to indicate severity or let the handler of the exception determine what level of urgency to assign.
* Messages should always end in a period.
* Lower-level, developer messages should be logged to sources that are available only to those with permission to view lower-level details.
* Applications should avoid showing sensitive information to end-users. This applies especially to web applications, which must never show exception traces in production code. The exact message returned by an exception can vary depending on the permission level of the executing code.
* Be as specific as possible when throwing exceptions.
* The message should include as much information as possible, though it is highly recommended to provide both a longer, low-level message (for logging and debugging) and a shorter, high-level message for presenting to the user.
* Error messages should describe how the user or developer can avoid the exception.

###	The Try* Pattern

The Try\* pattern is used by the .NET framework. Generally, Try\*-methods accept an `out` parameter on which to attempt an operation, returning true if it succeeds.

* If you provide a method using the Try* pattern (), you should also provide a non-try-based, exception-throwing variant as well.

      ```c#
      public IExpression Parse(string text, IMessageRecorder recorder)
      {
        // parse the string to an expression
      }

      public bool TryParse(string text, IMessageRecorder recorder, out IExpression result)
      {
        try
        {
          result = Parse(text, recorder);
          return true;
        }
        catch (ExpressionException)
        {
          result = null;
          return false;
        }
      }
      ```

In the example above, you’ll note the parse process also accepts an `IMessageRecorder`, which is used to record hints, warnings and errors. The function throws an `ExpressionException` if any unrecoverable errors were detected. The contents of the exception message should include the recorded messages as well.

## Exceptions

###	Defining Exceptions

* Do not simply create an exception type for every different error.
* Create a new type only if you want to expose additional properties on the exception; otherwise, use a standard type.
* Use a custom exception to hold any information that more completely describes the error (e.g. error codes or structures).

      ```c#
      throw new DatabaseException(errorInfo);
      ```
* Custom exceptions should always inherit from `Exception` (do not use `ApplicationException` as its use has been deprecated).
* Custom exceptions should be `public` so that other assemblies can catch them and extract information.
* Avoid constructing complex exception hierarchies; use your own exception base-classes if you actually will have code that needs to catch all exceptions of a particular sub-class.
* An exception should provide the three standard constructors (as well as the serialization constructor if you are supporting serialization) and should use the given parameter names:

      ```c#
      public class ConfigurationException : Exception
      {
        public ConfigurationException()
        { }

        public ConfigurationException(string message)
          : base(message)
        { }

        public ConfigurationException(string message, Exception inner)
          : base(message, innerException)
        { }

        public ConfigurationException(SerializationInfo info, StreamingContext context)
          : base(message, innerException)
        { }
      }
      ```
* Do not cause exceptions during the construction of another exception (this sometimes happens when formatting custom messages) as this will subsume the original exception and cause confusion.
* If an exception must be able to work across application domain and remoting boundaries, then it must be serializable.

###	Throwing Exceptions

* If a member cannot satisfy its post-condition (or, absent a post-condition, fulfill the promise implied in its name or specified in its documentation), it should throw an exception.
* Use standard exceptions where possible.
* Do not throw `Exception` or `SystemException`. [\[2\]](#footnote_2)
* Never throw `Exception`. Instead, use one of the standard .NET exceptions when possible. These include `InvalidOperationException`, `NotSupportedException`, `ArgumentException`, `ArgumentNullException` and `ArgumentOutOfRangeException`.
* When using an `ArgumentException` or descendent thereof, make sure that the `ParamName` property is non-empty.
* Your code should not explicitly or implicitly throw ^NullReferenceException`, `System.AccessViolationException`, `System.InvalidCastException`, or `System.IndexOutOfRangeException` as these indicate implementation details and possible attack points in your code. These exceptions are to be avoided with pre-conditions and/or argument-checking and should never be documented or accepted as part of the contract of a method.
* Do not throw `StackOverflowException` or `OutOfMemoryException`; these exceptions should only be thrown by the runtime.
* Do not explicitly throw exceptions from `finally` blocks (implicit exceptions are fine).

###	Catching Exceptions

* Be as specific as possible as to which exceptions are caught (even if this means you have to repeat some lines of handling code). This allows unexpected exceptions (e.g. `NullReferenceException`) to show up during testing instead of being swallowed and logged with other, expected errors.
* Catch and re-throw an exception in order to reset the internal state of an object to satisfy a post-condition.
* In the case of a misbehaving third-party component, catch specific exceptions as much as possible, but also catch all exceptions to prevent third-party problems from crashing the application.

      ```c#
      try
      {
        // Use third-party code
      }
      catch (DatabaseException)
      {
        // Handle problems with database
      }
      catch (ArgumentException)
      {
        // Handle problems with arguments
      }
      catch (Exception)
      {
        // Handle misbehaving third-party code
      }
      ```
* Do not catch exceptions and reroute them to an event; this practice separates the point-of-failure from the logging/collection point, increasing the likelihood that an exception is ignored and making debugging very difficult.

###	Wrapping Exceptions

* Only catch an exception if you plan to wrap it in another exception, if you plan to handle it by logging or setting an internal state, or
* Use an empty throw statement to re-throw the original exception in order to preserve the stack-trace.
* Wrapped exceptions should _always_ include the original exception in order to preserve the stack-trace.
* Lower-level exceptions from an implementation-specific subsection should be caught and wrapped before being allowed to bubble up to implementation-independent code (e.g. when handling database exceptions).

###	Suppressing Exceptions

* Only suppress expected exceptions; do not write a catch-all exception handler unless you are re-throwing the exception.
* You may only suppress an exception if you either set an internal state indicating the exception on the catching object or if you log it (e.g. to an `IMessageRecorder` or a `TraceSource`).
* If it is unsafe to continue executing after an error, consider calling `System.Environment.FailFast` instead of throwing an exception.

###	Specific Exception Types

* Catch and suppress `System.Exception` only from the most external layer of code in an application or module. In other words, use catch-all exception handling when the exception can be presented to the user or must be passed across an API-boundary (e.g. out of a DLL loaded by legacy code).
* Do not catch `ArgumentExceptions` unless causing the error is unavoidable (i.e. you should check preconditions before calling a method).
* Do not catch a `StackOverflowException` as the state of the running application that has encountered a stack-overflow is not defined.
* Catching an `OutOfMemoryException` should be done rarely or not at all.

## Generated code

* Use partial classes for generated code.
* Avoid modifying generated code unless there is an extremely good reason for doing so.
* Use a custom build step to perform the modification to make sure that the required change is not lost if the environment re-generates the file.
* Do not add application logic to `AssemblyInfo.cs`; add only attributes.

## Setting Timeouts

* Use `TimeSpans` to encapsulate timeout values.
* Prefer passing the timeout value as a parameter to the method that will use it rather than offering it as a property on the class. This more closely associates the timeout value with the operation that uses it.
* The method can decide which values of `TimeSpan` are valid (including `TimeSpan.Zero` and `TimeSpan.MaxValue`).

## Configuration & File System

* An assembly or application should never make any assumptions about the location from which it’s running; nor should it make assumptions about folder structure on a hard drive. Use the members of `System.Environment.SpecialFolder` instead.
* Do not use the registry to store application information; save user settings to a user-accessible file instead.

## Logging and Tracing

* Use `IMessageRecorder` and `IMessageStore` wherever possible to collect multiple errors from a process instead of throwing a single exception. Use these classes when output should go to a user or might be collected by a UI; otherwise, use tracing.
* Use `TraceSources` to send output to logs; do not log directly to the console or screen

##	Marking Members as Obsolete

*** TODO ***

How to use OBSOLETE: `[Obsolete]` tags are good, but `[Obsolete("Obsolete since 1.6.2.0: Use IMetaElementSearchAspect instead.")]` is better

## Performance

* If possible, set the list capacity in the constructor.

## Refactoring Names and Signatures

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

Whether to use loose or tight coupling for components depends on several factors. If a component on a lower-level must access functionality on a higher level, this can only be achieved with loose coupling: e.g. connecting the two by using one or more delegates or callbacks.
If the component on the higher level needs to be coupled to a component on a lower level, then it’s possible to have them be more tightly coupled by using an interface. The advantage of using an interface over a set or one or more callbacks is that changes to the semantics of how the coupling should occur can be enforced. The example below should make this much clearer.
Imagine a class that provides a single event to indicate that it has received data from somewhere.

```c#
public class DataTransmitter
{
  public event EventHandler<DataBundleEventArgs> DataReceived;
}
```

This is the class way of loosely coupling components; any component that is interested in receiving data can simply attach to this event, like this:

```c#
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

```c#
var transmitter = new DataTransmitter();
var listener = new DataListener(transmitter);
```

The transmitter and listener can be defined in completely different assemblies and need no dependency on any common code (other than the .NET runtime) in order to compile and run. If this is an absolute _must_ for your component, then this is the pattern to use for all events. Just be aware that the loose coupling may introduce _semantic_ errors—errors in usage that the compiler will not notice.
For example, suppose the transmitter is extended to include a new event, NoDataAvailableReceived.

```c#
public class DataTransmitter
{
  public event EventHandler<DataBundleEventArgs> DataReceived;
  public event EventHandler NoDataAvailableReceived;
}
```

Let’s assume that the previous version of the interface threw a timeout exception when it had not received data within a certain time window. Now, instead of throwing an exception, the transmitter triggers the new event instead. The code above will no longer indicate a timeout error (because no exception is thrown) nor will it indicate that no data was transmitted.

One way to fix this problem (once detected) is to hook the new event in the `DataListener` constructor. If the code is to remain highly decoupled—or if the interface cannot be easily changed—this is the only real solution.

Imagine now that the transmitter becomes more sophisticated and defines more events, as shown below.

```c#
public class DataTransmitter
{
  public event EventHandler<DataBundleEventArgs> DataReceived;
  public event EventHandler NoDataAvailableReceived;
  public event EventHandler ConnectionOpened;
  public event EventHandler ConnectionClosed;
  public event EventHandler<DataErrorEventArgs> ErrorOccured;
}
```

Clearly, a listener that attaches and responds appropriately to _all_ of these events will provide a much better user experience than one that does not. The loose coupling of the interface thus far requires all clients of this interface to be proactively aware that something has changed and, once again, the compiler is no help at all.

If we can change the interface—and if the components can include references to common code—then we can introduce tight coupling by defining an interface with methods instead of individual events.

```c#
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

Now when the transmitter requires changes to the `IDataListener` interface, the compiler will enforce that all listeners are also updated.

## Footnotes

1. <a name="footnote_1"></a>: It is assumed that this keyword be used in an environment (e.g. Visual Studio 2008) that offers the code-completion and quick navigation that makes implicitly-typed variables usable.
1. <a name="footnote_2"></a>: _Visual Studio_ generates code that throws an Exception when to indicate that it has not yet been implemented; these are only temporary and do not need to be changed.