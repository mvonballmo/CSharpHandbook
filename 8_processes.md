# Documentation

## General

* Include a `README.md` file at the root of the project that includes the following information:
  * Dependencies
  * Basic configuration
  * Basic command line
  * Links to other documentation
* Include a `LICENSE` file at the root of the project that describes licensing restrictions
* Include conceptual documentation for each concept/component to provide an overview and examples of how to use the product.
* Document members and types with XML documentation to enhance code-completion

## Members and Types

* Document public and protected elements.
* Do not document private or internal members. Naming and method-size rules mean these will document themselves.
* Document in English with US-English spelling and grammar.
* Prefer full sentences or clauses; do not use lists of keywords or short phrases.
* Include references to important members from class documentation.
* Stay consistent when documenting similar members (e.g. properties); it’s ok to repeat yourself or to use the same exact structure for all members (see below for examples) as long as the documentation is useful for that member.
* Use the prepositional possessive for code elements.
* Do not write "`<paramref name="prop">`’s value"; write "the value of `<paramref name="prop">`" instead.

## Dependencies

* Do not add `using` declarations or assembly references to resolve documentation references.
* Documentation should not introduce dependencies.
* Consider moving longer documentation out of the code and into higher-level conceptual documentation.
* Otherwise, include the required namespace in the documentation reference itself.

## XML Tags

* An API should document itself. Code documentation can sometimes be very obvious and simple. This indicates to the caller that it really _is_ that simple.
* Use a `<remarks>` section to indicate usage and to link to related members. It’s sometimes good to include references to other types or methods in descriptive sentences in addition to listing them in the `<seealso>` section.
* Use `<c>` tags for the keywords `null`, `false` and `true`.
* Use `<see>` tags to refer to properties, methods and classes.
* Use `<paramref>` and `<typeparamref>` tags to refer to method parameters.
* Use the `<inheritdoc/>` tag for method overrides or interface implementations.
* Use the `<include>` tag to include larger blocks of documentation (e.g. large `<remarks>` or `<example>` sections).
* Format  block tags onto separate lines. E.g. `<summary>`, `<param>`, `<remarks>` and `<returns>`

## Tool Support

* Do not leave generated documentation unchanged.

## Classes

* An abstract implementation of an interface should use the following form:
  ```csharp
  /// <summary>
  /// A base implementation of the <see cref="IMaker"/> interface.
  /// </summary>
  public abstract class MakerBase : IMaker { }
  ```
* The standard (or only) implementation of an interface should use the following form:
  ```csharp
  /// <summary>
  /// The standard implementation of the <see cref="IMaker"/> interface.
  /// </summary>
  public class Maker : IMaker { }
  ```
* For one of several implementations, use the following form:
  ```csharp
  /// <summary>
  /// An implementation of the <see cref="IMaker"/> interface that works with a
  /// Windows service.
  /// </summary>
  public class WindowsServerBasedMaker : IMaker { }
  ```

## Methods

* Document parameters in declaration order.
* Refer to the first parameter as "given"; subsequent parameter references do not need to be qualified. For example,
  ```csharp
  /// Gets the value of the given <paramref name="prop"/> in <paramref name="obj">.`
  ```
* The documentation should indicate which values are acceptable inputs for a parameter (e.g. whether or not it can be `null` or empty (for strings) or the range of acceptable values (for numbers). The example below demonstrates all of these principles:
  ```csharp
  /// <summary>
  /// Fills the <see cref="Body"> with random text using the given
  /// <paramref name="generator"> and <paramref name="seedValues">.
  /// </summary>
  /// <param name="generator">
  /// The generator to use to create the random text; cannot be <c>null</c>.
  /// </param>
  /// <param name="seedValues">
  /// The values with which to seed the random generator; cannot be <c>null</c>.
  /// </param>
  void FillWithRandomText(IRandomGenerator generator, string seedValues);
* For methods that return a Boolean value, use the following form:
  ```csharp
  /// <summary>
  /// Gets a value indicating whether the value of the given <paramref name="prop"/>
  /// in <paramref name="obj"> has changed since it was loaded from or last stored.
  /// </summary>
  /// <param name="obj">
  /// The object to test; cannot be <c>null</c>.
  /// </param>
  /// <param name="prop">
  /// The property to test; cannot be <c>null</c>.
  /// </param>
  /// <returns>
  /// <c>true</c> if the value has been modified; otherwise <c>false</c>.
  /// </returns>
  bool ValueModified(object obj, IMetaProperty prop);
  ```
* Do not document exceptions that are bugs (e.g. `ArgumentNullException`).
* Exceptions should begin with “If…” as shown in the example below:
  ```csharp
  /// <summary>
  /// Gets the <paramref name="value"/> as formatted according to its type.
  /// </summary>
  /// <param name="value">
  /// The value to format; can be <c>null</c>.
  /// </param>
  /// <param name="hints">
  /// Hints indicating context and formatting requirements.
  /// </param>
  /// <returns>
  /// The <paramref name="value"/> as formatted according to its type.
  /// </returns>
  /// <exception cref="FormatException">
  /// If <paramref name="value"/> cannot be formatted.
  /// </exception>
  string FormatValue(object value, CommandTextFormatHints hints);
  ```

## Constructors

* Though you can use inherited documentation for constructors, this is not recommended; instead, you should be as specific as possible on the parameter documentation.
* Where possible, the documentation for a parameter should consist only of indicating to which property the parameter is assigned and the acceptable inputs (as with other methods). Let the linked property documentation describe the effect of the parameter; this accounts for many constructor parameters.
* Parameters assigned to read-only properties should use the form "Initializes the value of ..." and parameters assigned to read/write properties should use the form: "Sets the initial value of ..."

The example below shows a class with constructor and properties documented according to the rules given:

```csharp
class SortOrderAspect
{
  /// <summary>
  /// Initializes a new instance of the <see cref="SortOrderAspect"/> class.
  /// </summary>
  /// <param name="sortOrderProperty">
  /// Initializes the value of <see cref="SortOrderProperty"/>.
  /// </param>
  /// <param name="sortOrderProperty">
  /// Sets the initial value of <see cref="Enabled"/>.
  /// </param>
  public SortOrderAspect(IMetaProperty sortOrderProperty, bool enabled)
  {
  }

  /// <summary>
  /// Gets the property used to create a manual sorting for objects using the
  /// <see cref="IMetaClass"/> to which this aspect is attached.
  /// </summary>
  IMetaProperty SortOrderProperty { get; private set; }

  /// <summary>
  /// Gets or sets a value indicating whether this <see cref="SortOrderAspect"/> is
  /// enabled.
  /// </summary>
  IMetaProperty Enabled { get; set; }
}
```

## Properties

* If a property has a non-public setter, do not include the "or sets" part to avoid confusion for public users of the property.
* Include only the `<summary>` tag. The `<value>` tag is not needed.
* The documentation for read-only properties must begin with “Gets”.
  ```csharp
  /// <summary>
  /// Gets the environment within which the database runs.
  /// </summary>
  IDatabaseEnvironment Environment { get; private set; }
  ```
* The documentation for read/write properties should begin with “Gets or sets”, as follows:
  ```csharp
  /// <summary>
  /// Gets or sets the database type.
  /// </summary>
  DatabaseType DatabaseType { get; }
  ```
* Boolean properties should have the following form, formatting the value element as follows:
  ```csharp
  /// <summary>
  /// Gets a value indicating whether the database in <see cref="Settings"/> exists.
  /// </summary>
  bool Exists { get; }
  ```
* For properties with generic names, take care to specify exactly what the property does, rather than writing vague documentation like “gets or sets a value indicating whether this object is enabled”. Tell the user what “enabled” means in the context of the property being documented:
  ```csharp
  /// <summary>
  /// Gets or sets a value indicating whether automatic updating of the sort-order
  /// is enabled.
  /// </summary>
  bool Enabled { get; }
  ```

### Full Example

The example below includes many of the best practices outlined in the previous sections. It includes `<seealso>`, `<exception>` and several `<paramref>` tags as well as clearly stating what it does with those parameters and their acceptable values. Finally, it includes extra detail in the `<remarks>` section instead of the `<summary>`.

```csharp
/// <summary>
/// Copies the entire contents of the given <paramref name="input"/> stream to the
/// given <paramref name="output"/> stream.
/// <param name="input">
/// The stream from which to copy data; cannot be <c>null</c>.
/// </param>
/// <param name="output">
/// The stream to which to copy data; cannot be <c>null</c>.
/// </param>
/// <remarks>
/// Uses a 32KB buffer; use the <see cref="CopyTo(Stream,Stream,int)"/> overload to
/// use a different buffer size.
/// </remarks>
/// <seealso cref="CopyTo(Stream,Stream,int)"/>
/// <exception cref="IOException">If the <paramref name="input"/> cannot be read
/// or is not at the head of the stream and cannot perform a seek or if the
/// <paramref name="output"/> cannot be written.</exception>
public static void CopyTo(this Stream input, Stream output)
```