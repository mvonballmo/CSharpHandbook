# Type Design

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