# `continue` statements

* Do not use `continue`.
* The following example is not allowed.
  ```csharp
  foreach (var search in searches)
  {
    if (!search.Path.Contains("CN="))
    {
      continue;
    }

    // Work with valid searches
  }
  ```
  Instead, use a condition to filter elements.
  ```csharp
  foreach (var search in searches)
  {
    if (search.Path.Contains("CN="))
    {
      // Work with valid searches
    }
  }
  ```
  Even better, use `Where()` to filter elements.
  ```csharp
  foreach (var search in searches.Where(s => s.Path.Contains("CN=")))
  {
    // Work with valid searches
  }
  ```
