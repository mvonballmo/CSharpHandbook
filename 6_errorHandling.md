
# Error Handling

## Strategies

* Prefer exceptions to return codes.
* Use a return code only where all results are valid.
* Use the `Try*`-pattern (illustrated below) to encapsulate methods that can fail.
* If errors/warnings are expected, then use an `ILogger` or similar construct to record those warnings rather than throwing and multiple catching exceptions.

## Terms

The following definitions are used below:

* _errors_ are thrown when handling input. An application must _handle_ and _recover_ from these.
* _bugs_ result from programming error. An application _may not_ handle or recover from these.
* An exception is _caught_ with a `catch` block that matches it
* An application _re-throws_ an exception with a naked `throw` statement
* An application _wraps_ an exception by throwing a new exception with the original exception as its inner exception.
* An exception is _handled_ if it is neither re-thrown nor wrapped
* An exception is _logged_ by writing it to a logging handler

## Errors

The following are examples of errors.

* Timed-out calls over a network
* Storage failure on a database
* Invalid input-file format
* Invalid user input

## Bugs

The following are examples of bugs.

* The system runs out of memory
* Dereferencing a `null` variable
* An invalid cast
* Any unexpected situation for which the software is not prepared

The most common exceptions in C# are bugs.

* `ArgumentException` and descendants
* `NullReferenceException`
* `ClassCastException`
* `OutOfMemoryException`
* `StackOverflowException`
* `InvalidOperationException`
* `AccessViolationException`
* `NotSupportedException`
* `NotImplementedException`

## Design-by-Contract

Use assertions at the beginning of a method to assert preconditions; assert post-conditions where appropriate.

* Throw `ArgumentNullExceptions` for preconditions and post-conditions.
* Do not use `Debug.Assert`.
* Do not remove constracts in release code unless you can prove a performance issue.
* Throw the exception on the same line as the check, to mirror the formatting of the assertion.
  ```csharp
  if (connection == null) { throw new ArgumentNullException("connection"); }
  ```
* If the assertion cannot be formulated in code, add a comment describing it instead.
* All methods and properties used to test pre-conditions must have the same visibility as the method being called.

## Throwing Exceptions

* If a member cannot satisfy its post-condition (or, absent a post-condition, fulfill the promise implied in its name or specified in its documentation), it should throw an exception.
* Use standard exceptions.
* Never throw `Exception`. Instead, use one of the standard .NET exceptions when possible. These include `InvalidOperationException`, `NotSupportedException`, `ArgumentException`, `ArgumentNullException` and `ArgumentOutOfRangeException`.
* When using an `ArgumentException` or descendent thereof, make sure that the `ParamName` property is non-empty.
* Your code should not explicitly or implicitly throw `NullReferenceException`, `System.AccessViolationException`, `System.InvalidCastException`, or `System.IndexOutOfRangeException` as these indicate implementation details and possible attack points in your code. These exceptions are to be avoided with pre-conditions and/or argument-checking and should never be documented or accepted as part of the contract of a method.
* Do not throw `StackOverflowException` or `OutOfMemoryException`; these exceptions should only be thrown by the runtime.
* Do not explicitly throw exceptions from `finally` blocks (implicit exceptions are fine).

## Catching Exceptions

* Do not handle bugs.
* Handle only specific, expected errors.
* Always log handled errors.
* Catch and re-throw an error in order to reset the internal state of an object.
* Do not log exceptions to an event handler; this practice separates the point-of-failure from the logging/collection point, increasing the likelihood that an exception is ignored and making debugging very difficult.

### Buggy Third-party code

The _only time_ it is appropriate to handle a bug is when third-party code has a bug _and you are sure that possibly corrupt state will not be re-used_. If the component is long-lived, you should not continue to use it after it has encountered a bug.

If the component is short-lived, it will be recycled and you can more-or-less safely ignore the bug. In the case of a misbehaving third-party component, catch the specific, known exception that is causing the problem and note it.

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

## Defining Exceptions

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
* If an exception must be able to work across network boundaries, then it must be serializable.
* Do not cause exceptions during the construction of another exception (this sometimes happens when formatting custom messages) as this will subsume the original exception and cause confusion.

### Wrapping Exceptions

* Only catch an exception to wrap it in another exception, log it or set an internal state
* Use an empty throw statement to re-throw the original exception in order to preserve the stack-trace.
* Wrapped exceptions should _always_ include the original exception in order to preserve the stack-trace.
* Lower-level exceptions from an implementation-specific subsection should be caught and wrapped before being allowed to bubble up to implementation-independent code (e.g. when handling database exceptions).

## The Try* Pattern

The Try\* pattern is used by the .NET framework. Generally, Try\*-methods accept an `out` parameter on which to attempt an operation, returning `true` if successful.

* The parameter should be named “result”. The method should be prefixed with “Try”.
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

## Error Messages

### Content

* Use complete sentences that end in a period.
* Do not use question marks or exclamation points.
* Be brief.
* Be specific.
* Provide information on how to prevent the error in the future.

The following message is too vague and wordy.

```csharp
"The file that the application was looking for in order to load the configuration could not be loaded from the user folder."
```

This message leaves a lot of questions open.

* Does the file exist?
* Is it empty?
* Is it corrupted?
* Can the application read it?
* Where exactly was the application looking?

Instead, use something like the following.

```csharp
"Permission to read file [~/.appConfig] was denied."
```

From this message the problem is clear and the user has many clues as to how to address the issue.

### Exceptions

* Log all exceptions.
* Include technical detail in a separate message in the exception (e.g. stored in the `Data` array with a standard key).
* Lower-level, developer messages should be logged to sources that are available only to those with permission to view lower-level details.
* Applications should avoid showing sensitive information to end-users. This applies especially to web applications, which must never show exception traces in production code. The exact message returned by an exception can vary depending on the permission level of the executing code.
* If data included in a message _could_ be empty, consider wrapping it in braces so that the message is clear even when the data is empty. For example, the following code might produce a confusing error message:
  ```csharp
  var message = $"The following expression {data} could not be parsed.";
  ```
  If `data` is empty, the caller (user or developer) sees only _The following expression could not be parsed._ If the message was instead defined as follows:
  ```csharp
  var message = $"The following expression [{data}] could not be parsed.";
  ```
  Then the caller sees _The following expression [] could not be parsed._ In this case it's more obvious that the expression was empty.
