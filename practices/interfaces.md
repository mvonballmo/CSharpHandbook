# Interfaces

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
