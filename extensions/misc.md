# One-off extensions

A handful of extensions that don't fit cleanly into one of the other categories &mdash; alert-bypass quit, stencil opening, selection / view rectangles, and the `IList<int>` &rarr; `Shape[]` conversion used after `DropManyU`.

To use the extensions, add a `using` directive at the top of your file:

```csharp
using VisioAutomation.Extensions;
```

## Available methods

| Method                                           | Receiver         | Purpose                                                                                                                                       |
| ------------------------------------------------ | ---------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| `Quit(force_close)`                              | `Application`    | `Application.Quit()`, but with the option to suppress the unsaved-changes prompt (sets `AlertResponse` to "No" first).                         |
| `Close(force_close)`                             | `Document`       | Same idea for closing a single document.                                                                                                      |
| `OpenStencil(filename)`                          | `Documents`      | Opens a file as a docked, read-only stencil; throws `VisioOperationException` if Visio refuses.                                                |
| `Select(shapes)`                                 | `Window`         | Replace the current selection with the given shapes.                                                                                          |
| `GetViewRect()` / `SetViewRect(rect)`            | `Window`         | View-rectangle accessors.                                                                                                                     |
| `GetWindowRect()` / `SetWindowRect(rect)`        | `Window`         | Window-rectangle accessors (uses `System.Drawing.Rectangle`).                                                                                 |
| `Shapes.GetShapesFromIDs(ids)`                   | `IVisio.Shapes`  | Turn an `IList<int>` (e.g., from `DropManyU`) back into `Shape` objects.                                                                       |

## See also

* [Application](../application.md) &mdash; `ApplicationHelper` covers a few application-level operations that aren't extension methods.
* [Stencils and masters](../stencils-and-masters.md) &mdash; uses `Documents.OpenStencil`.
* [Master dropping](dropping.md) &mdash; `DropManyU` returns `short[]` IDs that `Shapes.GetShapesFromIDs` converts back to objects.
* [Extension methods overview](../extension-methods.md).
