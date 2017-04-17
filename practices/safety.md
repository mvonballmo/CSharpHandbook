# Safe Programming

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

Instead of allowing `null` for a parameter, avoid null-checks with a null implementation.

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
