# Loose vs. Tight Coupling

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
