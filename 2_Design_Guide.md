# Design Guide

In general, design decisions that involve thinking about the following topics should not be made alone. You should apply these principles to come up with a design, but should always seek the advice and approval of at least one other team member before proceeding.

## Abstractions

The first rule of design is: don’t overdo it (YAGNI). Overdesign leads to a framework that offers unused functionality and has interfaces that are difficult to understand and implement. Only create abstractions where there will be more than one implementation or where there is a reasonable need to provide for other implementations in the future.

This leads directly to the second rule of design: “don’t under-design”. Understand your problem domain well enough before starting to code so that you accommodate reasonably foreseeable additional requirements. For example, you need to figure out whether multiple implementations will be required (in which case you should define interfaces) and whether any of those implementations will share code (in which case abstract interfaces or one or more abstract base classes are in order). You should create abstractions where they prevent repeated code—applying the DRY principle—or where they provide decoupling.

If you do create an abstraction, make sure that there are tests which run against the abstraction rather than a concrete implementation so that all future implementations can be tested. For example, database access for a particular database should include an abstraction and tests for that abstraction that can be used to verify all supported databases.

## Inheritance vs. Helpers

The rule here is to only use inheritance where it makes semantic sense to do so. If two classes could share code because they perform similar tasks, but aren’t really related, do not give them a common ancestor just to avoid repeating yourself. Extract the shared code into a helper class and use that class from both implementations. A helper class can be `static`, but may also be an instance.

## Interfaces vs. Abstract Classes

Whether or not to use interfaces is a hotly-debated topic. On the one hand, interfaces offer a clean abstraction and “interface” to a library component and, on the other hand, they restrict future upgrades by forcing new methods or properties on existing implementations. In a framework or library, you can safely add members to classes that have descendants in application code without forcing a change in that application code. However, abstract methods—which are necessary for very low-level objects because the implementation can’t be known—run into the same problems as new interface methods. Creating new, virtual methods with no implementation to avoid this problem is strongly frowned upon, as it fails to properly impart the intent of the method. [1](#footnote_1)

Where interfaces can also be very useful is in restricting write-access to certain properties or containers. That is, an interface can be declared with only a getter, but the implementation includes both a getter and setter. This allows an application to set the property when it works with an internal implementation, but to restrict the code receiving the interface to a read-only property. See “7.21 Restricting Access with Interfaces” for more information.

##	Modifying interfaces

* In general, be extremely careful of modifying interfaces that are used by code not under your control (i.e. code that has shipped and been integrated into other codebases).
* If a change needs to be made, it must be very clearly documented in the release notes for the code and must include tips for implementing/updating the implementation for the interface.
* For highly sensitive code—i.e. code in an interface that is highly likely to have been consumed by end-user code—you should use the process outlined in “7.29.1 − Safe Obsolescence”.
* Another solution is to develop a parallel path that consumes a new interface inherited from the existing one.

## Delegates vs. Interfaces

Both delegates and interfaces can be used to connect components in a loosely-coupled way. A delegate is more loosely-coupled than an interface because it specifies the absolute minimum amount of information needed in order to interoperate whereas an interface forces the implementing component to satisfy a set of clearly-defined functionality.

If the bridge between two components is truly that of an event sink communicating with an event listener, then you should use event handlers and delegates to communicate. However, if you start to have many such delegate connections between two components, you’ll want to improve clarity by defining an interface to more completely describe this relationship.

Another consideration is the prevailing model of the surrounding components. For example, the Windows Forms components and designer use events exclusively and so should your components if they are to integrate with the designer.

In other cases, where you have a handful of events that are highly interrelated, it is useful to create an interface so that components can be sure that they are handling the events correctly (and not forgetting to handle one or the other). See “7.30 − Loose vs. Tight Coupling” for an in-depth example.

## Methods vs. Properties

Use methods instead of properties in the following situations:

* For transformations or conversions, like `ToXml()` or `ToSql()`.
* If the value of the property is not cached internally, but is expensive to calculate, indicate this with a method call instead of a property (properties generally give the impression that they reference information stored with the object).
* If the result is not idempotent (yields the same result no matter how often it is called), it should be a method.
* If the property returns a copy of an internal state rather than a direct reference; this is especially significant with array properties, where repeated access is very inefficient.
* When a getter is not desired, use a method instead of a write-only property.

For all other situations in which both a property and a method are appropriate, properties have the following advantages over methods:

* Properties don’t require parentheses and result in cleaner code when called (especially when many calls are chained together).
* It clearly indicates that the value is a logical property of the construct instead of an operation.

## Choosing Types

* Use the least-derived possible type for local variables and method parameters; this makes the expected API as explicit and open as possible.
* Use existing interfaces wherever possible—even when declaring local or member variables. Interfaces should be useful in most instances; otherwise they’ve probably been designed poorly.
      ```c#
      IMessageStore messages = new MessageStore();
      IExpressionContext context = new ExpressionContext(this);
      ```
* Use the actual instantiated class for the member type when you need to access members not available in the interface. Do not modify the interface solely in order to keep using the interface instead of the class.

## Design-by-Contract

Use assertions at the beginning of a method to assert preconditions; assert post-conditions if appropriate.

* Throw standard system exceptions (see section 7.23.2 – Throwing Exceptions) for pre- and post-conditions. Avoid Debug.Assert as it is configured by default to show a dialog box—even in web applications and unit test runners—and this is never preferable to a real exception. It is understood that the assertion would be removed in release builds, but you should only really use it if there is a provable performance problem with using a conditioned exception instead.
* You may throw the exception on the same line as the check, to mirror the formatting of the assertion.
      ```c#
      if (connection == null) { throw new ArgumentNullException("connection"); }
      ```
* If the assertion cannot be formulated in code, add a comment describing it instead. [2](#footnote_2)
* If class invariants are not supported, describe the restrictions in the class documentation or note the invariant in commented form at the end of the class.
* All methods and properties used to test pre-conditions must have the same visibility as the method being called.

## Open vs. Closed APIs

This section used to be called “Virtual Methods” but could just as easily have been called “Virtual vs. Regular Methods” or “Protected vs. Private Methods” or “Sealed Classes and Methods vs. anyone using your APIs”. That last one was—kind of—a joke.

The point is that any element that is exposed to other code imposes a maintenance burden of some kind. Public and protected elements:

* Must be documented
* Must be tested (either with automated tests or manual testing)
* Must be properly designed (private members don’t need as much attention)
* Can only expose public types
* Cannot be refactored as mercilessly

By nature, C# is more closed in this regard because classes are internal and methods are non-virtual by default. This makes for APIs that require less maintenance but are also less open to other coders. This can be a very frustrating experience for those coders when they inherit from these classes only to find that misbehaving methods cannot be overridden, vital functionality is hidden inside private or virtual methods or classes are sealed so as to prevent inheritance entirely.

Very often, this leads to code duplication as entire swaths of code are copied via reverse-engineering just in order to apply a minor tweak. This is the case currently when working with the Silverlight framework from Microsoft where many classes are sealed and many methods are internal or private and very little of the API can be overridden.

Naturally, it would be nice to avoid frustrating other coders in this way, but making everything public or protected and virtual is also not the solution. Such blanket approaches fail to guide users of your API sufficiently. It is up to you to find a happy medium, exposing exactly the functionality that any user might need: no more, no less. Naturally, this will force you to actually think about the purpose of the class that you are writing and consider the use-cases for it.

With use-cases in mind, here are some points to consider when building a class.

* What are the odds that anyone will want to create a descendant of your class?
* If these odds are non-zero, do not seal the class.
* Does your class include functionality that descendants will want to reuse? In that case, make the method or property protected.
* Is it possible that descendants will want to change how that functionality works? In that case, make the method or property virtual.
* Anything else should be made private in order to avoid exposing too large an interface to both users of the public interface and descendants, which use the protected interface.

### Controlling API Size

* Be as stingy as possible when making methods public; smaller APIs are easier to understand.
* If another assembly needs a type to be public, consider whether that type could not remain internalized if the API were higher-level. Use the Object Browser to examine the public API.
* To this end, frameworks that are logically split into multiple assemblies can use the `InternalsVisibleTo` attributes to make “friend assemblies” and avoid making elements public. Given three assemblies, `Quino`, `QuinoWinform` and `QuinoWeb` (of which a standard Windows application would include only the first two), the `Quino` assembly can make its internals visible to `QuinoWinform`.

<a name="footnote_1">[1]</a> One exception to this rather strictly enforced rule is for classes that simply cannot be abstract. This will be the case for user-interface components that interact with a visual designer. The Visual Studio designer, for example, requires that all components be non-abstract and include a default constructor in order to be used. In these cases, empty virtual methods that throw a `NotImplementedException` are the only alternative. If you must use such a method, include a comment explaining the reason.

<a name="footnote_2">[2]</a> The spec# project at Microsoft Research provides an integration of Design-By-Contract mechanisms into the C# language; at some point in the future, this may be worthwhile to include.

## Namespaces

* Nesting depth is inversely related to abstraction. Declare high-level abstractions in the outer layers (e.g. `Encodo.Quino.Data`) and concrete types in inner layers (e.g. `Encodo.Quino.Data.Ado`).
* The namespace hierarchy should only be deep enough to reflect the actual complexity. Deep hierarchies are more  difficult to browse and understand.
* Avoid making too many namespaces; instead, use catch-all namespace suffixes, like “Utilities”, “Core” or “General” until it is clearer whether a class or group of classes warrant their own namespace. Refactoring is your friend here.

## Assemblies

* Use a separate assembly to improve decoupling and reduce dependencies.
* Top-level application assemblies should have as little code as possible. Use class libraries for most logic.

The example below illustrates the projects for a solution called “Calculator” with a _WPF_ application, a web-API application and a console application.
* `Calculator.Core`
* `Calculator.Core.Web`
* `Calculator.Core.Wpf`
* `Calculator.Web.Api`
* `Calculator.Wpf`
* `Calculator.Console`

The first three define libraries of functionality that is used by the next four applications. The server and console only use the `Calculator.Core` library whereas the _Winforms_ and _WPF_ applications use their respective libraries. Separating the renderer-dependent code into a separate library makes it much easier to add another application using the same renderer but performing a slightly different task. Only highly application-dependent code should be defined directly in an application project.
