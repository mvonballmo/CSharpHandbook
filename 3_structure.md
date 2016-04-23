# Structure

## File Contents

* Only one namespace per file is allowed.
* The namespace of the file must match its location in the project. That is, if the project’s root namespace is `Encodo.Parsers`, then the file located at `Csv/CsvParser.cs` should have the namespace `Encodo.Parsers.Csv`.
* Multiple classes/interfaces in one file are allowed, though you are encouraged to move mature APIS to a one-class-per-file structure.
* The contents of a file should be obvious from the file name.
* If there are multiple classes in a file, then the file name should describe the subsystem to which all these classes belong unless one of the classes is clearly the main class (e.g. the other classes are `Exception` or `EventArgs` descendants).
* If the file name does match a class in the file, the other types in that file should be supporting types for that class and may include other classes, `enums`, or `structs`. The class with the same name as the file must come first.
* If a file is intended to only ever have one element (class or interface), give it the same name as the element. If a file is already named this way, do not add more classes or interfaces (unless they are supporting types, as noted above); instead create a new file named after the subsystem and move all types there.
* An `enum` should never go in its own file; instead, place it in the file with the other types that use it. Generally, you should name this file “[Subsystem]Types.cs” where “Subsystem” is the name of sub-system to which the types belong.
* If a file has multiple classes and interfaces, the interfaces should go into their own file of the same name as the original file, but prefixed with “I”. For example, the interfaces for Encodo’s metadata framework are in `IMetaCore.cs`, whereas the classes are in `MetaCore.cs`.
* If a file starts with “I”, it should contain only interfaces. The only exception to this rule is for descendants of `EventArgs` and `enums` used by events declared in the interfaces.
* Do not mix third-party or generated code and manually written project or framework code in the same file; use partial classes instead.
* Tests for a file go in `<FileName>Tests.cs` (if there are a lot of tests, they should be split into several files, but always using the form `<FileName><Extra>Tests.cs`) where `Extra` identifies the group of tests found in the file. Tests should be defined in their own assembly to avoid dependencies on unit-testing assemblies. The tests for a class should appear in the same location as the class being tested. That is, the tests for the class `Encodo.Tools.Csv.CsvParser` should be in `Encodo.Testing.Tools.Csv.CsvParser`.
* Generated partial classes belong in a separate file, using the same root name as the user-editable file, but extended by an identifier to indicate its purpose or origin (as in the example below). This extra part must be Pascal-cased. For example:

      Company.cs          // user-modifiable file
      Company.Metadata.cs // properties generated from metadata

* Files should not be too large; files of 1000 lines or more are noticeably slower in the `Visual Studio` debugger. Separate logical groups of classes into multiple files using the rules above to avoid this problem (even in generated code).
* Each file should include an Encodo header, whether auto-generated or not; template files (like `*.html` or `*.xml`) should also include a header if this does not conflict with visual editing or IDE-synchronization. 
* The header should contain expandable tags for the check-in date, user and revision that can be maintained by a source-control system.
* Namespace `using` statements should go at the very top of the file, just after the header and just before the namespace declaration.

## Assemblies

* Assemblies should be named after their content using the following pattern: `Encodo.<Component>.Dll`.
* Fill out company, copyright and version for all projects, both projects and libraries.
* Use a separate assembly for external code whenever possible; avoid mixing third-party code into the same assembly as project or framework code.
* Only one `Main()` method per assembly is allowed; library assemblies should not have a `Main()` method.
* Do not introduce cyclic dependencies.
* Application and web assemblies should have as little code as possible. Business logic should go into a class library; view/controller logic should go in the application itself.
* Most long-lived projects will have a logical core and then code that provides a user interface for this core as rendered by a particular framework. A good practice is to keep interface-independent logic in a one assembly and rendering logic in an assembly named for that rendering logic.

The example below illustrates the projects for a solution called “Calculator” that supports rendering using _Microsoft WPF_, _DevExpress Winforms_, a console and application server mode.
  * `Calculator.Core`
  * `Calculator.Winform.Dx`
  * `Calculator.Wpf`
  * `Calculator.App.Winform`
  * `Calculator.App.Wpf`
  * `Calculator.App.Console`
  
The first three define libraries of functionality that is used by the next four applications. The server and console only use the `Calculator.Core` library whereas the _Winforms_ and _WPF_ applications use their respective libraries. Separating the renderer-dependent code into a separate library makes it much easier to add another application using the same renderer but performing a slightly different task. Only highly application-dependent code should be defined directly in an application project.

## Namespaces

### Usage

* Do not use the global namespace; the only exception is for ASP.NET pages that are generated into the global namespace.
* Avoid fully-qualified type names; use the `using` statement instead. 
* If the IDE inserts a fully-qualified type name in your code, you should fix it. If the unadorned name conflicts with other already-included namespaces, make an alias for the class with a `using` clause.
* Avoid putting a `using` statement inside a namespace (unless you must do so to resolve a conflict).
* Avoid deep namespace-hierarchies (five or more levels) as that makes it difficult to browse and understand.

### Naming

* Avoid making too many namespaces; instead, use catch-all namespace suffixes, like “Utilities”, “Core” or “General” until it is clearer whether a class or group of classes warrant their own namespace. Refactoring is your friend here.
* Do not include the version number in a namespace name.
* Use long-lived identifiers in a namespace name.
* Namespaces should be plural, as they will contain multiple types (e.g. `Encodo.Expressions` instead of `Encodo.Expression`).
* If your framework or application encompasses more than one tier, use the same namespace identifiers for similar tasks. For example, common data-access code goes in `Encodo.Data`, but metadata-based data-access code goes in `Encodo.Quino.Data`.
* Avoid using “reserved” namespace names like `System` because these will conflict with standard .NET namespaces and require resolution using the `global::` namespace prefix.

### Standard Prefixes

* Namespaces at Encodo start with `Encodo`
* Quino namespaces start with `Encodo.Quino`
* Namespaces for Encodo products start with `<ProductName>`
* Namespaces for customer products start with `<CustomerName>.<ProductName>`

### Standard Suffixes

Namespace suffixes are to be added to the end of an existing namespace under the following conditions:

Suffix | When to Use
--- | ---
Designer | Contains types that provide design-time functionality for a base namespace
Generator | Contains generated objects for a Quino model defined in a base namespace

### Encodo Namespaces

Listed below are the namespaces used by Encodo at the time of writing, with a description of their contents.

Namespace | Code Allowed
---| ---
`*.Properties` | Project-specific properties (e.g. properties for the Quino project are in Encodo.Quino.Properties)
Encodo | None; marker namespace only
Encodo.Data | Helper code for working with the System.Data namespace
Encodo.Core | Very generalized code, applicable to many problem domains; not product or project-specific
Encodo.Expressions | Basic expression tree builder and evaluator support (includes parsers for string format and expressions as well as operation evaluation for native operators)
Encodo.Messages | Code for issuing, recording and storing messages (e.g. errors, warnings, hints); not the same as Trace, which is much lower-level
Encodo.Security | Code describing general access control
Encodo.Testing | All tests for code in child namespaces of Encodo.
Encodo.Utilities | Code that is not product or project-specific, but addresses a very specific problem (like XML, tracing, logging etc.)
Encodo.Quino | None; marker namespace only; non-product-specific code that uses Quino metadata should be in a sub-namespaces of this one.
Encodo.Quino.Core | All domain-independent metadata definitions and implementations (e.g. IMetaClass is here, but IViewClassAspect is defined in Encodo.Quino.View)
Encodo.Quino.Data | Data access using Quino metadata
Encodo.Quino.Expressions | Expressions containing metadata references
Encodo.Quino.Meta | Contains definitions for core interfaces and classes in the Quino metadata library.
Encodo.Quino.Models | None; marker namespace only. Contains other namespaces for models used by Quino testing or internals. Generated objects should be in an Objects namespace (e.g. The Punchclock model resides in Encodo.Quino.Models.Punchclock and its objects in Encodo.Quino.Models.Punchclock.Objects)
Encodo.Quino.Objects | Concrete instances of persistable objects (e.g. GenericObject); expressly put into a separate namespace so that code cannot just use GenericObject (you have to add a namespace). This encourages developers to reference interfaces rather than use GenericObject directly.
Encodo.Quino.Persistence | Metadata-assisted storage and retrieval using the data layer.
Encodo.Quino.Properties | Properties for the Quino project only; do not place code here.
Encodo.Quino.Schema | Code related to database schema import and export.
Encodo.Quino.Testing | All tests for code in child namespaces of Encodo.Quino.
Encodo.Quino.View | Platform-independent visualization code.
Encodo.Quino.Winform | Winform-dependent code based on standard .NET controls.
Encodo.Quino.Winform.DX | DevExpress-dependent code
Encodo.Quino.Web | ASP.NET-dependent code

### Grouping and ordering

The namespaces at the top of the file should be in the following order:

System.* | .NET framework libraries
--- | ---
Third party | Non-Encodo third-party libraries
Encodo.* | Organize in order of dependency
Encodo.Quino.* | Organize in order of dependency