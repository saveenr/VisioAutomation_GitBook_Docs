# Exception types

VisioAutomation defines three exception types under `VisioAutomation.Exceptions`. They are derived from `System.Exception` (via `AutomationException`), so any plain `catch (Exception)` will also catch them &mdash; but matching the specific type lets you distinguish a library-thrown error from one bubbling up from the COM layer.

## `AutomationException`

Base class for everything the library throws on its own. Catch this when you want to handle library errors generally without catching unrelated `Exception` subclasses (`OutOfMemoryException`, `ThreadAbortException`, etc.).

```csharp
try
{
    // ... VisioAutomation calls ...
}
catch (VisioAutomation.Exceptions.AutomationException ex)
{
    System.Console.WriteLine($"Library error: {ex.Message}");
}
```

## `VisioOperationException`

Thrown when the library asked Visio to do something and Visio refused or didn't do it. For example, `Documents.OpenStencil` throws this when the stencil file can't be opened.

```csharp
try
{
    var stencil = doc.Application.Documents.OpenStencil("missing.vss");
}
catch (VisioAutomation.Exceptions.VisioOperationException ex)
{
    // The stencil could not be opened.
}
```

## `InternalAssertionException`

Thrown when the library encounters an inconsistency that "shouldn't be possible" &mdash; e.g., Visio returned a different section index than the one just requested. If you see this in production code, it's a library bug worth filing rather than a normal error condition; catching it makes sense only at the very top level of a script.

## See also

* [Application](application.md) &mdash; other application-level helpers, where some of these exceptions can fire.
