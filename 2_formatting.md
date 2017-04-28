# Formatting

## Whitespace and Symbols

### Blank Lines

In the following list, the phrase “surrounding code” refers to a line consisting of more than just an opening or closing brace. That is, no new line is required when an element is at the beginning or end of a methods or other block-level element.
Always place an empty line in the following places:

* Between the file header and the `namespace` declaration or the first `using` statement.
* Between the last `using` statement and the `namespace` declaration.
* Between types (`classes`, `structs`, `interfaces`, `delegates` or `enums`).
* Between public, protected and internal members.
* Between preconditions and ensuing code.
* Between post-conditions and preceding code.
* Between a call to a `base` method and ensuing code.
* Between `return` statements and surrounding code.
* Between block constructs (e.g. `while` loops or `switch` statements) and surrounding code.
* Between documented `enum` values; undocumented values may be grouped together.
* Between logical groups of code in a method; this notion is subjective and more a matter of style. You should use empty lines to improve readability, but should not overuse them.
* Between the last line of code in a block and a comment for the next block of code.
* Between statements that are broken up into multiple lines.
* Between a `#region` tag and the first line of code in that region. See next section.
* Between the last line of code in a region and the `#endregion` tag. See next section.

Do not place an empty line in the following places:

* After another empty line.
* Between retrieval code and handling for that code. Instead, they should be formatted together.
  ```csharp
  IMetaReadableObject obj = context.Find<IMetaReadableObject>();
  if (obj == null)
  {
    context.Recorder.Log(Level.Fatal, String.Format("Error!"));

    return null;
  }
  ```
* Between any line and a line that has only an opening or closing brace on it (i.e. there should be no leading or trailing newlines in a block).
* Between undocumented fields (usually private); if there are many such fields, you may use empty lines to group them logically.

### Line Breaks

* Use line-breaking only when necessary, as outlined below.
* No line should exceed 120 characters.
* Use as few line-breaks as possible.
* Line-breaking should occur at natural boundaries; the most common such boundary is between parameters in a method call or definition.
* A line-break at a boundary that defines a new block should be indented one more level.
* A line-break at any other boundary should be indented at the same level as the original line.
* The separator (e.g. a comma) between elements formatted onto multiple lines goes on the same line after the element. The IDE is much more helpful when formatting that way.
* The most natural line-breaking boundary is often before and after a list of elements. For example, the following method call has line-breaks at the beginning and end of the parameter list.
  ```csharp
  people.DataSource = CurrentCompany.Employees.GetList(
    connection, metaClass, GetFilter(), null
  );
  ```
* If one of the parameters is much longer, then you add line-breaking between the parameters; in that case, all parameters are formatted onto their own lines:
  ```csharp
  people.DataSource = CurrentCompany.Employees.GetList(
    connection,
    metaClass,
    GetFilter("Global.Applications.Updater.PersonList.Search"),
    null
  );
  ```
* Note in the examples above that the parameters are indented. If the assignment or method call was longer, they would no longer fit on the same line. In that case, you should use two levels of indenting.
  ```csharp
  Application.Model.people.DataSource =
    Global.ApplicationEnvironment.CurrentCompany.Employees.GetList(
      connection,
      metaClass,
      GetFilter("Global.Applications.Updater.PersonList.Search"),
      null
    );
  ```
* Even if there is a logical grouping for parameters, you should still apply line-breaking using the all-on-one-line or each-on-its-own-line rules stated above. For example, the following method specifying Cartesian coordinates feels natural, but is not well-supported by automatic formatting rules:
  ```csharp
  Geometry.PlotInBox(
    "Global.Applications.MainWindow",
    topLeft.X, topLeft.Y,
    bottomRight.X, bottomRight.Y
  );
  ```
  As nice as it might look, _do not use this line-breaking technique._

### Indenting and Spacing

* An indent is two spaces.
* Use a single space after a comma (e.g. between function arguments).
* There is no space after the leading parenthesis/bracket or before the closing parenthesis/bracket.
* There is no space between a method name and the leading parenthesis, but there is a space before the leading parenthesis of a flow-control statement.
* Use a single space to surround _all_ infix operators; there is no space between a prefix operator (e.g. “-” or “!”) and its single parameter.
* Do not use spacing to align type members on the same column (e.g. as with the explicitly assigned values for members of an enumerated type).

### Braces

* Curly braces should—with a few exceptions outlined below—go on their own line.
* A line with only an opening brace should never be followed by an empty line.
* A line with only a closing brace should never be preceded by an empty line.

#### Properties

* Simple getters and setters should go on the same line as all brackets.
* Abstract properties should have get, set and all braces on the same line.
* Complex getters and setters should have each bracket on its own line.
* Place Auto-Property Initializers on the same line.
  ```csharp
  public int Maximum { get; } = 45;
  ```

#### Methods

* Completely empty methods should have brackets placed on separate lines:
  ```csharp
  SomeClass(string name)
    : base(name)
  {
  }
  ```

### Parentheses

* Use parentheses only to improve clarity.
* Do not use parentheses for simple expressions. The following expression is clear enough without extra parentheses.
  ```csharp
  if (context != null && context.Count > 0)
  {
  }
  ```

## Language Elements

### Methods

#### Definitions

* Stay consistent with line-breaking in related methods within a class; if one is broken up onto multiple lines, then all related methods should be broken up onto multiple lines.
* The closing brace of a method definition goes on the same line as the last parameter (unlike method calls). This avoids having a line with a solitary closing parenthesis followed by a line with a solitary opening brace.
  ```csharp
  public static void SetupLookupDefinition(
    RepositoryItemLookUpEdit lookupOptions,
    IMetaClass metaClass)
  {
    // Implementation...
  }
  ```
* Generic method constraints should always be on their own line, with a single indent.
  ```csharp
  string GetNames<T>(IMetaCollection<T> elements, string separator, NameOption options
    where T : IMetaBase;
  ```
* The indent for a generic-method constraint stays the same, even with wrapped parameters.
  ```csharp
  public static void SetupLookupFromData<T>(
    RepositoryItemLookUpEdit lookupOptions,
    IDataList<T> dataList)
    where T : IMetaReadable
  {
    SetupLookupFromData<T>(lookupOptions, dataList, dataList.MetaClass);
  }
  ```

#### Calls

* The closing parenthesis of a method call goes on its own line to “close” the block (see example below).
  ```csharp
  result.Messages.Log(
    Level.Error,
    String.Format(
      "Class [{0}] has the same name as class [{1}].",
      dbCls.Identifier,
      classMap[cls.MetaId]
    )
  );
  ```
* If the result of calling a method is assigned to a variable, the call may be on the same line as the assignment if it fits.
  ```csharp
  people.DataSource = CurrentCompany.Employees.GetList(
    connection,
    ViewAspectTools.GetViewableWrapper(cls),
    GetFilter().Where(String.Format(“PersonId = {0}”, personId)),
    null
  );
  ```
* If the call does not fit easily—or if the method call is “too far away” from the ensuing parameters—you should move the call to its own line and indent it:
  ```csharp
  WindowTools.GetActiveWindow().GetActivePanel().GetActiveList().DataSource =
    CurrentCompany.Organization.MainOffice.Employees.GetList(
      connection,
      ViewAspectTools.GetViewableWrapper(cls),
      GetFilter().Where(String.Format(“PersonId = {0}”, personId)),
      null
    );
  ```

#### Chaining

* Chained method calls can be formatted onto multiple lines; if one chained method-call is formatted onto its own line, then they must all be on separate lines.
  ```csharp
  string contents = header
    .Replace("{Year}", DateTime.Now.Year.ToString())
    .Replace("{User}", "ENCODO")
    .Replace("{DateTime}", DateTime.Now.ToString());
  ```
* If a line of a chained method-call opens a new logical context, then ensuing lines should be indented to indicate this. For example, the following example joins tables together, with the last three statements applied to the last joined table. The indenting helps make this clear.
  ```csharp
  query
    .Join(Settings.Relations.Company)
    .Join(Company.Relations.Office)
    .Join(Office.Relations.Employees)
      .WhereEquals(Employee.Fields.Id, employee.Id)
      .OrderBy(Employee.Fields.LastName, SortDirection.Ascending)
      .OrderBy(Employee.Fields.FirstName, SortDirection.Ascending);
  ```

### Constructors

* Base constructors should be on a separate line, indented one level.
  ```csharp
  public class B : A
  {
    B(string name)
      : base(name)
    {
    }
  }
  ```

### Initializers

* Don't add a trailing comma for the last element.

#### Object Initializers

Longer initialization blocks should go on their own line; the rest can be formatted in the following ways (depending on line-length and preference):

* Shorter initialization blocks (one or two properties) can be specified on the same line:
  ```csharp
  var personOne = new Person { LastName = "Miller", FirstName = "John" };
  ```
* The `new`-clause is on the same line as the declaration:
  ```csharp
  var sizeAspect = new ViewPropertySizeAspect
  {
    VerticalSizeMode = SizeMode.Absolute,
    VerticalUnits = height
  };

  prop.Aspects.Add(sizeAspect);
  ```
* The new-clause is on its own line:
  ```csharp
  var sizeAspect =
    new ViewPropertySizeAspect
    {
      VerticalSizeMode = SizeMode.Absolute,
      VerticalUnits = height
    };

  prop.Aspects.Add(sizeAspect);
  ```
* The initializer is nested within the method-call on the same line as the declaration:
  ```csharp
  prop.Aspects.Add(new ViewPropertySizeAspect { VerticalUnits = height });
  ```
* The initializer is nested within the method-call on its own line:
  ```csharp
  prop.Aspects.Add(
    new ViewPropertySizeAspect
    {
      VerticalSizeMode = SizeMode.Absolute,
      VerticalUnits = height
    });
  ```
* Putting the `new` operator on the same line is also fine, but then you shouldn't indent the braces.
  ```csharp
  prop.Aspects.Add(new ViewPropertySizeAspect
  {
    VerticalSizeMode = SizeMode.Absolute,
    VerticalUnits = height
  });
  ```

If the initializer goes spans multiple lines, then the new-clause must also go on its own line.

#### Array Initializers

The type is usually optional (unless you’re initializing an empty array), so you should leave it empty.

### Lambdas

* Do not use anonymous delegates; use lambda notation instead.
* Do not use parentheses around a single parameter in a lambda expression.
* All rules for standard method calls also apply to method calls with lambdas.
* Longer lambdas should be written with an indent, as follows:
  ```csharp
  var persistentProperties =
    model.ReferencedProperties.FindAll(
      prop =>
      {
        // More code...

        return prop.Persistent;
      }
    );
  ```
* Short lambdas benefit can just be inlined:
  ```csharp
  public string[] Keys
  {
    get
    {
      return ToStrings(i => i.Name);
    }
  }
  ```
  Also OK, since the `get` body is short:
  ```csharp
  public string[] Keys
  {
    get { return ToStrings(i => i.Name); }
  }
  ```
  Even better, use expression-bodied members:
  ```csharp
  public string[] Keys => ToStrings(i => i.Name);
  ```
* Short lambdas are just parameters; treat them as you would any other parameters:
  ```csharp
  _context = new DataContext(
    Settings.Default.ConfigFileName,
    DatabaseType.PostgreSql,
    () => ModelGenerator.CreateModel()
  );
  ```
  In the example above each parameter is on its own line, as required.
* Keep parameters short in constructor bases.
  ```csharp
  public Application()
    : base(DatabaseType.PostgreSql, () => ModelGenerator.CreateModel())
  {
  }
  ```
* All rules for standard method calls also apply to method calls with lambda expressions.
* Very short lambda expressions may be written as a simple parameter on the same line as the method call:
  ```csharp
  ReportError(msg => MessageBox.Show(msg));
  ```
* Longer lambda expressions should go on their own line, with the closing parenthesis of the method call closing the block on another line. Any calls attached to the result—like `ToList()` or `Count()`—should go on the same line as the closing parenthesis.
  ```csharp
  people.DataSource = CurrentCompany.Employees.Where(
    item => item.LessonTimeId == null
  ).ToList();
  ```
* Longer lambda expressions should not be both wrapped and used in a `foreach`-statement; instead, use two statements as shown below. Use short parameter names in lambdas since they are used very close to their declaration (by definition).
  ```csharp
  var appointmentsForDates = data.Appointments.FindAll(
    a => a.StartTime >= startDate && a.EndTime <= endDate
  );

  foreach (var appointment in appointmentsForDates)
  {
    // Do something with each appointment
  }
  ```

### Multi-Line Text

* Longer string-formatting statements with newlines should be formatted using the verbatim strings (`@""`) and should avoid using concatenation:
  ```csharp
  result.SqlText = String.Format(
    @"FROM person
        LEFT JOIN
          employee
            ON person.employee_id = employee.id
        LEFT JOIN
          company
            ON person.company_id = company.id
        LEFT JOIN
          program
            ON company.program_id = program.id
        LEFT JOIN
          settings
            ON settings.program_id = program.id
      WHERE
        program.id = {0} AND person.hire_date <= '{2}';
    ",
    settings.ProgramId,
    state,
    offset.ToString("yyyy-MM-dd")
  );
  ```
* If the indenting in the string argument above is important, you may break indenting rules and place the text all the way to the left of the source in order to avoid picking up extra, unwanted spaces. However, you should consider externalizing such text to resources or text files.
* The trailing double-quote in the example above is not required, but is permitted; in this case, the code needs to include a newline at the end of the SQL statement.

### `return` Statements

* If a `return` statement is not the only statement in a method, it should be separated from other code by a single newline.
* Always use multi-line formatting for `return` statements so they're easy to see.
  ```csharp
  if (Count != other.Count)
  {
    return false;
  }
  ```
* Do _not_ use else with `return` statements.
  ```csharp
  if (a == 1)
  {
    return true;
  }
  else  // Not necessary
  {
    return false;
  }
  ```
  Instead, you should write the `return` as shown below.
  ```csharp
  if (a == 1)
  {
    return true;
  }

  return false;
  ```
  In this case, return the condition instead.
  ```csharp
  return a == 1;
  ```

### `switch` Statements

The following rules apply for all `switch` statements, including pattern-matching.

* Contents under `switch` statements should be indented.
* Braces for a case-label are not indented; this maintains a nice alignment with the brackets from the switch-statement.
* Use braces for longer code blocks under case-labels; leave a blank line above the break-statement to improve clarity.
  ```csharp
  switch (flavor)
  {
    case Flavor.Up:
    case Flavor.Down:
    {
      if (someConditionHolds)
      {
        // Do some work
      }

      // Do some more work

      break;
    }
    default:
      break;
  }
  ```
* Use braces to enforce tighter scoping for local variables used for only one `case`-label.
  ```csharp
  switch (flavor)
  {
    case Flavor.Up:
    case Flavor.Down:
    {
      int quarkIndex = 0; // valid within scope of case statement
      break;
    }
    case Flavor.Charm:
    case Flavor.Strange:
      int isTopOrBottom = false;  // valid within scope of switch statement
      break;
    default:
      break;
  }
  ```
* If brackets are used for a case-label, the break-statement should go inside those brackets so that the bracket provides some white-space from the next case-label.
  ```csharp
  switch (flavor)
  {
    case Flavor.Up:
    case Flavor.Down:
    {
      int quarkIndex = 0;
      break;
    }
    case Flavor.Charm:
    case Flavor.Strange:
    {
      int isTopOrBottom = false;
      break;
    }
    default:
    {
      handled = false;
      break;
    }
  }
  ```

### Ternary and Coalescing Operators

* Do not use line-breaking to format statements containing ternary and coalescing operators; instead, convert to an `if`/`else` statement.
* Do not use parentheses around the condition of a ternary expression. If the condition is not immediately recognizable, extract it to a local variable.
  ```csharp
  return _value != null ? _value.ToString() : "NULL";
  ```
* Prefix operators (e.g. `!`) and method calls should not have parentheses around them.
  ```csharp
  return !HasValue ? Value.ToString() : "EMPTY";
  ```

### Comments

* Place comments above referenced code.
* Indent comments at the same level as referenced code.

### Regions

* Do not use regions.
* A historical usage of `#region` tags is to delineate code regions to be ignored by ReSharper in generated files. Instead, tell ReSharper to ignore files with the pattern of the generated filenames (e.g. `*.Class.cs`).
