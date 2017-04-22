# Object Lifetime

* Use the `using` statement to precisely delineate the lifetime of `IDisposable` objects.
* Use `try`/`finally` blocks to manage other state (e.g. executing a matching `EndUpdate()` for a `BeginUpdate()`)
* Do not set local variables to `null`. They will be automatically de-referenced and cleaned up.

## `IDisposable`

* Implement `IDisposable` if your object uses disposable objects or system resources.
* Be careful about making your interfaces `IDisposable`; that pattern is a leaky abstraction and can be quite invasive.
* Implement `Dispose()` so that it can be safely called multiple times (see example below).
* Don't hide `Dispose()` in an explicit implementation. It confuses callers unnecessarily. If there is a more appropriate domain-specific name for `Dispose`, feel free to make a synonym.

## `Finalize`

* There are performance penalties for implementing `Finalize`. Implement it only if you actually have costly external resources.
* Call the `GC.SuppressFinalize` method from `Dispose` to prevent `Finalize` from being executed if `Dispose()` has already been called.
* `Finalize` should include only calls to `Dispose` and `base.Finalize()`.
* `Finalize` should never be `public`

## Destructors

* Avoid using destructors because they incur a performance penalty in the garbage collector.
* Do not access other object references inside the destructor as those objects may already have been garbage-collected (there is no guaranteed order-of-destruction in the IL or .NET runtime).

## Best Practices

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