# Type Design

## Read-only Interfaces

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

## Single-Responsibility Principle

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
