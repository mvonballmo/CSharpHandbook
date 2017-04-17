# Comments

## Formatting & Placement

* Place comments above referenced code.
* Indent comments at the same level as referenced code.

## Styles

* Use the single-line comment style—`//`—to indicate a comment.
* Use four slashes —`////`—to indicate a single line of code that has been temporarily commented.
* Use the multi-line comment style—`/*` … `*/`—to indicate a commented-out block of code. In general, code should not be checked in with such blocks.
* Consider using a compiler variable to define a non-compiling block of code; this practice avoids misusing a comment.
  ```csharp
  #if FALSE
        // commented code block
  #endif
  ```
* Use the single-line comment style with `TODO` to indicate an issue that must be addressed. Before a check-in, these issues must either be addressed or documented in the issue tracker, adding the URL of the issue to the TODO as follows:
  ```csharp
  // TODO http://issue-tracker.encodo.com/?id=5647: [Title of the issue in the issue tracker]
  ```

## Content

* Good variable and method names go a long way to making comments unnecessary.
* Comments should be in US-English; prefer a short style that gets right to the point.
* A comment need not be a full, grammatically-correct sentence. For example, the following comment is too long
  ```csharp
  // Using a granularity that is more than 50% of the size is not valid!
  int Granularity = Size / 5;
  ```
  Instead, you should stick to the essentials so that the warning is immediately clear:
  ```csharp
  int Granularity = Size / 5; // More than 50% is not valid!
  ```
* Comments should be spellchecked.
* Comments should not explain the obvious. In the following example, the comment is superfluous.
  ```csharp
  public const int Granularity = Size / 5; // granularity is 20% of size
  ```
* Use comments to explain algorithms or tricky bits that aren't immediately obvious from a quick read.
* Use comments to indicate where a hard-won bug-fix was added; if possible, include a reference to a URL in an issue tracker.
* Use comments to indicate assumptions not already evident from assertions or thrown exceptions.
* Longer comments should always precede the line being commented. Separate multi-line comments with an additional newline before the code.
* Short comments may appear to the right of the code being commented, but only for lines ending in semicolon (i.e. marking the end of a statement). For example:
  ```csharp
  int Granularity = Size / 5; // More than 50% is not valid!
  ```
* Comments on the same line as code should _never_ be wrapped to multiple lines.
* Replace comments with private methods with descriptive names. For example, the following code is commented, but a bit wordy.
  ```csharp
  public void MethodOne()
  {
    // Collect and aggregate results
    var projections = new List<Result>();
    foreach (var p in OriginalProjections)
    {
      // Do a bunch of stuff with p
      projections.Add(new Projection(p, i))
    }

    // Format the projections into a text report
    var lines = new List<string>();
    foreach (var projection in projections)
    {
      var line = string.Format($"Some text with a {projection}");

      // Work with the line

      lines.Add(line);
    }

    // Save the lines to file
    File.WriteAllLines(OutputPath, lines);
  }
  ```
  Instead, use methods to achieve the same clarity without comments. At the same time, we make use of better types (`IEnumerable<T>`) and constructs (`Select()`, `NotNull`) to streamline and improve the code even more.
  ```csharp
  public void MethodOne()
  {
    var projections = CalculateFinalProjections();
    var lines = CreateReport(projections);

    StoreReport(lines);
  }

  [NotNull]
  private IEnumerable<Projection> CalculateFinalProjections()
  {
    return OriginalProjections.Select(CreateFinalProjection);
  }

  [NotNull]
  private Projection CreateFinalProjection([NotNull] Projection p)
  {
    if (p == null) { throw new ArgumentNullException(nameof(p)); }

    // Do a bunch of stuff with p

    return new Projection(p, i));
  }

  [NotNull]
  private IEnumerable<string> CreateReport([NotNull] IEnumerable<Projection> projections)
  {
    if (projections == null) { throw new ArgumentNullException(nameof(projections)); }

    return projections.Select(p => string.Format($"Some text with a {p}"));
  }

  private void StoreReport([NotNull] IEnumerable<string> lines)
  {
    if (lines == null) { throw new ArgumentNullException(nameof(lines)); }

    File.WriteAllLines(OutputPath, lines);
  }
  ```
  This version obviously needs no comments: it is now clear what `MethodOne` does. In fact, we can streamline `MethodOne` to a single line without losing legibility. Using the syntactic constructs of the language eliminates the need for many, if not all, comments.
  ```csharp
  public void MethodOne()
  {
    StoreReport(CreateReport(CalculateFinalProjections()));
  }
  ```
* Replace comments with local variables with descriptive names.
  ```csharp
  // For new lessons that are within the time-span or not scheduled
  if (!lesson.Stored && ((StartTime <= lesson.StartTime && lesson.EndTime <= EndTime) || !lesson.Scheduled)) { }
  ```
  With local variables, however, the logic is much clearer and no comment is required:
  ```csharp
  bool lessonInTimeSpan = StartTime <= lesson.StartTime && lesson.EndTime <= EndTime;

  if (!lesson.Stored && (lessonInTimeSpan || !lesson.Scheduled)) { }
  ```