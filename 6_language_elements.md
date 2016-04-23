# Language Elements

## Declaration Order

* Constructors (in descending order of complexity)
* public constants
* properties
* public methods
* protected methods
* private methods
* protected fields
* private fields

## Visibility

* The visibility modifier is required for all types, methods and fields; this makes the intention explicit and consistent.
* The visibility keyword is always the first modifier.
* The `const` or `readonly` keyword, if present, comes immediately after the visibility modifier.
* The static keyword, if present, comes after the visibility modifier and readonly modifier.

      ```c#
      private readonly static string DefaultDatabaseName = "admin";
      ```
      
## Constants

* Declare all constants other than `0`, `1`, `true`, `false` and `null`. 
* Use `true` and `false` only for assignment, never for comparison.
* Avoid passing `true` or `false` for parameters; use an `enum` or constants to impart meaning instead.
* If there is a logical connection between two constants, indicate this by making the initialization of one dependent on the other.

      ```c#
      public const int DefaultCacheSize = 25;
      public const int DefaultGranularity = DefaultCacheSize / 5;
      ```
      
### readonly vs. const

The difference between `const` and `readonly` is that `const` is compiled and `readonly` is initialized at runtime.

* Use `const` only when the value really is constant (e.g. `NumberDaysInWeek`); otherwise, use `readonly`. 
* Though `readonly` for references only prevents writing of the reference, not the attached value, it is still a helpful hint for both the compiler and the reader.

### Strings and Resources

* Do not hardcode strings that will be presented to the user; use resources instead. For products in development, this text extraction can be performed after the code has crystallized somewhat.
* Resource identifiers should be alphanumeric, but may also include a dot (“.”) to logically nest resources.
* Do not use constants for strings; use resource tables instead (this aids translation, if necessary).
* Configuration data should be moved into application settings as soon as possible.
## Properties
* In the event that setting a property caused an exception, then the existing value should be restored.
* Use read-only properties if there is no logical reason for calling code to be able to change the value.
* Properties should be commutative; that is, it should not matter in which order you set them. Avoid enforcing an ordering by using a method to execute code that you would want to execute from the property setter. The following example is incorrect because setting the password before setting the name causes a login failure.

      ```c#
      class SecuritySystem
      {
        private string _userName;

        public string UserName
        {
          get { return _userName; }
          set { _userName = value; }
        }

        private int _password;

        public int Password
        {
          get { return _password; }
          set 
          { 
            _password = value;
            LogIn();
          }
        }

        protected void LogIn()
        {
          IPrincipal principal = Authenticate(UserName, Password);
        }

        private IPrincipal Authenticate(string UserName, int Password)
        {
          // Authenticate the user
        }
      }
      ```  
    Instead, you should take the call LogIn() out of the setter for Password and make the method public, so the class can be used like this instead:
    
      ```c#
      SecuritySystem system = new SecuritySystem();
      system.Password = "knockknock";
      system.UserName = "Encodo";
      system.LogIn();

      ```
    In this case, Password can be set before the UserName without causing any problems.
    
### Indexers

* Provide an indexed property only if it really makes sense in the context of the class.
* Indexes should be 0-based.

## Methods

* Methods should not have more than 200 lines of code 
* Avoid returning `null` for methods that return collections or strings. Instead, return an empty collection (declare a static empty list) or an empty string (`String.Empty`).
* Default implementations of empty methods should have both brackets on the same line:

      ```c#
      protected virtual void DoInitialize(IMessageRecorder recorder) { }
      ```
* Overrides of abstract methods or implementations of interface methods that are explicitly left empty should be marked with NOP:

      ```c#
      protected override void DoBeforeSave()
      {
        // NOP
      }
      ```
* Consider using partial methods to reduce the number of explicitly declared virtual methods if you are using C# 3.0.

### Virtual

* Prefer making `virtual` methods `protected` instead of `public`, but do not create an extra layer of method calls just to do so. If a method has logical pre-conditions or post-conditions (i.e. the pre-condition checks for more than just whether a parameter is null), consider making the method `protected` and wrapping it in a `public` method with the contracts in it (as below):

      ```c#
      public void Update(IQuery query)
      {
        Debug.Assert(query != null);
        Debug.Assert(query.Valid);
        Debug.Assert(Updatable);

        DoUpdate(query);

        Debug.Assert(UpToDate);
      }

      protected virtual void DoUpdate(IQuery query)
      {
        // Perform update
      }
      ```
    Always use “Do” as a prefix for such protected, helper methods.
* If a `protected` method is not `virtual`, make it `private` unless it will be used from a descendent.

### Overloads

* Overloads are encouraged for methods that are in the same family and either serve the same purpose or have similar behavior. Do not use the types of parameters to distinguish these functions from one another. For example, the following is incorrect

      ```c#
      void Update();
      void UpdateUsingQuery(IQuery query);
      void UpdateUsingSql(string sql);
      ```
    Instead, use an overload, letting the method signature describe the different functions. This reduces the perceived size of the API and makes it easier to understand.

      ```c#
      void Update();
      void Update(IQuery query);
      void Update(string sql);
      ```
* If an overloaded method must be marked `virtual`, make only one version `virtual` and define all of the others in terms of that one. Using the example above, this would yield:

      ```c#
      public void Update()
      {
        Update(QueryTools.NullQuery); // Accesses a static global “null” query
      }

      public virtual void Update(IQuery query)
      {
        // Perform update
      }

      public void UpdateUsingSql(string sql)
      {
        Update(new Query(sql));
      }
      ```
* If two or more overloads share a parameter, that parameter name should be the same in all overloads. 
* Similarly, standardize parameter positions as much as possible between overloads and even just similar methods.

### Parameters

* Methods should not have more than 5 parameters (consider using a `struct` instead).
* Methods should not have more than 2 `out` or `ref` parameters (consider using a `struct` instead).
* `ref`, then `out` parameters should come last in the list of parameters.
* The implementation of an interface method should use the same parameter name as that given in the interface method declaration.
* Do not declare reserved parameters (use overloads in future library versions instead).
* If a method follows the Try* pattern—which returns a bool indicating success, and accepts a single out parameter—the parameter should be named “result”. The method should be prefixed with “Try”.
* Do not assign new values to parameters; use a local variable instead. Assignments to parameters are easy-to-miss in larger methods. If you use a local variable instead, a reader knows right away to look for initializations of that variable rather than to look for changes to the parameter value.

### Constructors

* Base constructors should be on a separate line, indented one level.
* Consider including the default `base()` call in constructors to make it clear which constructor is called (and to provide a way of quickly jumping to the implementation in the IDE).
* A constructor is considered to be valid if it doesn’t crash and the object can be used without crashing or causing unwarranted exceptions (null reference, etc.). Any properties required by the constructor to make it valid should be passed in as parameters.
* All constructors should satisfy all class invariants; that is, you cannot require a user to set properties on an object in order to make it valid. A class may, however, require that some properties should be set before being able to use certain functions of a class. The example below shows such a class, which has an empty constructor and requires that certain properties are set before calling `Connect()` or `LogIn()`.

      ```c#
      internal abstract class BackEnd
      {
        void BackEnd()
        { }

        internal abstract string ServerName { get; set; }

        internal abstract string UserName { get; set; }
        
        internal abstract string Password { get; set; }

        internal abstract void Connect();

        internal abstract void LogIn();
      }
      ```
    As an aside, this is not a recommended design. The example above would work much better as follows:

      ```c#
      abstract class BackEnd
      {
        void BackEnd()
        { }

        abstract void Connect(IConnectionSettings settings);

        abstract void LogIn(IUser user);
      }
      ```
* Avoid doing more than setting properties in a constructor; provide a method on the class to perform any extra work after the object has been constructed.
* Avoid calling virtual methods from a constructor because the most-derived version will be called, but before the constructor for that most-derived class has executed. The example below illustrates this problem, where the override `CaffeineAddict.GoToWork()` uses Coffee before it has been initialized.

      ```c#
      public interface IBeverage
      {
        public bool Empty { get; }
      }

      public abstract class Employee
      {
        public Employee()
        {
          GoToWork();
        }

        protected abstract void GoToWork();

        protected void Drink(IBeverage beverage)
        {
          if (!beverage.Empty) // Crashes when initializing CaffeineAddict
          {
            // drink it
          }
        }
      }

      public class CaffeineAddict : Employee
      {
        public CaffeineAddict(IBeverage coffee)
          : base()
        {
          _coffee = coffee;
        }

        public IBeverage Coffee
        {
          get { return _coffee; }
        }

        protected override void GoToWork()
        {
          Drink(Coffee);
        }

        private IBeverage _coffee;
      }
      ```
* To avoid duplicating code, but also to avoid exposing an unwanted default constructor, use a `protected` default constructor. For example:

      ```c#
      protected Query()
      {
        _restrictions = new List<IRestriction>();
        _sorts = new List<ISort>();
      }

      public Query(IMetaClass model)
        : this()
      {
        Debug.Assert(model != null);
        Model = model;
      }

      public Query(IDataRelation relation)
        : this()
      {
        Debug.Assert(relation != null);
        Relation = relation;
      }
      ```
* Consider using a `static` factory method (e.g. on a `*Tools` class) if construction of an object is very complex or would require a large number of constructor parameters.

## Classes

* Never declare more than one field per line; each field should be an individually documentable entity.
* Do not use public or protected fields; use private fields exposed through properties instead.

### Abstract Classes

Please see section 2.1 – Abstractions for a discussion of when to use abstract classes.

* Do not define `public` or `protected internal` constructors for abstract types; instead, define a `protected` or `internal` one.
* Consider providing a partial implementation of an abstract class that handles some of the abstraction in a standard way; implementers can use this class as a base and avoid having to repeat code in their own implementations. Such classes should use the “Base” suffix.

### Static Classes

* Do not mark a class as static if it has instance members.
* Do not create too many static classes; instead, determine whether new functionality can be added to an existing static class.

### Sealed Classes & Methods

* Do not declare protected or virtual members on sealed classes
* Avoid sealing classes unless there is a very good reason for doing so (e.g. to improve reflection performance).
* Consider sealing only selected members instead of sealing an entire class.
* Consider sealing members that you have overridden if you don’t want descendents to avoid your implementation.

### Private and Internal Classes

* Make nested classes private wherever possible. Internal classes will also clutter the namespace and object browser. Expose them only if the class really has utility outside of the class within which it is nested.
* Use private classes for collect implementation methods for classes that get too large.
* If a class has multiple nested, private classes, consider moving them to a separate file and declaring them within another partial of the surrounding class.
* Consider whether all classes in an API must be exposed or only one or two of them. All others can be declared internal and reduce the surface of the API exposed by the assembly.

## Interfaces

Please see section 2.1 – Abstractions for a discussion of when to use interfaces.

* Use interfaces to “fake” multiple-inheritance.
* Define interfaces if there will be more than one implementation of a hierarchy; without multiple-inheritance, this is the only way to remain flexible as to the implementation.
* Define interfaces to clearly define what comprises an API; an interface will generally be smaller and more tightly-defined that the class that implements it. A class-based hierarchy runs the risk of mixing interface methods with implementation methods.
* Consider using a C# attribute instead of a marker interface (an interface with no members). This makes for a cleaner inheritance representation and indicates the use of the marker better (e.g. NUnit tests as well as the serializing subsystem for .NET use attributes instead of marker interfaces). 
* Re-use interfaces as much as possible to avoid having many very similar interfaces that cause confusion as to which one should be used where.
* Keep interfaces relatively small in order to ease implementation (5-10 members).
* Where possible, provide an abstract class or default descendent that application code can use for implementing an interface. This provides both an implementation example and some protection from future changes to the interface.
* Use interfaces where the functionality isn’t the direct purpose of the object or to expose a part of the class’s functionality (as with aspect-oriented programming).
* Use explicit interface implementation where appropriate to avoid expanding a class API unnecessarily.
* Each interface should be used at least once in non-testing code; otherwise, get rid of it.
* Always provide at least one, tested implementation of an interface.

## Structs

Consider defining a structure instead of a class if most of the following conditions apply:

* Instances of the type are small (16 bytes or less) and commonly short-lived.
* The type is commonly embedded in other types.
* The type logically represents a single value and is similar to a primitive type, like an `int` or a `double`.
* The type is immutable.
* The type will not be boxed frequently. [\[1\]](#footnote_1)

Use the following rules when defining a `struct`.

* Avoid methods; at most, have only one or two methods other than overrides and operator overloads.
* Provide parameterized constructors for initialization.
* Overload operators and equality as expected; implement `IEquatable` instead of overriding `Equals` in order to avoid the negative performance impact of boxing and un-boxing the value.
* A `struct` should be valid when uninitialized so that consumers can declare an instance without calling a constructor.
* Public fields are allowed (even encouraged) for structures used to communicate with external APIs through unmanaged code.

## Enumerations

* Always use enumerations for strongly-typed sets of values
* Use enumerations instead of lists of static constants _unless_ that list can be extended by descendent code; if the list is not logically open-ended, use an `enum`.
* Enumerations are like interfaces; be extremely careful of changing them when they are already included in code that is not under your control (e.g. used by a framework that is, in turn, used by external application code). If the enumeration must be changed, use the `ObsoleteAttribute` to mark members that are no longer in use.
* Do not assign a type to an `enum` unless absolutely necessary; use the default type of `Int32` whenever possible.
* Do not include sentinel values, such as `FirstValue` or `LastValue`.
* Do not assign explicit values to simple enumerations except to enforce specific values for storage in a database.
* The first value in an enumeration is the default; make sure that the most appropriate simple enumeration value is listed first.

### Bit-sets

* Use the `[Flags]` attribute to make a bit-set instead of a simple enumeration.
* Bit-sets always have plural names, whereas simple enumerations are singular.
* Assign explicit values for bit-sets in powers of two; use hexadecimal notation.
* The first value of a bit-set should always be `None` and equal to `0x00`.
* In bit-sets, feel free to include commonly-used aliases or combinations of flags to improve readability and maintainability. One such common value is `All`, which includes all available flags and, if included, should be defined last. For example:

      ```c#
      [Flags]
      public enum QuerySections
      {
        None = 0x00,
        Select = 0x01,
        From = 0x02,
        Where = 0x04,
        OrderBy = 0x08,
        NotOrderBy = All & ~OrderBy, 
        All = Select | From | Where | OrderBy,
      }
      ```
    The values `NotOrderBy` and `All` are aliases defined in terms of the other values. Note that the elements here are not aligned because it is expected that they will be documented, in which case column-alignment won’t make a difference in legibility.
* Avoid designing a bit-set when certain combinations of flags are invalid; in those cases, consider dividing the enumeration into two or more separate enumerations that are internally valid.

## Nested Types

* Nested types should not replace namespaces for organization.
* Use nested types if the inner type is logically within the other type (e.g. a `TableOfContents` class may have an `Options` inner class or a `Builder` inner class).
* Use nested types if the inner type should have access to all members of the outer type.
* Do not use public nested types unless you have a good reason for doing so (e.g. in the case of the `Options` class described above). 
* If a nested type needs a public constructor so that other types can create instances, then it probably shouldn’t be nested.
* Delegate declarations should not be nested within the type because this reduces re-use of delegate declarations between types.
* Use a nested type to group private or protected constants.

## Local Variables

* Declare a local variable as close as possible to its first use (and within the most appropriate scope).
* Local variables of the same type may be declared together, but only if they are not initialized. 

      ```c#
      IMetaEndpoint source, target;
      ```
* If a local variable is initialized, put the initialization on the same line as the declaration. If the line gets too long, use multiple lines as described in section 4.5 – Line Breaking.
* Local variables that need to be initialized cannot be declared on the same line unless they have the same initialization value.

      ```c#
      int startOfWord = firstCharacter = 0;
      ```

## Event Handlers

You should use the pattern and support classes for event-handling provided by the .NET library. 

* Do not expose delegates as `public` members; instead declare events using the `event` keyword.
* Do not add a method to a delegate with `new EventHandler(…);` instead, use delegate inference.
* Do not define custom delegates for event handling; instead use `EventHandler<T>`.
* Put all extra event data into an `EventArgs` descendent; subsequent versions can alter this descendent without changing the signature.
* Use `CancelEventArgs` as the base class if you need to be able to cancel an event.
* Neither the `sender` parameter nor the `args` parameter may be `null`; this avoids forcing event handlers to check for `null`.
* `EventsArgs` descendents should declare only properties, not methods or other application logic.

## Operators

* Be extremely careful when overloading operators; in general, you should only do so for `structs`. If you feel that an operator overload is especially clever, it probably isn’t; check with another developer before coding it.
* In other words: do not provide a conversion operator if such conversion is not clearly expected by the end users.
* Avoid overriding the == operator for reference types; override the `Equals()` method instead to avoid redefining reference equality. 
* If you do override Equals(), you should also override `GetHashCode()`.
* If you do override the == operator, consider overriding the other comparison operators (!=, <, <=, >, >=) as well.
* You should return false from the `Equals()` function if the objects cannot be compared. However, if they are different types, you may throw an exception.
* Do not mix and match conversion operators and types. The type `MetaString` can convert to `System.String` but should not convert to `System.Int32`. Instead, use a constructor to initialize from types that are not in the same domain.
* Use explicit conversions where the conversion may result in a loss of data; If the type to which the operator converts can represent all of the data from which it converts, feel free to define an implicit operator.
* Do not throw exceptions from implicit casts; implicit casts should only be used for operators that will never fail (or fail only in otherwise catastrophic situations).

## Loops & Conditions

### Loops

* Do not change the loop variable of a for-loop.
* Update while loop variables either at the beginning or the end of the loop.
* Keep loop bodies short and avoid excessive nesting.
* Consider using an inner class or other private methods if the body of a loop gets too complex.

### If Statements

* Do not compare to `true` or `false`; instead, compare pure Boolean expressions.
* Initialize Boolean values with simple expressions rather than using an if-statement; always use parentheses to delineate the assigned expression.

      ```c#
      bool needsUpdate = (Count > 0 && Objects[0].Modified);
      ```
* Always use brackets for flow-control blocks (`switch`, `if`, `while`, `for`, etc.)
* Do not add useless `else` blocks. An `if` statement may stand alone and an `else if` statement may be the last condition. [\[2\]](#footnote_2)

      ```c#
      if (a == b)
      {
        // Do something
      }
      else if (a > b)
      {
        // Do something else
      }

      // No final "else" required
      ``` 
* Do not force really complicated logic into an `if` statement; instead, use local variables to make the intent clearer. For example, imagine we have a lesson planner and want to find all unsaved lessons that are either unscheduled or are scheduled within a given time-frame. The following condition is too long and complicated to interpret quickly:

      ```c#
      if (!lesson.Stored && (((StartTime <= lesson.StartTime) && (lesson.EndTime <= EndTime)) || ! lesson.Scheduled))
      {
        // Do something with the lesson
      }
      ```
    Even trying to apply the line-breaking rules results in an unreadable mess:

      ```c#
      if (!lesson.Stored && 
        (((StartTime <= lesson.StartTime) && (lesson.EndTime <= EndTime)) || 
        ! lesson.Scheduled))
      {
        // Do something with the lesson
      }
      ```
    Even with this valiant effort, the intent of the ||-operator is difficult to discern. With local variables, however, the logic is much clearer:

      ```c#
      bool lessonInTimeSpan = ((StartTime <= lesson.StartTime) && (lesson.EndTime <= EndTime));
      if (!lesson.Stored && (lessonInTimeSpan || ! lesson.Scheduled))
      {
        // Do something with the lesson
      }
      ```

### Switch Statements

* Include a `default` statement that `asserts` or `throws` if all valid values are handled. This also applies for `enums` because the compiler does not realize that no default statement is needed.
* Use the following form when values initialized by the switch-statements are to be used elsewhere in the method.

      ```c#
      IDatabase result = null;
      switch (type)
      {
        case DatabaseType.PostgreSql:
          result = new PostgreSqlMetaDatabase();
          break;
        case DatabaseType.SqlServer:
          result = new SqlServerMetaDatabase();
          break;
        case DatabaseType.SQLite:
          result = new SqliteMetaDatabase();
          break;
        default:
          Debug.Assert(false, String.Format("Unknown database type: {0}", type));
      }

      // Work with "result".

      return result;
      ```
* In the case where the switch statement is the either the entire method or the final block in a method, use return statements directly from the case labels. In this case, the assertion is replaced with an exception or it won’t compile.

      ```c#
      switch (type)
      {
        case DatabaseType.PostgreSql:
          return new PostgreSqlMetaDatabase();
        case DatabaseType.SqlServer:
          return new SqlServerMetaDatabase();
        case DatabaseType.SQLite:
          return new SqliteMetaDatabase();
        default:
          throw new ArgumentException("type", String.Format("Unknown type: {0}", type));
      }
      ```
* The `default` label must always be the last label in the statement.

### Ternary and Coalescing Operators

The ternary operator is a specialized form of an `if`/`then` statement with the following form:

      ```c#
      return (_value != null) ? Value.ToString() : "NULL";
      ```
If the condition (`_value != null` in this case) is true, the operator returns the value after the question mark; otherwise, it returns the value after the colon.

The coalescing operator is a specialized form of the ternary operator, which has the following form:

      ```c#
      return Target ?? Source;
      ```
The operator returns the expression before the two question marks if it is not `null`; otherwise, it returns the expression after the two question marks.

* Use these operators for simple expressions and results.
* Do not use these operators with long conditions and values; instead, use an `if`/`then` statement.
* Do not break statements with these operators in them over multiple lines.

## Comments

### Formatting & Placement

* Comments are indented at the same level as the code they document.
* Place comments above the code being commented.

### Styles 

* Use the single-line comment style—`//`—to indicate a comment.
* Use four slashes —`////`—to indicate a single line of code that has been temporarily commented.
* Use the multi-line comment style—`/*` … `*/`—to indicate a commented-out block of code. In general, code should never be checked in with such blocks.
* Consider using a compiler variable to define a non-compiling block of code; this practice avoids misusing a comment.

      ```c#
      #if FALSE
            // commented code block
      #endif
      ```
* Use the single-line comment style with `TODO` to indicate an issue that must be addressed. Before a check-in, these issues must either be addressed or documented in the issue tracker, adding the URL of the issue to the TODO as follows:

      ```c#
      // TODO http://issue-tracker.encodo.com/?id=5647: [Title of the issue in the issue tracker]
      ```

### Content

* Good variable and method names go a long way to making comments unnecessary.
* Comments should be in US-English; prefer a short style that gets right to the point. 
* A comment need not be a full, grammatically-correct sentence. For example, the following comment is too long

      ```c#
      // Using a granularity that is more than 50% of the size causes a crash!

      int Granularity = Size / 5; 
      ```
    Instead, you should stick to the essentials so that the warning is immediately clear:

      ```c#
      int Granularity = Size / 5; // More than 50% causes a crash!
      ```
* Comments should be spellchecked. [\[3\]](#footnote_3)
* Comments should not explain the obvious. In the following example, the comment is superfluous.

      ```c#
      public const int Granularity = Size / 5; // granularity is 20% of size
      ```
* Use comments to explain algorithms or tricky bits that aren’t immediately obvious from a quick read.
* Use comments to indicate where a hard-won bug-fix was added; if possible, include a reference to a URL in an issue tracker.
* Use comments to indicate assumptions not already evident from assertions or thrown exceptions.
* Longer comments should always precede the line being commented. [\[4\]](#footnote_4)
* Short comments may appear to the right of the code being commented, but only for lines ending in semicolon (i.e. marking the end of a statement). For example:

      ```c#
      int Granularity = Size / 5; // More than 50% causes a crash!
      ```
* Comments on the same line as code should _never_ be wrapped to multiple lines.

## Grouping with `#region` Tags

* Use `#region` tags to distinguish groups of functions; use the auto-implement macro in _Visual Studio_ to ensure that interface implementations are surrounded in `#region` tags describing which interface is being implemented by the enclosed functions.
* Use regions for generated code blocks.
* You may use regions to demarcate logical groups of members or types.
* A region should generally enclose more than one element.

## Compiler Variables

* Avoid using `#define` in the code; use a compiler define in the project settings instead.
* Avoid suppressing compiler warnings.

### The [Conditional] Attribute

Use the `ConditionalAttribute` instead of the `#ifdef`/`#endif` pair wherever possible (i.e. for methods or classes).

      ```c#
      public class SomeClass
      {
        [Conditional("TRACE_ON")]
        public static void Msg(string msg)
        {
          Console.WriteLine(msg);
        }
      }
      ```
      
### \#if/#else/#endif

For other conditional compilation, use a static method in a static class instead of scattering conditional options throughout the code.

      ```c#
      public static class EncodoCompilerOptions
      {
        public static bool DeveloperBuild()
        {
      #if ENCODO_DEVELOPER
          return true;
      #else
          return false;
      #endif
        }
      }
      ```

This approach has the following advantages:

* The compiler checks all code paths instead of just the one satisfying the current options[\[5\]](#footnote_5); this avoids unknowingly retaining incompatible code in a library or application.
* Code formatting and indenting is not broken up by (possibly overlapping) compile conditions; the name `EncodoCompilerOptions` makes the connection to the compiler obvious enough.
* The compiler option is referenced only once, avoiding situations in which some code uses one compiler option (e.g. `ENCODO_DEVELOPER`) and other code uses another, misspelled option (e.g. `ENCODE_DEVELOPER`).

## Footnotes

1. <a name="footnote_1"></a>In scenarios that require a significant amount of boxing and un-boxing, value types perform poorly as compared to reference types.
1. <a name="footnote_2"></a>This is noted only because some style guides explicitly require that the last statement in an “if/else if” block is an empty “else” block if none is otherwise needed.
1. <a name="footnote_3"></a>CodeSpell is a good and relatively inexpensive spellchecker for Visual Studio 2005 and 2008; if you’re already using ReSharper, the Agent Smith plugin provides superb integration with multiple dictionaries.
1. <a name="footnote_4"></a>A newline separating the comment from its code is the recommended style, as it tends to separate the comments and the code into separate, but interleaved blocks. This is, however, just a suggestion.
1. <a name="footnote_5"></a>One drawback is that the editor doesn’t display the “unused” code as disabled, as it does when using compiler options directly.