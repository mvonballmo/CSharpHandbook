# Processes

## Documentation

### General

* Documentation should be written in English and must be grammatically correct (i.e. do not just use lists of keywords or short phrases; prefer full sentences or clauses).
* All documentation ends in a period.
* Use XML documentation to document public and protected elements.
* Method-level documentation is the absolute minimum (as it will be displayed by code-completion).
* Private and internal code should only be documented when needed; do not document small methods or methods whose purpose is evident from their implementation.
* Class documentation should include references to the important members for quick navigation and introduction to the class.
* External tutorials and samples are strongly encouraged.
* Frameworks should provide a more in-depth documentation of the API that includes high-level architecture, diagrams, examples and tutorials.
* Stay consistent when documenting similar members (e.g. properties); it’s ok to repeat yourself or to use the same exact structure for all members (see below for examples) as long as the documentation is useful for that member.
* Do not add "using namespace" declarations just to resolve documentation references;  instead, use as much of the namespace path as necessary in the documentation reference itself.  It’s better to reserve ``using` for actual code, as the reference from the documentation is not resolved by the C# compiler when generating an executable, whereas the `using` is.
* Do not ever add an assembly reference just to resolve a documentation reference. As above, for the code to pull in another assembly is fine, but the documentation should not introduce such a dependency.

### XML Tags

* Make sure the `<summary>` does not simply restate what is already obvious from the element; however, it should also be relatively short so that it is useful in code-completion.
* Use a `<remarks>` section to indicate usage and to link to related members. It’s sometimes good to include references to other types or methods in descriptive sentences in addition to listing them in the `<seealso>` section.
* Use `<c>` tags for the keywords null, false and true.
* Use `<see>` tags to refer to properties, methods and classes.
* Use `<paramref>` and `<typeparamref>` tags to refer to method parameters.
* Although you might want to use `<c>` tags for subsequent references to code elements if you want to keep the documentation readable in the code, it is highly recommended to use the `<paramref>` and `<typeparamref>` for all references because those can be refactored when the name changes.
* Use `<seealso>` tags to link documentation for overloads or related methods.
* Use the `<inheritdoc/>` tag for method overrides or interface implementations to avoid repeating documentation. [\[2\]](#footnote_2)  Add a `<remarks>` section to document specifics of the implementation (though this is actually quite rare, in practice).

      ```c#
      /// <inheritdoc/>
      bool Exists { get; }
      ```
* Use the `<include>` tag to include larger blocks of documentation (e.g. large `<remarks>` or `<example>` sections).
* Avoid applying possessives to code elements; instead of writing "`<paramref name="prop">`’s value", write "the value of `<paramref name="prop">`".

### Tool Support

* The automatically generated documentation from tools like _Ghostdoc_ isn’t too bad, but you should enhance it so that it offers more than just a reformulation of what was obvious from the signature.
* Avoid manually wrapping documentation; instead use a tool like the _Agent Smith_ plugin for _ReSharper_.

### Classes

* If the class is an abstract implementation of an interface, then use this formulation:

      ```c#
      /// <summary>
      /// A base implementation of the <see cref="IMaker"/> interface.
      /// </summary>
      public abstract class MakerBase : IMaker { }
      ```
* If the class is the standard (or only) implementation of an interface, then use this formulation:

      ```c#
      /// <summary>
      /// The standard implementation of the <see cref="IMaker"/> interface.
      /// </summary>
      public class Maker : IMaker { }
      ```
* If the class is one of several implementations, then use this formulation:

      ```c#
      /// <summary>
      /// An implementation of the <see cref="IMaker"/> interface that works with a
      /// Windows service.
      /// </summary>
      public class WindowsServerBasedMaker : IMaker { }
      ```

### Methods

* Parameters should be documented in the order that they appear in the method definition.
* Use the word "given" to refer to parameters.
* Summary documentation should not simply restate what is obvious from the method name and signature unless there is really nothing else to say.
* The documentation should indicate which values are acceptable inputs for a parameter (e.g. whether or not it can be `null` or empty (for strings) or the range of acceptable values (for numbers). The example below demonstrates all of these principles:

      ```c#
      /// <summary>
      /// Fills the <see cref="Body"> with random text using the given
      /// <paramref name="generator"> and <paramref name="seedValues">.
      /// </summary>
      /// <param name="generator"> The generator to use to create the random text;
      /// cannot be <c>null</c>.</param>
      /// <param name="seedValues">The values with which to seed the random generator;
      /// cannot be <c>null</c>.</param>
      void FillWithRandomText(IRandomGenerator generator, string seedValues);
* For methods that return a Boolean value, use the following form:

      ```c#
      /// <summary>
      /// Returns <c>true</c> if the value of <paramref name="prop"/> value has
      /// changed since it was loaded from or stored to the database; otherwise
      /// <c>false</c>.
      /// </summary>
      /// <param name="obj">The object to test; cannot be <c>null</c>.</param>
      /// <param name="prop">The property to test; cannot be <c>null</c>.</param>
      /// <returns>Returns <c>true</c> if the value has been modified; otherwise
      /// <c>false</c>.</returns>
      bool ValueModified(object obj, IMetaProperty prop);
* It is highly recommended that exceptions thrown by a method be documented.
* Exceptions should begin with “If…” as shown in the example below:

      ```c#
      /// <summary>
      /// Gets the <paramref name="value"/> as formatted according to its type.
      /// </summary>
      /// <param name="value">The value to format; can be <c>null</c>.</param>
      /// <param name="hints">Hints indicating context and formatting
      /// requirements.</param>
      /// <returns>The <paramref name="value"/> as formatted according to its
      /// type.</returns>
      /// <exception cref="FormatException">If <paramref name="value"/> cannot be
      /// formatted.</exception>
      string FormatValue(object value, CommandTextFormatHints hints);

### Constructors

* Though you can use inherited documentation for constructors, this is not recommended; instead, you should be as specific as possible on the parameter documentation. The default constructor documentation created by _Ghostdoc_ is generally quite good.
* Where possible, the documentation for a parameter should consist only of indicating to which property the parameter is assigned and the acceptable inputs (as with other methods). Let the linked property documentation describe the effect of the parameter; this accounts for many constructor parameters.
* Parameters assigned to read-only properties should use the form "Initializes the value of ..." and parameters assigned to read/write properties should use the form: "Sets the initial value of ..."

The example below shows a class with constructor and properties documented according to the rules given:

```c#
class SortOrderAspect
{
  /// <summary>
  /// Initializes a new instance of the <see cref="SortOrderAspect"/> class.
  /// </summary>
  /// <param name="sortOrderProperty">Initializes the value of <see
  /// cref="SortOrderProperty"/>.</param>
  /// <param name="sortOrderProperty">Sets the initial value of <see
  /// cref="Enabled"/>.</param>
  public SortOrderAspect(IMetaProperty sortOrderProperty, bool enabled)
  { }

  /// <summary>
  /// Gets the property used to create a manual sorting for objects using the
  /// <see cref="IMetaClass"/> to which this aspect is attached.
  /// </summary>
  /// <value>The property used to create a manual sorting for objects using the
  /// <see cref="IMetaClass"/> to which this aspect is attached.</value>
  IMetaProperty SortOrderProperty { get; private set; }

  /// <summary>
  /// Gets or sets a value indicating whether this <see cref="SortOrderAspect"/> is
  /// enabled.
  /// </summary>
  /// <value><c>true</c> if enabled; otherwise, <c>false</c>.</value>
  IMetaProperty Enabled { get; set; }
}
```

### Properties

* If a property has a non-public setter, do not include the "or sets" part to avoid confusion for public users of the property.
* Include both `<summary>` and `<value>` tags (they generate to different places in the documentation, despite their apparent redundancy). Yes, this means you will have repeated text (see the examples below).
* In general, copy the text from the "summary" to the "value", adjusting grammar so that it makes sense.
* The documentation for read-only properties should begin with “Gets” and the value elements should begin with the word “The”.

      ```c#
      /// <summary>
      /// Gets the environment within which the database runs.
      /// </summary>
      /// <value>The environment within which the database runs.</value>
      IDatabaseEnvironment Environment { get; private set; }
      ```
* The documentation for read/write properties should begin with “Gets or sets”, as follows:

      ```c#
      /// <summary>
      /// Gets or sets the database type.
      /// </summary>
      /// <value>The type of the database.</value>
      DatabaseType DatabaseType { get; }
      ```
* Boolean properties should have the following form, formatting the value element as follows:

      ```c#
      /// <summary>
      /// Gets a value indicating whether the database in <see cref="Settings"/> exists.
      /// </summary>
      /// <value><c>true</c> if the database in <see cref="Settings"/> exists;
      /// otherwise, <c>false</c>.</value>
      bool Exists { get; }
      ```
* For properties with generic names, take care to specify exactly what the property does, rather than writing vague documentation like “gets or sets a value indicating whether this object is enabled”. Tell the user what “enabled” means in the context of the property being documented:

      ```c#
      /// <summary>
      /// Gets or sets a value indicating whether automatic updating of the sort-order
      /// is enabled.
      /// </summary>
      /// <value><c>true</c> if sort-orders are automatically updated; otherwise,
      /// <c>false</c>.</value>
      bool Enabled { get; }
      ```

### Full Example

The example below includes many of the best practices outlined in the previous sections. It includes `<seealso>`, `<exception>` and several `<paramref>` tags as well as clearly stating what it does with those parameters and their acceptable values  as well as including extra detail in the `<remarks>` section instead of the `<summary>`.

```c#
/// <summary>
/// Copies the entire contents of the given <paramref name="input"/> stream to the
/// given <paramref name="output"/> stream.
/// <param name="input">The stream from which to copy data; cannot be
/// <c>null</c>.</param>
/// <param name="output">The stream to which to copy data; cannot be
/// <c>null</c>.</param>
/// <remarks>
/// Uses a 32KB buffer; use the <see cref="CopyTo(Stream,Stream,int)"/> overload to
/// use a different buffer size.
/// </remarks>
/// <seealso cref="CopyTo(Stream,Stream,int)"/>
/// <exception cref="ArgumentNullException">If either the <paramref name="input"/> or
/// <paramref name="output"/> is <c>null</c>.</exception>
/// <exception cref="ArgumentException">If the <paramref name="input"/> cannot be read
/// or is not at the head of the stream and cannot perform a seek or if the
/// <paramref name="output"/> cannot be written.</exception>
public static void CopyTo(this Stream input, Stream output)
```

### Testing

* All code paths should be tested in a high-level manner, using integration tests; there is no utility in doing method testing. [\[3\]](#footnote_3)
* Use the nUnit testing framework to create tests. [\[4\]](#footnote_4)

## ReSharper

This section discusses how to configure ReSharper to enforce the formatting rules outlined in this handbook.

TODO

## StyleCop/FxCop

***TODO***

## Releases

* Include range-checking if it doesn't hamper performance.
* Include debugging information.
* Include code optimization.

## Footnotes

1. <a name="footnote_1"></a>_ReSharper_ will offer to use namespaces in order to resolve documentation; you should ignore it.
2. <a name="footnote_2"></a>_Ghostdoc_ will make a copy of documentation from overridden or implemented interface methods. Do not use this documentation; used <inheritdoc/> instead.
3. <a name="footnote_3"></a>Classic unit-testing involves testing each and every property and method individually, which is largely a waste of time and often fails to test the interaction between components, which is actually more important.
4. <a name="footnote_4"></a>Both _ReSharper_ and _TestMatrix_ offer excellent _Visual Studio_ integration for _nUnit_ testing. _Visual Studio_’s own _msTest_ is currently being considered as a replacement for _nUnit_.
