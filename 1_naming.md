# Naming

## Characters

* Names contain only alphabetic characters.
* The underscore is allowed only as a leading character for `private` fields.
* Numbers are allowed only for local variables in tests and then only as a suffix.
* Do not use the @-symbol

## Words

* Use **US-English** (e.g. “color” rather than “colour”).
* Use **English grammar** (e.g. use `ImportableDatabase` instead of `DatabaseImportable`).
* Use only **standard abbreviations** (e.g. “XML” or “SQL”).
* Use **correct capitalization**. If a word is not hyphenated, then it does not need a capital letter in the camel- or Pascal-cased form. For example, “metadata” is written as `Metadata` in Pascal-case, not `MetaData`.
* Use **number names** instead of numbers (e.g. `partTwo` instead of `part2`).
* Do not use Hungarian notation or any other prefixing notation to "group" types or members.
* Use **whole words** or stick to accepted short forms (e.g. you may use `max` for `maximum` but prefer the suffix `Count` to the prefix `num`).

## Semantics

* A name must be semantically **meaningful** in its **scope**.
* A name should be as **short** as possible.
* A name communicates **intent**; prefer `UpdatesAutomatically` to `AutoUpdate`.
* A name reflects **semantics**, not storage details; prefer `InterestRate` to `DecimalRate`.
* A name should include explicit **units** where possible. The following property isn't clearly defined.
  ```csharp
  public Timeout { get; } = 3600;
  ```
  This looks like a timeout of either an hour or 3.6 seconds. Timeouts are _usually_ expression in milliseconds. We can fix this with documentation.
  ```csharp
  /// <summary>
  /// The number of milliseconds to wait before aborting the operation.
  /// </summary>
  public Timeout { get; } = 3600;
  ```
  This is a good start. Even better is to include the units in the name of the property.
  ```csharp
  /// <summary>
  /// The number of milliseconds to wait before aborting the operation.
  /// </summary>
  public TimeoutInMilliseconds { get; } = 3600;
  ```

## Case

The following table lists the capitalization and naming rules for different language elements.

* Pascal-case capitalizes every word in a name.
* Camel-case capitalizes all but the first word in a name.
* **Acronyms** longer than two letters are in Pascal-case (e.g. `Xml` or `Sql`). Acronyms at the beginning of a camel-case name are always all lowercase (e.g. `html`).

Language Element | Case
--- | ---
Class | Pascal
Interface | Pascal w/leading `I`
Struct | Pascal
Enumerated type | Pascal
Enumerated element | Pascal
Properties | Pascal
Generic parameters | Pascal
Tuple field | Pascal
Public or protected `readonly` or `const` field | Pascal
Private field | Camel with leading underscore
Method argument | Camel
Local variable | Camel
Attributes | Pascal with `Attribute` suffix
Exceptions | Pascal with `Exception` suffix
Event handlers | Pascal with `EventHandler` suffix

### Collision and Matching

* An element may not have the same name as its containing element (e.g. class `Expressions` in namespace `Expressions` or property `Company` on class `Company`).
* The most appropriate name for a property is often the same as its type (e.g. for `enum` properties).
* Names differing only by case may be defined within the same scope only for different language elements (e.g. a local variable and a property or a method parameter and a local parameter).
  ```csharp
  public void UpdateLength(int newLength, bool refreshViews)
  {
    int length = Length;
    // ...
  }
  ```

## Grouping

* Do not use a “library” prefix for types (e.g. instead of `QnoDatabase`, use a more descriptive name, like `MetaDatabase` or `RelationalDatabase`).
* You may use a prefix when it's more convenient for disambiguation, as for UI-control libraries. If you do use a prefix, use a whole word (e.g. prefer `Quino` to `Qno`) and apply it consistently.
* Avoid very generic type names (e.g. `Element`, `Node`, `Message` or `Log`), which collide with types from the framework or other commonly-used libraries. Use a more specific name, if at all possible.
* If there are multiple types encapsulating similar concepts (but with different implementations, for example), you should use a common suffix to group them. For example, all the expression node types in the Encodo expressions library end in the word Expression.

## Algorithm

### Local Variables and Parameters

A parameter name or local variable should have the same name as the type. It is valid to use shorter forms for longer type names, as long as the context is clear.

* `IMetaExpressionFactory`: can be `metaExpressionFactory`, `expressionFactory` or `factory`.
* `f` is not acceptable for parameters, but is acceptable for a local variable in a short body.

## Structure

### Assemblies

* Assemblies should be named after their content.
* Group assemblies with a common prefix (e.g. `Encodo` or `Quino`).
* Separate identifiers in assembly names with a period.
* The root namespace of an assembly does not have to match the assembly name.
* The `AssemblyInfo.cs` file must contain company, copyright and version information.

### Files

* The name of the file should match the type name.
* Generated partial classes belong in a separate file, using the same root name as the user-editable file, but extended by an identifier to indicate its purpose or origin (as in the example below). This extra part must be Pascal-cased. For example:
  ```csharp
  Company.cs          // user-modifiable file
  Company.Metadata.cs // properties generated from metadata
  ```
* If a generic type is the only type with that name, then do not include the parameter names in the filename. If there is a non-generic type and a generic type with the same name, then the filename should include the generic argument names enclosed in {}. E.g. If the types `Atom` and `Atom<TInput>` both exist, then the filenames should be `Atom.c` and `Atom{TInput}.cs`, respectively.
* Tests for a file go in `<FileName>Tests.cs` (if there are a lot of tests, they should be split into several files, but always using the form `<Extra><FileName>Tests.cs`) where `Extra` identifies the group of tests found in the file. Tests should be defined in their own assembly to avoid dependencies on unit-testing assemblies. The tests for a class should appear in the same location as the class being tested. That is, the tests for the class `Encodo.Tools.Csv.CsvParser` should be in `Encodo.Testing.Tools.Csv.CsvParser`.

### Namespaces

* The namespace must match the file location in the project. For example, if the project’s root namespace is `Encodo.Parsers`, then the file located at `Csv/CsvParser.cs` should have the namespace `Encodo.Parsers.Csv`.
* Namespaces start with `<CustomerName>.<ProductName>` (e.g. `Encodo.Quino.*` or `Encodo.Punchclock.*`)
* Namespaces should be plural, as they will contain multiple types (e.g. `Encodo.Expressions` instead of `Encodo.Expression`). This also reduces the risk of collision.
* If your framework or application encompasses more than one tier, use the same namespace identifiers for similar tasks. For example, common data-access code goes in `Encodo.Data`, but metadata-based data-access code goes in `Encodo.Quino.Data`.
* Do not use “reserved” namespace names like `System` because these will conflict with standard .NET namespaces and require resolution using the `global::` namespace prefix.

## Types

### Classes

* If a class implements a single interface, it should reflect this by incorporating the interface name into its own (e.g. `MetaList` implements `IList`).
* Static classes containing extension methods end in `Extensions`
* All other static classes should use the suffix `Tools`.
* `abstract` classes should use the suffix `Base`.

### Interfaces

* Prefix interfaces with the letter “I”.

### `enums`

* Simple enumerations have singular names, whereas bit-sets  have plural names.

### Generic Parameters

* If a class or method has a single generic parameter, use the letter `T`.
* If there are two generic parameters and they correspond to a key and a value, then use `K` and `V`.
* Generic methods on classes with a generic parameter should use `TResult`, where appropriate. The example below shows a generic class with such a method.
  ```csharp
  public class ListBookmarkSelection<T> : IBookmarkSelection
  {
    public IList<TResult> GetObjects<TResult>()
    {
      // Convert list contents from T to TResult
    }
  }
  ```
* Generic conversion functions should use `TInput` and `TOutput`, respectively.
  ```csharp
  public static IList<TOutput> ConvertList<TInput, TOutput>(IList<TInput> input)
  {
    // Convert list contents from TInput to TOutput
  }
  ```
* If there are multiple parameters, but no pattern, name the “contained” element `T` (if there is one) and the other parameters something specific starting with the letter T.

### Sequences and Lists

* Prefer the plural form (e.g. `appointments`) for sequences (e.g. `IEnumerable<T>`) and lists or arrays
* Use the suffix “List” when you want to emphasize that a parameter or property is a list (e.g. `appointmentList`).

## Members

### Properties

* Properties should be nouns or adjectives.
* Prepend “Is” to the name for Boolean properties only if the intent is unclear without it. The next example shows such a case:
  ```csharp
  public bool Empty { get; }
  public bool IsEmpty { get; }
  ```
  Even though it’s a property not a method, the first example might still be interpreted as a verb rather than an adjective. The second example adds the verb “Is” to avoid confusion, but both formulations are acceptable.
* A property’s backing field (if present) must be an underscore followed by the name of the property in camel case.
* Use common names, like `Item` or `Value`, for accessing the central property of a type.
* Do not include type information in property names. For example, for a property of type `IMetaRelation`, use the name Relation instead of the name `MetaRelation`.
* Make the identifier as short as possible without losing information. For example, if a class named `IViewContext` has a property of type `IViewContextHandler`, that property should be called `Handler`.
* If there are two properties that could be shortened in this way, then neither of them should be. If the class in the example above has another property of type `IEventListHandler`, then the properties should be named something like `ViewContextHandler` and `EventListHandler`, respectively.
* Avoid repeating information in a class member that is already in the class name. Suppose, there is an interface named `IMessages`; instances of this interface are typically named messages. That interface should not have a property named `Messages` because that would result in calls to `messages.Messages.Count`, which is redundant and not very readable. Instead, name the property something more semantically relevant, like `All`, so the call would read `messages.All.Count`.

### Methods

* Methods names should include a verb.
* Method names should not repeat information from the enclosing type. For example, an interface named `IMessages` should not have a method named `LogMessage`; instead name the method `Log`.
* State what a method does; do not describe the parameters (let code-completion and the signature do that for you).
* Methods that return values should indicate this in their name, like `GetList()`, `GetItem()` or `CreateDefaultDatabase()`. Though there is garbage collection in C#, you should still use `Get` to indicate retrieval of a local value and `Create` to indicate a factory method, which always creates a new reference. For example, instead of writing:
  ```csharp
  public IDataList<GenericObject> GetList(IMetaClass cls)
  {
    return ViewApplication.Application.CreateContext<GenericObject>(cls);
  }
  ```

You should write:

  ```csharp
  public IDataList<GenericObject> CreateList(IMetaClass cls)
  {
    return ViewApplication.Application.CreateContext<GenericObject>(cls);
  }
  ```
* Avoid defining everything as a noun or a manager. Prefer names that are logically relevant, like `Missile.Launch()` rather than `MissileLauncher.Execute(missile)`.
* Methods that set a single property value should begin with the verb `Set`.
* The most generalized version of a method name should be reserved for the method that the framework wishes to encourage or that is used most often. An example from [6] is reproduced below:

    > Suppose you have two event-delivery mechanisms, one for immediate (synchronous) delivery and one for delayed (asynchronous) delivery. The names `sendEventNow()` and `sendEventLater()` suggest themselves. Now, if you want to encourage your users to use synchronous delivery (e.g., because it is more lightweight), you could name the synchronous method `sendEvent()` and keep `sendEventLater()` for the asynchronous case.

### Extension Methods

* Extension methods for a given class or interface should appear in a class named after the class or interface being extended, plus the suffix “Tools”. For example, extension methods for the class `Enum` should appear in a class named `EnumTools`.
* In the case of interfaces, the leading “I” should be dropped from the class name. For example, extension methods for the interface `IEnumerable<T>` should appear in a class named `EnumerableTools`.

### Parameters

* Prefer whole words instead of abbreviations (use `index` instead of `idx`).
* Parameter names should be based on their intended use or purpose rather than their type (unless the type indicates the purpose adequately).
* Do not simply repeat the type for the parameter name; use a name that is as short as possible, but doesn't lose meaning. (E.g. a parameter of type `IDataContext` should be called `context` instead of `dataContext`.)
* However, if the method also, at some point, makes use of an `IViewContext`, you should make the parameter name more specific, using `dataContext` instead.
* For copy constructors or equality operators, name the object to be copied or compared `other`.

### Lambdas

* Do not use the highly non-expressive `x` as a parameter name.
* Instead, use a single-letter variable starting with the first letter of the type or formal parameter.
* Parameters in a lambda expression should follow the same conventions as for parameters in standard methods.

### Events

* Single events should be named with a noun followed by a descriptive verb in the past tense.
  ```csharp
  event EventHandler MessageDispatched;
  ```
* For paired events—one raised before executing some code and one raised after—use the gerund form (i.e. ending in “ing”) for the first event and the past tense (i.e. ending in “ed”) for the second event.
  ```csharp
  event EventHandler MessageDispatching;
  event EventHandler MessageDispatched;
  ```
* Event receivers are like any other methods and should be named according to their task, not the event to which they are attached. The following method updates the user interface; it does this regardless of whether it is attached as an event receiver for the `MessageDispatched` event.
  ```csharp
  void UpdateUserInterface(object sender, EventArgs args)
  {
    // Implementation
  }
  ```
* Never start an event receiver method with “On” because Microsoft uses that convention for raising events.
* To trigger an event, use `Raise[EventName]`; this method must be `protected` and `virtual` to allow descendants to perform work before and after calling the base method.
* If you are raising events for changes made to properties, use the pattern `Raise[Property]Changed`.

### Delegates

* Use a descriptive verb for delegate names, like the following examples:
  ```csharp
  delegate string TransformString(T item);
  delegate bool ItemExists(T item);
  delegate int CompareItems(T first, T second);
  ```
* Delegate method parameter names should use the same grammar as the type, so that it sound natural when executed:
  ```csharp
  public string[] ToStrings(TransformToString transform)
  {
    // ...
    result[] = transform(item);
    // ...
  }
  ```
* If a delegate is only used once or twice and has a relatively simple syntax, use `Func<>` for the parameter signature instead of declaring a delegate. The example above can be rewritten as:
   ```csharp
  public string[] ToStrings(Func<T, string> transform)
  {
    // ...
    result[] = transform(item);
    // ...
  }
  ```
  This practice makes it much easier to determine the expected signature in code-completion as well.

## Statements and Expressions

### Local Variables

Since local variables are limited to a much smaller scope and are not documented, the rules for name-selection are somewhat more relaxed.

* Avoid using `temp` or `i` or `idx` for loop indexes. Use the suffix `Index` together with a descriptive prefix, as in `colIndex` or `itemIndex` or `memberIndex`.
* Names need only be as specific as the scope requires.
* The more limited the scope, the more abbreviated the variable may be.
* Use real words where there is enough space to do so.
* Use single letters in lambdas where you're trying to save space.
* Use `_` instead of declaring useless local variables during deconstruction.

### Return Values

* If a method creates a local variable expressly for the purpose of returning it as the result, that variable should be named `result`.
  ```csharp
  object result = this[Fields.Id];

  if (result == null) { return null; }

  return (Int32)result;
  ```

### Compiler Variables

* Compiler variables are all capital letters, with words separated by underscores.
  ```csharp
  #if ENCODO_DEVELOPER
        return true;
  #else
        return false;
  #endif
  ```