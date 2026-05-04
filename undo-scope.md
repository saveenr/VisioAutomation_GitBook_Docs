# Undo scope

Visio batches edits into named **undo scopes** so the user can undo a multi-step automation in a single Ctrl-Z. `VisioAutomation.Application.UndoScope` is a thin `IDisposable` wrapper around the `BeginUndoScope` / `EndUndoScope` API that makes the begin / end pair impossible to forget.

## Wrap a series of edits

`using` the scope opens it; disposing it closes it. By default the scope is committed (i.e., the edits are recorded as a single undoable action).

```csharp
using (var scope = new VisioAutomation.Application.UndoScope(app, "Drop and label shapes"))
{
    var page = app.ActivePage;
    var s1 = page.DrawRectangle(0, 0, 1, 1);
    s1.Text = "A";
    var s2 = page.DrawRectangle(2, 0, 3, 1);
    s2.Text = "B";
}
// After Dispose, the user sees one Undo entry: "Drop and label shapes".
```

The scope's `Name` is the label Visio shows in its Undo / Redo menu.

## Roll back instead of committing

Set `Commit = false` before `Dispose` (or before the `using` block exits) to **discard** the scope &mdash; Visio will undo the contained edits as if they never happened. Useful when an exception fires partway through a transaction-like sequence.

```csharp
using (var scope = new VisioAutomation.Application.UndoScope(app, "Risky edit"))
{
    try
    {
        // ... edits ...
    }
    catch
    {
        scope.Commit = false;     // roll back on exception
        throw;
    }
}
```

## Properties

| Property | Type | Purpose |
| --- | --- | --- |
| `Name` | `string` | The label passed to `BeginUndoScope`. Read-only. |
| `ScopeID` | `int` | The token Visio assigned to the scope. Read-only. |
| `Commit` | `bool` | If `true` (the default), `Dispose` commits the scope; if `false`, it rolls back. |

The constructor throws `ArgumentException` for an empty `name` and `ArgumentNullException` for a null `app`.

## See also

* [Application](application.md) &mdash; other application-level helpers.
