# Safe Programming

* Use static typing wherever possible.

## Be Functional

* Make data immutable wherever possible.
* Make methods pure wherever possible.

## Avoid `null` references

* Make references non-nullable wherever possible.
* Use the `[NotNull]` attribute for parameters, fields and results. Enforce it with a runtime check.
* Always test parameters, local variables and fields that can be `null`.

Instead of allowing `null` for a parameter, avoid null-checks with a null implementation.

```csharp
interface ILogger
{
  bool Log(string message);
}

class NullLogger : ILogger
{
  void Log(string message)
  {
    // NOP
  }
}
```

## Local variables

* Do not re-use local variable names, even though the scoping rules are well-defined and allow it. This prevents surprising effects when the variable in the inner scope is removed and the code continues to compile because the variable in the outer scope is still valid.
* Do not modify a variable with a prefix or suffix operator more than once in an expression. The following statement is not allowed:
  ```csharp
  items[propIndex++] = ++propIndex;
  ```

## Side Effects

A side effect is a change in an object as a result of reading a property or calling a method that causes the result of the property or method to be different when called again.

* Prefer pure methods.
* Void methods have side effects by definition.
* Writing a property must cause a side effect.
* Reading a property should not cause a side effect. An exception is lazy-initialization to cache the result.
* Avoid writing methods that return results _and_ cause side effects. An exception is lazy-initialization to cache the result.

## ”Access to Modified Closure”

`IEnumerable<T>` sequences are evaluated lazily. ReSharper will warn of multiple enumeration.

You can accidentally change the value of a captured variable before the sequence is evaluated. Since _ReSharper_ will complain about this behavior even when it does not cause unwanted side-effects, it is important to understand which cases are actually problematic.

```csharp
var data = new[] { "foo", "bar", "bla" };
var otherData = new[] { "bla", "blu" };
var overlapData = new List<string>();

foreach (var d in data)
{
  if (otherData.Where(od => od == d).Any())
  {
    overlapData.Add(d);
  }
}

Assert.That(overlapData.Count, Is.EqualTo(1)); // "bla"
```

The reference to the variable `d` will be flagged by _ReSharper_ and marked as an _“access to a modified closure”_. This indicates that a variable referenced—or “captured”—by the lambda expression—closure—will have the last value assigned to it rather than the value that was assigned to it when the lambda was created.

In the example above, the lambda is created with the first value in the sequence, but since we only use the lambda once, and then always before the variable has been changed, we don’t have to worry about side-effects. _ReSharper_ can only detect that a variable referenced in a closure is being changed within its scope.

Even though there isn’t a problem in this case, rewrite the `foreach`-statement above as follows to eliminate the _access to modified closure_ warning.

```csharp
var data = new[] { "foo", "bar", "bla" };
var otherData = new[] { "bla", "blu" };
var overlapData = data.Where(d => otherData.Where(od => od == d).Any()).ToList();

Assert.That(overlapData.Count, Is.EqualTo(1)); // "bla"
```

Finally, use library functionality wherever possible. In this case, we should use `Intersect` to calculate the overlap (intersection).

```csharp
var data = new[] { "foo", "bar", "bla" };
var otherData = new[] { "bla", "blu" };
var overlapData = data.Intersect(otherData).ToList();

Assert.That(overlapData.Count, Is.EqualTo(1)); // "bla"
```

Remember to be aware of how items are compared. The `Intersects` method above compares using `Equals`, not reference-equality.

The following example does not yield the expected result:

```csharp
var data = new[] { "foo", "bar", "bla" };

var threshold = 2;
var twoLetterWords = data.Where(d => d.Length == threshold);

threshold = 3;
var threeLetterWords = data.Where(d => d.Length == threshold);

Assert.That(twoLetterWords.Count(), Is.EqualTo(0));
Assert.That(threeLetterWords.Count(), Is.EqualTo(3));
```

The lambda in `twoLetterWords` _references_ `threshold`, which is then changed before the lambda is evaluated with `Count()`. There is nothing wrong with this code, but the results can be surprising. Use `ToList()` to evaluate the lambda in `twoLetterWords` _before_ the threshold is changed.

```csharp
var data = new[] { "foo", "bar", "bla" };
var threshold = 2;
var twoLetterWords = data.Where(d => d.Length == threshold).ToList();

threshold = 3;
var threeLetterWords = data.Where(d => d.Length == threshold);

Assert.That(twoLetterWords.Count(), Is.EqualTo(0));
Assert.That(threeLetterWords.Count(), Is.EqualTo(3));
```

## "Collection was modified; enumeration operation may not execute."

Changing a sequence during enumeration causes a runtime error.

The following code will fail whenever `data` contains an element for which `IsEmpty` returns `true`.

```csharp
foreach (var d in data.Where(d => d.IsEmpty))
{
  data.Remove(d);
}
```

To avoid this problem, use an in-memory copy of the sequence instead. A good practice is to use `ToList()` to create the copy and to call it in the `foreach` statement so that it's clear why it's being used.

```csharp
foreach (var d in data.Where(d => d.IsEmpty).ToList())
{
  data.Remove(d);
}
```

## "Possible multiple enumeration of IEnumerable"

Suppose, in the example above, that we also want to know how many elements were empty. Let's start by extracting `emptyElements` to a variable.

```csharp
var emptyElements = data.Where(d => d.IsEmpty);
foreach (var d in emptyElements.ToList())
{
  data.Remove(d);
}

return emptyElements.Count();
```

Since `emptyElements` is evaluated lazily, the call to `Count()` to return the result will evaluate the iterator again, producing a sequence that is now empty—because the `foreach`-statement removed them all from data. The code above will always return zero.

A more critical look at the code above would discover that the `emptyElements` iterator is triggered twice: by the call to `ToList()` and `Count()` (ReSharper will helpfully indicate this with an inspection). Both `ToList()` and `Count()` logically iterate the entire sequence.

To fix the problem, we lift the call to `ToList()` out of the `foreach` statement and into the variable.

```csharp
var emptyElements = data.Where(d => d.IsEmpty).ToList();
foreach (var d in emptyElements)
{
  data.Remove(d);
}

return emptyElements.Count;
```

We can eliminate the `foreach` by directly re-assigning `data`, as shown below.

```csharp
var dataCount = data.Count;
var data = data.Where(d => !d.IsEmpty).ToList();

return dataCount – data.Count;
```

The first algorithm is more efficient when the majority of item in `data` are empty. The second algorithm is more efficient when the majority is non-empty.