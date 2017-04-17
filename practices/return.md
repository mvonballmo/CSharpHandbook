# `return` statements

* Prefer multiple return statements to local variables and nesting.
  ```csharp
  if (specialTaxRateApplies)
  {
    return CalculateSpecialTaxRate();
  }

  return CalculateRegularTaxRate();
  ```
* Compose smaller methods to avoid local variables for return values. For example, the following method uses a local variable rather than multiple returns.
  ```csharp
  bool result;

  if (SomeConditionHolds())
  {
    PerformOperationsForSomeCondition();

    result = false;
  }
  else
  {
    PerformOtherOperations();

    if (SomeOtherConditionHolds())
    {
      PerformOperationsForOtherCondition();

      result = false;
    }
    else
    {
      PerformFallbackOperations();

      result = true;
    }
  }

  return result;
  ```
  This method can be rewritten to return the value instead.
  ```csharp
  if (SomeConditionHolds())
  {
    PerformOperationsForSomeCondition();

    return false;
  }

  PerformOtherOperations();

  if (SomeOtherConditionHolds())
  {
    PerformOperationsForOtherCondition();

    return false;
  }

  PerformFallbackOperations();

  return true;
  ```

* The only code that may follow the last `return` statement is the body of an exception handler.
  ```csharp
  try
  {
    // Perform operations

    return true;
  }
  catch (Exception exception)
  {
    throw new DirectoryAuthenticatorException(exception);
  }
  ```
