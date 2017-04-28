# Design

Design decisions should not be made alone. You should apply these principles to come up with a design, but should always seek the advice and approval of at least one other team member before proceeding.

## Abstractions

The first rule of design is: don’t overdo it (YAGNI). Overdesign leads to a framework that offers unused functionality and has interfaces that are difficult to understand and implement. Only create abstractions where there will be more than one implementation or where there is a reasonable need to provide for other implementations in the future.

This leads directly to the second rule of design: “don’t under-design”. Understand your problem domain well enough before starting to code so that you accommodate reasonably foreseeable additional requirements. For example, you need to figure out whether multiple implementations will be required (in which case you should define interfaces) and whether any of those implementations will share code (in which case abstract interfaces or one or more abstract base classes are in order). You should create abstractions where they prevent repeated code—applying the DRY principle—or where they provide decoupling.

If you do create an abstraction, make sure that there are tests which run against the abstraction rather than a concrete implementation so that all future implementations can be tested. For example, database access for a particular database should include an abstraction and tests for that abstraction that can be used to verify all supported databases.

## Inheritance vs. Composition

The rule here is to only use inheritance where it makes semantic sense to do so. If two classes could share code because they perform similar tasks, but aren't really related, do not give them a common ancestor just to avoid repeating yourself. Extract the shared code into a helper class and use that class from both implementations. Prefer composition of instances over `static` helpers.

## Interfaces vs. Abstract Classes

Whether or not to use interfaces is a hotly-debated topic. On the one hand, interfaces offer a clean abstraction and “interface” to a library component and, on the other hand, they restrict future upgrades by forcing new methods or properties on existing implementations. In a framework or library, you can safely add members to classes that have descendants in application code without forcing a change in that application code. However, abstract methods—which are necessary for very low-level objects because the implementation can’t be known—run into the same problems as new interface methods. Creating new, virtual methods with no implementation to avoid this problem is strongly frowned upon, as it fails to properly impart the intent of the method.

One exception to this rather strictly enforced rule is for classes that simply cannot be abstract. This will be the case for user-interface components that interact with a visual designer. The Visual Studio designer, for example, requires that all components be non-abstract and include a default constructor in order to be used. In these cases, empty virtual methods that throw a `NotImplementedException` are the only alternative. If you must use such a method, include a comment explaining the reason.

Where interfaces can also be very useful is in restricting write-access to certain properties or containers. That is, an interface can be declared with only a getter, but the implementation includes both a getter and setter. This allows an application to set the property when it works with an internal implementation, but to restrict the code receiving the interface to a read-only property.

## Open vs. Closed APIs

This section used to be called “Virtual Methods” but could just as easily have been called “Virtual vs. Regular Methods” or “Protected vs. Private Methods” or “Sealed Classes and Methods vs. anyone using your APIs”. That last one was—kind of—a joke.

The point is that any element that is exposed to other code imposes a maintenance burden of some kind. Public and protected elements:

* Must be documented
* Must be tested (either with automated tests or manual testing)
* Must be properly designed (private members don’t need as much attention)
* Can only expose public types
* Cannot be refactored as mercilessly

By nature, C# is more closed in this regard because classes are internal and methods are non-virtual by default. This makes for APIs that require less maintenance but are also less open to other coders. This can be a very frustrating experience for those coders when they inherit from these classes only to find that misbehaving methods cannot be overridden, vital functionality is hidden inside private or virtual methods or classes are sealed so as to prevent inheritance entirely.

Very often, this leads to code duplication as entire swaths of code are copied via reverse-engineering just in order to apply a minor tweak. A historical case is the Silverlight framework from Microsoft where many classes are sealed and many methods are internal or private and very little of the API can be overridden.

Naturally, it would be nice to avoid frustrating other coders in this way, but making everything public or protected and virtual is also not the solution. Such blanket approaches fail to guide users of your API sufficiently. It is up to you to find a happy medium, exposing exactly the functionality that any user might need: no more, no less. Naturally, this will force you to actually think about the purpose of the class that you are writing and consider the use-cases for it.

With use-cases in mind, here are some points to consider when building a class.

* What are the odds that anyone will want to create a descendant of your class?
* If these odds are non-zero, do not seal the class.
* Does your class include functionality that descendants will want to reuse? In that case, make the method or property protected.
* Is it possible that descendants will want to change how that functionality works? In that case, make the method or property virtual.
* Anything else should be made private in order to avoid exposing too large an interface to both users of the public interface and descendants, which use the protected interface.

## Controlling API Size

* Be as stingy as possible when making methods public; smaller APIs are easier to understand.
* If another assembly needs a type to be public, consider whether that type could not remain internalized if the API were higher-level. Use the Object Browser to examine the public API.
* To this end, frameworks that are logically split into multiple assemblies can use the `InternalsVisibleTo` attributes to make “friend assemblies” and avoid making elements public. Given three assemblies, `Quino`, `QuinoWinform` and `QuinoWeb` (of which a standard Windows application would include only the first two), the `Quino` assembly can make its internals visible to `QuinoWinform`.

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

### Callbacks vs. Interfaces

Both callbacks and interfaces can be used to connect components in a loosely-coupled way. A delegate is more loosely-coupled than an interface because it specifies the absolute minimum amount of information needed in order to inter-operate whereas an interface forces the implementing component to satisfy a set of clearly-defined functionality.

If the bridge between two components is truly that of an event sink communicating with an event listener, then you should use event handlers and delegates to communicate. However, if you start to have many such delegate connections between two components, you’ll want to improve clarity by defining an interface to more completely describe this relationship.

Another consideration is the prevailing model of the surrounding components. For example, the Windows Forms components and designer use events exclusively and so should your components if they are to integrate with the designer.

In other cases, where you have a handful of events that are highly interrelated, it is useful to create an interface so that components can be sure that they are handling the events correctly (and not forgetting to handle one or the other).

### An Example

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

## Methods vs. Properties

Use methods instead of properties in the following situations:

* For transformations or conversions, like `ToXml()` or `ToSql()`.
* If the value of the property is not cached internally, but is expensive to calculate, indicate this with a method call instead of a property (properties generally give the impression that they reference information stored with the object).
* If the result is not idempotent (yields the same result no matter how often it is called), it should be a method.
* If the property returns a copy of an internal state rather than a direct reference; this is especially significant with array properties, where repeated access is very inefficient.
* When a getter is not desired, use a method instead of a set-only property.

For all other situations in which both a property and a method are appropriate, properties have the following advantages over methods:

* Properties don’t require parentheses and result in cleaner code when called (especially when many calls are chained together).
* It clearly indicates that the value is a logical property of the construct instead of an operation.

## Code > Comments

Good variable and method names go a long way to making comments unnecessary.

Replace comments with private methods with descriptive names. For example, the following code is commented, but a bit wordy.

```csharp
public void MethodOne()
{
  // Collect and aggregate results
  var projections = new List<Result>();
  foreach (var p in OriginalProjections)
  {
    // Do a bunch of stuff with p
    projections.Add(new Projection(p, i))
  }

  // Format the projections into a text report
  var lines = new List<string>();
  foreach (var projection in projections)
  {
    var line = string.Format($"Some text with a {projection}");

    // Work with the line

    lines.Add(line);
  }

  // Save the lines to file
  File.WriteAllLines(OutputPath, lines);
}
```

Instead, use methods to achieve the same clarity without comments. At the same time, we make use of better types (`IEnumerable<T>`) and constructs (`Select()`, `NotNull`) to streamline and improve the code even more.

```csharp
public void MethodOne()
{
  var projections = CalculateFinalProjections();
  var lines = CreateReport(projections);

  StoreReport(lines);
}

[NotNull]
private IEnumerable<Projection> CalculateFinalProjections()
{
  return OriginalProjections.Select(CreateFinalProjection);
}

[NotNull]
private Projection CreateFinalProjection([NotNull] Projection p)
{
  if (p == null) { throw new ArgumentNullException(nameof(p)); }

  // Do a bunch of stuff with p

  return new Projection(p, i));
}

[NotNull]
private IEnumerable<string> CreateReport([NotNull] IEnumerable<Projection> projections)
{
  if (projections == null) { throw new ArgumentNullException(nameof(projections)); }

  return projections.Select(p => string.Format($"Some text with a {p}"));
}

private void StoreReport([NotNull] IEnumerable<string> lines)
{
  if (lines == null) { throw new ArgumentNullException(nameof(lines)); }

  File.WriteAllLines(OutputPath, lines);
}
```
This version obviously needs no comments: it is now clear what `MethodOne` does. In fact, we can streamline `MethodOne` to a single line without losing legibility. Using the syntactic constructs of the language eliminates the need for many, if not all, comments.
```csharp
public void MethodOne()
{
  StoreReport(CreateReport(CalculateFinalProjections()));
}
```

### Convert to Variables

Replace comments with local variables with descriptive names.

```csharp
// For new lessons that are within the time-span or not scheduled
if (!lesson.Stored && ((StartTime <= lesson.StartTime && lesson.EndTime <= EndTime) || !lesson.Scheduled)) { }
```
With local variables, however, the logic is much clearer and no comment is required:
```csharp
bool lessonInTimeSpan = StartTime <= lesson.StartTime && lesson.EndTime <= EndTime;

if (!lesson.Stored && (lessonInTimeSpan || !lesson.Scheduled)) { }
```