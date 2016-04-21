# Formatting

The formatting rules were designed for use with C#. Where possible, they should be applied to other languages (CSS, JavaScript, etc.) as well.

## Indenting and Spacing

* An indent is two spaces ; it is never a tab.
* Use a single space after a comma (e.g. between function arguments).
* There is no space after the leading parenthesis/bracket or before the closing parenthesis/bracket.
* There is no space between a method name and the leading parenthesis, but there is a space before the leading parenthesis of a flow control statement.
* Use a single space to surround all  infix operators; there is no space between a prefix operator (e.g. “-” or “!”) and its argument.
* Do not use spacing to align type members on the same column (e.g. as with the members of an enumerated type).

## Braces

* Curly braces should—with a few exceptions outlined below—go on their own line.
* A line with only a closing brace should never be preceded by an empty line.
* A line with only an opening brace should never be followed by an empty line.

### Properties

* Simple getters and setters should go on the same line as all brackets.
* Abstract properties should have get, set and all braces on the same line
* Complex getters and setters should have each bracket on its own line.

This section applies to .NET 3.5 and newer.

* Prefer automatic properties as it saves a lot of typing and vastly improves readability.

### Methods

* Completely empty functions, like constructors, should have a space between brackets placed on the same line:

      ```
      SomeClass(string name)
        : base(name)
      { }
      ```

### Enumerations
* Use the trailing comma for the last member of an enumeration; this makes it easier to move them around, if needed.
### Return Statements
See 7.6 – Exit points (continue and return) for advice on how to use return statements.
* Use single-line, bracketed syntax for one-line returns with simple conditions:
if (Count != other.Count) { return false; }
* If a return statement is not the only statement in a method, it should be separated from other code by a single newline (or a line with only a bracket on it).
if (a == 1) { return true; }

return false;
* Do not use else with return statements (use the style shown above instead):
if (a == 1)
{
  return true;
}
else  // Not necessary
{
  return false;
}
### Switch Statements
* Contents under switch statements should be indented.
* Braces for a case-label are not indented; this maintains a nice alignment with the brackets from the switch-statement.
* Use braces for longer code blocks under case-labels; leave a blank line above the break-statement to improve clarity.
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
 
* Use braces to enforce tighter scoping for local variables used for only one case-label.
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
* If brackets are used for a case-label, the break-statement should go inside those brackets so that the bracket provides some white-space from the next case-label.
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
## Parentheses
* C# has a different operator precedence than Pascal or C, so you can write context != null && context.Count > 0 without confusing the compiler. However, you should use the form (context != null) && (context.Count > 0) for legibility’s sake.
* Do not use parentheses around the parameter(s) in a lambda expression.
* To make it more readable, use parentheses around the condition of a ternary expression if it uses an infix operator.
return (_value != null) ? Value.ToString() : "NULL";
* Prefix operators (e.g. “!”) and method calls should not have parentheses around them.
return !HasValue ? Value.ToString() : "EMPTY";
## Empty Lines
In the following list, the phrase “surrounding code” refers to a line consisting of more than just an opening or closing brace. That is, no new line is required when an element is at the beginning or end of a methods or other block-level element.
Always place an empty line in the following places:
* Between the file header and the namespace declaration or the first using statement.
* Between the last using statement and the namespace declaration.
* Between types (classes, structs, interfaces, delegates or enums).
* Between public, protected and internal members.
* Between preconditions and ensuing code.
* Between post-conditions and preceding code.
* Between a call to a base method and ensuing code.
* Between return statements and surrounding code (this does not apply to return statements at the beginning or end of methods).
* Between block constructs (e.g. while loops or switch statements) and surrounding code.
* Between documented enum values; undocumented may be grouped together.
* Between logical groups of code in a method; this notion is subjective and more a matter of style. You should use empty lines to improve readability, but should not overuse them.
* Between the last line of code in a block and a comment for the next block of code.
* Between statements that are broken up into multiple lines.
* Between a #region tag and the first line of code in that region.
* Between the last line of code in a region and the #endregion tag.
Do not place an empty line in the following places:
* After another empty line; the Encodo style uses only single empty lines.
* Between retrieval code and handling for that code. Instead, they should be formatted together.
IMetaReadableObject obj = context.Find<IMetaReadableObject>();
if (obj == null)
{
  context.Recorder.Log(Level.Fatal, String.Format("Error!”));
  return null;
}
* Between any line and a line that has only an opening or closing brace on it (i.e. there should be no leading or trailing newlines in a block).
* Between undocumented fields (usually private); if there are many such fields, you may use empty lines to group them by purpose.
## Line Breaking
* No line should exceed 100 characters; use the line-breaking rules listed below to break up a line.
* Use line-breaking only when necessary; do not adopt it as standard practice.
* If one or more line-breaks is required, use as few as possible.
* Line-breaking should occur at natural boundaries; the most common such boundary is between parameters in a method call or definition.
* Lines after such a line-break at such a boundary should be indented.
* The separator (e.g. a comma) between elements formatted onto multiple lines goes on the same line after the element; the IDE is much more helpful when formatting that way.
* The most natural line-breaking boundary is often before and after a list of elements. For example, the following method call has line-breaks at the beginning and end of the parameter list.
people.DataSource = CurrentCompany.Employees.GetList(
  connection, metaClass, GetFilter(), null
);
* If one of the parameters is much longer, then you add line-breaking between the parameters; in that case, all parameters are formatted onto their own lines:
people.DataSource = CurrentCompany.Employees.GetList(
  connection,
  metaClass,
  GetFilter("Global.Applications.Updater.PersonList.Search"),
  null
);
* Note in the examples above that the parameters are indented. If the assignment or method call was longer, they would no longer fit on the same line. In that case, you should use two levels of indenting.
Application.Model.people.DataSource = 
  Global.ApplicationEnvironment.CurrentCompany.Employees.GetList(
    connection,
    metaClass,
    GetFilter("Global.Applications.Updater.PersonList.Search"),
    null
  );
* If there is a logical grouping for parameters, you may apply line-breaking at those boundaries instead (breaking the all-on-one-line or each-on-its-own-line rule stated above). For example, the following method specifies Cartesian coordinates:
Geometry.PlotInBox(
  "Global.Applications.MainWindow",
  topLeft.X, topLeft.Y,
  bottomRight.X, bottomRight.Y
);
 
### Method Calls
* The closing parenthesis of a method call goes on its own line to “close” the block (see example below).
result.Messages.Log(
  Level.Error, 
  String.Format(
    "Class [{0}] has the same metaid as class [{1}].", 
    dbCls.Identifier, 
    classMap[cls.MetaId]
  )
);
* If the result of calling a method is assigned to a variable, the call may be on the same line as the assignment if it fits.
people.DataSource = CurrentCompany.Employees.GetList(
  connection,
  ViewAspectTools.GetViewableWrapper(cls),
  GetFilter().Where(String.Format(“PersonId = {0}”, personId)),
  null
);
* If the call does not fit easily—or if the function call is “too far away” from the ensuing parameters, you should move the call to its own line and indent it:
WindowTools.GetActiveWindow().GetActivePanel().GetActiveList().DataSource = 
  CurrentCompany.Organization.MainOffice.Employees.GetList(
    connection,
    ViewAspectTools.GetViewableWrapper(cls),
    GetFilter().Where(String.Format(“PersonId = {0}”, personId)),
    null
  );
### Method Definitions
* Stay consistent with line-breaking in related methods within a class; if one is broken up onto multiple lines, then all related methods should be broken up onto multiple lines.
* The closing brace of a method definition goes on the same line as the last parameter (unlike method calls). This avoids having a line with a solitary closing parenthesis followed by a line with a solitary opening brace.
public static void SetupLookupDefinition(
  RepositoryItemLookUpEdit lookupOptions,
  IMetaClass metaClass)
{
  // Implementation...
}
 
* Generic method constraints should be specified on their own line, with a single indent.
string GetNames<T>(IMetaCollection<T> elements, string separator, NameOption options) 
  where T : IMetaBase;
* The generic method constraint should line up with the parameters, if they are specified on their own lines.
public static void SetupLookupFromData<T>(
  RepositoryItemLookUpEdit lookupOptions, 
  IDataList<T> dataList) 
  where T : IMetaReadable
{
  SetupLookupFromData<T>(lookupOptions, dataList, dataList.MetaClass);
}
### Multi-Line Text
* Longer string-formatting statements with newlines should be formatted using the @-operator and should avoid using concatenation:
result.SqlText = String.Format(
  @"FROM person
      LEFT JOIN
        employee 
          ON person.employeeid = employee.id
      LEFT JOIN
        company
          ON person.companyid = company.id
      LEFT JOIN
        program
          ON company.programid = program.id     
      LEFT JOIN
        settings
          ON settings.programid = program.id
    WHERE
       program.id = {0} AND person.hiredate <= '{2}';
  ", 
  settings.ProgramId, 
  state, 
  offset.ToString("yyyy-MM-dd")
);
* If the indenting in the string argument above is important, you may break indenting rules and place the text all the way to the left of the source in order to avoid picking up extra, unwanted spaces. However, you should consider externalizing such text to resources or text files.
* The trailing double-quote in the example above is not required, but is permitted; in this case, the code needs to include a newline at the end of the SQL statement.
 
### Chained Method Calls
* Chained method calls can be formatted onto multiple lines; if one chained function call is formatted onto its own line, then they should all be.
string contents = header.
  Replace("{Year}", DateTime.Now.Year.ToString()).
  Replace("{User}", "ENCODO").
  Replace("{DateTime}", DateTime.Now.ToString());
* If a line of a chained method call opens a new logical context, then ensuing lines should be indented to indicate this. For example, the following example joins tables together, with the last three statements applied to the last joined table. The indenting helps make this clear.
query.
  Join(Settings.Relations.Company).
  Join(Company.Relations.Office).
  Join(Office.Relations.Employees).
    WhereEquals(Employee.Fields.Id, employee.Id)
    OrderBy(Employee.Fields.LastName, SortDirection.Ascending)
    OrderBy(Employee.Fields.FirstName, SortDirection.Ascending);
### Anonymous Delegates
* All rules for standard method calls also apply to method calls with delegates.
* Anonymous delegates are always written on multiple lines for clarity.
* Do not use parentheses for anonymous delegates if there are no parameters.
* Anonymous delegates should be written with an indent, as follows:
IMetaCollection<IMetaProperty> persistentProps = 
  model.ReferencedProperties.FindAll(
    delegate(IMetaProperty prop) 
    { 
      return prop.Persistent; 
    }
  );
* Even very short delegates benefit from writing in this fashion (the alternative is much messier and not so obviously a delegate when browsing through the code):
public string[] Keys
{
  get
  {
    return ToStrings(
      delegate(T item) 
      { 
        return item.Identifier; 
      }
    );
  }
}
 
* This notation is also useful for long function calls with many or long parameters. If, for example, a delegate is one of the parameters, then you should make a block out of the whole function call, like this:
_context = new DataContext(
  Settings.Default.ConfigFileName, 
  DatabaseType.PostgreSql,
  delegate
  {
    return ModelGenerator.CreateModel();
  }
);
In the example above each parameter is on its own line, as required.
* Here’s a fancy one, with a delegate in a constructor base; note that the closing parenthesis is on the same line as the closing brace of the delegate definition.
public Application()
  : base(
      DatabaseType.PostgreSql, 
      delegate() 
      { 
        return ModelGenerator.CreateModel(); 
      })
    { }
### Lambda Expressions 
This section applies to .NET 3.5 and newer.
* All rules for standard method calls also apply to method calls with lambda expressions.
* Very short lambda expressions may be written as a simple parameter on the same line as the method call:
ReportError(msg => MessageBox.Show(msg));
* Longer lambda expressions should go on their own line, with the closing parenthesis of the method call closing the block on another line. Any calls attached to the result—like ToList() or Count()—should go on the same line as the closing parenthesis.
people.DataSource = CurrentCompany.Employees.Where(
  item => item.LessonTimeId == null
).ToList();
* Longer lambda expressions should not be both wrapped and used in a foreach-statement; instead, use two statements as shown below.
var appointmentsForDates = data.Appointments.FindAll(
  appt => (appt.StartTime >= startDate) && (appt.EndTime <= endDate)
);

foreach (var appt in appointmentsForDates)
{
  // Do something with each appointment
}
 
### Object Initializers
This section applies to .NET 3.5 and newer.
Longer initialization blocks should go on their own line; the rest can be formatted in the following ways (depending on line-length and preference):
* Shorter initialization blocks (one or two properties) can be specified on the same line:
var personOne = new Person { LastName = "Miller", FirstName = "John" };
* The new-clause is on the same line as the declaration:
IViewPropertySizeAspect sizeAspect = new ViewPropertySizeAspect
{
  VerticalSizeMode = SizeMode.Absolute,
  VerticalUnits = height
};

prop.Aspects.Add(sizeAspect);
* The new-clause is on its own line:
IViewPropertySizeAspect sizeAspect = 
  new ViewPropertySizeAspect
  {
    VerticalSizeMode = SizeMode.Absolute,
    VerticalUnits = height
  };

prop.Aspects.Add(sizeAspect);
* The initializer is nested within the method-call on the same line as the declaration:
prop.Aspects.Add(new ViewPropertySizeAspect { VerticalUnits = height });
* The initializer is nested within the method-call on its own line:
prop.Aspects.Add(
  new ViewPropertySizeAspect
  {
    VerticalSizeMode = SizeMode.Absolute,
    VerticalUnits = height
  });
If the initializer goes spans multiple lines, then the new-clause must also go on its own line.
### Array Initializers
This section applies to .NET 3.5 and newer.
The type is usually optional (unless you’re initializing an empty array), so you should leave it empty.
### Ternary and Coalescing Operators
* Do not use line-breaking to format statements containing ternary and coalescing operators; instead, convert to an if/else statement.
