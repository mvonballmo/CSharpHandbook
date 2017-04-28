# Miscellaneous

## Generated Code

* Do not commit manual changes to generated code. Temporary changes for debugging are fine, but always be aware that your changes will be overwritten when code is re-generated.

## Configuration and File System

* An assembly should not assume its location.
* Do not use the registry to store application information; save user settings to a user-accessible file instead.

## Logging

* Do not log directly to any output (e.g. a file or the console).
* Avoid logging directly to global or static constructs.
* Instead, inject an interface into the method where needed (e.g. `ILogger`).

### `ValueTask<T>`

The `ValueTask<T>` is also a performance-improvement feature to improve task-handling when there are a _lot_ of tasks. Do not use these unless you have performance-sensitive code.

* Use `ValueTask<T>` where results will usually be returned synchronously.
* Use `ValueTask<T>` when Tasks cannot be cached and you memory-usage is a problem.
* Do not expose `ValueTask<T>` in public APIs.


