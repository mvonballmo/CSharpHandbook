# Casting

* Use a direct cast if you are sure of the type.
  ```csharp
  ((IWeapon)item).Fire();
  ```
* Use the `is`-operator when _testing_ but not _using_ the result of the cast.
  ```csharp
  return item is IWeapon;
  ```
* To use the result of the cast, use an `is`-expression in C# 7 and higher.
  ```csharp
  if (item == null) { throw new ArgumentNullException(nameof(item)); }

  if (item is IWeapon weapon)
  {
    return weapon.Fire();
  }

  return NullTurn.Default;
   ```
* Use a `switch` statement to match more than one or two patterns. Keep the argument precondition separate (even though it _could_ be the penultimate `case`).
  ```csharp
  if (item == null) { throw new ArgumentNullException(nameof(item)); }

  switch (item)
  {
    case IWeapon weapon:
      return weapon.Fire();
    case IMagic magic:
      return magic.Cast();
    default:
      return NullTurn.Default;
  }
  ```
* In C# 6 and lower, use the `as`-operator.
  ```csharp
  if (item == null) { throw new ArgumentNullException(nameof(item)); }

  var weapon = item as IWeapon;
  if (weapon != null)
  {
    return weapon.Fire();
  }

  var magic = item as IMagic;
  if (magic != null)
  {
    return magic.Cast();
  }

  return NullTurn.Default;
  ```
