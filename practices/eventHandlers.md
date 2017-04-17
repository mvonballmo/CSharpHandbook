# Event Handlers

Be aware of the following when raising events.

* Event handlers can affect performance.
* Event handlers can change the calling object.
* Event handlers can throw exceptions.
* Event handlers are not guaranteed to run in the calling thread.

## Race conditions

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